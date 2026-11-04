# 1. 题目翻译与考点
## 1.1 原题

Write a program (in C, C++, Java or Python) that inputs an array of 100 words and outputs the sorted words according to the alphabetical order. Marks will depend on the correctness, efficiency, and memory usage of your program.

## 1.2 中文翻译

编写一个程序（可以使用 C、C++、Java 或 Python），输入一个包含 100 个单词的数组，并按照字母顺序对这些单词进行排序，最后输出排序结果。评分将根据程序的正确性、运行效率和内存使用情况来决定。

### 1.3 题目关键词

| 英文 | 含义 | 编程中对应的内容 |
|---|---|---|
| input | 输入 | `input()` |
| array | 数组 | Python 中通常使用 `list`（列表） |
| word | 单词 | `str`（字符串） |
| sorted | 已排序的 | 排序结果 |
| alphabetical order | 字母顺序 | 按字符串字典序比较 |
| output | 输出 | `print()` |
| correctness | 正确性 | 结果和边界情况是否正确 |
| efficiency | 效率 | 时间复杂度、运行时间 |
| memory usage | 内存使用 | 是否创建不必要的数据副本 |

# 2. 港大相关笔试题型：公开经验整理

以下是申请者公开分享的回忆，不是学校发布的官方真题，也不代表每年固定题型。

## 2.1 MSc AI 的历史经验

- **2022 年申请者回忆：** 笔试包含微积分、线性代数、概率论，以及一道编程题；编程题涉及反复计算一个数各位数字的平方和，并判断最终是否会得到 1。
- **2023 年申请者回忆：** 笔试包含高等数学、线性代数、概率论，以及一道编程题；编程题是查找数组中缺失的数字。
- **其他 MSc AI 申请者分享：** 曾遇到打印九九乘法表这样的基础编程题。

### 2.2 港大 MSc CS（含 AI 方向）的相近参考

其他计算机硕士项目的申请者回忆中，还出现过：

- 十进制转二进制；
- 递归计算幂、数组逆序；
- 模拟灯泡多轮开关状态；
- 整数转罗马数字；
- 阅读代码并分析功能、时间复杂度；
- 数组、集合、搜索和简单动态规划相关题目。

**注意：MSc CS 和 MSc AI 是不同项目。** CS 的题目只能作为计算机基础能力的参考，不能当作 MSc AI 的固定考试范围。

### 2.3 备考判断

从这些公开回忆来看，编程部分经常涉及基础编程、简单算法和逻辑模拟，不宜只准备一种排序题，也没有证据说明考试只考简单排序。

建议优先掌握：

1. Python 基础语法、输入输出；
2. 字符串和列表操作；
3. 排序、查找、去重、计数；
4. 循环、条件判断、函数；
5. 字典、集合；
6. 常见时间复杂度；
7. 基础递归和简单模拟。

港大 MSc AI 官方招生页面列出的知识基础包括线性代数、微积分、概率论、入门统计学和计算机编程。具体考试题型仍以当年学校通知为准。

## 3. Python 基础概念

### 3.1 变量

变量用于给数据起名字，不需要像 Java 一样提前声明变量类型。

```python
age = 20
name = "Alice"
score = 91.5
is_passed = True
```

### 3.2 常见数据类型

| 类型 | 含义 | 示例 |
|---|---|---|
| `int` | 整数 | `10` |
| `float` | 浮点数 | `3.14` |
| `str` | 字符串 | `"hello"` |
| `bool` | 布尔值 | `True`、`False` |
| `list` | 列表，可修改 | `[1, 2, 3]` |
| `tuple` | 元组，创建后不能修改元素 | `(1, 2, 3)` |
| `set` | 集合，元素不重复 | `{1, 2, 3}` |
| `dict` | 字典，键值对 | `{"name": "Alice"}` |

查看类型：

```python
x = "hello"
print(type(x))       # <class 'str'>
```

