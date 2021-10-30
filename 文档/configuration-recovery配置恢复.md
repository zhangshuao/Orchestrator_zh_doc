# 配置: recovery

orchestrator将恢复拓扑的故障。您将指示orchestrator自动恢复哪些群集，以及期望人工恢复哪些群集。您将为orchestrator配置hooks钩子以移动VIP、更新服务发现等。

恢复取决于检测，在配置：故障检测中讨论(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/configuration-failure-detection.md)

请参阅拓扑恢复（https://hub.fastgit.org/openark/orchestrator/blob/master/docs/topology-recovery.md）以了解所有内容的恢复。

还考虑到MySQL拓扑本身需要遵循一些规则，请参见MySQL配置。（https://hub.fastgit.org/openark/orchestrator/blob/master/docs/configuration-recovery.md#mysql-configuration）

    {
      "RecoveryPeriodBlockSeconds": 3600,
      "RecoveryIgnoreHostnameFilters": [],
      "RecoverMasterClusterFilters": [
        "thiscluster",
        "thatcluster"
      ],
      "RecoverIntermediateMasterClusterFilters": [
        "*"
      ],
    }

在上述情况下：

* orchestrator将自动恢复所有群集的中间主机故障
* orchestrator将自动恢复两个指定群集的主故障；其他群集的主机将不会自动恢复。人类将能够开始恢复。
* 一旦集群经历了恢复，orchestrator将在接下来的3600秒（1小时）内阻止自动恢复。这是一种（anti-flapping）防拍打机制。

请再次注意，自动恢复是选择性加入。

### 提升动作

不同的环境要求在recovery/promotion恢复/升级时采取不同的操作

    {
      "ApplyMySQLPromotionAfterMasterFailover": true,
      "PreventCrossDataCenterMasterFailover": false,
      "PreventCrossRegionMasterFailover": false,
      "FailMasterPromotionOnLagMinutes": 0,
      "FailMasterPromotionIfSQLThreadNotUpToDate": true,
      "DelayMasterPromotionIfSQLThreadNotUpToDate": false,
      "MasterFailoverLostInstancesDowntimeMinutes": 10,
      "DetachLostReplicasAfterMasterFailover": true,
      "MasterFailoverDetachReplicaMasterHost": false,
      "MasterFailoverLostInstancesDowntimeMinutes": 0,
      "PostponeReplicaRecoveryOnLagMinutes": 0,
    }

### Hooks

这些挂钩可用于恢复：

* PreGracefulTakeoverProcesses: 在计划的、优雅的主机接管时执行，紧接着主机变为read-only之前。
* PreFailoverProcesses: 在orchestrator执行恢复操作之前立即执行。任何这些进程的故障（非零退出代码）都会中止恢复。
                        提示：这使您有机会根据系统的某些内部状态中止恢复。
* PostMasterFailoverProcesses: 在成功的master恢复结束时执行。
* PostIntermediateMasterFailoverProcesses: 在具有副本恢复的成功中间主机或复制组成员结束时执行。
* PostFailoverProcesses: 在任何成功恢复结束时执行（包括并添加到上述两项）。
* PostUnsuccessfulFailoverProcesses: 在任何不成功的恢复结束时执行。
* PostGracefulTakeoverProcesses: 在旧master被安置在新提升的master之下后，按计划、优雅的master接管执行。

任何以"&"结尾的进程命令都将异步执行，并且忽略该进程的失败。

以上所有内容都是orchestrator按定义顺序顺序执行的命令列表。

简单的实现可能如下所示：

    {
      "PreGracefulTakeoverProcesses": [
        "echo 'Planned takeover about to take place on {failureCluster}. Master will switch to read_only' >> /tmp/recovery.log"
      ],
      "PreFailoverProcesses": [
        "echo 'Will recover from {failureType} on {failureCluster}' >> /tmp/recovery.log"
      ],
      "PostFailoverProcesses": [
        "echo '(for all types) Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:{failedPort}; Successor: {successorHost}:{successorPort}' >> /tmp/recovery.log"
      ],
      "PostUnsuccessfulFailoverProcesses": [],
      "PostMasterFailoverProcesses": [
        "echo 'Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:     {failedPort}; Promoted: {successorHost}:{successorPort}' >> /tmp/recovery.log"
      ],
      "PostIntermediateMasterFailoverProcesses": [],
      "PostGracefulTakeoverProcesses": [
        "echo 'Planned takeover complete' >> /tmp/recovery.log"
      ],
    }

