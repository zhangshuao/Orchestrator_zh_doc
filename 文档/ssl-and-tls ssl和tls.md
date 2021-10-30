# SSL 和 TLS

Orchestrator支持web界面的SSL/TLS作为HTTPS。这可以是标准的服务器端证书，也可以配置Orchestrator以验证和筛选客户端提供的具有相互TLS的证书。

Orchestrator还允许使用证书对MySQL进行身份验证。

如果MySQL使用SSL加密进行复制，Orchestrator将尝试在恢复期间使用SSL配置复制。

**Web/API接口的HTTPS**

    您可以按如下方式设置SSL/TLS保护：
    {
        "UseSSL": true,
        "SSLPrivateKeyFile": "PATH_TO_CERT/orchestrator.key",
        "SSLCertFile": "PATH_TO_CERT/orchestrator.crt",
        "SSLCAFile": "PATH_TO_CERT/ca.pem",
    }

    如果不需要指定证书颁发机构，则SSLCA文件是可选的。这将通过web接口（和API）启用SSL，以便像普通HTTPS网页一样对通信进行加密。

    类似地，如果您正在使用Orchestrator代理，则可以为代理API设置此设置：
    {
        "AgentsUseSSL": true,
        "AgentSSLPrivateKeyFile": "PATH_TO_CERT/orchestrator.key",
        "AgentSSLCertFile": "PATH_TO_CERT/orchestrator.crt",
        "AgentSSLCAFile": "PATH_TO_CERT/ca.pem",
    }
    这可以是相同的SSL证书，但不一定是。

**相互TLS**

    它还支持相互TLS的概念。也就是说，必须提供对客户端和服务器都有效的证书。这通常用于保护内部网络中的服务对服务通信。这些证书通常由内部根证书签名。
    
    在这种情况下，证书 必须1）有效，2）用于正确的服务。正确的服务由对客户端证书的组织单位（OU）进行筛选决定。
    
    设置私有根CA不是一项简单的任务。指导如何成功实现这一目标超出了这些文件的范围
    
    考虑到这一点，您可以通过如上所述设置SSL来设置相互TLS，但也可以添加以下指令：
    {
        "UseMutualTLS": true,
        "SSLValidOUs": [ "service1", "service2" ],
    }

    这将打开客户端证书验证，并开始根据其OU筛选客户端。OU筛选是强制性的，因为没有它，使用相互TLS是毫无意义的。
    在这种情况下，service1和service2将能够连接到Orchestrator，前提是它们的证书有效，并且它们有一个具有确切服务名称的OU。

    
**MySQL身份验证**

    您还可以使用client客户机证书对mysql连接进行身份验证或加密。您可以通过以下方式加密到MySQL服务器Orchestrator使用的连接：
    
    {
        "MySQLOrchestratorUseMutualTLS": true,
        "MySQLOrchestratorSSLSkipVerify": true,
        "MySQLOrchestratorSSLPrivateKeyFile": "PATH_TO_CERT/orchestrator-database.key",
        "MySQLOrchestratorSSLCertFile": "PATH_TO_CERT/orchestrator-database.crt",
        "MySQLOrchestratorSSLCAFile": "PATH_TO_CERT/ca.pem",
    }
    
    类似地，拓扑数据库的连接可以通过以下方式进行加密：
    {
        "MySQLTopologyUseMutualTLS": true,
        "MySQLTopologySSLSkipVerify": true,
        "MySQLTopologySSLPrivateKeyFile": "PATH_TO_CERT/orchestrator-database.key",
        "MySQLTopologySSLCertFile": "PATH_TO_CERT/orchestrator-database.crt",
        "MySQLTopologySSLCAFile": "PATH_TO_CERT/ca.pem",
    }
    在这种情况下，所有拓扑服务器都必须响应提供的证书。当前没有只为某些服务器启用TLS的方法。

**MySQL SSL复制**

    如果Orchestrator能够将故障源配置为在恢复期间复制到新升级的源，则如果新升级的源是以这种方式配置的，则Orchestrator将尝试配置Master_SSL=1。
    Orchestrator当前不处理为恢复期间的复制配置源SSL证书。
    
