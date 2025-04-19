# 推送网关 API

客户端可能希望在 homeserver 接收到事件时接收推送通知。这由一个称为推送网关的独立实体管理。

## 概述

客户端的 homeserver 将接收到的事件信息转发给推送网关。然后，网关将推送通知提交给推送通知提供者（例如 APNS、GCM）。

```
                                   +--------------------+  +-----------------+
                  Matrix HTTP      |                    |  |                 |
                    通知协议        |   应用开发者        |  |   设备供应商     |
                                   |                    |  |                 |
           +-------------------+   | +----------------+ |  | +-------------+ |
           |                   |   | |                | |  | |             | |
           | Matrix 主服务器    +----->  推送网关      +------> 推送提供者   | |
           |                   |   | |                | |  | |             | |
           +-^-----------------+   | +----------------+ |  | +----+--------+ |
             |                     |                    |  |      |          |
    Matrix   |                     |                    |  |      |          |
 客户端/服务器 API   +              |                    |  |      |          |
             |      |              +--------------------+  +-----------------+
             |   +--+-+                                           |
             |   |    <-------------------------------------------+
             +---+    |
                 |    |          提供者推送协议
                 +----+

         移动设备或客户端
```

## API 标准

### 不支持的端点

如果收到对不支持（或未知）端点的请求，则服务器必须响应 404 `M_UNRECOGNIZED` 错误。

同样，405 `M_UNRECOGNIZED` 错误用于表示对已知端点的不支持方法。

## Homeserver 行为

这描述了“HTTP”推送器用于向推送网关发送事件通知的格式。如果端点返回 HTTP 错误代码，homeserver 应该使用指数退避在合理的时间内重试。

在推送事件通知时，homeserver 预计在 `/notify` 请求中包含所有与事件相关的字段。当 homeserver 执行 `format` 为 `"event_id_only"` 的推送时，只需填充 `event_id`、`room_id`、`counts` 和 `devices`。

