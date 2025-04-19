### 版本API

此API返回正在运行的Synapse版本。
当Synapse实例位于不转发“Server”头（也包含Synapse版本信息）的代理后面时，这很有用。

API为：

```
GET /_synapse/admin/v1/server_version
```

它返回如下JSON主体：

```json
{
    "server_version": "0.99.2rc1 (b=develop, abcdef123)"
}
```

*在Synapse 1.94.0中更改：* 从响应体中移除了`python_version`键。