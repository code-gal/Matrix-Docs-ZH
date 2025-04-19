   decode the contents of the JSON web token. Required if `enabled` is set to true.
* `algorithm`: The algorithm used to sign (or HMAC) the JSON web token.
   Supported algorithms are listed
   [here (section JWS)](https://docs.authlib.org/en/latest/specs/rfc7518.html).
   Required if `enabled` is set to true.
* `subject_claim`: Name of the claim containing a unique identifier for the user.
   Optional, defaults to `sub`.
* `display_name_claim`: Name of the claim containing the display name for the user. Optional.
   If provided, the display name will be set to the value of this claim upon first login.
* `issuer`: The issuer to validate the "iss" claim against. Optional. If provided the
   "iss" claim will be required and validated for all JSON web tokens.
* `audiences`: A list of audiences to validate the "aud" claim against. Optional.
   If provided the "aud" claim will be required and validated for all JSON web tokens.
   Note that if the "aud" claim is included in a JSON web token then
   validation will fail without configuring audiences.

Example configuration:
```yaml
jwt_config:
    enabled: true
    secret: "provided-by-your-issuer"
    algorithm: "provided-by-your-issuer"
    subject_claim: "name_of_claim"
    display_name_claim: "name_of_claim"
    issuer: "provided-by-your-issuer"
    audiences:
        - "provided-by-your-issuer"
```
---
### `password_config`

Use this setting to enable password-based logins.

This setting has the following sub-options:
* `enabled`: Defaults to true.
   Set to false to disable password authentication.
   Set to `only_for_reauth` to allow users with existing passwords to use them
   to reauthenticate (not log in), whilst preventing new users from setting passwords.
* `localdb_enabled`: Set to false to disable authentication against the local password
   database. This is ignored if `enabled` is false, and is only useful
   if you have other `password_providers`. Defaults to true.
* `pepper`: Set the value here to a secret random string for extra security.
   DO NOT CHANGE THIS AFTER INITIAL SETUP!
* `policy`: Define and enforce a password policy, such as minimum lengths for passwords, etc.
   Each parameter is optional. This is an implementation of MSC2000. Parameters are as follows:
   * `enabled`: Defaults to false. Set to true to enable.
   * `minimum_length`: Minimum accepted length for a password. Defaults to 0.
   * `require_digit`: Whether a password must contain at least one digit.
      Defaults to false.
   * `require_symbol`: Whether a password must contain at least one symbol.
      A symbol is any character that's not a number or a letter. Defaults to false.
   * `require_lowercase`: Whether a password must contain at least one lowercase letter.
      Defaults to false.
   * `require_uppercase`: Whether a password must contain at least one uppercase letter.
      Defaults to false.


Example configuration:
```yaml
password_config:
   enabled: false
   localdb_enabled: false
   pepper: "EVEN_MORE_SECRET"

   policy:
      enabled: true
      minimum_length: 15
      require_digit: true
      require_symbol: true
      require_lowercase: true
      require_uppercase: true
```
---
## Push
Configuration settings related to push notifications

---
### `push`

This setting defines options for push notifications.

This option has a number of sub-options. They are as follows:
* `enabled`: Enables or disables push notification calculation. Note, disabling this will also
   stop unread counts being calculated for rooms. This mode of operation is intended
   for homeservers which may only have bots or appservice users connected, or are otherwise
   not interested in push/unread counters. This is enabled by default.
* `include_content`: Clients requesting push notifications can either have the body of
   the message sent in the notification poke along with other details
   like the sender, or just the event ID and room ID (`event_id_only`).
   If clients choose the to have the body sent, this option controls whether the
   notification request includes the content of the event (other details
   like the sender are still included). If `event_id_only` is enabled, it
   has no effect.
   For modern android devices the notification content will still appear
   because it is loaded by the app. iPhone, however will send a
   notification saying only that a message arrived and who it came from.
   Defaults to true. Set to false to only include the event ID and room ID in push notification payloads.
* `group_unread_count_by_room: false`: When a push notification is received, an unread count is also sent.
   This number can either be calculated as the number of unread messages  for the user, or the number of *rooms* the
   user has unread messages in. Defaults to true, meaning push clients will see the number of
   rooms with unread messages in them. Set to false to instead send the number
   of unread messages.
* `jitter_delay`: Delays push notifications by a random amount up to the given
  duration. Useful for mitigating timing attacks. Optional, defaults to no
  delay. _Added in Synapse 1.84.0._

Example configuration:
```yaml
push:
  enabled: true
  include_content: false
  group_unread_count_by_room: false
  jitter_delay: "10s"
```
---
## Rooms
Config options relating to rooms.

---
### `encryption_enabled_by_default_for_room_type`

Controls whether locally-created rooms should be end-to-end encrypted by
default.

Possible options are "all", "invite", and "off". They are defined as:

* "all": any locally-created room
* "invite": any room created with the `private_chat` or `trusted_private_chat`
   room creation presets
* "off": this option will take no effect

The default value is "off".

Note that this option will only affect rooms created after it is set. It
will also not affect rooms created by other servers.

Example configuration:
```yaml
encryption_enabled_by_default_for_room_type: invite
```
---
### `user_directory`

This setting defines options related to the user directory.

This option has the following sub-options:
* `enabled`:  Defines whether users can search the user directory. If false then
   empty responses are returned to all queries. Defaults to true.
* `search_all_users`: Defines whether to search all users visible to your homeserver at the time the search is performed.
   If set to true, will return all users known to the homeserver matching the search query.
   If false, search results will only contain users
    visible in public rooms and users sharing a room with the requester.
    Defaults to false.

    NB. If you set this to true, and the last time the user_directory search
    indexes were (re)built was before Synapse 1.44, you'll have to
    rebuild the indexes in order to search through all known users.

    These indexes are built the first time Synapse starts; admins can
    manually trigger a rebuild via the API following the instructions
    [for running background updates](../administration/admin_api/background_updates.md#run),
    set to true to return search results containing all known users, even if that
    user does not share a room with the requester.
* `prefer_local_users`: Defines whether to prefer local users in search query results.
   If set to true, local users are more likely to appear above remote users when searching the
   user directory. Defaults to false.
* `show_locked_users`: Defines whether to show locked users in search query results. Defaults to false.

Example configuration:
```yaml
user_directory:
    enabled: false
    search_all_users: true
    prefer_local_users: true
    show_locked_users: true
```
---
### `user_consent`

For detailed instructions on user consent configuration, see [here](../../consent_tracking.md).

Parts of this section are required if enabling the `consent` resource under
[`listeners`](#listeners), in particular `template_dir` and `version`.

* `template_dir`: gives the location of the templates for the HTML forms.
  This directory should contain one subdirectory per language (eg, `en`, `fr`),
  and each language directory should contain the policy document (named as
  <version>.html) and a success page (success.html).

* `version`: specifies the 'current' version of the policy document. It defines
   the version to be served by the consent resource if there is no 'v'
   parameter.

* `server_notice_content`: if enabled, will send a user a "Server Notice"
   asking them to consent to the privacy policy. The [`server_notices` section](#server_notices)
   must also be configured for this to work. Notices will *not* be sent to
   guest users unless `send_server_notice_to_guests` is set to true.

* `block_events_error`, if set, will block any attempts to send events
   until the user consents to the privacy policy. The value of the setting is
   used as the text of the error.

* `require_at_registration`, if enabled, will add a step to the registration
   process, similar to how captcha works. Users will be required to accept the
   policy before their account is created.

* `policy_name` is the display name of the policy users will see when registering
   for an account. Has no effect unless `require_at_registration` is enabled.
   Defaults to "Privacy Policy".

Example configuration:
```yaml
user_consent:
  template_dir: res/templates/privacy
  version: 1.0
  server_notice_content:
    msgtype: m.text
    body: >-
      To continue using this homeserver you must review and agree to the
      terms and conditions at %(consent_uri)s
  send_server_notice_to_guests: true
  block_events_error: >-
    To continue using this homeserver you must review and agree to the
    terms and conditions at %(consent_uri)s
  require_at_registration: false
  policy_name: Privacy Policy
```
---
### `stats`

Settings for local room and user statistics collection. See [here](../../room_and_user_statistics.md)
for more.

* `enabled`: Set to false to disable room and user statistics. Note that doing
   so may cause certain features (such as the room directory) not to work
   correctly. Defaults to true.

Example configuration:
```yaml
stats:
  enabled: false
```
---
### `server_notices`

Use this setting to enable a room which can be used to send notices
from the server to users. It is a special room which users cannot leave; notices
in the room come from a special "notices" user id.

If you use this setting, you *must* define the `system_mxid_localpart`
sub-setting, which defines the id of the user which will be used to send the
notices.

Sub-options for this setting include:
* `system_mxid_display_name`: set the display name of the "notices" user
* `system_mxid_avatar_url`: set the avatar for the "notices" user
* `room_name`: set the room name of the server notices room
* `room_avatar_url`: optional string. The room avatar to use for server notice rooms. If set to the empty string `""`, notice rooms will not be given an avatar. Defaults to the empty string. _Added in Synapse 1.99.0._
* `room_topic`: optional string. The topic to use for server notice rooms. If set to the empty string `""`, notice rooms will not be given a topic. Defaults to the empty string.  _Added in Synapse 1.99.0._
* `auto_join`: boolean. If true, the user will be automatically joined to the room instead of being invited.
  Defaults to false. _Added in Synapse 1.98.0._

Note that the name, topic and avatar of existing server notice rooms will only be updated when a new notice event is sent.

Example configuration:
```yaml
server_notices:
  system_mxid_localpart: notices
  system_mxid_display_name: "Server Notices"
  system_mxid_avatar_url: "mxc://example.com/oumMVlgDnLYFaPVkExemNVVZ"
  room_name: "Server Notices"
  room_avatar_url: "mxc://example.com/oumMVlgDnLYFaPVkExemNVVZ"
  room_topic: "Room used by your server admin to notice you of important information"
  auto_join: true
```
---
### `enable_room_list_search`

Set to false to disable searching the public room list. When disabled
blocks searching local and remote room lists for local and remote
users by always returning an empty list for all queries. Defaults to true.

Example configuration:
```yaml
enable_room_list_search: false
```
---
### `alias_creation_rules`

The `alias_creation_rules` option allows server admins to prevent unwanted
alias creation on this server.

This setting is an optional list of 0 or more rules. By default, no list is
provided, meaning that all alias creations are permitted.

Otherwise, requests to create aliases are matched against each rule in order.
The first rule that matches decides if the request is allowed or denied. If no
rule matches, the request is denied. In particular, this means that configuring
an empty list of rules will deny every alias creation request.

Each rule is a YAML object containing four fields, each of which is an optional string:

* `user_id`: a glob pattern that matches against the creator of the alias.
* `alias`: a glob pattern that matches against the alias being created.
* `room_id`: a glob pattern that matches against the room ID the alias is being pointed at.
* `action`: either `allow` or `deny`. What to do with the request if the rule matches. Defaults to `allow`.

Each of the glob patterns is optional, defaulting to `*` ("match anything").
Note that the patterns match against fully qualified IDs, e.g. against
`@alice:example.com`, `#room:example.com` and `!abcdefghijk:example.com` instead
of `alice`, `room` and `abcedgghijk`.

Example configuration:

```yaml
# No rule list specified. All alias creations are allowed.
# This is the default behaviour.
alias_creation_rules:
```

```yaml
# A list of one rule which allows everything.
# This has the same effect as the previous example.
alias_creation_rules:
  - "action": "allow"
```

```yaml
# An empty list of rules. All alias creations are denied.
alias_creation_rules: []
```

```yaml
# A list of one rule which denies everything.
# This has the same effect as the previous example.
alias_creation_rules:
  - "action": "deny"
```

```yaml
# Prevent a specific user from creating aliases.
# Allow other users to create any alias
alias_creation_rules:
  - user_id: "@bad_user:example.com"
    action: deny

  - action: allow
```

```yaml
# Prevent aliases being created which point to a specific room.
alias_creation_rules:
  - room_id: "!forbiddenRoom:example.com"
    action: deny

  - action: allow
```

---
### `room_list_publication_rules`

The `room_list_publication_rules` option allows server admins to prevent
unwanted entries from being published in the public room list.

The format of this option is the same as that for
[`alias_creation_rules`](#alias_creation_rules): an optional list of 0 or more
rules. By default, no list is provided, meaning that all rooms may be
published to the room list.

Otherwise, requests to publish a room are matched against each rule in order.
The first rule that matches decides if the request is allowed or denied. If no
rule matches, the request is denied. In particular, this means that configuring
an empty list of rules will deny every alias creation request.

Requests to create a public (public as in published to the room directory) room which violates
the configured rules will result in the room being created but not published to the room directory.

Each rule is a YAML object containing four fields, each of which is an optional string:

* `user_id`: a glob pattern that matches against the user publishing the room.
* `alias`: a glob pattern that matches against one of published room's aliases.
  - If the room has no aliases, the alias match fails unless `alias` is unspecified or `*`.
  - If the room has exactly one alias, the alias match succeeds if the `alias` pattern matches that alias.
  - If the room has two or more aliases, the alias match succeeds if the pattern matches at least one of the aliases.
* `room_id`: a glob pattern that matches against the room ID of the room being published.
* `action`: either `allow` or `deny`. What to do with the request if the rule matches. Defaults to `allow`.

Each of the glob patterns is optional, defaulting to `*` ("match anything").
Note that the patterns match against fully qualified IDs, e.g. against
`@alice:example.com`, `#room:example.com` and `!abcdefghijk:example.com` instead
of `alice`, `room` and `abcedgghijk`.


Example configuration:

```yaml
# No rule list specified. Anyone may publish any room to the public list.
# This is the default behaviour.
room_list_publication_rules:
```

```yaml
# A list of one rule which allows everything.
# This has the same effect as the previous example.
room_list_publication_rules:
  - "action": "allow"
```

```yaml
# An empty list of rules. No-one may publish to the room list.
room_list_publication_rules: []
```

```yaml
# A list of one rule which denies everything.
# This has the same effect as the previous example.
room_list_publication_rules:
  - "action": "deny"
```

```yaml
# Prevent a specific user from publishing rooms.
# Allow other users to publish anything.
room_list_publication_rules:
  - user_id: "@bad_user:example.com"
    action: deny

  - action: allow
```

```yaml
# Prevent publication of a specific room.
room_list_publication_rules:
  - room_id: "!forbiddenRoom:example.com"
    action: deny

  - action: allow
```

```yaml
# Prevent publication of rooms with at least one alias containing the word "potato".
room_list_publication_rules:
  - alias: "#*potato*:example.com"
    action: deny

  - action: allow
```

---
### `default_power_level_content_override`

The `default_power_level_content_override` option controls the default power
levels for rooms.

Useful if you know that your users need special permissions in rooms
that they create (e.g. to send particular types of state events without
needing an elevated power level).  This takes the same shape as the
`power_level_content_override` parameter in the /createRoom API, but
is applied before that parameter.

Note that each key provided inside a preset (for example `events` in the example
below) will overwrite all existing defaults inside that key. So in the example
below, newly-created private_chat rooms will have no rules for any event types
except `com.example.foo`.

Example configuration:
```yaml
default_power_level_content_override:
   private_chat: { "events": { "com.example.foo" : 0 } }
   trusted_private_chat: null
   public_chat: null
```

The default power levels for each preset are:
```yaml
"m.room.name": 50
"m.room.power_levels": 100
"m.room.history_visibility": 100
"m.room.canonical_alias": 50
"m.room.avatar": 50
"m.room.tombstone": 100
"m.room.server_acl": 100
"m.room.encryption": 100
```

So a complete example where the default power-levels for a preset are maintained
but the power level for a new key is set is:
```yaml
default_power_level_content_override:
   private_chat:
    events:
      "com.example.foo": 0
      "m.room.name": 50
      "m.room.power_levels": 100
      "m.room.history_visibility": 100
      "m.room.canonical_alias": 50
