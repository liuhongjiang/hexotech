title: 设置ftp匿名下载的服务
date: 2015-10-14 17:12:25
tags:
categories: linux
---

经常要在服务器上设置一些匿名账号，方便CI build处理啊的包，其他人可以下载，所以简单写了一个如果设置匿名ftp的文章，免得每次都要去网上找。

<!--more-->

安装ftp server

```
sudo yum install vsftpd
```

修改配置文件`/etc/vsftpd/vsftpd.conf`

```
sudo vi /etc/vsftpd/vsftpd.conf
```

打开运行匿名用户登陆：

```
anonymous_enable=YES
```

配置匿名用户的根目录

```
anon_root=/home/some/path/
```

需要注意的一点，就是这个路径的所有目录的权限都是755，对于其他用户可以读和可执行的。

关闭或者设置防火墙
```
service iptables stop // centos 6

systemctl stop firewalld   // centos 7
systemctl disable firewalld
```


启动
```
sudo service vsftpd restart     // centos 6

systemctl restart vsftpd        // centos 7
```

将vsftpd设置为启动时，自动运行
```
chkconfig vsftpd on
```

## 问题
配置完成后，启动了ftp服务，但是通过浏览器，打开ftp，显示为空，没有任务内容。

使用ftp命令连接，显示下面这个错误

```
VSFTPD: Transfer Done (but failed to open directory)
```

原因是selinux将这个连接视为可疑的，解决办法就是讲selinx disable，然后重启服务器。

## 添加用户

请参考：

http://serverfault.com/questions/544850/create-new-vsftpd-user-and-lock-to-specify-home-login-directory

具体步骤：

1. Create user with `useradd [user_name]`.
2. Create user's password with `passwd [user_name]`. (You'll be prompted to specify the password).
3. Create FTP directory in `/var/ftp` and then bind to the `home` directory you wish to specify for this user with `mount --bind /var/www/vhosts/domain.com/ /var/ftp/custom_name/`.
4. Change user's home directory with `usermod -d /var/ftp/custom_name/ user_name`
5. In /etc/vsftpd/vsftpd.conf, ensure all all of the following are set:
```
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
```

Only list users in the vsftpd.chroot_list file if you want them to have full access to anywhere on the server. By not listing them in this file, you're saying restrict all vsftpd users to their specified home directory.

需要注意的是，为user设置的home directory，需要设置full access。

## reference

[How to stop/start and disable/enable Firewall on Redhat 7 Linux system](http://linuxconfig.org/how-to-stop-start-and-disable-enable-firewall-on-redhat-7-linux-system)

[How To Set Up vsftpd on CentOS 6](https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-on-centos-6--2)

[VSFTPD: Transfer Done (but failed to open directory)](http://blog.danielburrowes.com/2009/06/vsftpd-transfer-done-but-failed-to-open.html)
