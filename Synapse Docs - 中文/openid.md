### 配置 Synapse 以使用 OpenID Connect 提供者进行身份验证

Synapse 可以配置为使用 OpenID Connect Provider (OP)进行身份验证，而不是使用其自己的本地密码数据库。

只要支持授权码流程，任何 OP 都应该能与 Synapse 一起工作。这里有几种选择：

*   启动一个本地 OP。Synapse 已经与 Hydra 和 Dex 进行了测试。注意，要使 OP 工作，它应该在安全的（HTTPS）源下提供服务。使用自签名、本地信任的 CA 签署的证书应该可以工作。在这种情况下，启动 Synapse 时，设置 `SSL_CERT_FILE` 环境变量为 CA 的路径。
*   设置一个 SaaS OP，如 Google、Auth0 或 Okta。Synapse 已经与 Auth0 和 Google 进行了测试。

也可以使用其他提供授权码授权类型的 OAuth2 提供者，例如 Github。

#### 准备 Synapse

Synapse 中的 OpenID 集成使用 `authlib` 库，必须按以下方式安装：

*   相关库已包含在 `matrix.org` 提供的 Docker 镜像和 Debian 包中，因此无需进一步操作。
*   如果您将 Synapse 安装到虚拟环境中，请运行 `/path/to/env/bin/pip install matrix-synapse[oidc]` 来安装必要的依赖项。
*   有关其他安装机制，请参阅维护者提供的文档。

要启用 OpenID 集成，您应该在配置文件中的 `oidc_providers` 设置中添加一个部分。请参阅配置手册以获取一些示例设置，以及下面的文本以获取特定提供者的示例配置。

#### OIDC 后台注销

Synapse 支持接收 OpenID Connect 后台注销通知。

这允许 OpenID Connect 提供者在用户注销时通知 Synapse，以便 Synapse 可以结束该用户会话。可以通过在提供者配置中将 `backchannel_logout_enabled` 属性设置为 `true` 来启用此功能，并在您的 OpenID Connect 提供者中将以下 URL 设置为后台注销通知的目的地： `[synapse public baseurl]/_synapse/client/oidc/backchannel_logout`

#### 示例配置

以下是一些适用于 Synapse 的提供者配置。

##### Microsoft Azure Active Directory

Azure AD 可以作为 OpenID Connect 提供者。在 Azure AD 管理控制台的应用注册中注册一个新应用。您的应用的重定向 URI 应指向您的矩阵服务器： `[synapse public baseurl]/_synapse/client/oidc/callback`

转到“证书和机密”并注册一个新的客户端机密。记下您的目录（租户）ID，因为它将在 Azure 链接中使用。编辑您的 Synapse 配置文件并更改 `oidc_config` 部分：

```
oidc_providers:
  - idp_id: microsoft
    idp_name: Microsoft
    issuer: "https://login.microsoftonline.com/<tenant id>/v2.0"
    client_id: "<client id>"
    client_secret: "<client secret>"
    scopes: ["openid", "profile"]
    authorization_endpoint: "https://login.microsoftonline.com/<tenant id>/oauth2/v2.0/authorize"
    token_endpoint: "https://login.microsoftonline.com/<tenant id>/oauth2/v2.0/token"
    userinfo_endpoint: "https://graph.microsoft.com/oidc/userinfo"

    user_mapping_provider:
      config:
        localpart_template: "{{ user.preferred_username.split('@')[0] }}"
        display_name_template: "{{ user.name }}"
```

##### 苹果

配置“使用 Apple 登录”（SiWA）需要一个 Apple 开发者账户。

您需要为 SiWA 创建一个新的“服务 ID”，并创建并下载一个启用了“SiWA”的私钥。

除了私钥文件，您还需要：

*   客户端 ID：您为“服务 ID”提供的“标识符”
*   团队 ID：与您的开发者账户关联的 10 个字符 ID。
*   密钥 ID：密钥的 10 个字符标识符。

Apple 的开发者文档中有更多关于设置 SiWA 的信息。

突触配置将如下所示：

```
  - idp_id: apple
    idp_name: Apple
    issuer: "https://appleid.apple.com"
    client_id: "your-client-id" # Set to the "identifier" for your "ServicesID"
    client_auth_method: "client_secret_post"
    client_secret_jwt_key:
      key_file: "/path/to/AuthKey_KEYIDCODE.p8"  # point to your key file
      jwt_header:
        alg: ES256
        kid: "KEYIDCODE"   # Set to the 10-char Key ID
      jwt_payload:
        iss: TEAMIDCODE    # Set to the 10-char Team ID
    scopes: ["name", "email", "openid"]
    authorization_endpoint: https://appleid.apple.com/auth/authorize?response_mode=form_post
    user_mapping_provider:
      config:
        email_template: "{{ user.email }}"
```

