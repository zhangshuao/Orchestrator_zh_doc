# 脚本示例

本文档介绍有关orchestrator的脚本用法和想法。

### 显示具有别名的所有群集

    $ orchestrator-client -c clusters-alias
    mysql-9766.dc1.domain.net:3306,cl1
    mysql-0909.dc1.domain.net:3306,olap
    mysql-0246.dc1.domain.net:3306,mycluster
    mysql-1111.dc1.domain.net:3306,oltp1
    mysql-9002.dc1.domain.net:3306,oltp2
    mysql-3972.dc1.domain.net:3306,oltp3
    mysql-0019.dc1.domain.net:3306,oltp4

### 仅显示别名

    $ orchestrator-client -c clusters-alias | cut -d"," -f2 | sort
    cl1
    mycluster
    olap
    oltp1
    oltp2
    oltp3
    oltp4

### 集群master

    $ orchestrator-client -c which-cluster-master -alias mycluster
    mysql-0246.dc1.domain.net:3306

### 群集的所有实例

    $ orchestrator-client -c which-cluster-instances -alias mycluster
    mysql-0246.dc1.domain.net:3306
    mysql-1357.dc2.domain.net:3306
    mysql-bb00.dc1.domain.net:3306
    mysql-00ff.dc1.domain.net:3306
    mysql-8181.dc2.domain.net:3306
    mysql-2222.dc1.domain.net:3306
    mysql-ecec.dc2.domain.net:3306

    上面指出了orchestrator对复制图的了解。该列表包括可能offline/broken脱机/损坏的服务器。
    
### Shell循环实例

    $ orchestrator-client -c which-cluster-instances -alias mycluster | cut -d":" -f 1 | while read h ; do echo "Host is $h" ; done
    Host is mysql-0246.dc1.domain.net
    Host is mysql-1357.dc2.domain.net
    Host is mysql-bb00.dc1.domain.net
    Host is mysql-00ff.dc1.domain.net
    Host is mysql-8181.dc2.domain.net
    Host is mysql-2222.dc1.domain.net
    Host is mysql-ecec.dc2.domain.net

### 在群集上禁用半同步

    $ orchestrator-client -c which-cluster-instances -alias mycluster | while read i ; do
      orchestrator-client -c disable-semi-sync-master -i $i
    done
    mysql-0246.dc1.domain.net:3306
    mysql-1357.dc2.domain.net:3306
    mysql-bb00.dc1.domain.net:3306
    mysql-00ff.dc1.domain.net:3306
    mysql-8181.dc2.domain.net:3306
    mysql-2222.dc1.domain.net:3306
    mysql-ecec.dc2.domain.net:3306

### 在群集主机上启用半同步

    $ orchestrator-client -c which-cluster-master -alias mycluster | while read i ; do
      orchestrator-client -c enable-semi-sync-master -i $i
    done
    mysql-0246.dc1.domain.net:3306


### 让我们再试一次。这一次，在除主机外的所有实例上禁用半同步

    $ master=$(orchestrator-client -c which-cluster-master -alias mycluster)
    $ orchestrator-client -c which-cluster-instances -alias mycluster | grep -v $master | while read i ; do
      orchestrator-client -c disable-semi-sync-master -i $i
    done

### 同样，在所有副本上设置read-only只读

    $ orchestrator-client -c which-cluster-instances -alias mycluster | grep -v $master | while read i ; do
      orchestrator-client -c set-read-only -i $i
    done

