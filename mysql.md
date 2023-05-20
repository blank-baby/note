

## SQL语句

### 1.语句执行顺序

from >join on > where > group by > 分组函数  > having > select > distinct > order by > limit

### 2.聚合函数（分组函数）

- 这是多行处理函数
- 默认忽略NULL的值
- 这些函数不能直接使用在where语句后面
- 如果有group by 会在group by 后面执行
- **如果有group by，那select后面的字段只能跟分组函数和参与分组的字段**

### 3.触发器

对一个表的操作触发对另外一个表的操作是触发器

```sql
# 如果存在这个触发器的名字就删除这个名字
DROP TRIGGER IF EXISTS hasCalUpdate;
# 更换结束符为 //
delimiter //
# 创建触发器hasCalUpdate 在risk_standard 表插入数据时触发
create TRIGGER hasCalUpdate 在 after INSERT on risk_standard
for each row
begin
# 这是写要触发的sql语句
update risk_resource set is_cal = 1 where id = new.risk_resource_id;
end //
# 更换结束符为 ;
delimiter ;
```



查看触发器命令

```sql
SHOW TRIGGERS
```



## java连接mysql流程

### 正常连接

1. 加载驱动
2. 连接数据库
3. 获取操作数据的对象
4. 编写sql语句
5. 执行sql语句
6. 关闭资源

```java
//加载驱动
Class.forname("com.mysql.jdbc.Drive");

//连接数据库
Connection connection = DriverManager.getConnection(url, username, password);

//获取操作数据的对象
Statement statement = connection.createStatement();

//编写sql语句
sql = "select * from users";

//执行sql语句
Result re = statement.executeQuery(sql);
while(re.nest()){
    System.out.println("id="+re.getObject("id"));
    -----------------------------------------------
}

//关闭资源
statement.close();
connection.close();
```

### 预编译连接

1. 加载驱动
2. 连接数据库
3. 编写sql语句
4. 预编译
5. 执行sql语句
6. 关闭资源

```java
//加载驱动
Class.forname("com.mysql.jdbc.Drive");

//连接数据库
Connection connection = DriverManager.getConnection(url, username, password);

//获取sql语句
sql = "insert into users(id,name,password,email) values (?,?,?,?)";

//预编译
PreparedStatement preparedStatement = connection.prepareStatement(sql);
preparedStatement.setInt(1,1);
preparedStatement.setString(2,"jiruixn");
preparedStatement.setString(3,"123456");
preparedStatement.setString(4,"1148@qq.com");

//执行sql语句
int i = preparedStatement.executeUpdate(sql);
if(i>0)
   System.out.println("成功"); 

//关闭资源
preparedStatement.close();
connection.close();
```



## 事务

### mysql的事务

- 开启事务
- 事务提交 commit()
- 事务回滚 rollback()
- 关闭事务

```sql
-- SQL语句大小写没有关系
set autocommit = 1      -- 开启数据库事务自动提交
set autocommit = 0      -- 关闭数据库事务自动提交

--流程


-- 关闭事务自动提交
set autocommit = 1 

-- 开启一个事务
START TRANSACTION      
sql操作1
sql操作2

-- 提交 持久化(成功)
commit

-- 回滚(失败)
rollback

-- 事务结束
set autocommit = 1 

-- 了解
savepoint 保存点名	 -- 设置一个事务的保存点
rollback savepoint  -- 回滚到保存点名
release savepoint   -- 撤销保存点名

```



```java
public class Action {
    public static void main(String[] args) {
        try {
            conn = JdbcUtils.getConnection();
            //关闭数据库的自动提交功能， 开启事务
            conn.setAutoCommit(false);
            //自动开启事务
            String sql = "update account set money = money-500 where id = 1";
            ps =conn.prepareStatement(sql);
            ps.executeUpdate();
            String sql2 = "update account set money = money-500 where id = 2";
            ps=conn.prepareStatement(sql2);
            ps.executeUpdate();
 
            //业务完毕，提交事务
            conn.commit();
            System.out.println("操作成功");
        } catch (Exception e) {
            try {
                //如果失败，则默认回滚
                conn.rollback();//如果失败，回滚
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
            e.printStackTrace();
        }finally {
            try {
                JdbcUtils.release(conn,ps,rs);
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}

```



## mysql语句

### 1.运算符

> 等于运算符          =

