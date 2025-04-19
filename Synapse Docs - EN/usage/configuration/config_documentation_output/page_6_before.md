inhibit_user_in_use_error: true
```
---
## User session management
---
### `session_lifetime`

Time that a user's session remains valid for, after they log in.

Note that this is not currently compatible with guest logins.

Note also that this is calculated at login time: changes are not applied retrospectively to users who have already
logged in.

By default, this is infinite.

Example configuration:
```yaml
session_lifetime: 24h
```
---
### `refreshable_access_token_lifetime`

Time that an access token remains valid for, if the session is using refresh tokens.

For more information about refresh tokens, please see the [manual](Synapse%20Docs%20-%20EN/usage/configuration/user_authentication/refresh_tokens.md).

Note that this only applies to clients which advertise support for refresh tokens.

Note also that this is calculated at login time and refresh time: changes are not applied to
existing sessions until they are refreshed.

By default, this is 5 minutes.

Example configuration:
```yaml
refreshable_access_token_lifetime: 10m
```
---
### `refresh_token_lifetime`

Time that a refresh token remains valid for (provided that it is not
exchanged for another one first).
This option can be used to automatically log-out inactive sessions.
Please see the manual for more information.

Note also that this is calculated at login time and refresh time:
changes are not applied to existing sessions until they are refreshed.

By default, this is infinite.

Example configuration:
```yaml
refresh_token_lifetime: 24h
```
---
### `nonrefreshable_access_token_lifetime`

Time that an access token remains valid for, if the session is NOT
using refresh tokens.

Please note that not all clients support refresh tokens, so setting
this to a short value may be inconvenient for some users who will
then be logged out frequently.

Note also that this is calculated at login time: changes are not applied
retrospectively to existing sessions for users that have already logged in.

By default, this is infinite.

Example configuration:
```yaml
nonrefreshable_access_token_lifetime: 24h
```
---
### `ui_auth`

The amount of time to allow a user-interactive authentication session to be active.

This defaults to 0, meaning the user is queried for their credentials
before every action, but this can be overridden to allow a single
validation to be re-used.  This weakens the protections afforded by
the user-interactive authentication process, by allowing for multiple
(and potentially different) operations to use the same validation session.

This is ignored for potentially "dangerous" operations (including
deactivating an account, modifying an account password, adding a 3PID,
and minting additional login tokens).

Use the `session_timeout` sub-option here to change the time allowed for credential validation.

Example configuration:
```yaml
ui_auth:
    session_timeout: "15s"
```
---
### `login_via_existing_session`

Matrix supports the ability of an existing session to mint a login token for
another client.

Synapse disables this by default as it has security ramifications -- a malicious
client could use the mechanism to spawn more than one session.

The duration of time the generated token is valid for can be configured with the
`token_timeout` sub-option.

User-interactive authentication is required when this is enabled unless the
`require_ui_auth` sub-option is set to `False`.

Example configuration:
```yaml
login_via_existing_session:
    enabled: true
    require_ui_auth: false
    token_timeout: "5m"
```
---
## Metrics
Config options related to metrics.

---
### `enable_metrics`

Set to true to enable collection and rendering of performance metrics.
Defaults to false.

Example configuration:
```yaml
enable_metrics: true
```
---
### `sentry`

Use this option to enable sentry integration. Provide the DSN assigned to you by sentry
with the `dsn` setting.

 An optional `environment` field can be used to specify an environment. This allows
 for log maintenance based on different environments, ensuring better organization
 and analysis..

NOTE: While attempts are made to ensure that the logs don't contain
any sensitive information, this cannot be guaranteed. By enabling
this option the sentry server may therefore receive sensitive
information, and it in turn may then disseminate sensitive information
through insecure notification channels if so configured.

Example configuration:
```yaml
sentry:
    environment: "production"
    dsn: "..."
```
---
### `metrics_flags`

Flags to enable Prometheus metrics which are not suitable to be
enabled by default, either for performance reasons or limited use.
Currently the only option is `known_servers`, which publishes
`synapse_federation_known_servers`, a gauge of the number of
servers this homeserver knows about, including itself. May cause
performance problems on large homeservers.

Example configuration:
```yaml
metrics_flags:
    known_servers: true
