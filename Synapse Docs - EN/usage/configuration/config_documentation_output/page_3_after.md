此选项和相关选项决定了服务器级别的消息保留策略。

房间管理员和版主可以使用 `m.room.retention` 状态事件为他们的房间定义保留期，服务器管理员可以通过设置 `allowed_lifetime_min` 和 `allowed_lifetime_max` 配置选项来限制此保留期。

如果启用了此功能，Synapse 将定期查找并清除超过房间最大保留期的事件。Synapse 还会过滤通过联邦接收到的事件，以便应已清除的事件被忽略而不再存储。

消息保留策略功能默认是禁用的。您可以在 [这里](../../message_retention_policies.md) 阅读更多关于此功能的信息。

此设置具有以下子选项：
* `default_policy`：默认保留策略。如果设置，Synapse 将其应用于缺少 `m.room.retention` 状态事件的房间。此选项通过与之关联的 `min_lifetime` 和 `max_lifetime` 子选项进一步指定。请注意，`min_lifetime` 的值不是很重要，因为 Synapse 尚未考虑到这一点。

* `allowed_lifetime_min` 和 `allowed_lifetime_max`：保留策略限制。如果设置，房间状态中包含 `m.room.retention` 事件，该事件包含超出这些界限的 `min_lifetime` 或 `max_lifetime`，Synapse 在执行清除任务时将把房间的策略限制在这些范围内。

* `purge_jobs` 和相关 `shortest_max_lifetime` 和 `longest_max_lifetime` 子选项：服务器管理员可以在 `purge_jobs` 部分定义后台清除过期事件的作业设置。

  如果未为此选项提供配置，将会设置一个作业来每天删除每个房间中的已过期事件。

  每个作业的配置定义了作业负责的消息生命周期范围。例如，如果 `shortest_max_lifetime` 是 '2d' 而 `longest_max_lifetime` 是 '3d'，则作业将负责清除房间状态中定义 `max_lifetime` 超过2天且小于或等于3天的事件。范围的最小值和最大值都是可选的，例如，一个作业没有 `shortest_max_lifetime` 而 `longest_max_lifetime` 为 '3d' 将处理所有保留策略中 `max_lifetime` 小于或等于三天的房间。

  xxxx每个作业配置的原因是某些房间可能有低 `max_lifetime` 的保留策略，在这种情况下，比其他房间更频繁地清除过时消息（例如每12小时）是有必要的，但又不希望通过遍历所有房间的作业来执行这种清理，因为这可能会给服务器带来沉重负担。

  如果配置了任何清除作业，强烈建议至少有一个作业没有设置 `shortest_max_lifetime` 或 `longest_max_lifetime`，或者一个作业没有 `shortest_max_lifetime` 和另一个没有设置 `longest_max_lifetime`。否则，即使设置了 `allowed_lifetime_min` 和 `allowed_lifetime_max`，某些房间可能被忽略，因为将房间策略限制在这些值是在从 Synapse 数据库中检索策略后（这是使用清除作业配置中指定的范围完成的）。

示例配置：
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

与 TLS 相关的选项。

---
### `tls_certificate_path`

此选项指定用于 TLS 的 PEM 编码的 X509 证书。从 Synapse 1.0 开始，证书需要是由认可证书颁发机构签署的、有效且可验证的证书。默认值为无。

确保使用包含完整证书链（包括任何中间证书）的 `.pem` 文件（例如，使用 certbot 时，使用 `fullchain.pem` 作为证书，而不是 `cert.pem`）。

示例配置：
```yaml
tls_certificate_path: "CONFDIR/SERVERNAME.tls.crt"
```

---
### `tls_private_key_path`

用于 TLS 的 PEM 编码私钥。默认值为无。

示例配置：
```yaml
tls_private_key_path: "CONFDIR/SERVERNAME.tls.key"
```

---
### `federation_verify_certificates`

是否验证外向联邦请求的 TLS 服务器证书。

默认值是 true。如需禁用证书验证，将选项设置为 false。

示例配置：
```yaml
federation_verify_certificates: false
```

