# 生产中的Orchestrator部署

orchestrator部署是什么样子的？你需要在puppet/chef中设置什么？需要运行哪些服务以及在哪里运行？

## 部署服务&客户端

您将首先决定是要在共享后端数据库上运行orchestrator，还是使用raft设置。
有关某些选项，请参阅高可用性，以及orchestrator/raft vs 同步复制设置(https://github.com/openark/orchestrator/blob/master/docs/raft-vs-sync-repl.md)的比较和讨论。

    请遵循以下部署指南：
    * 在共享后端数据库(https://github.com/openark/orchestrator/blob/master/docs/deployment-shared-backend.md)上部署orchestrator
    * 通过raft部署(https://github.com/openark/orchestrator/blob/master/docs/deployment-raft.md)orchestrator

## 下一步

    orchestrator在动态环境中工作良好，能够适应资源清册、配置和拓扑的变化。
    它的动态特性表明，环境也应该在动态特性中与它相互作用。与硬编码配置不同，orchestrator乐于接受动态提示和请求，以改变其对拓扑的看法。
    一旦部署了orchestrator服务和客户端，请考虑执行以下操作以充分利用这一点。

### 发现拓扑

    orchestrator自动发现连接拓扑的boxes。如果一个新副本加入由orchestrator监控的现有集群，则在orchestrator下一步探测其master集群时会发现该副本。

    然而，orchestrator如何发现全新的拓扑？
    
    您可以要求orchestrator发现（探测）此类拓扑中的任何一台服务器，然后它将在整个拓扑中爬行。
    
    或者，您可以选择定期让orchestrator知道您拥有的任何一台生产服务器。在任何生产MySQL服务器上设置cronjob以读取：
    
        0 0 * * * root "/usr/bin/perl -le 'sleep rand 600' && /usr/bin/orchestrator-client -c discover -i this.hostname.com"
    
    在上面的例子中，每个主机每天让orchestrator了解自己一次；新启动的主机将在下一个午夜被发现。引入睡眠模式以避免所有服务器同时对orchestrator造成风暴。
        
    以上使用orchestrator-client(https://github.com/openark/orchestrator/blob/master/docs/orchestrator-client.md)，但如果在共享后端设置上运行，则可以使用orchestrator cli(https://github.com/openark/orchestrator/blob/master/docs/executing-via-command-line.md)。
    
### 添加提升规则

    有些服务器在发生故障切换时更适合升级。有些服务器不是很好的选择。示例：
    
    * 服务器的硬件配置较弱。你宁愿不promote升级它。
    * 服务器位于远程数据中心，您不想promote升级它。
    * 服务器用作备份源，并始终打开LVM快照。你不想promote升级它。
    * 服务器具有良好的设置，是理想的候选服务器。你更喜欢promote升级它。
    * 服务器是可以的，您没有任何特别的意见。

您将通过以下方式向orchestrator宣布您对给定服务器的首选项：
    
    orchestrator -c register-candidate -i ${::fqdn} --promotion-rule ${promotion_rule}

支持的promote升级规则包括： 
    
    * prefer
    * neutral
    * prefer_not
    * must_not

促销规则一小时后过期。这就是orchestrator的动态特性。您需要设置一个cron作业，该作业将宣布服务器的升级规则：

    */2 * * * * root "/usr/bin/perl -le 'sleep rand 10' && /usr/bin/orchestrator-client -c register-candidate -i this.hostname.com --promotion-rule prefer"

此设置来自生产环境。puppet会更新cron条目，以反映适当的升级规则。服务器此时可能有偏好，并且不希望在5分钟后出现。
集成您自己的服务发现方法和脚本，以提供最新的升级规则。

### 停机时间

当服务器出现问题时，它会：

    * 将显示在web界面上的"problems问题"下拉列表中。
    * 可以考虑恢复（例如：服务器已dead死亡，其所有副本现在都已损坏）。

您可以关闭服务器，以便：

    * 它不会显示在"problems"下拉列表中。
    * 将不考虑恢复。

停机正时通过以下方式进行：

    orchestrator-client -c begin-downtime -duration 30m -reason "testing" -owner myself

一些服务器可能会经常损坏；例如，自动恢复服务器；dev boxes开发箱；test boxes测试箱。对于这样的服务器，您可能需要连续停机。
实现这一点的一种方法是设置如此大的持续时间240000小时。但是，如果boxes盒子发生了变化，您需要记住结束停机时间。
继续采用动态方法，考虑：
    
    */2 * * * * root "/usr/bin/perl -le 'sleep rand 10' && /data/orchestrator/current/bin/orchestrator -c begin-downtime -i ${::fqdn} --duration=5m --owner=cron --reason=continuous_downtime"

每2分钟, 停机5分钟；这意味着当我们取消cronjob时，停机时间将在5分钟内到期。

上面显示的是orchestrator-client和orchestrator命令行界面的用法。
为完整起见，以下是如何通过直接API调用操作相同的操作：

    $ curl -s "http://my.orchestrator.service:80/api/begin-downtime/my.hostname/3306/wallace/experimenting+failover/45m"

orchestrator-client脚本运行这个API调用，将其包装并编码URL路径。如果您不想通过代理运行，它还可以自动检测leader。

### 伪GTID

如果您不使用GTID，您会很高兴知道orchestrator可以利用伪GTID实现与GTID类似的好处，例如关联两个不相关的服务器并从另一个服务器复制一个。
这意味着master和intermediate master中间主故障切换。

请阅读Pseudo GTID文档(https://github.com/openark/orchestrator/blob/master/docs/pseudo-gtid.md)页面上的更多内容。

orchestrator可以为您注入伪GTID条目。你的星团将神奇地拥有GTID般的超能力。关注自动伪GTID(https://github.com/openark/orchestrator/blob/master/docs/configuration-discovery-pseudo-gtid.md#automated-pseudo-gtid-injection)

### 填充元数据

orchestrator从服务器中提取一些元数据：

    * 此实例所属群集的别名是什么？
    * 服务器所属的数据中心是什么？
    * 此服务器上是否强制执行半同步？
    
这些详细信息通过以下查询提取：

    * DetectClusterAliasQuery
    * DetectClusterDomainQuery
    * DetectDataCenterQuery
    * DetectSemiSyncEnforcedQuery

或者通过作用于主机名的正则表达式：

    * DataCenterPattern
    * PhysicalEnvironmentPattern

查询可以通过将数据注入master服务器上的元数据表来满足。例如，您可以：

    CREATE TABLE IF NOT EXISTS cluster (
      anchor TINYINT NOT NULL,
      cluster_name VARCHAR(128) CHARSET ascii NOT NULL DEFAULT '',
      PRIMARY KEY (anchor)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
并使用例如1、my_cluster_name以及以下内容填充此表：

    {
      "DetectClusterAliasQuery": "select cluster_name from meta.cluster where anchor=1"
    }

请注意，orchestrator既不创建此类表，也不填充它们。您需要创建表，填充它们，并让orchestrator知道如何查询数据。

### 标记

orchestrator支持标记实例，以及按标记搜索实例。参见标签(https://github.com/openark/orchestrator/blob/master/docs/tags.md)



