title: 在Jenkins配置github的pull request builder
date: 2016-05-27 19:20:43
tags:
categories: jenkins
---

最近做的一个项目中使用了github的enterprise版本，然后之前的一位同事搭建了在github上提交pull request可以出发jenkins builder。

学习了这一过程，记录一下blog。

## 准备实验项目和jenkins

代码使用openstack的[python-cinderclient](https://github.com/openstack/python-cinderclient)为例：

下载代码，运行这个项目的单元测试命令`tox`，保证单元测试能够正常通过，这是一个python的项目，运行了py34
和py27，pep8的检查，运行结果如下：

```bash
......
_________________ summary _________________
  py34: commands succeeded
  py27: commands succeeded
  pep8: commands succeeded
  congratulations :)

```

然后准备jenkins，这里的build任务是在slave节点上运行的，slave安装的是centos7。

在jenkins中安装以下插件：

- SCM:
  - [Git Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin)
- Wrappers:
  - [Build Timeout Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build-timeout+Plugin)
- Triggers:
  - [GitHub Plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+Plugin)
  - [GitHub Pull Request Builder Plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+pull+request+builder+plugin)
- Publishers:
  - [Mattermost Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Mattermost+Plugin)

然后在slave节点上安装python的运行环境

```bash
# EPEL
yum -y install epel-release
# Development packages
yum -y groupinstall "Development tools"
yum -y install openssl-devel readline-devel bzip2-devel sqlite-devel zlib-devel ncurses-devel db4-devel expat-devel
yum -y install python-devel mysql-devel libxslt-devel libffi-devel
yum -y install libmemcached-devel
yum -y install libevent-devel
yum -y python-virtualenv
# Python 34
yum -y install python34 python34-devel
```

在slave节点上，切换到jenkins的账号

```
# Enter bash using jenkins users
su -s /bin/bash jenkins
```

然后需要将github的站点添加到slave node的knowhost里面，通过运行`ssh git@github.com`可以添加。


## 在jenkins上配置pull request builder


在jenkins中创建一个freestyle project, 命名为python-cinderclient-pr-builder。

注：我这个项目是创建在github的enterprise中的，可以等同于github。主要原因是我的jenkins没有外网ip，github无法访问，就不能设置webhook，无法实现根据pull request自动trigger build。

1. 填入project url

  {% img center /images/blogimages/2016/jenkins_pull_request_builder/github_option.png %}

2. 配置pull request build

  - 安装了[GitHub Pull Request Builder Plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+pull+request+builder+plugin)之后

  - Go to `Manage Jenkins -> Configure System -> GitHub pull requests builder` section

    如图下图配置
    {% img center /images/blogimages/2016/jenkins_pull_request_builder/ghprb_config.png %}

    需要注意：GitHub API URI是"https://api.github.com". 而GitHub Enterprise API URL使用路径"/api/v3"。

3. 添加Personal access tokens

  进入github的个人的profile页面，然后添加Personal access tokens的页面中，generate new token，如下图选择，重点是要选admin：repo_hook

  {% img center /images/blogimages/2016/jenkins_pull_request_builder/personal_token.png %}

4. 添加credential的配置
  生成token以后，在jenkins的`creadentials`里面添加一个新的证书，如图，password项就填刚才在github上生成的token

  {% img center /images/blogimages/2016/jenkins_pull_request_builder/add_credential.png %}

## 添加一个jenkins job


参考以下步骤，这个步骤来自于[ghprb插件的README](https://github.com/janinko/ghprb#creating-a-job)

* Create a new job.  
* Add the project's GitHub URL to the ``GitHub project`` field (the one you can enter into browser. eg: ``https://github.com/janinko/ghprb``)  
* Select Git SCM.  
* Add your GitHub ``Repository URL``.  
* Under Advanced, set ``Name`` to ``origin`` and:
  * If you **just** want to build PRs, set ``refspec`` to ``+refs/pull/*:refs/remotes/origin/pr/*``
  * If you want to build PRs **and** branches, set ``refspec`` to ``+refs/heads/*:refs/remotes/origin/* +refs/pull/*:refs/remotes/origin/pr/*`` (see note below about [parameterized builds](#parameterized-builds))
* In ``Branch Specifier``, enter ``${sha1}`` instead of the default ``*/master``.
* If you want to use the actual commit in the pull request, use ``${ghprbActualCommit}`` instead of ``${sha1}``
* Under ``Build Triggers``, check ``GitHub pull requests builder``.
  * Add admins for this specific job.  
  * If you want to use GitHub hooks for automatic testing, read the help for ``Use github hooks for build triggering`` in job configuration. Then you can check the checkbox.
  * In Advanced, you can modify:  
    * The crontab line for this specific job. This schedules polling to GitHub for new changes in Pull Requests.  
    * The whitelisted users for this specific job.  
    * The organisation names whose members are considered whitelisted for this specific job.  
* Save to preserve your changes.  

更具体的内容可以参考：

- [ghprb wiki](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+pull+request+builder+plugin)
- [github ghprb](https://github.com/janinko/ghprb)

完成以上的操作，这个插件就会在github项目的hooks和services里面自动添加一个webhook，如图：

{% img center /images/blogimages/2016/jenkins_pull_request_builder/webhook.png %}

## 添加build status description
------

添加一个在`post build action`中选择`set build status on Github commit`

{% img center /images/blogimages/2016/jenkins_pull_request_builder/set_build_status.png %}

然后在每一个build就会有一个status

{% img center /images/blogimages/2016/jenkins_pull_request_builder/build_status.png %}


## mattermost


ToDo

## jenkins job builder


ToDo

Please visit http://docs.openstack.org/infra/jenkins-job-builder/index.html for more details about those jenkins jobs.


## 一些有用的jenkins插件
******

- SCM:
  - [Git Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin)
- Wrappers:
  - [Ansi Color Plugin](https://wiki.jenkins-ci.org/display/JENKINS/AnsiColor+Plugin)
  - [Build Timeout Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build-timeout+Plugin)
  - [Timestamper Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Timestamper)
- Triggers:
  - [GitHub Plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+Plugin)
  - [GitHub Pull Request Builder Plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+pull+request+builder+plugin)
- Publishers:
  - [Cobertura Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Cobertura+Plugin)
  - [Mattermost Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Mattermost+Plugin)

reference
======

[install jenkins on centos](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Red+Hat+distributions)