---
### `federation_client_minimum_tls_version`

用于外向联邦请求的最低 TLS 版本。

默认设置为 `"1"`。可配置为 `"1"`、`"1.1"`、`"1.2"` 或 `"1.3"`。请注意，设置此值高于 `"1.2"` 将阻止与大多数公共 Matrix 网络进行联邦化：仅在完全私密的联邦设置下才配置为 `"1.3"`，并且可以确保 TLS 1.3 支持。

示例配置：
```yaml
federation_client_minimum_tls_version: "1.2"
```

---
### `federation_certificate_verification_whitelist`

跳过给定域白名单上的联邦证书验证。

此设置应仅在非常特殊的情况下使用，例如通过 Tor 隐藏服务进行联邦化等。对于家庭服务器的私有网络，您可能更希望使用私人 CA。

仅当 `federation_verify_certificates` 为 `true` 时有效。

示例配置：
```yaml
federation_certificate_verification_whitelist:
  - lon.example.com
  - "*.domain.com"
  - "*.onion"
```

---
### `federation_custom_ca_list`

用于联邦流量的自定义证书颁发机构列表。

此设置通常仅在家庭服务器的私有网络中使用。

请注意，此列表将替换操作环境提供的证书。证书必须为 PEM 格式。

示例配置：
```yaml
federation_custom_ca_list:
  - myCA1.pem
  - myCA2.pem
  - myCA3.pem
```

---
## Federation

与联邦相关的选项。

---
### `federation_domain_whitelist`

将联邦限制为给定的域白名单。 
请注意，我们建议在可能的情况下尽早限制您的联邦监听器以限制入站联邦流量，而不是仅依靠此应用层限制。 如果未指定，则默认是白名单一切。

请注意：这不会阻止服务器加入不在白名单上的服务器所在的房间。因此，此选项实际上仅适用于建立“私有联邦”，其中一组服务器都相互白名单并具有相同的白名单。

示例配置：
```yaml
federation_domain_whitelist:
  - lon.example.com
  - nyc.example.com
  - syd.example.com
```

---
### `federation_whitelist_endpoint_enabled`

启用用于获取联邦白名单配置的端点。

请求方法和路径为 `GET /_synapse/client/v1/config/federation_whitelist`，响应格式为：

```json
{
    "whitelist_enabled": true,  // 是否强制执行联邦白名单
    "whitelist": [  // 白名单允许哪些服务器名称
        "example.com"
    ]
}
```

如果 `whitelist_enabled` 为 `false`，则服务器允许与所有其他服务器联邦。

此端点需要身份验证。

示例配置：
```yaml
federation_whitelist_endpoint_enabled: true
```

---
### `federation_metrics_domains`

报告向给定域发送和从其接收 PDU 的 Prometheus 指标的年龄。这可用于大致了解入站和出站联邦上的“延迟”，但请注意，任何延迟都可能由于两端或中间网络的问题。

默认情况下，没有域会以这种方式进行监控。

示例配置：
```yaml
federation_metrics_domains:
  - matrix.org
  - example.com
```

---
### `allow_profile_lookup_over_federation`

设置为 false 禁止通过联邦进行资料查找。默认情况下，联邦 API 允许其他家庭服务器获取此家庭服务器上任何用户的资料数据。

示例配置：
```yaml
allow_profile_lookup_over_federation: false
```

---
### `allow_device_name_lookup_over_federation`

将此选项设置为 true 以允许通过联邦查找设备显示名称。默认情况下，联邦 API 阻止其他家庭服务器获取此家庭服务器上任何用户设备的显示名称。

示例配置：
```yaml
allow_device_name_lookup_over_federation: true
```

---
### `federation`

联邦部分定义了一些与联邦相关的子选项。

以下选项与配置一个请求的超时和重试逻辑有关，与其他请求无关。 
当某事或某人将等待请求得到答复时，使用短重试算法，而长重试则用于后台发生的请求，
比如发送联邦事务。

