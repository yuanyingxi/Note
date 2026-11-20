# 1. SQL 子句与逻辑执行顺序

**题目方向：** 聚合完成后用于过滤分组结果的子句是哪个？

**知识点：SQL 子句**

（1）逻辑执行顺序（**不是书写顺序**）

```text
FROM / JOIN  →  ON  →  WHERE  →  GROUP BY  →  HAVING  →  SELECT  →  DISTINCT  →  ORDER BY  →  LIMIT
   ①              ②       ③          ④           ⑤          ⑥           ⑦            ⑧         ⑨
```

记忆主线：**先找表（FROM）→ 再连表（ON）→ 再筛行（WHERE）→ 再分组（GROUP BY）→ 再筛组（HAVING）→ 再选列（SELECT）→ 排序（ORDER BY）→ 截断（LIMIT）。**

（2）五个子句的分工

|子句|作用|能否用聚合函数|例|
|---|---|---|---|
|`WHERE`|**分组前**筛**行**|❌ 不能|`WHERE age > 18`|
|`GROUP BY`|按列分组|—|`GROUP BY class`|
|**`HAVING`**|**分组后**筛**组**|✅ 能|`HAVING COUNT(*) > 5`|
|`ORDER BY`|排序输出|✅ 能（对结果排序）|`ORDER BY cnt DESC`|
|`LIMIT`|限制返回行数|❌|`LIMIT 10`|
|`DISTINCT`|去重|❌|`SELECT DISTINCT class`|

（3）为什么 `WHERE` 里不能写聚合函数

执行到 `WHERE` 时**还没分组**，`COUNT(*)` 无从算起。这不是「语法规定」，而是**执行顺序的必然结果**——理解了顺序就不用背这条规则。

（4）标准写法示例

```sql
SELECT class, COUNT(*) AS cnt
FROM Student
WHERE grade >= 60          -- 先筛行
GROUP BY class             -- 再分组
HAVING COUNT(*) > 5        -- 再筛组
ORDER BY cnt DESC;         -- 最后排序
```

（5）一句话

**`WHERE` 管行，`HAVING` 管组；先 `WHERE` 后 `GROUP`，先 `GROUP` 后 `HAVING`。**

**所属知识点：** 计算机基础 / 数据库 / SQL 子句。

# 2. 英文例题

Which SQL clause is used to filter **groups** after an aggregation has been performed?

A. WHERE

B. HAVING

C. ORDER BY

D. DISTINCT

E. LIMIT

**答案：B**

**解析：** 题干两个关键词 `groups`（分组后）与 `after an aggregation`（聚合已发生）直接锁定 `HAVING`。`WHERE` 在分组前过滤行，且不能用聚合函数。

**所属知识点：** 计算机基础 / 数据库 / SQL 子句。
