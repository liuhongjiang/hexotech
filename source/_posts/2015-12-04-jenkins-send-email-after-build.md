title: 配置jenkins发送邮件
date: 2015-12-04 11:06:24
tags:
categories: jenkins
---

最近在部署项目的jenkins环境的时候，希望实现一个功能， 就是每次build完以后，对比一下build的分支是否是github上面的最新代码。

然后再以邮件把检查的内容发出来。

## 访问github获取最新commit信息

关于如何实现访问github api，请参考上一篇文章[脚本调用github API](/2015/12/04/calling-github-api/)。访问的api是： [List commits on a repository](https://developer.github.com/v3/repos/commits/#list-commits-on-a-repository)

这个api可以通过路径访问具体的分支。返回信息如下， 其中的sha就是我们需要的commit信息：

```
# curl -H "Authorization:token 5c67b248e17ba2f225fd4fa52ca7xxxxxxxxxxxxx" https://api.github.com/repos/liuhongjiang/gittest/commits/release/v3
{
  "sha": "a20fbae206c9b31187ed70f0901ef08cf2eeb26a",
  "commit": {
    "author": {
      "name": "liuhongjiang",
      "email": "andrew.lhj@gmail.com",
      "date": "2015-07-28T00:56:55Z"
    },
    "committer": {
      "name": "liuhongjiang",
      "email": "andrew.lhj@gmail.com",
      "date": "2015-07-28T00:56:55Z"
    },
    "message": "v3",
    "tree": {
      "sha": "b847aed05d03a78d0ab53189a371ba40dea50a62",
      "url": "https://api.github.com/repos/liuhongjiang/gittest/git/trees/b847aed05d03a78d0ab53189a371ba40dea50a62"
    },
    "url": "https://api.github.com/repos/liuhongjiang/gittest/git/commits/a20fbae206c9b31187ed70f0901ef08cf2eeb26a",
    "comment_count": 0
  },
  .......
}
```

<!--more-->

## jenkins发送邮件

首先需要安装插件： [Email-ext plugin](https://wiki.jenkins-ci.org/display/JENKINS/Email-ext+plugin)

然后在项目配置的`Post-build Actions`里面可以添加'Editable Email Notification'.

之后可以就配置邮件的内容了。注意可以配置和修改邮件的trigger，我这里选择的是always。

先上图：
{% img center /images/blogimages/2015/jenkins_email.png %}


上面比较关键的地方是： 我选择了content type为html，default content和pre-send script。

由于我检查github的commit信息，只能在pre-send脚本中完成，所以我的邮件内容是通过default content和pre-send script生成的内容拼接起来的。

default content的内容如下：
```
</h2>Jenkins gittest build </h2>
<p>build branch: $GIT_BRANCH</p>
<p>build number: $BUILD_NUMBER</p>
<p>commit number: $GIT_COMMIT</p>
<p>package download url: build package</p>
```

pre-send script是groovy的脚本，内容如下：

```groovy
import groovy.json.JsonSlurper;

// get git branch
def var = build.getEnvVars()
var.each{k, v -> println("${k}:${v}") }
def branch = var.get("GIT_BRANCH")
def path_part = branch.split("/")
branch = "${path_part[1]}/${path_part[2]}"
def buildCommit = var.get("GIT_COMMIT")
def buildNumer = var.get("BUILD_NUMBER")

// get the latest commit of the branch on github
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = "curl https://api.github.com/repos/liuhongjiang/gittest/commits/${branch}?access_token=5c67b248e17ba2f225fd4fa5xxxxxxxxxxx".execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(5000)
logger.print("out> $sout err> $serr")
def jsonSlurper = new JsonSlurper()
def object = jsonSlurper.parseText(sout.toString())
def githubCommit = object.get("sha")


// get the email content
def emailContent = msg.getContent().getBodyPart(0).getContent()
//logger.print("$emailContent")


// subject
def subject = msg.getSubject()
subject += "-${branch}-" + buildNumer.padLeft(4, '0') + "-"
subject += buildCommit[0..7]
msg.setSubject(subject)
logger.print("Subject is: $subject")


//msg.setText("$emailContent, \n out> $sout err> $serr")
String newContent = """$emailContent
<h2>Current commit on github</h2>
<p> the branch is: $branch </p>
<p> current commit is: $githubCommit</p>"""

if (githubCommit == buildCommit) {
	newContent += """<h2>build commit and github latest commit <span style="color:green;font-weight: bold;">Match</span></h2>"""
} else {
	newContent += """<h2>build commit and github latest commit <span style="color:red;font-weight: bold;">Don't Match</span></h2>"""
}


msg.setContent(newContent, "text/html; charset=utf-8");
```

pre-send script自带了几个变量，分别msg, build, logger和cacel，我上面脚本的中比较关键的地方是对msg和build的处理，例如：

获取环境变量:
```
def var = build.getEnvVars()
def branch = var.get("GIT_BRANCH")
def buildCommit = var.get("GIT_COMMIT")
def buildNumer = var.get("BUILD_NUMBER")
```

获取和设置邮件的内容：
```
def emailContent = msg.getContent().getBodyPart(0).getContent()
msg.setContent(newContent, "text/html; charset=utf-8");
```

邮件内容的处理比较复杂，我这里只是简单的硬编码处理，如果上面选择的邮件content type变化了，这段代码就失效了。

获取和设置邮件的标题
```
def subject = msg.getSubject()
subject += "-${branch}-" + buildNumer.padLeft(4, '0') + "-"
subject += buildCommit[0..7]
msg.setSubject(subject)
```

其他操作参加msg和build对应的java类

[hudson.model.AbstractBuild 类](http://javadoc.jenkins-ci.org/hudson/model/AbstractBuild.html)

[msg MimeMessage 类](https://docs.oracle.com/javaee/6/api/javax/mail/internet/MimeMessage.html)

另外其实`Email-ext plugin`支持jelly这样的模板，但是配置还需要jenkins的admin到server上操作，我没有实验过，有兴趣的读者可以自己去试一下。
