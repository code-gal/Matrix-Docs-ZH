解码JSON Web令牌的内容。当`enabled`设置为true时必需提供。  
* `algorithm`：用于签署（或HMAC）JSON Web令牌的算法。  
  支持的算法列表在[此处（JWS部分）](https://docs.authlib.org/en/latest/specs/rfc7518.html)。  
  当`enabled`设置为true时必需提供。  
* `subject_claim`：包含用户唯一标识的声明的名称。  
  可选，默认值为`sub`。  
* `display_name_claim`：包含用户显示名称的声明名称。可选。  
  如果提供，显示名称将在第一次登录时设置为此声明的值。  
* `issuer`：用以验证"iss"声明的发行者。可选。如果提供，  
  "iss"声明将会是必需且所有JSON web令牌都将进行验证。  
* `audiences`：验证"aud"声明的受众列表。可选。  
  如果提供，则"aud"声明将是必需的并且针对所有JSON web令牌进行验证。  
  注意，如果在JSON web令牌中包含"aud"声明，  
  则在未配置audiences的情况下验证将失败。  

配置示例：  
```yaml  
jwt_config:  
    enabled: true  
    secret: "provided-by-your-issuer"  
    algorithm: "provided-by-your-issuer"  
    subject_claim: "name_of_claim"  
    display_name_claim: "name_of_claim"  
    issuer: "provided-by-your-issuer"  
    audiences:  
        - "provided-by-your-issuer"  
```  
---  
### `password_config`  

使用此设置启用基于密码的登录。  

此设置具有以下子选项：  
* `enabled`：默认值为true。  
   设置为false以禁用密码认证。  
   设置为`only_for_reauth`时允许持有现有密码的用户用其重新认证（而不是登录），以防止新用户设置密码。  
* `localdb_enabled`：设置为false以禁用本地密码  
   数据库的身份验证。如果`enabled`为false，则忽略此项，且仅在拥有其他`password_providers`时有用。默认值为true。  
* `pepper`：在此处设置一个秘密随机字符串以提高安全性。  
   初次设置后请勿更改！  
* `policy`：定义并实施密码策略，如最小密码长度等。  
   每个参数都是可选的。这是MSC2000的实现。参数如下：  
   * `enabled`：默认值为false。设置为true以启用。  
   * `minimum_length`：密码接受的最小长度。默认值为0。  
   * `require_digit`：密码是否必须包含至少一个数字。  
      默认值为false。  
   * `require_symbol`：密码是否必须包含至少一个符号。  
      符号是非数字或字母的任何字符。默认值为false。  
   * `require_lowercase`：密码是否必须包含至少一个小写字母。  
      默认值为false。  
   * `require_uppercase`：密码是否必须包含至少一个大写字母。  
      默认值为false。  

配置示例：  
```yaml  
password_config:  
   enabled: false  
   localdb_enabled: false  
   pepper: "EVEN_MORE_SECRET"  

   policy:  
      enabled: true  
      minimum_length: 15  
      require_digit: true  
      require_symbol: true  
      require_lowercase: true  
      require_uppercase: true  
```  
---  
## 推送  
与推送通知相关的配置设置

---  
### `push`  

此设置定义了推送通知的选项。  

此选项具有多个子选项，具体如下：  
* `enabled`：启用或禁用推送通知计算。注意，禁用此功能还将  
   停止计算房间的未读计数。此操作模式适用于可能  
   仅连接有机器人或应用服务用户的家庭服务器，或其他不关注推送/未读计数的服务器。默认情况下启用。  
* `include_content`：请求推送通知的客户端可以在通知请求中  
   发送消息的正文以及其他细节（如发送者），或者只发送事件ID和房间ID（`event_id_only`）。  
   如果客户端选择发送正文，则此选项控制 
   通知请求是否包括事件的内容（仍然包括其他细节如发送者）。如果`event_id_only`启用，  
   则此项无效。  
   对于现代安卓设备，通知内容仍会出现，  
   因为它是通过应用加载的。然而iPhone，  
   只会发出通知说有一条消息到了，提示是谁发来的。  
   默认值为true。设置为false，只在推送通知中包含事件ID和房间ID。  
* `group_unread_count_by_room: false`：收到推送通知时，也会发送未读计数。  
   此数值可以计算为用户的未读消息数量，或者是*房间*的数量  
   用户在其中有未读消息。默认值为true，意味着推送客户端将在其中看到未读消息的房间数量。  
   设置为false则换为发送未读消息的数量。  
* `jitter_delay`：将推送通知延迟一段最长为给定时间的随机时长。  
  有助于减轻时间攻击。可选，默认不延迟。_在Synapse 1.84.0中添加。_  

配置示例：  
```yaml  
push:  
  enabled: true  
  include_content: false  
  group_unread_count_by_room: false  
  jitter_delay: "10s"  
```  
---  
## 房间  
与房间相关的配置选项。

---  
### `encryption_enabled_by_default_for_room_type`  

控制本地创建的房间是否默认启用端到端加密。  

可能的选项为"all"、"invite"和"off"。其定义为：  

* "all"：任何本地创建的房间  
* "invite"：任何使用`private_chat`或`trusted_private_chat`  
   房间创建预设创建的房间  
* "off"：此选项不生效  

默认值为"off"。  

请注意，此选项仅影响设置后创建的房间。  
此外，它不会影响其他服务器创建的房间。  

配置示例：  
```yaml  
encryption_enabled_by_default_for_room_type: invite  
```  
---  
### `user_directory`  

此设置定义了与用户目录相关的选项。  

此选项具有以下子选项：  
* `enabled`：定义用户是否可以搜索用户目录。如果为false则  
   所有查询都将返回空响应。默认值为true。  
* `search_all_users`：定义是否搜索在搜索执行时对你的家庭服务器可见的所有用户。  
   如果设置为true，将返回所有已知与搜索查询匹配的用户。  
   如果为false，搜索结果仅包含对请求者可见的公共房间或共享房间的用户。  
   默认值为false。  

    注意：如果你设置此项为true，而上次重建user_directory搜索  
    索引是在Synapse 1.44之前，你需要  
    通过API手动触发重建以搜索所有已知用户。  

    这些索引在Synapse第一次启动时创建；管理员可以  
    按照  
    [运行后台更新的说明](../administration/admin_api/background_updates.md#run)手动触发重建  
    设置为true以返回包括所有已知用户的搜索结果，即使该用户没有与请求者共享房间。  
* `prefer_local_users`：定义在搜索查询结果中是否更偏向本地用户。  
   如果设置为true，本地用户在搜索用户目录时更有可能出现在远程用户之前。默认值为false。  
* `show_locked_users`：定义是否在搜索查询结果中显示被锁定的用户。默认值为false。  

配置示例：  
```yaml  
user_directory:  
    enabled: false  
    search_all_users: true  
    prefer_local_users: true  
    show_locked_users: true  
```  
---  
### `user_consent`  

有关用户同意配置的详细说明，请参见[这里](../../consent_tracking.md)。  

如果在`listeners`下启用`consent`资源，则本节的部分内容是必需的，特别是`template_dir`和`version`。  

* `template_dir`：指定HTML表单的模板所在的位置。  
  此目录应包含一个语言子目录（如，`en`，`fr`），  
  每个语言目录应包含政策文件（命名为  
  <version>.html）和一个成功页面（success.html）。  

* `version`：指定政策文件的"当前"版本。定义  
   必要时由同意资源提供的版本。  

* `server_notice_content`，如果启用，将向用户发送一个"服务器通知"  
   请求他们同意隐私政策。[`server_notices`部分](#server_notices)  
   也必须配置。通知将*不*发给客人用户，除非`s

end_server_notice_to_guests`设置为true。

* 如果设置`block_events_error`，则会阻止任何发送事件的尝试，
   直到用户同意隐私政策。设置的值将作为错误的文本使用。

* `require_at_registration`，如果启用，将会在注册过程中增加一个步骤，
   类似于验证码工作方式。用户必须在账户创建前同意政策。

* `policy_name`是用户在注册账户时看到的政策显示名称。仅在启用`require_at_registration`时有效。
   默认值为"Privacy Policy"。

配置示例：
```yaml
user_consent:
  template_dir: res/templates/privacy
  version: 1.0
  server_notice_content:
    msgtype: m.text
    body: >-
      要继续使用此家庭服务器，您必须查看并同意
      在%(consent_uri)s的条款和条件
  send_server_notice_to_guests: true
  block_events_error: >-
    要继续使用此家庭服务器，您必须查看并同意
    在%(consent_uri)s的条款和条件
  require_at_registration: false
  policy_name: Privacy Policy
```
---  
### `stats`  

用于本地房间和用户统计信息收集的设置。请参见[这里](../../room_and_user_statistics.md)  
查看更多。

* `enabled`：设置为false以禁用房间和用户统计信息。注意，  
   这可能导致某些功能（如房间目录）无法正常工作。默认值为true。  

配置示例：  
```yaml  
stats:  
  enabled: false  
```  
---  
### `server_notices`  

使用此设置可以启用一个房间，以便从服务器向用户发送通知。  
这是一个用户无法离开的特殊房间；房间内的通知来自一个特殊的"notices"用户ID。  

如果你使用此设置，你*必须*定义`system_mxid_localpart`  
子设置，该设置定义用于发送通知的用户ID。  

此设置的子选项包括：  
* `system_mxid_display_name`：设置"notices"用户的显示名称  
* `system_mxid_avatar_url`：设置"notices"用户的头像  
* `room_name`：设置服务器通知房间的房间名称  
* `room_avatar_url`：可选字符串。服务器通知房间使用的房间头像。如果设置为空字符串`""`，则通知房间将不会有头像。默认值为空字符串。_在Synapse 1.99.0中添加。_  
* `room_topic`：可选字符串。服务器通知房间使用的主题。如果设置为空字符串`""`，则通知房间将不会有主题。默认值为空字符串。_在Synapse 1.99.0中添加。_  
* `auto_join`：布尔值。如果为true，用户将自动加入房间而不是被邀请。  
  默认值为false。_在Synapse 1.98.0中添加。_  

请注意，现有的服务器通知房间的名称、主题和头像将仅在发送新通知事件时更新。  

配置示例：  
```yaml  
server_notices:  
  system_mxid_localpart: notices  
  system_mxid_display_name: "Server Notices"  
  system_mxid_avatar_url: "mxc://example.com/oumMVlgDnLYFaPVkExemNVVZ"  
  room_name: "Server Notices"  
  room_avatar_url: "mxc://example.com/oumMVlgDnLYFaPVkExemNVVZ"  
  room_topic: "Room used by your server admin to notice you of important information"  
  auto_join: true  
```  
---  
### `enable_room_list_search`  

设置为false以禁用搜索公共房间列表。当禁用时，  
通过始终为所有查询返回空列表来禁止搜索本地和远程房间列表的本地和远程用户。默认值为true。  

配置示例：  
```yaml  
enable_room_list_search: false  
```  
---  
### `alias_creation_rules`  

`alias_creation_rules`选项允许服务器管理员防止不必要的  
在此服务器上创建别名。  

此设置是一个可选的0个或多个规则列表。默认情况下，没有提供列表，  
意味着允许所有别名创建。  

否则，创建别名的请求将按顺序匹配每个规则。  
第一个匹配的规则决定请求是否被允许或拒绝。如果没有  
规则匹配，请求被拒绝。特别注意的是，配置  
一个空规则列表将拒绝所有别名创建请求。 

每个规则是一个YAML对象，包含四个字段，每个字段都是可选的字符串：

* `user_id`：一个与别名创建者匹配的通配符模式。
* `alias`：一个与正在创建的别名匹配的通配符模式。
* `room_id`：一个与别名指向的房间ID匹配的通配符模式。
* `action`：`allow`或`deny`。如果规则匹配，如何处理请求。默认值为`allow`。

每个通配符模式是可选的，默认值为`*`（匹配任何东西）。
注意，模式是与完整合格ID匹配的，例如与
`@alice:example.com`、`#room:example.com` 和`!abcdefghijk:example.com`而不是`alice`、`room`和`abcedgghijk`。

配置示例：

```yaml
# 未指定规则列表。所有别名创建均被允许。
# 这是默认行为。
alias_creation_rules:
```

```yaml
# 允许所有内容的单一规则列表。
# 与前一个示例具有相同效果。
alias_creation_rules:
  - "action": "allow"
```

```yaml
# 空规则列表。所有别名创建均被拒绝。
alias_creation_rules: []
```

```yaml
# 规则列表中一个部分拒绝所有内容。
# 与前一个示例具有相同效果。
alias_creation_rules:
  - "action": "deny"
```

```yaml
# 防止一个特定用户创建别名。
# 允许其他用户创建任何别名
alias_creation_rules:
  - user_id: "@bad_user:example.com"
    action: deny

  - action: allow
```

```yaml
# 阻止指向特定房间的别名。
alias_creation_rules:
  - room_id: "!forbiddenRoom:example.com"
    action: deny

  - action: allow
```

---  
### `room_list_publication_rules`  

`room_list_publication_rules`选项允许服务器管理员防止  
不需要的条目被发布到公共房间列表中。  

该选项的格式与  
[`alias_creation_rules`](#alias_creation_rules)相同：一个可选的0个或多个  
规则列表。默认情况下，没有提供列表，意味着所有房间  
都可以发布到房间列表中。  

否则，发布房间的请求将按顺序匹配每个规则。  
第一个匹配的规则决定请求是否被允许或拒绝。如果没有  
规则匹配，请求被拒绝。特别注意的是，配置  
一个空规则列表将拒绝所有别名创建请求。  

请求创建一个违反  
配置规则的公开（即发布到房间目录）的房间将导致  
房间被创建但不会发布到房间目录。  

每个规则是一个YAML对象，包含四个字段，每个字段都是可选的字符串：

* `user_id`：一个与发布房间的用户匹配的通配符模式。
* `alias`：一个与发布房间的别名之一匹配的通配符模式。
  - 如果房间没有别名，除非未指定或为`*`，否则别名匹配失败。
  - 如果房间仅有一个别名，别名匹配，如果`alias`模式匹配，则成功。
  - 如果房间有两个或多个别名，别名匹配，如果模式匹配多个别名中的至少一个，则成功。
* `room_id`：一个与被发布房间的房间ID匹配的通配符模式。
* `action`：`allow`或`deny`。如果规则匹配，如何处理请求。默认值为`allow`。

每个通配符模式是可选的，默认值为`*`（匹配任何东西）。
注意，模式是与完整合格ID匹配的，例如与
`@alice:example.com`、`#room:example.com` 和`!abcdefghijk:example.com`而不是`alice`、`room`和`abcedgghijk`。

示例配置：

```yaml
# 未指定规则列表。任何人都可以将任何房间发布到公共列表。
# 这是默认行为。
room_list_publication_rules:
```

```yaml
# 允许所有内容的单一规则列表。
# 与前一个示例具有相同效果。
room_list_publication_rules:
  - "action": "allow"
```

```yaml
# 空规则列表。无人可发布到房间列表。
room_list_publication_rules: []
```

```yaml
# 规则列表中一个部分拒绝所有内容。
# 与前一个示例具有相同效果。
room_list_publication_rules:
  - "action": "deny"
```

```yaml
# 阻止一个特定用户发布房间。
# 允许其他用户发布任何内容。
room_list_publication_rules:
  - user_id: "@bad_user:example.com"
    action: deny

  - action: allow
```

```yaml
# 防止发布特定房间。
room_list_publication_rules:
  - room_id: "!forbiddenRoom:example.com"
    action: deny

  - action: allow
```

```yaml
# 防止包含"potato"字样的别名之一的房间发布。
room_list_publication_rules:
  - alias: "#*potato*:example.com"
    action: deny

  - action: allow
```

---  
### `default_power_level_content_override`  

`default_power_level_content_override`选项控制房间的默认权限等级。  

如果您知道您的用户在他们创建的房间内需要特殊权限（例如，发送特定类型的状态事件而无需提升权限等级），则非常有用。此选项与/createRoom API中的`power_level_content_override`参数使用相同的结构，但在该参数之前应用。

请注意，在预设中每个提供的键（例如，下面示例中的`events`）将覆盖该键中的所有现有默认值。因此在下面的示例中，新创建的private_chat房间将不会有除`com.example.foo`外其他事件类型的规则。  

示例配置：  
```yaml  
default_power_level_content_override:  
   private_chat: { "events": { "com.example.foo" : 0 } }  
   trusted_private_chat: null  
   public_chat: null  
```  

每个预设的默认权限等级为：  
```yaml  
"m.room.name": 50  
"m.room.power_levels": 100  
"m.room.history_visibility": 100  
"m.room.canonical_alias": 50  
"m.room.avatar": 50  
"m.room.tombstone": 100  
"m.room.server_acl": 100  
"m.room.encryption": 100  
```  

因此一个保持每个预设的默认权限等级但为新键设置权限等级的完整示例为：  
```yaml  
default_power_level_content_override:  
   private_chat:  
    events:  
      "com.example.foo": 0  
      "m.room.name": 50  
      "m.room.power_levels": 100  
      "m.room.history_visibility": 100  
      "m.room.canonical_alias": 50  
```