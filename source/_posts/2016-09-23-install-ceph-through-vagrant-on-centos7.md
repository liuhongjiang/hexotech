title: install ceph through vagrant on centos7
date: 2016-09-23 10:42:55
tags:
---


This paper is for installing ceph on centos7 through vagrant.

## prepare

* enable epel and install development tools
```shell
sudo yum install epel-release
sudo yum groupinstall 'Development Tools'     # install development tools, not necessary
```

<!--more-->

* install virtualbox

```shell
sudo su
cd /etc/yum.repos.d
wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo

yum install dkms
yum install kernel-devel

yum install VirtualBox-5.1    # install the latest virtualbox
```

Reference:
https://wiki.centos.org/HowTos/Virtualization/VirtualBox

The installer will create the "vboxusers" group and create the necessary kernel modules if the development environment has been correctly configured.

For __each__ "username" that will run VirtualBox:

```shell
usermod -a -G vboxusers username
```


* intall vagrant

```shell
wget https://releases.hashicorp.com/vagrant/1.8.5/vagrant_1.8.5_x86_64.rpm # please use latest vagrant package
sudo rpm -ivh vagrant_1.8.5_x86_64.rpm
```

then install the vagrant centos7 box

https://atlas.hashicorp.com/centos/boxes/7

```shell
vagrant box add --provider virtualbox centos/7
```

## run the ceph installation

I have already create a repo for this process, which is verified on centos7.

please refer to this github repo: [vagrant-ceph-deploy](https://github.com/liuhongjiang/vagrant-ceph-deploy)


## problems and solutions

* vagrant up failed

If the following problems appears, you can run the `vboxmanage --version` to figure out which problems there.

```shell
$ vagrant up
VirtualBox is complaining that the kernel module is not loaded. Please
run `VBoxManage --version` or open the VirtualBox GUI to see the error
message which should contain instructions on how to fix this error.
```

fixed it by doing this:

```
tjwebb@latitude:xtuple-vagrant (master) $ sudo /etc/init.d/vboxdrv setup
[sudo] password for tjwebb:
Stopping VirtualBox kernel modules ...done.
Recompiling VirtualBox kernel modules ...done.
Starting VirtualBox kernel modules ...done.
```

refer to: https://github.com/lynnaloo/xtuple-vagrant/issues/13

* set vm using sata controller

when using [ceph ansible](https://github.com/ceph/ceph-ansible) to deploy ceph.

and the guests are centos you mayneed to change the storage controller to sata,

the following is how to change it by terminal, but also you can change it through GUI.

```
VBoxManage storagectl {vm-name or vm-uuid} --name SATA --add sata
```


* ubuntu install failed

The error and solution is here:
http://askubuntu.com/questions/148383/how-to-resolve-dpkg-error-processing-var-cache-apt-archives-python-apport-2-0

```
sudo apt-get clean
sudo apt-get update && sudo apt-get upgrade
sudo dpkg --configure -a
sudo apt-get -f install
```

if it's still reporting error, such as some error during processing some pacakge files, please try following solution:

````
sudo dpkg -i --force-overwrite <filename>
sudo dpkg -i --force-overwrite /var/cache/apt/archives/python-problem-report_2.0.1-0ubuntu9_all.deb

# then run
sudo apt-get -f install
```

## Reference
[ceph install manual](http://docs.ceph.com/docs/master/start/)
[vagrant add shell provision](https://www.vagrantup.com/docs/provisioning/shell.html)
