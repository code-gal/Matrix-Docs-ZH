已翻译内容如下：

```yaml
### `forget_rooms_on_leave`

设置为 true 时，当用户离开房间时，无论是正常方式还是通过踢出或禁止，都会自动忘记该房间。默认为 false。

示例配置：
```yaml
forget_rooms_on_leave: false
```
---
### `exclude_rooms_from_sync`
一个房间列表，从同步响应中排除。这对于希望将用户分组到一个房间的服务器管理员很有用，而这些用户无法从他们的客户端看到该房间。

默认情况下，没有房间被排除。

示例配置：
```yaml
exclude_rooms_from_sync:
    - "!foo:example.com"
```
---
## Opentracing
与 Opentracing 支持相关的配置选项。

---
### `opentracing`

这些设置启用和配置 opentracing，它实现了分布式跟踪。这使您能够观察横跨服务器的事件因果链，包括请求、键查找等，跨所有运行 synapse 或其他支持 opentracing 的服务（特别是那些使用 Jaeger 实现的服务）。

子选项包括：
* `enabled`：是否启用跟踪。设置为 true 以启用。默认禁用。
* `homeserver_whitelist`：我们希望发送和接收跨度上下文和跨度凭据的 homeservers 列表。
   请参阅 [这里](../../opentracing.md) 获取更多信息。
   这是一个正则表达式列表，将匹配 homeserver 的 `server_name`。
   默认情况下，它是空的，因此没有匹配到的服务器。
* `force_tracing_for_users`：# 一个 matrix 用户 ID 列表，这些用户的请求将始终被跟踪，
   即使跟踪系统因概率采样而可能丢弃该跟踪。
   默认情况下，该列表为空。
* `jaeger_config`：Jaeger 可配置以不同的速率采样跟踪。
   提供的所有 Jaeger 配置选项均可在此处设置。Jaeger 的配置主要与跟踪采样相关，可在 [此处](https://www.jaegertracing.io/docs/latest/sampling/) 查阅文档。

示例配置：
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
## 协调工作进程
与主配置文件中的工作进程相关的配置选项（通常称为 `homeserver.yaml`）。
可以通过运行多个称为 _workers_ 的 Synapse 进程水平扩展 Synapse 部署。
传入的请求在 workers 之间分配以处理较高负载。一些 workers 有特权，可以接受来自其他 workers 的请求。

因此，worker 配置分为两部分。

1. 第一部分（在本手册的这一节中）定义了可分担的任务委托给特权工作进程。这允许无特权工作进程代表其请求特权工作进程执行操作。
1. [第二部分](#individual-worker-configuration)
   控制个别工作进程的独立行为。

有关设置 workers 的指导，请参阅 [worker 文档](../../workers.md)。

---
### `worker_replication_secret`

用于通过主进程的复制 APIs 认证来自 work

ers 的 HTTP 请求的共享秘密。

默认情况下，此值被省略（相当于 `null`），这意味着 workers 和主进程之间的流量未经过认证。

示例配置：
```yaml
worker_replication_secret: "secret_secret"
```
---
### `start_pushers`

如果使用 [`pusher_instances`](#pusher_instances) 并配合 [`generic_workers`](../../workers.md#synapseappgeneric_worker)，则无需设置。

控制主进程上推送通知的发送。如果使用 [pusher worker](../../workers.md#synapseapppusher)，则设置为 `false`。默认为 `true`。

示例配置：
```yaml
start_pushers: false
```
---
### `pusher_instances`

可以通过运行 [`generic_worker`](../../workers.md#synapseappgeneric_worker) 并将其 [`worker_name`](#worker_name) 添加到 `pusher_instances` 映射中来缩放处理向 [sygnal](https://github.com/matrix-org/sygnal) 发送推送通知和电子邮件的进程。这样做会将此功能的处理从主进程中移除。可以将多个工作进程添加到此映射中，在这种情况下，工作会在它们之间平衡。更改此选项后，请确保主进程和所有推送工作进程重新启动。

单个工作进程的示例配置：
```yaml
pusher_instances:
  - pusher_worker1
