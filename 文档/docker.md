# Docker

有多个DockerFile可用于：

* 构建和测试orchestrator
* 创建分发文件
* 运行最小的orchestrator守护程序
* 运行一个3节点的raft设置
* 运行全面的CI环境

script/dock(https://github.com/openark/orchestrator/blob/master/script/dock)是一个方便的脚本，用于build/spawn每个docker映像。

## 构建和测试

首先，应该注意的是，您可以让GitHub操作为您完成所有工作：它将为您构建、完成测试并生成一个工件，一个Linux amd64的orchestrator二进制文件，所有这些都来自GitHub的平台。
您的计算机上不需要Docker或开发环境。见CI（https://github.com/openark/orchestrator/blob/master/docs/ci.md）

如果希望在主机上构建和测试，但不希望设置开发环境，请使用：

$ script/dock test
这将使用docker/Dockerfile.test（https://github.com/openark/orchestrator/blob/master/docker/Dockerfile.test）代表您进行构建、单元测试、集成测试和运行文档验证。

## 构建和运行

运行以下命令：

    $ script/dock alpine

它使用docker/Dockerfile在Alpine Linux上构建orchestrator并运行该服务。
Docker将端口：3000映射到您的机器上，您可以浏览到http://127.0.0.1:3000 访问orchestrator web界面。

如果在/etc/orchestrator.conf.json的容器中未绑定任何配置文件，则以下环境变量可用并生效

    * ORC_TOPOLOGY_USER：默认为orchestrator
    * ORC_TOPOLOGY_PASSWORD：默认为orchestrator
    * ORC_DB_HOST：默认为db
    * ORC_DB_PORT：默认为3306
    * ORC_DB_NAME：默认为orchestrator
    * ORC_USER：默认为orc_server_user
    * ORC_PASSWORD：默认为orc_server_password

要设置这些变量，可以将它们添加到环境文件中，在其中添加它们，如key=value（每行一对）。
然后可以将此环境文件传递给docker命令，并将--env-file=path/to/env-file添加到docker run命令。

## 创建包文件

运行以下命令：

$ script/dock pkg

要创建（通过fpm）发布包：
.deb
.rpm
.tgz

对于使用Systemd或SysVinit的Linux amd64，所有二进制文件或只是客户端脚本。它使用与官方版本（https://github.com/openark/orchestrator/releases）相同的方法。

使用Dockerfile.packaging(https://github.com/openark/orchestrator/blob/master/docker/Dockerfile.packaging)

## 运行完整的CI环境

执行：

$ script/dock system

运行完整的环境（请参阅ci-env.md），包括：

MySQL复制拓扑（通过dbdeployer）和心跳注入
作为orchestrator服务
HAProxy
Consul
consul-template
所有这些都是为了一起工作。它是测试orchestrator功能的好场所。

提示：
端口13306路由到当前拓扑主节点
MySQL拓扑在端口10111、10112、10113、10114上可用

连接到MySQL，user：ci，password：ci。
e.g.：mysqladmin -uci -pci -h 127.0.0.1 --port 13306 processlist
使用redeploy-ci-env重新创建MySQL拓扑，并重新创建和重新启动heartbeat、consul、consul-template和haproxy服务。这会将服务重置为其原始状态。

使用Dockerfile.system（https://github.com/openark/orchestrator/blob/master/docker/Dockerfile.system）

## 运行raft设置

执行：
$ script/dock raft
这将spin悬挂三个orchestrator服务：

监听 http://127.0.0.1:3007, advertising广告raft 在 127.0.0.1:10007
监听 http://127.0.0.1:3008, advertising广告raft 在 127.0.0.1:10008
监听 http://127.0.0.1:3009, advertising广告raft 在 127.0.0.1:10009

orchestrator-client配置为连接到任何节点。
