# 安装

有关生产部署，请参见Orchestrator部署(https://github.com/openark/orchestrator/blob/master/docs/deployment.md)。
以下文本将引导您完成手动安装方式以及使其正常工作所需的配置。

以下假设您将对orchestrator二进制文件和MySQL后端使用同一台机器。
如果没有，请使用适当的主机名替换127.0.0.1。将orch_backend_password替换为您自己的超级秘密密码。

#### 提取orchestrator二进制文件和

    * 从tarball解压
    提取从中下载的存档文件https://github.com/openark/orchestrator/releases 
    
    例如，假设您希望在 /usr/local/orchestrator下安装orchestrator： 
    
        sudo mkdir -p /usr/local
        sudo cd /usr/local
        sudo tar xzfv orchestrator-1.0.tar.gz

    * 从RPM安装
    
        sudo rpm -i orchestrator-1.0-1.x86_64.rpm
    
    * 从DEB安装
    
        sudo dpkg -i orchestrator_1.0_amd64.deb
    
    * 从repository仓库安装
    
        可以在中找到orchestrator包https://packagecloud.io/github/orchestrator
    

#### 安装后端MySQL服务器

为后端设置MySQL服务器，并调用以下命令：
    
    CREATE DATABASE IF NOT EXISTS orchestrator;
    CREATE USER 'orchestrator'@'127.0.0.1' IDENTIFIED BY 'orch_backend_password';
    GRANT ALL PRIVILEGES ON `orchestrator`.* TO 'orchestrator'@'127.0.0.1';

Orchestrator使用一个配置文件，位于/etc/Orchestrator.conf.json或二进制conf/orchestrator.conf.json或orchestrator.conf.json的相对路径中。

提示：安装的软件包包括一个名为orchestrator.conf.json.sample的文件，其中包含一些基本设置，您可以将这些设置用作orchestrator.conf.json的基线。
它位于/usr/local/orchestrator/orchestrator-sample.conf.json中，您还可以找到/usr/local/orchestrator/orchestrator-sample-sqlite.conf.json，
它具有面向sqlite的配置。这些示例文件也可以在orchestrator存储库中找到。（https://github.com/openark/orchestrator/tree/master/conf）

编辑orchestrator.conf.json以匹配上述内容，如下所示：

    ...
    "MySQLOrchestratorHost": "127.0.0.1",
    "MySQLOrchestratorPort": 3306,
    "MySQLOrchestratorDatabase": "orchestrator",
    "MySQLOrchestratorUser": "orchestrator",
    "MySQLOrchestratorPassword": "orch_backend_password",
    ...

#### 授予对所有MySQL服务器上orchestrator的访问权限

为了让orchestrator检测您的复制拓扑，它还必须在每个拓扑上都有一个帐户。在此阶段，所有拓扑必须使用相同的帐户（相同的用户、相同的密码）。
在您的每个母版上，发布以下内容：

    CREATE USER 'orchestrator'@'orch_host' IDENTIFIED BY 'orch_topology_password';
    GRANT SUPER, PROCESS, REPLICATION SLAVE, RELOAD ON *.* TO 'orchestrator'@'orch_host';
    GRANT SELECT ON mysql.slave_master_info TO 'orchestrator'@'orch_host';
    GRANT SELECT ON ndbinfo.processes TO 'orchestrator'@'orch_host'; -- Only for NDB Cluster

REPLICATION SLAVE 依赖于 SHOW SLAVE HOSTS, 为了在MySQL 5.6及更高版本的SHOW PROCESSLIST中扫描二进制日志以支持RESET SLAVE操作进程所需的伪GTID重载，需要查看副本进程，
如果使用master_info_repository='TABLE'，则让orchestrator访问mysql.slave_master_info表。
这将允许orchestrator在需要时获取复制凭据。如果需要的话。

将orch_host替换为主机名或orchestrator计算机（或者执行通配符操作）。明智地选择密码。编辑orchestrator.conf.json以匹配：

    "MySQLTopologyUser": "orchestrator",
    "MySQLTopologyPassword": "orch_topology_password",

考虑移动conf/orchestrator.conf.json到/etc/orchestrator.conf.json（两个位置都是有效的）

要在命令行模式或仅在HTTP API中执行orchestrator，只需orchestrator二进制文件即可。
要享受丰富的web界面，包括拓扑可视化和拖放拓扑更改，您需要参考资料目录及其下的所有内容。
如果你不确定，不要触摸；事情已经准备好了。
