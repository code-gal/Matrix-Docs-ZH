* `"main"`: All other database tables and sequences.

All databases will end up with additional tables used for tracking database schema migrations
and any pending background updates. Synapse will create these automatically on startup when checking for
and/or performing database schema migrations.

To migrate an existing database configuration (e.g. all tables on a single database) to a different
configuration (e.g. the "main" data store on one database, and "state" on another), do the following:

1. Take a backup of your existing database. Things can and do go wrong and database corruption is no joke!
2. Ensure all pending database migrations have been applied and background updates have run. The simplest
   way to do this is to use the `update_synapse_database` script supplied with your Synapse installation.

   ```sh
   update_synapse_database --database-config homeserver.yaml --run-background-updates
   ```

3. Copy over the necessary tables and sequences from one database to the other. Tables relating to database
   migrations, schemas, schema versions and background updates should **not** be copied.

   As an example, say that you'd like to split out the "state" data store from an existing database which
   currently contains all data stores.

   Simply copy the tables and sequences defined above for the "state" datastore from the existing database
   to the secondary database. As noted above, additional tables will be created in the secondary database
   when Synapse is started.

4. Modify/create the `databases` option in your `homeserver.yaml` to match the desired database configuration.
5. Start Synapse. Check that it starts up successfully and that things generally seem to be working.
6. Drop the old tables that were copied in step 3.

Only one of the options `database` or `databases` may be specified in your config, but not both.

Example configuration:

```yaml
databases:
  basement_box:
    name: psycopg2
    txn_limit: 10000
    data_stores: ["main"]
    args:
      user: synapse_user
      password: secretpassword
      dbname: synapse_main
      host: localhost
      port: 5432
      cp_min: 5
      cp_max: 10

  my_other_database:
    name: psycopg2
    txn_limit: 10000
    data_stores: ["state"]
    args:
      user: synapse_user
      password: secretpassword
      dbname: synapse_state
      host: localhost
      port: 5432
      cp_min: 5
      cp_max: 10
```
---
## Logging
Config options related to logging.

---
### `log_config`

