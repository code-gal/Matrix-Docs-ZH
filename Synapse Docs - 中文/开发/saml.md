### 作为开发人员如何在没有服务器的情况下测试SAML

https://fujifish.github.io/samling/samling.html (https://github.com/fujifish/samling) 是一个很好的资源，可以在不需要部署和配置复杂软件堆栈的情况下，能够在Synapse中测试SAML选项。

要使Synapse（因此Element）使用它：

1. 使用上面的samling.html URL或自行部署并访问IdP Metadata选项卡。
2. 将XML复制到剪贴板。
3. 在您的Synapse服务器上，创建一个新的文件`samling.xml`，放在`homeserver.yaml`旁边，内容为步骤2中的XML。
4. 编辑您的`homeserver.yaml`以包含：
   ```yaml
   saml2_config:
     sp_config:
       allow_unknown_attributes: true  # 解决AVA哈希的错误：https://github.com/IdentityPython/pysaml2/issues/388
       metadata:
         local: ["samling.xml"]
   ```
5. 确保您的`homeserver.yaml`中有`public_baseurl`的设置：
   ```yaml
   public_baseurl: http://localhost:8080/
   ```
6. 运行`apt-get install xmlsec1`和`pip install --upgrade --force 'pysaml2>=4.5.0'`以确保依赖项已安装并准备就绪。
7. 重启Synapse。

然后在Element中：

1. 访问登录页面并使用上面的`public_baseurl`指向您的homeserver。
2. 点击单点登录按钮。
3. 在samling页面上，输入一个名称标识符并为`uid=your_localpart`添加一个SAML属性。响应也必须被签名。
4. 点击“下一步”。
5. 点击“发布响应”（不更改任何内容）。
6. 您应该已登录。

如果您尝试重复此过程，您可能会使用之前提供的信息自动登录。要解决此问题，请在samling页面上打开开发者控制台（`F12`或`Ctrl+Shift+I`）并清除站点数据。在Chrome中，这将在应用程序选项卡上有一个按钮。