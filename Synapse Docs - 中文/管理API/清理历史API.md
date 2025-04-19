### 清除历史记录 API

清除历史记录 API 允许服务器管理员从数据库中清除历史事件，收回磁盘空间。

根据要清除的历史记录量，API 调用可能需要几分钟或更长时间。在此期间，用户将无法从被清除的点进一步翻页查看房间的历史记录。

请注意，Synapse 要求每个房间至少有一条消息，因此它永远不会删除房间中的最后一条消息。

要使用它，您需要通过提供一个 `access_token` 来进行身份验证以成为服务器管理员：请参见 Admin API。

API 是：

```
POST /_synapse/admin/v1/purge_history/<room_id>[/<event_id>]
```

默认情况下，本地用户发送的事件不会被删除，因为它们可能是此内容唯一存在的副本。（远程用户发送的事件会被删除。）

房间状态数据（如加入、离开、主题）总是被保留。

要删除本地消息事件，请在请求体中设置 `delete_local_events` ：

```json
{
   "delete_local_events": true
}
```

调用者必须指定房间中要清除到的点。这可以通过在 URI 中包含一个 event_id，或者在请求体中设置 `purge_up_to_event_id` 或 `purge_up_to_ts` 来指定。如果给出了一个事件 ID，该事件（以及同一图深度上的其他事件）将被保留。如果给出了 `purge_up_to_ts` ，它应该是自 Unix 纪元以来的时间戳，以毫秒为单位。

API 启动清理运行，并立即返回一个包含清理 ID 的 JSON 正文：

```json
{
    "purge_id": "<opaque id>"
}
```

#### 查询清理状态

可以使用第二个 API 轮询最近清理的更新；

```
GET /_synapse/admin/v1/purge_history_status/<purge_id>
```

此 API 返回如下所示的 JSON 主体：

```json
{
    "status": "active"
}
```

状态将是 `active` 、 `complete` 或 `failed` 之一。

如果 `status` 是 `failed` ，则会有一个字符串 `error` 包含错误信息。

#### 回收磁盘空间 (Postgres)

为了回收磁盘空间并将其归还给操作系统，您需要在数据库上运行 `VACUUM FULL;` 。

[https://www.postgresql.org/docs/current/sql-vacuum.html](https://www.postgresql.org/docs/current/sql-vacuum.html)