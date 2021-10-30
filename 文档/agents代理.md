# Agents代理

您可以选择在MySQL主机上安装orchestrator-agent(https://github.com/openark/orchestrator-agent)。orchestrator-agent * 是向orchestrator服务器注册并通过web API接受orchestrator请求的服务。

支持的请求与常规、操作系统和LVM操作有关，例如：

* 正在主机上停止/启动MySQL服务
* 获取MySQL操作系统信息，如数据目录、端口、磁盘空间使用情况
* 执行各种LVM操作，如查找LVM快照、装载/卸载快照
* 在主机之间传输数据（例如通过netcat）

orchestrator-agent是解决特定于主机的操作的持续工作。它最初是为了克服克隆和恢复问题而开发的，后来扩展到其他领域。

orchestrator-agent向orchestrator公开的信息和API允许orchestrator通过从新可用的快照获取数据来协调和操作新的或损坏的计算机的种子设定。
此外，它允许orchestrator通过查找实际具有最新快照的主机（最好在同一数据中心），自动建议给定MySQL计算机的数据源。

对于安全措施，代理需要令牌来操作除最简单请求外的所有请求。此令牌由代理随机生成，并与orchestrator协商。
orchestrator不公开代理的令牌（现在需要做一些工作来掩盖错误消息中的令牌）

