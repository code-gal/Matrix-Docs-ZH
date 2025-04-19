### eturnal TURN 服务器

以下部分描述了如何安装 eturnal（实现了 TURN REST API）。

#### `eturnal` 设置

##### 初始安装

`eturnal` TURN 服务器实现可从多种来源获得，例如原生包管理器、二进制包、源码安装或容器镜像。这里描述了所有这些方法。

在 Linux Shell 或 Docker 中快速测试的说明也已提供。

##### 配置

安装后， `eturnal` 通常会在这里提供一个默认配置文件： `/etc/eturnal.yml` （如果没有找到，则有一个备份文件在这里： `/opt/eturnal/etc/eturnal.yml` ）。它使用（对缩进敏感的！）YAML 格式。文件中包含进一步的说明。

以下是一些关于如何在主机上配置 eturnal 或使用 Docker 时的提示。您也可以进一步深入参考文档。

`eturnal` 开箱即用，默认配置即可运行。要启用 TURN 并将其与您的家庭服务器集成，需要编辑 `eturnal` 默认配置文件中的一些方面：

1. 家庭服务器的 `turn_shared_secret` 和 eturnal 的共享 `secret` 用于身份验证

两者需要具有相同的值。在 `eturnal` 的配置文件中取消注释并调整此行：

```
secret: "long-and-cryptic"     # Shared secret, CHANGE THIS.
```

生成 `secret` 的一种方法是使用 `pwgen` :
```
pwgen -s 64 1
```

2. 公共 IP 地址

如果您的 TURN 服务器位于 NAT 后面，NAT 网关必须有一个外部的、可公开访问的 IP 地址。 `eturnal` 会尝试自动检测公共 IP 地址，但也可以通过取消注释并调整此行来配置，使 `eturnal` 向连接的客户端通告该地址：

```
relay_ipv4_addr: "203.0.113.4" # The server's public IPv4 address.
```

如果您的 NAT 网关可以通过 IPv4 和 IPv6 访问，您可以配置 `eturnal` 来公布每个可用的地址：

```
relay_ipv4_addr: "203.0.113.4" # The server's public IPv4 address.
relay_ipv6_addr: "2001:db8::4" # The server's public IPv6 address (optional).
```

在广告外部IPv6地址时，请确保运行TURN服务器的系统的防火墙和网络设置已配置为接受IPv6流量，并且TURN服务器正在监听由NAT映射到外部IPv6地址的本地IPv6地址。

1. **日志记录**

    如果`eturnal`是由systemd启动的，日志文件默认写入`/var/log/eturnal`目录。要改为记录到[日志系统](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html)，可以在配置文件中将`log_dir`选项设置为`stdout`。

1. **安全考虑**

    请考虑您的安全设置。TURN允许用户请求一个中继，该中继将连接到任意IP地址和端口。以下配置被建议作为最低起点，[另见官方文档](https://eturnal.net/documentation/#blacklist)：

    ```yaml
    ## 拒绝从/到以下地址/网络的TURN中继：
    blacklist:                 # 这是默认的黑名单。
        - "127.0.0.0/8"        # IPv4环回。
        - "::1"                # IPv6环回。
        - recommended          # 扩展到建议被阻止的多个网络，但包括私有网络。如果eturnal服务于这些网络内的本地客户端/对等端，则需要将这些网络'白名单'。
    ```
为了将 IP 地址或特定（私有）网络列入白名单，您需要在配置文件中添加一个白名单部分，例如：

```yaml
whitelist:
    - "192.168.0.0/16"
    - "203.0.113.113"
    - "2001:db8::/64"
```

越具体越好。

1. TURNS（通过 TLS/DTLS 的 TURN）

也考虑支持 TLS/DTLS。为此，请在 `eturnal.yml` 配置文件中调整以下设置（TLS 部分不应再被注释）：
```YAML

listen:
    - ip: "::"
      port: 3478
      transport: udp
    - ip: "::"
      port: 3478
      transport: tcp
    - ip: "::"
      port: 5349
      transport: tls
```

#### TLS 证书/密钥文件（必须由'eturnal'用户可读！）：
```
tls_crt_file: /etc/eturnal/tls/crt.pem 
tls_key_file: /etc/eturnal/tls/key.pem 
```
在这种情况下，将 homeserver 的 `turn_uris` 设置中的 `turn:` 方案替换为 `turns:`。更多内容请参见[此处](../../usage/configuration/config_documentation.md#turn_uris)。

我们建议您在完成基本安装并使其正常工作后，再尝试设置 TLS/DTLS。

注意：如果您的 TLS 证书是由 Let's Encrypt 提供的，那么使用 Chromium 的 WebRTC 库的任何 Matrix 客户端将无法使用 TLS/DTLS。这目前包括 Element Android 和 iOS；有关更多详细信息，请参见它们各自的[问题](https://github.com/element-hq/element-android/issues/1533)和[问题](https://github.com/element-hq/element-ios/issues/2712)，以及底层的[WebRTC 问题](https://bugs.chromium.org/p/webrtc/issues/detail?id=11710)。请考虑使用 ZeroSSL 证书作为您的 TURN 服务器的可行替代方案。

1. 防火墙

确保您的防火墙允许流量进入您配置的 TURN 服务器监听的端口（默认情况下：3478 和 5349 用于 TURN 流量（请记住允许 TCP 和 UDP 流量），以及 49152-65535 端口用于 UDP 中继。）
2. 重新加载/重启 `eturnal`

配置文件中的更改需要 `eturnal` 重新加载/重启，可以通过以下方式实现：

```
eturnalctl reload
```

`eturnal` 在实际重新加载/重启之前执行配置检查，并在配置不正确时提供提示。

##### eturnalctl 操作脚本

`eturnal` 提供了一个方便的操作脚本，可以用来检查服务是否启动、重启服务、查询当前有多少活跃会话、更改日志记录行为等等。