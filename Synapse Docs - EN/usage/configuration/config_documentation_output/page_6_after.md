使用中的用户抑制错误：真

```yaml
---
## 用户会话管理
---
### `session_lifetime`

用户登录后，会话保持有效的时间。

注意，此功能当前与访客登录不兼容。

还需注意，此时间是在登录时计算的：对于已登录的用户，变更不会追溯应用。

默认情况下，这是无限的。

配置示例：
```yaml
session_lifetime: 24h
```
---
### `refreshable_access_token_lifetime`

如果会话使用刷新令牌，访问令牌保持有效的时间。

有关刷新令牌的更多信息，请参阅[手册](Synapse%20Docs%20-%20EN/usage/configuration/user_authentication/refresh_tokens.md)。

注意，这仅适用于宣布支持刷新令牌的客户端。

还需注意，此时间是在登录和刷新时计算的：对于现有会话，变更不会应用，直到会话被刷新。

默认情况下，这是5分钟。

配置示例：
```yaml
refreshable_access_token_lifetime: 10m
```
---
### `refresh_token_lifetime`

刷新令牌保持有效的时间（前提是它未被交换为另一个令牌）。
此选项可用于自动注销不活跃的会话。
更多信息请参阅手册。

还需注意，此时间是在登录和刷新时计算的：对于现有会话，变更不会应用，直到会话被刷新。

默认情况下，这是无限的。

配置示例：
```yaml
refresh_token_lifetime: 24h
```
---
### `nonrefreshable_access_token_lifetime`

如果会话**不**使用刷新令牌，访问令牌保持有效的时间。

请注意，并非所有客户端都支持刷新令牌，因此将此设置为较短的值可能会对某些用户造成不便，他们将会频繁注销。

还需注意，此时间是在登录时计算的：对已登录用户的现有会话，变更不会追溯应用。

默认情况下，这是无限的。

配置示例：
```yaml
nonrefreshable_access_token_lifetime: 24h
```
---
### `ui_auth`

允许用户交互式认证会话活动的时间量。

默认值为0，意味着每次操作前均会查询用户凭证，但可以覆盖此设置以允许重复使用一次验证。这会削弱用户交互式认证过程提供的保护，因为允许多个（并且可能是不同的）操作使用相同验证会话。

对于可能“危险”的操作（包括停用账户、修改账户密码、添加3PID和生成额外登录令牌），此设置会被忽略。

使用`session_timeout`子选项更改允许凭证验证的时间。

配置示例：
```yaml
ui_auth:
    session_timeout: "15s"
```
---
### `login_via_existing_session`

Matrix支持使用现有会话为其他客户端生成登录令牌的功能。

默认情况下，Synapse禁用此功能，因为它具有安全隐患——恶意客户端可能使用该机制生成多个会话。

生成的令牌的有效持续时间可以通过`token_timeout`子选项进行配置。

启用此功能时，除非`require_ui_auth`子选项设置为`False`，否则需要用户交互式认证。

配置示例：
```yaml
login_via_existing_session:
    enabled: true
    require_ui_auth: false
    token_timeout: "5m"
```
---
## 指标
与指标相关的配置选项。

---
### `enable_metrics`

设置为true以启用性能指标的收集和渲染。
默认值为false。

配置示例：
```yaml
enable_metrics: true
```
---
### `sentry`

使用该选项启用Sentry集成。通过`dsn`设置提供由Sentry分配给您的DSN。

可以使用可选的`environment`字段指定环境。这可基于不同的环境进行日志维护，确保更好的组织和分析。

注意：尽管尝试确保日志不包含任何敏感信息，但无法保证这一点。通过启用此选项，Sentry服务器可能会因此接收到敏感信息，并可能通过不安全的通知渠道传播这些敏感信息（如果此类配置）。

配置示例：
```yaml
sentry:
    environment: "production"
    dsn: "..."
```
---
### `metrics_flags`

用于启用默认情况下不适合启用Prometheus指标的标志，可能是出于性能原因，或用处有限。当前唯一的选项是`known_servers`，它发布`synapse_federation_known_servers`，即此home服务器知道的服务器数量的标志(包括其本身)。在大型home服务器上可能会导致性能问题。

配置示例：
```yaml
metrics_flags:
    known_servers: true