##### Auth0

Auth0 是一个托管的 SaaS 身份提供者解决方案。

1. 为 Synapse 创建一个常规的 Web 应用程序
2. 将允许的回调 URL 设置为 `[synapse public baseurl]/_synapse/client/oidc/callback`
3. 添加一个规则，名称任意，以添加 `preferred_username` 声明。 (有关如何创建规则的更多信息，请参见 https://auth0.com/docs/customize/rules/create-rules。)

代码示例

```
function addPersistenceAttribute(user, context, callback) {
  user.user_metadata = user.user_metadata || {};
  user.user_metadata.preferred_username = user.user_metadata.preferred_username || user.user_id;
  context.idToken.preferred_username = user.user_metadata.preferred_username;
```
  auth0.users.updateUserMetadata(user.user_id, user.user_metadata)
    .then(function(){
        callback(null, user, context);
    })
    .catch(function(err){
        callback(err);
    });
}

Synapse 配置：

```
oidc_providers:
  - idp_id: auth0
    idp_name: Auth0
    issuer: "https://your-tier.eu.auth0.com/" # TO BE FILLED
    client_id: "your-client-id" # TO BE FILLED
    client_secret: "your-client-secret" # TO BE FILLED
    scopes: ["openid", "profile"]
    user_mapping_provider:
      config:
        localpart_template: "{{ user.preferred_username }}"
        display_name_template: "{{ user.name }}"
```

##### Authentik

Authentik 是一个开源的身份提供者解决方案。

1. 在 Authentik 中创建一个提供者，类型为 OAuth2/OpenID。
2. 参数如下：

*   客户端类型：机密
*   JWT 算法：RS256
*   范围：OpenID、电子邮件和个人资料
*   RSA 密钥：选择任何可用的密钥
*   重定向 URI： `[synapse public baseurl]/_synapse/client/oidc/callback`

1. 在 Authentik 中为 synapse 创建一个应用程序并将其链接到提供者。
2. 注意您的应用程序的 slug、客户端 ID 和客户端密钥。

注意：Authentik 签名必须使用 RSA 密钥，ECC 密钥不适用。

Synapse 配置：

```
oidc_providers:
  - idp_id: authentik
    idp_name: authentik
    discover: true
    issuer: "https://your.authentik.example.org/application/o/your-app-slug/" # TO BE FILLED: domain and slug
    client_id: "your client id" # TO BE FILLED
    client_secret: "your client secret" # TO BE FILLED
    scopes:
      - "openid"
      - "profile"
      - "email"
    user_mapping_provider:
      config:
        localpart_template: "{{ user.preferred_username }}"
        display_name_template: "{{ user.preferred_username|capitalize }}" # TO BE FILLED: If your users have names in Authentik and you want those in Synapse, this should be replaced with user.name|capitalize.
```

##### Dex

Dex 是一个简单的、开源的 OpenID Connect 提供者。虽然它设计用于帮助构建一个完整的外部数据库提供者，但它也可以通过配置文件中的静态密码进行配置。

按照入门指南安装 Dex。

编辑 `examples/config-dev.yaml` 配置文件从 Dex 仓库中添加一个客户端：

```
staticClients:
- id: synapse
  secret: secret
  redirectURIs:
  - '[synapse public baseurl]/_synapse/client/oidc/callback'
  name: 'Synapse'
```

使用 `dex serve examples/config-dev.yaml` 运行。

Synapse 配置：

```
oidc_providers:
  - idp_id: dex
    idp_name: "My Dex server"
    skip_verification: true # This is needed as Dex is served on an insecure endpoint
    issuer: "http://127.0.0.1:5556/dex"
    client_id: "synapse"
    client_secret: "secret"
    scopes: ["openid", "profile"]
    user_mapping_provider:
      config:
        localpart_template: "{{ user.name }}"
        display_name_template: "{{ user.name|capitalize }}"
```

##### Django OAuth 工具包

django-oauth-toolkit 是一个 Django 应用程序，提供了开箱即用的所有端点、数据和逻辑，以向您的 Django 项目添加 OAuth2 功能。它也支持 OpenID Connect。

