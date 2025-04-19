
Matrix 客户端-服务器 API 和服务器-服务器 API 提供了实现一致的自包含联邦消息传递结构的手段。然而，它们在 Matrix 中实现自定义服务器端行为（例如网关、过滤器、可扩展钩子等）方面提供的手段有限。应用服务 API（AS API）定义了一个标准 API，以便在不考虑底层主服务器实现的情况下实现这种可扩展功能。

# 应用服务

应用服务是被动的，只能观察来自主服务器的事件。它们可以将事件注入到它们参与的房间中。它们不能阻止事件的发送，也不能修改正在发送的事件的内容。为了观察来自主服务器的事件，需要配置主服务器以将某些类型的流量传递给应用服务。这是通过手动配置主服务器与应用服务相关的信息来实现的。

## 注册

> [!info] 信息：
> 以前，应用服务可以通过 HTTP API 向主服务器注册。这被移除，因为它被视为安全风险。被攻陷的应用服务可以重新注册一个全局 `*` 正则表达式并嗅探主服务器上的所有流量。为了防止这种情况，应用服务现在必须通过配置文件注册，这些配置文件与主服务器配置文件链接。配置文件的添加允许主服务器管理员检查注册中的可疑正则表达式字符串。

应用服务注册用户 ID、房间别名和房间 ID 的“命名空间”。如果某个事件与任何命名空间匹配，则称应用服务对该事件“感兴趣”。

应用服务还可以声明它们是否应该是唯一可以管理指定命名空间的服务。这被称为“独占”命名空间。独占命名空间阻止人类和其他应用服务在该命名空间中创建/删除实体。通常，当房间代表另一个服务（例如 IRC）上的真实房间时，使用独占命名空间。当应用服务仅仅是增强房间本身时（例如提供日志记录或搜索功能），则使用非独占命名空间。

注册由一系列键值对表示，通常编码为 YAML 文件中的对象。其结构如下：

### `注册`