```
多个工作进程的示例配置：
```yaml
pusher_instances:
  - pusher_worker1
  - pusher_worker2
```

---
### `send_federation`

如果使用 [`federation_sender_instances`](#federation_sender_instances) 配合 [`generic_workers`](../../workers.md#synapseappgeneric_worker)，则无需设置。

控制主进程上出站联合事务的发送。
如果使用 [联合发送者工作进程](../../workers.md#synapseappfederation_sender)，则设置为 `false`。
默认为 `true`。

示例配置：
```yaml
send_federation: false
```
---
### `federation_sender_instances`

通过运行 [`generic_worker`](../../workers.md#synapseappgeneric_worker) 并将其 [`worker_name`](#worker_name) 添加到 `federation_sender_instances` 映射中，可以缩放处理发送出站联合请求的进程。这样做将从主进程中移除此功能的处理。可以将多个工作进程添加到此映射中，在这种情况下，工作会在它们之间平衡。

负载均衡的工作原理是任何出站联合请求将根据目标服务器名称的哈希分配给联合发送者工作进程。这意味着所有发送到相同目的地的请求将由同一工作进程实例处理。多个 `federation_sender_instances` 有助于在多个 服务器之间进行联合。

此配置设置必须在处理联合发送的所有工作进程之间共享，如果更改，所有联合发送者工作进程必须同时停止然后启动，以确保所有实例运行相同的配置（否则事件可能会丢失）。

单个工作进程的示例配置：
```yaml
federation_sender_instances:
  - federation_sender1
```
多个工作进程的示例配置：
```yaml
federation_sender_instances:
  - federation_sender1
  - federation_sender2
```
---
### `instance_map`

使用 workers 时，这应为从 [`worker_name`](#worker_name) 到工作进程的 HTTP 复制监听器（如果已配置）以及主进程的映射。在 [`stream_writers`](../../workers.md#stream-writers) 和 [`outbound_federation_restricted_to`](#outbound_federation_restricted_to) 下声明的每个工作进程都需要一个 HTTP 复制监听器，并且该监听器应该包含在 `instance_map` 中。主进程也需要在 `instance_map` 上有一个条目，如果即使只有一个其他工作进程存在，也应列在 `main` 下。确保端口与为 `replication` 监听器声明的 `listener` 块内部的配置相匹配。

示例配置：
```yaml
instance_map:
  main:
    host: localhost
    port: 8030
  worker1:
    host: localhost
    port: 8034
```
使用 UNIX 套接字的示例配置(#2)：
```yaml
instance_map:
  main:
    path: /run/synapse/main_replication.sock
  worker1:
    path: /run/synapse/worker1_replication.sock
```
---
### `stream_writers`

实验性功能: 使用 workers 时，可以定义哪个 workers 应该处理写入数据流，例如事件持久性和打字通知。此处指定的任何 worker 必须也在 [`instance_map`](#instance_map) 中。

可用数据流的列表请参阅
[worker 文档](../../workers.md#stream-writers)。

示例配置：
```yaml
stream_writers:
  events: worker1
  typing: worker1
```
---
### `outbound_federation_restricted_to`

使用 workers 时，可以限制出站联合流量只能通过特定的一组 workers。此处指定的任何 worker 必须也在 [`instance_map`](#instance_map) 中。
必须也配置 [`worker_replication_secret`](#worker_replication_secret) 以授权工人间的通讯。

```yaml
outbound_federation_restricted_to:
  - federation_sender1
  - federation_sender2
