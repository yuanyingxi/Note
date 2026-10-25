# 1. SQL 和 NoSQL 的区别
## 1.1 SQL 数据库

SQL 数据库通常指关系型数据库，主要代表有 MySQL、PostgreSQL、Oracle 等。

关系型数据库以表的形式组织数据：

- 行：表示一条记录，例如一个用户。
    
- 列：表示记录的属性，例如用户 ID、姓名、年龄。
    
- 表之间：可以通过主键、外键等建立关联。
    

适合需要事务、结构化数据和复杂关联查询的业务场景。

### 1.2 NoSQL 数据库

NoSQL 指非关系型数据库，常见的有 Redis、MongoDB 等。

不同 NoSQL 数据库可以采用不同的数据组织方式，例如：

- Redis：字符串、哈希表、列表、集合等数据结构。
    
- MongoDB：文档，通常以类似 JSON 的 BSON 格式组织数据。
    

适合缓存、文档存储等特定场景。

### 1.3 如何选择？

|对比|关系型数据库|NoSQL 数据库|
|---|---|---|
|数据组织|通常以二维表组织|根据产品不同，可以使用键值、文档等结构|
|关联查询|支持 JOIN 等复杂关联查询|取决于具体产品，跨数据分片的关联可能更复杂|
|事务|常见产品支持事务，具体取决于实现|部分产品也支持事务，能力因产品而异|
|扩展方式|水平扩展可能涉及复杂的数据分片和事务问题|部分产品针对特定数据模型更容易扩展|
|常见用途|订单、支付、用户等业务数据|缓存、文档数据等|

注意：

- **ACID**：原子性、一致性、隔离性、持久性，是事务的四个重要特性。
    
- **BASE**：基本可用、软状态、最终一致性，是部分分布式系统采用的设计理念。
    

不能简单认为关系型数据库一定支持所有事务特性，而 NoSQL 一定不支持事务。具体能力要看数据库产品、存储引擎和配置。

实际项目中也可以同时使用两者，例如 MySQL 保存业务数据，Redis 缓存热点数据。

## 2. 数据库三大范式

数据库范式用于规范表结构，减少数据冗余和更新异常。

### 2.1 第一范式（1NF）

要求表中的每个字段值具有原子性，即每个字段存储一个符合该字段定义的值，不应把需要独立处理的多个数据项塞进同一个字段。

例如，不推荐这样设计：

|id|name|phone_numbers|
|---|---|---|
|1|小明|13800000000, 13900000000|

如果需要单独查询、修改每个电话号码，将多个号码放在一个字段里就不方便。

可以拆分为：

|id|name|
|---|---|
|1|小明|

|user_id|phone|
|---|---|
|1|13800000000|
|1|13900000000|

### 2.2 第二范式（2NF）

在满足第一范式的基础上，要求每个非主属性都完全依赖于候选码，不能只依赖于联合主键的一部分。

例如，订单明细表使用 `(order_id, product_id)` 作为联合主键：

|order_id|product_id|product_name|quantity|
|---|---|---|---|
|1|101|键盘|2|
|1|102|鼠标|1|

其中：

- `quantity` 取决于订单和商品的组合。
    
- `product_name` 只取决于 `product_id`，不需要依赖整个联合主键。
    

因此，可以把商品信息拆分到商品表中，避免同一商品名称在多条订单明细中重复保存。

### 2.3 第三范式（3NF）

在满足第二范式的基础上，要求非主属性不传递依赖于候选码，即非主属性不应通过另一个非主属性间接依赖于候选码。

例如：

|student_id|class_id|class_name|
|---|---|---|
|1|10|计算机一班|
|2|10|计算机一班|

如果 `class_name` 只由 `class_id` 决定，那么班级名称实际上依赖于班级 ID，而不是直接依赖于学生 ID。

可以拆分为学生表和班级表：

- 学生表：`student_id`、`class_id`
    
- 班级表：`class_id`、`class_name`
    

这样，修改班级名称时只需要更新一处。

**总结：**

- 1NF：字段值具有原子性。
    
- 2NF：消除非主属性对候选码的部分依赖。
    
- 3NF：消除非主属性对候选码的传递依赖。
    

