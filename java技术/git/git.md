# mygit

### Git是什么？

Git是目前世界上最先进的分布式版本控制系统（没有之一）。



Linus在1991年创建了开源的Linux，从此，Linux系统不断发展，已经成为最大的服务器系统软件了。

Linus花了两周时间自己用C写了一个分布式版本控制系统，这就是Git！



### 集中式版本控制系统

版本库是集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，再把自己的活推送给中央服务器。中央服务器就好比是一个图书馆，你要改一本书，必须先从图书馆借出来，然后回到家自己改，改完了，再放回图书馆。



### 分布式版本控制系统

根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，这样，你工作的时候，就不需要联网了，因为版本库就在你自己的电脑上。既然每个人电脑上都有一个完整的版本库，那多个人如何协作呢？比方说你在自己电脑上改了文件A，你的同事也在他的电脑上改了文件A，这时，你们俩之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。



安装git后，配置机器信息

```
# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

注意`git config`命令的`--global`参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。

### 配置

```
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]
```

### 创建本地仓库

```
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]

# 查看隐藏文件
$ ls -ah
```

### 提交

```
# 添加文件到暂存区
$ git add [file1][file2]
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 提交到暂存区
$ git commit -m "message"

# 提交到远程仓库
$ git push

# 查看仓库状态
$ git status

# 查看修改的内容
$ git diff

# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2]
```

### 回退版本

```
# 查看git历史
$ git log
$ git log --pretty=oneline

# 回退版本
$ git reset --hard HEAD^
$ git reset --hard HEAD^^
$ git reset --hard HEAD~n

# 记录你的每一次命令
$ git reflog
$ git reset --hard commit_id
```

### 撤销内容

```
# 丢弃工作区内容
$ git checkout -- file

# 添加到了暂存区时，想丢弃修改
$ git reset HEAD [file]
$ git checkout -- file
```

### 删除

```
# 删除本地
$ rm [file]

# 删除git
$ git rm [file]
$ git commit -m "msg"

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

### 本地仓库连接远程仓库

```
echo "# this is gittest" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:nes-gnaw/gittest.git
git push -u origin master

# 查看远程库信息
$ git remote -v

# 删除已关联的GitHub远程库
$ git remote rm origin
```

### 修改本地连接仓库

```
$ git remote -v		-- 查看当前连接信息
$	git remote set-url origin http://47.97.114.252:9099/root/longge-saas-api.git		-- 更改至新链接
```



### 生成公钥

```
$ ssh-keygen -t rsa -C "xxx@xxx.com"  --id_rsa.pub文件
```

### 克隆远程仓库

```
git clone 'github.com' --ssh比https快
```

### 分支

```
# 创建分支并切换
$ git checkout -b dev
$ git switch -c dev	--两种写法

# 加上-b参数表示创建并切换，相当于以下两条命令
# 创建分支
$ git branch dev

# 切换分支
$ git checkout dev
$ git switch master

# 查看当前分支
$ git branch	--当前分支前面会标一个*号

# 合并指定分支到当前分支(快速合并)
$ git merge dev

# 删除分支
$ git branch -d dev
# 强行删除
$ git branch -D dev

# 分支的合并情况
$ git log --graph --pretty=oneline --abbrev-commit
# 分支合并图
$ git log --graph

# 普通合并
$ git merge --no-ff -m "merge with no-ff" dev
```

### BUG分支

```
# 储存当前分支的内容进度
$ git stash

# 查看储存列表
$ git stash list

# 恢复内容进度
方法一：
$ git stash apply	--恢复进度不删除存储区的内容
$ git stash drop	--删除存储区的内容

方法二：
$ git stash pop		--恢复并删除

# 恢复存储区的指定内容
$ git stash apply stash@{0}

# 复制”已提交“到当前分支
$ git cherry-pick <commit>
```

### 提交远程仓库

```
# 提交到主分支
$ git push origin master

# 提交到其他分支
$ git push origin dev
```

### 多人运动的流程

```
· 查看远程库信息，使用git remote -v；

· 本地新建的分支如果不推送到远程，对其他人就是不可见的；

· 从本地推送分支，使用git push origin branch-name，如果推送失败，先用git pull抓取远程的新提交；

· 在本地创建和远程分支对应的分支，使用git checkout -b branch-name origin/branch-name，本地和远程分支的名称最好一致；

· 建立本地分支和远程分支的关联，使用git branch --set-upstream branch-name origin/branch-name；

· 从远程抓取分支，使用git pull，如果有冲突，要先处理冲突。
```

### 整理分支提交

```
# rebase操作可以把本地未push的分叉提交历史整理成直线
$ git rebase --变基
# rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。
```

### tag标签

```
# 在当前分支的位置打一个新标签
$ git tag v1.0

# 查看所有标签
$ git tag

# 在指定的提交ID处达标签
$ git tag v0.9 f52c633

# 查看标签信息
git show <tagname>

# 用-a指定标签名，-m指定说明文字
$ git tag -a v0.1 -m "version 0.1 released" 1094adb

注：标签总是与commit挂钩，所以同一个标签可以同时出现在不同分支的commit处

# 删除标签
$ git tag -d v0.1

```

