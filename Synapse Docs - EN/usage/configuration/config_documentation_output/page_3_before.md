This option and the associated options determine message retention policy at the
server level.

Room admins and mods can define a retention period for their rooms using the
`m.room.retention` state event, and server admins can cap this period by setting
the `allowed_lifetime_min` and `allowed_lifetime_max` config options.

If this feature is enabled, Synapse will regularly look for and purge events
which are older than the room's maximum retention period. Synapse will also
filter events received over federation so that events that should have been
purged are ignored and not stored again.

The message retention policies feature is disabled by default. You can read more
about this feature [here](../../message_retention_policies.md).

This setting has the following sub-options:
* `default_policy`: Default retention policy. If set, Synapse will apply it to rooms that lack the
   'm.room.retention' state event. This option is further specified by the
   `min_lifetime` and `max_lifetime` sub-options associated with it. Note that the
    value of `min_lifetime` doesn't matter much because Synapse doesn't take it into account yet.

* `allowed_lifetime_min` and `allowed_lifetime_max`: Retention policy limits. If
   set, and the state of a room contains a `m.room.retention` event in its state
   which contains a `min_lifetime` or a `max_lifetime` that's out of these bounds,
   Synapse will cap the room's policy to these limits when running purge jobs.

* `purge_jobs` and the associated `shortest_max_lifetime` and `longest_max_lifetime` sub-options:
   Server admins can define the settings of the background jobs purging the
   events whose lifetime has expired under the `purge_jobs` section.

  If no configuration is provided for this option, a single job will be set up to delete
  expired events in every room daily.

  Each job's configuration defines which range of message lifetimes the job
  takes care of. For example, if `shortest_max_lifetime` is '2d' and
  `longest_max_lifetime` is '3d', the job will handle purging expired events in
  rooms whose state defines a `max_lifetime` that's both higher than 2 days, and
  lower than or equal to 3 days. Both the minimum and the maximum value of a
  range are optional, e.g. a job with no `shortest_max_lifetime` and a
  `longest_max_lifetime` of '3d' will handle every room with a retention policy
  whose `max_lifetime` is lower than or equal to three days.

  The rationale for this per-job configuration is that some rooms might have a
  retention policy with a low `max_lifetime`, where history needs to be purged
  of outdated messages on a more frequent basis than for the rest of the rooms
  (e.g. every 12h), but not want that purge to be performed by a job that's
  iterating over every room it knows, which could be heavy on the server.

  If any purge job is configured, it is strongly recommended to have at least
  a single job with neither `shortest_max_lifetime` nor `longest_max_lifetime`
  set, or one job without `shortest_max_lifetime` and one job without
  `longest_max_lifetime` set. Otherwise some rooms might be ignored, even if
  `allowed_lifetime_min` and `allowed_lifetime_max` are set, because capping a
  room's policy to these values is done after the policies are retrieved from
  Synapse's database (which is done using the range specified in a purge job's
  configuration).

Example configuration:
```yaml
retention:
  enabled: true
  default_policy:
    min_lifetime: 1d
    max_lifetime: 1y
  allowed_lifetime_min: 1d
  allowed_lifetime_max: 1y
  purge_jobs:
    - longest_max_lifetime: 3d
      interval: 12h
    - shortest_max_lifetime: 3d
      interval: 1d
```
---
## TLS

Options related to TLS.

---
### `tls_certificate_path`

This option specifies a PEM-encoded X509 certificate for TLS.
This certificate, as of Synapse 1.0, will need to be a valid and verifiable
certificate, signed by a recognised Certificate Authority. Defaults to none.

Be sure to use a `.pem` file that includes the full certificate chain including
any intermediate certificates (for instance, if using certbot, use
`fullchain.pem` as your certificate, not `cert.pem`).

Example configuration:
```yaml
tls_certificate_path: "CONFDIR/SERVERNAME.tls.crt"
```
---
### `tls_private_key_path`

PEM-encoded private key for TLS. Defaults to none.

Example configuration:
```yaml
tls_private_key_path: "CONFDIR/SERVERNAME.tls.key"
```
---
### `federation_verify_certificates`
Whether to verify TLS server certificates for outbound federation requests.

Defaults to true. To disable certificate verification, set the option to false.