- 有一个或两个参数为null，则结果就为null
- 比较的都是字符的话，按字符串比较
- 两个参数都为整数，按整数比较
- 一个字符串和整数比较，则mysql自动将字符串变为整数

> 安全的等于运算    <=>

用来判断null，当两个比较值都为null则返回1，有一个为null，另外一个不为null，则返回0，不会返回null。

> 判空的  IS NULL  或者 ISNULL

专门用来判断是否为空的语句



> 子查询

- any            满足任意一个子查询的结果
- some         满足任意一个子查询的结果
- all             满足全部子查询的结果
- exists         判断子查询是否存在，否则就不执行外查询
- in



> 合并查询结果    union 和union all

区别：使用union all不删除重复的结果，而用union 会删除重复的结果。







## mysql日志

### 二进制日志

```my.ini

[mysqld]
# binary log
# server-id必须配置 并且后面额数子必须唯一
server-id=1
log-bin=D:\\mysql\\log\\data
max_binlog_size = 10M

```

### 错误日志文件

```
[mysqld]

# error log
# 后面写上配置的路径，也可以不写。
log-error=D:\\mysql\\log\\data   
```



### 慢查询日志

```
# slow log

slow_query_log=ON
slow_query_log_file=D:\\mysql\\log\\slow
long_query_time=1
```

## mysql的锁



MyISAM、MEMORY只有表锁

- MyISAM表插入数据会往空隙中插入数据。
- MEMORY表插入数据会往空隙中插入数据。

InnoDB拥有表锁和行锁

### 表锁

#### 读锁 read

> 加锁方式：lock table 表名称  read

加读锁以后能进行的操作

|                                | 拥有读锁的线程 | 其他线程 |
| ------------------------------ | -------------- | :------: |
| 对锁定的表select               | 可以           |   可以   |
| 对锁定的表insert/update/delete | 不可以         |  不可以  |
| 对其他的表select               | 不可以         |   可以   |
| 对其他的表insert/update/delete | 不可以         |   可以   |

> 解锁方式：unlock tables

#### 读锁 read local

> 加锁方式：lock table 表名称  read local

加读锁以后能进行的操作

|                                | 拥有读锁的线程 |    其他线程    |
| ------------------------------ | -------------- | :------------: |
| 对锁定的表select               | 可以           |      可以      |
| 对锁定的表insert               | 不可以         | 可以（有条件） |
| 对锁定的表update/delete        | 不可以         |     不可以     |
| 对其他的表select               | 不可以         |      可以      |
| 对其他的表insert/update/delete | 不可以         |      可以      |

两种读锁的区别**:read local 的读锁允许其他线程让表的表尾插入数据。但是表中不能有间隙(表中有间断)，有间隙的表也不能插入数据**

> 解锁方式：unlock tables

#### 写锁

> 加锁方式：lock table 表名称 write

|                                       | 拥有读锁的线程 | 其他线程 |
| ------------------------------------- | -------------- | :------: |
| 对锁定的表select/insert/update/delete | 可以           |  不可以  |
| 对其他的表select/insert/update/delete | 不可以         |   可以   |

> 解锁方式：unlock tables

### 行锁

| 锁模式        | 共享锁（S锁） | 排他锁（X锁） |
| ------------- | ------------- | ------------- |
| 共享锁（S锁） | 兼容          | 冲突          |
| 排他锁（X锁） | 冲突          | 冲突          |

注意点：

- 行锁的只能给select语句加锁。
- **行锁只有使用索引条件检索数据是才能起作用（没where 或者where的id不是索引都会升级为表锁）**时才会起作用，不然加行锁就会变成表锁。
- 测试前关闭事务的自动提交。

#### 共享锁

>加锁方式 select id,name from user **lock in share mode**;

|                                       | 拥有读锁的线程 | 其他线程 |
| ------------------------------------- | -------------- | :------: |
| 对锁定的行select                      | 可以           |   可以   |
| 对锁定的行insert/update/delete        | 可以           |  不可以  |
| 对其他的表select/insert/update/delete | 可以           |   可以   |

> 解锁方式 commit      或者    rollback

#### 排他锁

> 加锁方式  自动：delete / update / insert   默认加上排他锁
>
> ​	           手动    select * from user where id = 15 **for update**