### 3.3 注释

```python
# 这是单行注释

"""
这是多行字符串，常用于函数或模块说明。
"""
```

## 4. 输入与输出

### 4.1 `input()`：接收输入

`input()` 从标准输入读取一行内容，返回值始终是字符串 `str`。

```python
name = input("请输入姓名：")
print(name)
```

输入数字时，需要转换类型：

```python
age = int(input("请输入年龄："))
price = float(input("请输入价格："))
```

常见类型转换：

| 函数 | 作用 | 示例 |
|---|---|---|
| `int(x)` | 转整数 | `int("12")` → `12` |
| `float(x)` | 转浮点数 | `float("3.5")` → `3.5` |
| `str(x)` | 转字符串 | `str(100)` → `"100"` |
| `list(x)` | 转列表 | `list("abc")` → `['a', 'b', 'c']` |
| `set(x)` | 转集合、去重 | `set([1, 1, 2])` → `{1, 2}` |

### 4.2 `print()`：输出

```python
print("Hello")
print(10)
print("年龄：", 20)
```

控制分隔符和结尾：

```python
print("A", "B", "C", sep="-")  # A-B-C
print("Hello", end=" ")
print("World")                    # Hello World
```

### 4.3 `split()`：拆分字符串

`split()` 按空白字符拆分字符串，返回一个列表。

```python
line = "apple banana orange"
words = line.split()
print(words)
# ['apple', 'banana', 'orange']
```

指定分隔符：

```python
text = "apple,banana,orange"
items = text.split(",")
```

注意：`input()` 只读取一行。如果输入格式是一行多个单词，可以用 `split()` 拆分；如果每个单词单独占一行，就要逐行读取。

### 4.4 `join()`：拼接字符串

`join()` 把字符串列表连接成一个字符串。

```python
words = ["apple", "banana", "orange"]
result = " ".join(words)
print(result)
# apple banana orange
```

其他分隔符：

```python
print(",".join(words))
# apple,banana,orange
```

记忆：`split()` 是字符串拆成列表；`join()` 是把字符串列表拼起来。

## 5. 字符串 `str` 常用方法

字符串是由字符组成的文本。字符串具有不可变性：调用替换、大小写转换等方法时，通常会返回一个新字符串，不会直接修改原字符串。

### 5.1 大小写转换

```python
s = "Hello Python"

print(s.lower())       # hello python
print(s.upper())       # HELLO PYTHON
print(s.capitalize())  # Hello python
print(s.title())       # Hello Python
print(s.casefold())    # 更适合做不区分大小写的比较
```

常用区别：

- `lower()`：转换为小写。
- `upper()`：转换为大写。
- `casefold()`：用于更彻底的 Unicode 不区分大小写比较；普通英文文本排序时也很常见。

### 5.2 去除首尾空白

```python
s = "  hello  "
print(s.strip())   # 'hello'
print(s.lstrip())  # 去掉左侧空白
print(s.rstrip())  # 去掉右侧空白
```

注意：`strip()` 默认去掉首尾空白，不会删除字符串中间的空格。

### 5.3 查找和替换

```python
s = "hello python"

print(s.find("python"))       # 找到时返回起始下标
print(s.find("java"))         # 找不到返回 -1
print(s.replace("python", "java"))
print(s.count("o"))           # 统计字符 o 出现次数
```

### 5.4 判断字符串内容

```python
print("123".isdigit())   # True，是否都是数字字符
print("abc".isalpha())   # True，是否都是字母字符
print("abc123".isalnum()) # True，是否都是字母或数字
print("hello".startswith("he")) # True，是否以指定内容开头
print("hello".endswith("lo"))   # True，是否以指定内容结尾
```

### 5.5 字符串下标与切片

