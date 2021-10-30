# 配置：发现、分类服务器

orchestrator将计算出集群、数据中心等的名称。

    {
      "ReplicationLagQuery": "select absolute_lag from meta.heartbeat_view",
      "DetectClusterAliasQuery": "select ifnull(max(cluster_name), '') as cluster_alias from meta.cluster where anchor=1",
      "DetectClusterDomainQuery": "select ifnull(max(cluster_domain), '') as cluster_domain from meta.cluster where anchor=1",
      "DataCenterPattern": "",
      "DetectDataCenterQuery": "select substring_index(substring_index(@@hostname, '-',3), '-', -1) as dc",
      "PhysicalEnvironmentPattern": "",
      "DetectSemiSyncEnforcedQuery": ""
    }

## 复制延迟

默认情况下，orchestrator使用SHOW SLAVE STATUS，并采用1秒的延迟粒度值。但是，此延迟不考虑链式复制事件中的级联延迟。
许多人使用定制的心跳机制，如pt-heartbeat。这提供了与master的"absolute"滞后，以及亚秒分辨率。

ReplicationLagQuery允许您设置自己的查询。

## 群集别名

在您的公司，不同的集群有共同的名称。"Main"、"Analytics"、"Shard031"等。但是MySQL集群本身并不知道这些名称。

DetectClusterAliasQuery是一种查询，通过该查询，您可以让orchestrator知道集群的名称。

名字很重要。您可能会使用它告诉orchestrator类似的内容："please auto recover this cluster请自动恢复此群集"，或"what are all the participating instances in this cluster此群集中的所有参与实例是什么"。

要使查询能够访问这些数据，一个技巧是在某个元模式中创建一个表：

    CREATE TABLE IF NOT EXISTS cluster (
      anchor TINYINT NOT NULL,
      cluster_name VARCHAR(128) CHARSET ascii NOT NULL DEFAULT '',
      cluster_domain VARCHAR(128) CHARSET ascii NOT NULL DEFAULT '',
      PRIMARY KEY (anchor)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

... 并按如下方式填充（例如，通过puppet/cron）：

    mysql meta -e "INSERT INTO cluster (anchor, cluster_name, cluster_domain) \
      VALUES (1, '${cluster_name}', '${cluster_domain}') \
      ON DUPLICATE KEY UPDATE \     cluster_name=VALUES(cluster_name), cluster_domain=VALUES(cluster_domain)"
      
也许您的主机命名约定会公开群集名称，您只需要对@@hostname进行简单的查询。

## 数据中心

orchestrator支持数据中心。它不仅可以在web界面上很好地为它们着色；但在运行故障切换时，它将考虑DC。

您将使用以下两种方法之一配置数据中心感知：

DataCenterPattern：要在fqdn上使用的正则表达式。e、 g.：“db-.*？-.*？[.]（.*？[.].myservice[.]com”
DetectDataCenterQuery：返回数据中心名称的查询

## 群集域

更不重要的是，主要是为了可见性，DetectClusterDomainQuery应该返回VIP或CNAME或集群主机的地址

## 半同步拓扑

在某些环境中，不仅要控制半同步副本的数量，还要控制副本是半同步还是异步副本。orchestrator可以检测到不需要的半同步配置，并切换半同步标志rpl_semi_sync_slave_enabled和rpl_semi_sync_master_enabled以纠正这种情况。

#### Semi-sync master (rpl_semi_sync_master_enabled)

如果DetectSemiscEnforcedQuery为新主机返回大于0的值，则orchestrator在主机故障切换（例如DeadMaster）期间启用半同步主机标志。
如果主标志被更改或设置不正确，orchestrator不会触发任何恢复。

半同步主机可以进入两种故障情况: LockedSemiSyncMaster(https://github.com/openark/orchestrator/blob/master/docs/failure-detection.md#lockedsemisyncmaster) 和 MasterWithTooManySemiSyncReplicas(https://github.com/openark/orchestrator/blob/master/docs/failure-detection.md#masterwithtoomanysemisyncreplicas)。orchestrator在恢复这两种情况之一的过程中禁用半同步副本上的半同步主标志。

#### Semi-sync replicas (rpl_semi_sync_slave_enabled)

orchestrator可以检测拓扑中的半同步副本数量是否不正确（LockedSemiSyncMaster(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/failure-detection.md#lockedsemisyncmaster)和MasterWithTooManySemiSyncReplicas(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/failure-detection.md#masterwithtoomanysemisyncreplicas)），然后可以通过相应地 启用/禁用 semi-sync半同步副本标志来纠正这种情况。

此行为可以通过以下选项控制：

    * DetectSyncEnforcedQuery：返回半同步优先级的查询（零表示异步副本；数字越大表示优先级越高）
    * EnforceExactSemiSyncReplicas：决定是否强制执行严格的半同步副本拓扑的标志。如果启用，LockedSemiSyncMaster和MasterWithTooManyReplicas的恢复将启用和禁用副本上的半同步，以完全基于优先级顺序匹配所需拓扑。
    * RecoverLockedSemiSyncMaster：决定是否从LockedSemiSyncMaster方案中恢复的标志。如果启用，LockedSemiSyncMaster的恢复将按优先级顺序在副本上启用（但从不禁用）半同步，以匹配主机等待计数。
      如果设置了EnforceExactSemiSyncReplicas，则此选项无效。如果您只想处理半同步副本太少的情况，而不想处理副本太多的情况，那么它非常有用。
    * reasonalLockedSemiSyncMasterSeconds：触发LockedSemiSyncMaster条件的秒数；如果未设置，则返回到合理的ReplicationLagSeconds
    
    优先级顺序由DetectSemiscEnforcedQuery（零表示异步副本；数字越大优先级越高）、升级规则（DetectPromotionRuleQuery）和主机名（fallback回退）定义。

示例1：强制实施严格的半同步副本拓扑，rpl_semi_sync_master_wait_for_slave_count=1：

    "DetectSemiSyncEnforcedQuery": "select priority from meta.semi_sync where cluster_member = @@hostname",
    "EnforceExactSemiSyncReplicas": true

假设这种拓扑结构，

         ,- replica1 (priority = 10, rpl_semi_sync_slave_enabled = 1)
    master 
         `- replica2 (priority = 20, rpl_semi_sync_slave_enabled = 1)

orchestrator将检测具有ToomAnySemisyncRecReplicates(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/failure-detection.md#masterwithtoomanysemisyncreplicas)场景的Master，并禁用replica1上的半同步（低优先级）。

示例2：强制执行弱半同步副本拓扑，rpl_semi_sync_master_wait_for_slave_count=1:

    "DetectSemiSyncEnforcedQuery": "select 2586",
    "DetectPromotionRuleQuery": "select promotion_rule from meta.promotion_rules where cluster_member = @@hostname",
    "RecoverLockedSemiSyncMaster": true

假设这种拓扑结构，
    
        ,- replica1 (priority = 2586, promotion rule = prefer, rpl_semi_sync_slave_enabled = 0)
    master 
        `- replica2 (priority = 2586, promotion rule = neutral, rpl_semi_sync_slave_enabled = 0)
   
orchestrator将检测到LockedSemiSyncMaster(https://github.com/openark/orchestrator/blob/master/docs/failure-detection.md#lockedsemisyncmaster)方案，并在replica1上启用半同步（更可取的升级规则）。

          