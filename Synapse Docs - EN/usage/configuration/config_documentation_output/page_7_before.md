   Most deployments only have a single IdP entity and so should omit this option.


Once SAML support is enabled, a metadata file will be exposed at
`https://<server>:<port>/_synapse/client/saml2/metadata.xml`, which you may be able to
use to configure your SAML IdP with. Alternatively, you can manually configure
the IdP to use an ACS location of
`https://<server>:<port>/_synapse/client/saml2/authn_response`.

Example configuration:
```yaml
saml2_config:
  sp_config:
    metadata:
      local: ["saml2/idp.xml"]
      remote:
        - url: https://our_idp/metadata.xml
    accepted_time_diff: 3

    service:
      sp:
        allow_unsolicited: true

    # The examples below are just used to generate our metadata xml, and you
    # may well not need them, depending on your setup. Alternatively you
    # may need a whole lot more detail - see the pysaml2 docs!
    description: ["My awesome SP", "en"]
    name: ["Test SP", "en"]

    ui_info:
      display_name:
        - lang: en
          text: "Display Name is the descriptive name of your service."
      description:
        - lang: en
          text: "Description should be a short paragraph explaining the purpose of the service."
      information_url:
        - lang: en
          text: "https://example.com/terms-of-service"
      privacy_statement_url:
        - lang: en
          text: "https://example.com/privacy-policy"
      keywords:
        - lang: en
          text: ["Matrix", "Element"]
      logo:
        - lang: en
          text: "https://example.com/logo.svg"
          width: "200"
          height: "80"

    organization:
      name: Example com
      display_name:
        - ["Example co", "en"]
      url: "http://example.com"

    contact_person:
      - given_name: Bob
        sur_name: "the Sysadmin"
        email_address: ["admin@example.com"]
        contact_type: technical

  saml_session_lifetime: 5m

  user_mapping_provider:
    # Below options are intended for the built-in provider, they should be
    # changed if using a custom module.
    config:
      mxid_source_attribute: displayName
      mxid_mapping: dotreplace

  grandfathered_mxid_source_attribute: upn

  attribute_requirements:
    - attribute: userGroup
      value: "staff"
    - attribute: department
      value: "sales"

  idp_entityid: 'https://our_idp/entityid'
```
---
### `oidc_providers`

List of OpenID Connect (OIDC) / OAuth 2.0 identity providers, for registration
and login. See [here](../../openid.md)
for information on how to configure these options.

For backwards compatibility, it is also possible to configure a single OIDC
provider via an `oidc_config` setting. This is now deprecated and admins are
advised to migrate to the `oidc_providers` format. (When doing that migration,
use `oidc` for the `idp_id` to ensure that existing users continue to be
recognised.)

Options for each entry include:
* `idp_id`: a unique identifier for this identity provider. Used internally
   by Synapse; should be a single word such as 'github'.
   Note that, if this is changed, users authenticating via that provider
   will no longer be recognised as the same user!
   (Use "oidc" here if you are migrating from an old `oidc_config` configuration.)

* `idp_name`: A user-facing name for this identity provider, which is used to
   offer the user a choice of login mechanisms.

* `idp_icon`: An optional icon for this identity provider, which is presented
   by clients and Synapse's own IdP picker page. If given, must be an
   MXC URI of the format `mxc://<server-name>/<media-id>`. (An easy way to
   obtain such an MXC URI is to upload an image to an (unencrypted) room
   and then copy the "url" from the source of the event.)

