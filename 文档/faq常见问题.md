# Frequently Asked Questions常见问题

### 谁应该使用orchestrator？

    DBA和ops拥有不止一个master单副本复制拓扑(single-master-single-replica)。
    
### orchestrator能为我做些什么？

    Orchestrator分析您的复制拓扑并提供信息和操作：您将能够可视化和操作这些拓扑（重构复制路径）。
    Orchestrator可以监视和恢复拓扑故障，包括master宕机。
    
### 这是另一个监控工具吗？

    不。Orchestrator严格来说不是一个监视工具。我们无意这样做；没有警报或电子邮件。
    不过，它确实提供了拓扑状态的在线可视化，并且需要一些自己的阈值来管理拓扑。
    
### orchestrator支持哪种复制？

    Orchestrator支持"纯旧MySQL复制"，即使用二进制日志文件和位置的复制。如果你不知道你在用什么，这可能就是你想要的。

### orchestrator是否支持基于行的复制？

    支持. 基于语句的复制和基于行的复制都受支持（事实上，这种区别与orchestrator无关）
    
### orchestrator是否支持半同步复制？

    支持
    
### orchestrator是否支持master-master（环）复制？

    es，用于两个master环（主动-主动、主动-被动）。

    Master-Master-Master[-Master...] 拓扑, 其中环由3个或更多master组成，不受支持且未经测试。他们感到气馁。而且是一种可憎的东西。
    
### orchestrator是否支持Galera复制？

    是和否. Orchestrator不知道Galera复制。如果在每个主节点下有三个Galera主节点和不同的副本拓扑，则orchestrator将它们视为三种不同的拓扑。

### orchestrator是否支持GTID复制？

    支持，Oracle GTID和MariaDB GTID都受支持。
    
### orchestrator是否支持5.6并行复制（每个schema的线程）？

    不支持。这是因为在并行复制中不支持START SLAVE UNTIL，并且SHOW SLAVE STATUS的输出不完整。在这方面没有预期的工作。
    
### orchestrator是否支持5.7并行复制？

    对当使用GTID时，你们都很好。当使用伪GTID时，必须启用顺序复制（http://github.com/doc/refman/5.7/en/replication-options-slave.html#sysvar_slave_preserve_commit_order）。
    
### orchestrator是否支持Multi-Master多源复制？

    不支持。不支持多master复制（例如，在MariaDB 10.0中）。
    
### orchestrator是否支持Tungsten复制？
    
    不支持.

### orchestrator是否支持MySQL组复制？


    部分地。MySQL 8.0支持单主模式下的复制组。支持的范围是：

    * Orchestrator知道所有组成员都是同一集群的一部分，在实例发现过程中检索复制组信息，将其存储在其数据库中，并通过API公开。
    * orchestrator web UI显示单个主组成员。它们如下所示：
        * 从主组复制的所有辅助组成员。
        * 所有组成员都有一个图标，显示他们是组成员（与传统的异步/半同步副本不同）。
        * 将鼠标悬停在上面提到的图标上，可以提供有关组中DB实例的状态和角色的信息。
    
    * 禁止对组成员进行某些重新定位操作。特别是，orchestrator将拒绝重新定位辅助组成员，因为根据定义，它总是从primary组复制。
      它还将拒绝将主组重新定位到同一组的次组下的尝试。
    * 来自失败组成员的传统异步/半同步副本将重新定位到其他组成员。

### orchestrator是否支持另一种类型的复制？

    不支持.
    
### orchestrator是否支持...

    不支持.
    
### orchestrator是开源的吗？

    对Orchestrator在Apache许可证2.0下以开源形式发布，可从以下站点获得：https://github.com/openark/orchestrator
    
### 谁开发了orchestrator，为什么？

    Orchestrator由GitHub的Shlomi Noach(https://github.com/shlomi-noach)（以前在Booking.com和Outbrain）开发，用于帮助管理多个大型复制拓扑；迄今为止节省的时间和人为错误几乎是无价的。
    
### orchestrator是否与包含多个MySQL主要版本的集群协同工作？

    部分地。这通常发生在升级无法停机的群集时。每个副本都将脱机并升级到新的主版本，然后添加回群集，直到所有副本都升级完毕。
    Orchestrator知道MySQL版本，并允许在较低版本的主版本或中间主版本下移动主版本较高的副本，但不允许在较低版本的主版本或中间主版本下移动，因为上游供应商通常不支持这一点，即使它实际上可以工作。
    在大多数情况下，orchestrator将做正确的事情，并且它将允许在拓扑中安全地移动这样的副本。
    MySQL 5.5/5.6和5.6/5.7都广泛使用了这一功能，但在MariaDB 10中却没有这么多。如果您发现与此相关的问题，请报告。


