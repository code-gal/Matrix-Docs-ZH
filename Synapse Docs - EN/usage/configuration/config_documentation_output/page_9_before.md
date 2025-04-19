      "m.room.canonical_alias": 50
      "m.room.avatar": 50
      "m.room.tombstone": 100
      "m.room.server_acl": 100
      "m.room.encryption": 100
   trusted_private_chat: null
   public_chat: null
```

---
### `forget_rooms_on_leave`

Set to true to automatically forget rooms for users when they leave them, either
normally or via a kick or ban. Defaults to false.

Example configuration:
```yaml
forget_rooms_on_leave: false
```
---
### `exclude_rooms_from_sync`
A list of rooms to exclude from sync responses. This is useful for server
administrators wishing to group users into a room without these users being able
to see it from their client.

By default, no room is excluded.

Example configuration:
```yaml
exclude_rooms_from_sync:
    - "!foo:example.com"
```

---
## Opentracing
Configuration options related to Opentracing support.

---
### `opentracing`

These settings enable and configure opentracing, which implements distributed tracing.
This allows you to observe the causal chains of events across servers
including requests, key lookups etc., across any server running
synapse or any other services which support opentracing
(specifically those implemented with Jaeger).

Sub-options include:
* `enabled`: whether tracing is enabled. Set to true to enable. Disabled by default.
* `homeserver_whitelist`: The list of homeservers we wish to send and receive span contexts and span baggage.
   See [here](../../opentracing.md) for more.
   This is a list of regexes which are matched against the `server_name` of the homeserver.
   By default, it is empty, so no servers are matched.
* `force_tracing_for_users`: # A list of the matrix IDs of users whose requests will always be traced,
   even if the tracing system would otherwise drop the traces due to probabilistic sampling.
    By default, the list is empty.
* `jaeger_config`: Jaeger can be configured to sample traces at different rates.
   All configuration options provided by Jaeger can be set here. Jaeger's configuration is
   mostly related to trace sampling which is documented [here](https://www.jaegertracing.io/docs/latest/sampling/).

Example configuration:
```yaml
opentracing:
    enabled: true
    homeserver_whitelist:
      - ".*"
    force_tracing_for_users:
      - "@user1:server_name"
      - "@user2:server_name"

    jaeger_config:
      sampler:
        type: const
        param: 1
      logging:
        false
```
---
## Coordinating workers
Configuration options related to workers which belong in the main config file
(usually called `homeserver.yaml`).
A Synapse deployment can scale horizontally by running multiple Synapse processes
called _workers_. Incoming requests are distributed between workers to handle higher
loads. Some workers are privileged and can accept requests from other workers.

As a result, the worker configuration is divided into two parts.

1. The first part (in this section of the manual) defines which shardable tasks
   are delegated to privileged workers. This allows unprivileged workers to make
   requests to a privileged worker to act on their behalf.
1. [The second part](#individual-worker-configuration)
   controls the behaviour of individual workers in isolation.

For guidance on setting up workers, see the [worker documentation](../../workers.md).

---
### `worker_replication_secret`

A shared secret used by the replication APIs on the main process to authenticate
HTTP requests from workers.

The default, this value is omitted (equivalently `null`), which means that
traffic between the workers and the main process is not authenticated.

Example configuration:
```yaml
worker_replication_secret: "secret_secret"
```
---
### `start_pushers`

Unnecessary to set if using [`pusher_instances`](#pusher_instances) with [`generic_workers`](../../workers.md#synapseappgeneric_worker).

Controls sending of push notifications on the main process. Set to `false`
if using a [pusher worker](../../workers.md#synapseapppusher). Defaults to `true`.

Example configuration:
```yaml
start_pushers: false
```
---
### `pusher_instances`

It is possible to scale the processes that handle sending push notifications to [sygnal](https://github.com/matrix-org/sygnal)
and email by running a [`generic_worker`](../../workers.md#synapseappgeneric_worker) and adding it's [`worker_name`](#worker_name) to
a `pusher_instances` map. Doing so will remove handling of this function from the main
process. Multiple workers can be added to this map, in which case the work is balanced
across them. Ensure the main process and all pusher workers are restarted after changing
this option.

Example configuration for a single worker:
```yaml
pusher_instances:
  - pusher_worker1
