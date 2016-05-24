---
layout: post
title: JWT notes
category: note
excerpt: npm module JWT-simple
---
随便扯一扯，很多人都说他安全 其实真的挺呵呵的 传输过程中 payload 只要能被截获，信息很容易就泄露出来了 所以这里面id 这种无关紧要的信息就行了 别闲着没事放密码进去

### JWT的组成

一个JWT实际上就是一个字符串，它由三部分组成，**头部**、**载荷**与**签名**。

我们先将上面的添加好友的操作描述成一个JSON对象。其中添加了一些其他的信息，帮助今后收到这个JWT的服务器理解这个JWT。

```
{
    "iss": "Jimmy",
    "iat": 1441593502,
    "exp": 1441594722,
    "aud": "www.example.com",
    "sub": "rssssss@example.com",
    "from_user": "B",
    "target_user": "A"
}
```

这里面的前五个字段都是由JWT的标准所定义的。

- `iss`: 该JWT的签发者
- `sub`: 该JWT所面向的用户
- `aud`: 接收该JWT的一方
- `exp`(expires): 什么时候过期，这里是一个Unix时间戳
- `iat`(issued at): 在什么时候签发的



以node 为例子(基于express)，先去npm上拉一个封装好的dependency  `npm install jwt-simple`

生成token:

```
//middleware

app.set('jwtTokenSecret', 'PEEPING_TOKEN');

//route路由(模拟前端一个用户登陆 信息匹配后生成token)
var expires = moment().add(90,'days').valueOf;
                var token = jwt.encode({
                  iss: user._id,
                  exp: expires
                }, app.get('jwtTokenSecret'));
                
```

验证token:

```
var token = req.headers['access_token'];
  console.log(req.headers);
  if (token) {
    try {
      var decoded = jwt.decode(token, app.get('jwtTokenSecret'));
      if (decoded.exp <= Date.now()) {
        res.send(403, 'Access token has expired');
      }else{
        req.userId =  decoded.iss;
        next();
      }
    } catch (err) {
      res.send(500, 'internal error');
    }
  } else {
    res.send(403, 'unauthorized');
  }
```

So easy ,对吧。

