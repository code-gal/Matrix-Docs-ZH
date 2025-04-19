### JWT 登录类型

Synapse附带了一种非标准的登录类型以支持[JSON Web Tokens](https://en.wikipedia.org/wiki/JSON_Web_Token)。一般来说，[登录端点](https://matrix.org/docs/spec/client_server/r0.6.1#login)的文档仍然有效（机制类似于[基于令牌的登录](https://matrix.org/docs/spec/client_server/r0.6.1#token-based)）。

要使用JSON Web Token登录，客户端应提交如下的`/login`请求：

```json
{
  "type": "org.matrix.login.jwt",
  "token": "<jwt>"
}
```

`token`字段应包含具有以下声明的JSON Web Token：

* 需要一个编码用户ID本地部分的声明。默认情况下，使用`sub`（主题）声明，或者可以在配置文件中设置自定义声明。
* 过期时间（`exp`）、不早于时间（`nbf`）和签发时间（`iat`）声明是可选的，但如果存在则会被验证。
* 发行者（`iss`）声明是可选的，但如果配置则是必需的并会被验证。
* 受众（`aud`）声明是可选的，但如果配置则是必需的并会被验证。如果未配置而提供受众声明，将导致验证失败。

如果令牌无效，主服务器必须响应`403 Forbidden`并附带错误代码`M_FORBIDDEN`。

与其他登录类型一样，还有其他字段（例如`device_id`和`initial_device_display_name`）可以包含在上述请求中。

#### 准备Synapse

Synapse中的JSON Web Token集成使用[`Authlib`](https://docs.authlib.org/en/latest/index.html)库，必须按如下方式安装：

* 相关库已包含在`matrix.org`提供的Docker镜像和Debian包中，因此无需进一步操作。

* 如果您将Synapse安装到虚拟环境中，请运行`/path/to/env/bin/pip install synapse[jwt]`以安装必要的依赖项。

* 对于其他安装机制，请参阅维护者提供的文档。

要启用JSON Web Token集成，您应在配置文件中添加一个`jwt_config`选项。请参阅[配置手册](Synapse%20Docs%20-%20EN/usage/configuration/config_documentation.md#jwt_config)以获取一些示例设置。

#### 作为开发人员测试JWT的方法

虽然JSON Web Tokens通常由外部服务器生成，但下面的示例使用本地生成的JWT。

1. 使用JWT登录配置Synapse，请注意此示例使用预共享密钥和HS256算法：

    ```yaml
    jwt_config:
        enabled: true
        secret: "my-secret-token"
        algorithm: "HS256"
    ```
2. 生成一个JSON Web Token：

    您可以使用以下简短的Python代码片段生成一个受HMAC保护的JWT。
    请确保`secret`和`header`中给出的算法与上述`jwt_config`中的条目匹配。

    ```python
    from authlib.jose import jwt

    header = {"alg": "HS256"}
    payload = {"sub": "user1", "aud": ["audience"]}
    secret = "my-secret-token"
    result = jwt.encode(header, payload, secret)
    print(result.decode("ascii"))
    ```

3. 查询登录类型并确保`org.matrix.login.jwt`存在：

    ```bash
    curl http://localhost:8080/_matrix/client/r0/login
    ```
4. 使用上面生成的JSON Web Token登录：

    ```bash
    $ curl http://localhost:8082/_matrix/client/r0/login -X POST \
        --data '{"type":"org.matrix.login.jwt","token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0LXVzZXIifQ.Ag71GT8v01UO3w80aqRPTeuVPBIBZkYhNTJJ-_-zQIc"}'
    {
        "access_token": "<access token>",
        "device_id": "ACBDEFGHI",
        "home_server": "localhost:8080",
        "user_id": "@test-user:localhost:8480"
    }
    ```

您现在应该能够使用返回的访问令牌查询客户端API。