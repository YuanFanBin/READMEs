# OpenStack Telemetry 云监控报警服务

* [Telemetry](#telemetry)
    * [Telemetry架构设计](#架构设计)
* [Gnocchi](#gnocchi-开源时间序列数据库)
    * [Gnocchi 介绍](#gnocchi-介绍)
    * [Gnocchi 支持的特性](#gnocchi-支持的特性)
    * [Gnocchi 架构](#gnocchi-架构)
    * [Gnocchi 后端存储(back-ends)](#gnocchi-后端存储back-ends)
    * [Gnocchi 如何理解聚合(aggregation)](#gnocchi-如何理解聚合aggregation)

## Telemetry

- [Ceilometer](https://docs.openstack.org/ceilometer/pike/index.html): 采集计量数据并加工预处理

- [Gnocchi](https://gnocchi.xyz/index.html): 资源索引和存储时序计量数据

- [Aodh](https://docs.openstack.org/aodh/latest/): 预警和计量通知服务

- [Panko](https://docs.openstack.org/panko/latest/): 事件存储服务

### 架构设计

![High-Level Architecture](https://docs.openstack.org/ceilometer/pike/_images/ceilo-arch.png)

### 学习资料

* [Pike - Ceilometer](https://docs.openstack.org/ceilometer/pike/index.html)
* [支持监控的数据类型](https://docs.openstack.org/ceilometer/pike/admin/index.html#data-types)
* [探索 OpenStack 之（16）：计量模块 Ceilometer 介绍及优化](http://www.cnblogs.com/sammyliu/p/4383289.html)
* [如何基于Openstack telemetry项目实现云监控报警服务](http://www.infoq.com/cn/articles/how-to-implement-cloud-monitoring-alarm-service)
* [OpenStack —— 计量服务Ceilometer(九)](http://blog.51cto.com/wzlinux/1964612)
* [OpenStack/Gnocchi简介和架构(附文档翻译)](http://blog.sina.com.cn/s/blog_6de3aa8a0102wk0y.html)

--------------------------------------------------------------------------------

## Gnocchi 开源时间序列数据库

### Gnocchi 介绍

Gnocchi解决了时间序列数据和资源大规模存储及索引问题。

Gnocchi被设计来处理数据的大量聚合操作，同时保持其高性能，可扩展性和容错性。

Gnocchi采用独特的方法来存储时间序列数据：不是存储原生数据，而是在存储数据前将其聚合。

由于Gnocchi在整合数据前已经做了聚合操作，因此获取相应聚合数据非常快，仅仅是读取已聚合结果。

### Gnocchi 支持的特性

- **HTTP REST interface**
- **Horizontal scalability**
- **Metric aggregation**
- **Measures batching support**
- Archiving policy
- **Metric value search**
- Structured resources
- Resource history
- Queryable resource indexer
- Multi-tenant
- Grafana support
- Prometheus Remote Write support
- Nagios/Icinga support
- Statsd protocol support
- Collectd plugin support
- InfluxDB line protocol ingestion support

### Gnocchi 架构

Gnocchi由如下3个服务组成：

- [HTTP REST API](https://gnocchi.xyz/stable_4.2/rest.html)：接收数据

- 可选的[statsd兼容守护程序](https://gnocchi.xyz/stable_4.2/statsd.html)：接收数据

- 异步处理守护程序(gnocchi-metricd)：在后台对接收到的数据进行统计计算，度量清理等

![architecture](https://gnocchi.xyz/stable_4.2/_images/architecture.svg)

**所有服务均无状态，因此非常方便水平扩展。**

正如架构图所示，Gnocchi还需要3个后端存储组件。

- 输入度量存储(An incoming measure storage)

- 聚合度量存储(An aggregated metric storage)

- 索引存储(An index)

### Gnocchi 后端存储(back-ends)

1. Incoming and storage drivers

Gnocchi可以利用不同的存储系统来存储测量及汇总数据

- File(default)

- Ceph(perferred): 利于水平扩展，Ceph提供更好的一致性

- OpenStack Swift: 利于水平扩展

- Amazon S3: 利于水平扩展

- Redis

2. Indexer driver

- PostgreSQL(perfered)

- MySQL(version >= 5.6.4)

索引器负责存储所有资源，归档策略，metrics以及其定义，类型和属性。索引器也会负责资源及资源指标之间的关系。

### Gnocchi 如何理解聚合(aggregation)

数据通过归档策略来做相应聚合，归档策略定义了需要做的聚合操作以及保留哪些聚合数据。

Gnocchi支持多种聚合方法：最大值，最小值，平均值，百分比，标准差等。

Gnocchi 使用三种不同的后端来存储数据：

- the incoming driver: 新传入的度量数据
- the storage drvier: 时间序列聚合数据
- the index driver: 索引数据

通常前两种使用同样的后端存储

### Gnocchi 如何设置归档策略


**时间序列数据库**

[时间序列数据库的秘密（1）- 介绍](http://www.infoq.com/cn/articles/database-timestamp-01)
[时间序列数据库的秘密（2）- 索引](http://www.infoq.com/cn/articles/database-timestamp-02)
[时间序列数据库的秘密（3）- 加载和分布式计算](http://www.infoq.com/cn/articles/database-timestamp-03)

--------------------------------------------------------------------------------

## 相关图片来源

[OpenSatck官方文档](https://docs.openstack.org/)
