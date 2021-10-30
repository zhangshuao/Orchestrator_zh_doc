# 配置：发现、名称解析

让orchestrator知道如何解析主机名。

大多数人都希望这样：

    {
      "HostnameResolveMethod": "default",
      "MySQLHostnameResolveMethod": "@@hostname",
    }
    
您的主机可以通过IP地址和/或短名称和/或fqdn和/或VIP相互引用。
orchestrator需要唯一且一致地标识主机。它通过解析目标主机名来实现。

许多人认为"MySQLHostnameResolveMethod："@@hostname"是最简单的。你的选择是：

    "HostnameResolveMethod"："cname"：对主机名执行cname解析
    "HostnameResolveMethod"："default"：不通过网络协议进行特殊解析
    "MySQLHostnameResolveMethod"："@@hostname"：发出一个select @@hostname命令
    "MySQLHostnameResolveMethod"："@@report_host"：发出一个select @@report_host，要求配置report_host
    "HostnameResolveMethod"："none"和"MySQLHostnameResolveMethod"：""：不执行任何操作。永远不要下决心。这可能适用于所有设备都始终使用IP地址的设置。


