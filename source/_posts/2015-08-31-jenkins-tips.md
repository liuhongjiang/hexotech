title: Jenkins tips
date: 2015-08-31 17:10:30
tags:
categories: jenkins
---

## 在windows上以service的方式安装jenkins slave节点

1. 在命令行运行，安装slave节点

```
java -jar slave.jar -jnlpUrl http://jenkins-server-ip:8080/jenkins/computer/{{node name}}/slave-agent.jnlp -secret 2d00ae81d59ff37df0fe9c33a5ed671afa2bc495715c0c161091a8e429973eb1 
```

2. 解决阻止自签名的问题
您的安全设置已阻止自签名的应用程序使用已过期的Java版本运行问题的解决
```
步骤：选择windows的开始-->控制面板-->在搜索栏中输入 java-->点击查询出来的Java图标-->安全-->把 高(推荐的最低设置)-->改成 中-->确定-->即可。
```

3. 以servcie的方式启动
在下面的页面上： http://jenkins-server-ip:8080/jenkins/computer/\{node name\}
```
操作步骤： 点击Launch-->保留-->把slave-agent.jnlp拷贝到C:\CI\jenkins-slave下-->使用javaws.exe(java web start launcher)-->继续-->
点击 我接受风险并希望运行此应用程序。-->运行-->File-->Install as a service-->确定-->是-->是
```

这样Jenkins节点以Windows Service的方式在运行了。

<!--more-->

4. 程序不能在Administrator的账号下面运行：

某些程序无法再admininstrator上运行，但是jenkins slave是以administrator启动的，只有通过修改jenkins service的登录账号解决。

步骤:
I too had a similar issue once. Try granting the Jenkins service "Logon as This account" right under services.msc and make sure the account you type there is the same as the one you use for running cmd.exe. 

```
在window运行中输入services.msc-->选择Jenkins-->Log On-->Logon as This account to This account : andrew/andrewpassword -->OK
```

重启Jenkins节点的服务,重新构建项目,就转换成了andrew用户来运行了.

参考： [Execute windows batch command from Jenkins fails but runs fine in cmd.exe](http://stackoverflow.com/questions/10952280/execute-windows-batch-command-from-jenkins-fails-but-runs-fine-in-cmd-exe)


## multi-branch branch for git

[travis-ci](http://blog.travis-ci.com/announcing-pull-request-support/)

[jenkins multi-branch-build plugin](https://github.com/mjdetullio/multi-branch-project-plugin)

## jenkins检查github的pull request

下面插件可以实现当在github上提交了一个pull request，他可以自动完成一些工作，例如对pull request加评论，验证pull request，对pull reques跑test等等。
没有具体试验过，以后有机会，试用一下，然后加评论。

[GitHub pull request builder plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+pull+request+builder+plugin)

## jenkins分支同步

我们的项目是在github上，一个比较大的问题就是同步经常出问题，一个同事使用了这两个插件解决了这个问题。

[CloudBees Folders Plugin](https://wiki.jenkins-ci.org/display/JENKINS/CloudBees+Folders+Plugin)

[CloudBees GitHub Branch Source Plugin](https://wiki.jenkins-ci.org/display/JENKINS/CloudBees+GitHub+Branch+Source+Plugin)


## jenkins配置使用github登录

安装[Github OAuth Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Github+OAuth+Plugin)， 然后在manage jenkins的Configure Global Security中，可以将Access Control配置使用Github Authentication Plugin。

配置图如下：

{% image center /images/blogimages/2015/jenkins-github-oauth.png %}


上图中的api配置路径，如果使用的是GitHub Enterprise，那么url的路径就要配置为`/api/v3`.

client id和client seceret是在github中生成的。

打开github，在edit profile的地方，选择applications -> developer applications, 生成一个新的application。
很多都是自动生成的，比较重要的一个是authorization callback url，这个也需要天固定的格式：

例如`http://myserver.com:8080/securityRealm/finishLogin`，其中myserver.com和端口8080对应你的jenkins的域名和端口。

下图是一个例子

{% image center /images/blogimages/2015/github-applications.png %}

在上图中，还可以给你的application加一个图标，如5所示。

## jenkins tool配置

在jenkins中，使用到很多工具，例如git，maven，jdk，node。可以在manage jenkins -> configure system中配置。

在配置中，如果选择了install automatically，那么jenkins就会在salve node上自动安装这个工具。这样如果在jenkins中使用docker node等这种方式的话，有可能每次都会去安装。

或者如果你的node已经安装了，jenkins也会去安装一次。

所以可以不选择install automatically， 然后配置一个XXX_HOME或者工具的path，那么jenkins就会自动去找个工具，而不是去安装。但是如果配置了工具的路径，这个配置是对所有node有效的。

如果对没有node需要单独配置的话，可以去manage node页面，在node的configure页面中配置tool location，这样这个配置项就就只对这个node生效。

当然还有一个jenkins plugin可以使用: [Custom Tools Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Custom+Tools+Plugin)，它甚至可以对单独的一个job配置工具路径。


## 参考
[jenkins main page](https://wiki.jenkins-ci.org/display/JENKINS/Use+Jenkins)

[installation jenkins on ubuntu](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu)

[enable jenkins user](https://wiki.jenkins-ci.org/display/JENKINS/Standard+Security+Setup)

[install jenkins on centos](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Red+Hat+distributions)

[install the last git on centos](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-a-centos-6-4-vps)

## issues

issue 1:

[unknown-option-git-config-local-reported-by-jenkins](http://stackoverflow.com/questions/23104110/unknown-option-git-config-local-reported-by-jenkins)

issue 2:

```
Failed to connect to repository : Command "git -c core.askpass=true ls-remote -h https://github.com/liuhongjiang/gittest.git HEAD" returned status code 128:
stdout: 
stderr: fatal: Unable to find remote helper for 'https'
```

[solution: git-clone-fatal-unable-to-find-remote-helper-for-https](http://stackoverflow.com/questions/8329485/git-clone-fatal-unable-to-find-remote-helper-for-https)