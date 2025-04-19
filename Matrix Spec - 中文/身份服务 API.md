# 身份服务 API

Matrix 客户端-服务器和服务器-服务器 API 主要通过 Matrix 用户标识符表达。偶尔，需要通过其他（“第三方”）标识符或“3PID”来引用用户，例如他们的电子邮件地址或电话号码。本身份服务规范描述了如何建立、验证和使用第三方标识符与 Matrix 用户标识符之间的映射。技术上，这种描述可以适用于任何 3PID，但实际上仅具体应用于电子邮件地址和电话号码。

## 一般原则[](https://spec.matrix.org/unstable/identity-service-api/#general-principles)

身份服务器的目的是验证、存储和回答有关用户身份的问题。特别是，它存储形式为“标识符 X 代表与标识符 Y 相同的用户”的关联，其中身份可能存在于不同的系统上（如电子邮件地址、电话号码、Matrix 用户 ID 等）。

身份服务器拥有一些私钥-公钥对。当被询问关于某个关联时，它将使用其私钥对关联的详细信息进行签名。客户端可以通过验证身份服务器的公钥签名来验证关于关联的声明。

通常，身份服务器被视为可靠的预言机。它们不一定提供已验证关联的证据，但声称已这样做。建立单个身份服务器的可信度留给客户端自行处理。

