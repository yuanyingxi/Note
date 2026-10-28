# 1. SQL 和 NoSQL 的区别
## 1.1 SQL 数据库

**SQL** 数据库通常指**关系型数据库**，主要代表有 MySQL、PostgreSQL、Oracle 等。

关系型数据库以表的形式组织数据：

- 行：表示一条记录，例如一个用户。
    
- 列：表示记录的属性，例如用户 ID、姓名、年龄。
    
- 表之间：可以通过主键、外键等建立关联。
    

适合需要事务、结构化数据和复杂关联查询的业务场景。

### 1.2 NoSQL 数据库

**NoSQL** 指**非关系型数据库**，常见的有 Redis、MongoDB 等。

不同 NoSQL 数据库可以采用不同的数据组织方式，例如：

- Redis：字符串、哈希表、列表、集合等数据结构。
    
- MongoDB：文档，通常以类似 JSON 的 BSON 格式组织数据。
    

适合缓存、文档存储等特定场景。

## 1.3 如何选择？

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

# 2. 数据库三大范式
## 2.0 候选码（Candidate Key）

**候选码是能够唯一确定一条记录，并且不包含多余属性的最小属性集合。**

理解候选码需要掌握两个条件：

- **唯一性**：通过候选码的值，可以唯一确定一条记录。
    
- **最小性**：候选码中的任何一个属性都不能删除，否则就无法保证唯一确定一条记录。
    

例如，有一张学生表：

|student_id|id_card|name|
|---|---|---|
|1|A001|小明|
|2|A002|小红|

假设每个学生的 `student_id` 和 `id_card` 都是唯一的，那么：

- `student_id` 可以唯一确定一个学生，因此它是候选码。
    
- `id_card` 也可以唯一确定一个学生，因此它也是候选码。
    
- `(student_id, id_card)` 虽然也能唯一确定一个学生，但其中任意一个字段单独就能做到，因此这个组合不是最小的，不属于候选码。
    

**候选码与主键的关系：**

- 候选码：所有满足唯一性和最小性的属性集合。
    
- 主键（Primary Key）：从候选码中选定的一个，用来作为表中记录的主要标识。
    
- 联合候选码：由多个属性组合而成的候选码。
    

例如，订单明细表中，假设 `(order_id, product_id)` 能唯一确定一条订单明细，且两个字段单独都不能唯一确定记录，那么它就是一个联合候选码。如果选用它作为记录的主键，它同时也是联合主键。

注意：**候选码不一定只有一个**；**主键则只能选定一个**（可以由多个字段组成）。
数据库范式用于规范表结构，减少数据冗余和更新异常。

## 2.1 第一范式（1NF）

要求表中的**每个字段值具有原子性**，即**每个字段存储一个符合该字段定义的值**，不应把需要独立处理的多个数据项塞进同一个字段。

例如，不推荐这样设计：

|id|name|phone_numbers|
|---|---|---|
|1|小明|13800000000, 13900000000|

如果需要单独查询、修改每个电话号码，将多个号码放在一个字段里就不方便。

可以拆分为两个不同的表，一个存储用户信息，一个用来存储电话号码：

|id|name|
|---|---|
|1|小明|

|user_id|phone|
|---|---|
|1|13800000000|
|1|13900000000|

## 2.2 第二范式（2NF）

第二范式要求：**在满足第一范式的基础上，每个非主属性都必须完全依赖（依赖，即唯一确定）于候选码，不能只依赖于候选码的一部分。**

先理解两个概念：

- **联合主键**：由**多个字段**共同组成**主键**，用来唯一确定一条记录。
    
- **非主属性**：**不属于任何候选码的属性**。初学时可以先简单理解为不属于主键的普通字段。
    

### 示例：订单明细表

假设我们开发一个购物系统，用户可以下订单，每个订单可以购买多种商品。

例如，订单 1 购买了键盘和鼠标：

|order_id|product_id|product_name|quantity|
|---|---|---|---|
|1|101|键盘|2|
|1|102|鼠标|1|
|2|101|键盘|1|

字段含义：

- `order_id`：订单编号。
    
- `product_id`：商品编号。
    
- `product_name`：商品名称。
    
- `quantity`：该订单购买该商品的数量。
    

为什么使用 `(order_id, product_id)` 作为联合主键？

因为同一个订单可以包含多种商品，同一种商品也可以出现在多个订单中。**订单编号和商品编号组合起来，才能唯一确定某个订单中的某种商品。**

例如，`(1, 101)` 表示订单 1 中的键盘，`(1, 102)` 表示订单 1 中的鼠标。

### 为什么这张表不满足第二范式？

重点看两个字段：

**1.** `quantity`**（购买数量）**

购买数量取决于某个订单中的某种商品。

例如：

- 订单 1 买了 2 个键盘。
    
- 订单 2 买了 1 个键盘。
    

仅知道订单编号，无法确定购买数量；仅知道商品编号，也无法确定购买数量。必须同时知道订单和商品，才能确定对应的购买数量。

因此，`quantity` 依赖于整个联合主键 `(order_id, product_id)`，符合第二范式对完全依赖的要求。

**2.** `product_name`**（商品名称）**

商品名称只由商品编号决定：

- 商品编号 `101` 对应键盘。
    
- 商品编号 `102` 对应鼠标。
    

无论商品出现在订单 1 还是订单 2 中，商品编号 `101` 都对应键盘。

因此，`product_name` 只依赖于联合主键的一部分 `product_id`，而不需要依赖 `order_id`。

这就属于**部分依赖**，违反了第二范式。

### 如何修改，才能满足第二范式？

把原来的表拆成两张表。

**第一张：订单明细表** `order_detail`

只保留与订单和商品组合有关的信息：

|order_id|product_id|quantity|
|---|---|---|
|1|101|2|
|1|102|1|
|2|101|1|

联合主键仍然是 `(order_id, product_id)`。其中，`quantity` 完全依赖于整个联合主键。

**第二张：商品表** `product`

单独存储商品自身的信息：

|product_id|product_name|
|---|---|
|101|键盘|
|102|鼠标|

这里，`product_id` 是主键，`product_name` 由 `product_id` 决定。

这样，商品名称不再重复保存在每一条订单明细中。

### 总结

第二范式主要解决的问题是：**当主键由多个字段组成时，普通字段不能只依赖其中一部分。**

判断时可以问自己：

> 这个字段是否必须知道联合主键的所有部分，才能确定它的值？

