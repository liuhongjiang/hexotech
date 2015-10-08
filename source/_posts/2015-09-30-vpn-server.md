title: vpn server 搭建
date: 2015-09-30 10:57:30
tags:
---

买了一个vps，装了一个vpn，记录一下相关的内容。

vps是在这里买的，最低配，一年人民币130左右，支持支付宝。

server的操作系统是ubuntu的14.04.

然后follow [ubuntu的教程](https://help.ubuntu.com/community/PPTPServer)，安装PPTPD.

<!-- more -->

```
sudo apt-get install pptpd
```

配置server IP and client IP, 配置文件为：`/etc/pptpd.conf`

```
localip 192.168.0.1
remoteip 192.168.0.100-200
```

这个要重点说一下，我开始一直没有明白这个localip和remoteip是什么含义。后来终于明白了，可以这么理解，PPTPserver和PPTP Client组成了一个虚拟的局域网。
在这个虚拟机网络中，localip是分配给PPTPServer， remoteip是分配给client。这个和server，client的实际ip没有任何关系。
所以建议用192.168.xxx.xxx，主要不要与你的实际ip冲突了，因为你的电脑也可能位于一个物理的局域网之内。

修改`/etc/ppp/pptpd-options`文件添加dns

```
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```
这个是google的dns，也可以用openDNS

然后添加登录vpn的账号和密码， 修改`/etc/ppp/chap-secrets`

```
# client        server  secret                  IP addresses
username * myPassword *
```

其中各个字段的含义:
```
client: 就是用户名
server：这没有具体研究什么含义，可以直接使用*
secret：用户登陆密码
IP address：用户的接入ip，可以用*代表运行所有的ip，也可以填入多个，用逗号分隔。
```

我的设置方法比较简单，server和IP address都设置为了`*`，表示没有任何限制。

重启服务：
```
service pptpd restart
```

修改`/etc/sysctl.conf`, 开启内核转发

```
net.ipv4.ip_forward=1
```

重载配置

```
sudo sysctl -p
```

修改iptable的转发规则，修改文件`/etc/rc.local`,

```
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
iptables -A FORWARD -p tcp --syn -s 192.168.0.0/24 -j TCPMSS --set-mss 1356
```

其中的ip是与前面的localip和remoteip相对应的

我自己的方法，就相对粗暴了，直接关闭防火墙：
```
ufw disable
```


然后重启server就可以使用了。


## 其他问题

我在搭建的过程中，遇到了很多问题。如果你遇到了问题，下面几篇文章可能可以帮助你。

如果[测试vpn](https://www.digitalocean.com/community/tutorials/how-to-setup-your-own-vpn-with-pptp).

查看日志，日志的位置是`/var/log/syslog`

打开debug日志：修改`/etc/pptpd.conf` 取消 debug的注释，重启服务。


查看ip：
```
dig +short myip.opendns.com @resolver1.opendns.com
```

查看是否运行：
```
netstat -anpl | grep pptpd
```

如果打开了debug日志，在连接过程可能会有断线情况，查看日志发现了下面的错误
```
Feb  5 13:07:52 plugh rsyslogd-2177: imuxsock begins to drop messages from pid 12105 due to rate-limiting
```

修复方法：

1. 关闭debug
2. 修改rsyslog.conf
The first solution is to simply increase the messages allowed and the time interval before rate-limiting occurs in rsyslog.  To do this, locate the rsyslog.conf and/or rsyslog.early.conf (usually in /etc) and add the following lines:

``` 
$SystemLogRateLimitInterval 10
$SystemLogRateLimitBurst 500
```

The second solution is to simply turn off rate-limiting for rsyslog, and to do this, add the following line to rsyslog.early.conf and/or rsyslog.conf using your favorite editor 

```
$SystemLogRateLimitInterval 0
```