# Bainu Login
## 概述
  目前Bainu登录支持**移动应用**和**移动网站**，是基于OAuth2.0协议标准构建的Bainu OAuth2.0授权登录系统。本文中主要介绍**移动网站**如何接入Bainu登录功能，如果**移动应用**想接入Bainu登录功能，请查看相关文档。

  移动网站OAuth2.0协议从Bainu5.2.0版本开始支持，低版本的Bainu不支持，开发者可以用UserAgent来判断当前Bainu版本。

## 准备工作
### 1. 开发者认证
   目前Bainu开放平台暂未开启公开注册，所以请将你的应用信息及开发者信息发送到 business@zuga-tech.com ，我们审核通过后会联系你。
   需要提供的信息（移动网站）：
   - **应用名称**
   - **蒙文名称**
   - **LOGO** （640*640）
   - **官方网站**
   - **授权域名**
   - **蒙文介绍**
   - **团队或个人介绍**
   - **联系方式**

   邮件申请通过后会你将获得：
   - **appId**
   - **secret_key** （用于您的网站服务器和Bainu服务器通讯时验证，请妥善保管）
   - **jsapi_ticket** （用于生成JS-SDK权限验证的签名，请妥善保管）
   - **可使用的api列表** （允许调用的JS接口列表）

### 2. 流程概述
  Bainu OAuth2.0授权登录让Bainu用户使用Bainu身份安全登录第三方移动应用或移动网站，在Bainu用户授权登录已接入Bainu OAuth2.0的第三方移动应用或移动网站后，第三方可以获取到用户的接口调用凭证（access_token），通过access_token可以进行Bainu开放平台授权关系接口调用，从而可实现获取Bainu用户基本开放信息和帮助用户实现基础开放功能等。
  
  Bainu OAuth2.0授权登录目前支持authorization_code模式，适用于拥有server端的应用授权。该模式整体流程为：
  1. 第三方发起Bainu授权登录请求，Bainu用户允许授权第三方应用后，Bainu会拉起应用或重定向到第三方网站，并且带上授权临时票据code参数。
  2. 通过code参数加上AppID和SecretKey等，通过API换取access_token。
  3. 通过access_token进行接口调用，获取用户基本数据资源或帮助用户实现基本操作。
  
## 授权流程
### 1. 用户同意授权，获取code
需要让用户登录授权操作时请在Bainu里打开下面链接，请注意此网页只能在Bainu里打开。授权完成之后网页自动跳转到redirect_uri。

```
http://bainu.zuga-tech.net/open/oauth2/authorize?app_id=APPID&redirect_uri=REDIRECT_URI&scope=SCOPE&state=STATE
```
|name|required|desc|
|----|--------|----|
|app_id|是|第三方应用唯一标识，由Bainu提供。|
|redirect_uri|是|授权后重定向的回调链接地址，请使用**urlEncode**对链接进行处理。|
|scope|是|应用授权作用域，base （不弹出授权页面，直接跳转，只能获取用户openid），userinfo （弹出授权页面，获取授权码code，并通过授权码可以换取access_token）|
|state|是|重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节|

用户授权登录之后，网页重定向到redirect_uri，并带上code或open_id
```
redirect_url?code=CODE&state=STATE 或 redirect_uri?open_id=OPENID&state=STATE
```

### 2. 通过code换取网页授权access_token
第三方应用或网站获得授权码之后后台调用OAuth2.0服务器获取AccessToken，每次调用此接口都会重新生成新的AccessToken和RefreshToken以及新的过期时间。强烈建议不要通过客户端调用此接口，由于SecretKey和AccessToken都属于绝密信息，泄露可能带来无法挽回的损失。
```
GET/POST http://bainu.zuga-tech.net/open/oauth2/access_token?app_id=APPID&secret_key=SECRET_KEY&code=CODE
```
|name|required|desc|
|----|--------|----|
|app_id|是|第三方应用唯一标识，由Bainu提供。|
|secret_key|是|第三方应用秘钥，由Bainu提供。|
|code|是|用户授权时获得的code。|

返回格式：
```
{
    ET: int,
    EM: string,
    M: {
        open_id: int,
        access_token: string,
        expires_in: int,
        refresh_token: string
    }
}
```
|name|type|desc|
|----|----|----|
|ET|int|Error Type， ET=0表明成功，否则失败。请看错误码列表。|
|EM|string|Error Message，错误描述。|
|M|array|数据部分，只有ET=0时有。|
|open_id|int|授权用户唯一标识，同一个第三方应用内保证唯一，同一个Bainu用户在同一个第三方应用上多次登录时此值不变。|
|access_token|string|接口调用凭证，应该存储在第三方应用的服务器端，不要传递给客户端。|
|expires_in|int|access_token有效期（秒），一般为2个小时。|
|refresh_token|string|刷新access_token时用到。|

