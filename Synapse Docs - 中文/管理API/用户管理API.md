### 用户管理 API

要使用它，您需要通过提供一个 `access_token` 来进行身份验证以成为服务器管理员：请参见 Admin API。

#### 查询用户账户

此 API 返回有关特定用户账户的信息。

API 是：

```
GET /_synapse/admin/v2/users/<user_id>
```

它返回如下所示的 JSON 主体：

```json
{
    "name": "@user:example.com",
    "displayname": "User", // can be null if not set
    "threepids": [
        {
            "medium": "email",
            "address": "<user_mail_1>",
            "added_at": 1586458409743,
            "validated_at": 1586458409743
        },
        {
            "medium": "email",
            "address": "<user_mail_2>",
            "added_at": 1586458409743,
            "validated_at": 1586458409743
        }
    ],
    "avatar_url": "<avatar_url>",  // can be null if not set
    "is_guest": 0,
    "admin": 0,
    "deactivated": 0,
    "erased": false,
    "shadow_banned": 0,
    "creation_ts": 1560432506,
    "appservice_id": null,
    "consent_server_notice_sent": null,
    "consent_version": null,
    "consent_ts": null,
    "external_ids": [
        {
            "auth_provider": "<provider1>",
            "external_id": "<user_id_provider_1>"
        },
        {
            "auth_provider": "<provider2>",
            "external_id": "<user_id_provider_2>"
        }
    ],
    "user_type": null,
    "locked": false
}
```

URL 参数：
*   `user_id` : 完全合格的用户 ID：例如， `@user:server.com` .

#### 创建或修改账户

此 API 允许管理员创建或修改具有特定 `user_id` 的用户账户。

此 API 是：

```
PUT /_synapse/admin/v2/users/<user_id>
```

其主体为：

```json
{
    "password": "user_password",
    "logout_devices": false,
    "displayname": "Alice Marigold",
    "avatar_url": "mxc://example.com/abcde12345",
    "threepids": [
        {
            "medium": "email",
            "address": "alice@example.com"
        },
        {
            "medium": "email",
            "address": "alice@domain.org"
        }
    ],
    "external_ids": [
        {
            "auth_provider": "example",
            "external_id": "12345"
        },
        {
            "auth_provider": "example2",
            "external_id": "abc54321"
        }
    ],
    "admin": false,
    "deactivated": false,
    "user_type": null,
    "locked": false
}
```

返回 HTTP 状态码：
*   `201` - 当创建新的用户对象时。
*   `200` - 当用户被修改时。

URL 参数：
*   `user_id` - 一个完全限定的用户 ID。例如， `@user:server.com` 。

Body 参数：
*   `password` - 字符串，可选。如果提供，用户的密码将被更新，并且所有设备将被登出，除非 `logout_devices` 被设置为 `false` 。
*   `logout_devices` - 布尔值，可选，默认为 `true` 。如果设置为 `false` ，即使提供了 `password` ，设备也不会被登出。
*   `displayname` - 字符串，可选。如果设置为空字符串 ( `""` )，用户的显示名称将被移除。
*   `avatar_url` - 字符串，可选。必须是 MXC URI。如果设置为空字符串 ( `""` )，则移除用户的头像。
*   `threepids` - 数组，可选。如果提供，将用户的第三方 ID（电子邮件、手机号码）完全替换为给定的列表。数组中的每个项目都是一个包含以下字段的对象：
*   `medium` - 字符串，必需。第三方 ID 的类型，可以是 `email` 或 `msisdn` （电话号码）。
*   `address` - 字符串，必填。第三方 ID 本身，例如 `alice@example.com` 对于 `email` 或 `447470274584` （对于带有国家代码“44”的电话号码）和 `19254857364` （对于带有国家代码“1”的电话号码）对于 `msisdn` 。注意：如果通过此选项从用户中移除一个 threepid，Synapse 也会尝试从它知道有绑定关系的任何身份服务器中移除该 threepid。
*   `external_ids` - 数组，可选。允许设置外部身份提供者的标识符以进行 SSO（单点登录）。更多详情请参见配置手册中的 sso 和 oidc_providers 部分。
*   `auth_provider` - 字符串，必填。外部身份提供者的唯一内部 ID。与 homeserver 配置中的 `idp_id` 相同。如果使用 OIDC，此值应以 `oidc-` 为前缀。注意，如果提供的值不在 homeserver 配置中，不会引发错误。
*   `external_id` - 字符串，必填。外部身份提供者中用户的标识符。当用户登录到身份提供者时，这必须是他们映射到的唯一 ID。
*   `admin` - 布尔值，可选，默认为 `false` 。是否用户是主服务器管理员，授予他们访问 Admin API 的权限等。
*   `deactivated` - 布尔值，可选。如果未指定，停用状态将保持不变。

注意：
*   对于密码字段，没有严格检查其存在的必要性。可以有无密码的活跃用户，例如当配置了 OIDC 身份验证时。您必须自己检查在重新激活用户时是否需要密码。
*   如果配置选项 `password_config.localdb_enabled` 设置为 `false` ，则无法设置密码。用户的密码在账户停用时会被清除，因此这里需要设置一个新的密码。

