# 1. 输入与输出

| 方法 | 作用 | 说明 |
|---|---|---|
| `input()` | 读取一行输入 | 返回字符串，需要时使用 `int()` 等函数转换类型 |
| `sys.stdin.read()` | 读取全部标准输入 | 需要先 `import sys`，适合多行或不确定行数的输入 |
| `print()` | 输出内容 | 可以输出字符串、数字、列表等 |
| `print(a, b)` | 输出多个内容 | 默认使用空格分隔 |
| `print(x, end="")` | 修改输出结尾 | 默认结尾是换行符 `\n` |
| `print(a, b, sep=",")` | 修改输出分隔符 | 默认使用空格分隔 |

**常见输入方式：**

```python
# 读取一行字符串
line = input()

# 读取一行，以空白字符拆分
words = input().split()

# 读取一行整数，并转换为整数列表
numbers = list(map(int, input().split()))

# 读取全部输入，并按照空白字符拆分
import sys
data = sys.stdin.read().split()

# 读取全部输入，并转换为整数列表
numbers = list(map(int, sys.stdin.read().split()))
```

# 2. 类型转换与常用内置函数

内置函数是 Python 自带的函数，可以直接调用，不需要通过列表、字符串等对象调用。

## 2.1 类型转换

| 函数 | 作用 | 示例 |
|---|---|---|
| `int(x)` | 转换为整数 | `int("12")` 得到 `12` |
| `float(x)` | 转换为浮点数 | `float("3.14")` 得到 `3.14` |
| `str(x)` | 转换为字符串 | `str(12)` 得到 `"12"` |
| `list(x)` | 转换为列表 | `list("abc")` 得到 `['a', 'b', 'c']` |
| `tuple(x)` | 转换为元组 | `tuple([1, 2])` 得到 `(1, 2)` |
| `set(x)` | 转换为集合 | `set([1, 1, 2])` 得到 `{1, 2}` |
| `dict(x)` | 创建字典 | `dict([("a", 1)])` 得到 `{'a': 1}` |

## 2.2 常用内置函数

| 函数 | 作用 | 示例含义 |
|---|---|---|
| `len(x)` | 获取元素个数 | `len([1, 2, 3])` 得到 `3` |
| `range(n)` | 生成整数序列 | `range(3)` 表示 `0, 1, 2` |
| `enumerate(x)` | 遍历时同时获取下标和元素 | 常用于需要元素位置的情况 |
| `max(x)` | 获取最大值 | `max([2, 5, 1])` 得到 `5` |
| `min(x)` | 获取最小值 | `min([2, 5, 1])` 得到 `1` |
| `sum(x)` | 对元素求和 | `sum([2, 5, 1])` 得到 `8` |
| `abs(x)` | 获取绝对值 | `abs(-5)` 得到 `5` |
| `sorted(x)` | 返回排序后的新列表 | 不修改原数据 |
| `reversed(x)` | 返回反向迭代器 | 可以使用 `list()` 转换为列表 |
| `all(x)` | 判断是否全部为真 | 所有元素都为真时返回 `True` |
| `any(x)` | 判断是否存在真值 | 至少一个元素为真时返回 `True` |
| `type(x)` | 获取对象类型 | `type(12)` 得到 `int` |

## 2.3 `map()`：批量处理元素

`map()` 是 Python 的内置函数，作用是将指定函数依次应用到可迭代对象中的每个元素。

**语法：**

```python
map(function, iterable)
```

其中：

- `function`：需要应用的函数。
- `iterable`：需要处理的可迭代对象，例如列表。

**注意：** `map()` 返回的是一个迭代器，不是列表。如果需要得到列表，可以使用 `list()` 转换。

**示例一：将字符串列表转换为整数列表**

```python
numbers = ["1", "2", "3"]
result = list(map(int, numbers))
print(result)
```

输出：

```text
[1, 2, 3]
```

**示例二：读取一行整数**

```python
numbers = list(map(int, input().split()))
```

假设输入：

```text
10 20 30
```

得到：

```python
[10, 20, 30]
```

执行过程：

1. `input()`：读取一行字符串。
2. `split()`：按照空白字符拆分，得到字符串列表。
3. `map(int, ...)`：将列表中的每个字符串转换为整数。
4. `list()`：将 `map()` 返回的迭代器转换为列表。

**示例三：使用 `lambda` 自定义处理方式**

```python
numbers = [1, 2, 3]
result = list(map(lambda x: x * 2, numbers))
print(result)
```

输出：

```text
[2, 4, 6]
```

`lambda x: x * 2` 表示一个简单函数，将传入的数字乘以 2。

