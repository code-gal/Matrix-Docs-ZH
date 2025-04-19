#### 通过 Grafana 图表了解 Synapse

可以使用 Prometheus 指标和 Grafana 来监控 Synapse 的许多内部状态。配置 Synapse 以提供指标的指南在这里，关于设置 Grafana 的信息在这里。在这个设置中，Prometheus 将定期抓取 Synapse 提供的信息并随时间存储其记录。Grafana 随后被用作查询和展示这些信息的界面，通过一系列漂亮的图表。

一旦你设置好 Grafana，并且假设你正在使用我们的 Grafana 仪表板模板，在调试一个缓慢/过载的 Synapse 时，请查找以下图表：

#### 消息事件发送时间

![image](https://user-images.githubusercontent.com/1342360/82239409-a1c8e900-9930-11ea-8081-e4614e0c63f4.png)

这与 CPU 和内存图表一起，是检查您的 Synapse 实例总体健康状况的一个好方法。它表示用户在您的家庭服务器上发送消息所需的时间。

#### 交易数量和交易持续时间

![image](https://user-images.githubusercontent.com/1342360/82239985-8d392080-9931-11ea-80d0-843ab2f22e1e.png)

![image](https://user-images.githubusercontent.com/1342360/82240050-ab068580-9931-11ea-98f1-f94671cbac9a.png)

这些图表显示了最频繁发生的数据库事务，以及那些执行时间最长的数据库事务。

![image](https://user-images.githubusercontent.com/1342360/82240192-e86b1300-9931-11ea-9aac-3e2c9bfa6fdc.png)

在第一个图表中，我们可以看到明显的峰值，对应着大量的 `get_user_by_id` 交易。这将是有用的信息，用于找出 Synapse 代码库中哪些部分可能对系统造成了沉重的负担。然而，请务必与交易持续时间进行交叉参考，后者指出 `get_users_by_id` 实际上是一个非常快的数据库交易，并没有像 `persist_events` 那样造成那么大的负载：

![image](https://user-images.githubusercontent.com/1342360/82240467-62030100-9932-11ea-8db9-917f2d977fe1.png)

尽管如此，值得调查一下为什么我们经常从数据库中获取用户，以及是否可以通过调整我们的缓存因子来减少查询次数。

`persist_events` 事务负责将新的房间事件保存到 Synapse 数据库中，因此通常会显示较高的交易持续时间。

#### 联邦

“联合”部分中的图表显示了有关传入和传出联合请求的信息。联合数据可以分为两种基本类型：

*   PDU（持久数据单元）- 房间事件：消息、状态事件（加入/离开）等。这些将永久存储在数据库中。
*   EDU（短暂数据单元）- 其他数据，无需永久存储，例如已读回执、正在输入通知。

“按类型划分的传出 EDU”图表显示了传出联邦请求中的 EDU 按类型： `m.device_list_update` , `m.direct_to_device` , `m.presence` , `m.receipt` , `m.typing` 。

如果您看到大量的 `m.presence` EDUs 并且 CPU 负载过高，您可以在 Synapse 配置中禁用 `presence` 。另见 #3971。

#### 缓存

![image](https://user-images.githubusercontent.com/1342360/82240572-8b239180-9932-11ea-96ff-6b5f0e57ebe5.png)

![image](https://user-images.githubusercontent.com/1342360/82240666-b8703f80-9932-11ea-86af-9f663988d8da.png)

这是一个非常有用的图表。它显示了 Synapse 尝试从缓存中检索数据的次数，但缓存中不包含该数据，因此导致了对数据库的调用。我们可以看到 `_get_joined_profile_from_event_id` 缓存被频繁请求，而且我们要找的数据往往没有被缓存。

将此与驱逐率图表进行交叉引用，该图表显示 `_get_joined_profile_from_event_id` 中的条目经常被驱逐：

![image](https://user-images.githubusercontent.com/1342360/82240766-de95df80-9932-11ea-8c15-5acfc57c48da.png)

我们可能应该考虑通过提高其缓存因子（单个缓存大小的乘数值）来增加该缓存的大小。有关如何操作的信息可以在这里找到（注意，通过配置文件配置单个缓存因子在 Synapse v1.14.0+版本中可用，而通过环境变量进行配置已支持很长时间）。请注意，这将增加 Synapse 的整体内存使用量。

#### 前向极限

![image](https://user-images.githubusercontent.com/1342360/82241440-13566680-9934-11ea-8b88-ba468db937ed.png)

前向极限是房间中 DAG 末端的叶子事件，即没有子事件的事件。房间中存在的这些事件越多，Synapse 需要执行的状态解析就越多（提示：这是一个昂贵的操作）。虽然 Synapse 有代码来防止在一个房间中同时存在太多这样的情况，但有时由于错误它们可能会再次出现。

如果一个房间有>10 个前向极端点，值得检查哪个房间是罪魁祸首，并可能使用#1760 中提到的 SQL 查询来移除它们。

#### 垃圾回收

![image](https://user-images.githubusercontent.com/1342360/82241911-da6ac180-9934-11ea-9a0d-a311fe22acd0.png)

垃圾回收时间的大幅波动（比这里显示的更大，我说的是几秒钟的范围），可能会导致 Synapse 性能出现很多问题。  
这更多是问题的指示器，也是其他问题的症状，所以请检查其他图表以了解可能的原因。

#### 最后的思考

如果您的 Synapse 实例仍然存在性能问题，并且您已经尝试了所有可能的方法，可能是系统资源不足。考虑增加 CPU 和 RAM，并利用工作模式来利用多个 CPU 核心/多台机器来运行您的家庭服务器。