注意：此 API 无法删除用户。有关停用和删除用户的更多详细信息，请参见停用账户。
*   `locked` - 布尔值，可选。如果未指定，锁定状态将保持不变。
*   `user_type` - 字符串或空值，可选。如果未提供，用户类型将不会更改。如果提供了 `null` ，用户类型将被清除。其他允许的选项有： `bot` 和 `support` 。

#### 列出账户
##### 列出账户 (V2)

此 API 返回所有本地用户账户。默认情况下，响应按用户 ID 升序排列。

```
GET /_synapse/admin/v2/users?from=0&limit=10&guests=false
```

返回如下所示的响应体：

```json
{
    "users": [
        {
            "name": "<user_id1>",
            "is_guest": 0,
            "admin": 0,
            "user_type": null,
            "deactivated": 0,
            "erased": false,
            "shadow_banned": 0,
            "displayname": "<User One>",
            "avatar_url": null,
            "creation_ts": 1560432668000,
            "locked": false
        }, {
            "name": "<user_id2>",
            "is_guest": 0,
            "admin": 1,
            "user_type": null,
            "deactivated": 0,
            "erased": false,
            "shadow_banned": 0,
            "displayname": "<User Two>",
            "avatar_url": "<avatar_url>",
            "creation_ts": 1561550621000,
            "locked": false
        }
    ],
    "next_token": "100",
    "total": 200
}
```

要分页，请检查 `next_token` ，如果存在，再次调用端点，并将 `from` 设置为 `next_token` 的值。这将返回一个新页面。

如果端点没有返回 `next_token` ，则没有更多的用户需要分页。

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 是可选的，仅返回包含此值的用户 ID 的用户。此参数在使用 `name` 参数时将被忽略。
*   `name` - 是可选的，仅返回用户 ID 本地部分或显示名称包含此值的用户。
*   `guests` - 表示布尔的字符串 - 是可选的，如果 `false` 将排除访客用户。默认为 `true` 以包括访客用户。当启用 MSC3861 时不支持此参数。参见 #15582
*   `admins` - 可选标志用于过滤管理员。如果 `true` ，仅查询管理员。如果 `false` ，查询中排除管理员。当标志缺失时（默认情况），搜索结果中包括管理员和非管理员。
*   `deactivated` - 表示布尔值的字符串 - 是可选的，如果 `true` 将包括已停用的用户。默认为 `false` 以排除已停用的用户。
*   `limit` - 表示正整数的字符串 - 是可选的，但用于分页，表示此次调用返回的最大项目数。默认为 `100` 。
*   `from` - 表示正整数值的字符串 - 是可选的，但用于分页，表示返回结果中的偏移量。这应该被视为一个不透明的值，而不是明确设置为除前一次调用 `next_token` 返回值之外的任何其他值。默认为 `0` 。
*   `order_by` - 排序返回的用户列表的方法。如果有重复的排序字段，第二排序总是按升序的 `name` ，这保证了稳定的排序。有效值包括：
*   `name` - 用户按 `name` 字母顺序排序。这是默认的。
*   `is_guest` - 用户按 `is_guest` 状态排序。
*   `admin` - 用户按 `admin` 状态排序。
*   `user_type` - 用户按 `user_type` 字母顺序排列。
*   `deactivated` - 用户按 `deactivated` 状态排序。
*   `shadow_banned` - 用户按 `shadow_banned` 状态排序。
*   `displayname` - 用户按 `displayname` 字母顺序排列。
*   `avatar_url` - 用户按头像 URL 字母顺序排序。
*   `creation_ts` - 用户按创建时间（毫秒）排序。
*   `last_seen_ts` - 用户按最后一次在线时间（毫秒）排序。
*   `dir` - 媒体排序方向。 `f` 表示向前， `b` 表示向后。将此值设置为 `b` 将反转上述排序顺序。默认为 `f` 。
*   `not_user_type` - 排除某些用户类型，例如机器人用户，不包括在请求中。可以多次提供。可能的值有 `bot` 、 `support` 或“空字符串”。“空字符串”在这里表示排除没有类型的用户。
*   `locked` - 表示布尔值的字符串 - 是可选的，如果为 `true` 将包括被锁定的用户。默认为 `false` 以排除被锁定的用户。注意：在 v1.93 中引入。

警告。数据库仅在列 `name` 和 `creation_ts` 上有索引。这意味着如果使用不同的排序顺序（ `is_guest` , `admin` , `user_type` , `deactivated` , `shadow_banned` , `avatar_url` 或 `displayname` ），这可能会对数据库造成很大负载，特别是在大型环境中。

**响应**

JSON 响应体中返回以下字段：