Example configuration:
```yaml
federation_verify_certificates: false
```
---
### `federation_client_minimum_tls_version`

The minimum TLS version that will be used for outbound federation requests.

Defaults to `"1"`. Configurable to `"1"`, `"1.1"`, `"1.2"`, or `"1.3"`. Note
that setting this value higher than `"1.2"` will prevent federation to most
of the public Matrix network: only configure it to `"1.3"` if you have an
entirely private federation setup and you can ensure TLS 1.3 support.

Example configuration:
```yaml
federation_client_minimum_tls_version: "1.2"
```
---
### `federation_certificate_verification_whitelist`

Skip federation certificate verification on a given whitelist
of domains.

This setting should only be used in very specific cases, such as
federation over Tor hidden services and similar. For private networks
of homeservers, you likely want to use a private CA instead.

Only effective if `federation_verify_certificates` is `true`.

Example configuration:
```yaml
federation_certificate_verification_whitelist:
  - lon.example.com
  - "*.domain.com"
  - "*.onion"
```
---
### `federation_custom_ca_list`

List of custom certificate authorities for federation traffic.

This setting should only normally be used within a private network of
homeservers.

Note that this list will replace those that are provided by your
operating environment. Certificates must be in PEM format.

Example configuration:
```yaml
federation_custom_ca_list:
  - myCA1.pem
  - myCA2.pem
  - myCA3.pem
```
---
## Federation

Options related to federation.

---
### `federation_domain_whitelist`

Restrict federation to the given whitelist of domains.
N.B. we recommend also firewalling your federation listener to limit
inbound federation traffic as early as possible, rather than relying
purely on this application-layer restriction.  If not specified, the
default is to whitelist everything.

Note: this does not stop a server from joining rooms that servers not on the
whitelist are in. As such, this option is really only useful to establish a
"private federation", where a group of servers all whitelist each other and have
the same whitelist.

Example configuration:
```yaml
federation_domain_whitelist:
  - lon.example.com
  - nyc.example.com
  - syd.example.com
```
---
### `federation_whitelist_endpoint_enabled`

Enables an endpoint for fetching the federation whitelist config.

The request method and path is `GET /_synapse/client/v1/config/federation_whitelist`, and the
response format is:

```json
{
    "whitelist_enabled": true,  // Whether the federation whitelist is being enforced
    "whitelist": [  // Which server names are allowed by the whitelist
        "example.com"
    ]
}
```

If `whitelist_enabled` is `false` then the server is permitted to federate with all others.

The endpoint requires authentication.

Example configuration:
```yaml
federation_whitelist_endpoint_enabled: true
```
---
### `federation_metrics_domains`

Report prometheus metrics on the age of PDUs being sent to and received from
the given domains. This can be used to give an idea of "delay" on inbound
and outbound federation, though be aware that any delay can be due to problems
at either end or with the intermediate network.

By default, no domains are monitored in this way.

Example configuration:
```yaml
federation_metrics_domains:
  - matrix.org
  - example.com
```
---
### `allow_profile_lookup_over_federation`

Set to false to disable profile lookup over federation. By default, the
Federation API allows other homeservers to obtain profile data of any user
on this homeserver.

Example configuration:
```yaml
allow_profile_lookup_over_federation: false
```
---
### `allow_device_name_lookup_over_federation`

Set this option to true to allow device display name lookup over federation. By default, the
Federation API prevents other homeservers from obtaining the display names of any user devices
on this homeserver.

Example configuration:
```yaml
allow_device_name_lookup_over_federation: true
```
---
### `federation`

The federation section defines some sub-options related to federation.

The following options are related to configuring timeout and retry logic for one request,
independently of the others.
Short retry algorithm is used when something or someone will wait for the request to have an
answer, while long retry is used for requests that happen in the background,
like sending a federation transaction.

* `client_timeout`: timeout for the federation requests. Default to 60s.
* `max_short_retry_delay`: maximum delay to be used for the short retry algo. Default to 2s.
* `max_long_retry_delay`: maximum delay to be used for the short retry algo. Default to 60s.
* `max_short_retries`: maximum number of retries for the short retry algo. Default to 3 attempts.
* `max_long_retries`: maximum number of retries for the long retry algo. Default to 10 attempts.

