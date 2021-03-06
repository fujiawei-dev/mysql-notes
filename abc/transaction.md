---
date: 2020-10-12T10:41:51+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "数据库事务"  # 文章标题
url:  "posts/mysql/abc/transaction"  # 设置网页永久链接
tags: [ "mysql", "sql", "数据库理论" ]  # 标签
series: [ "MySQL 学习笔记"]  # 系列
categories: [ "学习笔记"]  # 分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

## 事务的概念及其4个特性

事务（Transaction）是一个操作序列。这些操作要么都做，要么都不做，是一个不可分割的工作单位。事务通常以 `BEGIN TRANSACTION` 开始，以 `COMMIT` 或 `ROLLBACK` 操作结束，`COMMIT` 即提交，提交事务中所有的操作、事务正常结束。`ROLLBACK `即回滚，撤销已做的所有操作，回滚到事务开始时的状态。事务是数据库系统区别于文件系统的重要特性之一。

对于事务可以举一个简单的例子：转账，有A和B两个用户，A用户转100到B用户，如下所示：

```
A：----> 支出100，则 A-100
B：----> 收到100，则 B+100
```

A--->B 转账

对应如下SQL语句：

```sql
update account
set money=money - 100
where name = 'A';

update account
set money=money + 100
where name = 'B';
```

事务有4个特性，一般都称之为ACID特性，简单记为原一隔持（谐音：愿意各吃，即愿意各吃各的），如下表所示：

![](../imgs/acid.png)

- 原子性：事务在逻辑上是不可分割的操作单元，其所有语句要么执行，要么都撤销执行。当每个事物运行结束时，可以选择“提交”所做的数据修改，并将这些修改永久应用到数据库中。
- 一致性：
- 隔离性：
- 持久性：

## 事务的常见分类

从事务理论的角度来看，可以把事务分为以下几种类型：

- 扁平事务（Flat Transactions）
- 带有保存点的扁平事务（Flat Transactions with Savepoints）
- 链事务（Chained Transactions）
- 嵌套事务（Nested Transactions）
- 分布式事务（Distributed Transactions）

### 扁平事务

是事务类型中最简单的一种，但是在实际生产环境中，这可能是使用最频繁的事务，在扁平事务中，所有操作都处于同一层次，其由 BEGIN WORK 开始，由 COMMIT WORK 或 ROLLBACK WORK 结束，其间的操作是原子的，要么都执行，要么都回滚，因此，扁平事务是应用程序成为原子操作的基本组成模块。扁平事务虽然简单，但是在实际环境中使用最为频繁，也正因为其简单，使用频繁，故每个数据库系统都实现了对扁平事务的支持。扁平事务的主要限制是不能提交或者回滚事务的某一部分，或分几个步骤提交。

保存点（Savepoint）用来通知事务系统应该记住事务当前的状态，以便当之后发生错误时，事务能回到保存点当时的状态。对于扁平的事务来说，隐式的设置了一个保存点，然而在整个事务中，只有这一个保存点，因此，回滚只能会滚到事务开始时的状态。

扁平事务一般有三种不同的结果：①事务成功完成。在平常应用中约占所有事务的96%。②应用程序要求停止事务。比如应用程序在捕获到异常时会回滚事务，约占事务的3%。③外界因素强制终止事务。如连接超时或连接断开，约占所有事务的1%。

### 带有保存点的扁平事务

除了支持扁平事务支持的操作外，还允许在事务执行过程中回滚到同一事务中较早的一个状态。这是因为某些事务可能在执行过程中出现的错误并不会导致所有的操作都无效，放弃整个事务不合乎要求，开销太大。

### 链事务

是指**一个事务由多个子事务链式组成**，它可以被视为保存点模式的一个变种。带有保存点的扁平事务，当发生系统崩溃时，所有的保存点都将消失，这意味着当进行恢复时，事务需要从开始处重新执行，而不能从最近的一个保存点继续执行。链事务的思想是：在提交一个事务时，释放不需要的数据对象，将必要的处理上下文隐式地传给下一个要开始的事务，前一个子事务的提交操作和下一个子事务的开始操作合并成一个原子操作，这意味着下一个事务将看到上一个事务的结果，就好像在一个事务中进行一样。这样，在提交子事务时就可以释放不需要的数据对象，而不必等到整个事务完成后才释放。

![](../imgs/chained_transactions.png)

链事务与带有保存点的扁平事务的不同之处体现在：

① 带有保存点的扁平事务能回滚到任意正确的保存点，而**链事务中的回滚仅限于当前事务**，即只能恢复到最近的一个保存点。

