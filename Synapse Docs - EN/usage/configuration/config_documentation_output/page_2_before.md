        module: my_module.CustomRequestHandler
        config: {}

  # Turn on the twisted ssh manhole service on localhost on the given
  # port.
  - port: 9000
    bind_addresses: ['::1', '127.0.0.1']
    type: manhole
```
Example configuration #3:
```yaml
listeners:
  # Unix socket listener: Ideal for Synapse deployments behind a reverse proxy, offering
  # lightweight interprocess communication without TCP/IP overhead, avoid port
  # conflicts, and providing enhanced security through system file permissions.
  #
  # Note that x_forwarded will default to true, when using a UNIX socket. Please see
  # https://element-hq.github.io/synapse/latest/reverse_proxy.html.
  #
  - path: /run/synapse/main_public.sock
    type: http
    resources:
      - names: [client, federation]
```

---
### `manhole_settings`

Connection settings for the manhole. You can find more information
on the manhole [here](../../manhole.md). Manhole sub-options include:
* `username` : the username for the manhole. This defaults to 'matrix'.
* `password`: The password for the manhole. This defaults to 'rabbithole'.
* `ssh_priv_key_path` and `ssh_pub_key_path`: The private and public SSH key pair used to encrypt the manhole traffic.
  If these are left unset, then hardcoded and non-secret keys are used,
  which could allow traffic to be intercepted if sent over a public network.

Example configuration:
```yaml
manhole_settings:
  username: manhole
  password: mypassword
  ssh_priv_key_path: CONFDIR/id_rsa
  ssh_pub_key_path: CONFDIR/id_rsa.pub
