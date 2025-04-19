在当前版本的 Synapse 中，一些 Prometheus 指标以两个不同的名称发出，其中一个名称较旧但不符合 OpenMetrics 和 Prometheus 约定，另一个名称较新但符合。

Synapse v1.71.0 将默认关闭旧的指标名称。对于仍然依赖它们且尚未有机会更新其指标使用的管理员，可以在配置中指定 `enable_legacy_metrics: true` 来暂时重新启用它们。

Synapse v1.73.0 将**完全移除旧版指标名称**，届时将不再可能重新启用它们。

Synapse 存储库中 `contrib` 目录中包含的 Grafana 仪表板、Prometheus 记录规则和 Prometheus 控制台已更新，不再依赖旧版名称。这些可以在当前版本的 Synapse 上使用，因为当前版本的 Synapse 发出旧名称和新名称。

您可能需要更新您的警报规则或任何其他依赖 Prometheus 指标名称的规则。如果您想在默认禁用旧版名称之前测试您的更改，可以在家庭服务器配置中指定 `enable_legacy_metrics: false`。

受影响指标的列表可在[指标使用指南页面](https://element-hq.github.io/synapse/v1.69/metrics-howto.html?highlight=metrics%20deprecated#renaming-of-metrics--deprecation-of-old-names-in-12)上找到。