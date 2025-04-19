url_preview_ip_range_blacklist:
  - '127.0.0.0/8'
  - '10.0.0.0/8'
  - '172.16.0.0/12'
  - '192.168.0.0/16'
  - '100.64.0.0/10'
  - '192.0.0.0/24'
  - '169.254.0.0/16'
  - '192.88.99.0/24'
  - '198.18.0.0/15'
  - '192.0.2.0/24'
  - '198.51.100.0/24'
  - '203.0.113.0/24'
  - '224.0.0.0/4'
  - '::1/128'
  - 'fe80::/10'
  - 'fc00::/7'
  - '2001:db8::/32'
  - 'ff00::/8'
  - 'fec0::/10'
```
---
### `url_preview_ip_range_whitelist`

This option sets a list of IP address CIDR ranges that the URL preview spider is allowed
to access even if they are specified in `url_preview_ip_range_blacklist`.
This is useful for specifying exceptions to wide-ranging blacklisted
target IP ranges - e.g. for enabling URL previews for a specific private
website only visible in your network. Defaults to none.

Example configuration:
```yaml
url_preview_ip_range_whitelist:
   - '192.168.1.1'
```
---
### `url_preview_url_blacklist`

Optional list of URL matches that the URL preview spider is denied from
accessing.  This is a usability feature, not a security one. You should use
`url_preview_ip_range_blacklist` in preference to this, otherwise someone could
define a public DNS entry that points to a private IP address and circumvent
the blacklist. Applications that perform redirects or serve different content
when detecting that Synapse is accessing them can also bypass the blacklist.
This is more useful if you know there is an entire shape of URL that you know
that you do not want Synapse to preview.

Each list entry is a dictionary of url component attributes as returned
by urlparse.urlsplit as applied to the absolute form of the URL.  See
[here](https://docs.python.org/2/library/urlparse.html#urlparse.urlsplit) for more
information. Some examples are:

* `username`
* `netloc`
* `scheme`
* `path`

The values of the dictionary are treated as a filename match pattern
applied to that component of URLs, unless they start with a ^ in which
case they are treated as a regular expression match.  If all the
specified component matches for a given list item succeed, the URL is
blacklisted.

Example configuration:
```yaml
url_preview_url_blacklist:
  # blacklist any URL with a username in its URI
  - username: '*'

  # blacklist all *.google.com URLs
  - netloc: 'google.com'
  - netloc: '*.google.com'

  # blacklist all plain HTTP URLs
  - scheme: 'http'

  # blacklist http(s)://www.acme.com/foo
  - netloc: 'www.acme.com'
    path: '/foo'

  # blacklist any URL with a literal IPv4 address
  - netloc: '^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$'
```
---
### `max_spider_size`

The largest allowed URL preview spidering size in bytes. Defaults to 10M.

Example configuration:
```yaml
max_spider_size: 8M
```
---
### `url_preview_accept_language`

A list of values for the Accept-Language HTTP header used when
downloading webpages during URL preview generation. This allows
Synapse to specify the preferred languages that URL previews should
be in when communicating with remote servers.

Each value is a IETF language tag; a 2-3 letter identifier for a
language, optionally followed by subtags separated by '-', specifying
a country or region variant.

Multiple values can be provided, and a weight can be added to each by
using quality value syntax (;q=). '*' translates to any language.

Defaults to "en".

Example configuration:
```yaml
 url_preview_accept_language:
   - 'en-UK'
   - 'en-US;q=0.9'
   - 'fr;q=0.8'
   - '*;q=0.7'
```
---
### `oembed`

oEmbed allows for easier embedding content from a website. It can be
used for generating URLs previews of services which support it. A default list of oEmbed providers
is included with Synapse. Set `disable_default_providers` to true to disable using
these default oEmbed URLs. Use `additional_providers` to specify additional files with oEmbed configuration (each
should be in the form of providers.json). By default this list is empty.

Example configuration:
```yaml
oembed:
  disable_default_providers: true
  additional_providers:
    - oembed/my_providers.json
