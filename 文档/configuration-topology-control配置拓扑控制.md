# 配置：拓扑控制

以下配置影响orchestrator将更改应用于拓扑服务器的方式：

orchestrator将计算出集群、数据中心等的名称。

    {
      "UseSuperReadOnly": false,
    }

#### UseSuperReadOnly

默认情况下为false。如果为true，则每当要求orchestrator设置/清除 read_only时，它也会将更改应用于super_read_only。
从特定版本起，super_read_only在Oracle MySQL和Percona Server上可用。    
    
