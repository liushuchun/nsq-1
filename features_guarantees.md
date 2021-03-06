# 特性和担保

**NSQ** 是分布式实时消息系统.

## 特性

 * 没有SPOF支持分布式拓扑
 * 水平扩展(没有经纪人,无缝地添加更多的节点到集群)
 * 低延迟消息传递 ([性能][performance])
 * 结合负载均衡*和*多播消息路由风格
 * 擅长面向流媒体(高通量)和工作(低吞吐量)工作负载
 * 主要是内存中(除了高水位线消息透明地保存在磁盘上)
 * 运行时发现消费者找到生产者服务([nsqlookupd][nsqlookupd])
 * 传输层安全性 (TLS)
 * 数据格式不可知
 * 一些依赖项(容易部署)和理智的,有界,默认配置
 * 在任何语言简单 TCP 协议支持客户端库
 * HTTP接口统计、管理行为和生产者(*不需要客户端库发布*)
 * 为实时检测集成了 [statsd][statsd] 
 * 健壮的集群管理界面 ([nsqadmin][nsqadmin])

## 担保

对于任何分布式系统来说，都是通过一些列的智能取舍来实现目标。通过这些智能的取舍，我们希望能使得 NSQ 在部署到产品上的行为是可预期的，而这些对于我们来说又是透明的。 通过透明的现实我们希望设定预期如何这些权衡 **NSQ** 将部署在生产时的行为。

## 消息不可持久化（默认）

虽然系统支持消息的持久化存储在磁盘中（通过 `--mem-queue-size` ），不过默认情况下消息都在*内存*中.

如果将 `--mem-queue-size` 设置为 0，所有的消息将会存储到磁盘。我们不用担心消息会丢失，nsq 内部机制保证在程序关闭时持久化队列中的数据到硬盘，重启后就会恢复。


NSQ 没有内置的复制机制,却有各种各样的方法管理这种权衡.比如部署拓扑结构和技术，在容错的时候从属并持久化内容到磁盘。

## 消息最少会被投递一次

如上所述，这个假设成立于 `nsqd` 节点没有失败时。

因为各种原因，消息可以被投递多次（客户端超时，连接失效，重新排队，等等）。由客户端负责操作。

## 接收到的消息是*无序的*

不要依赖于投递给消费者的消息的顺序。

和投递消息机制类似，它是由重新队列，内存和磁盘存储的混合导致的，实际上，节点间不会共享任何信息。

它是相对的简单完成*松次序*，（例如，对于某个消费者来说，消息是由次序的，但是不恩给你作为一个整体跨集群），通过使用时间窗来接收消息，并在处理前排序（虽然为了维持这个变量，必须抛弃时间窗外的消息）。

## 消费者*最终*找出所有话题的生产者

这个服务([nsqlookupd][nsqlookupd]) 被设计成最终一致。`nsqlookupd` 节点不会维持状态，也不会回答查询。

网络分区并不会影响可用性，分区的双方仍然能回答查询。部署拓扑结果可以显著的建设这些问题。

[performance]: performance.md
[nsqlookupd]: https://github.com/bitly/nsq/tree/master/nsqlookupd/README.md
[nsqadmin]: https://github.com/bitly/nsq/tree/master/nsqadmin/README.md
[statsd]: https://github.com/etsy/statsd/
[graphite]: http://graphite.wikidot.com/
