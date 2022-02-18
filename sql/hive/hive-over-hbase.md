# hive over hbase

#### 要配置HBASE的zookeeper

```
SET hbase.zookeeper.quorum=------------------------------;
SET hbase.zookeeper.property.clientPort=2181;
SET zookeeper.znode.parent=/hbase;
```



#### 写入数据的表准备

要建成text表，并制定分隔符，最好设计一下rowkey

```
create table sf_bdp.identifier_hbase(
rowkey string,
area int ,
birth bigint ,
sex int)
row format delimited fields terminated by ',
'lines terminated by '\n'
stored as textfile;
```

#### 数据准备

```
insert overwrite table sf_bdp.identifier_hbase
select
case when abs(hash(idcard))%99<10
then concat('0',abs(hash(idcard))%99,substr(idcard,3))
else concat(abs(hash(idcard))%99,substr(idcard,3))
end as rowkey,
origin,
birthday,
gender
from
dm_cmdp_icds.icds_idcard_info_clean where inc_day>20180101 and substr(idcard,1,2)='DE'
and substr(birthday,1,4)>1900 and substr(birthday,1,4)<2018 and birthday is not null and gender is not null and origin is not null
and cast(substr(birthday,5,2) as int)<13 and cast(substr(birthday,7,2) as int)<32
```

#### HFILE 准备

```
./hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator="," -Dimporttsv.columns=HBASE_ROW_KEY,base:origin,base:birthday,base:gender -Dimporttsv.bulk.output=hdfs://sfbdp1/user/hive/warehouse/sf_bdp.db/hbase_hfile_temp user_identifier hdfs://sfbdp1/user/hive/warehouse/sf_bdp.db/identifier_hbase
```

#### load hfile 到hbase

```
./hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles hdfs://sfbdp1/user/hive/warehouse/sf_bdp.db/hbase_hfile_temp user_identifier
hdfs://CNSZ17PL1784:8020/user/hive/warehouse/sf_bdp.db/hbase_hfile_temp
```

#### 建立映射表

```
CREATE external TABLE sf_bdp.identifier_hbase_shadow(
key string comment "hbase rowkey",
origin string comment "手机号",
birthday string comment "对方手机号",
gender string comment "发生时间"
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = "base:origin,base:birthday,base:gender")
TBLPROPERTIES("hbase.table.name" = "user_identifier");
```

#### 注意事项：

生成loadfile需要hbase环境的配置信息 hbase-site.xmlloadfile的阶段需要访问hdfs，访问hdfs://sfbdp1/...可能有问题，直接填sfbdp1集群的主namenode节点，如果主从发生切换要注意修改hive访问hbase也有环境依赖，在某台机器上可能缺少依赖的jar包。建议在ide上使用

#### what’s more

hbase 表的设计hbase region 的split 和 compactmapreduce过程敬请期待<\<hive的配置详解>>