范式是设计参考，不代表所有业务都必须机械地拆分到完全相同的程度。

## 3. MySQL 连表查询

连表查询用于根据表之间的关联条件，查询多张表中的数据。

假设有两张表：

`employees`：

|id|name|department_id|
|---|---|---|
|1|小明|10|
|2|小红|NULL|

`departments`：

|id|name|
|---|---|
|10|技术部|
|20|财务部|

### 3.1 INNER JOIN：内连接

只返回两张表中能够匹配的记录。

```
SELECT e.name, d.name AS department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.id;
```

结果：

|name|department_name|
|---|---|
|小明|技术部|

小红没有匹配的部门，因此不会出现在结果中。

### 3.2 LEFT JOIN：左外连接

返回左表的所有记录，以及右表中匹配的记录。右表没有匹配时，对应字段为 `NULL`。

```
SELECT e.name, d.name AS department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.id;
```

结果包括小明和小红，小红的 `department_name` 为 `NULL`。

### 3.3 RIGHT JOIN：右外连接

返回右表的所有记录，以及左表中匹配的记录。

```
SELECT e.name, d.name AS department_name
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.id;
```

结果包括技术部和财务部。财务部没有匹配的员工，因此员工姓名为 `NULL`。

### 3.4 FULL OUTER JOIN：全外连接

返回两张表的所有记录，包括匹配和不匹配的记录。

MySQL 不直接支持 `FULL OUTER JOIN`，可以通过 `LEFT JOIN`、`RIGHT JOIN` 和 `UNION ALL` 等方式组合实现。

```
SELECT e.name AS employee_name,
       d.name AS department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.id

UNION ALL

SELECT e.name AS employee_name,
       d.name AS department_name
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.id
WHERE e.id IS NULL;
```

第二部分只补充右表中没有匹配左表的记录，避免重复返回已经匹配的记录。

### 3.5 总结

|类型|返回哪些记录|
|---|---|
|`INNER JOIN`|两表匹配的记录|
|`LEFT JOIN`|左表全部记录，以及右表匹配记录|
|`RIGHT JOIN`|右表全部记录，以及左表匹配记录|
|`FULL OUTER JOIN`|两表全部记录，MySQL 不直接支持|

注意：`ON` 用于指定两张表如何匹配，`WHERE` 用于过滤查询结果。对于外连接，把条件放在 `ON` 还是 `WHERE` 中，可能会改变结果。

## 4. 如何避免重复插入数据？

### 4.1 使用 UNIQUE 唯一约束

这是保证数据唯一性的关键方式。

```
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(100)
);
```

这里 `email` 具有唯一约束，不能插入两个相同的非空邮箱。

**为什么不能只先查询再插入？**

```
SELECT * FROM users WHERE email = 'a@example.com';
```

即使先查询发现邮箱不存在，并发请求也可能同时查到不存在，然后同时插入。因此，单纯的「先查再插」不能保证并发情况下的唯一性。

### 4.2 使用 `INSERT ... ON DUPLICATE KEY UPDATE`

发生主键或唯一键冲突时，更新已有记录。

```
INSERT INTO users (email, name)
VALUES ('a@example.com', '小明') AS new_row
ON DUPLICATE KEY UPDATE name = new_row.name;
```

这里使用 MySQL 8.0.19 及之后支持的新行别名写法。

### 4.3 使用 `INSERT IGNORE`

遇到重复键等可忽略的错误时，可以跳过相应插入。

```
INSERT IGNORE INTO users (email, name)
VALUES ('a@example.com', '小明');
```

注意：`IGNORE` 可能将部分数据错误降为警告，并不只处理重复键。

**总结：** 唯一约束负责保证数据唯一，`ON DUPLICATE KEY UPDATE` 和 `INSERT IGNORE` 用于处理冲突时的不同业务需求。

## 5. CHAR 和 VARCHAR 的区别

两者都是字符串类型，但存储方式不同。

|对比|`CHAR(n)`|`VARCHAR(n)`|
|---|---|---|
|类型|定长字符串|变长字符串|
|长度含义|最多 n 个字符|最多 n 个字符|
|存储特点|通常按固定长度处理，不足部分会补空格|根据实际内容长度存储，并额外记录长度|
|适用场景|长度比较固定的数据|长度变化较大的数据|

