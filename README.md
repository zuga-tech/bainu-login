# Bainu Login
## 概述
  目前Bainu登录支持**移动应用**和**移动网站**，是基于OAuth2.0协议标准构建的Bainu OAuth2.0授权登录系统。本文中主要介绍**移动网站**如何接入Bainu登录功能，如果移动应用想接入Bainu登录功能，请查看相关文档。

  移动网站OAuth2.0协议从Bainu5.2.0版本开始支持，低版本的Bainu不支持，开发者可以用UserAgent来判断Bainu版本。

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
|scope|是|应用授权作用域，snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid），snsapi_userinfo （弹出授权页面，获取授权码code，并通过授权码可以换取access_token）|
|state|是|重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节|

用户授权登录之后，网页重定向到redirect_uri，并带上code或open_id
```
redirect_url?code=CODE&state=STATE 或 redirect_uri?open_id=OPENID&state=STATE
```

### 2. 通过code换取网页授权access_token
第三方应用或网站获取授权码之后后台调用OAuth2.0服务器获取AccessToken，每次调用此接口都会重新生成新的AccessToken和RefreshToken以及新的过期时间。强烈建议不要通过客户端调用此接口，由于SecretKey和AccessToken都属于绝密信息，泄露可能带来无法挽回的损失。
```
http://bainu.zuga-tech.net/open/oauth2/access_token?app_id=APPID&secret_key=SECRET_KEY&code=CODE
```
|name|required|desc|
|----|--------|----|
|app_id|是|第三方应用唯一标识，由Bainu提供。|
|secret_key|是|第三方应用秘钥，由Bainu提供。|
|code|是|通过前面步骤获取的code。|

返回格式：
```
{
    'ET': int,
    'EM': string,
    'M': {
        'open_id': int,
        'access_token': string,
        'expires_in': int,
        'refresh_token': string
    }
}
```