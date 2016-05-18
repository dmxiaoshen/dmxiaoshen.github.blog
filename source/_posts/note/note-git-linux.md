title: Git的Push命令免去用户名密码以及设置ssh-key
date: 2015-09-12 15:17:21
categories: note 
tags: [git, linux]
---

## Git的Push命令免去用户名密码以及设置ssh-key

我们用git的时候push命令会让输入用户名密码，这个有时候很烦，能不能免去输入直接push么，当然是可以的。本文以linux环境和github为列。

当然我们先得搞清楚为什么会有这个情况，其实是因为git有两种提交方式ssh和https。前一种方式需要配置ssh-key，然后push的时候不用输入用户名密码。后一种就是需要用户名密码啦。

看到这里，你或许已经猜到解决方法了，没错，就是url配置成ssh提交。

进入正题:

查看.git/config文件会发现
```python
[remote "origin"]  
fetch = + refs/heads/*:refs/remotes/origin/*  
url = https://username@github.com/username/projectname.git 
```

果然是https提交方式.

需要改为:
```python
 [remote "origin"]  
 fetch = + refs/heads/*:refs/remotes/origin/*  
 url = git@github.com:username/projectname.git  
```

到这里其实已经从https提交变成ssh提交了。接下来要去github页面设置ssh-key信息。

等等，还要生成ssh-key信息呢。

确保本机安装了ssh。

### 检查SSH公钥

cd ~/.ssh

看看存不存在.ssh，如果存在的话，掠过下一步；不存在的请看下一步

### 生成SSH公钥

```python
$ ssh-keygen -t rsa -C "your_email@youremail.com" 
# Creates a new ssh key using the provided email Generating public/private rsa key pair. 
Enter file in which to save the key (/home/you/.ssh/id_rsa):
```
现在你可以看到，在自己的目录下，有一个.ssh目录，说明成功了

### 输入密码

这里要注意了，这的密码是指在验证连接的时候的密码，看本文标题我们知道，验证的时候我们不想要输入密码，所以这里不设置密码，也就是empty，直接按回车。

```python
Enter passphrase (empty for no passphrase): [Type a passphrase]   
Enter same passphrase again: [Type passphrase again]
```
##进入.ssh目录

cd ~/.ssh

会看到 id_rsa和id_rsa.pub 文件,请复制id_rsa.pub文件里面的内容，下一步有用。

### 添加SSH公钥到github

打开github，找到账户里面添加SSH，把idrsa.pub内容复制到key里面。

### 测试是否生效

使用下面的命令测试
ssh -T git@github.com

当你看到这些内容放入时候，直接yes
```python
The authenticity of host 'github.com (207.97.227.239)' can't be established. 
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48. 
Are you sure you want to continue connecting (yes/no)?
```
看到这个内容放入时候，说明就成功了。
```python
Hi username! 
You've successfully authenticated, but GitHub does not provide shell access.
```