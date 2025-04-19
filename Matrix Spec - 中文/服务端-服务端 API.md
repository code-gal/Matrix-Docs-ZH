# 服务器-服务器 API

Matrix 主服务器使用联邦 API（也称为服务器-服务器 API）相互通信。主服务器使用这些 API 实时推送消息给彼此，从彼此检索历史消息，并查询关于用户的个人资料和在线状态信息。

这些 API 是通过服务器之间的 HTTPS 请求实现的。这些 HTTPS 请求在 TLS 传输层使用公钥签名进行强身份验证，并在 HTTP 层使用 HTTP Authorization 头中的公钥签名进行身份验证。

主服务器之间的通信主要有三种类型：

持久数据单元（PDUs）：这些事件从一个主服务器广播到任何加入同一房间（由房间 ID 标识）的其他主服务器。它们被持久化存储，并记录房间的消息和状态历史。

类似于电子邮件，PDU 的发起服务器负责将该事件传递给其接收服务器。然而，PDUs 使用发起服务器的私钥签名，因此可以通过第三方服务器传递它们。

临时数据单元（EDUs）：这些事件在主服务器对之间推送。它们不被持久化，也不是房间历史的一部分，接收主服务器也不需要回复它们。

查询：这些是由一方发送 HTTPS GET 请求以获取某些信息并由另一方响应的单次请求/响应交互。它们不被持久化，也不包含长期重要的历史记录。它们只是请求查询时刻的快照状态。

EDUs 和 PDUs 进一步封装在称为事务的信封中，使用 HTTPS PUT 请求从源主服务器传输到目标主服务器。

# API 标准[](https://spec.matrix.org/v1.11/server-server-api/#api-standards)

Matrix 中服务器-服务器通信的强制基线是通过 HTTPS API 交换 JSON 对象。未来可能会指定更高效的传输方式作为可选扩展。

所有 `POST` 和 `PUT` 端点要求请求服务器提供包含（可能为空的）JSON 对象的请求体。请求服务器应为所有带有 JSON 主体的请求提供 `Content-Type` 头为 `application/json`，但这不是必需的。

同样，本规范中的所有端点要求目标服务器返回 JSON 对象。服务器必须为所有 JSON 响应包含 `Content-Type` 头为 `application/json`。

所有 JSON 数据，无论是请求还是响应，必须使用 UTF-8 编码。

## TLS[](https://spec.matrix.org/v1.11/server-server-api/#tls)

服务器-服务器通信必须通过 HTTPS 进行。

目标服务器必须提供由已知证书颁发机构签署的 TLS 证书。

请求服务器最终负责确定受信任的证书颁发机构，但强烈建议依赖操作系统的判断。服务器可以为管理员提供一种覆盖受信任机构列表的方法。服务器还可以为给定的域名或网络掩码白名单跳过证书验证，以便进行测试或在其他地方进行验证的网络中，例如 `.onion` 地址。

服务器在可能的情况下应尊重 SNI：应为预期的证书发送 SNI，除非该证书预期为 IP 地址，在这种情况下不支持 SNI，不应发送。

