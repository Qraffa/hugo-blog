---
title: cookie和session以及token
date: 2020-04-24 16:59:50
tags:
- 计算机网络
- HTTP
categories:
- 计算机网络
---

### cookie和session以及token

#### cookie

是一种在客户端存储数据的具体实现方式.每次客户端请求都可以携带上cookie,存放一些相关数据信息.

#### session

是一种会话状态的概念.主要的实现是将sessionId保存在服务端,以此来验证客户端发来的session会话.用户在登录后服务器为其生成一个sessionId,并保存在服务端,再将该sessionId发送回客户端.以后客户端的请求需要带上该sessionId,服务端就可以根据该id来验证用户身份.

<!--more-->

#### cookie与session

客户端通常是使用cookie来保存用户的sessionId

#### token

是一种登录凭证.服务端根据客户端发来的信息,对其进行加密,然后把该字符串作为凭证发回给用户,以后用户请求时只要带上这个凭证,服务端就根据该凭证来解密,获得里面的相关信息.

JWT(JSON Web Token)

JWT只要由头部,负载以及签名组成.

头部主要存放的是关于jwt的一些信息,比如签名使用的算法,类型等.

负载主要存放的是jwt的一些需要传递的数据,包含以下.也可以定义自定义字段.

- iss (issuer)：签发人
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众
- nbf (Not Before)：生效时间
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

签名是通过密钥对前面两部分进行加密

JWT的缺陷,由于服务端不保存JWT的信息,因此JWT一旦签发出去,在到期之前,该JWT都是有效的.而且JWT本身包含了一些认证信息,存在风险.

