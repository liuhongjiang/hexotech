title: 初识Docker
date: 2015-09-22 09:18:05
tags:
---

## install docker on ubuntu

下载安装
http://docs.docker.com/linux/step_one/

```
$ wget -qO- https://get.docker.com/ | sh
```

启动docker

	sudo service docker start

验证安装，这个命令将会下载hello-world的image，将运行一个container：

	docker run hello-world

<!-- more -->

如下图

{% image center /images/blogimages/2015/down-hello-world.png %}

## build docker

refer to: [http://docs.docker.com/linux/step_four/](http://docs.docker.com/linux/step_four/)

edit Dockerfile:

```
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay
```

then build with command:

```
docker build -t docker-whale .
```

tag image, for example:

```
docker tag 7d9495d03763 maryatdocker/docker-whale:latest
```

the use `docker login`, and the posh the image to docker hub:

```
docker push maryatdocker/docker-whale
```

then you can `docker pull` your image:

```
docker pull yourusername/docker-whale
```

also you can remove the image by `docker rmi`


## docker command

```
docker run         # 运行image
docker ps -a       # list container
docker rm          # 删除 container
docker rmi         # 删除 image
docker images      # list image
docker tag         # 给image打tag
docker login       # 登录到 docker hub
docker push        # 上传image到docker hub
docker pull        # 从docker hub上下载image
docker exec -it angry_poitras /bin/bash
				   # 在正在运行的container中执行命令
docker save        # 将image保存为一个tar包
docker load        # 从tar包，或者stdin加载image
docker commit      # 将container的修改保存为另外一个image
docker cp          # 在container和host之间拷贝文件和目录
```

例如，这是`docker ps`的例子：

{% image center /images/blogimages/2015/docker-ps.png %} 

这是`docker rm`命令的例子：

{% image center /images/blogimages/2015/docker-rm.png %}

## docker proxy


Edit `/etc/defaults/docker` and add the following lines:

```
export http_proxy 'http://user:password@proxy-host:proxy-port'
export https_proxy 'http://user:password@proxy-host:proxy-port'
export HTTP_PROXY 'http://user:password@proxy-host:proxy-port'
export HTTPS_PROXY 'http://user:password@proxy-host:proxy-port'
```

This should allow docker daemon to pull images from the central registry. However, if you need to configure the proxy in the Dockerfile (ie. if you're using apt-get to install packages), you'll need to declare it there too.

Add the following lines at the top of your Dockerfile:

```
ENV http_proxy 'http://user:password@proxy-host:proxy-port'
ENV https_proxy 'http://user:password@proxy-host:proxy-port'
ENV HTTP_PROXY 'http://user:password@proxy-host:proxy-port'
ENV HTTPS_PROXY 'http://user:password@proxy-host:proxy-port'
```

With those settings, your container should now build, using the proxy to access the outside world.
