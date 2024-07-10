[toc]

---

# 前言

> https://github.com/PyMySQL/PyMySQL



# 使用说明

本教程使用的 PyMySQL 1.0.2，PyMySQL 的使用分为如下几步。
与数据库服务器建立连接：conn=pymysql.Connect(...)。
获取游标对象（用于发送和接收数据）：cursor=conn.cursor()。
使用游标执行 SQL 语句：cursor.excute(sql)。此时返回的是执行该语句后数据库表中受影响的数据条数。
使用 fetch() 方法来获取执行的结果。
关闭连接：先关闭游标，再关闭连接。

下面进行基本操作的示例演示。

1) 新建数据库：
import pymysql
con =pymysql.connect(host='localhost',user='root',passwd='root',port=3306,db='business')
cursor = con.cursor()
cursor.execute('create database if not exists business default charset utf8')
con.commit()

2) 新建表：
cursor.execute('use business')
cursor.execute('''CREATE table if not exists boss(id int auto_increment primary key,
           name varchar(20) not null,
           salary int not null)''')

3) 增加数据：
sql = """INSERT into boss(name,salary)
      values ('Jack',91), ('Harden',1300), ('Pony',200)"""
cursor.execute(sql)
con.commit()

4) 删除数据：
cursor.execute('delete from boss where salary < 100')
con.commit() # 一定要提交，提交了语句才生效

5) 修改数据：
cursor.execute("UPDATE boss set salary = 2000 where name = 'Pony'")
con.commit()

6) 关闭数据库：
con.close()



# 封装库

```python
import pymysql
from timeit import default_timer

host = "localhost"
port = 3306
db = "ai_search"
user = "root"
password = "123456"


# 用PyMySQL操作数据库
def get_connection():
    conn = pymysql.connect(host=host, port=port, db=db, user=user, password=password)
    return conn


# 使用with优化代码
class UsingMysql(object):
    def __init__(self, commit=True, log_time=True, log_label="总用时"):
        """
        :param commit: 是否在最后提交事务(设置为False的时候方便单元测试)
        :param log_time: 是否打印程序运行总时间
        :param log_label: 自定义log的文字
        """
        self._log_time = log_time
        self._commit = commit
        self._log_label = log_label

    def __enter__(self):
        # 如果需要记录时间
        if self._log_time is True:
            self._start = default_timer()
        # 在进入的时候自动获取连接和cursor
        conn = get_connection()
        cursor = conn.cursor(pymysql.cursors.DictCursor)
        conn.autocommit = False
        self._conn = conn
        self._cursor = cursor
        return self

    def __exit__(self, *exc_info):
        # 提交事务
        if self._commit:
            self._conn.commit()
        # 在退出的时候自动关闭连接和cursor
        self._cursor.close()
        self._conn.close()
        if self._log_time is True:
            diff = default_timer() - self._start
            print("-- %s: %.6f 秒" % (self._log_label, diff))

    # 一系列封装的业务方法
    # 返回count
    def get_count(self, sql, params=None, count_key="count(id)"):
        self.cursor.execute(sql, params)
        data = self.cursor.fetchone()
        if not data:
            return 0
        return data[count_key]

    def fetch_one(self, sql, params=None):
        self.cursor.execute(sql, params)
        return self.cursor.fetchone()

    def fetch_all(self, sql, params=None):
        self.cursor.execute(sql, params)
        return self.cursor.fetchall()

    def fetch_by_pk(self, sql, pk):
        self.cursor.execute(sql, (pk,))
        return self.cursor.fetchall()

    def update_by_pk(self, sql, params=None):
        self.cursor.execute(sql, params)

    @property
    def cursor(self):
        return self._cursor


def check_it():
    """ """
    with UsingMysql(log_time=True) as um:
        um.cursor.execute("select count(id) as total from Product")
        data = um.cursor.fetchone()
        print("-- 当前数量: %d " % data["total"])


if __name__ == "__main__":
    check_it()

```

