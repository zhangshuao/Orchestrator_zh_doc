# 人工伪gtid注入

自动伪GTID是后来添加的，它取代了手动伪GTID注入的需要，建议使用。
但是，您仍然可以选择注入您自己的伪GTID。

要手动启用伪GTID，您需要：

    1.经常向二进制日志中注入唯一的条目
    2.配置orchestrator以识别此类条目
    3.可选地向orchestrator提示这些条目是按升序排列的

在二进制日志中注入一个条目就是发出一条语句。根据您使用的是基于语句的复制还是基于行的复制，这样的语句可以是INSERT、CREATE或其他。
请参考以下博客条目：Pseudo-GTID(http://code.openark.org/blog/mysql/pseudo-gtid)、Pseudo-GTID、基于行的复制(http://code.openark.org/blog/mysql/pseudo-gtid-row-based-replication)、使用Pseudo-GTID重构复制拓扑(http://code.openark.org/blog/mysql/refactoring-replication-topology-with-pseudo-gtid)以了解更多详细信息。

可通过以下方式注入伪GTID：

    * MySQL的event_scheduler事件调度程序(https://dev.mysql.com/doc/refman/5.7/en/event-scheduler.html)（参见下面的示例）
    * 外部注入，请参见示例文件：(https://github.com/openark/orchestrator/tree/master/resources/pseudo-gtid)
        * 用于注入伪gtid的脚本
        * start-stop脚本用作/etc/init.d/pseudo-gtid上的守护进程
        * puppet模块。

#### 通过DROP VIEW IF EXISTS & INSERT INTO ... ON DUPLICATE KEY UPDATE提升伪GTID

    create database if not exists meta;
    use meta;
    
    create table if not exists pseudo_gtid_status (
        anchor                      int unsigned not null,
        originating_mysql_host      varchar(128) charset ascii not null,
        originating_mysql_port      int unsigned not null,
        originating_server_id       int unsigned not null,
        time_generated              timestamp not null default current_timestamp,
        pseudo_gtid_uri             varchar(255) charset ascii not null,
        pseudo_gtid_hint            varchar(255) charset ascii not null,
        PRIMARY KEY (anchor)
    );
    
    drop event if exists create_pseudo_gtid_event;
    delimiter $$
    create event if not exists
    create_pseudo_gtid_event
    on schedule every 5 second starts current_timestamp
    on completion preserve
    enable
    do
        main: begin
        DECLARE lock_result INT;
        DECLARE CONTINUE HANDLER FOR SQLEXCEPTION BEGIN END;
    
        IF @@global.read_only = 1 THEN
            LEAVE main;
        END IF;
    
        SET lock_result = GET_LOCK('pseudo_gtid_status', 0);
        IF lock_result = 1 THEN
            set @connection_id := connection_id();
            set @now := now();
            set @rand := floor(rand()*(1 << 32));
            set @pseudo_gtid_hint := concat_ws(':', lpad(hex(unix_timestamp(@now)), 8, '0'), lpad(hex(@connection_id), 16, '0'), lpad(hex(@rand), 8, '0'));
            set @_create_statement := concat('drop ', 'view if exists `meta`.`_pseudo_gtid_', 'hint__asc:', @pseudo_gtid_hint, '`');
            PREPARE st FROM @_create_statement;
            EXECUTE st;
            DEALLOCATE PREPARE st;
    
            /*!50600
            SET innodb_lock_wait_timeout = 1;
             */
            set @serverid := @@server_id;
            set @hostname := @@hostname;
            set @port := @@port;
            set @pseudo_gtid := concat('pseudo-gtid://', @hostname, ':', @port, '/', @serverid, '/', date(@now), '/', time(@now), '/', @rand);
            insert into pseudo_gtid_status (
                anchor,
                originating_mysql_host,
                originating_mysql_port,
                originating_server_id,
                time_generated,
                pseudo_gtid_uri,
                pseudo_gtid_hint
            )
            values (1, @hostname, @port, @serverid, @now, @pseudo_gtid, @pseudo_gtid_hint)
            on duplicate key update
            originating_mysql_host = values(originating_mysql_host),
            originating_mysql_port = values(originating_mysql_port),
            originating_server_id = values(originating_server_id),
            time_generated = values(time_generated),
            pseudo_gtid_uri = values(pseudo_gtid_uri),
            pseudo_gtid_hint = values(pseudo_gtid_hint)
            ;
            SET lock_result = RELEASE_LOCK('pseudo_gtid_status');
        END IF;
    end main
    $$
    
    delimiter ;
    
    set global event_scheduler := 1;


以及匹配的配置条目：

    {
      "PseudoGTIDPattern": "drop view if exists .*?`_pseudo_gtid_hint__",
      "DetectPseudoGTIDQuery": "select count(*) as pseudo_gtid_exists from meta.pseudo_gtid_status where anchor = 1 and time_generated > now() - interval 2 day",
      "PseudoGTIDMonotonicHint": "asc:",
    }

上面尝试删除实际上不存在的视图。该语句实际上什么都不做，但会通过复制流传播。与前面的示例相反，它不会使用过度锁定。

虽然该语句在二进制日志中可见，但在数据本身中不可见。第二条语句注册表数据中的最新更新。它不是严格要求的，但有助于确保伪gtid正在运行。
DetectPseudoGTIDQuery配置允许orchestrator实际检查最近是否注入了伪GTID。

上面的逻辑还确保注入的伪gtid条目按升序排列。PseudoGTIDMonotonicHint配置与查询中的asc:hint相关。升序允许orchestrator在服务器二进制日志上搜索给定的伪GTID条目时执行进一步优化。

orchestrator仅在PseudoGTIDPattern配置变量为非空时启用伪GTID模式，但只能在运行时验证其正确性。

如果您的模式不正确（因此，orchestrator in无法在二进制日志中找到模式），您将无法通过伪GTID在拓扑中移动副本，并且只有在尝试移动时才能发现这一点。

如果使用orchestrator管理多个拓扑，则需要对所有拓扑使用相同的伪GTID注入方法，因为只有一个PseudoGTIDPattern值。
