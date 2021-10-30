# 大型环境中的Orchestrator配置

如果监视大量服务器，后端数据库可能成为瓶颈。以下注释涉及使用MySQL作为orchestrator后端。

一些配置选项允许您控制吞吐量。这些设置是：

* BufferInstanceWrites
* InstanceWriteBufferSize
* InstanceFlushIntervalMilliseconds
* DiscoveryMaxConcurrency

使用DiscoveryMaxOnCurrency限制orchestrator进行的并发发现的数量，并确保后端服务器的max_connections设置足够高，以允许orchestrator根据需要进行尽可能多的连接。

通过在orchestrator中设置BufferInstanceWrites:True，当轮询完成时，结果将被缓冲，直到InstanceFlushInterval毫秒已过或InstanceWriteBufferSize已进行缓冲写入。

缓冲写入按写入时间使用单个 insert ... on duplicate key update ... 调用，如果同一主机出现两次，则仅最后一次写入该主机的数据库。

InstanceFlushIntervalMiscles应远低于InstancePollSeconds，因为将此值设置得过高将意味着数据未写入orchestrator db后端。这可能导致最近未检查的问题。
此外，还针对后端数据库状态运行不同的运行状况检查，因此如果更新不够频繁，可能会导致Orchestrator无法正确检测不同的故障场景。

对于较大的Orchestrator环境，建议从以下值开始：

    ...
    "BufferInstanceWrites": true,
    "InstanceWriteBufferSize": 1000, 
    "InstanceFlushIntervalMilliseconds": 50,
    "DiscoveryMaxConcurrency": 1000,
    ...
  

