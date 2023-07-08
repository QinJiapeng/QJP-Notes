# Git

## Git概述

### 版本系统

SVN 是集中式版本控制系统，版本库是集中放在中央服务器的，而开发人员工作的时候，用的都是自己的电脑，所以首先要从中央服务器下载最新的版本，然后开发，开发完后，需要把自己开发的代码提交到中央服务器。

集中式版本控制工具缺点：服务器单点故障、容错性差

Git 是分布式版本控制系统（Distributed Version Control System，简称 DVCS） ，分为两种类型的仓库：

本地仓库和远程仓库：

* 本地仓库：是在开发人员自己电脑上的 Git 仓库		
* 远程仓库：是在远程服务器上的 Git 仓库



***



### 工作流程

1．从远程仓库中克隆代码到本地仓库

2．从本地仓库中 checkout 代码然后进行代码修改

3．在提交前先将代码提交到**暂存区**

4．提交到本地仓库。本地仓库中保存修改的各个历史版本

5．修改完成后，需要和团队成员共享代码时，将代码 push 到远程仓库



### Git安装

下载地址： <https://git-scm.com/download>



### 代码托管

Git 中存在两种类型的仓库，即本地仓库和远程仓库。那么我们如何搭建Git远程仓库呢？我们可以借助互联网上提供的一些代码托管服务来实现，其中比较常用的有 GitHub、码云、GitLab 等。

GitHub（地址：https://github.com/）是一个面向开源及私有软件项目的托管平台，因为只支持 Git 作为唯一的版本库格式进行托管，故名 GitHub

码云（地址： https://gitee.com/）是国内的一个代码托管平台，由于服务器在国内，所以相比于 GitHub，码云速度会更快

GitLab（地址： https://about.gitlab.com/ ）是一个用于仓库管理系统的开源项目，使用 Git 作为代码管理工具，并在此基础上搭建起来的 web 服务





***



## 环境配置

安装 Git 后首先要设置用户名称和 email 地址，因为每次 Git 提交都会使用该用户信息，此信息和注册的代码托管平台的信息无关

设置用户信息：

* git config --global user.name “Seazean”
* git config --global user.email "imseazean@gmail.com"  //用户名和邮箱可以随意填写，不会校对

查看配置信息：

* git config --list
* git config user.name

通过上面的命令设置的信息会保存在用户目录下 /.gitconfig 文件中



***



## 本地仓库

### 获取仓库

* **本地仓库初始化**

  1. 在电脑的任意位置创建一个空目录（例如 repo1）作为本地 Git 仓库

  2. 进入这个目录中，点击右键打开 Git bash 窗口

  3. 执行命令 **git init**

     如果在当前目录中看到 .git 文件夹（此文件夹为隐藏文件夹）则说明 Git 仓库创建成功

* **远程仓库克隆**
  通过 Git 提供的命令从远程仓库进行克隆，将远程仓库克隆到本地

  命令：git clone 远程 Git 仓库地址（HTTPS 或者 SSH）

* 生成 SSH 公钥步骤

  * 设置账户
  * cd ~/.ssh（查看是否生成过 SSH 公钥）user 目录下
  * 生成 SSH 公钥：`ssh-keygen -t rsa -C "email"`
    * -t 指定密钥类型，默认是 rsa ，可以省略
    * -C 设置注释文字，比如邮箱
    * -f 指定密钥文件存储文件名
  * 查看命令：cat ~/.ssh/id_rsa.pub
  * 公钥测试命令：ssh -T git@github.com





***






### 工作过程