请注意，此端点的大多数值和行为由客户端-服务器 API 的 [推送模块](https://spec.matrix.org/unstable/client-server-api#push-notifications) 描述。

###  POST /_matrix/push/v1/notify

---

此端点由 HTTP 推送器调用，以通知推送网关有关事件或更新用户未读通知的数量。在前一种情况下，它将包含有关事件的选定信息。在任何一种情况下，它都可能包含用户未读的不同类型事件的数量。这些计数可以与事件通知一起发送，也可以单独发送。

有关特定事件的通知通常会导致用户以某种方式被提醒。因此，有必要使用 `event_id` 字段执行此类通知的重复抑制，以避免此 HTTP API 的重试导致重复警报。更新未读通知计数的操作应是幂等的，因此不需要重复抑制。

通知发送到创建推送器时配置的 URL。这意味着 HTTP 路径可能会根据推送网关的不同而不同。

|   |   |
|---|---|
|速率限制：|否|
|需要认证：|否|

---

#### 请求

##### 请求体

| 名称             | 类型                                                                                                              | 描述             |
| -------------- | --------------------------------------------------------------------------------------------------------------- | -------------- |
| `notification` | [Notification](https://spec.matrix.org/unstable/push-gateway-api/#post_matrixpushv1notify_request_notification) | **必需：** 推送通知信息 |

| Notification          |                                                                                                       |                                                                                                |
| --------------------- | ----------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| 名称                    | 类型                                                                                                    | 描述                                                                                             |
| `content`             | `EventContent`                                                                                        | 事件中的 `content` 字段（如果存在）。如果事件没有内容或出于其他原因，推送器可以省略此字段。                                            |
| `counts`              | [Counts](https://spec.matrix.org/unstable/push-gateway-api/#post_matrixpushv1notify_request_counts)   | 这是一个字典，包含接收者用户当前未确认的通信数量。值为零的计数应省略。                                                            |
| `devices`             | [[Device](https://spec.matrix.org/unstable/push-gateway-api/#post_matrixpushv1notify_request_device)] | **必需：** 这是一个设备数组，通知应发送到这些设备。                                                                   |
| `event_id`            | `string`                                                                                              | 被通知事件的 Matrix 事件 ID。如果通知是关于特定 Matrix 事件的，这是必需的。对于仅包含更新徽章计数的通知，可以省略此 ID。此 ID 可以并且应被用于检测重复的通知请求。 |
| `prio`                | `string`                                                                                              | 通知的优先级。如果省略，默认假定为 `high`。推送网关可以使用此优先级以一种节省移动设备电池电量的方式传递不太紧急的通知。<br><br>可选值：`[high, low]`。      |
| `room_alias`          | `string`                                                                                              | 显示事件发生的房间的别名。                                                                                  |
| `room_id`             | `string`                                                                                              | 事件发生的房间 ID。如果通知与特定 Matrix 事件相关，这是必需的。                                                          |
| `room_name`           | `string`                                                                                              | 事件发生的房间名称。                                                                                     |
| `sender`              | `string`                                                                                              | 事件的发送者，与相应事件字段中的发送者相同。                                                                         |
| `sender_display_name` | `string`                                                                                              | 事件发生的房间中发送者的当前显示名称。                                                                            |
| `type`                | `string`                                                                                              | 事件的类型，与事件的 `type` 字段相同。                                                                        |
| `user_is_target`      | `boolean`                                                                                             | 如果接收通知的用户是成员事件的对象（即成员事件的 `state_key` 等于用户的 Matrix ID），则为 true。                                 |

| Counts         |           |                         |
| -------------- | --------- | ----------------------- |
| 名称             | 类型        | 描述                      |
| `missed_calls` | `integer` | 用户在其所属的所有房间中未确认的未接电话数量。 |
| `unread`       | `integer` | 用户在其所属的所有房间中未读消息的数量。    |

|Device|
|---|---|---|
|名称|类型|描述|
|---|---|---|
|`app_id`|`string`|**必需：** 创建推送器时给定的 `app_id`。|
|`data`|`PusherData`|一个包含额外推送器特定数据的字典。对于 ‘http’ 推送器，这是在创建推送器时传递的数据字典，减去 `url` 键。|
|`pushkey`|`string`|**必需：** 创建推送器时给定的 `pushkey`。|
|`pushkey_ts`|`integer`|推送键最后更新的 Unix 时间戳（以秒为单位）。|
|`tweaks`|`Tweaks`|一个字典，包含对通知呈现方式的自定义。这些由推送规则添加。|

##### 请求体示例

```json
{
  "notification": {
    "content": {
      "body": "I'm floating in a most peculiar way.",
      "msgtype": "m.text"
    },
    "counts": {
      "missed_calls": 1,
      "unread": 2
    },
    "devices": [
      {
        "app_id": "org.matrix.matrixConsole.ios",
        "data": {},
        "pushkey": "V2h5IG9uIGVhcnRoIGRpZCB5b3UgZGVjb2RlIHRoaXM/",
        "pushkey_ts": 12345678,
        "tweaks": {
          "sound": "bing"
        }
      }
    ],
    "event_id": "$3957tyerfgewrf384",
    "prio": "high",
    "room_alias": "#exampleroom:matrix.org",
    "room_id": "!slw48wfj34rtnrf:example.com",
    "room_name": "Mission Control",
    "sender": "@exampleuser:matrix.org",
    "sender_display_name": "Major Tom",
    "type": "m.room.message"
  }
}
```

---

#### 响应

|状态|描述|
|---|---|
|`200`|被拒绝的推送键列表。|

##### 200 响应

|名称|类型|描述|
|---|---|---|
|`rejected`|`[string]`|**必需：** 通知请求中所有无效的推送键列表。这些可能被上游网关拒绝，因为它们已过期或从未有效。homeserver 必须停止发送这些推送键的通知请求并移除相关的推送器。可能并不一定是请求中的通知失败：也可能是之前对同一推送键的通知失败。可能为空。|

```json
{
  "rejected": [
    "V2h5IG9uIGVhcnRoIGRpZCB5b3UgZGVjb2RlIHRoaXM/"
  ]
}
```