title: using kolla deploy ceph on vagrant vms
date: 2016-10-19 14:52:42
tags:
---


add disk to vm using vagrant
https://gist.github.com/leifg/4713995

LOOP OVER VM DEFINITIONS
https://www.vagrantup.com/docs/vagrantfile/tips.html#loop-over-vm-definitions


list block device
http://unix.stackexchange.com/questions/49786/finding-all-storage-devices-attached-to-a-linux-machine



allowed kolla to using sudo command

modify inventory file:
```
[ceph-osd]
ceph1 ansible_sudo=True
ceph2 ansible_sudo=True
ceph3 ansible_sudo=True
```

change /etc/fstab permission
```
 sudo chmod a+w /etc/fstab
```


docker:
```
Get https://192.168.60.31:5000/v1/_ping: http: server gave HTTP response to HTTPS client
```

http://stackoverflow.com/questions/38695515/can-not-pull-push-images-after-update-docker-to-1-12
