# Evernote Export

### HIVE执行计划 <a href="#hive-zhi-hang-ji-hua" id="hive-zhi-hang-ji-hua"></a>

开发中常见的问题：\
功能实现 – 什么事情是可以做到的，SQL的能力边界\
错误定位 – 内存错误，执行不结束\
执行性能 – 数据倾斜

* [HIVE执行计划](broken-reference)
  * [WHY TEZ:](broken-reference)
  * [HOW TO USE TEZ(HIVE)](broken-reference)
    * [mappers](broken-reference)
    * [reducers](broken-reference)
  * [HOW TO READ TEZ EXECUTION PLAN](broken-reference)
  * [EXAMPLES](broken-reference)
  * [HOW TO READ MAPREDUCE EXECUTION PLAN](broken-reference)
  * [MAPREDUCE EXAMPLES. COMPARE](broken-reference)

### WHY TEZ: <a href="#why-tez" id="why-tez"></a>

他们说要有图：

Mapreduce任务的构造：

TEZ任务的构造：

A typical Map reduce job has following steps:

> 1. Read data from file –>one disk access
> 2. Run mappers
> 3. Write map output –> second disk access
> 4. Run shuffle and sort –> read map output, third disk access
> 5. write shuffle and sort –> write sorted data for reducers –> fourth disk access
> 6. Run reducers which reads sorted data –> fifth disk output
> 7. Write reducers output –>sixth disk access

Tez works very similar to Spark (Tez was created by Hortonworks well before Spark):

> 1. Execute the plan but no need to read data from disk.
> 2. Once ready to do some calculations (similar to actions in spark), get the data from disk and perform all steps and produce output.

Only one read and one write.\
另外tez在小任务上也有一定的优化：

> ApplicationMaster缓冲池\
> 预先启动Container\
> Container重用

TIPS:\
任务越复杂，TEZ优化效果越明显。\
TEZ在成长的路上。可能还没有MR成熟。

### HOW TO USE TEZ(HIVE) <a href="#how-to-use-tezhive" id="how-to-use-tezhive"></a>

#### mappers <a href="#mappers" id="mappers"></a>

```
set tez.grouping.min-size=16777216;  
VS. 
set mapreduce.input.fileinputformat.split.minsize=16777216;
set tez.grouping.max-size=1073741824;  
VS.
set mapreduce.input.fileinputformat.split.minsize=1073741824; 
```

#### reducers <a href="#reducers" id="reducers"></a>

```
hive.exec.reducers.bytes.per.reducer(default 256000000);
hive.exec.reducers.max(default 1009);
hive.tez.auto.reducer.parallelism(default false);
```

> Turn on Tez’ auto reducer parallelism feature. When enabled, Hive will still estimate data sizes and set parallelism estimates. Tez will sample source vertices’ output sizes and adjust the estimates at runtime as necessary.