```
---
### `report_stats`

是否报告home服务器使用统计。最初是在生成配置时设置的。设置此选项为true或false以更改当前行为。请参阅[报告home服务器使用统计](../administration/monitoring/reporting_homeserver_usage_statistics.md)以获取有关所报告数据的信息。

Synapse启动后5分钟会报告统计，然后每3小时报告一次。

配置示例：
```yaml
report_stats: true
```
---
### `report_stats_endpoint`

用于报告home服务器使用统计的端点。
默认为https://matrix.org/report-usage-stats/push

配置示例：
```yaml
report_stats_endpoint: https://example.com/report-usage-stats/push
```
---
## API配置
与客户端/服务器API有关的配置设置

---
### `room_prejoin_state`

此设置控制在接收房间邀请或回应敲门请求时与用户共享的状态。默认情况下，与用户共享以下状态事件：

- `m.room.join_rules`
- `m.room.canonical_alias`
- `m.room.avatar`
- `m.room.encryption`
- `m.room.name`
- `m.room.create`
- `m.room.topic`

要更改默认行为，请使用以下子选项：
* `disable_default_event_types`: 布尔值。设置为`true`以禁用上述默认设置。如果启用此选项，则仅共享`additional_event_types`中列出的事件类型。默认值为`false`。
* `additional_event_types`: 要在共享的事件中包含的额外状态事件的列表。默认情况下，此列表为空（因此仅共享默认事件类型）。

  此列表中的每个条目应该是一个字符串或两个字符串的列表。
  * 独立的字符串`t`代表所有类型为`t`的事件（即无状态键限制）。
  * 字符串对`[t, s]`代表类型为`t`且状态键为`s`的单个事件。相同类型的事件可以在两个条目中出现具有不同的状态键：在这种情况下，两个状态键都将包括在预加入状态中。

配置示例：
```yaml
room_prejoin_state:
   disable_default_event_types: false
   additional_event_types:
     # 共享类型为`org.example.custom.event.typeA`的所有事件
     - org.example.custom.event.typeA
     # 仅共享类型为`org.example.custom.event.typeB`且状态键为"foo"的事件
     - ["org.example.custom.event.typeB", "foo"]
     # 仅共享类型为`org.example.custom.event.typeC`且状态键为"bar"或"baz"的事件
     - ["org.example.custom.event.typeC", "bar"]
     - ["org.example.custom.event.typeC", "baz"]
```

*Synapse 1.74中的更改：*管理员可以根据状态键过滤预加入状态中的事件。

---
### `track_puppeted_user_ips`

我们记录用于访问API的客户端IP地址，出于各种原因，包括在“您登录的位置”对话框中显示给用户。

默认情况下，通过admin API代理其他用户时，客户端IP地址记录在创建访问令牌的用户（即管理员用户）名下，而**不是**代理的用户。

将此选项设置为true以便IP地址也记录在代理用户名下。（这也意味着代理用户将被视为“活跃”用户，用于每月活跃用户跟踪-请参见上方的`limit_usage_by_mau`等。）

配置示例：
```yaml
track_puppeted_user_ips: true
```
---
### `app_service_config_files`

要使用的应用程序服务配置文件的列表。

配置示例：
```yaml
app_service_config_files:
  - app_service_1.yaml
  - app_service_2.yaml
