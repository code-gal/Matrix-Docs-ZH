有关如何正确设置配置文件和反向代理的信息，请参阅[worker文档](../workers.md)。以下是一个`generic_worker` worker配置文件的示例。
```yaml
{{#include workers/generic_worker.yaml}}
```

systemd自行管理守护进程化，因此确保没有配置文件设置`daemonize`或`worker_daemonize`。

所有workers的配置文件应位于`/etc/matrix-synapse/workers`中。如果您想使用不同的位置，请相应地编辑提供的`*.service`文件。

主进程不需要单独的配置文件。