The following options control the retry logic when communicating with a specific homeserver destination.
Unlike the previous configuration options, these values apply across all requests
for a given destination and the state of the backoff is stored in the database.

* `destination_min_retry_interval`: the initial backoff, after the first request fails. Defaults to 10m.
* `destination_retry_multiplier`: how much we multiply the backoff by after each subsequent fail. Defaults to 2.
* `destination_max_retry_interval`: a cap on the backoff. Defaults to a week.

Example configuration:
```yaml
federation:
  client_timeout: 180s
  max_short_retry_delay: 7s
  max_long_retry_delay: 100s
  max_short_retries: 5
  max_long_retries: 20
  destination_min_retry_interval: 30s
  destination_retry_multiplier: 5
  destination_max_retry_interval: 12h
```
---
## Caching

Options related to caching.

---
### `event_cache_size`

The number of events to cache in memory. Defaults to 10K. Like other caches,
this is affected by `caches.global_factor` (see below).

For example, the default is 10K and the global_factor default is 0.5.

Since 10K * 0.5 is 5K then the event cache size will be 5K.

The cache affected by this configuration is named as "*getEvent*".

Note that this option is not part of the `caches` section.

Example configuration:
```yaml
event_cache_size: 15K
```
---
### `caches` and associated values

A cache 'factor' is a multiplier that can be applied to each of
Synapse's caches in order to increase or decrease the maximum
number of entries that can be stored.

`caches` can be configured through the following sub-options:

* `global_factor`: Controls the global cache factor, which is the default cache factor
  for all caches if a specific factor for that cache is not otherwise
  set.

  This can also be set by the `SYNAPSE_CACHE_FACTOR` environment
  variable. Setting by environment variable takes priority over
  setting through the config file.

  Defaults to 0.5, which will halve the size of all caches.

  Note that changing this value also affects the HTTP connection pool.

* `per_cache_factors`: A dictionary of cache name to cache factor for that individual
   cache. Overrides the global cache factor for a given cache.

   These can also be set through environment variables comprised
   of `SYNAPSE_CACHE_FACTOR_` + the name of the cache in capital
   letters and underscores. Setting by environment variable
   takes priority over setting through the config file.
   Ex. `SYNAPSE_CACHE_FACTOR_GET_USERS_WHO_SHARE_ROOM_WITH_USER=2.0`

   Some caches have '*' and other characters that are not
   alphanumeric or underscores. These caches can be named with or
   without the special characters stripped. For example, to specify
   the cache factor for `*stateGroupCache*` via an environment
   variable would be `SYNAPSE_CACHE_FACTOR_STATEGROUPCACHE=2.0`.

* `expire_caches`: Controls whether cache entries are evicted after a specified time
   period. Defaults to true. Set to false to disable this feature. Note that never expiring
   caches may result in excessive memory usage.

* `cache_entry_ttl`: If `expire_caches` is enabled, this flag controls how long an entry can
  be in a cache without having been accessed before being evicted.
  Defaults to 30m.

* `sync_response_cache_duration`: Controls how long the results of a /sync request are
  cached for after a successful response is returned. A higher duration can help clients
  with intermittent connections, at the cost of higher memory usage.
  A value of zero means that sync responses are not cached.
  Defaults to 2m.

  *Changed in Synapse 1.62.0*: The default was changed from 0 to 2m.

