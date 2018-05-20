### 用户验证

rest\_framework允许TokenAuthentication与SessionAuthentication验证方式。

```
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    )
}
```

#### TokenAuthentication

此方案使用simple token-based HTTP Authentication方案。

token验证:一个用户在创建的时候就在后端创建一个token，用户请求时通过将token附带于request的hearder中，再数据库查询身份信息。登录失败原因则是不存在这个对应的token，或者这个token找到了但是对应的用户未激活。接着把token传给用户，存入浏览器 cookie，之后浏览器请求带上这个cookie，后端根据这个cookie值来查询用户，验证是否过期。

settings.py

```
INSTALLED_APPS = (
    ...
    'rest_framework.authtoken'
)
```

urls.py

```
from rest_framework.authtoken import views

path('api-token-auth/', views.obtain_auth_token),
```

向[http://127.0.0.1:8000/api-token-auth/发送post请求，body部分附带账号密码，服务器即会返回一个专属token，是由创建用户时产生。](http://127.0.0.1:8000/api-token-auth/发送post请求，body部分附带账号密码，服务器即会返回一个专属token，是由创建用户时产生。)

```
{
    "token": "7d926dda6c790f524b1e75054f0292708441ddd5"
}
```

再通过get方式，在headers加入

`Authorization:Token 7d926dda6c790f524b1e75054f0292708441ddd5`

请求[http://127.0.0.1:8000/novels/，后端程序就会解析token，在数据库中查询到用户信息。](http://127.0.0.1:8000/novels/，后端程序就会解析token，在数据库中查询到用户信息。)

#### SessionAuthentication

此方案，使用django的默认会话后端进行验证。会话验证适合于与网站在同一会话环境中的ajax客户端。

sessionauth在浏览器中比较常见，它会自动设置cookie，并将session等带到我们的服务器。

如果成功验证，SessionAuthentication会提供以下认证信息。

* request.user：django的User实例

* request.auth: None/Token

没有通过验证的会返回HTTP 403 Forbidden响应。

如果使用ajax类型的API，不安全的请求必须含有 CSRF token，例如PUT, PATCH, POST 或者 DELETE。

使用django的标准登录视图，将会确保登录函数是被保护的

REST中的CSRF验证与Django相比，有少量不同。Django中 需要支持session和non-session验证。 这意味着认证请求需要验证CSRF，而匿名用户不需要验证CSRF token.这种方法不适合登录函数，因为其都需要验证CSRF。

#### Json Web Token\(JWT\)

JWT 是一个开放标准\(RFC 7519\)，它定义了一种用于简洁，自包含的用于通信双方之间以 JSON 对象的形式安全传递信息的方法。JWT 可以使用 HMAC 算法或者是 RSA 的公钥密钥对进行签名。它具备两个特点：

* 简洁\(Compact\)

  可以通过URL, POST 参数或者在 HTTP header 发送，因为数据量小，传输速度快

* 自包含\(Self-contained\)

  负载中包含了所有用户所需要的信息，避免了多次查询数据库

##### JWT 组成

[![](https://ww4.sinaimg.cn/large/006tNc79gy1fbv54tfilmj31120b2wl9.jpg)](https://ww4.sinaimg.cn/large/006tNc79gy1fbv54tfilmj31120b2wl9.jpg)

* Header 头部

头部包含了两部分，token 类型和采用的加密算法

| { "alg": "HS256", "typ": "JWT" } |
| :--- |


它会使用 Base64 编码组成 JWT 结构的第一部分,如果你使用Node.js，可以用Node.js的包base64url来得到这个字符串。

> Base64是一种编码，也就是说，它是可以被翻译回原来的样子来的。它并不是一种加密过程。

* Payload 负载

这部分就是我们存放信息的地方了，你可以把用户 ID 等信息放在这里，JWT 规范里面对这部分有进行了比较详细的介绍，常用的由 iss（签发者），exp（过期时间），sub（面向的用户），aud（接收方），iat（签发时间）。

| { "iss": "lion1ou JWT", "iat": 1441593502, "exp": 1441594722, "aud": "www.example.com", "sub": "lion1ou@163.com" } |
| :--- |


同样的，它会使用 Base64 编码组成 JWT 结构的第二部分

* Signature 签名

前面两部分都是使用 Base64 进行编码的，即前端可以解开知道里面的信息。Signature 需要使用编码后的 header 和 payload 以及我们提供的一个密钥，然后使用 header 中指定的签名算法（HS256）进行签名。签名的作用是保证 JWT 没有被篡改过。

三个部分通过`.`连接在一起就是我们的 JWT 了，它可能长这个样子，长度貌似和你的加密算法和私钥有关系。

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`.`eyJpZCI6IjU3ZmVmMTY0ZTU0YWY2NGZmYzUzZGJkNSIsInhzcmYiOiI0ZWE1YzUwOGE2NTY2ZTc2MjQwNTQzZjhmZWIwNmZkNDU3Nzc3YmUzOTU0OWM0MDE2NDM2YWZkYTY1ZDIzMzBlIiwiaWF0IjoxNDc2NDI3OTMzfQ`.`PA3QjeyZSUh7H0GfE0vJaKW4LjKJuC3dVLQiY4hii8s`

其实到这一步可能就有人会想了，HTTP 请求总会带上 token，这样这个 token 传来传去占用不必要的带宽啊。如果你这么想了，那你可以去了解下 HTTP2，HTTP2 对头部进行了压缩，相信也解决了这个问题。

* 签名的目的

最后一步签名的过程，实际上是对头部以及负载内容进行签名，防止内容被窜改。如果有人对头部以及负载的内容解码之后进行修改，再进行编码，最后加上之前的签名组合形成新的JWT的话，那么服务器端会判断出新的头部和负载形成的签名和JWT附带上的签名是不一样的。如果要对新的头部和负载进行签名，在不知道服务器加密时用的密钥的话，得出来的签名也是不一样的。

* 信息暴露

在这里大家一定会问一个问题：Base64是一种编码，是可逆的，那么我的信息不就被暴露了吗？

是的。所以，在JWT中，不应该在负载里面加入任何敏感的数据。在上面的例子中，我们传输的是用户的User ID。这个值实际上不是什么敏感内容，一般情况下被知道也是安全的。但是像密码这样的内容就不能被放在JWT中了。如果将用户的密码放在了JWT中，那么怀有恶意的第三方通过Base64解码就能很快地知道你的密码了。

因此JWT适合用于向Web应用传递一些非敏感信息。JWT还经常用于设计用户认证和授权系统，甚至实现Web应用的单点登录。

##### JWT 使用

##### [![](https://ww3.sinaimg.cn/large/006tNc79gy1fbv63pzqocj30pj0h8t9m.jpg)](https://ww3.sinaimg.cn/large/006tNc79gy1fbv63pzqocj30pj0h8t9m.jpg)

1. 首先，前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP POST请求。建议的方式是通过SSL加密的传输（https协议），从而避免敏感信息被嗅探。
2. 后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload（负载），将其与头部分别进行Base64编码拼接后签名，形成一个JWT。形成的JWT就是一个形同lll.zzz.xxx的字符串。
3. 后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在localStorage或sessionStorage上，退出登录时前端删除保存的JWT即可。
4. 前端在每次请求时将JWT放入HTTP Header中的Authorization位。\(解决XSS和XSRF问题\)
5. 后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确；检查Token是否过期；检查Token的接收方是否是自己（可选）。
6. 验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。

##### JWT django实现

首先要安装

```
pip install djangorestframework-jwt
```

使用:

需要将jsonWebAuth加入到drf 的default auth class中

```
'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
```

对于用户post过来的token进行验证，将user取出来

* path中的配置

```
from rest_framework_jwt.views import obtain_jwt_token
#...

urlpatterns = [
    '',
    # ...

    url(r'^api-token-auth/', obtain_jwt_token),
]
```

配置path中的jwt

```
# jwt的token认证
path('jwt-auth/', obtain_jwt_token )
```

和TokenAuthentication验证方式基本一样，只是我们还需要设置Token有效时间。

```
# 与drf的jwt相关的设置
JWT_AUTH = {
    'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=3600),
    'JWT_AUTH_HEADER_PREFIX': 'Bearer',
}
```

JWT的主要作用在于（一）可附带用户信息，后端直接通过JWT获取相关信息。（二）使用本地保存，通过HTTP Header中的Authorization位提交验证。（三）易于设置Token有效时间

