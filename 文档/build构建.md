# 构建和测试

开发人员有多种方法来构建和测试orchestrator。

* 使用GitHub的CI，无需开发环境
* 使用Docker
* 在dev机器上本地构建

## 通过GitHub CI构建和测试

orchestrator CI 构建(https://github.com/openark/orchestrator/blob/master/docs/ci.md)将会:

* 构建
* 测试（单元，集成）
* 上传工件，与Linux amd64兼容的orchestrator二进制文件
该工件附加在构建的输出中，并根据GitHub操作策略在几个月内有效。
这样，开发人员只需要**git checkout/commit/push**，而不需要在他们的计算机上使用任何开发环境。
一旦CI完成（成功），开发人员可以下载二进制工件以在Linux环境中进行测试。


## 通过Docker构建和测试

要求：docker安装。
orchestrator提供各种docker构建(https://github.com/openark/orchestrator/blob/master/docs/docker.md)。对于开发者：

运行script/dock来构建和运行orchestrator服务
运行script/dock测试以构建orchestrator，运行单元测试、集成测试、文档测试
运行script/dock pkg构建orchestrator并创建分发包（.deb/.rpm/.tgz）
运行script/dock system构建并启动一个完整的CI环境，该环境包括MySQL拓扑、HAProxy、Consul,、Consul-template以及作为服务运行的orchestrator。

## 在dev机器上构建和测试

要求:
* go开发设置（此时需要go1.12或更高版本）
* Git
* gcc（作为orchestrator二进制文件的一部分构建SQLite所需）
* Linux、BSD或MacOS

运行:

    git clone git@github.com:openark/orchestrator.git
    cd orchestrator

## 构建

构建途径：

    ./script/build

这需要考虑GOPATH和其他各种因素。

或者，如果您愿意并且您的围棋环境已设置，您可以运行：

    go build -o bin/orchestrator -i go/cmd/orchestrator/main.go

## 运行

在bin/目录下查找工件，例如运行：

    bin/orchestrator --debug http

## 安装后端数据库

如果使用SQLite后端运行，则不需要数据库设置。
本节的其余部分假设您有一个MySQL后端。

为了让orchestrator检测您的复制拓扑，它还必须在每个拓扑上都有一个帐户。在此阶段，所有拓扑必须使用相同的帐户（相同的用户、相同的密码）。
在您的每个母版上，发布以下内容：

    CREATE USER 'orc_user'@'%' IDENTIFIED BY 'orc_password';
    GRANT SUPER, PROCESS, REPLICATION SLAVE, RELOAD ON *.* TO 'orc_user'@'%';

将%替换为特定 hostname/127.0.0.1/subnet。明智地选择密码。编辑orchestrator.conf.json以匹配：

    "MySQLTopologyUser": "orc_user",
    "MySQLTopologyPassword": "orc_password",



