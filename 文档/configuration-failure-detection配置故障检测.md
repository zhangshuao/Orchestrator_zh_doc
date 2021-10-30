# 配置：故障检测

orchestrator将始终检测拓扑(https://github.com/openark/orchestrator/blob/master/docs/failure-detection.md)的故障。作为配置事项，您可以设置轮询频率和orchestrator通知您此类检测的特定方式。

恢复在配置：恢复中讨论（https://github.com/openark/orchestrator/blob/master/docs/configuration-recovery.md）

    {
      "FailureDetectionPeriodBlockMinutes": 60,
    }
    
    orchestrator每秒运行一次检测。
    
FailureDetectionPeriodBlockMinutes是一种反垃圾邮件机制，阻止orchestrator一次又一次地通知相同的检测。

#### Hooks

配置orchestrator以对发现执行操作：

    {
      "OnFailureDetectionProcesses": [
        "echo 'Detected {failureType} on {failureCluster}. Affected replicas: {countReplicas}' >> /tmp/recovery.log"
      ],
    }

有许多神奇的变量（如上面的{failureCluster}）可以发送到外部hooks钩子。请参阅拓扑恢复中的完整列表(https://github.com/openark/orchestrator/blob/master/docs/topology-recovery.md)

#### MySQL 配置

由于故障检测使用MySQL拓扑本身作为信息源，因此建议您设置MySQL复制，以便清楚地指示或快速缓解错误。

* set global slave_net_timeout = 4，请参阅文档(https://dev.mysql.com/doc/refman/5.7/en/replication-options-slave.html#sysvar_slave_net_timeout), 
  这将在副本与其主副本之间设置一个短（2秒）的心跳间隔，并使副本能够快速识别故障。如果没有此设置，某些场景可能需要长达一分钟的时间来检测。
* CHANGE MASTER TO MASTER_CONNECT_RETRY=1, MASTER_RETRY_COUNT=86400. 如果复制失败，请每1秒进行一次副本尝试重新连接（默认值为60秒）。
  对于简短的网络问题，此设置尝试快速复制恢复，如果成功，将避免orchestrator执行常规failure/recovery故障/恢复操作。
  
  