# 3. 字符串常用方法

字符串的方法通过 `字符串.方法()` 的形式调用。例如：`text.lower()`。

| 方法 | 作用 | 说明 |
|---|---|---|
| `split()` | 拆分字符串 | 默认按照空白字符拆分，返回列表 |
| `join(iterable)` | 拼接字符串 | 使用指定字符串作为分隔符连接元素 |
| `strip()` | 去掉首尾空白 | 不会删除字符串中间的空格 |
| `lstrip()` | 去掉左侧空白 | 删除字符串开头的空白 |
| `rstrip()` | 去掉右侧空白 | 删除字符串末尾的空白 |
| `lower()` | 转换为小写 | 返回新字符串 |
| `upper()` | 转换为大写 | 返回新字符串 |
| `casefold()` | 转换为适合忽略大小写比较的形式 | 常用于不区分大小写的字符串比较 |
| `replace(old, new)` | 替换字符串内容 | 将指定内容替换为新内容 |
| `find(x)` | 查找子字符串 | 返回第一次出现的位置；不存在时返回 `-1` |
| `count(x)` | 统计出现次数 | 返回指定内容出现的次数 |
| `startswith(x)` | 判断是否以指定内容开头 | 返回布尔值 |
| `endswith(x)` | 判断是否以指定内容结尾 | 返回布尔值 |
| `isdigit()` | 判断是否全部为数字字符 | 返回布尔值 |
| `isalpha()` | 判断是否全部为字母 | 返回布尔值 |
| `isalnum()` | 判断是否全部为字母或数字 | 返回布尔值 |

**示例：**

```python
text = "  Hello World  "

print(text.strip())       # "Hello World"
print(text.lower())       # "  hello world  "
print(text.upper())       # "  HELLO WORLD  "
print(text.replace("World", "Python"))
```

**补充：字符串的 `split()` 和 `join()`**

```python
# split：将字符串拆分为列表
text = "apple banana orange"
words = text.split()
print(words)
# ['apple', 'banana', 'orange']

# join：将列表中的字符串拼接起来
result = " ".join(words)
print(result)
# apple banana orange
```

注意：

- `split()` 是字符串的方法，返回列表。
- `join()` 也是字符串的方法，调用形式是 `"分隔符".join(字符串列表)`。
- 字符串是不可变对象，`lower()`、`replace()` 等方法不会直接修改原字符串，而是返回新字符串。

# 4. 列表常用操作与方法

列表（`list`）用于存储多个元素，可以修改、添加和删除元素。列表既可以使用内置函数处理，也有列表对象自己提供的专属方法。

## 4.1 列表常用操作

| 操作 | 作用 | 示例 |
|---|---|---|
| `list[i]` | 获取指定下标的元素 | `words[0]` |
| `list[i] = x` | 修改指定下标的元素 | `words[0] = "apple"` |
| `list[a:b]` | 获取指定范围的元素 | `words[0:2]` |
| `x in list` | 判断元素是否存在 | `"apple" in words` |
| `len(list)` | 获取列表长度 | `len(words)` |

## 4.2 列表对象的常用方法

以下方法需要通过列表对象调用，例如 `words.append("apple")`。

| 方法 | 作用 | 说明 |
|---|---|---|
| `append(x)` | 添加一个元素 | 将 `x` 添加到列表末尾 |
| `extend(iterable)` | 添加多个元素 | 将可迭代对象中的元素逐个添加到列表末尾 |
| `insert(i, x)` | 在指定位置插入元素 | 在下标 `i` 处插入 `x` |
| `pop()` | 删除并返回元素 | 默认删除最后一个元素；也可以指定下标 |
| `remove(x)` | 删除指定元素 | 删除第一次出现的 `x` |
| `clear()` | 清空列表 | 删除所有元素 |
| `index(x)` | 查找元素下标 | 返回第一次出现的位置；不存在时会报错 |
| `count(x)` | 统计元素出现次数 | 返回指定元素在列表中出现的次数 |
| `reverse()` | 反转列表 | 直接修改原列表，不进行大小排序 |
| `sort()` | 对列表进行排序 | 直接修改原列表，默认升序 |
| `copy()` | 复制列表 | 返回一个浅拷贝的新列表 |

**重点理解：`list.sort()` 和 `list.count()`**

这两个都是列表对象自身提供的方法，需要通过列表对象调用。

### （1）`list.sort()`：列表排序方法

```python
numbers = [5, 2, 4, 1]
numbers.sort()
print(numbers)
```

输出：

```text
[1, 2, 4, 5]
```

特点：