```python
s = "Python"

print(s[0])     # P，第一个字符
print(s[1])     # y
print(s[-1])    # n，最后一个字符
print(s[0:3])   # Pyt，左闭右开
print(s[:2])    # Py
print(s[2:])    # thon
print(s[::-1])  # nohtyP，反转字符串
```

Python 下标从 `0` 开始；切片 `s[a:b]` 包含 a，不包含 b。

## 6. 列表 `list` 常用操作

列表用于存储多个元素，具有顺序、可以修改、允许重复等特点。它是 Python 中最常见的数据结构之一。

### 6.1 创建列表和访问元素

```python
numbers = [10, 20, 30, 40]

print(numbers[0])    # 10
print(numbers[-1])   # 40
print(len(numbers))  # 4
```

### 6.2 `append()`：在末尾添加一个元素

```python
numbers = [1, 2, 3]
numbers.append(4)
print(numbers)  # [1, 2, 3, 4]
```

### 6.3 `extend()`：在末尾添加多个元素

```python
numbers = [1, 2]
numbers.extend([3, 4])
print(numbers)  # [1, 2, 3, 4]
```

区别：

```python
a = [1, 2]
a.append([3, 4])
print(a)  # [1, 2, [3, 4]]，把整个列表作为一个元素添加

b = [1, 2]
b.extend([3, 4])
print(b)  # [1, 2, 3, 4]，逐个添加元素
```

### 6.4 `insert()`：在指定位置插入元素

```python
numbers = [1, 3, 4]
numbers.insert(1, 2)
print(numbers)  # [1, 2, 3, 4]
```

### 6.5 `remove()`：删除指定值

```python
numbers = [1, 2, 2, 3]
numbers.remove(2)
print(numbers)  # [1, 2, 3]，只删除第一次出现的 2
```

### 6.6 `pop()`：删除并返回元素

```python
numbers = [10, 20, 30]
x = numbers.pop()      # 默认删除最后一个
print(x)               # 30
print(numbers)         # [10, 20]

y = numbers.pop(0)     # 删除下标为 0 的元素
print(y)               # 10
```

### 6.7 `index()` 和 `count()`

```python
numbers = [10, 20, 20, 30]

print(numbers.index(20))  # 1，返回第一次出现的位置
print(numbers.count(20))  # 2，统计出现次数
```

如果使用 `index()` 查找不存在的元素，会抛出 `ValueError`；使用前应确认元素存在，或通过异常处理。

### 6.8 `reverse()`：原地反转列表

```python
numbers = [1, 2, 3]
numbers.reverse()
print(numbers)  # [3, 2, 1]
```

注意：`reverse()` 是修改原列表，不是排序。

### 6.9 `copy()`：浅拷贝

```python
a = [1, 2, 3]
b = a.copy()
b.append(4)

print(a)  # [1, 2, 3]
print(b)  # [1, 2, 3, 4]
```

## 7. 排序：`sorted()` 和 `sort()`（重点）

### 7.1 `sorted()`：返回一个新列表

```python
numbers = [5, 2, 4, 1, 3]
result = sorted(numbers)

print(result)   # [1, 2, 3, 4, 5]
print(numbers)  # [5, 2, 4, 1, 3]，原列表不变
```

特点：

- 可以接收列表、元组、集合等可迭代对象。
- 返回新的有序列表。
- 不改变原始列表。

### 7.2 `list.sort()`：直接修改原列表

```python
numbers = [5, 2, 4, 1, 3]
result = numbers.sort()

print(numbers)  # [1, 2, 3, 4, 5]
print(result)   # None
```

特点：

- 只能直接用于列表。
- 原地排序，修改原列表。
- 返回值是 `None`，不要写成 `numbers = numbers.sort()`，否则 `numbers` 会变成 `None`。

### 7.3 两者区别

| 对比 | `sorted(iterable)` | `list.sort()` |
|---|---|---|
| 是否生成新列表 | 是 | 否，直接修改原列表 |
| 是否修改原数据 | 否 | 是 |
| 可处理对象 | 任意可迭代对象 | 列表 |
| 返回值 | 新列表 | `None` |
| 适用情况 | 需要保留原始数据 | 不需要保留原始顺序，想直接排序 |

