# Redis6种淘汰策略

| 策略            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| volatile-lru    | 从已设置过期时间的KV集中优先对最近最少使用(less recently used)的数据淘汰 |
| volitile-ttl    | 从已设置过期时间的KV集中优先对剩余时间短(time to live)的数据淘汰 |
| volitile-random | 从已设置过期时间的KV集中随机选择数据淘汰                     |
| allkeys-lru     | 从所有KV集中优先对最近最少使用(less recently used)的数据淘汰 |
| allKeys-random  | 从所有KV集中随机选择数据淘汰                                 |
| noeviction      | 不淘汰策略，若超过最大内存，返回错误信息                     |