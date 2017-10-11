---
title: Git使用总结
date: 2015-04-10 22:52:41
tags:
- 工具
categories: 基础
---


### 1.配置：
**1.设置你的名字与邮箱（可与 git 帐号无关）：**

`git config --global user.name "Your Name"`
`git config --global user.email your@email.com`

缓存一段时间的 git 帐号密码（timeout 单位：秒）：

`git config --global credential.helper "cache --timeout=3600"`

**2.克隆、签出仓库：**
`git clone <url>`

服务器一般会给出两种连接方式
- 一种是SSH:需要SSH加密
- 一种是HTTP:直接可以用

<!--more-->
以上边的地址为例：
`git clone http://10.16.1.16/iOS/gitintro.git`
这样它默认会将文件以gitintro为名，建立在当前文件路径下
自定义：
`git clone http://10.16.1.16/iOS/gitintro.git mygit`
这样写它会将文件夹的名字从gitintro改为mygit

**另外：**如果该项目使用了，`git submodule`子模块 技术，那么还需要增加另外两步操作：
- `git submodule init`
- `git submodule update`

或者一步到位
`git clone --recursive http://10.16.1.16/iOS/gitintro.git mygit`

3.自建仓库
只要是一个文件夹，然后进行git初始化就可以
利用命令行将当前目录切换到你要做仓库的那个文件夹，
`cd xxx`
然后初始化仓库
`git init`

**3.查看远程服务器：**
`git remote -v`

进阶：
关联远程服务器：
`git reomte add <origin> <git-repo-url>`

查看配置信息，包括 remote、分支关联、本地配置等：
`cat .git/config`


### 2. 添加、删除、本地提交
添加新文件，确认文件修改：
`git add <file-path>`

取消添加新文件：
`git reset <file-path>`

从 git 删除文件（不再在 repo 里管理了）：
`git rm <file-path>`

查看本地未确认的修改（代码、内容变更）：
`git diff`

查看本地已确认（但未提交）的修改：
`git diff --cached`

查看本地修改的文件状态：
`git status`

本地提交（提交上述 add\rm 确认的修改、操作），生成一个版本：
`git commit -m "short commit message"`

追加到上一次本地提交：
`git commit --amend`

查看提交记录：
`git log`

简化版
`git reflog`

查看提交的详细修改记录：
`git log -p`

查看提交修改的文件信息：
`git log --stat`

其它：
让当前修改 AFK 一下，好干点别的事情：`git stash`
事情做完，把 AFK 前的修改找回来：`git stash pop`


### 3. 撤消
撤销一个文件的修改：
`git checkout -- <file-path>`

撤销当前目录的所有修改：
`git checkout -- .`

删除某个文件
`git rm xxxx`
### 4.版本回退
回退到任意版本，需要拿到commit id
`git reset --hard 3628164`
回退到上一个版本
 `git reset --hard HEAD^`
 回退到上100个版本
 `git reset --hard HEAD~100`

**但是这样简单的操作只能是把本地的工作空间的内容进行回退，那么如果我想要让远程库也回退怎么办呢？**

回想一下原理，当你对这个分支进行pull的时候
- 它会去版本库去检查有没有冲突
- 如果没有那么它直接会覆盖工作空间
- 如果有冲突它才会提示你解决冲突

那么现在，如果我已经到了第五个 commit，但是我想要回到第3个commit的状态，
`git reset --hard 87b6a3b`
你想要把这个状态更新到远程，先要commit，你会发现你commit是无效的，因为你这个commit已经存在了，一摸一样，所以你是不能commit的，现在要自己制造一些修改，然后再commit，就可以了。

**所以远程版本回退的根本是制造冲突，然后提交冲突**

**那么如果是这个文件已经删除了那么这个时候怎么办呢？**
答案是：
- 本地进行版本回退`git reset --hard 76824`
- 打开这个又恢复的文件，然后修改一些东西
- 执行 `add commit`
- 然后执行pull，最新的修改从远程拉回来，这个时候它会报冲突