- 如果必须知道所有部分，可能满足第二范式的要求。
    
- 如果只需要知道其中一部分，就存在部分依赖，需要考虑拆表。
    

注意：**如果一张表的所有候选码都是单字段主键，就不存在对联合主键的部分依赖，因此在满足第一范式等前提下，不会因为部分依赖而违反第二范式**。

## 2.3 第三范式（3NF）

第三范式要求：**在满足第二范式的基础上，非主属性不能传递依赖于候选码。**

简单来说，就是：**普通字段应该直接依赖于候选码，而不应该依赖于另一个普通字段，再间接依赖于候选码。**

#### 示例：学生表

假设一张学生表存储了学生的学号、班级编号和班级名称：

|student_id|class_id|class_name|
|---|---|---|
|1|10|计算机一班|
|2|10|计算机一班|
|3|20|软件工程一班|

字段含义：

- `student_id`：学生的学号，是这张表的主键。
    
- `class_id`：学生所属的班级编号。
    
- `class_name`：班级名称。
    

#### 为什么不满足第三范式？

先看这几个字段之间的依赖关系：

1. `student_id` 决定 `class_id`：通过学生学号，可以知道学生属于哪个班级。
    
2. `class_id` 决定 `class_name`：通过班级编号，可以知道班级名称。
    

因此，依赖关系是：

`student_id → class_id → class_name`

也就是说，`class_name` 并不是直接由学生学号决定的，而是先通过学号找到班级编号，再通过班级编号找到班级名称。

这就是**传递依赖**：

- `class_id` 是非主属性，它依赖于主键 `student_id`。
    
- `class_name` 依赖于非主属性 `class_id`，从而间接依赖于主键 `student_id`。
    

所以，这张表存在非主属性对主键的传递依赖，不满足第三范式。

#### 如何修改，才能满足第三范式？

将原来的表拆分为学生表和班级表。

**第一张：学生表** `student`

|student_id|class_id|
|---|---|
|1|10|
|2|10|
|3|20|

- `student_id` 是主键。
    
- `class_id` 表示学生所属的班级，也是关联班级表的外键。
    

**第二张：班级表** `class`

|class_id|class_name|
|---|---|
|10|计算机一班|
|20|软件工程一班|

- `class_id` 是主键。
    
- `class_name` 由 `class_id` 决定。
    

拆分后，学生表只记录学生与班级之间的关系，班级表专门存储班级信息。

例如，计算机一班更名时，只需要修改班级表中的一条记录，不需要逐条修改所有学生记录中的班级名称，从而减少数据冗余和更新异常。

#### 总结：数据库三大范式

- **第一范式（1NF）**：字段值具有原子性，每个字段存储一个符合其定义的值。
    
- **第二范式（2NF）**：消除非主属性对候选码的部分依赖。
    
- **第三范式（3NF）**：消除非主属性对候选码的传递依赖。
    

范式是数据库设计的参考原则。实际开发中，还需要结合查询性能、业务需求等因素决定是否进行适当的反规范化设计。

# 3. MySQL 连表查询

`JOIN` 用于根据指定条件，将两张表中的相关记录组合起来查询。

假设有两张表：员工表 `employees` 和部门表 `departments`。

**员工表** `employees`

|id|name|department_id|
|---|---|---|
|1|小明|10|
|2|小红|20|
|3|小李|NULL|

其中，`department_id` 表示员工所属的部门编号。小李没有分配部门。

**部门表** `departments`

|id|name|
|---|---|
|10|技术部|
|20|产品部|
|30|财务部|

其中，`id` 是部门编号。财务部目前没有员工。

连接条件通常写成：

```sql
ON e.department_id = d.id
```

表示员工的 `department_id` 与部门的 `id` 相等时，两条记录匹配。

这里的 `e` 和 `d` 分别是 `employees` 和 `departments` 的表别名；`AS department_name` 则是给查询结果中的列起别名，不会修改原表。

## 3.1 INNER JOIN：内连接

**只返回两张表中能够匹配的记录。**

```sql
SELECT e.name AS employee_name,
       d.name AS department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.id;
```

查询结果：

|employee_name|department_name|
|---|---|
|小明|技术部|
|小红|产品部|

解释：

- 小明的 `department_id = 10`，匹配技术部。
    
- 小红的 `department_id = 20`，匹配产品部。
    
- 小李没有部门编号，无法匹配任何部门，因此不出现在结果中。
    
- 财务部没有员工，也不会出现在结果中。
    

**`INNER JOIN` 也可以简写为 `JOIN`。**

## 3.2 LEFT JOIN：左外连接

**保留左表的所有记录；右表有匹配记录时一起返回，没有匹配时，右表对应字段为 `NULL`。**

```sql
SELECT e.name AS employee_name,
       d.name AS department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.id;
```

查询结果：

|employee_name|department_name|
|---|---|
|小明|技术部|
|小红|产品部|
|小李|NULL|

解释：

- 小明、小红都能匹配对应部门。
    
- 小李没有匹配的部门，但由于 `employees` 是左表，仍然保留小李这条记录。
    
- 因为找不到对应部门，`department_name` 为 `NULL`。
    
- 财务部没有员工，不会单独出现在结果中，因为它属于右表。
    

**记忆：左表全部保留，右表按条件匹配。**

## 3.3 RIGHT JOIN：右外连接

**保留右表的所有记录；左表有匹配记录时一起返回，没有匹配时，左表对应字段为 `NULL`。**

```sql
SELECT e.name AS employee_name,
       d.name AS department_name
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.id;
```

查询结果：

| employee_name | department_name |
| ------------- | --------------- |
| 小明            | 技术部             |
| 小红            | 产品部             |
| NULL          | 财务部             |

解释：

- 技术部匹配到小明。
    
- 产品部匹配到小红。
    
- 财务部没有匹配的员工，但由于 `departments` 是右表，仍然保留财务部这条记录。
    
- 找不到对应员工，因此 `employee_name` 为 `NULL`。
    
- 小李没有匹配的部门，且他属于左表，所以不会出现在结果中。
    

**记忆：右表全部保留，左表按条件匹配。**

### 3.4 FULL OUTER JOIN：全外连接

**保留两张表的所有记录：匹配的组合在一起返回，不匹配的记录也保留，另一张表对应的字段为 `NULL`。**

标准 SQL 可以写成：

```sql
SELECT e.name AS employee_name,
       d.name AS department_name
FROM employees e
FULL OUTER JOIN departments d
    ON e.department_id = d.id;
```