#### Hooks钩子参数和环境

orchestrator为所有hooks提供与failure/recovery故障/恢复相关的信息，如故障实例的标识、升级实例的标识、受影响的副本、故障类型、群集名称等。

此信息以两种方式独立传递，您可以选择使用其中一种或两种方式：

环境变量：orchestrator将设置以下可由hooks检索的变量：


    * ORC_FAILURE_TYPE
    * ORC_INSTANCE_TYPE ("master", "co-master", "intermediate-master")
    * ORC_IS_MASTER (true/false)
    * ORC_IS_CO_MASTER (true/false)
    * ORC_FAILURE_DESCRIPTION
    * ORC_FAILED_HOST
    * ORC_FAILED_PORT
    * ORC_FAILURE_CLUSTER
    * ORC_FAILURE_CLUSTER_ALIAS
    * ORC_FAILURE_CLUSTER_DOMAIN
    * ORC_COUNT_REPLICAS
    * ORC_IS_DOWNTIMED
    * ORC_AUTO_MASTER_RECOVERY
    * ORC_AUTO_INTERMEDIATE_MASTER_RECOVERY
    * ORC_ORCHESTRATOR_HOST
    * ORC_IS_SUCCESSFUL
    * ORC_LOST_REPLICAS
    * ORC_REPLICA_HOSTS
    * ORC_COMMAND ("force-master-failover", "force-master-takeover", "graceful-master-takeover" if applicable)

而且，如果恢复成功：

    * ORC_SUCCESSOR_HOST
    * ORC_SUCCESSOR_PORT
    * ORC_SUCCESSOR_BINLOG_COORDINATES
    * ORC_SUCCESSOR_ALIAS

命令行文本替换。orchestrator将替换*Processes命令中的以下魔术标记：


    * {failureType}
    * {instanceType} ("master", "co-master", "intermediate-master")
    * {isMaster} (true/false)
    * {isCoMaster} (true/false)
    * {failureDescription}
    * {failedHost}
    * {failedPort}
    * {failureCluster}
    * {failureClusterAlias}
    * {failureClusterDomain}
    * {countReplicas} (replaces {countSlaves})
    * {isDowntimed}
    * {autoMasterRecovery}
    * {autoIntermediateMasterRecovery}
    * {orchestratorHost}
    * {lostReplicas} (replaces {lostSlaves})
    * {countLostReplicas}
    * {replicaHosts} (replaces {slaveHosts})
    * {isSuccessful}
    * {command} ("force-master-failover", "force-master-takeover", "graceful-master-takeover" if applicable)

而且，如果恢复成功：

    * {successorHost}
    * {successorPort}
    * {successorBinlogCoordinates}
    * {successorAlias}

### MySQL配置

为了支持故障切换，您的MySQL拓扑必须满足一些要求。这些要求在很大程度上取决于您使用的topologies/configuration拓扑/配置类型。

* Oracle/Percona的GTID: 可升级服务器必须启用log_bin和log_slave_updates。复制副本必须使用AUTO_POSITION=1（通过将MASTER更改为MASTER_AUTO_POSITION=1）。
* MariaDB GTID:可升级服务器必须启用log_bin和log_slave_updates。
* 伪GTID（https://hub.fastgit.org/openark/orchestrator/blob/master/docs/configuration-recovery.md#pseudo-gtid）：可升级服务器必须启用log_bin和log_slave_updates。如果使用5.7/8.0并行复制，请将slave_preserve_commit_order=1。
* BinlogServers:可升级服务器必须启用log_bin。

还考虑通过MySQL配置(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/configuration-failure-detection.md#mysql-configuration)改进故障检测