* `client_timeout`：联邦请求的超时。默认值为 60 秒。
* `max_short_retry_delay`：用于短重试算法的最大延迟。默认值为 2 秒。
* `max_long_retry_delay`：用于长重试算法的最大延迟。默认值为 60 秒。
* `max_short_retries`：短重试算法的最大重试次数。默认值为 3 次尝试。
* `max_long_retries`：长重试算法的最大重试次数。默认值为 10 次尝试。

以下选项控制与特定家庭服务器目标通信时的重试逻辑。与以前的配置选项不同，这些值适用于给定目标的所有请求，并且退避状态存储在数据库中。

* `destination_min_retry_interval`：首次请求失败后的初始退避。默认是 10 分钟。
* `destination_retry_multiplier`：每次失败后乘以退避的倍率。默认是 2。
* `destination_max_retry_interval`：退避的上限。默认是一周。

示例配置：
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

与缓存相关的选项。

---
### `event_cache_size`

要在内存中缓存的事件数。默认值是 10K。就像其他缓存一样，这受 `caches.global_factor`（见下文）影响。

例如，默认值是 10K，全球因子默认是 0.5。

由于 10K * 0.5 是 5K，因此事件缓存大小将是 5K。

受此配置影响的缓存名为“*getEvent*”。

请注意，此选项不是 `caches` 部分的一部分。

示例配置：
```yaml
event_cache_size: 15K
```

---
### `caches` 和相关值

缓存“因子”是可以应用于 Synapse 各个缓存以增加或减少可存储的最大条目数的乘数。

`caches` 可以通过以下子选项配置：

* `global_factor`：控制全局缓存因子，即如果没有为该缓存另行设置具体因子，则默认所有缓存的缓存因子。

  也可以通过 `SYNAPSE_CACHE_FACTOR` 环境变量设置。通过环境变量设置的优先级高于
通过配置文件设置。

  默认是 0.5，这将所有缓存的大小减半。

  请注意，改变此值也会影响 HTTP 连接池。

* `per_cache_factors`：指定缓存名与该个别缓存因子之间的字典。覆盖特定缓存的全局缓存因子。

   这些也可以通过由 `SYNAPSE_CACHE_FACTOR_` + 缓存名的大写字母和下划线组成的环境变量进行设置。通过环境变量设置的优先级高于通过配置文件设置。
   例如 `SYNAPSE_CACHE_FACTOR_GET_USERS_WHO_SHARE_ROOM_WITH_USER=2.0`

   一些缓存有 '*' 和其他非字母数字或下划线的字符。这些缓存可以去掉特殊字符来命名，也可以不去掉。例如，若要通过环境变量指定 `*stateGroupCache*` 的缓存因子，使用 `SYNAPSE_CACHE_FACTOR_STATEGROUPCACHE=2.0`。

* `expire_caches`：控制缓存条目在指定时间后是否被驱逐。默认是 true。设置为 false 禁用此功能。请注意，从不驱逐缓存可能导致过多的内存使用。

* `cache_entry_ttl`：如果 `expire_caches` 启用，此标志控制一个条目在没有被访问的情况下能在缓存中保留多久，然后被驱逐。默认是 30m。

* `sync_response_cache_duration`：控制/sync请求的结果在成功响应返回后缓存多长时间。较高的持续时间可以帮助具有间歇性连接的客户端，但代价是更多的内存使用。设置为零意味着不同步响应进行缓存。默认是 2m。

  *此项在 Synapse 1.62.0 中更改*：默认从0改为2m。

