# 使用Orchestrator的第一步

您已安装、部署和配置Orchestrator。你能用它做什么？
常见命令的演练，主要在CLI端

#### Must必须
#### Discover发现

    您需要发现您的MySQL主机。或者浏览你的http://orchestrator:3000/web/discover页面并提交实例以进行查找，或：
    $ orchestrator-client -c discover -i some.mysql.instance.com:3306
    
    不需要：3306，因为DefaultInstancePort配置为3306。你亦可：
    $ orchestrator-client -c discover -i some.mysql.instance.com
    
    这将发现单个实例。但是：您是否也在运行orchestrator服务？它将从那里接收并询问该实例的主副本和副本，递归地继续，直到显示整个拓扑。
    
#### 信息

    我们现在假设您拥有orchestrator已知的拓扑（您已经发现了它）。假设some.mysql.instance.com属于一种拓扑。
    a.replica.3.instance.com属于另一个。你可以问以下问题：
    
    $ orchestrator-client -c clusters
    topology1.master.instance.com:3306
    topology2.master.instance.com:3306
    
    $ orchestrator-client -c which-master -i some.mysql.instance.com
    some.master.instance.com:3306
    
    $ orchestrator-client -c which-replicas -i some.mysql.instance.com
    a.replica.instance.com:3306
    another.replica.instance.com:3306
    
    $ orchestrator-client -c which-cluster -i a.replica.3.instance.com
    topology2.master.instance.com:3306
    
    $ orchestrator-client -c which-cluster-instances -i a.replica.3.instance.com
    topology2.master.instance.com:3306
    a.replica.1.instance.com:3306
    a.replica.2.instance.com:3306
    a.replica.3.instance.com:3306
    a.replica.4.instance.com:3306
    a.replica.5.instance.com:3306
    a.replica.6.instance.com:3306
    a.replica.7.instance.com:3306
    a.replica.8.instance.com:3306
    
    $ orchestrator-client -c topology -i a.replica.3.instance.com
    topology2.master.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.1.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.2.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
      + a.replica.3.instance.com:3306 [OK,5.6.17-log,STATEMENT]
      + a.replica.4.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
      + a.replica.5.instance.com:3306 [OK,5.6.17-log,STATEMENT]
    + a.replica.6.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
      + a.replica.7.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.8.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]

#### 移动东西

    您可以使用各种命令移动服务器。通用的"自动解决问题"命令是relocate和relocate-replicas：
    
    # Move a.replica.3.instance.com to replicate from a.replica.4.instance.com

    $ orchestrator-client -c relocate -i a.replica.3.instance.com:3306 -d a.replica.4.instance.com
    a.replica.3.instance.com:3306<a.replica.4.instance.com:3306
    
    $ orchestrator-client -c topology -i a.replica.3.instance.com
    topology2.master.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.1.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.2.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
      + a.replica.4.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
        + a.replica.3.instance.com:3306 [OK,5.6.17-log,STATEMENT]
      + a.replica.5.instance.com:3306 [OK,5.6.17-log,STATEMENT]
    + a.replica.6.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
      + a.replica.7.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.8.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    
    # Move the replicas of a.replica.2.instance.com to replicate from a.replica.6.instance.com
    
    $ orchestrator-client -c relocate-replicas -i a.replica.2.instance.com:3306 -d a.replica.6.instance.com
    a.replica.4.instance.com:3306
    a.replica.5.instance.com:3306
    
    $ orchestrator-client -c topology -i a.replica.3.instance.com
    topology2.master.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.1.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.2.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.6.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
      + a.replica.4.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
        + a.replica.3.instance.com:3306 [OK,5.6.17-log,STATEMENT]
      + a.replica.5.instance.com:3306 [OK,5.6.17-log,STATEMENT]
      + a.replica.7.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.8.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]

    relocate 和 relocate-replicas自动确定如何重新定位复制副本。也许通过GTID；可能是普通的binlog file:pos.或者可能有伪GTID，或者是否涉及binlog服务器？还支持其他变体。

    如果您想拥有更大的控制权：
    
    普通file:pos操作通过move-up, move-below完成
    伪GTID特定副本重定位、使用match, match-replicas, regroup-replicas。
    Binlog服务器操作通常通过repoint、repoint-replicas完成

#### 复制控制

    您很容易就能看到以下操作:
    $ orchestrator-client -c stop-replica -i a.replica.8.instance.com
    $ orchestrator-client -c start-replica -i a.replica.8.instance.com
    $ orchestrator-client -c restart-replica -i a.replica.8.instance.com
    $ orchestrator-client -c set-read-only -i a.replica.8.instance.com
    $ orchestrator-client -c set-writeable -i a.replica.8.instance.com
    
    通过干扰复制副本的主master中断复制：
    $ orchestrator-client -c detach-replica -i a.replica.8.instance.com

    别担心，这是可逆的：
    $ orchestrator-client -c reattach-replica -i a.replica.8.instance.com
    
#### Crash分析与恢复

    你的集群健康吗？
    
    $ orchestrator-client -c replication-analysis
    some.master.instance.com:3306 (cluster some.master.instance.com:3306): DeadMaster
    a.replica.6.instance.com:3306 (cluster topology2.master.instance.com:3306): DeadIntermediateMaster
    
    $ orchestrator-client -c topology -i a.replica.6.instance.com
    topology2.master.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.1.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.2.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.6.instance.com:3306 [last check invalid,5.6.17-log,STATEMENT,>>]
      + a.replica.4.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
        + a.replica.3.instance.com:3306 [OK,5.6.17-log,STATEMENT]
      + a.replica.5.instance.com:3306 [OK,5.6.17-log,STATEMENT]
      + a.replica.7.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.8.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    
    请orchestrator恢复上述已死亡的中间master：
    $ orchestrator-client -c recover -i a.replica.6.instance.com:3306
    a.replica.8.instance.com:3306
    
    $ orchestrator-client -c topology -i a.replica.8.instance.com
    topology2.master.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.1.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.2.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
    + a.replica.6.instance.com:3306 [last check invalid,5.6.17-log,STATEMENT,>>]
    + a.replica.8.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
      + a.replica.4.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
        + a.replica.3.instance.com:3306 [OK,5.6.17-log,STATEMENT]
      + a.replica.5.instance.com:3306 [OK,5.6.17-log,STATEMENT]
      + a.replica.7.instance.com:3306 [OK,5.6.17-log,STATEMENT,>>]
  
#### 更多

以上内容应该可以帮助您启动并运行。有关更多信息，请参阅手册（https://github.com/openark/orchestrator/blob/master/docs/toc.md）。对于列出的CLI命令，只需运行：

    orchestrator-client --help