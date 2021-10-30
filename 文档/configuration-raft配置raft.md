# 配置: raft

设置orchestrator/raft（https://hub.fastgit.org/openark/orchestrator/blob/master/docs/raft.md）群集以实现高可用性。

假设您将在3节点设置上运行orchestrator/raft，您将在每个节点上配置它：

    "RaftEnabled": true,
    "RaftDataDir": "<path.to.orchestrator.data.directory>",
    "RaftBind": "<ip.or.fqdn.of.this.orchestrator.node>",
    "DefaultRaftPort": 10008,
    "RaftNodes": [
    "<ip.or.fqdn.of.orchestrator.node1>",
    "<ip.or.fqdn.of.orchestrator.node2>",
    "<ip.or.fqdn.of.orchestrator.node3>"
    ],
 
一些细分：

* RaftEnabled必须设置为true，否则orchestrator将以shared-backend共享后端模式运行。
* RaftDataDir必须设置为可写入orchestrator的目录。如果目录不存在，orchestrator将尝试创建该目录。
* 必须设置RaftBind，请使用本地主机的IP地址或完整主机名。此IP或主机名也将作为RaftNodes变量之一列出。
* DefaultRaftPort可以设置为任何端口，但必须在所有部署中保持一致。
* raft节点应列出raft集群的所有节点。此列表将由IP地址或主机名组成，并将包括RaftBind中显示的此主机本身的值。

例如，以下可能是工作设置：

    "RaftEnabled": true,
    "RaftDataDir": "/var/lib/orchestrator",
    "RaftBind": "10.0.0.2",
    "DefaultRaftPort": 10008,
    "RaftNodes": [
    "10.0.0.1",
    "10.0.0.2",
    "10.0.0.3"
    ],
  
除此之外：

    "RaftEnabled": true,
    "RaftDataDir": "/var/lib/orchestrator",
    "RaftBind": "node-full-hostname-2.here.com",
    "DefaultRaftPort": 10008,
    "RaftNodes": [
    "node-full-hostname-1.here.com",
    "node-full-hostname-2.here.com",
    "node-full-hostname-3.here.com"
    ],
  
#### NAT、防火墙、路由

如果您的orchestrator/raft节点需要通过NAT网关进行通信，您还可以设置:

    * "RaftAdvertise": "<ip.or.fqdn.visible.to.other.nodes>"

到其他节点应联系的IP或主机名。否则，其他节点将尝试与"RaftBind"地址通信并失败。

Raft节点将向leader反向代理HTTP请求。orchestrator将尝试试探性地计算领导者的URL，将请求重定向到该URL。
若在NAT、重新路由端口等之后，orchestrator可能无法计算该URL。您可以配置：

    * "HTTPAdvertise": "scheme://hostname:port"

明确指定通过HTTP API访问节点的位置（假设该节点是前导节点）。例如，您可以："HTTPAdvertise": "http://my.public.hostname:3000"

#### Backend DB

raft设置支持MySQL或SQLite后端数据库。请参阅后端(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/configuration-backend.md)配置以了解其中一个。请阅读"高可用性(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/high-availability.md)"页面，了解使用这两种方法的场景、可能性和原因。

#### 单raft节点设置

在生产环境中，您需要使用多个raft节点，例如3或5。

在测试环境中，您可以运行由单个节点组成的orchestrator/raft设置。此节点将隐式地成为领导者，并将向自身播发raft消息。

要运行单节点orchestrator/raft，请配置空的RaftNodes：

    "RaftNodes": [],

或者，指定一个与RaftBind或RaftAdvertise相同的节点：
    
    "RaftEnabled": true,
    "RaftBind": "127.0.0.1",
    "DefaultRaftPort": 10008,
    "RaftNodes": [
    "127.0.0.1"
    ],   
    
    