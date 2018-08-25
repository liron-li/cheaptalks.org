---
title: "JWT简明入门手册"
date: 2017-08-30T01:37:56+08:00
lastmod: 2017-08-30T01:37:56+08:00
draft: false
tags: ["JWT", "协议"]
categories: []
author: "Liron"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

#### 什么是JWT
JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。

#### JWT的组成
JWT由3部分组成，分别是头部、载荷、和签名

##### 头部(heander)
JWT的头部中记录数据的类型和加密方式

```
{
  "typ": "JWT",
  "alg": "HS256"
}
```

##### 载荷(Payload)
Payload 是jwt的主体内容，页面间传递的信息会放在这里

```
{
    "iss": "mellifluous notes",
    "iat": 1490166407,
    "exp": 1491166407,
    "aud": "mellifluous.cn",
    "sub": "xx@example.com",
    "article_id": "1",
    "user_id": "2"
}
```

- `iss` : jwt的颁发者
- `iat` : 颁发时间
- `exp` : 有效期
- `aud` : audience 听众 受众
- `sub` : subject 主题

其他的字段为自定义内容

##### 签名
用于证明数据的来源，由加密算法生成

##### 完整的jwt示例（以php演示，其他语言原理一样）
- 首先对头部和载荷json字符进行`base64_encode`，然后将base64之后获得的字符用`.`连接

	```

		 base64_encode('{"typ":"JWT","alg":"HS256"}')

		base64_encode('{"iss":"mellifluous notes","iat":1490166407,"exp":1491166407,"aud":"mellifluous.cn","sub":"xx@example.com","article_id":"1","user_id":"2"}')
	```
	
	得到 eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJtZWxsaWZsdW91cyBub3RlcyIsImlhdCI6MTQ5MDE2NjQwNywiZXhwIjoxNDkxMTY2NDA3LCJhdWQiOiJtZWxsaWZsdW91cy5jbiIsInN1YiI6Inh4QGV4YW1wbGUuY29tIiwiYXJ0aWNsZV9pZCI6IjEiLCJ1c2VyX2lkIjoiMiJ9

- 对拼接的字符串进行加密生成的字符串就是签名。这里用HS256算法演示
	
	```

		base64_encode(hash_hmac('sha256', 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJtZWxsaWZsdW91cyBub3RlcyIsImlhdCI6MTQ5MDE2NjQwNywiZXhwIjoxNDkxMTY2NDA3LCJhdWQiOiJtZWxsaWZsdW91cy5jbiIsInN1YiI6Inh4QGV4YW1wbGUuY29tIiwiYXJ0aWNsZV9pZCI6IjEiLCJ1c2VyX2lkIjoiMiJ9', 'secret', true))
		
	```
	
	得到`7WJ3mX2lXR86E9tBfOU2y2UwOVLY00WlFFd23xw2im4=`

- 将头部、载荷、签名用`.`拼接即是一个完整的jwt

##### 服务器端获取jwt并验证来源
服务器端从get参数中取出jwt的数据。并根据加密算法进行验证来源