预期查询结果：

|employee_name|department_name|
|---|---|
|小明|技术部|
|小红|产品部|
|小李|NULL|
|NULL|财务部|

解释：

- 小明、小红：能够匹配部门，正常返回。
    
- 小李：没有匹配的部门，但仍然保留。
    
- 财务部：没有匹配的员工，也仍然保留。
    

**注意：MySQL 不直接支持 `FULL OUTER JOIN`，需要通过其他 SQL 组合实现。**

一种实现方式是使用 `LEFT JOIN` 和 `RIGHT JOIN` 配合 `UNION ALL`：

- `UNION ALL`：合并两条查询的结果，并保留重复行。
    
- `WHERE`：筛选符合条件的记录。
    
- `IS NULL`：判断字段是否为空值。
    
- `IS NOT NULL`：判断字段是否不为空值。

```sql
-- 第一部分：保留所有员工及其匹配的部门
SELECT e.name AS employee_name,
       d.name AS department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.id

UNION ALL

-- 第二部分：补充没有匹配员工的部门
SELECT e.name AS employee_name,
       d.name AS department_name
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.id
WHERE e.id IS NULL;
```

为什么第二部分要加 `WHERE e.id IS NULL`？

因为第一部分已经包含了所有匹配成功的记录。第二部分只需要补充那些没有匹配员工的部门，避免把已匹配的记录重复返回。

这里 `e.id` 是员工表的主键，不会为真实存在的员工记录取 `NULL`。因此，`e.id IS NULL` 可以识别没有匹配员工的部门。

## 3.5 四种 JOIN 的区别总结

| 连接方式              | 保留哪些记录    | 没有匹配时            |
| ----------------- | --------- | ---------------- |
| `INNER JOIN`      | 两边都能匹配的记录 | 不保留不匹配记录         |
| `LEFT JOIN`       | 左表全部记录    | 右表字段为 `NULL`     |
| `RIGHT JOIN`      | 右表全部记录    | 左表字段为 `NULL`     |
| `FULL OUTER JOIN` | 两张表的全部记录  | 未匹配一侧的字段为 `NULL` |

可以记住：

- `INNER`：只要匹配的。
    
- `LEFT`：左边的全部保留。
    
- `RIGHT`：右边的全部保留。
    
- `FULL`：两边的全部保留。

## 3.6 `AS`：指定别名

`AS` 用于给查询结果中的列或表起别名，方便阅读和引用，不会修改数据库中原本的字段名或表名。

#### 给查询结果的列起别名

```sql
SELECT d.name AS department_name
FROM departments d;
```

- `d.name`：查询 `departments` 表中的 `name` 字段。
    
- `AS department_name`：将查询结果中这一列的列名设为 `department_name`。
    

例如，原表数据：

|id|name|
|---|---|
|10|技术部|
|20|产品部|

查询结果：

|department_name|
|---|
|技术部|
|产品部|

`SELECT` **返回查询结果集**，其中包含查询得到的数据和对应的列名。`department_name` 是结果集中的列名，不是原表中新增的字段。

#### 给表起别名

```sql
SELECT e.name
FROM employees AS e;
```

- `employees AS e`：将 `employees` 表简称为 `e`。
    
- `e.name`：引用员工表中的 `name` 字段。
    

在 MySQL 中，表别名的 `AS` 可以省略：

```sql
FROM employees e
```

列别名的 `AS` 通常也可以省略：

```sql
SELECT d.name department_name
FROM departments d;
```

#### 别名的使用限制

列别名可以在部分 SQL 子句中继续使用，但并非所有地方都可以。

```sql
-- 通常可以：按照列别名排序
SELECT d.name AS department_name
FROM departments d
ORDER BY department_name;

-- 不可以：WHERE 中不能直接使用 SELECT 定义的列别名
SELECT d.name AS department_name
FROM departments d
WHERE department_name = '技术部';

-- 正确写法：使用原始字段筛选
SELECT d.name AS department_name
FROM departments d
WHERE d.name = '技术部';
```

原因是 **`WHERE` 在逻辑执行顺序上先于 `SELECT`**，因此执行 `WHERE` 时，`SELECT` 中定义的列别名还不可用；`ORDER BY` 则可以使用该列别名。

**总结：**

- 列别名：主要用于命名查询结果中的列，例如 `d.name AS department_name`。
    
- 表别名：用于简化表名、区分不同表中的同名字段，例如 `employees AS e`。
    
- 别名不会修改数据库原有的表结构或数据。

# 4. 如何避免重复插入数据？

在数据库中，避免重复数据通常需要结合**唯一约束**和**插入时的冲突处理方式**。

## 4.1 使用 UNIQUE 唯一约束

`UNIQUE` 用于限制某个字段或字段组合的值不能重复。

