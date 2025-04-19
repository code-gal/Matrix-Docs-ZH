以下模块 API 方法已被弃用，计划在 v1.71.0 中移除：

```python
def generate_short_term_login_token(
    self,
    user_id: str,
    duration_in_ms: int = (2 * 60 * 1000),
    auth_provider_id: str = "",
    auth_provider_session_id: Optional[str] = None,
) -> str:
    ...
```

它已被一个异步等效方法取代：

```python
async def create_login_token(
    self,
    user_id: str,
    duration_in_ms: int = (2 * 60 * 1000),
    auth_provider_id: Optional[str] = None,
    auth_provider_session_id: Optional[str] = None,
) -> str:
    ...
```

Synapse 将在模块使用已弃用的方法时记录警告，以帮助管理员找到使用它的模块。