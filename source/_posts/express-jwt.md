---
title: express编写JWT中间件
date: 2016-11-26 11:17:00
categories: node.js
tags:
  - node.js
  - express
  - jwt
---
> 最近由于项目需要，学习了一下JWT这种身份验证的方法。在这里做一下总结。
> 本文主要是做一些总结，参考文章链接：[JSON Web Token - 在Web应用间安全地传递信息](http://blog.leapoahead.com/2015/09/06/understanding-jwt/)

# 什么是JWT

JWT是一种用于双方之间传递安全信息的简洁的、URL安全的表述性声明规范。
- 简洁(Compact): 可以通过URL，POST参数或者在HTTP header发送，因为数据量小，传输速度也很快;
- 自包含(Self-contained)：负载中包含了所有用户所需要的信息，避免了多次查询数据库;

## JWT结构简介

JWT结构:
- `Headers`头部
- `Payload`负载
- `Signature`签名

整体结构为:
`xxxxx.yyyyy.zzzzz`

### Headers

在`headers`中通常包含了两部分: `token`类型和采用的加密算法

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
- alg: 算法类型
- typ: token类型

### Payload

`Token`的第二部分时负载,它包含了`claim`,`Claim`是一些实体(通常指的用户)的状态和额外的元数据,有三种类型的`claim`: `reserved`,`public`和`private`.

- `Reserved claims`:这些`claim`是JWT预先定义的,在JWT中并不会强制使用它们,而是推荐使用,常用的有:
- iss(签发者)
- exp(过期时间戳)
- sub(面向的用户)
- aud(接收方)
- iat(签发时间)
- `Public claims`: 根据需要定义自己的字段,注意应该避免冲突
- `Private claims`: 这些是自定义的字段,可以用来在双方之间交换信息

举个栗子:
  ```json
  {
    "sub": "4008517517",
    "name": "John Doe",
    "admin": true
  }
  ```
### Signature

创建签名需要使用编码后的`headers`和`payload`以及一个密钥(secret),使用`headers`中制定签名进行签名算法进行签名.例如如果希望使用HMAC SHA256算法,那么签名应该使用下列方式创建:

```java
HMACSHA256(
base64UrlEncode(header) + "." +
base64UrlEncode(payload),
secret)
```
签名用于验证消息的发送者以及消息是没有经过篡改的.

### 完整的JWT
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0Nzk0NTYxMzI5NjJ9.yWGqnRYMhdLOEHlLI3_WIpHPtX3QHEdpJQHx_B0meaA
```
## 实现步骤

- 客户端发送用户名与密码到服务器
- 服务器进行信息核对
- 服务器生成token(`payload`中包含的为`user_id`)
- 服务器应用返回token,以后每次用户的请求都带上token
- 服务器应用验证token有效性.例如:签名是否正确?token是否过期?检查token的接收方是否为自己(可选)?
- 服务器应用返回响应的信息

## node模块[jwt-simple](https://github.com/hokaccha/node-jwt-simple)

使用`express`框架，`jwt-simple`模块来构建中间件（另外，我在github上边也找到[express-jwt](https://github.com/auth0/express-jwt)模块，好像也还不错的样子，但是没有实践过）。

### 产生token

```js
import * as express from 'express';
import * as bodyParser from 'body-parser';
import * as jwt from 'jwt-simple';

let app = express();

app.set('jwtTokenSecret', 'phpsucks!');
app.use(bodyParser.json());

app.post('/', (req, res) => {
  let user = req.body;
  // 产生token过期时间，这里设置
  let expires = Date.now() + 7*24*60*60*1000;
  let token = jwt.encode({
    iss: user.id, // issuer 表明请求的实体
    exp: expires, // expires token的生命周期
    aud: 'jser'
  }, app.get('jwtTokenSecret'));

  res.json({
    token: token,
    expires: expires
  });
});
```
### 获取token并解析
```js
app.get('/', (req, res) => {
  // 获取token,这里默认是放在headers的authorization
  let token = req.headers.authorization;
  if (token) {
    let decoded = jwt.decode(token, app.get('jwtTokenSecret'));
    // 判断是否token已过期以及接收方是否为自己
    if(decoded.exp <= Date.now() || decoded.aud !== 'jser') {
      res.sendStatus(401)
    } else {
      res.sendStatus(200)
    }
  } else {
    res.sendStatus(401)
  }
});
```

相对于常用的session而言，JWT通过将状态的记录放置于客户端，从而降低服务端因反复查询数据库而产生的压力。这应该就是JWT的最大优点了吧！