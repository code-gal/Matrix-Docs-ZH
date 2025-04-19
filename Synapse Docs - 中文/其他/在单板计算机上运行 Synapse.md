#### 在资源受限设备如 SBC 上运行的性能影响总结

我在家里用 CubieTruck 运行我的家庭服务器已经有一段时间了，经常会回复类似于“你需要大量的内存才能加入大型房间”的说法，我的回答是“对我来说运行得很好”  
我认为整理一个可能遇到的问题总结可能会有用，作为一个缩减指南，可能会突出这些问题用于开发工作或最终作为文档。  
看起来一旦你达到大约 4x1.5GHz arm64 4GiB，这些问题就不再是问题了。

*   平台：2x1GHz armhf 2GiB 内存 单板计算机，SSD，PostgreSQL。

##### 在线状态

这是人们在资源受限的家庭服务器上体验不佳的主要原因。Element web 会频繁显示服务器离线，而 Python 进程会占用 100% 的 CPU。  
此功能用于判断其他用户是否活跃（有客户端应用程序在前台），因此更可能做出回应，但即使在房间内没有人说话时，也需要大量的网络活动来维持。

![Screenshot_2020-10-01_19-29-46](https://user-images.githubusercontent.com/71895/94848963-a47a3580-041c-11eb-8b6e-acb772b4259e.png)

虽然 synapse 在处理在线状态时确实存在一些性能问题 #3971，但根本问题在于，对于一个中心化的服务来说，这是一个几乎没有额外开销的简单功能，但联邦系统使其变得复杂 #8055。还有一个客户端配置选项，可以通过 enable_presence_by_hs_url 来禁用 UI 和空闲跟踪，以将最大的实例列入黑名单，但我没有注意到太大区别，所以我建议在服务器级别完全禁用此功能。

##### 加入

加入一个“大的”、联合的房间最初会在 Element web 中失败，并显示以下消息，但等待一段时间（10-60 分钟）后再尝试将能成功且无任何问题。  
所谓“大的”并不是指消息历史、用户数量、与主服务器的连接或甚至是状态事件的简单计数，而是指状态解析算法所需的时间。  
然而，这些数字都是合理的代理，因此我们可以将它们用作估计值，因为用户数量是你加入之前能看到的少数信息之一。

![Screenshot_2020-10-02_17-15-06](https://user-images.githubusercontent.com/71895/94945781-18771500-04d3-11eb-8419-83c2da73a341.png)

这是 #1211，并且希望通过查看 matrix-org/matrix-doc#2753 来缓解这个问题，这样至少在加入完成之前你就不需要等待以确定这是否是你想要的房间类型。请注意，你应该首先禁用在线状态，否则情况只会变得更糟 #3120。还有很多数据库交互，所以请确保你已经将数据从默认的 sqlite 迁移到 postgresql。个人建议要有耐心 - 一旦初始加入完成，实际上与房间互动很少会有问题，但如果你愿意，你可以完全屏蔽“大的”房间。

##### 会话

任何需要修改设备列表 #7721 的操作都将需要一段时间来传播，这会再次将客户端置于“离线”状态，直到完成。这包括登录和退出、编辑公开名称以及验证端到端加密。  
我推荐的主要缓解措施是保持长时间会话开启，例如通过使用 Firefox SSB 的“在应用模式中使用此站点”或 Chromium PWA 的“安装 Element”。

##### 推荐配置

将以下内容放入一个新文件 /etc/matrix-synapse/conf.d/sbc.yaml 中，以覆盖 homeserver.yaml 中的默认设置。

```
### Disable presence tracking, which is currently fairly resource intensive
### More info: https://github.com/matrix-org/synapse/issues/9478
use_presence: false

### Set a small complexity limit, preventing users from joining large rooms
### which may be resource-intensive to remain a part of.
#
### Note that this will not prevent users from joining smaller rooms that
### eventually become complex.
limit_remote_rooms:
  enabled: true
  complexity: 3.0

### Database configuration
database:
===  # Use postgres for the best performance===
  name: psycopg2
  args:
    user: matrix-synapse
===    # Generate a long, secure password using a password manager===
    password: hunter2
    database: matrix-synapse
    host: localhost
```

目前复杂度是通过 current_state_events / 500 来衡量的。你可以这样找到加入时间和最复杂的房间：

```
admin@homeserver:~undefined, $25}' | sort --human-numeric-sort
29.922sec/-0.002sec /_matrix/client/r0/join/%23debian-fasttrack%3Apoddery.com
182.088sec/0.003sec /_matrix/client/r0/join/%23decentralizedweb-general%3Amatrix.org
911.625sec/-570.847sec /_matrix/client/r0/join/%23synapse%3Amatrix.org

admin@homeserver:~$ sudo --user postgres psql matrix-synapse --command 'select canonical_alias, joined_members, current_state_events from room_stats_state natural join room_stats_current where canonical_alias is not null order by current_state_events desc fetch first 5 rows only'
        canonical_alias        | joined_members | current_state_events 
-------------------------------+----------------+----------------------
 #_oftc_#debian:matrix.org             |  871   |  52355
 #matrix:matrix.org                    |  6379  |  10684
 #irc:matrix.org                       |  461   |  3751
 #decentralizedweb-general:matrix.org  |  997   |  1509
 #whatsapp:maunium.net                 |  554   |  854
```