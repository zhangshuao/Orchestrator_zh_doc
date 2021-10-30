# 配置：键-值 存储

orchestrator支持以下键值存储：

* 基于关系表的内部存储
* Consul(https://hub.fastgit.org/hashicorp/consul)
* ZooKeeper(https://zookeeper.apache.org/)

orchestrator通过将集群的主机存储在KV中来支持主机发现。

     "KVClusterMasterPrefix": "mysql/master",
      "ConsulAddress": "127.0.0.1:8500",
      "ZkAddress": "srv-a,srv-b:12181,srv-c",
      "ConsulCrossDataCenterDistribution": true,

KVClusterMasterPrefix是用于主发现条目的前缀。例如，您的群集别名为mycluster，master主机为some.host-17.com，则您需要一个条目，其中：

* Key是mysql/master/mycluster
* Value是host-17.com:3306

注意：在ZooKeeper上，键将自动加上前缀 / 如果还没有。

#### 分类条目

除上述内容外，orchestrator还分解master条目并添加以下内容（通过上面的示例进行说明）：

* mysql/master/mycluster/hostname, 值是some.host-17.com
* mysql/master/mycluster/port, 值是3306 
* mysql/master/mycluster/ipv4, 值是192.168.0.1
* mysql/master/mycluster/ipv6, 值是<whatever>

将自动为任何主条目添加 /hostname、/port、/ipv4 和 /ipv6扩展。

### Stores存储

如果指定，则ConsulAddress表示可使用Consul HTTP服务的地址。如果未指定，则不会尝试Consul访问。

如果指定，ZKDAddress表示要连接到的一个或多个ZooKeeper服务器。每台服务器的默认端口为2181。以下各项都是等效的：

* srv-a,srv-b:12181,srv-c
* srv-a,srv-b:12181,srv-c:2181
* srv-a:2181,srv-b:12181,srv-c:2181

### Consul 专用

有关Consul的具体设置，请参见kv文档(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/kv.md)。