```
---
### `dummy_events_threshold`

Forward extremities can build up in a room due to networking delays between
homeservers. Once this happens in a large room, calculation of the state of
that room can become quite expensive. To mitigate this, once the number of
forward extremities reaches a given threshold, Synapse will send an
`org.matrix.dummy_event` event, which will reduce the forward extremities
in the room.

This setting defines the threshold (i.e. number of forward extremities in the room) at which dummy events are sent.
The default value is 10.

Example configuration:
```yaml
dummy_events_threshold: 5
```
---
### `delete_stale_devices_after`

An optional duration. If set, Synapse will run a daily background task to log out and
delete any device that hasn't been accessed for more than the specified amount of time.

Defaults to no duration, which means devices are never pruned.

**Note:** This task will always run on the main process, regardless of the value of
`run_background_tasks_on`. This is due to workers currently not having the ability to
delete devices.

Example configuration:
```yaml
delete_stale_devices_after: 1y
```
---
### `email`

Configuration for sending emails from Synapse.

Server admins can configure custom templates for email content. See
[here](../../templates.md) for more information.

This setting has the following sub-options:
* `smtp_host`: The hostname of the outgoing SMTP server to use. Defaults to 'localhost'.
* `smtp_port`: The port on the mail server for outgoing SMTP. Defaults to 465 if `force_tls` is true, else 25.

  _Changed in Synapse 1.64.0:_ the default port is now aware of `force_tls`.
* `smtp_user` and `smtp_pass`: Username/password for authentication to the SMTP server. By default, no
   authentication is attempted.
* `force_tls`: By default, Synapse connects over plain text and then optionally upgrades
   to TLS via STARTTLS. If this option is set to true, TLS is used from the start (Implicit TLS),
   and the option `require_transport_security` is ignored.
   It is recommended to enable this if supported by your mail server.

  _New in Synapse 1.64.0._
* `require_transport_security`: Set to true to require TLS transport security for SMTP.
   By default, Synapse will connect over plain text, and will then switch to
   TLS via STARTTLS *if the SMTP server supports it*. If this option is set,
   Synapse will refuse to connect unless the server supports STARTTLS.
* `enable_tls`: By default, if the server supports TLS, it will be used, and the server
   must present a certificate that is valid for 'smtp_host'. If this option
   is set to false, TLS will not be used.
* `notif_from`: defines the "From" address to use when sending emails.
    It must be set if email sending is enabled. The placeholder '%(app)s' will be replaced by the application name,
    which is normally set in `app_name`, but may be overridden by the
    Matrix client application. Note that the placeholder must be written '%(app)s', including the
    trailing 's'.
* `app_name`: `app_name` defines the default value for '%(app)s' in `notif_from` and email
   subjects. It defaults to 'Matrix'.
* `enable_notifs`: Set to true to allow users to receive e-mail notifications. If this is not set,
    users can configure e-mail notifications but will not receive them. Disabled by default.
* `notif_for_new_users`: Set to false to disable automatic subscription to email
   notifications for new users. Enabled by default.
* `notif_delay_before_mail`: The time to wait before emailing about a notification.
  This gives the user a chance to view the message via push or an open client.
  Defaults to 10 minutes.

  _New in Synapse 1.99.0._
* `client_base_url`: Custom URL for client links within the email notifications. By default
   links will be based on "https://matrix.to". (This setting used to be called `riot_base_url`;
   the old name is still supported for backwards-compatibility but is now deprecated.)
* `validation_token_lifetime`: Configures the time that a validation email will expire after sending.
   Defaults to 1h.
* `invite_client_location`: The web client location to direct users to during an invite. This is passed
   to the identity server as the `org.matrix.web_client_location` key. Defaults
   to unset, giving no guidance to the identity server.
* `subjects`: Subjects to use when sending emails from Synapse. The placeholder '%(app)s' will
   be replaced with the value of the `app_name` setting, or by a value dictated by the Matrix client application.
   In addition, each subject can use the following placeholders: '%(person)s', which will be replaced by the displayname
   of the user(s) that sent the message(s), e.g. "Alice and Bob", and '%(room)s', which will be replaced by the name of the room the
   message(s) have been sent to, e.g. "My super room". In addition, emails related to account administration will
   can use the '%(server_name)s' placeholder, which will be replaced by the value of the
   `server_name` setting in your Synapse configuration.

   Here is a list of subjects for notification emails that can be set:
     * `message_from_person_in_room`: Subject to use to notify about one message from one or more user(s) in a
        room which has a name. Defaults to "[%(app)s] You have a message on %(app)s from %(person)s in the %(room)s room..."
     * `message_from_person`: Subject to use to notify about one message from one or more user(s) in a
        room which doesn't have a name. Defaults to "[%(app)s] You have a message on %(app)s from %(person)s..."
     * `messages_from_person`: Subject to use to notify about multiple messages from one or more users in
        a room which doesn't have a name. Defaults to "[%(app)s] You have messages on %(app)s from %(person)s..."
     * `messages_in_room`: Subject to use to notify about multiple messages in a room which has a
        name. Defaults to "[%(app)s] You have messages on %(app)s in the %(room)s room..."
     * `messages_in_room_and_others`: Subject to use to notify about multiple messages in multiple rooms.
        Defaults to "[%(app)s] You have messages on %(app)s in the %(room)s room and others..."
     * `messages_from_person_and_others`: Subject to use to notify about multiple messages from multiple persons in
        multiple rooms. This is similar to the setting above except it's used when
        the room in which the notification was triggered has no name. Defaults to
        "[%(app)s] You have messages on %(app)s from %(person)s and others..."
     * `invite_from_person_to_room`: Subject to use to notify about an invite to a room which has a name.
        Defaults to  "[%(app)s] %(person)s has invited you to join the %(room)s room on %(app)s..."
     * `invite_from_person`: Subject to use to notify about an invite to a room which doesn't have a
        name. Defaults to "[%(app)s] %(person)s has invited you to chat on %(app)s..."
     * `password_reset`: Subject to use when sending a password reset email. Defaults to "[%(server_name)s] Password reset"
     * `email_validation`: Subject to use when sending a verification email to assert an address's
        ownership. Defaults to "[%(server_name)s] Validate your email"

Example configuration:

```yaml
email:
  smtp_host: mail.server
  smtp_port: 587
  smtp_user: "exampleusername"
  smtp_pass: "examplepassword"
  force_tls: true
  require_transport_security: true
  enable_tls: false
  notif_from: "Your Friendly %(app)s homeserver <noreply@example.com>"
  app_name: my_branded_matrix_server
  enable_notifs: true
  notif_for_new_users: false
  client_base_url: "http://localhost/riot"
  validation_token_lifetime: 15m
  invite_client_location: https://app.element.io

  subjects:
    message_from_person_in_room: "[%(app)s] You have a message on %(app)s from %(person)s in the %(room)s room..."
    message_from_person: "[%(app)s] You have a message on %(app)s from %(person)s..."
    messages_from_person: "[%(app)s] You have messages on %(app)s from %(person)s..."
    messages_in_room: "[%(app)s] You have messages on %(app)s in the %(room)s room..."
    messages_in_room_and_others: "[%(app)s] You have messages on %(app)s in the %(room)s room and others..."
    messages_from_person_and_others: "[%(app)s] You have messages on %(app)s from %(person)s and others..."
    invite_from_person_to_room: "[%(app)s] %(person)s has invited you to join the %(room)s room on %(app)s..."
    invite_from_person: "[%(app)s] %(person)s has invited you to chat on %(app)s..."
    password_reset: "[%(server_name)s] Password reset"
    email_validation: "[%(server_name)s] Validate your email"
