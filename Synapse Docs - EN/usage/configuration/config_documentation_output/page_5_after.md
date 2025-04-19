```
--- 
### `url_preview_ip_range_whitelist`

此选项设置一组允许URL预览爬虫访问的IP地址CIDR范围，即使它们在`url_preview_ip_range_blacklist`中被列出。这对于指定广泛的黑名单目标IP范围的例外情况很有用，例如仅在您的网络中可见的特定私有网站启用URL预览。默认没有。
示例配置:
```yaml
url_preview_ip_range_whitelist:
   - '192.168.1.1'
```
---
### `url_preview_url_blacklist`

阻止URL预览爬虫访问的URL匹配列表。此功能目的是提高可用性，而非安全性。因此，应优先使用`url_preview_ip_range_blacklist`，否则可能有人定义指向私有IP地址的公共DNS条目，从而绕过黑名单。某些重定向请求或检查到Synapse访问后提供不同内容的应用也可能绕过黑名单。此功能更适用于显然不希望Synapse预览的URL形状。
每个列表条目都是一个URL组件属性字典，如`urlparse.urlsplit`应用于绝对URL形式之后所返回的信息。详见[这里](https://docs.python.org/2/library/urlparse.html#urlparse.urlsplit)。一些例子包括:
* `username`
* `netloc`
* `scheme`
* `path`

字典的值被视作应用于URL组件的文件名匹配模式，除非以^开头，此时被视为正则表达式匹配。若为某列表项指定的所有组件匹配成功，则URL被列入黑名单。
示例配置:
```yaml
url_preview_url_blacklist:
  # 黑名单中含有用户名的任何URL
  - username: '*'
  
  # 黑名单中所有*.google.com的URL
  - netloc: 'google.com'
  - netloc: '*.google.com'
  
  # 黑名单中所有普通HTTP URL
  - scheme: 'http'
  
  # 黑名单http(s)://www.acme.com/foo
  - netloc: 'www.acme.com'
    path: '/foo'
  
  # 黑名单中含有字面意义IPv4地址的任何URL
  - netloc: '^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$'
```
---
### `max_spider_size`

允许的最大URL预览爬虫数据大小，以字节为单位。默认为10M。
示例配置:
```yaml
max_spider_size: 8M
```
---
### `url_preview_accept_language`

下载网页期间，用于URL预览生成时的HTTP头Accept-Language值列表。此选项允许Synapse在与远程服务器通信时指定URL预览所应使用的首选语言。
每个值是一个IETF语言标记；2-3个字母的标识符，可以选择性地随附以-分隔的子标签，指定国家或区域变体。
可以提供多个值，每个值可以通过使用质量值语法(;q=)加权。'*'表示任何语言。
默认为“en”。
示例配置:
```yaml
 url_preview_accept_language:
   - 'en-UK'
   - 'en-US;q=0.9'
   - 'fr;q=0.8'
   - '*;q=0.7'
```
---
### `oembed`

oEmbed便于从网站中嵌入内容。它可用于生成支持服务的URL预览。Synapse包含默认的oEmbed提供者列表。设置`disable_default_providers`为true以禁用使用这些默认的oEmbed URL。使用`additional_providers`指定具有oEmbed配置的额外文件（每个文件应采用providers.json格式）。默认情况下该列表为空。
示例配置:
```yaml
oembed:
  disable_default_providers: true
  additional_providers:
    - oembed/my_providers.json
```
---
## Captcha

完整的验证码设置详情见[这里](../../CAPTCHA_SETUP.md)。

---
### `recaptcha_public_key`

此服务器的ReCAPTCHA公钥。如果启用了[`enable_registration_captcha`](#enable_registration_captcha)，必须指定。
示例配置:
```yaml
recaptcha_public_key: "YOUR_PUBLIC_KEY"
```
---
### `recaptcha_private_key`

此服务器的ReCAPTCHA私钥。如果启用了[`enable_registration_captcha`](#enable_registration_captcha)，必须指定。
示例配置:
```yaml
recaptcha_private_key: "YOUR_PRIVATE_KEY"
```
---
### `enable_registration_captcha`

设置为`true`以在用户注册帐户时要求完成CAPTCHA测试。需要有效的ReCaptcha公钥/私钥。默认为`false`。
请注意，[`enable_registration`](#enable_registration)也必须设置为允许帐户注册。
示例配置:
```yaml
enable_registration_captcha: true
```
---
### `recaptcha_siteverify_api`

用于验证`m.login.recaptcha`响应的API终端。默认为`https://www.recaptcha.net/recaptcha/api/siteverify`。
示例配置:
```yaml
recaptcha_siteverify_api: "https://my.recaptcha.site"
```
---
## TURN
对在Synapse中添加TURN服务器的选项。

