# OAuth2.0-

>OAuth 2.0 是一种授权协议。

## 一、四种角色

- **资源所有者** `resource owner`：能够授予对受保护资源的访问权的实体。 当资源所有者是人时，它被称为 end-user。

- **资源服务器** `reosource server`：存放受保护资源的服务器，能够通过 access token 来请求和响应这些受保护的资源。
- **客户端** `client`：请求受保护资源的的一方就可以被看作一个客户端。（这个客户端只是一个概念，具体实现可以是服务器，应用程序，或者 Html 网页 等等，一个资源服务器在请求另一个资源服务器的受保护资源时，其也被视为一个客户端）。
- **授权服务器** `authorization server`：当客户端成功通过认证后，向其颁发 token 的服务器。

## 二、四种模式

授权许可的总结表格：

![2022-09-05_142147](https://img.qinweizhao.com/2022/09/2022-09-05_142147.png)

### 1、授权码

![2022-09-05_143733](https://img.qinweizhao.com/2022/09/2022-09-05_143733.png)

#### 授权请求

客户端向授权端点发起请求时，其 URI 中的 QueryString，必须添加以下参数：

| 参数          | 必传 | 描述                                                     |
| ------------- | ---- | -------------------------------------------------------- |
| response_type | 是   | 值必须是 "code"                                          |
| client_id     | 是   | 客户端标识                                               |
| redirect_uri  | 否   | 重定向地址，如果客户端在授权服务器中注册时已提供则可不传 |
| scope         | 否   | 请求访问的范围。                                         |
| state         | 否   | 推荐携带此值，用于防止跨站请求伪造                       |

请求示例：

```http
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
```

#### 授权响应

如果资源所有者许可访问请求，授权服务器颁发授权码，通过向重定向 URI 的查询部分添加下列参数传递授权码至客户端：

| 参数  | 必传 | 描述                                                         |
| ----- | ---- | ------------------------------------------------------------ |
| code  | 是   | 授权码必须在颁发后很快过期以减小泄露风险。推荐的最长的授权码生命周期是10分钟。客户端不能使用授权码超过一次。如果一个授权码被使用一次以上，授权服务器必须拒绝该请求并应该撤销（如可能）先前发出的基于该授权码的所有令牌。授权码与客户端标识和重定向 URI 绑定。 |
| state | 否   | 当授权请求携带此参数时则必传，值原封不动回传                 |

例如，授权服务器通过发送以下 HTTP 响应重定向用户代理：

```http
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA&state=xyz
```

客户端必须忽略无法识别的响应参数。 OAuth 未定义授权代码字符串的大小。 客户端应该避免对授权码的大小做出假设。 授权服务器应该记录它发出的任何值的大小。

#### 访问令牌请求

客户端发起向授权服务器的令牌端点发起一个 POST 请求，其 Content-type 必须为 “application/x-www-form-urlencoded”，并在其请求体中需要包含以下参数：

| 参数         | 必传 | 描述                                                         |
| ------------ | ---- | ------------------------------------------------------------ |
| grant_type   | 是   | 值必须是 “authorization_code”                                |
| code         | 是   | 值为上一步从授权服务器中收到的授权码                         |
| redirect_uri | 是   | 如果授权请求中携带了redirect_uri参数，则这里的值必须其相同   |
| client_id    | 是   | 如果客户端没有和授权服务器进行过 Client Credentials 的身份验证，则必须携带 |

如果客户端类型为机密或客户端颁发了客户端凭据(或分配了其他认证要求)，则客户端必须向授权服务器进行 Client Credentials 的身份验证。

例如：

```http
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
     &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb
```

授权服务器必须：

要求机密客户端或任何被颁发了客户端凭据（或有其他身份验证要求）的客户端进行客户端身份验证，若包括了客户端身份验证，验证客户端身份，确保授权码颁发给了通过身份验证的机密客户端，或者如果客户端是公开的，确保代码颁发给了请求中的“client_id”，验证授权码是有效的，并确保给出了 “redirect_uri” 参数，若 “redirect_uri” 参数包含在初始授权请求中，确保它们的值是相同的。

#### 访问令牌响应

成功响应示例：

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
    "access_token":"2YotnFZFEjr1zCsicMWpAA",
    "token_type":"example",
    "expires_in":3600,
    "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
    "example_parameter":"example_value"
}
```

### 2、简化模式

![2022-09-05_155703](https://img.qinweizhao.com/2022/09/2022-09-05_155703.png)

**不常用，主要针对那些无后台的系统，直接通过web跳转授权。**

#### 授权请求

客户端向授权端点发起请求时，其 URI 中的 QueryString，必须添加以下参数

| 参数          | 必传 | 描述                                                     |
| ------------- | ---- | -------------------------------------------------------- |
| response_type | 是   | 值必须是 "token"                                         |
| client_id     | 是   | 客户端标识                                               |
| redirect_uri  | 否   | 重定向地址，如果客户端在授权服务器中注册时已提供则可不传 |
| scope         | 否   | 请求访问的范围。                                         |
| state         | 否   | 推荐携带此值，用于防止跨站请求伪造                       |

请求示例：

```http
GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
```

#### 授权响应

如果资源所有者许可访问请求，授权服务器直接颁发访问令牌，通过向重定向 URI 的 Hash 部分添加下列参数传递 access token 至客户端：

| 参数         | 必传 | 描述                                                         |
| ------------ | ---- | ------------------------------------------------------------ |
| access_token | 是   | 授权服务器颁发的访问令牌。                                   |
| token_type   | 是   | 颁发的令牌的类型，其值是大小写不敏感的。（一般是 Bearer）    |
| expires_in   | 否   | 推荐的。以秒为单位的访问令牌生命周期。例如，值“3600”表示访问令牌将在从生成响应时的1小时后到期。如果省略，则授权服务器应该通过其他方式提供过期时间，或者记录默认值。 |
| scope        | 否   | 若与客户端请求的 scope 范围相同则可以不传，否则必需返回此值。 |
| state        | 否   | 当授权请求携带此参数时则必传，值原封不动回传                 |

示例：

```http
HTTP/1.1 302 Found
Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA&state=xyz&token_type=example&expires_in=3600
```

### 3、密码模式

![2022-09-05_155703](https://img.qinweizhao.com/2022/09/2022-09-05_155703.png)

**直接通过用户名、密码获取令牌。**

#### 访问令牌请求

客户端向授权服务器的令牌端点发起一个 POST 请求，其 Content-type 必须为 “application/x-www-form-urlencoded”，并在其请求体中需要包含以下参数：

| 参数       | 必传 | 描述                 |
| ---------- | ---- | -------------------- |
| grant_type | 是   | 值必须是 “password”  |
| username   | 是   | 资源所有者的用户名。 |
| password   | 是   | 资源所有者的密码。   |
| scope      | 是   | 请求访问的范围       |

如果客户端类型是机密的或客户端被颁发了客户端凭据，则客户端必须要与授权服务器进行身份验证（request header 中携带 Authorization，值为 Base64(clientId:clientSecret) ）。

例如：

```http
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded
grant_type=password&username=johndoe&password=A3ddj3w
```

授权服务器必须：

- 要求机密客户端或任何被颁发了客户端凭据（或有其他身份验证要求）的客户端进行客户端身份验证。
- 若包括了客户端身份验证，验证客户端身份
- 使用它现有的密码验证算法验证资源所有者的密码凭据。

由于这种访问令牌请求使用了资源所有者的密码，授权服务器必须保护端点防止暴力攻击（例如，使用速率限制或生成警报）。

#### 访问令牌响应

如果访问令牌请求有效且已授权，授权服务器将发出访问令牌和可选的刷新令牌。 如果请求客户端认证失败或无效，授权服务器将返回一个错误响应。 一个成功响应的示例如下:

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"2YotnFZFEjr1zCsicMWpAA",
  "token_type":"example",
  "expires_in":3600,
  "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
  "example_parameter":"example_value"
}
```

### 4、客户端模式

![2022-09-05_155703](https://img.qinweizhao.com/2022/09/2022-09-05_155703.png)

**这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。**

#### 访问令牌请求

客户端向授权服务器的令牌端点发起一个 POST 请求，其 Content-type 必须为 “application/x-www-form-urlencoded”，并在其请求体中需要包含以下参数：

| 参数       | 必传 | 描述                          |
| ---------- | ---- | ----------------------------- |
| grant_type | 是   | 值必须是 “client_credentials” |
| scope      | 是   | 请求访问的范围                |

请求示例：

```http
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
```

#### 访问令牌响应

如果访问令牌请求有效且已授权，授权服务器将发出访问令牌，但并不包含刷新令牌。 如果请求客户端认证失败或无效，授权服务器将返回一个错误响应。

成功响应示例:

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
    "access_token":"2YotnFZFEjr1zCsicMWpAA",
    "token_type":"example",
    "expires_in":3600,
    "example_parameter":"example_value"
}
```