```
---
### `max_event_delay_duration`

The maximum allowed duration by which sent events can be delayed, as per
[MSC4140](https://github.com/matrix-org/matrix-spec-proposals/pull/4140).
Must be a positive value if set.

Defaults to no duration (`null`), which disallows sending delayed events.

Example configuration:
```yaml
max_event_delay_duration: 24h
```

## Homeserver blocking
Useful options for Synapse admins.

---

### `admin_contact`

How to reach the server admin, used in `ResourceLimitError`. Defaults to none.

Example configuration:
```yaml
admin_contact: 'mailto:admin@server.com'
```
---
### `hs_disabled` and `hs_disabled_message`

Blocks users from connecting to the homeserver and provides a human-readable reason
why the connection was blocked. Defaults to false.

Example configuration:
```yaml
hs_disabled: true
hs_disabled_message: 'Reason for why the HS is blocked'
```
---
### `limit_usage_by_mau`

This option disables/enables monthly active user blocking. Used in cases where the admin or
server owner wants to limit to the number of monthly active users. When enabled and a limit is
reached the server returns a `ResourceLimitError` with error type `Codes.RESOURCE_LIMIT_EXCEEDED`.
Defaults to false. If this is enabled, a value for `max_mau_value` must also be set.

See [Monthly Active Users](../administration/monthly_active_users.md) for details on how to configure MAU.

Example configuration:
```yaml
limit_usage_by_mau: true
```
---
### `max_mau_value`

This option sets the hard limit of monthly active users above which the server will start
blocking user actions if `limit_usage_by_mau` is enabled. Defaults to 0.

Example configuration:
```yaml
max_mau_value: 50
```
---
### `mau_trial_days`

The option `mau_trial_days` is a means to add a grace period for active users. It
means that users must be active for the specified number of days before they
can be considered active and guards against the case where lots of users
sign up in a short space of time never to return after their initial
session. Defaults to 0.

Example configuration:
```yaml
mau_trial_days: 5
```
---
### `mau_appservice_trial_days`

The option `mau_appservice_trial_days` is similar to `mau_trial_days`, but applies a different
trial number if the user was registered by an appservice. A value
of 0 means no trial days are applied. Appservices not listed in this dictionary
use the value of `mau_trial_days` instead.

Example configuration:
```yaml
mau_appservice_trial_days:
  my_appservice_id: 3
  another_appservice_id: 6
```
---
### `mau_limit_alerting`

The option `mau_limit_alerting` is a means of limiting client-side alerting
should the mau limit be reached. This is useful for small instances
where the admin has 5 mau seats (say) for 5 specific people and no
interest increasing the mau limit further. Defaults to true, which
means that alerting is enabled.

Example configuration:
```yaml
mau_limit_alerting: false
```
---
### `mau_stats_only`

If enabled, the metrics for the number of monthly active users will
be populated, however no one will be limited based on these numbers. If `limit_usage_by_mau`
is true, this is implied to be true. Defaults to false.

Example configuration:
```yaml
mau_stats_only: true
```
---
### `mau_limit_reserved_threepids`

Sometimes the server admin will want to ensure certain accounts are
never blocked by mau checking. These accounts are specified by this option.
Defaults to none. Add accounts by specifying the `medium` and `address` of the
reserved threepid (3rd party identifier).

Example configuration:
```yaml
mau_limit_reserved_threepids:
  - medium: 'email'
    address: 'reserved_user@example.com'
