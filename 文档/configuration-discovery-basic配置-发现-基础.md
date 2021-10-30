# 配置：基本发现

让orchestrator知道如何查询MySQL拓扑，提取哪些信息。

    {
      "MySQLTopologyCredentialsConfigFile": "/etc/mysql/orchestrator-topology.cnf",
      "InstancePollSeconds": 5,
      "DiscoverByShowSlaveHosts": false,
    }

MySQLTopologyCredentialsConfigFile遵循与MySQLOrchestratorCredentialsConfigFile类似的规则。您可以选择使用纯文本凭据：

[client]
user=orchestrator
password=orc_topology_password

或者，您可以选择使用纯文本凭据：

{
  "MySQLTopologyUser": "orchestrator",
  "MySQLTopologyPassword": "orc_topology_password",
}

orchestrator将每InstancePollSeconds探测每台服务器一次。

在所有MySQL拓扑上，授予以下权限：

    CREATE USER 'orchestrator'@'orc_host' IDENTIFIED BY 'orc_topology_password';
    GRANT SUPER, PROCESS, REPLICATION SLAVE, REPLICATION CLIENT, RELOAD ON *.* TO 'orchestrator'@'orc_host';
    GRANT SELECT ON meta.* TO 'orchestrator'@'orc_host';
    GRANT SELECT ON ndbinfo.processes TO 'orchestrator'@'orc_host'; -- Only for NDB Cluster
    GRANT SELECT ON performance_schema.replication_group_members TO 'orchestrator'@'orc_host'; -- Only for Group Replication / InnoDB cluster