*   `users` - 一个包含用户信息的对象数组。用户对象包含以下字段：
*   `name` - 字符串 - 完全限定的用户 ID（例如 `@user:server.com` ）。
*   `is_guest` - 布尔值 - 该用户是否为访客账户的状态。
*   `admin` - 布尔值 - 该用户是否为服务器管理员的状态。
*   `user_type` - 字符串 - 用户类型。普通用户的类型是 `None` 。这允许特定用户类型的行为。还有类型 `support` 和 `bot` 。
*   `deactivated` - 布尔值 - 用户是否被标记为已停用的状态。
*   `erased` - 布尔值 - 用户是否被标记为已删除的状态。
*   `shadow_banned` - bool - 该用户是否被标记为影子封禁的状态。
*   `displayname` - string - 如果用户设置了显示名称，则为该用户的显示名称。
*   `avatar_url` - string - 如果用户设置了头像，则为该用户的头像 URL。
*   `creation_ts` - 整数 - 用户创建时间戳（毫秒）。
*   `last_seen_ts` - 整数 - 用户最后活动时间戳（毫秒）。
*   `locked` - 布尔值 - 用户是否被标记为锁定。注意：在 v1.93 版本中引入。
*   `next_token` : 表示正整数字符串 - 分页指示。见上文。
*   `total` - 整数 - 媒体总数。

在 Synapse 1.93 中添加： `locked` 查询参数和响应字段。

##### 列出账户 (V3)

此 API 返回所有本地用户账户（参见 v2）。与 v2 相比，查询参数 `deactivated` 的处理方式有所不同。

```
GET /_synapse/admin/v3/users
```

**参数**

*   `deactivated` - 可选标志，用于过滤已停用用户。如果 `true` ，仅返回已停用用户。如果 `false` ，查询中排除已停用用户。当标志不存在时（默认），用户不会根据停用状态进行过滤。

#### 查询用户当前会话

此 API 返回有关特定用户的活动会话的信息。

端点是：

```
GET /_synapse/admin/v1/whois/<user_id>
```

和：

```
GET /_matrix/client/r0/admin/whois/<userId>
```

另见：客户端服务器 API Whois。

它返回如下所示的 JSON 主体：

```json
{
    "user_id": "<user_id>",
    "devices": {
        "": {
            "sessions": [
                {
                    "connections": [
                        {
                            "ip": "1.2.3.4",
                            "last_seen": 1417222374433,
                            "user_agent": "Mozilla/5.0 ..."
                        },
                        {
                            "ip": "1.2.3.10",
                            "last_seen": 1417222374500,
                            "user_agent": "Dalvik/2.1.0 ..."
                        }
                    ]
                }
            ]
        }
    }
}
```

`last_seen` 是以自 Unix 纪元以来的毫秒为单位测量的。

#### 停用账户

此 API 用于停用账户。它会移除活动的访问令牌，重置密码，并删除第三方 ID（以防止用户请求密码重置）。

它还可以将用户标记为 GDPR 已删除。这意味着用户发送的消息仍然可以被在消息发送时在房间内的任何人看到，但对之后加入房间的用户不可见。

API 是：

```
POST /_synapse/admin/v1/deactivate/<user_id>
```

其主体为：

```json
{
    "erase": true
}
```

擦除参数是可选的，默认为 `false` 。为了向后兼容，可以传递一个空的主体。

当停用用户时，执行以下操作：

*   尝试从身份服务器解绑 3PIDs
*   从主服务器移除所有 3PIDs
*   删除所有设备和端到端加密密钥
*   删除所有访问令牌
*   删除所有推送者
*   删除密码哈希
*   从用户所在的所有房间中移除
*   从用户目录中移除用户
*   拒绝所有待处理的邀请
*   删除与用户相关的所有账户有效性信息
*   删除称为账户数据的任意数据存储。例如，这包括：
*   忽略用户列表；
*   推送规则；
*   密钥存储密钥；和
*   交叉签名密钥。

如果 `erase` 设置为 `true` ，则在停用期间执行以下附加操作：

*   移除用户的显示名称
*   移除用户的头像 URL
*   标记用户为已删除

以下操作不会执行。列表可能不完整。

