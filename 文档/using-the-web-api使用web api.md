# 使用web API

orchestrator提供了一个复杂的web API。

热衷于web开发的人会注意到（通过Firebug或开发工具）web界面是如何完全依赖JSON API请求的。

### 开发人员可以使用API实现自动化。

*简单介绍几个API命令*

举例来说:

    * /api/instance/:host/:port: 读取并返回实例的详细信息（例如 /api/instance/mysql10/3306）
    * /api/discover/:host/:port: discover给定实例（运行的orchestrator服务将从那里获取该实例并递归扫描整个拓扑）
    * /api/relocate/:host/:port/:belowHost/:belowPort（尝试）将一个实例移动到另一个实例下面。orchestrator选择最佳的行动方案。
    * /api/relocate-replicas/:host/:port/:belowHost/:belowPort: （尝试）将一个实例的副本移动到另一个实例下面。orchestrator选择最佳的行动方案。
    * /api/recover/:host/:post：在给定实例上启动恢复，假设存在要恢复的内容。
    * /api/force-master-failover/:mycluster：强制在给定集群上立即进行故障切换。

### 完整列表

    事实上的清单是代码，请参阅api.go(https://github.hub.com/openark/orchestrator/blob/master/go/http/api.go)（向下滚动到RegisterRequests）。
    
    您可能还希望查看orchestrator-client （源代码https://github.hub.com/openark/orchestrator/blob/master/resources/bin/orchestrator-client），了解命令行界面如何转换为API调用。
    
    或者，只需使用orchestrator-client作为API客户端，这就是它的用途。
    
### 实例JSON故障损坏

    许多API调用返回实例对象，描述单个MySQL服务器。此示例之后是字段细分：
    
    {
        "Key": {
            "Hostname": "mysql.02.instance.com",
            "Port": 3306
        },
        "Uptime": 45,
        "ServerID": 101,
        "Version": "5.6.22-log",
        "ReadOnly": false,
        "Binlog_format": "ROW",
        "LogBinEnabled": true,
        "LogReplicationUpdatesEnabled": true,
        "SelfBinlogCoordinates": {
            "LogFile": "mysql-bin.015656",
            "LogPos": 15082,
            "Type": 0
        },
        "MasterKey": {
            "Hostname": "mysql.01.instance.com",
            "Port": 3306
        },
        "ReplicationSQLThreadRuning": true,
        "ReplicationIOThreadRuning": true,
        "HasReplicationFilters": false,
        "SupportsOracleGTID": true,
        "UsingOracleGTID": true,
        "UsingMariaDBGTID": false,
        "UsingPseudoGTID": false,
        "ReadBinlogCoordinates": {
            "LogFile": "mysql-bin.015993",
            "LogPos": 20146,
            "Type": 0
        },
        "ExecBinlogCoordinates": {
            "LogFile": "mysql-bin.015993",
            "LogPos": 20146,
            "Type": 0
        },
        "RelaylogCoordinates": {
            "LogFile": "mysql_sandbox21088-relay-bin.000051",
            "LogPos": 16769,
            "Type": 1
        },
        "LastSQLError": "",
        "LastIOError": "",
        "SecondsBehindMaster": {
            "Int64": 0,
            "Valid": true
        },
        "SQLDelay": 0,
        "ExecutedGtidSet": "230ea8ea-81e3-11e4-972a-e25ec4bd140a:1-49",
        "ReplicationLagSeconds": {
            "Int64": 0,
            "Valid": true
        },
        "Replicas": [ ],
        "ClusterName": "mysql.01.instance.com:3306",
        "DataCenter": "",
        "PhysicalEnvironment": "",
        "ReplicationDepth": 1,
        "IsCoMaster": false,
        "IsLastCheckValid": true,
        "IsUpToDate": true,
        "IsRecentlyChecked": true,
        "SecondsSinceLastSeen": {
            "Int64": 9,
            "Valid": true
        },
        "CountMySQLSnapshots": 0,
        "IsCandidate": false,
        "UnresolvedHostname": ""
    }
    
    实例的结构不断发展，文档总是落后。话虽如此，关键属性是：
    Key: 实例的唯一指示符：主机和端口的组合
    ServerID: MySQL server_id参数
    Version: MySQL版本
    ReadOnly: 全局 read_only 布尔值
    Binlog_format: 全局 binlog_format MySQL参数
    LogBinEnabled: 是否启用二进制日志
    LogReplicationUpdatesEnabled: 是否启用 log_slave_updates MySQL参数
    SelfBinlogCoordinates: 二进制日志文件&定位此实例的写入位置（如SHOW MASTER STATUS）
    MasterKey: master的主机名和端口（如果有） 
    ReplicationSQLThreadRuning: 直接显示SHOW SLAVE STATUS's Slave_SQL_Running
    ReplicationIOThreadRuning: 直接显示 SHOW SLAVE STATUS's Slave_IO_Running
    HasReplicationFilters: 如果存在任何复制筛选器，则为true
    SupportsOracleGTID: 如果配置为gtid_mode（Oracle MySQL>=5.6），则为true (Oracle MySQL >= 5.6)
    UsingOracleGTID: 如果通过Oracle GTID复制副本，则为true
    UsingMariaDBGTID: 如果通过MariaDB GTID复制副本，则为true 
    UsingPseudoGTID: 如果已知复制副本具有Pseudo-GTID坐标，则为true（请参阅相关的DetectPseudoGTIDQuery配置） true if replica known to have Pseudo-GTID coordinates (see related DetectPseudoGTIDQuery config)
    ReadBinlogCoordinates: (复制时) 从master读取的坐标 (IO_THREAD 轮询的内容)
    ExecBinlogCoordinates: (复制时) 当前正在执行的master坐标（SQL_线程执行的内容） (SQL_THREAD 执行的内容)
    RelaylogCoordinates: (复制时) 当前正在执行的中继日志坐标
    LastSQLError: 显示 SHOW SLAVE STATUS
    LastIOError: 显示 SHOW SLAVE STATUS
    SecondsBehindMaster:  SHOW SLAVE STATUS的秒数映射到Seconds_Behind_Master "有效"：false表示空值 direct mapping from SHOW SLAVE STATUS's Seconds_Behind_Master "Valid": false indicates a NULL
    SQLDelay: 配置 MASTER_DELAY
    ExecutedGtidSet: 如果使用Oracle GTID，则执行的GTID集
    ReplicationLagSeconds: 提供ReplicationLagQuery时，计算的副本延迟；否则与SecondsBehindMaster相同
    Replicas: MySQL副本列表（主机名和端口）
    ClusterName: 与此实例关联的集群的名称；唯一标识群集
    DataCenter: （元数据）数据中心的名称，由DataCenterPattern配置变量推断
    PhysicalEnvironment:（元数据）环境的名称，由PhysicalEnvironmentPattern配置变量推断
    ReplicationDepth: 与master的距离（master为0，直接replica副本为1，依此类推） 
    IsCoMaster: 当此实例是master-master一部分时为true
    IsLastCheckValid: 上次读取此实例的尝试是否成功
    IsUpToDate: 此数据是否为最新数据
    IsRecentlyChecked: 最近是否对此实例进行了读取尝试
    SecondsSinceLastSeen: 自上次成功访问此实例以来经过的时间
    CountMySQLSnapshots: 已知快照数（由orchestrator-agent提供的数据）
    IsCandidate: (元数据) 如果此实例已通过register-candidate CLI命令标记为候选实例，则为true。可用于故障恢复，以确定故障切换选项的优先级
    UnresolvedHostname: 将此主机命名为unsolves to，如register-hostname-unsolve CLI命令所示

### 备忘单

以下是一些API使用的有用示例：
    
    * 获取有关群集的常规信息：
    curl -s "http://my.orchestrator.service.com/api/cluster-info/my_cluster" | jq .
    {
      "ClusterName": "my-cluster-fqdn:3306",
      "ClusterAlias": "my_cluster",
      "ClusterDomain": "my-cluster.com",
      "CountInstances": 10,
      "HeuristicLag": 0,
      "HasAutomatedMasterRecovery": true,
      "HasAutomatedIntermediateMasterRecovery": true
    }
    
    * 查找my_cluster中没有二进制日志记录的主机：
    curl -s "http://my.orchestrator.service.com/api/cluster/alias/my_cluster" | jq '.[] | select(.LogBinEnabled==false) .Key.Hostname' -r

    * 查找my_cluster master服务器的直接副本：
    curl -s "http://my.orchestrator.service.com/api/cluster/alias/my_cluster" | jq '.[] | select(.ReplicationDepth==1) .Key.Hostname' -r
    或:
    master=$(curl -s "http://my.orchestrator.service.com/api/cluster-info/my_cluster" | jq '.ClusterName' | tr ':' '/')
    curl -s "http://my.orchestrator.service.com/api/instance-replicas/${master}" | jq '.[] | .Key.Hostname' -r

    * 查找my_cluster中的所有中间主机
    curl -s "http://my.orchestrator.service.com/api/cluster/alias/my_cluster" | jq '.[] | select(.MasterKey.Hostname!="") | select(.Replicas!=[]) .Key.Hostname'
