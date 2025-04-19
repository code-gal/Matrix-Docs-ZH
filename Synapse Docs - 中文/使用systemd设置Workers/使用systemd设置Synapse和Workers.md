这是一个使用systemd管理Synapse的设置，包括支持管理workers。它为主服务器提供了一个`matrix-synapse`服务，以及一个`matrix-synapse-worker@`服务模板，用于您需要的任何workers。此外，为了将所需服务分组，它设置了一个`matrix-synapse.target`。

请参阅[system](https://github.com/element-hq/synapse/tree/develop/docs/systemd-with-workers/system/)文件夹以获取systemd单元文件。

[workers](https://github.com/element-hq/synapse/tree/develop/docs/systemd-with-workers/workers/)文件夹包含`generic_worker` worker的示例配置。