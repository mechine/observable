# sqlite-wal

**1.什么是WAL？**

&#x20;     WAL的全称是Write Ahead Logging，它是很多数据库中用于实现原子事务的一种机制，SQLite在3.7.0版本引入了该特性。

&#x20;     **2.WAL如何工作？**

&#x20;     ****      在引入WAL机制之前，SQLite使用rollback journal机制实现原子事务。

&#x20;     rollback journal机制的原理是：在修改数据库文件中的数据之前，先将修改所在分页中的数据备份在另外一个地方，然后才将修改写入到数据库文件中；如果事务失败，则将备份数据拷贝回来，撤销修改；如果事务成功，则删除备份数据，提交修改。

&#x20;     WAL机制的原理是：修改并不直接写入到数据库文件中，而是写入到另外一个称为WAL的文件中；如果事务失败，WAL中的记录会被忽略，撤销修改；如果事务成功，它将在随后的某个时间被写回到数据库文件中，提交修改。

&#x20;     同步WAL文件和数据库文件的行为被称为checkpoint（检查点），它由SQLite自动执行，默认是在WAL文件积累到1000页修改的时候；当然，在适当的时候，也可以手动执行checkpoint，SQLite提供了相关的接口。执行checkpoint之后，WAL文件会被清空。

&#x20;     在读的时候，SQLite将在WAL文件中搜索，找到最后一个写入点，记住它，并忽略在此之后的写入点（这保证了读写和读读可以并行执行）；随后，它确定所要读的数据所在页是否在WAL文件中，如果在，则读WAL文件中的数据，如果不在，则直接读数据库文件中的数据。

&#x20;     在写的时候，SQLite将之写入到WAL文件中即可，但是必须保证独占写入，因此写写之间不能并行执行。

&#x20;     WAL在实现的过程中，使用了共享内存技术，因此，所有的读写进程必须在同一个机器上，否则，无法保证数据一致性。

&#x20;     **3.WAL的优点与缺点**

&#x20;     ****      优点：

&#x20;     1.读和写可以完全地并发执行，不会互相阻塞（但是写之间仍然不能并发）。

&#x20;     2.WAL在大多数情况下，拥有更好的性能（因为无需每次写入时都要写两个文件）。

&#x20;     3.磁盘I/O行为更容易被预测。

&#x20;     缺点：

&#x20;     1.访问数据库的所有程序必须在同一主机上，且支持共享内存技术。

&#x20;     2.每个数据库现在对应3个文件：\<yourdb>.db，\<yourdb>-wal，\<yourdb>-shm。

&#x20;     3.当写入数据达到GB级的时候，数据库性能将下降。

&#x20;     4.3.7.0之前的SQLite无法识别启用了WAL机制的数据库文件。

&#x20;     **4.WAL引入的兼容性问题**

&#x20;     在启用了WAL之后，数据库文件格式的版本号由1升级到了2，因此，3.7.0之前的SQLite无法识别启用了WAL机制的数据库文件。

&#x20;     禁用WAL会使数据库文件格式的版本号恢复到1，从而可以被SQLite 3.7.0之前的版本识别。

&#x20;     **5.WAL引入的性能问题**

&#x20;     ****      在一般情况下，WAL会提高SQLite的事务性能；但是在某些极端情况下，却会导致SQLite事务性能的下降。

&#x20;     1.在事务执行时间较长或者要修改的数据量达到GB级的时候，WAL文件会被占用，它会暂时阻止checkpoint的执行（checkpoint会清空WAL文件），这将导致WAL文件变得很大，增加寻址时间，最终导致读写性能的下降。

&#x20;     2.当checkpoint执行的时候，会降低当时的读写性能，因此，WAL可能会导致周期性的性能下降。

&#x20;     **6.与WAL相关的PRAGMA和接口**

&#x20;     ****      PRAGMA journal\_mode

&#x20;     PRAGMA wal\_checkpoint

&#x20;     PRAGMA wal\_autocheckpoint

&#x20;     sqlite3\_wal\_checkpoint

&#x20;     sqlite3\_wal\_autocheckpoint

&#x20;     sqlite3\_wal\_hook
