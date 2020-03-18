[TOC]

# 1. 什么是事务？

​	事务是逻辑上的一组操作，是执行的最小单位，要么都执行，要么都不执行。

# 2. ACID？

**原子性**（Atomicity）：事务是执行的最小单位，要么做，要么不做。

**一致性**（Consistency）：事务执行前后，数据保持一致，多个事务对同一数据读取结果相同

**隔离性**（Isolation）：一个事务执行不被其他事务干扰

**持久性**（Durability）：事务一旦被提交，永久修改。

# 3. 并发事务带来哪些问题？如何解决？

事务A和事务B都在对同一数据做修改：

​        **丢失修改**：事务A进行更新，事务B也进行更新，造成覆盖。

事务A改，事务B读：

​        **脏读**：事务A读取数据并进行修改，但未提交，事务B也读取数据，此时读取的就是“脏数据”

​        **不可重复读**：事务B多次读取同一数据不一致，可能读到事务A修改提交后的数据。

​        **幻读**：事务B多次读取的某一条件数据不一致，可能事务A在进行新增或者删除操作。

测试【开两个mysql连接】

① 脏读

```mysql
// INNODB默认隔离级别是  可重复
SELECT @@TRANSACTION_ISOLATION;
// 修改为  读未提交【最低隔离级别】 
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

// 事务1										// 事务2
START TRANSACTION;
SELECT * FROM EMPLOYEE WHERE ID=1;[1000]
											START TRANSACTION;
											SELECT * FROM EMPLOYEE WHERE ID=1;[1000]
											UPDATE EMPLOYEE SET SALARY=500 WHERE ID=1; 
SELECT * FROM EMPLOYEE WHERE ID=1;[500]

------------------------
解决办法：READ COMMITTED
```

② 不可重复读

```mysql
// 修改为  读已提交 
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;

// 事务1										// 事务2
START TRANSACTION;
SELECT * FROM EMPLOYEE WHERE ID=1;[1000]
											START TRANSACTION;
											SELECT * FROM EMPLOYEE WHERE ID=1;[1000]
											UPDATE EMPLOYEE SET SALARY=500 WHERE ID=1; 
											COMMIT;
SELECT * FROM EMPLOYEE WHERE ID=1;[500]

------------------------
解决办法：REPEATABLE READ
```

③ 幻读

```mysql
// InnoDB的读已提交已经解决了幻读问题，所以用MyISAM测试
show table status from ssm where name='employee';
alter table employee engine=MyISAM;

// 修改为  读已提交 
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;

// 事务1									// 事务2
START TRANSACTION;
SELECT COUNT(*) FROM EMPLOYEE WHERE SALARY>1000;[4]
										START TRANSACTION;
										SELECT COUNT(*) FROM EMPLOYEE WHERE SALARY>1000;[4]
										INSERT EMPLOYEE VALUES(20,1500); 
										COMMIT;
SELECT COUNT(*) FROM EMPLOYEE WHERE SALARY>1000;[5]

------------------------
解决办法：InnoDB，其他引擎需要设置隔离级别为SERIALIZABLE
```

# 4.什么是视图？有什么作用？

视图是一个虚拟表，具有表结构，但不存在数据文件，数据是在查询视图时动态查询视图所引用的表来返回。

```mysql
//创建 CREATE VIEW myView AS SELECT name,age FROM employee;
//使用 SELECT * FROM myView;
```

作用：简化业务逻辑；对客户端隐藏真实的表结构

# 5.什么是第一范式？第二范式？第三范式？

第一范式（1NF）：每一列都是不可分割的基本数据项。

第二范式（2NF）：每一行可以被唯一区分，并且实体的非主属性必须完全依赖于主关键字。

第三范式（3NF）：数据表中不包含已经在其他表中包含的非主关键字信息。（如employee表只存dep_id即可，不需要再存dep的其他信息）