```
---
### `report_stats`

Whether or not to report homeserver usage statistics. This is originally
set when generating the config. Set this option to true or false to change the current
behavior. See
[Reporting Homeserver Usage Statistics](../administration/monitoring/reporting_homeserver_usage_statistics.md)
for information on what data is reported.

Statistics will be reported 5 minutes after Synapse starts, and then every 3 hours
after that.

Example configuration:
```yaml
report_stats: true
```
---
### `report_stats_endpoint`

The endpoint to report homeserver usage statistics to.
Defaults to https://matrix.org/report-usage-stats/push

Example configuration:
```yaml
report_stats_endpoint: https://example.com/report-usage-stats/push
```
---
## API Configuration
Config settings related to the client/server API

---
### `room_prejoin_state`

This setting controls the state that is shared with users upon receiving an
invite to a room, or in reply to a knock on a room. By default, the following
state events are shared with users:

- `m.room.join_rules`
- `m.room.canonical_alias`
- `m.room.avatar`
- `m.room.encryption`
- `m.room.name`
- `m.room.create`
- `m.room.topic`

To change the default behavior, use the following sub-options:
* `disable_default_event_types`: boolean. Set to `true` to disable the above
  defaults. If this is enabled, only the event types listed in
  `additional_event_types` are shared. Defaults to `false`.
* `additional_event_types`: A list of additional state events to include in the
  events to be shared. By default, this list is empty (so only the default event
  types are shared).

  Each entry in this list should be either a single string or a list of two
  strings.
  * A standalone string `t` represents all events with type `t` (i.e.
    with no restrictions on state keys).
  * A pair of strings `[t, s]` represents a single event with type `t` and
    state key `s`. The same type can appear in two entries with different state
    keys: in this situation, both state keys are included in prejoin state.

Example configuration:
```yaml
room_prejoin_state:
   disable_default_event_types: false
   additional_event_types:
     # Share all events of type `org.example.custom.event.typeA`
     - org.example.custom.event.typeA
     # Share only events of type `org.example.custom.event.typeB` whose
     # state_key is "foo"
     - ["org.example.custom.event.typeB", "foo"]
     # Share only events of type `org.example.custom.event.typeC` whose
     # state_key is "bar" or "baz"
     - ["org.example.custom.event.typeC", "bar"]
     - ["org.example.custom.event.typeC", "baz"]
```

*Changed in Synapse 1.74:* admins can filter the events in prejoin state based
on their state key.

---
### `track_puppeted_user_ips`

We record the IP address of clients used to access the API for various
reasons, including displaying it to the user in the "Where you're signed in"
dialog.

By default, when puppeting another user via the admin API, the client IP
address is recorded against the user who created the access token (ie, the
admin user), and *not* the puppeted user.

Set this option to true to also record the IP address against the puppeted
user. (This also means that the puppeted user will count as an "active" user
for the purpose of monthly active user tracking - see `limit_usage_by_mau` etc
above.)

Example configuration:
```yaml
track_puppeted_user_ips: true
```
---
### `app_service_config_files`

A list of application service config files to use.

Example configuration:
```yaml
app_service_config_files:
  - app_service_1.yaml
  - app_service_2.yaml
