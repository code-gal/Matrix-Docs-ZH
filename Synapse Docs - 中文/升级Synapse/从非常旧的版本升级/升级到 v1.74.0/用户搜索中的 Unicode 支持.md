此版本引入了对[改进的用户搜索处理 Unicode 字符](https://github.com/matrix-org/synapse/pull/14464)的可选支持。

如果您想利用此功能，您需要安装 PyICU、ICU 本机依赖项及其开发头文件，以便 PyICU 可以构建，因为没有可用的预构建轮子。

您可以按照[PyICU 文档](https://pypi.org/project/PyICU/)进行操作，然后执行 `pip install matrix-synapse[user-search]` 进行 PyPI 安装。

Docker 镜像和 Debian 包不需要任何特定操作，因为它们已经包含或指定 ICU 作为显式依赖项。