例如：

```
CHAR(10)
VARCHAR(10)
```

两者都以 10 个字符作为声明长度上限，但实际存储方式不同。

`VARCHAR` 通常需要额外的长度记录空间，具体开销与最大长度有关。

**注意：**

- `VARCHAR(10)` 中的 10 表示字符数，不是字节数。
    
- 使用 `utf8mb4` 时，一个字符最多可以占用 4 个字节。
    
- 不能简单认为 `CHAR` 一定比 `VARCHAR` 快，应该根据实际数据特点选择。
    

## 6. VARCHAR 后面的数字代表什么？

例如：

```
name VARCHAR(10)
```

这里的 `10` 表示最多存储 10 个字符，而不是 10 个字节。

假设使用 `utf8mb4` 字符集，某些字符最多需要 4 个字节，因此 10 个字符的内容最多可能占用 40 个字节，此外还需要长度记录空间。

实际可声明长度还受到字符集、行大小等限制。

## 7. INT(1) 和 INT(10) 有什么区别？

在 MySQL 中，旧版本的 `INT(1)` 和 `INT(10)` 中的数字表示显示宽度，而不是存储长度或数值范围。

- `INT` 本身占用 4 个字节。
    
- `INT(1)` 和 `INT(10)` 的整数存储范围相同。
    
- 显示宽度不会限制数字的位数。
    

整数显示宽度和 `ZEROFILL` 从 MySQL 8.0.17 开始被弃用。

**实际开发中直接使用** `**INT**` **即可。** 如果需要显示 `0001` 这样的编号，通常应由应用层格式化，或者根据业务需求使用字符串类型。

## 8. TEXT 数据类型可以无限大吗？

不可以。MySQL 提供了不同容量的 TEXT 类型：

|类型|最大长度|
|---|---|
|`TINYTEXT`|255 字节|
|`TEXT`|65,535 字节，约 64 KB|
|`MEDIUMTEXT`|16,777,215 字节，约 16 MB|
|`LONGTEXT`|4,294,967,295 字节，约 4 GB|

这里的限制按字节计算，不是字符数。

实际存储和传输还会受到字符集、`max_allowed_packet`、内存及网络开销等因素影响。

如果存储的是图片、视频等大型文件，通常可以考虑使用对象存储保存文件，在数据库中只保存文件地址和相关元数据。

## 9. 为什么金额建议使用 DECIMAL？

金额计算通常需要精确的小数结果，因此一般建议使用 `DECIMAL`，而不是 `FLOAT` 或 `DOUBLE`。

- `DECIMAL`：精确的十进制数值类型。
    
- `FLOAT`、`DOUBLE`：近似数值类型，部分十进制小数不能被精确表示。
    

例如：

```
price DECIMAL(10, 2)
```

其中：

- `10`：总精度，即总位数。
    
- `2`：小数位数。
    
- 整数部分最多 8 位，小数部分最多 2 位。
    

也可以用整数保存最小货币单位，例如用 `BIGINT` 保存分。

注意：即使数据库使用 `DECIMAL`，应用层也应使用精确数值类型进行金额运算，并明确舍入规则，避免先在应用层产生浮点误差。

## 10. IP 地址如何存储？

### 10.1 使用字符串存储

例如：

```
CREATE TABLE ip_records (
    id INT PRIMARY KEY AUTO_INCREMENT,
    ip_address VARCHAR(15)
);

INSERT INTO ip_records (ip_address)
VALUES ('192.168.1.1');
```

优点：直观，方便查看和处理。

缺点：占用空间相对较多，进行数值范围比较时不够方便。

### 10.2 使用整数存储 IPv4

IPv4 地址本质上是 32 位数据，可以转换成无符号整数。

```
CREATE TABLE ip_records (
    id INT PRIMARY KEY AUTO_INCREMENT,
    ip_address INT UNSIGNED
);

INSERT INTO ip_records (ip_address)
VALUES (INET_ATON('192.168.1.1'));

SELECT INET_NTOA(ip_address)
FROM ip_records;
```

- `INET_ATON()`：将 IPv4 地址转换为整数。
    
