#### 一些对 Synapse 管理员有用的 SQL 查询

#### 完整矩阵数据库的大小

```
SELECT pg_size_pretty( pg_database_size( 'matrix' ) );
```

##### 结果示例:

```
pg_size_pretty 
----------------
 6420 MB
(1 row)
```

#### 按行数显示前 20 个较大的表

```
SELECT relname, n_live_tup AS "rows"
  FROM pg_stat_user_tables
  ORDER BY n_live_tup DESC
  LIMIT 20;
```

此查询速度快，但可能非常近似，要获取精确的行数，请使用：

```
SELECT COUNT(*) FROM <table_name>;
```

##### 结果示例:

```
state_groups_state - 161687170
event_auth - 8584785
event_edges - 6995633
event_json - 6585916
event_reference_hashes - 6580990
events - 6578879
received_transactions - 5713989
event_to_state_groups - 4873377
stream_ordering_to_exterm - 4136285
current_state_delta_stream - 3770972
event_search - 3670521
state_events - 2845082
room_memberships - 2785854
cache_invalidation_stream - 2448218
state_groups - 1255467
state_group_edges - 1229849
current_state_events - 1222905
users_in_public_rooms - 364059
device_lists_stream - 326903
user_directory_search - 316433
```

#### 显示按存储大小排名前 20 的较大表

```
SELECT nspname || '.' || relname AS "relation",
    pg_size_pretty(pg_total_relation_size(c.oid)) AS "total_size"
  FROM pg_class c
  LEFT JOIN pg_namespace n ON (n.oid = c.relnamespace)
  WHERE nspname NOT IN ('pg_catalog', 'information_schema')
    AND c.relkind <> 'i'
    AND nspname !~ '^pg_toast'
  ORDER BY pg_total_relation_size(c.oid) DESC
  LIMIT 20;
```

##### 结果示例:

```
public.state_groups_state - 27 GB
public.event_json - 9855 MB
public.events - 3675 MB
public.event_edges - 3404 MB
public.received_transactions - 2745 MB
public.event_reference_hashes - 1864 MB
public.event_auth - 1775 MB
public.stream_ordering_to_exterm - 1663 MB
public.event_search - 1370 MB
public.room_memberships - 1050 MB
public.event_to_state_groups - 948 MB
public.current_state_delta_stream - 711 MB
public.state_events - 611 MB
public.presence_stream - 530 MB
public.current_state_events - 525 MB
public.cache_invalidation_stream - 466 MB
public.receipts_linearized - 279 MB
public.state_groups - 160 MB
public.device_lists_remote_cache - 124 MB
public.state_group_edges - 122 MB
```

#### 按状态事件计数显示前 20 个较大的房间

当您使用管理员 API 并设置参数 `order_by=state_events` 时，您将获得相同的信息。

```
SELECT r.name, s.room_id, s.current_state_events
  FROM room_stats_current s
  LEFT JOIN room_stats_state r USING (room_id)
  ORDER BY current_state_events DESC
  LIMIT 20;
```

并按 state_group_events 计数：

```
SELECT rss.name, s.room_id, COUNT(s.room_id)
  FROM state_groups_state s
  LEFT JOIN room_stats_state rss USING (room_id)
  GROUP BY s.room_id, rss.name
  ORDER BY COUNT(s.room_id) DESC
  LIMIT 20;
```

加相同，但为了性能原因移除了 join：

```
SELECT s.room_id, COUNT(s.room_id)
  FROM state_groups_state s
  GROUP BY s.room_id 
  ORDER BY COUNT(s.room_id) DESC
  LIMIT 20;
```

#### 显示最近 1 天内新事件数量最多的前 20 个房间：

```
SELECT e.room_id, r.name, COUNT(e.event_id) cnt
  FROM events e
  LEFT JOIN room_stats_state r USING (room_id)
  WHERE e.origin_server_ts >= DATE_PART('epoch', NOW() - INTERVAL '1 day') * 1000
  GROUP BY e.room_id, r.name 
  ORDER BY cnt DESC
  LIMIT 20;
```

