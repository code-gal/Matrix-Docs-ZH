从 Synapse 的源代码检出构建现在需要一个最近的 Rust 编译器（当前为 Rust 1.58.1，但请参阅[平台依赖政策](https://element-hq.github.io/synapse/latest/deprecation_policy.html)）。

使用以下安装方式的用户不受影响：

- 来自 `matrixdotorg` 的 Docker 镜像，
- 来自 Matrix.org 的 Debian 包，或
- 通过 `pip install matrix-synapse` 安装的 PyPI 轮子（在支持的平台和架构上）