- `INET_NTOA()`：将整数转换回 IPv4 地址。
    

这种方式节省空间，也便于数值比较。

如果需要同时支持 IPv4 和 IPv6，可以考虑 `VARBINARY(16)`，配合 `INET6_ATON()` 和 `INET6_NTOA()` 进行转换。

## 11. 什么是外键约束？

外键用于维护表之间的引用关系，避免子表引用不存在的父表记录。

例如，学生和课程之间是多对多关系，可以设计三张表：

```
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE courses (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE student_courses (
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

`student_courses` 是中间表，用于记录学生选修了哪些课程。

如果插入的 `student_id` 或 `course_id` 在对应的父表中不存在，数据库就会拒绝这条记录。

### 删除父表记录时的处理方式

|选项|作用|
|---|---|
|`RESTRICT`|存在关联记录时，拒绝删除|
|`CASCADE`|删除父记录时，级联删除子记录|
|`SET NULL`|将子记录的外键设置为 `NULL`，要求字段允许为空|

外键可以将引用完整性检查交给数据库，但也会增加约束检查和锁管理开销。实际项目中是否使用，需要结合业务、数据一致性要求和架构设计决定。

## 12. IN 和 EXISTS 的区别

两者都可以用于子查询，但关注点不同。

### 12.1 IN：判断值是否属于一个集合

```
SELECT *
FROM customers
WHERE country IN ('中国', '新加坡');
```

表示只查询 `country` 为中国或新加坡的客户。

也可以配合子查询：

```
SELECT *
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
);
```

### 12.2 EXISTS：判断子查询是否存在结果

```
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

表示只查询至少有一笔订单的客户。

`EXISTS` 不关心子查询返回什么值，只关心是否至少返回一行。

### 12.3 两者的区别

|对比|`IN`|`EXISTS`|
|---|---|---|
|判断内容|某个值是否属于结果集合|子查询是否至少返回一行|
|常见用途|集合成员判断|判断关联记录是否存在|
|性能|取决于索引、数据量和执行计划|同样取决于索引、数据量和执行计划|

不能简单地认为 `IN` 一定比 `EXISTS` 快，或者反过来。MySQL 优化器可能对两者采用类似的执行策略。

**注意** `**NULL**`**：** `NOT IN` 特别容易受到子查询中 `NULL` 值的影响。例如：

```
SELECT 1 NOT IN (2, NULL);
```

结果是 `NULL`，而不是 `TRUE`。

如果要查询没有订单的客户，可以使用 `NOT EXISTS`：

```
SELECT c.*
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

## 13. MySQL 常用函数

### 13.1 字符串函数

|函数|作用|示例|
|---|---|---|
|`CONCAT()`|拼接字符串|`CONCAT('Hello', 'World')`|
|`LENGTH()`|返回字符串的字节数|`LENGTH('abc')`|
|`CHAR_LENGTH()`|返回字符串的字符数|`CHAR_LENGTH('abc')`|
|`SUBSTRING()`|截取子字符串|`SUBSTRING('Hello', 1, 3)`|
|`REPLACE()`|替换字符串中的内容|`REPLACE('abc', 'a', 'x')`|

注意：在多字节字符集下，`LENGTH()` 和 `CHAR_LENGTH()` 的结果可能不同。

例如，使用 `utf8mb4` 时：

```
SELECT LENGTH('小林'), CHAR_LENGTH('小林');
```

通常结果分别为 6 和 2。

### 13.2 数值函数

|函数|作用|示例|
|---|---|---|
|`ABS()`|绝对值|`ABS(-10)`|
|`POWER()`|幂运算|`POWER(2, 3)`|

### 13.3 日期和时间函数

|函数|作用|
|---|---|
|`NOW()`|返回当前日期和时间|
|`CURDATE()`|返回当前日期|

例如：

```
SELECT NOW();
SELECT CURDATE();
```

### 13.4 聚合函数

聚合函数用于对多条记录进行统计或计算。

|函数|作用|
|---|---|
|`COUNT()`|统计数量|
|`SUM()`|求和|
|`AVG()`|求平均值|
|`MAX()`|求最大值|
|`MIN()`|求最小值|

例如：

```
SELECT
    COUNT(*) AS order_count,
    SUM(price) AS total_price,
    AVG(price) AS average_price,
    MAX(price) AS max_price,
    MIN(price) AS min_price
