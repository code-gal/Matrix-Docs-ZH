大多数部署只有一个 IdP 实体，因此应该省略此选项。

启用 SAML 支持后，将在 `https://<server>:<port>/_synapse/client/saml2/metadata.xml` 处提供一个元数据文件，您可以用来配置您的 SAML IdP。或者，您可以手动配置 IdP 以使用 ACS 位置 `https://<server>:<port>/_synapse/client/saml2/authn_response`。

示例配置：
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

    # 以下示例仅用于生成我们的元数据 xml，具体设置可能根据您的部署不同而有所改变。或者您可能需要详细信息丰富配置 - 请参阅pysaml2文档！
    description: ["My awesome SP", "en"]
    name: ["Test SP", "en"]

    ui_info:
      display_name:
        - lang: en
          text: "显示名称是您服务的描述名称。"
      description:
        - lang: en
          text: "描述应为简短的段落，解释服务的用途。"
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
    # 以下选项用于内置提供程序，如果使用自定义模块，应进行更改。
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