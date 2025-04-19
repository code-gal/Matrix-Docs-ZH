# 配置 Synapse

本文档旨在指导您进行 Synapse 配置。可以通过这里记录的众多配置设置来修改 Synapse 实例的行为——包括每个配置项的说明、默认值、如何更改默认值及该设置所影响的行为。此外，每个设置还包括示例配置。如果您不想花太多时间考虑选项，生成的配置会为所有值设置合理的默认值。但请注意，数据库默认是 SQLite，这不推荐用于生产环境。您可以在[这里](../../setup/installation.md#using-postgresql)阅读更多关于此主题的内容。

## 配置惯例

可以使用数字后接字母的形式设置需要 时间 时长的配置选项。字母含义如下：

* `s` = 秒
* `m` = 分钟
* `h` = 小时
* `d` = 天
* `w` = 星期
* `y` = 年

例如，设置 `redaction_retention_period: 5m` 会在五分钟后从数据库移除被删减的消息，而不是五个月。

另外，涉及大小的配置选项使用以下后缀：

* `K` = KiB，或 1024 字节
* `M` = MiB，或 1,048,576 字节
* `G` = GiB，或 1,073,741,824 字节
* `T` = TiB，或 1,099,511,627,776 字节

例如，设置 `max_avatar_size: 10M` 意味着 Synapse 不会接受大于 10,485,760 字节的用户头像文件。

## 配置验证

可以使用以下命令验证配置文件：
```bash
python -m synapse.config read <config key to print> -c <path to config>
```

要验证整个文件，请省略 `read <config key to print>`：
```bash
python -m synapse.config -c <path to config>
```

查看其他选项设置，请查阅帮助参考：
```bash
python -m synapse.config --help
```

### YAML
配置文件是一个 [YAML](https://yaml.org/) 文件，这意味着如果您希望配置文件被正确读取，必须遵循某些语法规则。几个需要了解的知识点：
* 配置中设置前的 `#` 将注释掉该设置，并应用默认值（如果可用），或者 Synapse 将忽略该设置。因此，在下面的示例 #1 中，配置将被读取并应用，而示例 #2 中将不会读取该设置，并将应用默认值。

   示例 #1：
   ```yaml
   pid_file: DATADIR/homeserver.pid
   ```
   示例 #2：
   ```yaml
   #pid_file: DATADIR/homeserver.pid
   ```
* 缩进很重要！设置前的缩进会决定某个设置是作为另一个设置的一部分读取，还是单独考虑。因此，在示例 #1 中，`enabled` 设置被读取为 `presence` 设置的子选项，并将正确应用。

  然而，示例 #2 中 `enabled` 设置前缺少缩进，意味着在读取配置时，Synapse 会将 `presence` 和 `enabled` 视为不同的设置。在这种情况下，`presence` 没有值，因此应用默认值，而 `enabled` 是一个 Synapse 不识别的选项，因此被忽略。

  示例 #1：
  ```yaml
  presence:
    enabled: false
  ```
  示例 #2：
  ```yaml
  presence:
  enabled: false
  ```
  
  在本手册中，所有顶级设置（无缩进）都在其部分开头标识（即“### `example_setting`”），如果有子选项，则在部分正文中标识和列出。此外，每个设置都提供了用法示例，并显示了正确的缩进。

## 模块

服务器管理员可以通过外部模块扩展 Synapse 的功能。

参见[这里](../../modules/index.md)以获取有关如何配置或创建 Synapse 自定义模块的更多文档。

---

### `modules`

使用 `module` 子选项在此选项下添加模块以扩展功能。然后，`module` 设置有一个子选项 `config`，用于定义 `module` 的一些配置。

默认值为无。

示例配置：
```yaml
modules:
  - module: my_super_module.MySuperClass
    config:
      do_thing: true
  - module: my_other_super_module.SomeClass
    config: {}
```
---
## 服务器

定义您的 homeserver 名称和其他基本选项。

---

### `server_name`

设置服务器的公共域名。

`server_name` 名称将出现在您服务器上创建的用户名和房间地址结尾。例如，如果 `server_name` 是 example.com，您服务器上的用户名将采用 `@user:example.com` 格式。

大多数情况下，您应该避免使用 matrix 特定子域名例如 matrix.example.com 或 synapse.example.com 作为 `server_name`，原因与不使用 user@email.example.com 作为邮箱地址相同。参见[这里](../../delegate.md)以获取有关如何在子域上托管 Synapse 的信息，同时保持干净的 `server_name`。

`server_name` 不能在以后更改，因此在启动 Synapse 之前正确配置这一点非常重要。它应该全部为小写，并可以包含显式端口。

此选项没有默认值。

示例配置 #1：
```yaml
server_name: matrix.org
```
示例配置 #2：
```yaml
server_name: localhost:8080
```
---
### `pid_file`

当 Synapse 作为守护进程运行时，用于存储 pid 的文件。默认值为无。

示例配置：
```yaml
pid_file: DATADIR/homeserver.pid
```
---
### `web_client_location`

重定向到的 web 客户端的绝对 URL。默认值为无。

示例配置：
```yaml
web_client_location: https://riot.example.com/
```
---
### `public_baseurl`

客户端访问此家庭服务器使用的公共基本 URL（不包括 _matrix/...）。这是用户可能在其客户端的“自定义家庭服务器 URL”字段中输入的 URL。如果您在反向代理中使用 Synapse，则应使用通过代理访问 Synapse 的 URL。否则，应使用访问 Synapse 的客户端 HTTP 监听器的 URL（见下文 ['listeners'](#listeners)）。

默认值为 `https://<server_name>/`。

示例配置：
```yaml
public_baseurl: https://example.com/
```
---
### `serve_server_wellknown`

默认情况下，其他服务器将尝试在端口 8448 上访问我们的服务器，这在某些环境中可能很不方便。

如果 `https://<server_name>/` 从端口 443 路由到 Synapse，则此选项配置 Synapse 在 `https://<server_name>/.well-known/matrix/server` 上提供一个文件。这将告知其他服务器将流量发送到端口 443。

此选项当前默认为 false。

有关更多信息，请查看[传入的联邦流量的委派](../../delegate.md)。

示例配置：
```yaml
serve_server_wellknown: true
```
---
### `extra_well_known_client_content`

此选项允许服务器运行者在[面向客户端的 `.well-known` 响应](https://spec.matrix.org/latest/client-server-api/#well-known-uri)中添加任意键值对。请注意，必须提供 `public_baseurl` 配置选项以便 Synapse 能够在 `/.well-known/matrix/client` 服务响应。

如果提供此选项，则会将给定的 yaml 解析为 json，并在 `/.well-known/matrix/client` 端点上与标准属性一起提供。

*添加于 Synapse 1.62.0。*

示例配置：
```yaml
extra_well_known_client_content :
  option1: value1
  option2: value2
```
---
### `soft_file_limit`

设置 Synapse 可以使用的文件描述符的软限制。零表示 Synapse 应将软限制设置为硬限制。默认值为 0。

示例配置：
```yaml
soft_file_limit: 3
```
---
### `presence`

存在感跟踪允许用户查看其他本地和远程用户的状态（例如在线/离线）。将 `enabled` 子选项设置为 false 可以禁用此家庭服务器上的存在感跟踪。默认值为 true。此选项取代了之前的顶级 'use_presence' 选项。

示例配置：
```yaml
presence:
  enabled: false
  include_offline_users_on_sync: false
```

`enabled` 也可以设置为特殊的 "untracked" 值，忽略通过客户端和联邦接收的更新，同时仍然接受来自[模块 API](../../modules/index.md)的更新。

*“untracked”选项添加于 Synapse 1.96.0。*

当客户端执行初始或 `full_state` 同步时，离线用户的存在感结果默认不包含。如果将 `include_offline_users_on_sync` 设置为 `true`，则始终在结果中包含离线用户。默认值为 false。

---
### `require_auth_for_profile_requests`

是否需要认证才能通过客户端 API 获取其他用户的个人资料数据（头像、显示名称）。默认值为 false。请注意，个人资料数据通过联邦 API 也可用，除非将 `allow_profile_lookup_over_federation` 设置为 false。

示例配置：
```yaml
require_auth_for_profile_requests: true
```
---
### `limit_profile_requests_to_users_who_share_rooms`

使用此选项要求用户和另一用户共享房间才能获取其个人资料信息。仅对客户端-服务器请求进行检查。来自其他服务器的个人资料请求应由请求服务器进行检查。默认值为 false。

示例配置：
```yaml
limit_profile_requests_to_users_who_share_rooms: true
```
---
### `include_profile_data_on_invite`

使用此选项以防止在用户加入之前其个人资料数据被获取并显示在房间中。默认情况下，无论上述两个设置的值如何，以及用户是否共享服务器，用户的个人资料数据都会被包含在邀请事件中。默认值为 true。

示例配置：
```yaml
include_profile_data_on_invite: false
```
---
### `allow_public_rooms_without_auth`

如果设置为 true，则无需认证即可通过客户端 API 访问服务器的公共房间目录，这意味着任何人都可以查询房间目录。默认值为 false。

示例配置：
```yaml
allow_public_rooms_without_auth: true
```
---
### `allow_public_rooms_over_federation`

如果设置为 true，则允许任何其他家庭服务器通过联邦获取服务器的公共房间目录。默认值为 false。

示例配置：
```yaml
allow_public_rooms_over_federation: true
```
---
### `default_room_version`

此服务器上新创建的房间的默认房间版本。

已知的房间版本列在[这里](https://spec.matrix.org/latest/rooms/#complete-list-of-room-versions)

例如，对于房间版本1，`default_room_version` 应设置为 "1"。

目前默认值为 ["10"](https://spec.matrix.org/v1.5/rooms/v10/)。

*更改于 Synapse 1.76:* 默认房间版本从[9](https://spec.matrix.org/v1.5/rooms/v9/) 增加到 [10](https://spec.matrix.org/v1.5/rooms/v10/)。

示例配置：
```yaml
default_room_version: "8"
```
---
### `gc_thresholds`

要传递给 `gc.set_threshold` 的垃圾回收阈值参数，如果定义的话。默认值为无。

示例配置：
```yaml
gc_thresholds: [700, 10, 10]
```
---
### `gc_min_interval`

设置各代之间每次垃圾回收的最短时间（秒计），无论垃圾回收阈值如何。这确保我们不会太频繁地进行垃圾回收。`[1s, 10s, 30s]` 的值指示两次连续的第0代垃圾回收之间必须有一秒钟的间隔，以此类推。

默认值为 `[1s, 10s, 30s]`。

示例配置：
```yaml
gc_min_interval: [0.5s, 30s, 1m]
```
---
### `filter_timeline_limit`

设置在获取和同步操作中时间线返回事件的限制。默认值为100。-1 表示没有上限。

示例配置：
```yaml
filter_timeline_limit: 5000
```
---
### `block_non_admin_invites`

是否阻止本服务器上用户（本地服务器管理员发送的除外）接收的房间邀请。默认值为 false。

示例配置：
```yaml
block_non_admin_invites: true
```
---
### `enable_search`

如果设置为 false，新消息将不会被索引以供搜索，用户在搜索消息时会收到错误。默认值为 true。

示例配置：
```yaml
enable_search: false
```
---
### `ip_range_blacklist`

此选项阻止将外发请求发送到指定的被黑名单的 IP 地址 CIDR 范围。如果未指定此选项，则默认是私有 IP 地址范围（参见下面的示例）。

黑名单适用于联邦请求、身份服务器、推送服务器以及用于检查第三方邀请事件键合法性的请求。

（无论是否在此显式列出，0.0.0.0 和 :: 总是被列入黑名单，因为它们对应不可以路由的地址。）

此选项取代了 Synapse v1.25.0 中的 `federation_ip_range_blacklist`。

注意：使用 HTTP 代理时，此值将被忽略。

示例配置：
```yaml
ip_range_blacklist:
  - '127.0.0.0/8'
  - '10.0.0.0/8'
  - '172.16.0.0/12'
  - '192.168.0.0/16'
  - '100.64.0.0/10'
  - '192.0.0.0/24'
  - '169.254.0.0/16'
  - '192.88.99.0/24'
  - '198.18.0.0/15'
  - '192.0.2.0/24'
  - '198.51.100.0/24'
  - '203.0.113.0/24'
  - '224.0.0.0/4'
  - '::1/128'
  - 'fe80::/10'
  - 'fc00::/7'
  - '2001:db8::/32'
  - 'ff00::/8'
  - 'fec0::/10'
```
---
### `ip_range_whitelist`

允许用于联邦、身份服务器、推送服务器以及检查第三方邀请事件键合法性的 IP 地址 CIDR 范围列表。这对于指定广泛黑名单目标 IP 范围的例外情况非常有用，例如仅在您的网络中可见的推送服务器的通信。

此白名单会覆盖 `ip_range_blacklist`，默认值为空列表。

示例配置：
```yaml
ip_range_whitelist:
   - '192.168.1.1'
```
---
### `listeners`

Synapse 应该监听的端口列表、其用途及其配置。

每个监听器的子选项包括：

* `port`: 要绑定的 TCP 端口。

* `tag`: 日志名称中的端口别名。如设置，日志中记录的是标签而不是端口。默认值为 `None`，为可选项，仅对 `type: http` 的监听器有效。参见文档[请求日志格式](../administration/request_log.md)。

* `bind_addresses`: 要监听的本地地址列表。默认值是 “所有本地接口”。

* `type`: 监听器类型。通常为 `http`，但其他有效选项包括：

   * `manhole`: （查看文档[这里](../../manhole.md)），

   * `metrics`: （查看文档[这里](../../metrics-howto.md)），

* `tls`: 设置为 true 以启用此监听器的 TLS。将使用 tls_private_key_path / tls_certificate_path中指定的 TLS 密钥/证书。

* `x_forwarded`: 仅对“http”监听器有效。设置为 true 以使用 X-Forwarded-For 头作为客户端 IP。当 Synapse 位于[反向代理](../../reverse_proxy.md)之后时很有用。

* `request_id_header`: 从每个传入请求中提取的头部，用于产生请求 ID。请求 ID 用于[日志](../administration/request_log.md#request-log-format)和跟踪以关联和匹配请求。在未设置时，Synapse 将自动生成顺序请求 ID。当 Synapse 位于[反向代理](../../reverse_proxy.md)之后时，此选项很有用。

   *添加于 Synapse 1.68.0。*

* `resources`: 仅对“http”监听器有效。在此端口上托管的资源列表。每个资源的子选项是：

   * `names`: HTTP 资源名称的列表。有效资源名称列表参见下文。

   * `compress`: 设置为 true 以启用 HTTP 正文的 gzip 压缩。此项目前仅支持 `client`、`consent`、`metrics` 和 `federation` 资源。

* `additional_resources`: 仅对“http”监听器有效。通过动态模块加载的附加端点的地图。

Unix 套接字支持（*添加于 Synapse 1.89.0*）：
* `path`: Unix 套接字的路径和文件名。确保它位于具有读写权限的目录中，并且已经存在（不会创建目录）。默认值为 `None`。
  * **注意**：在相同的 `监听器` 上同时使用 `path` 和 `port` 选项不兼容。
  * 使用 Unix 套接字时，`x_forwarded` 选项默认为 true，可以省略。
  * 其他与 UNIX 套接字一起使用无意义的选项，如 `bind_addresses` 和 `tls`，将被忽略，可以移除。
* `mode`: 设置 UNIX 套接字的文件权限。默认值为 `666`
* **注意：** 必须设定为 `type: http` （不支持 `metrics` 和 `manhole`)。也要确保在 `resources` -> `names` 中不包括 `metrics`。

有效资源名称是：

* `client`: 客户端-服务器 API (/_matrix/client)。也意味着 `media` 和 `static`。如果配置为主进程，也意味着 Synapse 管理员 API (/_synapse/admin)。

* `consent`: 用户同意表单 (/_matrix/consent)。查看[这里](../../consent_tracking.md)了解更多。

* `federation`: 服务器-服务器 API (/_matrix/federation)。也意味着 `media`、`keys`、`openid`

* `keys`: 键发现 API (/_matrix/key)。

* `media`: 媒体 API (/_matrix/media)。

* `metrics`: 度量接口。查看[这里](../../metrics-howto.md)。（不兼容 Unix 套接字）

* `openid`: OpenID 认证。查看[这里](../../openid.md)。

* `replication`: HTTP 复制 API (/_synapse/replication)。查看[这里](../../workers.md)。

* `static`: 在 synapse/static 下的静态资源 (/_matrix/static)。（主要用于“回退身份验证”。）

* `health`: [健康检查端点 ](../../reverse_proxy.md#health-check-endpoint)。该端点在默认情况下对所有其他资源处于活动状态，且不必单独激活。只有当您想在已指定端口上或者为[工作者](../../workers.md)和无监听器的容器（例如[应用程序服务](../../workers.md#notifying-application-services)）上专门使用健康检查端点才有用。

示例配置 #1：
```yaml
listeners:
  # 启用 TLS 的监听器：用于当 matrix 流量直接发送到 synapse 时。
  #
  # （请注意，您还需给 Synapse 配置一个 TLS 密钥和证书：见以下 TLS 部分。）
  #
  - port: 8448
    type: http
    tls: true
    resources:
      - names: [client, federation]
```
示例配置 #2：
```yaml
listeners:
  # 不安全的 HTTP 监听器：用于当 matrix 流量经过一个解开 TLS 的反向代理时。
  #
  # 如果您计划使用反向代理，请参阅
  # https://element-hq.github.io/synapse/latest/reverse_proxy.html。
  #
  - port: 8008
    tls: false
    type: http
    x_forwarded: true
    bind_addresses: ['::1', '127.0.0.1']

    resources:
      - names: [client, federation]
        compress: false

    # 附加资源示例：
    additional_resources:
      "/_matrix/my/custom/endpoint":
        module: my_module.CustomRequestHandler
```