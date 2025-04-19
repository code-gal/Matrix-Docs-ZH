Synapse v1.79.0 弃用[`on_threepid_bind`](Synapse%20Docs%20-%20EN/modules/third_party_rules_callbacks.md#on_threepid_bind))“第三方规则”Synapse 模块回调方法，转而使用新的模块方[`on_add_user_third_party_identifier`](Synapse%20Docs%20-%20EN/modules/third_party_rules_callbacks.md#on_add_user_third_party_identifier))。`on_threepid_bind` 将在未来版本的 Synapse 中移除。您应检查部署中使用的任何 Synapse 模块是否使用了 `on_threepid_bind`，并尽可能更新它们。

新方法的参数和功能相同。

更名的理由是旧方法的名称 `on_threepid_bind` 具有误导性。用户被认为将其第三方 ID“绑定”到其 Matrix ID 仅当他们通过[身份服务器](https://spec.matrix.org/latest/identity-service-api/)这样做时（以便其他家庭服务器的用户可以找到他们）。但在这种情况下不会调用此方法——它仅在用户在本地家庭服务器上添加第三方标识符时被调用。

模块开发者可能还对 Synapse v1.79.0 中添加的相[`on_remove_user_third_party_identifier`](Synapse%20Docs%20-%20EN/modules/third_party_rules_callbacks.md#on_remove_user_third_party_identifier))模块回调方法感兴趣。此新方法在用户从其帐户中移除第三方标识符时被调用。