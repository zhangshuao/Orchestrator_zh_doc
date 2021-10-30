# Security安全

在HTTP模式（API或Web）下操作时，对orchestrator的访问可能会受到以下限制：

* 基本身份验证


    将以下内容添加到orchestrator的配置文件：
        "AuthenticationMethod": "basic",
        "HTTPAuthUser":         "dba_team",
        "HTTPAuthPassword":     "time_for_dinner"

    使用基本身份验证，只有一个凭证，没有角色。

Orchestrator的配置文件包含MySQL服务器的凭据以及上面指定的基本身份验证凭据。保持安全（例如chmod 600）。

* 基本身份验证，扩展

    
    将以下内容添加到orchestrator的配置文件：
        "AuthenticationMethod": "multi",
        "HTTPAuthUser":         "dba_team",
        "HTTPAuthPassword":     "time_for_dinner"
   
    多重身份验证的工作原理与basic类似，但也接受用户使用任何密码进行只读。只读用户可以查看所有内容，但无法通过API执行写入操作（例如停止副本、重新写入副本、发现新实例等）
   
           
* Headers标头身份验证

    
    通过反向代理转发的头进行身份验证（例如，Apache2将请求转发给orchestrator）。要求：
        "AuthenticationMethod": "proxy",
        "AuthUserHeader": "X-Forwarded-User",
    
    您需要将反向代理配置为通过HTTP头发送经过身份验证的用户的名称，并使用与AuthUserHeader配置相同的头名称。
    
     
    例如，Apache2设置可能如下所示：
        RequestHeader unset X-Forwarded-User
        RewriteEngine On
        RewriteCond %{LA-U:REMOTE_USER} (.+)
        RewriteRule .* - [E=RU:%1,NS]
        RequestHeader set X-Forwarded-User %{RU}e
         
    代理身份验证允许角色。一些用户是超级用户，其余只是普通用户。允许高级用户更改拓扑，而普通用户处于只读模式。要指定已知DBA的列表，请使用：
    "PowerAuthUsers": [
         "wallace", "gromit", "shaun"
         ],
         
    或者，不管怎样，您可以通过以下方式将整个orchestrator进程变为只读：
        "ReadOnly": "true",
        
    您可以将ReadOnly与您喜欢的任何身份验证方法相结合。
    
