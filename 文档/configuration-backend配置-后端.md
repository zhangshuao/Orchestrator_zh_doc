# 配置: backend后端

让orchestrator知道在哪里可以找到后端数据库。在此设置中，orchestrator将在端口3000上提供HTTP服务。

    {
      "Debug": false,
      "ListenAddress": ":3000",
    }

您可以选择MySQL后端或SQLite后端。请参阅"高可用性"(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/high-availability.md)页面，了解使用这两种方法的场景、可能性和原因。

## MySQL backend后端

您需要设置schema模式&credentials凭据：

    {
      "MySQLOrchestratorHost": "orchestrator.backend.master.com",
      "MySQLOrchestratorPort": 3306,
      "MySQLOrchestratorDatabase": "orchestrator",
      "MySQLOrchestratorCredentialsConfigFile": "/etc/mysql/orchestrator-backend.cnf",
    }

请注意MySQLOrchestratorCredentialsConfigFile。其形式如下：

    [client]
    user=orchestrator_srv
    password=${ORCHESTRATOR_PASSWORD}

其中，user或password可以是明文，也可以从环境中获取其值。

或者，您可以选择在配置文件中使用纯文本凭据：

    {
      "MySQLOrchestratorUser": "orchestrator_srv",
      "MySQLOrchestratorPassword": "orc_server_password",
    }

##### MySQL后端数据库设置

对于MySQL后端数据库，您需要授予必要的特权：

    CREATE USER 'orchestrator_srv'@'orc_host' IDENTIFIED BY 'orc_server_password';
    GRANT ALL ON orchestrator.* TO 'orchestrator_srv'@'orc_host';

## SQLite backend后端

默认后端是MySQL。要设置SQLite，请使用：

    {
      "BackendDB": "sqlite",
      "SQLite3DataFile": "/var/lib/orchestrator/orchestrator.db",  
    }

SQLite嵌入在orchestrator中。

如果SQLite3DataFile指示的文件不存在，orchestrator将创建它。它将需要对给定path/file的写入权限。