3PID 类型在 [3PID Types](https://spec.matrix.org/unstable/appendices#3pid-types) 附录中描述。

## API 标准[](https://spec.matrix.org/unstable/identity-service-api/#api-standards)

Matrix 中身份服务器通信的强制基线是通过 HTTP API 交换 JSON 对象。通信需要使用 HTTPS。

所有 `POST` 和 `PUT` 端点，除历史原因的 [`POST /_matrix/identity/v2/account/logout`](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2accountlogout) 外，要求客户端提供包含（可能为空的）JSON 对象的请求体。客户端应为所有带有 JSON 主体的请求提供 `Content-Type` 头为 `application/json`，但这不是强制的。

同样，所有端点要求服务器返回 JSON 对象。服务器必须为所有 JSON 响应包含 `Content-Type` 头为 `application/json`。

所有 JSON 数据，无论是请求还是响应，必须使用 UTF-8 编码。

### 标准错误响应[](https://spec.matrix.org/unstable/identity-service-api/#standard-error-response)

在 Matrix API 级别发生的任何错误必须返回“标准错误响应”。这是一个看起来像这样的 JSON 对象：

```json
{
  "errcode": "<error code>",
  "error": "<error message>"
}
```

`error` 字符串将是一个人类可读的错误消息，通常是解释出错原因的句子。`errcode` 字符串将是一个唯一的字符串，可用于处理错误消息，例如 `M_FORBIDDEN`。根据错误可能会有其他键，但键 `error` 和 `errcode` 必须始终存在。

一些标准错误代码如下：

`M_NOT_FOUND` 请求的资源无法找到。

`M_MISSING_PARAMS` 请求缺少一个或多个参数。

`M_INVALID_PARAM` 请求包含一个或多个无效参数。

`M_SESSION_NOT_VALIDATED` 会话尚未验证。

`M_NO_VALID_SESSION` 无法为给定参数找到会话。

`M_SESSION_EXPIRED` 会话已过期，必须续订。

`M_INVALID_EMAIL` 提供的电子邮件地址无效。

`M_EMAIL_SEND_ERROR` 发送电子邮件时出错。通常在尝试验证给定电子邮件地址的所有权时看到。

`M_INVALID_ADDRESS` 提供的第三方地址无效。

`M_SEND_ERROR` 发送通知时出错。通常在尝试验证给定第三方地址的所有权时看到。

`M_UNRECOGNIZED` 请求包含无法识别的值，例如未知的令牌或媒介。

如果服务器无法理解请求，也会使用此响应。如果端点未实现，预计将返回 404 HTTP 状态码；如果端点已实现，但使用了错误的 HTTP 方法，则返回 405 HTTP 状态码。

`M_THREEPID_IN_USE` 第三方标识符已被其他用户使用。通常此错误将有一个额外的 `mxid` 属性以指示谁拥有该第三方标识符。

`M_UNKNOWN` 发生了未知错误。

## 隐私[](https://spec.matrix.org/unstable/identity-service-api/#privacy)

身份是一个隐私敏感的问题。虽然身份服务器存在是为了提供身份信息，但应限制访问以避免泄露可能敏感的数据。特别是，应避免能够构建大规模身份连接。为此，通常 API 应允许将 3PID 映射到 Matrix 用户身份，但不能反向映射（即不应能够获取与 Matrix 用户 ID 关联的所有 3PID，或获取与 3PID 关联的所有 3PID）。

## Web 浏览器客户端[](https://spec.matrix.org/unstable/identity-service-api/#web-browser-clients)

可以预期，一些客户端将被编写为在 Web 浏览器或类似环境中运行。在这些情况下，身份服务器应响应预检请求，并在所有请求中提供跨域资源共享（CORS）头。

当客户端使用预检（OPTIONS）请求接近服务器时，服务器应为该路由响应 CORS 头。服务器在所有请求中返回的推荐 CORS 头为：

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, Authorization
```

## API 版本检查[](https://spec.matrix.org/unstable/identity-service-api/#api-version-check)

### GET /_matrix/identity/versions[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityversions)

---

**在 `v1.1` 中添加**

获取服务器支持的规范版本。

值将采用 `vX.Y` 或历史情况下的 `rX.Y.Z` 形式。有关更多信息，请参阅[规范版本控制](https://spec.matrix.org/unstable/#specification-versions)。

服务器报告所有支持的版本，包括补丁版本。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

#### 请求

无请求参数或请求体。

---

#### 响应

|状态|描述|
|---|---|
|`200`|服务器支持的版本。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`versions`|`[string]`|**必需：** 支持的版本。|

```json
{
  "versions": [
    "r0.2.0",
    "r0.2.1",
    "v1.1"
  ]
}
```

## 认证[](https://spec.matrix.org/unstable/identity-service-api/#authentication)

身份服务 API 中的大多数端点需要认证，以确保请求用户已接受所有相关政策，并且被允许进行请求。

身份服务器使用类似于客户端-服务器 API 的访问令牌概念的方案来认证用户。身份服务器提供的访问令牌不能用于认证客户端-服务器 API 请求。

访问令牌可以通过请求头提供，使用认证 Bearer 方案：`Authorization: Bearer TheTokenHere`。

客户端也可以通过查询字符串参数提供访问令牌：`access_token=TheTokenHere`。此方法已弃用，以防止访问令牌在访问/HTTP 日志中泄露，客户端不应使用。

身份服务器必须支持这两种方法。

> [!info] 信息
> **[在 `v1.11` 中更改]** 通过查询字符串参数发送访问令牌现已弃用。

当凭据需要但缺失或无效时，HTTP 调用将返回状态 401 和错误代码 `M_UNAUTHORIZED`。

### GET /_matrix/identity/v2/account[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2account)

---

获取有关使用请求中访问令牌的用户的信息。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

无请求参数或请求体。

---

#### 响应

|状态|描述|
|---|---|
|`200`|令牌持有者的信息。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是 `M_TERMS_NOT_SIGNED` 错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`user_id`|`string`|**必需：** 注册令牌的用户 ID。|

```json
{
  "user_id": "@alice:example.org"
}
```

##### 403 响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

### POST /_matrix/identity/v2/account/logout[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2accountlogout)

---

注销访问令牌，防止其用于认证对服务器的未来请求。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

无请求参数或请求体。

---

#### 响应

|状态|描述|
|---|---|
|`200`|令牌已成功注销。|
|`401`|令牌未注册或服务器未知。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是 `M_TERMS_NOT_SIGNED` 错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|

##### 200 响应

```json
{}
```

##### 401 响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "M_UNKNOWN_TOKEN",
  "error": "Unrecognised access token"
}
```

##### 403 响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

### POST /_matrix/identity/v2/account/register[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2accountregister)

---

将来自家庭服务器的 OpenID 令牌交换为访问身份服务器的访问令牌。请求体与客户端-服务器 API 中 `/openid/request_token` 返回的值相同。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

#### 请求

##### 请求体
|OpenIdCredentials|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`access_token`|`string`|**必需：** 消费者可以用来验证生成令牌的人的身份的访问令牌。这被提供给联邦 API `GET /openid/userinfo` 以验证用户的身份。|
|`expires_in`|`integer`|**必需：** 此令牌过期前的秒数，届时必须生成新的令牌。|
|`matrix_server_name`|`string`|**必需：** 消费者在尝试验证用户身份时应使用的家庭服务器域。|
|`token_type`|`string`|**必需：** 字符串 `Bearer`。|

##### 请求体示例

```json
{}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|一个可以用于认证对身份服务器的未来请求的令牌。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`token`|`string`|**必需：** 一个不透明的字符串，表示用于认证对身份服务器的未来请求的令牌。|

```json
{
  "token": "abc123_OpaqueString"
}
```

## 服务条款[](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)

鼓励身份服务器拥有服务条款（或类似政策），以确保用户已同意其数据被服务器处理。为此，身份服务器可以对几乎任何经过认证的 API 端点响应 HTTP 403 和错误代码 `M_TERMS_NOT_SIGNED`。错误代码用于指示用户必须接受新的服务条款才能继续。

所有支持认证的端点都可以返回 `M_TERMS_NOT_SIGNED` 错误。当客户端收到错误时，预计会调用 `GET /terms` 以了解服务器提供的条款。客户端将其与用户的 `m.accepted_terms` 账户数据（稍后描述）进行比较，并向用户提供接受仍缺失的服务条款的选项。在用户做出选择后（如果适用），客户端发送请求到 `POST /terms` 以指示用户的接受。服务器不能期望客户端会发送所有待处理条款的接受，客户端也不应期望服务器在其下一个请求中不会再次响应 `M_TERMS_NOT_SIGNED`。用户刚刚接受的条款将附加到 `m.accepted_terms`。

### `m.accepted_terms`[](https://spec.matrix.org/unstable/identity-service-api/#maccepted_terms)

---

用户先前已接受的条款 URL 列表。客户端应使用此列表以避免向用户展示他们已同意的条款。

|   |   |
|---|---|
|事件类型：|消息事件|

#### 内容

|名称|类型|描述|
|---|---|---|
|`accepted`|`[string]`|用户先前已接受的 URL 列表。当用户同意新条款时应附加到此列表中。|

#### 示例

```json
{
  "content": {
    "accepted": [
      "https://example.org/somewhere/terms-1.2-en.html",
      "https://example.org/somewhere/privacy-1.2-en.html"
    ]
  },
  "type": "m.accepted_terms"
}
```

### GET /_matrix/identity/v2/terms[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2terms)

---

获取服务器提供的所有服务条款。客户端预计会筛选条款以确定哪些条款需要用户接受。注意，此端点不需要认证。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

#### 请求

无请求参数或请求体。

---

#### 响应

|状态|描述|
|---|---|
|`200`|服务器提供的服务条款。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`policies`|{string: [Policy Object](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2terms_response-200_policy-object)}|**必需：** 服务器提供的政策。从任意 ID（在此版本的规范中未使用）映射到政策对象。|

|Policy Object|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`version`|`string`|**必需：** 政策的版本。对此没有要求，可能是“alpha”、语义版本或任意。|
|<其他属性>|[国际化政策](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2terms_response-200_internationalised-policy)|指定语言的政策信息。|

|国际化政策|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`name`|`string`|**必需：** 政策的翻译名称。|
|`url`|`string`|**必需：** URL，应该包含政策 ID、版本和语言，以便呈现给用户作为政策。URL 应包含所有三个标准，以避免在未来更新政策时发生冲突：例如，如果这是“[https://example.org/terms.html"](https://example.org/terms.html%22)，那么服务器将无法更新它，因为客户端已经将该 URL 添加到 `m.accepted_terms` 集合中。|

```json
{
  "policies": {
    "privacy_policy": {
      "en": {
        "name": "Privacy Policy",
        "url": "https://example.org/somewhere/privacy-1.2-en.html"
      },
      "fr": {
        "name": "Politique de confidentialité",
        "url": "https://example.org/somewhere/privacy-1.2-fr.html"
      },
      "version": "1.2"
    },
    "terms_of_service": {
      "en": {
        "name": "Terms of Service",
        "url": "https://example.org/somewhere/terms-2.0-en.html"
      },
      "fr": {
        "name": "Conditions d'utilisation",
        "url": "https://example.org/somewhere/terms-2.0-fr.html"
      },
      "version": "2.0"
    }
  }
}
```

### POST /_matrix/identity/v2/terms[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2terms)

---

由客户端调用以指示用户已接受/同意包含的 URL 集。服务器不得假设客户端将发送所有先前接受的 URL，因此应将提供的 URL 附加到服务器已知的已接受 URL 中。

客户端必须提供以用户呈现的语言的政策 URL。服务器应将接受任何一种语言的 URL 视为接受该政策的所有其他语言。

服务器应避免返回 `M_TERMS_NOT_SIGNED`，因为客户端可能不会一次接受所有条款。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求体

|名称|类型|描述|
|---|---|---|
|`user_accepts`|`[string]`|**必需：** 用户在此请求中接受的 URL。|

##### 请求体示例

```json
{
  "user_accepts": [
    "https://example.org/somewhere/terms-2.0-en.html"
  ]
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|服务器已视用户为接受提供的 URL。|

##### 200 响应

```json
{}
```

## 状态检查[](https://spec.matrix.org/unstable/identity-service-api/#status-check)

### GET /_matrix/identity/v2[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2)

---

检查身份服务器是否在此 API 端点可用。

要发现身份服务器在特定 URL 上可用，可以查询此端点并返回一个空对象。

这主要用于自动发现和健康检查目的，由作为身份服务器客户端的实体使用。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

#### 请求

无请求参数或请求体。

---

#### 响应

|状态|描述|
|---|---|
|`200`|身份服务器已准备好处理请求。|

##### 200 响应

```json
{}
```

## 密钥管理[](https://spec.matrix.org/unstable/identity-service-api/#key-management)

身份服务器拥有一些长期的私钥-公钥对。这些以 `algorithm:identifier` 的方案命名，例如 `ed25519:0`。在签署关联时，标准的[签署 JSON](https://spec.matrix.org/unstable/appendices#signing-json)算法适用。

身份服务器还可以跟踪一些短期的私钥-公钥对，这些可能具有与服务的长期密钥不同的使用和生命周期特征。

### GET /_matrix/identity/v2/pubkey/ephemeral/isvalid[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2pubkeyephemeralisvalid)

---

检查短期公钥是否有效。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

#### 请求

##### 请求参数
|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`public_key`|`string`|**必需：** 要检查的未填充的 base64 编码公钥。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|公钥的有效性。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`valid`|`boolean`|**必需：** 公钥是否被识别并且当前有效。|

```json
{
  "valid": true
}
```

### GET /_matrix/identity/v2/pubkey/isvalid[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2pubkeyisvalid)

---

检查长期公钥是否有效。只要密钥存在，响应应始终相同。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

#### 请求

##### 请求参数
|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`public_key`|`string`|**必需：** 要检查的未填充的 base64 编码公钥。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|公钥的有效性。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`valid`|`boolean`|**必需：** 公钥是否被识别并且当前有效。|

```json
{
  "valid": true
}
```

### GET /_matrix/identity/v2/pubkey/{keyId}[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2pubkeykeyid)

---

获取传递的密钥 ID 的公钥。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

#### 请求

##### 请求参数
|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`keyId`|`string`|**必需：** 密钥的 ID。这应采用算法:标识符的形式，其中算法标识签名算法，标识符是一个不透明的字符串。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|公钥存在。|
|`404`|未找到公钥。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`public_key`|`string`|**必需：** 未填充的 Base64 编码公钥。|

```json
{
  "public_key": "VXuGitF39UH5iRfvbIknlvlAVKgD1BsLDMvBf0pmp7c"
}
```

##### 404 响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "The public key was not found"
}
```

## 关联查找[](https://spec.matrix.org/unstable/identity-service-api/#association-lookup)

#### GET /_matrix/identity/v2/hash_details[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2hash_details)

---

从服务器获取用于哈希标识符的参数。这可以包括本规范中定义的任何算法。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

##### 请求

无请求参数或请求体。

---

##### 响应

|状态|描述|
|---|---|
|`200`|哈希函数信息。|

###### 200 响应

|名称|类型|描述|
|---|---|---|
|`algorithms`|`[string]`|**必需：** 服务器支持的算法。必须至少包含 `sha256`。|
|`lookup_pepper`|`string`|**必需：**<br><br>客户端在哈希标识符时必须使用的 pepper，并且在执行查找时必须提供给 `/lookup` 端点。<br><br>服务器应经常旋转此字符串。|

```json
{
  "algorithms": [
    "none",
    "sha256"
  ],
  "lookup_pepper": "matrixrocks"
}
```

#### POST /_matrix/identity/v2/lookup[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2lookup)

---

查找已绑定给定 3PID 的 Matrix 用户 ID 集（如果绑定可用）。请注意，地址的格式在本规范后面定义。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

##### 请求

###### 请求体

|名称|类型|描述|
|---|---|---|
|`addresses`|`[string]`|**必需：**<br><br>要查找的地址。此处条目的格式取决于使用的 `algorithm`。请注意，格式错误或格式不正确的查询将导致无匹配。<br><br>请注意，地址是区分大小写的：查看 [3PID Types](https://spec.matrix.org/unstable/appendices#3pid-types) 以验证标识符在提交/哈希之前的预期大小写。|
|`algorithm`|`string`|**必需：** 客户端用于编码 `addresses` 的算法。这应该是 `/hash_details` 中的可用选项之一。|
|`pepper`|`string`|**必需：** 来自 `/hash_details` 的 pepper。即使 `algorithm` 不使用它，也需要。|

###### 请求体示例

```json
{
  "addresses": [
    "4kenr7N9drpCJ4AfalmlGQVsOn3o2RHjkADUpXJWZUc",
    "nlo35_T5fzSGZzJApqu8lgIudJvmOQtDaHtr-I4rU7I"
  ],
  "algorithm": "sha256",
  "pepper": "matrixrocks"
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|任何匹配 `addresses` 的关联。|
|`400`|客户端的请求在某种程度上无效。一个可能的问题可能是 `pepper` 在服务器旋转后无效 - 这会以 `M_INVALID_PEPPER` 错误代码呈现。客户端应在这种情况下调用 `/hash_details` 以获取新的 pepper，注意避免重试循环。`M_INVALID_PARAM` 也可以返回以指示客户端提供了服务器未知的 `algorithm`。|

###### 200 响应

|名称|类型|描述|
|---|---|---|
|`mappings`|`{string: string}`|**必需：** `addresses` 到 Matrix 用户 ID 的任何适用映射。没有关联的地址将不包括在内，这可以使此属性为空对象。|

```json
{
  "mappings": {
    "4kenr7N9drpCJ4AfalmlGQVsOn3o2RHjkADUpXJWZUc": "@alice:example.org"
  }
}
```

###### 400 响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "M_INVALID_PEPPER",
  "error": "Unknown or invalid pepper - has it been rotated?"
}
```

### 客户端行为[](https://spec.matrix.org/unstable/identity-service-api/#client-behaviour)

在执行查找之前，客户端应请求 `/hash_details` 端点以确定服务器支持的算法（下面将更详细描述）。然后，客户端使用此信息形成 `/lookup` 请求并从服务器接收已知绑定。

客户端必须至少支持 `sha256` 算法。

### 服务器行为[](https://spec.matrix.org/unstable/identity-service-api/#server-behaviour)

服务器在收到 `/lookup` 请求后，将查询与其已知的绑定进行比较，必要时对其已知的标识符进行哈希处理以验证与请求的精确匹配。

服务器必须至少支持 `sha256` 算法。

### 算法[](https://spec.matrix.org/unstable/identity-service-api/#algorithms)

某些算法是规范的一部分，但客户端和服务器可以使用 `/hash_details` 协商其他格式。

#### `sha256`[](https://spec.matrix.org/unstable/identity-service-api/#sha256)

此算法是客户端和服务器至少必须支持的。它也是查找的首选算法。

使用此算法时，客户端首先将查询转换为以空格分隔的字符串，格式为 `<address> <medium> <pepper>`。`<pepper>` 从 `/hash_details` 获取，`<medium>` 通常是 `email` 或 `msisdn`（均为小写），`<address>` 是要搜索的 3PID。例如，如果客户端想知道 `alice@example.org` 的绑定，它将首先将查询格式化为 `alice@example.org email ThePepperGoesHere`。

> [!info] 理由：
> 媒介和 pepper 被附加到地址以防止每个 3PID 的公共前缀，帮助防止攻击者预先计算哈希函数的内部状态。

在格式化每个查询后，字符串通过 [RFC 4634](https://tools.ietf.org/html/rfc4634) 定义的 SHA-256 运行。然后将结果字节使用 URL 安全的[未填充 Base64](https://spec.matrix.org/unstable/appendices#unpadded-base64) 编码（类似于[房间版本 4 的事件 ID 格式](https://spec.matrix.org/unstable/rooms/v4#event-ids)）。

使用 pepper `matrixrocks` 的查询示例集为：

```
"alice@example.com email matrixrocks" -> "4kenr7N9drpCJ4AfalmlGQVsOn3o2RHjkADUpXJWZUc"
"bob@example.com email matrixrocks"   -> "LJwSazmv46n0hlMlsb_iYxI0_HXEqy_yj6Jm636cdT8"
"18005552067 msisdn matrixrocks"      -> "nlo35_T5fzSGZzJApqu8lgIudJvmOQtDaHtr-I4rU7I"
```

然后将哈希集作为 `/lookup` 中的 `addresses` 数组提供。请注意，使用的 pepper 必须在 `/lookup` 请求中作为 `pepper` 提供。

#### `none`[](https://spec.matrix.org/unstable/identity-service-api/#none)

此算法在身份服务器上执行明文查找。通常不应使用此算法，因为未哈希标识符的安全问题，但某些场景（如基于 LDAP 的身份服务器）阻止使用哈希标识符。身份服务器（以及可选的客户端）可以使用此算法执行这些类型的查找。

与 `sha256` 算法类似，客户端将查询转换为以空格分隔的字符串，格式为 `<address> <medium>` - 注意缺少 `<pepper>`。例如，如果客户端想知道 `alice@example.org` 的绑定，它将查询格式化为 `alice@example.org email`。

然后将格式化的字符串作为 `/lookup` 中的 `addresses` 提供。请注意，仍然需要提供 `pepper`，并且必须提供以确保客户端已首先对 `/hash_details` 进行了适当的请求。

-----------------------

### 安全注意事项[](https://spec.matrix.org/unstable/identity-service-api/#security-considerations)

> [!info] 信息:
> [MSC2134](https://github.com/matrix-org/matrix-spec-proposals/blob/main/proposals/2134-identity-hash-lookup.md) 提供了关于本规范部分安全注意事项的更多信息。本节涵盖了规范为何如此的高层次细节。

通常，当客户端有一个未知的3PID（第三方标识符）并希望找到对应的Matrix用户ID时，会使用查找端点。客户端通常在邀请新用户加入房间或在用户的通讯录中搜索尚未发现的Matrix用户时进行这种查找。如果恶意的身份服务器以明文形式接收到这些未知信息，可能会收集这些信息并进行恶意操作。为了保护可能没有Matrix标识符绑定到其3PID地址的用户隐私，规范试图使得收集3PID变得困难。

> [!info] 理由:
> 虽然哈希标识符并不完美，但它能显著增加收集标识符所需的努力。特别是电话号码，即使使用哈希也很难保护，但哈希显然比不使用哈希要好。
> 
> 哈希的替代方法是使用bcrypt或类似方法进行多轮加密，但由于需要服务于移动客户端和有限硬件的客户端，解决方案需要保持相对轻量。

客户端应谨慎对待不经常轮换其pepper的服务器，以及可能使用弱pepper的服务器——这些服务器可能试图暴力破解标识符或使用彩虹表来挖掘地址。同样，支持`none`算法的客户端应至少警告用户将标识符以明文形式发送到身份服务器的风险。

地址仍然可能通过计算的彩虹表反转，给定一些标识符，例如电话号码、常见的电子邮件地址域和泄露的地址都很容易计算。例如，电话号码大约有12位数字，使其比电子邮件地址更容易成为攻击目标。

## 建立关联[](https://spec.matrix.org/unstable/identity-service-api/#establishing-associations)

创建关联的流程是基于会话的。

在一个会话中，可以证明拥有一个3PID。一旦建立了这种关系，用户可以在该3PID和一个Matrix用户ID之间形成关联。注意，这种关联仅被证明是单向的；用户可以将任何Matrix用户ID与经过验证的3PID关联，即我可以声称任何我拥有的电子邮件地址与@billg:microsoft.com关联。

会话是有时间限制的；会话被认为在创建时被修改，然后在其中进行验证时被修改。会话只能在其最近一次修改后的24小时内检查验证，并且只能在会话内进行验证。任何在过期后尝试执行这些操作的行为都会被拒绝，应该创建并使用新的会话。

要启动会话，客户端向适当的`/requestToken`端点发出请求。身份服务器然后向用户发送验证令牌，用户将令牌提供给客户端。客户端然后将令牌提供给适当的`/submitToken`端点，完成会话。在此时，客户端应`/bind`第三方标识符或将其留给其他实体绑定。

### 验证令牌的格式[](https://spec.matrix.org/unstable/identity-service-api/#format-of-a-validation-token)

验证令牌的格式由身份服务器决定：它应选择适合3PID类型的格式。（例如，期望用户从SMS消息中复制包含标点符号的长密码短语到客户端是不合适的。）

无论身份服务器使用何种格式，验证令牌最多必须由255个Unicode代码点组成。客户端必须不加修改地传递令牌。

### 电子邮件关联[](https://spec.matrix.org/unstable/identity-service-api/#email-associations)

#### POST /_matrix/identity/v2/validate/email/requestToken[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2validateemailrequesttoken)

---

创建用于验证电子邮件地址的会话。

身份服务器将发送包含令牌的电子邮件。如果将来向身份服务器提供该令牌，则表明该用户能够读取该电子邮件地址的电子邮件，因此我们验证该电子邮件地址的所有权。

注意，主服务器提供代理此API的API，在其上添加额外的行为，例如，`/register/email/requestToken`专为注册帐户时使用，因此会通知用户给定的电子邮件地址是否已在服务器上注册。

注意：为了与本规范的先前草案保持向后兼容，参数也可以指定为`application/x-form-www-urlencoded`数据。然而，这种用法已被弃用。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

##### 请求

###### 请求体

|名称|类型|描述|
|---|---|---|
|`client_secret`|`string`|**必需:** 由客户端生成的唯一字符串，用于标识验证尝试。它必须是由字符`[0-9a-zA-Z.=_-]`组成的字符串。其长度不得超过255个字符且不得为空。|
|`email`|`string`|**必需:** 要验证的电子邮件地址。|
|`next_link`|`string`|可选。当验证完成时，身份服务器将用户重定向到此URL。在通过POST请求提交3PID验证信息时，此选项被忽略。|
|`send_attempt`|`integer`|**必需:** 服务器仅在`send_attempt`是一个大于其已见到的最近一次的数字时才发送电子邮件，范围限定为该`email` + `client_secret`对。这是为了避免在POST用户和身份服务器之间请求重试的情况下重复发送相同的电子邮件。如果他们不这样做，服务器应该响应成功但不重新发送电子邮件。|

###### 请求体示例

```json
{
  "client_secret": "monkeys_are_GREAT",
  "email": "alice@example.org",
  "next_link": "https://example.org/congratulations.html",
  "send_attempt": 1
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|会话已创建。|
|`400`|发生错误。一些可能的错误是:<br><br>- `M_INVALID_EMAIL`: 提供的电子邮件地址无效。<br>- `M_EMAIL_SEND_ERROR`: 验证电子邮件无法发送。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|

###### 200响应

|名称|类型|描述|
|---|---|---|
|`sid`|`string`|**必需:** 会话ID。会话ID是由身份服务器生成的不透明字符串。它们必须完全由字符`[0-9a-zA-Z.=_-]`组成。其长度不得超过255个字符且不得为空。|

```json
{
  "sid": "123abc"
}
```

###### 400响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_INVALID_EMAIL",
  "error": "The email address is not valid"
}
```

###### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

#### GET /_matrix/identity/v2/validate/email/submitToken[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2validateemailsubmittoken)

---

验证电子邮件地址的所有权。

如果三个参数与`requestToken`调用生成的一组一致，则认为电子邮件地址的所有权已被验证。这不会公开发布任何信息，也不会将电子邮件地址与任何Matrix用户ID关联。具体来说，对`/lookup`的调用不会显示绑定。

注意，与POST版本相比，此端点将由终端用户使用，因此响应应为人类可读。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

##### 请求

###### 请求参数
|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`client_secret`|`string`|**必需:** 提供给`requestToken`调用的客户端密钥。|
|`sid`|`string`|**必需:** 由`requestToken`调用生成的会话ID。|
|`token`|`string`|**必需:** 由`requestToken`调用生成并通过电子邮件发送给用户的令牌。|

---

##### 响应

|状态|描述|
|---|---|
|`200`|电子邮件地址已验证。|
|`3XX`|电子邮件地址已验证，并且`requestToken`调用提供了`next_link`参数。用户必须被重定向到`next_link`参数提供的URL。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|
|`4XX`|验证失败。|

###### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

#### POST /_matrix/identity/v2/validate/email/submitToken[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2validateemailsubmittoken)

---

验证电子邮件地址的所有权。

如果三个参数与`requestToken`调用生成的一组一致，则认为电子邮件地址的所有权已被验证。这不会公开发布任何信息，也不会将电子邮件地址与任何Matrix用户ID关联。具体来说，对`/lookup`的调用不会显示绑定。

身份服务器可以不区分大小写地匹配令牌，或执行其他映射操作，如unicode规范化。是否这样做是身份服务器的实现细节。客户端必须始终不加修改地传递令牌。

注意：为了与本规范的先前草案保持向后兼容，参数也可以指定为`application/x-form-www-urlencoded`数据。然而，这种用法已被弃用。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

##### 请求

###### 请求体

|名称|类型|描述|
|---|---|---|
|`client_secret`|`string`|**必需:** 提供给`requestToken`调用的客户端密钥。|
|`sid`|`string`|**必需:** 由`requestToken`调用生成的会话ID。|
|`token`|`string`|**必需:** 由`requestToken`调用生成并通过电子邮件发送给用户的令牌。|

###### 请求体示例

```json
{
  "client_secret": "monkeys_are_GREAT",
  "sid": "1234",
  "token": "atoken"
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|验证成功。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|

###### 200响应

|名称|类型|描述|
|---|---|---|
|`success`|`boolean`|**必需:** 验证是否成功。|

```json
{
  "success": true
}
```

###### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

### 电话号码关联[](https://spec.matrix.org/unstable/identity-service-api/#phone-number-associations)

#### POST /_matrix/identity/v2/validate/msisdn/requestToken[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2validatemsisdnrequesttoken)

---

创建用于验证电话号码的会话。

身份服务器将发送包含令牌的SMS消息。如果将来向身份服务器提供该令牌，则表明该用户能够读取该电话号码的SMS，因此我们验证该电话号码的所有权。

注意，主服务器提供代理此API的API，在其上添加额外的行为，例如，`/register/msisdn/requestToken`专为注册帐户时使用，因此会通知用户给定的电话号码是否已在服务器上注册。

注意：为了与本规范的先前草案保持向后兼容，参数也可以指定为`application/x-form-www-urlencoded`数据。然而，这种用法已被弃用。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

##### 请求

###### 请求体

|名称|类型|描述|
|---|---|---|
|`client_secret`|`string`|**必需:** 由客户端生成的唯一字符串，用于标识验证尝试。它必须是由字符`[0-9a-zA-Z.=_-]`组成的字符串。其长度不得超过255个字符且不得为空。|
|`country`|`string`|**必需:** `phone_number`中号码应解析为如果从该国家拨打的两字母大写ISO-3166-1 alpha-2国家代码。|
|`next_link`|`string`|可选。当验证完成时，身份服务器将用户重定向到此URL。在通过POST请求提交3PID验证信息时，此选项被忽略。|
|`phone_number`|`string`|**必需:** 要验证的电话号码。|
|`send_attempt`|`integer`|**必需:** 服务器仅在`send_attempt`是一个大于其已见到的最近一次的数字时才发送SMS，范围限定为该`country` + `phone_number` + `client_secret`三重组合。这是为了避免在POST用户和身份服务器之间请求重试的情况下重复发送相同的SMS。如果他们不这样做，服务器应该响应成功但不重新发送SMS。|

###### 请求体示例

```json
{
  "client_secret": "monkeys_are_GREAT",
  "country": "GB",
  "next_link": "https://example.org/congratulations.html",
  "phone_number": "07700900001",
  "send_attempt": 1
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|会话已创建。|
|`400`|发生错误。一些可能的错误是:<br><br>- `M_INVALID_ADDRESS`: 提供的电话号码无效。<br>- `M_SEND_ERROR`: 验证SMS无法发送。<br>- `M_DESTINATION_REJECTED`: 身份服务器无法向提供的国家或地区发送SMS。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|

###### 200响应

|名称|类型|描述|
|---|---|---|
|`sid`|`string`|**必需:** 会话ID。会话ID是由身份服务器生成的不透明字符串。它们必须完全由字符`[0-9a-zA-Z.=_-]`组成。其长度不得超过255个字符且不得为空。|

```json
{
  "sid": "123abc"
}
```

###### 400响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_INVALID_ADDRESS",
  "error": "The phone number is not valid"
}
```

###### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

#### GET /_matrix/identity/v2/validate/msisdn/submitToken[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv2validatemsisdnsubmittoken)

---

验证电话号码的所有权。

如果三个参数与`requestToken`调用生成的一组一致，则认为电话号码地址的所有权已被验证。这不会公开发布任何信息，也不会将电话号码与任何Matrix用户ID关联。具体来说，对`/lookup`的调用不会显示绑定。

注意，与POST版本相比，此端点将由终端用户使用，因此响应应为人类可读。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

##### 请求

###### 请求参数
|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`client_secret`|`string`|**必需:** 提供给`requestToken`调用的客户端密钥。|
|`sid`|`string`|**必需:** 由`requestToken`调用生成的会话ID。|
|`token`|`string`|**必需:** 由`requestToken`调用生成并发送给用户的令牌。|

---

##### 响应

|状态|描述|
|---|---|
|`200`|电话号码已验证。|
|`3XX`|电话号码地址已验证，并且`requestToken`调用提供了`next_link`参数。用户必须被重定向到`next_link`参数提供的URL。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|
|`4XX`|验证失败。|

###### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

#### POST /_matrix/identity/v2/validate/msisdn/submitToken[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2validatemsisdnsubmittoken)

---

验证电话号码的所有权。

如果三个参数与`requestToken`调用生成的一组一致，则认为电话号码的所有权已被验证。这不会公开发布任何信息，也不会将电话号码地址与任何Matrix用户ID关联。具体来说，对`/lookup`的调用不会显示绑定。

身份服务器可以不区分大小写地匹配令牌，或执行其他映射操作，如unicode规范化。是否这样做是身份服务器的实现细节。客户端必须始终不加修改地传递令牌。

注意：为了与本规范的先前草案保持向后兼容，参数也可以指定为`application/x-form-www-urlencoded`数据。然而，这种用法已被弃用。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

##### 请求

###### 请求体

|名称|类型|描述|
|---|---|---|
|`client_secret`|`string`|**必需:** 提供给`requestToken`调用的客户端密钥。|
|`sid`|`string`|**必需:** 由`requestToken`调用生成的会话ID。|
|`token`|`string`|**必需:** 由`requestToken`调用生成并发送给用户的令牌。|

###### 请求体示例

```json
{
  "client_secret": "monkeys_are_GREAT",
  "sid": "1234",
  "token": "atoken"
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|验证成功。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|

###### 200响应

|名称|类型|描述|
|---|---|---|
|`success`|`boolean`|**必需:** 验证是否成功。|

```json
{
  "success": true
}
```

###### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

### 通用[](https://spec.matrix.org/unstable/identity-service-api/#general)

#### POST /_matrix/identity/v2/3pid/bind[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv23pidbind)

---

发布会话与Matrix用户ID之间的关联。

对会话的任何3pid进行的未来`/lookup`调用将返回此关联。

注意：为了与本规范的先前草案保持向后兼容，参数也可以指定为`application/x-form-www-urlencoded`数据。然而，这种用法已被弃用。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

##### 请求

###### 请求体

|名称|类型|描述|
|---|---|---|
|`client_secret`|`string`|**必需:** 提供给`requestToken`调用的客户端密钥。|
|`mxid`|`string`|**必需:** 要与3pid关联的Matrix用户ID。|
|`sid`|`string`|**必需:** 由`requestToken`调用生成的会话ID。|

###### 请求体示例

```json
{
  "client_secret": "monkeys_are_GREAT",
  "mxid": "@ears:matrix.org",
  "sid": "1234"
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|关联已发布。|
|`400`|关联未发布。<br><br>如果会话尚未验证，则`errcode`将为`M_SESSION_NOT_VALIDATED`。如果会话已超时，则`errcode`将为`M_SESSION_EXPIRED`。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|
|`404`|未找到会话ID或客户端密钥|

###### 200响应

|名称|类型|描述|
|---|---|---|
|`address`|`string`|**必需:** 正在查找的用户的3pid地址。|
|`medium`|`string`|**必需:** 3pid的媒体类型。|
|`mxid`|`string`|**必需:** 与3pid关联的Matrix用户ID。|
|`not_after`|`integer`|**必需:** 关联不再被认为有效的unix时间戳。|
|`not_before`|`integer`|**必需:** 关联不被认为有效之前的unix时间戳。|
|`signatures`|`{string: {string: string}}`|**必需:** 验证身份服务器的签名，显示如果您信任验证身份服务，则应信任关联。|
|`ts`|`integer`|**必需:** 验证关联的unix时间戳。|

```json
{
  "address": "louise@bobs.burgers",
  "medium": "email",
  "mxid": "@ears:matrix.org",
  "not_after": 4582425849161,
  "not_before": 1428825849161,
  "signatures": {
    "matrix.org": {
      "ed25519:0": "ENiU2YORYUJgE6WBMitU0mppbQjidDLanAusj8XS2nVRHPu+0t42OKA/r6zV6i2MzUbNQ3c3MiLScJuSsOiVDQ"
    }
  },
  "ts": 1428825849161
}
```

###### 400响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_SESSION_NOT_VALIDATED",
  "error": "This validation session has not yet been completed"
}
```

###### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

###### 404响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_NO_VALID_SESSION",
  "error": "No valid session was found matching that sid and client secret"
}
```

#### GET /_matrix/identity/v2/3pid/getValidated3pid[](https://spec.matrix.org/unstable/identity-service-api/#get_matrixidentityv23pidgetvalidated3pid)

---

确定给定的3pid是否已被用户验证。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

##### 请求

###### 请求参数
|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`client_secret`|`string`|**必需:** 提供给`requestToken`调用的客户端密钥。|
|`sid`|`string`|**必需:** 由`requestToken`调用生成的会话ID。|

---

##### 响应

|状态|描述|
|---|---|
|`200`|会话的验证信息。|
|`400`|会话尚未验证。<br><br>如果会话尚未验证，则`errcode`将为`M_SESSION_NOT_VALIDATED`。如果会话已超时，则`errcode`将为`M_SESSION_EXPIRED`。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|
|`404`|未找到会话ID或客户端密钥。|

###### 200响应

|名称|类型|描述|
|---|---|---|
|`address`|`string`|**必需:** 正在查找的3pid的地址。|
|`medium`|`string`|**必需:** 3pid的媒体类型。|
|`validated_at`|`integer`|**必需:** 时间戳，以毫秒为单位，指示3pid被验证的时间。|

```json
{
  "address": "louise@bobs.burgers",
  "medium": "email",
  "validated_at": 1457622739026
}
```

###### 400响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_SESSION_NOT_VALIDATED",
  "error": "This validation session has not yet been completed"
}
```

###### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

###### 404响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_NO_VALID_SESSION",
  "error": "No valid session was found matching that sid and client secret"
}
```

#### POST /_matrix/identity/v2/3pid/unbind[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv23pidunbind)

---

移除会话与Matrix用户ID之间的关联。

对会话的任何3pid进行的未来`/lookup`调用将不返回已移除的关联。

身份服务器应通过以下两种方式之一对请求进行身份验证：

1. 请求由控制`user_id`的主服务器签名。
2. 请求包括`sid`和`client_secret`参数，如`/3pid/bind`，这证明了3PID的所有权。

如果此端点返回JSON Matrix错误，则该错误应传递给通过主服务器请求解绑的客户端，如果主服务器代表客户端操作。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

##### 请求

###### 请求体

|名称|类型|描述|
|---|---|---|
|`client_secret`|`string`|提供给`requestToken`调用的客户端密钥。|
|`mxid`|`string`|**必需:** 要从3pid中移除的Matrix用户ID。|
|`sid`|`string`|由`requestToken`调用生成的会话ID。|
|`threepid`|[3PID](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv23pidunbind_request_3pid)|**必需:** 要移除的3PID。必须与使用`sid`和`client_secret`进行身份验证此请求时生成会话的3PID匹配。|

|3PID|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`address`|`string`|**必需:** 要移除的3PID地址。|
|`medium`|`string`|**必需:** 来自[3PID类型](https://spec.matrix.org/unstable/appendices/#3pid-types)附录的媒体，匹配要解绑的标识符的媒体。|

###### 请求体示例

```json
{
  "client_secret": "monkeys_are_GREAT",
  "mxid": "@ears:example.org",
  "sid": "1234",
  "threepid": {
    "address": "monkeys_have_ears@example.org",
    "medium": "email"
  }
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|关联已成功移除。|
|`400`|如果响应体不是JSON Matrix错误，身份服务器不支持解绑。如果响应体中有JSON Matrix错误，请求方应尊重错误。|
|`403`|用于验证请求的凭据无效。如果身份服务器不支持所选的身份验证方法（例如阻止主服务器解绑标识符），也可能返回此错误。<br><br>另一个常见的错误代码是`M_TERMS_NOT_SIGNED`，用户需要[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)才能继续。|
|`404`|如果响应体不是JSON Matrix错误，身份服务器不支持解绑。如果响应体中有JSON Matrix错误，请求方应尊重错误。|
|`501`|如果响应体不是JSON Matrix错误，身份服务器不支持解绑。如果响应体中有JSON Matrix错误，请求方应尊重错误。|

###### 200响应

```json
{}
```

###### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_FORBIDDEN",
  "error": "Invalid homeserver signature"
}
```

## 邀请存储[](https://spec.matrix.org/unstable/identity-service-api/#invitation-storage)

身份服务器可以存储对用户3PID的待处理邀请，这些邀请将在3PID与Matrix用户ID关联时被检索并可以被通知或查找。

稍后，如果该特定3PID的所有者将其与Matrix用户ID绑定，身份服务器将尝试通过[/3pid/onbind](https://spec.matrix.org/unstable/server-server-api#put_matrixfederationv13pidonbind)端点向Matrix用户的主服务器发出HTTP POST请求。请求必须使用身份服务器的长期私钥签名。

### POST /_matrix/identity/v2/store-invite[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2store-invite)

---

存储对用户3pid的待处理邀请。

除了下面指定的请求参数外，还可以指定任意数量的其他参数。这些参数可以用于下面描述的邀请消息生成。

服务将生成一个随机令牌和一个用于接受邀请的临时密钥。

服务还会为邀请者生成一个`display_name`，这是`address`的一个经过编辑的版本，不会泄露`address`的完整内容。

服务会持久记录所有上述信息。

它还会生成一封包含所有这些数据的电子邮件，发送到`address`参数，通知他们邀请。电子邮件应引用此处请求中的`inviter_name`、`room_name`、`room_avatar`和`room_type`（如果存在）。

此外，生成的临时公钥将在请求`/_matrix/identity/v2/pubkey/ephemeral/isvalid`时列为有效。

目前，邀请只能针对`email`媒体的3pid发出。

请求中的可选字段应尽可能由服务器填充。身份服务器可以在通知`address`待处理邀请时使用这些变量以供显示。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

#### 请求

##### 请求体

|名称|类型|描述|
|---|---|---|
|`address`|`string`|**必需:** 被邀请用户的电子邮件地址。|
|`medium`|`string`|**必需:** 字面字符串`email`。|
|`room_alias`|`string`|用户被邀请的房间的Matrix房间别名。这应从`m.room.canonical_alias`状态事件中检索。|
|`room_avatar_url`|`string`|用户被邀请的房间的内容URI。这应从`m.room.avatar`状态事件中检索。|
|`room_id`|`string`|**必需:** 用户被邀请的Matrix房间ID|
|`room_join_rules`|`string`|用户被邀请的房间的`join_rule`。这应从`m.room.join_rules`状态事件中检索。|
|`room_name`|`string`|用户被邀请的房间名称。这应从`m.room.name`状态事件中检索。|
|`room_type`|`string`|`m.room.create`事件的`content`中的`type`。如果创建事件没有指定`type`，则不包括此字段。|
|`sender`|`string`|**必需:** 发出邀请的用户的Matrix用户ID|
|`sender_avatar_url`|`string`|发出邀请的用户ID的头像的内容URI。|
|`sender_display_name`|`string`|发出邀请的用户ID的显示名称。|

##### 请求体示例

```json
{
  "address": "foo@example.com",
  "medium": "email",
  "room_alias": "#somewhere:example.org",
  "room_avatar_url": "mxc://example.org/s0meM3dia",
  "room_id": "!something:example.org",
  "room_join_rules": "public",
  "room_name": "Bob's Emporium of Messages",
  "room_type": "m.space",
  "sender": "@bob:example.com",
  "sender_avatar_url": "mxc://example.org/an0th3rM3dia",
  "sender_display_name": "Bob Smith"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|邀请已存储。|
|`400`|发生错误。<br><br>如果3pid已绑定到Matrix用户ID，错误代码将为`M_THREEPID_IN_USE`。如果媒体不受支持，错误代码将为`M_UNRECOGNIZED`。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`display_name`|`string`|**必需:** 生成的（编辑过的）显示名称。|
|`public_keys`|[[PublicKey](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2store-invite_response-200_publickey)]|**必需:** [服务器的长期公钥，生成的临时公钥]列表。|
|`token`|`string`|**必需:** 生成的令牌。必须是由字符`[0-9a-zA-Z.=_-]`组成的字符串。其长度不得超过255个字符且不得为空。|

|PublicKey|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`key_validity_url`|`string`|**必需:** 通过将其作为`public_key`查询参数传递来检查此密钥有效性的端点的URI。参见[密钥管理](https://spec.matrix.org/unstable/identity-service-api/#key-management)。|
|`public_key`|`string`|**必需:** 使用[unpadded Base64](https://spec.matrix.org/unstable/appendices/#unpadded-base64)编码的公钥。|

```json
{
  "display_name": "f...@b...",
  "public_keys": [
    {
      "key_validity_url": "https://example.com/_matrix/identity/v2/pubkey/isvalid",
      "public_key": "serverPublicKeyBase64"
    },
    {
      "key_validity_url": "https://example.com/_matrix/identity/v2/pubkey/ephemeral/isvalid",
      "public_key": "ephemeralPublicKeyBase64"
    }
  ],
  "token": "sometoken"
}
```

##### 400响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_THREEPID_IN_USE",
  "error": "Binding already known",
  "mxid": "@alice:example.com"
}
```

##### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

## 临时邀请签名[](https://spec.matrix.org/unstable/identity-service-api/#ephemeral-invitation-signing)

为了帮助可能无法自行执行加密的客户端，身份服务器提供了一些加密功能以帮助接受邀请。这比客户端自己执行要不安全，但在无法实现的情况下可能有用。

### POST /_matrix/identity/v2/sign-ed25519[](https://spec.matrix.org/unstable/identity-service-api/#post_matrixidentityv2sign-ed25519)

---

签署邀请详情。

身份服务器将查找在`store-invite`调用中存储的`token`，并获取邀请的发送者。

|   |   |
|---|---|
|速率限制:|否|
|需要认证:|是|

---

#### 请求

##### 请求体

|名称|类型|描述|
|---|---|---|
|`mxid`|`string`|**必需:** 接受邀请的用户的Matrix用户ID。|
|`private_key`|`string`|**必需:** 私钥，编码为[Unpadded base64](https://spec.matrix.org/unstable/appendices/#unpadded-base64)。|
|`token`|`string`|**必需:** 来自`store-invite`调用的令牌。|

##### 请求体示例

```json
{
  "mxid": "@foo:bar.com",
  "private_key": "base64encodedkey",
  "token": "sometoken"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|mxid、发送者和令牌的签名JSON。|
|`403`|用户必须执行某些操作才能使用此端点。一个例子是`M_TERMS_NOT_SIGNED`错误，用户必须[同意更多条款](https://spec.matrix.org/unstable/identity-service-api/#terms-of-service)。|
|`404`|未找到令牌。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`mxid`|`string`|**必需:** 接受邀请的用户的Matrix用户ID。|
|`sender`|`string`|**必需:** 发送邀请的用户的Matrix用户ID。|
|`signatures`|`{string: {string: string}}`|**必需:** mxid、发送者和令牌的签名。|
|`token`|`string`|**必需:** 邀请的令牌。|

```json
{
  "mxid": "@foo:bar.com",
  "sender": "@baz:bar.com",
  "signatures": {
    "my.id.server": {
      "ed25519:0": "def987"
    }
  },
  "token": "abc123"
}
```

##### 403响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TERMS_NOT_SIGNED",
  "error": "Please accept our updated terms of service before continuing"
}
```

##### 404响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_UNRECOGNIZED",
  "error": "Didn't recognize token"
}
```