### 7.4 升序和降序：`reverse`

```python
numbers = [3, 1, 5, 2]

print(sorted(numbers))                 # [1, 2, 3, 5]
print(sorted(numbers, reverse=True))   # [5, 3, 2, 1]
```

`reverse=True` 表示降序；默认 `False`，表示升序。

### 7.5 按指定规则排序：`key`

`key` 用来指定排序依据。排序时，Python 会先对每个元素计算 key，再根据 key 的结果排序。

```python
words = ["pear", "apple", "banana"]
print(sorted(words, key=len))
# ['pear', 'apple', 'banana']  # 按长度升序
```

按照字符串长度排序：

```python
words = ["a", "banana", "pear", "kiwi"]
print(sorted(words, key=len))
# ['a', 'pear', 'kiwi', 'banana']
```

### 7.6 不区分大小写的字符串排序

普通字符串比较会考虑字符编码顺序，因此大小写混合时，大写字母和小写字母的排序位置可能与日常字典习惯不同。

```python
words = ["banana", "Apple", "cherry", "apple"]

print(sorted(words))
print(sorted(words, key=str.casefold))
```

`key=str.casefold` 表示排序时先按不区分大小写的形式比较。它不会要求把原字符串改成小写，输出仍保留原来的拼写。

如果题目没有说明是否区分大小写，应先确认要求；普通英文单词的字母顺序通常可以考虑不区分大小写的比较。涉及中文、重音符号或特定语言规则时，排序规则可能更复杂。

### 7.7 排序的时间复杂度

Python 内置排序通常使用 Timsort。一般情况下可将时间复杂度记为：

- 平均/常见情况：`O(n log n)`；
- 已经部分有序的数据，可能利用已有顺序而更快；
- 排序是稳定的：如果两个元素的排序 key 相同，它们原来的相对顺序会保留。

对于 100 个单词，内置排序通常足够高效。手写冒泡排序、选择排序等 `O(n²)` 算法通常没有必要，除非题目明确要求实现排序算法本身。

## 8. 循环与条件判断

### 8.1 `if / elif / else`

```python
score = 85

if score >= 90:
    print("A")
elif score >= 80:
    print("B")
else:
    print("C")
```

注意 Python 使用缩进表示代码块，通常使用 4 个空格。

### 8.2 `for` 循环遍历列表

```python
words = ["apple", "banana", "pear"]

for word in words:
    print(word)
```

`word` 每次依次取得列表中的一个元素。

### 8.3 `range()`：生成整数序列

```python
for i in range(5):
    print(i)  # 0 到 4
```

常见形式：

```python
range(5)       # 0, 1, 2, 3, 4
range(1, 5)    # 1, 2, 3, 4
range(1, 10, 2) # 1, 3, 5, 7, 9
```

### 8.4 `enumerate()`：同时获得下标和元素

```python
words = ["apple", "banana", "pear"]

for index, word in enumerate(words):
    print(index, word)
```

适合需要同时知道元素位置和元素内容的场景。

### 8.5 `while` 循环

```python
count = 0
while count < 3:
    print(count)
    count += 1
```

当循环次数不确定、需要满足某个条件后才停止时，经常使用 `while`。

### 8.6 `break` 和 `continue`

- `break`：立即结束当前循环。
- `continue`：跳过本轮剩余代码，进入下一轮循环。

```python
for number in range(5):
    if number == 2:
        continue
    if number == 4:
        break
    print(number)
# 输出 0、1、3
```

## 9. 字典 `dict`：统计频次（重点）

字典存储键值对，常用于计数、查找和建立映射关系。

### 9.1 创建和访问

```python
student = {"name": "Alice", "age": 20}
print(student["name"])  # Alice
```

