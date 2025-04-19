### 查询媒体

这些 API 允许从主服务器提取媒体信息。

关于 `media_id` 的格式以及媒体在文件系统中的存储方式的详细信息已在媒体存储库中记录。

要使用它，您需要通过提供一个 `access_token` 来进行身份验证以成为服务器管理员：请参见 Admin API。

#### 列出房间中的所有媒体

此 API 获取房间中已知媒体的列表。然而，它只显示来自未加密事件或房间的媒体。

API 是：

```
GET /_synapse/admin/v1/room/<room_id>/media
```

API 返回如下所示的 JSON 正文：

```json
{
  "local": [
    "mxc://localhost/xwvutsrqponmlkjihgfedcba",
    "mxc://localhost/abcdefghijklmnopqrstuvwx"
  ],
  "remote": [
    "mxc://matrix.org/xwvutsrqponmlkjihgfedcba",
    "mxc://matrix.org/abcdefghijklmnopqrstuvwx"
  ]
}
```

#### 列出用户上传的所有媒体

通过使用列出用户上传的媒体管理员 API，可以实现列出本地用户上传的所有媒体。

### 隔离媒体

隔离媒体意味着它被标记为用户无法访问。它适用于任何本地媒体，以及任何远程媒体的本地缓存副本。

媒体文件本身（以及任何缩略图）不会从服务器上删除。

#### 通过 ID 隔离媒体

此 API 隔离单个本地或远程媒体。

请求：

```
POST /_synapse/admin/v1/media/quarantine/<server_name>/<media_id>

{}
```

其中 `server_name` 的形式为 `example.org` ，而 `media_id` 的形式为 `abcdefg12345...` 。

响应：

```json
{}
```

#### 通过 ID 从隔离区移除媒体

此 API 将单个本地或远程媒体从隔离区中移除。

请求：

```
POST /_synapse/admin/v1/media/unquarantine/<server_name>/<media_id>

{}
```

其中 `server_name` 的形式为 `example.org` ，而 `media_id` 的形式为 `abcdefg12345...` 。

响应：

```json
{}
```

#### 在房间中隔离媒体

此 API 将房间中的所有本地和远程媒体进行隔离。

请求：

```
POST /_synapse/admin/v1/room/<room_id>/media/quarantine

{}
```

其中 `room_id` 的形式为 `!roomid12345:example.org` 。

响应：

```json
{
  "num_quarantined": 10
}
```

JSON 响应体中返回以下字段：

*   `num_quarantined` : integer - 成功隔离的媒体项目数量

请注意，有一个旧的端点， `POST /_synapse/admin/v1/quarantine_media/<room_id>` ，其功能相同。然而，它已被弃用，可能会在未来的版本中移除。

#### 隔离用户的所有媒体

此 API 会隔离本地用户上传的所有本地媒体。也就是说，如果您想隔离远程主服务器上的用户上传的媒体，您应该使用其他 API。

请求：

```
POST /_synapse/admin/v1/user/<user_id>/media/quarantine

{}
```

URL 参数

*   `user_id` : string - 用户 ID 的形式为 `@bob:example.org`

响应：

```json
{
  "num_quarantined": 10
}
```

JSON 响应体中返回以下字段：

*   `num_quarantined` : integer - 成功隔离的媒体项目数量

#### 保护媒体不被隔离

此 API 使用上述 API 保护单个本地媒体不被隔离。这对于贴纸包和其他共享媒体非常有用，您不希望这些媒体被隔离，特别是在房间内隔离媒体时。

请求：

```
POST /_synapse/admin/v1/media/protect/<media_id>

{}
```

其中 `media_id` 的形式为 `abcdefg12345...` 。

响应：

```json
{}
```

#### 取消媒体隔离保护

此 API 撤销了对媒体的保护。

请求：

```
POST /_synapse/admin/v1/media/unprotect/<media_id>

{}
```

其中 `media_id` 的形式为 `abcdefg12345...` 。

响应：

```json
{}
```

### 删除本地媒体 

此 API 会从您服务器的磁盘上删除本地媒体。这包括从远程主服务器下载的任何本地缩略图和媒体副本。此 API 不会影响已上传到外部媒体存储库的媒体（例如 https://github.com/turt2live/matrix-media-repo/）。 另见[清除远程媒体 API](#purge-remote-media-api)。

#### 删除特定本地媒体 

删除特定的 `media_id` .

请求：

```
DELETE /_synapse/admin/v1/media/<server_name>/<media_id>

{}
```

URL 参数

*   `server_name` : string - 您的本地服务器的名称（例如 `matrix.org` ）
*   `media_id` : 字符串 - 媒体 ID（例如 `abcdefghijklmnopqrstuvwx` ）

响应：

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

#### 按日期或大小删除本地媒体

请求：

```
POST /_synapse/admin/v1/media/delete?before_ts=<before_ts>

{}
```

在 Synapse v1.78.0 中已弃用：此 API 在已弃用的端点可用：

```
POST /_synapse/admin/v1/media/<server_name>/delete?before_ts=<before_ts>

{}
```

URL 参数

*   `server_name` : string - 您的本地服务器的名称（例如 `matrix.org` ）。在 Synapse v1.78.0 中已弃用。
*   `before_ts` : string representing a positive integer - Unix 时间戳（以毫秒为单位）。在此时间戳之前最后使用的文件将被删除。这是最后访问的时间戳，而不是文件创建的时间戳。
*   `size_gt` : Optional - string representing a positive integer - 媒体的大小（以字节为单位）。大于此大小的文件将被删除。默认为 `0` 。
*   `keep_profiles` : 可选 - 表示布尔值的字符串 - 切换以删除仍在图像数据中使用的文件（例如用户头像、房间头像）。如果 `false` 这些文件将被删除。默认为 `true` 。

响应：

```json
{
  "deleted_media": [
    "abcdefghijklmnopqrstuvwx",
    "abcdefghijklmnopqrstuvwz"
  ],
  "total": 2
}
```

JSON 响应体中返回以下字段：

*   `deleted_media` : 字符串数组 - 删除的 `media_id` 列表
*   `total` : 整数 - 删除的 `media_id` 总数

#### 删除用户上传的媒体

您可以在用户管理 API 中找到如何删除用户上传的多个媒体的详细信息。

### 清除远程媒体 API

清除远程媒体 API 允许服务器管理员清除旧的缓存远程媒体。

API 是：

```
POST /_synapse/admin/v1/purge_media_cache?before_ts=<unix_timestamp_in_ms>

{}
```

URL 参数

*   `before_ts` : 表示正整数的字符串 - Unix 时间戳（毫秒）。所有在此时间戳之前最后访问的缓存媒体将被移除。

响应：

```json
{
  "deleted": 10
}
```

JSON 响应体中返回以下字段：

*   `deleted` : 整数 - 成功删除的媒体项目数量

如果用户重新请求已清除的远程媒体，synapse 将从原始服务器重新请求该媒体。