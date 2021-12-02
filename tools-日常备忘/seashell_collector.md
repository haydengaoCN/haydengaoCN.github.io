# seashell collector

## grep

* 指定搜索的文件类型

  ```bash
  grep --include \*.cc --include \*.h .
  ```

* 反选

  ```bash
  cat log | grep "male" | grep -v "female"
  ```

* Ignore binary

  ```bash
  grep -I "hello" .
  ```

* 多个关键字

  ```bash
  grep -E '123|abc' filename  # 找出文件（filename）中包含123或者包含abc的行
  grep pattern1 files | grep pattern2 # 显示既匹配 pattern1 又匹配 pattern2 的行
  ```


* 显示前后几行

  ```bash
  grep -C 5 foo file # 显示file文件里匹配foo字串那行以及上下5行
  grep -B 5 foo file # 显示foo及前5行
  grep -A 5 foo file # 显示foo及后5行
  ```

  

## vim

* highlight multiple words

  ```bash
  /\vword1|word2|word3
  /word1\|word2\|word3
  ```

## scp

```bash
# copy display_server from devcloud to local; -P set port
# scp -v -C src_file dst_file; just lick cp.
scp -v -P 36000 root@9.134.151.228:/root/Desktop/main/build64_release/display/display_processor_test .
md5sum display_server # check md5
```

## 磁盘容量

* 查看文件大小

  ```terminal
  du -h -d 1
  ```

  

## gcc

* gcc include path

  ```bash
  C_INCLUDE_PATH # for c head files
  CPLUS_INCLUDE_PATH # for c++ head files
  CPATH # for both head files
  gcc -xc -E -v - # default search path for c
  gcc -xc++ -E -v - # default search path for c++
  ```

* ld search path

  ```bash
  LD_LIBRARY_PATH # runtime
  LIBRARY_PATH # compile
  ld --verbose | grep SEARCH_DIR | tr -s ' ;' \\012 # show ld search path
  ```

* 指定message语言

  ```bash
  export LANG=en
  ```

## 查看c/c++静态库内容

```bash
nm -C libschnoeck.a | less  # add -C option if C++ content is included
```



## cmake

* 指定 gcc/g++ 路径

```bash
CC=gcc CXX=g++ cmake ..
```

## Check CentOS/REDHAT Version

`hostnamectl`

[Check CentOS Version]([https://www.thegeekdiary.com/how-to-check-centos-version/#:~:text=You%20can%20find%20out%20which,details%20of%20the%20uname%20command.&text=You%20can%20also%20Verify%20the,issue%20with%20the%20installed%20kernel.](https://www.thegeekdiary.com/how-to-check-centos-version/#:~:text=You can find out which,details of the uname command.&text=You can also Verify the,issue with the installed kernel.))

### Check the CentOS/RHEL OS Update Level

```bash
cat /etc/centos-release
cat /etc/os-release
cat /etc/redhat-release
cat /etc/system-release
```

### Check the Running Kernel version

--check centos kernel version and architecture

```bash
uname -s -r # 
```

## vscode

* 关闭右边缩略图

  点击文件-首选项-设置,搜索"editor.minimap.enabled",默认值为打钩,去掉即可；

## 宏macros

* "##''表示拼接
* g++ -E src.cc 会进行宏替换

## rm

* 反向删除

  ```bash
  shopt -s extglob
  rm -r !(file1)  # keep file1
  rm -r !(file1 | file2)  # keep file1, file2
  ```

  

## history

* with timestamp

  ```bash
  HISTTIMEFORMAT="%d/%m/%y %T "
  history
  ```

* !num

* !!

  执行上一次命令

## glog

* `v` (`int`, default=0)

  Show all `VLOG(m)` messages for `m` less or equal the value of this flag. 比m低的会出；

* `minloglevel` (`int`, default=0, which is `INFO`)

  Log messages at or above this level. Again, the numbers of severity levels `INFO`, `WARNING`, `ERROR`, and `FATAL` are 0, 1, 2, and 3, respectively. 比m高的会出；

## MAC

* terminal给tab命名

  shift + command + i
  
* 强制退出 App

  同时按住三个按键：Option、Command 和 Esc (Escape) 键。或者，从屏幕左上角的苹果菜单  中选取“强制退出”。（这类似于在 PC 上按下 Control-Alt-Delete。）

## JSON

* support type

  | **Data Type** | **Description**                                              |
  | ------------- | ------------------------------------------------------------ |
  | Number        | It includes real number, integer or a floating number        |
  | String        | It consists of any text or Unicode double-quoted with backslash escapement |
  | Boolean       | The Boolean data type represents either True or False values |
  | Null          | The Null value denotes that the associated variable doesn't have any value |
  | Object        | It is a collection of key-value pairs and always separated by a comma and enclosed in curly brackets. |
  | Array         | It is an ordered sequence of values separated.               |

* Array

  - It is an ordered collection of values.  **有序**
  - You should use an array when the key names are sequential integers. **索引的时候用整型**
  - It should be enclosed inside square brackets which should be separated by ',' (comma) **方括号括起，内部逗号分隔**

  ```json
  {
     "eBooks":[
        {
           "language":"Pascal",
           "edition":"third"
        },
        {
           "language":"Python",
           "edition":"four"
        },
        {
           "language":"SQL",
           "edition":"second"
        }
     ]
  }
  ```

* Object

  - An object should be enclosed in curly braces, **花括号分隔**
  - It should be an unordered set of name or value pairs.**无序的KV对**
  - Name should be followed by ":" (colon)and the name/value pairs need to be separated using "," (comma).
  - You can use it when key names are arbitrary strings.

  ```json
  {
  "id": 110,	‬‬‬‬‬‬‬‬‬‬‬‬‬‬‬‬‬‬‬‬‬‬
  "language": "Python",
  "price": 1900,
  }
  ```

  

## screen

  [screen](https://linuxize.com/post/how-to-use-linux-screen/)

  ```bash
  screen # 创建一个匿名screen
    screen -r # 连回来
  ```

  

* start named session
   ```bash
   screen -S session_name
   ```

* detach 

  ```bash
  Ctrl+a d
  ```

* Reattach to a Linux Screen

   ```bash
   screen -r
   screen -ls
   ```

   