例如，用户表中的邮箱不允许重复：

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    name VARCHAR(50)
);
```

语法说明：

- `CREATE TABLE users`：创建 `users` 表。
    
- `PRIMARY KEY`：设置主键，用于唯一标识一条记录。
    
- `UNIQUE`：设置唯一约束，防止不同记录使用相同的邮箱。
    

假设表中已经存在：

|id|email|name|
|---|---|---|
|1|[a@example.com](mailto:a@example.com)|小明|

再执行下面的语句：

```sql
INSERT INTO users (id, email, name)
VALUES (2, 'a@example.com', '小红');
```

由于邮箱违反唯一约束，插入会失败，原有记录不会被自动修改。

**注意：** 避免重复数据，首先应该通过数据库约束保证数据唯一，而不是只依赖程序提前查询。因为多个请求可能同时检查到邮箱不存在，然后同时尝试插入。

## 4.2 使用 INSERT ... ON DUPLICATE KEY UPDATE：冲突时更新

如果业务要求：**记录不存在就插入，记录已存在就更新**，可以使用这种语法。

```sql
INSERT INTO users (id, email, name)
VALUES (2, 'a@example.com', '小红')
ON DUPLICATE KEY UPDATE name = '小红';
```

语法说明：

- `INSERT INTO users (id, email, name)`：指定要插入数据的表和字段。
    
- `VALUES (2, 'a@example.com', '小红')`：指定准备插入的字段值，依次对应前面列出的字段。
    
- `ON DUPLICATE KEY UPDATE`：如果插入时发生主键或唯一索引冲突，就执行后面的更新操作。
    
- `name = '小红'`：将冲突记录的 `name` 更新为 `小红`。
    

假设邮箱 `a@example.com` 已经属于小明，执行后不会新增第二个用户，而是把已有记录的姓名改为小红。

如果没有发生主键或唯一索引冲突，则正常插入新记录。

**注意：** 这种语法并不是只检查邮箱。只要发生主键或唯一索引冲突，就可能触发**更新**，因此要谨慎设计更新字段，避免误修改已有数据。

## 4.3 使用 INSERT IGNORE：冲突时跳过插入

如果业务要求：**遇到重复记录时保留原数据，不更新它**，可以使用 `INSERT IGNORE`。

```sql
INSERT IGNORE INTO users (id, email, name)
VALUES (2, 'a@example.com', '小红');
```

语法说明：

- `INSERT IGNORE INTO`：尝试插入数据，遇到某些可忽略的错误时，不按普通插入错误的方式中止处理。
    
- `VALUES (...)`：指定准备插入的数据。
    

假设邮箱 `a@example.com` 已经属于小明，那么这次插入会因邮箱重复而被跳过，原有记录仍然是小明，不会被更新。

**注意：** `IGNORE` 不仅影响重复键错误，还可能将部分数据错误降为警告。因此，不要把它当作通用的错误处理方式；使用时应检查警告和实际插入结果。

## 4.4 两种冲突处理方式的区别

假设邮箱 `a@example.com` 已经属于小明，现在尝试插入同邮箱、姓名为小红的数据。

|写法|遇到重复邮箱时|原有姓名|
|---|---|---|
|普通 `INSERT`|插入失败|小明|
|`INSERT ... ON DUPLICATE KEY UPDATE`|更新已有记录|小红|
|`INSERT IGNORE`|跳过重复插入|小明|

## 4.5 补充：AS new_row 是什么？

MySQL 8.0.19 及之后的版本支持在 `INSERT ... ON DUPLICATE KEY UPDATE` 中给待插入的新数据起别名。

例如：

```sql
INSERT INTO users (email, name)
VALUES ('a@example.com', '小红') AS new_row
ON DUPLICATE KEY UPDATE name = new_row.name;
```

其中：

- `AS new_row`：给本次准备插入的数据起别名。
    
- `new_row.name`：引用新数据中的 `name` 值，也就是 `小红`。
    
- `name = new_row.name`：发生冲突时，将已有记录的 `name` 更新为新数据中的姓名。
    

这种写法适合在更新时引用本次插入的数据。初学阶段先理解其作用即可，不必优先记忆这类细节。

### 总结

- `UNIQUE`：从数据库约束层面防止重复值，必须理解。
    
- **普通** `INSERT`：尝试插入新记录，违反唯一约束时会报错。
    
- `ON DUPLICATE KEY UPDATE`：发生主键或唯一索引冲突时更新已有记录。
    
- `INSERT IGNORE`：遇到重复键等可忽略错误时跳过相关插入，但也可能掩盖其他数据问题。
    
- `AS new_row`：给待插入的数据起别名，方便在冲突更新时引用新值。
    

实际使用哪一种方式，取决于业务需求：冲突后需要报错、更新，还是保留旧数据并跳过插入。

# 5. CHAR 和 VARCHAR 的区别

两者都是字符串类型，但存储方式不同。

| 对比   | `CHAR(n)`          | `VARCHAR(n)`       |
| ---- | ------------------ | ------------------ |
| 类型   | 定长字符串              | 变长字符串              |
| 长度含义 | 最多 n 个字符           | 最多 n 个字符           |
| 存储特点 | 通常按固定长度处理，不足部分会补空格 | 根据实际内容长度存储，并额外记录长度 |
| 适用场景 | **长度比较固定的数据**          | **长度变化较大的数据**          |

例如：

```sql
CHAR(10)
VARCHAR(10)
```

两者都以 10 个字符作为声明长度上限，但实际存储方式不同。

`VARCHAR` 通常需要额外的长度记录空间，具体开销与最大长度有关。

**注意：**

- `VARCHAR(10)` 中的 10 表示字符数，不是字节数。
    
- 使用 `utf8mb4` 时，一个字符最多可以占用 4 个字节。
    
- 不能简单认为 `CHAR` 一定比 `VARCHAR` 快，应该根据实际数据特点选择。
    

# 6. VARCHAR 后面的数字代表什么？

例如：

```sql
name VARCHAR(10)
```

这里的 `10` 表示**最多存储 10 个字符**，而不是 10 个字节。

假设使用 `utf8mb4` 字符集，**某些字符最多需要 4 个字节**，因此 10 个字符的内容最多可能占用 40 个字节，此外还需要长度记录空间。

实际可声明长度还受到字符集、行大小等限制。

# 7. TEXT 数据类型可以无限大吗？

不可以。MySQL 提供了不同容量的 TEXT 类型：

| 类型           | 最大长度                    |
| ------------ | ----------------------- |
| `TINYTEXT`   | 255 字节                  |
| `TEXT`       | 65,535 字节，约 64 KB       |
| `MEDIUMTEXT` | 16,777,215 字节，约 16 MB   |
| `LONGTEXT`   | 4,294,967,295 字节，约 4 GB |

这里的限制按字节计算，不是字符数。

实际存储和传输还会受到字符集、`max_allowed_packet`、内存及网络开销等因素影响。

如果存储的是图片、视频等大型文件，通常可以考虑使用对象存储保存文件，在数据库中只保存文件地址和相关元数据。

#### 大型文件的存储方式：对象存储（OSS）

图片、视频等大型文件通常不直接存入数据库，而是保存到**对象存储服务**（如阿里云 OSS）中，数据库只保存文件地址和相关元数据。

例如：

**1. OSS 中存储文件**

```
OSS：
  images/avatar.jpg
  videos/demo.mp4
```

**2. MySQL 中保存文件信息**

|id|file_name|file_url|
|---|---|---|
|1|avatar.jpg|`https://example.com/images/avatar.jpg`|
|2|demo.mp4|`https://example.com/videos/demo.mp4`|

其中：

- `file_name`：文件名称。
    
- `file_url`：文件访问地址，用于定位或访问 OSS 中的文件。
    

**基本流程：**

1. 用户上传图片或视频。
    
2. 后端将文件上传到 OSS。
    
3. OSS 返回文件标识或访问地址。
    
4. 后端将文件地址及相关信息保存到 MySQL。
    
5. 需要查看文件时，系统根据数据库中保存的信息访问 OSS 获取文件。
    

