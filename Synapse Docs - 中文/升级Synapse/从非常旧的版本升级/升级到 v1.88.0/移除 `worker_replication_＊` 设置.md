如之前在[升级到 v1.84.0](#upgrading-to-v1840)中提到的，此版本的 Synapse 中将移除以下已弃用的设置：

- [`worker_replication_host`](https://element-hq.github.io/synapse/v1.86/usage/configuration/config_documentation.html#worker_replication_host)
- [`worker_replication_http_port`](https://element-hq.github.io/synapse/v1.86/usage/configuration/config_documentation.html#worker_replication_http_port)
- [`worker_replication_http_tls`](https://element-hq.github.io/synapse/v1.86/usage/configuration/config_documentation.html#worker_replication_http_tls)

请确保您已迁移到在共享配置的 `instance_map` 中使用 `main`（或在必要时创建一个）。如果您有任何工作者，这是必需的；单进程（整体）安装的管理员无需执行任何操作。

有关示例，请参阅下面的[升级到 v1.84.0](#upgrading-to-v1840)。