### 我们真的不需要循环。我们可以使用ccql

    ccql(https://github/github/ccql)是一个并发的多服务器MySQL客户端。一般来说，它可以很好地使用脚本，特别是orchestrator。

    $ orchestrator-client -c which-cluster-instances -alias mycluster | grep -v $master | ccql -C ~/.my.cnf -q "set global read_only=1"

### 提取master主机名（no："3306"）

    $ master_host=$(orchestrator-client -c which-cluster-master -alias mycluster | cut -d":" -f1)
    $ echo $master_host
    mysql-0246.dc1.domain.net

    下面我们将使用master_host变量。
    
### 使用API显示特定主机的所有数据

    $ orchestrator-client -c api -path instance/$master_host/3306 | jq .
    {
      "Key": {
        "Hostname": "mysql-0246.dc1.domain.net",
        "Port": 3306
      },
      "InstanceAlias": "",
      "Uptime": 12203,
      "ServerID": 65884260,
      "ServerUUID": "3e87bd92-2be0-13e8-ac1b-008cda544064",
      "Version": "5.7.18-log",
      "VersionComment": "MySQL Community Server (GPL)",
      "FlavorName": "MySQL",
      "ReadOnly": false,
      "Binlog_format": "ROW",
      "BinlogRowImage": "FULL",
      "LogBinEnabled": true,
      "LogReplicationUpdatesEnabled": true,
      "SelfBinlogCoordinates": {
        "LogFile": "mysql-bin.000002",
        "LogPos": 333006336,
        "Type": 0
      },
      "MasterKey": {
        "Hostname": "",
        "Port": 0
      },
      "IsDetachedMaster": false,
      "ReplicationSQLThreadRuning": false,
      "ReplicationIOThreadRuning": false,
      "HasReplicationFilters": false,
      "GTIDMode": "OFF",
      "SupportsOracleGTID": false,
      "UsingOracleGTID": false,
      "UsingMariaDBGTID": false,
      "UsingPseudoGTID": true,
      "ReadBinlogCoordinates": {
        "LogFile": "",
        "LogPos": 0,
        "Type": 0
      },
      "ExecBinlogCoordinates": {
        "LogFile": "",
        "LogPos": 0,
        "Type": 0
      },
      "IsDetached": false,
      "RelaylogCoordinates": {
        "LogFile": "",
        "LogPos": 0,
        "Type": 1
      },
      "LastSQLError": "",
      "LastIOError": "",
      "SecondsBehindMaster": {
        "Int64": 0,
        "Valid": false
      },
      "SQLDelay": 0,
      "ExecutedGtidSet": "",
      "GtidPurged": "",
      "ReplicationLagSeconds": {
        "Int64": 0,
        "Valid": true
      },
      "Replicas": [
        {
          "Hostname": "mysql-2222.dc1.domain.net",
          "Port": 3306
        },
        {
          "Hostname": "mysql-00ff.dc1.domain.net",
          "Port": 3306
        },
        {
          "Hostname": "mysql-1357.dc2.domain.net",
          "Port": 3306
        }
      ],
      "ClusterName": "mysql-0246.dc1.domain.net:3306",
      "SuggestedClusterAlias": "mycluster",
      "DataCenter": "dc1",
      "PhysicalEnvironment": "",
      "ReplicationDepth": 0,
      "IsCoMaster": false,
      "HasReplicationCredentials": false,
      "ReplicationCredentialsAvailable": false,
      "SemiSyncEnforced": false,
      "SemiSyncMasterEnabled": true,
      "SemiSyncReplicaEnabled": false,
      "LastSeenTimestamp": "2018-03-21 04:40:38",
      "IsLastCheckValid": true,
      "IsUpToDate": true,
      "IsRecentlyChecked": true,
      "SecondsSinceLastSeen": {
        "Int64": 2,
        "Valid": true
      },
      "CountMySQLSnapshots": 0,
      "IsCandidate": false,
      "PromotionRule": "neutral",
      "IsDowntimed": false,
      "DowntimeReason": "",
      "DowntimeOwner": "",
      "DowntimeEndTimestamp": "",
      "ElapsedDowntime": 0,
      "UnresolvedHostname": "",
      "AllowTLS": false,
      "LastDiscoveryLatency": 7233416
    }

### 从JSON中提取主机名：

    $ orchestrator-client -c api -path instance/$master_host/3306 | jq .Key.Hostname -r
    mysql-0246.dc1.domain.net

### 从JSON中提取master主机名:

    $ orchestrator-client -c api -path instance/$master_host/3306 | jq .MasterKey.Hostname -r
    (empty, this is the master)

### 列出集群中所有主机名的另一种方法：使用API和jq
    
    $ orchestrator-client -c api -path cluster/alias/mycluster | jq .[].Key.Hostname -r
    mysql-0246.dc1.domain.net
    mysql-1357.dc2.domain.net
    mysql-bb00.dc1.domain.net
    mysql-00ff.dc1.domain.net
    mysql-8181.dc2.domain.net
    mysql-2222.dc1.domain.net
    mysql-ecec.dc2.domain.net

### 显示群集中每个成员的主主机：

    $ orchestrator-client -c api -path cluster/alias/mycluster | jq .[].MasterKey.Hostname -r
    
    mysql-0246.dc1.domain.net
    mysql-00ff.dc1.domain.net
    mysql-0246.dc1.domain.net
    mysql-bb00.dc1.domain.net
    mysql-0246.dc1.domain.net
    mysql-bb00.dc1.domain.net

### 特定实例的主主机名是什么？

    $ orchestrator-client -c api -path instance/mysql-bb00.dc1.domain.net/3306 | jq .MasterKey.Hostname -r
    mysql-00ff.dc1.domain.net

### 一个特定实例有多少个副本？

    $ orchestrator-client -c api -path instance/$master_host/3306 | jq '.Replicas | length'
    3

### 每个集群成员有多少个副本？

    $ orchestrator-client -c api -path cluster/alias/mycluster | jq '.[].Replicas | length'
    3
    0
    2
    1
    0
    0
    0


### 列出所有副本的另一种方法

    我们过滤掉那些没有show slave status的输出：
    
    $ orchestrator-client -c which-cluster-instances -alias mycluster | ccql -C ~/.my.cnf -q "show slave status" | awk '{print $1}'
    mysql-00ff.dc1.domain.net:3306
    mysql-bb00.dc1.domain.net:3306
    mysql-2222.dc1.domain.net:3306
    mysql-ecec.dc2.domain.net:3306
    mysql-1357.dc2.domain.net:3306
    mysql-8181.dc2.domain.net:3306

### 接下来，在所有群集实例上重新启动复制

    $ orchestrator-client -c which-cluster-instances -alias mycluster | ccql -C ~/.my.cnf -q "show slave status" | awk '{print $1}' | ccql -C ~/.my.cnf -q "stop slave; start slave;"

### 我希望对复制应用更改，而不更改复制副本的状态（如果它正在运行，我希望它保持运行。如果它没有运行，我不希望启动复制）

    $ orchestrator-client -c restart-replica-statements -i mysql-bb00.dc1.domain.net -query "change master to auto_position=1" | jq .[] -r
    stop slave io_thread;
    stop slave sql_thread;
    change master to auto_position=1;
    start slave sql_thread;
    start slave io_thread;
    
与之相比：

    $ orchestrator-client -c stop-replica -i mysql-bb00.dc1.domain.net
    mysql-bb00.dc1.domain.net:3306

    $ orchestrator-client -c restart-replica-statements -i mysql-bb00.dc1.domain.net -query "change master to auto_position=1" | jq .[] -r
    change master to auto_position=1;

以上只是输出语句，我们需要将它们推回服务器：
    
    orchestrator-client -c restart-replica-statements -i mysql-bb00.dc1.domain.net -query "change master to auto_position=1" | jq .[] -r | mysql -h mysql-bb00.dc1.domain.net

### 哪个DC（数据中心）是特定实例？

本问题和下一个问题假设已配置DetectDataCenterQuery或DataCenterPattern。

    $ orchestrator-client -c api -path instance/mysql-bb00.dc1.domain.net/3306 | jq '.DataCenter'
    dc1

### 在哪个DC中部署了群集，每个DC中有多少台主机？

    $ orchestrator-client -c api -path cluster/mycluster | jq '.[].DataCenter' -r | sort | uniq -c
      4 dc1
      3 dc2
  
### 哪些复制副本正在跨DC复制？

    $ orchestrator-client -c api -path cluster/mycluster |
        jq '.[] | select(.MasterKey.Hostname != "") |
            (.Key.Hostname + ":" + (.Key.Port | tostring) + " " + .DataCenter + " " + .MasterKey.Hostname + "/" + (.MasterKey.Port | tostring))' -r |
        while read h dc m ; do
          orchestrator-client -c api -path "instance/$m" | jq '.DataCenter' -r |
            { read master_dc ; [ "$master_dc" != "$dc" ] && echo $h ; } ;
        done
    
    mysql-bb00.dc1.domain.net:3306
    mysql-8181.dc2.domain.net:3306