```
---
## Captcha

See [here](../../CAPTCHA_SETUP.md) for full details on setting up captcha.

---
### `recaptcha_public_key`

This homeserver's ReCAPTCHA public key. Must be specified if
[`enable_registration_captcha`](#enable_registration_captcha) is enabled.

Example configuration:
```yaml
recaptcha_public_key: "YOUR_PUBLIC_KEY"
```
---
### `recaptcha_private_key`

This homeserver's ReCAPTCHA private key. Must be specified if
[`enable_registration_captcha`](#enable_registration_captcha) is
enabled.

Example configuration:
```yaml
recaptcha_private_key: "YOUR_PRIVATE_KEY"
```
---
### `enable_registration_captcha`

Set to `true` to require users to complete a CAPTCHA test when registering an account.
Requires a valid ReCaptcha public/private key.
Defaults to `false`.

Note that [`enable_registration`](#enable_registration) must also be set to allow account registration.

Example configuration:
```yaml
enable_registration_captcha: true
```
---
### `recaptcha_siteverify_api`

The API endpoint to use for verifying `m.login.recaptcha` responses.
Defaults to `https://www.recaptcha.net/recaptcha/api/siteverify`.

Example configuration:
```yaml
recaptcha_siteverify_api: "https://my.recaptcha.site"
```
---
## TURN
Options related to adding a TURN server to Synapse.

---
### `turn_uris`

The public URIs of the TURN server to give to clients.

Example configuration:
```yaml
turn_uris: [turn:example.org]
```
---
### `turn_shared_secret`

The shared secret used to compute passwords for the TURN server.

Example configuration:
```yaml
turn_shared_secret: "YOUR_SHARED_SECRET"
```
---
### `turn_shared_secret_path`

