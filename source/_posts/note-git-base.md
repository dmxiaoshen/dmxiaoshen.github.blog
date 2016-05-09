title: Git/Github基础用法
date: 2014-11-12 10:17:11
categories: note
tags: [git]
---

## Git/Github基础用法

> Git是一个开源的分布式版本控制系统，用以有效、高速的处理从很小到非常大的项目版本管理。  
Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。  
Torvalds 开始着手开发 Git 是为了作为一种过渡方案来替代 BitKeeper，后者之前一直是 Linux 内核开发人员在全球使用的主要源代码工具。开放源码社区中的有些人觉得 BitKeeper 的许可证并不适合开放源码社区的工作，因此 Torvalds 决定着手研究许可证更为灵活的版本控制系统。尽管最初 Git 的开发是为了辅助 Linux 内核开发的过程，但是我们已经发现在很多其他自由软件项目中也使用了 Git。例如 最近就迁移到 Git 上来了，很多 Freedesktop 的项目也迁移到了 Git 上。

### 安装及配置    
本文以在windows下使用为例，下载[msysgit][1]并安装。安装完以后做些设置(可以不设置):  

*  **Git Bash快捷图标->右键->属性->选项->勾选快速编辑模式** 这样就可以在命令界面很方便的使用复制和粘贴(用左键选取要复制的，点右键直接就可以复制，粘贴时只需点一下右键)。
*  **Git Bash快捷图标->右键->快捷方式->起始位置** 把项目地址放这里就可以了，这样默认打开就是该地址。  

### 配置本地用户和邮箱  
在这之前，需要注册一个github账号，这里不多说了，假设已经有github账号。命令行输入:  
```bash
 $ git config --global user.name "yourgithubrname"  
 $ git config --global user.email "yourgithubemail"  
```

### git常用命令  
#### git init  
**将一个目录初始化为git仓库**，命令行进入该目录，执行:`$ git init`,执行完毕，目录中会多一个**.git**文件夹，仓库建立完毕。  

#### git clone  
**克隆一个已有的git仓库**，如在github上获取别人的项目，执行:`$ git clone [url]`,如需加别名，则在url后面加上即可。  

#### git status  
**查看代码缓存与当前工作目录的状态**，执行:`$ git status`,如需查看简短输出，则输入`$ git status -s`  
文件及文件夹会有如下状态(状态有红和绿两种颜色，绿色表示已加入缓存，红色表示未加入缓存):  

* **?** 未添加  
* **M** 已修改  
* **D** 已删除  
* **A** 已添加

#### git add  
**添加文件到缓存**,执行`$ git add [filename]`，添加一个文件，如需添加所有，则`$ git add --all`  

#### git diff  
**默认显示的是尚未缓存的改动**,`$ git diff`。  
`$ git diff --cached`，显示已缓存的改动。  
`$ git diff HEAD`，显示已缓存的与未缓存的所有改动。  

#### git commit  
**记录缓存内容快照**，执行:`$ git commit -m "comment注释"`，这样就把那些已经提交到缓存的内容给记录到快照了。  
如果是有新增文件，则必须先执行git add之后再执行git commit才完成快照存储。  
如果只是修改没有新增文件，则可执行:`$ git commit -a -m "comment注释"`，这样可以省去git add这一步。

#### git reset  
**取消已缓存的内容**，执行`$ git reset`,默认取消上一次所有的缓存，  
如果想只取消个别文件，则执行`$ git reset HEAD -- [filename]`

#### git branch  
**列出本地分支**,`$ git branch`  
如想新增分支，则执行`$ git branch [newbranchname]`，  
切换分支，则执行`$ git checkout [branchname]`，  
删除分支，则执行`$ git branch -d [branchname]`。

#### git merge  
**将分支合并到当前分支**，执行`$ git merge [branchname]`。  
具体会涉及到很多合并问题，如冲突等，这里不细说。  

#### git log  
**显示一个分支中的提交更改记录**，执行`$ git log`，  
如想显示简洁列表，执行`$ git log --oneline`，  
如想显示分支合并情况，执行`$ git log --graph`。  

#### git tag  
**给当前记录打标签**，执行`$ git tag -a v1.0`，  
如想给之前的一个快照打标签，执行`$ git tag -a v0.9 [code]`，该code是执行git log --oneline时显示地7位code  
执行git log --decorate可看到打的标签。  

#### git remote  
**列出远端别名**，执行`$ git remote`，  
如想查看细节，执行`$ git remote -v`。  
**git remote add**，为项目添加远端，执行`$ git remote add [alias] [url]`。这里alias是别名，一般用origin，url是远端地址。  
**git remote rm**，删除现存某个别名，执行`$ git remote rm [alias]`。  

#### git fetch和git merge  
**从远端更新并合并代码**，执行`$ git fetch [alias]`，  
合并`$ git merge [alias]/[branch]`，合并到当前本地分支。 

#### git push  
**推送分支到远端**，执行`$ git push [alias] [branch]`，一般如果本地只有一个远端仓库，一个分支，则执行git push即可。


参考网址：<http://www.ihref.com/read-16369.html>

[1]: http://msysgit.github.io/ "msysgit"