### 3. 刷新access_token（如果需要）
access_token是调用授权关系接口的调用凭证，由于access_token有效期（目前为2个小时）较短，当access_token超时后，可以使用refresh_token进行刷新，access_token刷新结果有两种
1. 若access_token已超时，那么进行refresh_token会获取一个新的access_token，新的超时时间。
2. 若access_token未超时，那么进行refresh_token不会改变access_token，但超时时间会刷新，相当于续期access_token。

需要注意的是如果refresh_token过期的话，只能让用户重新授权了。
```
GET/POST http://bainu.zuga-tech.net/open/oauth2/refresh_token?open_id=OPENID&refresh_token=REFRESH_TOKEN
```
|name|required|desc|
|----|--------|----|
|open_id|是|授权用户唯一标识，同一个第三方应用内保证唯一，同一个Bainu用户在同一个第三方应用上多次登录时此值不变。|
|refresh_token|是|通过上面步骤获得的refresh_token。|

返回格式：
```
{
    ET: int,
    EM: string,
    M: {
        open_id: int,
        access_token: string,
        expires_in: int,
        refresh_token: string
    }
}
```
|name|type|desc|
|----|----|----|
|ET|int|Error Type， ET=0表明成功，否则失败。请看错误码列表。|
|EM|string|Error Message，错误描述。|
|M|array|数据部分，只有ET=0时有。|
|open_id|int|授权用户唯一标识，同一个第三方应用内保证唯一，同一个Bainu用户在同一个第三方应用上多次登录时此值不变。|
|access_token|string|接口调用凭证，应该存储在第三方应用的服务器端，不要传递给客户端。|
|expires_in|int|access_token有效期（秒），一般为2个小时。|
|refresh_token|string|刷新access_token时用到。|

### 4. 检验access_token是否有效（如果需要）
```
GET/POST http://bainu.zuga-tech.net/open/oauth2/auth?open_id=OPENID&access_token=ACCESS_TOKEN
```
|name|required|desc|
|----|--------|----|
|open_id|是|授权用户唯一标识，同一个第三方应用内保证唯一，同一个Bainu用户在同一个第三方应用上多次登录时此值不变。|
|access_token|是|接口调用凭证，应该存储在第三方应用的服务器端，不要传递给客户端。|

返回格式：
```
{
    ET: int,
    EM: string
}
```
|name|type|desc|
|----|----|----|
|ET|int|Error Type， ET=0表明access_token有效。请看错误码列表。|
|EM|string|Error Message，错误描述。|

### 5. 通过access_token拉取用户信息
```
GET/POST http://bainu.zuga-tech.net/open/oauth2/user_info?open_id=OPENID&access_token=ACCESS_TOKEN
```
|name|required|desc|
|----|--------|----|
|open_id|是|授权用户唯一标识，同一个第三方应用内保证唯一，同一个Bainu用户在同一个第三方应用上多次登录时此值不变。|
|access_token|是|接口调用凭证，应该存储在第三方应用的服务器端，不要传递给客户端。|

返回格式：
```
{
    ET: int,
    EM: string,
    M: {
        open_id: int,
        nick_name: string,
        profile: string,
        profile_thumb: string,
        gender: int
    }
}
```
|name|type|desc|
|----|----|----|
|ET|int|Error Type， ET=0表明成功，否则失败。请看错误码列表。|
|EM|string|Error Message，错误描述。|
|M|array|数据部分，只有ET=0时有。|
|open_id|int|授权用户唯一标识，同一个第三方应用内保证唯一，同一个Bainu用户在同一个第三方应用上多次登录时此值不变。|
|nick_name|string|用户昵称|
|profile|string|用户头像|
|profile_thumb|string|用户头像缩略图|
|gender|int|用户性别，0:女，1:男|

## 错误码
|ET |desc|
|---|----|
|0|成功|
|1|失败|
|2|参数错误|
|3|第三方应用不可用|
|4|用户不可用|
|5|Secret key 不正确|
|6|授权码错误|
|7|Access token错误|
|8|Access token过期|
|9|Refresh token错误|
|10|Refresh token过期|