### tag标签远程操作

```
# 可以提交一个标签
$ git push origin <tagname>

# 提交全部标签
$ git push origin --tags

# 删除一个本地标签
$ git tag -d <tagname>

# 删除一个远程标签
$ git push origin :refs/tags/<tagname>
```

### 关联多个远程库

```
# 远程库的名称叫github，不叫origin
$ git remote add github git@github.com:michaelliao/learngit.git

# 远程库的名称叫gitee，不叫origin
$ git remote add gitee git@gitee.com:liaoxuefeng/learngit.git

# 查看远程库信息
$ git remote -v
gitee	git@gitee.com:liaoxuefeng/learngit.git (fetch)
gitee	git@gitee.com:liaoxuefeng/learngit.git (push)
github	git@github.com:michaelliao/learngit.git (fetch)
github	git@github.com:michaelliao/learngit.git (push)

# 推送到不同的库
$ git push github master
$ git push gitee master
```

### 个性化设置

```
# 让Git显示颜色
$ git config --global color.ui true
```

### .gitignore文件

```
# Windows:
Thumbs.db
ehthumbs.db
Desktop.ini

# Python:
*.py[cod]
*.so
*.egg
*.egg-info
dist
build

# My configurations:
db.ini
deploy_key_rsa

# 强制将忽略文件添加到暂存区
$ git add -f App.class

# 检查规则
$ git check-ignore -v App.class
.gitignore:3:*.class	App.class --第3行规则忽略了该文件
```

### 命令别名

```
$ git config --global alias.st status
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
$ git config --global alias.unstage 'reset HEAD'
$ git config --global alias.last 'log -1'
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

每个仓库的Git配置文件都放在.git/config文件中，别名就在[alias]后面，要删除别名，直接把对应的行删掉即可。
```

### 搭建git服务器

```
假设你已经有sudo权限的用户账号，下面，正式开始安装。

第一步，安装git：

$ sudo apt-get install git
第二步，创建一个git用户，用来运行git服务：

$ sudo adduser git
第三步，创建证书登录：

收集所有需要登录的用户的公钥，就是他们自己的id_rsa.pub文件，把所有公钥导入到/home/git/.ssh/authorized_keys文件里，一行一个。

第四步，初始化Git仓库：

先选定一个目录作为Git仓库，假定是/srv/sample.git，在/srv目录下输入命令：

$ sudo git init --bare sample.git
Git就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：

$ sudo chown -R git:git sample.git
第五步，禁用shell登录：

出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：

git:x:1001:1001:,,,:/home/git:/bin/bash
改为：

git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。

第六步，克隆远程仓库：

现在，可以通过git clone命令克隆远程仓库了，在各自的电脑上运行：

$ git clone git@server:/srv/sample.git
Cloning into 'sample'...
warning: You appear to have cloned an empty repository.
剩下的推送就简单了。

管理公钥
如果团队很小，把每个人的公钥收集起来放到服务器的/home/git/.ssh/authorized_keys文件里就是可行的。如果团队有几百号人，就没法这么玩了，这时，可以用Gitosis来管理公钥。

这里我们不介绍怎么玩Gitosis了，几百号人的团队基本都在500强了，相信找个高水平的Linux管理员问题不大。

管理权限
有很多不但视源代码如生命，而且视员工为窃贼的公司，会在版本控制系统里设置一套完善的权限控制，每个人是否有读写权限会精确到每个分支甚至每个目录下。因为Git是为Linux源代码托管而开发的，所以Git也继承了开源社区的精神，不支持权限控制。不过，因为Git支持钩子（hook），所以，可以在服务器端编写一系列脚本来控制提交等操作，达到权限控制的目的。Gitolite就是这个工具。

这里我们也不介绍Gitolite了，不要把有限的生命浪费到权限斗争中。

小结
搭建Git服务器非常简单，通常10分钟即可完成；

要方便管理公钥，用Gitosis；

要像SVN那样变态地控制权限，用Gitolite。
```

### 打包

```
git archive -o hmfms.zip HEAD $(git diff 22f5a...ca359a --name-only)
打包文件生成在项目同级路径下
```

### 新账户

```
1. ssh-keygen -t rsa -C "你的邮箱"  回车，查看rsa目录，先进入该目录，（将里面已有的id_rsa、id_rsa.pub两个文件进行复制到另一个目录）
2. 进入git官网，在settings里面生成ssh密钥，key里面填写id_rsa.pub里面的内容
3. 新密钥添加到SSH agent中：
ssh-agent bash
ssh-add ~/.ssh/id_rsa_work

切换账户：命令切换
git config --global user.name "YOURUSERNAME"
git config --global user.email "YOUREMAIL"

查看用户名和邮箱地址 
$ git config user.name 
$ git config user.email

查看账户：ssh -T git@github.com
```

