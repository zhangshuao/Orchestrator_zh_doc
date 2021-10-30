# tags标签

**orchestrator**支持标记实例和按标记搜索。

标记是作为服务提供给用户的，不由orchestrator在内部使用。

### Tag标签 命令

    支持以下命令。分项数字如下：
    * orchestrator-client -c tag -i some.instance --tag name=value
    * orchestrator-client -c tag -i some.instance --tag name
    * orchestrator-client -c untag -i some.instance -t name
    * orchestrator-client -c untag-all -t name=value
    * orchestrator-client -c tags -i some.instance
    * orchestrator-client -c tag-value -i some.instance -t name
    * orchestrator-client -c tagged -t name
    * orchestrator-client -c tagged -t name=value
    * orchestrator-client -c tagged -t name=

    这些API endpoint端点：
    * api/tag/:host/:port/:tagName/:tagValue
    * api/tag/:host/:port?tag=name
    * api/tag/:host/:port?tag=name%3Dvalue
    * api/untag/:host/:port/:tagName
    * api/untag/:host/:port?tag=name
    * api/untag-all/:tagName/:tagValue
    * api/untag-all?tag=name%3Dvalue
    * api/tags/:host/:port
    * api/tag-value/:host/:port/:tagName
    * api/tag-value/:host/:port?tag=name
    * api/tagged?tag=name
    * api/tagged?tag=name%3Dvalue
    * api/tagged?tag=name%3D

### Tag标签，概述

    标记的形式可以是name=value，也可以是form name，在这种情况下，值被隐式设置为空字符串。名称可以采用以下格式：
    
    * word
    * some-other-word
    * some_word_word_yet

    尽管没有严格执行，但应避免使用特殊字符/标点符号。
    
### Tagging标记

    -c tag 或 api/tag将现有标记添加或替换到实例中。orchestrator不会指明标记是否预先存在，也不会提供以前的值（如果有）。
    
    例如:
    $ orchestrator-client -c tag -i db-host-01:3306 --tag vttablet_alias=dc1-0123456789
    
    在上面的示例中，我们选择创建一个名为vttablet_alias的标记，该标记带有一个值。
    
    标记是针对每个实例的。实例本身不受此操作的影响。orchestrator将标记维护为元数据。该实例不需要可用。

### 取消标记：单个实例

    -c untag 或 api/untag 从给定实例中移除标记（如果存在）。如果标记确实存在，orchestrator将输出实例名称；
    如果标记不存在，orchestrator将输出空输出。
    
    您可以：
        * 指定标记名称和标记值：仅当标记等于该值时，才会删除该标记。
        * 仅指定标记名：标记将被删除，而不管其值如何。

    例如:
    $ orchestrator-client -c untag -i db-host-01:3306 --tag vttablet_alias

### 取消标记：多个实例

    -c untag-all 或 api/untag-all 从值匹配的所有实例中删除标记。请注意，必须提供标签值。

    例如:
    $ orchestrator-client -c untag-all --tag vttablet_alias=dc1-0123456789
      
### 列出实例标记

    对于给定的实例，-c tags或api/tags列出了所有已知的标记。

    例如：
    $ orchestrator-client -c tag -i db-host-01:3306 --tag vttablet_alias=dc1-0123456789
    $ orchestrator-client -c tag -i db-host-01:3306 --tag old-hardware
    
    $ orchestrator-client -c tags -i db-host-01:3306
    old-hardware=
    vttablet_alias=dc1-0123456789

    列出的标记按名称排序。请注意，我们添加了没有值的旧硬件标记。它导出为old-hardware=,，隐式为空值。
    
### 列出集群的实例标记

    对于给定的实例或集群别名-c topology-tags或api/topology-tags列出了集群拓扑以及每个实例的所有已知标记。
    
    例如:
    $ orchestrator-client -c tag -i db-host-01:3306 --tag vttablet_alias=dc1-0123456789
    $ orchestrator-client -c tag -i db-host-01:3306 --tag old-hardware
    
    $ orchestrator-client -c topology-tags -alias mycluster
    db-host-01:3306     [0s,ok,5.7.23-log,rw,ROW,>>,GTID,P-GTID] [vttablet_alias=dc1-0123456789, old-hardware]
    + db-host-02:3306   [0s,ok,5.7.23-log,ro,ROW,>>,GTID,P-GTID] []
    
    $ orchestrator-client -c topology-tags -i db-host-01:3306
    db-host-01:3306     [0s,ok,5.7.23-log,rw,ROW,>>,GTID,P-GTID] [vttablet_alias=dc1-0123456789, old-hardware]
    + db-host-02:3306   [0s,ok,5.7.23-log,ro,ROW,>>,GTID,P-GTID] []

### 获取特定标记的值

    -c tag-value 或 api/tag-value 返回实例上特定标记的值。
    
    例如:
    $ orchestrator-client -c tag -i db-host-01:3306 --tag vttablet_alias=dc1-0123456789
    $ orchestrator-client -c tag -i db-host-01:3306 --tag old-hardware
    $ orchestrator-client -c tag-value -i db-host-01:3306 --tag vttablet_alias
    dc1-0123456789
    $ orchestrator-client -c tag-value -i db-host-01:3306 --tag old-hardware
    
    # <empty value>
    $ orchestrator-client -c tag-value -i db-host-01:3306 --tag no-such-tag
    tag no-such-tag not found for db-host-01:3306
    # in stderr

### 按标记搜索实例

    -c tagged 或 api/tagged按标记列出实例，如下所示：
    
    * -c tagged -tag name=value: 列出name存在且等于value的实例。
    * -c tagged -tag name: 列出名称存在的实例，而不考虑其值。
    * -c tagged -tag name=: 列出name存在且具有空值的实例。
    * -c tagged -tag name,role=backup: 列出按名称标记的实例（无论其值如何），并且也用role=backup标记
    * -c tagged -tag !name: 列出不存在名为name的标记的实例，无论其值如何
    * -c tagged -tag ~name: ~是！的同义词!
    * -c tagged -tag name,~role: 列出按名称标记的实例（不考虑其值），以及不按角色标记的实例（不考虑其值）
    * -c tagged -tag ~role=backup: 列出用role标记但值不是backup的实例。请注意，这与-c taged-tag~role有什么不同，后者首先会列出没有role标记的实例。

### 标签，内部

    标记与实例关联，但关联是orchestrator内部的，不会影响实际的MySQL实例。
    标记存储在后端表的内部。标签通过raft协议发布；由筏长执行的标记操作（标记、解除标记）将应用于筏板随动件。
    
### 用例

    对标签的需求来自具有不同用例的多个用户。
    
        * Vitess(http://github.com/vitess.io/vitess)用户的一个常见用例是需要将实例与vttablet别名关联。
        * 用户可能希望应用基于标签的促销逻辑。虽然orchestrator在任何决策过程中都不在内部使用标记，但用户可以基于标记设置提升规则，或者应用不同的故障切换操作。
        
