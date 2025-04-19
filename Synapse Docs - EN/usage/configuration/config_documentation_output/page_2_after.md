```
# 启用本地给定端口的Twisted SSH Manhole服务。
# Unix socket监听器：适合于在反向代理后面的Synapse部署，提供轻量级的进程间通信，没有TCP/IP的开销，避免端口冲突，并通过系统文件权限提供增强的安全性。
# 注意，当使用UNIX socket时，x_forwarded默认会设为true。更多信息请参见https://element-hq.github.io/synapse/latest/reverse_proxy.html。
### `manhole_settings`
Manhole的连接设定。可以在[这里](../../manhole.md)找到更多关于manhole的信息。Manhole子选项包括：
* `username`：manhole的用户名。默认是 'matrix'。
* `password`：manhole的密码。默认是 'rabbithole'。
* `ssh_priv_key_path` 和 `ssh_pub_key_path`：加密manhole流量所用的私人和公共SSH密钥对。
  如果未设置这些，则会使用硬编码且非机密的密钥，这可能会使流量在公共网络上传输时被拦截。
### `dummy_events_threshold`
由于不同家庭服务器之间的网络延迟，转发极端事件可能会在一个房间中累积。一旦这种情况发生在一个大房间中，计算该房间的状态可能会变得相当昂贵。为了缓解这种情况，一旦转发极端事件的数量达到给定的阈值，Synapse会发送一个`org.matrix.dummy_event`事件，将减少该房间的转发极端事件。
此设置定义了发送虚拟事件的阈值（即房间内转发极端事件的数量）。默认值是10。
### `delete_stale_devices_after`
一个可选的时长。如果设置，Synapse将运行一个每日后台任务来注销并删除超过指定时间未被访问的设备。
默认为没有时长，这意味着设备永远不会被删除。
**注意：**无论`run_background_tasks_on`的值如何，此任务将始终在主进程上运行。因为工作人员目前没有能力删除设备。
### `email`
Synapse的邮件发送配置。
服务器管理员可以配置电子邮件内容的自定义模板。更多信息请参见[这里](../../templates.md)。
此设置有以下子选项：
* `smtp_host`：要使用的外发SMTP服务器的主机名。默认是 'localhost'。
* `smtp_port`：邮件服务器上的外发SMTP端口。如果`force_tls`为true则默认为465，否则为25。
  _在Synapse 1.64.0中更改：_默认端口现在会考虑到`force_tls`。
* `smtp_user` 和 `smtp_pass`：用于SMTP服务器认证的用户名/密码。默认情况下，不会进行身份验证。
* `force_tls`：默认情况下，Synapse通过明文连接，然后可选地通过STARTTLS升级到TLS。如果此选项设置为true，则从一开始使用TLS（隐式TLS），并忽略选项`require_transport_security`。如果您的邮件服务器支持，建议启用此功能。
  _在Synapse 1.64.0中新增：_
* `require_transport_security`：设置为true以要求SMTP的TLS传输安全。默认情况下，Synapse会通过明文连接，然后在SMTP服务器支持的情况下切换到TLS via STARTTLS。如果此选项被设置，Synapse将拒绝连接，除非服务器支持STARTTLS。
* `enable_tls`：默认情况下，如果服务器支持TLS，则会使用，并且服务器必须提供一个验证'ded_sm'的有效证书。如果此选项设置为false，将不使用TLS。
* `notif_from`：定义发送邮件时使用的"From"地址。必须设置，如果有启用电子邮件发送。占位符'%(app)s'将被应用程序名称所取代，通常在'app_name'中设置，但可能被Matrix客户端应用程序覆盖。注意，占位符必须写作'%(app)s'，包括末尾的's'。
* `app_name`：`app_name`定义了'%(app)s'在`notif_from`和电子邮件主题中的默认值。默认为'Matrix'。
* `enable_notifs`：设置为true以允许用户接收电子邮件通知。如果未设置，用户可以配置电子邮件通知但不会接收到。默认禁用。
* `notif_for_new_users`：设置为false以禁用为新用户自动订阅电子邮件通知。默认启用。
* `notif_delay_before_mail`：发邮件通知前等待的时间。这给用户一个通过推送或开放客户端查看消息的机会。默认是10分钟。
  _在Synapse 1.99.0中新增：_
* `client_base_url`：邮件通知中的客户端链接的自定义URL。默认情况下，链接将基于"https://matrix.to"。（此设置以前称为`riot_base_url`；旧名称仍然支持以确保向后兼容，但现已弃用。）
* `validation_token_lifetime`：配置验证邮件在发送后失效的时间。默认是1h。
* `invite_client_location`：在邀请过程中不在用户的位置重定向到web客户端位置。这被传递到身份服务器作为`org.matrix.web_client_location`键。默认为未设置，未给出身份服务器指导。
* `subjects`：用于发送来自Synapse的电子邮件通知时的主题。占位符'%(app)s'将被'app_name'设置的值替换，或者由Matrix客户端应用程序 dictcated。如果设置了主题，还可使用以下替换占位符：'%(person)s'，将被发送消息的人（或多人的）显示名称替换，例如"爱丽丝和鲍勃"，'%(room)s'，将被消息发送到的房间的名称替换，例如"我的超级房间"。此外，与帐户管理相关的电子邮件还可以使用'%(server_name)s'占位符，将被在Synapse配置中server_name的设置的值替换。
  这里是通知邮件可以设置的主题列表：
    * `message_from_person_in_room`：用于通知房间名称里的一个或多个用户发送的一条消息的主题。默认为"[%(app)s] %(进某个应用发来的信息在%(ání)s %()房间信息上的信息。"
    * `message_from_person`：用于通知无房名房间的一个或多个用户发来的消息的主题。默认为"[%(app)s] %(person)s在%(app)s上给您发了消息..."
    * `messages_from_person`：用于通知无房名房间的一个或多个用户所发多条消息的主题。默认为"[%(app)s] 您在%(app)s上有来自%(晓某)的信息...(啬"
    * `messages_in_room`：用于通知有房名房间的多个信息的主题。默认为"[%(app)s] 您在%(app)s上有来自%(room)s房间的信息..."
    * `messages_in_room_and_others`：用于通知多个房间里的多个信息。默认为"[%(app)s] 您在%(app)s上有来自%(room)s房间的信息， 和其他房间的信息..."
    * `messages_from_person_and_others`：用于通知多个房间里多个用户发来的消息。与上面的设置相似，只是在引发通知的房间没有名字时使用。默认为"[%(app)s] 您在%(app)s上的消息来自%(person)s和其他..."
    * `invite_from_person_to_room`：用于通知加入有房名房间邀请的主题。默认为"[%(app)s] %(person)s已邀请您加入%(room)s房间在%(app)s上..."
    * `invite_from_person`：用于通知加入无房名房间的邀请。默认为"[%(app)s] %(person)s已邀请您在%(app)s上聊天..."
    * `password_reset`：用于发送密码重设邮件の主题。默认是 "[%(server_name)s] 密码重置"
    * `email_validation`：用于发送验证邮件以确认地址所有权の主题。默认是 "[%(server_name)s] 验证您的邮件"
### `max_event_delay_duration`
根据[MSC4140](https://github.com/matrix-org/matrix-spec-proposals/pull/4140)所规定的发送事件可能被推迟的最大允许时长。必须为正值如果设置。
默认为无时长（`null`），这不允许发送推迟事件。
## 服务器封锁
对 Synapse 管理员有用的选项。
### `admin_contact`
如何联系服务器管理员，在 `ResourceLimitError` 中使用。默认为 none。
### `hs_disabled` 和 `hs_disabled_message`
阻止用户连接到服务和给出一个人类可读的原因为什么连接被阻止。默认为 false。
### `limit_usage_by_mau`
此选项可禁用/启用每月活动用户限制。用于在管理员或服务器拥有者希望限制的情况下，限制用户月活。启用时，如果达到限制，服务器返回一个错误类型为"Codes.RESOURCE_LIMIT_EXCEEDED"的"ResourceLimitError"。默认是false。如果启用，必须还设置一个值为`max_mau_value`。
参见每月活动用户的更多详情。
```

### Footnotes:
- 注意区分`path`与`port`在配置中的应用，不得误用。
- YAML格式要求严格注意缩进及格式排列，以确保配置文件解析正确。
