在升级之前，请检查从您当前安装的版本升级到当前版本的Synapse是否需要任何特殊步骤。可能需要的额外说明列在本文档后面。

- 检查您的Python和PostgreSQL版本是否仍受支持。

  Synapse遵循[Python](https://endoflife.date/python)和[PostgreSQL](https://endoflife.date/postgresql)的上游生命周期，并移除对不再维护的版本的支持。

  网站<https://endoflife.date>也提供了方便的摘要。

- 如果Synapse是使[预构建包](Synapse%20Docs%20-%20EN/setup/installation.md#prebuilt-packages))安装的，您需要按照正常流程升级这些包。

- 如果Synapse是使用pip安装的，则通过运行以下命令升级到最新版本：

  ```bash
  pip install --upgrade matrix-synapse
  ```

- 如果Synapse是从源代码安装的，则：

  1. 获取最新版本的源代码。Git用户可以运行`git pull`来完成此操作。

  2. 如果您在虚拟环境中运行Synapse，请在升级前确保激活它。例如，如果Synapse安装在`~/synapse/env`中的虚拟环境中，则运行：

     ```bash
     source ~/synapse/env/bin/activate
     pip install --upgrade .
     ```
     在方括号中包含任何相关的额外内容，例如`pip install --upgrade ".[postgres,oidc]"`。

  3. 如果您使用`poetry`管理Synapse安装，请运行：
     ```bash
     poetry install
     ```
     使用`--extras`包含任何相关的额外内容，例如`poetry install --extras postgres --extras oidc`。最简单的方法可能是运行`poetry install --extras all`。

  4. 重启Synapse：

     ```bash
     synctl restart
     ```

要检查您的更新是否成功，您可以检查正在运行的服务器版本：

```bash
  # 如果Synapse未配置为监听8008端口，您可能需要替换'localhost:8008'。
  curl http://localhost:8008/_synapse/admin/v1/server_version
```