|                                       | 拥有读锁的线程 | 其他线程 |
| ------------------------------------- | -------------- | :------: |
| 对锁定的行select                      | 可以           |   可以   |
| 对锁定的行insert/update/delete        | 可以           |  不可以  |
| 对其他的表select/insert/update/delete | 可以           |   可以   |

## mysql优化

### 1.索引失效

#### 1.like查询

只有当%号不在第一个位置时才会起作用，否则失效

#### 2.多列索引

对于多列索引，只有查询条件中使用了这些字段中的第一个字段时才会被使用

#### 3.or查询 

只有两边的字段都有索引才会起作用

### 2.优化sql语句

####  order by

- 对order by的字段进行索引
- 不要对where 和 order by的sql语句进行表达式或者函数

> 不要对下面这些情况使用索引

- order by 混合使用asc和desc
- where的字段和order by的字段不一致
- 对不同的关键字进行order by排序

#### group by

group by 会对查询的结果的自动排序，可以使用order by null禁止排序

```sql
select id,count(data) from test group by id order by null
```

### 3.嵌套查询

使用子查询时可以使用连接（join）来代替。 连接不用建立临时表，查询速度比子查询快。

### 4.优化插入速度

#### 1. 对MyISAM引擎优化

##### 1.禁用索引

> 对于非空表，插入大量语句会建立索引，可以关闭索引，插入完毕再开启索引

```sql
alert table 表名称 disable keys   //关闭
alert table 表名称 enable keys    //打开
```

##### 2.禁用唯一性索引

> 插入数据表会进行唯一性检验，插入大量语句会很慢，可以关闭索引，插入完毕再开启索引

```sql
set unique_checks = 0;       //关闭
set unique_checks = 1;       //打开
```

##### 3.批量插入

使用insert插入大量语句尽量一条语句插入完毕，不要分开插入。

##### 4.Load命令批量导入



#### 2. 对InnoDB

##### 1.禁用唯一性索引

> 插入数据表会进行唯一性检验，插入大量语句会很慢，可以关闭索引，插入完毕再开启索引

```sql
set unique_checks = 0;       //关闭
set unique_checks = 1;       //打开
```

##### 2.禁用外键检查

> 插入数据之前禁止对外键的检查，插入成功再恢复对外键的检查。

```sql
set foreign_key_checks = 0;       //关闭
set foreign_key_checks = 1;       //打开
```

##### 2.禁用自动提交

> 插入大量数据禁止自动提交，插入完毕一起提交

```sql
set autocommit = 0;       //关闭
set autocommit = 1;       //打开
```

##### 3.批量插入





### 5.优化表结构

#### 1.字段较多的表拆分为多个表

将一个表中使用频率较低的字段拆分出来

#### 2. 增加中间表

对于经常进行联表查询的字段设置中间表。这样不用每次都去联表，直接对中间表进行查询

#### 3. 增加冗余字段（根据实际需求）

### 6. 分析检查表

#### 1.分析表

```sql
analyze [local] table 表名称;
```

> local是可选的  ，当加上时会不写进二进制文件日志

结果分析：

- table 表名字
- Op 执行的操作 
- Msg_type：status、info、note、warning、error
- Msg_text 显示信息

#### 2.检查表

```sql
check table 表名称 选项;
```

> 选项只会对MyISAM类型的表有用，对InnoDB的没用。CHECK  TABLE 语句在执行过程中会给表加读锁

选项

- quick  不扫描航
- fast 只检查没有被正确关闭的表
- changed 只检查上次检查后被更新的表和没有被正确关闭的表
- medium 扫描行，也可以使用，已验证被删除的连接是有效的。也可以计算出各行的关键字校验和，并使用计算出的校验和验证这一点
- extended 对每行的关键字进行一个全面的关键字查找。这可以确保表是100%一致的。

#### 3.优化表

```sql
optimize [local] table 表名称
```

> optimiz 可以删除和更新造成的文件碎片。执行过程中会给表加读锁

## mysql日常问题记录

### 1. 不删除表让id从1开始

第一步：ALTER TABLE 表名 DROP `id`;

第二步：ALTER TABLE `表名 ` ADD id INT PRIMARY KEY NOT NULL AUTO_INCREMENT FIRST;

### 2. 删除elasticsearch单条数据

```bash
curl -X POST 'http://49.233.2.176:9200/rsp1/_delete_by_query' -H 'content-Type:application/json' -d '{"query": { "match": {"name": "AD"}}}'
```

