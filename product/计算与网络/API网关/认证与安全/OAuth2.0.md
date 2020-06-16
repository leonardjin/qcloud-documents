## OAuth2.0 概述
OAuth2.0 是一个开放授权标准，它允许用户让**第三方应用**访问该用户在某服务的**特定私有资源**但是**不提供账号密码信息**给第三方应用。
>OAuth2.0 是一个授权协议，不是认证协议。

### OAuth2.0 角色说明
OAuth2.0 有以下四个角色： 
- Resource Owner：资源拥有者。
- Resource Server：资源服务器。
- Client：第三方应用客户端，指任何可以消费资源服务器的第三方应用。
- Authorization Server：授权服务器，管理上述三个角色的中间层。

### 授权流程
![](https://main.qcloudimg.com/raw/27814a835f31fd1511ef3247764ee1c7.png)
（A）：客户端请求资源所有者的授权。
（B）：资源所有者同意授权。
（C）：客户端获得了资源所有者的授权之后，向授权服务器申请授权令牌
（D）：授权服务器验证客户端无误后发放授权令牌
（E）：客户端拿到授权令牌之后请求资源服务器发送用户信息
（F）：资源服务器验证令牌无误后将用户信息发放给客户端


## API 网关 OAuth2.0 操作方法
   腾讯云API 网关对外提供了 OAuth2.0 功能，为了能够快速的使用该功能，首先提供简化版的使用方式，准备以下：
 - 分发token的授权服务器（当前提供python3的Demo），
 - API网关配置，同一个服务下创建一个授权API，一个业务API，其中授权API是用来校验请求业务API中携带的token是否合法，以及提供获取token的链接，

#### 授权服务器的搭建
代码路径： https://github.com/TencentCloud/apigateway-demo/tree/master/apigateway-oauth-python-demo
1. 生成 RSA 公私钥, 文件 produce_key.py,依赖库 python-jwt
执行以下代码，将生成三个文件，public_pem，priv_pem，pulic，其中public_pem，priv_pem分别为pem格式的公钥以及私钥，pulic为json格式的公钥，该文件的内容用于配置API网关的授权API，用于验证 JWT 签名，具体格式如下：
{"e":"AQAB","kty":"RSA","n":"43nSuC6lmGLogEPgFVwaaxAmPDzmZcocRB4Jed_dHc-sV7rcAcNB0iHyuGfNkfOAE2uhHVjdXuO6DBYGz4pnTwRZ5_wFrW0DlrlJQAXSvg6B2N1uda_aqySNw3rrvdh38rVG7HxFmyPbLXcpJtyfkiRNyZ1WhSpH0NciIRrFbW2mKRtOZsBGfBgmNqPGcGrMA71cuqNAQ9RMKmAF37iGXkx0tWMBQ_PL2aviHhtsiPbT3zIO7qUG3cleBHnS61kid3K8F38z9-5Hj-1zdTIP8iS4rAt4FmhvKvtOocRPYGq0W_dLLxmi4DYgIV2GJE93WyZ1EUvgRGhpcHvyT65z4w"}
私钥： AS 保存。

2. 运行授权服务， 文件 server.py
以下的服务是基于bottle搭建，运行前安装bottle，pip3 install bottle
首先执行produce_key.py生成三个文件后，再执行server.py，用于提供token，执行后可以简单验证生成token是否成功。
```
curl localhost:8080/token 
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTIyNzgwODksImZvbyI6ImJhciIsImlhdCI6MTU5MjI3Nzc4OSwianRpIjoibFY1TS10S2oxMEdtV0pJcHotM01GUSIsIm5iZiI6MTU5MjI3Nzc4OSwid3VwIjo5MH0.aHyZo2jgkNxVRDMtEiRBU4-n0pMfa0gocu92KQBe-nmbFoeI_5EWTJ8XFNnSIuoCAIFvrd9MSUX2DNVTg0woXukjoKOTjZSx4txknaXs1aApdvW74FVddCrMtdLrKh_VlwPOrEaOGesmtfcR3RN8xWnj1oedPW-HKPEqVpIAIIWO8ilCBFF-5yffcnFGIbfYO0t7OeBBviCQnQjWAmQHnteOZm0CBeG22k7rlnjH96qE_kyq7DHQqGmURjlpGxoXRC6E-AiV-3mYrCGnsAosEltuIUtq8VIbTZabSobFDE92C8us4GFtIVJQB2NWgeB3Hxgpz3Dlb4NCCcCkZbryEQ
```
#### 配置腾讯云API 网关的授权API，业务API
  创建授权API ，前提已经创建一个API网关服务，鉴权类型选择OAuth2.0,OAuth模式选择授权API，下一步后端配置选择自己的个人服务器地址，token位置选择header，公钥为4.1中执行文件produce_key.py生成的public文件中的内容，创建完点击完成。
  图片1
  图片2
  创建业务API，鉴权类型选择OAuth2.0,OAuth模式选择业务API，关联授权API选择刚刚创建的授权API，本示例中选择Mock类型，返回hello world。
  图片3
  图片4


#### 示例演示
  先获取token ，本文直接从授权服务器快速获取token，当然为保护授权服务器本身，可以请求API网关的授权API地址来获取。请求授权服务器获取token

```
请求授权API 获取token
curl http://service-cmrrdq86-1251890925.gz.apigw.tencentcs.com:80/token
返回结果：
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTIyNzk3MTAsImZvbyI6ImJhciIsImlhdCI6MTU5MjI3OTQxMCwianRpIjoiZlBGYlFZRkR4REx3d0lXTFl0aHBBQSIsIm5iZiI6MTU5MjI3OTQxMCwid3VwIjo5MH0.0JQquNRVCQ8n9hPV-mJi6Mku_7G3T1jFp68Sk2AYBijpzzBMQ1KOcREyo9G6QOpvdctynGOAPkL3cwqeTzkFhWgGj633pu_MdLjlectEBMGyVQIv6pL8OBMCHMQzTUTpHWJ_NoUkLpRLKGqZFFcXW8q7v4KeCbf8xHUa9OCH5VF2JxYOnFWDVgucSqao06r0Jaq64LDwKIhLw77ujheKpcBjRrf1kqoIpqk2qhb8CzxM36g_DawMadzKmX49dT-k7auNnI2xUtu5CZdXZ3lSmLeicXfGjc66rrH_acqUqipZRKeeQ5F3Ma467jPQaTeOKiCMHwS2_yp-sXNU2GzxOA
```

```
请求业务API
curl http://service-cmrrdq86-1251890925.gz.apigw.tencentcs.com:80/work -H'Authorization:Bearer id_token="eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTIyNzk3MTAsImZvbyI6ImJhciIsImlhdCI6MTU5MjI3OTQxMCwianRpIjoiZlBGYlFZRkR4REx3d0lXTFl0aHBBQSIsIm5iZiI6MTU5MjI3OTQxMCwid3VwIjo5MH0.0JQquNRVCQ8n9hPV-mJi6Mku_7G3T1jFp68Sk2AYBijpzzBMQ1KOcREyo9G6QOpvdctynGOAPkL3cwqeTzkFhWgGj633pu_MdLjlectEBMGyVQIv6pL8OBMCHMQzTUTpHWJ_NoUkLpRLKGqZFFcXW8q7v4KeCbf8xHUa9OCH5VF2JxYOnFWDVgucSqao06r0Jaq64LDwKIhLw77ujheKpcBjRrf1kqoIpqk2qhb8CzxM36g_DawMadzKmX49dT-k7auNnI2xUtu5CZdXZ3lSmLeicXfGjc66rrH_acqUqipZRKeeQ5F3Ma467jPQaTeOKiCMHwS2_yp-sXNU2GzxOA"'
返回结果：
hello world
```