### 9.2 `get()`：安全获取值

```python
scores = {"Alice": 90}
print(scores.get("Alice", 0))  # 90
print(scores.get("Bob", 0))    # 0，不会因为键不存在而报错
```

### 9.3 统计元素出现次数

```python
words = ["apple", "banana", "apple", "pear"]
count = {}

for word in words:
    count[word] = count.get(word, 0) + 1

print(count)
# {'apple': 2, 'banana': 1, 'pear': 1}
```

这是常见的“哈希表计数”方法，平均情况下每次字典查找和插入通常接近 `O(1)`。

也可以使用 `collections.Counter`：

```python
from collections import Counter

words = ["apple", "banana", "apple", "pear"]
count = Counter(words)
print(count["apple"])  # 2
```

## 10. 集合 `set`：去重和判断存在

集合中的元素不重复，适合去重和成员判断。

### 10.1 去重

```python
numbers = [1, 2, 2, 3, 3, 3]
unique_numbers = set(numbers)
print(unique_numbers)  # {1, 2, 3}
```

注意：集合不承诺像列表那样按原始插入顺序展示。若需要保留列表顺序，可使用其他方式去重。

### 10.2 判断元素是否存在

```python
seen = {"apple", "banana"}
print("apple" in seen)  # True
print("pear" in seen)   # False
```

集合成员判断平均时间复杂度通常为 `O(1)`；列表成员判断需要逐个检查，时间复杂度为 `O(n)`。

## 11. 常见列表/算法操作模板

### 11.1 求长度

```python
numbers = [2, 5, 1]
print(len(numbers))
```

### 11.2 求最大值、最小值、总和

```python
numbers = [2, 5, 1]
print(max(numbers))  # 5
print(min(numbers))  # 1
print(sum(numbers))  # 8
```

### 11.3 判断元素是否存在

```python
numbers = [1, 3, 5]
print(3 in numbers)      # True
print(4 not in numbers)  # True
```

### 11.4 反转列表

```python
numbers = [1, 2, 3]
print(numbers[::-1])  # [3, 2, 1]，返回新列表
```

与 `reverse()` 的区别：切片 `[::-1]` 返回反转后的新列表；`reverse()` 直接修改原列表。

### 11.5 列表推导式

```python
numbers = [1, 2, 3, 4, 5]
squares = [x * x for x in numbers]
print(squares)  # [1, 4, 9, 16, 25]
```

带条件：

```python
even_numbers = [x for x in numbers if x % 2 == 0]
print(even_numbers)  # [2, 4]
```

### 11.6 自定义函数

```python
def square(x):
    return x * x

print(square(5))  # 25
```

- `def`：定义函数。
- 参数 `x`：函数接收的数据。
- `return`：返回计算结果。
- `print()`：只是输出，不等于返回值。

## 12. 时间复杂度速查

时间复杂度描述输入规模增长时，程序运行步骤大致如何增长。

| 复杂度 | 常见含义 | 例子 |
|---|---|---|
| `O(1)` | 常数时间 | 按下标访问列表元素 |
| `O(log n)` | 对数时间 | 二分查找（有序数组） |
| `O(n)` | 线性时间 | 遍历列表一次、线性查找 |
| `O(n log n)` | 线性对数 | 高效排序，如 Python 内置排序的典型复杂度 |
| `O(n²)` | 平方时间 | 两层各遍历 n 次的嵌套循环、冒泡排序 |

判断复杂度的简单方法：

```python
# O(n)：循环一次
for x in numbers:
    print(x)

# O(n^2)：外层 n 次，内层最多 n 次
for x in numbers:
    for y in numbers:
        print(x, y)
```

如果输入规模 n 翻倍：`O(n)` 的工作量大约翻倍；`O(n²)` 的工作量大约变为 4 倍。

## 13. 考试中容易出现的错误

