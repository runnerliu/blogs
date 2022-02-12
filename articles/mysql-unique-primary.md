## Mysql 索引 - UNIQUE KEY 与 PRIMARY KEY

定义了 UNIQUE 约束的字段中不能包含重复值，可以为一个或多个字段定义 UNIQUE 约束。因此，UNIQUE 即可以在字段级也可以在表级定义， 在 UNIQUED 约束的字段上可以包含空值。UNIQUED 可空，可以在一个表里的一个或多个字段定义；

PRIMARY KEY 不可空不可重复，在一个表里可以定义联合主键；

一般来说，PRIMARY KEY = UNIQUE KEY +  NOT NULL

UNIQUE KEY 就是唯一，当你需要限定你的某个表字段每个值都唯一，没有重复值时使用。比如说，如果你有一个person 表，并且表中有个身份证的 column，那么你就可以指定该字段为UNIQUE KEY。 从技术的角度来看，PRIMARY KEY和UNIQUE KEY有很多相似之处。但还是有以下区别： 

- 作为 PRIMARY KEY 的域/域组不能为NULL，而 UNIQUE KEY 可以。 
- 在一个表中只能有一个PRIMARY KEY，而多个 UNIQUE KEY 可以同时存在。 

更大的区别在逻辑设计上。PRIMARY KEY 一般在逻辑设计中用作记录标识，这也是设置 PRIMARY KEY 的本来用意，而UNIQUE KEY 只是为了保证域/域组的唯一性。 

PRIMARY KEY 的语法：alter table name add constraint key name primary key( columns); 

UNIQUE KEY 的语法：alter table name add constraint key name unique( columns); 

一个表只能有一个主键，但是可以有好多个UNIQUE KEY，而且 UNIQUE KEY 可以为 NULL 值，如员工的电话号码一般就用UNIQUE KEY，因为电话号码肯定是唯一的，但是有的员工可能没有电话。