```
---
### `track_appservice_user_ips`

Defaults to false. Set to true to enable tracking of application service IP addresses.
Implicitly enables MAU tracking for application service users.

Example configuration:
```yaml
track_appservice_user_ips: true
```
---
### `use_appservice_legacy_authorization`

Whether to send the application service access tokens via the `access_token` query parameter
per older versions of the Matrix specification. Defaults to false. Set to true to enable sending
access tokens via a query parameter.

**Enabling this option is considered insecure and is not recommended. **

Example configuration:
```yaml
use_appservice_legacy_authorization: true
```

---
### `macaroon_secret_key`

A secret which is used to sign
- access token for guest users,
- short-term login token used during SSO logins (OIDC or SAML2) and
- token used for unsubscribing from email notifications.

If none is specified, the `registration_shared_secret` is used, if one is given;
otherwise, a secret key is derived from the signing key.

Example configuration:
```yaml
macaroon_secret_key: <PRIVATE STRING>
```
---
### `form_secret`

A secret which is used to calculate HMACs for form values, to stop
falsification of values. Must be specified for the User Consent
forms to work.

Example configuration:
```yaml
form_secret: <PRIVATE STRING>
```
---
## Signing Keys
Config options relating to signing keys

---
### `signing_key_path`

Path to the signing key to sign events and federation requests with.

*New in Synapse 1.67*: If this file does not exist, Synapse will create a new signing
key on startup and store it in this file.

Example configuration:
```yaml
signing_key_path: "CONFDIR/SERVERNAME.signing.key"
```
---
### `old_signing_keys`

The keys that the server used to sign messages with but won't use
to sign new messages. For each key, `key` should be the base64-encoded public key, and
`expired_ts`should be the time (in milliseconds since the unix epoch) that
it was last used.

It is possible to build an entry from an old `signing.key` file using the
`export_signing_key` script which is provided with synapse.

Example configuration:
```yaml
old_signing_keys:
  "ed25519:id": { key: "base64string", expired_ts: 123456789123 }
```
---
### `key_refresh_interval`

How long key response published by this server is valid for.
Used to set the `valid_until_ts` in `/key/v2` APIs.
Determines how quickly servers will query to check which keys
are still valid. Defaults to 1d.

Example configuration:
```yaml
key_refresh_interval: 2d
```
---
### `trusted_key_servers`

The trusted servers to download signing keys from.

When we need to fetch a signing key, each server is tried in parallel.

Normally, the connection to the key server is validated via TLS certificates.
Additional security can be provided by configuring a `verify key`, which
will make synapse check that the response is signed by that key.

This setting supersedes an older setting named `perspectives`. The old format
is still supported for backwards-compatibility, but it is deprecated.

`trusted_key_servers` defaults to matrix.org, but using it will generate a
warning on start-up. To suppress this warning, set
`suppress_key_server_warning` to true.

If the use of a trusted key server has to be deactivated, e.g. in a private
federation or for privacy reasons, this can be realised by setting
an empty array (`trusted_key_servers: []`). Then Synapse will request the keys
directly from the server that owns the keys. If Synapse does not get keys directly
from the server, the events of this server will be rejected.

Options for each entry in the list include:
* `server_name`: the name of the server. Required.
* `verify_keys`: an optional map from key id to base64-encoded public key.
   If specified, we will check that the response is signed by at least
   one of the given keys.
* `accept_keys_insecurely`: a boolean. Normally, if `verify_keys` is unset,
   and `federation_verify_certificates` is not `true`, synapse will refuse
   to start, because this would allow anyone who can spoof DNS responses
   to masquerade as the trusted key server. If you know what you are doing
   and are sure that your network environment provides a secure connection
   to the key server, you can set this to `true` to override this behaviour.

Example configuration #1:
```yaml
trusted_key_servers:
  - server_name: "my_trusted_server.example.com"
    verify_keys:
      "ed25519:auto": "abcdefghijklmnopqrstuvwxyzabcdefghijklmopqr"
  - server_name: "my_other_trusted_server.example.com"
```
Example configuration #2:
```yaml
trusted_key_servers:
  - server_name: "matrix.org"
