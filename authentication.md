[TOC]
## session认证
- server获取clien中的cookie的特定字段(session_id等)进行验证。
- 用户增多会导致开销增大

## jwt
### 流程
- 用户使用用户名密码来请求服务器
- 服务器进行验证用户的信息
- 服务器通过验证发送给用户一个token
- 客户端存储token，并在每次请求时附送上这个token值
- 服务端验证token值，并返回数据

### 构成
- 由`.`分割的三部分：`$header.$payload.$signature`
- `header`头部包含两部分,用`base64`加密。包括`声明类型`,`声明加密算法` 通常使用`HS256`
  + `RS256` (采用SHA-256 的 RSA 签名) 是一种非对称算法, 它使用公共/私钥对: 标识提供方采用私钥生成签名, JWT 的使用方获取公钥以验证签名。由于公钥 (与私钥相比) 不需要保护, 因此大多数标识提供方使其易于使用方获取和使用 (通常通过一个元数据URL)。
  + `HS256` (带有 SHA-256 的 HMAC 是一种对称算法, 双方之间仅共享一个 密钥。由于使用相同的密钥生成签名和验证签名, 因此必须注意确保密钥不被泄密。
```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

- `payload`载荷，用于存放有效信息。包括`标准中注册的声明`,`公共的声明`,`私有的声明`例如下
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
- `payload`.`标准中注册的声明`：
    + `iss`: jwt签发者
    + `sub`: jwt所面向的用户
    + `aud`: 接收jwt的一方
    + `exp`: jwt的过期时间，这个过期时间必须要大于签发时间
    + `nbf`: 定义在什么时间之前，该jwt都是不可用的.
    + `iat`: jwt的签发时间
    + `jti`: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
- `payload`.`公共的声明`：公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.
- `payload`.`私有的声明`：私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

- `signature`签证
```javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);
var signature = HMACSHA256(encodedString, 'secret'); 
```

- `secret`保存在服务端里，生产jwt也在服务端，secret就是用来进行jwt的签发和jwt验证，所以它就是你服务端的私钥，不可泄漏。

- 应用/使用
```javascript
fetch('api/user/1', {
  headers: {
    'Authorization': 'Bearer ' + token
  }
})
```

## OATUH2.0
### 名词定义
- `HTTP service`HTTP服务提供商，如谷歌(具有用户的一些资源)
- `Resource Owner`资源所有者，指用户
- `User Agent`用户代理，如浏览器
- `Third-party application`第三方应用程序，也叫`Client`
- `Authorization server`认证服务器，`HTTP service`专门用于处理认证的服务器。
- `Resource server`资源服务器，`HTTP service`存放用户资源的服务器。

### oatu2流程
- 用户打开客户端以后，`Resource Owner`要求`Client`给予授权。
- `Resource Owner`同意并授权给`Client`。
- `Client`使用上一步获得的授权，向`Authorization server`申请`access token`。
- `Authorization server`对`Client`进行认证以后，同意发放`access token`。
- `Client`使用`access token`，向`Resource server`申请获取资源。
- `Resource server`确认`access token`无误，同意向`Client`开放资源。

### `Client`授权模式
- `authorization code`授权码模式
- `implicit`简化模式
- `resource owner password credentials`密码模式
- `client credentials`客户端模式

#### `authorization code`授权码模式
- `User Agent`用`client`与`redirect_url`请求`Authorization Server`
- `Authorization Server`返回页面，用户同意授权
- `Authorization Server`返回授权码