Django 端的配置：

1. 添加一个应用程序： `https://example.com/admin/oauth2_provider/application/add/` 并选择如下参数：

*   `Redirect uris`: `https://synapse.example.com/_synapse/client/oidc/callback`
*   `Client type`: `Confidential`
*   `Authorization grant type`: `Authorization code`
*   `Algorithm`: `HMAC with SHA-2 256`

1. 您可以自定义 Django 提供给 synapse 的声明（可选）：代码示例

```
class CustomOAuth2Validator(OAuth2Validator):
```
    def get_additional_claims(self, request):
        return {
            "sub": request.user.email,
            "email": request.user.email,
            "first_name": request.user.first_name,
            "last_name": request.user.last_name,
        }

然后您的 synapse 配置是：

```
oidc_providers:
  - idp_id: django_example
    idp_name: "Django Example"
    issuer: "https://example.com/o/"
    client_id: "your-client-id"  # CHANGE ME
    client_secret: "your-client-secret"  # CHANGE ME
    scopes: ["openid"]
    user_profile_method: "userinfo_endpoint"  # needed because oauth-toolkit does not include user information in the authorization response
    user_mapping_provider:
      config:
        localpart_template: "{{ user.email.split('@')[0] }}"
        display_name_template: "{{ user.first_name }} {{ user.last_name }}"
        email_template: "{{ user.email }}"
```

##### 脸书

1. 您将需要一个 Facebook 开发者账户。您可以在这里注册一个。
2. 在开发者控制台的应用页面上，选择“创建应用”，然后选择“构建连接体验”。
3. 创建应用后，添加“Facebook 登录”并选择“Web”。您不需要在这里填写整个表格。
4. 在左侧菜单中，打开“产品”/“Facebook 登录”/“设置”。
*   将 `[synapse public baseurl]/_synapse/client/oidc/callback` 添加为 OAuth 重定向 URL。
5. 在左侧菜单中，打开“设置/基本”。在这里你可以复制“应用 ID”和“应用密钥”以供下方使用。

Synapse 配置：

```
  - idp_id: facebook
    idp_name: Facebook
    idp_brand: "facebook"  # optional: styling hint for clients
    discover: false
    issuer: "https://www.facebook.com"
    client_id: "your-client-id" # TO BE FILLED
    client_secret: "your-client-secret" # TO BE FILLED
    scopes: ["openid", "email"]
    authorization_endpoint: "https://facebook.com/dialog/oauth"
    token_endpoint: "https://graph.facebook.com/v9.0/oauth/access_token"
    jwks_uri: "https://www.facebook.com/.well-known/oauth/openid/jwks/"
    user_mapping_provider:
      config:
        display_name_template: "{{ user.name }}"
        email_template: "{{ user.email }}"
```

相关文档：