```
---
### `server_context`

This option is used by phonehome stats to group together related servers.
Defaults to none.

Example configuration:
```yaml
server_context: context
```
---
### `limit_remote_rooms`

When this option is enabled, the room "complexity" will be checked before a user
joins a new remote room. If it is above the complexity limit, the server will
disallow joining, or will instantly leave. This is useful for homeservers that are
resource-constrained. Options for this setting include:
* `enabled`: whether this check is enabled. Defaults to false.
* `complexity`: the limit above which rooms cannot be joined. The default is 1.0.
* `complexity_error`: override the error which is returned when the room is too complex with a
   custom message.
* `admins_can_join`: allow server admins to join complex rooms. Default is false.

Room complexity is an arbitrary measure based on factors such as the number of
users in the room.

Example configuration:
```yaml
limit_remote_rooms:
  enabled: true
  complexity: 0.5
  complexity_error: "I can't let you do that, Dave."
  admins_can_join: true
```
---
### `require_membership_for_aliases`

Whether to require a user to be in the room to add an alias to it.
Defaults to true.

Example configuration:
```yaml
require_membership_for_aliases: false
```
---
### `allow_per_room_profiles`

Whether to allow per-room membership profiles through the sending of membership
events with profile information that differs from the target's global profile.
Defaults to true.

Example configuration:
```yaml
allow_per_room_profiles: false
```
---
### `max_avatar_size`

The largest permissible file size in bytes for a user avatar. Defaults to no restriction.
Use M for MB and K for KB.

Note that user avatar changes will not work if this is set without using Synapse's media repository.

Example configuration:
```yaml
max_avatar_size: 10M
```
---
### `allowed_avatar_mimetypes`

The MIME types allowed for user avatars. Defaults to no restriction.

Note that user avatar changes will not work if this is set without
using Synapse's media repository.

Example configuration:
```yaml
allowed_avatar_mimetypes: ["image/png", "image/jpeg", "image/gif"]
```
---
### `redaction_retention_period`

How long to keep redacted events in unredacted form in the database. After
this period redacted events get replaced with their redacted form in the DB.

Synapse will check whether the rentention period has concluded for redacted
events every 5 minutes. Thus, even if this option is set to `0`, Synapse may
still take up to 5 minutes to purge redacted events from the database.

Defaults to `7d`. Set to `null` to disable.

Example configuration:
```yaml
redaction_retention_period: 28d
```
---
### `forgotten_room_retention_period`

How long to keep locally forgotten rooms before purging them from the DB.

Defaults to `null`, meaning it's disabled.

Example configuration:
```yaml
forgotten_room_retention_period: 28d
```
---
### `user_ips_max_age`

How long to track users' last seen time and IPs in the database.

Defaults to `28d`. Set to `null` to disable clearing out of old rows.

Example configuration:
```yaml
user_ips_max_age: 14d
```
---
### `request_token_inhibit_3pid_errors`

Inhibits the `/requestToken` endpoints from returning an error that might leak
information about whether an e-mail address is in use or not on this
homeserver. Defaults to false.
Note that for some endpoints the error situation is the e-mail already being
used, and for others the error is entering the e-mail being unused.
If this option is enabled, instead of returning an error, these endpoints will
act as if no error happened and return a fake session ID ('sid') to clients.

Example configuration:
```yaml
request_token_inhibit_3pid_errors: true
```
---
### `next_link_domain_whitelist`

A list of domains that the domain portion of `next_link` parameters
must match.

This parameter is optionally provided by clients while requesting
validation of an email or phone number, and maps to a link that
users will be automatically redirected to after validation
succeeds. Clients can make use this parameter to aid the validation
process.

The whitelist is applied whether the homeserver or an identity server is handling validation.

The default value is no whitelist functionality; all domains are
allowed. Setting this value to an empty list will instead disallow
all domains.

Example configuration:
```yaml
next_link_domain_whitelist: ["matrix.org"]
```
---
### `templates` and `custom_template_directory`

These options define templates to use when generating email or HTML page contents.
The `custom_template_directory` determines which directory Synapse will try to
find template files in to use to generate email or HTML page contents.
If not set, or a file is not found within the template directory, a default
template from within the Synapse package will be used.

See [here](../../templates.md) for more
information about using custom templates.

Example configuration:
```yaml
templates:
  custom_template_directory: /path/to/custom/templates/
```
---
### `retention`

This option and the associated options determine message retention policy at the
