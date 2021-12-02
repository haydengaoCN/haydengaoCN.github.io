#  git  usages

## git basic 

### three main stages

- modified 

  修改过的文件

- staged 

  文件被add

- committed

  文件被commit

![Working tree, staging area, and Git directory.](one_4_all.assets/areas.png)

![Git data transport commands](one_4_all.assets/MgaV9.png)

* git status

  git status 显示staged files，
  commit将会上传“Changeds to be commited"下的文件, 也就是staged  area。
  git commit -a参数会隐式的将所有tracked file上传到staged area，也就是帮你跳过了add阶段，注意这样做可能会将不需要的文件上传。

* git diff

  git diff 比较的是local files 与 staged files。

  > 比较逻辑就是 unstaged file 与 staged file比较，注意commit之后staged file会被清空，此时unstaged file 比较对象是committed file。

* git rm

  删除文件

  - `git rm` 文件从local 移除，下次commit时，repo内的该文件也会消失；
  - `git rm -f` 如果文件已经位于staged area
  - `git rm --cached` 保留local的文件 但是告诉git不再trace该文件
    `git rm log/\*.log`

* git 重命名文件

  `git mv file_from file _to`，git非常聪明的意识到你进行了重名。

  in my case, git自动将file_to增加到staged  area。

### first-time setup

* git config

  参数不同，影响的层次不同

  - all_users_all_repos

    `git config --system`   ->  /etc/gitconfig

  - one_user_all_repos

    `git config --global` ->  ~/.gitconfig  or  ~/.config/git/config

  - one_user_one_repo

    `git config --local` ->  .git/confg   current git repo

   查看设置信息以及具体来源

  ​	`git config --list --show-origin`

* set up identity

  ```bash
  git config --global user.name "John Doe"
  git config --global user.email johndoe@example.com
  git config --global credential.helper "store"  # 记住密码
  git config --global legit.uiLang en # zh
  ```

* set up editor

  ```console
  git config --global core.editor vim/emacs
  ```

### viewing commit history

```bash
git log -p -2 # diff lastest 2 commits
git log --stat # show in table
git log --oneline --graph # 
git log --since=2.weeks # time
```

### Undoing Things

#### 修改上一次的commit

```bash
git commit -m "first commit"
git add forget_file
git commit --amend # 这里可以正常接参数，比如-m
```

最终只会产生一次commit, second commit 会取代上一次commit

#### 修改staged area内的文件

比如git status 显示staged area内有两个文件file0, file1; 现在你想使file1状态变为unstaged

```bash
git reset HEAD file1 # fiel1变为unstaged,但是内容不会变。
```

#### 恢复本地的文件

local files做了修改 但是想恢复原来的样子（这里指你最近一次提交的样子）

```bash
git checkout -- file0
git reset --hard HEAD # your repository will be back to the last committed state
```

> It’s important to understand that `git checkout -- ` is a dangerous command. Any local changes you made to that file are gone — Git just replaced that file with the most **recently-committed version**. Don’t ever use this command unless you absolutely know that you don’t want those unsaved local changes.

撤销本地所有修改变动（

### git tags

* 查看tags

  ```terminal
  git tag  # get tag list
  git tag -l "v1.8*"
  ```

* 创建tags

  ```bash
  git tag -a v1.4 -m "my version 1.4" # annotated tags
  git tag v1.4-lw # light-weight tags
  git tag -a v1.2 9fceb02 # 给特定的commit增加tag
  git show v1.4 # 查看tag的详细信息
  ```

* 上传tags

  若不上传，只在本地可见

  ```bash
  git push origin v1.5 # push one tag
  git push origin --tags # push all tags
  ```

* 删除tags

  ```bash
  git tag -d v1.4-lw # 删除本地tag
  git push origin --delete <tagname> # 推送到origin
  ```

* 切换到特定的tags

  ```bash
  git checkout v2.0.0 #
  # or
  git checkout -b version2 v2.0.0 #
  ```

## git branch

### 创建 & 切换 branch