```

更多信息请参阅 [worker 文档](../../workers.md#restrict-outbound-federation-traffic-to-a-specific-set-of-workers)。

_在 Synapse 1.89.0 中新增。_

---
### `run_background_tasks_on`

用于运行后台任务（例如清理过期数据）的 [worker](../../workers.md#background-tasks)。如果未提供，则默认为主进程。

示例配置：
```yaml
run_background_tasks_on: worker1
```
---
### `update_user_directory_from_worker`

用于更新用户目录的 [worker](../../workers.md#updating-the-user-directory)。如果未提供，则默认为主进程。

示例配置：
```yaml
update_user_directory_from_worker: worker1
```

_在 Synapse 1.59.0 中新增。_

---
### `notify_appservices_from_worker`

用于向应用服务发送输出流量的 [worker](../../workers.md#notifying-application-services)。如果未提供，则默认为主进程。

示例配置：
```yaml
notify_appservices_from_worker: worker1
```

_在 Synapse 1.59.0 中新增。_

---
### `media_instance_running_background_jobs`

用于为媒体存储库运行后台任务的 [worker](../../workers.md#synapseappmedia_repository)。如果运行多个媒体存储库，则必须配置一个实例来运行后台任务。如果未提供，则默认为主进程或单独的 `media_repository` 工作进程。

示例配置：
```yaml
media_instance_running_background_jobs: worker1
```

_在 Synapse 1.16.0 中新增。_

---
### `redis`

使用 workers 时的 Redis 配置。使用 workers 时 *必须* 启用此设置。
此设置具有以下子选项：
* `enabled`：是否使用 Redis 支持。默认不启用。
* `host` 和 `port`：可选的主机和端口用于连接 redis。默认为 localhost 和 6379
* `path`：本地 Unix 套接字文件的完整路径。**如果使用此选项，`host` 和 `port` 将被忽略。** 默认为 `/tmp/redis.sock'
* `password`：可选的密码，如果在 Redis 实例上配置。
* `password_path`：`password` 的替代选项，从外部文件读取密码。文件应为纯文本文件，仅包含密码。Synapse 会在启动时从给定文件读入密码一次。
* `dbid`：可选的 redis dbid，如果需要连接到特定的 redis 逻辑 db。
* `use_tls`：是否使用 tls 连接。默认不启用。
* `certificate_file`：可选的证书文件路径
* `private_key_file`：可选的私钥文件路径
* `ca_file`：可选的 CA 证书文件路径。使用此选项或：
* `ca_path`：包含 CA 证书文件的文件夹路径

  _在 Synapse 1.78.0 中增加。_

  _在 Synapse 1.84.0 中更改：增加 use_tls, certificate_file, private_key_file, ca_file 和 ca_path 属性_

  _在 Synapse 1.85.0 中更改：增加 path 选项以使用本地 Unix 套接字_

  _在 Synapse 1.116.0 中更改：增加 password_path_

示例配置：
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
## 个别工作进程配置
这些选项在其工作进程配置文件中配置个别工作进程。
配置主进程时不应提供它们。