---
### `turn_uris`

提供给客户的TURN服务器的公共URI。
示例配置:
```yaml
turn_uris: [turn:example.org]
```
---
### `turn_shared_secret`

用于计算TURN服务器密码的共享密钥。
示例配置:
```yaml
turn_shared_secret: "YOUR_SHARED_SECRET"
```
---
### `turn_shared_secret_path`

[`turn_shared_secret`](#turn_shared_secret)的替代方案：允许在外部文件中指定共享密钥。
该文件应为纯文本文件，仅包含共享密钥。Synapse在启动时从给定文件中读取共享密钥。
示例配置:
```yaml
turn_shared_secret_path: /path/to/secrets/file
```
_在Synapse 1.116.0中添加。_

---
### `turn_username` 和 `turn_password`

如果TURN服务器需要它们且不使用令牌，则使用的用户名和密码。
示例配置:
```yaml
turn_username: "TURNSERVER_USERNAME"
turn_password: "TURNSERVER_PASSWORD"
```
---
### `turn_user_lifetime`

生成的TURN凭证的有效时长。默认为1小时。
示例配置:
```yaml
turn_user_lifetime: 2h
```
---
### `turn_allow_guests`

是否允许访客使用TURN服务器。默认值为true，否则VoIP对访客将不可靠。然而，这确实引入了一点安全风险，因为它允许用户在未先注册有效帐户（例如通过CAPTCHA）的情况下连接到任意端点。
示例配置:
```yaml
turn_allow_guests: false
```
---
## Registration ##

注册可以使用本手册[Ratelimiting](#ratelimiting)部分中的参数进行速率限制。

---
### `enable_registration`

启用新用户注册。默认为`false`。
强烈建议如果启用注册，设置一个或多个以下选项，以避免您的服务器被"机器人"滥用：

 * [`enable_registration_captcha`](#enable_registration_captcha)
 * [`registrations_require_3pid`](#registrations_require_3pid)
 * [`registration_requires_token`](#registration_requires_token)

（为了启用没有任何验证的注册，还必须设置[`enable_registration_without_verification`](#enable_registration_without_verification)。）

请注意，即使此设置被禁用，如果设置了[`registration_shared_secret`](#registration_shared_secret)，通过admin API仍然可以创建新帐户。
示例配置:
```yaml
enable_registration: true
```
---
### `enable_registration_without_verification`

启用无电子邮件或验证码验证的注册。注意：此选项*不*建议使用，因为没有验证的注册已知是垃圾邮件和滥用的途径。默认为`false`。没有效果，除非[`enable_registration`](#enable_registration)也启用。
示例配置:
```yaml
enable_registration_without_verification: true
```
---
### `registrations_require_3pid`

如果设置此项，用户在注册帐户时必须提供所有指定类型的[3PID](https://spec.matrix.org/latest/appendices/#3pid-types)。
注意，[`enable_registration`](#enable_registration)也必须设置为允许帐户注册。
示例配置:
```yaml
registrations_require_3pid:
  - email
  - msisdn
```
---
### `disable_msisdn_registration`

明确禁用在注册流程中询问MSISDN（如果已将MSISDN设置为必需，该设置将覆盖`registrations_require_3pid`）。
示例配置:
```yaml
disable_msisdn_registration: true
```
---
### `allowed_local_3pids`

要求用户只能将指定格式的3PID与此服务器上的帐户关联，如通过`medium`和`pattern`子选项所规定的。`pattern`是一个[类Perl的正则表达式](https://docs.python.org/3/library/re.html#module-re)。
有关3PID、允许的`medium`类型及其`address`语法的更多信息，请参阅[Matrix spec](https://spec.matrix.org/latest/appendices/#3pid-types)。
示例配置:
```yaml
allowed_local_3pids:
  - medium: email
    pattern: '^[^@]+@matrix\.org$'
  - medium: email
    pattern: '^[^@]+@vector\.im$'
  - medium: msisdn
    pattern: '^44\d{10}$'
```
---
### `enable_3pid_lookup`

启用此服务器对身份服务器的3PID查找请求。默认为true。
示例配置:
```yaml
enable_3pid_lookup: false
```
---
### `registration_requires_token`

要求用户在注册期间提交一个令牌。可以使用管理[API](../administration/admin_api/registration_tokens.md)管理令牌。禁用此选项不会删除任何先前生成的令牌。默认为`false`。设置为`true`以启用。
注意，[`enable_registration`](#enable_registration)也必须设置为允许帐户注册。
示例配置:
```yaml
registration_requires_token: true
```
---
### `registration_shared_secret`

如果设置，允许任何持有共享密钥的人注册标准或管理帐户，即使没有设置[`enable_registration`](#enable_registration)。
这主要用于`register_new_matrix_user`脚本（参见[注册用户](../../setup/installation.md#registering-a-user)）；然而，该接口已[记录](../../admin_api/register_api.html)。
另见[`registration_shared_secret_path`](#registration_shared_secret_path)。
示例配置:
```yaml
registration_shared_secret: <PRIVATE STRING>
```
---
### `registration_shared_secret_path`

[`registration_shared_secret`](#registration_shared_secret)的替代方案：允许在外部文件中指定共享密钥。
该文件应为纯文本文件，仅包含共享密钥。
如果此文件不存在，Synapse将创建一个新的共享密钥，并在启动时将其存储在此文件中。
示例配置:
```yaml
registration_shared_secret_path: /path/to/secrets/file
```
_在Synapse 1.67.0中添加。_
---
### `bcrypt_rounds`

设置用于生成密码哈希的bcrypt轮数。较大的数字会增加生成哈希所需的工作因数。默认轮数为12（相当于2^12轮）。注意，增加此值会以指数速度增加注册或登录所需的时间 - 例如24 => 2^24轮将需要超过20分钟。
示例配置:
```yaml
bcrypt_rounds: 14
```
---
### `allow_guest_access`

允许用户以访客身份注册，无需密码/电子邮件等，并参加此服务器上已对匿名用户开放的房间。默认为false。
示例配置:
```yaml
allow_guest_access: true
```
---
### `default_identity_server`

我们建议用户在此服务器上登录时应该使用的身份服务器。
（默认情况下，没有建议，因此由客户端决定。此设置被忽略，除非`public_baseurl`也显式设置。）
示例配置:
```yaml
default_identity_server: https://matrix.org
```
---
### `account_threepid_delegates`

将电话号码验证委托给身份服务器。
当用户希望将电话号码添加到他们的帐户时，我们需要验证他们确实拥有该手机号，这需要发送一条短信（SMS）。目前Synapse不支持自己发送这些短信，而是将此任务委托给身份服务器。所使用身份服务器的基本URI由`account_threepid_delegates.msisdn`选项指定。
如果没有指定，Synapse将不允许用户将电话号码添加到他们的帐户。
（处理这些请求的服务器必须回答Matrix Identity Service API概述的`/requestToken`端点请求[规范说明](https://matrix.org/docs/spec/identity_service/latest)。）
_在Synapse 1.64.0中弃用_：`email`选项已弃用。
_在Synapse 1.66.0中移除_：`email`选项已被移除。若存在，Synapse将在启动时报告配置错误。
示例配置:
```yaml
account_threepid_delegates:
    msisdn: http://localhost:8090  # 将短信发送委托给此本地进程
```
---
### `enable_set_displayname`

允许用户在初次设置后更改其显示名称。基于第三方目录内容进行用户预配时很有用。
不适用于服务器管理员。默认为true。
示例配置:
```yaml
enable_set_displayname: false
```
---
### `enable_set_avatar_url`

允许用户在初次设置后更改其头像。基于第三方目录内容进行用户预配时很有用。
不适用于服务器管理员。默认为true。
示例配置:
```yaml
enable_set_avatar_url: false
```
---
### `enable_3pid_changes`

是否允许用户更改与其帐户关联的第三方ID（电子邮件地址和msisdn）。
默认为true。
示例配置:
```yaml
enable_3pid_changes: false
```
---
### `auto_join_rooms`

在此服务器上注册的用户将自动加入在此选项下列出的房间。
默认情况下，此列表中包含的任何房间别名将在homeserver上第一个用户注册时作为公开可加入房间创建。如果房间已经存在，请确保它是可公开加入的房间，即房间的加入规则必须设置为“公开”。有关自动加入房间的更多选项见下文。
由于空间在底层也是房间，因此也可以使用空间别名。
示例配置:
```yaml
auto_join_rooms:
  - "#exampleroom:example.com"
  - "#anotherexampleroom:example.com"
```
---
### `autocreate_auto_join_rooms`

在`auto_join_rooms`指定的情况下，设置此标志确保这些房间在homeserver上第一个用户注册时通过创建它们而存在。此选项不会创建空间。
默认情况下，自动创建的房间可从任何联邦服务器公开加入。使用`autocreate_auto_join_rooms_federated`和`autocreate_auto_join_room_preset`设置自定义此行为。
设置为false表示如果房间未手动创建，用户无法被自动加入，因为它们不存在。
默认为true。
示例配置:
```yaml
autocreate_auto_join_rooms: false
```
---
### `autocreate_auto_join_rooms_federated`

自动创建的在`auto_join_rooms`中列出的房间是否可通过联合访问。仅在`autocreate_auto_join_rooms`为真时报有效果。
请注意，房间创建后，其联合可访问性不能更改。
默认为true：房间可被来自其他服务器的用户加入。设置为false可防止其他homeserver用户加入这些房间。
示例配置:
```yaml
autocreate_auto_join_rooms_federated: false
```
---
### `autocreate_auto_join_room_preset`

自动创建`auto_join_rooms`房间时使用的房间预设。仅在`autocreate_auto_join_rooms`为真时生效。
此选项可能的值有:
* "public_chat": 房间对任何人（包括联合服务器，如果`autocreate_auto_join_rooms_federated`为真）开放加入（默认）。
* "private_chat": 加入这些房间需要邀请。
* "trusted_private_chat": 加入此房间需要邀请，且受邀者在加入后分配权级为100。
每个预设将以与调用时提供为`preset`参数相同的方式设置房间数据
[`POST /_matrix/client/v3/createRoom`](https://spec.matrix.org/latest/client-server-api/#post_matrixclientv3createroom)客户端-服务器API端点。
如果使用“private_chat”或“trusted_private_chat”的值，那么也必须配置`auto_join_mxid_localpart`。
默认为“public_chat”。
示例配置:
```yaml
autocreate_auto_join_room_preset: private_chat
```
---
### `auto_join_mxid_localpart`

当`autocreate_auto_join_rooms`为真时，创建`auto_join_rooms`房间所使用的用户id的本地部分。如果未提供，则将使用注册的第一个用户帐户创建这些房间。
该用户id也用于邀请新用户加入任何设置为仅限邀请的自动加入房间。
如果设置了`autocreate_auto_join_room_preset`为“private_chat”或“trusted_private_chat”，则必须配置。
请注意，若已有房间，该用户必须加入并拥有邀请新成员的适当权限。
示例配置:
```yaml
auto_join_mxid_localpart: system
```
---
### `auto_join_rooms_for_guests`

当指定`auto_join_rooms`时，设置此标志为false可阻止访客帐户被自动加入这些房间。
默认为true。
示例配置:
```yaml
auto_join_rooms_for_guests: false
```
---
### `inhibit_user_in_use_error`

是否抑制当注册新帐户时用户ID已存在时产生的错误。如果开启，向`/register/available`发出的请求将始终显示用户ID可用，而在使用已存在用户名完成注册时Synapse不会引发错误。
默认为false。
示例配置:
```yaml
inhibit_user_in_use_error: true
```