*   删除 SSO ID 的映射
*   删除用户上传的媒体（包括头像图片）
*   删除发送和接收的消息
*   移除用户的创建（注册）时间戳
*   [移除速率限制覆盖](#override-ratelimiting-for-users)
*   从月活跃用户中移除
*   移除用户的同意信息（同意版本和时间戳）

#### 重置密码

注意：当 MSC3861 启用时，此 API 将被禁用。参见 #15582

更改另一用户的密码。这将自动将用户从所有设备中注销。

API 是：

```
POST /_synapse/admin/v1/reset_password/<user_id>
```

其主体为：

```json
{
   "new_password": "<secret>",
   "logout_devices": true
}
```

参数 `new_password` 是必需的。参数 `logout_devices` 是可选的，默认为 `true` 。

#### 获取用户是否为服务器管理员

注意：当 MSC3861 启用时，此 API 将被禁用。参见 #15582

API 是：

```
GET /_synapse/admin/v1/users/<user_id>/admin
```

返回如下所示的响应体：

```json
{
    "admin": true
}
```

#### 更改用户是否为服务器管理员

注意：当 MSC3861 启用时，此 API 将被禁用。参见 #15582

请注意，您不能降级自己。

API 是：

```
PUT /_synapse/admin/v1/users/<user_id>/admin
```

其主体为：

```json
{
    "admin": true
}
```

#### 列出用户的房间成员身份

获取特定 `user_id` 是成员的所有 `room_id` 的列表。

API 是：

```
GET /_synapse/admin/v1/users/<user_id>/joined_rooms
```

返回如下所示的响应体：

```
    {
        "joined_rooms": [
            "!DuGcnbhHGaSZQoNQR:matrix.org",
            "!ZtSaPCawyWtxfWiIy:matrix.org"
        ],
        "total": 2
    }
```

服务器返回用户和服务器共同成员的房间列表。如果用户是本地用户，则返回用户是成员的所有房间。

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定：例如， `@user:server.com` 。

**响应**

JSON 响应体中返回以下字段：

*   `joined_rooms` - `room_id` 的数组。
*   `total` - 房间数量。

#### 账户数据

获取特定 `user_id` 的账户数据信息。

API 是：

```
GET /_synapse/admin/v1/users/<user_id>/accountdata
```

返回如下所示的响应体：

```json
{
    "account_data": {
        "global": {
            "m.secret_storage.key.LmIGHTg5W": {
                "algorithm": "m.secret_storage.v1.aes-hmac-sha2",
                "iv": "fwjNZatxg==",
                "mac": "eWh9kNnLWZUNOgnc="
            },
            "im.vector.hide_profile": {
                "hide_profile": true
            },
            "org.matrix.preview_urls": {
                "disable": false
            },
            "im.vector.riot.breadcrumb_rooms": {
                "rooms": [
                    "!LxcBDAsDUVAfJDEo:matrix.org",
                    "!MAhRxqasbItjOqxu:matrix.org"
                ]
            },
            "m.accepted_terms": {
                "accepted": [
                    "https://example.org/somewhere/privacy-1.2-en.html",
                    "https://example.org/somewhere/terms-2.0-en.html"
                ]
            },
            "im.vector.setting.breadcrumbs": {
                "recent_rooms": [
                    "!MAhRxqasbItqxuEt:matrix.org",
                    "!ZtSaPCawyWtxiImy:matrix.org"
                ]
            }
        },
        "rooms": {
            "!GUdfZSHUJibpiVqHYd:matrix.org": {
                "m.fully_read": {
                    "event_id": "$156334540fYIhZ:matrix.org"
                }
            },
            "!tOZwOOiqwCYQkLhV:matrix.org": {
                "m.fully_read": {
                    "event_id": "$xjsIyp4_NaVl2yPvIZs_k1Jl8tsC_Sp23wjqXPno"
                }
            }
        }
    }
}
```

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定：例如， `@user:server.com` 。

**响应**

JSON 响应体中返回以下字段：

*   `account_data` - 包含用户账户数据的地图
*   `global` - 包含用户全局账户数据的地图
*   `rooms` - 包含用户每个房间账户数据的地图

#### 用户媒体

##### 列出用户上传的媒体

获取特定 `user_id` 创建的所有本地媒体的列表。这些媒体包括用户自己上传的（本地媒体），以及如果功能启用，用户请求的 URL 预览图片。

默认情况下，响应按创建日期降序和媒体 ID 升序排列。最新的媒体在顶部。您可以使用参数 `order_by` 和 `dir` 更改排序。

API 是：

```
GET /_synapse/admin/v1/users/<user_id>/media
```

返回如下所示的响应体：

```json
{
  "media": [
    {
      "created_ts": 100400,
      "last_access_ts": null,
      "media_id": "qXhyRzulkwLsNHTbpHreuEgo",
      "media_length": 67,
      "media_type": "image/png",
      "quarantined_by": null,
      "safe_from_quarantine": false,
      "upload_name": "test1.png"
    },
    {
      "created_ts": 200400,
      "last_access_ts": null,
      "media_id": "FHfiSnzoINDatrXHQIXBtahw",
      "media_length": 67,
      "media_type": "image/png",
      "quarantined_by": null,
      "safe_from_quarantine": false,
      "upload_name": "test2.png"
    },
    {
      "created_ts": 300400,
      "last_access_ts": 300700,
      "media_id": "BzYNLRUgGHphBkdKGbzXwbjX",
      "media_length": 1337,
      "media_type": "application/octet-stream",
      "quarantined_by": null,
      "safe_from_quarantine": false,
      "upload_name": null
    }
  ],
  "next_token": 3,
  "total": 2
}
```

要分页，请检查 `next_token` ，如果存在，再次调用端点，并将 `from` 设置为 `next_token` 的值。这将返回一个新页面。

如果端点未返回 `next_token` ，则没有更多的报告需要分页。

**参数**

以下参数应在 URL 中设置：
*   `user_id` - 字符串 - 完全限定：例如， `@user:server.com` 。
*   `limit` : 表示正整数字符串 - 是可选的，但用于分页，表示此次调用返回的最大项目数。默认为 `100` 。
*   `from` : 表示正整数字符串 - 是可选的，但用于分页，表示返回结果中的偏移量。这应该被视为一个不透明值，而不是明确设置为除前一次调用的 `next_token` 返回值之外的任何其他值。默认为 `0` 。
*   `order_by` - 排序返回的媒体列表的方法。如果有重复的排序字段，第二排序总是按升序的 `media_id` ，这保证了稳定的排序。有效值为：
*   `media_id` - 媒体按 `media_id` 字母顺序排列。
*   `upload_name` - 媒体按上传时的名称字母顺序排列。
*   `created_ts` - 媒体按内容上传时间（毫秒）排序。从小到大。这是默认设置。
*   `last_access_ts` - 媒体按内容最后访问时间（毫秒）排序。从小到大。
*   `media_length` - 媒体按媒体长度（字节）排序。从小到大。
*   `media_type` - 媒体按 MIME 类型字母顺序排序。
*   `quarantined_by` - 媒体按发起隔离请求的用户 ID 字母顺序排列。
*   `safe_from_quarantine` - 如果媒体安全不被隔离，则按状态排序。
*   `dir` - 媒体排序方向。 `f` 表示向前， `b` 表示向后。将此值设置为 `b` 将反转上述排序顺序。默认为 `f` 。

如果 `order_by` 和 `dir` 都没有设置，默认顺序是最新的媒体在顶部（对应于 `order_by` = `created_ts` 和 `dir` = `b` ）。

注意。数据库仅在列 `media_id` 、 `user_id` 和 `created_ts` 上有索引。这意味着如果使用不同的排序顺序（ `upload_name` 、 `last_access_ts` 、 `media_length` 、 `media_type` 、 `quarantined_by` 或 `safe_from_quarantine` ），这可能会对数据库造成很大的负载，特别是在大型环境中。

**响应**

JSON 响应体中返回以下字段：

*   `media` - 一个包含媒体信息的对象数组。媒体对象包含以下字段：
*   `created_ts` - 整数 - 内容上传时的时间戳（毫秒）。
*   `last_access_ts` - 整数或空值 - 内容最后访问时的时间戳（毫秒）。如果尚未访问，则为 null。
*   `media_id` - 字符串 - 用于引用媒体的 ID。有关格式的详细信息请参见媒体库文档。
*   `media_length` - 整数 - 媒体长度（以字节为单位）。
*   `media_type` - 字符串 - 媒体的 MIME 类型。
*   `quarantined_by` - 字符串或空值 - 发起隔离请求的用户 ID。如果未隔离则为 null。
*   `safe_from_quarantine` - bool - 此媒体是否安全，不会被隔离的状态。
*   `upload_name` - string or null - 媒体上传时的名称。如果上传时未提供则为 Null。
*   `next_token` : integer - 分页指示。见上文。
*   `total` - 整数 - 媒体总数。

##### 删除用户上传的媒体

此 API 删除特定 `user_id` 在您服务器磁盘上创建的本地媒体，包括任何本地缩略图。

此 API 不会影响已上传到外部媒体仓库的媒体（例如 https://github.com/turt2live/matrix-media-repo/）。

默认情况下，API 按创建日期降序和媒体 ID 升序删除媒体。最新的媒体将首先被删除。您可以使用参数 `order_by` 和 `dir` 更改顺序。如果未设置 `limit` ，API 将每次请求删除 `100` 个文件。

API 是：

```
DELETE /_synapse/admin/v1/users/<user_id>/media
```

返回如下所示的响应体：

```json
{
  "deleted_media": [
    "abcdefghijklmnopqrstuvwx"
  ],
  "total": 1
}
```

JSON 响应体中返回以下字段：

*   `deleted_media` : 字符串数组 - 删除的 `media_id` 列表
*   `total` : 整数 - 删除的 `media_id` 总数

注意：这里没有 `next_token` 。这对于删除媒体没有用，因为删除媒体后，剩余的媒体会有新的顺序。

**参数**

此 API 与列出用户上传的媒体具有相同的参数。您可以使用这些参数，例如限制一次删除的文件数量，或首先删除最大/最小或最新/最旧的文件。

#### 以用户身份登录

注意：当 MSC3861 启用时，此 API 将被禁用。参见 #15582

获取一个可用于作为该用户身份验证的访问令牌。当管理员希望代表用户执行操作时非常有用。

API 是：

```
POST /_synapse/admin/v1/users/<user_id>/login
{}
```

可以在请求体中指定一个可选的 `valid_until_ms` 字段作为整数时间戳，用于指定令牌何时过期。默认情况下令牌不会过期。请注意，此 API 不允许用户以自己的身份登录（以创建更多令牌）。

返回如下所示的响应体：

```json
{
    "access_token": "<opaque_access_token_string>"
}
```

此 API 不会为用户生成新设备，因此不会出现在他们的 `/devices` 列表中，通常目标用户不应能察觉他们已被登录。

要使令牌过期，请使用标准的 `/logout` API 调用令牌。

注意：如果管理员用户从任何设备调用 `/logout/all` ，令牌将过期，但如果目标用户执行相同的操作，令牌不会过期。

#### 允许替换主交叉签名密钥而不需要用户交互式认证

此端点不供服务器管理员使用；我们在这里描述它是为了完整性。

此 API 暂时允许用户替换其主交叉签名密钥，而无需通过用户交互式认证（UIA）。这在 Synapse 将其认证委托给 Matrix 认证服务时很有用；因为在这种情况下，Synapse 无法执行 UIA。

API 是

```
POST /_synapse/admin/v1/users/<user_id>/_allow_cross_signing_replacement_without_uia
{}
```

如果用户不存在，或者存在但没有主交叉签名密钥，这将返回状态码 `404 Not Found` 。

否则，将返回如下响应体，状态为 `200 OK` ：

```json
{
    "updatable_without_uia_before_ms": 1234567890
}
```

响应体是一个带有单个字段的 JSON 对象：

*   `updatable_without_uia_before_ms` : 整数。在此时间戳（以毫秒为单位）之前，用户可以替换其交叉签名密钥而无需通过 UIA。

_在 Synapse 1.97.0 中添加。_

#### 用户设备

##### 列出所有设备

获取特定 `user_id` 的所有设备信息。

API 是：

```
GET /_synapse/admin/v2/users/<user_id>/devices
```

返回如下所示的响应体：

```json
{
  "devices": [
    {
      "device_id": "QBUAZIFURK",
      "display_name": "android",
      "last_seen_ip": "1.2.3.4",
      "last_seen_user_agent": "Mozilla/5.0 (X11; Linux x86_64; rv:103.0) Gecko/20100101 Firefox/103.0",
      "last_seen_ts": 1474491775024,
      "user_id": "<user_id>"
    },
    {
      "device_id": "AUIECTSRND",
      "display_name": "ios",
      "last_seen_ip": "1.2.3.5",
      "last_seen_user_agent": "Mozilla/5.0 (X11; Linux x86_64; rv:103.0) Gecko/20100101 Firefox/103.0",
      "last_seen_ts": 1474491775025,
      "user_id": "<user_id>"
    }
  ],
  "total": 2
}
```

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定：例如， `@user:server.com` 。

**响应**

JSON 响应体中返回以下字段：

*   `devices` - 一个包含设备信息的对象数组。设备对象包含以下字段：
*   `device_id` - 设备的标识符。
*   `display_name` - 用户为此设备设置的显示名称。如果未设置名称，则不存在。
*   `last_seen_ip` - 此设备最后一次被看到的 IP 地址。（出于效率原因，可能会有几分钟的延迟。）
*   `last_seen_user_agent` - 设备上次被看到时的用户代理。（出于效率原因，可能会有几分钟的延迟）。
*   `last_seen_ts` - 设备上次被看到时的时间戳（自 Unix 纪元以来的毫秒数）。（出于效率原因，可能会有几分钟的延迟）。
*   `user_id` - 设备的所有者。
*   `total` - 用户设备的总数。

##### 创建设备

为特定的 `user_id` 和 `device_id` 创建一个新设备。如果 `device_id` 已经存在，则不执行任何操作。

API 是：

```
POST /_synapse/admin/v2/users/<user_id>/devices

{
  "device_id": "QBUAZIFURK"
}
```

返回一个空的 JSON 字典。

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定：例如， `@user:server.com` 。

JSON 请求体中需要包含以下字段：

*   `device_id` - 要创建的设备 ID。

##### 删除多个设备

删除特定 `user_id` 的给定设备，并使与之关联的任何访问令牌失效。

API 是：

```
POST /_synapse/admin/v2/users/<user_id>/delete_devices

{
  "devices": [
    "QBUAZIFURK",
    "AUIECTSRND"
  ]
}
```

返回一个空的 JSON 字典。

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定：例如， `@user:server.com` 。

JSON 请求体中需要包含以下字段：

*   `devices` - 要删除的设备 ID 列表。

##### 显示设备

通过 `device_id` 获取特定 `user_id` 的单个设备信息。

API 是：

```
GET /_synapse/admin/v2/users/<user_id>/devices/<device_id>
```

返回如下所示的响应体：

```json
{
  "device_id": "<device_id>",
  "display_name": "android",
  "last_seen_ip": "1.2.3.4",
  "last_seen_user_agent": "Mozilla/5.0 (X11; Linux x86_64; rv:103.0) Gecko/20100101 Firefox/103.0",
  "last_seen_ts": 1474491775024,
  "user_id": "<user_id>"
}
```

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定：例如， `@user:server.com` 。
*   `device_id` - 要检索的设备。

**响应**

JSON 响应体中返回以下字段：

*   `device_id` - 设备的标识符。
*   `display_name` - 用户为此设备设置的显示名称。如果未设置名称，则不存在。
*   `last_seen_ip` - 此设备最后一次被看到的 IP 地址。（出于效率原因，可能会有几分钟的延迟。）
*   `last_seen_user_agent` - 设备上次被看到时的用户代理。（出于效率原因，可能会有几分钟的延迟）。
*   `last_seen_ts` - 设备上次被看到时的时间戳（自 Unix 纪元以来的毫秒数）。（出于效率原因，可能会有几分钟的延迟）。
*   `user_id` - 设备的所有者。

##### 更新设备

更新给定 `device_id` 上特定 `user_id` 的元数据。

API 是：

```
PUT /_synapse/admin/v2/users/<user_id>/devices/<device_id>

{
  "display_name": "My other phone"
}
```

返回一个空的 JSON 字典。

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定：例如， `@user:server.com` 。
*   `device_id` - 要更新的设备。

JSON 请求体中需要包含以下字段：

*   `display_name` - 此设备的新显示名称。如果未提供，则显示名称保持不变。

##### 删除设备

删除指定的 `device_id` 对于特定的 `user_id` ，并使与之关联的任何访问令牌失效。

API 是：

```
DELETE /_synapse/admin/v2/users/<user_id>/devices/<device_id>

{}
```

返回一个空的 JSON 字典。

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定：例如， `@user:server.com` 。
*   `device_id` - 要删除的设备。

#### 列出所有推送者

获取特定 `user_id` 的所有推送者的信息。

API 是：

```
GET /_synapse/admin/v1/users/<user_id>/pushers
```

返回如下所示的响应体：

```json
{
  "pushers": [
    {
      "app_display_name":"HTTP Push Notifications",
      "app_id":"m.http",
      "data": {
        "url":"example.com"
      },
      "device_display_name":"pushy push",
      "kind":"http",
      "lang":"None",
      "profile_tag":"",
      "pushkey":"a@example.com"
    }
  ],
  "total": 1
}
```

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定：例如， `@user:server.com` 。

**响应**

JSON 响应体中返回以下字段：
*   `pushers` - 包含当前用户推送者的数组
*   `app_display_name` - string - 一个字符串，允许用户识别哪个应用程序拥有这个推送者。
*   `app_id` - string - 这是一个应用程序的反向 DNS 风格标识符。最大长度为 64 个字符。
*   `data` - 推送者实现本身的信息字典。
    *   `url` - 字符串 - 如果 `kind` 是 `http` 则必需。用于发送通知的 URL。  
    *   `format` - 字符串 - 发送通知到推送网关时使用的格式。
*   `device_display_name` - 字符串 - 一个字符串，允许用户识别哪个设备拥有这个推送器。
*   `profile_tag` - 字符串 - 此字符串决定了此推送器执行的设备特定规则集。
*   `kind` - 字符串 - 推送器的类型。"http" 是一种发送 HTTP 推送的推送器。
*   `lang` - 字符串 - 接收通知的首选语言（例如 'en' 或 'en-US'）
*   `profile_tag` - 字符串 - 此字符串决定了此推送器执行的设备特定规则集。
*   `pushkey` - 字符串 - 这是此推送器的唯一标识符。最大长度为 512 字节。
*   `total` - 整数 - 推送者的数量。

另见关于推送者的客户端-服务器 API 规范。

#### 控制用户是否被影子封禁

影子封禁是管理恶意或严重滥用用户的有用工具。  
被影子封禁的用户会收到客户端-服务器 API 请求的成功响应，但这些事件不会传播到房间中。这可以作为一种有效的工具，因为（希望）用户在意识到自己被管理之前需要更长的时间来转向另一个账户。

影子封禁用户应作为最后手段使用，并且可能会导致客户端出现混乱或损坏的行为。被影子封禁的用户不会收到任何通知，通常更合适的做法是禁止或踢出滥用用户。  
一个被影子封禁的用户将无法与服务器上的任何人联系。

要影子封禁一个用户，API 是：

```
POST /_synapse/admin/v1/users/<user_id>/shadow_ban
```

要取消影子封禁一个用户，API 是：

```
DELETE /_synapse/admin/v1/users/<user_id>/shadow_ban
```

在两种情况下都会返回一个空的 JSON 字典。

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定的 MXID：例如， `@user:server.com` 。用户必须是本地的。

#### 覆盖用户的速率限制

此 API 允许覆盖或禁用特定用户的速率限制。有特定的 API 来设置、获取和删除速率限制。

##### 获取速率限制状态

API 是：

```
GET /_synapse/admin/v1/users/<user_id>/override_ratelimit
```

返回如下所示的响应体：

```json
{
  "messages_per_second": 0,
  "burst_count": 0
}
```

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定的 MXID：例如， `@user:server.com` 。用户必须是本地的。

**响应**

JSON 响应体中返回以下字段：

*   `messages_per_second` - 整数 - 每秒可以执行的操作数量。 `0` 表示对该用户禁用限速。
*   `burst_count` - 整数 - 在被限制之前可以执行的操作数量。

如果没有设置自定义速率限制，将返回一个空的 JSON 字典。

```json
{}
```

##### 设置速率限制

API 是：

```
POST /_synapse/admin/v1/users/<user_id>/override_ratelimit
```

返回如下所示的响应体：

```json
{
  "messages_per_second": 0,
  "burst_count": 0
}
```

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定的 MXID：例如， `@user:server.com` 。用户必须是本地的。

Body 参数：

*   `messages_per_second` - 正整数，可选。每秒可以执行的操作数量。默认为 `0` 。
*   `burst_count` - 正整数，可选。可以执行的操作次数限制前可以执行的操作数。默认为 `0` 。

要禁用用户的速率限制，请将两个值都设置为 `0` 。

**响应**

JSON 响应体中返回以下字段：

*   `messages_per_second` - 整数 - 每秒可以执行的操作数。
*   `burst_count` - 整数 - 在被限制之前可以执行的操作数量。

##### 删除速率限制

API 是：

```
DELETE /_synapse/admin/v1/users/<user_id>/override_ratelimit
```

返回一个空的 JSON 字典。

```json
{}
```

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 完全限定的 MXID：例如， `@user:server.com` 。用户必须是本地的。

#### 检查用户名可用性

检查用户名是否在服务器上可用且有效。有关更多信息，请参见客户端-服务器 API。

即使服务器上禁用了注册，此端点仍将工作，与 `/_matrix/client/r0/register/available` 不同。

API 是：

```
GET /_synapse/admin/v1/username_available?username=$localpart
```

请求和响应格式与 [/_matrix/client/r0/register/available](https://matrix.org/docs/spec/client_server/r0.6.0#get-matrix-client-r0-register-available) API 相同。

#### 根据身份提供者的 ID 查找用户

API 为：

```
GET /_synapse/admin/v1/auth_providers/$provider/users/$external_id
```

当用户匹配给定提供者的给定 ID 时，返回 HTTP 状态码 `200`，响应体如下：

```json
{
    "user_id": "@hello:example.org"
}
```

**参数**

以下参数应在 URL 中设置：

- `provider` - 身份提供者的 ID，如 [`GET /_matrix/client/v3/login`](https://spec.matrix.org/latest/client-server-api/#post_matrixclientv3login) API 中 `m.login.sso` 认证方法所示。
- `external_id` - 身份提供者的用户 ID。通常对应 OIDC 提供者的 `sub` 声明，或 SAML2 提供者的 `uid` 证明。

`external_id` 可能包含非 URL 安全的字符（通常是 `/`、`:` 或 `@`），因此建议对这些参数进行 URL 编码。

**错误**

如果未找到用户，则返回 `404` HTTP 状态码，响应体如下：

```json
{
    "errcode":"M_NOT_FOUND",
    "error":"User not found"
}
```

_在 Synapse 1.68.0 中添加。_

#### 根据第三方 ID（ThreePID 或 3PID）查找用户

API 为：

```
GET /_synapse/admin/v1/threepid/$medium/users/$address
```

当用户匹配给定媒介的给定地址时，返回 HTTP 状态码 `200`，响应体如下：

```json
{
    "user_id": "@hello:example.org"
}
```

**参数**

以下参数应在 URL 中设置：

- `medium` - 第三方 ID 的类型，可以是 `email` 或 `msisdn`。
- `address` - 第三方 ID 的值。

`address` 可能包含非 URL 安全的字符，因此建议对这些参数进行 URL 编码。

**错误**

如果未找到用户，则返回 `404` HTTP 状态码，响应体如下：

```json
{
    "errcode":"M_NOT_FOUND",
    "error":"User not found"
}
```

_在 Synapse 1.72.0 中添加。_

#### 撤销用户的所有事件

这API是
```
POST /_synapse/admin/v1/user/$user_id/redact

{
  "rooms": ["!roomid1", "!roomid2"]
}
```

如果提供一个空列表作为 `rooms` 的键，所有用户所在房间的事件都将被撤回，否则请求中提供的房间中的所有事件都将被撤回。

API 启动撤回过程并立即返回一个带有撤回 ID 的 JSON 体，该 ID 可用于查询撤回过程的状态：

```json
{
    "redact_id": "<opaque id>"
}
```

**参数**

以下参数应在 URL 中设置：

*   `user_id` - 用户的完全限定 MXID：例如， `@user:server.com` 。

必须提供以下 JSON 主体参数：

*   `rooms` - 一个房间列表，用于撤回用户在其中的事件。如果提供一个空列表，将撤回用户所在所有房间中的所有事件

_在 Synapse 1.116.0 中添加。_

以下 JSON 主体参数是可选的：

*   `reason` - 请求屏蔽的原因，例如“垃圾信息”、“滥用”等。这将包含在每个屏蔽事件中，并对用户可见。
*   `limit` - 对用户事件数量的限制，用于搜索可以被屏蔽的事件（事件按从新到旧的顺序屏蔽），如果未提供，默认为 1000

#### 检查脱敏过程的状态

可以查询用户事件脱敏后台任务的状态。可以在任务完成后的 24 小时内查询状态，或者直到 Synapse 重启（以先发生者为准）。

API 是：

```
GET /_synapse/admin/v1/user/redact_status/$redact_id
```

返回如下的响应体：

```json
{
  "status": "active",
  "failed_redactions": [],
}
```
**参数**

以下参数应在URL中设置：

* `redact_id` - 字符串 - 此次编辑过程的ID，在请求编辑时提供。

**响应**

JSON响应体中返回以下字段：

- `status` - 字符串 - 表示编辑作业状态的字符串之一：scheduled/active/completed/failed
- `failed_redactions` - 字典 - 字典的键是无法编辑的事件ID，如果有的话，值是导致编辑失败的相应错误

_在Synapse 1.116.0中添加。_