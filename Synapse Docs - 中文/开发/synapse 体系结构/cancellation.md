### 取消
有时，请求需要很长时间才能服务，客户端在Synapse产生响应之前断开连接。为了避免浪费资源，Synapse可以取消为标记为`@cancellable`的选择端点的请求处理。

Synapse利用Twisted的`Deferred.cancel()`功能来实现取消。`@cancellable`装饰器本身不执行任何操作，仅作为一个标志，向开发人员和其他代码表明一个方法可以被取消。

#### 为端点启用取消
1. 检查端点方法及其调用树中的任何`async`函数是否正确处理取消。请参阅[正确处理取消](#handling-cancellation-correctly)以获取需要注意的事项列表。
2. 将`@cancellable`装饰器添加到`on_GET/POST/PUT/DELETE`方法。不建议使非`GET`方法可取消，因为在某些数据库更新中途取消不太可能被正确处理。

#### Mechanics
取消有两个阶段：向下传播`cancel()`调用，然后是从阻塞的`await`中向上传播`CancelledError`。
Twisted和asyncio都有取消机制。

|               | 方法                | 异常                                   | 异常继承自             |
|---------------|---------------------|----------------------------------------|------------------------|
| Twisted       | `Deferred.cancel()` | `twisted.internet.defer.CancelledError`| `Exception` (!)        |
| asyncio       | `Task.cancel()`     | `asyncio.CancelledError`               | `BaseException`        |

##### Deferred.cancel()
当Synapse开始处理请求时，它使用`defer.ensureDeferred`运行负责处理它的异步方法，该方法返回一个`Deferred`。例如：

```python
def do_something() -> Deferred[None]:
    ...

@cancellable
async def on_GET() -> Tuple[int, JsonDict]:
    d = make_deferred_yieldable(do_something())
    await d
    return 200, {}

request = defer.ensureDeferred(on_GET())
```

当客户端提前断开连接时，Synapse检查`on_GET`上是否存在`@cancellable`装饰器。由于`on_GET`是可取消的，因此在`defer.ensureDeferred`中调用的`Deferred`（即`request`）上调用`Deferred.cancel()`。Twisted知道`request`正在等待哪个`Deferred`并将`cancel()`调用传递给`d`。

被等待的`Deferred`，`d`，可能有自己的`cancel()`处理并将调用传递给其他`Deferred`。

最终，一个`Deferred`通过用`CancelledError`解析自己来处理`cancel()`调用。

##### CancelledError
`CancelledError`从`await`中被抛出并向上冒泡，按照正常的Python异常处理。

#### 正确处理取消
一般来说，在编写可能会被取消的代码时，必须考虑两件事：
 * 从`await`中抛出的`CancelledError`的影响。
 * `Deferred`被`cancel()`的影响。

处理取消不正确的代码示例包括：
 * 吞掉`CancelledError`的`try-except`块。
 * 在多个请求之间共享可能被取消的相同`Deferred`的代码。
 * 启动一些不受取消影响的处理，但使用来自可取消代码的日志记录上下文的代码。日志记录上下文将在取消时完成，而未取消的处理仍在使用它。

下面列出了一些常见的模式，详细说明。

##### `async` 函数调用
Synapse中的大多数函数从取消的角度来看相对简单：它们不对`Deferred`做任何事情，并且纯粹调用和`await`其他`async`函数。

一个`async`函数如果其自身代码正确处理取消并且它调用的所有异步函数正确处理取消，则它正确处理取消。例如：
```python
async def do_two_things() -> None:
    check_something()
    await do_something()
    await do_something_else()
```
如果`do_something`和`do_something_else`正确处理取消，则`do_two_things`正确处理取消。

也就是说，当检查一个函数是否正确处理取消时，需要递归地检查其实现及其所有的`async`函数调用。

由于`check_something`不是`async`，因此不需要检查。

##### CancelledErrors
由于Twisted的`CancelledError`是`Exception`，因此很容易意外捕获并抑制它们。必须小心确保`CancelledError`被允许向上传播。

<table width="100%">
<tr>
<td width="50%" valign="top">

**错误**:
```python
try:
    await do_something()
except Exception:
===    # `CancelledError`在这里被吞掉了。===
    logger.info(...)
```
</td>
<td width="50%" valign="top">

**正确**:
```python
try:
    await do_something()
except CancelledError:
    raise
except Exception:
    logger.info(...)
```
</td>
</tr>
<tr>
<td width="50%" valign="top">

**可以**:
```python
try:
    check_something()
===    # 这里永远不会抛出`CancelledError`。===
except Exception:
    logger.info(...)
```
</td>
<td width="50%" valign="top">

**正确**:
```python
try:
    await do_something()
except ValueError:
    logger.info(...)
```
</td>
</tr>
</table>

  ```python
  # faster_joins的工作原理是什么？

  这是一组正在进行的笔记，目标有两个：
  - 作为参考，解释Synapse如何实现更快的加入；
  - 记录我们选择背后的理由。

  另请参见[MSC3902](https://github.com/matrix-org/matrix-spec-proposals/pull/3902)。

  关键思想由[MSC3706](https://github.com/matrix-org/matrix-spec-proposals/pull/3706)描述。这允许服务器请求对联邦`/send_join`端点的轻量级响应。这被称为**更快的加入**，也称为**部分加入**。在这些笔记中，我们通常使用“部分”这个词，因为它与数据库模式相匹配。
  ```