```
---
### `track_appservice_user_ips`

默认为false。设置为true以启用对应用程序服务IP地址的跟踪。
隐式启用应用程序服务用户的MAU跟踪。

配置示例：
```yaml
track_appservice_user_ips: true
```
---
### `use_appservice_legacy_authorization`

是否通过`access_token`查询参数发送应用程序服务访问令牌
符合Matrix规范的旧版本。默认为false。设置为true以启用通过查询参数发送访问令牌。

**启用此选项被视为不安全且不推荐。**

配置示例：
```yaml
use_appservice_legacy_authorization: true
```

---
### `macaroon_secret_key`

用于签名的密钥
- 访客用户访问令牌，
- 在SSO登录（OIDC或SAML2）期间使用的短期登录令牌和
- 用于取消接收邮件通知的令牌。

如果未指定此密钥，将使用`registration_shared_secret`，如果存在；
否则，将从签名密钥中派生一个秘密密钥。

配置示例：
```yaml
macaroon_secret_key: <PRIVATE STRING>
```
---
### `form_secret`

用于计算表单值HMAC的密钥，以防止值的伪造。必须指定以使用户同意
表单正常工作。

配置示例：
```yaml
form_secret: <PRIVATE STRING>
```
---
## 签名密钥
与签名密钥相关的配置选项

---
### `signing_key_path`

用于签署事件和联邦请求的签名密钥的路径。

*Synapse 1.67中的新增功能*：如果此文件不存在，Synapse将在启动时创建新签名密钥，并将其存储在此文件中。

配置示例：
```yaml
signing_key_path: "CONFDIR/SERVERNAME.signing.key"
```
---
### `old_signing_keys`

服务器用于签署消息但不用于签署新消息的密钥。对于每个密钥，`key`应为base64编码的公钥，`expired_ts`应为自Unix纪元以来最后使用的时间（毫秒）。

可以使用与synapse一起提供的`export_signing_key`脚本从旧的`signing.key`文件中构建一个条目。

配置示例：
```yaml
old_signing_keys:
  "ed25519:id": { key: "base64string", expired_ts: 123456789123 }
```
---
### `key_refresh_interval`

由此服务器发布的密钥响应的有效期。
用于在`/key/v2` APIs中设置`valid_until_ts`。
决定服务器查询以检查哪些密钥有效的频率。默认为1天。

配置示例：
```yaml
key_refresh_interval: 2d
```
---
### `trusted_key_servers`

用于下载签名密钥的可信服务器。

当我们需要获取签名密钥时，每个服务器都是并行尝试的。

通常，通过TLS证书验证与密钥服务器的连接。
可以通过配置`verify key`来提供额外的安全性，这将使Synapse检查响应是否由该密钥签署。

此设置取代旧的名为`perspectives`的设置。旧格式仍然支持以保持向后兼容，但已被弃用。

`trusted_key_servers`默认为matrix.org，但使用它将在启动时产生警告。要抑制此警告，请将`suppress_key_server_warning`设置为true。

如果必须停用可信密钥服务器的使用，例如在私人联盟中或出于隐私原因，这可以通过设置空数组（`trusted_key_servers: []`）来实现。然后，Synapse将直接从拥有密钥的服务器请求密钥。如果Synapse无法直接从服务器获取密钥，则此服务器的事件将被拒绝。

列表中每个条目的选项包括：
* `server_name`: 服务器名称。必填。
* `verify_keys`: 一个可选的从密钥ID到base64编码的公钥的映射。
   如果指定，我们将检查响应是否由所提供密钥中的至少一个签署。
* `accept_keys_insecurely`: 布尔值。通常，如果`verify_keys`未设置，且`federation_verify_certificates`不为`true`，Synapse将拒绝启动，因为这将允许任何能够伪造DNS响应的人假冒可信密钥服务器。如果您知道自己在做什么，并确定您的网络环境提供一个安全的连接到密钥服务器，您可以设置此项为`true`以覆盖此行为。

配置示例#1：
```yaml
trusted_key_servers:
  - server_name: "my_trusted_server.example.com"
    verify_keys:
      "ed25519:auto": "abcdefghijklmnopqrstuvwxyzabcdefghijklmopqr"
  - server_name: "my_other_trusted_server.example.com"
```
配置示例#2：
```yaml
trusted_key_servers:
  - server_name: "matrix.org"
