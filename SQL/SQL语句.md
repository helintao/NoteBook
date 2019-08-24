# 创建表
## 语法
```sql
create table <表名> (<列名><数据类型>[列级完整性约束条件][,<列名><数据类型>[列级完整性约束条件]]...);
```

## 列子
```sql
create table Stu
(
    Id int not null unique primary key,
    name varchar(20) not null,
    age int null,
    gender varchar(4) null
)
```

# 删除表
## 语法
```sql
drop table <表名>
```

# 清除表
## 语法
```sql
truncate table <表名>
```

# 修改表
## 语法
```sql
-- 添加列
alter table <表名> [add <新列名><数据类型>[列级完整性约束条件]

-- 删除列
alter table <表名> [drop column <列名>]

-- 修改列
alter table <表名> [modify column <列名> <数据类型> [列级完整性约束条件]]
```

# 查询
## 语法
```sql
SELECT [ALL | DISTINCT] <目标列表达式>[,<目标列表达式>]…
  FROM <表名或视图名>[,<表名或视图名>]…
  [WHERE <条件表达式>]
  [GROUP BY <列名> [HAVING <条件表达式>]]
  [ORDER BY <列名> [ASC|DESC]…]

-- SQL查询语句的顺序：SELECT、FROM、WHERE、GROUP BY、HAVING、ORDER BY。SELECT、FROM是必须的，HAVING子句只能与GROUP BY搭配使用。
```

# 插入
## 语法
```sql
-- 插入不存在的数据
INSERT INTO <表名> [(字段名[,字段名]…)] VALUES (常量[,常量]…)

-- 将查询的数据插入到数据表中
INSERT INTO <表名> [(字段名[,字段名]…)] SELECT 查询语句
```

# 更新
## 语法
```sql
UPDATE <表名> SET 列名=值表达式[,列名=值表达式…][WHERE 条件表达式]
```

# 删除
## 语法
```sql
DELETE FROM <表名> [WHERE 条件表达式]
```