```
And for multiple workers:
```yaml
pusher_instances:
  - pusher_worker1
  - pusher_worker2
```

---
### `send_federation`

Unnecessary to set if using [`federation_sender_instances`](#federation_sender_instances) with [`generic_workers`](../../workers.md#synapseappgeneric_worker).

Controls sending of outbound federation transactions on the main process.
Set to `false` if using a [federation sender worker](../../workers.md#synapseappfederation_sender).
Defaults to `true`.

Example configuration:
```yaml
send_federation: false
```
---
### `federation_sender_instances`

It is possible to scale the processes that handle sending outbound federation requests
by running a [`generic_worker`](../../workers.md#synapseappgeneric_worker) and adding it's [`worker_name`](#worker_name) to
a `federation_sender_instances` map. Doing so will remove handling of this function from
the main process. Multiple workers can be added to this map, in which case the work is
balanced across them. 

The way that the load balancing works is any outbound federation request will be assigned 
to a federation sender worker based on the hash of the destination server name. This
means that all requests being sent to the same destination will be processed by the same
worker instance. Multiple `federation_sender_instances` are useful if there is a federation
with multiple servers.

This configuration setting must be shared between all workers handling federation
sending, and if changed all federation sender workers must be stopped at the same time
and then started, to ensure that all instances are running with the same config (otherwise
events may be dropped).

Example configuration for a single worker:
```yaml
federation_sender_instances:
  - federation_sender1
```
And for multiple workers:
```yaml
federation_sender_instances:
  - federation_sender1
  - federation_sender2
```
---
### `instance_map`

When using workers this should be a map from [`worker_name`](#worker_name) to the HTTP
replication listener of the worker, if configured, and to the main process. Each worker
declared under [`stream_writers`](../../workers.md#stream-writers) and
[`outbound_federation_restricted_to`](#outbound_federation_restricted_to) needs a HTTP
replication listener, and that listener should be included in the `instance_map`. The
main process also needs an entry on the `instance_map`, and it should be listed under
`main` **if even one other worker exists**. Ensure the port matches with what is
declared inside the `listener` block for a `replication` listener.


Example configuration:
```yaml
instance_map:
  main:
    host: localhost
    port: 8030
  worker1:
    host: localhost
    port: 8034
```
Example configuration(#2, for UNIX sockets):
```yaml
instance_map:
  main:
    path: /run/synapse/main_replication.sock
  worker1:
    path: /run/synapse/worker1_replication.sock
```
---
### `stream_writers`

Experimental: When using workers you can define which workers should
handle writing to streams such as event persistence and typing notifications.
Any worker specified here must also be in the [`instance_map`](#instance_map).

See the list of available streams in the
[worker documentation](../../workers.md#stream-writers).

Example configuration:
```yaml
stream_writers:
  events: worker1
  typing: worker1
```
---
### `outbound_federation_restricted_to`

When using workers, you can restrict outbound federation traffic to only go through a
specific subset of workers. Any worker specified here must also be in the
[`instance_map`](#instance_map).
[`worker_replication_secret`](#worker_replication_secret) must also be configured to
authorize inter-worker communication.

```yaml
outbound_federation_restricted_to:
  - federation_sender1
  - federation_sender2
