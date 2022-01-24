# 理论基础

## Oauth1.0和Oauth2.0

https://www.zhihu.com/question/19851243

```
+----------+                                           +----------+
 |          |--(A)- Obtaining a Request Token --------->|          |
 |          |                                           |          |
 |          |<-(B)- Request Token ----------------------|          |
 |          |       (Unauthorized)                      |          |
 |          |                                           |          |
 |          |      +--------+                           |          |
 |          |>-(C)-|       -+-(C)- Directing ---------->|          |
 |          |      |       -+-(D)- User authenticates ->|          |
 |          |      |        |      +----------+         | Service  |
 | Consumer |      | User-  |      |          |         | Provider |
 |          |      | Agent -+-(D)->|   User   |         |          |
 |          |      |        |      |          |         |          |
 |          |      |        |      +----------+         |          |
 |          |<-(E)-|       -+-(E)- Request Token ------<|          |
 |          |      +--------+      (Authorized)         |          |
 |          |                                           |          |
 |          |--(F)- Obtaining a Access Token ---------->|          |
 |          |                                           |          |
 |          |<-(G)- Access Token -----------------------|          |
 +----------+                                           +----------+
```

**OAuth**1.0a

**OAuth**1.0存在[安全漏洞](http://oauth.net/advisories/2009-1/).

简单点来说，这是一种会话固化攻击，和常见的会话劫持攻击不同的是，在会话固化攻击中，攻击者会初始化一个合法的会话，然后诱使用户在这个会话上完成后续操作，从而达到攻击的目的。反映到**OAuth**1.0上，攻击者会先申请Request Token，然后诱使用户授权这个Request Token，接着针对回调地址的使用，又存在以下几种攻击手段：

- 如果Service Provider没有限制回调地址（应用设置没有限定根域名一致），那么攻击者可以把**oauth**_callback设置成成自己的URL，当User完成授权后，通过这个URL自然就能拿到User的Access Token。
- 如果Consumer不使用回调地址（桌面或手机程序），而是通过User手动拷贝粘贴Request Token完成授权的话，那么就存在一个竞争关系，只要攻击者在User授权后，抢在User前面发起请求，就能拿到User的Access Token。

为了修复安全问题，[OAuth1.0a](http://oauth.net/core/1.0a/)出现了（[RFC5849](http://tools.ietf.org/html/rfc5849)），主要修改了以下细节：

- Consumer申请Request Token时，必须传递**oauth**_callback，而Consumer申请Access Token时，不需要传递**oauth**_callback。通过前置**oauth**_callback的传递时机，让**oauth**_callback参与签名，从而避免攻击者假冒**oauth**_callback。
- Service Provider获得User授权后重定向User到Consumer时，返回**oauth**_verifier，它会被用在Consumer申请Access Token的过程中。攻击者无法猜测它的值。

**OAuth**2.0

**OAuth**1.0虽然在安全性上经过修补已经没有问题了，但还存在其它的缺点，其中最主要的莫过于以下两点：其一，签名逻辑过于复杂，对开发者不够友好；其二，授权流程太过单一，除了Web应用以外，对桌面、移动应用来说不够友好。

为了弥补这些短板，[OAuth2.0](http://tools.ietf.org/html/draft-ietf-oauth-v2)做了以下改变：

首先，去掉签名，改用SSL（HTTPS）确保安全性，所有的token不再有对应的secret存在，这也直接导致**OAuth**2.0不兼容老版本。

其次，针对不同的情况使用不同的授权流程，和老版本只有一种授权流程相比，新版本提供了四种授权流程，可依据客观情况选择。

## Oauth2.0

### oauth2.0的简单解释

http://www.ruanyifeng.com/blog/2019/04/oauth_design.html

令牌（token）与密码（password）的作用是一样的，都可以进入系统，但是有三点差异。

（1）令牌是短期的，到期会自动失效，用户自己无法修改。密码一般长期有效，用户不修改，就不会发生变化。

（2）令牌可以被数据所有者撤销，会立即失效。以上例而言，屋主可以随时取消快递员的令牌。密码一般不允许被他人撤销。

（3）令牌有权限范围（scope），比如只能进小区的二号门。对于网络服务来说，只读令牌就比读写令牌更安全。密码一般是完整权限。

上面这些设计，保证了令牌既可以让第三方应用获得权限，同时又随时可控，不会危及系统安全。这就是 OAuth 2.0 的优点。

### Oauth2.0的四种方式

https://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html

OAuth 2.0 的标准是 [RFC 6749](https://tools.ietf.org/html/rfc6749) 文件。该文件先解释了 OAuth 是什么。

```OAuth 引入了一个授权层，用来分离两种不同的角色：客户端和资源所有者。......资源所有者同意以后，资源服务器可以向客户端颁发令牌。客户端通过令牌，去请求数据。```

也就是说，**OAuth 2.0 规定了四种获得令牌的流程。你可以选择最适合自己的那一种，向第三方应用颁发令牌。**下面就是这四种授权方式。

```
授权码（authorization-code）

隐藏式（implicit）

密码式（password）：

客户端凭证（client credentials）
```

**不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：客户端 ID（client ID）和客户端密钥（client secret）。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的。**

- 授权码

**授权码（authorization code）方式，指的是第三方应用先申请一个授权码，然后再用该码获取令牌。**

这种方式是最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。

第一步，A 网站提供一个链接，用户点击后就会跳转到 B 网站，授权用户数据给 A 网站使用。下面就是 A 网站跳转 B 网站的一个示意链接。

```javascript
https://b.com/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

上面 URL 中，`response_type`参数表示要求返回授权码（`code`），`client_id`参数让 B 知道是谁在请求，`redirect_uri`参数是 B 接受或拒绝请求后的跳转网址，`scope`参数表示要求的授权范围（这里是只读）。

第二步，用户跳转后，B 网站会要求用户登录，然后询问是否同意给予 A 网站授权。用户表示同意，这时 B 网站就会跳回`redirect_uri`参数指定的网址。跳转时，会传回一个授权码，就像下面这样。

```javascript
https://a.com/callback?code=AUTHORIZATION_CODE
```

上面 URL 中，`code`参数就是授权码。

第三步，A 网站拿到授权码以后，就可以在后端，向 B 网站请求令牌。

```javascript
https://b.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

上面 URL 中，`client_id`参数和`client_secret`参数用来让 B 确认 A 的身份（`client_secret`参数是保密的，因此只能在后端发请求），`grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码，`code`参数是上一步拿到的授权码，`redirect_uri`参数是令牌颁发后的回调网址。

第四步，B 网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`指定的网址，发送一段 JSON 数据。

```javascript
{    
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```

上面 JSON 数据中，`access_token`字段就是令牌，A 网站在后端拿到了。

- 隐藏式

有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。**RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。**

第一步，A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用。

```
https://b.com/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

上面 URL 中，`response_type`参数为`token`，表示要求直接返回令牌。

第二步，用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回`redirect_uri`参数指定的跳转网址，并且把令牌作为 URL 参数，传给 A 网站。

```javascript
https://a.com/callback#token=ACCESS_TOKEN
```

上面 URL 中，`token`参数就是令牌，A 网站因此直接在前端拿到令牌。

注意，令牌的位置是 URL 锚点（fragment），而不是查询字符串（querystring），这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险。

这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

- 密码式

**如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。**

第一步，A 网站要求用户提供 B 网站的用户名和密码。拿到以后，A 就直接向 B 请求令牌。

```javascript
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
```

上面 URL 中，`grant_type`参数是授权方式，这里的`password`表示"密码式"，`username`和`password`是 B 的用户名和密码。

第二步，B 网站验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A 因此拿到令牌。

这种方式需要用户给出自己的用户名/密码，显然风险很大，因此只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。

- 凭证式

**最后一种方式是凭证式（client credentials），适用于没有前端的命令行应用，即在命令行下请求令牌。**

第一步，A 应用在命令行向 B 发出请求。

```javascript
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

第二步，B 网站验证通过以后，直接返回令牌。

这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

> https://datatracker.ietf.org/doc/html/rfc6749#section-4.4
>
> 一般用来访问用户上下文的信息,而不是用户的信息.

- 令牌的使用

A 网站拿到令牌以后，就可以向 B 网站的 API 请求数据了。

此时，每个发到 API 的请求，都必须带有令牌。具体做法是在请求的头信息，加上一个`Authorization`字段，令牌就放在这个字段里面。

```bash
curl -H "Authorization: Bearer ACCESS_TOKEN" \
"https://api.b.com"
```

上面命令中，`ACCESS_TOKEN`就是拿到的令牌。

- 更新令牌

令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，很可能体验不好，而且也没有必要。OAuth 2.0 允许用户自动更新令牌。

具体方法是，B 网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌（refresh token 字段）。令牌到期前，用户使用 refresh token 发一个请求，去更新令牌。

```javascript
https://b.com/oauth/token?
  grant_type=refresh_token&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&
  refresh_token=REFRESH_TOKEN
```

上面 URL 中，`grant_type`参数为`refresh_token`表示要求更新令牌，`client_id`参数和`client_secret`参数用于确认身份，`refresh_token`参数就是用于更新令牌的令牌。

B 网站验证通过以后，就会颁发新的令牌。

### gitlab oauth第三方登录示例教程

如何通过 OAuth 获取 API 数据。

很多网站登录时，允许使用第三方网站的身份，这称为"第三方登录"。

- 第三方登录的原理

所谓第三方登录，实质就是 OAuth 授权。用户想要登录 A 网站，A 网站让用户提供第三方网站的数据，证明自己的身份。获取第三方网站的身份数据，就需要 OAuth 授权。

举例来说，A 网站允许 Gitlab 登录，背后就是下面的流程。

```
1. A 网站让用户跳转到 GitHub。
2. GitHub 要求用户登录，然后询问"A 网站要求获得 xx 权限，你是否同意？"
3. 用户同意，GitHub 就会重定向回 A 网站，同时发回一个授权码。
4. A 网站使用授权码，向 GitHub 请求令牌。
5. GitHub 返回令牌.
6. A 网站使用令牌，向 GitHub 请求用户数据。
```

- 应用登记

一个应用要求 OAuth 授权，必须先到对方网站登记，让对方知道是谁在请求。

https://smart.gitlab.biomind.com.cn/-/profile/applications

![](https://gitee.com/lpdswing/image/raw/master/img/202201241137304.png)

应用的名称随便填，跳转网址填写 `http://localhost:8081/oauth/redirect`。

提交表单以后，返回客户端 ID（client ID）和客户端密钥（client secret），这就是应用的身份识别码。

- 浏览器跳转登录gitlab

![](https://gitee.com/lpdswing/image/raw/master/img/202201241139057.png)

- 授权码

![](https://gitee.com/lpdswing/image/raw/master/img/202201241145339.png)

![](https://gitee.com/lpdswing/image/raw/master/img/202201241146507.png)

- 后端实现

https://smart.gitlab.biomind.com.cn/LIPENG/oauth-demo



### 参考文章

- Introduce OAuth 1.0

https://docs.authlib.org/en/latest/oauth/1/intro.html

- Introduce OAuth 2.0

https://docs.authlib.org/en/latest/oauth/2/intro.html

- Introduce OpenID Connect

https://docs.authlib.org/en/latest/oauth/oidc/index.html

- oauth2协议中文翻译

  https://colobu.com/2017/04/28/oauth2-rfc6749/

- https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html

## OpenID Connect简介

https://segmentfault.com/a/1190000023938486

# keycloak

docker为例:

https://www.keycloak.org/getting-started/getting-started-docker

默认用户密码 admin/admin

## 服务器安装

Reference: https://www.keycloak.org/docs/latest/server_installation/index.html#guide-overview

Keycloak 构建在 WildFly 应用服务器(原名叫Jboss), 是个javaee企业级的中间件软件架构.

## 重要目录

![](https://gitee.com/lpdswing/image/raw/master/img/202201121356506.png)

## 操作模式

独立运行模式仅在您想运行一个且只有一个 Keycloak 服务器实例时才有用。它不适用于集群部署，并且所有缓存都是非分布式且仅限本地的。不建议您在生产中使用独立模式，因为您将遇到单点故障。如果您的独立模式服务器出现故障，用户将无法登录。

### 独立集群模式启动

```bash
$ .../bin/standalone.sh --server-config=standalone-ha.xml
```

## 保护应用程序

### 概述

Keycloak 支持 OpenID Connect（OAuth 2.0 的扩展）和 SAML 2.0。在保护客户端和服务时，您需要决定的第一件事是您将使用两者中的哪一个。如果您愿意，您还可以选择使用 OpenID Connect 保护一些，使用 SAML 保护其他一些。

#### 适配器

- Python
  - [oidc](https://pypi.org/project/oic/) (generic)

#### 支持的协议

- OpenID连接

​	[OpenID Connect](https://openid.net/connect/) (OIDC) 是一种身份验证协议，是[OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749)的扩展。虽然 OAuth 2.0 只是一个构建授权协议的框架并且主要是不完整的，但 OIDC 是一个成熟的身份验证和授权协议。OIDC 还大量使用[Json Web Token](https://jwt.io/) (JWT) 标准集。这些标准定义了身份令牌 JSON 格式以及以紧凑和网络友好的方式对数据进行数字签名和加密的方法。

使用 OIDC 时实际上有两种类型的用例。第一个是要求 Keycloak 服务器为他们验证用户身份的应用程序。成功登录后，应用程序将收到一个*身份令牌*和一个*访问令牌*。该*令牌身份* 包括了有关用户，如用户名，电子邮件和其他个人资料信息。该*令牌访问*进行数字领域签名，并包括访问信息（如用户角色映射）的应用程序可以用来确定用户允许应用访问哪些资源。

第二种类型的用例是想要访问远程服务的客户端。在这种情况下，客户端要求 Keycloak 获取一个*访问令牌，*它可以用来代表用户调用其他远程服务。Keycloak 对用户进行身份验证，然后请求用户同意授予对请求它的客户端的访问权限。客户端然后接收*访问令牌*。此*访问令牌* 由领域进行数字签名。客户端可以使用此*访问令牌*对远程服务进行 REST 调用。REST 服务提取*访问令牌*，验证*令牌*的签名，然后根据令牌中的访问信息决定是否处理请求。

- SAML2.0

[SAML 2.0](http://saml.xml.org/saml-specifications)是与 OIDC 类似的规范，但更老、更成熟。它源于 SOAP 和过多的 WS-* 规范，因此它往往比 OIDC 更冗长一些。SAML 2.0 主要是一种身份验证协议，通过在身份验证服务器和应用程序之间交换 XML 文档来工作。XML 签名和加密用于验证请求和响应。

在 Keycloak 中，SAML 服务于两种类型的用例：浏览器应用程序和 REST 调用。

使用 SAML 时实际上有两种类型的用例。第一个是要求 Keycloak 服务器为他们验证用户身份的应用程序。成功登录后，应用程序将收到一个 XML 文档，其中包含称为 SAML 断言的内容，该断言指定有关用户的各种属性。此 XML 文档由领域进行数字签名，并包含访问信息（如用户角色映射），应用程序可以使用这些信息来确定允许用户访问应用程序上的哪些资源。

第二种类型的用例是想要访问远程服务的客户端。在这种情况下，客户端要求 Keycloak 获取 SAML 断言，它可以用来代表用户在其他远程服务上调用。

- OpenID Connect 与 SAML

在 OpenID Connect 和 SAML 之间进行选择不仅仅是使用较新的协议 (OIDC) 代替较旧的更成熟的协议 (SAML)。

**在大多数情况下，Keycloak 建议使用 OIDC。**

SAML 往往比 OIDC 更冗长。

除了交换数据的冗长之外，如果您比较规范，您会发现 OIDC 旨在与 Web 一起使用，而 SAML 被改造为在 Web 之上工作。例如，OIDC 也更适合 HTML5/JavaScript 应用程序，因为它比 SAML 更容易在客户端实现。由于令牌采用 JSON 格式，因此它们更容易被 JavaScript 使用。您还会发现一些不错的功能，它们可以更轻松地在 Web 应用程序中实现安全性。例如，查看规范用来轻松确定用户是否仍在登录的[iframe 技巧](https://openid.net/specs/openid-connect-session-1_0.html#ChangeNotification)。

SAML 有它的用途。当您看到 OIDC 规范不断发展时，您会看到它们实现了 SAML 多年来拥有的越来越多的功能。我们经常看到的是，人们之所以选择 SAML 而不是 OIDC，是因为认为它更成熟，而且他们已经拥有使用它保护的现有应用程序。

#### 端点

- http://127.0.0.1:8080/auth/realms/myrealm/.well-known/openid-configuration

![](https://gitee.com/lpdswing/image/raw/master/img/202201121609408.png)

- token

  ![](https://gitee.com/lpdswing/image/raw/master/img/202201121616091.png)

- userinfo

  ![](https://gitee.com/lpdswing/image/raw/master/img/202201121638776.png)
