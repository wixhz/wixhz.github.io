---
title: OAuth
date: 2023-09-02 09:52:56
tags:
---

简单来说，OAuth2.0是一种**授权协议**，它通过颁发访问令牌的机制而不是使用用户名和密码来请求交互，保证第三方应用只有在**成功获取授权之后**才可以访问授权者的数据。通过这套协议，用户仅仅通过授权操作就可以免去重复注册的流程，是目前web上重要的安全手段。

OAuth2.0提供了四种许可类型：

- **授权码许可（Authorization Code）**
- 隐式许可（Implicit）
- 客户端凭据许可（Client Credentials）
- 资源拥有者凭据许可（Resource Owner Password Credentials）

# OAuth2.0的基本流程

OAuth2.0里有4种角色，分别是资源所有者、三方软件、授权服务和受保护资源，授权码许可的流程如下。

![Oauth](OAuth/Oauth2.0.png)

## 重定向的必要性

如果没有重定向，只有纯粹后端逻辑的交互，那么用户在第三方平台跳转到授权服务后，用户与三方平台的连接关系就中断了，相当于用户一直停留在授权服务的页面。

## 授权码的必要性

用户在不同平台千万另一个平台时，由于域名的不同，需要使用重定向功能讲用户引导过去。与此同时由于域名的不同，cookie这些保存登陆态的信息无法携带跳转，即使用户在一个平台授权登录后其他平台仍然无法感知，因此需要携带一些信息告诉其他平台，用户已经授权过了。为了保证信息的安全性，携带的信息不能是用户密钥，于是使用了授权码机制（一种临时的、间接的凭证）。

第三方软件最终是需要拿到安全保密性要求极高的访问令牌才能操作用户数据，而两次重定向操作是暴露在浏览器上的，有了授权码的参与，保证了访问令牌只在后端服务之间传输。 使用授权码的目的之一，就是把浏览器和第三方软件的信息做一个隔离，确保浏览器看不到第三方软件最重要的访问令牌 access_token 的值

## 三方软件的流程

1. 注册信息。三方软件首先需要去授权平台申请app_id、app_serect、回调地址、申请权限等信息。
2. 引导授权。最终目的是为了获取access_token.
3. 使用访问令牌。拿到令牌就可以去操作用户授权的数据，官方规范列出了三种使用令牌的方式：

   - **Form-Encoded Body Parameter**（表单参数），安全性最高。

     ```java
     POST /resource HTTP/1.1
     Host: server.example.com
     Content-Type: application/x-www-form-urlencoded
     
     access_token=b1a64d5c-5e0c-4a70-9711-7af6568a61fb
     
     ```

   - URI Query Parameter（URI 查询参数），路径参数需要编码、并且容易泄漏。

     ```ja
     GET /resource?access_token=b1a64d5c-5e0c-4a70-9711-7af6568a61fb HTTP/1.1
     Host: server.example.com
     
     ```

   - Authorization Request Header Field（授权请求头部字段），需要代码额外处理Authorization头部信息。

     ```java
     GET /resource HTTP/1.1
     Host: server.example.com
     Authorization: Bearer b1a64d5c-5e0c-4a70-9711-7af6568a61fb
     
     ```
4. 使用刷新令牌。**刷新令牌是一次性的，使用之后就会失效**。
5. 三方软件请求不同的受保护资源服务，会由API GATEWAY 跳转到不同的受保护资源服务，省去每个资源服务都需要权限校验的操作。

   ![image-20230903104511589](OAuth/image-20230903104511589.png)

## 授权服务的流程

授权服务就是负责颁发令牌的服务，OAuth2.0的核心就是授权服务，而授权服务的核心是令牌。

### 获取授权码的步骤

1. 三方软件首先要去授权服务进行注册，然后授权服务会给三方软件一个app_id和app_secret等信息，方便以后授权时的身份校验，同时注册时还会告诉授权服务以后请求受保护资源的访问范围，也就是scope。
2. 三方软件以后请求授权时，授权服务会先去验证三方软件的基本信息，比如是否注册过、回调地址是否合法。因为这个过程由浏览器的web端重定向操作，每一步都有可被冒充的风险，因此需要对三方软件进行合法性校验。
3. 当验证通过并且用户授权后，授权服务会对用户授权范围和三方软件注册时指定的scope进行比对，防止三方软件申请scope范围外的权限（最小权限法则）。
4. 生成授权码code并将code和app_id、user、申请的scope关联，与此同时给code设置一个过期时间
5. 重定向到三方软件，并把code返回给软件。注意code只是换取access_token的临时凭证，无法代替access_token。

### 获取access_token的步骤