还请注意上面关于 [协调 workers 集群](#coordinating-workers) 的配置。

有关设置 work

ers 的指导，请参阅 [worker 文档](../../workers.md)。

---
### `worker_app`

工作进程的类型。目前可用的工作进程应用程序在 [worker 文档](../../workers.md#available-worker-applications) 中列出。

最常见的工作进程是
[`synapse.app.generic_worker`](../../workers.md#synapseappgeneric_worker)。

示例配置：
```yaml
worker_app: synapse.app.generic_worker
```
---
### `worker_name`

worker 的唯一名称。worker 需要一个名称，以便在进一步的参数和日志文件中识别。我们强烈建议给每个 worker 一个唯一的 `worker_name`。

示例配置：
```yaml
worker_name: generic_worker1
```
---
### `worker_listeners`

worker 可以处理 HTTP 请求。为此，必须声明一个 `worker_listeners` 选项，与共享配置中的 [`listeners` 选项](#listeners) 相同。

在 [`stream_writers`](#stream_writers) 和 [`instance_map`](#instance_map) 中声明的 workers 需要在此处包含一个 `replication` 监听器，以便接受来自其他 workers 的内部 HTTP 请求。

示例配置：
```yaml
worker_listeners:
  - type: http
    port: 8083
    resources:
      - names: [client, federation]
```
使用 UNIX 套接字和 `replication` 监听器的示例配置(#2)：
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

worker 可能有一个用于 [`manhole`](../../manhole.md) 的监听器。
这允许服务器管理员访问 worker 上的 Python shell。

示例配置：
```yaml
worker_manhole: 9000
```

这是一个简短形式：
```yaml
worker_listeners:
  - port: 9000
    bind_addresses: ['127.0.0.1']
    type: manhole
```

它还需要额外的 [`manhole_settings`](#manhole_settings) 配置。

---
### `worker_daemonize`

指定是否将 worker 作为守护进程启动。
如果 Synapse 由 [systemd](../../systemd-with-workers/) 管理，则必须省略此选项或将其设置为 `false`。

默认值为 `false`。

示例配置：
```yaml
worker_daemonize: true
```
---
### `worker_pid_file`

以守护进程运行 worker 时，我们需要一个地方存储 worker 的 [PID](https://en.wikipedia.org/wiki/Process_identifier)。
此选项定义了那个 "pid 文件" 的位置。

如果 `worker_daemonize` 为 `true`，则此选项为必需，否则忽略。没有预设值。

另请参阅主 Synapse 进程的 [`pid_file` 选项](#pid_file)。

示例配置：
```yaml
worker_pid_file: DATADIR/generic_worker1.pid
```
---
### `worker_log_config`

此选项指定了一个 yaml python 日志配置文件，如 [here](https://docs.python.org/3/library/logging.config.html#configuration-dictionary-schema) 中描述的。
另请参阅主 Synapse 进程的 [`log_config` 选项](#log_config)。

示例配置：
```yaml
worker_log_config: /etc/matrix-synapse/generic-worker-log.yaml
```
---
## 后台更新
与后台更新相关的配置设置。

---
### `background_updates`

后台更新是在后台批量运行的数据库更新。
可以配置运行更新的持续时间、最小批量大小、默认批量大小、是否在批次之间停顿以及如果停顿，停多长时间。这有助于加速或减慢更新。
此设置具有以下子选项：
* `background_update_duration_ms`：运行后台更新一批所需的毫秒数。默认为 100。
   设置不同的时间以更改默认值。
* `sleep_enabled`：是否在更新之间停顿。默认为 true。设置为 false 以更改默认值。
* `sleep_duration_ms`：如果在更新之间停顿，停顿多长时间（毫秒）。默认为 1000。
   设置持续时间以更改默认值。
* `min_batch_size`：后台更新批次的最小大小。必须大于 0。默认为 1。
   设置大小以更改默认值。
* `default_batch_size`：新后台更新首次迭代使用的批量大小。默认值为 100。
   设置大小以更改默认值。

示例配置：
```yaml
background_updates:
    background_update_duration_ms: 500
    sleep_enabled: false
    sleep_duration_ms: 300
    min_batch_size: 10
    default_batch_size: 50
```
---
## 自动接受邀请
与自动接受邀请相关的配置设置。

---
### `auto_accept_invites`

自动接受邀请控制用户是否会收到邀请请求，或在收到邀请时是否自动加入房间。将 `enabled` 子选项设为 true 以启用自动接受邀请。默认为 false。
此设置具有以下子选项：
* `enabled`：是否运行自动接受邀请逻辑。默认为 false。
* `only_for_direct_messages`：是否应为所有房间类型自动接受邀请，还是仅限于直接消息。默认为 false。
* `only_from_local_users`：是否仅自动接受来自本地 homeserver 的用户邀请。默认为 false。
* `worker_to_run_on`：运行此模块的工作进程。必须匹配 "worker_name"。如果未设置或为 `null`，

则在主进程上接受邀请。

注意：不要在启用并安装 `synapse_auto_accept_invite` 模块时启用此设置，以避免竞争执行相同任务，并可能导致不良行为。例如，可能从单个邀请生成多个加入事件。

示例配置：
```yaml
auto_accept_invites:
    enabled: true
    only_for_direct_messages: true
    only_from_local_users: true
    worker_to_run_on: "worker_1"
```
```