# hbase-wal







![](<../../.gitbook/assets/image (2) (1).png>)



![](<../../.gitbook/assets/image (4).png>)



**WAL的持久化的级别有如下几种：**\
SKIP\_WAL：不写wal日志,这种可以较大提高写入的性能，但是会存在数据丢失的危险，只有在大批量写入的时候才使用(出错了可以重新运行)，其他情况不建议使用。\
ASYNC\_WAL：异步写入\
SYNC\_WAL：同步写入wal日志文件，保证数据写入了DataNode节点。\
FSYNC\_WAL: 目前不支持了，表现是与SYNC\_WAL是一致的\
USE\_DEFAULT: 如果没有指定持久化级别，则默认为USE\_DEFAULT, 这个为使用Hbase全局默认级别(SYNC\_WAL)wal写入



















