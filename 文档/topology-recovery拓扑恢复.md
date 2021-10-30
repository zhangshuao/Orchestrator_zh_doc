# 支持的拓扑和版本

orchestrator支持以下设置:

* 普通老MySQL复制；经典的一个，基于日志文件+位置
* GTID复制。Oracle GTID和MariaDB GTID都受支持。
* 基于语句的复制（SBR）
* 基于行的复制（RBR）
* 半同步复制
* 单主机（又名标准）复制
* Master-Master 圆圈中的两个节点）复制
* 5.7并行复制
    * 当使用GTID时，没有进一步的限制。
    * 在顺序中使用伪GTID时，必须启用复制（请参阅slave_preserve_commit_order（http://dev.mysql.com/doc/refman/5.7/en/replication-options-slave.html#sysvar_slave_preserve_commit_order））。

不支持以下设置：
* Master-master... - 环中有3个或更多节点的master（循环）复制
* 5.6并行（每个schema线程）复制
* 多主机复制（一个副本从多个主机复制）
* Tungsten复制器
另请注意：

两个主节点支持 Master-master (ring环) 复制，不支持环中三个或更多主节点的拓扑。

Galera/XtraDB群集复制不受严格支持：orchestrator将无法识别Galera拓扑中的共同主节点是相关的。
在编辑者看来，每一个这样的主节点都是自己独特拓扑结构的负责人。

支持在同一主机上具有多个MySQL实例的复制拓扑。例如，orchestrator的测试环境由四个实例组成，所有实例都运行在同一台机器上，即MySQLSandbox。
然而，MySQL缺乏副本和主机之间的信息共享，这使得orchestrator无法自上而下地分析拓扑，因为主机不知道其副本正在侦听哪些端口。
默认的假设是复制副本与其主副本在同一端口上侦听。
在一台机器上（在同一网络上）有多个实例，这是不可能的。
在这种情况下，必须配置MySQL实例的report_host和report_port（读取更多http://code.openark.org/blog/mysql/the-importance-of-report_host-report_port）参数，并将orchestrator的配置参数DiscoverByShowSlaveHosts设置为true。
