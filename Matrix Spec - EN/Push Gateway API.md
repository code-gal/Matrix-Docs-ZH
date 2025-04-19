> [Matrix Specification](https://spec.matrix.org/unstable/) |  [Push Gateway API](https://spec.matrix.org/unstable/push-gateway-api/)


# Push Gateway API

Clients may want to receive push notifications when events are received at the homeserver. This is managed by a distinct entity called the Push Gateway.

## Overview[](https://spec.matrix.org/unstable/push-gateway-api/#overview)

A client’s homeserver forwards information about received events to the push gateway. The gateway then submits a push notification to the push notification provider (e.g. APNS, GCM).

```
                                   +--------------------+  +-----------------+
                  Matrix HTTP      |                    |  |                 |
             Notification Protocol |   App Developer    |  |   Device Vendor |
                                   |                    |  |                 |
           +-------------------+   | +----------------+ |  | +-------------+ |
           |                   |   | |                | |  | |             | |
           | Matrix homeserver +----->  Push Gateway  +------>Push Provider| |
           |                   |   | |                | |  | |             | |
           +-^-----------------+   | +----------------+ |  | +----+--------+ |
             |                     |                    |  |      |          |
    Matrix   |                     |                    |  |      |          |
 Client/Server API  +              |                    |  |      |          |
             |      |              +--------------------+  +-----------------+
             |   +--+-+                                           |
             |   |    <-------------------------------------------+
             +---+    |
                 |    |          Provider Push Protocol
                 +----+

         Mobile Device or Client
```

## API standards[](https://spec.matrix.org/unstable/push-gateway-api/#api-standards)

### Unsupported endpoints[](https://spec.matrix.org/unstable/push-gateway-api/#unsupported-endpoints)

If a request for an unsupported (or unknown) endpoint is received then the server must respond with a 404 `M_UNRECOGNIZED` error.

Similarly, a 405 `M_UNRECOGNIZED` error is used to denote an unsupported method to a known endpoint.

## Homeserver behaviour[](https://spec.matrix.org/unstable/push-gateway-api/#homeserver-behaviour)

This describes the format used by “HTTP” pushers to send notifications of events to Push Gateways. If the endpoint returns an HTTP error code, the homeserver SHOULD retry for a reasonable amount of time using exponential backoff.

When pushing notifications for events, the homeserver is expected to include all of the event-related fields in the `/notify` request. When the homeserver is performing a push where the `format` is `"event_id_only"`, only the `event_id`, `room_id`, `counts`, and `devices` are required to be populated.

Note that most of the values and behaviour of this endpoint is described by the Client-Server API’s [Push Module](https://spec.matrix.org/unstable/client-server-api#push-notifications).

# POST /_matrix/push/v1/notify[](https://spec.matrix.org/unstable/push-gateway-api/#post_matrixpushv1notify)

---

This endpoint is invoked by HTTP pushers to notify a push gateway about an event or update the number of unread notifications a user has. In the former case it will contain selected information about the event. In either case it may contain numeric counts of the number of unread events of different types the user has. The counts may be sent along with a notification about an event or by themselves.

Notifications about a particular event will normally cause the user to be alerted in some way. It is therefore necessary to perform duplicate suppression for such notifications using the `event_id` field to avoid retries of this HTTP API causing duplicate alerts. The operation of updating counts of unread notifications should be idempotent and therefore do not require duplicate suppression.

Notifications are sent to the URL configured when the pusher is created. This means that the HTTP path may be different depending on the push gateway.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|No|

---

## Request

### Request body

|Name|Type|Description|
|---|---|---|
|`notification`|`[Notification](https://spec.matrix.org/unstable/push-gateway-api/#post_matrixpushv1notify_request_notification)`|**Required:** Information about the push notification|

|Notification|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|`EventContent`|The `content` field from the event, if present. The pusher may omit this if the event had no content or for any other reason.|
|`counts`|`[Counts](https://spec.matrix.org/unstable/push-gateway-api/#post_matrixpushv1notify_request_counts)`|This is a dictionary of the current number of unacknowledged communications for the recipient user. Counts whose value is zero should be omitted.|
|`devices`|`[[Device](https://spec.matrix.org/unstable/push-gateway-api/#post_matrixpushv1notify_request_device)]`|**Required:** This is an array of devices that the notification should be sent to.|
|`event_id`|`string`|The Matrix event ID of the event being notified about. This is required if the notification is about a particular Matrix event. It may be omitted for notifications that only contain updated badge counts. This ID can and should be used to detect duplicate notification requests.|
|`prio`|`string`|The priority of the notification. If omitted, `high` is assumed. This may be used by push gateways to deliver less time-sensitive notifications in a way that will preserve battery power on mobile devices.<br><br>One of: `[high, low]`.|
|`room_alias`|`string`|An alias to display for the room in which the event occurred.|
|`room_id`|`string`|The ID of the room in which this event occurred. Required if the notification relates to a specific Matrix event.|
|`room_name`|`string`|The name of the room in which the event occurred.|
|`sender`|`string`|The sender of the event as in the corresponding event field.|
|`sender_display_name`|`string`|The current display name of the sender in the room in which the event occurred.|
|`type`|`string`|The type of the event as in the event’s `type` field.|
|`user_is_target`|`boolean`|This is true if the user receiving the notification is the subject of a member event (i.e. the `state_key` of the member event is equal to the user’s Matrix ID).|

|Counts|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`missed_calls`|`integer`|The number of unacknowledged missed calls a user has across all rooms of which they are a member.|
|`unread`|`integer`|The number of unread messages a user has across all of the rooms they are a member of.|

|Device|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`app_id`|`string`|**Required:** The `app_id` given when the pusher was created.|
|`data`|`PusherData`|A dictionary of additional pusher-specific data. For ‘http’ pushers, this is the data dictionary passed in at pusher creation minus the `url` key.|
|`pushkey`|`string`|**Required:** The `pushkey` given when the pusher was created.|
|`pushkey_ts`|`integer`|The unix timestamp (in seconds) when the pushkey was last updated.|
|`tweaks`|`Tweaks`|A dictionary of customisations made to the way this notification is to be presented. These are added by push rules.|

### Request body example

```
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

## Responses

|Status|Description|
|---|---|
|`200`|A list of rejected push keys.|

### 200 response

|Name|Type|Description|
|---|---|---|
|`rejected`|`[string]`|**Required:** A list of all pushkeys given in the notification request that are not valid. These could have been rejected by an upstream gateway because they have expired or have never been valid. Homeservers must cease sending notification requests for these pushkeys and remove the associated pushers. It may not necessarily be the notification in the request that failed: it could be that a previous notification to the same pushkey failed. May be empty.|

```
{
  "rejected": [
    "V2h5IG9uIGVhcnRoIGRpZCB5b3UgZGVjb2RlIHRoaXM/"
  ]
}
```