* `"main"`: 其他所有数据库表和序列。

所有数据库将最终包含用于跟踪数据库模式迁移和任何未完成的后台更新的附加表。Synapse将在启动时自动创建这些表，以便检查和/或进行数据库模式迁移。

要将现有的数据库配置（例如，所有表在一个数据库中）迁移到其他配置（例如，在一个数据库上存储“main”数据，另一个数据库存储“state”），请执行以下操作：

1. 备份现有数据库。事情可能会出错，而数据库损坏不是小事！
2. 确保所有未完成的数据库迁移已被应用且后台更新已运行。最简单的方法是使用 Synapse 安装程序中提供的 `update_synapse_database` 脚本。

   ```sh
   update_synapse_database --database-config homeserver.yaml --run-background-updates
   ```

3. 从一个数据库复制必要的表和序列到另一个数据库。与数据库迁移、模式、模式版本和后台更新相关的表**不**应该被复制。

   例如，假设您想从现有的数据库中分离出“state”数据存储，而该数据库目前包含所有数据存储。

   只需从现有数据库中复制“state”数据存储上述定义的表和序列到第二个数据库。正如上面所述，Synapse 启动时将在第二个数据库中创建附加的表。

4. 修改/创建 `homeserver.yaml` 中的 `databases` 选项，以匹配所需的数据库配置。
5. 启动 Synapse。检查它是否成功启动并且功能通常正常。
6. 删除第三步中复制的旧表。

在配置中只可以指定 `database` 或 `databases` 中的一个选项，而不能同时指定。

示例配置：

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
## 日志记录
与日志记录相关的配置选项。

---
### `log_config`

此选项指定一个 YAML 格式的 Python 日志配置文件，详细描述见
[这里](https://docs.python.org/3/library/logging.config.html#configuration-dictionary-schema)。

示例配置：
```yaml
log_config: "CONFDIR/SERVERNAME.log.config"
```
---
## 速率限制
与 Synapse 中速率限制相关的选项。

每个速率限制配置由两个参数组成：
   - `per_second`: 客户端每秒可以发送的请求数量。
   - `burst_count`: 客户端在被限制前可以发送的请求数量。
---
### `rc_message`


客户端消息的速率限制设置。

这是基于客户端使用的帐户限制发送消息的选项。默认值为：`per_second: 0.2`, `burst_count: 10`。

示例配置：
```yaml
rc_message:
  per_second: 0.5
  burst_count: 15
```
---
### `rc_registration`

此选项基于客户端的 IP 地址限制注册请求。
默认值为 `per_second: 0.17`, `burst_count: 3`。

示例配置：
```yaml
rc_registration:
  per_second: 0.15
  burst_count: 2
```
---