* `idp_brand`: An optional brand for this identity provider, allowing clients
   to style the login flow according to the identity provider in question.
   See the [spec](https://spec.matrix.org/latest/) for possible options here.

* `discover`: set to false to disable the use of the OIDC discovery mechanism
  to discover endpoints. Defaults to true.

* `issuer`: Required. The OIDC issuer. Used to validate tokens and (if discovery
   is enabled) to discover the provider's endpoints.

* `client_id`: Required. oauth2 client id to use.

* `client_secret`: oauth2 client secret to use. May be omitted if
  `client_secret_jwt_key` is given, or if `client_auth_method` is 'none'.
  Must be omitted if `client_secret_path` is specified.

* `client_secret_path`: path to the oauth2 client secret to use. With that
   it's not necessary to leak secrets into the config file itself.
   Mutually exclusive with `client_secret`. Can be omitted if
   `client_secret_jwt_key` is specified.

   *Added in Synapse 1.91.0.*

* `client_secret_jwt_key`: Alternative to client_secret: details of a key used
   to create a JSON Web Token to be used as an OAuth2 client secret. If
   given, must be a dictionary with the following properties:

  * `key`: a pem-encoded signing key. Must be a suitable key for the
    algorithm specified. Required unless `key_file` is given.

  * `key_file`: the path to file containing a pem-encoded signing key file.
     Required unless `key` is given.

  * `jwt_header`: a dictionary giving properties to include in the JWT
     header. Must include the key `alg`, giving the algorithm used to
     sign the JWT, such as "ES256", using the JWA identifiers in
     RFC7518.

  * `jwt_payload`: an optional dictionary giving properties to include in
    the JWT payload. Normally this should include an `iss` key.

* `client_auth_method`: auth method to use when exchanging the token. Valid
   values are `client_secret_basic` (default), `client_secret_post` and
   `none`.

* `pkce_method`: Whether to use proof key for code exchange when requesting
   and exchanging the token. Valid values are: `auto`, `always`, or `never`. Defaults
   to `auto`, which uses PKCE if supported during metadata discovery. Set to `always`
   to force enable PKCE or `never` to force disable PKCE.

* `scopes`: list of scopes to request. This should normally include the "openid"
   scope. Defaults to `["openid"]`.

* `authorization_endpoint`: the oauth2 authorization endpoint. Required if
   provider discovery is disabled.

* `token_endpoint`: the oauth2 token endpoint. Required if provider discovery is
   disabled.

* `userinfo_endpoint`: the OIDC userinfo endpoint. Required if discovery is
   disabled and the 'openid' scope is not requested.

* `jwks_uri`: URI where to fetch the JWKS. Required if discovery is disabled and
   the 'openid' scope is used.

* `skip_verification`: set to 'true' to skip metadata verification. Use this if
   you are connecting to a provider that is not OpenID Connect compliant.
   Defaults to false. Avoid this in production.

* `user_profile_method`: Whether to fetch the user profile from the userinfo
   endpoint, or to rely on the data returned in the id_token from the `token_endpoint`.
   Valid values are: `auto` or `userinfo_endpoint`.
   Defaults to `auto`, which uses the userinfo endpoint if `openid` is
   not included in `scopes`. Set to `userinfo_endpoint` to always use the
   userinfo endpoint.

* `additional_authorization_parameters`: String to string dictionary that will be passed as
   additional parameters to the authorization grant URL.

* `allow_existing_users`: set to true to allow a user logging in via OIDC to
   match a pre-existing account instead of failing. This could be used if
   switching from password logins to OIDC. Defaults to false.

* `enable_registration`: set to 'false' to disable automatic registration of new
   users. This allows the OIDC SSO flow to be limited to sign in only, rather than
   automatically registering users that have a valid SSO login but do not have
   a pre-registered account. Defaults to true.

* `user_mapping_provider`: Configuration for how attributes returned from a OIDC
   provider are mapped onto a matrix user. This setting has the following
   sub-properties:

     * `module`: The class name of a custom mapping module. Default is
       `synapse.handlers.oidc.JinjaOidcMappingProvider`.
        See [OpenID Mapping Providers](../../sso_mapping_providers.md#openid-mapping-providers)
        for information on implementing a custom mapping provider.

     * `config`: Configuration for the mapping provider module. This section will
        be passed as a Python dictionary to the user mapping provider
        module's `parse_config` method.

        For the default provider, the following settings are available:

       * `subject_template`: Jinja2 template for a unique identifier for the user.
         Defaults to `{{ user.sub }}`, which OpenID Connect compliant providers should provide.

         This replaces and overrides `subject_claim`.

       * `subject_claim`: name of the claim containing a unique identifier
         for the user. Defaults to 'sub', which OpenID Connect
         compliant providers should provide.

         *Deprecated in Synapse v1.75.0.*

       * `picture_template`: Jinja2 template for an url for the user's profile picture.
         Defaults to `{{ user.picture }}`, which OpenID Connect compliant providers should
         provide and has to refer to a direct image file such as PNG, JPEG, or GIF image file.

         This replaces and overrides `picture_claim`.

         Currently only supported in monolithic (single-process) server configurations
         where the media repository runs within the Synapse process.

       * `picture_claim`: name of the claim containing an url for the user's profile picture.
         Defaults to 'picture', which OpenID Connect compliant providers should provide
         and has to refer to a direct image file such as PNG, JPEG, or GIF image file.

         Currently only supported in monolithic (single-process) server configurations
         where the media repository runs within the Synapse process.

         *Deprecated in Synapse v1.75.0.*

       * `localpart_template`: Jinja2 template for the localpart of the MXID.
          If this is not set, the user will be prompted to choose their
          own username (see the documentation for the `sso_auth_account_details.html`
          template). This template can use the `localpart_from_email` filter.

       * `confirm_localpart`: Whether to prompt the user to validate (or
          change) the generated localpart (see the documentation for the
          'sso_auth_account_details.html' template), instead of
          registering the account right away.

       * `display_name_template`: Jinja2 template for the display name to set
          on first login. If unset, no displayname will be set.

       * `email_template`: Jinja2 template for the email address of the user.
          If unset, no email address will be added to the account.

       * `extra_attributes`: a map of Jinja2 templates for extra attributes
          to send back to the client during login. Note that these are non-standard and clients will ignore them
          without modifications.

     When rendering, the Jinja2 templates are given a 'user' variable,
     which is set to the claims returned by the UserInfo Endpoint and/or
     in the ID Token.

* `backchannel_logout_enabled`: set to `true` to process OIDC Back-Channel Logout notifications.
  Those notifications are expected to be received on `/_synapse/client/oidc/backchannel_logout`.
  Defaults to `false`.

* `backchannel_logout_ignore_sub`: by default, the OIDC Back-Channel Logout feature checks that the
  `sub` claim matches the subject claim received during login. This check can be disabled by setting
  this to `true`. Defaults to `false`.

  You might want to disable this if the `subject_claim` returned by the mapping provider is not `sub`.

It is possible to configure Synapse to only allow logins if certain attributes
match particular values in the OIDC userinfo. The requirements can be listed under
`attribute_requirements` as shown here:
```yaml
attribute_requirements:
     - attribute: family_name
       value: "Stephensson"
     - attribute: groups
       value: "admin"
```
All of the listed attributes must match for the login to be permitted. Additional attributes can be added to
userinfo by expanding the `scopes` section of the OIDC config to retrieve
additional information from the OIDC provider.

If the OIDC claim is a list, then the attribute must match any value in the list.
Otherwise, it must exactly match the value of the claim. Using the example
above, the `family_name` claim MUST be "Stephensson", but the `groups`
claim MUST contain "admin".

Example configuration:
```yaml
oidc_providers:
  # Generic example
  #
  - idp_id: my_idp
    idp_name: "My OpenID provider"
    idp_icon: "mxc://example.com/mediaid"
    discover: false
    issuer: "https://accounts.example.com/"
    client_id: "provided-by-your-issuer"
    client_secret: "provided-by-your-issuer"
    client_auth_method: client_secret_post
    scopes: ["openid", "profile"]
    authorization_endpoint: "https://accounts.example.com/oauth2/auth"
    token_endpoint: "https://accounts.example.com/oauth2/token"
    userinfo_endpoint: "https://accounts.example.com/userinfo"
    jwks_uri: "https://accounts.example.com/.well-known/jwks.json"
    additional_authorization_parameters:
      acr_values: 2fa
    skip_verification: true
    enable_registration: true
    user_mapping_provider:
      config:
        subject_claim: "id"
        localpart_template: "{{ user.login }}"
        display_name_template: "{{ user.name }}"
        email_template: "{{ user.email }}"
    attribute_requirements:
      - attribute: userGroup
        value: "synapseUsers"
```
---
### `cas_config`

Enable Central Authentication Service (CAS) for registration and login.
Has the following sub-options:
* `enabled`: Set this to true to enable authorization against a CAS server.
   Defaults to false.
* `idp_name`: A user-facing name for this identity provider, which is used to
   offer the user a choice of login mechanisms.
* `idp_icon`: An optional icon for this identity provider, which is presented
   by clients and Synapse's own IdP picker page. If given, must be an
   MXC URI of the format `mxc://<server-name>/<media-id>`. (An easy way to
   obtain such an MXC URI is to upload an image to an (unencrypted) room
   and then copy the "url" from the source of the event.)
* `idp_brand`: An optional brand for this identity provider, allowing clients
   to style the login flow according to the identity provider in question.
   See the [spec](https://spec.matrix.org/latest/) for possible options here.
* `server_url`: The URL of the CAS authorization endpoint.
* `protocol_version`: The CAS protocol version, defaults to none (version 3 is required if you want to use "required_attributes").
* `displayname_attribute`: The attribute of the CAS response to use as the display name.
   If no name is given here, no displayname will be set.
* `required_attributes`:  It is possible to configure Synapse to only allow logins if CAS attributes
   match particular values. All of the keys given below must exist
   and the values must match the given value. Alternately if the given value
   is `None` then any value is allowed (the attribute just must exist).
   All of the listed attributes must match for the login to be permitted.
* `enable_registration`: set to 'false' to disable automatic registration of new
   users. This allows the CAS SSO flow to be limited to sign in only, rather than
   automatically registering users that have a valid SSO login but do not have
   a pre-registered account. Defaults to true.
* `allow_numeric_ids`: set to 'true' allow numeric user IDs (default false).
   This allows CAS SSO flow to provide user IDs composed of numbers only.
   These identifiers will be prefixed by the letter "u" by default.
   The prefix can be configured using the "numeric_ids_prefix" option.
   Be careful to choose the prefix correctly to avoid any possible conflicts
   (e.g. user 1234 becomes u1234 when a user u1234 already exists).
* `numeric_ids_prefix`: the prefix you wish to add in front of a numeric user ID
   when the "allow_numeric_ids" option is set to "true".
   By default, the prefix is the letter "u" and only alphanumeric characters are allowed.

   *Added in Synapse 1.93.0.*

Example configuration:
```yaml
cas_config:
  enabled: true
  server_url: "https://cas-server.com"
  protocol_version: 3
  displayname_attribute: name
  required_attributes:
    userGroup: "staff"
    department: None
  enable_registration: true
  allow_numeric_ids: true
  numeric_ids_prefix: "numericuser"
```
---
### `sso`

Additional settings to use with single-sign on systems such as OpenID Connect,
SAML2 and CAS.

Server admins can configure custom templates for pages related to SSO. See
[here](../../templates.md) for more information.

Options include:
* `client_whitelist`: A list of client URLs which are whitelisted so that the user does not
   have to confirm giving access to their account to the URL. Any client
   whose URL starts with an entry in the following list will not be subject
   to an additional confirmation step after the SSO login is completed.
   WARNING: An entry such as "https://my.client" is insecure, because it
   will also match "https://my.client.evil.site", exposing your users to
   phishing attacks from evil.site. To avoid this, include a slash after the
   hostname: "https://my.client/".
   The login fallback page (used by clients that don't natively support the
   required login flows) is whitelisted in addition to any URLs in this list.
   By default, this list contains only the login fallback page.
* `update_profile_information`: Use this setting to keep a user's profile fields in sync with information from
   the identity provider. Currently only syncing the displayname is supported. Fields
   are checked on every SSO login, and are updated if necessary.
   Note that enabling this option will override user profile information,
   regardless of whether users have opted-out of syncing that
   information when first signing in. Defaults to false.


Example configuration:
```yaml
sso:
    client_whitelist:
      - https://riot.im/develop
      - https://my.custom.client/
    update_profile_information: true
```
---
### `jwt_config`

JSON web token integration. The following settings can be used to make
Synapse JSON web tokens for authentication, instead of its internal
password database.

Each JSON Web Token needs to contain a "sub" (subject) claim, which is
used as the localpart of the mxid.

Additionally, the expiration time ("exp"), not before time ("nbf"),
and issued at ("iat") claims are validated if present.

Note that this is a non-standard login type and client support is
expected to be non-existent.

See [here](../../jwt.md) for more.

Additional sub-options for this setting include:
* `enabled`: Set to true to enable authorization using JSON web
   tokens. Defaults to false.
* `secret`: This is either the private shared secret or the public key used to
   decode the contents of the JSON web token. Required if `enabled` is set to true.
