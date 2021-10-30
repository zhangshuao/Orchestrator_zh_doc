# CI 环境

一个辅助项目orchestrator-ci-env（https://github.com/openark/orchestrator-ci-env）提供了一个MySQL复制环境，人们可以使用它来评估/测试 evaluate/test orchestrator。用例：

您希望检查orchestrator在测试环境中的行为。
您希望测试故障转移和主发现。
您希望开发对orchestrator的更改，并需要一个可复制的环境。

如果您已经有了一些具有MySQL复制拓扑或dbdeployer（https://www.dbdeployer.com/）设置的登台环境，但orchestrator-ci-env提供了一个基于Docker、可复制且无依赖性的环境，则可以执行上述所有操作。

orchestrator-ci-env 与system CI(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/ci.md#system)和Dockerfile.system(https://hub.fastgit.org/openark/orchestrator/blob/master/docs/docker.md#run-full-ci-environment)中使用的环境相同。

## Setup设置

orchestrator-ci-env 通过SSH或HTTPS克隆

    $ git clone git@github.com:openark/orchestrator-ci-env.git

或

    $ git clone https://github.com/openark/orchestrator-ci-env.git

## 运行环境

要求: Docker
$ cd orchestrator-ci-env
$ script/dock

这将建立和运行一个环境，其中包括：

    * 通过DBDeployer(https://www.dbdeployer.com/)的复制拓扑，带有心跳注入
    * HAProxy (http://www.haproxy.org/)
    * Consul (https://www.consul.io/)
    * consul-template (https://hub.fastgit.org/hashicorp/consul-template)
    
Docker将公开这些端口：

    * 10111、10112、10113、10114:复制拓扑中的MySQL主机。最初，10111是master。
    * 13306: 由HAProxy公开并路由到当前MySQL拓扑主机。

## 使用环境运行orchestrator

假设orchestrator内置在bin/orchestrator中（./script/build，如果不是）：

    $ bin/orchestrator --config=conf/orchestrator-ci-env.conf.json --debug http

conf/orchestrator-ci-env.conf.json(https://github.com/openark/orchestrator/blob/master/conf/orchestrator-ci-env.conf.json)设计用于与orchestrator-ci-env配合使用
您可以选择更改SQLite3DataFile的值，默认情况下该值位于/tmp上。

## 使用环境运行系统测试

当orchestrator运行时（请参见上文），在orchestrator的repo路径中打开另一个终端。

运行：
$ ./tests/system/test.sh
对于所有测试，或

$ ./tests/system/test.sh <name-or-regex>
对于特定测试，例如../tests/system/test.sh relocate-single

破坏性测试（如failover）需要完全重建复制拓扑。系统测试CI同时运行orchestrator和CI环境，测试可以指示CI环境重建复制。
但是，如果在本地docker上运行ci-env，则测试无法指示复制重建。破坏性测试结束时，您需要在ci-env容器上手动运行 ./script/deploy-replication。


