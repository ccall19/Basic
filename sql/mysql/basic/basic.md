# 一些数据库操作
````SQL
SHOW DATABASES; -- 输出所有的数据库
USE databaseName; -- 使用 指定 的数据库
SHOW TABLES; -- 输出数据库中的所有表
DESC tableName; -- 描述表结构
CREATE DATABASE databaseName; -- 创建数据库
DROP DATABASE databaseName; -- 删除数据库
MYSQLDUMP -u userName -p databaseName tableName -> name.sql; -- 表名可省略,导出数据库数据 至 指定文件
MYSQL -u userName -p databaseName < name.sql -- 导入数据到指定数据库
````

# 一些表操作
![alt text](image.png)

# 增删改查
WHERE 均可省略
## 插入一/多条数据
````sql
INSERT INTO tableName (columnName0,columnName1...) VALUES (对应 字段 的值),(...),(...);
````
##  查询数据
````sql
SELECT * FROM tableName WHERE 条件语句; -- SELECT 后边表示 要展示的 字段，或基于字段的 计算、统计值，包括表达式、聚合函数
````
##  更新数据
````sql
UPDATE tableName SET columnName = newValue WHERE 条件语句;
````
## 删除数据
````sql
DELETE FROM tableName WHERE 条件语句;
````

# 一些常用语句
````sql
-- 用于 WHERE 条件语句中
-- 条件 与或
AND
OR
IN (... , ...)  -- 是否存在范围中
BETWEEN ... AND ... -- 表示一个范围

NOT -- 取反
IS NULL -- 判空 ，IS NOT NULL

-- 排序
ORDER BY columnName -- 按指定列 升序排序
ORDER BY columnName DESC -- 按指定列 降序排序

-- 去重
SELECT DISTINCT columnName FROM ...;

-- 连接
-- 并集
UNION -- 合并两个查询结果
UNION ALL -- 可以重复输出 满足两个语句的 同一条数据
-- 交集
INTERSECT 
-- 差集
EXCEPT

-- 通配符，详细见下文图片
LIKE -- 通配符匹配 % ：任意数量字符，_ : 一个字符。 WHERE name LIKE 'GGBon_' or name LIKE 'GG%'
REGEXP -- 正则表达式。WHERE name REGEXP '[GBn]'

-- 聚合函数，详细见下文图片
SELECT COUNT(*) FROM databaseName; -- 统计数据量
SELECT AVG(columnName) FROM databaseName; -- 针对列 计算 均值

GROUP BY -- 针对字段值分类聚合，结合聚合函数使用，比如统计不同等级数据量：SELECT level ,COUNT(level) FROM tableName GROUP BY level;

HAVING -- 用来对 检索结果进行 过滤，比如检索 level 大于 4:  SELECT level ,COUNT(level) FROM tableName GROUP BY level HAVING COUNT(level) > 4;
-- 以及可以结合 ORDER BY 排序 

LIMIT -- 限制输出，LIMIT 3; 表示输出 3 条数据；或者， LIMIT 偏移量/偏移起始位置 , 返回数量，即 ： LIMIT 3,4; 表示从 3 开始，总共返回 4 条数据。 ** 这也是分页查询原理**
````
## 正则表达式通配符
![alt text](image-1.png)
## 聚合函数
![alt text](image-2.png)

# 子查询
一条语句表达式可以 写入 另一条语句，即内嵌的 语句 的输出 可以作为外部 语句 的输入来使用。 
````sql
SELECT * FROM player WHERE level > (SELECT AVG(level) FROM player);
SELECT level,(SELECT AVG(level) FROM player) as avg FROM player;

as -- 别名
````
![alt text](image-3.png)
# 执行一条 select 语句，期间发生了什么？