FROM orders;
```

## 14. COUNT(*)、COUNT(1) 和 COUNT(字段) 的区别

```
SELECT COUNT(*) FROM users;
SELECT COUNT(1) FROM users;
SELECT COUNT(name) FROM users;
```

|写法|含义|
|---|---|
|`COUNT(*)`|统计行数，包括字段为 `NULL` 的行|
|`COUNT(1)`|对每行统计一个非 `NULL` 常量，结果与 `COUNT(*)` 相同|
|`COUNT(name)`|只统计 `name` 不为 `NULL` 的行|

例如：

|id|name|
|---|---|
|1|小明|
|2|NULL|
|3|小红|

那么：

- `COUNT(*)` 的结果是 3。
    
- `COUNT(1)` 的结果是 3。
    
- `COUNT(name)` 的结果是 2。
    

通常不需要为了性能专门将 `COUNT(*)` 改成 `COUNT(1)`。对于 InnoDB，统计总行数通常需要扫描可见记录，具体扫描哪个索引由优化器决定。

## 15. NULL 和空字符串的区别

`**NULL**` **表示值缺失或未知，空字符串** `**''**` **则表示值存在，但内容为空。**

例如：

|name|含义|
|---|---|
|`NULL`|没有提供或没有记录姓名|
|`''`|姓名字段的内容是空字符串|
|`'小明'`|姓名是小明|

查询时：

```
-- 查询 name 为 NULL 的记录
SELECT * FROM users WHERE name IS NULL;

-- 查询 name 为空字符串的记录
SELECT * FROM users WHERE name = '';
```

判断 `NULL` 必须使用 `IS NULL` 或 `IS NOT NULL`，不能使用 `= NULL`。

原因是普通比较运算遇到 `NULL`，结果通常是 `UNKNOWN`，而 `WHERE` 只保留结果为 `TRUE` 的记录。

MySQL 还提供 NULL 安全等于运算符 `<=>`，用于将两个 `NULL` 也判断为相等。

## 16. DELETE、TRUNCATE 和 DROP 的区别

|对比|`DELETE`|`TRUNCATE`|`DROP`|
|---|---|---|---|
|作用|删除记录|清空整张表|删除整张表|
|是否支持 `WHERE`|支持|不支持|不适用|
|表结构是否保留|保留|保留|不保留|
|自增计数器|通常不会重置|通常会重置|表被删除|
|事务回滚|InnoDB 中，在事务提交前可以回滚|不能通过普通事务回滚撤销|不能通过普通事务回滚撤销|

### 16.1 DELETE

删除符合条件的记录：

```
DELETE FROM users
WHERE id = 1;
```

对于 InnoDB，`DELETE` 是事务操作，会产生相应的日志和记录变更，并且会逐行触发 `DELETE` 触发器。

### 16.2 TRUNCATE

清空整张表：

```
TRUNCATE TABLE users;
```

表结构保留，但数据被清空。通常会重置自增计数器，不会逐行触发 `DELETE` 触发器。

### 16.3 DROP

删除整张表：

```
DROP TABLE users;
```

表的数据和表结构都不再保留。

**注意：** MySQL 中 `TRUNCATE` 和普通 `DROP TABLE` 属于 DDL 操作，会隐式提交，不能像普通 InnoDB 数据修改那样通过 `ROLLBACK` 撤销。

## 17. SQL 查询语句的逻辑执行顺序

SQL 的书写顺序与逻辑处理顺序不同。

### 17.1 逻辑处理顺序

```
FROM / JOIN
    ↓
ON
    ↓
WHERE
    ↓
GROUP BY
    ↓
HAVING
    ↓
SELECT
    ↓
DISTINCT
    ↓
ORDER BY
    ↓
