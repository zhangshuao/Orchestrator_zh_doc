# 故障检测

orchestrator使用整体方法来检测master设备和中间master设备故障。

例如，在一种简单的方法中，监控工具将探测master服务器，并在无法联系或查询master服务器时发出警报。
这种方法容易受到网络故障引起的误报的影响。这种简单的方法通过运行间隔为t-long间隔的n个测试来进一步减少误报。
在某些情况下，这会减少误报的机会，然后在发生实际故障时增加响应时间。

orchestrator利用复制拓扑。它不仅观察服务器本身，还观察其副本。例如，要诊断master死机场景，orchestrator必须：

未能联系到上述master
能够联系master的副本，并确认他们也看不到master。

orchestrator不是按时间对错误进行分类，而是由多个观察者对复制拓扑服务器本身进行分类。
事实上，当一个master的所有副本都同意它们无法联系其master时，复制拓扑实际上就被破坏了，故障切换是合理的。

orchestrator的整体故障检测方法在生产中非常可靠

### 检测和恢复

检测并不总是导致恢复(https://github.com/openark/orchestrator/blob/master/docs/topology-recovery.md)。有些情况下不希望进行恢复：

    * 群集未列出自动故障切换。
    * 管理员用户已指示不应在特定服务器上进行恢复。
    * 管理员用户已全局禁用恢复。
    * 前一次在同一集群上的恢复不久前完成，并且发生anti-flapping takes place
    * 故障类型不值得恢复。

在所需的场景中，检测后立即恢复。在其他情况下（如被阻止的恢复），恢复可能会在几分钟后进行检测。
检测独立于恢复，并且始终处于启用状态。OnFailureDetectionProcesss每次检测都会执行hooks，请参阅故障检测配置。(https://github.com/openark/orchestrator/blob/master/docs/configuration-failure-detection.md)
 
### 故障检测场景

请遵守以下潜在故障列表：
    
    * DeadMaster
    * DeadMasterAndReplicas
    * DeadMasterAndSomeReplicas
    * DeadMasterWithoutReplicas
    * UnreachableMasterWithLaggingReplicas
    * UnreachableMaster
    * LockedSemiSyncMaster
    * MasterWithTooManySemiSyncReplicas
    * AllMasterReplicasNotReplicating
    * AllMasterReplicasNotReplicatingOrDead
    * DeadCoMaster
    * DeadCoMasterAndSomeReplicas
    * DeadIntermediateMaster
    * DeadIntermediateMasterWithSingleReplicaFailingToConnect
    * DeadIntermediateMasterWithSingleReplica
    * DeadIntermediateMasterAndSomeReplicas
    * DeadIntermediateMasterAndReplicas
    * AllIntermediateMasterReplicasFailingToConnectOrDead
    * AllIntermediateMasterReplicasNotReplicating
    * UnreachableIntermediateMasterWithLaggingReplicas
    * UnreachableIntermediateMaster
    * BinlogServerFailingToConnectToMaster

简单看一下一些示例，下面是orchestrator如何得出失败结论的：

**DeadMaster:**
    
    1.Master MySQL访问失败
    2.Master服务器的所有副本都无法复制
    
    这就形成了一个潜在的恢复过程

**DeadMasterAndSomeReplicas:**

    1.Master MySQL访问失败
    2.它的一些副本集也是不可达的
    3.其余副本复制失败
    
    这就形成了一个潜在的恢复过程
    
**UnreachableMaster:**

    1.Master MySQL访问失败
    2.但它有复制副本
    
    这不利于恢复过程。但是，为了改进分析，orchestrator将紧急重新读取副本，以确定他们是否真的对master服务器感到happy满意（在这种情况下，orchestrator可能由于网络故障而看不到master服务器），
    或者他们确实在花时间确定复制失败。
    
**DeadIntermediateMaster:**

    1.无法访问中间master服务器（具有 副本的副本）
    2.它的所有副本都无法复制
    
    这是一个潜在的恢复过程。
    
**UnreachableMasterWithLaggingReplicas:**

    1.无法联系到Master
    2.它的所有直接副本（不包括SQL延迟）都是滞后的
    
    当master过载时，可能会发生这种情况。客户端会看到"Too many connections"，而很久以前就连接过的复制副本则声称master节点没有问题。
    类似地，如果master由于某些元数据操作而被锁定，则客户端将在连接时被阻止，而副本可能会声称一切正常。
    但是，由于应用程序无法连接到主应用程序，因此不会写入任何实际数据，并且当使用心跳机制（如pt-heartbeat）时，我们可以观察到副本上的延迟越来越大。
    
    orchestrator通过在所有master的即时副本上重新启动复制来响应此场景。这将关闭这些副本上的旧客户端连接，并尝试启动新的客户端连接。
    它们现在可能无法连接，从而导致所有副本上的完全复制失败。这将引导协调器分析DeadMaster。
    
**LockedSemiSyncMaster:**

    1.Master在启用半同步的情况下运行（rpl_semi_sync_master_enabled=1）
    2.连接的半同步复制副本数量低于预期的rpl_semi_sync_master_wait_for_slave_count 
    3.rpl_semi_sync_master_timeout足够高，以便master锁定写入，而不会退回到异步复制
    
    此条件仅在经过合理的LockedSemiSyncMasterSeconds后触发。如果未设置reasonalLockedSemiSyncMasterSeconds，它将在reasonalReplicationLagSeconds后触发。

    此情况的补救措施可以是在主机上禁用半同步，或启动（或启用）足够的半同步副本。
    
    如果启用了EnforceExactSemiSyncReplicas，orchestrator将确定所需的半同步拓扑，并在副本上启用/禁用半同步以匹配它。
    所需拓扑由优先级顺序（见下文）和master等待计数定义。
    
    如果已启用RecoverLockedSemiSyncMaster，则orchestrator将按优先级顺序在副本上启用（但从不禁用）半同步，直到半同步副本的数量与主机等待计数匹配。
    请注意，如果设置了EnforceExactSemiSyncReplicas，RecoverLockedSemiSyncMaster将无效。
    
    优先级顺序由DetectSamisyncEnforcedQuery（数字越高优先级越高）、升级规则（DetectPromotionRuleQuery）和hostname主机名（回退）定义。
    
    如果EnforceExactSemiSyncReplicas和RecoverLockedSemiSyncMaster都已禁用（默认），则orchestrator不会为此类分析调用任何恢复过程。
    
    有关更多详细信息，请参阅半同步拓扑文档。(https://github.com/openark/orchestrator/blob/master/docs/configuration-discovery-classifying.md#semi-sync-topology)
    
**MasterWithTooManySemiSyncReplicas:**

    1.Master在启用半同步的情况下运行（rpl_semi_sync_master_enabled=1）
    2.连接的半同步副本数高于预期的rpl_semi_sync_master_wait_for_slave_count
    3.EnforceExactSemiSyncReplicas已启用（如果未启用此标志，则不会触发此分析）

    如果启用了EnforceExactSemiSyncReplicas，orchestrator将确定所需的半同步拓扑，并在副本上启用/禁用半同步以匹配它。所需拓扑由优先级顺序和master等待计数定义。

    优先级顺序由DetectSamisyncEnforcedQuery（数字越高优先级越高）、升级规则（DetectPromotionRuleQuery）和hostname主机名（回退）定义。
    
    如果禁用了EnforceExactSemiSyncReplicas（默认），orchestrator将不会为此类型的分析调用任何恢复过程。

    有关更多详细信息，请参阅半同步拓扑文档。(https://github.com/openark/orchestrator/blob/master/docs/configuration-discovery-classifying.md#semi-sync-topology)
    
### 无益的失败

orchestrator对以下场景不感兴趣，虽然orchestrator可以使用这些信息和状态，但它不会将这些场景本身视为失败；
没有调用检测hooks，显然也没有尝试恢复：

* 简单副本失败（保留在复制拓扑图上）异常：半同步副本导致LockedSemiSyncMaster
* 复制延迟，甚至严重。

### 可见度

最新分析可通过以下途径获得：

* 命令行: orchestrator-client -c replication-analysis 或 orchestrator -c replication-analysis
* Web API: /api/replication-analysis
* Web: /web/clusters-analysis/（Clusters->Failure analysis故障分析）。这是一个不完整的问题列表，只突出了可采取行动的问题。

阅读下一篇：拓扑恢复(https://github.com/openark/orchestrator/blob/master/docs/topology-recovery.md)