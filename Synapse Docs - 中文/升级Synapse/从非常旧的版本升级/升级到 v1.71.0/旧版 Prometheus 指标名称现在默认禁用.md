Synapse v1.71.0 默认禁用旧版 Prometheus 指标名称。对于仍然依赖它们且尚未有机会更新其指标使用的管理员，仍然可以在配置中指定 `enable_legacy_metrics: true` 来暂时重新启用它们。

Synapse v1.73.0 将**完全移除旧版指标名称**，届时将不再可能重新启用它们。

如果您不使用指标或已经更新了您的 Grafana 仪表板、Prometheus 控制台和警报规则，则无需采取任何措施。

请参阅[v1.69.0：旧版 Prometheus 指标名称的弃用](#deprecation-of-legacy-prometheus-metric-names)。