MORE:\
[How initial task parallelism works](https://cwiki.apache.org/confluence/display/TEZ/How+initial+task+parallelism+works)\
[Hive + Tez: A Performance Deep Dive](https://www.slideshare.net/Hadoop\_Summit/w-235phall1pandey)\
[Understanding Considerations to Move Existing MapReduce Jobs in Hive to Tez](https://docs.qubole.com/en/latest/user-guide/hive/hive-tez-tuning.html)\
[Demystify Apache Tez Memory Tuning - Step by Step](https://community.hortonworks.com/articles/14309/demystify-tez-tuning-step-by-step.html)\
[Hive on Tez : How to control the number of Mappers and Reducers](http://www.openkb.info/2017/05/hive-on-tez-how-to-control-number-of.html)

### HOW TO READ TEZ EXECUTION PLAN <a href="#how-to-read-tez-execution-plan" id="how-to-read-tez-execution-plan"></a>

tez:\
vertices：顶点\
edge：边

BROADCAST

> Output on this edge produced by any source task is available to all destination tasks.

CUSTOM

> Custom routing defined by the user.

ONE\_TO\_ONE

> Output on this edge produced by the i-th source task is available to the i-th destination task.

SCATTER\_GATHER

> The i-th output on this edge produced by all source tasks is available to the same destination task.

CONTAINS\
\>

### EXAMPLES <a href="#examples" id="examples"></a>

`explain select count(waybill_no) from gdl.tt_waybill_info where inc_day=20181104 ;`

```
Plan optimized by CBO.
Vertex dependency in root stage
Reducer 2 <- Map 1 (SIMPLE_EDGE)
Stage-0
Fetch Operator
    limit:-1
Stage-1
Reducer 2
File Output Operator [FS_7]
Group By Operator [GBY_5] (rows=1 width=8)
Output:["_col0"],aggregations:["count(VALUE._col0)"]
        <-Map 1 [SIMPLE_EDGE]
SHUFFLE [RS_4]
Group By Operator [GBY_3] (rows=1 width=8)
Output:["_col0"],aggregations:["count(waybill_no)"]
Select Operator [SEL_2] (rows=6695357 width=183)
Output:["waybill_no"]
TableScan [TS_0] (rows=6695357 width=183)
                  gdl@tt_waybill_info,tt_waybill_info,Tbl:COMPLETE,Col:NONE,Output:["waybill_no"]
```

`explain select count(distinct waybill_no) from gdl.tt_waybill_info where inc_day=20181104 ;`

```
Plan optimized by CBO.
Vertex dependency in root stage
Reducer 2 <- Map 1 (SIMPLE_EDGE)
Reducer 3 <- Reducer 2 (SIMPLE_EDGE)
Stage-0
Fetch Operator
    limit:-1
Stage-1
Reducer 3
File Output Operator [FS_12]
Group By Operator [GBY_10] (rows=1 width=8)
Output:["_col0"],aggregations:["count(VALUE._col0)"]
        <-Reducer 2 [SIMPLE_EDGE]
SHUFFLE [RS_9]
Group By Operator [GBY_8] (rows=1 width=8)
Output:["_col0"],aggregations:["count(_col0)"]
Group By Operator [GBY_5] (rows=3347678 width=183)
Output:["_col0"],keys:KEY._col0
              <-Map 1 [SIMPLE_EDGE]
SHUFFLE [RS_4]
PartitionCols:_col0
Group By Operator [GBY_3] (rows=6695357 width=183)
Output:["_col0"],keys:waybill_no
Select Operator [SEL_2] (rows=6695357 width=183)
Output:["waybill_no"]
TableScan [TS_0] (rows=6695357 width=183)
                        gdl@tt_waybill_info,tt_waybill_info,Tbl:COMPLETE,Col:NONE,Output:["waybill_no"]
```

`explain select dest_division_code, count(*) from gdl.tt_waybill_info where inc_day=20181102 group by dest_division_code;`

```
Plan optimized by CBO.
Vertex dependency in root stage
Reducer 2 <- Map 1 (SIMPLE_EDGE)
Stage-0
Fetch Operator
    limit:-1
Stage-1
Reducer 2
File Output Operator [FS_7]
Group By Operator [GBY_5] (rows=5972109 width=183)
Output:["_col0","_col1"],aggregations:["count(VALUE._col0)"],keys:KEY._col0
        <-Map 1 [SIMPLE_EDGE]
SHUFFLE [RS_4]
PartitionCols:_col0
Group By Operator [GBY_3] (rows=11944218 width=183)
Output:["_col0","_col1"],aggregations:["count()"],keys:dest_division_code
Select Operator [SEL_2] (rows=11944218 width=183)
Output:["dest_division_code"]
TableScan [TS_0] (rows=11944218 width=183)
                  gdl@tt_waybill_info,tt_waybill_info,Tbl:COMPLETE,Col:NONE,Output:["dest_division_code"]
```

`explain select dest_division_code, rn from (select dest_division_code, row_number()over(order by dest_division_code) rn from gdl.tt_waybill_info where inc_day=20181102) t1 where rn < 10 ;`

```
Plan optimized by CBO.
Vertex dependency in root stage
Reducer 2 <- Map 1 (SIMPLE_EDGE)
Stage-0
Fetch Operator
    limit:-1
Stage-1
Reducer 2
File Output Operator [FS_9]
Select Operator [SEL_5] (rows=3981406 width=183)
Output:["_col0","_col1"]
Filter Operator [FIL_10] (rows=3981406 width=183)
            predicate:(row_number_window_0 < 10)
PTF Operator [PTF_4] (rows=11944218 width=183)
Function definitions:[{},{"name:":"windowingtablefunction","order by:":"_col15 ASC NULLS FIRST","partition by:":"0"}]
Select Operator [SEL_3] (rows=11944218 width=183)
Output:["_col15"]
              <-Map 1 [SIMPLE_EDGE]
SHUFFLE [RS_2]
PartitionCols:0
TableScan [TS_0] (rows=11944218 width=183)
                    gdl@tt_waybill_info,tt_waybill_info,Tbl:COMPLETE,Col:NONE,Output:["dest_division_code"]
```

`explain select waybill_no,real_all_fee_rmb from gdl.tt_waybill_info where inc_day=20181102 order by real_all_fee_rmb limit 10 ;`

```
Plan optimized by CBO.
Vertex dependency in root stage
Reducer 2 <- Map 1 (SIMPLE_EDGE)
Stage-0
Fetch Operator
    limit:10
Stage-1
Reducer 2
File Output Operator [FS_6]
Limit [LIM_5] (rows=10 width=183)
Number of rows:10
Select Operator [SEL_4] (rows=11944218 width=183)
Output:["_col0","_col1"]
          <-Map 1 [SIMPLE_EDGE]
SHUFFLE [RS_3]
Select Operator [SEL_2] (rows=11944218 width=183)
Output:["_col0","_col1"]
TableScan [TS_0] (rows=11944218 width=183)
                  gdl@tt_waybill_info,tt_waybill_info,Tbl:COMPLETE,Col:NONE,Output:["waybill_no","real_all_fee_rmb"]
```

融合报表之取件行为规范：

```
explain
insert overwrite table dm_next.pickup_standard_dtl partition (inc_day)
select a.waybill_no, --运单号
       a.dept_code, --操作网点
       a.emp_code, --操作员工号
       d.emp_name, --操作员姓名
       a.bar_scan_tm, --54操作时间
       a.bar_scan_lng, --54经度
       a.bars_scan_lat, --54维度
       a.src_order_no, --订单号
       a.order_tm, --下单时间
       a.shipper_name, --寄件人信息
       a.shipper_phone, --寄件人电话
       a.shipper_mobile, --寄件人手机号
       a.shipper_comp_name, --寄件人公司名称
       nvl(b.dist_code ,a.shipper_city_code ) as shipper_city_code , --寄件人城市代码
       a.shipper_province_name, --寄件人省名
       a.shipper_city_name, --寄件人市名
       a.shipper_county_name, --寄件人乡镇名
       a.shipper_addr, --寄件人地址
       a.order_address_lng, --寄件人经度
       a.order_address_lat, --寄件人纬度
       a.order_address_precision, --精确地址-
       a.order_address_level, --地址类型
       a.bars_scan_lat_type,
       a.bars_scan_lat_type as bars_scan_lng_type,
       a.order_address_lat_type,
       a.order_address_lng_type,
       nvl(f.order_type_desc, a.order_type_code) as order_type_code, --来源渠道
       nvl(g.pickup_type_desc, a.pickup_type_code) as pickup_type_code, --寄件方式
       a.order_status_code, --订单状态
       a.distance,
-----是否异常 正常0异常1
case
--以下条件全部满足，则返回是,1、54GPS值不为0/null 2、订单经纬度不为0/null 3、订单地址精度=2 4、54 gps定位类型为gps定位或wifi定位 5、54到寄件地点距离<=0.5公里
when a.distance is not null and --距离不为空表示订单和54经纬度都有效
              a.distance <= 500 
--add by 681392  非自寄件
and m.waybill_no is  null 
then
0
--订单地址经纬度无效 or 54经纬度无效 or 订单城市代码<>54城市代码 or 54到寄件地点距离 >30公里  一个无效就是“无效”
when bars_scan_lat_type = 0 or order_address_lat_type = 0 or
              distance > 30000 or (e.dist_code <> shipper_city_code and
              shipper_city_code is not null) or 
              m.waybill_no is not  null               then
2
else
1
end as is_54_pickup_usual,      
case
when a.distance is not null and a.distance <= 500  and m.waybill_no is null  then
'合规'
--add by 681392 自寄件
when m.waybill_no is not null then  '自寄件'
when a.bars_scan_lat_type = 0 and a.order_address_lat_type = 0 then
'地址经纬度与54（上门收件）gps皆无效'  -- modify 20180816 by 80002542
when a.order_address_lat_type = 0  or  (a.distance is not null and  distance > 30000 )then
'订单地址经纬度无效'
when a.bars_scan_lat_type = 0 then
'54（上门收件）gps无效' -- modify 20180816 by 80002542         
when a.distance is not null and a.distance > 500 and
              a.dept_54_dist is not null and a.dept_54_dist <= 300 then
'在网点做54（上门收件）' -- modify 20180816 by 80002542
when a.distance is not null and a.distance > 500 and
              ((a.dept_54_dist is not null and a.dept_54_dist > 300) or (a.dept_54_dist is null ) )and
              (e.dist_code = b.dist_code or b.dist_code is null or
              e.dist_code is null) then
'未在客户处做54（上门收件）' -- modify 20180816 by 80002542
when a.distance is not null and a.distance > 500 and
              ((a.dept_54_dist is not null and a.dept_54_dist > 300) or (a.dept_54_dist is null ) ) and
              (e.dist_code <> b.dist_code and e.dist_code is not null and
              b.dist_code is not null) then
'异地寄件'
end as pickup_unusual_reason,
       a.is_sch_order,
       a.load_tm, --跑数时间
       a.eventgpstime, -- gps定位时间
       a.dept_longitude,
       a.dept_latitude,
       a.dept_54_dist,
       e.stat_dept_code as emp_dept_code, --员工所属网点
       a.eventaccuracy, --54gps精度
       a.valid_flag, --读取gis返回的是否有效地址类型。1代表有效，0代表无效
       a.geo_src, --读取gis返回的地理编码坐标来源信息，显示123即可
       e.hq_code,
       e.hq_name,
       e.area_code,
       e.area_name,
       e.division_code,
       e.division_name,
       e.dept_name,
       a.adcode,
       e.dist_code as city_code ,
       a.eventpositionmethod,
       a.base_id,
       a.group_id,
       a.all_unit_arear_code,      
--add by 681392 是否自寄件
case when m.waybill_no is not  null then 1 else  0 end as is_self_send ,
       a.inc_day
from tmp_dm_next.pickup_standard_dtl_20181101 a 
left join (select emp_code,dept_code,emp_name from gdl.tt_emp_info  where inc_day = '20181101') d on cast(a.emp_code as int)= cast(d.emp_code as int)
left join (select dept_code,dept_name,division_code,division_name,area_code,area_name,hq_code,hq_name,dist_code, stat_dept_code  from gdl.zipper_dim_department t where t.dw_start_date<='20181101' and t.dw_end_date >='20181101') e on nvl(d.dept_code,a.dept_code ) = e.dept_code
left join  dim.dim_order_type f on a.src_sys_type=f.src_sys_type and a.order_type_code = f.order_type_code
left join  dim.dim_pickup_type g on a.src_sys_type=g.src_sys_type  and a.pickup_type_code=g.pickup_type_code
left join (select dept_code,dist_code from gdl.zipper_dim_department t where t.dw_start_date<='20181101' and t.dw_end_date >='20181101') b on a.shipper_city_code  = b.dept_code
--add by 681392 是否自寄件
left join (  select mailno as waybill_no,self_send_flg  from  ods_inc_sgs_core.tt_receive_order_explode t where t.inc_day ='20181101'and self_send_flg ='true' group by   mailno,self_send_flg  ) m 
on a.waybill_no = m.waybill_no
;
```

```
STAGE DEPENDENCIES:
  Stage-1 is a root stage
  Stage-2 depends on stages: Stage-1
  Stage-0 depends on stages: Stage-2
  Stage-3 depends on stages: Stage-0
STAGE PLANS:
  Stage: Stage-1
    Tez
      DagId: hive_20181105204828_6538ed3a-0470-4576-8fae-cf28076c710b:2
      Edges:
        Reducer 10 <- Map 9 (SIMPLE_EDGE)
        Reducer 2 <- Map 1 (SIMPLE_EDGE), Map 4 (SIMPLE_EDGE), Map 5 (BROADCAST_EDGE), Map 6 (BROADCAST_EDGE), Map 7 (BROADCAST_EDGE), Map 8 (BROADCAST_EDGE)
        Reducer 3 <- Reducer 10 (SIMPLE_EDGE), Reducer 2 (SIMPLE_EDGE)
      DagName: tmp_datascience_fake_occupy
      Vertices:
        Map 1 
            Map Operator Tree:
                TableScan
                  alias: a
                  Statistics: Num rows: 9941795 Data size: 447380775 Basic stats: COMPLETE Column stats: NONE
                  Select Operator
                    expressions: waybill_no (type: string),... all_unit_arear_code (type: string), inc_day (type: string)
                    outputColumnNames: _col0, ..., _col43
                    Statistics: Num rows: 9941795 Data size: 447380775 Basic stats: COMPLETE Column stats: NONE
                    Reduce Output Operator
                      key expressions: UDFToInteger(_col2) (type: int)
sort order: +
                      Map-reduce partition columns: UDFToInteger(_col2) (type: int)
                      Statistics: Num rows: 9941795 Data size: 447380775 Basic stats: COMPLETE Column stats: NONE
                      value expressions: _col0 (type: string),..., _col43 (type: string)
        Map 4 
            Map Operator Tree:
                TableScan
                  alias: tt_emp_info
                  Statistics: Num rows: 1864525 Data size: 275949700 Basic stats: COMPLETE Column stats: NONE
                  Select Operator
                    expressions: emp_code (type: string), dept_code (type: string), emp_name (type: string)
                    outputColumnNames: _col0, _col1, _col2
                    Statistics: Num rows: 1864525 Data size: 275949700 Basic stats: COMPLETE Column stats: NONE
                    Reduce Output Operator
                      key expressions: UDFToInteger(_col0) (type: int)
sort order: +
                      Map-reduce partition columns: UDFToInteger(_col0) (type: int)
                      Statistics: Num rows: 1864525 Data size: 275949700 Basic stats: COMPLETE Column stats: NONE
                      value expressions: _col1 (type: string), _col2 (type: string)
        Map 5 
            Map Operator Tree:
                TableScan
                  alias: t
                  Statistics: Num rows: 312749 Data size: 14699203 Basic stats: COMPLETE Column stats: NONE
                  Filter Operator
                    predicate: ((dw_start_date <= '20181101') and (dw_end_date >= '20181101')) (type: boolean)
                    Statistics: Num rows: 34749 Data size: 1633203 Basic stats: COMPLETE Column stats: NONE
                    Select Operator
                      expressions: dept_code (type: string), dept_name (type: string), division_code (type: string), division_name (type: string), area_code (type: string), area_name (type: string), hq_code (type: string), hq_name (type: string), dist_code (type: string), stat_dept_code (type: string)
                      outputColumnNames: _col0, _col1, _col2, _col3, _col4, _col5, _col6, _col7, _col8, _col9
                      Statistics: Num rows: 34749 Data size: 1633203 Basic stats: COMPLETE Column stats: NONE
                      Reduce Output Operator
                        key expressions: _col0 (type: string)
sort order: +
                        Map-reduce partition columns: _col0 (type: string)
                        Statistics: Num rows: 34749 Data size: 1633203 Basic stats: COMPLETE Column stats: NONE
                        value expressions: _col1 (type: string), _col2 (type: string), _col3 (type: string), _col4 (type: string), _col5 (type: string), _col6 (type: string), _col7 (type: string), _col8 (type: string), _col9 (type: string)
        Map 6 
            Map Operator Tree:
                TableScan
                  alias: f
                  Statistics: Num rows: 69 Data size: 276 Basic stats: COMPLETE Column stats: NONE
                  Select Operator
                    expressions: src_sys_type (type: int), order_type_code (type: string), order_type_desc (type: string)
                    outputColumnNames: _col0, _col1, _col2
                    Statistics: Num rows: 69 Data size: 276 Basic stats: COMPLETE Column stats: NONE
                    Reduce Output Operator
                      key expressions: _col0 (type: int), _col1 (type: string)
sort order: ++
                      Map-reduce partition columns: _col0 (type: int), _col1 (type: string)
                      Statistics: Num rows: 69 Data size: 276 Basic stats: COMPLETE Column stats: NONE
                      value expressions: _col2 (type: string)
        Map 7 
            Map Operator Tree:
                TableScan
                  alias: g
                  Statistics: Num rows: 25 Data size: 75 Basic stats: COMPLETE Column stats: NONE
                  Select Operator
                    expressions: src_sys_type (type: int), pickup_type_code (type: string), pickup_type_desc (type: string)
                    outputColumnNames: _col0, _col1, _col2
                    Statistics: Num rows: 25 Data size: 75 Basic stats: COMPLETE Column stats: NONE
                    Reduce Output Operator
                      key expressions: _col0 (type: int), _col1 (type: string)
sort order: ++
                      Map-reduce partition columns: _col0 (type: int), _col1 (type: string)
                      Statistics: Num rows: 25 Data size: 75 Basic stats: COMPLETE Column stats: NONE
                      value expressions: _col2 (type: string)
        Map 8 
            Map Operator Tree:
                TableScan
                  alias: t
                  Statistics: Num rows: 312749 Data size: 14699203 Basic stats: COMPLETE Column stats: NONE
                  Filter Operator
                    predicate: ((dw_start_date <= '20181101') and (dw_end_date >= '20181101')) (type: boolean)
                    Statistics: Num rows: 34749 Data size: 1633203 Basic stats: COMPLETE Column stats: NONE
                    Select Operator
                      expressions: dept_code (type: string), dist_code (type: string)
                      outputColumnNames: _col0, _col1
                      Statistics: Num rows: 34749 Data size: 1633203 Basic stats: COMPLETE Column stats: NONE
                      Reduce Output Operator
                        key expressions: _col0 (type: string)
sort order: +
                        Map-reduce partition columns: _col0 (type: string)
                        Statistics: Num rows: 34749 Data size: 1633203 Basic stats: COMPLETE Column stats: NONE
                        value expressions: _col1 (type: string)
        Map 9 
            Map Operator Tree:
                TableScan
                  alias: t
                  Statistics: Num rows: 10425884 Data size: 8012083378 Basic stats: COMPLETE Column stats: NONE
                  Filter Operator
                    predicate: (self_send_flg = 'true') (type: boolean)
                    Statistics: Num rows: 5212942 Data size: 4006041689 Basic stats: COMPLETE Column stats: NONE
                    Select Operator
                      expressions: mailno (type: string)
                      outputColumnNames: mailno
                      Statistics: Num rows: 5212942 Data size: 4006041689 Basic stats: COMPLETE Column stats: NONE
                      Group By Operator
                        keys: mailno (type: string)
mode: hash
                        outputColumnNames: _col0
                        Statistics: Num rows: 5212942 Data size: 4006041689 Basic stats: COMPLETE Column stats: NONE
                        Reduce Output Operator
                          key expressions: _col0 (type: string)
sort order: +
                          Map-reduce partition columns: _col0 (type: string)
                          Statistics: Num rows: 5212942 Data size: 4006041689 Basic stats: COMPLETE Column stats: NONE
        Reducer 10 
            Reduce Operator Tree:
              Group By Operator
                keys: KEY._col0 (type: string)
mode: mergepartial
                outputColumnNames: _col0
                Statistics: Num rows: 2606471 Data size: 2003020844 Basic stats: COMPLETE Column stats: NONE
                Reduce Output Operator
                  key expressions: _col0 (type: string)
sort order: +
                  Map-reduce partition columns: _col0 (type: string)
                  Statistics: Num rows: 2606471 Data size: 2003020844 Basic stats: COMPLETE Column stats: NONE
        Reducer 2 
            Reduce Operator Tree:
              Merge Join Operator
                condition map:
                     Left Outer Join0 to 1
                keys:
0 UDFToInteger(_col2) (type: int)
1 UDFToInteger(_col0) (type: int)
                outputColumnNames: _col0, ..._col46
                Statistics: Num rows: 10935974 Data size: 492118863 Basic stats: COMPLETE Column stats: NONE
                Map Join Operator
                  condition map:
                       Left Outer Join0 to 1
                  keys:
0 NVL(_col45,_col1) (type: string)
1 _col0 (type: string)
                  outputColumnNames: _col0,... _col56
input vertices:
1 Map 5
                  Statistics: Num rows: 12029571 Data size: 541330761 Basic stats: COMPLETE Column stats: NONE
                  HybridGraceHashJoin: true
                  Map Join Operator
                    condition map:
                         Left Outer Join0 to 1
                    keys:
0 _col37 (type: int), _col24 (type: string)
1 _col0 (type: int), _col1 (type: string)
                    outputColumnNames: _col0, ..._col59
input vertices:
1 Map 6
                    Statistics: Num rows: 13232528 Data size: 595463850 Basic stats: COMPLETE Column stats: NONE
                    HybridGraceHashJoin: true
                    Map Join Operator
                      condition map:
                           Left Outer Join0 to 1
                      keys:
0 _col37 (type: int), _col25 (type: string)
1 _col0 (type: int), _col1 (type: string)
                      outputColumnNames: _col0, ..._col62
input vertices:
1 Map 7
                      Statistics: Num rows: 14555781 Data size: 655010249 Basic stats: COMPLETE Column stats: NONE
                      HybridGraceHashJoin: true
                      Map Join Operator
                        condition map:
                             Left Outer Join0 to 1
                        keys:
0 _col12 (type: string)
1 _col0 (type: string)
                        outputColumnNames: _col0, ... _col64
input vertices:
1 Map 8
                        Statistics: Num rows: 16011359 Data size: 720511289 Basic stats: COMPLETE Column stats: NONE
                        HybridGraceHashJoin: true
                        Reduce Output Operator
                          key expressions: _col0 (type: string)
sort order: +
                          Map-reduce partition columns: _col0 (type: string)
                          Statistics: Num rows: 16011359 Data size: 720511289 Basic stats: COMPLETE Column stats: NONE
                          value expressions: _col1 (type: string),..., _col64 (type: string)
        Reducer 3 
            Reduce Operator Tree:
              Merge Join Operator
                condition map:
                     Left Outer Join0 to 1
                keys:
0 _col0 (type: string)
1 _col0 (type: string)
                outputColumnNames: _col0... _col65
                Statistics: Num rows: 17612495 Data size: 792562435 Basic stats: COMPLETE Column stats: NONE
                Select Operator
                  expressions: _col0 (type: string)... _col43 (type: string)
                  outputColumnNames: _col0... _col56
                  Statistics: Num rows: 17612495 Data size: 792562435 Basic stats: COMPLETE Column stats: NONE
                  File Output Operator
                    compressed: false
                    Statistics: Num rows: 17612495 Data size: 792562435 Basic stats: COMPLETE Column stats: NONE
                    table:
input format: org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat
                        output format: org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat
                        serde: org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe
                        name: dm_next.pickup_standard_dtl
  Stage: Stage-2
    Dependency Collection
  Stage: Stage-0
    Move Operator
      tables:
          partition:
            inc_day 
          replace: true
          table:
input format: org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat
              output format: org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat
              serde: org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe
              name: dm_next.pickup_standard_dtl
  Stage: Stage-3
    Stats-Aggr Operator
```

### HOW TO READ MAPREDUCE EXECUTION PLAN <a href="#how-to-read-mapreduce-execution-plan" id="how-to-read-mapreduce-execution-plan"></a>

### MAPREDUCE EXAMPLES. COMPARE <a href="#mapreduce-examples-compare" id="mapreduce-examples-compare"></a>

`explain select count(waybill_no) from gdl.tt_waybill_info where inc_day=20181104 ;`

```
STAGE DEPENDENCIES:
  Stage-1 is a root stage
  Stage-0 depends on stages: Stage-1
STAGE PLANS:
  Stage: Stage-1
    Map Reduce
      Map Operator Tree:
          TableScan
            alias: tt_waybill_info
            Statistics: Num rows: 6695357 Data size: 1225250331 Basic stats: COMPLETE Column stats: NONE
Select Operator
              expressions: waybill_no (type: string)
              outputColumnNames: waybill_no
Statistics: Num rows: 6695357 Data size: 1225250331 Basic stats: COMPLETE Column stats: NONE
Group By Operator
                aggregations: count(waybill_no)
mode: hash
                outputColumnNames: _col0
Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
                Reduce Output Operator
sort order: 
Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
value expressions: _col0 (type: bigint)
      Reduce Operator Tree:
Group By Operator
          aggregations: count(VALUE._col0)
mode: mergepartial
          outputColumnNames: _col0
Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
File Output Operator
            compressed: false
Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
table:
input format: org.apache.hadoop.mapred.SequenceFileInputFormat
output format: org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat
                serde: org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
  Stage: Stage-0
Fetch Operator
limit: -1
      Processor Tree:
        ListSink
```

`explain select count(distinct waybill_no) from gdl.tt_waybill_info where inc_day=20181104 ;`

```
STAGE DEPENDENCIES:
  Stage-1 is a root stage
  Stage-0 depends on stages: Stage-1
STAGE PLANS:
  Stage: Stage-1
    Map Reduce
      Map Operator Tree:
          TableScan
            alias: tt_waybill_info
            Statistics: Num rows: 6695357 Data size: 1225250331 Basic stats: COMPLETE Column stats: NONE
Select Operator
              expressions: waybill_no (type: string)
              outputColumnNames: waybill_no
Statistics: Num rows: 6695357 Data size: 1225250331 Basic stats: COMPLETE Column stats: NONE
Group By Operator
                aggregations: count(DISTINCT waybill_no)
keys: waybill_no (type: string)
mode: hash
                outputColumnNames: _col0, _col1
Statistics: Num rows: 6695357 Data size: 1225250331 Basic stats: COMPLETE Column stats: NONE
                Reduce Output Operator
key expressions: _col0 (type: string)
sort order: +
Statistics: Num rows: 6695357 Data size: 1225250331 Basic stats: COMPLETE Column stats: NONE
      Reduce Operator Tree:
Group By Operator
          aggregations: count(DISTINCT KEY._col0:0._col0)
mode: mergepartial
          outputColumnNames: _col0
Statistics: Num rows: 1 Data size: 16 Basic stats: COMPLETE Column stats: NONE
File Output Operator
            compressed: false
Statistics: Num rows: 1 Data size: 16 Basic stats: COMPLETE Column stats: NONE
table:
input format: org.apache.hadoop.mapred.SequenceFileInputFormat
output format: org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat
                serde: org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
  Stage: Stage-0
Fetch Operator
limit: -1
      Processor Tree:
        ListSink
```