1. 验证三方软件是否存在，app_id和app_secret是否正确。
2. 验证code是否正确，**code应该使用一次就失效，防止被冒用**。
3. 生成access_token，将access_token与app_id、user关联，设置access_token过期时间并将之与scope关联。

### 刷新access_token

当access_token过期后，为了防止频繁需要用户授权，OAuth2.0引入了刷新令牌，规定用户在一定时间内无需手动授权就可以继续使用三方软件。

1. 授权服务在颁发access_token的时候会同步颁发一个refresh_token给三方软件，refresh_token的失效时间会比access_token更长。这样当access_token失效时，三方软件就会使用refresh_token请求授权服务。
2. 授权服务收到refresh_token后，会验证token类型、app_id、app_secret是否合法。
3. 重新生成新的access_token和refresh_token并发给三方服务。**一个刷新令牌被使用以后，授权服务需要将其废弃，并重新颁发一个刷新令牌**，不过refresh_token过期时间不会重新刷新。

### 使用JWT

授权服务的核心就是访问令牌，访问令牌没有具体的格式，目前使用最多的就是JWT令牌了。

> JSON Web Token（JWT）是一个开放标准（RFC 7519），它定义了一种紧凑的、自包含的方式，用于作为 JSON 对象在各方之间安全地传输信息。

JWT 结构化体可以分为 HEADER（头部）、PAYLOAD（数据体）和 SIGNATURE（签名）三部分。经过签名之后的 JWT 的整体结构，是被句点符号分割的三段内容，结构为 header.payload.signature 。

- header：表示装载令牌类型和算法等信息。其中，typ 表示第二部分 PAYLOAD 是 JWT 类型，alg 表示使用 HS256 对称签名的算法。
- payload：表示JWT的数据体，可以是一切合法json格式的数据。
- signature：表示对 JWT 信息的签名。

![image-20230902214920832](OAuth/image-20230902214920832.png)

优点：

1. 用计算代替存储。通过对JWT进行解密就可以拿到json格式的数据，其中可以存放用户授权的范围，省去了调用rpc服务的时间。
2. 签名保障了传输过程的安全性。

注意：由于服务器不存储JWT，当用户取消授权时需要同步取消JWT的有效性。这时可以把加密算法缩小到用户级别，保证每个用户的加密密钥都不一样，这样当用户取消授权时只要同步修改加密密钥，即可使JWT失效。

# 可能存在的安全漏洞

OAuth2.0作为安全协议，如果使用不当或不遵守规范，很容易产生一些安全漏洞。

## CSRF攻击

恶意软件让浏览器向已完成用户身份认证的网站发起请求，并执行有害的操作，就是跨站请求伪造攻击，攻击过程的本质上就是，软件 A（攻击者）用自己的授权码 codeA 的值，通过 CSRF 攻击，“替换”了软件 B 的授权码的值。

![image-20230903175033067](OAuth/image-20230903175033067.png)

- 受害者登录a.com，并保留了登录凭证（Cookie）。
- 攻击者引诱受害者访问了b.com。
- b.com 向 a.com 发送了一个请求：a.com/act=xx。浏览器会默认携带a.com的Cookie。
- a.com接收到请求后，对请求进行验证，并确认是受害者的凭证，误以为是受害者自己发送的请求。
- a.com以受害者的名义执行了act=xx。
- 攻击完成，攻击者在受害者不知情的情况下，冒充受害者，让a.com执行了自己定义的操作。



解决办法：OAuth 2.0 中建议使用一个随机值 state 参数，当某三方服务请求授权码的时候附带一个自己生成 state 参数值，同时授权服务也要按照规则将这个随机的 state 值跟授权码 code 一起返回给三方服务，当三方服务接收授权码的时候，就要在服务端做一个 state 参数值的比对校验，如果相同就继续流程，否则直接拒绝后续流程。这样就可以避免获取到一个假的 code 进行后续流程。

## XSS攻击

XSS 攻击的主要手段是将恶意脚本注入到请求的输入中，攻击者可以通过注入的恶意脚本来进行攻击行为，比如搜集数据等。



解决办法：对非法信息做转义过滤，如包含`<script>`、`<img>`、`<a>`等标签

## 水平越权

水平越权指的是请求到达受保护资源服务时，服务端只做了token校验，没有检查这个请求获取的数据是否归属于当前授权的用户，导致不法者用自己获得的授权来访问受保护资源服务的时候，就有可能获取其他用户的数据。



解决办法：不但检查access_token的合法性，更要检查请求的范围、数据的归属。

## 授权码失窃

![image-20230903220637576](OAuth/image-20230903220637576.png)

问题关键点：

- 授权服务校验code的时候没有对code的归属app做校验。
- code使用以后没有及时实效。
- 回调地址没有做校验。