| 名称                 | 类型                                                                                                      | 描述                                                           |
| ------------------ | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `as_token`         | `string`                                                                                                | **必需：** 应用服务用于向主服务器认证请求的秘密令牌。                                |
| `hs_token`         | `string`                                                                                                | **必需：** 主服务器用于向应用服务认证请求的秘密令牌。                                |
| `id`               | `string`                                                                                                | **必需：** 应用服务的唯一用户定义 ID，永远不会更改。                               |
| `namespaces`       | [Namespaces](https://spec.matrix.org/v1.11/application-service-api/#definition-registration_namespaces) | **必需：** 应用服务感兴趣的命名空间。                                        |
| `protocols`        | `[string]`                                                                                              | 应用服务提供的外部协议（例如 IRC）。                                         |
| `rate_limited`     | `boolean`                                                                                               | 来自伪装用户的请求是否有限速。发送者被排除在外。                                     |
| `sender_localpart` | `string`                                                                                                | **必需：** 与应用服务关联的用户的本地部分。如果此用户是事件的目标，或是事件发生的房间的成员，则事件将发送到 AS。 |
| `url`              | `string`                                                                                                | **必需：** 应用服务的 URL。可以在域名后包含路径。如果不需要流量，可以选择设置为 null。           |

| 命名空间 |
|---|---|---|
| 名称 | 类型 | 描述 |
|---|---|---|
| `aliases` | [[Namespace](https://spec.matrix.org/v1.11/application-service-api/#definition-registration_namespace)] | 定义应用服务感兴趣的房间别名的命名空间列表。所有在与命名空间匹配的别名的房间中发送的事件将发送到 AS。 |
| `rooms` | [[Namespace](https://spec.matrix.org/v1.11/application-service-api/#definition-registration_namespace)] | 定义应用服务感兴趣的房间 ID 的命名空间列表。所有在与命名空间匹配的 ID 的房间中发送的事件将发送到 AS。 |
| `users` | [[Namespace](https://spec.matrix.org/v1.11/application-service-api/#definition-registration_namespace)] | 定义应用服务感兴趣的用户 ID 的命名空间列表，除了其 `sender_localpart`。如果本地用户与命名空间之一匹配，则事件将发送到 AS，或者是事件发生的房间的成员。 |

| 命名空间 |
|---|---|---|
| 名称 | 类型 | 描述 |
|---|---|---|
| `exclusive` | `boolean` | **必需：** 一个真或假值，表示此应用服务是否对该命名空间内的事件具有独占访问权。 |
| `regex` | `string` | **必需：** 定义该命名空间包含哪些值的 POSIX 正则表达式。 |

#### 示例

```json
{}
```

独占用户和别名命名空间应在符号后以下划线开头，以避免与主服务器上的其他用户发生冲突。应用服务还应尝试在保留的命名空间中识别它们代表的服务。例如，`@_irc_.*` 将是一个为处理 IRC 的应用服务注册的好命名空间。

下面是一个 IRC 桥接应用服务的注册文件示例：

```
id: "IRC Bridge"
url: "http://127.0.0.1:1234"
as_token: "30c05ae90a248a4188e620216fa72e349803310ec83e2a77b34fe90be6081f46"
hs_token: "312df522183efd404ec1cd22d2ffa4bbc76a8c1ccf541dd692eef281356bb74e"
sender_localpart: "_irc_bot" # 将导致 @_irc_bot:example.org
namespaces:
  users:
    - exclusive: true
      regex: "@_irc_bridge_.*"
  aliases:
    - exclusive: false
      regex: "#_irc_bridge_.*"
  rooms: []
```

> [!info] 信息：
> 对于 `users` 命名空间，应用服务只能注册对本地用户（即用户 ID 以本地主服务器的 `server_name` 结尾的用户）的兴趣。即使用户与 `users` 命名空间之一匹配，影响其他主服务器上的用户的事件也不会发送到应用服务（当然，除非事件影响应用服务对另一个房间感兴趣的房间 - 例如，因为房间中有另一个应用服务感兴趣的用户）。
> 
> 对于 `rooms` 和 `aliases` 命名空间，所有在匹配房间中的事件将发送到应用服务。

> [!warning] 警告：
> 如果相关主服务器有多个应用服务，则每个 `as_token` 和 `id` 必须在每个应用服务中唯一，因为这些用于识别应用服务。主服务器必须强制执行这一点。

## 主服务器 -> 应用服务 API

### 授权

**[在 `v1.4` 中更改]**

主服务器在向应用服务发出请求时，必须包含一个 `Authorization` 头，其中包含应用服务注册中的 `hs_token`。应用服务必须验证提供的 `Bearer` 令牌是否与其已知的 `hs_token` 匹配，如果不匹配，则请求失败并返回 `M_FORBIDDEN` 错误。

`Authorization` 头的格式类似于 [客户端-服务器 API](https://spec.matrix.org/v1.11/client-server-api/#client-authentication)：`Bearer TheHSTokenGoesHere`。

> [!info] 信息：
> 在此规范的早期版本中，使用了 `access_token` 查询参数。服务器应仅在支持规范的旧版本时发送此查询参数。
> 
> 如果发送 `query_string`，建议与 `Authorization` 头一起发送以实现最大兼容性。
>
> 如果同时提供，应用服务应确保两者匹配。

### 旧版路由

应用服务规范的早期草案中有一些在野外使用了很长时间的端点。应用服务规范现在在所有端点上定义了一个版本，以便与 Matrix 规范的其余部分和未来更兼容。

主服务器在与应用服务通信时应首先尝试使用指定的端点。然而，如果应用服务收到的 HTTP 状态代码不表示成功（例如：404、500、501 等），则主服务器应回退到应用服务的旧端点。

旧端点具有完全相同的请求体和响应格式，只是属于不同的路径。每个的等效路径如下：

- `/_matrix/app/v1/transactions/{txnId}` 应回退到 `/transactions/{txnId}`
- `/_matrix/app/v1/users/{userId}` 应回退到 `/users/{userId}`
- `/_matrix/app/v1/rooms/{roomAlias}` 应回退到 `/rooms/{roomAlias}`
- `/_matrix/app/v1/thirdparty/protocol/{protocol}` 应回退到 `/_matrix/app/unstable/thirdparty/protocol/{protocol}`
- `/_matrix/app/v1/thirdparty/user/{user}` 应回退到 `/_matrix/app/unstable/thirdparty/user/{user}`
- `/_matrix/app/v1/thirdparty/location/{location}` 应回退到 `/_matrix/app/unstable/thirdparty/location/{location}`
- `/_matrix/app/v1/thirdparty/user` 应回退到 `/_matrix/app/unstable/thirdparty/user`
- `/_matrix/app/v1/thirdparty/location` 应回退到 `/_matrix/app/unstable/thirdparty/location`

主服务器应定期重试较新的端点，因为应用服务可能已更新。

### 未知路由

如果收到对不支持（或未知）端点的请求，则服务器必须响应 404 `M_UNRECOGNIZED` 错误。

同样，405 `M_UNRECOGNIZED` 错误用于表示对已知端点的不支持方法。

### 推送事件

应用服务 API 提供了一个事务 API 用于发送事件列表。每个事件列表包括一个事务 ID，其工作原理如下：

```
    典型
    HS ---> AS : 主服务器发送带有事务 ID T 的事件。
       <---    : 应用服务返回 200 OK。
```

```
    AS ACK 丢失
    HS ---> AS : 主服务器发送带有事务 ID T 的事件。
       <-/-    : AS 200 OK 丢失。
    HS ---> AS : 主服务器使用相同的事务 ID T 重试。
       <---    : 应用服务返回 200 OK。如果 AS 已经处理了这些事件，它可以对该请求不作任何操作（它知道是否是相同的事件基于事务 ID）。
```

发送到应用服务的事件应线性化，就像它们来自事件流一样。主服务器必须维护一个事务队列以发送到应用服务。如果无法访问应用服务，主服务器应指数级退避，直到应用服务再次可访问。由于应用服务无法修改事件，因此这些请求可以在不阻塞主服务器其他方面的情况下进行。主服务器在重试时不得更改（例如添加更多）它们打算在该事务 ID 中发送的事件，因为应用服务可能已经处理了这些事件。

#### PUT /_matrix/app/v1/transactions/{txnId}

---

当主服务器想要将事件（或事件批次）推送到应用服务时调用此 API。

请注意，应用服务应通过 `state_key` 的存在来区分状态事件和消息事件，而不是通过事件类型。

|   |   |
|---|---|
|限速：|否|
|需要认证：|是|

---

##### 请求

###### 请求参数
|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`txnId`|`string`|**必需：** 此事件集的事务 ID。主服务器生成这些 ID，用于确保请求的幂等性。|

###### 请求体

|名称|类型|描述|
|---|---|---|
|`events`|[[ClientEvent](https://spec.matrix.org/v1.11/application-service-api/#put_matrixappv1transactionstxnid_request_clientevent)]|**必需：** 事件列表，格式与客户端-服务器 API 相同。|

|ClientEvent|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|`object`|**必需：** 由发送该事件的客户端创建的事件主体。|
|`event_id`|`string`|**必需：** 此事件的全局唯一标识符。|
|`origin_server_ts`|`integer`|**必需：** 事件在源主服务器发送时的时间戳（自 Unix 纪元以来的毫秒数）。|
|`room_id`|`string`|**必需：** 与此事件关联的房间 ID。|
|`sender`|`string`|**必需：** 发送此事件的用户的完全限定 ID。|
|`state_key`|`string`|仅当且仅当此事件是状态事件时存在。使此状态在房间中唯一的键。请注意，它通常是一个空字符串。<br><br>以 `@` 开头的状态键保留用于引用用户 ID，例如房间成员。除少数事件外，使用给定用户 ID 作为状态键设置的状态事件必须仅由该用户设置。|
|`type`|`string`|**必需：** 事件的类型。|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/application-service-api/#put_matrixappv1transactionstxnid_request_unsigneddata)|包含有关事件的可选额外信息。|

|UnsignedData|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`age`|`integer`|自事件发送以来经过的时间（以毫秒为单位）。此字段由本地主服务器生成，如果两个服务器中的至少一个的本地时间不同步，可能会不正确，这可能导致年龄为负或大于实际年龄。|
|`membership`|`string`|请求用户在事件时的房间成员身份。<br><br>此属性是请求用户的 `m.room.member` 状态在事件点的 `membership` 属性的值，包括事件引起的任何更改。如果用户在事件时尚未加入房间（即，他们没有 `m.room.member` 状态），则此属性设置为 `leave`。<br><br>主服务器应尽可能填充此属性，但如果需要可以省略（例如，如果计算值很昂贵，服务器可能选择仅在加密房间中实现它）。通常不会在通过应用服务事务 API 推送到应用服务的事件中填充该属性（在这种情况下，没有明确的“请求用户”定义）。<br><br>**在 `v1.11` 中添加**|
|`prev_content`|`EventContent`|此事件的先前 `content`。此字段由本地主服务器生成，仅在事件是状态事件且客户端有权限查看先前内容时返回。  <br>  <br>**在 `v1.2` 中更改：** 以前，此字段在返回的事件的顶层指定，而不是在 `unsigned` 中（除了 [`GET .../notifications`](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv3notifications) 端点），尽管实际上没有已知的服务器实现尊重这一点。|
|`redacted_because`|`ClientEvent`|如果有，撤销此事件的事件。|
|`transaction_id`|`string`|客户端提供的 [transaction ID](https://spec.matrix.org/v1.11/client-server-api/#transaction-identifiers)，例如，通过 `PUT /_matrix/client/v3/rooms/{roomId}/send/{eventType}/{txnId}` 提供，如果客户端接收到的事件与发送该事件的客户端相同。|

###### 请求体示例

```json
{
  "events": [
    {
      "content": {
        "avatar_url": "mxc://example.org/SEsfnsuifSDFSSEF",
        "displayname": "Alice Margatroid",
        "membership": "join",
        "reason": "Looking for support"
      },
      "event_id": "$143273582443PhrSn:example.org",
      "origin_server_ts": 1432735824653,
      "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
      "sender": "@example:example.org",
      "state_key": "@alice:example.org",
      "type": "m.room.member",
      "unsigned": {
        "age": 1234,
        "membership": "join"
      }
    },
    {
      "content": {
        "body": "This is an example text message",
        "format": "org.matrix.custom.html",
        "formatted_body": "<b>This is an example text message</b>",
        "msgtype": "m.text"
      },
      "event_id": "$143273582443PhrSn:example.org",
      "origin_server_ts": 1432735824653,
      "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
      "sender": "@example:example.org",
      "type": "m.room.message",
      "unsigned": {
        "age": 1234,
        "membership": "join"
      }
    }
  ]
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|事务处理成功。|

###### 200 响应

```json
{}
```

### Pinging [](https://spec.matrix.org/unstable/application-service-api/#pinging)

**[在 `v1.7` 中添加]**

应用服务 API 包括一个 ping 机制，允许应用服务确保主服务器可以访问应用服务。应用服务可以使用此机制来检测配置错误并适当报告。

使用此机制的实现应注意在临时问题的情况下不完全失败，例如优雅地处理应用服务在主服务器之前启动的情况。

机制的工作原理如下（注意：为了简洁省略了人类可读的 `error` 字段）：

**典型**

```
AS ---> HS : /_matrix/client/v1/appservice/{appserviceId}/ping {"transaction_id": "meow"}
    HS ---> AS : /_matrix/app/v1/ping {"transaction_id": "meow"}
    HS <--- AS : 200 OK {}
AS <--- HS : 200 OK {"duration_ms": 123}
```

**不正确的 `hs_token`**

```
AS ---> HS : /_matrix/client/v1/appservice/{appserviceId}/ping {"transaction_id": "meow"}
    HS ---> AS : /_matrix/app/v1/ping {"transaction_id": "meow"}
    HS <--- AS : 403 Forbidden {"errcode": "M_FORBIDDEN"}
AS <--- HS : 502 Bad Gateway {"errcode": "M_BAD_STATUS", "status": 403, "body": "{\"errcode\": \"M_FORBIDDEN\"}"}
```

**无法连接到应用服务**

```
AS ---> HS : /_matrix/client/v1/appservice/{appserviceId}/ping {"transaction_id": "meow"}
    HS -/-> AS : /_matrix/app/v1/ping {"transaction_id": "meow"}
AS <--- HS : 502 Bad Gateway {"errcode": "M_CONNECTION_FAILED"}
```

`/_matrix/app/v1/ping` 端点在此处描述。[`/_matrix/client/v1/appservice/{appserviceId}/ping`](https://spec.matrix.org/v1.11/application-service-api/#post_matrixclientv1appserviceappserviceidping) 端点在下面的客户端-服务器 API 扩展部分中。

#### POST /_matrix/app/v1/ping

---

**在 `v1.7` 中添加**

此 API 由主服务器调用，以确保连接正常工作且主服务器拥有的 `hs_token` 正确。

目前，这仅由主服务器作为应用服务调用 [`POST /_matrix/client/v1/appservice/{appserviceId}/ping`](https://spec.matrix.org/v1.11/application-service-api/#post_matrixclientv1appserviceappserviceidping) 的直接结果调用。

|   |   |
|---|---|
|限速：|否|
|需要认证：|是|

---

##### 请求

###### 请求体

|名称|类型|描述|
|---|---|---|
|`transaction_id`|`string`|ping 的事务 ID，直接从 `POST /_matrix/client/v1/appservice/{appserviceId}/ping` 调用中复制。|

###### 请求体示例

```json
{
  "transaction_id": "mautrix-go_1683636478256400935_123"
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|提供的 `hs_token` 有效，ping 请求成功。|

###### 200 响应

```json
{}
```

### 查询

应用服务 API 包括两个查询 API：用于房间别名和用户 ID。应用服务应创建查询的实体（如果需要）。在此过程中，应用服务阻止主服务器，直到实体被创建和配置。如果主服务器没有收到对此请求的响应，主服务器应重试几次，然后超时。这应导致发起此请求的客户端（例如加入房间别名）收到 HTTP 状态 408 “请求超时”。

> [!info] 理由：
> 阻止主服务器并期望应用服务使用客户端-服务器 API 创建实体比其他方法（例如返回初始同步样式 JSON blob 并让 HS 配置房间/用户）更简单、更灵活。这也意味着不需要“后渠道”来通知应用服务有关实体的信息，例如房间 ID 到房间别名的映射。

#### GET /_matrix/app/v1/users/{userId}

---

此端点由主服务器在应用服务上调用，以查询给定用户 ID 的存在。主服务器只会查询应用服务的 `users` 命名空间内的用户 ID。当主服务器收到应用服务命名空间中未知用户 ID 的事件时（例如房间邀请），将发送此请求。

|   |   |
|---|---|
|限速：|否|
|需要认证：|是|

---

##### 请求

###### 请求参数
|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`userId`|`string`|**必需：** 被查询的用户 ID。|

---

##### 响应

|状态|描述|
|---|---|
|`200`|应用服务指示该用户存在。应用服务必须使用客户端-服务器 API 创建用户。|
|`401`|主服务器未向应用服务提供凭据。可选的错误信息可以包含在此响应的正文中。|
|`403`|主服务器提供的凭据被拒绝。|
|`404`|应用服务指示该用户不存在。可选的错误信息可以包含在此响应的正文中。|

###### 200 响应

```json
{}
```

###### 401 响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403 响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404 响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/rooms/{roomAlias}

---

此端点由主服务器在应用服务上调用，以查询给定房间别名的存在。主服务器只会查询应用服务的 `aliases` 命名空间内的房间别名。当主服务器收到请求加入应用服务命名空间内的房间别名时，将发送此请求。

|   |   |
|---|---|
|限速：|否|
|需要认证：|是|

---

##### 请求

###### 请求参数
|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomAlias`|`string`|**必需：** 被查询的房间别名。|

---

##### 响应

|状态|描述|
|---|---|
|`200`|应用服务指示该房间别名存在。应用服务必须使用客户端-服务器 API 创建房间并将其与查询的房间别名关联。在响应之前可以设置有关房间的其他信息，例如其名称和主题。|
|`401`|主服务器未向应用服务提供凭据。可选的错误信息可以包含在此响应的正文中。|
|`403`|主服务器提供的凭据被拒绝。|
|`404`|应用服务指示该房间别名不存在。可选的错误信息可以包含在此响应的正文中。|

###### 200 响应

```json
{}
```

###### 401 响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403 响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404 响应
|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

### 第三方网络

应用服务可以通过其注册配置向主服务器声明其支持的协议。这些网络通常用于应用服务管理的第三方服务，如IRC。应用服务可以根据客户端-服务器API扩展中定义的协议，填充其注册协议的Matrix房间目录。

每个协议可能有多个“位置”（也称为“第三方位置”或“3PLs”）。协议中的位置是第三方网络中的一个地方，例如IRC频道。第三方网络的用户也可以由应用服务表示。

可以通过应用服务定义的字段（如显示名称或其他属性）来搜索位置和用户。当客户端请求主服务器在特定“网络”（协议）中搜索时，搜索字段将传递给应用服务进行过滤。

#### GET /_matrix/app/v1/thirdparty/location

从Matrix房间别名中检索第三方网络位置的数组。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

##### 请求

###### 请求参数

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`alias`|`string`|要查找的Matrix房间别名。|

##### 响应

|状态|描述|
|---|---|
|`200`|找到的所有第三方位置。|
|`401`|主服务器未向应用服务提供凭据。响应体中可以包含可选的错误信息。|
|`403`|主服务器提供的凭据被拒绝。|
|`404`|未找到与给定参数匹配的映射。|

###### 200响应

`Location`数组。

|Location|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`alias`|`string`|**必需：** Matrix房间的别名。|
|`fields`|`object`|**必需：** 用于标识此第三方位置的信息。|
|`protocol`|`string`|**必需：** 第三方位置所属的协议ID。|

```json
[
  {
    "alias": "#freenode_#matrix:matrix.org",
    "fields": {
      "channel": "#matrix",
      "network": "freenode"
    },
    "protocol": "irc"
  }
]
```

###### 401响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/thirdparty/location/{protocol}

检索通向匹配的第三方位置的Matrix门户房间列表。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

##### 请求

###### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`protocol`|`string`|**必需：** 协议ID。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`fields`|`{string: string}`|一个或多个自定义字段，传递给应用服务以帮助识别第三方位置。|

##### 响应

|状态|描述|
|---|---|
|`200`|至少找到一个门户房间。|
|`401`|主服务器未向应用服务提供凭据。响应体中可以包含可选的错误信息。|
|`403`|主服务器提供的凭据被拒绝。|
|`404`|未找到与给定参数匹配的映射。|

###### 200响应

`Location`数组。

|Location|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`alias`|`string`|**必需：** Matrix房间的别名。|
|`fields`|`object`|**必需：** 用于标识此第三方位置的信息。|
|`protocol`|`string`|**必需：** 第三方位置所属的协议ID。|

```json
[
  {
    "alias": "#freenode_#matrix:matrix.org",
    "fields": {
      "channel": "#matrix",
      "network": "freenode"
    },
    "protocol": "irc"
  }
]
```

###### 401响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/thirdparty/protocol/{protocol}

当主服务器希望向客户端展示应用服务支持的各种第三方网络的特定信息时，会调用此API。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

##### 请求

###### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`protocol`|`string`|**必需：** 协议ID。|

##### 响应

|状态|描述|
|---|---|
|`200`|协议已找到并返回元数据。|
|`401`|主服务器未向应用服务提供凭据。响应体中可以包含可选的错误信息。|
|`403`|主服务器提供的凭据被拒绝。|
|`404`|未找到与给定路径匹配的协议。|

###### 200响应

|Protocol|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`field_types`|{string: [Field Type]}|**必需：** `user_fields`和`location_fields`中定义字段的类型定义。每个数组中的条目必须在此处有一个条目。如果未定义字段，则可以为空对象。|
|`icon`|`string`|**必需：** 表示第三方协议图标的内容URI。|
|`instances`|[[Protocol Instance]]|**必需：** 表示独立配置实例的对象列表。例如，如果同一应用服务提供多个IRC网络。|
|`location_fields`|`[string]`|**必需：** 可用于标识第三方位置的字段。这些字段应按顺序排列，以建议实体可能被分组的方式，其中较高的分组首先排列。例如，网络名称应在频道名称之前搜索。|
|`user_fields`|`[string]`|**必需：** 可用于标识第三方用户的字段。这些字段应按顺序排列，以建议实体可能被分组的方式，其中较高的分组首先排列。例如，网络名称应在用户昵称之前搜索。|

|Field Type|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`placeholder`|`string`|**必需：** 作为字段值的有效示例的占位符。|
|`regexp`|`string`|**必需：** 用于验证字段值的正则表达式。这可能相对粗略，因为提供此协议的应用服务可能会应用额外的验证或过滤。|

|Protocol Instance|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`desc`|`string`|**必需：** 协议的人类可读描述，例如名称。|
|`fields`|`object`|**必需：** 客户端可以用来搜索的字段的预设值。|
|`icon`|`string`|可选的表示协议的内容URI。覆盖在更高级别的协议对象中提供的图标。|
|`network_id`|`string`|**必需：** 所有实例中唯一的标识符。|

```json
{
  "field_types": {
    "channel": {
      "placeholder": "#foobar",
      "regexp": "#[^\\s]+"
    },
    "network": {
      "placeholder": "irc.example.org",
      "regexp": "([a-z0-9]+\\.)*[a-z0-9]+"
    },
    "nickname": {
      "placeholder": "username",
      "regexp": "[^\\s#]+"
    }
  },
  "icon": "mxc://example.org/aBcDeFgH",
  "instances": [
    {
      "desc": "Freenode",
      "fields": {
        "network": "freenode"
      },
      "icon": "mxc://example.org/JkLmNoPq",
      "network_id": "freenode"
    }
  ],
  "location_fields": [
    "network",
    "channel"
  ],
  "user_fields": [
    "network",
    "nickname"
  ]
}
```

###### 401响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/thirdparty/user

从Matrix用户ID中检索第三方用户的数组。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

##### 请求

###### 请求参数

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`userid`|`string`|要查找的Matrix用户ID。|

##### 响应

|状态|描述|
|---|---|
|`200`|第三方用户的数组。|
|`401`|主服务器未向应用服务提供凭据。响应体中可以包含可选的错误信息。|
|`403`|主服务器提供的凭据被拒绝。|
|`404`|未找到与给定参数匹配的映射。|

###### 200响应

`User`数组。

|User|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`fields`|`object`|**必需：** 用于标识此第三方位置的信息。|
|`protocol`|`string`|**必需：** 第三方位置所属的协议ID。|
|`userid`|`string`|**必需：** 表示第三方用户的Matrix用户ID。|

```json
[
  {
    "fields": {
      "user": "jim"
    },
    "protocol": "gitter",
    "userid": "@_gitter_jim:matrix.org"
  }
]
```

###### 401响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 404响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/thirdparty/user/{protocol}

主服务器为了检索与第三方网络上的用户相关联的Matrix用户ID而调用此API，给定一组用户参数。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

##### 请求

###### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`protocol`|`string`|**必需：** 协议ID。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`fields`|`{string: string}`|一个或多个自定义字段，传递给应用服务以帮助识别用户。|

##### 响应

|状态|描述|
|---|---|
|`200`|找到的Matrix用户ID。|
|`401`|主服务器未向应用服务提供凭据。响应体中可以包含可选的错误信息。|
|`403`|主服务器提供的凭据被拒绝。|
|`404`|未找到与给定参数匹配的用户。|

###### 200响应

`User`数组。

|User|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`fields`|`object`|**必需：** 用于标识此第三方位置的信息。|
|`protocol`|`string`|**必需：** 第三方位置所属的协议ID。|
|`userid`|`string`|**必需：** 表示第三方用户的Matrix用户ID。|

```json
[
  {
    "fields": {
      "user": "jim"
    },
    "protocol": "gitter",
    "userid": "@_gitter_jim:matrix.org"
  }
]
```

###### 401响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

## 客户端-服务器API扩展

应用服务可以通过向主服务器标识自己为应用服务来使用更强大的客户端-服务器API。

本节中定义的端点必须由主服务器在客户端-服务器API中支持，仅应用服务可访问。

### 身份断言

客户端-服务器API从每个请求中提供的`access_token`推断用户ID。为了避免应用服务需要跟踪每个用户的访问令牌，应用服务应通过提供其`as_token`作为`access_token`以及应用服务希望冒充的用户来识别自己。

输入：

- 应用服务令牌（`as_token`）
- AS命名空间中的用户ID以作为。

注意：

- 这适用于客户端-服务器API的所有方面，除了账户管理。
- `as_token`插入到`access_token`中，通常是客户端令牌所在的位置，例如通过查询字符串或`Authorization`头。这是为了允许应用服务重用客户端SDK。
- `access_token`应通过`Authorization`头提供，以防止令牌意外出现在HTTP请求日志中。

应用服务可以通过在请求上使用`user_id`查询字符串参数来指定要作为的虚拟用户。查询字符串中指定的用户必须由应用服务的`user`命名空间之一覆盖。如果缺少该参数，主服务器将假定应用服务打算作为注册的`sender_localpart`属性所暗示的用户。

示例请求为：

```
GET /_matrix/client/v3/account/whoami?user_id=@_irc_user:example.org
Authorization: Bearer YourApplicationServiceTokenHere
```

### 时间戳调整

**[在`v1.3`中添加]**

应用服务可以更改与事件关联的时间戳，允许应用服务更好地表示事件发送的“真实”时间。虽然这不会影响事件的服务器端排序，但可以允许应用服务更好地表示事件何时会被发送/接收，例如在桥接的情况下，远程网络可能会有轻微的延迟，应用服务希望将正确的时间桥接到消息上。

当以应用服务身份验证请求时，调用者可以附加一个`ts`查询字符串参数来更改结果事件的`origin_server_ts`。尝试将时间戳设置为`origin_server_ts`不接受的任何内容应被服务器拒绝为错误请求。

如果不存在，服务器的行为不变：服务器的本地系统时间将用于提供时间戳，表示“现在”。

`ts`查询字符串参数仅在以下端点上有效：

- `PUT /rooms/{roomId}/send/{eventType}/{txnId}`
- `PUT /rooms/{roomId}/state/{eventType}/{stateKey}`

其他端点，例如`/kick`，不支持`ts`：相反，调用者可以使用`PUT /state`端点来模拟其他API的行为。

> [!warning] 警告：
> 更改事件的时间不会更改事件的服务器端（DAG）排序。事件仍将附加在DAG的末端，就像时间戳设置为“现在”一样。未来的MSC，例如[MSC2716](https://github.com/matrix-org/matrix-spec-proposals/pull/2716)，预计将提供允许DAG顺序操作的功能（用于历史导入和类似行为）。

### 服务器管理员样式权限

主服务器需要给予应用服务对其命名空间的完全控制，包括用户和房间别名。这意味着AS应该能够管理其命名空间中的任何用户和房间别名。无需进行额外的API更改即可将房间别名的控制权授予AS。

创建用户需要API更改以：

- 绕过验证码。
- 拥有“无密码”用户。

这涉及完全绕过注册流程。这是通过在`/register`请求上包括`as_token`，以及使用`m.login.application_service`登录类型来设置所需的用户ID而无需密码来实现的。

```
POST /_matrix/client/v3/register
Authorization: Bearer YourApplicationServiceTokenHere

Content:
{
  type: "m.login.application_service",
  username: "_irc_example"
}
```

同样，登录用户需要API更改，以允许AS在不需要用户密码的情况下登录。这是通过在`/login`请求上包括`as_token`，以及使用`m.login.application_service`登录类型来实现的：

**[在`v1.2`中添加]**

```
POST /_matrix/client/v3/login
Authorization: Bearer YourApplicationServiceTokenHere

Content:
{
  type: "m.login.application_service",
  "identifier": {
    "type": "m.id.user",
    "user": "_irc_example"
  }
}
```

尝试在其定义的命名空间之外创建用户或别名的应用服务，或者尝试登录为其定义的命名空间之外的用户，将收到错误代码`M_EXCLUSIVE`。同样，尝试在应用服务定义的命名空间内创建用户或别名的普通用户也将收到相同的`M_EXCLUSIVE`错误代码，但仅当应用服务将命名空间定义为`exclusive`时。

如果使用`m.login.application_service`登录类型调用`/register`或`/login`，但没有有效的`as_token`，端点将返回错误代码`M_MISSING_TOKEN`或`M_UNKNOWN_TOKEN`以及401作为HTTP状态代码。这与客户端-服务器API中的无效身份验证行为相同（参见[使用访问令牌](https://spec.matrix.org/v1.11/client-server-api/#using-access-tokens)）。

### Pinging

**[在`v1.7`中添加]**

这是客户端-服务器API的伴随端点，用于上面描述的ping机制。

#### POST /_matrix/client/v1/appservice/{appserviceId}/ping

**在`v1.7`中添加**

此API请求主服务器调用应用服务上的`/_matrix/app/v1/ping`端点，以确保主服务器可以与应用服务通信。

此API需要使用应用服务访问令牌（`as_token`）而不是典型的客户端访问令牌。此API不能由未被识别为应用服务的用户调用。此外，路径中的应用服务ID必须与用于身份验证请求的应用服务的`as_token`相同。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

##### 请求

###### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`appserviceId`|`string`|**必需：** 要ping的应用服务的应用服务ID。必须与用于身份验证请求的应用服务的`as_token`相同。|

###### 请求体

|名称|类型|描述|
|---|---|---|
|`transaction_id`|`string`|传递给`/_matrix/app/v1/ping`调用的可选事务ID。|

###### 请求体示例

```json
{
  "transaction_id": "mautrix-go_1683636478256400935_123"
}
```

##### 响应

|状态|描述|
|---|---|
|`200`|ping成功。|
|`400`|应用服务没有配置URL。错误代码为`M_URL_NOT_SET`。|
|`403`|用于身份验证请求的访问令牌不属于应用服务，或属于路径中不同的应用服务。错误代码为`M_FORBIDDEN`。|
|`502`|应用服务返回了错误状态，或连接失败。错误代码为`M_BAD_STATUS`或`M_CONNECTION_FAILED`。对于错误状态，响应可能包括`status`和`body`字段，分别包含HTTP状态代码和响应体文本，以帮助调试。|
|`504`|与应用服务的连接超时。错误代码为`M_CONNECTION_TIMEOUT`。|

###### 200响应

|名称|类型|描述|
|---|---|---|
|`duration_ms`|`integer`|**必需：** 从主服务器的角度来看，`/_matrix/app/v1/ping`请求所花费的时间（毫秒）。|

```json
{
  "duration_ms": 123
}
```

###### 400响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_URL_NOT_SET",
  "error": "Application service doesn't have a URL configured"
}
```

###### 403响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_FORBIDDEN",
  "error": "Provided access token is not the appservice's as_token"
}
```

###### 502响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`body`|`string`|应用服务返回的HTTP响应体。|
|`errcode`|`string`|**必需：** 错误代码。<br><br>之一：[M_BAD_STATUS, M_CONNECTION_FAILED]。|
|`error`|`string`|人类可读的错误信息。|
|`status`|`integer`|应用服务返回的HTTP状态代码。|

```json
{
  "body": "{\"errcode\": \"M_UNKNOWN_TOKEN\"}",
  "errcode": "M_BAD_STATUS",
  "error": "Ping returned status 401",
  "status": 401
}
```

###### 504响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_CONNECTION_TIMEOUT",
  "error": "Connection to application service timed out"
}
```

### 使用`/sync`和`/events`

希望使用客户端-服务器API的`/sync`或`/events`的应用服务必须使用虚拟用户（通过查询字符串提供`user_id`）来进行。预计应用服务使用推送给它的事务来处理事件，而不是与`sender_localpart`暗示的用户同步。

### 应用服务房间目录

应用服务可以为其定义的第三方协议维护自己的房间目录。这些房间目录可以通过在`/publicRooms`客户端-服务器端点上的附加参数由客户端访问。

#### PUT /_matrix/client/v3/directory/list/appservice/{networkId}/{roomId}

更新应用服务房间目录中给定房间的可见性。

此API类似于客户端用于更新主服务器更一般房间目录的房间目录可见性API。

此API需要使用应用服务访问令牌（`as_token`）而不是典型的客户端访问令牌。此API不能由未被识别为应用服务的用户调用。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

##### 请求

###### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`networkId`|`string`|**必需：** 要更新房间列表的协议（网络）ID。此ID由应用服务提供，列为支持的协议。|
|`roomId`|`string`|**必需：** 要添加到目录的房间ID。|

###### 请求体

|名称|类型|描述|
|---|---|---|
|`visibility`|`string`|**必需：** 房间在目录中是否可见（公共）或不可见（私有）。<br><br>之一：[public, private]。|

###### 请求体示例

```json
{
  "visibility": "public"
}
```

##### 响应

|状态|描述|
|---|---|
|`200`|房间的目录可见性已更新。|

###### 200响应

```json
{}
```

## 从第三方网络引用消息

应用服务应在其发出的事件内容中包含一个`external_url`，以指示消息的来源。这通常适用于将其他网络桥接到Matrix的应用服务，例如IRC，其中可能有一个HTTP URL可供参考。

客户端应为用户提供一种访问`external_url`的方法（如果存在）。客户端还应确保URL具有`https`或`http`方案，然后再使用它。

事件上存在`external_url`并不一定意味着事件是由应用服务发送的。客户端应谨慎对待其中包含的URL，因为它可能不是事件来源的合法引用。