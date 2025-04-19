Redis 支持在 v1.13.0 中添加，并在 v1.18.0 中成为推荐的方法。它取代了旧的直接 TCP 连接（自 v1.18.0 起已弃用）到主进程。使用 Redis，所有工作者和主进程连接到 Redis，而不是所有工作者连接到主进程，Redis 在进程之间中继复制命令。这可以显著节省主进程的 CPU，并且是即将进行的性能改进的前提条件。

要迁移到 Redis，请添加[`redis` 配置](./workers.md#shared-configuration)，并从主进程的配置中移除 TCP `replication` 监听器，以及从工作者配置中移除 `worker_replication_port`。请注意，仍然需要一个带有 `replication` 资源的 HTTP 监听器。