* `cache_autotuning` and its sub-options `max_cache_memory_usage`, `target_cache_memory_usage`, and
   `min_cache_ttl` work in conjunction with each other to maintain a balance between cache memory
   usage and cache entry availability. You must be using [jemalloc](../administration/admin_faq.md#help-synapse-is-slow-and-eats-all-my-ramcpu)
   to utilize this option, and all three of the options must be specified for this feature to work. This option
   defaults to off, enable it by providing values for the sub-options listed below. Please note that the feature will not work
   and may cause unstable behavior (such as excessive emptying of caches or exceptions) if all of the values are not provided.
   Please see the [Config Conventions](#config-conventions) for information on how to specify memory size and cache expiry
   durations.
     * `max_cache_memory_usage` sets a ceiling on how much memory the cache can use before caches begin to be continuously evicted.
        They will continue to be evicted until the memory usage drops below the `target_cache_memory_usage`, set in
        the setting below, or until the `min_cache_ttl` is hit. There is no default value for this option.
     * `target_cache_memory_usage` sets a rough target for the desired memory usage of the caches. There is no default value
        for this option.
     * `min_cache_ttl` sets a limit under which newer cache entries are not evicted and is only applied when
        caches are actively being evicted/`max_cache_memory_usage` has been exceeded. This is to protect hot caches
        from being emptied while Synapse is evicting due to memory. There is no default value for this option.

Example configuration:
```yaml
event_cache_size: 15K
caches:
  global_factor: 1.0
  per_cache_factors:
    get_users_who_share_room_with_user: 2.0
  sync_response_cache_duration: 2m
  cache_autotuning:
    max_cache_memory_usage: 1024M
    target_cache_memory_usage: 758M
    min_cache_ttl: 5m
```

### Reloading cache factors

The cache factors (i.e. `caches.global_factor` and `caches.per_cache_factors`)  may be reloaded at any time by sending a
[`SIGHUP`](https://en.wikipedia.org/wiki/SIGHUP) signal to Synapse using e.g.

```commandline
kill -HUP [PID_OF_SYNAPSE_PROCESS]
```

If you are running multiple workers, you must individually update the worker
config file and send this signal to each worker process.

If you're using the [example systemd service](https://github.com/element-hq/synapse/blob/develop/contrib/systemd/matrix-synapse.service)
file in Synapse's `contrib` directory, you can send a `SIGHUP` signal by using
`systemctl reload matrix-synapse`.

---
## Database
Config options related to database settings.

---
### `database`

The `database` setting defines the database that synapse uses to store all of
its data.

Associated sub-options:

* `name`: this option specifies the database engine to use: either `sqlite3` (for SQLite)
  or `psycopg2` (for PostgreSQL). If no name is specified Synapse will default to SQLite.

* `txn_limit` gives the maximum number of transactions to run per connection
  before reconnecting. Defaults to 0, which means no limit.

* `allow_unsafe_locale` is an option specific to Postgres. Under the default behavior, Synapse will refuse to
  start if the postgres db is set to a non-C locale. You can override this behavior (which is *not* recommended)
  by setting `allow_unsafe_locale` to true. Note that doing so may corrupt your database. You can find more information
  [here](../../postgres.md#fixing-incorrect-collate-or-ctype) and [here](https://wiki.postgresql.org/wiki/Locale_data_changes).

* `args` gives options which are passed through to the database engine,
  except for options starting with `cp_`, which are used to configure the Twisted
  connection pool. For a reference to valid arguments, see:
    * for [sqlite](https://docs.python.org/3/library/sqlite3.html#sqlite3.connect)
    * for [postgres](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-PARAMKEYWORDS)
    * for [the connection pool](https://docs.twistedmatrix.com/en/stable/api/twisted.enterprise.adbapi.ConnectionPool.html#__init__)

For more information on using Synapse with Postgres,
see [here](../../postgres.md).

Example SQLite configuration:
```yaml
database:
  name: sqlite3
  args:
    database: /path/to/homeserver.db
```

Example Postgres configuration:
```yaml
database:
  name: psycopg2
  txn_limit: 10000
  args:
    user: synapse_user
    password: secretpassword
    dbname: synapse
    host: localhost
    port: 5432
    cp_min: 5
    cp_max: 10
```
---
### `databases`

The `databases` option allows specifying a mapping between certain database tables and
database host details, spreading the load of a single Synapse instance across multiple
database backends. This is often referred to as "database sharding". This option is only
supported for PostgreSQL database backends.

**Important note:** This is a supported option, but is not currently used in production by the
Matrix.org Foundation. Proceed with caution and always make backups.

`databases` is a dictionary of arbitrarily-named database entries. Each entry is equivalent
to the value of the `database` homeserver config option (see above), with the addition of
a `data_stores` key. `data_stores` is an array of strings that specifies the data store(s)
(a defined label for a set of tables) that should be stored on the associated database
backend entry.

The currently defined values for `data_stores` are:

* `"state"`: Database that relates to state groups will be stored in this database.

  Specifically, that means the following tables:
  * `state_groups`
  * `state_group_edges`
  * `state_groups_state`

  And the following sequences:
  * `state_groups_seq_id`

* `"main"`: All other database tables and sequences.
