# 入门

docker为例:

https://www.keycloak.org/getting-started/getting-started-docker

默认用户密码 admin/admin

# 服务器安装

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