- `sort()` 是列表对象的方法。
- 默认按照升序排序。
- 直接修改原列表。
- 返回值是 `None`，不能通过 `numbers = numbers.sort()` 获取排序后的列表。

还可以指定排序方式：

```python
numbers = [3, 1, 5, 2]
numbers.sort(reverse=True)
print(numbers)
# [5, 3, 2, 1]
```

### （2）`list.count(x)`：统计元素出现次数

```python
numbers = [1, 2, 2, 3, 2, 4]
result = numbers.count(2)
print(result)
```

输出：

```text
3
```

特点：

- `count()` 是列表对象的方法。
- `count(x)` 用于统计元素 `x` 在列表中出现的次数。
- 不修改原列表。
- 返回一个整数。

### （3）`list.append()` 和 `list.extend()` 的区别

```python
numbers = [1, 2]

# append：将整个对象作为一个元素添加
numbers.append([3, 4])
print(numbers)
# [1, 2, [3, 4]]
```

```python
numbers = [1, 2]

# extend：将可迭代对象中的元素逐个添加
numbers.extend([3, 4])
print(numbers)
# [1, 2, 3, 4]
```

**记忆：**

- `append()`：添加一个元素，即使这个元素本身是一个列表，也会整体添加。
- `extend()`：逐个添加可迭代对象中的元素。

# 5. 排序：`sort()` 和 `sorted()`

这两个排序方式容易混淆，需要重点掌握。

| 方法 | 类型 | 作用 | 是否修改原列表 |
|---|---|---|---|
| `list.sort()` | 列表对象的方法 | 对列表进行排序 | 是 |
| `sorted(iterable)` | Python 内置函数 | 返回排序后的新列表 | 否 |

## 5.1 使用 `sort()` 排序

```python
words = ["banana", "apple", "orange"]
words.sort()
print(words)
```

输出：

```text
['apple', 'banana', 'orange']
```

`sort()` 直接修改原列表，不会返回排序后的列表。

## 5.2 使用 `sorted()` 排序

```python
numbers = [5, 2, 4, 1]
result = sorted(numbers)

print(result)  # [1, 2, 4, 5]
print(numbers) # [5, 2, 4, 1]
```

`sorted()` 返回一个新的列表，原列表保持不变。

## 5.3 降序排序

`reverse=True` 可以作为 `sort()` 或 `sorted()` 的参数，表示按照降序排序。

```python
numbers = [3, 1, 5, 2]

# 使用 sort()
numbers.sort(reverse=True)
print(numbers)

# 使用 sorted()
result = sorted(numbers, reverse=True)
print(result)
```

输出：

```text
[5, 3, 2, 1]
[5, 3, 2, 1]
```

## 5.4 使用 `key` 指定排序依据

`key` 用于指定排序时的比较依据。

**按照字符串长度排序：**

```python
words = ["banana", "kiwi", "apple", "fig"]
result = sorted(words, key=len)
print(result)
```

输出：

```text
['fig', 'kiwi', 'apple', 'banana']
```

这里 `key=len` 表示按照每个字符串的长度进行排序。

**忽略大小写排序：**

```python
words = ["banana", "Apple", "cherry", "apple"]
result = sorted(words, key=str.casefold)
print(result)
```

输出：

```text
['Apple', 'apple', 'banana', 'cherry']
```

`key=str.casefold` 表示按照忽略大小写后的字符串进行比较，但结果仍然保留原字符串的大小写。

**注意：** 如果题目没有要求忽略大小写，就不必额外使用 `key=str.casefold`。

# 6. 字典与集合
## 6.1 字典（`dict`）

字典通过键值对（`key: value`）存储数据。

**常用操作：**

| 方法 / 操作 | 作用 | 说明 |
|---|---|---|
| `dict[key]` | 获取指定键对应的值 | 键不存在时会报错 |
| `dict[key] = value` | 添加或修改键值对 | 键存在时修改，不存在时添加 |
| `dict.get(key, default)` | 获取指定键对应的值 | 键不存在时返回默认值 |
| `keys()` | 获取所有键 | 返回可迭代的字典视图 |
| `values()` | 获取所有值 | 返回可迭代的字典视图 |
| `items()` | 获取所有键值对 | 每个元素是一个 `(key, value)` 元组 |
| `update(other)` | 更新字典 | 添加或覆盖其他字典中的键值对 |
| `pop(key)` | 删除指定键 | 删除并返回对应的值 |
| `clear()` | 清空字典 | 删除所有键值对 |

**示例：**

```python
student = {
    "name": "Tom",
    "age": 20
}

# 获取值
print(student.get("name"))
# Tom

# 获取不存在的键，返回默认值
print(student.get("score", 0))
# 0

# 添加或修改
student["score"] = 90

# 遍历键值对
for key, value in student.items():
    print(key, value)
```