```

Also see the [worker
documentation](../../workers.md#restrict-outbound-federation-traffic-to-a-specific-set-of-workers)
for more info.

_Added in Synapse 1.89.0._

---
### `run_background_tasks_on`

The [worker](../../workers.md#background-tasks) that is used to run
background tasks (e.g. cleaning up expired data). If not provided this
defaults to the main process.

Example configuration:
```yaml
run_background_tasks_on: worker1
```
---
### `update_user_directory_from_worker`

The [worker](../../workers.md#updating-the-user-directory) that is used to
update the user directory. If not provided this defaults to the main process.

Example configuration:
```yaml
update_user_directory_from_worker: worker1
```

_Added in Synapse 1.59.0._

---
### `notify_appservices_from_worker`

The [worker](../../workers.md#notifying-application-services) that is used to
send output traffic to Application Services. If not provided this defaults
to the main process.

Example configuration:
```yaml
notify_appservices_from_worker: worker1
```

_Added in Synapse 1.59.0._

---
### `media_instance_running_background_jobs`

The [worker](../../workers.md#synapseappmedia_repository) that is used to run
background tasks for media repository. If running multiple media repositories
you must configure a single instance to run the background tasks. If not provided
this defaults to the main process or your single `media_repository` worker.

Example configuration:
```yaml
media_instance_running_background_jobs: worker1
```

_Added in Synapse 1.16.0._

---
### `redis`

Configuration for Redis when using workers. This *must* be enabled when using workers.
This setting has the following sub-options:
* `enabled`: whether to use Redis support. Defaults to false.
* `host` and `port`: Optional host and port to use to connect to redis. Defaults to
   localhost and 6379
* `path`: The full path to a local Unix socket file. **If this is used, `host` and
 `port` are ignored.** Defaults to `/tmp/redis.sock'
* `password`: Optional password if configured on the Redis instance.
* `password_path`: Alternative to `password`, reading the password from an
   external file. The file should be a plain text file, containing only the
   password. Synapse reads the password from the given file once at startup.
* `dbid`: Optional redis dbid if needs to connect to specific redis logical db.
* `use_tls`: Whether to use tls connection. Defaults to false.
* `certificate_file`: Optional path to the certificate file
* `private_key_file`: Optional path to the private key file
* `ca_file`: Optional path to the CA certificate file. Use this one or:
* `ca_path`: Optional path to the folder containing the CA certificate file

  _Added in Synapse 1.78.0._

  _Changed in Synapse 1.84.0: Added use\_tls, certificate\_file, private\_key\_file, ca\_file and ca\_path attributes_

  _Changed in Synapse 1.85.0: Added path option to use a local Unix socket_

  _Changed in Synapse 1.116.0: Added password\_path_

Example configuration:
```yaml
redis:
  enabled: true
  host: localhost
  port: 6379
  password_path: <path_to_the_password_file>
  # OR password: <secret_password>
  dbid: <dbid>
  #use_tls: True
  #certificate_file: <path_to_the_certificate_file>
  #private_key_file: <path_to_the_private_key_file>
  #ca_file: <path_to_the_ca_certificate_file>
```
---
## Individual worker configuration
These options configure an individual worker, in its worker configuration file.
They should be not be provided when configuring the main process.

