# 键-值 存储

orchestrator支持以下键值存储：

* 基于关系表的内部存储
* Consul(https://github.com/hashicorp/consul)
* ZooKeeper(https://zookeeper.apache.org/)

另请参见键-值配置。(https://github.com/openark/orchestrator/blob/master/docs/configuration-kv.md)

### 该时间键-值（aka KV）存储用于：

    * master发现
    
### master发现、键-值 和 failover故障转移  aaaaaaaa
        
    其目标是，基于Concul或ZookKeeper的服务发现将能够服务于master发现和/或根据集群的master标识和更改采取行动。
    
    最常见的场景是更新代理以将集群的写流量定向到特定的master。例如，可以通过consul-template设置HAProxy，以便consul-template基于orchestrator编写的键值存储填充single-host master池。
    
    orchestrator在master故障切换时更新所有KV存储。
    
    * 填充主条目
    
    群集的master条目填充在：
    
        * 遇到新群集，或遇到没有现有KV条目的master群集。此检查自动并定期运行。
    
            * 定期检查首先咨询协调人的内部KV存储。仅当内部存储尚未包含主条目时，才会尝试填充外部存储（Consul、Zookeeper）。因此，定期检查只会注入外部KV一次。
    
        * 实际故障转移：orchestrator使用新master服务器的标识覆盖现有条目
    
        * 手动输入人口请求：
            * orchestrator-client -c submit-masters-to-kv-stores，以向KV提交所有集群的主机，或
            * orchestrator-client -c submit-masters-to-kv-stores -alias mycluster 向kv提交mycluster主机
    
            请参阅orchestrator-client.md(https://github.com/openark/orchestrator/blob/master/docs/orchestrator-client)文档。您也可以使用orchestrator命令行调用。
    
    或者，您可以通过以下方式直接访问API：
    /api/submit-masters-to-kv-stores，或
    /api/submit-masters-to-kv-stores/:alias，或
    
    分别地
    
    实际故障切换和手动请求都将覆盖任何现有的内部和外部KV条目。

### KV 和 orchestrator/raft

在orchestrator/raft设置中，所有KV写入都通过raft协议。因此，一旦leader确定需要对KV存储进行写入，它就会将请求发布到所有raft节点。
每个节点将根据自己的配置独立应用写操作。

Implications启示

举例来说，假设您在3个数据中心设置中运行orchestrator/raft，每个DC一个节点。另外，假设您在每个DC上都设置了Consul。Consul的设置通常是跨DC的，可能是跨DC的异步复制。

master节点故障切换后，每个orchestrator节点都将使用新的master节点标识更新Consul。

如果您的Consul运行跨DC复制，则同一KV更新可能运行两次：一次通过Consul复制，一次通过本地orchestrator节点。
这两个更新相同且一致，因此可以安全运行。

如果您的Consul设置没有相互复制，orchestrator是使主发现在Consul群集之间保持一致的唯一方法。
您可以获得raft附带的所有优秀特性：如果一个DC是网络分区的，那么该DC中的orchestrator节点将不会接收到KV更新事件，并且在一段时间内，Consul集群也不会。
然而，一旦重新获得网络访问权，orchestrator成员将及时了解事件日志，并将KV更新应用于本地Consul集群。设置最终是一致的。

主故障切换后不久，orchestrator将生成raft快照。这并不是严格要求的，但却是一个有用的操作：在orchestrator节点重新启动时，快照会阻止orchestrator重放KV写入。
在故障切换和failover-and-failback回切事件中，这一点尤其有趣，在这种情况下，远程KV像consul一样，可能会为同一集群获得两个更新。
快照可以缓解此类事件。

### orchestrator 专用

或者，您可以配置：

    "ConsulCrossDataCenterDistribution": true,

…除了上述流程之外，还可以（并且将）进行。

使用ConcursCrossDatacenterDistribution，orchestrator会对扩展的Concul群集列表运行额外的定期更新。

orchestrator leader节点每分钟查询一次其配置的orchestrator服务器，以获取已知数据中心的列表。(https://www.consul.io/api/catalog.html#list-datacenters)
然后，它遍历这些数据中心集群，并使用master节点的当前身份更新每个集群。

如果每个orchestrator节点的领事数据中心多于一个本地领事，则需要此功能。
我们在上面演示了在orchestrator/raft设置中，每个节点如何更新其本地Consul集群。
但是，对于任何orchestrator节点来说都不是本地的Consul集群不受该方法的影响。
ConsultCrossDatacenterDistribution是包含所有其他DC的方式。

#### Consul 事务支持

通过配置以下各项启用原子Consul事务支持(https://www.consul.io/api-docs/txn)：

    "ConsulKVStoreProvider": "consul-txn",

注意：此功能需要Consul版本0.7或更高版本。

这会导致Orchestrator在分发一个或多个Concul KVs时使用Concul事务。
在一个事务中从服务器读取KVs，在第二个事务中执行任何必要的更新。

Orchestrator按键前缀将KV更新分组为5到64个操作组（默认为5个）。
此分组确保以原子方式更新单个集群（5 x KVs）。将ConsulMaxKVsPerTransaction配置设置从5（默认）增加到最多64（Consul事务API限制），
允许将更多操作分组到更少的事务中，但更多操作可能会同时失败。
