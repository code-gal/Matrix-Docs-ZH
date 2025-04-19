### coturn TURN 服务器

以下部分描述了如何安装 coturn（实现了 TURN REST API）。

#### `coturn` 设置

##### 初始安装

TURN 守护进程 `coturn` 可以从多种来源获取，例如原生包管理器，或从源代码安装。

###### 基于 Debian 和 Ubuntu 的发行版

只需安装 Debian 包：

```
sudo apt install coturn
```

这将安装并启动一个名为 `coturn` 的 systemd 服务。

###### 源码安装

1. 从 GitHub 下载最新版本。解压缩并 `cd` 到目录中。
2. 配置它：

```
./configure
```

您可能需要安装 `libevent2` ：如果需要，请按照您的操作系统推荐的方式进行安装。您可以忽略关于缺乏数据库支持的警告：此目的不需要数据库。
3. 构建并安装它：
```sh
make
sudo make install
   ```
   
##### 配置

1. 在 `/etc/turnserver.conf` 中创建或编辑配置文件。相关行，带有示例值，是：

```
use-auth-secret
static-auth-secret=[your secret key here]
realm=turn.myserver.org
```

查看 `turnserver.conf` 了解选项的解释。生成 `static-auth-secret` 的一种方法是使用 `pwgen` :

```
pwgen -s 64 1
```

必须指定 `realm` ，但其值有些随意。（它作为身份验证流程的一部分发送给客户端。）通常将其设置为您的服务器名称。
2. 您很可能会希望配置 `coturn` 来将日志写入某个位置。最简单的方法通常是将它们发送到系统日志：

```
syslog
```

(在这种情况下，日志将通过 `journalctl -u coturn` 在 systemd 系统上可用)。或者，可以配置 `coturn` 写入日志文件 - 请查看随 `coturn` 提供的示例配置文件。
3. 考虑您的安全设置。TURN 允许用户请求一个中继，该中继将连接到任意 IP 地址和端口。建议以下配置作为最低起点：
```yaml
	# VoIP 流量全部是 UDP。没有理由让用户通过中继连接到任意 TCP 端点。===
	no-tcp-relay
	# 不要让中继尝试连接到您网络内的私有 IP 地址范围（如果有的话）===
	# 考虑到 TURN 服务器可能位于您的防火墙之后，请记住也包括任何特权公共 IP。===
    denied-peer-ip=10.0.0.0-10.255.255.255
    denied-peer-ip=192.168.0.0-192.168.255.255
    denied-peer-ip=172.16.0.0-172.31.255.255
	# 推荐额外阻止的本地对等体，以减轻对内部服务的外部访问。===
	# https://www.rtcsec.com/article/slack-webrtc-turn-compromise-and-bug-bounty/#how-to-fix-an-open-turn-relay-to-address-this-vulnerability===
    no-multicast-peers
    denied-peer-ip=0.0.0.0-0.255.255.255
    denied-peer-ip=100.64.0.0-100.127.255.255
    denied-peer-ip=127.0.0.0-127.255.255.255
    denied-peer-ip=169.254.0.0-169.254.255.255
    denied-peer-ip=192.0.0.0-192.0.0.255
    denied-peer-ip=192.0.2.0-192.0.2.255
    denied-peer-ip=192.88.99.0-192.88.99.255
    denied-peer-ip=198.18.0.0-198.19.255.255
    denied-peer-ip=198.51.100.0-198.51.100.255
    denied-peer-ip=203.0.113.0-203.0.113.255
    denied-peer-ip=240.0.0.0-255.255.255.255
	# 特别处理转发服务器本身，以便客户端->TURN->TURN->客户端的流量能够正常工作===
	# 这应该是转发服务器的监听 IP 之一===
	allowed-peer-ip=10.0.0.1
	# 考虑是否要限制每个用户（或总体）的中继流量配额，以避免 DoS 攻击的风险。===
	user-quota=12 # 每个视频通话 4 个流，因此 12 个流 = 每个用户 3 个同时进行的中继通话。 
	total-quota=1200 
```

1. 也请考虑支持 TLS/DTLS。为此，请在 `turnserver.conf` 中添加以下设置：

```
  # TLS 证书，包括中间证书。===
  # 对于 Let's Encrypt 证书，在此使用 `fullchain.pem` 。===
  cert=/path/to/fullchain.pem
  # TLS 私钥文件===
  pkey=/path/to/privkey.pem
  # 确保禁用 TLS/DTLS 的配置行被注释掉或移除===
  #no-tls 
  #no-dtls
```

在这种情况下，将 `turn_uris` 设置中的 `turn:` 方案替换为 `turns:`。

我们建议您在完成基本安装并使其正常工作后，再尝试设置 TLS/DTLS 。

注意：如果您的 TLS 证书是由 Let's Encrypt 提供的，任何使用 Chromium 的 WebRTC 库的 Matrix 客户端都将无法使用 TLS/DTLS 。这目前包括 Element Android 和 iOS；有关更多详细信息，请查看它们各自的[问题](https://github.com/element-hq/element-android/issues/1533)和[问题](https://github.com/element-hq/element-ios/issues/2712)，以及底层的[WebRTC 问题](https://bugs.chromium.org/p/webrtc/issues/detail?id=11710)。请考虑使用 ZeroSSL 证书作为您的 TURN 服务器的可行替代方案。

1. 确保您的防火墙允许流量进入您配置的 TURN 服务器监听的端口（默认情况下：3478 和 5349 用于 TURN 流量（请记住允许 TCP 和 UDP 流量），以及 49152-65535 端口用于 UDP 中继。）
2. 如果您的 TURN 服务器位于 NAT 后面，NAT 网关必须有一个外部的、可公开访问的 IP 地址。您必须配置 `coturn` 以向连接的客户端通告该地址：

```
external-ip=EXTERNAL_NAT_IPv4_ADDRESS
```

您可以选择性地限制 TURN 服务器仅监听由 NAT 映射到外部地址的本地地址：

```
listening-ip=INTERNAL_TURNSERVER_IPv4_ADDRESS
```

如果您的 NAT 网关可以通过 IPv4 和 IPv6 访问，您可以配置 `coturn` 来公布每个可用的地址：

```
external-ip=EXTERNAL_NAT_IPv4_ADDRESS
external-ip=EXTERNAL_NAT_IPv6_ADDRESS
```

在公布外部 IPv6 地址时，请确保运行 TURN 服务器的系统的防火墙和网络设置已配置为接受 IPv6 流量，并且 TURN 服务器正在监听由 NAT 映射到外部 IPv6 地址的本地 IPv6 地址。
3. （重新）启动 TURN 服务器：

*   如果您使用了 Debian 包（或自己设置了 systemd 单元）：

    sudo systemctl restart coturn

*   如果您是从源代码构建的：

    /usr/local/bin/turnserver -o