* `cache_autotuning` 及其子选项 `max_cache_memory_usage`、`target_cache_memory_usage` 和 `min_cache_ttl` 相互作用，维持在缓存内存使用和缓存条目可用性之间的平衡。您必须使用 [jemalloc](../administration/admin_faq.md#help-synapse-is-slow-and-eats-all-my-ramcpu) 才能利用此选项，并且必须指定以下列出的子选项的所有值以使此功能生效。请注意，如果未提供所有值，则该功能将无法工作并可能导致不稳定行为（例如过度清空缓存或异常）。请参阅 [配置常新增](#config-conventions) 以获取有关如何指定内存大小和缓存过期持续时间的信息。
     * `max_cache_memory_usage` 设置缓存可能使用的内存的上限，超出此内存前，条目会连续驱逐。直到内存使用降至以下设置所设定的 `target_cache_memory_usage` 或 `min_cache_ttl` 被达到。此选项没有默认值。
     * `target_cache_memory_usage` 设置缓存的目标内存使用量没有默认值。
     * `min_cache_ttl` 设置缓存条目不会被驱逐的限制，仅在超过 `max_cache_memory_usage` 时应用于缓存条目。此选项没有默认值。

示例配置：
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

### 重新加载缓存因子

缓存因子（即 `caches.global_factor` 和 `caches.per_cache_factors`）可以随时重新加载，通过向 Synapse 发送[`SIGHUP`](https://en.wikipedia.org/wiki/SIGHUP)信号实现，例如

```commandline
kill -HUP [PID_OF_SYNAPSE_PROCESS]
```

如果您运行多个工作进程，必须单独更新工作进程配置文件，并将此信号发送到每个工作进程。

如果您使用 [示例 systemd 服务](https://github.com/element-hq/synapse/blob/develop/contrib/systemd/matrix-synapse.service)文件，您可以通过使用 `systemctl reload matrix-synapse` 命令来发送 `SIGHUP` 信号。

---
## Database
与数据库设置相关的配置选项。

---
### `database`

`database` 设置定义了 Synapse 用于存储其所有数据的数据库。

相关子选项：

* `name`：此选项指定数据库引擎：`sqlite3`（用于 SQLite）或 `psycopg2`（用于 PostgreSQL）。如果未指定名称，Synapse 将默认为 SQLite。

* `txn_limit` 设置每个连接之前运行的最大事务数，然后重新连接。默认是 0，表示没有限制。

* `allow_unsafe_locale` 是 Postgres 特有的选项。在默认行为下，Synapse 将在 PostgreSQL 数据库设置为非 C 本地化时拒绝启动。您可以通过将 `allow_unsafe_locale` 设置为 true 来覆盖此行为（这*不建议*），请注意，这可能会损坏您的数据库。您可以在[这里](../../postgres.md#fixing-incorrect-collate-or-ctype)和[这里](https://wiki.postgresql.org/wiki/Locale_data_changes)找到更多信息。

* `args` 提供传递给数据库引擎的选项，但以 `cp_` 开头的选项除外，这些选项用于配置 Twisted 连接池。有关有效参数的参考，请参阅：
    * [sqlite](https://docs.python.org/3/library/sqlite3.html#sqlite3.connect)
    * [postgres](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-PARAMKEYWORDS)
    * [连接池](https://docs.twistedmatrix.com/en/stable/api/twisted.enterprise.adbapi.ConnectionPool.html#__init__)

有关使用 Synapse 与 Postgres 的更多信息，请参阅[这里](../../postgres.md)。

SQLite 示例配置：
```yaml
database:
  name: sqlite3
  args:
    database: /path/to/homeserver.db
```

Postgres 示例配置：
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

`databases` 选项允许指定某些数据库表和数据库主机详细信息之间的映射，将单个 Synapse 实例的负载分散到多个数据库后端。这通常被称为“数据库分片”。此选项仅支持 PostgreSQL 数据库后端。

**重要提示：** 这是一个支持的选项，但目前不由 Matrix.org Foundation 在生产中使用。谨慎使用并始终备份。

`databases` 是一个任意命名的数据库条目字典。每个条目等价于 `database` 服务器配置选项的值（见上文），并增加了一个 `data_stores` 键。`data_stores` 是一个字符串数组，指定应该存储在关联数据库后端条目中的数据存储（为一组表定义的标签）。

`data_stores` 目前定义的值有：

* `"state"`：与状态组相关的数据库将存储在此数据库中。

  具体来说，这意味着以下表：
  * `state_groups`
  * `state_group_edges`
  * `state_groups_state`

  以及以下序列：
  * `state_groups_seq_id`

* `"main"`：所有其他数据库表和序列。