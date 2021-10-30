# 执行

### 作为web/API服务执行

假设您已在/usr/local/orchestrator下安装orchestrator：

    cd /usr/local/orchestrator && ./orchestrator http
    
Orchestrator将开始侦听端口3000。将浏览器指向http://your.host:3000/ 你准备好出发了。您可以跳到下一节。

如果您喜欢debug调试消息，请发出：

    cd /usr/local/orchestrator && ./orchestrator --debug http
    
或者，如果出现错误，则更详细：

    cd /usr/local/orchestrator && ./orchestrator --debug --stack http
    
上面按该顺序查找/etc/orchestrator.conf.json、conf/orchestrator.conf.json、orchestrator.conf.json中的配置。
经典的做法是将配置放在/etc/orchestrator.conf.json中。由于它包含MySQL服务器的凭据，您可能希望限制对该文件的访问。
您可以选择为配置文件使用不同的位置，在这种情况下，请执行：
    
    cd /usr/local/orchestrator && ./orchestrator --debug --config=/path/to/config.file http
    
默认情况下，Web/API服务将对所有已知服务器发出连续、无限的轮询。这将使orchestrator的数据保持最新。
您通常需要此行为，但可以禁用它，使orchestrator仅服务于API/Web，但从不更新实例状态：

    cd /usr/local/orchestrator && ./orchestrator--discovery=false http

以上内容对于开发和测试非常有用。您可能希望保持默认设置。


