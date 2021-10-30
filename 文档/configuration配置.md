# 配置

记录和解释所有配置变量变成了一项艰巨的任务，就像用文字解释代码一样。在修剪和简化配置方面有一项持续的工作。

事实上的配置列表位于config.go（https://hub.fastgit.org/openark/orchestrator/blob/master/go/config/config.go）中。

毫无疑问，您对配置一些基本组件感兴趣：后端数据库、主机和发现。
您可以选择使用伪GTID。您可能希望orchestrator在出现故障时发出通知，或者希望运行全面的自动恢复。

使用以下小步骤配置orchestrator：

    * Backend(https://github.com/openark/orchestrator/blob/master/docs/configuration-backend.md)
    * Discovery: basic (https://github.com/openark/orchestrator/blob/master/docs/configuration-discovery-basic.md)
    * Discovery: resolving names (https://github.com/openark/orchestrator/blob/master/docs/configuration-discovery-resolve.md)
    * Discovery: classifying servers (https://github.com/openark/orchestrator/blob/master/docs/configuration-discovery-classifying.md)
    * Discovery: Pseudo-GTID (https://github.com/openark/orchestrator/blob/master/docs/configuration-discovery-pseudo-gtid.md)
    * Topology control (https://github.com/openark/orchestrator/blob/master/docs/configuration-topology-control.md)
    * Failure detection (https://github.com/openark/orchestrator/blob/master/docs/configuration-failure-detection.md)
    * Recovery (https://github.com/openark/orchestrator/blob/master/docs/configuration-recovery.md)
    * Raft(https://github.com/openark/orchestrator/blob/master/docs/configuration-raft.md): 为高可用性配置 orchestrator/raft集群 (https://hub.fastgit.org/openark/orchestrator/blob/master/docs/raft.md)
    * Security: 见安全(https://github.com/openark/orchestrator/blob/master/docs/security.md)部分
    * Key-Value stores(https://github.com/openark/orchestrator/blob/master/docs/configuration-kv.md): 为master发现配置和使用键值存储。
    * 有关适用于大型orchestrator环境的某些设置的提示(https://github.com/openark/orchestrator/blob/master/docs/configuration-large.md)

#### 配置示例文件

    为方便起见，此示例配置(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/configuration-sample.md)是GitHub上生产编排器配置的一种修订形式。
