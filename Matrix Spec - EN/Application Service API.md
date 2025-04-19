>  [Matrix Specification](https://spec.matrix.org/v1.11/) | [Application Service API](https://spec.matrix.org/v1.11/application-service-api/)

# Application Service API

The Matrix client-server API and server-server APIs provide the means to implement a consistent self-contained federated messaging fabric. However, they provide limited means of implementing custom server-side behaviour in Matrix (e.g. gateways, filters, extensible hooks etc). The Application Service API (AS API) defines a standard API to allow such extensible functionality to be implemented irrespective of the underlying homeserver implementation.

# Application Services[](https://spec.matrix.org/v1.11/application-service-api/#application-services)

Application services are passive and can only observe events from the homeserver. They can inject events into rooms they are participating in. They cannot prevent events from being sent, nor can they modify the content of the event being sent. In order to observe events from a homeserver, the homeserver needs to be configured to pass certain types of traffic to the application service. This is achieved by manually configuring the homeserver with information about the application service.

## Registration[](https://spec.matrix.org/v1.11/application-service-api/#registration)
> [!info] info:
> Previously, application services could register with a homeserver via HTTP APIs. This was removed as it was seen as a security risk. A compromised application service could re-register for a global `*` regex and sniff _all_ traffic on the homeserver. To protect against this, application services now have to register via configuration files which are linked to the homeserver configuration file. The addition of configuration files allows homeserver admins to sanity check the registration for suspicious regex strings.

Application services register “namespaces” of user IDs, room aliases and room IDs. An application service is said to be “interested” in a given event if it matches any of the namespaces.

An application service can also state whether they should be the only ones who can manage a specified namespace. This is referred to as an “exclusive” namespace. An exclusive namespace prevents humans and other application services from creating/deleting entities in that namespace. Typically, exclusive namespaces are used when the rooms represent real rooms on another service (e.g. IRC). Non-exclusive namespaces are used when the application service is merely augmenting the room itself (e.g. providing logging or searching facilities).

The registration is represented by a series of key-value pairs, which is normally encoded as an object in a YAML file. It has the following structure:

### `Registration`

---
|Registration|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`as_token`|`string`|**Required:** A secret token that the application service will use to authenticate requests to the homeserver.|
|`hs_token`|`string`|**Required:** A secret token that the homeserver will use authenticate requests to the application service.|
|`id`|`string`|**Required:** A unique, user-defined ID of the application service which will never change.|
|`namespaces`|[Namespaces](https://spec.matrix.org/v1.11/application-service-api/#definition-registration_namespaces)|**Required:** The namespaces that the application service is interested in.|
|`protocols`|`[string]`|The external protocols which the application service provides (e.g. IRC).|
|`rate_limited`|`boolean`|Whether requests from masqueraded users are rate-limited. The sender is excluded.|
|`sender_localpart`|`string`|**Required:** The localpart of the user associated with the application service. Events will be sent to the AS if this user is the target of the event, or is a joined member of the room where the event occurred.|
|`url`|`string`|**Required:** The URL for the application service. May include a path after the domain name. Optionally set to null if no traffic is required.|

|Namespaces|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`aliases`|[[Namespace](https://spec.matrix.org/v1.11/application-service-api/#definition-registration_namespace)]|A list of namespaces defining the room aliases that the application service is interested in. All events sent in a room with an alias which matches one of the namespaces will be sent to the AS.|
|`rooms`|[[Namespace](https://spec.matrix.org/v1.11/application-service-api/#definition-registration_namespace)]|A list of namespaces defining the room IDs that the application service is interested in. All events sent in a room with an ID which matches one of the namespaces will be sent to the AS.|
|`users`|[[Namespace](https://spec.matrix.org/v1.11/application-service-api/#definition-registration_namespace)]|A list of namespaces defining the user IDs that the application service is interested in, in addition to its `sender_localpart`. Events will be sent to the AS if a local user matching one of the namespaces is the target of the event, or is a joined member of the room where the event occurred.|

|Namespace|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`exclusive`|`boolean`|**Required:** A true or false value stating whether this application service has exclusive access to events within this namespace.|
|`regex`|`string`|**Required:** A POSIX regular expression defining which values this namespace includes.|

#### Examples

```
{}
```

Exclusive user and alias namespaces should begin with an underscore after the sigil to avoid collisions with other users on the homeserver. Application services should additionally attempt to identify the service they represent in the reserved namespace. For example, `@_irc_.*` would be a good namespace to register for an application service which deals with IRC.

An example registration file for an IRC-bridging application service is below:

```
id: "IRC Bridge"
url: "http://127.0.0.1:1234"
as_token: "30c05ae90a248a4188e620216fa72e349803310ec83e2a77b34fe90be6081f46"
hs_token: "312df522183efd404ec1cd22d2ffa4bbc76a8c1ccf541dd692eef281356bb74e"
sender_localpart: "_irc_bot" # Will result in @_irc_bot:example.org
namespaces:
  users:
    - exclusive: true
      regex: "@_irc_bridge_.*"
  aliases:
    - exclusive: false
      regex: "#_irc_bridge_.*"
  rooms: []
```

> [!info] info:
> For the `users` namespace, application services can only register interest in _local_ users (i.e., users whose IDs end with the `server_name` of the local homeserver). Events affecting users on other homeservers are not sent to an application service, even if the user happens to match the one of the `users` namespaces (unless, of course, the event affects a room that the application service is interested in for another room - for example, because there is another user in the room that the application service is interested in).
> 
> For the `rooms` and `aliases` namespaces, all events in a matching room will be sent to the application service.

> [!warning] warning:
> If the homeserver in question has multiple application services, each `as_token` and `id` MUST be unique per application service as these are used to identify the application service. The homeserver MUST enforce this.

## Homeserver -> Application Service API[](https://spec.matrix.org/v1.11/application-service-api/#homeserver--gt-application-service-api)

### Authorization[](https://spec.matrix.org/v1.11/application-service-api/#authorization)

**[Changed in `v1.4`]**

Homeservers MUST include an `Authorization` header, containing the `hs_token` from the application service’s registration, when making requests to the application service. Application services MUST verify that the provided `Bearer` token matches their known `hs_token`, failing the request with an `M_FORBIDDEN` error if it does not match.

The format of the `Authorization` header is similar to the [Client-Server API](https://spec.matrix.org/v1.11/client-server-api/#client-authentication): `Bearer TheHSTokenGoesHere`.

> [!info] info:
> In previous versions of this specification, an `access_token` query parameter was used instead. Servers should only send this query parameter if supporting legacy versions of the specification.
> 
> If sending the `query_string`, it is encouraged to send it alongside the `Authorization` header for maximum compatibility.
>
> Application services should ensure both match if both are provided.

### Legacy routes[](https://spec.matrix.org/v1.11/application-service-api/#legacy-routes)

Previous drafts of the application service specification had a mix of endpoints that have been used in the wild for a significant amount of time. The application service specification now defines a version on all endpoints to be more compatible with the rest of the Matrix specification and the future.

Homeservers should attempt to use the specified endpoints first when communicating with application services. However, if the application service receives an HTTP status code that does not indicate success (i.e.: 404, 500, 501, etc) then the homeserver should fall back to the older endpoints for the application service.

The older endpoints have the exact same request body and response format, they just belong at a different path. The equivalent path for each is as follows:

- `/_matrix/app/v1/transactions/{txnId}` should fall back to `/transactions/{txnId}`
- `/_matrix/app/v1/users/{userId}` should fall back to `/users/{userId}`
- `/_matrix/app/v1/rooms/{roomAlias}` should fall back to `/rooms/{roomAlias}`
- `/_matrix/app/v1/thirdparty/protocol/{protocol}` should fall back to `/_matrix/app/unstable/thirdparty/protocol/{protocol}`
- `/_matrix/app/v1/thirdparty/user/{user}` should fall back to `/_matrix/app/unstable/thirdparty/user/{user}`
- `/_matrix/app/v1/thirdparty/location/{location}` should fall back to `/_matrix/app/unstable/thirdparty/location/{location}`
- `/_matrix/app/v1/thirdparty/user` should fall back to `/_matrix/app/unstable/thirdparty/user`
- `/_matrix/app/v1/thirdparty/location` should fall back to `/_matrix/app/unstable/thirdparty/location`

Homeservers should periodically try again for the newer endpoints because the application service may have been updated.

### Unknown routes[](https://spec.matrix.org/v1.11/application-service-api/#unknown-routes)

If a request for an unsupported (or unknown) endpoint is received then the server must respond with a 404 `M_UNRECOGNIZED` error.

Similarly, a 405 `M_UNRECOGNIZED` error is used to denote an unsupported method to a known endpoint.

### Pushing events[](https://spec.matrix.org/v1.11/application-service-api/#pushing-events)

The application service API provides a transaction API for sending a list of events. Each list of events includes a transaction ID, which works as follows:

```
    Typical
    HS ---> AS : Homeserver sends events with transaction ID T.
       <---    : Application Service sends back 200 OK.
```

```
    AS ACK Lost
    HS ---> AS : Homeserver sends events with transaction ID T.
       <-/-    : AS 200 OK is lost.
    HS ---> AS : Homeserver retries with the same transaction ID of T.
       <---    : Application Service sends back 200 OK. If the AS had processed these
                 events already, it can NO-OP this request (and it knows if it is the
                 same events based on the transaction ID).
```

The events sent to the application service should be linearised, as if they were from the event stream. The homeserver MUST maintain a queue of transactions to send to the application service. If the application service cannot be reached, the homeserver SHOULD backoff exponentially until the application service is reachable again. As application services cannot _modify_ the events in any way, these requests can be made without blocking other aspects of the homeserver. Homeservers MUST NOT alter (e.g. add more) events they were going to send within that transaction ID on retries, as the application service may have already processed the events.

#### PUT /_matrix/app/v1/transactions/{txnId} [](https://spec.matrix.org/v1.11/application-service-api/#put_matrixappv1transactionstxnid) 

---

This API is called by the homeserver when it wants to push an event (or batch of events) to the application service.

Note that the application service should distinguish state events from message events via the presence of a `state_key`, rather than via the event type.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters
|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`txnId`|`string`|**Required:** The transaction ID for this set of events. Homeservers generate these IDs and they are used to ensure idempotency of requests.|

###### Request body

|Name|Type|Description|
|---|---|---|
|`events`|[[ClientEvent](https://spec.matrix.org/v1.11/application-service-api/#put_matrixappv1transactionstxnid_request_clientevent)]|**Required:** A list of events, formatted as per the Client-Server API.|

|ClientEvent|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`content`|`object`|**Required:** The body of this event, as created by the client which sent it.|
|`event_id`|`string`|**Required:** The globally unique identifier for this event.|
|`origin_server_ts`|`integer`|**Required:** Timestamp (in milliseconds since the unix epoch) on originating homeserver when this event was sent.|
|`room_id`|`string`|**Required:** The ID of the room associated with this event.|
|`sender`|`string`|**Required:** Contains the fully-qualified ID of the user who sent this event.|
|`state_key`|`string`|Present if, and only if, this event is a _state_ event. The key making this piece of state unique in the room. Note that it is often an empty string.<br><br>State keys starting with an `@` are reserved for referencing user IDs, such as room members. With the exception of a few events, state events set with a given user’s ID as the state key MUST only be set by that user.|
|`type`|`string`|**Required:** The type of the event.|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/application-service-api/#put_matrixappv1transactionstxnid_request_unsigneddata)|Contains optional extra information about the event.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The time in milliseconds that has elapsed since the event was sent. This field is generated by the local homeserver, and may be incorrect if the local time on at least one of the two servers is out of sync, which can cause the age to either be negative or greater than it actually is.|
|`membership`|`string`|The room membership of the user making the request, at the time of the event.<br><br>This property is the value of the `membership` property of the requesting user’s [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) state at the point of the event, including any changes caused by the event. If the user had yet to join the room at the time of the event (i.e, they have no `m.room.member` state), this property is set to `leave`.<br><br>Homeservers SHOULD populate this property wherever practical, but they MAY omit it if necessary (for example, if calculating the value is expensive, servers might choose to only implement it in encrypted rooms). The property is _not_ normally populated in events pushed to application services via the application service transaction API (where there is no clear definition of “requesting user”).<br><br>**Added in `v1.11`**|
|`prev_content`|`EventContent`|The previous `content` for this event. This field is generated by the local homeserver, and is only returned if the event is a state event, and the client has permission to see the previous content.  <br>  <br>**Changed in `v1.2`:** Previously, this field was specified at the top level of returned events rather than in `unsigned` (with the exception of the [`GET .../notifications`](https://spec.matrix.org/v1.11/client-server-api/#get_matrixclientv3notifications) endpoint), though in practice no known server implementations honoured this.|
|`redacted_because`|`ClientEvent`|The event that redacted this event, if any.|
|`transaction_id`|`string`|The client-supplied [transaction ID](https://spec.matrix.org/v1.11/client-server-api/#transaction-identifiers), for example, provided via `PUT /_matrix/client/v3/rooms/{roomId}/send/{eventType}/{txnId}`, if the client being given the event is the same one which sent it.|

###### Request body example

```
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

##### Responses

|Status|Description|
|---|---|
|`200`|The transaction was processed successfully.|

###### 200 response

```
{}
```

### Pinging[](https://spec.matrix.org/v1.11/application-service-api/#pinging)

**[Added in `v1.7`]**

The application service API includes a ping mechanism to allow appservices to ensure that the homeserver can reach the appservice. Appservices may use this mechanism to detect misconfigurations and report them appropriately.

Implementations using this mechanism should take care to not fail entirely in the event of temporary issues, e.g. gracefully handling cases where the appservice is started before the homeserver.

The mechanism works as follows (note: the human-readable `error` fields have been omitted for brevity):

**Typical**

```
AS ---> HS : /_matrix/client/v1/appservice/{appserviceId}/ping {"transaction_id": "meow"}
    HS ---> AS : /_matrix/app/v1/ping {"transaction_id": "meow"}
    HS <--- AS : 200 OK {}
AS <--- HS : 200 OK {"duration_ms": 123}
```

**Incorrect `hs_token`**

```
AS ---> HS : /_matrix/client/v1/appservice/{appserviceId}/ping {"transaction_id": "meow"}
    HS ---> AS : /_matrix/app/v1/ping {"transaction_id": "meow"}
    HS <--- AS : 403 Forbidden {"errcode": "M_FORBIDDEN"}
AS <--- HS : 502 Bad Gateway {"errcode": "M_BAD_STATUS", "status": 403, "body": "{\"errcode\": \"M_FORBIDDEN\"}"}
```

**Can’t connect to appservice**

```
AS ---> HS : /_matrix/client/v1/appservice/{appserviceId}/ping {"transaction_id": "meow"}
    HS -/-> AS : /_matrix/app/v1/ping {"transaction_id": "meow"}
AS <--- HS : 502 Bad Gateway {"errcode": "M_CONNECTION_FAILED"}
```

The `/_matrix/app/v1/ping` endpoint is described here. The [`/_matrix/client/v1/appservice/{appserviceId}/ping`](https://spec.matrix.org/v1.11/application-service-api/#post_matrixclientv1appserviceappserviceidping) endpoint is under the Client-Server API extensions section below.

#### POST /_matrix/app/v1/ping [](https://spec.matrix.org/v1.11/application-service-api/#post_matrixappv1ping) 

---

**Added in `v1.7`**

This API is called by the homeserver to ensure that the connection works and the `hs_token` the homeserver has is correct.

Currently this is only called by the homeserver as a direct result of the application service calling [`POST /_matrix/client/v1/appservice/{appserviceId}/ping`](https://spec.matrix.org/v1.11/application-service-api/#post_matrixclientv1appserviceappserviceidping).

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request body

|Name|Type|Description|
|---|---|---|
|`transaction_id`|`string`|A transaction ID for the ping, copied directly from the `POST /_matrix/client/v1/appservice/{appserviceId}/ping` call.|

###### Request body example

```
{
  "transaction_id": "mautrix-go_1683636478256400935_123"
}
```

---

##### Responses

|Status|Description|
|---|---|
|`200`|The provided `hs_token` is valid and the ping request was successful.|

###### 200 response

```
{}
```

### Querying[](https://spec.matrix.org/v1.11/application-service-api/#querying)

The application service API includes two querying APIs: for room aliases and for user IDs. The application service SHOULD create the queried entity if it desires. During this process, the application service is blocking the homeserver until the entity is created and configured. If the homeserver does not receive a response to this request, the homeserver should retry several times before timing out. This should result in an HTTP status 408 “Request Timeout” on the client which initiated this request (e.g. to join a room alias).

> [!info] RATIONALE:
> Blocking the homeserver and expecting the application service to create the entity using the client-server API is simpler and more flexible than alternative methods such as returning an initial sync style JSON blob and get the HS to provision the room/user. This also meant that there didn’t need to be a “backchannel” to inform the application service about information about the entity such as room ID to room alias mappings.

#### GET /_matrix/app/v1/users/{userId} [](https://spec.matrix.org/v1.11/application-service-api/#get_matrixappv1usersuserid) 

---

This endpoint is invoked by the homeserver on an application service to query the existence of a given user ID. The homeserver will only query user IDs inside the application service’s `users` namespace. The homeserver will send this request when it receives an event for an unknown user ID in the application service’s namespace, such as a room invite.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters
|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`userId`|`string`|**Required:** The user ID being queried.|

---

##### Responses

|Status|Description|
|---|---|
|`200`|The application service indicates that this user exists. The application service MUST create the user using the client-server API.|
|`401`|The homeserver has not supplied credentials to the application service. Optional error information can be included in the body of this response.|
|`403`|The credentials supplied by the homeserver were rejected.|
|`404`|The application service indicates that this user does not exist. Optional error information can be included in the body of this response.|

###### 200 response

```
{}
```

###### 401 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/rooms/{roomAlias} [](https://spec.matrix.org/v1.11/application-service-api/#get_matrixappv1roomsroomalias) 

---

This endpoint is invoked by the homeserver on an application service to query the existence of a given room alias. The homeserver will only query room aliases inside the application service’s `aliases` namespace. The homeserver will send this request when it receives a request to join a room alias within the application service’s namespace.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters
|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`roomAlias`|`string`|**Required:** The room alias being queried.|

---

##### Responses

|Status|Description|
|---|---|
|`200`|The application service indicates that this room alias exists. The application service MUST have created a room and associated it with the queried room alias using the client-server API. Additional information about the room such as its name and topic can be set before responding.|
|`401`|The homeserver has not supplied credentials to the application service. Optional error information can be included in the body of this response.|
|`403`|The credentials supplied by the homeserver were rejected.|
|`404`|The application service indicates that this room alias does not exist. Optional error information can be included in the body of this response.|

###### 200 response

```
{}
```

###### 401 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

### Third-party networks[](https://spec.matrix.org/v1.11/application-service-api/#third-party-networks)

Application services may declare which protocols they support via their registration configuration for the homeserver. These networks are generally for third-party services such as IRC that the application service is managing. Application services may populate a Matrix room directory for their registered protocols, as defined in the Client-Server API Extensions.

Each protocol may have several “locations” (also known as “third-party locations” or “3PLs”). A location within a protocol is a place in the third-party network, such as an IRC channel. Users of the third-party network may also be represented by the application service.

Locations and users can be searched by fields defined by the application service, such as by display name or other attribute. When clients request the homeserver to search in a particular “network” (protocol), the search fields will be passed along to the application service for filtering.

#### GET /_matrix/app/v1/thirdparty/location [](https://spec.matrix.org/v1.11/application-service-api/#get_matrixappv1thirdpartylocation) 

---

Retrieve an array of third-party network locations from a Matrix room alias.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters
|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`alias`|`string`|The Matrix room alias to look up.|

---

##### Responses

|Status|Description|
|---|---|
|`200`|All found third-party locations.|
|`401`|The homeserver has not supplied credentials to the application service. Optional error information can be included in the body of this response.|
|`403`|The credentials supplied by the homeserver were rejected.|
|`404`|No mappings were found with the given parameters.|

###### 200 response

Array of `Location`.

|Location|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`alias`|`string`|**Required:** An alias for a matrix room.|
|`fields`|`object`|**Required:** Information used to identify this third-party location.|
|`protocol`|`string`|**Required:** The protocol ID that the third-party location is a part of.|

```
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

###### 401 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/thirdparty/location/{protocol} [](https://spec.matrix.org/v1.11/application-service-api/#get_matrixappv1thirdpartylocationprotocol) 

---

Retrieve a list of Matrix portal rooms that lead to the matched third-party location.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters
|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`protocol`|`string`|**Required:** The protocol ID.|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`fields`|`{string: string}`|One or more custom fields that are passed to the application service to help identify the third-party location.|

---

##### Responses

|Status|Description|
|---|---|
|`200`|At least one portal room was found.|
|`401`|The homeserver has not supplied credentials to the application service. Optional error information can be included in the body of this response.|
|`403`|The credentials supplied by the homeserver were rejected.|
|`404`|No mappings were found with the given parameters.|

###### 200 response

Array of `Location`.

|Location|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`alias`|`string`|**Required:** An alias for a matrix room.|
|`fields`|`object`|**Required:** Information used to identify this third-party location.|
|`protocol`|`string`|**Required:** The protocol ID that the third-party location is a part of.|

```
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

###### 401 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/thirdparty/protocol/{protocol} [](https://spec.matrix.org/v1.11/application-service-api/#get_matrixappv1thirdpartyprotocolprotocol) 

---

This API is called by the homeserver when it wants to present clients with specific information about the various third-party networks that an application service supports.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters
|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`protocol`|`string`|**Required:** The protocol ID.|

---

##### Responses

|Status|Description|
|---|---|
|`200`|The protocol was found and metadata returned.|
|`401`|The homeserver has not supplied credentials to the application service. Optional error information can be included in the body of this response.|
|`403`|The credentials supplied by the homeserver were rejected.|
|`404`|No protocol was found with the given path.|

###### 200 response

|Protocol|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`field_types`|{string: [Field Type](https://spec.matrix.org/v1.11/application-service-api/#get_matrixappv1thirdpartyprotocolprotocol_response-200_field-type)}|**Required:**<br><br>The type definitions for the fields defined in the `user_fields` and `location_fields`. Each entry in those arrays MUST have an entry here. The `string` key for this object is field name itself.<br><br>May be an empty object if no fields are defined.|
|`icon`|`string`|**Required:** A content URI representing an icon for the third-party protocol.|
|`instances`|[[Protocol Instance](https://spec.matrix.org/v1.11/application-service-api/#get_matrixappv1thirdpartyprotocolprotocol_response-200_protocol-instance)]|**Required:** A list of objects representing independent instances of configuration. For example, multiple networks on IRC if multiple are provided by the same application service.|
|`location_fields`|`[string]`|**Required:** Fields which may be used to identify a third-party location. These should be ordered to suggest the way that entities may be grouped, where higher groupings are ordered first. For example, the name of a network should be searched before the name of a channel.|
|`user_fields`|`[string]`|**Required:** Fields which may be used to identify a third-party user. These should be ordered to suggest the way that entities may be grouped, where higher groupings are ordered first. For example, the name of a network should be searched before the nickname of a user.|

|Field Type|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`placeholder`|`string`|**Required:** An placeholder serving as a valid example of the field value.|
|`regexp`|`string`|**Required:** A regular expression for validation of a field’s value. This may be relatively coarse to verify the value as the application service providing this protocol may apply additional validation or filtering.|

|Protocol Instance|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`desc`|`string`|**Required:** A human-readable description for the protocol, such as the name.|
|`fields`|`object`|**Required:** Preset values for `fields` the client may use to search by.|
|`icon`|`string`|An optional content URI representing the protocol. Overrides the one provided at the higher level Protocol object.|
|`network_id`|`string`|**Required:** A unique identifier across all instances.|

```
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

###### 401 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/thirdparty/user [](https://spec.matrix.org/v1.11/application-service-api/#get_matrixappv1thirdpartyuser) 

---

Retrieve an array of third-party users from a Matrix User ID.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters
|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`userid`|`string`|The Matrix User ID to look up.|

---

##### Responses

|Status|Description|
|---|---|
|`200`|An array of third-party users.|
|`401`|The homeserver has not supplied credentials to the application service. Optional error information can be included in the body of this response.|
|`403`|The credentials supplied by the homeserver were rejected.|
|`404`|No mappings were found with the given parameters.|

###### 200 response

Array of `User`.

|User|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`fields`|`object`|**Required:** Information used to identify this third-party location.|
|`protocol`|`string`|**Required:** The protocol ID that the third-party location is a part of.|
|`userid`|`string`|**Required:** A Matrix User ID represting a third-party user.|

```
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

###### 401 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

#### GET /_matrix/app/v1/thirdparty/user/{protocol} [](https://spec.matrix.org/v1.11/application-service-api/#get_matrixappv1thirdpartyuserprotocol) 

---

This API is called by the homeserver in order to retrieve a Matrix User ID linked to a user on the third-party network, given a set of user parameters.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`protocol`|`string`|**Required:** The protocol ID.|

|query parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`fields`|`{string: string}`|One or more custom fields that are passed to the application service to help identify the user.|

---

##### Responses

|Status|Description|
|---|---|
|`200`|The Matrix User IDs found with the given parameters.|
|`401`|The homeserver has not supplied credentials to the application service. Optional error information can be included in the body of this response.|
|`403`|The credentials supplied by the homeserver were rejected.|
|`404`|No users were found with the given parameters.|

###### 200 response

Array of `User`.

|User|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`fields`|`object`|**Required:** Information used to identify this third-party location.|
|`protocol`|`string`|**Required:** The protocol ID that the third-party location is a part of.|
|`userid`|`string`|**Required:** A Matrix User ID represting a third-party user.|

```
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

###### 401 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_UNAUTHORIZED"
}
```

###### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_FORBIDDEN"
}
```

###### 404 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "COM.EXAMPLE.MYAPPSERVICE_NOT_FOUND"
}
```

## Client-Server API Extensions[](https://spec.matrix.org/v1.11/application-service-api/#client-server-api-extensions)

Application services can use a more powerful version of the client-server API by identifying itself as an application service to the homeserver.

Endpoints defined in this section MUST be supported by homeservers in the client-server API as accessible only by application services.

### Identity assertion[](https://spec.matrix.org/v1.11/application-service-api/#identity-assertion)

The client-server API infers the user ID from the `access_token` provided in every request. To avoid the application service from having to keep track of each user’s access token, the application service should identify itself to the Client-Server API by providing its `as_token` for the `access_token` alongside the user the application service would like to masquerade as.

Inputs:

- Application service token (`as_token`)
- User ID in the AS namespace to act as.

Notes:

- This applies to all aspects of the Client-Server API, except for Account Management.
- The `as_token` is inserted into `access_token` which is usually where the client token is, such as via the query string or `Authorization` header. This is done on purpose to allow application services to reuse client SDKs.
- The `access_token` should be supplied through the `Authorization` header where possible to prevent the token appearing in HTTP request logs by accident.

The application service may specify the virtual user to act as through use of a `user_id` query string parameter on the request. The user specified in the query string must be covered by one of the application service’s `user` namespaces. If the parameter is missing, the homeserver is to assume the application service intends to act as the user implied by the `sender_localpart` property of the registration.

An example request would be:

```
GET /_matrix/client/v3/account/whoami?user_id=@_irc_user:example.org
Authorization: Bearer YourApplicationServiceTokenHere
```

### Timestamp massaging[](https://spec.matrix.org/v1.11/application-service-api/#timestamp-massaging)

**[Added in `v1.3`]**

Application services can alter the timestamp associated with an event, allowing the application service to better represent the “real” time an event was sent at. While this doesn’t affect the server-side ordering of the event, it can allow an application service to better represent when an event would have been sent/received at, such as in the case of bridges where the remote network might have a slight delay and the application service wishes to bridge the proper time onto the message.

When authenticating requests as an application service, the caller can append a `ts` query string argument to change the `origin_server_ts` of the resulting event. Attempting to set the timestamp to anything other than what is accepted by `origin_server_ts` should be rejected by the server as a bad request.

When not present, the server’s behaviour is unchanged: the local system time of the server will be used to provide a timestamp, representing “now”.

The `ts` query string argument is only valid on the following endpoints:

- [`PUT /rooms/{roomId}/send/{eventType}/{txnId}`](https://spec.matrix.org/v1.11/client-server-api/#put_matrixclientv3roomsroomidsendeventtypetxnid)
- [`PUT /rooms/{roomId}/state/{eventType}/{stateKey}`](https://spec.matrix.org/v1.11/client-server-api/#put_matrixclientv3roomsroomidstateeventtypestatekey)

Other endpoints, such as `/kick`, do not support `ts`: instead, callers can use the `PUT /state` endpoint to mimic the behaviour of the other APIs.

> [!warning] warning:
> Changing the time of an event does not change the server-side (DAG) ordering for the event. The event will still be appended at the tip of the DAG as though the timestamp was set to “now”. Future MSCs, like [MSC2716](https://github.com/matrix-org/matrix-spec-proposals/pull/2716), are expected to provide functionality which can allow DAG order manipulation (for history imports and similar behaviour).

### Server admin style permissions[](https://spec.matrix.org/v1.11/application-service-api/#server-admin-style-permissions)

The homeserver needs to give the application service _full control_ over its namespace, both for users and for room aliases. This means that the AS should be able to manage any users and room alias in its namespace. No additional API changes need to be made in order for control of room aliases to be granted to the AS.

Creation of users needs API changes in order to:

- Work around captchas.
- Have a ‘passwordless’ user.

This involves bypassing the registration flows entirely. This is achieved by including the `as_token` on a `/register` request, along with a login type of `m.login.application_service` to set the desired user ID without a password.

```
POST /_matrix/client/v3/register
Authorization: Bearer YourApplicationServiceTokenHere

Content:
{
  type: "m.login.application_service",
  username: "_irc_example"
}
```

Similarly, logging in as users needs API changes in order to allow the AS to log in without needing the user’s password. This is achieved by including the `as_token` on a `/login` request, along with a login type of `m.login.application_service`:

**[Added in `v1.2`]**

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

Application services which attempt to create users or aliases _outside_ of their defined namespaces, or log in as users outside of their defined namespaces will receive an error code `M_EXCLUSIVE`. Similarly, normal users who attempt to create users or aliases _inside_ an application service-defined namespace will receive the same `M_EXCLUSIVE` error code, but only if the application service has defined the namespace as `exclusive`.

If `/register` or `/login` is called with the `m.login.application_service` login type, but without a valid `as_token`, the endpoints will return an error with the `M_MISSING_TOKEN` or `M_UNKNOWN_TOKEN` error code and 401 as the HTTP status code. This is the same behavior as invalid auth in the client-server API (see [Using access tokens](https://spec.matrix.org/v1.11/client-server-api/#using-access-tokens)).

### Pinging[](https://spec.matrix.org/v1.11/application-service-api/#pinging-1)

**[Added in `v1.7`]**

This is the client-server API companion endpoint for the [pinging](https://spec.matrix.org/v1.11/application-service-api/#pinging) mechanism described above.

#### POST /_matrix/client/v1/appservice/{appserviceId}/ping [](https://spec.matrix.org/v1.11/application-service-api/#post_matrixclientv1appserviceappserviceidping) 

---

**Added in `v1.7`**

This API asks the homeserver to call the [`/_matrix/app/v1/ping`](https://spec.matrix.org/v1.11/application-service-api/#post_matrixappv1ping) endpoint on the application service to ensure that the homeserver can communicate with the application service.

This API requires the use of an application service access token (`as_token`) instead of a typical client’s access token. This API cannot be invoked by users who are not identified as application services. Additionally, the appservice ID in the path must be the same as the appservice whose `as_token` is being used.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters
|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`appserviceId`|`string`|**Required:** The appservice ID of the appservice to ping. This must be the same as the appservice whose `as_token` is being used to authenticate the request.|

###### Request body

|Name|Type|Description|
|---|---|---|
|`transaction_id`|`string`|An optional transaction ID that is passed through to the `/_matrix/app/v1/ping` call.|

###### Request body example

```
{
  "transaction_id": "mautrix-go_1683636478256400935_123"
}
```

---

##### Responses

|Status|Description|
|---|---|
|`200`|The ping was successful.|
|`400`|The application service doesn’t have a URL configured. The errcode is `M_URL_NOT_SET`.|
|`403`|The access token used to authenticate the request doesn’t belong to an appservice, or belongs to a different appservice than the one in the path. The errcode is `M_FORBIDDEN`.|
|`502`|The application service returned a bad status, or the connection failed. The errcode is `M_BAD_STATUS` or `M_CONNECTION_FAILED`.<br><br>For bad statuses, the response may include `status` and `body` fields containing the HTTP status code and response body text respectively to aid with debugging.|
|`504`|The connection to the application service timed out. The errcode is `M_CONNECTION_TIMEOUT`.|

###### 200 response

|Name|Type|Description|
|---|---|---|
|`duration_ms`|`integer`|**Required:** The duration in milliseconds that the [`/_matrix/app/v1/ping`](https://spec.matrix.org/v1.11/application-service-api/#post_matrixappv1ping) request took from the homeserver’s point of view.|

```
{
  "duration_ms": 123
}
```

###### 400 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_URL_NOT_SET",
  "error": "Application service doesn't have a URL configured"
}
```

###### 403 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_FORBIDDEN",
  "error": "Provided access token is not the appservice's as_token"
}
```

###### 502 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`body`|`string`|The HTTP response body returned by the appservice.|
|`errcode`|`string`|**Required:** An error code.<br><br>One of: `[M_BAD_STATUS, M_CONNECTION_FAILED]`.|
|`error`|`string`|A human-readable error message.|
|`status`|`integer`|The HTTP status code returned by the appservice.|

```
{
  "body": "{\"errcode\": \"M_UNKNOWN_TOKEN\"}",
  "errcode": "M_BAD_STATUS",
  "error": "Ping returned status 401",
  "status": 401
}
```

###### 504 response
|Error|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`errcode`|`string`|**Required:** An error code.|
|`error`|`string`|A human-readable error message.|

```
{
  "errcode": "M_CONNECTION_TIMEOUT",
  "error": "Connection to application service timed out"
}
```

### Using `/sync` and `/events`[](https://spec.matrix.org/v1.11/application-service-api/#using-sync-and-events)

Application services wishing to use `/sync` or `/events` from the Client-Server API MUST do so with a virtual user (provide a `user_id` via the query string). It is expected that the application service use the transactions pushed to it to handle events rather than syncing with the user implied by `sender_localpart`.

### Application service room directories[](https://spec.matrix.org/v1.11/application-service-api/#application-service-room-directories)

Application services can maintain their own room directories for their defined third-party protocols. These room directories may be accessed by clients through additional parameters on the `/publicRooms` client-server endpoint.

#### PUT /_matrix/client/v3/directory/list/appservice/{networkId}/{roomId} [](https://spec.matrix.org/v1.11/application-service-api/#put_matrixclientv3directorylistappservicenetworkidroomid) 

---

Updates the visibility of a given room on the application service’s room directory.

This API is similar to the room directory visibility API used by clients to update the homeserver’s more general room directory.

This API requires the use of an application service access token (`as_token`) instead of a typical client’s access_token. This API cannot be invoked by users who are not identified as application services.

|   |   |
|---|---|
|Rate-limited:|No|
|Requires authentication:|Yes|

---

##### Request

###### Request parameters

|path parameters|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`networkId`|`string`|**Required:** The protocol (network) ID to update the room list for. This would have been provided by the application service as being listed as a supported protocol.|
|`roomId`|`string`|**Required:** The room ID to add to the directory.|

###### Request body

|Name|Type|Description|
|---|---|---|
|`visibility`|`string`|**Required:** Whether the room should be visible (public) in the directory or not (private).<br><br>One of: `[public, private]`.|

###### Request body example

```
{
  "visibility": "public"
}
```

---

##### Responses

|Status|Description|
|---|---|
|`200`|The room’s directory visibility has been updated.|

###### 200 response

```
{}
```

## Referencing messages from a third-party network [](https://spec.matrix.org/v1.11/application-service-api/#referencing-messages-from-a-third-party-network)

Application services should include an `external_url` in the `content` of events it emits to indicate where the message came from. This typically applies to application services that bridge other networks into Matrix, such as IRC, where an HTTP URL may be available to reference.

Clients should provide users with a way to access the `external_url` if it is present. Clients should additionally ensure the URL has a scheme of `https` or `http` before making use of it.

The presence of an `external_url` on an event does not necessarily mean the event was sent from an application service. Clients should be wary of the URL contained within, as it may not be a legitimate reference to the event’s source.