

**python 实现连接数据库 mysql 的步骤：**

一、引入PyMysql或者MySQLdb

二、获取与数据库的连接

三、执行SQL语句和存储过程

四、关闭数据库连接



## PyMysql

PyMysql 是 Python3.x 版本中用于连接 MySQL 服务器的库，python2.x 中使用Mysqldb库。



## 安装 PyMysql

```shell
pip install PyMysql 
```



## 示例连接数据库

```python
import pymysql

# 打开数据库连接
db = pymysql.connect(host='127.0.0.1',port=3306,user='root',passwd='admin',db='glance',charset='utf8')

# 使用cursor()方法获取操作游标 
cursor = db.cursor()
sql = " select * from  images"

# 使用execute方法执行SQL语句 
cursor.execute(sql)

# 使用 fetchone() 方法获取一条数据 
data = cursor.fetchone()
print(data)

# 关闭数据库连接 
db.close()
```



如果select本身取的时候有多条数据时：

cursor.fetchone()：将只取最上面的第一条结果，返回单个元组如('id','title')，然后多次使用cursor.fetchone()，依次取得下一条结果，直到为空。

cursor.fetchall() : 将返回所有结果，返回二维元组，如(('id','title'),('id','title'))



### 问题

普通的操作无论是 fetchall() 还是 fetchone() 都是先将数据载入到本地再进行计算，如果存在大量的数据会导致内存资源消耗光。



[](https://www.cnblogs.com/kingron/p/13875223.html)