② 对于锁的处理，两者也不相同，链事务在执行 COMMIT 后即释放了当前所持有的锁，而带有保存点的扁平事务不影响迄今为止所持有的锁。

### 嵌套事务

是一个层次结构框架，由一个顶层事务（Top-Level Transaction）控制着各个层次的事务，顶层事务之下嵌套的事务被称为子事务（Subtransaction），其控制着每一个局部的变换，子事务本身也可以是嵌套事务。因此，嵌套事务的层次结构可以看成是一棵树。

### 分布式事务

通常是在一个分布式环境下运行的扁平事务，因此，需要根据数据所在位置访问网络中不同节点的数据库资源。例如，一个银行用户从招商银行的账户向工商银行的账户转账1000元，这里需要用到分布式事务，因为不能仅调用某一家银行的数据库就完成任务。

## XA 事务

XA（eXtended Architecture）是指由 X/Open 组织提出的分布式交易处理的规范。XA 是一个分布式事务协议，由 Tuxedo 提出，所以，分布式事务也称为 XA 事务。XA 协议主要定义了事务管理器（TM，Transaction Manager，协调者）和资源管理器（RM，Resource Manager，参与者）之间的接口。其中，资源管理器往往由数据库实现，例如 Oracle、DB2、MySQL，这些商业数据库都实现了 XA 接口，而事务管理器作为全局的调度者，负责各个本地资源的提交和回滚。XA 事务是基于两阶段提交（Two-phase Commit，2PC）协议实现的，可以保证数据的强一致性，许多分布式关系型数据管理系统都采用此协议来完成分布式。阶段一为准备阶段，即所有的参与者准备执行事务并锁住需要的资源。当参与者 Ready 时，向 TM 汇报自己已经准备好。阶段二为提交阶段。当 TM 确认所有参与者都 Ready 后，向所有参与者发送 COMMIT 命令。

XA 事务允许不同数据库的分布式事务，只要参与在全局事务中的每个节点都支持 XA 事务。Oracle、MySQL和 SQL Server 都支持 XA 事务。

- XA 事务由一个或多个资源管理器（RM）、一个事务管理器（TM）以及一个应用程序（Application Program）组成。
- 资源管理器：提供访问事务资源的方法。通常一个数据库就是一个资源管理器。
- 事务管理器：协调参与全局事务中的各个事务。需要和参与全局事务的所有资源管理器进行通信。
- 应用程序：定义事务的边界。

XA 事务的缺点是性能不佳，且 XA 无法满足高并发场景。一个数据库的事务和多个数据库间的 XA 事务性能会相差很多。因此，要尽量避免使用 XA 事务，例如可以将数据写入本地，用高性能的消息系统分发数据，或使用数据库复制等技术。只有在其他办法都无法实现业务需求，且性能不是瓶颈时才使用 XA。

## 事务的4种隔离级别（Isolation Level）

当多个线程都开启事务操作数据库中的数据时，数据库系统要能进行隔离操作，以保证各个线程获取数据的准确性，所以，对于不同的事务，采用不同的隔离级别会有不同的结果。如果不考虑事务的隔离性，那么会发生下表所示的3种问题：

![](../imgs/isolation.png)

脏读和不可重复读的区别为：脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是在同一个事务范围内多次查询同一条数据却返回了不同的数据值，这是由于在查询间隔期间，该条数据被另一个事务修改并提交了。

幻读和不可重复读的区别为：幻读和不可重复读都是读取了另一个事务中已经提交的数据，不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一个数据整体（例如数据的条数）。

在SQL标准中定义了4种隔离级别，每一种级别都规定了一个事务中所做的修改，哪些是在事务内和事务间可见的，哪些是不可见的。较低级别的隔离通常可以执行更高的并发，系统的开销也更低。SQL标准定义的四个隔离级别为：Read Uncommitted（未提交读）、Read Committed（提交读）、Repeatable Read（可重复读）、Serializable（可串行化）。

![](../imgs/isolation_level.png)

### Read Uncommitted（未提交读，读取未提交内容）

在该隔离级别，所有事务都可以看到其他未提交事务的执行结果，即在未提交读级别，事务中的修改，即使没有提交，对其他事务也都是可见的，该隔离级别很少用于实际应用。读取未提交的数据，也被称之为脏读（Dirty Read）。该隔离级别最低，并发性能最高。

### Read Committed（提交读，读取提交内容）

这是大多数数据库系统的默认隔离级别。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。换句话说，一个事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的。

### Repeatable Read（可重复读）