**优点：** 减少数据库存储大型文件的压力，便于文件管理、扩展和分发。

**注意：** 文件地址不一定是永久公开的 URL，也可能是对象 Key 或临时签名 URL。实际项目中常将对象 Key 保存到数据库，需要访问时再生成临时访问地址。

# 8. 为什么金额建议使用 DECIMAL？

**金额计算通常需要精确的小数结果，因此一般建议使用 `DECIMAL`，而不是 `FLOAT` 或 `DOUBLE`。**

- **`DECIMAL`：精确的十进制数值类型。**
    
- **`FLOAT`、`DOUBLE`：近似数值类型，部分十进制小数不能被精确表示。**
    

例如：

```sql
price DECIMAL(10, 2)
```

其中：

- `10`：总精度，即总位数。
    
- `2`：小数位数。
    
- 整数部分最多 8 位，小数部分最多 2 位。
    

也可以用整数保存最小货币单位，例如用 `BIGINT` 保存分。

注意：即使数据库使用 `DECIMAL`，应用层也应使用精确数值类型进行金额运算，并明确舍入规则，避免先在应用层产生浮点误差。

# 9. IP 地址如何存储？
## 9.1 使用字符串存储

例如：

```sql
CREATE TABLE ip_records (
    id INT PRIMARY KEY AUTO_INCREMENT,
    ip_address VARCHAR(15)
);

INSERT INTO ip_records (ip_address)
VALUES ('192.168.1.1');
```

优点：直观，方便查看和处理。

缺点：占用空间相对较多，进行数值范围比较时不够方便。

## 9.2 使用整数存储 IPv4

**IPv4 地址本质上是 32 位数据，可以转换成无符号整数。**