![](https://img.jiapeng.store/img/202307081712456.png)

版本库：.git 隐藏文件夹就是版本库，版本库中存储了很多配置信息、日志信息和文件版本信息等

工作目录（工作区）：包含 .git 文件夹的目录就是工作目录，主要用于存放开发的代码

暂存区：.git 文件夹中有很多文件，其中有一个 index 文件就是暂存区，也可以叫做 stage，暂存区是一个临时保存修改文件的地方

![](https://img.jiapeng.store/img/202307081712458.png)



***



### 文件操作

#### 常用命令

| 命令                    | 作用                                                         |
| ----------------------- | ------------------------------------------------------------ |
| git status              | 查看 git 状态 （文件是否进行了添加、提交操作）               |
| git add filename        | 添加，将指定文件添加到暂存区                                 |
| git commit -m 'message' | 提交，将暂存区文件提交到本地仓库，删除暂存区的该文件         |
| git commit --amend      | 修改 commit 的 message                                       |
| git rm filename         | 删除，删除工作区的文件，不是仓库，需要提交                   |
| git mv filename         | 移动或重命名工作区文件                                       |
| git reset filename      | 使用当前分支上的修改覆盖暂存区，**将暂存区的文件取消暂存**   |
| git checkout filename   | 使用暂存区的修改覆盖工作目录，用来撤销本次修改(危险)         |
| git log                 | 查看日志（ git 提交的历史日志）                              |
| git reflog              | 可以查看所有分支的所有操作记录（包括已经被删除的 commit 记录的操作） |

**其他指令**：可以跳过暂存区域直接从分支中取出修改，或者直接提交修改到分支中

* git commit -a 直接把所有文件的修改添加到暂存区然后执行提交
* git checkout HEAD -- files 取出最后一次修改，可以用来进行回滚操作



***



#### 文件状态

* Git 工作目录下的文件存在两种状态：
  * untracked 未跟踪（未被纳入版本控制）
  * tracked 已跟踪（被纳入版本控制）
    * Unmodified 未修改状态
    * Modified 已修改状态
    * Staged 已暂存状态

* 查看文件状态：文件的状态会随着我们执行 Git 的命令发生变化
  * git status 查看文件状态
  * git status –s 查看更简洁的文件状态



***



#### 文件忽略

一般我们总会有些文件无需纳入Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以在工作目录中创建一个名为 .gitignore 的文件（文件名称固定），列出要忽略的文件模式。下面是一个示例：

```sh
# no .a files
*.a
# but do track lib.a, even though you're ignoring .a files above
!lib.a
# only ignore the TODO file in the current directory, not subdir/TODO
/TODO
# ignore all files in the build/ directory
build/
# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt
# ignore all .pdf files in the doc/ directory
doc/**/*.pdf
```



***





## 远程仓库

### 工作流程

Git 有四个工作空间的概念，分别为 工作空间、暂存区、本地仓库、远程仓库。

pull = fetch + merge

fetch 是从远程仓库更新到本地仓库，pull是从远程仓库直接更新到工作空间中

![](https://img.jiapeng.store/img/202307081712459.png)





***



### 查看仓库

git remote：显示所有远程仓库的简写  

git remote -v：显示所有远程仓库  

git remote show <shortname>：显示某个远程仓库的详细信息



### 添加仓库

git remote add <shortname><url>：添加一个新的远程仓库，并指定一个可以引用的简写



### 克隆仓库

git clone <url>(HTTPS or SSH)：克隆远程仓库

Git 克隆的是该 Git 仓库服务器上的几乎所有数据（包括日志信息、历史记录等），而不仅仅是复制工作所需要的文件，当你执行 git clone 命令的时候，默认配置下远程 Git 仓库中的每一个文件的每一个版本都将被拉取下来。



### 删除仓库

git remote rm <shortname>：移除远程仓库，从本地移除远程仓库的记录，并不会影响到远程仓库



### 拉取仓库

git fetch  <shortname>：从远程仓库获取最新版本到本地仓库，不会自动 merge  

git pull <shortname> <branchname>：从远程仓库获取最新版本并 merge 到本地仓库

注意：如果当前本地仓库不是从远程仓库克隆，而是本地创建的仓库，并且**仓库中存在文件**，此时再从远程仓库拉取文件的时候会报错（fatal: refusing to merge unrelated histories ），解决此问题可以在 git pull 命令后加入参数 --allow-unrelated-histories



### 推送仓库

git push <shortname><branchname>：上传本地指定分支到远程仓库





***



## 版本管理

![](https://img.jiapeng.store/img/202307081712460.png)

命令：git reset --hard 版本唯一索引值





***



## 分支管理

### 查看分支

git branch：列出所有本地分支  

git branch -r：列出所有远程分支  

git branch -a：列出所有本地分支和远程分支



### 创建分支

git branch  branch-name：新建一个分支，但依然停留在当前分支

git checkout -b branch-name：新建一个分支，并切换到该分支



### 推送分支 

git push origin branch-name：推送到远程仓库，origin 是引用名



### 切换分支

git checkout branch-name：切换到 branch-name 分支



### 合并分支

git merge branch-name：合并指定分支到当前分支

有时候合并操作不会如此顺利。 如果你在两个不同的分支中，对同一个文件的同一个部分进行了不同的修改，Git 就没办法合并它们，同时会提示文件冲突。此时需要我们打开冲突的文件并修复冲突内容，最后执行 git add 命令来标识冲突已解决

​	![](https://img.jiapeng.store/img/202307081712461.png)



### 删除分支

git branch -d branch-name：删除分支

git push origin –d branch-name：删除远程仓库中的分支   （origin 是引用名）

如果要删除的分支中进行了开发动作，此时执行删除命令并不会删除分支，如果坚持要删除此分支，可以将命令中的 -d 参数改为 -D：git branch -D branch-name



***



## 标签管理

### 查看标签

git tag：列出所有 tag

git show tag-name：查看 tag 详细信息

标签作用：在开发的一些关键时期，使用标签来记录这些关键时刻，保存快照，例如发布版本、有重大修改、升级的时候、会使用标签记录这些时刻，来永久标记项目中的关键历史时刻



### 新建标签

git tag tag-name：新建标签，如（git tag v1.0.1）



### 推送标签

git push [remotename] [tagname]：推送到远程仓库

git push [remotename] --tags：推送所有的标签



### 切换标签

git checkout tag-name：切换标签



### 删除标签

git tag -d tag-name：删除本地标签

git push origin :refs/tags/ tag-name：删除远程标签



***



## IDEA操作

### 环境配置

File → Settings 打开设置窗口，找到 Version Control 下的 git 选项

选择 git 的安装目录后可以点击 Test 按钮测试是否正确配置：D:\Program Files\Git\cmd\git.exe



### 创建仓库

1、VCS → Import into Version Control → Create Git Repository

2、选择工程所在的目录,这样就创建好本地仓库了

3、点击git后边的对勾,将当前项目代码提交到本地仓库

​	注意: 项目中的配置文件不需要提交到本地仓库中,提交时,忽略掉即可



### 文件操作

右键项目名打开菜单 Git → Add → commit



### 版本管理

* 版本对比
  	![](https://img.jiapeng.store/img/202307081712462.png)

* 版本切换方式一：控制台 Version Control → Log → 右键 Reset Current Branch → Reset，这种切换会抛弃原来的提交记录
  	![](https://img.jiapeng.store/img/202307081712463.png)
* 版本切换方式二：控制台 Version Control → Log → Revert Commit → Merge → 处理代码 → commit，这种切换会当成一个新的提交记录，之前的提交记录也都保留
  ![](https://img.jiapeng.store/img/202307081712464.png)

​           ![](https://img.jiapeng.store/img/202307081712465.png)





***



### 分支管理

* 创建分支：VCS → Git → Branches → New Branch → 给分支起名字 → ok
* 切换分支：idea 右下角 Git → 选择要切换的分支 → checkout
* 合并分支：VCS → Git → Merge changes → 选择要合并的分支 → merge
* 删除分支：idea 右下角 → 选中要删除的分支 → Delete



***



### 推送仓库

1. VCS → Git → Push → 点击 master Define remote
2. 将远程仓库的 url 路径复制过来 → Push
   ![](https://img.jiapeng.store/img/202307081712466.png)



***



### 克隆仓库

File → Close Project → Checkout from Version Control → Git → 指定远程仓库的路径 → 指定本地存放的路径 → clone

![](https://img.jiapeng.store/img/202307081712467.png)