可重复读可以确保同一个事务，在多次读取同样数据的时候，得到同样的结果。可重复读解决了脏读的问题，不过理论上，这会导致另一个棘手的问题：幻读（Phantom Read）。MySQL数据库中的InnoDB和Falcon存储引擎通过MVCC（Multi-Version Concurrent Control，多版本并发控制）机制解决了该问题。需要注意的是，多版本只是解决不可重复读问题，而加上间隙锁（也就是它这里所谓的并发控制）才解决了幻读问题。

### Serializable（可串行化、序列化）

这是最高的隔离级别，它通过强制事务排序，强制事务串行执行，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能出现大量的超时现象和锁竞争。实际应用中也很少用到这个隔离级别，只有在非常需要确保数据的一致性而且可以接受没有并发的情况下，才考虑用该级别。这是花费代价最高但是最可靠的事务隔离级别。

不同的隔离级别有不同的现象，并有不同的锁和并发机制，隔离级别越高，数据库的并发性能就越差。

## Oracle、MySQL 和 SQL Server 中的事务隔离级别

![](../imgs/oracle_mysql_sqlserver_isolation.png)

![](../imgs/oracle_mysql_sqlserver_isolation2.png)

### Oracle 中的事务隔离级别

Oracle数据库支持Read Committed（提交读）和Serializable（可串行化）这两种事务隔离级别，提交读是Oracle数据库默认的事务隔离级别，Oracle不支持脏读。SYS用户不支持Serializable（可串行化）隔离级别。

### MySQL中的事务隔离级别

MySQL数据库支持Read Uncommitted（未提交读）、Read Committed（提交读）、Repeatable Read（可重复读）和Serializable（可串行化）这4种事务隔离级别，其中，Repeatable Read（可重复读）是MySQL数据库的默认隔离级别。

### SQL Server中的事务隔离级别

SQL Server共支持6种事务隔离级别，分别为：Read Uncommitted（未提交读）、Read Committed（提交读）、Repeatable Read（可重复读）、Serializable（可串行化）、Snapshot（快照）、Read Committed Snapshot（已经提交读隔离）。SQL Server数据库默认的事务隔离级别是Read Committed（提交读）。

## CAP定理（CAP theorem）

CAP定理又称CAP原则是一个衡量系统设计的准则。CAP定理指的是在一个分布式系统中，Consistency（一致性）、Availability（可用性）、Partition Tolerance（分区容错性），三者不可兼得。

- C（一致性）：所有节点在同一时间的数据完全一致；
- A（可用性）：服务一直可用，每个请求都能接收到一个响应，无论响应成功或失败；
- P（分区容错性）：分布式系统在遇到某节点或网络分区故障的时候，仍然能够对外提供满足一致性和可用性的服务。

任何分布式系统在可用性、一致性、分区容错性方面，不能兼得，最多只能得其二。因此，任何分布式系统的设计只是在三者中的不同取舍而已。所以，就有了3个分类：CA数据库，CP数据库和AP数据库。传统的关系型数据库在功能支持上通常很宽泛，从简单的键值查询，到复杂得多表联合查询再到事务机制的支持。而与之不同的是，NoSQL系统通常注重性能和扩展性，而非事务机制，因为事务就是强一致性的体现。

- CP数据库考虑的是一致性和分区容错性，这种数据库对分布式系统内的通信要求比较高，因为要保持数据的一致性，需要做大量的交互，如Oracle RAC、Sybase集群。虽然Oracle RAC具备一定的扩展性，但当节点达到一定数目时，性能（即可用性）就会下降很快，并且节点之间的网络开销还在，需要实时同步各节点之间的数据。CP数据库通常性能不是特别高，例如火车售票系统。
- AP数据库考虑的是实用性和分区容忍性，即外部访问数据，可以更快地得到回应，例如博客系统。这时候，数据的一致性就可能得不到满足或者对一致性要求低一些，各节点之间的数据同步没有那么快，但能保存数据的最终一致性。比如一个数据，可能外部一个进程在改写这个数据，同时另一个进程在读这个数据，此时，数据显现是不一致的。但是有一点，就是数据库会满足一个最终一致性的概念，即过程可能是不一致的，但是到某一个终点，数据就会一致起来。当前热炒的NoSQL大多是典型的AP类型数据库。

### CAP 和 ACID 一致性的区别

一般事务ACID中的一致性是有关数据库规则的描述，如果数据表结构定义一个字段值是唯一的，那么一致性系统将解决所有操作中导致这个字段值非唯一性的情况，如果带有一个外键的一行记录被删除，那么其外键相关记录也应该被删除，这就是ACID一致性意思。

CAP理论的一致性是保证同一个数据在所有不同服务器上的拷贝都是相同的，这是一种逻辑保证，而不是物理，因为网络速度限制，在不同服务器上这种复制是需要时间的，集群通过阻止客户端查看不同节点上还未同步的数据维持逻辑视图。
