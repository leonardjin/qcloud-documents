## CKafka 支持的版本？是否暴露 zookeeper？

支持 0.9~0.10 版本，内部兼容。
不开放 zookeeper，不提供 zk 地址。

##  新接入客户端时生产/消费错误

- 检查 telnet 是否通（网络问题，是否 Kafka 和生产者在相同网络环境下）。
- 访问的 vip - port 是否配置正确。
- topic 白名单是否开启，如果开启需要配置正确的 IP 才能访问。

## kafka console 客户端测试看不到数据
-	消费者采用 latest 时候只会获取最后的数据，需要同时保持生产才可以看到相应数据。
-	改为 earliest 方式消费数据。

## 生产一段时间后发现持续性错误
- 查看是否流量流控，在监控页面上查询是否有波峰，升级实例大小解决。
- 查看是否容量流控，在监控页面上查询实例的容量，升级实例大小或者修改数据保存时间。

## 流控相关说明
-	流控是针对实例的，实例下所有 topic 都会受影响。
-	容量满后可以消费，但是无法继续生产。
-	流量=实际流量\*副本数目
-	堆积=实际堆积（副本堆积也算）

## 消费异常
-	查看是否流量流控，在 barad 上查询是否有波峰，升级实例大小解决。
-	查看消费分组是否超过数量。
-	如果因为网络频繁 rebalance，建议调整客户端超时时间。
-	是否拉取过期的 offset，消息过期会被删除，如果用过期 offset 拉取会失败。

