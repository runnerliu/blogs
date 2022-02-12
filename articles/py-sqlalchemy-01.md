## Python 库 - SQLAlchemy 事务异常

项目框架采用 Tornado ，SQLAlchemy 作为数据库ORM，简陋的代码如下：

```
def query(self, filters, orders=''):
    try:
        if not isinstance(filters, str) or not isinstance(orders, str):
            return False
        if not len(orders):
            return self.s.query(Node).filter(filters).first()
        else:
            r = self.s.query(Node).filter(filters).order_by(orders).all()
            if len(r):
                return r[0]
            return None
    except Exception as e:
        self.logger.error(traceback.format_exc())
        return None
    finally:
        Session.remove()
```

但是在跑了一段时间后出现了如下问题：

> (sqlalchemy.exc.InvalidRequestError) Can't reconnect until invalid transaction is rolled back

错误中很明显是数据库的连接由于事务某些错误出现了问题，SQLAlchemy 在尝试重新连接时失败了。

几经翻找资料后，问题产生原因如下：

从数据库连接池（pool）中获取的 `connection` 没有以 `session.commit()` 或 `session.rollback()` 或 `session.close()` 的某一种放回 pool 中。这时 `connection` 的事务（`transaction` ）没有完结，而在后续与数据库交互中，由于某些原因（如死锁、超时）数据库连接池中的 `connection` 又死掉了，当获取到这个连接时，SQLAlchemy 尝试重新连接。但由于 `transaction` 还没完结，无法重连。就抛出了上述错误。

解决办法：

1. 显示调用事务结束

   使用 `try...except...` 代码块，`except` 中捕获到异常时，调用 `session.rollback()` 回滚事务。

2. 打开 `autocommit`

   使用 SQLAlchemy 的初始化方式为：

   ```
   Base = declarative_base()
   engine = create_engine(
       "mysql+pymysql://{}:{}@{}:{}/{}".format(
           mysql_config['default']['user'],
           mysql_config['default']['password'],
           mysql_config['default']['host'],
           mysql_config['default']['port'],
           mysql_config['default']['name'],
       ),
       encoding="utf-8",
       echo=False,
       pool_recycle=mysql_config['connect_pool']['pool_recycle'],
       pool_size=mysql_config['connect_pool']['pool_size']
   )
   Session = scoped_session(sessionmaker(bind=engine))
   ```

   默认是使用事务操作，我们可以在初始化语句中加上 `autocommit=true` 关闭事务：

   ```
   engine = create_engine(
       "mysql+pymysql://{}:{}@{}:{}/{}?autocommit=true".format(
           mysql_config['default']['user'],
           mysql_config['default']['password'],
           mysql_config['default']['host'],
           mysql_config['default']['port'],
           mysql_config['default']['name'],
       ),
       encoding="utf-8",
       echo=False,
       pool_recycle=mysql_config['connect_pool']['pool_recycle'],
       pool_size=mysql_config['connect_pool']['pool_size']
   )
   ```

   这样生成的查询语句就会立即执行。**注意：**个人感觉这并不是一个好方法，还是老老实实捕获异常吧。