**注意：**

```python
"name" in student
```

对于字典，`in` 默认判断的是键是否存在，而不是值是否存在。

## 6.2 集合（`set`）

集合用于存储不重复的元素，具有自动去重的特点。

**常用操作：**

| 方法 / 操作 | 作用 | 说明 |
|---|---|---|
| `set(iterable)` | 创建集合 | 自动去除重复元素 |
| `add(x)` | 添加元素 | 添加一个元素 |
| `update(iterable)` | 添加多个元素 | 将可迭代对象中的元素添加到集合 |
| `remove(x)` | 删除元素 | 元素不存在时会报错 |
| `discard(x)` | 删除元素 | 元素不存在时不会报错 |
| `pop()` | 删除并返回一个元素 | 集合没有固定顺序 |
| `clear()` | 清空集合 | 删除所有元素 |
| `len(set)` | 获取元素数量 | 返回集合中元素的个数 |
| `x in set` | 判断元素是否存在 | 返回布尔值 |

**示例：**

```python
numbers = [1, 2, 2, 3, 3, 4]

# 转换成集合，自动去重
result = set(numbers)
print(result)
```

结果包含：

```text
{1, 2, 3, 4}
```

注意：集合不保证列表式的顺序，不能通过下标访问元素。

# 7. 常用语法

| 语法 | 作用 | 说明 |
|---|---|---|
| `if / elif / else` | 条件判断 | 根据条件执行不同代码 |
| `for x in list` | 遍历列表 | 逐个获取列表中的元素 |
| `for i in range(n)` | 循环指定次数 | `i` 从 `0` 到 `n-1` |
| `while condition` | 条件循环 | 条件满足时重复执行 |
| `break` | 结束循环 | 立即跳出当前循环 |
| `continue` | 跳过当前轮 | 直接进入下一轮循环 |
| `def function():` | 定义函数 | 创建可重复调用的函数 |
| `return` | 返回结果 | 结束函数并返回结果 |
| `lambda` | 定义匿名函数 | 用于简单的单行函数 |
| `if __name__ == "__main__":` | 主程序入口判断 | 判断当前文件是否作为主程序运行 |

**补充：`enumerate()` 的使用**

`enumerate()` 是内置函数，遍历时可以同时获取元素的下标和值。

```python
words = ["apple", "banana", "orange"]

for index, word in enumerate(words):
    print(index, word)
```

输出：

```text
0 apple
1 banana
2 orange
```

# 8. 常用例子

## 8.1 读取一行并拆分单词

```python
line = input()
words = line.split()
print(words)
```

输入：

```text
apple banana orange
```

输出：

```text
['apple', 'banana', 'orange']
```

## 8.2 读取多行输入并拆分

```python
import sys

words = sys.stdin.read().split()
print(words)
```

输入：

```text
banana apple
orange pear
```

输出：

```text
['banana', 'apple', 'orange', 'pear']
```

`sys.stdin.read()` 会读取全部输入，`split()` 再把空格、换行和制表符等空白字符作为分隔符。

## 8.3 使用 `map()` 读取整数列表

```python
numbers = list(map(int, input().split()))
print(numbers)
```

输入：

```text
10 20 30 40
```

输出：

```text
[10, 20, 30, 40]
```

## 8.4 使用 `sort()` 排序

```python
words = ["banana", "apple", "orange"]
words.sort()
print(words)
```

输出：

```text
['apple', 'banana', 'orange']
```

`sort()` 会直接修改原列表。

## 8.5 使用 `sorted()` 排序

```python
numbers = [5, 2, 4, 1]
result = sorted(numbers)

print(result)   # [1, 2, 4, 5]
print(numbers)  # [5, 2, 4, 1]
```

`sorted()` 会返回新列表，原列表保持不变。

## 8.6 使用 `count()` 统计元素出现次数

```python
words = ["apple", "banana", "apple", "orange", "apple"]
result = words.count("apple")
print(result)
```

输出：

```text
3
```

`count()` 是列表对象的方法，用于统计指定元素出现的次数。

## 8.7 使用 `join()` 拼接单词

```python
words = ["apple", "banana", "orange"]
result = " ".join(words)
print(result)
```

输出：

```text
apple banana orange
```

## 8.8 列表添加元素、获取长度

```python
words = ["apple", "banana"]
words.append("orange")

print(len(words))
print(words)
```

输出：

```text
3
['apple', 'banana', 'orange']
```

## 8.9 遍历列表并同时获取下标

