> [Matrix Specification](https://spec.matrix.org/v1.11/) | [Room Versions](https://spec.matrix.org/v1.11/rooms/)

# Room Versions

Rooms are central to how Matrix operates, and have strict rules for what is allowed to be contained within them. Rooms can also have various algorithms that handle different tasks, such as what to do when two or more events collide in the underlying DAG. To allow rooms to be improved upon through new algorithms or rules, “room versions” are employed to manage a set of expectations for each room. New room versions are assigned as needed.

There is no implicit ordering or hierarchy to room versions, and their principles are immutable once placed in the specification. Although there is a recommended set of versions, some rooms may benefit from features introduced by other versions. Rooms move between different versions by “upgrading” to the desired version. Due to versions not being ordered or hierarchical, this means a room can “upgrade” from version 2 to version 1, if it is so desired.

### Feature matrix[](https://spec.matrix.org/v1.11/rooms/#feature-matrix)

Some functionality is only available in specific room versions, such as knocking. The table below shows which versions support which features from a client’s perspective. Server implementations are still welcome to reference the following table, however the detailed per-version specifications are more likely to be of interest.


|Feature \ Version|1|2|3|4|5|6|7|8|9|10|11|
|---|---|---|---|---|---|---|---|---|---|---|---|
|**Knocking**|❌|❌|❌|❌|❌|❌|✔|✔|✔|✔|✔|
|**Restricted join rules**|❌|❌|❌|❌|❌|❌|❌|✔|✔|✔|✔|
|**`knock_restricted` join rule**|❌|❌|❌|❌|❌|❌|❌|❌|❌|✔|✔|

### Complete list of room versions[](https://spec.matrix.org/v1.11/rooms/#complete-list-of-room-versions)

Room versions are divided into two distinct groups: stable and unstable. Stable room versions may be used by rooms safely. Unstable room versions are everything else which is either not listed in the specification or flagged as unstable for some other reason. Versions can switch between stable and unstable periodically for a variety of reasons, including discovered security vulnerabilities and age.

Clients should not ask room administrators to upgrade their rooms if the room is running a stable version. Servers SHOULD use **room version 10** as the default room version when creating new rooms.

The available room versions are:

- [Version 1](https://spec.matrix.org/v1.11/rooms/v1) - **Stable**. The initial room version.
- [Version 2](https://spec.matrix.org/v1.11/rooms/v2) - **Stable**. Implements State Resolution Version 2.
- [Version 3](https://spec.matrix.org/v1.11/rooms/v3) - **Stable**. Introduces events whose IDs are the event’s hash.
- [Version 4](https://spec.matrix.org/v1.11/rooms/v4) - **Stable**. Builds on v3 by using URL-safe base64 for event IDs.
- [Version 5](https://spec.matrix.org/v1.11/rooms/v5) - **Stable**. Introduces enforcement of signing key validity periods.
- [Version 6](https://spec.matrix.org/v1.11/rooms/v6) - **Stable**. Alters several authorization rules for events.
- [Version 7](https://spec.matrix.org/v1.11/rooms/v7) - **Stable**. Introduces knocking.
- [Version 8](https://spec.matrix.org/v1.11/rooms/v8) - **Stable**. Adds a join rule to allow members of another room to join without invite.
- [Version 9](https://spec.matrix.org/v1.11/rooms/v9) - **Stable**. Builds on v8 to fix issues when redacting some membership events.
- [Version 10](https://spec.matrix.org/v1.11/rooms/v10) - **Stable**. Enforces integer-only power levels and adds `knock_restricted` join rule.
- [Version 11](https://spec.matrix.org/v1.11/rooms/v11) - **Stable**. Clarifies the redaction algorithm.

### Room version grammar[](https://spec.matrix.org/v1.11/rooms/#room-version-grammar)

Room versions are used to change properties of rooms that may not be compatible with other servers. For example, changing the rules for event authorization would cause older servers to potentially end up in a split-brain situation due to not understanding the new rules.

A room version is defined as a string of characters which MUST NOT exceed 32 codepoints in length. Room versions MUST NOT be empty and MUST contain only the characters `a-z`, `0-9`, `.`, and `-`.

Room versions are not intended to be parsed and should be treated as opaque identifiers. Room versions consisting only of the characters `0-9` and `.` are reserved for future versions of the Matrix protocol.

The complete grammar for a legal room version is:

```
room_version = 1*room_version_char
room_version_char = DIGIT
                  / %x61-7A         ; a-z
                  / "-" / "."
```

Examples of valid room versions are:

- `1` (would be reserved by the Matrix protocol)
- `1.2` (would be reserved by the Matrix protocol)
- `1.2-beta`
- `com.example.version`

## Room Version 1

This room version is the first ever version for rooms, and contains the building blocks for other room versions.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v1/#client-considerations)

Clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v1/#redactions) section below.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v1/#redactions)

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows key `membership`.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows key `creator`.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows key `join_rule`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.aliases`](https://spec.matrix.org/v1.11/client-server-api#historical-events) allows key `aliases`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v1/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

The algorithms defined here should only apply to version 1 rooms. Other algorithms may be used by other room versions, and as such servers should be aware of which version room they are dealing with prior to executing a given algorithm.

> [!warning] warning:
> Although there are many rooms using room version 1, it is known to have undesirable effects. Servers implementing support for room version 1 should be aware that restrictions should be generally relaxed and that inconsistencies may occur.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v1/#redactions-1)

[See above](https://spec.matrix.org/v1.11/rooms/v1/#redactions).

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v1/#event-ids)

An event has exactly one event ID. Event IDs in this room version have the format:

```
$opaque_id:domain
```

where `domain` is the [server name](https://spec.matrix.org/v1.11/appendices/#server-name) of the homeserver which created the room, and `opaque_id` is a locally-unique string.

The domain is used only for namespacing to avoid the risk of clashes of identifiers between different homeservers. There is no implication that the room or event in question is still available at the corresponding homeserver.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v1/#event-format)

Events in version 1 rooms have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room versions 1 and 2.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|[[string\|[Event Hash](https://spec.matrix.org/v1.11/rooms/v1/#definition-persistent-data-unit_event-hash)]]|**Required:**<br><br>Event IDs and reference hashes for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`event_id`|`string`|**Required:** The event ID for the PDU.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v1/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|[[string\|[Event Hash](https://spec.matrix.org/v1.11/rooms/v1/#definition-persistent-data-unit_event-hash)]]|**Required:**<br><br>Event IDs and reference hashes for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v1/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$af232176:example.org",
    {
      "sha256": "abase64encodedsha256hashshouldbe43byteslong"
    }
  ],
  "content": {
    "key": "value"
  },
  "depth": 12,
  "event_id": "$a4ecee13e2accdadf56c1025:example.com",
  "hashes": {
    "sha256": "thishashcoversallfieldsincasethisisredacted"
  },
  "origin": "example.com",
  "origin_server_ts": 1404838188000,
  "prev_events": [
    "$af232176:example.org",
    {
      "sha256": "abase64encodedsha256hashshouldbe43byteslong"
    }
  ],
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v1/#deprecated-event-content-schemas)

Events sent into rooms of this version can have formats which are different from their normal schema. Those cases are documented here.

> [!warning] warning:
> The behaviour described here is preserved strictly for backwards compatibility only. A homeserver should take reasonable precautions to prevent users from sending these so-called “malformed” events, and must never rely on the behaviours described here as a default.

###### `m.room.power_levels` events accept values as strings[](https://spec.matrix.org/v1.11/rooms/v1/#mroompower_levels-events-accept-values-as-strings)

In order to maintain backwards compatibility with early implementations, each of the integer-valued properties within [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) events can be encoded as strings instead of integers. This includes the nested values within the `events`, `notifications` and `users` properties. For example, the following is a valid `m.room.power_levels` event in this room version:

```
{
  "content": {
    "ban": "50",
    "events": {
      "m.room.power_levels": "100"
    },
    "events_default": "0",
    "state_default": "50",
    "users": {
      "@example:localhost": "100"
    },
    "users_default": "0"
  },
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "state_key": "",
  "type": "m.room.power_levels"
}
```

When the value is representative of an integer, they must be the following format:

- a single base 10 integer, no float values or decimal points, optionally with any number of leading zeroes (`"100"`, `"000100"`);
- optionally prefixed with a single `-` or `+` character before the integer (`"+100"`, `"-100"`).
- optionally with any number of leading or trailing whitespace characters (`" 100 "`, `" 00100 "`, `" +100 "`, `" -100 "`);

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules)

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

The rules are as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Considering the event’s `auth_events`:
    1. If there are duplicate entries for a given `type` and `state_key` pair, reject.
    2. If there are entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification, reject.
    3. If there are entries which were themselves rejected under the [checks performed on receipt of a PDU](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu), reject.
    4. If there is no `m.room.create` event among the entries, reject.
3. If the `content` of the `m.room.create` event in the room state has the property `m.federate` set to `false`, and the `sender` domain of the event does not match the `sender` domain of the create event, reject.
4. If type is `m.room.aliases`:
    1. If event has no `state_key`, reject.
    2. If sender’s domain doesn’t matches `state_key`, reject.
    3. Otherwise, allow.
5. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` then allow if membership state is `invite` or `join`.
        5. If the `join_rule` is `public`, allow.
        6. Otherwise, reject.
    3. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    4. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite` or `join`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    5. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    6. Otherwise, the membership is unknown. Reject.
6. If the `sender`’s current membership state is not `join`, reject.
7. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
8. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
9. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
10. If type is `m.room.power_levels`:
    1. If the `users` property in `content` is not an object with keys that are valid user IDs with values that are integers (or a string that is an integer), reject.
    2. If there is no previous `m.room.power_levels` event in the room, allow.
    3. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is greater than the `sender`’s current power level, reject.
        2. If the new value is greater than the `sender`’s current power level, reject.
    4. For each entry being changed in, or removed from, the `events` property:
        1. If the current value is greater than the `sender`’s current power level, reject.
    5. For each entry being added to, or changed in, the `events` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. Otherwise, allow.
11. If type is `m.room.redaction`:
    1. If the `sender`’s power level is greater than or equal to the _redact level_, allow.
    2. If the domain of the `event_id` of the event being redacted is the same as the domain of the `event_id` of the `m.room.redaction`, allow.
    3. Otherwise, reject.
12. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v1/#state-resolution)

> [!warning] warning:
> Room version 1 is known to have bugs that can cause the state of rooms to reset to older versions of the room’s state. For example this could mean that users who had joined the room may be removed from the room, admins and moderators could lose their power level, and users who have been banned from the room may be able to rejoin. Other state events such as the the room’s name or topic could also reset to a previous version.
> 
> This is fixed in the state resolution algorithm introduced in room version 2.

The room state _S′_(_E_) after an event _E_ is defined in terms of the room state _S_(_E_) before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E′)_, _S′(E″)_, …} after the `prev_events` {_E′_, _E″_, …}. of _E_.

The _resolution_ of a set of states is defined as follows. The resolved state is built up in a number of passes; here we use _R_ to refer to the results of the resolution so far.

- Start by setting _R_ to the union of the states to be resolved, excluding any _conflicting_ events.
- First we resolve conflicts between `m.room.power_levels` events. If there is no conflict, this step is skipped, otherwise:
    - Assemble all the `m.room.power_levels` events from the states to be resolved into a list.
    - Sort the list by ascending `depth` then descending `sha1(event_id)`.
    - Add the first event in the list to _R_.
    - For each subsequent event in the list, check that the event would be allowed by the authorization rules for a room in state _R_. If the event would be allowed, then update _R_ with the event and continue with the next event in the list. If it would not be allowed, stop and continue below with `m.room.join_rules` events.
- Repeat the above process for conflicts between `m.room.join_rules` events.
- Repeat the above process for conflicts between `m.room.member` events.
- No other events affect the authorization rules, so for all other conflicts, just pick the event with the highest depth and lowest `sha1(event_id)` that passes authentication in _R_ and add it to _R_.

A _conflict_ occurs between states where those states have different `event_ids` for the same `(event_type, state_key)`. The events thus affected are said to be _conflicting_ events.

#### Canonical JSON[](https://spec.matrix.org/v1.11/rooms/v1/#canonical-json)

Servers MUST NOT strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json) for the reasons described there.

## Room Version 2

This room version builds on [version 1](https://spec.matrix.org/v1.11/rooms/v1) with an improved state resolution algorithm.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v2/#client-considerations)

There are no client considerations introduced in this room version. Clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v2/#redactions) section below for a full overview of the algorithm.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v2/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the details contained here, and can safely ignore their presence.

Room version 2 uses the base components of [room version 1](https://spec.matrix.org/v1.11/rooms/v1), changing only the state resolution algorithm.

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v2/#state-resolution)

**[New in this version]**

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v2/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v2/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v2/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

### Unchanged from v1[](https://spec.matrix.org/v1.11/rooms/v2/#unchanged-from-v1)

The following sections have not been modified since v1, but are included for completeness.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v2/#redactions)

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows key `membership`.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows key `creator`.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows key `join_rule`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.aliases`](https://spec.matrix.org/v1.11/client-server-api#historical-events) allows key `aliases`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v2/#event-ids)

An event has exactly one event ID. Event IDs in this room version have the format:

```
$opaque_id:domain
```

where `domain` is the [server name](https://spec.matrix.org/v1.11/appendices/#server-name) of the homeserver which created the room, and `opaque_id` is a locally-unique string.

The domain is used only for namespacing to avoid the risk of clashes of identifiers between different homeservers. There is no implication that the room or event in question is still available at the corresponding homeserver.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v2/#event-format)

Events in rooms of this version have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room versions 1 and 2.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|[[string\|[Event Hash](https://spec.matrix.org/v1.11/rooms/v2/#definition-persistent-data-unit_event-hash)]]|**Required:**<br><br>Event IDs and reference hashes for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`event_id`|`string`|**Required:** The event ID for the PDU.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v2/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|[[string\|[Event Hash](https://spec.matrix.org/v1.11/rooms/v2/#definition-persistent-data-unit_event-hash)]]|**Required:**<br><br>Event IDs and reference hashes for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v2/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$af232176:example.org",
    {
      "sha256": "abase64encodedsha256hashshouldbe43byteslong"
    }
  ],
  "content": {
    "key": "value"
  },
  "depth": 12,
  "event_id": "$a4ecee13e2accdadf56c1025:example.com",
  "hashes": {
    "sha256": "thishashcoversallfieldsincasethisisredacted"
  },
  "origin": "example.com",
  "origin_server_ts": 1404838188000,
  "prev_events": [
    "$af232176:example.org",
    {
      "sha256": "abase64encodedsha256hashshouldbe43byteslong"
    }
  ],
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v2/#deprecated-event-content-schemas)

Events sent into rooms of this version can have formats which are different from their normal schema. Those cases are documented here.

> [!warning] warning:
> The behaviour described here is preserved strictly for backwards compatibility only. A homeserver should take reasonable precautions to prevent users from sending these so-called “malformed” events, and must never rely on the behaviours described here as a default.

###### `m.room.power_levels` events accept values as strings[](https://spec.matrix.org/v1.11/rooms/v2/#mroompower_levels-events-accept-values-as-strings)

In order to maintain backwards compatibility with early implementations, each of the integer-valued properties within [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) events can be encoded as strings instead of integers. This includes the nested values within the `events`, `notifications` and `users` properties. For example, the following is a valid `m.room.power_levels` event in this room version:

```
{
  "content": {
    "ban": "50",
    "events": {
      "m.room.power_levels": "100"
    },
    "events_default": "0",
    "state_default": "50",
    "users": {
      "@example:localhost": "100"
    },
    "users_default": "0"
  },
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "state_key": "",
  "type": "m.room.power_levels"
}
```

When the value is representative of an integer, they must be the following format:

- a single base 10 integer, no float values or decimal points, optionally with any number of leading zeroes (`"100"`, `"000100"`);
- optionally prefixed with a single `-` or `+` character before the integer (`"+100"`, `"-100"`).
- optionally with any number of leading or trailing whitespace characters (`" 100 "`, `" 00100 "`, `" +100 "`, `" -100 "`);

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v2/#authorization-rules)

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

The rules are as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Considering the event’s `auth_events`:
    1. If there are duplicate entries for a given `type` and `state_key` pair, reject.
    2. If there are entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification, reject.
    3. If there are entries which were themselves rejected under the [checks performed on receipt of a PDU](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu), reject.
    4. If there is no `m.room.create` event among the entries, reject.
3. If the `content` of the `m.room.create` event in the room state has the property `m.federate` set to `false`, and the `sender` domain of the event does not match the `sender` domain of the create event, reject.
4. If type is `m.room.aliases`:
    1. If event has no `state_key`, reject.
    2. If sender’s domain doesn’t matches `state_key`, reject.
    3. Otherwise, allow.
5. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` then allow if membership state is `invite` or `join`.
        5. If the `join_rule` is `public`, allow.
        6. Otherwise, reject.
    3. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    4. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite` or `join`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    5. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    6. Otherwise, the membership is unknown. Reject.
6. If the `sender`’s current membership state is not `join`, reject.
7. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
8. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
9. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
10. If type is `m.room.power_levels`:
    1. If the `users` property in `content` is not an object with keys that are valid user IDs with values that are integers (or a string that is an integer), reject.
    2. If there is no previous `m.room.power_levels` event in the room, allow.
    3. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is greater than the `sender`’s current power level, reject.
        2. If the new value is greater than the `sender`’s current power level, reject.
    4. For each entry being changed in, or removed from, the `events` property:
        1. If the current value is greater than the `sender`’s current power level, reject.
    5. For each entry being added to, or changed in, the `events` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. Otherwise, allow.
11. If type is `m.room.redaction`:
    1. If the `sender`’s power level is greater than or equal to the _redact level_, allow.
    2. If the domain of the `event_id` of the event being redacted is the same as the domain of the `event_id` of the `m.room.redaction`, allow.
    3. Otherwise, reject.
12. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

#### Canonical JSON[](https://spec.matrix.org/v1.11/rooms/v2/#canonical-json)

Servers MUST NOT strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json) for the reasons described there.


## Room Version 3

This room version builds on [version 2](https://spec.matrix.org/v1.11/rooms/v2) with an improved event format.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v3/#client-considerations)

This room version changes the format for event IDs sent to clients. Clients should be aware that these event IDs may contain slashes and other potentially problematic characters. Clients should be treating event IDs as opaque identifiers and should not be attempting to parse them into a usable form, just like with other room versions.

Clients should expect to see event IDs changed from the format of `$randomstring:example.org` to something like `$acR1l0raoZnm60CBwAVgqbZqoO/mYU81xysh1u7XcJk` (note the lack of domain and the potentially problematic slash).

Though unchanged in this room version, clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v3/#redactions) section below for a full overview.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v3/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

Room version 3 uses the event format described here in addition to all the remaining behaviour described by [room version 2](https://spec.matrix.org/v1.11/rooms/v2).

#### Handling redactions[](https://spec.matrix.org/v1.11/rooms/v3/#handling-redactions)

In room versions 1 and 2, redactions were explicitly part of the [authorization rules](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules) under Rule 11. As of room version 3, these conditions no longer exist as represented by [this version’s authorization rules](https://spec.matrix.org/v1.11/rooms/v3/#authorization-rules).

While redactions are always accepted by the authorization rules for events, they should not be sent to clients until both the redaction event and the event the redaction affects have been received, and can be validated. If both events are valid and have been seen by the server, then the server applies the redaction if one of the following conditions is met:

1. The power level of the redaction event’s `sender` is greater than or equal to the _redact level_.
2. The domain of the redaction event’s `sender` matches that of the original event’s `sender`.

If the server would apply a redaction, the redaction event is also sent to clients. Otherwise, the server simply waits for a valid partner event to arrive where it can then re-check the above.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v3/#event-ids)

> [!info] RATIONALE:
> In other room versions (namely version 1 and 2) the event ID is a distinct field from the remainder of the event, which must be tracked as such. This leads to complications where servers receive multiple events with the same ID in either the same or different rooms where the server cannot easily keep track of which event it should be using. By removing the use of a dedicated event ID, servers are required to track the hashes on an event to determine its ID.

**[New in this version]** The event ID is the [reference hash](https://spec.matrix.org/v1.11/server-server-api#calculating-the-reference-hash-for-an-event) of the event encoded using [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64), prefixed with `$`. A resulting event ID using this approach should look similar to `$CD66HAED5npg6074c6pDtLKalHjVfYb2q4Q3LZgrW6o`.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v3/#event-format)

When events are sent over federation, the `event_id` field is no longer included. A server receiving an event should compute the relevant event ID for itself.

Additionally, the format of the `auth_events` and `prev_events` fields are changed: instead of lists of `(event_id, hash)` pairs, they are now plain lists of event IDs.

These changes to the format of an event mean that servers must be aware of the version of the room containing an incoming event, so that the event can be correctly parsed and handled. This is facilitated via changes to the server-server API (such as the inclusion of `room_version` in the response to [`GET /_matrix/federation/v1/make_join/{roomId}/{userId}`](https://spec.matrix.org/v1.11/server-server-api/#get_matrixfederationv1make_joinroomiduserid).)

The complete structure of a event in a v3 room is shown below.

##### `Persistent Data Unit`

---

A persistent data unit (event) for room version 3 and beyond.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|`[string]`|**Required:**<br><br>Event IDs for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v3/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|`[string]`|**Required:**<br><br>Event IDs for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v3/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$base64encodedeventid",
    "$adifferenteventid"
  ],
  "content": {
    "key": "value"
  },
  "depth": 12,
  "hashes": {
    "sha256": "thishashcoversallfieldsincasethisisredacted"
  },
  "origin": "example.com",
  "origin_server_ts": 1404838188000,
  "prev_events": [
    "$base64encodedeventid",
    "$adifferenteventid"
  ],
  "redacts": "$some/old+event",
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v3/#deprecated-event-content-schemas)

Events sent into rooms of this version can have formats which are different from their normal schema. Those cases are documented here.

> [!warning] warning:
> The behaviour described here is preserved strictly for backwards compatibility only. A homeserver should take reasonable precautions to prevent users from sending these so-called “malformed” events, and must never rely on the behaviours described here as a default.

###### `m.room.power_levels` events accept values as strings[](https://spec.matrix.org/v1.11/rooms/v3/#mroompower_levels-events-accept-values-as-strings)

In order to maintain backwards compatibility with early implementations, each of the integer-valued properties within [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) events can be encoded as strings instead of integers. This includes the nested values within the `events`, `notifications` and `users` properties. For example, the following is a valid `m.room.power_levels` event in this room version:

```
{
  "content": {
    "ban": "50",
    "events": {
      "m.room.power_levels": "100"
    },
    "events_default": "0",
    "state_default": "50",
    "users": {
      "@example:localhost": "100"
    },
    "users_default": "0"
  },
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "state_key": "",
  "type": "m.room.power_levels"
}
```

When the value is representative of an integer, they must be the following format:

- a single base 10 integer, no float values or decimal points, optionally with any number of leading zeroes (`"100"`, `"000100"`);
- optionally prefixed with a single `-` or `+` character before the integer (`"+100"`, `"-100"`).
- optionally with any number of leading or trailing whitespace characters (`" 100 "`, `" 00100 "`, `" +100 "`, `" -100 "`);

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v3/#authorization-rules)

> [!info] info:
> **[New in this version]** `m.room.redaction` events are subject to auth rules in the same way as any other event. In practice, that means they will normally be allowed by the auth rules, unless the `m.room.power_levels` event sets a power level requirement for `m.room.redaction`events via the `events` or `events_default` properties. In particular, the _redact level_ is **not** considered by the auth rules.
> 
> The ability to send a redaction event does not mean that the redaction itself should be performed. Receiving servers must perform additional checks, as described in the [Handling Redactions](https://spec.matrix.org/v1.11/rooms/v3/#handling-redactions) section.

**[New in this version]** In room versions 1 and 2, events need a signature from the domain of the `event_id` in order to be considered valid. This room version does not include an `event_id` over federation in the same respect, so does not need a signature from that server. The event must still be signed by the server denoted by the `sender` property, however.

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

The complete list of rules, as of room version 3, is as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Considering the event’s `auth_events`:
    1. If there are duplicate entries for a given `type` and `state_key` pair, reject.
    2. If there are entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification, reject.
    3. If there are entries which were themselves rejected under the [checks performed on receipt of a PDU](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu), reject.
    4. If there is no `m.room.create` event among the entries, reject.
3. If the `content` of the `m.room.create` event in the room state has the property `m.federate` set to `false`, and the `sender` domain of the event does not match the `sender` domain of the create event, reject.
4. If type is `m.room.aliases`:
    1. If event has no `state_key`, reject.
    2. If sender’s domain doesn’t matches `state_key`, reject.
    3. Otherwise, allow.
5. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` then allow if membership state is `invite` or `join`.
        5. If the `join_rule` is `public`, allow.
        6. Otherwise, reject.
    3. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    4. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite` or `join`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    5. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    6. Otherwise, the membership is unknown. Reject.
6. If the `sender`’s current membership state is not `join`, reject.
7. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
8. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
9. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
10. If type is `m.room.power_levels`:
    1. If `users` property in `content` is not an object with keys that are valid user IDs with values that are integers (or a string that is an integer), reject.
    2. If there is no previous `m.room.power_levels` event in the room, allow.
    3. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is greater than the `sender`’s current power level, reject.
        2. If the new value is greater than the `sender`’s current power level, reject.
    4. For each entry being changed in, or removed from, the `events` property:
        1. If the current value is greater than the `sender`’s current power level, reject.
    5. For each entry being added to, or changed in, the `events` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. Otherwise, allow.
11. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

### Unchanged from v2[](https://spec.matrix.org/v1.11/rooms/v3/#unchanged-from-v2)

The following sections have not been modified since v2, but are included for completeness.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v3/#redactions)

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows key `membership`.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows key `creator`.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows key `join_rule`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.aliases`](https://spec.matrix.org/v1.11/client-server-api#historical-events) allows key `aliases`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v3/#state-resolution)

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v3/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v3/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v3/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

#### Canonical JSON[](https://spec.matrix.org/v1.11/rooms/v3/#canonical-json)

Servers MUST NOT strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json) for the reasons described there.

## Room Version 4

This room version builds on [version 3](https://spec.matrix.org/v1.11/rooms/v3) using a different encoding for event IDs.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v4/#client-considerations)

This room version changes the format form event IDs sent to clients. Clients should already be treating event IDs as opaque identifiers, and should not be concerned with the format of them. Clients should still encode the event ID when including it in a request path.

Clients should expect to see event IDs changed from the format of `$randomstring:example.org` to something like `$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg` (note the lack of domain).

Though unchanged in this room version, clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v4/#redactions) section below for a full overview.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v4/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

Room version 4 uses the same algorithms defined in [room version 3](https://spec.matrix.org/v1.11/rooms/v3), however using URL-safe base64 to generate the event ID.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v4/#event-ids)

> [!info] RATIONALE:
> Room version 3 generated event IDs that were difficult for client implementations which were not encoding the event ID to function in those rooms. It additionally raised concern due to the `/` character being interpreted differently by some reverse proxy software, and generally made administration harder.

The event ID is the [reference hash](https://spec.matrix.org/v1.11/server-server-api#calculating-the-reference-hash-for-an-event) of the event encoded using a variation of [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64) which replaces the 62nd and 63rd characters with `-` and `_` instead of using `+` and `/`. This matches [RFC4648’s definition of URL-safe base64](https://tools.ietf.org/html/rfc4648#section-5).

Event IDs are still prefixed with `$` and might result in looking like `$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg`.

### Unchanged from v3[](https://spec.matrix.org/v1.11/rooms/v4/#unchanged-from-v3)

The following sections have not been modified since v3, but are included for completeness.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v4/#redactions)

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows key `membership`.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows key `creator`.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows key `join_rule`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.aliases`](https://spec.matrix.org/v1.11/client-server-api#historical-events) allows key `aliases`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.

#### Handling redactions[](https://spec.matrix.org/v1.11/rooms/v4/#handling-redactions)

In room versions 1 and 2, redactions were explicitly part of the [authorization rules](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules) under Rule 11. As of room version 3, these conditions no longer exist as represented by [this version’s authorization rules](https://spec.matrix.org/v1.11/rooms/v4/#authorization-rules).

While redactions are always accepted by the authorization rules for events, they should not be sent to clients until both the redaction event and the event the redaction affects have been received, and can be validated. If both events are valid and have been seen by the server, then the server applies the redaction if one of the following conditions is met:

1. The power level of the redaction event’s `sender` is greater than or equal to the _redact level_.
2. The domain of the redaction event’s `sender` matches that of the original event’s `sender`.

If the server would apply a redaction, the redaction event is also sent to clients. Otherwise, the server simply waits for a valid partner event to arrive where it can then re-check the above.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v4/#event-format)

The event format is the same as [room version 3](https://spec.matrix.org/v1.11/rooms/v3#event-format), however the event IDs in the following example are updated to reflect the changes in this room version.

Events in rooms of this version have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room version 4 and beyond.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|`[string]`|**Required:**<br><br>Event IDs for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v4/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|`[string]`|**Required:**<br><br>Event IDs for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v4/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$urlsafe_base64_encoded_eventid",
    "$a-different-event-id"
  ],
  "content": {
    "key": "value"
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
  "redacts": "$some-old_event",
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v4/#deprecated-event-content-schemas)

Events sent into rooms of this version can have formats which are different from their normal schema. Those cases are documented here.

> [!warning] warning:
> The behaviour described here is preserved strictly for backwards compatibility only. A homeserver should take reasonable precautions to prevent users from sending these so-called “malformed” events, and must never rely on the behaviours described here as a default.

###### `m.room.power_levels` events accept values as strings[](https://spec.matrix.org/v1.11/rooms/v4/#mroompower_levels-events-accept-values-as-strings)

In order to maintain backwards compatibility with early implementations, each of the integer-valued properties within [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) events can be encoded as strings instead of integers. This includes the nested values within the `events`, `notifications` and `users` properties. For example, the following is a valid `m.room.power_levels` event in this room version:

```
{
  "content": {
    "ban": "50",
    "events": {
      "m.room.power_levels": "100"
    },
    "events_default": "0",
    "state_default": "50",
    "users": {
      "@example:localhost": "100"
    },
    "users_default": "0"
  },
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "state_key": "",
  "type": "m.room.power_levels"
}
```

When the value is representative of an integer, they must be the following format:

- a single base 10 integer, no float values or decimal points, optionally with any number of leading zeroes (`"100"`, `"000100"`);
- optionally prefixed with a single `-` or `+` character before the integer (`"+100"`, `"-100"`).
- optionally with any number of leading or trailing whitespace characters (`" 100 "`, `" 00100 "`, `" +100 "`, `" -100 "`);

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v4/#authorization-rules)

In room versions 1 and 2, events need a signature from the domain of the `event_id` in order to be considered valid. This room version does not include an `event_id` over federation in the same respect, so does not need a signature from that server. The event must still be signed by the server denoted by the `sender` property, however.

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

The complete list of rules, as of room version 3, is as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Considering the event’s `auth_events`:
    1. If there are duplicate entries for a given `type` and `state_key` pair, reject.
    2. If there are entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification, reject.
    3. If there are entries which were themselves rejected under the [checks performed on receipt of a PDU](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu), reject.
    4. If there is no `m.room.create` event among the entries, reject.
3. If the `content` of the `m.room.create` event in the room state has the property `m.federate` set to `false`, and the `sender` domain of the event does not match the `sender` domain of the create event, reject.
4. If type is `m.room.aliases`:
    1. If event has no `state_key`, reject.
    2. If sender’s domain doesn’t matches `state_key`, reject.
    3. Otherwise, allow.
5. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` then allow if membership state is `invite` or `join`.
        5. If the `join_rule` is `public`, allow.
        6. Otherwise, reject.
    3. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    4. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite` or `join`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    5. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    6. Otherwise, the membership is unknown. Reject.
6. If the `sender`’s current membership state is not `join`, reject.
7. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
8. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
9. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
10. If type is `m.room.power_levels`:
    1. If `users` property in `content` is not an object with keys that are valid user IDs with values that are integers (or a string that is an integer), reject.
    2. If there is no previous `m.room.power_levels` event in the room, allow.
    3. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is greater than the `sender`’s current power level, reject.
        2. If the new value is greater than the `sender`’s current power level, reject.
    4. For each entry being changed in, or removed from, the `events` property:
        1. If the current value is greater than the `sender`’s current power level, reject.
    5. For each entry being added to, or changed in, the `events` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. Otherwise, allow.
11. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v4/#state-resolution)

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v4/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v4/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v4/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

#### Canonical JSON[](https://spec.matrix.org/v1.11/rooms/v4/#canonical-json)

Servers MUST NOT strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json) for the reasons described there.

## Room Version 5

This room version builds on [version 4](https://spec.matrix.org/v1.11/rooms/v4) while enforcing signing key validity periods for events.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v5/#client-considerations)

There are no client considerations introduced in this room version. Clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v5/#redactions) section below for a full overview of the algorithm.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v5/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

Room version 5 uses the same algorithms defined in [room version 4](https://spec.matrix.org/v1.11/rooms/v4), ensuring that signing key validity is respected.

#### Signing key validity period[](https://spec.matrix.org/v1.11/rooms/v5/#signing-key-validity-period)

When validating event signatures, servers MUST enforce the `valid_until_ts` property from a key request is at least as large as the `origin_server_ts` for the event being validated. Servers missing a copy of the signing key MUST try to obtain one via the [GET /_matrix/key/v2/server](https://spec.matrix.org/v1.11/server-server-api#get_matrixkeyv2server) or [POST /_matrix/key/v2/query](https://spec.matrix.org/v1.11/server-server-api#post_matrixkeyv2query) APIs. When using the `/query` endpoint, servers MUST set the `minimum_valid_until_ts` property to prompt the notary server to attempt to refresh the key if appropriate.

Servers MUST use the lesser of `valid_until_ts` and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.

### Unchanged from v4[](https://spec.matrix.org/v1.11/rooms/v5/#unchanged-from-v4)

The following sections have not been modified since v4, but are included for completeness.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v5/#redactions)

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows key `membership`.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows key `creator`.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows key `join_rule`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.aliases`](https://spec.matrix.org/v1.11/client-server-api#historical-events) allows key `aliases`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.

#### Handling redactions[](https://spec.matrix.org/v1.11/rooms/v5/#handling-redactions)

In room versions 1 and 2, redactions were explicitly part of the [authorization rules](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules) under Rule 11. As of room version 3, these conditions no longer exist as represented by [this version’s authorization rules](https://spec.matrix.org/v1.11/rooms/v5/#authorization-rules).

While redactions are always accepted by the authorization rules for events, they should not be sent to clients until both the redaction event and the event the redaction affects have been received, and can be validated. If both events are valid and have been seen by the server, then the server applies the redaction if one of the following conditions is met:

1. The power level of the redaction event’s `sender` is greater than or equal to the _redact level_.
2. The domain of the redaction event’s `sender` matches that of the original event’s `sender`.

If the server would apply a redaction, the redaction event is also sent to clients. Otherwise, the server simply waits for a valid partner event to arrive where it can then re-check the above.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v5/#event-ids)

The event ID is the [reference hash](https://spec.matrix.org/v1.11/server-server-api#calculating-the-reference-hash-for-an-event) of the event encoded using a variation of [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64) which replaces the 62nd and 63rd characters with `-` and `_` instead of using `+` and `/`. This matches [RFC4648’s definition of URL-safe base64](https://tools.ietf.org/html/rfc4648#section-5).

Event IDs are still prefixed with `$` and might result in looking like `$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg`.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v5/#event-format)

Events in rooms of this version have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room version 4 and beyond.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|`[string]`|**Required:**<br><br>Event IDs for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v5/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|`[string]`|**Required:**<br><br>Event IDs for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v5/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$urlsafe_base64_encoded_eventid",
    "$a-different-event-id"
  ],
  "content": {
    "key": "value"
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
  "redacts": "$some-old_event",
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v5/#deprecated-event-content-schemas)

Events sent into rooms of this version can have formats which are different from their normal schema. Those cases are documented here.

> [!warning] warning:
> The behaviour described here is preserved strictly for backwards compatibility only. A homeserver should take reasonable precautions to prevent users from sending these so-called “malformed” events, and must never rely on the behaviours described here as a default.

###### `m.room.power_levels` events accept values as strings[](https://spec.matrix.org/v1.11/rooms/v5/#mroompower_levels-events-accept-values-as-strings)

In order to maintain backwards compatibility with early implementations, each of the integer-valued properties within [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) events can be encoded as strings instead of integers. This includes the nested values within the `events`, `notifications` and `users` properties. For example, the following is a valid `m.room.power_levels` event in this room version:

```
{
  "content": {
    "ban": "50",
    "events": {
      "m.room.power_levels": "100"
    },
    "events_default": "0",
    "state_default": "50",
    "users": {
      "@example:localhost": "100"
    },
    "users_default": "0"
  },
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "state_key": "",
  "type": "m.room.power_levels"
}
```

When the value is representative of an integer, they must be the following format:

- a single base 10 integer, no float values or decimal points, optionally with any number of leading zeroes (`"100"`, `"000100"`);
- optionally prefixed with a single `-` or `+` character before the integer (`"+100"`, `"-100"`).
- optionally with any number of leading or trailing whitespace characters (`" 100 "`, `" 00100 "`, `" +100 "`, `" -100 "`);

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v5/#authorization-rules)

In room versions 1 and 2, events need a signature from the domain of the `event_id` in order to be considered valid. This room version does not include an `event_id` over federation in the same respect, so does not need a signature from that server. The event must still be signed by the server denoted by the `sender` property, however.

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

The complete list of rules, as of room version 3, is as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Considering the event’s `auth_events`:
    1. If there are duplicate entries for a given `type` and `state_key` pair, reject.
    2. If there are entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification, reject.
    3. If there are entries which were themselves rejected under the [checks performed on receipt of a PDU](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu), reject.
    4. If there is no `m.room.create` event among the entries, reject.
3. If the `content` of the `m.room.create` event in the room state has the property `m.federate` set to `false`, and the `sender` domain of the event does not match the `sender` domain of the create event, reject.
4. If type is `m.room.aliases`:
    1. If event has no `state_key`, reject.
    2. If sender’s domain doesn’t matches `state_key`, reject.
    3. Otherwise, allow.
5. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` then allow if membership state is `invite` or `join`.
        5. If the `join_rule` is `public`, allow.
        6. Otherwise, reject.
    3. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    4. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite` or `join`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    5. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    6. Otherwise, the membership is unknown. Reject.
6. If the `sender`’s current membership state is not `join`, reject.
7. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
8. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
9. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
10. If type is `m.room.power_levels`:
    1. If `users` property in `content` is not an object with keys that are valid user IDs with values that are integers (or a string that is an integer), reject.
    2. If there is no previous `m.room.power_levels` event in the room, allow.
    3. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is greater than the `sender`’s current power level, reject.
        2. If the new value is greater than the `sender`’s current power level, reject.
    4. For each entry being changed in, or removed from, the `events` property:
        1. If the current value is greater than the `sender`’s current power level, reject.
    5. For each entry being added to, or changed in, the `events` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. Otherwise, allow.
11. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v5/#state-resolution)

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v5/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v5/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v5/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

#### Canonical JSON [](https://spec.matrix.org/v1.11/rooms/v5/#canonical-json)

Servers MUST NOT strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json) for the reasons described there.

## Room Version 6

This room version builds on [version 5](https://spec.matrix.org/v1.11/rooms/v5) while changing various authorization rules performed on events.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v6/#client-considerations)

There are no client considerations introduced in this room version. Clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v6/#redactions) section below for a full overview of the algorithm.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v6/#redactions)

**[New in this version]** All significant meaning for `m.room.aliases` has been removed from the redaction algorithm. The remaining rules are the same as past room versions.

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows key `membership`.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows key `creator`.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows key `join_rule`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v6/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

Room version 6 makes the following alterations to algorithms described in [room version 5](https://spec.matrix.org/v1.11/rooms/v5).

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v6/#redactions-1)

[See above](https://spec.matrix.org/v1.11/rooms/v6/#redactions).

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v6/#authorization-rules)

**[New in this version]** Rule 4, which related specifically to events of type `m.room.aliases`, is removed. `m.room.aliases` events must still pass authorization checks relating to state events.

**[New in this version]** Additionally, the authorization rules for events of type `m.room.power_levels` now include a `notifications` property under `content`. This updates rules 10.4 and 10.5 (now 9.4 and 9.5), which checked the `events` property.

Events must be signed by the server denoted by the `sender` property.

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

> [!info] info:
> `m.room.redaction` events are subject to auth rules in the same way as any other event. In practice, that means they will normally be allowed by the auth rules, unless the `m.room.power_levels` event sets a power level requirement for `m.room.redaction` events via the `events` or `events_default` properties. In particular, the _redact level_ is **not** considered by the auth rules.
> 
> The ability to send a redaction event does not mean that the redaction itself should be performed. Receiving servers must perform additional checks, as described in the [Handling Redactions](https://spec.matrix.org/v1.11/rooms/v6/#handling-redactions) section.

The rules are as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Reject if event has `auth_events` that:
    1. have duplicate entries for a given `type` and `state_key` pair
    2. have entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification.
3. If event does not have a `m.room.create` in its `auth_events`, reject.
4. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` then allow if membership state is `invite` or `join`.
        5. If the `join_rule` is `public`, allow.
        6. Otherwise, reject.
    3. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    4. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite` or `join`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    5. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    6. Otherwise, the membership is unknown. Reject.
5. If the `sender`’s current membership state is not `join`, reject.
6. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
7. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
8. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
9. If type is `m.room.power_levels`:
    1. If the `users` property in `content` is not an object with keys that are valid user IDs with values that are integers (or a string that is an integer), reject.
    2. If there is no previous `m.room.power_levels` event in the room, allow.
    3. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is higher than the `sender`’s current power level, reject.
        2. If the new value is higher than the `sender`’s current power level, reject.
    4. **[Changed in this version]** For each entry being changed in, or removed from, the `events` or `notifications` properties:
        1. If the current value is greater than the `sender`’s current power level, reject.
    5. **[Changed in this version]** For each entry being added to, or changed in, the `events` or `notifications` properties:
        1. If the new value is greater than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. Otherwise, allow.
10. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

#### Canonical JSON[](https://spec.matrix.org/v1.11/rooms/v6/#canonical-json)

Servers MUST strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json). This translates to a 400 `M_BAD_JSON` error on most endpoints, or discarding of events over federation. For example, the Federation API’s `/send` endpoint would discard the event whereas the Client Server API’s `/send/{eventType}` endpoint would return a `M_BAD_JSON` error.

### Unchanged from v5[](https://spec.matrix.org/v1.11/rooms/v6/#unchanged-from-v5)

The following sections have not been modified since v5, but are included for completeness.

#### Handling redactions[](https://spec.matrix.org/v1.11/rooms/v6/#handling-redactions)

In room versions 1 and 2, redactions were explicitly part of the [authorization rules](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules) under Rule 11. As of room version 3, these conditions no longer exist as represented by [this version’s authorization rules](https://spec.matrix.org/v1.11/rooms/v6/#authorization-rules).

While redactions are always accepted by the authorization rules for events, they should not be sent to clients until both the redaction event and the event the redaction affects have been received, and can be validated. If both events are valid and have been seen by the server, then the server applies the redaction if one of the following conditions is met:

1. The power level of the redaction event’s `sender` is greater than or equal to the _redact level_.
2. The domain of the redaction event’s `sender` matches that of the original event’s `sender`.

If the server would apply a redaction, the redaction event is also sent to clients. Otherwise, the server simply waits for a valid partner event to arrive where it can then re-check the above.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v6/#event-ids)

The event ID is the [reference hash](https://spec.matrix.org/v1.11/server-server-api#calculating-the-reference-hash-for-an-event) of the event encoded using a variation of [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64) which replaces the 62nd and 63rd characters with `-` and `_` instead of using `+` and `/`. This matches [RFC4648’s definition of URL-safe base64](https://tools.ietf.org/html/rfc4648#section-5).

Event IDs are still prefixed with `$` and might result in looking like `$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg`.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v6/#event-format)

Events in rooms of this version have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room version 4 and beyond.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|`[string]`|**Required:**<br><br>Event IDs for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v6/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|`[string]`|**Required:**<br><br>Event IDs for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v6/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$urlsafe_base64_encoded_eventid",
    "$a-different-event-id"
  ],
  "content": {
    "key": "value"
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
  "redacts": "$some-old_event",
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v6/#deprecated-event-content-schemas)

Events sent into rooms of this version can have formats which are different from their normal schema. Those cases are documented here.

> [!warning] warning:
> The behaviour described here is preserved strictly for backwards compatibility only. A homeserver should take reasonable precautions to prevent users from sending these so-called “malformed” events, and must never rely on the behaviours described here as a default.

###### `m.room.power_levels` events accept values as strings[](https://spec.matrix.org/v1.11/rooms/v6/#mroompower_levels-events-accept-values-as-strings)

In order to maintain backwards compatibility with early implementations, each of the integer-valued properties within [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) events can be encoded as strings instead of integers. This includes the nested values within the `events`, `notifications` and `users` properties. For example, the following is a valid `m.room.power_levels` event in this room version:

```
{
  "content": {
    "ban": "50",
    "events": {
      "m.room.power_levels": "100"
    },
    "events_default": "0",
    "state_default": "50",
    "users": {
      "@example:localhost": "100"
    },
    "users_default": "0"
  },
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "state_key": "",
  "type": "m.room.power_levels"
}
```

When the value is representative of an integer, they must be the following format:

- a single base 10 integer, no float values or decimal points, optionally with any number of leading zeroes (`"100"`, `"000100"`);
- optionally prefixed with a single `-` or `+` character before the integer (`"+100"`, `"-100"`).
- optionally with any number of leading or trailing whitespace characters (`" 100 "`, `" 00100 "`, `" +100 "`, `" -100 "`);

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v6/#state-resolution)

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v6/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v6/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v6/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

#### Signing key validity period[](https://spec.matrix.org/v1.11/rooms/v6/#signing-key-validity-period)

When validating event signatures, servers MUST enforce the `valid_until_ts` property from a key request is at least as large as the `origin_server_ts` for the event being validated. Servers missing a copy of the signing key MUST try to obtain one via the [GET /_matrix/key/v2/server](https://spec.matrix.org/v1.11/server-server-api#get_matrixkeyv2server) or [POST /_matrix/key/v2/query](https://spec.matrix.org/v1.11/server-server-api#post_matrixkeyv2query) APIs. When using the `/query` endpoint, servers MUST set the `minimum_valid_until_ts` property to prompt the notary server to attempt to refresh the key if appropriate.

Servers MUST use the lesser of `valid_until_ts` and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.

## Room Version 7

This room version builds on [version 6](https://spec.matrix.org/v1.11/rooms/v6) to introduce knocking as a possible join rule and membership state.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v7/#client-considerations)

This is the first room version to support knocking completely. As such, users will not be able to knock on rooms which are not based off v7.

Though unchanged in this room version, clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v7/#redactions) section below for a full overview.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v7/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

Room version 7 adds new authorization rules for events to support knocking. [Room version 6](https://spec.matrix.org/v1.11/rooms/v6) has details of other authorization rule changes, as do the versions v6 is based upon.

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v7/#authorization-rules)

**[New in this version]** For checks performed upon `m.room.member` events, a new point for `membership=knock` is added.

Events must be signed by the server denoted by the `sender` property.

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

> [!info] info:
> `m.room.redaction` events are subject to auth rules in the same way as any other event. In practice, that means they will normally be allowed by the auth rules, unless the `m.room.power_levels` event sets a power level requirement for `m.room.redaction` events via the `events` or `events_default` properties. In particular, the _redact level_ is **not** considered by the auth rules.
> 
> The ability to send a redaction event does not mean that the redaction itself should be performed. Receiving servers must perform additional checks, as described in the [Handling redactions](https://spec.matrix.org/v1.11/rooms/v7/#handling-redactions) section.

The rules are as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Reject if event has `auth_events` that:
    1. have duplicate entries for a given `type` and `state_key` pair
    2. have entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification.
3. If event does not have a `m.room.create` in its `auth_events`, reject.
4. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. **[Changed in this version]** If the `join_rule` is `invite` or `knock` then allow if membership state is `invite` or `join`.
        5. If the `join_rule` is `public`, allow.
        6. Otherwise, reject.
    3. If `membership` is `invite`:
        1. If `content` has `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    4. If `membership` is `leave`:
        1. **[Changed in this version]** If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite`, `join`, or `knock`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    5. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    6. **[New in this version]** If `membership` is `knock`:
        1. If the `join_rule` is anything other than `knock`, reject.
        2. If `sender` does not match `state_key`, reject.
        3. If the `sender`’s current membership is not `ban`, `invite`, or `join`, allow.
        4. Otherwise, reject.
    7. Otherwise, the membership is unknown. Reject.
5. If the `sender`’s current membership state is not `join`, reject.
6. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
7. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
8. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
9. If type is `m.room.power_levels`:
    1. If the `users` property in `content` is not an object with keys that are valid user IDs with values that are integers (or a string that is an integer), reject.
    2. If there is no previous `m.room.power_levels` event in the room, allow.
    3. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is higher than the `sender`’s current power level, reject.
        2. If the new value is higher than the `sender`’s current power level, reject.
    4. For each entry being changed in, or removed from, the `events` or `notifications` properties:
        1. If the current value is greater than the `sender`’s current power level, reject.
    5. For each entry being added to, or changed in, the `events` or `notifications` properties:
        1. If the new value is greater than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. Otherwise, allow..
10. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

### Unchanged from v6[](https://spec.matrix.org/v1.11/rooms/v7/#unchanged-from-v6)

The following sections have not been modified since v6, but are included for completeness.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v7/#redactions)

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows key `membership`.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows key `creator`.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows key `join_rule`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.

#### Handling redactions[](https://spec.matrix.org/v1.11/rooms/v7/#handling-redactions)

In room versions 1 and 2, redactions were explicitly part of the [authorization rules](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules) under Rule 11. As of room version 3, these conditions no longer exist as represented by [this version’s authorization rules](https://spec.matrix.org/v1.11/rooms/v7/#authorization-rules).

While redactions are always accepted by the authorization rules for events, they should not be sent to clients until both the redaction event and the event the redaction affects have been received, and can be validated. If both events are valid and have been seen by the server, then the server applies the redaction if one of the following conditions is met:

1. The power level of the redaction event’s `sender` is greater than or equal to the _redact level_.
2. The domain of the redaction event’s `sender` matches that of the original event’s `sender`.

If the server would apply a redaction, the redaction event is also sent to clients. Otherwise, the server simply waits for a valid partner event to arrive where it can then re-check the above.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v7/#event-ids)

The event ID is the [reference hash](https://spec.matrix.org/v1.11/server-server-api#calculating-the-reference-hash-for-an-event) of the event encoded using a variation of [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64) which replaces the 62nd and 63rd characters with `-` and `_` instead of using `+` and `/`. This matches [RFC4648’s definition of URL-safe base64](https://tools.ietf.org/html/rfc4648#section-5).

Event IDs are still prefixed with `$` and might result in looking like `$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg`.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v7/#event-format)

Events in rooms of this version have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room version 4 and beyond.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|`[string]`|**Required:**<br><br>Event IDs for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v7/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|`[string]`|**Required:**<br><br>Event IDs for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v7/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$urlsafe_base64_encoded_eventid",
    "$a-different-event-id"
  ],
  "content": {
    "key": "value"
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
  "redacts": "$some-old_event",
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v7/#deprecated-event-content-schemas)

Events sent into rooms of this version can have formats which are different from their normal schema. Those cases are documented here.

> [!warning] warning:
> The behaviour described here is preserved strictly for backwards compatibility only. A homeserver should take reasonable precautions to prevent users from sending these so-called “malformed” events, and must never rely on the behaviours described here as a default.

###### `m.room.power_levels` events accept values as strings[](https://spec.matrix.org/v1.11/rooms/v7/#mroompower_levels-events-accept-values-as-strings)

In order to maintain backwards compatibility with early implementations, each of the integer-valued properties within [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) events can be encoded as strings instead of integers. This includes the nested values within the `events`, `notifications` and `users` properties. For example, the following is a valid `m.room.power_levels` event in this room version:

```
{
  "content": {
    "ban": "50",
    "events": {
      "m.room.power_levels": "100"
    },
    "events_default": "0",
    "state_default": "50",
    "users": {
      "@example:localhost": "100"
    },
    "users_default": "0"
  },
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "state_key": "",
  "type": "m.room.power_levels"
}
```

When the value is representative of an integer, they must be the following format:

- a single base 10 integer, no float values or decimal points, optionally with any number of leading zeroes (`"100"`, `"000100"`);
- optionally prefixed with a single `-` or `+` character before the integer (`"+100"`, `"-100"`).
- optionally with any number of leading or trailing whitespace characters (`" 100 "`, `" 00100 "`, `" +100 "`, `" -100 "`);

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v7/#state-resolution)

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v7/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v7/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v7/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

#### Canonical JSON[](https://spec.matrix.org/v1.11/rooms/v7/#canonical-json)

Servers MUST strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json). This translates to a 400 `M_BAD_JSON` error on most endpoints, or discarding of events over federation. For example, the Federation API’s `/send` endpoint would discard the event whereas the Client Server API’s `/send/{eventType}` endpoint would return a `M_BAD_JSON` error.

#### Signing key validity period[](https://spec.matrix.org/v1.11/rooms/v7/#signing-key-validity-period)

When validating event signatures, servers MUST enforce the `valid_until_ts` property from a key request is at least as large as the `origin_server_ts` for the event being validated. Servers missing a copy of the signing key MUST try to obtain one via the [GET /_matrix/key/v2/server](https://spec.matrix.org/v1.11/server-server-api#get_matrixkeyv2server) or [POST /_matrix/key/v2/query](https://spec.matrix.org/v1.11/server-server-api#post_matrixkeyv2query) APIs. When using the `/query` endpoint, servers MUST set the `minimum_valid_until_ts` property to prompt the notary server to attempt to refresh the key if appropriate.

Servers MUST use the lesser of `valid_until_ts` and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.

## Room Version 8

This room version builds on [version 7](https://spec.matrix.org/v1.11/rooms/v7) to introduce a new join rule that allows members to join the room based on membership in another room.

> [!warning] warning:
> This room version is known to have issues relating to redactions of member join events. [Room version 9](https://spec.matrix.org/v1.11/rooms/v9) should be preferred over v8 when creating rooms.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v8/#client-considerations)

Clients are encouraged to expose the option for the join rule in their user interface for supported room versions.

The new join rule, `restricted`, is described in the [Client-Server API](https://spec.matrix.org/v1.11/client-server-api/#restricted-rooms).

Clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v8/#redactions) section below for a full overview.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v8/#redactions)

**[New in this version]** `m.room.join_rules` events now keep `allow` in addition to other keys in `content` when being redacted.

> [!warning] warning:
> [Room version 9](https://spec.matrix.org/v1.11/rooms/v9) adds additional cases of protected properties for behaviour related to restricted rooms (the functionality introduced in v8). v9 is preferred over v8 when creating new rooms.

The full redaction algorithm follows.

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of one of the following event types:

- `m.room.member` allows key `membership`.
- `m.room.create` allows key `creator`.
- `m.room.join_rules` allows keys `join_rule`, `allow`.
- `m.room.power_levels` allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- `m.room.history_visibility` allows key `history_visibility`.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v8/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

Room version 8 adds a new join rule to allow members of a room to join another room without invite. Otherwise, the room version inherits all properties of [Room version 7](https://spec.matrix.org/v1.11/rooms/v7).

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v8/#authorization-rules)

**[New in this version]** For checks performed upon `m.room.member` events, new points for handling `content.join_authorised_via_users_server` are added (Rule 4.2 and 4.3.5).

Events must be signed by the server denoted by the `sender` property.

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

> [!info] info:
> `m.room.redaction` events are subject to auth rules in the same way as any other event. In practice, that means they will normally be allowed by the auth rules, unless the `m.room.power_levels` event sets a power level requirement for `m.room.redaction` events via the `events` or `events_default` properties. In particular, the _redact level_ is **not** considered by the auth rules.
> 
> The ability to send a redaction event does not mean that the redaction itself should be performed. Receiving servers must perform additional checks, as described in the [Handling Redactions](https://spec.matrix.org/v1.11/rooms/v8/#handling-redactions) section.

The rules are as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Considering the event’s `auth_events`:
    1. If there are duplicate entries for a given `type` and `state_key` pair, reject.
    2. If there are entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification, reject.
    3. If there are entries which were themselves rejected under the [checks performed on receipt of a PDU](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu), reject.
    4. If there is no `m.room.create` event among the entries, reject.
3. If the `content` of the `m.room.create` event in the room state has the property `m.federate` set to `false`, and the `sender` domain of the event does not match the `sender` domain of the create event, reject.
4. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. **[New in this version]** If `content` has a `join_authorised_via_users_server` property:
        1. If the event is not validly signed by the homeserver of the user ID denoted by the key, reject.
    3. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` or `knock` then allow if membership state is `invite` or `join`.
        5. **[New in this version]** If the `join_rule` is `restricted`:
            1. If membership state is `join` or `invite`, allow.
            2. If the `join_authorised_via_users_server` key in `content` is not a user with sufficient permission to invite other users, reject.
            3. Otherwise, allow.
        6. If the `join_rule` is `public`, allow.
        7. Otherwise, reject.
    4. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    5. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite`, `join`, or `knock`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    6. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    7. If `membership` is `knock`:
        1. If the `join_rule` is anything other than `knock`, reject.
        2. If `sender` does not match `state_key`, reject.
        3. If the `sender`’s current membership is not `ban`, `invite`, or `join`, allow.
        4. Otherwise, reject.
    8. Otherwise, the membership is unknown. Reject.
5. If the `sender`’s current membership state is not `join`, reject.
6. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
7. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
8. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
9. If type is `m.room.power_levels`:
    1. If the `users` property in `content` is not an object with keys that are valid user IDs with values that are integers (or a string that is an integer), reject.
    2. If there is no previous `m.room.power_levels` event in the room, allow.
    3. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is higher than the `sender`’s current power level, reject.
        2. If the new value is higher than the `sender`’s current power level, reject.
    4. For each entry being changed in, or removed from, the `events` or `notifications` properties:
        1. If the current value is greater than the `sender`’s current power level, reject.
    5. For each entry being added to, or changed in the `events` or `notifications` properties:
        1. If the new value is greater than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. Otherwise, allow.
10. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v8/#redactions-1)

[See above](https://spec.matrix.org/v1.11/rooms/v8/#redactions).

### Unchanged from v7[](https://spec.matrix.org/v1.11/rooms/v8/#unchanged-from-v7)

The following sections have not been modified since v7, but are included for completeness.

#### Handling redactions[](https://spec.matrix.org/v1.11/rooms/v8/#handling-redactions)

In room versions 1 and 2, redactions were explicitly part of the [authorization rules](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules) under Rule 11. As of room version 3, these conditions no longer exist as represented by [this version’s authorization rules](https://spec.matrix.org/v1.11/rooms/v8/#authorization-rules).

While redactions are always accepted by the authorization rules for events, they should not be sent to clients until both the redaction event and the event the redaction affects have been received, and can be validated. If both events are valid and have been seen by the server, then the server applies the redaction if one of the following conditions is met:

1. The power level of the redaction event’s `sender` is greater than or equal to the _redact level_.
2. The domain of the redaction event’s `sender` matches that of the original event’s `sender`.

If the server would apply a redaction, the redaction event is also sent to clients. Otherwise, the server simply waits for a valid partner event to arrive where it can then re-check the above.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v8/#event-ids)

The event ID is the [reference hash](https://spec.matrix.org/v1.11/server-server-api#calculating-the-reference-hash-for-an-event) of the event encoded using a variation of [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64) which replaces the 62nd and 63rd characters with `-` and `_` instead of using `+` and `/`. This matches [RFC4648’s definition of URL-safe base64](https://tools.ietf.org/html/rfc4648#section-5).

Event IDs are still prefixed with `$` and might result in looking like `$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg`.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v8/#event-format)

Events in rooms of this version have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room version 4 and beyond.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|`[string]`|**Required:**<br><br>Event IDs for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v8/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|`[string]`|**Required:**<br><br>Event IDs for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v8/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$urlsafe_base64_encoded_eventid",
    "$a-different-event-id"
  ],
  "content": {
    "key": "value"
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
  "redacts": "$some-old_event",
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v8/#deprecated-event-content-schemas)

Events sent into rooms of this version can have formats which are different from their normal schema. Those cases are documented here.

> [!warning] warning:
> The behaviour described here is preserved strictly for backwards compatibility only. A homeserver should take reasonable precautions to prevent users from sending these so-called “malformed” events, and must never rely on the behaviours described here as a default.

###### `m.room.power_levels` events accept values as strings[](https://spec.matrix.org/v1.11/rooms/v8/#mroompower_levels-events-accept-values-as-strings)

In order to maintain backwards compatibility with early implementations, each of the integer-valued properties within [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) events can be encoded as strings instead of integers. This includes the nested values within the `events`, `notifications` and `users` properties. For example, the following is a valid `m.room.power_levels` event in this room version:

```
{
  "content": {
    "ban": "50",
    "events": {
      "m.room.power_levels": "100"
    },
    "events_default": "0",
    "state_default": "50",
    "users": {
      "@example:localhost": "100"
    },
    "users_default": "0"
  },
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "state_key": "",
  "type": "m.room.power_levels"
}
```

When the value is representative of an integer, they must be the following format:

- a single base 10 integer, no float values or decimal points, optionally with any number of leading zeroes (`"100"`, `"000100"`);
- optionally prefixed with a single `-` or `+` character before the integer (`"+100"`, `"-100"`).
- optionally with any number of leading or trailing whitespace characters (`" 100 "`, `" 00100 "`, `" +100 "`, `" -100 "`);

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v8/#state-resolution)

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v8/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v8/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v8/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

#### Canonical JSON[](https://spec.matrix.org/v1.11/rooms/v8/#canonical-json)

Servers MUST strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json). This translates to a 400 `M_BAD_JSON` error on most endpoints, or discarding of events over federation. For example, the Federation API’s `/send` endpoint would discard the event whereas the Client Server API’s `/send/{eventType}` endpoint would return a `M_BAD_JSON` error.

#### Signing key validity period[](https://spec.matrix.org/v1.11/rooms/v8/#signing-key-validity-period)

When validating event signatures, servers MUST enforce the `valid_until_ts` property from a key request is at least as large as the `origin_server_ts` for the event being validated. Servers missing a copy of the signing key MUST try to obtain one via the [GET /_matrix/key/v2/server](https://spec.matrix.org/v1.11/server-server-api#get_matrixkeyv2server) or [POST /_matrix/key/v2/query](https://spec.matrix.org/v1.11/server-server-api#post_matrixkeyv2query) APIs. When using the `/query` endpoint, servers MUST set the `minimum_valid_until_ts` property to prompt the notary server to attempt to refresh the key if appropriate.

Servers MUST use the lesser of `valid_until_ts` and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.

## Room Version 9

This room version builds on [version 8](https://spec.matrix.org/v1.11/rooms/v8) to add additional redaction rules that were unintentionally missed when incorporating v8.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v9/#client-considerations)

See [room version 8](https://spec.matrix.org/v1.11/rooms/v8) for specific details regarding the addition of restricted rooms.

Clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v9/#redactions) section below for a full overview.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v9/#redactions)

**[New in this version]** [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) events now keep `join_authorised_via_users_server` in addition to other keys in `content` when being redacted.

> [!info] RATIONALE:
> Without the `join_authorised_via_users_server` property, redacted join events can become invalid when verifying the auth chain of a given event, thus creating a split-brain scenario where the user is able to speak from one server’s perspective but most others will continually reject their events.
> 
> This can theoretically be worked around with a rejoin to the room, being careful not to use the faulty events as `prev_events`, though instead it is encouraged to use v9 rooms over v8 rooms to outright avoid the situation.
> 
> [Issue #3373](https://github.com/matrix-org/matrix-doc/issues/3373) has further information.

The full redaction algorithm follows.

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows keys `membership`, `join_authorised_via_users_server`.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows key `creator`.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows keys `join_rule`, `allow`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v9/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

Room version 8 added a new `restricted` join rule to allow members of a room to join another room without invite. Room version 9 is based upon v8 with the following considerations.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v9/#redactions-1)

[See above](https://spec.matrix.org/v1.11/rooms/v9/#redactions).

### Unchanged from v8[](https://spec.matrix.org/v1.11/rooms/v9/#unchanged-from-v8)

The following sections have not been modified since v8, but are included for completeness.

#### Handling redactions[](https://spec.matrix.org/v1.11/rooms/v9/#handling-redactions)

In room versions 1 and 2, redactions were explicitly part of the [authorization rules](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules) under Rule 11. As of room version 3, these conditions no longer exist as represented by [this version’s authorization rules](https://spec.matrix.org/v1.11/rooms/v9/#authorization-rules).

While redactions are always accepted by the authorization rules for events, they should not be sent to clients until both the redaction event and the event the redaction affects have been received, and can be validated. If both events are valid and have been seen by the server, then the server applies the redaction if one of the following conditions is met:

1. The power level of the redaction event’s `sender` is greater than or equal to the _redact level_.
2. The domain of the redaction event’s `sender` matches that of the original event’s `sender`.

If the server would apply a redaction, the redaction event is also sent to clients. Otherwise, the server simply waits for a valid partner event to arrive where it can then re-check the above.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v9/#event-ids)

The event ID is the [reference hash](https://spec.matrix.org/v1.11/server-server-api#calculating-the-reference-hash-for-an-event) of the event encoded using a variation of [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64) which replaces the 62nd and 63rd characters with `-` and `_` instead of using `+` and `/`. This matches [RFC4648’s definition of URL-safe base64](https://tools.ietf.org/html/rfc4648#section-5).

Event IDs are still prefixed with `$` and might result in looking like `$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg`.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v9/#event-format)

Events in rooms of this version have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room version 4 and beyond.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|`[string]`|**Required:**<br><br>Event IDs for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v9/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|`[string]`|**Required:**<br><br>Event IDs for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v9/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$urlsafe_base64_encoded_eventid",
    "$a-different-event-id"
  ],
  "content": {
    "key": "value"
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
  "redacts": "$some-old_event",
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v9/#deprecated-event-content-schemas)

Events sent into rooms of this version can have formats which are different from their normal schema. Those cases are documented here.

> [!warning] warning:
> The behaviour described here is preserved strictly for backwards compatibility only. A homeserver should take reasonable precautions to prevent users from sending these so-called “malformed” events, and must never rely on the behaviours described here as a default.

###### `m.room.power_levels` events accept values as strings[](https://spec.matrix.org/v1.11/rooms/v9/#mroompower_levels-events-accept-values-as-strings)

In order to maintain backwards compatibility with early implementations, each of the integer-valued properties within [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) events can be encoded as strings instead of integers. This includes the nested values within the `events`, `notifications` and `users` properties. For example, the following is a valid `m.room.power_levels` event in this room version:

```
{
  "content": {
    "ban": "50",
    "events": {
      "m.room.power_levels": "100"
    },
    "events_default": "0",
    "state_default": "50",
    "users": {
      "@example:localhost": "100"
    },
    "users_default": "0"
  },
  "origin_server_ts": 1432735824653,
  "room_id": "!jEsUZKDJdhlrceRyVU:example.org",
  "sender": "@example:example.org",
  "state_key": "",
  "type": "m.room.power_levels"
}
```

When the value is representative of an integer, they must be the following format:

- a single base 10 integer, no float values or decimal points, optionally with any number of leading zeroes (`"100"`, `"000100"`);
- optionally prefixed with a single `-` or `+` character before the integer (`"+100"`, `"-100"`).
- optionally with any number of leading or trailing whitespace characters (`" 100 "`, `" 00100 "`, `" +100 "`, `" -100 "`);

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v9/#authorization-rules)

Events must be signed by the server denoted by the `sender` property.

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

> [!info] info:
> `m.room.redaction` events are subject to auth rules in the same way as any other event. In practice, that means they will normally be allowed by the auth rules, unless the `m.room.power_levels` event sets a power level requirement for `m.room.redaction` events via the `events` or `events_default` properties. In particular, the _redact level_ is **not** considered by the auth rules.
>
>The ability to send a redaction event does not mean that the redaction itself should be performed. Receiving servers must perform additional checks, as described in the [Handling Redactions](https://spec.matrix.org/v1.11/rooms/v9/#handling-redactions) section.

The rules are as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Considering the event’s `auth_events`:
    1. If there are duplicate entries for a given `type` and `state_key` pair, reject.
    2. If there are entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification, reject.
    3. If there are entries which were themselves rejected under the [checks performed on receipt of a PDU](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu), reject.
    4. If there is no `m.room.create` event among the entries, reject.
3. If the `content` of the `m.room.create` event in the room state has the property `m.federate` set to `false`, and the `sender` domain of the event does not match the `sender` domain of the create event, reject.
4. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `content` has a `join_authorised_via_users_server` property:
        1. If the event is not validly signed by the homeserver of the user ID denoted by the key, reject.
    3. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` or `knock` then allow if membership state is `invite` or `join`.
        5. If the `join_rule` is `restricted`:
            1. If membership state is `join` or `invite`, allow.
            2. If the `join_authorised_via_users_server` key in `content` is not a user with sufficient permission to invite other users, reject.
            3. Otherwise, allow.
        6. If the `join_rule` is `public`, allow.
        7. Otherwise, reject.
    4. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    5. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite`, `join`, or `knock`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    6. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    7. If `membership` is `knock`:
        1. If the `join_rule` is anything other than `knock`, reject.
        2. If `sender` does not match `state_key`, reject.
        3. If the `sender`’s current membership is not `ban`, `invite`, or `join`, allow.
        4. Otherwise, reject.
    8. Otherwise, the membership is unknown. Reject.
5. If the `sender`’s current membership state is not `join`, reject.
6. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
7. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
8. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
9. If type is `m.room.power_levels`:
    1. If the `users` property in `content` is not an object with keys that are valid user IDs with values that are integers (or a string that is an integer), reject.
    2. If there is no previous `m.room.power_levels` event in the room, allow.
    3. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is higher than the `sender`’s current power level, reject.
        2. If the new value is higher than the `sender`’s current power level, reject.
    4. For each entry being changed in, or removed from, the `events` or `notifications` properties:
        1. If the current value is greater than the `sender`’s current power level, reject.
    5. For each entry being added to, or changed in the `events` or `notifications` properties:
        1. If the new value is greater than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. Otherwise, allow.
10. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v9/#state-resolution)

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v9/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v9/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v9/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

#### Canonical JSON[](https://spec.matrix.org/v1.11/rooms/v9/#canonical-json)

Servers MUST strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json). This translates to a 400 `M_BAD_JSON` error on most endpoints, or discarding of events over federation. For example, the Federation API’s `/send` endpoint would discard the event whereas the Client Server API’s `/send/{eventType}` endpoint would return a `M_BAD_JSON` error.

#### Signing key validity period[](https://spec.matrix.org/v1.11/rooms/v9/#signing-key-validity-period)

When validating event signatures, servers MUST enforce the `valid_until_ts` property from a key request is at least as large as the `origin_server_ts` for the event being validated. Servers missing a copy of the signing key MUST try to obtain one via the [GET /_matrix/key/v2/server](https://spec.matrix.org/v1.11/server-server-api#get_matrixkeyv2server) or [POST /_matrix/key/v2/query](https://spec.matrix.org/v1.11/server-server-api#post_matrixkeyv2query) APIs. When using the `/query` endpoint, servers MUST set the `minimum_valid_until_ts` property to prompt the notary server to attempt to refresh the key if appropriate.

Servers MUST use the lesser of `valid_until_ts` and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.

## Room Version 10

This room version builds on [version 9](https://spec.matrix.org/v1.11/rooms/v9) to enforce that power level values be integers, and to introduce a new `knock_restricted` join rule, allowing prospective members to more easily join such a room.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v10/#client-considerations)

This room version adds a new `knock_restricted` join rule to allow prospective members of a room join either through knocking (introduced in [room version 7](https://spec.matrix.org/v1.11/rooms/v7)) or through “join restrictions” (introduced in [room version 8](https://spec.matrix.org/v1.11/rooms/v8) and refined in [room version 9](https://spec.matrix.org/v1.11/rooms/v9)).

Clients should render the new join rule accordingly for such rooms. For example:

```
This room is:
[ ] Public
[x] Private

Join rules (disabled when Public):
[x] Allow members of `#space:example.org` to join
[x] Allow knocking
```

Clients which implement the redaction algorithm locally should refer to the [redactions](https://spec.matrix.org/v1.11/rooms/v10/#redactions) section below for a full overview.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v10/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

[Room version 7](https://spec.matrix.org/v1.11/rooms/v7) added “knocking” and [room version 8](https://spec.matrix.org/v1.11/rooms/v8) added “join restrictions” (refined by [room version 9](https://spec.matrix.org/v1.11/rooms/v9)) — both allow prospective members an avenue to join, however it was not possible to use the two mechanisms together. This room version adds a new `knock_restricted` join rule as a mix of the two behaviours, allowing a user to join the room if they meet either the `restricted` join rule criteria or the `knock` join rule criteria.

This room version additionally requires that values in the power levels event be integers and not string representations, unlike other room versions.

Room version 10 is based upon room version 9 with the following considerations.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v10/#event-format)

The event format is unchanged by this room version. See [below](https://spec.matrix.org/v1.11/rooms/v10/#event-format-1) for details on the current event format.

##### Deprecated event content schemas[](https://spec.matrix.org/v1.11/rooms/v10/#deprecated-event-content-schemas)

While this room version does not change the event format specifically, some deprecated behaviours are strictly no longer supported.

###### Values in `m.room.power_levels` events must be integers[](https://spec.matrix.org/v1.11/rooms/v10/#values-in-mroompower_levels-events-must-be-integers)

In other room versions, such as [v9](https://spec.matrix.org/v1.11/rooms/v9/#mroompower_levels-events-accept-values-as-strings), power levels could be represented as strings for backwards compatibility.

This backwards compatibility is removed in this room version - power levels MUST NOT be represented as strings within this room version. Power levels which are not correctly structured are rejected under the authorization rules below.

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v10/#authorization-rules)

Events must be signed by the server denoted by the `sender` property.

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

> [!info] info:
> `m.room.redaction` events are subject to auth rules in the same way as any other event. In practice, that means they will normally be allowed by the auth rules, unless the `m.room.power_levels` event sets a power level requirement for `m.room.redaction` events via the `events` or `events_default` properties. In particular, the _redact level_ is **not** considered by the auth rules.
> 
> The ability to send a redaction event does not mean that the redaction itself should be performed. Receiving servers must perform additional checks, as described in the [Handling redactions](https://spec.matrix.org/v1.11/rooms/v10/#handling-redactions) section.

The rules are as follows:

1. If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. If `content` has no `creator` property, reject.
    5. Otherwise, allow.
2. Considering the event’s `auth_events`:
    1. If there are duplicate entries for a given `type` and `state_key` pair, reject.
    2. If there are entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification, reject.
    3. If there are entries which were themselves rejected under the [checks performed on receipt of a PDU](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu), reject.
    4. If there is no `m.room.create` event among the entries, reject.
3. If the `content` of the `m.room.create` event in the room state has the property `m.federate` set to `false`, and the `sender` domain of the event does not match the `sender` domain of the create event, reject.
4. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `content` has a `join_authorised_via_users_server` key:
        1. If the event is not validly signed by the homeserver of the user ID denoted by the key, reject.
    3. If `membership` is `join`:
        1. If the only previous event is an `m.room.create` and the `state_key` is the creator, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` or `knock` then allow if membership state is `invite` or `join`.
        5. **[Changed in this version]** If the `join_rule` is `restricted` or `knock_restricted`:
            1. If membership state is `join` or `invite`, allow.
            2. If the `join_authorised_via_users_server` key in `content` is not a user with sufficient permission to invite other users, reject.
            3. Otherwise, allow.
        6. If the `join_rule` is `public`, allow.
        7. Otherwise, reject.
    4. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    5. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite`, `join`, or `knock`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    6. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    7. If `membership` is `knock`:
        1. **[Changed in this version]** If the `join_rule` is anything other than `knock` or `knock_restricted`, reject.
        2. If `sender` does not match `state_key`, reject.
        3. If the `sender`’s current membership is not `ban`, `invite`, or `join`, allow.
        4. Otherwise, reject.
    8. Otherwise, the membership is unknown. Reject.
5. If the `sender`’s current membership state is not `join`, reject.
6. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
7. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
8. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
9. If type is `m.room.power_levels`:
    1. **[New in this version]** If any of the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, or `invite` in `content` are present and not an integer, reject.
    2. **[New in this version]** If either of the properties `events` or `notifications` in `content` are present and not an object with values that are integers, reject.
    3. If the `users` property in `content` is not an object with keys that are valid user IDs with values that are integers, reject.
    4. If there is no previous `m.room.power_levels` event in the room, allow.
    5. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is higher than the `sender`’s current power level, reject.
        2. If the new value is higher than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `events` or `notifications` properties:
        1. If the current value is greater than the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `events` or `notifications` properties:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    9. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    10. Otherwise, allow.
10. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

### Unchanged from v9[](https://spec.matrix.org/v1.11/rooms/v10/#unchanged-from-v9)

The following sections have not been modified since v9, but are included for completeness.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v10/#redactions)

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `prev_state`
- `auth_events`
- `origin`
- `origin_server_ts`
- `membership`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows keys `membership`, `join_authorised_via_users_server`.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows key `creator`.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows keys `join_rule`, `allow`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.

#### Handling redactions[](https://spec.matrix.org/v1.11/rooms/v10/#handling-redactions)

In room versions 1 and 2, redactions were explicitly part of the [authorization rules](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules) under Rule 11. As of room version 3, these conditions no longer exist as represented by [this version’s authorization rules](https://spec.matrix.org/v1.11/rooms/v10/#authorization-rules).

While redactions are always accepted by the authorization rules for events, they should not be sent to clients until both the redaction event and the event the redaction affects have been received, and can be validated. If both events are valid and have been seen by the server, then the server applies the redaction if one of the following conditions is met:

1. The power level of the redaction event’s `sender` is greater than or equal to the _redact level_.
2. The domain of the redaction event’s `sender` matches that of the original event’s `sender`.

If the server would apply a redaction, the redaction event is also sent to clients. Otherwise, the server simply waits for a valid partner event to arrive where it can then re-check the above.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v10/#event-ids)

The event ID is the [reference hash](https://spec.matrix.org/v1.11/server-server-api#calculating-the-reference-hash-for-an-event) of the event encoded using a variation of [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64) which replaces the 62nd and 63rd characters with `-` and `_` instead of using `+` and `/`. This matches [RFC4648’s definition of URL-safe base64](https://tools.ietf.org/html/rfc4648#section-5).

Event IDs are still prefixed with `$` and might result in looking like `$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg`.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v10/#event-format-1)

Events in rooms of this version have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room version 4 and beyond.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|`[string]`|**Required:**<br><br>Event IDs for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v10/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|`[string]`|**Required:**<br><br>Event IDs for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`redacts`|`string`|For redaction events, the ID of the event being redacted.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v10/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$urlsafe_base64_encoded_eventid",
    "$a-different-event-id"
  ],
  "content": {
    "key": "value"
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
  "redacts": "$some-old_event",
  "room_id": "!UcYsUzyxTGDxLBEvLy:example.org",
  "sender": "@alice:example.com",
  "signatures": {
    "example.com": {
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v10/#state-resolution)

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v10/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v10/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v10/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

#### Canonical JSON [](https://spec.matrix.org/v1.11/rooms/v10/#canonical-json)

Servers MUST strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json). This translates to a 400 `M_BAD_JSON` error on most endpoints, or discarding of events over federation. For example, the Federation API’s `/send` endpoint would discard the event whereas the Client Server API’s `/send/{eventType}` endpoint would return a `M_BAD_JSON` error.

#### Signing key validity period[](https://spec.matrix.org/v1.11/rooms/v10/#signing-key-validity-period)

When validating event signatures, servers MUST enforce the `valid_until_ts` property from a key request is at least as large as the `origin_server_ts` for the event being validated. Servers missing a copy of the signing key MUST try to obtain one via the [GET /_matrix/key/v2/server](https://spec.matrix.org/v1.11/server-server-api#get_matrixkeyv2server) or [POST /_matrix/key/v2/query](https://spec.matrix.org/v1.11/server-server-api#post_matrixkeyv2query) APIs. When using the `/query` endpoint, servers MUST set the `minimum_valid_until_ts` property to prompt the notary server to attempt to refresh the key if appropriate.

Servers MUST use the lesser of `valid_until_ts` and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.


## Room Version 11

This room version builds on [version 10](https://spec.matrix.org/v1.11/rooms/v10) while clarifying redaction rules.

### Client considerations[](https://spec.matrix.org/v1.11/rooms/v11/#client-considerations)

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v11/#redactions)

**[New in this version]** The top-level `origin`, `membership`, and `prev_state` properties are no longer protected from redaction. The [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) event now keeps the entire `content` property. The [`m.room.redaction`](https://spec.matrix.org/v1.11/client-server-api#mroomredaction) event keeps the `redacts` property under `content`. The [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) event keeps the `invite` property under `content`.

The full redaction algorithm follows.

Upon receipt of a redaction event, the server must strip off any keys not in the following list:

- `event_id`
- `type`
- `room_id`
- `sender`
- `state_key`
- `content`
- `hashes`
- `signatures`
- `depth`
- `prev_events`
- `auth_events`
- `origin_server_ts`

The content object must also be stripped of all keys, unless it is one of the following event types:

- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) allows keys `membership`, `join_authorised_via_users_server`. Additionally, it allows the `signed` key of the `third_party_invite` key.
- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) allows all keys.
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules) allows keys `join_rule`, `allow`.
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels) allows keys `ban`, `events`, `events_default`, `invite`, `kick`, `redact`, `state_default`, `users`, `users_default`.
- [`m.room.history_visibility`](https://spec.matrix.org/v1.11/client-server-api#mroomhistory_visibility) allows key `history_visibility`.
- [`m.room.redaction`](https://spec.matrix.org/v1.11/client-server-api#mroomredaction) allows key `redacts`.

#### Event format[](https://spec.matrix.org/v1.11/rooms/v11/#event-format)

Clients should no longer depend on the `creator` property in the `content` of [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate) events. In all room versions, clients can rely on `sender` instead to determine a room creator.

The format of [`m.room.redaction`](https://spec.matrix.org/v1.11/client-server-api#mroomredaction) events has been modified. Client should look for the `redacts` key under `content` instead of a top-level event property.

The `third_party_invite` key of [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember) events is no longer redacted, _but_ will only contain the `signed` key after redaction.

### Server implementation components[](https://spec.matrix.org/v1.11/rooms/v11/#server-implementation-components)

> [!warning] warning:
> The information contained in this section is strictly for server implementors. Applications which use the Client-Server API are generally unaffected by the intricacies contained here. The section above regarding client considerations is the resource that Client-Server API use cases should reference.

This room version updates the redaction algorithm and modifies how servers should create `m.room.create` and `m.room.redaction` events.

Room version 11 is based upon room version 10 with the following considerations.

#### Redactions[](https://spec.matrix.org/v1.11/rooms/v11/#redactions-1)

[See above](https://spec.matrix.org/v1.11/rooms/v11/#redactions).

#### Event format[](https://spec.matrix.org/v1.11/rooms/v11/#event-format-1)

The core event format is the same as [room version 10](https://spec.matrix.org/v1.11/rooms/v10#event-format). However, this room version changes some properties of some event types.

Events in rooms of this version have the following structure:

##### `Persistent Data Unit`

---

A persistent data unit (event) for room version 11 and beyond.

|Persistent Data Unit|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`auth_events`|`[string]`|**Required:**<br><br>Event IDs for the authorization events that would allow this event to be in the room.<br><br>Must contain less than or equal to 10 events. Note that if the relevant auth event selection rules are used, this restriction should never be encountered.|
|`content`|`object`|**Required:** The content of the event.|
|`depth`|`integer`|**Required:** The maximum depth of the `prev_events`, plus one. Must be less than the maximum value for an integer (2^63 - 1). If the room’s depth is already at the limit, the depth must be set to the limit.|
|`hashes`|[Event Hash](https://spec.matrix.org/v1.11/rooms/v11/#definition-persistent-data-unit_event-hash)|**Required:** Content hashes of the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`origin_server_ts`|`integer`|**Required:** Timestamp in milliseconds on origin homeserver when this event was created.|
|`prev_events`|`[string]`|**Required:**<br><br>Event IDs for the most recent events in the room that the homeserver was aware of when it made this event.<br><br>Must contain less than or equal to 20 events.|
|`room_id`|`string`|**Required:** Room identifier.|
|`sender`|`string`|**Required:** The ID of the user sending the event.|
|`signatures`|`{string: {string: string}}`|**Required:** Signatures for the PDU, following the algorithm specified in [Signing Events](https://spec.matrix.org/v1.11/server-server-api/#signing-events).|
|`state_key`|`string`|If this key is present, the event is a state event, and it will replace previous events with the same `type` and `state_key` in the room state.|
|`type`|`string`|**Required:** Event type|
|`unsigned`|[UnsignedData](https://spec.matrix.org/v1.11/rooms/v11/#definition-persistent-data-unit_unsigneddata)|Additional data added by the origin server but not covered by the `signatures`.|

|Event Hash|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`sha256`|`string`|**Required:** The hash.|

|UnsignedData|
|---|---|---|
|Name|Type|Description|
|---|---|---|
|`age`|`integer`|The number of milliseconds that have passed since this message was sent.|

###### Examples

```
{
  "auth_events": [
    "$urlsafe_base64_encoded_eventid",
    "$a-different-event-id"
  ],
  "content": {
    "key": "value"
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
      "ed25519:key_version:": "these86bytesofbase64signaturecoveressentialfieldsincludinghashessocancheckredactedpdus"
    }
  },
  "type": "m.room.message",
  "unsigned": {
    "age": 4612
  }
}
```

##### Remove the `creator` property of `m.room.create` events[](https://spec.matrix.org/v1.11/rooms/v11/#remove-the-creator-property-of-mroomcreate-events)

The `content` of a `m.room.create` event no longer has a `creator` property, which previously was always equivalent to the `sender` of the event.

##### Moving the `redacts` property of `m.room.redaction` events to a `content` property[](https://spec.matrix.org/v1.11/rooms/v11/#moving-the-redacts-property-of-mroomredaction-events-to-a-content-property)

The `redacts` property of `m.room.redaction` events is moved from a top-level event property to a property under the event `content`.

For backwards-compatibility with older clients, servers should add a `redacts` property to the top level of `m.room.redaction` events when serving such events over the Client-Server API.

For improved compatibility with newer clients, servers should add a `redacts` property to the `content` of `m.room.redaction` events in _older_ room versions when serving such events over the Client-Server API.

#### Authorization rules[](https://spec.matrix.org/v1.11/rooms/v11/#authorization-rules)

Events must be signed by the server denoted by the `sender` property.

The types of state events that affect authorization are:

- [`m.room.create`](https://spec.matrix.org/v1.11/client-server-api#mroomcreate)
- [`m.room.member`](https://spec.matrix.org/v1.11/client-server-api#mroommember)
- [`m.room.join_rules`](https://spec.matrix.org/v1.11/client-server-api#mroomjoin_rules)
- [`m.room.power_levels`](https://spec.matrix.org/v1.11/client-server-api#mroompower_levels)
- [`m.room.third_party_invite`](https://spec.matrix.org/v1.11/client-server-api#mroomthird_party_invite)

> [!info] info:
> Power levels are inferred from defaults when not explicitly supplied. For example, mentions of the `sender`’s power level can also refer to the default power level for users in the room.

> [!info] info:
> `m.room.redaction` events are subject to auth rules in the same way as any other event. In practice, that means they will normally be allowed by the auth rules, unless the `m.room.power_levels` event sets a power level requirement for `m.room.redaction` events via the `events` or `events_default` properties. In particular, the _redact level_ is **not** considered by the auth rules.
> 
> The ability to send a redaction event does not mean that the redaction itself should be performed. Receiving servers must perform additional checks, as described in the [Handling redactions](https://spec.matrix.org/v1.11/rooms/v11/#handling-redactions) section.

The rules are as follows:

1. **[Changed in this version]** If type is `m.room.create`:
    1. If it has any `prev_events`, reject.
    2. If the domain of the `room_id` does not match the domain of the `sender`, reject.
    3. If `content.room_version` is present and is not a recognised version, reject.
    4. Otherwise, allow.
2. Considering the event’s `auth_events`:
    1. If there are duplicate entries for a given `type` and `state_key` pair, reject.
    2. If there are entries whose `type` and `state_key` don’t match those specified by the [auth events selection](https://spec.matrix.org/v1.11/server-server-api#auth-events-selection) algorithm described in the server specification, reject.
    3. If there are entries which were themselves rejected under the [checks performed on receipt of a PDU](https://spec.matrix.org/v1.11/server-server-api/#checks-performed-on-receipt-of-a-pdu), reject.
    4. If there is no `m.room.create` event among the entries, reject.
3. If the `content` of the `m.room.create` event in the room state has the property `m.federate` set to `false`, and the `sender` domain of the event does not match the `sender` domain of the create event, reject.
4. If type is `m.room.member`:
    1. If there is no `state_key` property, or no `membership` property in `content`, reject.
    2. If `content` has a `join_authorised_via_users_server` key:
        1. If the event is not validly signed by the homeserver of the user ID denoted by the key, reject.
    3. If `membership` is `join`:
        1. **[Changed in this version]** If the only previous event is an `m.room.create` and the `state_key` is the sender, allow.
        2. If the `sender` does not match `state_key`, reject.
        3. If the `sender` is banned, reject.
        4. If the `join_rule` is `invite` or `knock` then allow if membership state is `invite` or `join`.
        5. If the `join_rule` is `restricted` or `knock_restricted`:
            1. If membership state is `join` or `invite`, allow.
            2. If the `join_authorised_via_users_server` key in `content` is not a user with sufficient permission to invite other users, reject.
            3. Otherwise, allow.
        6. If the `join_rule` is `public`, allow.
        7. Otherwise, reject.
    4. If `membership` is `invite`:
        1. If `content` has a `third_party_invite` property:
            1. If _target user_ is banned, reject.
            2. If `content.third_party_invite` does not have a `signed` property, reject.
            3. If `signed` does not have `mxid` and `token` properties, reject.
            4. If `mxid` does not match `state_key`, reject.
            5. If there is no `m.room.third_party_invite` event in the current room state with `state_key` matching `token`, reject.
            6. If `sender` does not match `sender` of the `m.room.third_party_invite`, reject.
            7. If any signature in `signed` matches any public key in the `m.room.third_party_invite` event, allow. The public keys are in `content` of `m.room.third_party_invite` as:
                1. A single public key in the `public_key` property.
                2. A list of public keys in the `public_keys` property.
            8. Otherwise, reject.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If _target user_’s current membership state is `join` or `ban`, reject.
        4. If the `sender`’s power level is greater than or equal to the _invite level_, allow.
        5. Otherwise, reject.
    5. If `membership` is `leave`:
        1. If the `sender` matches `state_key`, allow if and only if that user’s current membership state is `invite`, `join`, or `knock`.
        2. If the `sender`’s current membership state is not `join`, reject.
        3. If the _target user_’s current membership state is `ban`, and the `sender`’s power level is less than the _ban level_, reject.
        4. If the `sender`’s power level is greater than or equal to the _kick level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        5. Otherwise, reject.
    6. If `membership` is `ban`:
        1. If the `sender`’s current membership state is not `join`, reject.
        2. If the `sender`’s power level is greater than or equal to the _ban level_, and the _target user_’s power level is less than the `sender`’s power level, allow.
        3. Otherwise, reject.
    7. If `membership` is `knock`:
        1. If the `join_rule` is anything other than `knock` or `knock_restricted`, reject.
        2. If `sender` does not match `state_key`, reject.
        3. If the `sender`’s current membership is not `ban`, `invite`, or `join`, allow.
        4. Otherwise, reject.
    8. Otherwise, the membership is unknown. Reject.
5. If the `sender`’s current membership state is not `join`, reject.
6. If type is `m.room.third_party_invite`:
    1. Allow if and only if `sender`’s current power level is greater than or equal to the _invite level_.
7. If the event type’s _required power level_ is greater than the `sender`’s power level, reject.
8. If the event has a `state_key` that starts with an `@` and does not match the `sender`, reject.
9. If type is `m.room.power_levels`:
    1. If any of the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, or `invite` in `content` are present and not an integer, reject.
    2. If either of the properties `events` or `notifications` in `content` are present and not an object with values that are integers, reject.
    3. If the `users` property in `content` is not an object with keys that are valid user IDs with values that are integers, reject.
    4. If there is no previous `m.room.power_levels` event in the room, allow.
    5. For the properties `users_default`, `events_default`, `state_default`, `ban`, `redact`, `kick`, `invite` check if they were added, changed or removed. For each found alteration:
        1. If the current value is higher than the `sender`’s current power level, reject.
        2. If the new value is higher than the `sender`’s current power level, reject.
    6. For each entry being changed in, or removed from, the `events` or `notifications` properties:
        1. If the current value is greater than the `sender`’s current power level, reject.
    7. For each entry being added to, or changed in, the `events` or `notifications` properties:
        1. If the new value is greater than the `sender`’s current power level, reject.
    8. For each entry being changed in, or removed from, the `users` property, other than the `sender`’s own entry:
        1. If the current value is greater than or equal to the `sender`’s current power level, reject.
    9. For each entry being added to, or changed in, the `users` property:
        1. If the new value is greater than the `sender`’s current power level, reject.
    10. Otherwise, allow.
10. Otherwise, allow.

> [!info] info:
> Some consequences of these rules:
> 
> - Unless you are a member of the room, the only permitted operations (apart from the initial create/join) are: joining a public room; accepting or rejecting an invitation to a room.
> - To unban somebody, you must have power level greater than or equal to both the kick _and_ ban levels, _and_ greater than the target user’s power level.

### Unchanged from v10[](https://spec.matrix.org/v1.11/rooms/v11/#unchanged-from-v10)

The following sections have not been modified since v10, but are included for completeness.

#### Handling redactions[](https://spec.matrix.org/v1.11/rooms/v11/#handling-redactions)

In room versions 1 and 2, redactions were explicitly part of the [authorization rules](https://spec.matrix.org/v1.11/rooms/v1/#authorization-rules) under Rule 11. As of room version 3, these conditions no longer exist as represented by [this version’s authorization rules](https://spec.matrix.org/v1.11/rooms/v11/#authorization-rules).

While redactions are always accepted by the authorization rules for events, they should not be sent to clients until both the redaction event and the event the redaction affects have been received, and can be validated. If both events are valid and have been seen by the server, then the server applies the redaction if one of the following conditions is met:

1. The power level of the redaction event’s `sender` is greater than or equal to the _redact level_.
2. The domain of the redaction event’s `sender` matches that of the original event’s `sender`.

If the server would apply a redaction, the redaction event is also sent to clients. Otherwise, the server simply waits for a valid partner event to arrive where it can then re-check the above.

#### Event IDs[](https://spec.matrix.org/v1.11/rooms/v11/#event-ids)

The event ID is the [reference hash](https://spec.matrix.org/v1.11/server-server-api#calculating-the-reference-hash-for-an-event) of the event encoded using a variation of [Unpadded Base64](https://spec.matrix.org/v1.11/appendices#unpadded-base64) which replaces the 62nd and 63rd characters with `-` and `_` instead of using `+` and `/`. This matches [RFC4648’s definition of URL-safe base64](https://tools.ietf.org/html/rfc4648#section-5).

Event IDs are still prefixed with `$` and might result in looking like `$Rqnc-F-dvnEYJTyHq_iKxU2bZ1CI92-kuZq3a5lr5Zg`.

#### State resolution[](https://spec.matrix.org/v1.11/rooms/v11/#state-resolution)

The room state _S′(E)_ after an event _E_ is defined in terms of the room state _S(E)_ before _E_, and depends on whether _E_ is a state event or a message event:

- If _E_ is a message event, then _S′(E)_ = _S(E)_.
- If _E_ is a state event, then _S′(E)_ is _S(E)_, except that its entry corresponding to the `event_type` and `state_key` of _E_ is replaced by the `event_id` of _E_.

The room state _S(E)_ before _E_ is the _resolution_ of the set of states {_S′(E_1_)_, _S′(E_2_)_, …} after the `prev_event`s {_E_1, _E_2, …} of _E_. The resolution of a set of states is given in the algorithm below.

##### Definitions[](https://spec.matrix.org/v1.11/rooms/v11/#definitions)

The state resolution algorithm for version 2 rooms uses the following definitions, given the set of room states {_S_1, _S_2, …}:

**Power events.** A _power event_ is a state event with type `m.room.power_levels` or `m.room.join_rules`, or a state event with type `m.room.member` where the `membership` is `leave` or `ban` and the `sender` does not match the `state_key`. The idea behind this is that power events are events that might remove someone’s ability to do something in the room.

**Unconflicted state map and conflicted state set.** The keys of the state maps _Si_ are 2-tuples of strings of the form _K_ = `(event_type, state_key)`. The values _V_ are state events. The key-value pairs (_K_, _V_) across all state maps _Si_ can be divided into two collections. If a given key _K_ is present in every _Si_ with the same value _V_ in each state map, then the pair (_K_, _V_) belongs to the _unconflicted state map_. Otherwise, _V_ belongs to the _conflicted state set_.

Note that the unconflicted state map only has one event for each key _K_, whereas the conflicted state set may contain multiple events with the same key.

**Auth chain.** The _auth chain_ of an event _E_ is the set containing all of _E_’s auth events, all of _their_ auth events, and so on recursively, stretching back to the start of the room. Put differently, these are the events reachable by walking the graph induced by an event’s `auth_events` links.

**Auth difference.** The _auth difference_ is calculated by first calculating the full auth chain for each state _S__i_, that is the union of the auth chains for each event in _S__i_, and then taking every event that doesn’t appear in every auth chain. If _C__i_ is the full auth chain of _S__i_, then the auth difference is  ∪ _C__i_ −  ∩ _C__i_.

**Full conflicted set.** The _full conflicted set_ is the union of the conflicted state set and the auth difference.

**Reverse topological power ordering.** The _reverse topological power ordering_ of a set of events is the lexicographically smallest topological ordering based on the DAG formed by auth events. The reverse topological power ordering is ordered from earliest event to latest. For comparing two topological orderings to determine which is the lexicographically smallest, the following comparison relation on events is used: for events _x_ and _y_, _x_ < _y_ if

1. _x_’s sender has _greater_ power level than _y_’s sender, when looking at their respective `auth_event`s; or
2. the senders have the same power level, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the senders have the same power level and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

The reverse topological power ordering can be found by sorting the events using Kahn’s algorithm for topological sorting, and at each step selecting, among all the candidate vertices, the smallest vertex using the above comparison relation.

**Mainline ordering.** Let _P_ = _P_0 be an `m.room.power_levels` event. Starting with _i_ = 0, repeatedly fetch _P__i_+1, the `m.room.power_levels` event in the `auth_events` of _Pi_. Increment _i_ and repeat until _Pi_ has no `m.room.power_levels` event in its `auth_events`. The _mainline of P_0 is the list of events [_P_0 , _P_1, … , _Pn_], fetched in this way.

Let _e_ = _e0_ be another event (possibly another `m.room.power_levels` event). We can compute a similar list of events [_e_1, …, _em_], where _e__j_+1 is the `m.room.power_levels` event in the `auth_events` of _ej_ and where _em_ has no `m.room.power_levels` event in its `auth_events`. (Note that the event we started with, _e0_, is not included in this list. Also note that it may be empty, because _e_ may not cite an `m.room.power_levels` event in its `auth_events` at all.)

Now compare these two lists as follows.

- Find the smallest index _j_ ≥ 1 for which _ej_ belongs to the mainline of _P_.
- If such a _j_ exists, then _ej_ = _Pi_ for some unique index _i_ ≥ 0. Otherwise set _i_ = ∞, where ∞ is a sentinel value greater than any integer.
- In both cases, the _mainline position_ of _e_ is _i_.

Given mainline positions calculated from _P_, the _mainline ordering based on_ _P_ of a set of events is the ordering, from smallest to largest, using the following comparison relation on events: for events _x_ and _y_, _x_ < _y_ if

1. the mainline position of _x_ is **greater** than the mainline position of _y_ (i.e. the auth chain of _x_ is based on an earlier event in the mainline than _y_); or
2. the mainline positions of the events are the same, but _x_’s `origin_server_ts` is _less_ than _y_’s `origin_server_ts`; or
3. the mainline positions of the events are the same and the events have the same `origin_server_ts`, but _x_’s `event_id` is _less_ than _y_’s `event_id`.

**Iterative auth checks.** The _iterative auth checks algorithm_ takes as input an initial room state and a sorted list of state events, and constructs a new room state by iterating through the event list and applying the state event to the room state if the state event is allowed by the [authorization rules](https://spec.matrix.org/v1.11/server-server-api#authorization-rules). If the state event is not allowed by the authorization rules, then the event is ignored. If a `(event_type, state_key)` key that is required for checking the authorization rules is not present in the state, then the appropriate state event from the event’s `auth_events` is used if the auth event is not rejected.

##### Algorithm[](https://spec.matrix.org/v1.11/rooms/v11/#algorithm)

The _resolution_ of a set of states is obtained as follows:

1. Select the set _X_ of all _power events_ that appear in the _full conflicted set_. For each such power event _P_, enlarge _X_ by adding the events in the auth chain of _P_ which also belong to the full conflicted set. Sort $X$ into a list using the _reverse topological power ordering_.
2. Apply the _iterative auth checks algorithm_, starting from the _unconflicted state map_, to the list of events from the previous step to get a partially resolved state.
3. Take all remaining events that weren’t picked in step 1 and order them by the mainline ordering based on the power level in the partially resolved state obtained in step 2.
4. Apply the _iterative auth checks algorithm_ on the partial resolved state and the list of events from the previous step.
5. Update the result by replacing any event with the event with the same key from the _unconflicted state map_, if such an event exists, to get the final resolved state.

##### Rejected events[](https://spec.matrix.org/v1.11/rooms/v11/#rejected-events)

Events that have been rejected due to failing auth based on the state at the event (rather than based on their auth chain) are handled as usual by the algorithm, unless otherwise specified.

Note that no events rejected due to failure to auth against their auth chain should appear in the process, as they should not appear in state (the algorithm only uses events that appear in either the state sets or in the auth chain of the events in the state sets).

> [!info] RATIONALE:
> This helps ensure that different servers’ view of state is more likely to converge, since rejection state of an event may be different. This can happen if a third server gives an incorrect version of the state when a server joins a room via it (either due to being faulty or malicious). Convergence of state is a desirable property as it ensures that all users in the room have a (mostly) consistent view of the state of the room. If the view of the state on different servers diverges it can lead to bifurcation of the room due to e.g. servers disagreeing on who is in the room.
> 
> Intuitively, using rejected events feels dangerous, however:
> 
> 1. Servers cannot arbitrarily make up state, since they still need to pass the auth checks based on the event’s auth chain (e.g. they can’t grant themselves power levels if they didn’t have them before).
> 2. For a previously rejected event to pass auth there must be a set of state that allows said event. A malicious server could therefore produce a fork where it claims the state is that particular set of state, duplicate the rejected event to point to that fork, and send the event. The duplicated event would then pass the auth checks. Ignoring rejected events would therefore not eliminate any potential attack vectors.

Rejected auth events are deliberately excluded from use in the iterative auth checks, as auth events aren’t re-authed (although non-auth events are) during the iterative auth checks.

#### Canonical JSON[](https://spec.matrix.org/v1.11/rooms/v11/#canonical-json)

Servers MUST strictly enforce the JSON format specified in the [appendices](https://spec.matrix.org/v1.11/appendices#canonical-json). This translates to a 400 `M_BAD_JSON` error on most endpoints, or discarding of events over federation. For example, the Federation API’s `/send` endpoint would discard the event whereas the Client Server API’s `/send/{eventType}` endpoint would return a `M_BAD_JSON` error.

#### Signing key validity period[](https://spec.matrix.org/v1.11/rooms/v11/#signing-key-validity-period)

When validating event signatures, servers MUST enforce the `valid_until_ts` property from a key request is at least as large as the `origin_server_ts` for the event being validated. Servers missing a copy of the signing key MUST try to obtain one via the [GET /_matrix/key/v2/server](https://spec.matrix.org/v1.11/server-server-api#get_matrixkeyv2server) or [POST /_matrix/key/v2/query](https://spec.matrix.org/v1.11/server-server-api#post_matrixkeyv2query) APIs. When using the `/query` endpoint, servers MUST set the `minimum_valid_until_ts` property to prompt the notary server to attempt to refresh the key if appropriate.

Servers MUST use the lesser of `valid_until_ts` and 7 days into the future when determining if a key is valid. This is to avoid a situation where an attacker publishes a key which is valid for a significant amount of time without a way for the homeserver owner to revoke it.
