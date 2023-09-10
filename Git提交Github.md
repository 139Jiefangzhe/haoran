# Git教程

#### Git 工作区、暂存区和版本库

------

##### 基本概念

###### 我们先来理解下 Git 工作区、暂存区和版本库概念：

- ###### **工作区：**就是你在电脑里能看到的目录。

- ###### **暂存区：**英文叫 stage 或 index。一般存放在 **.git** 目录下的 index 文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。

- ###### **版本库：**工作区有一个隐藏目录 **.git**，这个不算工作区，而是 Git 的版本库。

![img](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910114016328-1184197107.jpg)

```
图中左侧为工作区，右侧为版本库。在版本库中标记为 "index" 的区域是暂存区（stage/index），标记为 "master" 的是 master 分支所代表的目录树。

图中我们可以看出此时 "HEAD" 实际是指向 master 分支的一个"游标"。所以图示的命令中出现 HEAD 的地方可以用 master 来替换。

图中的 objects 标识的区域为 Git 的对象库，实际位于 ".git/objects" 目录下，里面包含了创建的各种对象及内容。

当对工作区修改（或新增）的文件执行 git add 命令时，暂存区的目录树被更新，同时工作区修改（或新增）的文件内容被写入到对象库中的一个新的对象中，而该对象的ID被记录在暂存区的文件索引中。

当执行提交操作（git commit）时，暂存区的目录树写到版本库（对象库）中，master 分支会做相应的更新。即 master 指向的目录树就是提交时暂存区的目录树。

当执行 git reset HEAD 命令时，暂存区的目录树会被重写，被 master 分支指向的目录树所替换，但是工作区不受影响。

当执行 git rm --cached <file> 命令时，会直接从暂存区删除文件，工作区则不做出改变。

当执行 git checkout . 或者 git checkout -- <file> 命令时，会用暂存区全部或指定的文件替换工作区的文件。这个操作很危险，会清除工作区中未添加到暂存区中的改动。

当执行 git checkout HEAD . 或者 git checkout HEAD <file> 命令时，会用 HEAD 指向的 master 分支中的全部或者部分文件替换暂存区和以及工作区中的文件。这个命令也是极具危险性的，因为不但会清除工作区中未提交的改动，也会清除暂存区中未提交的改动。
```

=============================================================================

## git使用

```
git init 初始化仓库
git init github 指定仓库初始化
初始化后，会在 github 目录下会出现一个名为 .git 的目录，所有 Git 需要的数据和资源都存放在这个目录中。
.git为隐藏文件
git add *.c
git add readme.md
git add .  将目录下所有文件保存至暂存区
git commit -m ”提交至git仓库“
注： 在 Linux 系统中，commit 信息使用单引号 '，Windows 系统，commit 信息使用双引号 "。
所以在 git bash 中 git commit -m '提交说明' 这样是可以的，在 Windows 命令行中就要使用双引号 git commit -m "提交说明"。
git status 查看修改或删除文件
git log 查看记录
```

## git clone

```
 git clone git://github.com/schacon/grit.git
 如果要自己定义要新建的项目目录名称，可以在上面的命令末尾指定新的名字：
 git clone git://github.com/schacon/grit.git   mygrit
```

#### 配置

```
git config

```

![img](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910114041391-1103744573.png)

```
git config -e    # 针对当前仓库 
git config -e --global   # 针对系统上所有仓库
设置提交代码时的用户信息：

git config --global user.name "runoob"
git config --global user.email test@runoob.com
```

![img](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910114103007-1323694774.png)

### ![img](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910114129230-2141313627.png)

[Git 分支管理 | 菜鸟教程 (runoob.com)](https://www.runoob.com/git/git-branch.html)

!![img](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910114145242-1128940428.png)

```
创建分支
git branch (branchname)
切换分支命令:
git checkout (branchname) 切换时git用该分支最后的快照替换工作目录
git branch 列出分支 无参数列出在本地的分支

```

![img](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910114200199-71266798.png)

```
 git init 的时候，默认情况下 Git 就会为你创建 master 分支。
```

```
git branch -d branchname 删除分支
git merge test 将test分支合并到主分支


```

### git 标签

```
-a 选项意为"创建一个带注解的标签"。 不用 -a 选项也可以执行的，但它不会记录这标签是啥时候打的，谁打的，也不会让你添加个标签的注解。 我推荐一直创建带注解的标签。

$ git tag -a v1.0 
```

## git和GitHub

```
git remote add [shortname] [url]
添加一个新的远程仓库，可以指定一个简单的名字，以便将来引用
```

```
Git 仓库和 GitHub 仓库之间的传输是通过SSH加密的，所以我们需要配置验证信息
ssh-keygen -t rsa -C "youremail@example.com"
```

```
$ git remote -v
origin    git@github.com:tianqixin/runoob-git-test.git (fetch) 拉取
origin    git@github.com:tianqixin/runoob-git-test.git (push)  上传

# 添加仓库 origin2
$ git remote add origin2 git@github.com:tianqixin/runoob-git-test.git

$ git remote -v
origin    git@github.com:tianqixin/runoob-git-test.git (fetch)
origin    git@github.com:tianqixin/runoob-git-test.git (push)
origin2    git@github.com:tianqixin/runoob-git-test.git (fetch)
origin2    git@github.com:tianqixin/runoob-git-test.git (push)

# 删除仓库 origin2
$ git remote rm origin2
$ git remote -v
origin    git@github.com:tianqixin/runoob-git-test.git (fetch)
origin    git@github.com:tianqixin/runoob-git-test.git (push)
```

## 本地git推送到GitHub命令

```
$ mkdir runoob-git-test                     # 创建测试目录
$ cd runoob-git-test/                       # 进入测试目录
$ echo "# 菜鸟教程 Git 测试" >> README.md     # 创建 README.md 文件并写入内容
$ ls                                        # 查看目录下的文件
README
$ git init                                  # 初始化
$ git add README.md  # 添加文件
  git add README.md  # 添加目录下所有文件 
$ git commit -m "添加 README.md 文件"        # 提交并备注信息
[master (root-commit) 0205aab] 添加 README.md 文件
 1 file changed, 1 insertion(+)
 create mode 100644 README.md

# 提交到 Github
$ git remote add origin（自定义名字） git@github.com:tianqixin/runoob-git-test.git
$ git push -u origin master

```