```python
words = ["apple", "banana", "orange"]

for index, word in enumerate(words):
    print(index, word)
```

输出：

```text
0 apple
1 banana
2 orange
```

## 8.10 使用 `map()` 批量转换数据

```python
numbers = ["1", "2", "3", "4"]

# 将每个字符串转换为整数
numbers = list(map(int, numbers))
print(numbers)
```

输出：

```text
[1, 2, 3, 4]
```

# 9. 综合例子

## 9.1 读取单词、排序并输出

```python
import sys

def main():
    # 读取全部输入，并按照空白字符拆分成单词列表
    words = sys.stdin.read().split()

    # 对原列表进行升序排序
    words.sort()

    # 使用空格连接单词并输出
    print(" ".join(words))


if __name__ == "__main__":
    main()
```

输入：

```text
banana apple orange
pear grape
```

输出：

```text
apple banana grape orange pear
```

**执行顺序：**

1. `sys.stdin.read()`：读取全部输入。
2. `split()`：将输入拆分成单词列表。
3. `sort()`：调用列表的排序方法，对原列表进行升序排序。
4. `" ".join(words)`：调用字符串的 `join()` 方法，使用空格拼接单词。
5. `print()`：输出结果。

## 9.2 读取整数、排序并计算总和

```python
import sys

def main():
    # 读取全部输入，并转换为整数列表
    numbers = list(map(int, sys.stdin.read().split()))

    # 排序
    numbers.sort()

    # 计算总和
    total = sum(numbers)

    print(numbers)
    print(total)


if __name__ == "__main__":
    main()
```

输入：

```text
5 2 4 1 3
```

输出：

```text
[1, 2, 3, 4, 5]
15
```

这里涉及的主要方法：

- `sys.stdin.read()`：读取全部输入。
- `split()`：拆分字符串。
- `map(int, ...)`：批量转换为整数。
- `list()`：转换为列表。
- `sort()`：列表原地排序。
- `sum()`：计算列表元素总和。

# 10. 考前重点区分
## 10.1 内置函数与对象方法的区别

| 内置函数 | 对象方法 | 区别 |
|---|---|---|
| `len(list)` | 无 | 获取列表长度 |
| `sorted(list)` | `list.sort()` | 前者返回新列表，后者修改原列表 |
| `list(iterable)` | 无 | 将可迭代对象转换为列表 |
| `set(iterable)` | 无 | 将可迭代对象转换为集合 |
| `map(function, iterable)` | 无 | 对可迭代对象中的元素批量应用函数 |
| 无 | `list.append(x)` | 向列表末尾添加元素 |
| 无 | `list.count(x)` | 统计列表中某个元素出现的次数 |
| 无 | `list.reverse()` | 反转原列表 |
| 无 | `str.split()` | 拆分字符串 |
| 无 | `str.join(iterable)` | 拼接字符串 |
| 无 | `dict.get(key)` | 获取字典中的值 |

## 10.2 几组容易混淆的方法

| 对比 | 核心区别 |
|---|---|
| `append()` 和 `extend()` | 前者添加一个整体元素，后者逐个添加元素 |
| `remove()` 和 `pop()` | 前者按照元素值删除，后者按照下标删除并返回元素 |
| `sort()` 和 `sorted()` | 前者修改原列表，后者返回新列表 |
| `reverse()` 和 `sort(reverse=True)` | 前者反转现有顺序，后者按照降序重新排序 |
| `split()` 和 `join()` | 前者拆分字符串，后者拼接字符串 |
| `dict[key]` 和 `dict.get(key)` | 前者键不存在时报错，后者可以返回默认值 |
| `remove()` 和 `discard()` | 集合中前者删除不存在的元素会报错，后者不会 |
| `map()` 和 `list()` | 前者负责批量处理元素，后者负责创建列表 |

**最后记住：**

- `len()`、`map()`、`sorted()`、`sum()`、`max()`、`min()`、`enumerate()`：直接调用的 Python 内置函数。
- `append()`、`extend()`、`count()`、`sort()`、`reverse()`：列表对象的方法。
- `split()`、`join()`、`replace()`、`lower()`：字符串对象的方法。
- `get()`、`keys()`、`values()`、`items()`：字典对象的方法。

做算法题时，尤其要熟悉以下常见写法：

```python
# 读取一行整数，转换为列表
numbers = list(map(int, input().split()))

# 统计列表中某个元素出现的次数
count = numbers.count(2)

# 对原列表升序排序
numbers.sort()

# 返回一个排序后的新列表
result = sorted(numbers)

# 使用空格拼接字符串列表
result = " ".join(words)
```