```
---
### `suppress_key_server_warning`

Set the following to true to disable the warning that is emitted when the
`trusted_key_servers` include 'matrix.org'. See above.

Example configuration:
```yaml
suppress_key_server_warning: true
```
---
### `key_server_signing_keys_path`

The signing keys to use when acting as a trusted key server. If not specified
defaults to the server signing key.

Can contain multiple keys, one per line.

Example configuration:
```yaml
key_server_signing_keys_path: "key_server_signing_keys.key"
```
---
## Single sign-on integration

The following settings can be used to make Synapse use a single sign-on
provider for authentication, instead of its internal password database.

You will probably also want to set the following options to `false` to
disable the regular login/registration flows:
   * [`enable_registration`](#enable_registration)
   * [`password_config.enabled`](#password_config)

---
### `saml2_config`

Enable SAML2 for registration and login. Uses pysaml2. To learn more about pysaml and
to find a full list options for configuring pysaml, read the docs [here](https://pysaml2.readthedocs.io/en/latest/).

At least one of `sp_config` or `config_path` must be set in this section to
enable SAML login. You can either put your entire pysaml config inline using the `sp_config`
option, or you can specify a path to a psyaml config file with the sub-option `config_path`.
This setting has the following sub-options:

* `idp_name`: A user-facing name for this identity provider, which is used to
   offer the user a choice of login mechanisms.
* `idp_icon`: An optional icon for this identity provider, which is presented
   by clients and Synapse's own IdP picker page. If given, must be an
   MXC URI of the format `mxc://<server-name>/<media-id>`. (An easy way to
   obtain such an MXC URI is to upload an image to an (unencrypted) room
   and then copy the "url" from the source of the event.)
* `idp_brand`: An optional brand for this identity provider, allowing clients
   to style the login flow according to the identity provider in question.
   See the [spec](https://spec.matrix.org/latest/) for possible options here.
* `sp_config`: the configuration for the pysaml2 Service Provider. See pysaml2 docs for format of config.
   Default values will be used for the `entityid` and `service` settings,
   so it is not normally necessary to specify them unless you need to
   override them. Here are a few useful sub-options for configuring pysaml:
   * `metadata`: Point this to the IdP's metadata. You must provide either a local
      file via the `local` attribute or (preferably) a URL via the
      `remote` attribute.
   * `accepted_time_diff: 3`: Allowed clock difference in seconds between the homeserver and IdP.
      Defaults to 0.
   * `service`: By default, the user has to go to our login page first. If you'd like
     to allow IdP-initiated login, set `allow_unsolicited` to true under `sp` in the `service`
     section.
* `config_path`: specify a separate pysaml2 configuration file thusly:
  `config_path: "CONFDIR/sp_conf.py"`
* `saml_session_lifetime`: The lifetime of a SAML session. This defines how long a user has to
   complete the authentication process, if `allow_unsolicited` is unset. The default is 15 minutes.
* `user_mapping_provider`: Using this option, an external module can be provided as a
   custom solution to mapping attributes returned from a saml provider onto a matrix user. The
   `user_mapping_provider` has the following attributes:
  * `module`: The custom module's class.
  * `config`: Custom configuration values for the module. Use the values provided in the
     example if you are using the built-in user_mapping_provider, or provide your own
     config values for a custom class if you are using one. This section will be passed as a Python
     dictionary to the module's `parse_config` method. The built-in provider takes the following two
     options:
      * `mxid_source_attribute`: The SAML attribute (after mapping via the attribute maps) to use
          to derive the Matrix ID from. It is 'uid' by default. Note: This used to be configured by the
          `saml2_config.mxid_source_attribute option`. If that is still defined, its value will be used instead.
      * `mxid_mapping`: The mapping system to use for mapping the saml attribute onto a
         matrix ID. Options include: `hexencode` (which maps unpermitted characters to '=xx')
         and `dotreplace` (which replaces unpermitted characters with '.').
         The default is `hexencode`. Note: This used to be configured by the
         `saml2_config.mxid_mapping option`. If that is still defined, its value will be used instead.
* `grandfathered_mxid_source_attribute`: In previous versions of synapse, the mapping from SAML attribute to
   MXID was always calculated dynamically rather than stored in a table. For backwards- compatibility, we will look for `user_ids`
   matching such a pattern before creating a new account. This setting controls the SAML attribute which will be used for this
   backwards-compatibility lookup. Typically it should be 'uid', but if the attribute maps are changed, it may be necessary to change it.
   The default is 'uid'.
* `attribute_requirements`: It is possible to configure Synapse to only allow logins if SAML attributes
    match particular values. The requirements can be listed under
   `attribute_requirements` as shown in the example. All of the listed attributes must
    match for the login to be permitted.
* `idp_entityid`: If the metadata XML contains multiple IdP entities then the `idp_entityid`
   option must be set to the entity to redirect users to.
   Most deployments only have a single IdP entity and so should omit this option.
