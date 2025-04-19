### 共享密钥注册

注意：当 MSC3861 启用时，此 API 将被禁用。参见  [#15582](https://github.com/matrix-org/synapse/pull/15582)

此 API 允许以管理和非交互方式创建用户。这通常用于用管理员账户引导 Synapse 实例。

要向服务器验证身份，您需要共享密钥（在 homeserver 配置中为 ([`registration_shared_secret`](../usage/configuration/config_documentation.md#registration_shared_secret) ），以及一次性随机数。如果未配置注册共享密钥，则此 API 不会启用。

要获取 nonce，您需要从 API 请求一个：

```
> GET /_synapse/admin/v1/register

< {"nonce": "thisisanonce"}
```

一旦您有了 nonce，您可以向同一个 URL 发送一个 `POST` ，并在 JSON 主体中包含 nonce、用户名、密码、是否为管理员（可选，默认为 False）以及内容的 HMAC 摘要。您还可以设置显示名称（可选，默认为 `username` ）。

例如：

```
> POST /_synapse/admin/v1/register
> {
   "nonce": "thisisanonce",
   "username": "pepper_roni",
   "displayname": "Pepper Roni",
   "password": "pizza",
   "admin": true,
   "mac": "mac_digest_here"
  }

< {
   "access_token": "token_here",
   "user_id": "@pepper_roni:localhost",
   "home_server": "test",
   "device_id": "device_id_here"
  }
```

MAC 是 HMAC-SHA1 算法的十六进制摘要输出，密钥为共享密钥，内容为 nonce、用户、密码、字符串“admin”或“notadmin”，以及可选的 user_type，每个部分由 NUL 分隔。

如果您有 Bash 和 OpenSSL，以下是生成 HMAC 摘要的简单方法：

```bash
### 更新这些值，然后将此代码块粘贴到bash终端
nonce='thisisanonce'
username='pepper_roni'
password='pizza'
admin='admin'
secret='shared_secret'

printf '%s\0%s\0%s\0%s' "$nonce" "$username" "$password" "$admin" |
  openssl sha1 -hmac "$
    ^/_matrix/client/(api/v1|r0|v3|unstable)/rooms/.*/state$}'
```

在 Python 中生成的示例：

```python
import hmac, hashlib

def generate_mac(nonce, user, password, admin=False, user_type=None):

    mac = hmac.new(
      key=shared_secret,
      digestmod=hashlib.sha1,
    )

    mac.update(nonce.encode('utf8'))
    mac.update(b"\x00")
    mac.update(user.encode('utf8'))
    mac.update(b"\x00")
    mac.update(password.encode('utf8'))
    mac.update(b"\x00")
    mac.update(b"admin" if admin else b"notadmin")
    if user_type:
        mac.update(b"\x00")
        mac.update(user_type.encode('utf8'))

    return mac.hexdigest()
```