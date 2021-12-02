# C++ code snippet

## utf-8编码

```C++
#include <iostream>
#include <string>

bool isequal(const std::string& lh, const std::string& rh) {
  if (lh.size() != rh.size()) {
    return false;
  }
  for (auto idx = 0; idx < lh.size(); ++idx) {
    if (lh[idx] != rh[idx]) {
      return false;
    }
  }
  return true;
}

int main() {
  // std::string x = "\351\201\207";
  std::string x = "\351\201\207\351\276\231_02";
  std::string y = "遇";
  std::cout << "x:" << x << ",size:" << x.size() << std::endl;
  std::cout << "y:" << y << ",size:" << y.size() << std::endl;
  std::cout << "x == y ? " << isequal(x, y) << std::endl;

  return 0;
}
```

## 单例singleton

[boost实现](https://www.boost.org/doc/libs/1_47_0/boost/pool/detail/singleton.hpp)

```C++
// T must be: no-throw default constructible and no-throw destructible
template <typename T>
struct singleton_default
{
  private:
    struct object_creator
    {
      // This constructor does nothing more than ensure that instance()
      //  is called before main() begins, thus creating the static
      //  T object before multithreading race issues can come up.
      object_creator() { singleton_default<T>::instance(); }
      inline void do_nothing() const { }
    };
    static object_creator create_object;

    singleton_default();

  public:
    typedef T object_type;

    // If, at any point (in user code), singleton_default<T>::instance()
    //  is called, then the following function is instantiated.
    static object_type & instance()
    {
      // This is the object that we return a reference to.
      // It is guaranteed to be created before main() begins because of
      //  the next line.
      static object_type obj;

      // The following line does nothing else than force the instantiation
      //  of singleton_default<T>::create_object, whose constructor is
      //  called before main() begins.
      create_object.do_nothing();

      return obj;
    }
};
template <class T>
typename singleton_default<T>::object_creator singleton_default<T>::create_object;
```

1）如何保证在进入main函数之前，该类已经实例化？(好处是避免多线程竞争)

* static local变量的初始化时机是首次调用这个static成员函数的时候，这意味着可能在main之前也可能在main之后。
* 这需要利用一个模板类的static成员变量的生命周期来保证该“static local变量”的初始化在main之前；

2）static成员变量如果没有在static成员函数当中被引用过，那么它自己的初始化时间也是不确定的（保证在main之前但初始化顺序不确定，甚至可能被优化掉）
*  所以需要在某个static成员函数当中引用那么一下，来确保该“static local变量”的初始化一定被执行；

3）`inline`关键字能去掉嘛？

* 避免激进优化下直接把create_object.do_nothing()作为空函数优化去掉，同时又保证真的激进优化的时候不会增加冗余代码。

## 线程条件变量同步

```c++
#include <iostream>
#include <condition_variable>
#include <mutex>
#include <thread>

std::mutex mutex_;
std::condition_variable condVar;

bool dataReady;

void doTheWork(){
  std::cout << "Processing shared data." << std::endl;
}

void waitingForWork(){
    std::cout << "Worker: Waiting for work." << std::endl;

    std::unique_lock<std::mutex> lck(mutex_);
    condVar.wait(lck,[]{return dataReady;});  // going on when receving signal or lambda returning true
    doTheWork();
    std::cout << "Work done." << std::endl;
}

void setDataReady(){
    std::lock_guard<std::mutex> lck(mutex_);
    dataReady=true;
    std::cout << "Sender: Data is ready."  << std::endl;
    condVar.notify_one();
}

int main(){

  std::cout << std::endl;

  std::thread t1(waitingForWork);
  std::thread t2(setDataReady);

  t1.join();
  t2.join();

  std::cout << std::endl; 
}
```

[conditional_variable_sync](https://www.modernescpp.com/index.php/condition-variables)

1. unique_lock v.s. lock_guard

   The difference is that you can lock and unlock a `std::unique_lock`. `std::lock_guard` will be locked only once on construction and unlocked on destruction.[locks](https://stackoverflow.com/questions/20516773/stdunique-lockstdmutex-or-stdlock-guardstdmutex)

   总之，`unique_lock`可以中途释放，比如代码19行，wait没准备好的话，锁就被释放了；

   现在一般使用`scoped_lock`取代`lock_guard`;

## std:make_shared & std::make_unique

一直没记住这个语法，索性抄写下来

```C++
#include <memory>

/*
template< class T, class... Args >
shared_ptr<T> make_shared( Args&&... args );
*/
class MyObject;
auto shptr = std::make_shared<MyObject>();  //  () depends for construction

/*
template< class T, class... Args >
unique_ptr<T> make_unique( Args&&... args );
*/
auto unptr = std::make_unique<MyObject>(); // also () depends for construction
```

## chrono literals && timer

```C++
#include <chrono>
#include <iostream>
#include <thread>

using std::literals::chrono_literals::operator""s;

int main() {
  auto start = std::chrono::steady_clock::now();
  {
    using std::literals::chrono_literals::operator""s;
		std::this_thread::sleep_for(1s);
  }
  auto end = std::chrono::steady_clock::now();
  
  std::cout << "Elapsed time in nanoseconds: "
            << std::chrono::duration_cast<std::chrono::nanoseconds>(end - start).count()
            << " ns" << std::endl;
 
  std::cout << "Elapsed time in microseconds: "
            << std::chrono::duration_cast<std::chrono::microseconds>(end - start).count()
            << " µs" << std::endl;
 
  std::cout << "Elapsed time in milliseconds: "
            << std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count()
            << " ms" << std::endl;
 
  std::cout << "Elapsed time in seconds: "
            << std::chrono::duration_cast<std::chrono::seconds>(end - start).count()
            << " sec";
  return 0;
}
```

## 输出/读取文件

```C++
#include <string>
#include <fstream>
#include <iostream>

void Write2File() {
  std::string file_name = "./greeting.txt"
  std::ofstream out(file_name, std::ios::trunc | std::ios::binary);
  if (out.is_open()) {
    out << "hello to disk." << std::endl;
    out.close();
    std::cout << "Data dump to:" << file_name;
  } 
}

void ReadFromFile() {
  std::string file_name = "./greeting.txt";
  std::ifstream in(file_name, std::ifstream::in | std::ifstream::binary);
  if (in.is_open()) {
    std::string line;
    while (std::getline(in, line)) {  // reading line by line
      std::cout << line << std::endl;
    }
  }
}

int main() {
  return 0;
}
```

## 优先队列/堆

```C++
#include <functional>
#include <queue>
#include <vector>
#include <iostream>
 
template<typename T>
void print_queue(T q) { // NB: pass by value so the print uses a copy
    while(!q.empty()) {
        std::cout << q.top() << ' ';
        q.pop();
    }
    std::cout << '\n';
}
 
int main() {
    std::priority_queue<int> q;  // 默认是最大堆
 
    const auto data = {1,8,5,6,3,4,0,9,7,2};
 
    for(int n : data)
        q.push(n);
 
    print_queue(q);
 
    std::priority_queue<int, std::vector<int>, std::greater<int>>  // 最小堆
        q2(data.begin(), data.end());
 
    print_queue(q2);
 
    // Using lambda to compare elements.
    auto cmp = [](int left, int right) { return (left ^ 1) < (right ^ 1); };
    std::priority_queue<int, std::vector<int>, decltype(cmp)> q3(cmp);
 
    for(int n : data)
        q3.push(n);
 
    print_queue(q3);
}
```

## protobuf msg

```C++
void FsBusiMsg::ReqMsgProfile() {
  if (!FLAGS_enable_msg_profile) {
    LOG(INFO) << "[msg profile]No need msg profiling.";
    return;
  }
  static constexpr int msg_profile_report_ids[] =
      {1895299, 1895300, 1895301, 1895302, 1895303,
       1895304, 1895305, 1895306, 1895307, 1895308};
  const auto desc = m_request.GetDescriptor();
  const auto refl = m_request.GetReflection();
  int field_nums = desc->field_count();
  int tot_msg_sizes = 0;
  LOG(INFO) << "[msg profile]Has " << field_nums << " fields.";
  for (auto idx = 0; idx < field_nums; ++idx) {
    const auto field = desc->field(idx);
    const auto& msg = refl->GetMessage(m_request, field);
    int cur_msg_size = 0;
    if (!field->is_repeated()) {
      cur_msg_size = msg.ByteSize();
    } else {
      for (const auto& v : msg) {
        cur_msg_size += v.ByteSize();
      }
    }
    Attr_API(msg_profile_report_ids[idx % 10], cur_msg_size);
    VLOG(1) << "[msg profile]Field " << field->name().c_str()
            << " has byte sizes:" << cur_msg_size;
    tot_msg_sizes += cur_msg_size;
  }
  Attr_API(1895310, tot_msg_sizes);
  LOG(INFO) << "[msg profile]Field nums:" << field_nums
            << ", total byte sizes:" << tot_msg_sizes
            << ", msg size:" << m_request.ByteSize();
}
```