1. **忘记 `input()` 返回字符串。** 输入数字后参与数学运算，需要先用 `int()` 或 `float()` 转换。
2. **混淆 `sort()` 和 `sorted()`。** `sort()` 修改原列表并返回 `None`；`sorted()` 返回新列表。
3. **把 `append()` 和 `extend()` 混淆。** 前者把一个对象作为整体追加；后者逐个追加可迭代对象中的元素。
4. **下标从 0 开始。** 最后一个元素可以用 `list[-1]` 访问。
5. **切片右边界不包含。** `list[0:3]` 只取下标 0、1、2。
6. **字符串方法通常返回新字符串。** 例如 `s.lower()` 不会直接改变 `s`。
7. **不要把 `print()` 当成 `return`。** 前者负责显示，后者负责把结果交给调用者。
8. **注意重复元素。** 普通排序会保留重复元素，不会自动去重。
9. **注意大小写排序规则。** `Apple` 与 `apple` 的比较可能和日常字典习惯不同，可根据题意使用 `key=str.casefold`。
10. **确认输入格式。** 是一行输入多个单词，还是每个单词单独一行，会影响读取方式。

## 14. 常见英文编程题词汇

| 英文 | 中文 |
|---|---|
| ascending order | 升序 |
| descending order | 降序 |
| alphabetical order | 字母顺序 |
| lexicographical order | 字典序 |
| reverse | 反转 / 逆序 |
| search | 查找 |
| find | 找到 / 查找 |
| missing number | 缺失的数字 |
| duplicate | 重复元素 |
| unique | 唯一的 / 不重复的 |
| frequency | 出现频率 |
| occurrence | 出现次数 |
| maximum / minimum | 最大值 / 最小值 |
| compare | 比较 |
| swap | 交换 |
| iteration | 迭代 / 一次循环过程 |
| recursion | 递归 |
| array / list | 数组 / 列表 |
| index | 下标 / 索引 |
| complexity | 复杂度 |
| time complexity | 时间复杂度 |
| space complexity | 空间复杂度 |
| memory usage | 内存使用量 |
| edge case | 边界情况 |

## 15. 考前优先记忆清单

### 必须熟练

- `input()`、`print()`、`int()`、`str()`；
- `split()`、`join()`、`strip()`；
- `len()`、`range()`、`enumerate()`；
- `for`、`while`、`if`、`break`、`continue`；
- 列表的 `append()`、`extend()`、`pop()`、`remove()`、`index()`；
- `sorted()`、`sort()`、`reverse`、`key`；
- 字符串的 `lower()`、`upper()`、`casefold()`、切片；
- 字典 `dict` 的 `get()` 和计数方法；
- 集合 `set` 的去重和成员判断；
- `O(1)`、`O(n)`、`O(n log n)`、`O(n²)`。

### 有时间再看

- 递归；
- 二分查找；
- 冒泡排序、选择排序、插入排序的基本思想；
- `Counter`；
- 列表推导式；
- 简单异常处理 `try/except`。

## 16. 参考资料

- [HKU MSc AI 官方 Admissions 页面](https://www.mscai.hku.hk/admissions/)：招生知识基础与申请安排。
- [一亩三分地：2022 HKU MSc AI 笔试经验](https://www.1point3acres.com/bbs/thread-866666-1-1.html)。
- [一亩三分地：2023 HKU MSc AI 笔试经验](https://www.1point3acres.com/bbs/thread-996898-1-1.html)。
- [Python 官方文档：排序指南](https://docs.python.org/zh-cn/3/howto/sorting.html)：`sorted()`、`list.sort()`、`key`、`reverse` 和稳定排序。
- [Python 官方文档：数据结构](https://docs.python.org/3/tutorial/datastructures.html)：列表方法。

> 说明：往届笔试信息来自申请者个人回忆，可能存在记忆偏差；不同年份、不同项目的考试形式可能变化。请以当年港大邮件和官方说明为准。
