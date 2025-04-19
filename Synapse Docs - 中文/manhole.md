使用 synapse 人孔
=============

“人孔”允许服务器管理员访问正在运行的 Synapse 安装上的 Python 壳。这是一种非常强大的管理和调试机制。

**_安全警告_**

请注意，这将赋予所有具有服务器 shell 访问权限的用户对 synapse 的管理访问权限。因此，在不受信任的用户具有 shell 访问权限的环境中不应启用此功能。

#### 配置 manhole

要启用它，首先在您的 `homeserver.yaml` 中添加 `manhole` 监听器配置。您可以在配置手册中找到如何操作的信息。如果您使用的是 docker，配置会略有不同。

###### Docker 配置

如果您使用 Docker，请将 `bind_addresses` 设置为 `['0.0.0.0']` ，如下所示：

```
listeners:
  - port: 9000
    bind_addresses: ['0.0.0.0']
    type: manhole
```

当使用 `docker run` 启动服务器时，您需要将命令更改为以下内容以包含 `manhole` 端口转发。下面的 `-p 127.0.0.1:9000:9000` 很重要：它确保只有本地用户可以访问 `manhole` 。

```
docker run -d --name synapse \
    --mount type=volume,src=synapse-data,dst=/data \
    -p 8008:8008 \
    -p 127.0.0.1:9000:9000 \
    vectorim/synapse:latest
```

###### 原生配置

如果您不使用 Docker，请将 `bind_addresses` 设置为 `['::1', '127.0.0.1']` ，如图所示。下面的示例中的 `bind_addresses` 很重要：它确保只有本地用户可以访问 `manhole` 。

```
listeners:
  - port: 9000
    bind_addresses: ['::1', '127.0.0.1']
    type: manhole
```

##### 安全设置

以下配置选项可用：

*   `username` - 人孔的用户名（默认为 `matrix` ）
*   `password` - 人孔的密码（默认为 `rabbithole` ）
*   `ssh_priv_key` - 私有 SSH 密钥的路径（默认为硬编码值）
*   `ssh_pub_key` - 公共 SSH 密钥的路径（默认为硬编码值）

例如：

```
manhole_settings:
  username: manhole
  password: mypassword
  ssh_priv_key: "/home/synapse/manhole_keys/id_rsa"
  ssh_pub_key: "/home/synapse/manhole_keys/id_rsa.pub"
```

#### 访问 synapse manhole

然后重启 synapse，并将 ssh 客户端指向 localhost 上的 9000 端口，使用在 `homeserver.yaml` 中配置的用户名和密码 - 使用默认配置，这将是：

```
ssh -p9000 matrix@localhost
```

然后在提示时输入密码（默认是 `rabbithole` ）。

这提供了一个 Python REPL，其中 `hs` 提供了对 `synapse.server.HomeServer` 对象的访问 - 这反过来又提供了对进程的许多其他部分的访问。

请注意，在 Synapse 1.41 之前，任何返回协程的调用都需要用 `ensureDeferred` 包装。

作为一个简单的例子，从数据库中检索一个事件：

```
>>> from twisted.internet import defer
>>> defer.ensureDeferred(hs.get_datastores().main.get_event('undefinedyeQaw:matrix.org', type='m.room.create', state_key=''>>
```