An alternative to [`turn_shared_secret`](#turn_shared_secret):
allows the shared secret to be specified in an external file.

The file should be a plain text file, containing only the shared secret.
Synapse reads the shared secret from the given file once at startup.

Example configuration:
```yaml
turn_shared_secret_path: /path/to/secrets/file
```

_Added in Synapse 1.116.0._

---
### `turn_username` and `turn_password`

The Username and password if the TURN server needs them and does not use a token.

Example configuration:
```yaml
turn_username: "TURNSERVER_USERNAME"
turn_password: "TURNSERVER_PASSWORD"
```
---
### `turn_user_lifetime`

How long generated TURN credentials last. Defaults to 1h.

Example configuration:
```yaml
turn_user_lifetime: 2h
```
---
### `turn_allow_guests`

Whether guests should be allowed to use the TURN server. This defaults to true, otherwise
VoIP will be unreliable for guests. However, it does introduce a slight security risk as
it allows users to connect to arbitrary endpoints without having first signed up for a valid account (e.g. by passing a CAPTCHA).

Example configuration:
```yaml
turn_allow_guests: false
```
---
## Registration ##

Registration can be rate-limited using the parameters in the [Ratelimiting](#ratelimiting) section of this manual.

---
### `enable_registration`

Enable registration for new users. Defaults to `false`.

It is highly recommended that if you enable registration, you set one or more
or the following options, to avoid abuse of your server by "bots":

 * [`enable_registration_captcha`](#enable_registration_captcha)
 * [`registrations_require_3pid`](#registrations_require_3pid)
 * [`registration_requires_token`](#registration_requires_token)

(In order to enable registration without any verification, you must also set
[`enable_registration_without_verification`](#enable_registration_without_verification).)

Note that even if this setting is disabled, new accounts can still be created
via the admin API if
[`registration_shared_secret`](#registration_shared_secret) is set.

Example configuration:
```yaml
enable_registration: true
```
---
### `enable_registration_without_verification`

Enable registration without email or captcha verification. Note: this option is *not* recommended,
as registration without verification is a known vector for spam and abuse. Defaults to `false`. Has no effect
unless [`enable_registration`](#enable_registration) is also enabled.

Example configuration:
```yaml
enable_registration_without_verification: true
```
---
### `registrations_require_3pid`

If this is set, users must provide all of the specified types of [3PID](https://spec.matrix.org/latest/appendices/#3pid-types) when registering an account.

Note that [`enable_registration`](#enable_registration) must also be set to allow account registration.

Example configuration:
```yaml
registrations_require_3pid:
  - email
  - msisdn
```
---
### `disable_msisdn_registration`

Explicitly disable asking for MSISDNs from the registration
flow (overrides `registrations_require_3pid` if MSISDNs are set as required).

Example configuration:
```yaml
disable_msisdn_registration: true
```
---
### `allowed_local_3pids`

Mandate that users are only allowed to associate certain formats of
3PIDs with accounts on this server, as specified by the `medium` and `pattern` sub-options.
`pattern` is a [Perl-like regular expression](https://docs.python.org/3/library/re.html#module-re).

More information about 3PIDs, allowed `medium` types and their `address` syntax can be found [in the Matrix spec](https://spec.matrix.org/latest/appendices/#3pid-types).

Example configuration:
```yaml
allowed_local_3pids:
  - medium: email
    pattern: '^[^@]+@matrix\.org$'
  - medium: email
    pattern: '^[^@]+@vector\.im$'
  - medium: msisdn
    pattern: '^44\d{10}$'
```
---
### `enable_3pid_lookup`

Enable 3PIDs lookup requests to identity servers from this server. Defaults to true.

Example configuration:
```yaml
enable_3pid_lookup: false
```
---
### `registration_requires_token`

Require users to submit a token during registration.
Tokens can be managed using the admin [API](../administration/admin_api/registration_tokens.md).
Disabling this option will not delete any tokens previously generated.
Defaults to `false`. Set to `true` to enable.


Note that [`enable_registration`](#enable_registration) must also be set to allow account registration.

Example configuration:
```yaml
registration_requires_token: true
```
---
### `registration_shared_secret`

If set, allows registration of standard or admin accounts by anyone who has the
shared secret, even if [`enable_registration`](#enable_registration) is not
set.

This is primarily intended for use with the `register_new_matrix_user` script
(see [Registering a user](../../setup/installation.md#registering-a-user));
however, the interface is [documented](../../admin_api/register_api.html).

See also [`registration_shared_secret_path`](#registration_shared_secret_path).

Example configuration:
```yaml
registration_shared_secret: <PRIVATE STRING>
```

---
### `registration_shared_secret_path`

An alternative to [`registration_shared_secret`](#registration_shared_secret):
allows the shared secret to be specified in an external file.

The file should be a plain text file, containing only the shared secret.

If this file does not exist, Synapse will create a new shared
secret on startup and store it in this file.

Example configuration:
```yaml
registration_shared_secret_path: /path/to/secrets/file
```

_Added in Synapse 1.67.0._

---
### `bcrypt_rounds`

Set the number of bcrypt rounds used to generate password hash.
Larger numbers increase the work factor needed to generate the hash.
The default number is 12 (which equates to 2^12 rounds).
N.B. that increasing this will exponentially increase the time required
to register or login - e.g. 24 => 2^24 rounds which will take >20 mins.
Example configuration:
```yaml
bcrypt_rounds: 14
```
---
### `allow_guest_access`

Allows users to register as guests without a password/email/etc, and
participate in rooms hosted on this server which have been made
accessible to anonymous users. Defaults to false.

Example configuration:
```yaml
allow_guest_access: true
```
---
### `default_identity_server`

The identity server which we suggest that clients should use when users log
in on this server.

(By default, no suggestion is made, so it is left up to the client.
This setting is ignored unless `public_baseurl` is also explicitly set.)

Example configuration:
```yaml
default_identity_server: https://matrix.org
```
---
### `account_threepid_delegates`

Delegate verification of phone numbers to an identity server.

When a user wishes to add a phone number to their account, we need to verify that they
actually own that phone number, which requires sending them a text message (SMS).
Currently Synapse does not support sending those texts itself and instead delegates the
task to an identity server. The base URI for the identity server to be used is
specified by the `account_threepid_delegates.msisdn` option.

If this is left unspecified, Synapse will not allow users to add phone numbers to
their account.

(Servers handling the these requests must answer the `/requestToken` endpoints defined
by the Matrix Identity Service API
[specification](https://matrix.org/docs/spec/identity_service/latest).)

*Deprecated in Synapse 1.64.0*: The `email` option is deprecated.

*Removed in Synapse 1.66.0*: The `email` option has been removed.
If present, Synapse will report a configuration error on startup.

Example configuration:
```yaml
account_threepid_delegates:
    msisdn: http://localhost:8090  # Delegate SMS sending to this local process
```
---
### `enable_set_displayname`

Whether users are allowed to change their displayname after it has
been initially set. Useful when provisioning users based on the
contents of a third-party directory.

Does not apply to server administrators. Defaults to true.

Example configuration:
```yaml
enable_set_displayname: false
```
---
### `enable_set_avatar_url`

Whether users are allowed to change their avatar after it has been
initially set. Useful when provisioning users based on the contents
of a third-party directory.

Does not apply to server administrators. Defaults to true.

Example configuration:
```yaml
enable_set_avatar_url: false
```
---
### `enable_3pid_changes`

Whether users can change the third-party IDs associated with their accounts
(email address and msisdn).

Defaults to true.

Example configuration:
```yaml
enable_3pid_changes: false
```
---
### `auto_join_rooms`

Users who register on this homeserver will automatically be joined
to the rooms listed under this option.

By default, any room aliases included in this list will be created
as a publicly joinable room when the first user registers for the
homeserver. If the room already exists, make certain it is a publicly joinable
room, i.e. the join rule of the room must be set to 'public'. You can find more options
relating to auto-joining rooms below.

As Spaces are just rooms under the hood, Space aliases may also be
used.

Example configuration:
```yaml
auto_join_rooms:
  - "#exampleroom:example.com"
  - "#anotherexampleroom:example.com"
```
---
### `autocreate_auto_join_rooms`

Where `auto_join_rooms` are specified, setting this flag ensures that
the rooms exist by creating them when the first user on the
homeserver registers. This option will not create Spaces.

By default the auto-created rooms are publicly joinable from any federated
server. Use the `autocreate_auto_join_rooms_federated` and
`autocreate_auto_join_room_preset` settings to customise this behaviour.

Setting to false means that if the rooms are not manually created,
users cannot be auto-joined since they do not exist.

Defaults to true.

Example configuration:
```yaml
autocreate_auto_join_rooms: false
```
---
### `autocreate_auto_join_rooms_federated`

Whether the rooms listed in `auto_join_rooms` that are auto-created are available
via federation. Only has an effect if `autocreate_auto_join_rooms` is true.

Note that whether a room is federated cannot be modified after
creation.

Defaults to true: the room will be joinable from other servers.
Set to false to prevent users from other homeservers from
joining these rooms.

Example configuration:
```yaml
autocreate_auto_join_rooms_federated: false
```
---
### `autocreate_auto_join_room_preset`

The room preset to use when auto-creating one of `auto_join_rooms`. Only has an
effect if `autocreate_auto_join_rooms` is true.

Possible values for this option are:
* "public_chat": the room is joinable by anyone, including
  federated servers if `autocreate_auto_join_rooms_federated` is true (the default).
* "private_chat": an invitation is required to join these rooms.
* "trusted_private_chat": an invitation is required to join this room and the invitee is
  assigned a power level of 100 upon joining the room.

Each preset will set up a room in the same manner as if it were provided as the `preset` parameter when
calling the
[`POST /_matrix/client/v3/createRoom`](https://spec.matrix.org/latest/client-server-api/#post_matrixclientv3createroom)
Client-Server API endpoint.

If a value of "private_chat" or "trusted_private_chat" is used then
`auto_join_mxid_localpart` must also be configured.

Defaults to "public_chat".

Example configuration:
```yaml
autocreate_auto_join_room_preset: private_chat
```
---
### `auto_join_mxid_localpart`

The local part of the user id which is used to create `auto_join_rooms` if
`autocreate_auto_join_rooms` is true. If this is not provided then the
initial user account that registers will be used to create the rooms.

The user id is also used to invite new users to any auto-join rooms which
are set to invite-only.

It *must* be configured if `autocreate_auto_join_room_preset` is set to
"private_chat" or "trusted_private_chat".

Note that this must be specified in order for new users to be correctly
invited to any auto-join rooms which have been set to invite-only (either
at the time of creation or subsequently).

Note that, if the room already exists, this user must be joined and
have the appropriate permissions to invite new members.

Example configuration:
```yaml
auto_join_mxid_localpart: system
```
---
### `auto_join_rooms_for_guests`

When `auto_join_rooms` is specified, setting this flag to false prevents
guest accounts from being automatically joined to the rooms.

Defaults to true.

Example configuration:
```yaml
auto_join_rooms_for_guests: false
```
---
### `inhibit_user_in_use_error`

Whether to inhibit errors raised when registering a new account if the user ID
already exists. If turned on, requests to `/register/available` will always
show a user ID as available, and Synapse won't raise an error when starting
a registration with a user ID that already exists. However, Synapse will still
raise an error if the registration completes and the username conflicts.

Defaults to false.

Example configuration:
```yaml
inhibit_user_in_use_error: true
