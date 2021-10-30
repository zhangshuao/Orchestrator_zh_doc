# 通过命令行执行

另请参阅Orchestrator第一步(https://github.com/openark/orchestrator/blob/master/docs/first-steps.md)页面。

orchestrator支持从命令行运行操作的两种方式：

* 使用orchestrator二进制文件（本文档的主题）

    * 您将在ops/app Box上部署orchestrator，但不会将其作为服务运行。
    * 您将部署orchestrator二进制文件的配置文件，以便能够连接到后端数据库。
    
* 使用orchestrator-client脚本。

    * 您只需要在ops/app框上使用orchestrator-client脚本。
    * 您不需要任何配置文件或二进制文件。
    * 您需要指定ORCHESTRATOR_API环境变量。

这两者（大部分）是相容的。本文档讨论第一个选项。

以下是命令行示例的概要。为简单起见，我们假设orchestrator在您的路径中。
如果不是，请将orchestrator替换为/path/to/orchestrator。

    下面的示例使用测试mysqlsandbox拓扑，其中所有实例都位于同一主机127.0.0.1和不同端口上。22987是master，22988, 22989, 22990是副本。
    
显示当前已知的群集（复制拓扑）：

    orchestrator -c clusters

    上面按该顺序查找/etc/orchestrator.conf.json、conf/orchestrator.conf.json、orchestrator.conf.json中的配置。
    经典的做法是将配置放在/etc/orchestrator.conf.json中。由于它包含MySQL服务器的凭据，您可能希望限制对该文件的访问。

您可以选择为配置文件使用不同的位置，在这种情况下，请执行：

    orchestrator -c clusters --config=/path/to/config.file
    
    -c代表命令，是强制性的。
    
发现一个新实例（"teach" orchestrator了解您的拓扑结构）。Orchestrator将自动递归地向上钻取master链（如果有）和向下钻取replicas副本链（如果有），以检测整个拓扑： 

    orchestrator -c discover -i 127.0.0.1:22987
    
    -例如，i的格式必须为hostname:port。

做同样的事情，并且更加详细：

    orchestrator -c discover -i 127.0.0.1:22987 --debug
    orchestrator -c discover -i 127.0.0.1:22987 --debug --stack

    --debug在所有操作中都很有用--stack打印（大多数）错误的代码堆栈跟踪，对于开发和测试或提交错误报告非常有用。

forget实例（可以通过上面的discover命令手动或自动重新发现实例）：
    
    orchestrator -c forget -i 127.0.0.1:22987
    
打印拓扑实例的ASCII树。通过-i传递集群名称（请参见上面的集群命令）：

    orchestrator -c topology -i 127.0.0.1:22987
    
    样本输出：
    127.0.0.1:22987
    + 127.0.0.1:22989
      + 127.0.0.1:22988
    + 127.0.0.1:22990
   
在拓扑中移动复制副本：   
   
    orchestrator -c relocate -i 127.0.0.1:22988 -d 127.0.0.1:22987
    
    结果拓扑：
    127.0.0.1:22987
    + 127.0.0.1:22989
    + 127.0.0.1:22988
    + 127.0.0.1:22990
    
上述情况恰好将复制副本向上移动了一级。但是，relocate命令接受任何有效的目标。重新定位找出移动复制副本的最佳方法。
如果启用了GTID，请使用它。如果伪GTID可用，请使用它。如果涉及binlog服务器，请使用它。
如果orchestrator对涉及的特定坐标有进一步的了解，请使用它。否则，只需使用普通的旧binlog日志file:pos math。

与relocate类似，您可以通过relocate-replicas来移动多个副本。这会将replicas-of-an-instance移动到另一台服务器下面。
    
    结果拓扑： 
    10.0.0.1:3306
    + 10.0.0.2:3306
      + 10.0.0.3:3306
      + 10.0.0.4:3306
      + 10.0.0.5:3306
    + 10.0.0.6:3306
   
    orchestrator -c relocate-replicas -i 10.0.0.2:3306 -d 10.0.0.6 
    
    结果如下：
    10.0.0.1:3306
    + 10.0.0.2:3306
    + 10.0.0.6:3306
      + 10.0.0.3:3306
      + 10.0.0.4:3306
      + 10.0.0.5:3306
      
    您可以使用--pattern来筛选受影响的复制副本。  
    
其他命令使您能够更细粒度地控制服务器的重新定位方式。考虑经典二进制日志文件：POS重新指向副本的方式：

在拓扑上向上移动副本（使其成为其主副本或其"grandparent祖父母"的直接副本）：

    orchestrator -c move-up -i 127.0.0.1:22988
    
上述命令仅在实例有grandparent祖辈且没有副本延迟等问题时才会成功。

将复制副本移动到其同级之下：

    orchestrator -c move-below -i 127.0.0.1:22988 -d 127.0.0.1:22990 --debug

上述命令仅在127.0.0.1:22988和127.0.0.1:22990是同级（同一master的副本）时才会成功，它们都没有问题（例如副本延迟），同级可以是实例的主机（即具有二进制日志、log_slave_updates、无版本冲突等）

将实例设为read-only或可写：

    orchestrator -c set-read-only -i 127.0.0.1:22988
    orchestrator -c set-writeable -i 127.0.0.1:22988

