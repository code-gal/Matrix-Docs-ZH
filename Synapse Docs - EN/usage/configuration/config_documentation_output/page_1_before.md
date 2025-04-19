# Configuring Synapse

This is intended as a guide to the Synapse configuration. The behavior of a Synapse instance can be modified
through the many configuration settings documented here — each config option is explained,
including what the default is, how to change the default and what sort of behaviour the setting governs.
Also included is an example configuration for each setting. If you don't want to spend a lot of time
thinking about options, the config as generated sets sensible defaults for all values. Do note however that the
database defaults to SQLite, which is not recommended for production usage. You can read more on this subject
[here](../../setup/installation.md#using-postgresql).

## Config Conventions

Configuration options that take a time period can be set using a number
followed by a letter. Letters have the following meanings:

* `s` = second
* `m` = minute
* `h` = hour
* `d` = day
* `w` = week
* `y` = year

For example, setting `redaction_retention_period: 5m` would remove redacted
messages from the database after 5 minutes, rather than 5 months.

In addition, configuration options referring to size use the following suffixes:

* `K` = KiB, or 1024 bytes
* `M` = MiB, or 1,048,576 bytes
* `G` = GiB, or 1,073,741,824 bytes
* `T` = TiB, or 1,099,511,627,776 bytes

For example, setting `max_avatar_size: 10M` means that Synapse will not accept files larger than 10,485,760 bytes
for a user avatar.

## Config Validation

The configuration file can be validated with the following command:
```bash
python -m synapse.config read <config key to print> -c <path to config>
```

To validate the entire file, omit `read <config key to print>`:
```bash
python -m synapse.config -c <path to config>
```

To see how to set other options, check the help reference:
```bash
python -m synapse.config --help
```

### YAML
The configuration file is a [YAML](https://yaml.org/) file, which means that certain syntax rules
apply if you want your config file to be read properly. A few helpful things to know:
* `#` before any option in the config will comment out that setting and either a default (if available) will
   be applied or Synapse will ignore the setting. Thus, in example #1 below, the setting will be read and
   applied, but in example #2 the setting will not be read and a default will be applied.

   Example #1:
   ```yaml
   pid_file: DATADIR/homeserver.pid
   ```
   Example #2:
   ```yaml
   #pid_file: DATADIR/homeserver.pid
   ```
* Indentation matters! The indentation before a setting
  will determine whether a given setting is read as part of another
  setting, or considered on its own. Thus, in example #1, the `enabled` setting
  is read as a sub-option of the `presence` setting, and will be properly applied.

  However, the lack of indentation before the `enabled` setting in example #2 means
  that when reading the config, Synapse will consider both `presence` and `enabled` as
  different settings. In this case, `presence` has no value, and thus a default applied, and `enabled`
  is an option that Synapse doesn't recognize and thus ignores.

  Example #1:
  ```yaml
  presence:
    enabled: false
  ```
  Example #2:
  ```yaml
  presence:
  enabled: false
  ```
  In this manual, all top-level settings (ones with no indentation) are identified
  at the beginning of their section (i.e. "### `example_setting`") and
  the sub-options, if any, are identified and listed in the body of the section.
  In addition, each setting has an example of its usage, with the proper indentation
  shown.

## Modules

Server admins can expand Synapse's functionality with external modules.

See [here](../../modules/index.md) for more
documentation on how to configure or create custom modules for Synapse.


---
### `modules`

Use the `module` sub-option to add modules under this option to extend functionality.
The `module` setting then has a sub-option, `config`, which can be used to define some configuration
for the `module`.

Defaults to none.

Example configuration:
```yaml
modules:
  - module: my_super_module.MySuperClass
    config:
      do_thing: true
  - module: my_other_super_module.SomeClass
    config: {}
```
---
## Server

Define your homeserver name and other base options.

---
### `server_name`

This sets the public-facing domain of the server.

The `server_name` name will appear at the end of usernames and room addresses
created on your server. For example if the `server_name` was example.com,
usernames on your server would be in the format `@user:example.com`

In most cases you should avoid using a matrix specific subdomain such as
matrix.example.com or synapse.example.com as the `server_name` for the same
reasons you wouldn't use user@email.example.com as your email address.
See [here](../../delegate.md)
for information on how to host Synapse on a subdomain while preserving
a clean `server_name`.

The `server_name` cannot be changed later so it is important to
configure this correctly before you start Synapse. It should be all
lowercase and may contain an explicit port.

There is no default for this option.

Example configuration #1:
```yaml
server_name: matrix.org
```
Example configuration #2:
```yaml
server_name: localhost:8080
```
---
### `pid_file`

When running Synapse as a daemon, the file to store the pid in. Defaults to none.

Example configuration:
```yaml
pid_file: DATADIR/homeserver.pid
```
---
### `web_client_location`

The absolute URL to the web client which `/` will redirect to. Defaults to none.

Example configuration:
```yaml
web_client_location: https://riot.example.com/
```
---
### `public_baseurl`

The public-facing base URL that clients use to access this Homeserver (not
including _matrix/...). This is the same URL a user might enter into the
'Custom Homeserver URL' field on their client. If you use Synapse with a
reverse proxy, this should be the URL to reach Synapse via the proxy.
Otherwise, it should be the URL to reach Synapse's client HTTP listener (see
['listeners'](#listeners) below).

Defaults to `https://<server_name>/`.

Example configuration:
```yaml
public_baseurl: https://example.com/
```
---
### `serve_server_wellknown`

By default, other servers will try to reach our server on port 8448, which can
be inconvenient in some environments.

Provided `https://<server_name>/` on port 443 is routed to Synapse, this
option configures Synapse to serve a file at `https://<server_name>/.well-known/matrix/server`.
This will tell other servers to send traffic to port 443 instead.

This option currently defaults to false.

See [Delegation of incoming federation traffic](../../delegate.md) for more
information.

Example configuration:
```yaml
serve_server_wellknown: true
```
---
### `extra_well_known_client_content `

This option allows server runners to add arbitrary key-value pairs to the [client-facing `.well-known` response](https://spec.matrix.org/latest/client-server-api/#well-known-uri).
Note that the `public_baseurl` config option must be provided for Synapse to serve a response to `/.well-known/matrix/client` at all.

If this option is provided, it parses the given yaml to json and
serves it on `/.well-known/matrix/client` endpoint
alongside the standard properties.

*Added in Synapse 1.62.0.*

Example configuration:
```yaml
extra_well_known_client_content :
  option1: value1
  option2: value2
```
---
### `soft_file_limit`

Set the soft limit on the number of file descriptors synapse can use.
Zero is used to indicate synapse should set the soft limit to the hard limit.
Defaults to 0.

Example configuration:
```yaml
soft_file_limit: 3
```
---
### `presence`

Presence tracking allows users to see the state (e.g online/offline)
of other local and remote users. Set the `enabled` sub-option to false to
disable presence tracking on this homeserver. Defaults to true.
This option replaces the previous top-level 'use_presence' option.

Example configuration:
```yaml
presence:
  enabled: false
  include_offline_users_on_sync: false
```

`enabled` can also be set to a special value of "untracked" which ignores updates
received via clients and federation, while still accepting updates from the
[module API](../../modules/index.md).

*The "untracked" option was added in Synapse 1.96.0.*

When clients perform an initial or `full_state` sync, presence results for offline users are
not included by default. Setting `include_offline_users_on_sync` to `true` will always include
offline users in the results. Defaults to false.

---
### `require_auth_for_profile_requests`

Whether to require authentication to retrieve profile data (avatars, display names) of other
users through the client API. Defaults to false. Note that profile data is also available
via the federation API, unless `allow_profile_lookup_over_federation` is set to false.

Example configuration:
```yaml
require_auth_for_profile_requests: true
```
---
### `limit_profile_requests_to_users_who_share_rooms`

Use this option to require a user to share a room with another user in order
to retrieve their profile information. Only checked on Client-Server
requests. Profile requests from other servers should be checked by the
requesting server. Defaults to false.

Example configuration:
```yaml
limit_profile_requests_to_users_who_share_rooms: true
```
---
### `include_profile_data_on_invite`

Use this option to prevent a user's profile data from being retrieved and
displayed in a room until they have joined it. By default, a user's
profile data is included in an invite event, regardless of the values
of the above two settings, and whether or not the users share a server.
Defaults to true.

Example configuration:
```yaml
include_profile_data_on_invite: false
```
---
### `allow_public_rooms_without_auth`

If set to true, removes the need for authentication to access the server's
public rooms directory through the client API, meaning that anyone can
query the room directory. Defaults to false.

Example configuration:
```yaml
allow_public_rooms_without_auth: true
```
---
### `allow_public_rooms_over_federation`

If set to true, allows any other homeserver to fetch the server's public
rooms directory via federation. Defaults to false.

Example configuration:
```yaml
allow_public_rooms_over_federation: true
```
---
### `default_room_version`

The default room version for newly created rooms on this server.

Known room versions are listed [here](https://spec.matrix.org/latest/rooms/#complete-list-of-room-versions)

For example, for room version 1, `default_room_version` should be set
to "1".

Currently defaults to ["10"](https://spec.matrix.org/v1.5/rooms/v10/).

_Changed in Synapse 1.76:_ the default version room version was increased from [9](https://spec.matrix.org/v1.5/rooms/v9/) to [10](https://spec.matrix.org/v1.5/rooms/v10/).

Example configuration:
```yaml
default_room_version: "8"
```
---
### `gc_thresholds`

The garbage collection threshold parameters to pass to `gc.set_threshold`, if defined.
Defaults to none.

Example configuration:
```yaml
gc_thresholds: [700, 10, 10]
```
---
### `gc_min_interval`

The minimum time in seconds between each GC for a generation, regardless of
the GC thresholds. This ensures that we don't do GC too frequently. A value of `[1s, 10s, 30s]`
indicates that a second must pass between consecutive generation 0 GCs, etc.

Defaults to `[1s, 10s, 30s]`.

Example configuration:
```yaml
gc_min_interval: [0.5s, 30s, 1m]
```
---
### `filter_timeline_limit`

Set the limit on the returned events in the timeline in the get
and sync operations. Defaults to 100. A value of -1 means no upper limit.


Example configuration:
```yaml
filter_timeline_limit: 5000
```
---
### `block_non_admin_invites`

Whether room invites to users on this server should be blocked
(except those sent by local server admins). Defaults to false.

Example configuration:
```yaml
block_non_admin_invites: true
```
---
### `enable_search`

If set to false, new messages will not be indexed for searching and users
will receive errors when searching for messages. Defaults to true.

Example configuration:
```yaml
enable_search: false
```
---
### `ip_range_blacklist`

This option prevents outgoing requests from being sent to the specified blacklisted IP address
CIDR ranges. If this option is not specified then it defaults to private IP
address ranges (see the example below).

The blacklist applies to the outbound requests for federation, identity servers,
push servers, and for checking key validity for third-party invite events.

(0.0.0.0 and :: are always blacklisted, whether or not they are explicitly
listed here, since they correspond to unroutable addresses.)

This option replaces `federation_ip_range_blacklist` in Synapse v1.25.0.

Note: The value is ignored when an HTTP proxy is in use.

Example configuration:
```yaml
ip_range_blacklist:
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
### `ip_range_whitelist`

List of IP address CIDR ranges that should be allowed for federation,
identity servers, push servers, and for checking key validity for
third-party invite events. This is useful for specifying exceptions to
wide-ranging blacklisted target IP ranges - e.g. for communication with
a push server only visible in your network.

This whitelist overrides `ip_range_blacklist` and defaults to an empty
list.

Example configuration:
```yaml
ip_range_whitelist:
   - '192.168.1.1'
```
---
### `listeners`

List of ports that Synapse should listen on, their purpose and their
configuration.

Sub-options for each listener include:

* `port`: the TCP port to bind to.

* `tag`: An alias for the port in the logger name. If set the tag is logged instead
of the port. Default to `None`, is optional and only valid for listener with `type: http`.
See the docs [request log format](../administration/request_log.md).

* `bind_addresses`: a list of local addresses to listen on. The default is
       'all local interfaces'.

* `type`: the type of listener. Normally `http`, but other valid options are:

   * `manhole`: (see the docs [here](../../manhole.md)),

   * `metrics`: (see the docs [here](../../metrics-howto.md)),

* `tls`: set to true to enable TLS for this listener. Will use the TLS key/cert specified in tls_private_key_path / tls_certificate_path.

* `x_forwarded`: Only valid for an 'http' listener. Set to true to use the X-Forwarded-For header as the client IP. Useful when Synapse is
   behind a [reverse-proxy](../../reverse_proxy.md).

* `request_id_header`: The header extracted from each incoming request that is
   used as the basis for the request ID. The request ID is used in
   [logs](../administration/request_log.md#request-log-format) and tracing to
   correlate and match up requests. When unset, Synapse will automatically
   generate sequential request IDs. This option is useful when Synapse is behind
   a [reverse-proxy](../../reverse_proxy.md).

   _Added in Synapse 1.68.0._

* `resources`: Only valid for an 'http' listener. A list of resources to host
   on this port. Sub-options for each resource are:

   * `names`: a list of names of HTTP resources. See below for a list of valid resource names.

   * `compress`: set to true to enable gzip compression on HTTP bodies for this resource. This is currently only supported with the
     `client`, `consent`, `metrics` and `federation` resources.

* `additional_resources`: Only valid for an 'http' listener. A map of
   additional endpoints which should be loaded via dynamic modules.

Unix socket support (_Added in Synapse 1.89.0_):
* `path`: A path and filename for a Unix socket. Make sure it is located in a
  directory with read and write permissions, and that it already exists (the directory
  will not be created). Defaults to `None`.
  * **Note**: The use of both `path` and `port` options for the same `listener` is not
    compatible.
  * The `x_forwarded` option defaults to true  when using Unix sockets and can be omitted.
  * Other options that would not make sense to use with a UNIX socket, such as
    `bind_addresses` and `tls` will be ignored and can be removed.
* `mode`: The file permissions to set on the UNIX socket. Defaults to `666`
* **Note:** Must be set as `type: http` (does not support `metrics` and `manhole`).
  Also make sure that `metrics` is not included in `resources` -> `names`


Valid resource names are:

* `client`: the client-server API (/_matrix/client). Also implies `media` and `static`.
  If configuring the main process, the Synapse Admin API (/_synapse/admin) is also implied.

* `consent`: user consent forms (/_matrix/consent). See [here](../../consent_tracking.md) for more.

* `federation`: the server-server API (/_matrix/federation). Also implies `media`, `keys`, `openid`

* `keys`: the key discovery API (/_matrix/key).

* `media`: the media API (/_matrix/media).

* `metrics`: the metrics interface. See [here](../../metrics-howto.md). (Not compatible with Unix sockets)

* `openid`: OpenID authentication. See [here](../../openid.md).

* `replication`: the HTTP replication API (/_synapse/replication). See [here](../../workers.md).

* `static`: static resources under synapse/static (/_matrix/static). (Mostly useful for 'fallback authentication'.)

* `health`: the [health check endpoint](../../reverse_proxy.md#health-check-endpoint). This endpoint
  is by default active for all other resources and does not have to be activated separately.
  This is only useful if you want to use the health endpoint explicitly on a dedicated port or
  for [workers](../../workers.md) and containers without listener e.g.
  [application services](../../workers.md#notifying-application-services).

Example configuration #1:
```yaml
listeners:
  # TLS-enabled listener: for when matrix traffic is sent directly to synapse.
  #
  # (Note that you will also need to give Synapse a TLS key and certificate: see the TLS section
  # below.)
  #
  - port: 8448
    type: http
    tls: true
    resources:
      - names: [client, federation]
```
Example configuration #2:
```yaml
listeners:
  # Insecure HTTP listener: for when matrix traffic passes through a reverse proxy
  # that unwraps TLS.
  #
  # If you plan to use a reverse proxy, please see
  # https://element-hq.github.io/synapse/latest/reverse_proxy.html.
  #
  - port: 8008
    tls: false
    type: http
    x_forwarded: true
    bind_addresses: ['::1', '127.0.0.1']

    resources:
      - names: [client, federation]
        compress: false

    # example additional_resources:
    additional_resources:
      "/_matrix/my/custom/endpoint":
        module: my_module.CustomRequestHandler
