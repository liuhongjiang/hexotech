title: vagrant multiple vms
date: 2016-10-11 10:22:02
tags:
---

最近的一个工作需要在centos的terminal下面使用vagrant和virtual box，最后发现virtualbox对命令行的支持非常好，另外vagrant的Vagrantfile其实可以支持对多个vm的管理。

vagrant的multi-machine定义，可以参见vagrant的文档： [vagrant MULTI-MACHINE](https://www.vagrantup.com/docs/multi-machine/)

## 设置hostname
```
config.vm.define "osd1" do |osd1|
  osd1.vm.hostname = "osd1"
end
```

## 设置相互信任关系

通过命令行，我们可以通过下面的命令设置server的信任关系
```
ssh-keygen
ssh-copy-id vagrant@osd1  # vagrant 的默认密码也是vagrant
```

但在vm provision的过程中，由于`ssh-copy-id`需要输入密码，所以不能自动完成。

一个可行的办法是，将private key和public key都准备好，放入到vagrantfile所在的目录下，然后在vagrantfile中加入以下provision的代码：

```
config.vm.define "mon1" do |mon1|
    mon1.vm.provision "file", source: "ssh-keys/id_rsa.pub", destination: "/home/vagrant/.ssh/id_rsa.pub"
    mon1.vm.provision "file", source: "ssh-keys/id_rsa", destination: "/home/vagrant/.ssh/id_rsa"
    mon1.vm.provision "shell", inline: "cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys"
end
```

对于所有的vm都执行上面的代码，那么所有的vm的privatekey和public key都相同，而且都加入到了authroized_keys里面，那么vm相互之间ssh，就不需要密码了。

## ssh config

在ssh的配置中，下面两个配置项比较有用：
`PasswordAuthentication yes`, 允许ssh使用password登录

`StrictHostKeyChecking no`，自动保存known_hosts。

```
# add ssh password
sudo sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config

# add ssh known_hosts automatically
sudo sed -i "s/^\tGSSAPIAuthentication yes/\tGSSAPIAuthentication yes\n\tStrictHostKeyChecking no/g" /etc/ssh/ssh_config
```

## virtualbox commands

vboxmanage list runningvms
vboxmanage list vms

## 参考
[vagrant MULTI-MACHINE](https://www.vagrantup.com/docs/multi-machine/)
[vagrant tips](https://www.vagrantup.com/docs/vagrantfile/tips.html)