Note also the configuration above for
[coordinating a cluster of workers](#coordinating-workers).

For guidance on setting up workers, see the [worker documentation](../../workers.md).

---
### `worker_app`

The type of worker. The currently available worker applications are listed
in [worker documentation](../../workers.md#available-worker-applications).

The most common worker is the
[`synapse.app.generic_worker`](../../workers.md#synapseappgeneric_worker).

Example configuration:
```yaml
worker_app: synapse.app.generic_worker
```
---
### `worker_name`

A unique name for the worker. The worker needs a name to be addressed in
further parameters and identification in log files. We strongly recommend
giving each worker a unique `worker_name`.

Example configuration:
```yaml
worker_name: generic_worker1
```
---
### `worker_listeners`

A worker can handle HTTP requests. To do so, a `worker_listeners` option
must be declared, in the same way as the [`listeners` option](#listeners)
in the shared config.

Workers declared in [`stream_writers`](#stream_writers) and [`instance_map`](#instance_map)
 will need to include a `replication` listener here, in order to accept internal HTTP
requests from other workers.

Example configuration:
```yaml
worker_listeners:
  - type: http
    port: 8083
    resources:
      - names: [client, federation]
```
Example configuration(#2, using UNIX sockets with a `replication` listener):
```yaml
worker_listeners:
  - type: http
    path: /run/synapse/worker_replication.sock
    resources:
      - names: [replication]
  - type: http
    path: /run/synapse/worker_public.sock
    resources:
      - names: [client, federation]
```
---
### `worker_manhole`

A worker may have a listener for [`manhole`](../../manhole.md).
It allows server administrators to access a Python shell on the worker.

Example configuration:
```yaml
worker_manhole: 9000
```

This is a short form for:
```yaml
worker_listeners:
  - port: 9000
    bind_addresses: ['127.0.0.1']
    type: manhole
```

It needs also an additional [`manhole_settings`](#manhole_settings) configuration.

---
### `worker_daemonize`

Specifies whether the worker should be started as a daemon process.
If Synapse is being managed by [systemd](../../systemd-with-workers/), this option
must be omitted or set to `false`.

Defaults to `false`.

Example configuration:
```yaml
worker_daemonize: true
```
---
### `worker_pid_file`

When running a worker as a daemon, we need a place to store the
[PID](https://en.wikipedia.org/wiki/Process_identifier) of the worker.
This option defines the location of that "pid file".

This option is required if `worker_daemonize` is `true` and ignored
otherwise. It has no default.

See also the [`pid_file` option](#pid_file) option for the main Synapse process.

Example configuration:
```yaml
worker_pid_file: DATADIR/generic_worker1.pid
```
---
### `worker_log_config`

This option specifies a yaml python logging config file as described
[here](https://docs.python.org/3/library/logging.config.html#configuration-dictionary-schema).
See also the [`log_config` option](#log_config) option for the main Synapse process.

Example configuration:
```yaml
worker_log_config: /etc/matrix-synapse/generic-worker-log.yaml
```
---
## Background Updates
Configuration settings related to background updates.

---
### `background_updates`

Background updates are database updates that are run in the background in batches.
The duration, minimum batch size, default batch size, whether to sleep between batches and if so, how long to
sleep can all be configured. This is helpful to speed up or slow down the updates.
This setting has the following sub-options:
* `background_update_duration_ms`: How long in milliseconds to run a batch of background updates for. Defaults to 100.
   Set a different time to change the default.
* `sleep_enabled`: Whether to sleep between updates. Defaults to true. Set to false to change the default.
* `sleep_duration_ms`: If sleeping between updates, how long in milliseconds to sleep for. Defaults to 1000.
   Set a duration to change the default.
* `min_batch_size`: Minimum size a batch of background updates can be. Must be greater than 0. Defaults to 1.
   Set a size to change the default.
* `default_batch_size`: The batch size to use for the first iteration of a new background update. The default is 100.
   Set a size to change the default.

Example configuration:
```yaml
background_updates:
    background_update_duration_ms: 500
    sleep_enabled: false
    sleep_duration_ms: 300
    min_batch_size: 10
    default_batch_size: 50
```
---
## Auto Accept Invites
Configuration settings related to automatically accepting invites.

---
### `auto_accept_invites`

Automatically accepting invites controls whether users are presented with an invite request or if they
are instead automatically joined to a room when receiving an invite. Set the `enabled` sub-option to true to
enable auto-accepting invites. Defaults to false.
This setting has the following sub-options:
* `enabled`: Whether to run the auto-accept invites logic. Defaults to false.
* `only_for_direct_messages`: Whether invites should be automatically accepted for all room types, or only
   for direct messages. Defaults to false.
* `only_from_local_users`: Whether to only automatically accept invites from users on this homeserver. Defaults to false.
* `worker_to_run_on`: Which worker to run this module on. This must match 
  the "worker_name". If not set or `null`, invites will be accepted on the
  main process.

NOTE: Care should be taken not to enable this setting if the `synapse_auto_accept_invite` module is enabled and installed.
The two modules will compete to perform the same task and may result in undesired behaviour. For example, multiple join
events could be generated from a single invite.

Example configuration:
```yaml
auto_accept_invites:
    enabled: true
    only_for_direct_messages: true
    only_from_local_users: true
    worker_to_run_on: "worker_1"
```

