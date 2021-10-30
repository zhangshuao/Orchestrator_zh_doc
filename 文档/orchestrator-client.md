# orchestrator-client

orchestrator-client是一个使用方便的命令行界面包装API调用的脚本。

它可以自动确定orchestrator设置的leader，并在这种情况下将所有请求转发给leader。

它非常模仿orchestrator命令行界面。

使用orchestrator-client，您可以：

    * 不需要到处安装orchestrator二进制文件；仅在运行服务的主机上
    * 不需要到处部署orchestrator配置；仅在服务主机上
    * 不需要访问后端数据库
    * 需要访问HTTP api
    * 需要设置ORCHESTRATOR_API环境变量。
    
        * 为代理提供一个endpoint端点，例如。
        export ORCHESTRATOR_API=https://orchestrator.myservice.com:3000/api
        
        * 或者提供所有orchestrator endpoint端点，orchestrator-client将自动选择leader（不需要代理），例如。
        export ORCHESTRATOR_API="https://orchestrator.host1:3000/api https://orchestrator.host2:3000/api https://orchestrator.host3:3000/api"
        
    * 您可以在/etc/profile.d/orchestrator-client.sh中设置环境。如果此文件存在，它将由orchestrator-client内联。
    
#### 样本使用

显示当前已知的群集（复制拓扑）：
    
    orchestrator-client -c clusters
    
Discover, forget一个实例：    

    orchestrator-client -c discover -i 127.0.0.1:22987
    orchestrator-client -c forget -i 127.0.0.1:22987

打印拓扑实例的ASCII树。通过-i传递集群名称（请参见上面的clusters命令）：

    orchestrator-client -c topology -i 127.0.0.1:22987
    
    样本输出:
    127.0.0.1:22987
    + 127.0.0.1:22989
      + 127.0.0.1:22988
    + 127.0.0.1:22990
    
    在拓扑中移动复制副本：
    
    orchestrator-client -c relocate -i 127.0.0.1:22988 -d 127.0.0.1:22987
    
    拓扑结果:

    127.0.0.1:22987
    + 127.0.0.1:22989
    + 127.0.0.1:22988
    + 127.0.0.1:22990
    
    等
    
#### 幕后花絮

命令行接口为API调用提供了一个很好的包装器，然后将API调用的输出从JSON格式转换为文本格式。
例如，命令：

    orchestrator-client -c discover -i 127.0.0.1:22987
    
翻译为（为方便起见，此处进行了简化）：

    curl "$ORCHESTRATOR_API/discover/127.0.0.1/22987" | jq '.Details | .Key'

#### Meta元命令

* orchestrator-client -c help: 列出所有可用命令
* orchestrator-client -c which-api: API endpoint端点orchestrator-client用于调用命令的输出。当通过$ORCHESTRATOR_API提供多个endpoints端点时，这非常有用。
* orchestrator-client -c api -path clusters: 调用一个通用HTTP api调用（在本例中为集群）并返回原始JSON响应。

         
