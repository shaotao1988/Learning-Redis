数据库事务
========

数据库事务的目的
-------------

1. 为数据库操作序列提供从失败中恢复到正常状态的方法
2. 当多个应用并发访问数据库时，提供隔离机制防止多个应用之间的操作相互干扰  

数据库事务的ACID特性
-----------------

1. Atomicity 原子性：事务作为一个整体，要么全被执行，要么全不执行
2. Consistency 一致性：数据库的状态必须从一个状态转换为另一个一致的状态。一致状态的含义是数据库中的数据满足完整性约束。
3. Isolated 隔离性： 多个事务并发执行时，相互之间互不干扰
4. Durablity 持久性： 已提交的事务对数据库的更改应永久保存在数据库中

事务隔离性的4个级别
----------------

1. Read uncommitted 未授权读，该级别允许读取未提交的数据（脏读）。
    eg：如果一个事务A修改了一条记录，但并未提交，此时事务B读取记录r，如果事务A因某种原因回滚了，那么事务B就读取了一条无效的记录。
2. Read committed 授权读，也叫只读已提交数据，不允许脏读。
    eg：如果事务A读取一条记录r1，并修改为r2，事务A尚未提交时，事务B读取该记录，读取到的是原来的r1，只有当事务A提交以后，B读取到的才是r2.即不会读中间数据，只读已提交数据。
    授权读是不可重复读的，比如事务B在事务A提交前和提交后分别读取记录r，读取到的是2个不同的结果r1和r2.
3. Repeatable read 可重复读，在一个事务中多次读取同一条记录时，读取结果相同。当事务A读取记录时，需要对该记录加锁，不允许其他事务修改该条记录。但是还是无法解决幻读的问题，即事务A读取与搜索条件项匹配的若干行，A提交之前，事务B以插入或删除行等方式修改事务A的结果集，然后再提交。当事务A重复读取时，却发现数据发生了变化，造成幻读。
4. Serializable 序列化，最高的事务隔离级别，在该级别下，事务只能顺序执行。

悲观锁与乐观锁
------------

悲观锁和乐观锁是并发控制采取的主要技术手段，是一种思想，在各种并发控制系统中广泛使用。  
在DBMS系统中，它们和数据库提供的锁机制（行锁、表锁、排他锁、共享锁）是不同的概念，事实上悲观锁正是利用数据库本身提供的锁机制实现的。  

### 悲观锁

悲观锁，指的是对数据被外界（包括本系统其它事务，以及其它系统的事务）修改持悲观态度，即数据被并发修改的概率很大，主要用于数据争用激烈，或者事务回滚成本比加锁保护成本高的场景。  
悲观锁的流程如下：

1. 操作任意记录前，先尝试为记录加上排他锁
2. 如果加锁失败，说明该记录正在被其他事务操作，那么当前事务需要等待或抛出异常
3. 如果加锁成功，则对记录做操作，事务完成后解锁。如果期间有其它事务对该记录做操作，加锁时都会失败。

缺点：加锁会产生额外开销，并增加死锁的机会；另外只读型事务不会产生冲突，没必要加锁。

### 乐观锁

乐观锁相对悲观锁而言，认为数据一般情况下不会有冲突，所以只在数据进行提交更新的时候，才会正式对数据的冲突与否进行检查，如果发现冲突，再进行回滚。  

相对悲观锁，乐观锁在实现时不会使用数据库的锁机制，一般实现乐观锁的方式是记录数据版本号或打时间戳。  
>数据版本：为数据增加的一个版本标识。当读取数据时，将版本标识一起读出，数据每更新一次，同时对版本标识进行更新。当提交更新时，判断数据库表对应记录的当前版本与第一次读取出来的版本标识是否一致，如果一致则予以更新，不一致则认为是过期数据。  