* 创建branch

  ```bash
  git branch testing # 
  git log --oneline --decorate # 查看branch, HEAD指向当前的branch(的最新一个commit)
  ```

* 切换branch

  ```bash
  git checkout testing # 此时HEAD已经指向test
  git log --oneline --decorate --graph --all # 显示branch
  ```
  
* `git branch -b testing`  等价于先创建分支然后再切换到新建的分支。

* 推送到远程分支

  ```bash
  git push origin testing
  git push -u origin testing # first push
  ```

* 删除branch

  ```bash
  git branch -d testing  # delete local branch
  git push <remote_name> --delete <branch_name> # delete remote branch
  ```

### 合并 branch

#### merge directly

```bash
git checkout master # 切换回master
git merge iss53 # 融合，此时master前进到testing的最新commit
git branch -d iss53  # 删除分支
```

* git mergetool

  指定合并工具`git mergetool --tool=vimdiff`

* 只要修改最后的文件即可

  ![Three snapshots used in a typical merge.](/Users/hayden/Documents/basic/one_4_all.assets/basic-merging-1-20200612194515244.png)

  ![A merge commit.](/Users/hayden/Documents/basic/one_4_all.assets/basic-merging-2.png)
#### rebase

  ```bash
git rebase master # now in experiemnt
git checkout master
git merge experiment
  ```

![Merging to integrate diverged work history.](one_4_all.assets/basic-rebase-2.png)

![Rebasing the change introduced in `C4` onto `C3`.](one_4_all.assets/basic-rebase-3.png)

![Fast-forwarding the `master` branch.](one_4_all.assets/basic-rebase-4.png)

### 放弃merge

```bash
git checkout master
git merge topic
git revert -m 1 HEAD # 放弃topic的内容， 1指的是master, 2 指的是topic
```



### branch management

```bash
git branch  # 不加参数会列出所有的branch, *表示当前的branch
git branch -v # 列出每个branch的最新的commit
git branch --merged # 已经merge的分支，-d 可直接删掉
git branch --no-merged # 尚未merge的分支，-D 强制删除
```

## git default/short

* remote + branch

  ```bash
  git checkout -b testing
  git checkput -b testing origin/master
  
  git pull
  git pull remote/current_branch
  
  git branch -vv 
  git remote show origin # these two lines show remote branch you are tracking
  ```




* short

  ```bash
  git show HEAD^ # HEAD's parent
  git show HEAD^2 # HEAD's second parent
  git show HEAD~ # like ^
  git show HEAD~2 # or
  git show HEAD~~
  ```

## git tools

### 查看push影响的commit

```bash
git log origin/master..HEAD # 若是push到origin master的话则会被提交的commits
```

### git clean

```bash
git clean -n # 查看会被清理的文件
git clean -f # 清理
```



### git stash

```bash
git stash # Saved working directory and index state  此时staged zone清空
git stash push # stash only modified and staged tracked files
git stash --keep-index # 功能一样，但是保留index
git stash -u # stask un-tracked files

git stash list

git stash apply # git stash apply stash@{0} local file恢复，但是未stage
git stash apply stash@{1}
git stash apply --index # reapply the staged changes

git stash pop # apply & drop stash@{0}
git stash drop stash@{1} # 

git stash branch <new branchname> # creates a new branch; 
# checks out the commit you were on when you stashed your work; 
# creates a new branch checks out the commit you were on when you stashed your work  
# reapplies your work there, and then drops the stash if it applies successfully
```



### git diff

[head指的是最近的commit](https://stackoverflow.com/questions/1587846/how-do-i-show-the-changes-which-have-been-staged/1587877)

![Simple Git diffs](one_4_all.assets/tVHYO.png)

* **git diff**

  Shows the changes between the working directory and the index. This shows what has been changed, but is not staged for a commit.

* **git diff --cached**

  Shows the changes between the index and the HEAD (which is the last commit on this branch). This shows what has been added to the index and staged for a commit.

* **git diff HEAD**

  Shows all the changes between the working directory and HEAD (which includes changes in the index). This shows all the changes since the last commit, whether or not they have been staged for commit or not.

