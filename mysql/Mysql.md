# 1、使用自增主键增长到最大值后会怎么样？
- 当使用int或者bigint作为自增主键时，当达到最大值时，下次在
申请的主键还是最大值，那么在插入的时候就会报主键冲突。
- 当没有指定自增主键时，mysql默认会为我们生成一个row_id作为主键，row_id的最大值为2^48 - 1
当达到最大值后，再次申请的row_id会从0再次开始循环，并且数据会覆盖之前已存在的数据，不会报错
（这就会导致数据丢失）

# 2、采用主从架构时，如果解决从库读取延迟问题？
- 最简单的方式，对于需要马上获取到最新的结果的数据，直接走主库查询，不走从库查询
- 判断主从无延迟方案：只能报错数据发送到从库，从库已经执行完成的情况，但是对于主库执行完成后，还未将binlog进行同步的情况
还是存在延迟读问题。
  - 每次在从库执行查询语句前，执行命令show slave status,查看seconds_behind_master是否为0，如果为0无延迟
  - 对比位点确保没有主备延迟：通过Master_Log_File（主日志文件）Read_Master_Log_Pos（读到的主库最新位点）和Relay_Master_Log_File（从库接受到的主库日志文件）Exec_Master_Log_Pos
（从库读到的最新位点）；Master_Log_File 和 Relay_Master_Log_File、Read_Master_Log_Pos 和 Exec_Master_Log_Pos 这两组值完全相同，表示没有延迟。
  - 对比GTID集合，首先需要开启了GTID，在show slave status中Auto_Position=1表示开启了GTID，判断Retrieved_Gtid_Set（备库收到的GTID集合）
    Executed_Gtid_Set（备库已经执行的GTID集合）相等，无延迟
- 半同步机制semi-sync：当一个更新在主库完成后，将binlog发送给从库，从库收到binlog日志后，回复一个ack给主库，主库才认为
这次更新完成。通过semi-sync + 位点判断，解决延迟读取问题。
- 等主库位点：通过如下命令,
  - 当一个更新事务在主库上完成后，执行show master status 得到当前主库执行到的File和Position
  - 选择一个从库执行 select master_pos_wait(file, pos[, timeout]);
  - 如果返回值是 >=0 的正整数，则在这个从库执行查询语句
  - 不然就降级到主库上进行查询
```sql
# 在从库执行如下命令，其中file 和 pos 指的是主库上的日志文件名和位点，timeout可选值，语句执行等待时间
# 命令执行结果会
    · 正常返回一个正整数
    · 执行期间，备库同步线程发生异常，则返回 NULL
    · 等待超过 timeout 秒，就返回 -1
    · 刚开始执行的时候，就发现已经执行过这个位置了，则返回 0
select master_pos_wait(file, pos[, timeout]);

```
- GTID方案:前提是数据库已经开启了GTID
  - 在主库上执行一条更新事务后，获取到对应GTID值
  - 根据这个值，选择一个从库执行select wait_for_executed_gtid_set(gtid1, 1)；
  - 返回值为0，则在从库进行查询
  - 其他值，则到主库上进行查询
```sql
# 个库执行的事务中包含传入的 gtid_set，返回 0，否则超时返回1
# 在mysql5.7.6 版本之后，允许执行万更新语句后，返回对应的事务的GTID，
# 这样可以减少使用show master status命令
select wait_for_executed_gtid_set(gtid_set, 1);
```
# 3、如果表A两条数据，表B有两条数据，那么语句 select * from A left join B on A.param = B.param 可能有几条结果？
> 答案：2~4条
> 
> 2条的情况：如果表A和表B两个关联的数据不相等，或者刚好一一对应，那么返回的结果就是2条
> 
> 3条的情况：如果表A中的两个数据不一致，其中一条数据和表B刚好都匹配，则返回3条数据。
> 
> 4条的情况：如果表A中的数据相同并且都能和表B的数据都能匹配上，返回的结果为4条

# 4、如果表A两条数据，表B有两条数据，那么语句 select * from A inner join B on A.param = B.param 可能有几条结果？
> 答案:0、1、2、4条
> 
> 0条的情况： 如果A B关联的字段两个表的数据不相等。
> 
> 1条的情况：如果A B关联是字段刚好有一条数据A B字段相同。
> 
> 2条的情况：如果A B关联的字段刚好两张表字段一一对应，或者A中一条数据对应B中两条数据
> 
> 4条的情况：如果A的关联字段值相同，切和B中两条数据都相同，则返回4条。

# 5、