```
---
### `suppress_key_server_warning`

设置以下值为true以禁用当`trusted_key_servers`包含'matrix.org'时发出的警告。请参阅上文。

配置示例：
```yaml
suppress_key_server_warning: true
```
---
### `key_server_signing_keys_path`

作为可信密钥服务器使用时使用的签名密钥。如果未指定，则默认为服务器签名密钥。

可以包含多个密钥，每行一个。

配置示例：
```yaml
key_server_signing_keys_path: "key_server_signing_keys.key"
```
---
## 单点登录集成

以下设置可以使Synapse使用单点登录提供程序进行认证，而不是其内部密码数据库。

您可能还想将以下选项设置为`false`以禁用常规登录/注册流程：
   * [`enable_registration`](#enable_registration)
   * [`password_config.enabled`](#password_config)

---
### `saml2_config`

启用SAML2用于注册和登录。使用pysaml2。要了解有关pysaml的更多信息并找完整列表的选项，阅读文档[此处](https://pysaml2.readthedocs.io/en/latest/)。

至少必须在此节中设置`sp_config`或`config_path`之一以启用SAML登录。您可以使用`sp_config`选项在线插入完整的pysaml配置，或者可以使用子选项`config_path`指定一个psyaml配置文件的路径。
此设置具有以下子选项：

* `idp_name`: 此身份提供者的用户界面名称，用于为用户提供登录机制的选择。
* `idp_icon`: 此身份提供者的可选图标，由客户端和Synapse自己的IdP选择页面呈现。如果给出，必须是格式为`mxc://<server-name>/<media-id>`的MXC URI。（获取此类MXC URI的简便方法是在某个（未加密的）房间中上传一张图片，然后从事件源中复制“url”。）
* `idp_brand`: 此身份提供者的可选品牌，使客户端根据特定的身份提供者调整登录流程样式。
   有关可能的选项，请参阅[规范](https://spec.matrix.org/latest/)。
* `sp_config`: pysaml2服务提供者的配置。有关配置格式，请参阅pysaml2文档。
   默认为`entityid`和`service`设置使用默认值，因此通常不需要指定，除非您需要覆盖它们。这里有一些用于配置pysaml的有用子选项：
   * `metadata`: 指向IdP的元数据。您必须通过`local`属性提供一个本地文件，或（首选）通过`remote`属性提供一个URL。
   * `accepted_time_diff: 3`: 允许的home服务器与IdP之间的时钟差（以秒为单位）。
      默认为0。
   * `service`: 默认情况下，用户必须首先访问我们的登录页面。如果您希望允许IdP发起的登录，请在`service`部分的`sp`中将`allow_unsolicited`设置为true。
* `config_path`: 可以通过如下方式指定一个独立的pysaml2配置文件：
  `config_path: "CONFDIR/sp_conf.py"`
* `saml_session_lifetime`: SAML会话的寿命。这定义了用户必须完成认证过程的时间（如果`allow_unsolicited`未设置）。默认为15分钟。
* `user_mapping_provider`: 使用此选项，可以提供一个外部模块作为自定义解决方案，将从saml提供者返回的属性映射到matrix用户。`user_mapping_provider`具有以下属性：
  * `module`: 自定义模块的类。
  * `config`: 模块的自定义配置值。如果您使用内建的user_mapping_provider，请使用示例中提供的值，或者如果您使用的是自定义类，则提供您自己的配置值。
     此部分将作为Python字典传递给模块的`parse_config`方法。内建提供者接受以下两个选项：
      * `mxid_source_attribute`: 用于从中派生Matrix ID的SAML属性（在通过属性映射后）。默认值为'uid'。注意：这过去是通过`saml2_config.mxid_source_attribute选项`进行配置的。如果仍然定义了该选项，则将使用其值。
      * `mxid_mapping`: 用于将saml属性映射到matrix ID的映射系统。选项包括：`hexencode`（将不允许的字符映射为'=xx'）和`dotreplace`（将不允许的字符替换为'.'）。默认值为`hexencode`。注意：这过去是通过`saml2_config.mxid_mapping选项`进行配置的。如果它仍然定义了，则将使用其值。
* `grandfathered_mxid_source_attribute`: 在之前的synapse版本中，SAML属性到MXID的映射总是动态计算的，而不是存储在表中。为了向后兼容，我们将在创建新帐户之前寻找匹配此模式的`user_ids`。
   此设置控制将用于此向后兼容性查找的SAML属性。通常应为'uid'，但如果属性映射已更改，可能需要更改它。默认值为'uid'。
* `attribute_requirements`: 可以将Synapse配置为仅在SAML属性匹配特定值时允许登录。
    可以按如下例在`attribute_requirements`下列出这些要求。列出的所有属性必须匹配，登录才被允许。
* `idp_entityid`: 如果元数据XML包含多个IdP实体，则必须设置`idp_entityid`选项为要重定向用户的实体。
   大多数部署只包含一个IdP实体，因此应省略此选项。
```