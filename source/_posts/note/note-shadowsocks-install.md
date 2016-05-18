title: Shadowsocks服务器搭建(CentOS为例）
data: 2016-05-09 14:48:44
categories: note
tags: shadowsocks
---

## Shadowsocks服务器搭建(CentOS为例）

### 1.安装
```
yum update
yum install python-setuptools && easy_install pip
pip install shadowsocks
```

### 2.配置
`vi /etc/shadowsocks.json`

没有就新建。文件内容如下：

> {
> "server":"0.0.0.0",
> "server_port":443,
> "password":"yourpassword",
> "timeout":600,
> "method":"aes-256-cfb",
> "fast_open":false,
> "workers": 1
> }

如果想配置多个ss帐号，改成如下：

> {
>     "server":"0.0.0.0",
>     "local_port":1080,
>     "port_password":
>         {
>         "port1":"password1",
>         "port2":"password2"
>         },
>     "timeout":600,
>     "method":"aes-256-cfb",
>     "fast_open": false,
>     "workers": 1
> }

各字段含义如下：
**server**：服务器 IP地址 (IPv4/IPv6)
**server_port**：服务器监听的端口，一般设为80，443等，注意不要设为使用中的端口
**password**：设置密码，自定义
**timeout**：超时时间（秒）
**method**：加密方法，可选择 “aes-256-cfb”, “rc4-md5”等等。推荐使用 “rc4-md5”
**fast_open**：true 或 false。如果你的服务器 Linux 内核在3.7+，可以开启 fast_open 以降低延迟。
**workers**：workers数量，默认为 1。

TIPS: 加密方式**推荐使用**rc4-md5，因为 RC4 比 AES 速度快好几倍，如果用在路由器上会带来显著性能提升。旧的 RC4 加密之所以不安全是因为 Shadowsocks 在每个连接上重复使用 key，没有使用 IV。现在已经重新正确实现，可以放心使用。

### 3.启动
`ssserver -c /etc/shadowsocks.json -d start`

如此就算是安装成功了。接下来可能会面临一些问题。

**需要开机启动**
修改rc.local文件，运行命令 `vi /etc/rc.local`
内容如下：

> #!/bin/sh
> ssserver -c /etc/shadowsocks.json -d start
重启生效。

**服务起来了，但是连不上（telnet端口不通）**
那是因为centos防火墙没开放刚才配置文件里需要的端口。
centos 6 ：
`vi /etc/sysconfig/iptables`
添加一行
`-A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT`
如此就开放了443端口
`service iptables restart`
重启iptables服务以生效。

centos 7：
查看443端口是否开放
`firewall-cmd --query-port=443/tcp`

开放443端口
`firewall-cmd --add-port=443/tcp`
**停止/重启服务**
`ssserver -d stop`
或者
`killall ssserver`
如果killall命令找不到，则
`yum install psmisc`

重启
`ssserver -c /etc/shadowsocks.json -d restart`
**查看日志**
`less /var/log/shadowsocks.log`

