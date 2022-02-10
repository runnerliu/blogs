## Mysql 系列 - InnoDB 与 MyISAM

### 存储结构

MyISAM：每个MyISAM在磁盘上存储成三个文件。第一个文件的名字以表的名字开始，扩展名指出文件类型。.frm文件存储表定义。数据文件的扩展名为 .MYD  (MYData)。索引文件的扩展名是 .MYI  (MYIndex)；

InnoDB：所有的表都保存在同一个数据文件中（也可能是多个文件，或者是独立的表空间文件），InnoDB表的大小只受限于操作系统文件的大小，一般为2GB。

### 存储空间

MyISAM：可被压缩，存储空间较小。支持三种不同的存储格式：静态表(默认，但是注意数据末尾不能有空格，会被去掉)、动态表、压缩表；

InnoDB：需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引。

### 事务支持

MyISAM：强调的是性能，每次查询具有原子性，其执行数度比 InnoDB 类型更快，但是不提供事务支持；

InnoDB：提供事务支持事务，外部键等高级数据库功能。 具有事务、回滚和崩溃修复能力的事务安全型表。

### CURD操作

MyISAM：如果执行大量的 SELECT，MyISAM 是更好的选择。(因为没有支持行级锁)，在增删的时候需要锁定整个表格，效率会低一些。相关的是 InnoDB 支持行级锁，删除插入的时候只需要锁定该行就行，效率较高；

InnoDB：如果你的数据执行大量的 INSERT 或 UPDATE，出于性能方面的考虑，应该使用 InnoDB 表。DELETE 从性能上InnoDB 更优，但 DELETE FROM table 时，InnoDB 不会重新建立表，而是一行一行的删除，在 InnoDB 上如果要清空保存有大量数据的表，最好使用`truncate table ` 这个命令。

### 外键

MyISAM：不支持；InnoDB：支持。

### 锁机制

InnoDB 支持数据行锁定，MyISAM 不支持行锁定，只支持锁定整个表。即 MyISAM 同一个表上的读锁和写锁是互斥的，

MyISAM 并发读写时如果等待队列中既有读请求又有写请求，默认写请求的优先级高，即使读请求先到，所以 MyISAM 不适合于有大量查询和修改并存的情况，那样查询进程会长时间阻塞。因为 MyISAM 是锁表，所以某项读操作比较耗时会使其他写进程饿死。

### 索引

InnoDB 从5.6.4才开始支持全文索引，而 MyISAM 一直支持。全文索引是指对 char、varchar 和 text 中的每个词（停用词除外）建立倒排序索引。MyISAM的全文索引其实很简单，因为它不支持中文分词，必须由使用者分词后加入空格再写到数据表里，而且少于4个汉字的词会和停用词一样被忽略掉。

### 其他

MyISAM 支持 GIS 数据，InnoDB 不支持。即 MyISAM 支持以下空间数据对象：Point，Line，Polygon，Surface等。

对于AUTO_INCREMENT 类型的字段，InnoDB 中必须包含只有该字段的索引，但是在 MyISAM 表中，可以和其他字段一起建立联合索引。

InnoDB 中不保存表的具体行数，也就是说，执行 `select count(*) from table` 时，InnoDB 要扫描一遍整个表来计算有多少行，但是 MyISAM 只要简单的读出保存好的行数即可。注意的是，当 ` count(*)` 语句包含 where 条件时，两种表的操作是一样的。

InnoDB 表的行锁也不是绝对的，假如在执行一个 SQL 语句时 MySQL 不能确定要扫描的范围，InnoDB 表同样会锁全表，例如 `update table set num=1 where name like '%aaa%' ` 

