# 配置: 发现，伪GTID

    orchestrator将识别二进制日志中的神奇提示，使其能够像处理GTID一样处理非GTID拓扑，包括重新定位副本、智能故障切换等等。
    
    请参阅伪GTID(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/pseudo-gtid.md)

### 自动伪GTID注入

orchestrator可以为您注入伪GTID条目并为您省去麻烦。您将配置：

    {
      "AutoPseudoGTID": true,
    }

您可以忽略任何其他与伪GTID相关的配置（它们都将被orchestrator隐式覆盖）。

您还需要在MySQL服务器上授予以下权限：

    GRANT DROP ON _pseudo_gtid_.* to 'orchestrator'@'orch_host';
    
注意：_pseudo_gtid_ schema不需要存在。没有必要创建它。orchestrator将运行以下表单的查询： 

    drop view if exists `_pseudo_gtid_`.`_asc:5a64a70e:00000001:c7b8154ff5c3c6d8`

这些语句除了在二进制日志中充当神奇的标记外，什么都不会做。

orchestrator将仅在允许的情况下尝试注入伪GTID。如果要将伪GTID注入限制到特定集群，可以通过仅授予orchestrator要注入伪GTID的集群上的权限来实现。
您可以通过以下方式禁用特定群集上的伪GTID注入：

    REVOKE DROP ON _pseudo_gtid_.* FROM 'orchestrator'@'orch_host';

自动伪GTID注入是一个较新的开发，它取代了您运行自己的伪GTID注入的需要。

如果您希望在运行手动伪GTID注入后启用自动伪GTID注入，您将很高兴地注意到：

* 您将不再需要管理伪GTID service / event调度器。
* 特别是，在master设备故障切换时，不需要在old/promoted主设备disable/enable伪GTID。

### 人工伪GTID注入

建议使用自动伪GTID(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/configuration-discovery-pseudo-gtid.md#automated-pseudo-gtid-injection)方法。

如果您希望自己注入伪GTID，我们建议您进行如下配置：

    {
      "PseudoGTIDPattern": "drop view if exists `meta`.`_pseudo_gtid_hint__asc:",
      "PseudoGTIDPatternIsFixedSubstring": true,
      "PseudoGTIDMonotonicHint": "asc:",
      "DetectPseudoGTIDQuery": "select count(*) as pseudo_gtid_exists from meta.pseudo_gtid_status where anchor = 1 and time_generated > now() - interval 2 hour",
    }
    
上述假设：

* 你有一个 meta schema
* 您将通过这个示例（https://hub.fastgit.org/openark/orchestrator/tree/master/resources/pseudo-gtid）脚本注入伪GTID条目