This option specifies a yaml python logging config file as described
[here](https://docs.python.org/3/library/logging.config.html#configuration-dictionary-schema).

Example configuration:
```yaml
log_config: "CONFDIR/SERVERNAME.log.config"
```
---
## Ratelimiting
Options related to ratelimiting in Synapse.

Each ratelimiting configuration is made of two parameters:
   - `per_second`: number of requests a client can send per second.
   - `burst_count`: number of requests a client can send before being throttled.
---
### `rc_message`


Ratelimiting settings for client messaging.

This is a ratelimiting option for messages that ratelimits sending based on the account the client
is using. It defaults to: `per_second: 0.2`, `burst_count: 10`.

Example configuration:
```yaml
rc_message:
  per_second: 0.5
  burst_count: 15
```
---
### `rc_registration`

This option ratelimits registration requests based on the client's IP address.
It defaults to `per_second: 0.17`, `burst_count: 3`.

Example configuration:
```yaml
rc_registration:
  per_second: 0.15
  burst_count: 2
```
---
### `rc_registration_token_validity`

This option checks the validity of registration tokens that ratelimits requests based on
the client's IP address.
Defaults to `per_second: 0.1`, `burst_count: 5`.

Example configuration:
```yaml
rc_registration_token_validity:
  per_second: 0.3
  burst_count: 6
```
---
### `rc_login`

This option specifies several limits for login:
* `address` ratelimits login requests based on the client's IP
      address. Defaults to `per_second: 0.003`, `burst_count: 5`.

* `account` ratelimits login requests based on the account the
  client is attempting to log into. Defaults to `per_second: 0.003`,
  `burst_count: 5`.

* `failed_attempts` ratelimits login requests based on the account the
  client is attempting to log into, based on the amount of failed login
  attempts for this account. Defaults to `per_second: 0.17`, `burst_count: 3`.

Example configuration:
```yaml
rc_login:
  address:
    per_second: 0.15
    burst_count: 5
  account:
    per_second: 0.18
    burst_count: 4
  failed_attempts:
    per_second: 0.19
    burst_count: 7
```
---
### `rc_admin_redaction`

This option sets ratelimiting redactions by room admins. If this is not explicitly
set then it uses the same ratelimiting as per `rc_message`. This is useful
to allow room admins to deal with abuse quickly.

Example configuration:
```yaml
rc_admin_redaction:
  per_second: 1
  burst_count: 50
```
---
### `rc_joins`

This option allows for ratelimiting number of rooms a user can join. This setting has the following sub-options:

* `local`: ratelimits when users are joining rooms the server is already in.
   Defaults to `per_second: 0.1`, `burst_count: 10`.

* `remote`: ratelimits when users are trying to join rooms not on the server (which
  can be more computationally expensive than restricting locally). Defaults to
  `per_second: 0.01`, `burst_count: 10`

Example configuration:
```yaml
rc_joins:
  local:
    per_second: 0.2
    burst_count: 15
  remote:
    per_second: 0.03
    burst_count: 12
```
---
### `rc_joins_per_room`

This option allows admins to ratelimit joins to a room based on the number of recent
joins (local or remote) to that room. It is intended to mitigate mass-join spam
waves which target multiple homeservers.

By default, one join is permitted to a room every second, with an accumulating
buffer of up to ten instantaneous joins.

Example configuration (default values):
```yaml
rc_joins_per_room:
  per_second: 1
  burst_count: 10
```

_Added in Synapse 1.64.0._

---
### `rc_3pid_validation`

This option ratelimits how often a user or IP can attempt to validate a 3PID.
Defaults to `per_second: 0.003`, `burst_count: 5`.

Example configuration:
```yaml
rc_3pid_validation:
  per_second: 0.003
  burst_count: 5
```
---
### `rc_invites`

This option sets ratelimiting how often invites can be sent in a room or to a
specific user. `per_room` defaults to `per_second: 0.3`, `burst_count: 10`,
`per_user` defaults to `per_second: 0.003`, `burst_count: 5`, and `per_issuer`
defaults to `per_second: 0.3`, `burst_count: 10`.

Client requests that invite user(s) when [creating a
room](https://spec.matrix.org/v1.2/client-server-api/#post_matrixclientv3createroom)
will count against the `rc_invites.per_room` limit, whereas
client requests to [invite a single user to a
room](https://spec.matrix.org/v1.2/client-server-api/#post_matrixclientv3roomsroomidinvite)
will count against both the `rc_invites.per_user` and `rc_invites.per_room` limits.

Federation requests to invite a user will count against the `rc_invites.per_user`
limit only, as Synapse presumes ratelimiting by room will be done by the sending server.

The `rc_invites.per_user` limit applies to the *receiver* of the invite, rather than the
sender, meaning that a `rc_invite.per_user.burst_count` of 5 mandates that a single user
cannot *receive* more than a burst of 5 invites at a time.

In contrast, the `rc_invites.per_issuer` limit applies to the *issuer* of the invite, meaning that a `rc_invite.per_issuer.burst_count` of 5 mandates that single user cannot *send* more than a burst of 5 invites at a time.

_Changed in version 1.63:_ added the `per_issuer` limit.

Example configuration:
```yaml
rc_invites:
  per_room:
    per_second: 0.5
    burst_count: 5
  per_user:
    per_second: 0.004
    burst_count: 3
  per_issuer:
    per_second: 0.5
    burst_count: 5
```

---
### `rc_third_party_invite`

This option ratelimits 3PID invites (i.e. invites sent to a third-party ID
such as an email address or a phone number) based on the account that's
sending the invite. Defaults to `per_second: 0.2`, `burst_count: 10`.

Example configuration:
```yaml
rc_third_party_invite:
  per_second: 0.2
  burst_count: 10
```
---
### `rc_media_create`

This option ratelimits creation of MXC URIs via the `/_matrix/media/v1/create`
endpoint based on the account that's creating the media. Defaults to
`per_second: 10`, `burst_count: 50`.

Example configuration:
```yaml
rc_media_create:
  per_second: 10
  burst_count: 50
```
---
### `rc_federation`

Defines limits on federation requests.

The `rc_federation` configuration has the following sub-options:
* `window_size`: window size in milliseconds. Defaults to 1000.
* `sleep_limit`: number of federation requests from a single server in
   a window before the server will delay processing the request. Defaults to 10.
* `sleep_delay`: duration in milliseconds to delay processing events
   from remote servers by if they go over the sleep limit. Defaults to 500.
* `reject_limit`: maximum number of concurrent federation requests
   allowed from a single server. Defaults to 50.
* `concurrent`: number of federation requests to concurrently process
   from a single server. Defaults to 3.

Example configuration:
```yaml
rc_federation:
  window_size: 750
  sleep_limit: 15
  sleep_delay: 400
  reject_limit: 40
  concurrent: 5
```
---
### `federation_rr_transactions_per_room_per_second`

Sets outgoing federation transaction frequency for sending read-receipts,
per-room.

If we end up trying to send out more read-receipts, they will get buffered up
into fewer transactions. Defaults to 50.

Example configuration:
```yaml
federation_rr_transactions_per_room_per_second: 40
```
---
## Media Store
Config options related to Synapse's media store.

---
### `enable_authenticated_media`

When set to true, all subsequent media uploads will be marked as authenticated, and will not be available over legacy
unauthenticated media endpoints (`/_matrix/media/(r0|v3|v1)/download` and `/_matrix/media/(r0|v3|v1)/thumbnail`) - requests for authenticated media over these endpoints will result in a 404. All media, including authenticated media, will be available over the authenticated media endpoints `_matrix/client/v1/media/download` and `_matrix/client/v1/media/thumbnail`. Media uploaded prior to setting this option to true will still be available over the legacy endpoints. Note if the setting is switched to false
after enabling, media marked as authenticated will be available over legacy endpoints. Defaults to false, but
this will change to true in a future Synapse release.

Example configuration:
```yaml
enable_authenticated_media: true
```
---
### `enable_media_repo`

Enable the media store service in the Synapse master. Defaults to true.
Set to false if you are using a separate media store worker.

Example configuration:
```yaml
enable_media_repo: false
```
---
### `media_store_path`

Directory where uploaded images and attachments are stored.

Example configuration:
```yaml
media_store_path: "DATADIR/media_store"
```
---
### `max_pending_media_uploads`

How many *pending media uploads* can a given user have? A pending media upload
is a created MXC URI that (a) is not expired (the `unused_expires_at` timestamp
has not passed) and (b) the media has not yet been uploaded for. Defaults to 5.

Example configuration:
```yaml
max_pending_media_uploads: 5
```
---
### `unused_expiration_time`

How long to wait in milliseconds before expiring created media IDs. Defaults to
"24h"

Example configuration:
```yaml
unused_expiration_time: "1h"
```
---
### `media_storage_providers`

Media storage providers allow media to be stored in different
locations. Defaults to none. Associated sub-options are:
* `module`: type of resource, e.g. `file_system`.
* `store_local`: whether to store newly uploaded local files
* `store_remote`: whether to store newly downloaded local files
* `store_synchronous`: whether to wait for successful storage for local uploads
* `config`: sets a path to the resource through the `directory` option

Example configuration:
```yaml
media_storage_providers:
  - module: file_system
    store_local: false
    store_remote: false
    store_synchronous: false
    config:
       directory: /mnt/some/other/directory
```
---
### `max_upload_size`

The largest allowed upload size in bytes.

If you are using a reverse proxy you may also need to set this value in
your reverse proxy's config. Defaults to 50M. Notably Nginx has a small max body size by default.
See [here](../../reverse_proxy.md) for more on using a reverse proxy with Synapse.

Example configuration:
```yaml
max_upload_size: 60M
```
---
### `max_image_pixels`

Maximum number of pixels that will be thumbnailed. Defaults to 32M.

Example configuration:
```yaml
max_image_pixels: 35M
```
---
### `remote_media_download_burst_count`

Remote media downloads are ratelimited using a [leaky bucket algorithm](https://en.wikipedia.org/wiki/Leaky_bucket), where a given "bucket" is keyed to the IP address of the requester when requesting remote media downloads. This configuration option sets the size of the bucket against which the size in bytes of downloads are penalized - if the bucket is full, ie a given number of bytes have already been downloaded, further downloads will be denied until the bucket drains.  Defaults to 500MiB. See also `remote_media_download_per_second` which determines the rate at which the "bucket" is emptied and thus has available space to authorize new requests.

Example configuration:
```yaml
remote_media_download_burst_count: 200M
```
---
### `remote_media_download_per_second`

Works in conjunction with `remote_media_download_burst_count` to ratelimit remote media downloads - this configuration option determines the rate at which the "bucket" (see above) leaks in bytes per second. As requests are made to download remote media, the size of those requests in bytes is added to the bucket, and once the bucket has reached it's capacity, no more requests will be allowed until a number of bytes has "drained" from the bucket. This setting determines the rate at which bytes drain from the bucket, with the practical effect that the larger the number, the faster the bucket leaks, allowing for more bytes downloaded over a shorter period of time. Defaults to 87KiB per second. See also `remote_media_download_burst_count`.

Example configuration:
```yaml
remote_media_download_per_second: 40K
```
---
### `prevent_media_downloads_from`

A list of domains to never download media from. Media from these
domains that is already downloaded will not be deleted, but will be
inaccessible to users. This option does not affect admin APIs trying
to download/operate on media.

This will not prevent the listed domains from accessing media themselves.
It simply prevents users on this server from downloading media originating
from the listed servers.

This will have no effect on media originating from the local server. This only
affects media downloaded from other Matrix servers, to control URL previews see
[`url_preview_ip_range_blacklist`](#url_preview_ip_range_blacklist) or
[`url_preview_url_blacklist`](#url_preview_url_blacklist).

Defaults to an empty list (nothing blocked).

Example configuration:
```yaml
prevent_media_downloads_from:
  - evil.example.org
  - evil2.example.org
```
---
### `dynamic_thumbnails`

Whether to generate new thumbnails on the fly to precisely match
the resolution requested by the client. If true then whenever
a new resolution is requested by the client the server will
generate a new thumbnail. If false the server will pick a thumbnail
from a precalculated list. Defaults to false.

Example configuration:
```yaml
dynamic_thumbnails: true
```
---
### `thumbnail_sizes`

List of thumbnails to precalculate when an image is uploaded. Associated sub-options are:
* `width`
* `height`
* `method`: i.e. `crop`, `scale`, etc.

Example configuration:
```yaml
thumbnail_sizes:
  - width: 32
    height: 32
    method: crop
  - width: 96
    height: 96
    method: crop
  - width: 320
    height: 240
    method: scale
  - width: 640
    height: 480
    method: scale
  - width: 800
    height: 600
    method: scale
```
---
### `media_retention`

Controls whether local media and entries in the remote media cache
(media that is downloaded from other homeservers) should be removed
under certain conditions, typically for the purpose of saving space.

Purging media files will be the carried out by the media worker
(that is, the worker that has the `enable_media_repo` homeserver config
option set to 'true'). This may be the main process.

The `media_retention.local_media_lifetime` and
`media_retention.remote_media_lifetime` config options control whether
media will be purged if it has not been accessed in a given amount of
time. Note that media is 'accessed' when loaded in a room in a client, or
otherwise downloaded by a local or remote user. If the media has never
been accessed, the media's creation time is used instead. Both thumbnails
and the original media will be removed. If either of these options are unset,
then media of that type will not be purged.

Local or cached remote media that has been
[quarantined](../../admin_api/media_admin_api.md#quarantining-media-in-a-room)
will not be deleted. Similarly, local media that has been marked as
[protected from quarantine](../../admin_api/media_admin_api.md#protecting-media-from-being-quarantined)
will not be deleted.

Example configuration:
```yaml
media_retention:
    local_media_lifetime: 90d
    remote_media_lifetime: 14d
```
---
### `url_preview_enabled`

This setting determines whether the preview URL API is enabled.
It is disabled by default. Set to true to enable. If enabled you must specify a
`url_preview_ip_range_blacklist` blacklist.

Example configuration:
```yaml
url_preview_enabled: true
```
---
### `url_preview_ip_range_blacklist`

List of IP address CIDR ranges that the URL preview spider is denied
from accessing.  There are no defaults: you must explicitly
specify a list for URL previewing to work.  You should specify any
internal services in your network that you do not want synapse to try
to connect to, otherwise anyone in any Matrix room could cause your
synapse to issue arbitrary GET requests to your internal services,
causing serious security issues.

(0.0.0.0 and :: are always blacklisted, whether or not they are explicitly
listed here, since they correspond to unroutable addresses.)

This must be specified if `url_preview_enabled` is set. It is recommended that
you use the following example list as a starting point.

Note: The value is ignored when an HTTP proxy is in use.

Example configuration:
```yaml
url_preview_ip_range_blacklist:
