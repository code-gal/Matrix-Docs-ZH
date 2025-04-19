您的主服务器配置文件需要以下额外的键：

1. [`turn_uris`](Synapse%20Docs%20-%20EN/usage/configuration/config_documentation.md#turn_uris))
2. [`turn_shared_secret`](Synapse%20Docs%20-%20EN/usage/configuration/config_documentation.md#turn_shared_secret))
3. [`turn_user_lifetime`](Synapse%20Docs%20-%20EN/usage/configuration/config_documentation.md#turn_user_lifetime))
4. [`turn_allow_guests`](Synapse%20Docs%20-%20EN/usage/configuration/config_documentation.md#turn_allow_guests))

例如，以下是`matrix.org`配置文件的相关部分。`turn_uris`适用于监听默认端口的TURN服务器，无TLS。

```yaml
turn_uris: [ "turn:turn.matrix.org?transport=udp", "turn:turn.matrix.org?transport=tcp" ]
turn_shared_secret: "n0t4ctuAllymatr1Xd0TorgSshar3d5ecret4obvIousreAsons"
turn_user_lifetime: 86400000
turn_allow_guests: true
```

更新主服务器配置后，您必须重启Synapse：

* 如果您使用synctl：
  ```sh
  # 根据Synapse的安装方式，synctl可能已经在您的PATH中。如果没有，您可能需要激活虚拟环境。
  synctl restart
  ```
* 如果您使用systemd：
  ```sh
  systemctl restart matrix-synapse.service
  ```
... 然后重新加载任何客户端（或等待一小时以刷新其设置）。