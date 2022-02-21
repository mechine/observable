# 数据库CDC实践

### mysql

查看是否打开

```
show variables like 'log_bin';
```

如果打开了查看形式

```
show variables like '%binlog_format%';
show variables like '%binlog_row_image%';
```

/etc/my.cnf

```
server-id = 123
log_bin = mysql-bin
binlog_format = row
binlog_row_image = full
expire_logs_days = 10
gtid_mode = on
enforce_gtid_consistency = on
```

授权

```
CREATE USER 'roma'@'%' IDENTIFIED BY 'password';
GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'roma'@'%';
```



### sqlserver

配置

```
if exists(select 1 from sys.databases where name='fdiromatest' and is_cdc_enabled=0)
begin
    exec sys.sp_cdc_enable_db
end
```

检查

```
select is_cdc_enabled from sys.databases where name='fdiromatest'
```

表级别

```
IF EXISTS(SELECT 1 FROM sys.tables WHERE name='baris' AND is_tracked_by_cdc = 0)
BEGIN
    EXEC sys.sp_cdc_enable_table
        @source_schema = 'dbo', -- source_schema
        @source_name = 'baris', -- table_name
        @capture_instance = NULL, -- capture_instance
        @supports_net_changes = 1, -- supports_net_changes
        @role_name = NULL -- role_name
END
```

表级别检查

```
SELECT is_tracked_by_cdc FROM sys.tables WHERE name='baris'
```



如果系统表结构发生了变化或者有表级别调整，则需要重新开启CDC配置，配置步骤如下： 先关闭CDC配置，请根据实际情况填写schema和name。

```
EXEC sys.sp_cdc_disable_table 
 @source_schema = N'dbo',
 @source_name = 'baris', 
 @capture_instance ='all'
```

检查

```
IF EXISTS(SELECT 1 FROM sys.tables WHERE name='baris' AND is_tracked_by_cdc = 0) 
 BEGIN 
     EXEC sys.sp_cdc_enable_table 
         @source_schema = 'dbo', -- source_schema 
         @source_name = 'baris', -- table_name 
         @capture_instance = NULL, -- capture_instance 
         @supports_net_changes = 1, -- supports_net_changes 
         @role_name = NULL -- role_name 
 END
```

### oracle

```
shutdown immediate;
startup mount;
alter database archivelog;
alter database open;

archive log list;

alter system set db_recovery_file_dest_size = 100G;
alter system set db_recovery_file_dest = '/opt/oracle/oradata/recovery_area' scope=spfile;
```

添加用户

略



