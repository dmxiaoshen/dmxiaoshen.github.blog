title: Hexo博客简单搭建
date: 2014-11-25 16:17:11
categories: note
tags: [hexo, 博客]
---

## Hexo博客简单搭建

> Hexo 是一款基于node 的静态博客网站生成器  
作者 : tommy351是一个台湾的在校大学生  
相比其他的静态网页生成器而言有着，生成静态网页最快，插件丰富（已经移植了Octopress 插件）。  
关于如何建立一个Hello World级别的Hexo，官方github主页有英文文档。  
<https://github.com/tommy351/hexo>


### 说明  

本博客就是根据Hexo生成的，这里简单记录下如何安装及配置，**本教程基于windows系统**。  

### 安装  

#### 1. 安装git  

下载[msysgit](http://msysgit.github.io/)并安装，非常简单。

#### 2. 安装Node.js  

下载[Node.js](http://www.nodejs.org/)(windows版本),根据提示安装即可，非常简单。  

#### 3. 安装Hexo  

启动Git Bash(其实有更快的方法，任意位置右击鼠标，选择Git Bash)，执行命令:  

```bash
npm install -g hexo
```

### 生成及本地搭建  

安装完成以后，你可以在你喜欢的目录建立hexo文件夹（文件夹名字可任取，这里用hexo）。  
选中该文件夹，**右键->Git Bash Here**,进入命令窗口，执行:  

```bash
hexo init
```

接着执行:  

```bash
npm install
```

用以安装各种依赖的包。  

到此，本地的Hexo博客已经搭建起来了。如想看效果，执行:  

```bash
hexo generate  
hexo server
```

浏览器输入**localhost:4000**就可以访问本地博客了。  
其实，以上命令可以简化为:  

```bash
hexo g
hexo s
```

### 部署到github  

上面的步骤只是在本地生成了博客，但是别人不能访问到，这里主要讲如何把博客部署到github上，这样，  
别人就可以通过链接访问博客了，说的通俗点就是博客放到外网上了。  

**首先需要一个github账号**,如果没有，请自行注册[github](https://github.com/),如果已有账号，请接着往下看。  

**强调一下，这里要牢记你的github账号，下面有依赖，举个例子，比如账号是dmxiaoshen**  

#### 1. 创建repository  

登录github，在自己的主页右下角，新建一个repository。这时，注意根据上面的账号，我们创建repository的名字是  
**dmxiaoshen.github.io**,(这里dmxiaoshen其实就是账号名，必须一致)。  

#### 2. 部署  

回到本地，进入刚才建立的hexo文件夹，会看到**_config.yml**文件，编辑它，修改以下的地方:  

```xml
deploy:
  type: github
  repository: https://github.com/dmxiaoshen/dmxiaoshen.github.io.git
  branch: master
```

**把这里的dmxiaoshen都改换成你的账号名。**  

最好先执行下:  

```bash
 $ git config --global user.name "yourgithubrname"  
 $ git config --global user.email "yourgithubemail"  
```

做下全局配置。

接着执行以下命令，即可完成部署:  

```bash
hexo generate
hexo deploy
```

在执行**hexo deploy**过程中，会让你输入github的账号和密码，输入即可，账号输入会有显示，密码输入不显示但是不要紧，输入就行，回车确定。

到此，部署完成。浏览器访问**dmxiaoshen.github.io**(dmxiaoshen换成你的账号)就可以访问了.  

### 补充

npm install 下载那里如果很慢，可以设置国内镜像，比如淘宝**npm config set registry https://registry.npm.taobao.org**

在Hexo 3.0版本后deploy git 被分开的，所以需要安装，安装命令如下**npm install hexo-deployer-git --save**
安装好后在尝试一下就ok。此时需要修改**_config.yml**文件为
```xml
deploy:
  type: git
  repository: https://github.com/dmxiaoshen/dmxiaoshen.github.io.git
  branch: master
```

用到**git clone**时可能需要添加ssh key，命令为**ssh-keygen -t rsa -C "youremail@youreamil.com"**.

**PS**:每次使用命令都要在该hexo文件夹下。每次新增或修改日志，都需先执行hexo generate才能保存。