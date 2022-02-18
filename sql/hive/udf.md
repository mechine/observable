# udf

## UDF开发

Differences Between UDF, UDAF and UDTF:[UDF:](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)UDF is a user-defined function that takes a single input value and produces a single output value. When used in a query, we can call it once for each row in the result set.Example:input.toString().toUpperCase();input.toString().toLowerCase();The above methods will convert a string of lowercase to uppercase and vice versa.[UDAF:](https://cwiki.apache.org/confluence/display/Hive/GenericUDAFCaseStudy)UDAF is a user-defined aggregate function (UDAF) that accepts a group of values and returns a single value. Users can implement UDAFs to summarize and condense sets of rows in the same style as the built-in COUNT, MAX(), SUM(), and AVG() functions.

| Function                | Purpose                                                                                                                                                                                                                                                                                                                                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| init                    | Called by Hive to initialize an instance of your UDAF evaluator class.                                                                                                                                                                                                                                                                                                                         |
| 备注                      | 初始化evaluator                                                                                                                                                                                                                                                                                                                                                                                   |
| getNewAggregationBuffer | Return an object that will be used to store temporary aggregation results.                                                                                                                                                                                                                                                                                                                     |
| 备注                      | 返回一个对象用来存储聚合的临时结果                                                                                                                                                                                                                                                                                                                                                                              |
| iterate                 | Process a new row of data into the aggregation buffer                                                                                                                                                                                                                                                                                                                                          |
| 备注                      | 处理新的一行到上面提到那个对象中                                                                                                                                                                                                                                                                                                                                                                               |
| terminatePartial        | Return the contents of the current aggregation in a persistable way. Here persistable means the return value can only be built up in terms of Java primitives, arrays, primitive wrappers (e.g. Double), Hadoop Writables, Lists, and Maps. Do NOT use your own classes (even if they implement java.io.Serializable), otherwise you may get strange errors or (probably worse) wrong results. |
| 备注                      | 返回的结果会被持久化，只能使用java的基本类型或者hadoop的基本类型，否则可能出现其他异常，即使你的类实现了序列化接口也没用…                                                                                                                                                                                                                                                                                                                             |
| merge                   | Merge a partial aggregation returned by terminatePartial into the current aggregation                                                                                                                                                                                                                                                                                                          |
| 备注                      | 合并结果集                                                                                                                                                                                                                                                                                                                                                                                          |
| terminate               | Return the final result of the aggregation to Hive                                                                                                                                                                                                                                                                                                                                             |
| 备注                      | 将结果返回给hive                                                                                                                                                                                                                                                                                                                                                                                     |

[UDTF:](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide+UDTF)UDTF is a User Defined Table Generating Function that operates on a single row and produces multiple rows a table as output.select explode\_map(properties) as (col1,col2) from src;

> 直接select中使用

select a, explode\_map(properties) as (col1,col2) from src;

> 不可以添加其他字段使用

select explode\_map(explode\_map(properties)) from src;

> 不可以嵌套调用

select explode\_map(properties) as (col1,col2) from src group by col1, col2;

> 不可以和group by/cluster by/distribute by/sort by一起使用

select src.id, mytable.col1, mytable.col2 from src lateral view explode\_map(properties) mytable as col1, col2;

> 此方法更为方便日常使用。执行过程相当于单独执行了两次抽取，然后union到一个表里。
