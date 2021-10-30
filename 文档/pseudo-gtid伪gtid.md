# 伪GTID

伪GTID是一种将唯一项注入二进制日志的方法，这样它们就可以用于在没有直接连接的情况下match/sync匹配/同步副本，或者用于match/sync匹配/同步主副本已损坏corrupted/已死亡dead的副本。

伪GTID对不使用GTID的用户很有吸引力。Pseudo-GTID具有GTID的大部分优点，但没有做出GTID所要求的承诺。
使用Pseudo-GTID，您可以保留现有拓扑，无论您运行的是哪个版本的MySQL。

## 伪GTID的优点

* 启用master主故障切换。
* 启用中间主intermediate master故障切换。
* 任意重构，将副本从一个位置重新定位到另一个位置（即使是那些没有二进制日志记录的副本）。
* 供应商中立；在Oracle和MariaDB上都能工作，即使两者结合在一起。
* 没有配置更改。复制设置保持原样。
* 没有承诺。您可以随时选择远离伪GTID；停止写P-GTID条目。
* Pseudo GTID意味着对运行以下各项的复制副本进行崩溃安全复制：
    * log-slave-updates
    * sync_binlog=1
* 与MySQL 5.6上的GTID不同，服务器不必运行log-slave-updates，尽管建议使用log-slave-updates。

## 自动伪GTID注入

orchestrator可以为您注入伪GTID条目。请参阅自动伪GTID(https://github.com/openark/orchestrator/blob/master/docs/configuration-discovery-pseudo-gtid.md#automated-pseudo-gtid-injection)

## 人工伪GTID注入

自动伪GTID是后来添加的，它取代了手动伪GTID注入的需要，建议使用。
但是，您仍然可以选择注入您自己的伪GTID。

请参阅手动伪GTID注入(https://github.com/openark/orchestrator/blob/master/docs/pseudo-gtid-manual-injection.md)

## 限制

* 不支持Active-Active master-master复制
    * 支持主动-被动主机复制，其中仅在主动主机上注入伪GTID。

* 不运行log-slave-updates的复制副本通过中继日志同步。MySQL默认主动清除中继日志意味着，如果master服务器上发生崩溃，并且副本的中继日志刚刚被rotated滚动（即立即也被清除），那么中继日志中就没有用于修复拓扑的伪GTID信息
    * 频繁注入P-GTID可缓解此问题。我们每5秒注入一次P-GTID。

* 当副本读取基于语句的复制中继日志并中继基于行的复制二进制日志时（即，主副本具有binlog_格式=语句，副本具有binlog_格式=行），则orchestrator通过中继日志匹配伪GTID。
有关中继日志的限制，请参见上面的项目符号。

* 您无法匹配两台服务器，其中一台是完全RBR（接收和写入基于行的复制日志），另一台是完全SBR。当从基于SBR的拓扑迁移到RBR拓扑时，可能会发生这种情况。

* 当从5.6复制到5.7时，已知一个边缘情况场景：5.7将匿名语句添加到二进制日志中，orchestrator知道如何跳过。
但是，如果5.6->5.7复制中断（例如，dead master），并且匿名ANONYMOUS语句是二进制日志中的最后一条语句，则orchestrator此时无法对齐服务器。

## 部署伪GTID

请遵循deployment部署，伪GTID。(https://github.com/openark/orchestrator/blob/master/docs/deployment.md#pseudo-gtid)

### 使用伪GTID

通过Web：

![avatar](./图片/pseudo-gtid/图1.png)

通过命令行：

    orchestrator-client -c relocate -i some.server.to.relocate -d under.some.other.server
    
relocate命令将自动识别已启用伪GTID。