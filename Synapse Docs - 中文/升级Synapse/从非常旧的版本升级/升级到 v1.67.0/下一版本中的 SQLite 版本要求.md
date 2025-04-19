从下一个主要版本（v1.68.0）开始，Synapse 将需要 SQLite 3.27.0 或更高版本。Synapse v1.67.0 将是最后一个支持 SQLite 版本 3.22 到 3.26 的主要版本。

使用来自 Matrix.org 的 Docker 镜像或 Debian 包的用户将不受影响。如果您是从源代码安装的，您应该检查 Python 使用的 SQLite 版本：

```shell
python -c "import sqlite3; print(sqlite3.sqlite_version)"
```

如果版本过旧，请参考您的发行版以获取升级建议。