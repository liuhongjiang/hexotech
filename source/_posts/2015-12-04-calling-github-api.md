title: 脚本调用github API
date: 2015-12-04 10:39:42
tags:
catgories: github
---

最近在部署项目的jenkins环境的时候，希望实现一个功能， 就是每次build完以后，对比一下build的分支是否是github上面的最新代码。

所以就想在jenkins中使用脚本去github获取最新的commit信息。这个可以通过github的api实现。

## 直接访问api directly

最简单的方式是通过用户名和密码直接访问github的api：

```
 curl -u "liuhongjiang:{password}" https://api.github.com
```

<!-- more -->

## 使用personal token
github的修改个人设置中的`Personal access tokens`中生成一个token, 生成是可以选择权限，然后可以将这个token作为oAuth的token使用。

注意，这个toke只会在生成的时候看见一次，以后就看不见了，所以需要保持好。

下面是访问方式， token是`5c67b248e17baxxxxxxxxxxxxxxxxxxxxxxx`:

```
curl  https://api.github.com/?access_token=5c67b248e17baxxxxxxxxxxxxxxxxxxxxxxx  
curl -H "Authorization:token 5c67b248e17baxxxxxxxxxxxxxxxxxxxxxxx" https://api.github.com
```

## application oauth
同样在github的修改个人设置中的`application`中的`developer application`的tab中，申请一个应用，这样就可以用这个应用来生成client id和client securet，然后就可以使用它们来生成token：

使用程序生成，参考文档：[https://developer.github.com/v3/oauth_authorizations/#oauth-authorizations-api](https://developer.github.com/v3/oauth_authorizations/#oauth-authorizations-api)


下面是一个生成token的示例：
```
# curl -u"liuhongjiang:{password}" -X POST --data '{"scopes":["public_repo"], "note":"admin script", "client_id": "6f82bb4dxxxxxxxxxx", "client_secret": "4f5be7f1750785eac2exxxxxxxxxxxxxxxxxxxxxxxx"}' https://api.github.com/authorizations     
{
  "id": 25261111,
  "url": "https://api.github.com/authorizations/25261111",
  "app": {
    "name": "mybuildapp",
    "url": "http://localhost:8888/",
    "client_id": "6f82bb4dxxxxxxxxxx"
  },
  "token": "0ce8d9d6e5cff9c6096b610a2xxxxxxxxx",
  "hashed_token": "daf93cdfc81f432b8d18790dfc56d73ea401ae499xxxxxxxxxxxxxxxxxxxxx",
  "token_last_eight": "ac18xxxxx",
  "note": "admin script",
  "note_url": null,
  "created_at": "2015-12-03T06:56:28Z",
  "updated_at": "2015-12-03T06:56:28Z",
  "scopes": [
    "public_repo"
  ],
  "fingerprint": null
}
```

接下来就可以使用上面的`"token": "0ce8d9d6e5cff9c6096b610a2xxxxxxxxx"`访问github api了：

```
curl  https://api.github.com/?access_token=0ce8d9d6e5cff9c6096b610a2xxxxxxxxx  
curl -H "Authorization:token 0ce8d9d6e5cff9c6096b610a2xxxxxxxxx" https://api.github.com
```

但发现一个问题，如果我的账号加入了一个私有项目，好像无法通过这个oauth token访问那个私有项目。上面的personal token是可以访问的。

## 参考

[github Authentication](https://developer.github.com/v3/#authentication)
[Create a new authorization](https://developer.github.com/v3/oauth_authorizations/#create-a-new-authorization)
[Web Application Flow](https://developer.github.com/v3/oauth/#web-application-flow)