它说，这个冲突是修改和删除的冲突，也就是说，一个版本库中它已经被删除了，可是在另外一个版本库中它是被修改的，现在你就要解决冲突，怎么解决呢？原来的冲突最多是这个文件中的代码不一致了，这个时候找你打开代码，删删剪剪就好了，可是现在文件直接没有了咋弄？
再次git pull
它说你可以用git add/rm xxx来决定你是保留冲突中的哪一方


### 5. 分支
创建分支（基于当前分支）：
`git branch <new-branch>`

基于某个分支创建分支：
`git branch <new-branch> <base-branch>`

查看本地分支：
`git branch`

切换分支：
`git checkout <branch>`

创建新分支并立即切过去：
`git checkout -b <new-branch> [base-branch]`

合并一个分支到当前分支（删除该分支后不保留commit纪录）：
`git merge <other-branch>`

合并分支且删除该分支后保留commit纪录
`git merge --no-ff -m"xxx" <other-branch> `

回滚一个分支的合并：
`git revert -m (1|2) <merge-commit>`
参考： http://pms.kugou.com/zentao/doc-view-171.html

rebase 把分支的 commit 在基础分支上重放（与 merge 的区别很重要，请只在本地分支 rebase！！！）：
`git rebase <base-branch>`

安全删除（已经合并过的）本地分支：
`git branch -d <branch>`

强行删除本地分支（请先 -d，如果 -d 失败，检查无误后才 -D）：
`git branch -D <branch>`


### 6. 跟服务器同步：
查看远程分支：
`git branch -r`

签出服务器分支工作（会创建本地与服务器分支关联）：
`git checkout <remote-branch-name>`
或
`git checkout -b <local-branch-name> origin/<remote-branch-name>`

（已存在的）本地分支关联到服务器分支：
`git branch -u origin/<remote-branch> <local-branch>`
或：
`git branch --set-upstream-to=origin/<remote-branch> <local-branch>`

获取服务器（所有的）更新，并合并当前远程分支的更新：
`git pull`

使用 rebase 的方式 pull，可能会少产生一条 merge 虚 commit：
`git pull --rebase`

获取某个远程分支的更新（并且合并到当前分支！！）：
`git pull origin <branch>`

仅拉取服务器更新，但不合并或 rebase：
`git fetch [origin [<remote-branch-name>]]`

rebase 当前分支已经 fetch 到本地的服务器更新：
`git rebase`

----------------
**push**
push新建的本地分支并且建立连接
`git push --set-upstream origin <local-branch>`
`git push origin <local-branch>`
`git push -u origin <local-branch>`

删除服务器远程分支：
`git push origin :<remote-branch>`
或 git 1.7 以后：
`git push origin --delete <remote-branch>`

push该分支的内容更新，前提是已经有默认的对应的分支，如果没有就会报错
`git push`
![Alt text](./1443163940619.png)

但是这种方式不需要提前关联，因为你已经指明了，我打算把我这个分支推送给谁：
`git push origin <branch>`

清除已被删除远程分支的本地残余：
`git remote prune origin`

### 7.fork源码之后与源码保持同步

1.切换到要保持同步的分支

2.添加一个别名，指向ngqa项目的位置 
`git remote add ngqa https://github.com/howe/ngqa.git`

3.pull一下它下边的某个分支，一定要确定一下这个分支是不是有内容`git pull ngqa master`

4.然后在sourcetree解决一下冲突，一般是点一下最下边的那个红色选框

5.然后`git push`

6.如果出现如下问题
>  "please enter a commit message to explain why this merge is necessary,especially if it merges an updated upstream into a topic branch

解决方案：
>1-press "i"
2-write your merge message
3-press "esc"
4-write ":wq" then press enter

### 8.配置SSH
- 查看git用户名
`git config user.name`
- 查看git邮箱
`git config user.email`
- 修改git用户名和邮箱
`git config --global user.name "username"`
`git config --global user.email "email"`
- 查看是否已经有了ssh密钥：
`cd ~/.ssh`
- 生成SSH
`ssh-keygen -t rsa -C "你的邮箱"`
- 查看SSH
`cat ~/.ssh/id_rsa.pub`
将下面出现的这些个东西都拷贝到你的SSH面板中


### 9.问题集锦
- 如果你明明已经有了权限但是还是不能clone代码，那可能是服务器设置了DNS解析，你需要更改本机的DNS解析地址和服务器一致
- 冲突解决