#### 显示过去一个月内在主服务器上按发送事件（消息）排名前 20 的用户：

注意。此查询不使用任何索引，可能会很慢并对数据库造成负载。

```
SELECT COUNT(*), sender
  FROM events
  WHERE (type = 'm.room.encrypted' OR type = 'm.room.message')
    AND origin_server_ts >= DATE_PART('epoch', NOW() - INTERVAL '1 month') * 1000
  GROUP BY sender
  ORDER BY COUNT(*) DESC
  LIMIT 20;
```

#### 显示所需用户的最后 100 条消息，包括房间名称：

```
SELECT e.room_id, r.name, e.event_id, e.type, e.content, j.json
  FROM events e
  LEFT JOIN event_json j USING (room_id)
  LEFT JOIN room_stats_state r USING (room_id)
  WHERE sender = '@LOGIN:example.com'
    AND e.type = 'm.room.message'
  ORDER BY stream_ordering DESC
  LIMIT 100;
```

#### 显示带有名称的房间，按房间中的事件排序

**使用 Bash 进行排序和排序**

```
echo "SELECT event_json.room_id, room_stats_state.name FROM event_json, room_stats_state \
WHERE room_stats_state.room_id = event_json.room_id" | psql -d synapse -h localhost -U synapse_user -t \
| sort | uniq -c | sort -n
```

`psql` 命令行参数的文档：https://www.postgresql.org/docs/current/app-psql.html

**使用 SQL 进行排序和订序**

```
SELECT COUNT(*), event_json.room_id, room_stats_state.name
  FROM event_json, room_stats_state
  WHERE room_stats_state.room_id = event_json.room_id
  GROUP BY event_json.room_id, room_stats_state.name
  ORDER BY COUNT(*) DESC
  LIMIT 50;
```

##### 结果示例:

```
   9459  !FPUfgzXYWTKgIrwKxW:matrix.org              | This Week in Matrix
   9459  !FPUfgzXYWTKgIrwKxW:matrix.org              | This Week in Matrix (TWIM)
  17799  !iDIOImbmXxwNngznsa:matrix.org              | Linux in Russian
  18739  !GnEEPYXUhoaHbkFBNX:matrix.org              | Riot Android
  23373  !QtykxKocfZaZOUrTwp:matrix.org              | Matrix HQ
  39504  !gTQfWzbYncrtNrvEkB:matrix.org              | ru.[matrix]
  43601  !iNmaIQExDMeqdITdHH:matrix.org              | Riot
  43601  !iNmaIQExDMeqdITdHH:matrix.org              | Riot Web/Desktop
```

#### 通过房间 ID 列表查找房间状态信息

您在使用管理员 API 时会得到相同的信息。

```
SELECT rss.room_id, rss.name, rss.canonical_alias, rss.topic, rss.encryption,
    rsc.joined_members, rsc.local_users_in_room, rss.join_rules
  FROM room_stats_state rss
  LEFT JOIN room_stats_current rsc USING (room_id)
  WHERE room_id IN (
    '!OGEhHVWSdvArJzumhm:matrix.org',
    '!YTvKGNlinIzlkMTVRl:matrix.org' 
  );
```

#### 显示一段时间未在线的用户和设备

```
SELECT user_id, device_id, user_agent, TO_TIMESTAMP(last_seen / 1000) AS "last_seen"
  FROM devices
  WHERE last_seen < DATE_PART('epoch', NOW() - INTERVAL '3 month') * 1000;
```

#### 清除远程用户设备列表的缓存

强制重新同步远程用户的设备列表 - 如果您以某种方式缓存了错误状态，并且远程服务器不会发送设备列表更新。

```
INSERT INTO device_lists_remote_resync
VALUES ('USER_ID', (EXTRACT(epoch FROM NOW()) * 1000)::BIGINT);
```

配置
--