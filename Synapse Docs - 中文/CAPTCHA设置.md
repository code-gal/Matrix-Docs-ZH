### 概述
可以在您的家庭服务器上启用验证码，以帮助防止机器人注册账户。Synapse目前使用Google的reCAPTCHA服务，这需要从Google获取API密钥。

#### 获取API密钥

1. 在[Google reCAPTCHA网站](https://www.google.com/recaptcha/admin/create)创建一个新站点。
1. 将标签设置为您想要的任何内容。
1. 将类型设置为使用“我不是机器人”复选框选项的reCAPTCHA v2。这是唯一与Synapse兼容的验证码类型。
1. 将您的服务器的公共主机名添加到授权域列表中，该主机名在`homeserver.yaml`中的`public_baseurl`设置。如果您没有设置`public_baseurl`，请使用`server_name`。
1. 同意服务条款并提交。
1. 复制您的站点密钥和秘密密钥，并将它们添加到您的`homeserver.yaml`配置文件中：
    ```yaml
    recaptcha_public_key: YOUR_SITE_KEY
    recaptcha_private_key: YOUR_SECRET_KEY
    ```
1. 启用新注册的验证码：
    ```yaml
    enable_registration_captcha: true
    ```
1. 转到您刚刚创建的验证码的设置页面。
1. 取消选中“验证reCAPTCHA解决方案的来源”复选框，以便验证码可以在任何客户端显示。如果不禁用此选项，则必须指定允许显示验证码的每个客户端的域名。

#### 配置用于身份验证的IP

reCAPTCHA API要求发送解决验证码的用户的IP地址。如果客户端通过代理或负载均衡器连接，可能需要使用`X-Forwarded-For`（XFF）头而不是原始IP地址。这可以通过在`homeserver.yaml`配置文件的`listeners`部分使用`x_forwarded`指令来配置。