鼓励服务器使用 [Certificate Transparency](https://www.certificate-transparency.org/) 项目。

## 不支持的端点[](https://spec.matrix.org/v1.11/server-server-api/#unsupported-endpoints)

如果收到对不支持（或未知）端点的请求，则服务器必须响应 404 `M_UNRECOGNIZED` 错误。

同样，405 `M_UNRECOGNIZED` 错误用于表示对已知端点的不支持方法。

# 服务器发现[](https://spec.matrix.org/v1.11/server-server-api/#server-discovery)

## 解析服务器名称[](https://spec.matrix.org/v1.11/server-server-api/#resolving-server-names)

每个 Matrix 主服务器由一个由主机名和可选端口组成的服务器名称标识，如[语法](https://spec.matrix.org/v1.11/appendices#server-name)所述。在适用的情况下，委托的服务器名称使用相同的语法。

服务器名称解析为要连接的 IP 地址和端口，并具有影响要发送的证书和 `Host` 头的各种条件。整体过程如下：

1. 如果主机名是 IP 字面量，则应使用该 IP 地址以及给定的端口号，如果未给出端口，则为 8448。目标服务器必须提供 IP 地址的有效证书。请求中的 `Host` 头应设置为服务器名称，包括端口（如果服务器名称包含一个）。
    
2. 如果主机名不是 IP 字面量，并且服务器名称包含显式端口，则使用 CNAME、AAAA 或 A 记录解析主机名为 IP 地址。请求发送到解析的 IP 地址和给定端口，`Host` 头为原始服务器名称（带端口）。目标服务器必须提供主机名的有效证书。
    
3. 如果主机名不是 IP 字面量，则向 `https://<hostname>/.well-known/matrix/server` 发出常规 HTTPS 请求，期望使用本节稍后定义的架构。应遵循 30x 重定向，但应避免重定向循环。请求服务器应缓存对 `/.well-known` 端点的响应（无论成功与否）。服务器应尊重响应中存在的缓存控制头，或者在头不存在时使用合理的默认值。推荐的合理默认值是 24 小时。服务器还应对响应施加最大缓存时间：建议为 48 小时。建议将错误缓存长达一小时，并鼓励服务器对重复失败进行指数退避。`/.well-known` 请求的架构在本节稍后。如果响应无效（错误的 JSON、缺少属性、非 200 响应等），跳到步骤 4。如果响应有效，解析 `m.server` 属性为 `<delegated_hostname>[:<delegated_port>]` 并按以下方式处理：
    
    1. 如果 `<delegated_hostname>` 是 IP 字面量，则应使用该 IP 地址以及 `<delegated_port>` 或 8448（如果未提供端口）。目标服务器必须提供 IP 地址的有效 TLS 证书。请求必须使用包含 IP 地址的 `Host` 头发送，包括端口（如果提供了一个）。
    2. 如果 `<delegated_hostname>` 不是 IP 字面量，并且存在 `<delegated_port>`，则通过查找 `<delegated_hostname>` 的 CNAME、AAAA 或 A 记录来发现 IP 地址。使用结果 IP 地址和 `<delegated_port>`。请求必须使用 `<delegated_hostname>:<delegated_port>` 的 `Host` 头发送。目标服务器必须提供 `<delegated_hostname>` 的有效证书。
    3. **[在 `v1.8` 中添加]** 如果 `<delegated_hostname>` 不是 IP 字面量且没有 `<delegated_port>`，则为 `_matrix-fed._tcp.<delegated_hostname>` 查找 SRV 记录。这可能导致另一个主机名（使用 AAAA 或 A 记录解析）和端口。请求应发送到解析的 IP 地址和端口，`Host` 头包含 `<delegated_hostname>`。目标服务器必须提供 `<delegated_hostname>` 的有效证书。
    4. **[已弃用]** 如果 `<delegated_hostname>` 不是 IP 字面量，没有 `<delegated_port>`，并且未找到 `_matrix-fed._tcp.<delegated_hostname>` SRV 记录，则为 `_matrix._tcp.<delegated_hostname>` 查找 SRV 记录。这可能导致另一个主机名（使用 AAAA 或 A 记录解析）和端口。请求应发送到解析的 IP 地址和端口，`Host` 头包含 `<delegated_hostname>`。目标服务器必须提供 `<delegated_hostname>` 的有效证书。
    5. 如果未找到 SRV 记录，则使用 CNAME、AAAA 或 A 记录解析 IP 地址。然后请求发送到解析的 IP 地址和 8448 端口，使用 `<delegated_hostname>` 的 `Host` 头。目标服务器必须提供 `<delegated_hostname>` 的有效证书。
4. **[在 `v1.8` 中添加]** 如果 `/.well-known` 请求导致错误响应，则通过解析 `_matrix-fed._tcp.<hostname>` 的 SRV 记录找到服务器。这可能导致一个主机名（使用 AAAA 或 A 记录解析）和端口。请求发送到解析的 IP 地址和端口，`Host` 头包含 `<hostname>`。目标服务器必须提供 `<hostname>` 的有效证书。
    
5. **[已弃用]** 如果 `/.well-known` 请求导致错误响应，并且未找到 `_matrix-fed._tcp.<hostname>` SRV 记录，则通过解析 `_matrix._tcp.<hostname>` 的 SRV 记录找到服务器。这可能导致一个主机名（使用 AAAA 或 A 记录解析）和端口。请求发送到解析的 IP 地址和端口，`Host` 头包含 `<hostname>`。目标服务器必须提供 `<hostname>` 的有效证书。
    
6. 如果 `/.well-known` 请求返回错误响应，并且未找到 SRV 记录，则使用 CNAME、AAAA 和 A 记录解析 IP 地址。请求发送到解析的 IP 地址，使用 8448 端口和包含 `<hostname>` 的 `Host` 头。目标服务器必须提供 `<hostname>` 的有效证书。
    
> [!info] 信息:
> 我们要求 `<hostname>` 而不是 `<delegated_hostname>` 用于 SRV 委派的原因是：
> 
> 1. DNS 是不安全的（并非所有域都有 DNSSEC），因此委派的目标必须通过 TLS 证明它是 `<hostname>` 的有效委托。
> 2. 与 [RFC6125](https://datatracker.ietf.org/doc/html/rfc6125#section-6.2.1) 和其他使用 SRV 记录的应用程序（如 [XMPP](https://datatracker.ietf.org/doc/html/rfc6120#section-13.7.2.1)）的建议一致。

> [!info] 信息:
> 请注意，SRV 记录的目标可能 _不能_ 是 CNAME，如 [RFC2782](https://www.rfc-editor.org/rfc/rfc2782.html) 所规定：
> 
> > 名称不得是别名（根据 RFC 1034 或 RFC 2181 的定义）

> [!info] 信息:
> 步骤 3.4 和 5 已弃用，因为它们使用了 IANA 未注册的服务名称。它们可能会在未来版本的规范中被移除。服务器管理员被鼓励使用 `.well-known` 而不是任何形式的 SRV 记录。
> 
> IANA 对 8448 端口和 `matrix-fed` 的注册可以在[这里](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=matrix-fed)找到。

### GET /.well-known/matrix/server[](https://spec.matrix.org/v1.11/server-server-api/#getwell-knownmatrixserver)

---

获取 Matrix 主服务器之间服务器-服务器通信的委托服务器信息。服务器应遵循 30x 重定向，谨慎避免重定向循环，并使用正常的 X.509 证书验证。

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
|`200`|委托服务器信息。此响应的 `Content-Type` 应为 `application/json`，但解析响应的服务器应假定主体是 JSON，无论类型如何。解析 JSON 失败或结果解析 JSON 中提供的数据无效不应导致发现失败 - 请参阅服务器发现过程以获取继续的信息。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`m.server`|`string`|用于委托服务器-服务器通信的服务器名称，带有可选端口。委托服务器名称使用与[附录中的服务器名称](https://spec.matrix.org/v1.11/appendices/#server-name)相同的语法。|

```json
{
  "m.server": "delegated.example.com:1234"
}
```

## 服务器实现[](https://spec.matrix.org/v1.11/server-server-api/#server-implementation)

### GET /_matrix/federation/v1/version[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1version)

---

获取此主服务器的实现名称和版本。

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
|`200`|此主服务器的实现名称和版本。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`server`|[Server](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1version_response-200_server)||

|服务器|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`name`|`string`|标识此实现的任意名称。|
|`version`|`string`|此实现的版本。版本格式取决于实现。|

```json
{
  "server": {
    "name": "My_Homeserver_Implementation",
    "version": "ArbitraryVersionNumber"
  }
}
```

## 检索服务器密钥[](https://spec.matrix.org/v1.11/server-server-api/#retrieving-server-keys)

> [!info] 信息:
> 曾经有一个“版本 1”的密钥交换。由于缺乏意义，它已从规范中移除。可以从[历史草案](https://github.com/matrix-org/matrix-doc/blob/51faf8ed2e4a63d4cfd6d23183698ed169956cc0/specification/server_server_api.rst#232version-1)中查看。

每个主服务器在 `/_matrix/key/v2/server` 下发布其公钥。主服务器通过直接获取 `/_matrix/key/v2/server` 或通过使用 `/_matrix/key/v2/query/{serverName}` API 查询中间公证服务器来查询密钥。中间公证服务器代表另一个服务器查询 `/_matrix/key/v2/server` API，并用自己的密钥签署响应。服务器可以查询多个公证服务器以确保它们都报告相同的公钥。

这种方法借鉴了 [Perspectives Project](https://web.archive.org/web/20170702024706/https://perspectives-project.org/) 的方法，但进行了修改以包含 NACL 密钥并使用 JSON 而不是 XML。它的优点是避免了单一信任根，因为每个服务器可以自由选择信任哪些公证服务器，并可以通过查询其他服务器来证实给定公证服务器返回的密钥。

### 发布密钥[](https://spec.matrix.org/v1.11/server-server-api/#publishing-keys)

主服务器在 `/_matrix/key/v2/server` 的 JSON 对象中发布其签名密钥。响应包含一个 `verify_keys` 列表，这些密钥对主服务器进行的联邦请求和事件签名有效。它包含一个 `old_verify_keys` 列表，这些密钥仅对事件签名有效。

#### GET /_matrix/key/v2/server[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2server)

---

获取主服务器发布的签名密钥。主服务器可以有任意数量的活动密钥，并且可能有多个旧密钥。

中间公证服务器应缓存响应一半的生命周期，以避免提供过时的响应。发起服务器应避免返回有效期少于一小时的响应，以避免重复请求即将过期的证书。请求服务器应限制查询证书的频率，以避免向服务器发送大量请求。

如果服务器未能响应此请求，中间公证服务器应继续返回其从服务器接收到的最后一个响应，以便仍然可以检查旧事件的签名。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

##### 请求

无请求参数或请求体。

---

##### 响应

|状态|描述|
|---|---|
|`200`|主服务器的密钥|

###### 200 响应

|服务器密钥|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`old_verify_keys`|{string: [Old Verify Key](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2server_response-200_old-verify-key)}|服务器曾经使用的公钥以及何时停止使用它们。<br><br>对象的键是算法和版本的组合（`ed25519` 是算法，`0ldK3y` 是示例中的版本）。结合在一起，这形成了密钥 ID。版本必须具有与正则表达式 `[a-zA-Z0-9_]` 匹配的字符。|
|`server_name`|`string`|**必需：** 主服务器的 DNS 名称。|
|`signatures`|`{string: {string: string}}`|使用 `verify_keys` 签署的此对象的数字签名。<br><br>签名是使用[签署 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算的。|
|`valid_until_ts`|`integer`|以毫秒为单位的 POSIX 时间戳，表示应刷新有效密钥列表的时间。在房间版本 1、2、3 和 4 中必须忽略此字段。根据[房间版本规范](https://spec.matrix.org/v1.11/rooms)，在此时间戳之后使用的密钥必须被视为无效。<br><br>服务器在确定密钥是否有效时必须使用此字段和 7 天后的较小值。这是为了避免攻击者发布一个有效时间较长且主服务器所有者无法撤销的密钥。|
|`verify_keys`|{string: [Verify Key](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2server_response-200_verify-key)}|**必需：**<br><br>主服务器用于验证数字签名的公钥。<br><br>对象的键是算法和版本的组合（`ed25519` 是算法，`abc123` 是示例中的版本）。结合在一起，这形成了密钥 ID。版本必须具有与正则表达式 `[a-zA-Z0-9_]` 匹配的字符。|

|旧验证密钥|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`expired_ts`|`integer`|**必需：** 此密钥过期的 POSIX 时间戳（毫秒）。|
|`key`|`string`|**必需：** [无填充 base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) 编码的密钥。|

|验证密钥|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`key`|`string`|**必需：** [无填充 base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) 编码的密钥。|

```json
{
  "old_verify_keys": {
    "ed25519:0ldk3y": {
      "expired_ts": 1532645052628,
      "key": "VGhpcyBzaG91bGQgYmUgeW91ciBvbGQga2V5J3MgZWQyNTUxOSBwYXlsb2FkLg"
    }
  },
  "server_name": "example.org",
  "signatures": {
    "example.org": {
      "ed25519:auto2": "VGhpcyBzaG91bGQgYWN0dWFsbHkgYmUgYSBzaWduYXR1cmU"
    }
  },
  "valid_until_ts": 1652262000000,
  "verify_keys": {
    "ed25519:abc123": {
      "key": "VGhpcyBzaG91bGQgYmUgYSByZWFsIGVkMjU1MTkgcGF5bG9hZA"
    }
  }
}
```

### 通过其他服务器查询密钥[](https://spec.matrix.org/v1.11/server-server-api/#querying-keys-through-another-server)

服务器可以通过公证服务器查询其他服务器的密钥。公证服务器可以是另一个主服务器。公证服务器将通过使用 `/_matrix/key/v2/server` API 从查询的服务器检索密钥。公证服务器还将在返回结果之前对查询服务器的响应进行签名。

公证服务器可以通过使用缓存的响应返回离线或在提供其自身密钥时遇到问题的服务器的密钥。可以从多个服务器查询密钥以减轻 DNS 欺骗的影响。

#### POST /_matrix/key/v2/query[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query)

---

以批量格式查询来自多个服务器的密钥。接收（公证）服务器必须对查询服务器返回的密钥进行签名。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

##### 请求

###### 请求体

|名称|类型|描述|
|---|---|---|
|`server_keys`|{string: {string: [Query Criteria](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query_request_query-criteria)}}|**必需：**<br><br>查询标准。对象的外部 `string` 键是服务器名称（例如：`matrix.org`）。内部 `string` 键是要查询特定服务器的密钥 ID。如果未给出要查询的密钥 ID，公证服务器应查询所有密钥。如果未给出服务器，公证服务器必须在响应中返回一个空的 `server_keys` 数组。<br><br>公证服务器可以返回多个密钥，无论给定的密钥 ID 如何。|

|查询标准|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`minimum_valid_until_ts`|`integer`|以毫秒为单位的 POSIX 时间戳，表示返回的证书需要有效的时间，以便对请求服务器有用。<br><br>如果未提供，则使用公证服务器确定的当前时间。|

###### 请求体示例

```json
{
  "server_keys": {
    "example.org": {
      "ed25519:abc123": {
        "minimum_valid_until_ts": 1234567890
      }
    }
  }
}
```

---

##### 响应

|状态|描述|
|---|---|
|`200`|由公证服务器签署的查询服务器的密钥。离线且没有缓存密钥的服务器将不包含在结果中。这可能导致一个空数组。|

###### 200 响应

|名称|类型|描述|
|---|---|---|
|`server_keys`|[[Server Keys](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query_response-200_server-keys)]|由公证服务器签署的查询服务器的密钥。|

|服务器密钥|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`old_verify_keys`|{string: [Old Verify Key](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query_response-200_old-verify-key)}|服务器曾经使用的公钥以及何时停止使用它们。<br><br>对象的键是算法和版本的组合（`ed25519` 是算法，`0ldK3y` 是示例中的版本）。结合在一起，这形成了密钥 ID。版本必须具有与正则表达式 `[a-zA-Z0-9_]` 匹配的字符。|
|`server_name`|`string`|**必需：** 主服务器的 DNS 名称。|
|`signatures`|`{string: {string: string}}`|使用 `verify_keys` 签署的此对象的数字签名。<br><br>签名是使用[签署 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算的。|
|`valid_until_ts`|`integer`|以毫秒为单位的 POSIX 时间戳，表示应刷新有效密钥列表的时间。在房间版本 1、2、3 和 4 中必须忽略此字段。根据[房间版本规范](https://spec.matrix.org/v1.11/rooms)，在此时间戳之后使用的密钥必须被视为无效。<br><br>服务器在确定密钥是否有效时必须使用此字段和 7 天后的较小值。这是为了避免攻击者发布一个有效时间较长且主服务器所有者无法撤销的密钥。|
|`verify_keys`|{string: [Verify Key](https://spec.matrix.org/v1.11/server-server-api/#post_matrixkeyv2query_response-200_verify-key)}|**必需：**<br><br>主服务器用于验证数字签名的公钥。<br><br>对象的键是算法和版本的组合（`ed25519` 是算法，`abc123` 是示例中的版本）。结合在一起，这形成了密钥 ID。版本必须具有与正则表达式 `[a-zA-Z0-9_]` 匹配的字符。|

|旧验证密钥|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`expired_ts`|`integer`|**必需：** 此密钥过期的 POSIX 时间戳（毫秒）。|
|`key`|`string`|**必需：** [无填充 base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) 编码的密钥。|

|验证密钥|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`key`|`string`|**必需：** [无填充 base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) 编码的密钥。|

```json
{
  "server_keys": [
    {
      "old_verify_keys": {
        "ed25519:0ldk3y": {
          "expired_ts": 1532645052628,
          "key": "VGhpcyBzaG91bGQgYmUgeW91ciBvbGQga2V5J3MgZWQyNTUxOSBwYXlsb2FkLg"
        }
      },
      "server_name": "example.org",
      "signatures": {
        "example.org": {
          "ed25519:abc123": "VGhpcyBzaG91bGQgYWN0dWFsbHkgYmUgYSBzaWduYXR1cmU"
        },
        "notary.server.com": {
          "ed25519:010203": "VGhpcyBpcyBhbm90aGVyIHNpZ25hdHVyZQ"
        }
      },
      "valid_until_ts": 1652262000000,
      "verify_keys": {
        "ed25519:abc123": {
          "key": "VGhpcyBzaG91bGQgYmUgYSByZWFsIGVkMjU1MTkgcGF5bG9hZA"
        }
      }
    }
  ]
}
```

#### GET /_matrix/key/v2/query/{serverName}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2queryservername)

---

查询另一个服务器的密钥。接收（公证）服务器必须对查询服务器返回的密钥进行签名。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

##### 请求

###### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`serverName`|`string`|**必需：** 要查询的服务器的 DNS 名称|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`minimum_valid_until_ts`|`integer`|以毫秒为单位的 POSIX 时间戳，表示返回的证书需要有效的时间，以便对请求服务器有用。<br><br>如果未提供，则使用公证服务器确定的当前时间。|

---

##### 响应

|状态|描述|
|---|---|
|`200`|服务器的密钥，如果服务器无法访问且没有可用的缓存密钥，则为空数组。|

###### 200 响应

|名称|类型|描述|
|---|---|---|
|`server_keys`|[[Server Keys](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2queryservername_response-200_server-keys)]|由公证服务器签署的查询服务器的密钥。|

|服务器密钥|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`old_verify_keys`|{string: [Old Verify Key](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2queryservername_response-200_old-verify-key)}|服务器曾经使用的公钥以及何时停止使用它们。<br><br>对象的键是算法和版本的组合（`ed25519` 是算法，`0ldK3y` 是示例中的版本）。结合在一起，这形成了密钥 ID。版本必须具有与正则表达式 `[a-zA-Z0-9_]` 匹配的字符。|
|`server_name`|`string`|**必需：** 主服务器的 DNS 名称。|
|`signatures`|`{string: {string: string}}`|使用 `verify_keys` 签署的此对象的数字签名。<br><br>签名是使用[签署 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算的。|
|`valid_until_ts`|`integer`|以毫秒为单位的 POSIX 时间戳，表示应刷新有效密钥列表的时间。在房间版本 1、2、3 和 4 中必须忽略此字段。根据[房间版本规范](https://spec.matrix.org/v1.11/rooms)，在此时间戳之后使用的密钥必须被视为无效。<br><br>服务器在确定密钥是否有效时必须使用此字段和 7 天后的较小值。这是为了避免攻击者发布一个有效时间较长且主服务器所有者无法撤销的密钥。|
|`verify_keys`|{string: [Verify Key](https://spec.matrix.org/v1.11/server-server-api/#get_matrixkeyv2queryservername_response-200_verify-key)}|**必需：**<br><br>主服务器用于验证数字签名的公钥。<br><br>对象的键是算法和版本的组合（`ed25519` 是算法，`abc123` 是示例中的版本）。结合在一起，这形成了密钥 ID。版本必须具有与正则表达式 `[a-zA-Z0-9_]` 匹配的字符。|

|旧验证密钥|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`expired_ts`|`integer`|**必需：** 此密钥过期的 POSIX 时间戳（毫秒）。|
|`key`|`string`|**必需：** [无填充 base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) 编码的密钥。|

|验证密钥|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`key`|`string`|**必需：** [无填充 base64](https://spec.matrix.org/v1.11/appendices/#unpadded-base64) 编码的密钥。|

```json
{
  "server_keys": [
    {
      "old_verify_keys": {
        "ed25519:0ldk3y": {
          "expired_ts": 1532645052628,
          "key": "VGhpcyBzaG91bGQgYmUgeW91ciBvbGQga2V5J3MgZWQyNTUxOSBwYXlsb2FkLg"
        }
      },
      "server_name": "example.org",
      "signatures": {
        "example.org": {
          "ed25519:abc123": "VGhpcyBzaG91bGQgYWN0dWFsbHkgYmUgYSBzaWduYXR1cmU"
        },
        "notary.server.com": {
          "ed25519:010203": "VGhpcyBpcyBhbm90aGVyIHNpZ25hdHVyZQ"
        }
      },
      "valid_until_ts": 1652262000000,
      "verify_keys": {
        "ed25519:abc123": {
          "key": "VGhpcyBzaG91bGQgYmUgYSByZWFsIGVkMjU1MTkgcGF5bG9hZA"
        }
      }
    }
  ]
}
```

# 认证[](https://spec.matrix.org/v1.11/server-server-api/#authentication)

## 请求认证[](https://spec.matrix.org/v1.11/server-server-api/#request-authentication)

每个由家庭服务器发出的HTTP请求都使用公钥数字签名进行认证。请求方法、目标和主体通过将它们包装在一个JSON对象中并使用JSON签名算法进行签名来进行签名。生成的签名作为Authorization头添加，认证方案为`X-Matrix`。注意，目标字段应包括以`/_matrix/...`开头的完整路径，包括`?`和任何查询参数（如果存在），但不应包括前导的`https:`，也不应包括目标服务器的主机名。

步骤1 签署JSON：

```json
{
    "method": "POST",
    "uri": "/target",
    "origin": "origin.hs.example.com",
    "destination": "destination.hs.example.com",
    "content": <JSON-parsed request body>,
    "signatures": {
        "origin.hs.example.com": {
            "ed25519:key1": "ABCDEF..."
        }
    }
}
```

上面JSON中的服务器名称是每个参与的家庭服务器的服务器名称。来自[服务器名称解析部分](https://spec.matrix.org/v1.11/server-server-api/#resolving-server-names)的委托不影响这些——使用的是委托之前的服务器名称。这一条件在整个请求签名过程中适用。

步骤2 添加Authorization头：

```
POST /target HTTP/1.1
Authorization: X-Matrix origin="origin.hs.example.com",destination="destination.hs.example.com",key="ed25519:key1",sig="ABCDEF..."
Content-Type: application/json

<JSON-encoded request body>
```

Python代码示例：

```python
def authorization_headers(origin_name, origin_signing_key,
                          destination_name, request_method, request_target,
                          content=None):
    request_json = {
         "method": request_method,
         "uri": request_target,
         "origin": origin_name,
         "destination": destination_name,
    }

    if content is not None:
        # 假设内容已经解析为JSON
        request_json["content"] = content

    signed_json = sign_json(request_json, origin_name, origin_signing_key)

    authorization_headers = []

    for key, sig in signed_json["signatures"][origin_name].items():
        authorization_headers.append(bytes(
            "X-Matrix origin=\"%s\",destination=\"%s\",key=\"%s\",sig=\"%s\"" % (
                origin_name, destination_name, key, sig,
            )
        ))

    return ("Authorization", authorization_headers[0])

```

Authorization头的格式在[RFC 9110的第11.4节](https://datatracker.ietf.org/doc/html/rfc9110#section-11.4)中给出。总之，头以认证方案`X-Matrix`开头，后跟一个或多个空格，后跟一个用逗号分隔的参数列表，参数以name=value对的形式书写。每个逗号周围允许有零个或多个空格和制表符。名称不区分大小写，顺序无关紧要。如果值包含在`token`中不允许的字符，则必须用引号括起来，如[RFC 9110的第5.6.2节](https://datatracker.ietf.org/doc/html/rfc9110#section-5.6.2)所定义；如果值是有效的`token`，则可以选择是否用引号括起来。引用的值可以包含反斜杠转义字符。在解析头时，接收方必须取消转义字符。即，反斜杠字符对被替换为反斜杠后面的字符。

为了与旧服务器兼容，发送方应：

- 在`X-Matrix`后仅包含一个空格，
- 仅使用小写名称，
- 避免在参数值中使用反斜杠，
- 避免在name=value对之间的逗号周围包含空格。

为了与旧服务器兼容，接收方应允许在值中包含冒号，而不要求值用引号括起来。

要包含的认证参数有：

- `origin`：发送服务器的服务器名称。这与步骤1中描述的JSON中的`origin`字段相同。
- `destination`：**[在`v1.3`中添加]** 接收服务器的服务器名称。这与步骤1中描述的JSON中的`destination`字段相同。为了与旧服务器兼容，接收方应接受没有此参数的请求，但必须始终发送它。如果包含此属性，但值与接收服务器的名称不匹配，接收服务器必须以HTTP状态码401 Unauthorized拒绝请求。
- `key`：用于签署请求的发送服务器密钥的ID，包括算法名称。
- `signature`：步骤1中计算的JSON签名。

未知参数将被忽略。

> [!info] 信息：
> **[在`v1.11`中更改]** 本节以前引用了[RFC 7235](https://datatracker.ietf.org/doc/html/rfc7235#section-2.1)和[RFC 7230](https://datatracker.ietf.org/doc/html/rfc9110#section-5.6.2)，这些已被RFC 9110取代，但对这里涉及的部分没有更改。

## 响应认证[](https://spec.matrix.org/v1.11/server-server-api/#response-authentication)

响应通过TLS服务器证书进行认证。家庭服务器在认证连接的服务器之前不应发送请求，以避免信息泄露给窃听者。

## 客户端TLS证书[](https://spec.matrix.org/v1.11/server-server-api/#client-tls-certificates)

请求在HTTP层而不是TLS层进行认证，因为像Matrix这样的HTTP服务通常部署在处理TLS的负载均衡器后面，这些负载均衡器使得检查TLS客户端证书变得困难。

家庭服务器可以提供TLS客户端证书，接收家庭服务器可以检查客户端证书是否与源家庭服务器的证书匹配。

# 事务[](https://spec.matrix.org/v1.11/server-server-api/#transactions)

家庭服务器之间的EDU和PDU的传输是通过事务消息的交换进行的，这些消息被编码为JSON对象，通过HTTP PUT请求传递。事务仅对交换它们的家庭服务器对有意义；它们不是全局有意义的。

事务的大小有限制；最多可以包含50个PDU和100个EDU。

### PUT /_matrix/federation/v1/send/{txnId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1sendtxnid)

---

将表示实时活动的消息推送到另一个服务器。目标名称将设置为接收服务器本身的名称。事务体中的每个嵌入的PDU将被处理。

发送服务器必须等待并重试以获得200 OK响应，然后再向接收服务器发送具有不同`txnId`的事务。

注意，事件的格式根据房间版本而不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`txnId`|`string`|**必需：** 事务ID。|

##### 请求体

|事务|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`edus`|[[临时数据单元](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1sendtxnid_request_ephemeral-data-unit)]|临时消息列表。如果没有要发送的临时消息，可以省略。不得包含超过100个EDU。|
|`origin`|`string`|**必需：** 发送此事务的家庭服务器的`server_name`。|
|`origin_server_ts`|`integer`|**必需：** 事务开始时在源家庭服务器上的POSIX时间戳（毫秒）。|
|`pdus`|`[PDU]`|**必需：** 房间的持久更新列表。不得包含超过50个PDU。注意，事件的格式根据房间版本而不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|

|临时数据单元|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|`object`|**必需：** 临时消息的内容。|
|`edu_type`|`string`|**必需：** 临时消息的类型。|

##### 请求体示例

```json
{
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "pdus": [
    {
      "content": {
        "see_room_version_spec": "事件格式根据房间版本而变化。"
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|处理事务的结果。即使一个或多个PDU处理失败，服务器也应使用此响应。|

##### 200响应

|PDU处理结果|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`pdus`|{[事件ID](https://spec.matrix.org/v1.11/appendices#event-ids): [PDU处理结果](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1sendtxnid_response-200_pdu-processing-result)}|**必需：** 原始事务中的PDU。字符串键表示处理的PDU（事件）的ID。|

|PDU处理结果|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`error`|`string`|处理此PDU时出错的人类可读描述。如果没有错误，则可以认为PDU已成功处理。|

```json
{
  "pdus": {
    "$failed_event:example.org": {
      "error": "您不被允许向此房间发送消息。"
    },
    "$successful_event:example.org": {}
  }
}
```

# PDUs[](https://spec.matrix.org/v1.11/server-server-api/#pdus)

每个PDU包含一个房间事件，源服务器希望将其发送到目标。

PDU的`prev_events`字段标识事件的“父级”，从而通过将它们链接到有向无环图（DAG）中来在房间内的事件上建立部分排序。发送服务器应填充此字段，包含房间中尚未看到子事件的所有事件——从而证明该事件在所有其他已知事件之后。

例如，考虑一个房间，其事件形成如下所示的DAG。服务器在此房间中创建新事件时，应将新事件的`prev_events`字段填充为`E4`和`E6`，因为这两个事件尚未有子事件：

```
E1
^
|
E2 <--- E5
^       ^

|       |
E3      E6
^

|
E4
```

有关PDU外观的完整模式，请参见[房间版本规范](https://spec.matrix.org/v1.11/rooms)。

## 接收到PDU时执行的检查[](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu)

每当服务器从远程服务器接收到事件时，接收服务器必须确保事件：

1. 是有效事件，否则将被丢弃。要使事件有效，必须包含`room_id`，并且必须符合该[房间版本](https://spec.matrix.org/v1.11/rooms)的事件格式。
2. 通过签名检查，否则将被丢弃。
3. 通过哈希检查，否则在进一步处理之前将被编辑。
4. 通过基于事件的认证事件的授权规则，否则将被拒绝。
5. 通过基于事件之前状态的授权规则，否则将被拒绝。
6. 通过基于房间当前状态的授权规则，否则将被“软失败”。

这些检查的进一步细节，以及如何处理失败，详述如下。

[签署事件](https://spec.matrix.org/v1.11/server-server-api/#signing-events)部分有更多关于事件上期望的哈希和签名的信息，以及如何计算它们。

### 定义[](https://spec.matrix.org/v1.11/server-server-api/#definitions)

所需权限级别

给定事件类型具有相关的_所需权限级别_。这由当前[`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api/#mroompower_levels)事件给出。事件类型要么在`events`部分中显式列出，要么由`state_default`或`events_default`给出，具体取决于事件是否为状态事件。

邀请级别、踢出级别、禁止级别、编辑级别

当前[`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api/#mroompower_levels)状态中的`invite`、`kick`、`ban`和`redact`属性给出的级别。如果未指定，邀请级别默认为0。踢出级别、禁止级别和编辑级别如果未指定，则各自默认为50。

目标用户

对于[`m.room.member`](https://spec.matrix.org/v1.11/client-server-api/#mroommember)状态事件，由事件的`state_key`给出的用户。

> [!warning] 警告：
> 
> 一些[房间版本](https://spec.matrix.org/v1.11/rooms)接受权限级别值以字符串而不是整数表示。这仅用于向后兼容。家庭服务器应采取合理的预防措施，以防止用户发送带有字符串值的新权限级别事件（例如：通过拒绝API请求），并且绝不应将房间中的默认权限级别填充为字符串值。
> 
> 有关更多信息，请参见[房间版本规范](https://spec.matrix.org/v1.11/rooms)。

### 授权规则[](https://spec.matrix.org/v1.11/server-server-api/#authorization-rules)

事件是否被授权的规则取决于一组状态。给定事件根据不同的状态集多次检查，如上所述。每个房间版本可以有不同的算法来决定规则的工作方式以及应用哪些规则。有关更详细的信息，请参见[房间版本规范](https://spec.matrix.org/v1.11/rooms)。

#### 认证事件选择[](https://spec.matrix.org/v1.11/server-server-api/#auth-events-selection)

PDU的`auth_events`字段标识赋予发送者发送事件权限的事件集。房间中的`m.room.create`事件的`auth_events`为空；对于其他事件，它应为房间状态的以下子集：

- `m.room.create`事件。
    
- 当前的`m.room.power_levels`事件（如果有）。
    
- 发送者当前的`m.room.member`事件（如果有）。
    
- 如果类型为`m.room.member`：
    
    - 目标的当前`m.room.member`事件（如果有）。
    - 如果`membership`为`join`或`invite`，则当前的`m.room.join_rules`事件（如果有）。
    - 如果`membership`为`invite`且`content`包含`third_party_invite`属性，则当前的`m.room.third_party_invite`事件，其`state_key`与`content.third_party_invite.signed.token`匹配（如果有）。
    - 如果`content.join_authorised_via_users_server`存在，并且[房间版本支持受限房间](https://spec.matrix.org/v1.11/rooms/#feature-matrix)，则`m.room.member`事件，其`state_key`与`content.join_authorised_via_users_server`匹配。

### 拒绝[](https://spec.matrix.org/v1.11/server-server-api/#rejection)

如果事件被拒绝，则不应将其转发给客户端，也不应在服务器生成的任何新事件中作为prev事件包含。来自其他服务器的后续事件如果仍然通过授权规则，则应允许引用被拒绝的事件。检查中使用的状态应正常计算，除了不更新为被拒绝的事件（如果它是状态事件）。

如果传入事务中的事件被拒绝，这不应导致事务请求以错误响应进行响应。

> [!info] 信息：
> 这意味着事件可能包含在房间DAG中，即使它们应该被拒绝。

> [!info] 信息：
> 这与编辑事件形成对比，编辑事件仍然可以影响房间的状态。例如，编辑的`join`事件仍将导致用户被视为已加入。

### 软失败[](https://spec.matrix.org/v1.11/server-server-api/#soft-failure)

> [!info] 理由：
> 
> 防止用户通过创建引用DAG旧部分的事件来规避禁令（或其他权限限制）是很重要的。例如，被禁止的用户可以通过让他们的服务器发送引用他们被禁止之前的事件的事件来继续向房间发送消息。注意，这样的事件是完全有效的，我们不能简单地拒绝它们，因为不可能区分这样的事件和延迟的合法事件。因此，我们必须接受这样的事件，并让它们像往常一样参与状态解析和联邦协议。然而，服务器可能选择不将这样的事件发送给他们的客户端，以便最终用户实际上看不到这些事件。
> 
> 当这种情况发生时，服务器通常可以很明显地看到，因为它们可以看到新事件实际上没有通过基于“当前状态”（即所有前向极限的解析状态）的授权。虽然事件在技术上是有效的，但服务器可以选择不通知客户端新事件。
> 
> 这阻止了服务器通过这种方式发送规避禁令等的事件，因为最终用户实际上看不到这些事件。
> 
当家庭服务器通过联邦接收到新事件时，它还应检查事件是否通过基于房间当前状态的授权检查（以及基于事件状态的授权检查）。如果事件未通过基于房间_当前状态_的授权检查（但通过了基于该事件状态的授权检查），则应“软失败”。

当事件“软失败”时，不应将其转发给客户端，也不应被家庭服务器创建的新事件引用（即，它们不应被添加到服务器的房间前向极限列表中）。软失败事件以其他方式正常处理。

> [!info] 信息：
> 如果收到的进一步事件引用软失败事件，软失败事件将正常参与状态解析。状态解析算法的任务是确保恶意事件不能通过这种机制注入到房间状态中。

> [!info] 信息：
> 因为软失败状态事件正常参与状态解析，所以这样的事件可能会出现在房间的当前状态中。在这种情况下，应以通常的方式通知客户端软失败事件（例如，通过在同步响应的`state`部分中发送它）。

> [!info] 信息：
> 软失败事件应在适当的情况下返回给联邦请求（例如，在`/event/<event_id>`中）。注意，软失败事件仅在请求包含引用软失败事件的事件时才在`/backfill`和`/get_missing_events`响应中返回。

示例

例如，考虑事件图：

```
  A
 /
B
```

其中`B`是对用户`X`的禁令。如果用户`X`试图通过发送事件`C`来设置主题以规避禁令：

```
  A
 / \
B   C
```

在`B`之后接收到`C`的服务器应软失败事件`C`，因此不会将`C`转发给其客户端，也不会发送引用`C`的任何事件。

如果稍后另一个服务器发送引用`B`和`C`的事件`D`（如果它在`B`之前接收到`C`，这可能发生）：

```
  A
 / \
B   C
 \ /
  D
```

那么服务器将正常处理`D`。`D`被发送给服务器的客户端（假设`D`通过授权检查）。`D`的状态可能解析为包含`C`的状态，在这种情况下，客户端也应被告知状态已更改为包含`C`。(_注意_：这取决于使用的确切状态解析算法。在算法的原始版本中，`C`将处于解析状态，而在后来的版本中，算法尝试优先考虑禁令而不是主题更改。）

注意，这本质上等同于一个服务器根本没有接收到`C`的情况，因此向另一个服务器请求`C`分支的状态。

让我们回到发送`D`之前的图：

```
  A
 / \
B   C
```

如果房间中的所有服务器在`C`之前看到`B`，因此软失败`C`，那么任何新事件`D'`都不会引用`C`：

```
  A
 / \
B   C
|
D'
```

### 检索事件授权信息[](https://spec.matrix.org/v1.11/server-server-api/#retrieving-event-authorization-information)

家庭服务器可能缺少事件授权信息，或者希望与其他服务器核对以确保接收到正确的授权链。这些API为家庭服务器提供了获取所需信息的途径。

#### GET /_matrix/federation/v1/event_auth/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1event_authroomideventid)

---

检索给定事件的完整授权链。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

##### 请求

###### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`eventId`|`string`|**必需：** 要获取授权链的事件ID。|
|`roomId`|`string`|**必需：** 要获取授权链的房间ID。|

---

##### 响应

|状态|描述|
|---|---|
|`200`|事件的授权链。|

###### 200响应

|名称|类型|描述|
|---|---|---|
|`auth_chain`|`[PDU]`|**必需：** 形成给定事件的授权链的[PDUs](https://spec.matrix.org/v1.11/server-server-api/#pdus)。事件格式根据房间版本而变化——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|

```json
{
  "auth_chain": [
    {
      "content": {
        "see_room_version_spec": "事件格式根据房间版本而变化。"
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

# EDUs[](https://spec.matrix.org/v1.11/server-server-api/#edus)

与PDUs相比，EDUs没有ID、房间ID或“前置”ID列表。它们旨在用于非持久性数据，如用户在线状态、输入通知等。

## `临时数据单元`

---

一个临时数据单元。

|临时数据单元|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|`object`|**必需：** 临时消息的内容。|
|`edu_type`|`string`|**必需：** 临时消息的类型。|

### 示例

```json
{
  "content": {
    "key": "value"
  },
  "edu_type": "m.presence"
}
```

# 房间状态解析[](https://spec.matrix.org/v1.11/server-server-api/#room-state-resolution)

房间的_状态_是`(event_type, state_key)`到`event_id`的映射。每个房间从空状态开始，每个被接受到房间中的状态事件更新该房间的状态。

当每个事件有一个单一的`prev_event`时，很明显每个事件之后房间的状态应该是什么。然而，当事件图中的两个分支合并时，这些分支的状态可能不同，因此必须使用_状态解析_算法来确定结果状态。

例如，考虑以下事件图（最旧的事件E0在顶部）：

```
  E0
  |
  E1
 /  \
E2  E4

|    |
E3   |
 \  /
  E5
```

假设E3和E4都是设置房间名称的`m.room.name`事件。在E5时房间的名称应该是什么？

用于状态解析的算法取决于房间版本。有关每个房间版本算法的描述，请参见[房间版本规范](https://spec.matrix.org/v1.11/rooms)。

# 回填和检索缺失事件[](https://spec.matrix.org/v1.11/server-server-api/#backfilling-and-retrieving-missing-events)

一旦家庭服务器加入房间，它就会接收到该房间中其他家庭服务器发出的所有事件，因此从那一刻起就知道房间的整个历史。由于该房间中的用户可以通过`/messages`客户端API端点请求历史，因此他们可能会向后退到家庭服务器本身成为该房间成员之前的历史。

为了应对这种情况，联邦API提供了一个服务器到服务器的`/messages`客户端API的类似物，允许一个家庭服务器从另一个服务器获取历史。这就是`/backfill` API。

要请求更多历史，请求的家庭服务器选择另一个它认为可能有更多历史的家庭服务器（最有可能的是，这应该是当前历史最早点房间中一些现有用户的家庭服务器），并发出`/backfill`请求。

类似于回填房间的历史，服务器可能没有图中的所有事件。该服务器可以使用`/get_missing_events` API获取其缺失的事件。

### GET /_matrix/federation/v1/backfill/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1backfillroomid)

---

检索给定房间中发生的先前PDU的滑动窗口历史。从`v`参数中给出的PDU ID开始，检索`v`中给出的PDU及其之前的PDU，最多达到`limit`给出的总数。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需：** 要回填的房间ID。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`limit`|`integer`|**必需：** 要检索的PDU的最大数量，包括给定的事件。|
|`v`|`[string]`|**必需：** 要从中回填的事件ID。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|包含给定事件之前的PDU的事务，包括给定事件，最多达到给定的限制。<br><br>**注意：** 尽管PDU定义要求`prev_events`和`auth_events`的数量有限制，但回填的响应不得在这些特定限制上进行验证。<br><br>由于历史原因，可能会出现以前接受的事件现在会被这些限制拒绝的情况。事件应按通常方式由`/send`、`/get_missing_events`和其余端点拒绝。|

##### 200响应

|事务|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`origin`|`string`|**必需：** 发送此事务的家庭服务器的`server_name`。|
|`origin_server_ts`|`integer`|**必需：** 事务开始时在源家庭服务器上的POSIX时间戳（毫秒）。|
|`pdus`|`[PDU]`|**必需：** 房间的持久更新列表。注意，事件的格式根据房间版本而不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|

```json
{
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "pdus": [
    {
      "content": {
        "see_room_version_spec": "事件格式根据房间版本而变化。"
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

### POST /_matrix/federation/v1/get_missing_events/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1get_missing_eventsroomid)

---

检索发送者缺失的先前事件。这是通过对`latest_events`的`prev_events`进行广度优先遍历来完成的，忽略任何在`earliest_events`中的事件，并在`limit`处停止。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需：** 要搜索的房间ID。|

##### 请求体

|名称|类型|描述|
|---|---|---|
|`earliest_events`|`[string]`|**必需：** 发送者已经拥有的最新事件ID。在检索`latest_events`的先前事件时将跳过这些事件。|
|`latest_events`|`[string]`|**必需：** 要检索先前事件的事件ID。|
|`limit`|`integer`|要检索的最大事件数。默认为10。|
|`min_depth`|`integer`|要检索的事件的最小深度。默认为0。|

##### 请求体示例

```json
{
  "earliest_events": [
    "$missing_event:example.org"
  ],
  "latest_events": [
    "$event_that_has_the_missing_event_as_a_previous_event:example.org"
  ],
  "limit": 10
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|`latest_events`的先前事件，排除任何`earliest_events`，最多达到提供的`limit`。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`events`|`[PDU]`|**必需：** 缺失的事件。事件格式根据房间版本而变化——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|

```json
{
  "events": [
    {
      "content": {
        "see_room_version_spec": "事件格式根据房间版本而变化。"
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

# 检索事件[](https://spec.matrix.org/v1.11/server-server-api/#retrieving-events)

在某些情况下，家庭服务器可能缺少特定事件或无法通过回填轻松确定的房间信息。这些API为家庭服务器提供了在时间线的给定点获取事件和房间状态的选项。

### GET /_matrix/federation/v1/event/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1eventeventid)

---

检索单个事件。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`eventId`|`string`|**必需：** 要获取的事件ID。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|包含请求事件的单个PDU的事务。|

##### 200响应

|事务|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`origin`|`string`|**必需：** 发送此事务的家庭服务器的`server_name`。|
|`origin_server_ts`|`integer`|**必需：** 事务开始时在源家庭服务器上的POSIX时间戳（毫秒）。|
|`pdus`|`[PDU]`|**必需：** 单个PDU。注意，事件的格式根据房间版本而不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|

```json
{
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "pdus": [
    {
      "content": {
        "see_room_version_spec": "事件格式根据房间版本而变化。"
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

### GET /_matrix/federation/v1/state/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1stateroomid)

---

检索给定事件的房间状态快照。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需：** 要获取状态的房间ID。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`event_id`|`string`|**必需：** 房间中要检索状态的事件ID。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|在考虑请求事件引起的任何状态更改之前，房间的完全解析状态。包括事件的授权链。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`auth_chain`|`[PDU]`|**必需：** 构成房间状态的完整授权事件集及其授权事件，递归地。注意，事件的格式根据房间版本而不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|
|`pdus`|`[PDU]`|**必需：** 给定事件的房间的完全解析状态。注意，事件的格式根据房间版本而不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|

```json
{
  "auth_chain": [
    {
      "content": {
        "see_room_version_spec": "事件格式根据房间版本而变化。"
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ],
  "pdus": [
    {
      "content": {
        "see_room_version_spec": "事件格式根据房间版本而变化。"
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ]
}
```

### GET /_matrix/federation/v1/state_ids/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1state_idsroomid)

---

检索给定事件的房间状态快照，以事件ID的形式。这与调用`/state/{roomId}`的功能相同，但返回的只是事件ID而不是完整事件。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需：** 要获取状态的房间ID。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`event_id`|`string`|**必需：** 房间中要检索状态的事件ID。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|在考虑请求事件引起的任何状态更改之前，房间的完全解析状态。包括事件的授权链。|
|`404`|给定的`event_id`未找到，或者服务器不知道该事件的状态以返回任何有用的信息。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`auth_chain_ids`|`[string]`|**必需：** 构成房间状态的完整授权事件集及其授权事件，递归地。|
|`pdu_ids`|`[string]`|**必需：** 给定事件的房间的完全解析状态。|

```json
{
  "auth_chain_ids": [
    "$an_event:example.org"
  ],
  "pdu_ids": [
    "$an_event:example.org"
  ]
}
```

##### 404响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误消息。|

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "无法找到事件$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg"
}
```

### GET /_matrix/federation/v1/timestamp_to_event/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1timestamp_to_eventroomid)

---

**在 `v1.6` 中添加**

获取最接近给定时间戳的事件ID，方向由 `dir` 参数指定。

这主要用于处理相应的[客户端-服务器端点](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv1roomsroomidtimestamp_to_event)，当服务器没有所有的房间历史记录，并且没有适合的事件接近请求的时间戳时。

决定何时向另一个主服务器请求更接近的事件的启发式方法由主服务器实现决定，尽管启发式方法可能基于最近的事件是一个前向/后向极限，表明它在可能更接近的事件间隙旁边。

一个好的启发式方法是首先尝试在房间中存在时间最长的服务器，因为它们最有可能拥有我们询问的任何内容。

在本地主服务器收到响应后，它应使用 `origin_server_ts` 属性确定返回的事件是否比它在本地找到的最近事件更接近请求的时间戳。如果是，它应尝试通过[`/event/{event_id}`](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1eventeventid)端点回填此事件，以便客户端可以查询。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需：** 要搜索的房间ID|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`dir`|`string`|**必需：** 搜索的方向。`f` 表示向前，`b` 表示向后。<br><br>可选值：`[f, b]`。|
|`ts`|`integer`|**必需：** 从给定的时间戳开始搜索，以毫秒为单位从Unix纪元开始计算。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|找到符合搜索参数的事件。|
|`404`|未找到事件。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`event_id`|`string`|**必需：** 找到的事件ID|
|`origin_server_ts`|`integer`|**必需：** 事件的时间戳，以毫秒为单位从Unix纪元开始计算。|

```json
{
  "event_id": "$143273582443PhrSn:example.org",
  "origin_server_ts": 1432735824653
}
```

##### 404响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "Unable to find event from 1432684800000 in forward direction"
}
```

# 加入房间[](https://spec.matrix.org/v1.11/server-server-api/#joining-rooms)

当一个新用户希望加入一个用户的主服务器已经知道的房间时，主服务器可以通过检查房间的状态立即确定这是否允许。如果可以接受，它可以生成、签名并发出一个新的 `m.room.member` 状态事件，将用户添加到该房间中。当主服务器尚未知道房间时，它不能直接这样做。相反，它必须通过一个较长的多阶段握手过程，首先选择一个已经参与该房间的远程主服务器，并利用它来协助加入过程。这就是远程加入握手。

此握手涉及希望加入的新成员的主服务器（此处称为“加入”服务器）、托管用户请求加入的房间别名的目录服务器以及已有房间成员的主服务器（称为“驻留”服务器）。

总之，远程加入握手包括加入服务器查询目录服务器以获取房间别名的信息；接收房间ID和加入候选列表。然后，加入服务器请求其中一个驻留者的房间信息。它使用此信息构建一个 `m.room.member` 事件，最后将其发送给驻留服务器。

从概念上讲，这三个角色是不同的主服务器。在实践中，目录服务器可能驻留在房间中，因此可能被加入服务器选择为协助驻留者。同样，加入服务器可能在事件构建的两个阶段中选择相同的候选驻留者，尽管原则上每次都可以使用任何有效的候选者。因此，任何加入握手可能涉及从两个到四个主服务器，尽管大多数情况下实际上只使用两个。

```
+---------+          +---------------+            +-----------------+ +-----------------+
| 客户端  |          |   加入服务器   |            |    目录服务器    | |  常驻服务器     |
+---------+          +---------------+            +-----------------+ +-----------------+
     |                       |                             |                   |
     | 加入请求               |                             |                   |
     |---------------------->|                             |                   |
     |                       |                             |                   |
     |                       | 目录请求                     |                   |
     |                       |---------------------------->|                   |
     |                       |                             |                   |
     |                       |          目录响应            |                   |
     |                       |<----------------------------|                   |
     |                       |                             |                   |
     |                       | 发起加入请求                 |                   |
     |                       |------------------------------------------------>|
     |                       |                             |                   |
     |                       |                             | 发起加入响应       |
     |                       |<------------------------------------------------|
     |                       |                             |                   |
     |                       | 发送加入请求                 |                   |
     |                       |------------------------------------------------>|
     |                       |                             |                   |
     |                       |                             | 发送加入响应       |
     |                       |<------------------------------------------------|
     |                       |                             |                   |
     |         加入响应       |                             |                   |
     |<----------------------|                             |                   |
     |                       |                             |                   |
```

握手的第一部分通常涉及使用目录服务器通过[`/query/directory`](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1querydirectory) API端点请求房间ID和加入候选者。在新用户因收到邀请而加入房间的情况下，加入用户的主服务器可以通过选择该邀请消息的原始服务器作为加入候选者来优化此步骤。然而，加入服务器应意识到邀请的原始服务器可能已经离开房间，因此应准备好在此优化失败时回退到常规加入流程。

一旦加入服务器获得房间ID和加入候选者，它就需要获得足够的房间信息以填写 `m.room.member` 事件的必填字段。它通过从候选列表中选择一个驻留者，并使用 `GET /make_join` 端点来获得此信息。驻留服务器随后将回复足够的信息，以便加入服务器填写事件。

加入服务器应在驻留服务器接收到的模板事件上添加或替换 `origin`、`origin_server_ts` 和 `event_id`。然后由加入服务器签署此事件。

为了完成加入握手，加入服务器将此新事件提交给用于 `GET /make_join` 的驻留服务器，使用 `PUT /send_join` 端点。

驻留主服务器随后在此事件上添加其签名，并将其接受到房间的事件图中。加入服务器接收新加入房间的完整状态集以及新签名的成员事件。驻留服务器还必须将事件发送给参与房间的其他服务器。

### GET /_matrix/federation/v1/make_join/{roomId}/{userId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_joinroomiduserid)

---

请求接收服务器返回发送服务器需要的信息，以准备加入事件以进入房间。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需：** 即将加入的房间ID。|
|`userId`|`string`|**必需：** 加入事件的用户ID。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`ver`|`[string]`|发送服务器支持的房间版本。默认为 `[1]`。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|用于其余[加入房间](https://spec.matrix.org/v1.11/server-server-api/#joining-rooms)握手的模板。请注意，事件格式因房间版本而异 - 检查[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的响应正文更详细地描述了常见事件字段，可能缺少PDU的其他必需字段。**|
|`400`|请求无效，服务器尝试加入的房间版本未列在 `ver` 参数中，或者服务器无法验证[受限房间条件](https://spec.matrix.org/v1.11/server-server-api/#restricted-rooms)。<br><br>错误应传递给客户端，以便它们可以向用户提供更好的反馈。<br><br>在 `v1.2` 中新增，可能发生以下错误条件：<br><br>如果房间是[受限的](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms)，并且服务器无法验证任何条件，则必须使用 `errcode` `M_UNABLE_TO_AUTHORISE_JOIN`。这可能发生在服务器不知道作为条件列出的任何房间的情况下，例如。<br><br>`M_UNABLE_TO_GRANT_JOIN` 用于表示应尝试不同的服务器进行加入。这通常是因为驻留服务器可以看到加入用户满足一个或多个条件，例如在[受限房间](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms)的情况下，但驻留服务器无法满足 `join_authorised_via_users_server` 在结果 `m.room.member` 事件上的授权规则。|
|`403`|加入服务器尝试加入的房间不允许用户加入。|
|`404`|接收服务器未知加入服务器尝试加入的房间。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`event`|[事件模板](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_joinroomiduserid_response-200_event-template)|一个未签名的模板事件。请注意，事件格式因房间版本而异 - 检查[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|
|`room_version`|`string`|服务器尝试加入的房间版本。如果未提供，则假定房间版本为“1”或“2”。|

|事件模板|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[成员事件内容](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_joinroomiduserid_response-200_membership-event-content)|**必需：** 事件的内容。|
|`origin`|`string`|**必需：** 驻留主服务器的名称。|
|`origin_server_ts`|`integer`|**必需：** 驻留主服务器添加的时间戳。|
|`sender`|`string`|**必需：** 加入成员的用户ID。|
|`state_key`|`string`|**必需：** 加入成员的用户ID。|
|`type`|`string`|**必需：** 值为 `m.room.member`。|

|成员事件内容|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`join_authorised_via_users_server`|`string`|如果房间是[受限的](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms)并通过可用条件之一加入，则必需。如果用户响应邀请，则不需要。<br><br>属于正在加入的房间中的驻留服务器的任意用户ID，该用户ID能够向其他用户发出邀请。这用于稍后验证 `m.room.member` 事件的授权规则。<br><br>**在 `v1.2` 中添加**|
|`membership`|`string`|**必需：** 值为 `join`。|

```json
{
  "event": {
    "content": {
      "join_authorised_via_users_server": "@anyone:resident.example.org",
      "membership": "join"
    },
    "origin": "example.org",
    "origin_server_ts": 1549041175876,
    "room_id": "!somewhere:example.org",
    "sender": "@someone:example.org",
    "state_key": "@someone:example.org",
    "type": "m.room.member"
  },
  "room_version": "2"
}
```

##### 400响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|可读的错误信息。|
|`room_version`|`string`|房间的版本。如果 `errcode` 是 `M_INCOMPATIBLE_ROOM_VERSION`，则必需。|

```json
{
  "errcode": "M_INCOMPATIBLE_ROOM_VERSION",
  "error": "Your homeserver does not support the features required to join this room",
  "room_version": "3"
}
```

##### 403响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_FORBIDDEN",
  "error": "You are not invited to this room"
}
```

##### 404响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "Unknown room"
}
```

### PUT /_matrix/federation/v1/send_join/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_joinroomideventid)

---

> [!warning] 警告：
> 此API已弃用，将在未来版本中移除。

**注意：** 服务器应优先使用v2 `/send_join` 端点。

将签名的加入事件提交给驻留服务器，以便将其接受到房间的图中。请注意，事件格式因房间版本而异 - 检查[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的请求和响应正文更详细地描述了常见事件字段，可能缺少PDU的其他必需字段。**

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`eventId`|`string`|**必需：** 加入事件的事件ID。|
|`roomId`|`string`|**必需：** 即将加入的房间ID。|

##### 请求正文

|名称|类型|描述|
|---|---|---|
|`content`|[成员事件内容](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_joinroomideventid_request_membership-event-content)|**必需：** 事件的内容。|
|`origin`|`string`|**必需：** 加入主服务器的名称。|
|`origin_server_ts`|`integer`|**必需：** 加入主服务器添加的时间戳。|
|`sender`|`string`|**必需：** 加入成员的用户ID。|
|`state_key`|`string`|**必需：** 加入成员的用户ID。|
|`type`|`string`|**必需：** 值为 `m.room.member`。|

|成员事件内容|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`join_authorised_via_users_server`|`string`|如果房间是[受限的](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms)并通过可用条件之一加入，则必需。如果用户响应邀请，则不需要。<br><br>属于正在加入的房间中的驻留服务器的任意用户ID，该用户ID能够向其他用户发出邀请。这用于稍后验证 `m.room.member` 事件的授权规则。<br><br>驻留服务器拥有提供的用户ID必须在事件上有有效签名。如果驻留服务器正在接收 `/send_join` 请求，则必须在发送或将事件持久化到其他服务器之前添加签名。<br><br>**在 `v1.2` 中添加**|
|`membership`|`string`|**必需：** 值为 `join`。|

##### 请求正文示例

```json
{
  "content": {
    "membership": "join"
  },
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|加入事件已被接受到房间中。|

##### 200响应

`integer, Room State` 数组。

|房间状态|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`auth_chain`|`[PDU]`|**必需：**<br><br>加入事件之前整个当前房间状态的授权链。<br><br>请注意，事件格式因房间版本而异 - 检查[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|
|`origin`|`string`|**必需：** 驻留服务器的DNS名称。|
|`state`|`[PDU]`|**必需：**<br><br>加入事件之前解析的当前房间状态。<br><br>事件格式因房间版本而异 - 检查[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|

```json
[
  200,
  {
    "auth_chain": [
      {
        "content": {
          "see_room_version_spec": "The event format changes depending on the room version."
        },
        "room_id": "!somewhere:example.org",
        "type": "m.room.minimal_pdu"
      }
    ],
    "event": {
      "auth_events": [
        "$urlsafe_base64_encoded_eventid",
        "$a-different-event-id"
      ],
      "content": {
        "join_authorised_via_users_server": "@arbitrary:resident.example.com",
        "membership": "join"
      },
      "depth": 12,
      "hashes": {
        "sha256": "thishashcoversallfieldsincasethisisredacted"
      },
      "origin": "example.com",
      "origin_server_ts": 1404838188000,
      "prev_events": [
        "$urlsafe_base64_encoded_eventid",
        "$a-different-event-id"
      ],
      "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
      "sender": "@alice:example.com",
      "signatures": {
        "example.com": {
          "ed25519:key_version": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
        },
        "resident.example.com": {
          "ed25519:other_key_version": "a different signature"
        }
      },
      "state_key": "@alice:example.com",
      "type": "m.room.member",
      "unsigned": {
        "age": 4612
      }
    },
    "origin": "matrix.org",
    "state": [
      {
        "content": {
          "see_room_version_spec": "The event format changes depending on the room version."
        },
        "room_id": "!somewhere:example.org",
        "type": "m.room.minimal_pdu"
      }
    ]
  }
]
```

### PUT /_matrix/federation/v2/send_join/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2send_joinroomideventid)

---

**注意：** 此API与v1 API几乎相同，除了响应格式已修复。

此端点优于v1 API，因为它提供了更标准化的响应格式。接收400、404或其他状态代码的发送者，表明此端点不可用，应重试使用v1 API。

将签名的加入事件提交给驻留服务器，以便将其接受到房间的图中。请注意，事件格式因房间版本而异 - 检查[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的请求和响应正文更详细地描述了常见事件字段，可能缺少PDU的其他必需字段。**

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`eventId`|`string`|**必需：** 加入事件的事件ID。|
|`roomId`|`string`|**必需：** 即将加入的房间ID。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`omit_members`|`boolean`|如果为 `true`，表示调用服务器可以接受简化的响应，其中成员事件从 `state` 中省略，冗余事件从 `auth_chain` 中省略。<br><br>如果要加入的房间在其当前状态中没有 `m.room.name` 或 `m.room.canonical_alias` 事件，驻留服务器应确定房间摘要中定义的 `m.heroes` 属性中包含的房间成员。驻留服务器应在响应 `state` 字段中包含这些成员的成员事件，并在响应 `auth_chain` 字段中包含这些成员事件的授权链。<br><br>**在 `v1.6` 中添加**|

##### 请求正文

|名称|类型|描述|
|---|---|---|
|`content`|[成员事件内容](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2send_joinroomideventid_request_membership-event-content)|**必需：** 事件的内容。|
|`origin`|`string`|**必需：** 加入主服务器的名称。|
|`origin_server_ts`|`integer`|**必需：** 加入主服务器添加的时间戳。|
|`sender`|`string`|**必需：** 加入成员的用户ID。|
|`state_key`|`string`|**必需：** 加入成员的用户ID。|
|`type`|`string`|**必需：** 值为 `m.room.member`。|

|成员事件内容|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`join_authorised_via_users_server`|`string`|如果房间是[受限的](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms)并通过可用条件之一加入，则必需。如果用户响应邀请，则不需要。<br><br>属于正在加入的房间中的驻留服务器的任意用户ID，该用户ID能够向其他用户发出邀请。这用于稍后验证 `m.room.member` 事件的授权规则。<br><br>驻留服务器拥有提供的用户ID必须在事件上有有效签名。如果驻留服务器正在接收 `/send_join` 请求，则必须在发送或将事件持久化到其他服务器之前添加签名。<br><br>**在 `v1.2` 中添加**|
|`membership`|`string`|**必需：** 值为 `join`。|

##### 请求正文示例

```json
{
  "content": {
    "membership": "join"
  },
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|加入事件已被接受到房间中。|
|`400`|请求在某种程度上无效。<br><br>错误应传递给客户端，以便它们可以向用户提供更好的反馈。<br><br>在 `v1.2` 中新增，可能发生以下错误条件：<br><br>如果房间是[受限的](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms)，并且服务器无法验证任何条件，则必须使用 `errcode` `M_UNABLE_TO_AUTHORISE_JOIN`。这可能发生在服务器不知道作为条件列出的任何房间的情况下，例如。<br><br>`M_UNABLE_TO_GRANT_JOIN` 用于表示应尝试不同的服务器进行加入。这通常是因为驻留服务器可以看到加入用户满足一个或多个条件，例如在[受限房间](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms)的情况下，但驻留服务器无法满足 `join_authorised_via_users_server` 在结果 `m.room.member` 事件上的授权规则。|
|`403`|加入服务器尝试加入的房间不允许用户加入。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`auth_chain`|`[PDU]`|**必需：**<br><br>新加入事件的所有授权链事件，以及在 `state` 中返回的任何事件的授权链。<br><br>如果 `omit_members` 查询参数设置为 `true`，则返回的任何事件可能会从 `auth_chain` 中省略，无论是否从 `state` 中省略成员事件。<br><br>请注意，事件格式因房间版本而异 - 检查[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。<br><br>  <br>  <br>**在 `v1.6` 中更改：** 重新措辞以仅考虑 `state` 中返回的状态事件，并允许省略冗余事件。|
|`event`|`SignedMembershipEvent`|驻留服务器发送给其他服务器的成员事件，包括驻留服务器的签名。如果房间是[受限的](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms)，并且加入用户通过其中一个条件获得授权，则必需。<br><br>**在 `v1.2` 中添加**|
|`members_omitted`|`boolean`|如果 `m.room.member` 事件已从 `state` 中省略，则为 `true`。<br><br>**在 `v1.6` 中添加**|
|`origin`|`string`|**必需：** 驻留服务器的DNS名称。|
|`servers_in_room`|`[string]`|如果 `members_omitted` 为真，则**必需**。<br><br>加入前房间中活跃的服务器列表（即，拥有已加入成员的服务器）。<br><br>**在 `v1.6` 中添加**|
|`state`|`[PDU]`|**必需：**<br><br>加入事件之前解析的当前房间状态。<br><br>如果请求的 `omit_members` 设置为 `true`，则可以从响应中省略类型为 `m.room.member` 的事件以减少响应的大小。如果这样做，`members_omitted` 必须设置为 `true`。<br><br>  <br>  <br>**在 `v1.6` 中更改：** 允许省略成员事件。|

```json
{
  "auth_chain": [
    {
      "content": {
        "see_room_version_spec": "The event format changes depending on the room version."
      },
      "room_id": "!somewhere:example.org",
      "type": "m.room.minimal_pdu"
    }
  ],
  "event": {
    "auth_events": [
      "$urlsafe_base64_encoded_eventid",
      "$a-different-event-id"
    ],
    "content": {
      "join_authorised_via_users_server": "@arbitrary:resident.example.com",
      "membership": "join"
    },
    "depth": 12,
    "hashes": {
      "sha256": "thishashcoversallfieldsincasethisisredacted"
    },
    "origin": "example.com",
    "origin_server_ts": 1404838188000,
    "prev_events": [
      "$urlsafe_base64_encoded_eventid",
      "$a-different-event-id"
    ],
    "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
    "sender": "@alice:example.com",
    "signatures": {
      "example.com": {
        "ed25519:key_version": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
        },
        "resident.example.com": {
          "ed25519:other_key_version": "a different signature"
        }
      },
      "state_key": "@alice:example.com",
      "type": "m.room.member",
      "unsigned": {
        "age": 4612
      }
    },
    "members_omitted": true,
    "origin": "matrix.org",
    "servers_in_room": [
      "matrix.org",
      "example.com"
    ],
    "state": [
      {
        "content": {
          "see_room_version_spec": "The event format changes depending on the room version."
        },
        "room_id": "!somewhere:example.org",
        "type": "m.room.minimal_pdu"
      }
    ]
  }
]
```

##### 400响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_UNABLE_TO_GRANT_JOIN",
  "error": "This server cannot send invites to you."
}
```

##### 403响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_FORBIDDEN",
  "error": "You are not invited to this room"
}
```

## 受限房间[](https://spec.matrix.org/v1.11/server-server-api/#restricted-rooms)

受限房间在[客户端-服务器API](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms)中有详细描述，并且在支持受限加入规则的房间版本中可用。

处理加入受限房间请求的驻留服务器必须确保加入服务器满足 `m.room.join_rules` 指定的至少一个条件。如果没有可用的条件，或者没有符合所需的模式，则认为加入服务器未满足所有条件。

驻留服务器在 `/make_join` 和 `/send_join` 上使用400 `M_UNABLE_TO_AUTHORISE_JOIN` 错误表示驻留服务器无法验证任何条件，通常是因为驻留服务器没有条件所需房间的状态信息。

驻留服务器在 `/make_join` 和 `/send_join` 上使用400 `M_UNABLE_TO_GRANT_JOIN` 错误表示加入服务器应尝试不同的服务器。这通常是因为驻留服务器可以看到加入用户满足一个条件，尽管驻留服务器无法满足 `join_authorised_via_users_server` 在结果 `m.room.member` 事件上的授权规则。

如果加入服务器未满足所有条件，则驻留服务器使用403 `M_FORBIDDEN` 错误。

# 敲击房间[](https://spec.matrix.org/v1.11/server-server-api/#knocking-rooms)

房间可以通过加入规则允许敲击，如果允许，这为用户提供了一种请求加入（被邀请）房间的方式。敲击一个服务器已经是房间居民的房间的用户可以直接发送敲击事件，而无需使用此过程，但与[加入房间](https://spec.matrix.org/v1.11/server-server-api/#joining-rooms)类似，服务器必须通过握手方式将敲击发送给其代表。

握手与加入房间的握手基本相同，只是“加入服务器”变成了“敲击服务器”，调用的API不同（`/make_knock` 和 `/send_knock`）。

服务器可以通过离开房间来撤回敲击，如下所述拒绝邀请。

### GET /_matrix/federation/v1/make_knock/{roomId}/{userId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_knockroomiduserid)

---

**在 `v1.1` 中添加**

请求接收服务器返回发送服务器需要的信息，以准备房间的敲击事件。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需：** 即将敲击的房间ID。|
|`userId`|`string`|**必需：** 敲击事件的用户ID。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`ver`|`[string]`|**必需：** 发送服务器支持的房间版本。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|用于其余[敲击房间](https://spec.matrix.org/v1.11/server-server-api/#knocking-rooms)握手的模板。请注意，事件格式因房间版本而异 - 检查[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的响应正文更详细地描述了常见事件字段，可能缺少PDU的其他必需字段。**|
|`400`|请求无效或服务器尝试敲击的房间版本未列在 `ver` 参数中。<br><br>错误应传递给客户端，以便它们可以向用户提供更好的反馈。|
|`403`|敲击服务器或用户不允许敲击房间，例如当服务器/用户被禁止或房间未设置为接收敲击时。|
|`404`|接收服务器未知敲击服务器尝试敲击的房间。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`event`|[事件模板](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_knockroomiduserid_response-200_event-template)|**必需：** 一个未签名的模板事件。请注意，事件格式因房间版本而异 - 检查[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|
|`room_version`|`string`|**必需：** 服务器尝试敲击的房间版本。|

|事件模板|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[成员事件内容](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_knockroomiduserid_response-200_membership-event-content)|**必需：** 事件的内容。|
|`origin`|`string`|**必需：** 驻留主服务器的名称。|
|`origin_server_ts`|`integer`|**必需：** 驻留主服务器添加的时间戳。|
|`sender`|`string`|**必需：** 敲击成员的用户ID。|
|`state_key`|`string`|**必需：** 敲击成员的用户ID。|
|`type`|`string`|**必需：** 值为 `m.room.member`。|

|成员事件内容|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需：** 值为 `knock`。|

```json
{
  "event": {
    "content": {
      "membership": "knock"
    },
    "origin": "example.org",
    "origin_server_ts": 1549041175876,
    "room_id": "!somewhere:example.org",
    "sender": "@someone:example.org",
    "state_key": "@someone:example.org",
    "type": "m.room.member"
  },
  "room_version": "7"
}
```

##### 400响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|可读的错误信息。|
|`room_version`|`string`|房间的版本。如果 `errcode` 是 `M_INCOMPATIBLE_ROOM_VERSION`，则必需。|

```json
{
  "errcode": "M_INCOMPATIBLE_ROOM_VERSION",
  "error": "Your homeserver does not support the features required to knock on this room",
  "room_version": "7"
}
```

##### 403响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_FORBIDDEN",
  "error": "You are not permitted to knock on this room"
}
```

##### 404响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "Unknown room"
}
```


### PUT /_matrix/federation/v1/send_knock/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_knockroomideventid)

---

**在 `v1.1` 中新增**

提交一个签名的敲门事件到驻留服务器，以便其接受进入房间的图中。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的请求和响应主体更详细地描述了常见事件字段，可能缺少PDU所需的其他字段。**

|   |   |
|---|---|
|限速:|否|
|需要认证:|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`eventId`|`string`|**必需:** 敲门事件的事件ID。|
|`roomId`|`string`|**必需:** 即将被敲门的房间ID。|

##### 请求主体

|名称|类型|描述|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_knockroomideventid_request_membership-event-content)|**必需:** 事件的内容。|
|`origin`|`string`|**必需:** 敲门的主服务器名称。|
|`origin_server_ts`|`integer`|**必需:** 由敲门主服务器添加的时间戳。|
|`sender`|`string`|**必需:** 敲门成员的用户ID。|
|`state_key`|`string`|**必需:** 敲门成员的用户ID。|
|`type`|`string`|**必需:** 值为 `m.room.member`。|

|Membership Event Content|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需:** 值为 `knock`。|

##### 请求主体示例

```json
{
  "content": {
    "membership": "knock"
  },
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|关于房间的信息，用于向客户端传递关于敲门的信息。|
|`403`|敲门的服务器或用户不被允许敲门，例如当服务器/用户被禁止或房间未设置为接收敲门时。|
|`404`|敲门服务器试图敲门的房间对接收服务器未知。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`knock_room_state`|[[StrippedStateEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_knockroomideventid_response-200_strippedstateevent)]|**必需:** 一个[简化状态事件](https://spec.matrix.org/v1.11/client-server-api/#stripped-state)的列表，帮助敲门的发起者识别房间。|

|StrippedStateEvent|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|`EventContent`|**必需:** 事件的 `content`。|
|`sender`|`string`|**必需:** 事件的 `sender`。|
|`state_key`|`string`|**必需:** 事件的 `state_key`。|
|`type`|`string`|**必需:** 事件的 `type`。|

```json
{
  "knock_room_state": [
    {
      "content": {
        "name": "Example Room"
      },
      "sender": "@bob:example.org",
      "state_key": "",
      "type": "m.room.name"
    },
    {
      "content": {
        "join_rule": "knock"
      },
      "sender": "@bob:example.org",
      "state_key": "",
      "type": "m.room.join_rules"
    }
  ]
}
```

##### 403响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_FORBIDDEN",
  "error": "You are not permitted to knock on this room"
}
```

##### 404响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "Unknown room"
}
```

# 邀请进入房间[](https://spec.matrix.org/v1.11/server-server-api/#inviting-to-a-room)

当一个主服务器上的用户邀请同一主服务器上的另一个用户时，主服务器可以自行签署成员事件并跳过此处定义的过程。然而，当一个用户邀请另一个主服务器上的用户时，必须向该主服务器发出请求以签署和验证事件。

请注意，邀请用于表示敲门已被接受。因此，如果邀请事件未直接引用敲门，接收服务器应准备手动将先前的敲门与邀请链接起来。

### PUT /_matrix/federation/v1/invite/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid)

---

邀请远程用户进入房间。一旦事件由邀请主服务器和被邀请主服务器签署，邀请主服务器可以将其发送给房间中的所有服务器。

服务器应优先使用v2 API进行邀请，而不是v1 API。接收v1邀请请求的服务器必须假设房间版本为 `"1"` 或 `"2"`。

请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的请求和响应主体更详细地描述了常见事件字段，可能缺少PDU所需的其他字段。**

|   |   |
|---|---|
|限速:|否|
|需要认证:|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`eventId`|`string`|**必需:** 由邀请服务器生成的邀请事件ID。|
|`roomId`|`string`|**必需:** 用户被邀请进入的房间ID。|

##### 请求主体

|InviteEvent|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_request_membership-event-content)|**必需:** 事件的内容，需与[客户端-服务器API](https://spec.matrix.org/v1.11/client-server-api/)中可用的内容匹配。必须包含 `membership` 为 `invite`。|
|`origin`|`string`|**必需:** 邀请主服务器的名称。|
|`origin_server_ts`|`integer`|**必需:** 由邀请主服务器添加的时间戳。|
|`sender`|`string`|**必需:** 发送原始 `m.room.third_party_invite` 的用户的矩阵ID。|
|`state_key`|`string`|**必需:** 被邀请成员的用户ID。|
|`type`|`string`|**必需:** 值为 `m.room.member`。|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_request_unsigneddata)|事件旁边包含的信息，未签名。可能包含此处未列出的更多信息。|

|Membership Event Content|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需:** 值为 `invite`。|

|UnsignedData|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`invite_room_state`|[[StrippedStateEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_request_strippedstateevent)]|一个可选的[简化状态事件](https://spec.matrix.org/v1.11/client-server-api/#stripped-state)列表，帮助邀请的接收者识别房间。|

|StrippedStateEvent|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|`EventContent`|**必需:** 事件的 `content`。|
|`sender`|`string`|**必需:** 事件的 `sender`。|
|`state_key`|`string`|**必需:** 事件的 `state_key`。|
|`type`|`string`|**必需:** 事件的 `type`。|

##### 请求主体示例

```json
{
  "content": {
    "membership": "invite"
  },
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@joe:elsewhere.com",
  "type": "m.room.member",
  "unsigned": {
    "invite_room_state": [
      {
        "content": {
          "name": "Example Room"
        },
        "sender": "@bob:example.org",
        "state_key": "",
        "type": "m.room.name"
      },
      {
        "content": {
          "join_rule": "invite"
        },
        "sender": "@bob:example.org",
        "state_key": "",
        "type": "m.room.join_rules"
      }
    ]
  }
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|添加了被邀请服务器的签名的事件。事件的所有其他字段应保持不变。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|
|`403`|不允许邀请。这可能是由于多种原因，包括：<br><br>- 发送者不允许向目标用户/主服务器发送邀请。<br>- 主服务器不允许任何人邀请其用户。<br>- 主服务器拒绝参与房间。|

##### 200响应

`integer, Event Container` 数组。

|Event Container|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`event`|[InviteEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_response-200_inviteevent)|**必需:** 一个邀请事件。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|

|InviteEvent|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1inviteroomideventid_response-200_membership-event-content)|**必需:** 事件的内容，需与[客户端-服务器API](https://spec.matrix.org/v1.11/client-server-api/)中可用的内容匹配。必须包含 `membership` 为 `invite`。|
|`origin`|`string`|**必需:** 邀请主服务器的名称。|
|`origin_server_ts`|`integer`|**必需:** 由邀请主服务器添加的时间戳。|
|`sender`|`string`|**必需:** 发送原始 `m.room.third_party_invite` 的用户的矩阵ID。|
|`state_key`|`string`|**必需:** 被邀请成员的用户ID。|
|`type`|`string`|**必需:** 值为 `m.room.member`。|

|Membership Event Content|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需:** 值为 `invite`。|

```json
[
  200,
  {
    "event": {
      "content": {
        "membership": "invite"
      },
      "origin": "example.org",
      "origin_server_ts": 1549041175876,
      "room_id": "!somewhere:example.org",
      "sender": "@someone:example.org",
      "signatures": {
        "elsewhere.com": {
          "ed25519:k3y_versi0n": "SomeOtherSignatureHere"
        },
        "example.com": {
          "ed25519:key_version": "SomeSignatureHere"
        }
      },
      "state_key": "@someone:example.org",
      "type": "m.room.member",
      "unsigned": {
        "invite_room_state": [
          {
            "content": {
              "name": "Example Room"
            },
            "sender": "@bob:example.org",
            "state_key": "",
            "type": "m.room.name"
          },
          {
            "content": {
              "join_rule": "invite"
            },
            "sender": "@bob:example.org",
            "state_key": "",
            "type": "m.room.join_rules"
          }
        ]
      }
    }
  }
]
```

##### 403响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_FORBIDDEN",
  "error": "User cannot invite the target user."
}
```

### PUT /_matrix/federation/v2/invite/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid)

---

**注意:** 此API与v1 API几乎相同，除了请求主体不同，响应格式已修复。

邀请远程用户进入房间。一旦事件由邀请主服务器和被邀请主服务器签署，邀请主服务器可以将其发送给房间中的所有服务器。

此端点优于v1 API，因为它对服务器更有用。接收400或404响应的发送者应重试使用v1 API，因为服务器可能较旧，如果房间版本为“1”或“2”。

请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的请求和响应主体更详细地描述了常见事件字段，可能缺少PDU所需的其他字段。**

|   |   |
|---|---|
|限速:|否|
|需要认证:|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`eventId`|`string`|**必需:** 由邀请服务器生成的邀请事件ID。|
|`roomId`|`string`|**必需:** 用户被邀请进入的房间ID。|

##### 请求主体

|名称|类型|描述|
|---|---|---|
|`event`|[InviteEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_request_inviteevent)|**必需:** 一个邀请事件。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|
|`invite_room_state`|[[StrippedStateEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_request_strippedstateevent)]|一个可选的[简化状态事件](https://spec.matrix.org/v1.11/client-server-api/#stripped-state)列表，帮助邀请的接收者识别房间。|
|`room_version`|`string`|**必需:** 用户被邀请进入的房间版本。|

|InviteEvent|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_request_membership-event-content)|**必需:** 事件的内容，需与[客户端-服务器API](https://spec.matrix.org/v1.11/client-server-api/)中可用的内容匹配。必须包含 `membership` 为 `invite`。|
|`origin`|`string`|**必需:** 邀请主服务器的名称。|
|`origin_server_ts`|`integer`|**必需:** 由邀请主服务器添加的时间戳。|
|`sender`|`string`|**必需:** 发送原始 `m.room.third_party_invite` 的用户的矩阵ID。|
|`state_key`|`string`|**必需:** 被邀请成员的用户ID。|
|`type`|`string`|**必需:** 值为 `m.room.member`。|

|Membership Event Content|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需:** 值为 `invite`。|

|StrippedStateEvent|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|`EventContent`|**必需:** 事件的 `content`。|
|`sender`|`string`|**必需:** 事件的 `sender`。|
|`state_key`|`string`|**必需:** 事件的 `state_key`。|
|`type`|`string`|**必需:** 事件的 `type`。|

##### 请求主体示例

```json
{
  "event": {
    "content": {
      "membership": "invite"
    },
    "origin": "matrix.org",
    "origin_server_ts": 1234567890,
    "sender": "@someone:example.org",
    "state_key": "@joe:elsewhere.com",
    "type": "m.room.member"
  },
  "invite_room_state": [
    {
      "content": {
        "name": "Example Room"
      },
      "sender": "@bob:example.org",
      "state_key": "",
      "type": "m.room.name"
    },
    {
      "content": {
        "join_rule": "invite"
      },
      "sender": "@bob:example.org",
      "state_key": "",
      "type": "m.room.join_rules"
    }
  ],
  "room_version": "2"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|添加了被邀请服务器的签名的事件。事件的所有其他字段应保持不变。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|
|`400`|请求无效或服务器尝试加入的房间版本不在 `ver` 参数中列出。<br><br>错误应传递给客户端，以便它们可以向用户提供更好的反馈。|
|`403`|不允许邀请。这可能是由于多种原因，包括：<br><br>- 发送者不允许向目标用户/主服务器发送邀请。<br>- 主服务器不允许任何人邀请其用户。<br>- 主服务器拒绝参与房间。|

##### 200响应

|Event Container|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`event`|[InviteEvent](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_response-200_inviteevent)|**必需:** 一个邀请事件。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|

|InviteEvent|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2inviteroomideventid_response-200_membership-event-content)|**必需:** 事件的内容，需与[客户端-服务器API](https://spec.matrix.org/v1.11/client-server-api/)中可用的内容匹配。必须包含 `membership` 为 `invite`。|
|`origin`|`string`|**必需:** 邀请主服务器的名称。|
|`origin_server_ts`|`integer`|**必需:** 由邀请主服务器添加的时间戳。|
|`sender`|`string`|**必需:** 发送原始 `m.room.third_party_invite` 的用户的矩阵ID。|
|`state_key`|`string`|**必需:** 被邀请成员的用户ID。|
|`type`|`string`|**必需:** 值为 `m.room.member`。|

|Membership Event Content|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需:** 值为 `invite`。|

```json
{
  "event": {
    "content": {
      "membership": "invite"
    },
    "origin": "example.org",
    "origin_server_ts": 1549041175876,
    "room_id": "!somewhere:example.org",
    "sender": "@someone:example.org",
    "signatures": {
      "elsewhere.com": {
        "ed25519:k3y_versi0n": "SomeOtherSignatureHere"
      },
      "example.com": {
        "ed25519:key_version": "SomeSignatureHere"
      }
    },
    "state_key": "@someone:example.org",
    "type": "m.room.member",
    "unsigned": {
      "invite_room_state": [
        {
          "content": {
            "name": "Example Room"
          },
          "sender": "@bob:example.org",
          "state_key": "",
          "type": "m.room.name"
        },
        {
          "content": {
            "join_rule": "invite"
          },
          "sender": "@bob:example.org",
          "state_key": "",
          "type": "m.room.join_rules"
        }
      ]
    }
  }
}
```

##### 400响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|可读的错误信息。|
|`room_version`|`string`|房间的版本。如果 `errcode` 是 `M_INCOMPATIBLE_ROOM_VERSION`，则为必需。|

```json
{
  "errcode": "M_INCOMPATIBLE_ROOM_VERSION",
  "error": "Your homeserver does not support the features required to join this room",
  "room_version": "3"
}
```

##### 403响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_FORBIDDEN",
  "error": "User cannot invite the target user."
}
```

# 离开房间（拒绝邀请）[](https://spec.matrix.org/v1.11/server-server-api/#leaving-rooms-rejecting-invites)

通常，主服务器可以发送适当的 `m.room.member` 事件以让用户离开房间，拒绝本地邀请或撤回敲门。来自其他主服务器的远程邀请/敲门不涉及图中的服务器，因此需要另一种方法来拒绝邀请。加入房间并立即离开不建议，因为客户端和服务器会将其解释为接受邀请，然后离开房间，而不是拒绝邀请。

类似于[加入房间](https://spec.matrix.org/v1.11/server-server-api/#joining-rooms)握手，希望离开房间的服务器首先向驻留服务器发送 `/make_leave` 请求。在拒绝邀请的情况下，驻留服务器可能是发送邀请的服务器。在从 `/make_leave` 接收到模板事件后，离开服务器签署事件并用自己的 `event_id` 替换它。然后通过 `/send_leave` 发送给驻留服务器。驻留服务器将事件发送给房间中的其他服务器。

### GET /_matrix/federation/v1/make_leave/{roomId}/{userId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_leaveroomiduserid)

---

请求接收服务器返回发送服务器需要的信息，以准备离开事件以退出房间。

|   |   |
|---|---|
|限速:|否|
|需要认证:|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需:** 即将离开的房间ID。|
|`userId`|`string`|**必需:** 离开事件将针对的用户ID。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|用于调用 `/send_leave` 的模板。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的响应主体更详细地描述了常见事件字段，可能缺少PDU所需的其他字段。**|
|`403`|请求未授权。这可能意味着用户不在房间中。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`event`|[Event Template](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_leaveroomiduserid_response-200_event-template)|一个未签名的模板事件。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。|
|`room_version`|`string`|服务器尝试离开的房间版本。如果未提供，房间版本假定为“1”或“2”。|

|Event Template|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_leaveroomiduserid_response-200_membership-event-content)|**必需:** 事件的内容。|
|`origin`|`string`|**必需:** 驻留主服务器的名称。|
|`origin_server_ts`|`integer`|**必需:** 由驻留主服务器添加的时间戳。|
|`sender`|`string`|**必需:** 离开成员的用户ID。|
|`state_key`|`string`|**必需:** 离开成员的用户ID。|
|`type`|`string`|**必需:** 值为 `m.room.member`。|

|Membership Event Content|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需:** 值为 `leave`。|

```json
{
  "event": {
    "content": {
      "membership": "leave"
    },
    "origin": "example.org",
    "origin_server_ts": 1549041175876,
    "room_id": "!somewhere:example.org",
    "sender": "@someone:example.org",
    "state_key": "@someone:example.org",
    "type": "m.room.member"
  },
  "room_version": "2"
}
```

##### 403响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需:** 错误代码。|
|`error`|`string`|可读的错误信息。|

```json
{
  "errcode": "M_FORBIDDEN",
  "error": "User is not in the room."
}
```

### PUT /_matrix/federation/v1/send_leave/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_leaveroomideventid)

---

> [!warning] 警告:
> 此API已弃用，将在未来版本中删除。

**注意:** 服务器应优先使用v2 `/send_leave` 端点。

提交一个签名的离开事件到驻留服务器，以便其接受进入房间的图中。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的请求和响应主体更详细地描述了常见事件字段，可能缺少PDU所需的其他字段。**

|   |   |
|---|---|
|限速:|否|
|需要认证:|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`eventId`|`string`|**必需:** 离开事件的事件ID。|
|`roomId`|`string`|**必需:** 即将离开的房间ID。|

##### 请求主体

|名称|类型|描述|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1send_leaveroomideventid_request_membership-event-content)|**必需:** 事件的内容。|
|`depth`|`integer`|**必需:** 此字段必须存在，但被忽略；可以为0。|
|`origin`|`string`|**必需:** 离开主服务器的名称。|
|`origin_server_ts`|`integer`|**必需:** 由离开主服务器添加的时间戳。|
|`sender`|`string`|**必需:** 离开成员的用户ID。|
|`state_key`|`string`|**必需:** 离开成员的用户ID。|
|`type`|`string`|**必需:** 值为 `m.room.member`。|

|Membership Event Content|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需:** 值为 `leave`。|

##### 请求主体示例

```json
{
  "content": {
    "membership": "leave"
  },
  "depth": 12,
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|一个空响应，表示事件已被接收主服务器接受进入图中。|

##### 200响应

`integer, Empty Object` 数组。

```json
[
  200,
  {}
]
```

### PUT /_matrix/federation/v2/send_leave/{roomId}/{eventId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2send_leaveroomideventid)

---

**注意:** 此API与v1 API几乎相同，除了响应格式已修复。

此端点优于v1 API，因为它提供了更标准化的响应格式。接收400、404或其他状态码指示此端点不可用的发送者应重试使用v1 API。

提交一个签名的离开事件到驻留服务器，以便其接受进入房间的图中。请注意，根据房间版本，事件格式会有所不同——请查看[房间版本规范](https://spec.matrix.org/v1.11/rooms)以获取精确的事件格式。**此处的请求和响应主体更详细地描述了常见事件字段，可能缺少PDU所需的其他字段。**

|   |   |
|---|---|
|限速:|否|
|需要认证:|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`eventId`|`string`|**必需:** 离开事件的事件ID。|
|`roomId`|`string`|**必需:** 即将离开的房间ID。|

##### 请求主体

|名称|类型|描述|
|---|---|---|
|`content`|[Membership Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv2send_leaveroomideventid_request_membership-event-content)|**必需:** 事件的内容。|
|`depth`|`integer`|**必需:** 此字段必须存在，但被忽略；可以为0。|
|`origin`|`string`|**必需:** 离开主服务器的名称。|
|`origin_server_ts`|`integer`|**必需:** 由离开主服务器添加的时间戳。|
|`sender`|`string`|**必需:** 离开成员的用户ID。|
|`state_key`|`string`|**必需:** 离开成员的用户ID。|
|`type`|`string`|**必需:** 值为 `m.room.member`。|

|Membership Event Content|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需:** 值为 `leave`。|

##### 请求主体示例

```json
{
  "content": {
    "membership": "leave"
  },
  "depth": 12,
  "origin": "matrix.org",
  "origin_server_ts": 1234567890,
  "sender": "@someone:example.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|一个空响应，表示事件已被接收主服务器接受进入图中。|

##### 200响应

```json
{}
```

# 第三方邀请[](https://spec.matrix.org/v1.11/server-server-api/#third-party-invites)

> [!info] 信息:
> 关于第三方邀请的更多信息，请参阅[客户端-服务器API](https://spec.matrix.org/v1.11/client-server-api)中的第三方邀请模块。

当用户想要在房间中邀请另一个用户但不知道要邀请的矩阵ID时，他们可以使用第三方标识符（例如电子邮件或电话号码）进行邀请。

此标识符及其与矩阵ID的绑定由实现[身份服务API](https://spec.matrix.org/v1.11/identity-service-api)的身份服务器验证。

## 第三方标识符存在关联的情况[](https://spec.matrix.org/v1.11/server-server-api/#cases-where-an-association-exists-for-a-third-party-identifier)

如果第三方标识符已绑定到矩阵ID，身份服务器上的查找请求将返回它。然后，邀请由邀请主服务器处理为标准的 `m.room.member` 邀请事件。这是最简单的情况。

## 第三方标识符不存在关联的情况[](https://spec.matrix.org/v1.11/server-server-api/#cases-where-an-association-doesnt-exist-for-a-third-party-identifier)

如果第三方标识符未绑定到任何矩阵ID，邀请主服务器将请求身份服务器为此标识符存储邀请，并将其发送给绑定到其矩阵ID的任何人。它还将在房间中发送一个 `m.room.third_party_invite` 事件，以指定显示名称、令牌和身份服务器作为邀请存储请求的响应提供的公钥。

当具有待处理邀请的第三方标识符绑定到矩阵ID时，身份服务器将发送POST请求到ID的主服务器，如身份服务API的[邀请存储](https://spec.matrix.org/v1.11/identity-service-api#invitation-storage)部分所述。

以下过程适用于身份服务器发送的每个邀请：

被邀请的主服务器将创建一个包含特殊 `third_party_invite` 部分的 `m.room.member` 邀请事件，其中包含身份服务器提供的令牌和签名对象。

如果被邀请的主服务器在邀请来自的房间中，它可以验证事件并发送它。

然而，如果被邀请的主服务器不在邀请来自的房间中，它将需要请求房间的主服务器验证事件。

### PUT /_matrix/federation/v1/3pid/onbind[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv13pidonbind)

---

用于身份服务器通知主服务器其用户之一已成功绑定第三方标识符，包括身份服务器已知的任何待处理的房间邀请。

|   |   |
|---|---|
|限速:|否|
|需要认证:|否|

---

#### 请求

##### 请求主体

|名称|类型|描述|
|---|---|---|
|`address`|`string`|**必需:** 第三方标识符本身。例如，电子邮件地址。|
|`invites`|[[Third-party Invite](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv13pidonbind_request_third-party-invite)]|**必需:** 第三方标识符收到的待处理邀请列表。|
|`medium`|`string`|**必需:** 第三方标识符的类型。目前只有“email”是可能的值。|
|`mxid`|`string`|**必需:** 现在绑定到第三方标识符的用户。|

|Third-party Invite|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`address`|`string`|**必需:** 收到邀请的第三方标识符。|
|`medium`|`string`|**必需:** 第三方邀请发出的类型。目前只使用“email”。|
|`mxid`|`string`|**必需:** 现在绑定的收到邀请的用户ID。|
|`room_id`|`string`|**必需:** 邀请有效的房间ID。|
|`sender`|`string`|**必需:** 发送邀请的用户ID。|
|`signed`|[Identity Server Signatures](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv13pidonbind_request_identity-server-signatures)|**必需:** 使用长期私钥的身份服务器签名。|

|Identity Server Signatures|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`mxid`|`string`|**必需:** 已绑定到第三方标识符的用户ID。|
|`signatures`|{string: [Identity Server Domain Signature](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv13pidonbind_request_identity-server-domain-signature)}|**必需:** 身份服务器的签名。`string` 键是身份服务器的域名，例如 vector.im|
|`token`|`string`|**必需:** 一个令牌。|

|Identity Server Domain Signature|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`ed25519:0`|`string`|**必需:** 签名。|

##### 请求主体示例

```json
{
  "address": "alice@example.com",
  "invites": [
    {
      "address": "alice@example.com",
      "medium": "email",
      "mxid": "@alice:matrix.org",
      "room_id": "!somewhere:example.org",
      "sender": "@bob:matrix.org",
      "signed": {
        "mxid": "@alice:matrix.org",
        "signatures": {
          "vector.im": {
            "ed25519:0": "SomeSignatureGoesHere"
          }
        },
        "token": "Hello World"
      }
    }
  ],
  "medium": "email",
  "mxid": "@alice:matrix.org"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|主服务器已处理通知。|

##### 200响应

```json
{}
```

### PUT /_matrix/federation/v1/exchange_third_party_invite/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1exchange_third_party_inviteroomid)

---

接收服务器将验证请求主体中给出的部分 `m.room.member` 事件。如果有效，接收服务器将根据[邀请进入房间](https://spec.matrix.org/v1.11/server-server-api/#inviting-to-a-room)部分发出邀请，然后返回对此请求的响应。

|   |   |
|---|---|
|限速:|否|
|需要认证:|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需:** 要交换第三方邀请的房间ID|

##### 请求主体

|名称|类型|描述|
|---|---|---|
|`content`|[Event Content](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1exchange_third_party_inviteroomid_request_event-content)|**必需:** 事件内容|
|`room_id`|`string`|**必需:** 事件所属的房间ID。必须与路径中给出的ID匹配。|
|`sender`|`string`|**必需:** 发送原始 `m.room.third_party_invite` 事件的用户ID。|
|`state_key`|`string`|**必需:** 被邀请用户的用户ID|
|`type`|`string`|**必需:** 事件类型。必须为 `m.room.member`|

|Event Content|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`membership`|`string`|**必需:** 成员状态。必须为 `invite`|
|`third_party_invite`|[Third-party Invite](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1exchange_third_party_inviteroomid_request_third-party-invite)|**必需:** 第三方邀请|

|Third-party Invite|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`display_name`|`string`|**必需:** 可以显示的名称，用于代表用户而不是他们的第三方标识符。|
|`signed`|[Invite Signatures](https://spec.matrix.org/v1.11/server-server-api/#put_matrixfederationv1exchange_third_party_inviteroomid_request_invite-signatures)|**必需:** 已签名的内容块，服务器可以用来验证事件。|

|Invite Signatures|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`mxid`|`string`|**必需:** 被邀请的矩阵用户ID|
|`signatures`|`{string: {string: string}}`|**必需:**<br><br>此事件的服务器签名。<br><br>签名是使用[签名JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算的。|
|`token`|`string`|**必需:** 用于验证事件的令牌|

##### 请求主体示例

```json
{
  "content": {
    "membership": "invite",
    "third_party_invite": {
      "display_name": "alice",
      "signed": {
        "mxid": "@alice:localhost",
        "signatures": {
          "magic.forest": {
            "ed25519:3": "fQpGIW1Snz+pwLZu6sTy2aHy/DYWWTspTJRPyNp0PKkymfIsNffysMl6ObMMFdIJhk6g6pwlIqZ54rxo8SLmAg"
          }
        },
        "token": "abc123"
      }
    }
  },
  "room_id": "!abc123:matrix.org",
  "sender": "@joe:matrix.org",
  "state_key": "@someone:example.org",
  "type": "m.room.member"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|邀请已成功发出。|

##### 200响应

```json
{}
```

### 验证邀请[](https://spec.matrix.org/v1.11/server-server-api/#verifying-the-invite)

当主服务器接收到一个带有 `third_party_invite` 对象的 `m.room.member` 邀请事件时，它必须验证最初被邀请到房间的第三方标识符与声称绑定到它的矩阵ID之间的关联已被验证，而无需依赖第三方服务器。

为此，它将从房间的状态事件中获取 `m.room.third_party_invite` 事件，其状态键与 `m.room.member` 事件内容中的 `third_party_invite` 对象中的 `token` 键的值匹配，以获取身份服务器最初提供的公钥。

然后，它将使用这些密钥验证 `m.room.member` 事件内容中的 `third_party_invite` 对象中的 `signed` 对象是否由同一身份服务器签署。

由于此 `signed` 对象只能在第三方标识符与矩阵ID绑定时由身份服务器发出的POST请求中交付，并且包含被邀请用户的矩阵ID和存储邀请时提供的令牌，此验证将证明 `m.room.member` 邀请事件来自拥有被邀请第三方标识符的用户。

# 公共房间目录[](https://spec.matrix.org/v1.11/server-server-api/#public-room-directory)

为了补充[客户端-服务器API](https://spec.matrix.org/v1.11/client-server-api)的房间目录，主服务器需要一种方法来查询另一个服务器的公共房间。这可以通过向要检索房间目录的服务器的 `/publicRooms` 端点发出请求来完成。

### GET /_matrix/federation/v1/publicRooms[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1publicrooms)

---

获取主服务器的所有公共房间。这不应返回列在另一个主服务器目录中的房间，只返回列在接收主服务器目录中的房间。

|   |   |
|---|---|
|限速:|否|
|需要认证:|是|

---

#### 请求

##### 请求参数

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`include_all_networks`|`boolean`|是否包含主服务器上应用服务定义的所有网络/协议。默认为false。|
|`limit`|`integer`|返回的最大房间数。默认为0（无限制）。|
|`since`|`string`|从上一次调用此端点获取更多房间的分页令牌。|
|`third_party_instance_id`|`string`|从主服务器请求的特定第三方网络/协议。仅当 `include_all_networks` 为false时可用。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|主服务器的公共房间列表。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`chunk`|[[PublicRoomsChunk](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1publicrooms_response-200_publicroomschunk)]|**必需:** 公共房间的分页块。|
|`next_batch`|`string`|响应的分页令牌。缺少此令牌意味着没有更多结果可获取，客户端应停止分页。|
|`prev_batch`|`string`|允许获取先前结果的分页令牌。缺少此令牌意味着在此批次之前没有结果，即这是第一个批次。|
|`total_room_count_estimate`|`integer`|公共房间总数的估计值，如果服务器有估计值。|

|PublicRoomsChunk|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`avatar_url`|[URI](http://tools.ietf.org/html/rfc3986)|房间头像的URL（如果设置）。|
|`canonical_alias`|`string`|房间的规范别名（如果有）。|
|`guest_can_join`|`boolean`|**必需:** 客户用户是否可以加入房间并参与其中。如果可以，他们将像其他用户一样受到普通权限级别规则的约束。|
|`join_rule`|`string`|房间的加入规则。未出现时，房间被视为 `public`。请注意，具有 `invite` 加入规则的房间不应出现在此处，但具有 `knock` 规则的房间由于其接近公共的性质而会出现。|
|`name`|`string`|房间的名称（如果有）。|
|`num_joined_members`|`integer`|**必需:** 加入房间的成员数。|
|`room_id`|`string`|**必需:** 房间的ID。|
|`room_type`|`string`|房间的 `type`（来自[`m.room.create`](https://spec.matrix.org/v1.11/client-server-api/#mroomcreate)），如果有。<br><br>**在 `v1.4` 中新增**|
|`topic`|`string`|房间的主题（如果有）。|
|`world_readable`|`boolean`|**必需:** 客户用户是否可以在不加入的情况下查看房间。|

```json
{
  "chunk": [
    {
      "avatar_url": "mxc://bleecker.street/CHEDDARandBRIE",
      "guest_can_join": false,
      "join_rule": "public",
      "name": "CHEESE",
      "num_joined_members": 37,
      "room_id": "!ol19s:bleecker.street",
      "room_type": "m.space",
      "topic": "Tasty tasty cheese",
      "world_readable": true
    }
  ],
  "next_batch": "p190q",
  "prev_batch": "p1902",
  "total_room_count_estimate": 115
}
```

### POST /_matrix/federation/v1/publicRooms[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1publicrooms)

---

列出服务器上的公共房间，并提供可选的过滤器。

此API返回分页响应。房间按加入成员数排序，最大房间优先。

请注意，此端点接收并返回与客户端-服务器API的 `POST /publicRooms` 端点中看到的相同格式。

|   |   |
|---|---|
|限速:|否|
|需要认证:|是|

---

#### 请求

##### 请求主体

|名称|类型|描述|
|---|---|---|
|`filter`|[Filter](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1publicrooms_request_filter)|应用于结果的过滤器。|
|`include_all_networks`|`boolean`|是否包含主服务器上应用服务已知的所有网络/协议。默认为false。|
|`limit`|`integer`|限制返回的结果数量。|
|`since`|`string`|上一次请求的分页令牌，允许服务器获取下一个（或上一个）房间批次。分页方向仅由提供的令牌决定，而不是通过显式标志。|
|`third_party_instance_id`|`string`|从主服务器请求的特定第三方网络/协议。仅当 `include_all_networks` 为false时可用。|

|Filter|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`generic_search_term`|`string`|可选字符串，用于在房间元数据中搜索，例如名称、主题、规范别名等。|
|`room_types`|`[string\|null]`|可选的[房间类型](https://spec.matrix.org/v1.11/client-server-api/#types)列表，用于搜索。要包括没有房间类型的房间，请在此列表中指定 `null`。未指定时，返回所有适用的房间（无论类型如何）。<br><br>**在 `v1.4` 中新增**|

##### 请求主体示例

```json
{
  "filter": {
    "generic_search_term": "foo",
    "room_types": [
      null,
      "m.space"
    ]
  },
  "include_all_networks": false,
  "limit": 10,
  "third_party_instance_id": "irc"
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|服务器上的房间列表。|

##### 200响应

|名称|类型|描述|
|---|---|---|
|`chunk`|[[PublicRoomsChunk](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1publicrooms_response-200_publicroomschunk)]|**必需:** 公共房间的分页块。|
|`next_batch`|`string`|响应的分页令牌。缺少此令牌意味着没有更多结果可获取，客户端应停止分页。|
|`prev_batch`|`string`|允许获取先前结果的分页令牌。缺少此令牌意味着在此批次之前没有结果，即这是第一个批次。|
|`total_room_count_estimate`|`integer`|公共房间总数的估计值，如果服务器有估计值。|

|PublicRoomsChunk|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`avatar_url`|[URI](http://tools.ietf.org/html/rfc3986)|房间头像的URL（如果设置）。|
|`canonical_alias`|`string`|房间的规范别名（如果有）。|
|`guest_can_join`|`boolean`|**必需:** 客户用户是否可以加入房间并参与其中。如果可以，他们将像其他用户一样受到普通权限级别规则的约束。|
|`join_rule`|`string`|房间的加入规则。未出现时，房间被视为 `public`。请注意，具有 `invite` 加入规则的房间不应出现在此处，但具有 `knock` 规则的房间由于其接近公共的性质而会出现。|
|`name`|`string`|房间的名称（如果有）。|
|`num_joined_members`|`integer`|**必需:** 加入房间的成员数。|
|`room_id`|`string`|**必需:** 房间的ID。|
|`room_type`|`string`|房间的 `type`（来自[`m.room.create`](https://spec.matrix.org/v1.11/client-server-api/#mroomcreate)），如果有。<br><br>**在 `v1.4` 中新增**|
|`topic`|`string`|房间的主题（如果有）。|
|`world_readable`|`boolean`|**必需:** 客户用户是否可以在不加入的情况下查看房间。|

```json
{
  "chunk": [
    {
      "avatar_url": "mxc://bleecker.street/CHEDDARandBRIE",
      "guest_can_join": false,
      "join_rule": "public",
      "name": "CHEESE",
      "num_joined_members": 37,
      "room_id": "!ol19s:bleecker.street",
      "room_type": "m.space",
      "topic": "Tasty tasty cheese",
      "world_readable": true
    }
  ],
  "next_batch": "p190q",
  "prev_batch": "p1902",
  "total_room_count_estimate": 115
}
```


# 空间[](https://spec.matrix.org/v1.11/server-server-api/#spaces)

为了补充[客户端-服务器 API 的空间模块](https://spec.matrix.org/v1.11/client-server-api/#spaces)，主服务器需要一种方法从其他服务器查询空间信息。

### GET /_matrix/federation/v1/hierarchy/{roomId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1hierarchyroomid)

---

**在 `v1.2` 中添加**

客户端-服务器 [`GET /hierarchy`](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv1roomsroomidhierarchy) 端点的联邦版本。与客户端-服务器 API 版本不同，此端点不进行分页。相反，返回请求服务器可以合理窥视/加入的所有空间房间的子房间。请求服务器负责进一步过滤用户请求的结果。

仅考虑房间的 [`m.space.child`](https://spec.matrix.org/v1.11/client-server-api/#mspacechild) 状态事件。无效的子房间和父事件不在此端点的覆盖范围内。

对此端点的响应应缓存一段时间。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`roomId`|`string`|**必需：** 要获取层次结构的空间的房间 ID。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`suggested_only`|`boolean`|可选（默认 `false`）标志，指示服务器是否仅考虑建议的房间。建议的房间在其 [`m.space.child`](https://spec.matrix.org/v1.11/client-server-api/#mspacechild) 事件内容中进行了注释。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|空间房间及其子房间。|
|`404`|服务器不知道该房间，或者请求服务器无法窥视/加入它（如果尝试的话）。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`children`|[[SpaceHierarchyChildRoomsChunk](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1hierarchyroomid_response-200_spacehierarchychildroomschunk)]|**必需：** 空间子房间的摘要。请求服务器无法窥视/加入的房间将被排除。|
|`inaccessible_children`|`[string]`|**必需：**<br><br>请求服务器没有可行方法窥视/加入的房间 ID 列表。响应服务器无法提供详细信息的房间将被直接排除在响应之外。<br><br>假设请求和响应服务器都表现良好，请求服务器应将这些房间 ID 视为从任何地方都无法访问。它们不应被重新请求。|
|`room`|[SpaceHierarchyParentRoom](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1hierarchyroomid_response-200_spacehierarchyparentroom)|**必需：** 请求房间的摘要。|

|SpaceHierarchyChildRoomsChunk|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`allowed_room_ids`|`[string]`|如果房间是[受限房间](https://spec.matrix.org/v1.11/server-server-api/#restricted-rooms)，这些是加入规则中指定的房间 ID。否则为空或省略。|
|`avatar_url`|[URI](http://tools.ietf.org/html/rfc3986)|房间头像的 URL（如果设置了）。|
|`canonical_alias`|`string`|房间的规范别名（如果有）。|
|`guest_can_join`|`boolean`|**必需：** 客户端用户是否可以加入房间并参与其中。如果可以，他们将像其他用户一样遵循普通权限级别规则。|
|`join_rule`|`string`|房间的加入规则。如果不存在，房间被假定为`public`。|
|`name`|`string`|房间的名称（如果有）。|
|`num_joined_members`|`integer`|**必需：** 加入房间的成员数量。|
|`room_id`|`string`|**必需：** 房间的 ID。|
|`room_type`|`string`|房间的`类型`（来自 [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api/#mroomcreate)），如果有。<br><br>**在 `v1.4` 中添加**|
|`topic`|`string`|房间的主题（如果有）。|
|`world_readable`|`boolean`|**必需：** 客户端用户是否可以在不加入的情况下查看房间。|

|SpaceHierarchyParentRoom|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`allowed_room_ids`|`[string]`|如果房间是[受限房间](https://spec.matrix.org/v1.11/server-server-api/#restricted-rooms)，这些是加入规则中指定的房间 ID。否则为空或省略。|
|`avatar_url`|[URI](http://tools.ietf.org/html/rfc3986)|房间头像的 URL（如果设置了）。|
|`canonical_alias`|`string`|房间的规范别名（如果有）。|
|`children_state`|[[StrippedStateEvent](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1hierarchyroomid_response-200_strippedstateevent)]|**必需：**<br><br>空间房间的 [`m.space.child`](https://spec.matrix.org/v1.11/client-server-api/#mspacechild) 事件，表示为[剥离状态事件](https://spec.matrix.org/v1.11/client-server-api/#stripped-state)，并添加了 `origin_server_ts` 键。<br><br>如果房间不是空间房间，这应该是空的。|
|`guest_can_join`|`boolean`|**必需：** 客户端用户是否可以加入房间并参与其中。如果可以，他们将像其他用户一样遵循普通权限级别规则。|
|`join_rule`|`string`|房间的加入规则。如果不存在，房间被假定为`public`。|
|`name`|`string`|房间的名称（如果有）。|
|`num_joined_members`|`integer`|**必需：** 加入房间的成员数量。|
|`room_id`|`string`|**必需：** 房间的 ID。|
|`room_type`|`string`|房间的`类型`（来自 [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api/#mroomcreate)），如果有。<br><br>**在 `v1.4` 中添加**|
|`topic`|`string`|房间的主题（如果有）。|
|`world_readable`|`boolean`|**必需：** 客户端用户是否可以在不加入的情况下查看房间。|

|StrippedStateEvent|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|`EventContent`|**必需：** 事件的`内容`。|
|`origin_server_ts`|`integer`|**必需：** 事件的`origin_server_ts`。|
|`sender`|`string`|**必需：** 事件的`发送者`。|
|`state_key`|`string`|**必需：** 事件的`state_key`。|
|`type`|`string`|**必需：** 事件的`类型`。|

```json
{
  "children": [
    {
      "allowed_room_ids": [
        "!upstream:example.org"
      ],
      "avatar_url": "mxc://example.org/abcdef2",
      "canonical_alias": "#general:example.org",
      "children_state": [
        {
          "content": {
            "via": [
              "remote.example.org"
            ]
          },
          "origin_server_ts": 1629422222222,
          "sender": "@alice:example.org",
          "state_key": "!b:example.org",
          "type": "m.space.child"
        }
      ],
      "guest_can_join": false,
      "join_rule": "restricted",
      "name": "The ~~First~~ Second Space",
      "num_joined_members": 42,
      "room_id": "!second_room:example.org",
      "room_type": "m.space",
      "topic": "Hello world",
      "world_readable": true
    }
  ],
  "inaccessible_children": [
    "!secret:example.org"
  ],
  "room": {
    "allowed_room_ids": [],
    "avatar_url": "mxc://example.org/abcdef",
    "canonical_alias": "#general:example.org",
    "children_state": [
      {
        "content": {
          "via": [
            "remote.example.org"
          ]
        },
        "origin_server_ts": 1629413349153,
        "sender": "@alice:example.org",
        "state_key": "!a:example.org",
        "type": "m.space.child"
      }
    ],
    "guest_can_join": false,
    "join_rule": "public",
    "name": "The First Space",
    "num_joined_members": 42,
    "room_id": "!space:example.org",
    "room_type": "m.space",
    "topic": "No other spaces were created first, ever",
    "world_readable": true
  }
}
```

##### 404 响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "Room does not exist."
}
```

# 输入通知[](https://spec.matrix.org/v1.11/server-server-api/#typing-notifications)

当服务器的用户发送输入通知时，这些通知需要发送到房间中的其他服务器，以便他们的用户了解相同的状态。接收服务器应验证用户是否在房间中，并且是属于发送服务器的用户。

## `m.typing`

---

房间中用户的输入通知 EDU。

|m.typing|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[输入通知](https://spec.matrix.org/v1.11/server-server-api/#definition-mtyping_typing-notification)|**必需：** 输入通知。|
|`edu_type`|`string`|**必需：** 字符串 `m.typing`<br><br>之一：`[m.typing]`。|

|输入通知|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`room_id`|`string`|**必需：** 用户输入状态已更新的房间。|
|`typing`|`boolean`|**必需：** 用户是否在房间中输入。|
|`user_id`|`string`|**必需：** 输入状态已更改的用户 ID。|

### 示例

```json
{
  "content": {
    "room_id": "!somewhere:matrix.org",
    "typing": true,
    "user_id": "@john:matrix.org"
  },
  "edu_type": "m.typing"
}
```

# 在线状态[](https://spec.matrix.org/v1.11/server-server-api/#presence)

服务器 API 的在线状态完全基于以下 EDU 的交换。没有涉及 PDU 或联邦查询。

服务器应仅发送接收服务器可能感兴趣的用户的在线状态更新。例如，接收服务器与给定用户共享一个房间。

## `m.presence`

---

表示发送主服务器用户的在线状态更新的 EDU。

|m.presence|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[在线状态更新](https://spec.matrix.org/v1.11/server-server-api/#definition-mpresence_presence-update)|**必需：** 在线状态更新和请求。|
|`edu_type`|`string`|**必需：** 字符串 `m.presence`<br><br>之一：`[m.presence]`。|

|在线状态更新|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`push`|[[用户在线状态更新](https://spec.matrix.org/v1.11/server-server-api/#definition-mpresence_user-presence-update)]|**必需：** 接收服务器可能感兴趣的在线状态更新列表。|

|用户在线状态更新|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`currently_active`|`boolean`|如果用户可能正在与其客户端交互，则为真。这可能通过用户在最近几分钟内有 `last_active_ago` 来指示。默认为 false。|
|`last_active_ago`|`integer`|**必需：** 自用户上次执行操作以来经过的毫秒数。|
|`presence`|`string`|**必需：** 用户的在线状态。<br><br>之一：`[offline, unavailable, online]`。|
|`status_msg`|`string`|可选的描述以伴随在线状态。|
|`user_id`|`string`|**必需：** 此在线状态 EDU 所属的用户 ID。|

### 示例

```json
{
  "content": {
    "push": [
      {
        "currently_active": true,
        "last_active_ago": 5000,
        "presence": "online",
        "status_msg": "Making cupcakes",
        "user_id": "@john:matrix.org"
      }
    ]
  },
  "edu_type": "m.presence"
}
```

# 回执[](https://spec.matrix.org/v1.11/server-server-api/#receipts)

回执是用于为给定事件传达标记的 EDU。目前支持的唯一回执类型是“已读回执”，即用户已阅读到事件图中的哪个位置。

不需要发送用户发送的事件的已读回执。通过发送事件，暗示用户已阅读到该事件。

## `m.receipt`

---

表示发送主服务器用户的回执更新的 EDU。接收回执时，服务器应仅更新 EDU 中列出的条目。之前收到的回执未出现在 EDU 中的，不应删除或以其他方式操作。

|m.receipt|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|{[房间 ID](https://spec.matrix.org/v1.11/appendices#room-ids): [房间回执](https://spec.matrix.org/v1.11/server-server-api/#definition-mreceipt_room-receipts)}|**必需：** 特定房间的回执。字符串键是其下回执所属的房间 ID。|
|`edu_type`|`string`|**必需：** 字符串 `m.receipt`<br><br>之一：`[m.receipt]`。|

|房间回执|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`m.read`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): [用户已读回执](https://spec.matrix.org/v1.11/server-server-api/#definition-mreceipt_user-read-receipt)}|**必需：** 房间中用户的已读回执。字符串键是回执所属的用户 ID。|

|用户已读回执|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`data`|[已读回执元数据](https://spec.matrix.org/v1.11/server-server-api/#definition-mreceipt_read-receipt-metadata)|**必需：** 已读回执的元数据。|
|`event_ids`|`[string]`|**必需：** 用户已阅读到的极端事件 ID。|

|已读回执元数据|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`thread_id`|`string`|此回执意图属于的线程的根线程事件的 ID（或 `main`）。如果未指定，已读回执为_未线程化_（默认）。<br><br>**在 `v1.4` 中添加**|
|`ts`|`integer`|**必需：** 用户阅读已读回执中指定事件的 POSIX 时间戳（毫秒）。|

### 示例

```json
{
  "content": {
    "!some_room:example.org": {
      "m.read": {
        "@john:matrix.org": {
          "data": {
            "ts": 1533358089009
          },
          "event_ids": [
            "$read_this_event:matrix.org"
          ]
        }
      }
    }
  },
  "edu_type": "m.receipt"
}
```

# 信息查询[](https://spec.matrix.org/v1.11/server-server-api/#querying-for-information)

查询是一种从主服务器检索有关资源（如用户或房间）信息的方法。此处的端点通常与客户端在客户端-服务器 API 上的请求结合使用，以完成调用。

可以进行多种类型的查询。首先描述了表示所有查询的通用端点，然后是可以进行的更具体的查询。

### GET /_matrix/federation/v1/query/directory[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1querydirectory)

---

执行查询以获取映射的房间 ID 和房间中居民主服务器的列表，针对给定的房间别名。主服务器应仅查询属于目标服务器（由别名中的 DNS 名称标识）的房间别名。

服务器可能希望缓存此查询的响应，以避免过于频繁地请求信息。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`room_alias`|`string`|**必需：** 要查询的房间别名。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|房间的对应房间 ID 和已知居民主服务器列表。|
|`404`|未找到房间别名。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`room_id`|`string`|**必需：** 映射到查询房间别名的房间 ID。|
|`servers`|`[string]`|**必需：** 可能持有给定房间的服务器名称数组。此列表可能包括或不包括回答查询的服务器。|

```json
{
  "room_id": "!roomid1234:example.org",
  "servers": [
    "example.org",
    "example.com",
    "another.example.com:8449"
  ]
}
```

##### 404 响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "Room alias not found."
}
```

### GET /_matrix/federation/v1/query/profile[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1queryprofile)

---

执行查询以获取给定用户的个人资料信息，例如显示名称或头像。主服务器应仅查询属于目标服务器（由用户 ID 中的 DNS 名称标识）的用户的个人资料。

服务器可能希望缓存此查询的响应，以避免过于频繁地请求信息。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`field`|`string`|要查询的字段。如果指定，服务器将仅在响应中返回给定字段。如果未指定，服务器将返回用户的完整个人资料。<br><br>之一：`[displayname, avatar_url]`。|
|`user_id`|`string`|**必需：** 要查询的用户 ID。必须是接收主服务器本地的用户。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|用户的个人资料。如果请求中指定了 `field`，则响应中应仅包含匹配的字段。如果未指定 `field`，响应应包括用户个人资料中可以公开的字段，例如显示名称和头像。<br><br>如果用户的个人资料中没有设置特定字段，服务器应将其排除在响应体之外或将其值设为 `null`。|
|`404`|用户不存在或没有个人资料。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`avatar_url`|`string`|用户头像的 URL。如果用户没有设置头像，可能会省略。|
|`displayname`|`string`|用户的显示名称。如果用户没有设置显示名称，可能会省略。|

```json
{
  "avatar_url": "mxc://matrix.org/MyC00lAvatar",
  "displayname": "John Doe"
}
```

##### 404 响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_NOT_FOUND",
  "error": "User does not exist."
}
```

### GET /_matrix/federation/v1/query/{queryType}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1queryquerytype)

---

在接收主服务器上执行单个查询请求。查询字符串参数取决于正在进行的查询类型。已知的查询类型被指定为此定义的扩展的独立端点。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`queryType`|`string`|**必需：** 要进行的查询类型|

---

#### 响应

|状态|描述|
|---|---|
|`200`|查询响应。模式根据正在进行的查询而有所不同。|

# OpenID[](https://spec.matrix.org/v1.11/server-server-api/#openid)

第三方服务可以交换由客户端-服务器 API 先前生成的访问令牌，以获取有关用户的信息。这可以帮助验证用户的身份，而无需授予对用户帐户的完全访问权限。

由 OpenID API 生成的访问令牌仅适用于 OpenID API，不能用于其他用途。

### GET /_matrix/federation/v1/openid/userinfo[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1openiduserinfo)

---

交换 OpenID 访问令牌以获取生成令牌的用户的信息。目前，这仅公开令牌所有者的 Matrix 用户 ID。

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
|`access_token`|`string`|**必需：** 要获取所有者信息的 OpenID 访问令牌。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|生成 OpenID 访问令牌的用户的信息。|
|`401`|令牌未被识别或已过期。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`sub`|`string`|**必需：** 生成令牌的 Matrix 用户 ID。|

```json
{
  "sub": "@alice:example.com"
}
```

##### 401 响应

|错误|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_UNKNOWN_TOKEN",
  "error": "Access token unknown or expired"
}
```

# 设备管理[](https://spec.matrix.org/v1.11/server-server-api/#device-management)

用户设备的详细信息必须有效地发布给其他用户并保持最新。这对于可靠的端到端加密至关重要，以便用户知道哪些设备参与了房间。它也是设备间消息传递正常工作的必要条件。本节旨在补充客户端-服务器 API 的[设备管理模块](https://spec.matrix.org/v1.11/client-server-api#device-management)。

Matrix 目前使用自定义的发布订阅系统，通过联邦同步给定用户的设备列表信息。当服务器希望首次确定远程用户的设备列表时，它应通过远程服务器上的 `/user/keys/query` API 的结果填充本地缓存。然而，缓存的后续更新应通过消费 `m.device_list_update` EDU 来应用。每个新的 `m.device_list_update` EDU 描述了给定用户的一个设备的增量更改，应替换本地服务器设备列表缓存中的任何现有条目。服务器必须将 `m.device_list_update` EDU 发送给与给定本地用户共享房间的所有服务器，并且必须在该用户的设备列表更改时发送（即对于新设备或已删除设备，当该用户加入包含尚未接收该用户设备列表更新的服务器的房间时，或设备信息（如设备的人类可读名称）发生变化时）。

服务器按原始用户发送 `m.device_list_update` EDU 序列，每个都有一个唯一的 `stream_id`。它们还包括一个指向此更新相对于的最近的 EDU 的指针在 `prev_id` 字段中。为了简化可能同时发送多个 EDU 的集群服务器的实现，`prev_id` 字段应包括所有尚未在 EDU 中引用的 `m.device_list_update` EDU。如果 EDU 是由服务器串行发出的，EDU 中应该只有一个 `prev_id`。

这形成了一个简单的 `m.device_list_update` EDU 有向无环图，显示服务器需要接收到哪些 EDU 才能将更新应用到其远程用户设备列表的本地副本。如果服务器接收到一个它不识别的 `prev_id` 的 EDU，它必须通过调用 `/user/keys/query API` 重新同步其列表并恢复该过程。响应包含一个 `stream_id`，应与后续 `m.device_list_update` EDU 相关联。

### GET /_matrix/federation/v1/user/devices/{userId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid)

---

获取用户所有设备的信息

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`userId`|`string`|**必需：** 要检索设备的用户 ID。必须是接收主服务器本地的用户。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|用户的设备。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`devices`|[[用户设备](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid_response-200_user-device)]|**必需：** 用户的设备。可能为空。|
|`master_key`|[CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid_response-200_crosssigningkey)|用户的主交叉签名密钥。|
|`self_signing_key`|[CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid_response-200_crosssigningkey)|用户的自签名密钥。|
|`stream_id`|`integer`|**必需：** 描述返回设备列表版本的给定 user_id 的唯一 ID。与 `m.device_list_update` EDU 中的 `stream_id` 字段匹配，以增量更新返回的 device_list。|
|`user_id`|`string`|**必需：** 请求设备的用户 ID。|

|用户设备|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`device_display_name`|`string`|设备的可选显示名称。|
|`device_id`|`string`|**必需：** 设备 ID。|
|`keys`|[DeviceKeys](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1userdevicesuserid_response-200_devicekeys)|**必需：** 设备的身份密钥。|

|DeviceKeys|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`algorithms`|`[string]`|**必需：** 此设备支持的加密算法。|
|`device_id`|`string`|**必需：** 这些密钥所属设备的 ID。必须与登录时使用的设备 ID 匹配。|
|`keys`|`{string: string}`|**必需：** 公共身份密钥。属性名称应采用 `<algorithm>:<device_id>` 格式。密钥本身应按密钥算法指定的方式编码。|
|`signatures`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: string}}|**必需：**<br><br>设备密钥对象的签名。一个从用户 ID 到 `<algorithm>:<device_id>` 到签名的映射。<br><br>签名是使用[签名 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算的。|
|`user_id`|`string`|**必需：** 设备所属用户的 ID。必须与登录时使用的用户 ID 匹配。|

|CrossSigningKey|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`keys`|`{string: string}`|**必需：** 公钥。对象必须只有一个属性，其名称为 `<algorithm>:<unpadded_base64_public_key>`，其值为未填充的 base64 公钥。|
|`signatures`|`Signatures`|密钥的签名，使用[签名 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算。主密钥可选。其他密钥必须由用户的主密钥签名。|
|`usage`|`[string]`|**必需：** 密钥的用途。|
|`user_id`|`string`|**必需：** 密钥所属用户的 ID。|

```json
{
  "devices": [
    {
      "device_display_name": "Alice's Mobile Phone",
      "device_id": "JLAFKJWSCS",
      "keys": {
        "algorithms": [
          "m.olm.v1.curve25519-aes-sha2",
          "m.megolm.v1.aes-sha2"
        ],
        "device_id": "JLAFKJWSCS",
        "keys": {
          "curve25519:JLAFKJWSCS": "3C5BFWi2Y8MaVvjM8M22DBmh24PmgR0nPvJOIArzgyI",
          "ed25519:JLAFKJWSCS": "lEuiRJBit0IG6nUf5pUzWTUEsRVVe/HJkoKuEww9ULI"
        },
        "signatures": {
          "@alice:example.com": {
            "ed25519:JLAFKJWSCS": "dSO80A01XiigH3uBiDVx/EjzaoycHcjq9lfQX0uWsqxl2giMIiSPR8a4d291W1ihKJL/a+myXS367WT6NAIcBA"
          }
        },
        "user_id": "@alice:example.com"
      }
    }
  ],
  "master_key": {
    "keys": {
      "ed25519:base64+master+public+key": "base64+master+public+key"
    },
    "usage": [
      "master"
    ],
    "user_id": "@alice:example.com"
  },
  "self_signing_key": {
    "keys": {
      "ed25519:base64+self+signing+public+key": "base64+self+signing+master+public+key"
    },
    "signatures": {
      "@alice:example.com": {
        "ed25519:base64+master+public+key": "signature+of+self+signing+key"
      }
    },
    "usage": [
      "self_signing"
    ],
    "user_id": "@alice:example.com"
  },
  "stream_id": 5,
  "user_id": "@alice:example.org"
}
```

## `m.device_list_update`

---

**在 `v1.1` 中添加**

一个 EDU，让服务器在其用户向其帐户添加新设备时相互推送详细信息，以便 E2E 加密正确定位给定用户的当前设备集。当现有设备获得新的交叉签名签名时，也会发送此事件。

|m.device_list_update|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[设备列表更新](https://spec.matrix.org/v1.11/server-server-api/#definition-mdevice_list_update_device-list-update)|**必需：** 详细信息已更改的设备的描述。|
|`edu_type`|`string`|**必需：** 字符串 `m.device_list_update`。<br><br>之一：`[m.device_list_update]`。|

|设备列表更新|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`deleted`|`boolean`|如果服务器宣布此设备已被删除，则为真。|
|`device_display_name`|`string`|此设备的公共人类可读名称。如果设备没有名称，将不存在。|
|`device_id`|`string`|**必需：** 详细信息正在更改的设备的 ID。|
|`keys`|[DeviceKeys](https://spec.matrix.org/v1.11/server-server-api/#definition-mdevice_list_update_devicekeys)|此设备的更新身份密钥（如果有）。如果设备没有定义 E2E 密钥，可能会缺失。|
|`prev_id`|`[integer]`|为此用户发送的任何先前 m.device_list_update EDU 的 stream_ids，这些 EDU 尚未在 EDU 的 prev_id 字段中引用。如果接收服务器不识别任何 prev_id，则意味着 EDU 已丢失，服务器应通过 `/user/keys/query` 查询设备列表的快照，以便正确解释未来的 `m.device_list_update` EDU。对于序列中的第一个 EDU，可能会缺失或为空。|
|`stream_id`|`integer`|**必需：** 服务器为此更新发送的 ID，对于给定 user_id 唯一。用于识别服务器广播的 `m.device_list_update` EDU 序列中的任何间隙。|
|`user_id`|`string`|**必需：** 拥有此设备的用户 ID。|

|DeviceKeys|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`algorithms`|`[string]`|**必需：** 此设备支持的加密算法。|
|`device_id`|`string`|**必需：** 这些密钥所属设备的 ID。必须与登录时使用的设备 ID 匹配。|
|`keys`|`{string: string}`|**必需：** 公共身份密钥。属性名称应采用 `<algorithm>:<device_id>` 格式。密钥本身应按密钥算法指定的方式编码。|
|`signatures`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: string}}|**必需：**<br><br>设备密钥对象的签名。一个从用户 ID 到 `<algorithm>:<device_id>` 到签名的映射。<br><br>签名是使用[签名 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算的。|
|`user_id`|`string`|**必需：** 设备所属用户的 ID。必须与登录时使用的用户 ID 匹配。|

### 示例

```json
{
  "content": {
    "device_display_name": "Mobile",
    "device_id": "QBUAZIFURK",
    "keys": {
      "algorithms": [
        "m.olm.v1.curve25519-aes-sha2",
        "m.megolm.v1.aes-sha2"
      ],
      "device_id": "JLAFKJWSCS",
      "keys": {
        "curve25519:JLAFKJWSCS": "3C5BFWi2Y8MaVvjM8M22DBmh24PmgR0nPvJOIArzgyI",
        "ed25519:JLAFKJWSCS": "lEuiRJBit0IG6nUf5pUzWTUEsRVVe/HJkoKuEww9ULI"
      },
      "signatures": {
        "@alice:example.com": {
          "ed25519:JLAFKJWSCS": "dSO80A01XiigH3uBiDVx/EjzaoycHcjq9lfQX0uWsqxl2giMIiSPR8a4d291W1ihKJL/a+myXS367WT6NAIcBA"
        }
      },
      "user_id": "@alice:example.com"
    },
    "prev_id": [
      5
    ],
    "stream_id": 6,
    "user_id": "@john:example.com"
  },
  "edu_type": "m.device_list_update"
}
```

# 端到端加密[](https://spec.matrix.org/v1.11/server-server-api/#end-to-end-encryption)

本节补充了客户端-服务器 API 的[端到端加密模块](https://spec.matrix.org/v1.11/client-server-api#end-to-end-encryption)。有关端到端加密的详细信息，请参阅该模块。

此处定义的 API 旨在能够将大部分客户端请求代理到联邦，并将响应也代理到客户端。

### POST /_matrix/federation/v1/user/keys/claim[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysclaim)

---

声明一次性密钥以用于预密钥消息。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求体

|名称|类型|描述|
|---|---|---|
|`one_time_keys`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: string}}|**必需：** 要声明的密钥。一个从用户 ID 到设备 ID 到算法名称的映射。请求的用户必须是接收主服务器本地的。|

##### 请求体示例

```json
{
  "one_time_keys": {
    "@alice:example.com": {
      "JLAFKJWSCS": "signed_curve25519"
    }
  }
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|声明的密钥。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`one_time_keys`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: {string: string\|[KeyObject](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysclaim_response-200_keyobject)}}}|**必需：**<br><br>查询设备的一次性密钥。一个从用户 ID 到设备到 `<algorithm>:<key_id>` 到密钥对象的映射。<br><br>有关密钥对象格式的更多信息，请参阅[客户端-服务器密钥算法](https://spec.matrix.org/v1.11/client-server-api/#key-algorithms)部分。|

|KeyObject|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`key`|`string`|**必需：** 使用未填充的 base64 编码的密钥。|
|`signatures`|`{string: {string: string}}`|**必需：**<br><br>密钥对象的签名。<br><br>签名是使用[签名 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算的。|

```json
{
  "one_time_keys": {
    "@alice:example.com": {
      "JLAFKJWSCS": {
        "signed_curve25519:AAAAHg": {
          "key": "zKbLg+NrIjpnagy+pIY6uPL4ZwEG2v+8F9lmgsnlZzs",
          "signatures": {
            "@alice:example.com": {
              "ed25519:JLAFKJWSCS": "FLWxXqGbwrb8SM3Y795eB6OA8bwBcoMZFXBqnTn58AYWZSqiD45tlBVcDa2L7RwdKXebW/VzDlnfVJ+9jok1Bw"
            }
          }
        }
      }
    }
  }
}
```

### POST /_matrix/federation/v1/user/keys/query[](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery)

---

返回给定用户的当前设备和身份密钥。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|是|

---

#### 请求

##### 请求体

|名称|类型|描述|
|---|---|---|
|`device_keys`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): [string]}|**必需：** 要下载的密钥。一个从用户 ID 到设备 ID 列表或为空列表以指示对应用户的所有设备的映射。请求的用户必须是接收主服务器本地的。|

##### 请求体示例

```json
{
  "device_keys": {
    "@alice:example.com": []
  }
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|设备信息。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`device_keys`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: [DeviceKeys](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery_response-200_devicekeys)}}|**必需：** 查询设备的信息。一个从用户 ID 到设备 ID 到设备信息的映射。对于每个设备，返回的信息将与通过 `/keys/upload` 上传的信息相同，并添加一个 `unsigned` 属性。|
|`master_keys`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): [CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery_response-200_crosssigningkey)}|查询用户的主交叉签名密钥的信息。一个从用户 ID 到主密钥信息的映射。对于每个密钥，返回的信息将与通过 `/keys/device_signing/upload` 上传的信息相同，以及用户允许看到的通过 `/keys/signatures/upload` 上传的签名。<br><br>**在 `v1.1` 中添加**|
|`self_signing_keys`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): [CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery_response-200_crosssigningkey)}|查询用户的自签名密钥的信息。一个从用户 ID 到自签名密钥信息的映射。对于每个密钥，返回的信息将与通过 `/keys/device_signing/upload` 上传的信息相同。<br><br>**在 `v1.1` 中添加**|

|DeviceKeys|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`algorithms`|`[string]`|**必需：** 此设备支持的加密算法。|
|`device_id`|`string`|**必需：** 这些密钥所属设备的 ID。必须与登录时使用的设备 ID 匹配。|
|`keys`|`{string: string}`|**必需：** 公共身份密钥。属性名称应采用 `<algorithm>:<device_id>` 格式。密钥本身应按密钥算法指定的方式编码。|
|`signatures`|{[用户 ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: string}}|**必需：**<br><br>设备密钥对象的签名。一个从用户 ID 到 `<algorithm>:<device_id>` 到签名的映射。<br><br>签名是使用[签名 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算的。|
|`unsigned`|[UnsignedDeviceInfo](https://spec.matrix.org/v1.11/server-server-api/#post_matrixfederationv1userkeysquery_response-200_unsigneddeviceinfo)|由中间服务器添加到设备密钥信息的附加数据，不受签名覆盖。|
|`user_id`|`string`|**必需：** 设备所属用户的 ID。必须与登录时使用的用户 ID 匹配。|

|UnsignedDeviceInfo|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`device_display_name`|`string`|用户在设备上设置的显示名称。|

|CrossSigningKey|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`keys`|`{string: string}`|**必需：** 公钥。对象必须只有一个属性，其名称为 `<algorithm>:<unpadded_base64_public_key>`，其值为未填充的 base64 公钥。|
|`signatures`|`Signatures`|密钥的签名，使用[签名 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算。主密钥可选。其他密钥必须由用户的主密钥签名。|
|`usage`|`[string]`|**必需：** 密钥的用途。|
|`user_id`|`string`|**必需：** 密钥所属用户的 ID。|

```json
{
  "device_keys": {
    "@alice:example.com": {
      "JLAFKJWSCS": {
        "algorithms": [
          "m.olm.v1.curve25519-aes-sha2",
          "m.megolm.v1.aes-sha2"
        ],
        "device_id": "JLAFKJWSCS",
        "keys": {
          "curve25519:JLAFKJWSCS": "3C5BFWi2Y8MaVvjM8M22DBmh24PmgR0nPvJOIArzgyI",
          "ed25519:JLAFKJWSCS": "lEuiRJBit0IG6nUf5pUzWTUEsRVVe/HJkoKuEww9ULI"
        },
        "signatures": {
          "@alice:example.com": {
            "ed25519:JLAFKJWSCS": "dSO80A01XiigH3uBiDVx/EjzaoycHcjq9lfQX0uWsqxl2giMIiSPR8a4d291W1ihKJL/a+myXS367WT6NAIcBA"
          }
        },
        "unsigned": {
          "device_display_name": "Alice's mobile phone"
        },
        "user_id": "@alice:example.com"
      }
    }
  }
}
```

## `m.signing_key_update`

---

**在 `v1.1` 中添加**

一个 EDU，让服务器在其用户更新其交叉签名密钥时相互推送详细信息。

|m.signing_key_update|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[签名密钥更新](https://spec.matrix.org/v1.11/server-server-api/#definition-msigning_key_update_signing-key-update)|**必需：** 更新的签名密钥。|
|`edu_type`|`string`|**必需：** 字符串 `m.signing_update`。<br><br>之一：`[m.signing_key_update]`。|

|签名密钥更新|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`master_key`|[CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#definition-msigning_key_update_crosssigningkey)|交叉签名密钥|
|`self_signing_key`|[CrossSigningKey](https://spec.matrix.org/v1.11/server-server-api/#definition-msigning_key_update_crosssigningkey)|交叉签名密钥|
|`user_id`|`string`|**必需：** 交叉签名密钥已更改的用户 ID。|

|CrossSigningKey|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`keys`|`{string: string}`|**必需：** 公钥。对象必须只有一个属性，其名称为 `<algorithm>:<unpadded_base64_public_key>`，其值为未填充的 base64 公钥。|
|`signatures`|`Signatures`|密钥的签名，使用[签名 JSON](https://spec.matrix.org/v1.11/appendices/#signing-json)中描述的过程计算。主密钥可选。其他密钥必须由用户的主密钥签名。|
|`usage`|`[string]`|**必需：** 密钥的用途。|
|`user_id`|`string`|**必需：** 密钥所属用户的 ID。|

### 示例

```json
{
  "content": {
    "master_key": {
      "keys": {
        "ed25519:base64+master+public+key": "base64+master+public+key"
      },
      "usage": [
        "master"
      ],
      "user_id": "@alice:example.com"
    },
    "self_signing_key": {
      "keys": {
        "ed25519:base64+self+signing+public+key": "base64+self+signing+master+public+key"
      },
      "signatures": {
        "@alice:example.com": {
          "ed25519:base64+master+public+key": "signature+of+self+signing+key"
        }
      },
      "usage": [
        "self_signing"
      ],
      "user_id": "@alice:example.com"
    },
    "user_id": "@alice:example.com"
  },
  "edu_type": "m.signing_key_update"
}
```

# 设备间消息发送[](https://spec.matrix.org/v1.11/server-server-api/#send-to-device-messaging)

设备间消息发送的服务器API基于`m.direct_to_device` EDU。不涉及PDU或联邦查询。

每个设备间消息应使用以下EDU发送到目标服务器：

## `m.direct_to_device`

---

一种EDU，允许服务器将发送事件直接推送到远程服务器上的特定设备，例如，维护本地设备和远程设备之间的Olm E2E加密消息通道。

|m.direct_to_device|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`content`|[设备消息](https://spec.matrix.org/v1.11/server-server-api/#definition-mdirect_to_device_to-device-message)|**必需：** 设备间消息的描述。|
|`edu_type`|`string`|**必需：** 字符串 `m.direct_to_device`。<br><br>之一：`[m.direct_to_device]`。|

|设备消息|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`message_id`|`string`|**必需：** 消息的唯一ID，用于幂等性。任意utf8字符串，最大长度为32个代码点。|
|`messages`|{[用户ID](https://spec.matrix.org/v1.11/appendices#user-identifiers): {string: 设备消息内容}}|**必需：** 要发送的消息内容。这些内容以用户ID映射到设备ID映射到消息主体的形式排列。设备ID也可以是`*`，表示用户的所有已知设备。|
|`sender`|`string`|**必需：** 发送者的用户ID。|
|`type`|`string`|**必需：** 消息的事件类型。|

### 示例

```json
{
  "content": {
    "message_id": "hiezohf6Hoo7kaev",
    "messages": {
      "@alice:example.org": {
        "IWHQUZUIAH": {
          "algorithm": "m.megolm.v1.aes-sha2",
          "room_id": "!Cuyf34gef24t:localhost",
          "session_id": "X3lUlvLELLYxeTx4yOVu6UDpasGEVO0Jbu+QFnm0cKQ",
          "session_key": "AgAAAADxKHa9uFxcXzwYoNueL5Xqi69IkD4sni8LlfJL7qNBEY..."
        }
      }
    },
    "sender": "@john:example.com",
    "type": "m.room_key_request"
  },
  "edu_type": "m.direct_to_device"
}
```

# 内容库[](https://spec.matrix.org/v1.11/server-server-api/#content-repository)

事件的附件（图像、文件等）通过[客户端-服务器API](https://spec.matrix.org/v1.11/client-server-api/#content-repository)中描述的内容库上传到主服务器。当服务器希望提供来自远程服务器的内容时，需要向远程服务器请求媒体。

服务器必须使用[Matrix内容URI](https://spec.matrix.org/v1.11/client-server-api/#matrix-content-mxc-uris)中描述的服务器。格式为`mxc://{ServerName}/{MediaID}`，服务器必须使用以下端点从`ServerName`下载媒体。

> [!info] 原因：
> **[在`v1.11`中更改]** 之前建议服务器使用[客户端-服务器API中的内容库模块](https://spec.matrix.org/v1.11/client-server-api/#content-repository)描述的`/_matrix/media/*`端点，但这些端点已被弃用。引入了需要身份验证的新端点。自然地，由于服务器不是用户，它们无法向这些端点提供所需的访问令牌。相反，服务器必须在收到`404 M_UNRECOGNIZED`错误时尝试下面描述的端点，然后再回退到已弃用的`/_matrix/media/*`端点。回退时，服务器必须确保将`allow_remote`设置为`false`。

### GET /_matrix/federation/v1/media/download/{mediaId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1mediadownloadmediaid)

---

**在`v1.11`中添加**

|   |   |
|---|---|
|速率限制：|是|
|需要身份验证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`mediaId`|`string`|**必需：** 来自`mxc://` URI的媒体ID（路径组件）。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`timeout_ms`|`integer`|客户端愿意等待开始接收数据的最大毫秒数，以防内容尚未上传。默认值为20000（20秒）。内容库应对该参数施加最大值。内容库可能在超时前响应。<br><br>**在`v1.7`中添加**|

---

#### 响应

|状态|描述|
|---|---|
|`200`|之前上传的内容。|
|`429`|此请求被速率限制。|
|`502`|内容太大，服务器无法提供。|
|`504`|内容尚不可用。将返回带有`errcode` `M_NOT_YET_UPLOADED`的[标准错误响应](https://spec.matrix.org/v1.11/client-server-api/#standard-error-response)。|

##### 200响应

|头|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`Content-Type`|`string`|必须是`multipart/mixed`。|

|Content-Type|描述|
|---|---|
|`multipart/mixed`|**必需。** 必须包含一个`boundary`（根据[RFC 2046](https://datatracker.ietf.org/doc/html/rfc2046#section-5.1)）分隔正好两个部分：<br><br>第一部分具有`Content-Type`头为`application/json`，描述媒体的元数据（如果有）。目前，这将始终是一个空对象。<br><br>第二部分是：<br><br>1. 媒体本身的字节，使用适当的`Content-Type`和`Content-Disposition`头；<br>    <br>2. 或者一个`Location`头，重定向调用者到可以检索媒体的位置。`Location`的URL应具有适当的`Content-Type`和`Content-Disposition`头，描述媒体。<br>    <br>    当`Location`存在时，服务器不应缓存URL。远程服务器可能对其有效性施加了时间限制。如果调用者需要最新的URL，应重新请求媒体下载。|

##### 429响应

|RateLimitError|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** M_LIMIT_EXCEEDED错误代码|
|`error`|`string`|人类可读的错误信息。|
|`retry_after_ms`|`integer`|客户端在再次尝试请求前应等待的毫秒数。|

```json
{
  "errcode": "M_LIMIT_EXCEEDED",
  "error": "Too many requests",
  "retry_after_ms": 2000
}
```

##### 502响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TOO_LARGE",
  "error": "Content is too large to serve"
}
```

##### 504响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_NOT_YET_UPLOADED",
  "error": "Content has not yet been uploaded"
}
```

### GET /_matrix/federation/v1/media/thumbnail/{mediaId}[](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1mediathumbnailmediaid)

---

**在`v1.11`中添加**

从内容库下载内容的缩略图。有关更多信息，请参见[客户端-服务器API缩略图](https://spec.matrix.org/v1.11/client-server-api/#thumbnails)部分。

|   |   |
|---|---|
|速率限制：|是|
|需要身份验证：|是|

---

#### 请求

##### 请求参数

|路径参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`mediaId`|`string`|**必需：** 来自`mxc://` URI的媒体ID（路径组件）。|

|查询参数|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`animated`|`boolean`|指示是否优先从服务器获取动画缩略图（如果可能）。动画缩略图通常使用内容类型`image/gif`、`image/png`（APNG格式）、`image/apng`和`image/webp`，而不是常见的静态`image/png`或`image/jpeg`内容类型。<br><br>当为`true`时，服务器应返回动画缩略图（如果可能且支持）。当为`false`时，服务器不得返回动画缩略图。例如，返回静态`image/png`或`image/jpeg`缩略图。当未提供时，服务器不应返回动画缩略图。<br><br>服务器在支持动画时应优先返回`image/webp`缩略图。<br><br>当为`true`且媒体无法动画化时，例如JPEG或PDF，服务器应表现得如同`animated`为`false`。<br><br>**在`v1.11`中添加**|
|`height`|`integer`|**必需：** 缩略图的_期望_高度。实际缩略图可能大于指定的大小。|
|`method`|`string`|期望的调整大小方法。有关更多信息，请参见[客户端-服务器API缩略图](https://spec.matrix.org/v1.11/client-server-api/#thumbnails)部分。<br><br>之一：`[crop, scale]`。|
|`timeout_ms`|`integer`|客户端愿意等待开始接收数据的最大毫秒数，以防内容尚未上传。默认值为20000（20秒）。内容库应对该参数施加最大值。内容库可能在超时前响应。<br><br>**在`v1.7`中添加**|
|`width`|`integer`|**必需：** 缩略图的_期望_宽度。实际缩略图可能大于指定的大小。|

---

#### 响应

|状态|描述|
|---|---|
|`200`|请求内容的缩略图。|
|`400`|请求对服务器没有意义，或服务器无法生成内容的缩略图。例如，调用者请求了非整数维度或要求负尺寸图像。|
|`413`|本地内容太大，服务器无法生成缩略图。|
|`429`|此请求被速率限制。|
|`502`|远程内容太大，服务器无法生成缩略图。|
|`504`|内容尚不可用。将返回带有`errcode` `M_NOT_YET_UPLOADED`的[标准错误响应](https://spec.matrix.org/v1.11/client-server-api/#standard-error-response)。|

##### 200响应

|头|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`Content-Type`|`string`|必须是`multipart/mixed`。|

|Content-Type|描述|
|---|---|
|`multipart/mixed`|**必需。** 必须包含一个`boundary`（根据[RFC 2046](https://datatracker.ietf.org/doc/html/rfc2046#section-5.1)）分隔正好两个部分：<br><br>第一部分具有`Content-Type`头为`application/json`，描述媒体的元数据（如果有）。目前，这将始终是一个空对象。<br><br>第二部分是：<br><br>1. 媒体本身的字节，使用适当的`Content-Type`和`Content-Disposition`头；<br>    <br>2. 或者一个`Location`头，重定向调用者到可以检索媒体的位置。`Location`的URL应具有适当的`Content-Type`和`Content-Disposition`头，描述媒体。<br>    <br>    当`Location`存在时，服务器不应缓存URL。远程服务器可能对其有效性施加了时间限制。如果调用者需要最新的URL，应重新请求媒体下载。<br>    <br><br>第二部分的`Content-Type`应为以下之一：<br><br>- `image/png`（可能是APNG类型）<br>- `image/apng`<br>- `image/jpeg`<br>- `image/gif`<br>- `image/webp`|

##### 400响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_UNKNOWN",
  "error": "Cannot generate thumbnails for the requested content"
}
```

##### 413响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TOO_LARGE",
  "error": "Content is too large to thumbnail"
}
```

##### 429响应

|RateLimitError|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** M_LIMIT_EXCEEDED错误代码|
|`error`|`string`|人类可读的错误信息。|
|`retry_after_ms`|`integer`|客户端在再次尝试请求前应等待的毫秒数。|

```json
{
  "errcode": "M_LIMIT_EXCEEDED",
  "error": "Too many requests",
  "retry_after_ms": 2000
}
```

##### 502响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_TOO_LARGE",
  "error": "Content is too large to thumbnail"
}
```

##### 504响应

|Error|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`errcode`|`string`|**必需：** 错误代码。|
|`error`|`string`|人类可读的错误信息。|

```json
{
  "errcode": "M_NOT_YET_UPLOADED",
  "error": "Content has not yet been uploaded"
}
```

# 服务器访问控制列表（ACLs）[](https://spec.matrix.org/v1.11/server-server-api/#server-access-control-lists-acls)

服务器ACL及其目的在客户端-服务器API的[服务器ACL](https://spec.matrix.org/v1.11/client-server-api#server-access-control-lists-acls-for-rooms)部分中描述。

当远程服务器发出请求时，必须验证其是否被服务器ACL允许。如果服务器被拒绝访问房间，接收服务器必须以403 HTTP状态码和`errcode`为`M_FORBIDDEN`进行回复。

以下端点前缀必须受到保护：

- `/_matrix/federation/v1/send`（基于每个PDU）
- `/_matrix/federation/v1/make_join`
- `/_matrix/federation/v1/make_leave`
- `/_matrix/federation/v1/send_join`
- `/_matrix/federation/v2/send_join`
- `/_matrix/federation/v1/send_leave`
- `/_matrix/federation/v2/send_leave`
- `/_matrix/federation/v1/invite`
- `/_matrix/federation/v2/invite`
- `/_matrix/federation/v1/make_knock`
- `/_matrix/federation/v1/send_knock`
- `/_matrix/federation/v1/state`
- `/_matrix/federation/v1/state_ids`
- `/_matrix/federation/v1/backfill`
- `/_matrix/federation/v1/event_auth`
- `/_matrix/federation/v1/get_missing_events`

# 签署事件[](https://spec.matrix.org/v1.11/server-server-api/#signing-events)

签署事件的复杂性在于服务器可以选择删除事件的非必要部分。

## 为传出事件添加哈希和签名[](https://spec.matrix.org/v1.11/server-server-api/#adding-hashes-and-signatures-to-outgoing-events)

在签署事件之前，事件的_内容哈希_按照如下描述进行计算。哈希使用[无填充Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64)编码，并存储在事件对象的`hashes`对象中，位于`sha256`键下。

然后事件对象被_编辑_，遵循[编辑算法](https://spec.matrix.org/v1.11/client-server-api#redactions)。最后，使用服务器的签名密钥（另见[检索服务器密钥](https://spec.matrix.org/v1.11/server-server-api/#retrieving-server-keys)）按照[签署JSON](https://spec.matrix.org/v1.11/appendices#signing-json)中描述的方式进行签名。

然后将签名复制回原始事件对象。

有关签名事件的示例，请参见[房间版本规范](https://spec.matrix.org/v1.11/rooms)。

## 验证接收事件上的哈希和签名[](https://spec.matrix.org/v1.11/server-server-api/#validating-hashes-and-signatures-on-received-events)

当服务器通过联邦从另一个服务器接收事件时，接收服务器应检查该事件的哈希和签名。

首先检查签名。事件按照[编辑算法](https://spec.matrix.org/v1.11/client-server-api#redactions)进行编辑，并按照[检查签名](https://spec.matrix.org/v1.11/appendices#checking-for-a-signature)中描述的算法检查来自原始服务器的签名。注意，无论我们收到完整事件还是编辑后的副本，这一步都应成功。

事件上预期的签名是：

- `sender`的服务器，除非邀请是第三方邀请的结果。发送者必须已经匹配第三方邀请，实际上发送事件的服务器可能是不同的服务器。
- 对于房间版本1和2，创建`event_id`的服务器。其他房间版本不在联邦中跟踪`event_id`，因此不需要这些服务器的签名。

如果签名被认为是有效的，则按照如下描述计算预期的内容哈希。接收到的事件的`hashes`属性中的内容哈希被base64解码，并将两者进行比较以确定是否相等。

如果哈希检查失败，则假定这是因为我们只得到了事件的编辑版本。为了强制执行这一点，接收服务器应使用其计算的编辑副本，而不是收到的完整副本。

## 计算事件的参考哈希[](https://spec.matrix.org/v1.11/server-server-api/#calculating-the-reference-hash-for-an-event)

事件的_参考哈希_涵盖事件的基本字段，包括内容哈希。它用于某些房间版本中的事件标识符。有关更多信息，请参见[房间版本规范](https://spec.matrix.org/v1.11/rooms)。其计算如下。

1. 事件通过编辑算法处理。
2. 从事件中移除`signatures`和`unsigned`属性（如果存在）。
3. 事件转换为[规范JSON](https://spec.matrix.org/v1.11/appendices#canonical-json)。
4. 对结果JSON对象计算sha256哈希。

## 计算事件的内容哈希[](https://spec.matrix.org/v1.11/server-server-api/#calculating-the-content-hash-for-an-event)

事件的_内容哈希_涵盖完整事件，包括_未编辑_的内容。其计算如下。

首先，移除任何现有的`unsigned`、`signature`和`hashes`成员。然后将结果对象编码为[规范JSON](https://spec.matrix.org/v1.11/appendices#canonical-json)，并使用SHA-256对JSON进行哈希。

## 示例代码[](https://spec.matrix.org/v1.11/server-server-api/#example-code)

```python
def hash_and_sign_event(event_object, signing_key, signing_name):
    # 首先我们需要对事件对象进行哈希。
    content_hash = compute_content_hash(event_object)
    event_object["hashes"] = {"sha256": encode_unpadded_base64(content_hash)}

    # 删除所有在事件被编辑时会被移除的键。
    # 哈希不会被删除，并覆盖事件中的所有键。
    # 这意味着我们可以判断任何非必要键是否被修改或删除。
    stripped_object = strip_non_essential_keys(event_object)

    # 签署编辑后的JSON对象。签名仅覆盖基本键和哈希。
    # 这意味着即使事件被编辑，我们也可以检查签名。
    signed_object = sign_json(stripped_object, signing_key, signing_name)

    # 将签名从编辑后的事件复制到原始事件。
    event_object["signatures"] = signed_object["signatures"]

def compute_content_hash(event_object):
    # 在我们移除任何键之前，复制事件。
    event_object = dict(event_object)

    # "unsigned"下的键可以被其他服务器修改。
    # 它们对于传递诸如事件年龄等信息很有用，这些信息在传输中会发生变化。
    # 由于它们可以被修改，我们需要将它们排除在哈希之外。
    event_object.pop("unsigned", None)

    # 签名将依赖于"hashes"键的当前值。
    # 我们不能添加新的哈希而不使现有签名无效。
    event_object.pop("signatures", None)

    # 如果我们决定迁移离开SHA-2，"hashes"键可能包含多个算法。
    # 我们不想在我们的哈希中包含现有的哈希输出，所以我们排除"hashes"字典。
    event_object.pop("hashes", None)

    # 使用规范编码对JSON进行编码，以便我们在每个服务器上对同一JSON对象获得相同的字节。
    event_json_bytes = encode_canonical_json(event_object)

    return hashlib.sha256(event_json_bytes)

```

# 安全注意事项[](https://spec.matrix.org/v1.11/server-server-api/#security-considerations)

当一个域名的所有权发生变化时，新域名控制者可以伪装成之前的所有者，接收消息（类似于电子邮件）并请求其他服务器的过去消息。在未来，像[MSC1228](https://github.com/matrix-org/matrix-spec-proposals/issues/1228)这样的提案将解决这个问题。