```sql
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

# 10. 什么是外键约束？
### 10.1 外键的作用

**外键（Foreign Key）用于建立两张表之间的引用关系，保证子表引用的父表记录真实存在。**

例如，一个学生可以选修多门课程，一门课程也可以被多个学生选修。这是多对多关系，可以通过三张表实现。

## 10.2 创建三张表

**第一张：学生表** `students`

```sql
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
```

示例数据：

|id|name|
|---|---|
|1|小明|
|2|小红|

**第二张：课程表** `courses`

```sql
CREATE TABLE courses (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
```

示例数据：

|id|name|
|---|---|
|101|Java|
|102|MySQL|

**第三张：学生选课表** `student_courses`

```sql
CREATE TABLE student_courses (
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

这张表是**中间表**，用于记录每个学生选修了哪些课程。

例如：

|student_id|course_id|
|---|---|
|1|101|
|1|102|
|2|101|

表示：

- 小明选修了 Java 和 MySQL。
    
- 小红选修了 Java。
    

## 10.3 REFERENCES 是什么意思？

重点看这段 SQL：

```sql
FOREIGN KEY (student_id) REFERENCES students(id)
```

可以拆成三部分理解：

- `FOREIGN KEY (student_id)`：指定 `student_id` 是外键字段。
    
- `REFERENCES students(id)`：指定它引用 `students` 表中的 `id` 字段。
    
- `students(id)`：其中 `students` 是被引用的表，`id` 是被引用的字段。
    

因此，这条约束的意思是：

`student_courses.student_id` **中的值，必须能够在** `students.id` **中找到对应记录。**

例如：

假设 `students` 表中只有 `id = 1` 和 `id = 2`，那么：

```sql
-- 合法：学生 1 存在，课程 101 也存在
INSERT INTO student_courses (student_id, course_id)
VALUES (1, 101);

-- 非法：学生 3 不存在
INSERT INTO student_courses (student_id, course_id)
VALUES (3, 101);

-- 非法：课程 999 不存在
INSERT INTO student_courses (student_id, course_id)
VALUES (1, 999);
```

后两条插入会违反外键约束，因此数据库会拒绝执行。

这里需要注意：`REFERENCES` 本身不是单独执行的查询语句，而是外键定义中的关键字，用来声明**当前表的字段引用哪张表的哪个字段**。

## 10.4 外键和主键有什么关系？

| 对比          | 主键（PRIMARY KEY） | 外键（FOREIGN KEY）              |
| ----------- | --------------- | ---------------------------- |
| 主要作用        | 唯一标识本表中的一条记录    | 维护表之间的引用关系                   |
| 是否允许重复      | 不允许             | 通常允许重复                       |
| 是否允许 `NULL` | 不允许             | 取决于字段定义及约束条件                 |
| 示例          | `students.id`   | `student_courses.student_id` |

例如，多个学生选课记录可以使用相同的 `student_id = 1`，因为一个学生可以选修多门课程。

但 `students.id` 必须唯一，因为它用于区分不同的学生。

此外，外键引用的字段通常需要是被引用表的主键或具有适用唯一约束的键。上面的例子中，`students.id` 和 `courses.id` 都是主键。

## 10.5 删除父表记录时怎么办？

**外键不仅会检查插入和更新，也可以限制或处理删除父表记录的操作。**

例如，小明已经有选课记录。如果直接删除小明，那么 `student_courses` 中的相关记录就会失去对应的学生。

可以通过外键的删除规则决定如何处理：

|选项|作用|
|---|---|
|`RESTRICT`|存在关联记录时，拒绝删除父表记录|
|`CASCADE`|删除父表记录时，自动删除关联的子表记录|
|`SET NULL`|删除父表记录时，将子表对应的外键设为 `NULL`，要求该字段允许为空|

例如，可以这样定义外键：

```sql
FOREIGN KEY (student_id)
REFERENCES students(id)
ON DELETE CASCADE
```

其中：

- **`ON DELETE`：指定删除被引用记录时的处理规则。**
    
- **`CASCADE`：级联删除相关选课记录。**
    

需要注意，级联删除可能导致多条关联数据被一并删除，使用前必须确认符合业务要求。

## 10.6 实际开发中一定要使用外键吗？

外键可以让数据库自动检查引用关系，避免子表引用不存在的父表记录。

但在实际项目中，是否使用外键需要结合架构和业务考虑：

- **使用外键**：数据库自动维护引用完整性，减少无效关联数据。
    
- **不使用外键**：应用程序需要自行保证数据一致性，可能更方便进行分库分表或跨服务设计，但实现和维护成本也会增加。
    

外键检查还会带来一定的约束检查和锁管理开销，具体影响取决于数据库和业务负载。

**总结：** `FOREIGN KEY` 用于定义外键，`REFERENCES` 用于指定被引用的表和字段。外键的核心作用是维护表之间的引用关系，而 `ON DELETE` 等规则则决定删除父表记录时如何处理相关子表记录。

# 11. IN 和 EXISTS 的区别

两者都可以用于子查询，但关注点不同。

## 11.1 IN：判断值是否属于一个集合
### 11.1.1 什么是 IN？

`IN` 用于判断某个字段的值是否在指定的多个值中。

```sql
SELECT *
FROM customers
WHERE country IN ('中国', '新加坡');
```

含义：查询 `customers` 表中，`country` 为中国或新加坡的所有客户。

上面的 SQL 相当于：

```sql
SELECT *
FROM customers
WHERE country = '中国'
   OR country = '新加坡';
```

因此，`IN` 可以简化多个 `OR` 条件。

### 11.1.2 什么是子查询？

**子查询（Subquery）就是写在另一条 SQL 语句内部的查询语句。**

例如，我们有两张表：

**customers（客户表）**

|customer_id|name|
|---|---|
|1|小明|
|2|小红|
|3|小李|

**orders（订单表）**

|order_id|customer_id|
|---|---|
|101|1|
|102|3|

其中，`orders` 表的 `customer_id` 表示订单属于哪个客户。

现在，如果我们想查询**所有下过订单的客户**，可以先查询订单表，找出下过订单的客户 ID：

```sql
SELECT customer_id
FROM orders;
```

查询结果为：

|customer_id|
|---|
|1|
|3|

然后，再根据这些 ID 查询客户表，就能找到小明和小李。

我们可以把这两步合并成一条 SQL：

```sql
SELECT *
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
);
```

这里有两条查询语句：

- **外层查询**：`SELECT * FROM customers WHERE ...`，负责从客户表中找出符合条件的客户。
    
- **内层查询（子查询）**：`SELECT customer_id FROM orders`，负责查出订单表中出现过的客户 ID。
    

内层查询放在括号中，作为外层查询的条件，因此称为**子查询**。

### 11.1.3 子查询和 IN 是怎么配合的？

可以把它理解为以下过程：

1. 子查询从 `orders` 表中查出客户 ID：`1、3`。
    
2. `IN` 判断 `customers` 表中每个客户的 ID 是否属于 `1、3`。
    
3. 只保留符合条件的客户，最终查出小明和小李。
    

因此，这条 SQL 的含义是：

**查询那些客户 ID 出现在订单表中的客户。**

### 11.1.4 总结

| 概念         | 含义                  |
| ---------- | ------------------- |
| `IN`       | 判断一个值是否属于指定的多个值     |
| 子查询        | 嵌套在另一条 SQL 语句内部的查询  |
| `IN (子查询)` | 判断某个值是否属于子查询返回的结果集合 |

注意：子查询不一定要和 `IN` 一起使用，后续学习其他 SQL 语法时，还会遇到子查询的其他用法。

## 11.2 EXISTS：判断是否存在符合条件的记录

**`EXISTS` 用于判断子查询是否至少返回一条记录：**

- **存在符合条件的记录：返回 `TRUE`，保留当前记录。**
    
- **不存在符合条件的记录：返回 `FALSE`，排除当前记录。**
    

例如，查询所有至少有一笔订单的客户：

```sql
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

### 如何理解这条 SQL？

假设客户表中有小明、小红、小李：

- 小明有订单，符合条件，保留。
    
- 小红没有订单，不符合条件，排除。
    
- 小李有订单，符合条件，保留。
    

最终只返回小明和小李。

其中：

- `WHERE EXISTS (...)`：检查括号中的查询是否存在结果。
    
- `o.customer_id = c.customer_id`：检查订单是否属于当前正在判断的客户。
    
- `SELECT 1`：只需要检查有没有符合条件的记录，不需要获取订单的具体字段。这里的 `1` 只是一个固定值。

#### 总结

`EXISTS` **关心的是“有没有”，而不是“有多少”或“具体是什么”。**

例如，查询有订单的客户时，客户有 1 笔订单还是 100 笔订单，只要存在符合条件的订单，就能通过 `EXISTS` 的判断。

## 11.3 IN、NOT IN 与 EXISTS 的区别
### 11.3.1 IN 和 NOT IN

- `IN`：判断值是否在指定的集合中。
    
- `NOT IN`：判断值是否不在指定的集合中。
    

例如：

```sql
SELECT 1 IN (1, 2, 3);      -- TRUE
SELECT 1 NOT IN (1, 2, 3);  -- FALSE
SELECT 4 NOT IN (1, 2, 3);  -- TRUE
```

### 11.3.2 为什么 NOT IN 遇到 NULL 会有问题？

先理解一个概念：`NULL` **表示未知或缺失的值，不能简单地把它当成普通数字或字符串。**

例如：

```sql
SELECT 1 NOT IN (2, NULL);
```

这相当于判断：

> 数字 1 是否既不等于 2，也不等于 NULL？

**我们知道，1 不等于 2；但是，`NULL` 的值未知，我们无法确定 1 是否等于它。**

因此，整个判断结果是 `NULL`（未知），而不是 `TRUE`。

可以对比下面三个例子：

| SQL 表达式              | 结果      | 原因               |
| -------------------- | ------- | ---------------- |
| `1 NOT IN (2, 3)`    | `TRUE`  | 1 不在集合中          |
| `1 NOT IN (1, 2)`    | `FALSE` | 1 在集合中           |
| `1 NOT IN (2, NULL)` | `NULL`  | 因为存在未知值，无法确定整体结果 |

在 SQL 的 `WHERE` 条件中，只有结果为 `TRUE` 的记录才会被保留；结果为 `FALSE` 或 `NULL` 的记录都会被过滤掉。

因此，如果 `NOT IN` 的子查询结果中包含 `NULL`，就可能导致原本想查询的记录也被过滤掉。

### 11.3.3 NOT EXISTS 如何避免这个问题？

例如，查询没有下过订单的客户。

**使用 NOT IN：**

```sql
SELECT *
FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id
    FROM orders
);
```

如果 `orders.customer_id` 中存在 `NULL`，这个查询可能无法返回预期的客户。

**使用 NOT EXISTS：**

```sql
SELECT *
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

这段 SQL 的意思是：

- 检查订单表中是否存在属于当前客户的订单。
    
- 如果存在，`NOT EXISTS` 为 `FALSE`，排除该客户。
    
- 如果不存在，`NOT EXISTS` 为 `TRUE`，保留该客户。
    

这里判断的是是否存在满足 `o.customer_id = c.customer_id` 的订单。订单表中其他记录的 `customer_id` 即使为 `NULL`，也不会仅仅因为这个 `NULL` 就让整个 `NOT EXISTS` 判断失效。

### 11.3.4 总结

| 写法           | 作用             | 需要注意的地方              |
| ------------ | -------------- | -------------------- |
| `IN`         | 判断值是否在集合中      | 适合判断是否匹配             |
| `NOT IN`     | 判断值是否不在集合中     | 子查询结果含 `NULL` 时要特别小心 |
| `EXISTS`     | 判断是否存在符合条件的记录  | 关注有没有匹配记录            |
| `NOT EXISTS` | 判断是否不存在符合条件的记录 | 常用于查询没有关联记录的数据       |

**记忆重点：** 查询“没有关联记录”的数据时，`NOT EXISTS` 通常更稳妥，不容易受到子查询结果中 `NULL` 的影响。

# 12. MySQL 常用函数
## 12.1 字符串函数

|函数|作用|示例|
|---|---|---|
|`CONCAT()`|拼接字符串|`CONCAT('Hello', 'World')`|
|`LENGTH()`|返回字符串的字节数|`LENGTH('abc')`|
|`CHAR_LENGTH()`|返回字符串的字符数|`CHAR_LENGTH('abc')`|
|`SUBSTRING()`|截取子字符串|`SUBSTRING('Hello', 1, 3)`|
|`REPLACE()`|替换字符串中的内容|`REPLACE('abc', 'a', 'x')`|

注意：在多字节字符集下，`LENGTH()` 和 `CHAR_LENGTH()` 的结果可能不同。

例如，使用 `utf8mb4` 时：

```sql
SELECT LENGTH('小林'), CHAR_LENGTH('小林');
```

通常结果分别为 6 和 2。

## 12.2 数值函数

| 函数        | 作用  | 示例            |
| --------- | --- | ------------- |
| `ABS()`   | 绝对值 | `ABS(-10)`    |
| `POWER()` | 幂运算 | `POWER(2, 3)` |

## 12.3 日期和时间函数

| 函数          | 作用        |
| ----------- | --------- |
| `NOW()`     | 返回当前日期和时间 |
| `CURDATE()` | 返回当前日期    |

例如：

```sql
SELECT NOW();
SELECT CURDATE();
```

## 12.4 聚合函数

聚合函数用于对多条记录进行统计或计算。

|函数|作用|
|---|---|
|`COUNT()`|统计数量|
|`SUM()`|求和|
|`AVG()`|求平均值|
|`MAX()`|求最大值|
|`MIN()`|求最小值|

例如：

```sql
SELECT
    COUNT(*) AS order_count,
    SUM(price) AS total_price,
    AVG(price) AS average_price,
    MAX(price) AS max_price,
    MIN(price) AS min_price
FROM orders;
```

# 13. COUNT(\*)、COUNT(1) 和 COUNT(字段) 的区别

```sql
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

# 14. NULL 和空字符串的区别

`NULL` **表示值缺失或未知，空字符串** `''` **则表示值存在，但内容为空。**

例如：

|name|含义|
|---|---|
|`NULL`|没有提供或没有记录姓名|
|`''`|姓名字段的内容是空字符串|
|`'小明'`|姓名是小明|

查询时：

```sql
-- 查询 name 为 NULL 的记录
SELECT * FROM users WHERE name IS NULL;

-- 查询 name 为空字符串的记录
SELECT * FROM users WHERE name = '';
```

判断 `NULL` 必须使用 `IS NULL` 或 `IS NOT NULL`，不能使用 `= NULL`。

原因是普通比较运算遇到 `NULL`，结果通常是 `UNKNOWN`，而 `WHERE` 只保留结果为 `TRUE` 的记录。

MySQL 还提供 NULL 安全等于运算符 `<=>`，用于将两个 `NULL` 也判断为相等。

# 15. DELETE、TRUNCATE 和 DROP 的区别

|对比|`DELETE`|`TRUNCATE`|`DROP`|
|---|---|---|---|
|作用|删除记录|清空整张表|删除整张表|
|是否支持 `WHERE`|支持|不支持|不适用|
|表结构是否保留|保留|保留|不保留|
|自增计数器|通常不会重置|通常会重置|表被删除|
|事务回滚|InnoDB 中，在事务提交前可以回滚|不能通过普通事务回滚撤销|不能通过普通事务回滚撤销|

## 15.1 DELETE

删除符合条件的记录：

```sql
DELETE FROM users
WHERE id = 1;
```

对于 InnoDB，`DELETE` 是事务操作，会产生相应的日志和记录变更，并且会逐行触发 `DELETE` 触发器。

## 15.2 TRUNCATE

清空整张表：

```sql
TRUNCATE TABLE users;
```

表结构保留，但数据被清空。通常会重置自增计数器，不会逐行触发 `DELETE` 触发器。

## 15.3 DROP

删除整张表：

```sql
DROP TABLE users;
```

表的数据和表结构都不再保留。

**注意：** MySQL 中 `TRUNCATE` 和普通 `DROP TABLE` 属于 DDL 操作，会隐式提交，不能像普通 InnoDB 数据修改那样通过 `ROLLBACK` 撤销。

# 16. GROUP BY 和 HAVING：分组与分组筛选
## 16.1 GROUP BY：对数据进行分组

**`GROUP BY` 用于按照指定字段，将具有相同字段值的记录划分到同一组中。**

例如，`orders`（订单表）包含以下数据：

|order_id|user_id|amount|
|---|---|---|
|101|1|100|
|102|1|200|
|103|2|150|
|104|2|80|
|105|3|300|

如果想统计每个用户有多少笔订单，可以使用 `GROUP BY`：

```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id;
```

查询结果：

|user_id|order_count|
|---|---|
|1|2|
|2|2|
|3|1|

这里涉及几个概念：

- `GROUP BY user_id`：将 `user_id` 相同的订单划分到同一组。
    
- `COUNT(*)`：统计每组中有多少条记录。
    
- `AS order_count`：给查询结果中的统计列起别名，方便阅读。
    

例如，用户 1 有两笔订单，因此分到用户 1 的组中有两条记录，`COUNT(*)` 的结果就是 2。

**注意：** 分组后，查询结果通常是一组对应一行，而不是把每一笔原始订单都单独显示出来。

## 16.2 常见的聚合函数

**聚合函数用于对一组记录进行统计或计算，经常与 `GROUP BY` 配合使用。**

|函数|作用|示例|
|---|---|---|
|`COUNT(*)`|统计记录数量|统计订单数|
|`SUM(amount)`|计算总和|统计订单总金额|
|`AVG(amount)`|计算平均值|计算平均订单金额|
|`MAX(amount)`|计算最大值|找出最大订单金额|
|`MIN(amount)`|计算最小值|找出最小订单金额|

例如，统计每个用户的订单总金额：

```sql
SELECT user_id, SUM(amount) AS total_amount
FROM orders
GROUP BY user_id;
```

查询结果：

|user_id|total_amount|
|---|---|
|1|300|
|2|230|
|3|300|

这里的 `SUM(amount)` 会分别计算每个用户组中所有订单金额的总和。

## 16.3 HAVING：筛选分组后的结果

**`HAVING` 用于筛选分组后的结果，通常与 `GROUP BY` 配合使用。**

例如，查询订单数量不少于 2 笔的用户：

```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 2;
```

查询结果：

|user_id|order_count|
|---|---|
|1|2|
|2|2|

执行过程：

1. `FROM orders`：读取订单表。
    
2. `GROUP BY user_id`：按照用户 ID 分组。
    
3. `COUNT(*)`：统计每个用户的订单数量。
    
4. `HAVING COUNT(*) >= 2`：只保留订单数量不少于 2 笔的用户组。
    
5. `SELECT`：输出用户 ID 和订单数量。
    

用户 3 只有一笔订单，不符合条件，因此被排除。

## 16.4 WHERE 和 HAVING 的区别

两者都能筛选数据，但筛选的**阶段**不同。

| 对比项      | WHERE             | HAVING          |
| -------- | ----------------- | --------------- |
| 筛选对象     | 分组前的原始记录          | 分组后的结果          |
| 执行逻辑     | 在 `GROUP BY` 之前   | 在 `GROUP BY` 之后 |
| 常见用途     | 筛选符合条件的订单         | 筛选符合统计条件的用户组    |
| 能否用于聚合条件 | 通常不能直接筛选当前查询的聚合结果 | 可以              |

例如，查询已支付订单数量不少于 2 笔的用户：

```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
WHERE status = 'paid'
GROUP BY user_id
HAVING COUNT(*) >= 2;
```

假设 `status` 表示订单状态，`paid` 表示已支付。

这条 SQL 的含义是：

- `WHERE status = 'paid'`：先排除未支付的订单。
    
- `GROUP BY user_id`：再按照用户分组。
    
- `HAVING COUNT(*) >= 2`：最后筛选出已支付订单不少于 2 笔的用户。
    

**记忆方法：**

- **`WHERE`：先筛选哪些原始记录可以参与统计。**
    
- **`GROUP BY`：把符合条件的记录分组。**
    
- **`HAVING`：再筛选哪些组可以出现在最终结果中。**
    

## 16.5 综合示例

需求：查询已支付订单总金额不少于 300 的用户，并按照总金额从高到低排序。

```sql
SELECT user_id, SUM(amount) AS total_amount
FROM orders
WHERE status = 'paid'
GROUP BY user_id
HAVING SUM(amount) >= 300
ORDER BY total_amount DESC;
```

各部分的作用：

|SQL 部分|作用|
|---|---|
|`FROM orders`|指定订单表|
|`WHERE status = 'paid'`|只统计已支付订单|
|`GROUP BY user_id`|按用户分组|
|`HAVING SUM(amount) >= 300`|只保留订单总金额不少于 300 的用户组|
|`SELECT`|输出用户 ID 和订单总金额|
|`ORDER BY total_amount DESC`|按总金额降序排列，金额高的排在前面|

**其中，`DESC` 表示降序，`ASC` 表示升序。`ORDER BY` 默认按照升序排列。**

# 17. SQL 查询语句的逻辑执行顺序

SQL 的书写顺序与逻辑处理顺序不同。
## 17.0 DISTINCT 和 LIMIT

#### DISTINCT：去除重复记录

`DISTINCT` 用于去除查询结果中的重复行。

例如，查询订单表中所有不重复的用户 ID：

```sql
SELECT DISTINCT user_id
FROM orders;
```

如果多个订单属于同一个用户，该用户的 ID 只会在结果中出现一次。

#### LIMIT：限制返回的记录数量

`LIMIT` 用于限制查询结果最多返回多少行。

例如，查询订单表中金额最高的 3 笔订单：

```sql
SELECT *
FROM orders
ORDER BY amount DESC
LIMIT 3;
```

- `ORDER BY amount DESC`：按照订单金额降序排列。
    
- `LIMIT 3`：只返回排序后的前 3 条记录。

## 17.1 逻辑处理顺序

```text
FROM / JOIN / AS(表别名)：确定查询哪些表
    ↓
ON：确定表之间如何关联
    ↓
WHERE：筛选原始记录
    ↓
GROUP BY：对记录分组
    ↓
HAVING：筛选分组结果
    ↓
SELECT / AS(列别名)：选择要输出的列或计算结果
    ↓
DISTINCT：去除重复行
    ↓
ORDER BY：排序
    ↓
LIMIT：限制返回的行数
```

这是用于理解 SQL 的逻辑顺序，不代表 MySQL 内部必须严格按照这个顺序执行每一步。优化器可以调整实际执行计划。

# 18. SQL 练习：查询有 02 课程、没有 01 课程的学生

假设有两张表：

- `Student`：学生信息。
    
- `Score`：学生的课程成绩，包含 `sid`、`cid`、`score`。
    

目标：查询选了 02 课程，但没有选 01 课程的学生及其 02 课程成绩。

### 方法一：使用 LEFT JOIN

```sql
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

```sql
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

-- 或者（我更喜欢）
SELECT s.sid  -- 如果同一个学生的同一门课程可能出现多条记录，这里就需要加上 DISTINCT 
FROM Student s
JOIN Score sc
    ON sc.sid = s.sid
WHERE sc.cid = '02'
  AND NOT EXISTS (
      SELECT 1
      FROM Score sc1
      WHERE sc1.sid = s.sid
        AND sc1.cid = '01'
  );
```

思路：

- `EXISTS`：至少存在一条 02 课程记录。
    
- `NOT EXISTS`：不存在任何 01 课程记录。
    

这种写法直接表达业务条件，通常更容易理解。