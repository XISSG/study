---
title: JWT
updated: 2023-07-19 15:51:55Z
created: 2023-07-19 15:51:45Z
latitude: 30.57281600
longitude: 104.06680100
altitude: 0.0000
---

# JWT
JWT （json web token）一种信息格式，多用于登录信息的验证
## 格式：
>hhhh.ppppp.ttttt（baseURL编码后的格式）

head包含数据格式JWT和MAC算法HS256等
claims包含自定义信息或标准信息
signature包含MAC信息对信息的完整性真实性进行验证

## 标准字段名称有：
### 标准声明（Registered Claims）
* iss（Issuer）：表示该JWT的签发者。
* sub（Subject）：表示该JWT所面向的用户。
* aud（Audience）：表示该JWT的接收者。
### 公共声明（Public Claims）
* exp（Expiration Time）：表示过期时间，即该JWT的有效期。
* nbf（Not Before）：表示在此之前，该JWT不应该被接受。
* iat（Issued At）：表示该JWT的签发时间。
* jti（JWT ID）：表示该JWT的唯一标识符。
