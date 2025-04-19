在使用工作者时，

- `worker_replication_host`
- `worker_replication_http_port`
- `worker_replication_http_tls`

现在应从各个工作者的 YAML 配置中移除，主进程应改为添加到共享 YAML 配置的 `instance_map` 中，使用名称 `main`。

旧的 `worker_replication_*` 设置现在被视为已弃用，预计将在 Synapse v1.88.0 中移除。