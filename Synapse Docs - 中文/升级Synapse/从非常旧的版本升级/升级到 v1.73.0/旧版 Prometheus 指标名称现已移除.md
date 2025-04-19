Synapse v1.69.0 包含对旧版 Prometheus 指标名称的弃用，并提供了禁用它们的选项。Synapse v1.71.0 默认禁用旧版 Prometheus 指标名称。

此版本 v1.73.0 完全移除了这些旧版 Prometheus 指标名称。这也意味着 `enable_legacy_metrics` 配置选项已被移除；将不再可能重新启用旧版指标名称。

如果您使用指标并且尚未更新您的 Grafana 仪表板、Prometheus 控制台或警报规则，请考虑在升级到此版本时进行更新。请注意，包含的 Grafana 仪表板在 v1.72.0 中已更新，以更正一些在默认禁用旧版指标时遗漏的指标名称。

有关更多背景信息，请参阅[v1.69.0：旧版 Prometheus 指标名称的弃用](#deprecation-of-legacy-prometheus-metric-names)。