*   [手动构建登录流程](https://developers.facebook.com/docs/facebook-login/manually-build-a-login-flow)
*   [使用 Facebook 的 Graph API](https://developers.facebook.com/docs/graph-api/using-graph-api/)
*   [对用户端点的引用](https://developers.facebook.com/docs/graph-api/reference/user)

Facebook 确实有一个 OIDC 发现端点，但它有一个 `response_types_supported` ，它排除了“code”（我们依赖的，并且在他们的文档中也有提到），所以我们必须禁用发现并手动配置 URI。

##### GitHub

GitHub 有点特殊，因为它不是一个 OpenID Connect 兼容的提供者，而只是一个普通的 OAuth2 提供者。

`/user` API 端点可用于检索已认证用户的信息。由于 Synapse 登录机制需要一个属性来唯一标识用户，而该端点不返回 `sub` 属性，因此必须设置一个替代的 `subject_claim` 。

1. 创建一个新的 OAuth 应用程序：https://github.com/settings/applications/new。
2. 将回调 URL 设置为 `[synapse public baseurl]/_synapse/client/oidc/callback` 。

Synapse 配置：

```
oidc_providers:
  - idp_id: github
    idp_name: Github
    idp_brand: "github"  # optional: styling hint for clients
    discover: false
    issuer: "https://github.com/"
    client_id: "your-client-id" # TO BE FILLED
    client_secret: "your-client-secret" # TO BE FILLED
    authorization_endpoint: "https://github.com/login/oauth/authorize"
    token_endpoint: "https://github.com/login/oauth/access_token"
    userinfo_endpoint: "https://api.github.com/user"
    scopes: ["read:user"]
    user_mapping_provider:
      config:
        subject_claim: "id"
        localpart_template: "{{ user.login }}"
        display_name_template: "{{ user.name }}"
```

##### GitLab

1. 创建一个新应用。
2. 添加 `read_user` 和 `openid` 作用域。
3. 添加此回调 URL： `[synapse public baseurl]/_synapse/client/oidc/callback`

Synapse 配置：

```
oidc_providers:
  - idp_id: gitlab
    idp_name: Gitlab
    idp_brand: "gitlab"  # optional: styling hint for clients
    issuer: "https://gitlab.com/"
    client_id: "your-client-id" # TO BE FILLED
    client_secret: "your-client-secret" # TO BE FILLED
    client_auth_method: "client_secret_post"
    scopes: ["openid", "read_user"]
    user_profile_method: "userinfo_endpoint"
    user_mapping_provider:
      config:
        localpart_template: '{{ user.nickname }}'
        display_name_template: '{{ user.name }}'
```

##### Gitea

Gitea 像 Github 一样，不是 OpenID 提供者，而只是一个 OAuth2 提供者。

`/user` API 端点可用于检索已认证用户的信息。由于 Synapse 登录机制需要一个属性来唯一标识用户，而该端点不返回 `sub` 属性，因此必须设置一个替代的 `subject_claim` 。

1. 创建一个新应用。
2. 添加此回调 URL： `[synapse public baseurl]/_synapse/client/oidc/callback`

Synapse 配置：

```
oidc_providers:
  - idp_id: gitea
    idp_name: Gitea
    discover: false
    issuer: "https://your-gitea.com/"
    client_id: "your-client-id" # TO BE FILLED
    client_secret: "your-client-secret" # TO BE FILLED
    client_auth_method: client_secret_post
    scopes: [] # Gitea doesn't support Scopes
    authorization_endpoint: "https://your-gitea.com/login/oauth/authorize"
    token_endpoint: "https://your-gitea.com/login/oauth/access_token"
    userinfo_endpoint: "https://your-gitea.com/api/v1/user"
    user_mapping_provider:
      config:
        subject_claim: "id"
        localpart_template: "{{ user.login }}"
        display_name_template: "{{ user.full_name }}"
```

##### 谷歌

Google 是经过 OpenID 认证的身份验证和授权提供者。

1. 在 Google API 控制台中设置一个项目（参见文档）。
2. 在“凭证”下为 Web 应用程序添加一个“OAuth 客户端 ID”。
3. 复制客户端 ID 和客户端密钥，并将以下内容添加到您的 synapse 配置中：

oidc_providers:
  - idp_id: google
    idp_name: Google
    idp_brand: "google"  # optional: styling hint for clients
    issuer: "https://accounts.google.com/"
    client_id: "your-client-id" # TO BE FILLED
    client_secret: "your-client-secret" # TO BE FILLED
    scopes: ["openid", "profile", "email"] # email is optional, read below
    user_mapping_provider:
      config:
        localpart_template: "{{ user.given_name|lower }}"
        display_name_template: "{{ user.name }}"
        email_template: "{{ user.email }}" # needs "email" in scopes above
4. 回到 Google 控制台，添加此授权重定向 URI： `[synapse public baseurl]/_synapse/client/oidc/callback` 。

##### Keycloak

Keycloak 是由 Red Hat 维护的开源身份提供者。

Keycloak 支持 OIDC 后台注销，它会向 Synapse 发送注销通知，这样当用户从 Keycloak 注销时，Synapse 用户也会被注销。这可以通过在 Synapse 配置中将 `backchannel_logout_enabled` 设置为 `true` ，并在 Keycloak 中设置“后台注销 URL”来选择性启用。

按照《入门指南》安装 Keycloak 并设置一个领域。

1. 点击侧边栏中的 `Clients` 然后点击 `Create`
2. 按以下内容填写字段：

| Field | Value |
|-----------|-----------|
| Client ID | `synapse` |
| Client Protocol | `openid-connect` |

3. 点击 `Save`
4. 按以下内容填写字段：

| Field | Value |
|-----------|-----------|
| Client ID | `synapse` |
| Enabled | `On` |
| Client Protocol | `openid-connect` |
| Access Type | `confidential` |
| Valid Redirect URIs | `[synapse public baseurl]/_synapse/client/oidc/callback` |
| Backchannel Logout URL (optional) | `[synapse public baseurl]/_synapse/client/oidc/backchannel_logout` |
| Backchannel Logout Session Required (optional) | `On` |

5. 点击 `Save`
6. 在“凭证”选项卡中，更新以下字段：

| Field | Value |
|-------|-------|
| Client Authenticator | `Client ID and Secret` |

7. 点击 `Regenerate Secret`
8. 复制密钥

```
oidc_providers:
  - idp_id: keycloak
    idp_name: "My KeyCloak server"
    issuer: "https://127.0.0.1:8443/realms/{realm_name}"
    client_id: "synapse"
    client_secret: "copy secret generated from above"
    scopes: ["openid", "profile"]
    user_mapping_provider:
      config:
        localpart_template: "{{ user.preferred_username }}"
        display_name_template: "{{ user.name }}"
    backchannel_logout_enabled: true # Optional
```

##### LemonLDAP

LemonLDAP::NG 是一个开源的身份提供者解决方案。

1. 在 LemonLDAP::NG 中创建 OpenID Connect 依赖方
2. 参数如下：

*   新依赖方（ `Options > Basic > Client ID` ）的基本菜单下的客户端 ID
*   客户端密钥 ( `Options > Basic > Client secret` )
*   JWT 算法：新依赖方（ `Options > Security > ID Token signature algorithm` 和 `Options > Security > Access Token signature algorithm` ）的安全菜单中的 RS256
*   范围：OpenID、电子邮件和个人资料
*   强制声明进入 `id_token` ( `Options > Advanced > Force claims to be returned in ID Token` )
*   允许的登录重定向地址 ( `Options > Basic > Allowed redirection addresses for login` ) : `[synapse public baseurl]/_synapse/client/oidc/callback`

Synapse 配置：

```
oidc_providers:
  - idp_id: lemonldap
    idp_name: lemonldap
    discover: true
    issuer: "https://auth.example.org/" # TO BE FILLED: replace with your domain
    client_id: "your client id" # TO BE FILLED
    client_secret: "your client secret" # TO BE FILLED
    scopes:
      - "openid"
      - "profile"
      - "email"
    user_mapping_provider:
      config:
        localpart_template: "{{ user.preferred_username }}}"
===        # TO BE FILLED: If your users have names in LemonLDAP::NG and you want those in Synapse, this should be replaced with user.name|capitalize or any valid filter.===
        display_name_template: "{{ user.preferred_username|capitalize }}"
```

##### Mastodon

Mastodon 实例提供 OAuth API，允许这些实例用作 Synapse 的单点登录提供者。

第一步是使用 Mastodon 实例的创建应用程序 API（另见此处）将 Synapse 注册为应用程序。有几种方法可以做到这一点，但在下面的示例中我们使用的是 CURL。

此示例假设：

*   Mastodon 实例网站 URL 是 `https://your.mastodon.instance.url` ，并且
*   Synapse 将被注册为名为 `my_synapse_app` 的应用程序。

发送以下请求，将您的 Synapse 安装中的 `synapse_public_baseurl` 的值替换进去。

```
curl -d "client_name=my_synapse_app&redirect_uris=https://[synapse_public_baseurl]/_synapse/client/oidc/callback" -X POST https://your.mastodon.instance.url/api/v1/apps
```

您应该会收到类似于以下的响应。请务必保存。

```json
{"client_id":"someclientid_123","client_secret":"someclientsecret_123","id":"12345","name":"my_synapse_app","redirect_uri":"https://[synapse_public_baseurl]/_synapse/client/oidc/callback","website":null,"vapid_key":"somerandomvapidkey_123"}
```

由于 Synapse 登录机制需要一个属性来唯一标识用户，而 Mastodon 的端点不返回 `sub` 属性，因此必须设置一个替代的 `subject_template` 。您的 Synapse 配置应包括以下内容：

```
oidc_providers:
  - idp_id: my_mastodon
    idp_name: "Mastodon Instance Example"
    discover: false
    issuer: "https://your.mastodon.instance.url/@admin"
    client_id: "someclientid_123"    
    client_secret: "someclientsecret_123"
    authorization_endpoint: "https://your.mastodon.instance.url/oauth/authorize"
    token_endpoint: "https://your.mastodon.instance.url/oauth/token"
    userinfo_endpoint: "https://your.mastodon.instance.url/api/v1/accounts/verify_credentials"
    scopes: ["read"]
    user_mapping_provider:
      config:
        subject_template: "{{ user.id }}"
        localpart_template: "{{ user.username }}"
        display_name_template: "{{ user.display_name }}"
```

请注意，字段 `client_id` 和 `client_secret` 是从上面的 CURL 响应中获取的。

##### Shibboleth 与 OIDC 插件

Shibboleth 是一个被大学广泛使用的开放标准身份提供者解决方案。

1. Shibboleth 需要安装并正确运行 OIDC 插件。
2. 在 IdP 端创建一个新的配置，确保 `client_id` 和 `client_secret` 是随机生成的数据。

```json
{
    "client_id": "SOME-CLIENT-ID",
    "client_secret": "SOME-SUPER-SECRET-SECRET",
    "response_types": ["code"],
    "grant_types": ["authorization_code"],
    "scope": "openid profile email",
    "redirect_uris": ["https://[synapse public baseurl]/_synapse/client/oidc/callback"]
}
```

Synapse 配置：

```
oidc_providers:
===  # Shibboleth IDP===
  #
  - idp_id: shibboleth
    idp_name: "Shibboleth Login"
    discover: true
    issuer: "https://YOUR-IDP-URL.TLD"
    client_id: "YOUR_CLIENT_ID"
    client_secret: "YOUR-CLIENT-SECRECT-FROM-YOUR-IDP"
    scopes: ["openid", "profile", "email"]
    allow_existing_users: true
    user_profile_method: "userinfo_endpoint"
    user_mapping_provider:
      config:
        subject_claim: "sub"
        localpart_template: "{{ user.sub.split('@')[0] }}"
        display_name_template: "{{ user.name }}"
        email_template: "{{ user.email }}"
```

##### Twitch

1. 在 Twitch 上设置开发者账户
2. 通过创建一个应用获取 OAuth 2.0 凭证
3. 添加此 OAuth 重定向 URL： `[synapse public baseurl]/_synapse/client/oidc/callback`

Synapse 配置：

```
oidc_providers:
  - idp_id: twitch
    idp_name: Twitch
    issuer: "https://id.twitch.tv/oauth2/"
    client_id: "your-client-id" # TO BE FILLED
    client_secret: "your-client-secret" # TO BE FILLED
    client_auth_method: "client_secret_post"
    user_mapping_provider:
      config:
        localpart_template: "{{ user.preferred_username }}"
        display_name_template: "{{ user.name }}"
```

##### 推特

_使用 Twitter 作为身份提供者需要使用 Synapse 1.75.0 或更高版本。_

1. 在 Twitter 上设置开发者账户
2. 创建项目和应用。
3. 启用用户认证，并在“应用类型”下选择“Web 应用、自动化应用或机器人”。
4. 在“应用信息”下将回调 URL 设置为 `[synapse public baseurl]/_synapse/client/oidc/callback` 。
5. 在“密钥和令牌”选项卡下获取 OAuth 2.0 凭证，复制“OAuth 2.0 客户端 ID 和客户端密钥”

Synapse 配置：

```
oidc_providers:
  - idp_id: twitter
    idp_name: Twitter
    idp_brand: "twitter"  # optional: styling hint for clients
    discover: false  # Twitter is not OpenID compliant.
    issuer: "https://twitter.com/"
    client_id: "your-client-id" # TO BE FILLED
    client_secret: "your-client-secret" # TO BE FILLED
    pkce_method: "always"
===    # offline.access providers refresh tokens, tweet.read and users.read needed for userinfo request.===
    scopes: ["offline.access", "tweet.read", "users.read"]
    authorization_endpoint: https://twitter.com/i/oauth2/authorize
    token_endpoint: https://api.twitter.com/2/oauth2/token
    userinfo_endpoint: https://api.twitter.com/2/users/me?user.fields=profile_image_url
    user_mapping_provider:
      config:
        subject_template: "{{ user.data.id }}"
        localpart_template: "{{ user.data.username }}"
        display_name_template: "{{ user.data.name }}"
        picture_template: "{{ user.data.profile_image_url }}"
```

##### XWiki

在您的 XWiki 实例中安装 OpenID Connect Provider 扩展。

Synapse 配置：

```
oidc_providers:
  - idp_id: xwiki
    idp_name: "XWiki"
    issuer: "https://myxwikihost/xwiki/oidc/"
    client_id: "your-client-id" # TO BE FILLED
    client_auth_method: none
    scopes: ["openid", "profile"]
    user_profile_method: "userinfo_endpoint"
    user_mapping_provider:
      config:
        localpart_template: "{{ user.preferred_username }}"
        display_name_template: "{{ user.name }}"
```