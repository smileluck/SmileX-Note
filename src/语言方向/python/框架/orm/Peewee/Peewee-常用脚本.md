

---

# Peewee-常用方法
## 根据现有库创建类

1. 安装 `Peewee`
   ```shell
   pip install peewee
   ```
2. 找到 `pwiz.py` 脚本（这里以mysql数据库为例）
    - mysql数据库
    ```shell
    python -m pwiz -e mysql -H localhost -u username -P password -p 3306 database_name > models.py
    ```
    - postgreSQl数据库
    ```shell
    python -m pwiz -e postgresql -H localhost -u username -P password -p 5432 database_name > models.py
    ```
## 事务使用
在 Peewee 里，你可以借助不同的方式来启用事务，以此确保数据库操作的原子性。以下为你介绍几种常见的使用事务的方法：

### 1. 使用 `atomic()` 上下文管理器
`atomic()` 是一个上下文管理器，它能够让你把一系列数据库操作包裹在一个事务当中。要是在这个过程里出现异常，事务就会回滚；若所有操作都成功，事务则会提交。

以下是一个简单示例：
```python
from peewee import *

# 连接 SQLite 数据库
db = SqliteDatabase('example.db')

# 定义模型类
class User(Model):
    name = CharField()
    age = IntegerField()

    class Meta:
        database = db

# 创建表
db.create_tables([User])

try:
    with db.atomic():
        # 插入第一条记录
        user1 = User.create(name='Alice', age=25)
        # 插入第二条记录
        user2 = User.create(name='Bob', age=30)
        # 模拟一个错误，触发回滚
        raise ValueError("Something went wrong!")
except ValueError:
    print("Transaction rolled back due to an error.")
else:
    print("Transaction committed successfully.")
    
```
    

在上述代码中，`with db.atomic()` 把两条插入操作包裹起来，当作一个事务。由于模拟了一个错误，所以事务会回滚，数据库里不会有新记录插入。

### 2. 使用 `Database.manual_commit()` 和 `Database.commit()`、`Database.rollback()`
你也可以手动开启、提交或者回滚事务。

以下是示例代码：
```python
from peewee import *

# 连接 SQLite 数据库
db = SqliteDatabase('example.db')

# 定义模型类
class User(Model):
    name = CharField()
    age = IntegerField()

    class Meta:
        database = db

# 创建表
db.create_tables([User])

# 手动开启事务
db.manual_commit(True)
try:
    # 插入第一条记录
    User.create(name='Charlie', age=35)
    # 插入第二条记录
    User.create(name='David', age=40)
    # 提交事务
    db.commit()
    print("Transaction committed successfully.")
except Exception as e:
    # 回滚事务
    db.rollback()
    print(f"Transaction rolled back due to an error: {e}")
finally:
    # 恢复自动提交模式
    db.manual_commit(False)
    
```
    

在这个例子中，借助 `db.manual_commit(True)` 手动开启事务，使用 `db.commit()` 提交事务，用 `db.rollback()` 回滚事务。最后，使用 `db.manual_commit(False)` 恢复自动提交模式。

### 3. 在函数上使用 `@db.atomic()` 装饰器
你还能使用 `@db.atomic()` 装饰器把一个函数里的所有数据库操作包裹在一个事务中。

示例如下：
```python
from peewee import *

# 连接 SQLite 数据库
db = SqliteDatabase('example.db')

# 定义模型类
class User(Model):
    name = CharField()
    age = IntegerField()

    class Meta:
        database = db

# 创建表
db.create_tables([User])

@db.atomic()
def add_users():
    # 插入第一条记录
    User.create(name='Eve', age=45)
    # 插入第二条记录
    User.create(name='Frank', age=50)

try:
    add_users()
    print("Transaction committed successfully.")
except Exception as e:
    print(f"Transaction rolled back due to an error: {e}")
    
```
    

在这个代码里，`@db.atomic()` 装饰器把 `add_users()` 函数里的操作包裹在一个事务中。若函数执行时出现异常，事务就会回滚。 