LIMIT
```

这是用于理解 SQL 的逻辑顺序，不代表 MySQL 内部必须严格按照这个顺序执行每一步。优化器可以调整实际执行计划。

### 17.2 WHERE 和 HAVING 的区别

- `WHERE`：筛选分组之前的记录。
    
- `HAVING`：筛选分组之后的结果，常用于聚合条件。
    

例如，查询订单数量不少于 3 的用户：

```
SELECT user_id, COUNT(*) AS order_count
FROM orders
WHERE status = 'paid'
GROUP BY user_id
HAVING COUNT(*) >= 3
ORDER BY order_count DESC;
```

执行逻辑：

1. `FROM`：读取订单表。
    
2. `WHERE`：只保留已支付的订单。
    
3. `GROUP BY`：按照用户 ID 分组。
    
4. `HAVING`：只保留订单数量不少于 3 的用户组。
    
5. `SELECT`：输出用户 ID 和订单数量。
    
6. `ORDER BY`：按照订单数量降序排序。
    

注意：`SELECT` 中定义的别名通常不能直接用于 `WHERE`，因为逻辑上 `WHERE` 先于 `SELECT` 处理。

## 18. SQL 练习：查询有 02 课程、没有 01 课程的学生

假设有两张表：

- `Student`：学生信息。
    
- `Score`：学生的课程成绩，包含 `sid`、`cid`、`score`。
    

目标：查询选了 02 课程，但没有选 01 课程的学生及其 02 课程成绩。

### 方法一：使用 LEFT JOIN

```
SELECT s.sid, sc2.score
FROM Student s
LEFT JOIN Score sc1
    ON s.sid = sc1.sid AND sc1.cid = '01'
LEFT JOIN Score sc2
    ON s.sid = sc2.sid AND sc2.cid = '02'
WHERE sc1.sid IS NULL
  AND sc2.sid IS NOT NULL;
```

思路：

1. 通过第一次 `LEFT JOIN` 查找学生是否选了 01 课程。
    
2. 通过第二次 `LEFT JOIN` 查找学生是否选了 02 课程。
    
3. `sc1.sid IS NULL`：没有匹配到 01 课程。
    
4. `sc2.sid IS NOT NULL`：匹配到了 02 课程。
    

这里使用 `sid` 判断是否匹配成功，前提是课程记录的 `sid` 不为 `NULL`。

### 方法二：使用 EXISTS 和 NOT EXISTS

```
SELECT s.sid
FROM Student s
WHERE EXISTS (
    SELECT 1
    FROM Score sc
    WHERE sc.sid = s.sid
      AND sc.cid = '02'
)
AND NOT EXISTS (
    SELECT 1
    FROM Score sc
    WHERE sc.sid = s.sid
      AND sc.cid = '01'
);
```

思路：

- `EXISTS`：至少存在一条 02 课程记录。
    
- `NOT EXISTS`：不存在任何 01 课程记录。
    

这种写法直接表达业务条件，通常更容易理解。

## 19. SQL 基础总结

1. **SQL 与 NoSQL**：根据数据模型、事务需求、查询模式和扩展方式选择。
    
2. **数据库范式**：1NF 强调字段原子性，2NF 消除部分依赖，3NF 消除传递依赖。
    
3. **连表查询**：理解内连接、左外连接、右外连接和 MySQL 不直接支持的全外连接。
    
4. **防止重复插入**：使用主键或唯一约束保证唯一性，再根据需求选择冲突处理方式。
    
5. **数据类型**：理解 `CHAR`、`VARCHAR`、`INT`、`TEXT`、`DECIMAL` 的特点和限制。
    
6. **约束**：主键、唯一约束、外键分别承担不同的完整性要求。
    
7. **IN 和 EXISTS**：分别侧重集合成员判断与子查询结果是否存在。
    
8. **聚合函数**：区分 `COUNT(*)`、`COUNT(1)` 和 `COUNT(字段)`。
    
9. **NULL**：使用 `IS NULL` 判断，不要将 `NULL` 和空字符串混为一谈。
    
10. **删除操作**：区分 `DELETE`、`TRUNCATE` 和 `DROP`。
    
11. **SQL 逻辑执行顺序**：重点掌握 `WHERE` 在分组前过滤，`HAVING` 在分组后过滤。
    
12. **SQL 题型**：学会将“存在某条件、但不存在另一条件”转换成 `EXISTS`、`NOT EXISTS` 或 `LEFT JOIN` 查询。