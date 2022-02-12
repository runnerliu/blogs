## Mysql 系列 - 事务

学习 MySQL 、SQL Server 或者 Oracle 这些关系型数据库的时候，必须了解的概念之一就是事务，在各种大中小型项目中，事务的使用非常频繁和广泛。本文简单介绍 MySQL 中事务的四大特性，以及在不考虑事务隔离时，对数据的不同操作会产生的一些影响，同时介绍了事务的四中隔离级别。

### 事务 ACID 特性

#### 原子性：Atomicity

事务必须是原子工作单元；对于其执行的数据修改操作，要么全都执行，要么全都不执行。通常，与某个事务关联的操作具有共同的目标，并且是相互依赖的。如果系统只执行这些操作的一个子集，则可能会破坏事务的总体目标。原子性消除了系统处理操作子集的可能性。

#### 一致性：Consistency

事务在完成时，必须使所有的数据都保持一致状态。在相关数据库中，所有规则都必须应用于事务的修改，以保持所有数据的完整性。事务结束时，所有的内部数据结构（如 B 树索引或双向链表）都必须是正确的。某些维护一致性的责任由应用程序开发人员承担，他们必须确保应用程序已强制所有已知的完整性约束。例如，当开发用于转帐的应用程序时，应避免在转帐过程中任意移动小数点。假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性。

#### 隔离性：Isolation

由并发事务所作的修改必须与任何其它并发事务所作的修改隔离。事务查看数据时数据所处的状态，要么是另一并发事务修改它之前的状态，要么是另一事务修改它之后的状态，事务不会查看中间状态的数据。这称为隔离性，因为它能够重新装载起始数据，并且重播一系列事务，以使数据结束时的状态与原始事务执行的状态相同。当事务可序列化时将获得最高的隔离级别。在此级别上，从一组可并行执行的事务获得的结果与通过连续运行每个事务所获得的结果相同。由于高度隔离会限制可并行执行的事务数，所以一些应用程序降低隔离级别以换取更大的吞吐量。

即要达到这么一种效果：对于任意两个并发的事务 T1 和 T2，在事务 T1 看来，T2 要么在 T1 开始之前就已经结束，要么在 T1 结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。

#### 持久性：Durability

事务完成之后，它对于系统的影响是永久性的。该修改即使出现致命的系统故障也将一直保持。

例如我们在操作数据库时，在提交事务方法后，提示用户事务操作完成，当我们程序执行完成直到看到提示后，就可以认定事务以及正确提交，即使这时候数据库出现了问题，也必须要将我们的事务完全执行完成，否则就会造成我们看到提示事务处理完毕，但是数据库因为故障而没有执行事务的重大错误。

以上介绍完事务的 ACID 特性，现在重点来说明下事务的隔离性，当多个线程都开启事务操作数据库中的数据时，数据库系统要能进行隔离操作，以保证各个线程获取数据的准确性，在介绍数据库提供的各种隔离级别之前，我们先看看如果不考虑事务的隔离性，会发生的几种问题：

### 并发事务的问题

#### 丢失更新

当两个或多个事务选择同一行，然后基于最初选定的值更新该行时，会发生丢失更新问题。每个事务都不知道其它事务的存在。最后的更新将重写由其它事务所做的更新，这将导致数据丢失。

例如，两个编辑人员制作了同一文档的电子复本。每个编辑人员独立地更改其复本，然后保存更改后的复本，这样就覆盖了原始文档。最后保存其更改复本的编辑人员覆盖了第一个编辑人员所做的更改。如果在第一个编辑人员完成之后第二个编辑人员才能进行更改，则可以避免该问题。

#### 脏读

读“脏”数据是指事务 T1 修改某一数据，并将其写回磁盘，事务 T2 读取同一数据后，T1 由于某种原因被除撤消，而此时T1 把已修改过的数据又恢复原值，T2 读到的数据与数据库的数据不一致，则 T2 读到的数据就为“脏”数据，即不正确的数据。简单来说就是，事务 T2 读取了事务 T1 还未提交的更新。

例如：一个编辑人员正在更改电子文档。在更改过程中，另一个编辑人员复制了该文档（该复本包含到目前为止所做的全部更改）并将其分发给预期的用户。此后，第一个编辑人员认为所做的更改是错误的，于是删除了所做的编辑并保存了文档。分发给用户的文档包含不再存在的编辑内容，并且这些编辑内容应认为从未存在过。如果在第一个编辑人员确定最终更改前任何人都不能读取更改的文档，则可以避免该问题。

#### 不可重复读

指事务 T1 读取数据后，事务 T2 执行更新操作，使 T1 无法读取前一次结果。

#### 幻读

按一定条件从数据库中读取了某些记录后，T2 删除了其中部分记录，当 T1 再次按相同条件读取数据时，发现某些记录消失；T1 按一定条件从数据库中读取某些数据记录后，T2 插入了一些记录，当 T1 再次按相同条件读取数据时，发现多了一些记录。

### 事务隔离级别

#### 读取未提交内容：Read Uncommitted

- 所有事务都可以看到其他未提交事务的执行结果
- 一个事务更新的时候不允许另一更新，但允许另一事务读取，不会出现丢失更新
- 该级别引发的问题是 - 脏读：读取到了未提交的数据

该隔离级别可以通过“排他写锁”实现。

#### 读取提交内容：Read Committed

- 这是大多数数据库系统的默认隔离级别（但不是 MySQL 默认的）
- 一个事务更新的时候不允许读取，必须等到更新事务提交后才能读取
- 这种隔离级别出现的问题是 - 不可重复读：不可重复读意味着我们在同一个事务中执行完全相同的select语句时可能看到不一样的结果。导致这种情况的原因可能有：
  - 有一个交叉的事务有新的commit，导致了数据的改变
  - 一个数据库被多个实例操作时，同一事务的其他实例在该实例处理其间可能会有新的commit

#### 可重复读：Repeatable Read

- **这是 MySQL 的默认事务隔离级别**
- 一个事务读取时，不允许更新，但允许插入
- 此级别可能出现的问题 - 幻读：当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行
- InnoDB 和 Falcon 存储引擎通过多版本并发控制(MVCC，Multiversion Concurrency Control)机制解决了该问题

#### 序列化：Serializable

- 这是最高的隔离级别
- 它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁
- 在这个级别，可能导致大量的超时现象和锁竞争



Read More：

> [数据库事务隔离级别通俗理解](https://www.oschina.net/question/258230_134502)
>
> [数据库事务](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1/9744607?fr=aladdin)
>
> [事务隔离级别](https://baike.baidu.com/item/%E4%BA%8B%E5%8A%A1%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB/2638091?fr=aladdin)
>
> [更改MySQL的默认事务隔离级别](http://blog.csdn.net/u012712087/article/details/46402433)
>
> [数据库事务的四大特性以及事务的隔离级别](http://www.cnblogs.com/fjdingsd/p/5273008.html)
>
> [[MySQL]--通过例子理解事务的4中隔离级别](http://www.cnblogs.com/snsdzjlz320/p/5761387.html)
