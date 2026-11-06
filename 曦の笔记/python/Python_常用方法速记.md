## 1. 输入与输出

### 1.1 输入函数（含参数）

| 签名 | 作用与结论 | 参数说明 | 返回值 / 注意 |
| --- | --- | --- | --- |
| `input(prompt="")` | 读取一行标准输入 | `prompt`：可选提示串，会先写到标准输出且**不换行**；OJ / 笔试中不要传，否则污染输出 | 返回 `str`，不含行末换行；遇 EOF 抛 `EOFError` |
| `sys.stdin.read(size=-1)` | 一次读取全部输入 | `size`：最多读取的字符数，默认 `-1` 表示读到 EOF | 返回完整字符串（含换行）；需 `import sys`；适合多行、行数不定 |
| `sys.stdin.readline()` | 读取一行 | `※` 可传 `size` 限制字符数 | 返回 `str` 且**保留行末 `\n`**，常配 `.strip()` |
| `sys.stdin.read().split()` | 读全部并按空白拆词 | `split()` 默认按任意空白，自动跨行、丢弃空字段 | 返回 `list[str]`，是多行输入最稳妥写法 |

### 1.2 `print()` 参数详解（重点）

| 参数 | 默认值 | 含义 / 使用要点 |
| --- | --- | --- |
| `*objects` | 无 | 任意多个待输出对象，自动按 `str()` 转换；可用 `*` 展开列表 |
| `sep` | `" "` | 多个对象之间的分隔符；设为 `""` 可无缝拼接 |
| `end` | `"\n"` | 输出结尾；`end=""` 用于不换行连续输出 |
| `file` | `sys.stdout` | 输出目标，可为文件对象或 `sys.stderr` `※` |
| `flush` | `False` | 是否立即刷新缓冲区 `※`；交互 / 实时输出时设 `True` |

| 结论 | 内容 |
| --- | --- |
| 返回值 | `None`，不能写 `x = print(...)` |
| 调用形式 | `print(a, b, sep=",", end="")`；关键字参数须放在所有位置参数之后 |
| 与 `f-string` 配合 | 需要精确格式（补零、小数位）时用 `print(f"...")`，不要靠 `sep` 拼 |

### 1.3 标准读入套路（结论式，不展开示例）

| 场景 | 标准写法 |
| --- | --- |
| 单个整数 | `n = int(input())` |
| 一行多个整数（赋给变量） | `a, b = map(int, input().split())` |
| 一行多个整数（成列表） | `a = list(map(int, input().split()))` |
| 按指定分隔符拆 | `h, m = map(int, s.split(":"))` |
| 去掉首尾空白 | `s = input().strip()` |
| 全部输入转整数列表 | `list(map(int, sys.stdin.read().split()))` |
| `※` 多行定长 | `rows = [input().split() for _ in range(n)]` |

## 2. 类型转换与常用内置函数

### 2.1 类型转换

| 签名 | 作用与结论 | 参数说明 | 返回值 / 注意 |
| --- | --- | --- | --- |
| `int(x=0, base=10)` | 转整数 | `x`：`str` / `float` / `bytes` / `bool`；`base`：仅当 `x` 为字符串时有效，可取 `2..36` 或 `0`（按 `0x` 等前缀自动判断），默认 `10` | `float` 向 0 截断；非法字符串抛 `ValueError` |
| `float(x=0.0)` | 转浮点数 | `x`：数字或可解析的字符串 | 返回 `float`；`/` 的结果永远是 `float` |
| `str(object="")` | 转字符串 | `object`：任意对象，调用其 `__str__` | 返回 `str` |
| `bool(x=False)` | 转布尔 | `x` 省略时为 `False` | `0`、`0.0`、`""`、`[]`、`{}`、`None` 为假，其余一般为真 |
| `list(iterable=())` | 转列表 | `iterable`：任意可迭代对象；`str` 会被拆成单字符 | 返回新 `list`；常用于物化 `map` / `range` 等迭代器 |
| `tuple(iterable=())` | 转元组 | 同上 | 返回 `tuple`，不可变 |
| `set(iterable=())` | 转集合 | 同上 | 自动去重、无序、不能用下标访问 |
| `dict(mapping=(), **kwargs)` | 建字典 | 可传键值对序列、另一字典或 `key=value` | 返回 `dict` |

### 2.2 数值与算术内置函数

| 签名 | 作用与结论 | 参数说明 | 返回值 / 注意 |
| --- | --- | --- | --- |
| `abs(x)` | 绝对值 | `x`：数值 | 返回同类型 |
| `divmod(a, b)` | 同时取商和余数 `※` | `a, b`：数值 | 返回 `(a // b, a % b)` 元组 |
| `pow(base, exp, mod=None)` | 幂 / 模幂 `※` | `mod` 给定时等价于 `(base**exp) % mod`，但更快 | 返回数值；`b` 为 0 时抛 `ZeroDivisionError` |
| `round(number, ndigits=None)` | 四舍五入 `※` | `ndigits`：保留小数位；省略则返回 `int` | 采用**银行家舍入**，`.5` 取最接近的偶数；不是「逢五进一」 |
| `chr(i)` / `ord(c)` | 码点与字符互转 `※` | `i`：`0..0x10FFFF`；`c`：单个字符 | `chr` 返回长度为 1 的 `str`；`ord` 返回 `int` |
| `max(iterable, *, key=None, default=...)` | 最大值 | `key`：单参数函数，返回比较键；`default`：iterable 为空时的返回值（必须关键字传） | 也支持 `max(a, b, c, ...)` 多位置参数形式 |
| `min(...)` | 最小值 | 参数与 `max` 完全一致 | 同上 |
| `sum(iterable, start=0)` | 求和 | `start`：累加初值，默认 `0` | 不能用于拼接字符串（用 `"".join(...)`） |
| `type(object)` | 获取对象的类型 | `object`：任意对象 | 返回类型对象；只用于查看，判断类型优先用 `isinstance` |
| `※ isinstance(obj, classinfo)` | 类型判断 | `classinfo` 可为类型或类型元组 | 返回 `bool`；比 `type(x) is ...` 更稳妥（含继承关系） |

### 2.3 迭代与聚合内置函数

| 签名 | 作用与结论 | 参数说明 | 返回值 / 注意 |
| --- | --- | --- | --- |
| `len(s)` | 元素个数 | `s`：`str` / `list` / `dict` / `set` / `tuple` 等 | 对 `dict` 返回**键**的数量 |
| `range(start=0, stop, step=1)` | 生成整数序列 | `range(stop)` 时 `start=0`；区间为左闭右开 `[start, stop)`；`step` 不可为 `0`，可为负 | 返回惰性 `range` 对象，**不是列表**；可重复迭代、支持 `len()` 与下标 |
| `enumerate(iterable, start=0)` | 带下标遍历 | `start`：下标的起始值，默认 `0` | 返回 `(index, value)` 迭代器；写成 `for i, v in enumerate(a)` |
| `map(function, iterable, *iterables)` | 批量应用函数 | `function`：接收 1 个或多个参数的函数；传多个 iterable 时按位置配对，长度以**最短**为准 | 返回**迭代器**：惰性、一次性、需 `list()` 物化；`function` 可为 `lambda` 或 `int` 等类型构造器 |
| `filter(function, iterable)` | 按条件筛选 `※` | `function` 返回真值则保留；传 `None` 时保留本身为真的元素 | 返回迭代器 |
| `zip(*iterables, strict=False)` | 并行打包 `※` | 默认以最短的 iterable 截断；`strict=True`（3.10+）长度不等时抛 `ValueError` | 返回元组迭代器 |
| `reversed(seq)` | 反向遍历 | `seq` 需支持 `__len__` 与 `__getitem__`，或为 `range` / `list` / `tuple` / `str` | 返回迭代器，**不修改**原对象；与 `list.reverse()` 不同 |
| `all(iterable)` | 全部为真 | 空迭代器返回 `True` | 短路求值 |
| `any(iterable)` | 至少一个为真 | 空迭代器返回 `False` | 短路求值 |
| `sorted(iterable, *, key=None, reverse=False)` | 排序 | 见 §7 | 返回**新列表**，不改动原数据 |

### 2.4 重点函数参数对照（速记）

| 函数 | 易错参数 | 正确理解 |
| --- | --- | --- |
| `int(x, base)` | `base` | 只有 `x` 是字符串时才生效；`int("08")` 合法，`int(8, 2)` 报错 |
| `range(a, b, c)` | 三参数顺序 | `range(stop)` / `range(start, stop)` / `range(start, stop, step)`；`range(5, 1, -1)` 才产生递减序列 |
| `map(f, it)` | 返回值 | 迭代器不是列表；`list(map(int, input().split()))` 中的 `list()` 不可省（除非只解包给变量） |
| `max` / `min` | `key` / `default` | 只能关键字传；`default` 只用于单个 iterable 为空的情形 |
| `sorted` | `key` / `reverse` | `key` 是**函数**（如 `key=len`、`key=str.casefold`），不是比较结果；`reverse` 是布尔值 |
| `sum(it, start)` | `start` | 累加初值；`sum([], start=0)` 返回 `0` |
| `round(x, n)` | `ndigits` | 省略返回整数；`round(2.5)` 得 `2` 而不是 `3` |

## 3. 运算符与表达式

| 运算符 / 表达式 | 结论 |
| --- | --- |
| `+ - * /` | `/` 永远返回 `float`，即使能整除 |
| `//` | 向下取整（floor）；`-7 // 3 = -3`，不是 `-2` |
| `%` | 取模，结果符号**跟随除数**；`-7 % 3 = 2`；恒有 `a == (a // b) * b + a % b` |
| `**` | 幂运算，优先级高于一元负号：`-2**2 = -4` |
| `divmod(a, b)` | 一次拿到 `(a // b, a % b)` |
| `+= -= *= //= %= **=` | 增强赋值；对不可变对象等价于重新绑定 |
| `== != < <= > >=` | 支持链式：`1 < x < 5` 等价于 `1 < x and x < 5` |
| `is` / `is not` | 判断是否为**同一对象**，不是值相等；小整数缓存易造成误判 |
| `and` / `or` | 短路求值，返回**操作数本身**而非布尔；`a or b` 常用于取默认值 |
| `not` | 返回 `bool` |
| `A if cond else B` | 条件表达式（三元）；可整体放进 `print()` |
| `& ^ ~ << >>` | 位运算 `※`；与 `and` / `or` 完全不同 |
| `in` / `not in` | 对 `list` / `str` 是线性查找；对 `set` / `dict` 平均 `O(1)`；对 `dict` 判的是**键** |

## 4. 字符串常用方法

> 字符串是**不可变对象**：所有方法都返回新字符串，不会原地修改；必须写回变量才生效。

### 4.1 拆分与拼接（重点参数）

| 签名 | 作用与结论 | 参数说明 | 返回值 / 注意 |
| --- | --- | --- | --- |
| `str.split(sep=None, maxsplit=-1)` | 拆分成列表 | `sep=None`（默认）：按任意连续空白（空格 / 制表 / 换行）拆分，自动丢弃首尾空白与空字段；`sep` 给定字符串：严格按该子串拆分并**保留空字段**，不能是空串；`maxsplit`：最多拆几次，结果最多 `maxsplit+1` 段，`-1`（默认）不限 | 返回 `list[str]`；`split("")` 抛 `ValueError` |
| `str.rsplit(sep=None, maxsplit=-1)` | 从右侧开始拆 `※` | 参数同 `split`；只在 `maxsplit >= 0` 时与 `split` 结果不同 | 返回 `list[str]` |
| `str.splitlines(keepends=False)` | 按各类换行符拆 `※` | `keepends=True` 时保留行末换行符 | 返回 `list[str]` |
| `"sep".join(iterable)` | 拼接字符串 | 调用者字符串是**分隔符**；`iterable` 的元素必须**全是 `str`** | 返回拼接后的 `str`；数字列表需先 `map(str, ...)` |
| `str.partition(sep)` | 三段切分 `※` | `sep`：分隔符 | 返回 `(前, sep, 后)`；找不到时为 `(原串, "", "")`，不报错 |

| 结论 | 内容 |
| --- | --- |
| 分隔符选择 | 空格分隔用 `split()`；`"HH:MM"` 必须用 `split(":")`；逗号分隔用 `split(",")` |
| 拆后转整数 | `list(map(int, ...))` 或 `a, b = map(int, ...)`；解包时变量个数必须等于分段数 |
| `split` 与 `join` | 互为逆操作；`split` 返回列表，`join` 返回字符串；两者都是字符串方法 |

### 4.2 去空白与大小写

| 签名 | 作用与结论 | 参数说明 | 返回值 / 注意 |
| --- | --- | --- | --- |
| `str.strip(chars=None)` | 去首尾 | `chars=None`（默认）：去首尾空白（空格 / `\t` / `\n` / `\r`）；给定字符串时按**字符集合**去除首尾出现在其中的任意字符 | 返回新串；**不删除中间**的空白；"按字符集合，不是按子串" |
| `str.lstrip(chars=None)` / `rstrip(...)` | 只去左 / 右侧 | 参数同 `strip` | 返回新串 |
| `str.lower()` / `upper()` | 转小 / 大写 | 无参数 | 返回新串 |
| `str.casefold()` | 忽略大小写比较专用 | 无参数 | 比 `lower()` 更彻底（如德语 `ß`）；配合 `key=str.casefold` |
| `※ str.capitalize()` / `title()` / `swapcase()` | 首字母大写 / 每词首字母大写 / 大小写互换 | 无参数 | 返回新串 |

### 4.3 查找、替换与判断

| 签名 | 作用与结论 | 参数说明 | 返回值 / 注意 |
| --- | --- | --- | --- |
| `str.find(sub, start=0, end=len)` | 查子串位置 | `start` / `end`：限定搜索区间（切片式索引） | 找不到返回 `-1`，**不报错** |
| `str.index(sub, start=0, end=len)` | 查子串位置 `※` | 同上 | 找不到抛 `ValueError` |
| `str.count(sub, start=0, end=len)` | 统计次数 | 同上 | 返回**非重叠**出现次数 |
| `str.replace(old, new, count=-1)` | 替换 | `count`：最多替换次数，`-1`（默认）全部替换 | 返回新串；`old` 为空串时会在字符间插入 |
| `str.startswith(prefix, start=0, end=len)` | 前缀判断 | `prefix` 可为**元组**，任一匹配即真 | 返回 `bool` |
| `str.endswith(suffix, start=0, end=len)` | 后缀判断 | `suffix` 同样可为元组 | 返回 `bool` |
| `str.isdigit()` / `isalpha()` / `isalnum()` | 字符类别判断 | 无参数 | 空串返回 `False`；`isdigit` 对上标 / 罗马数字也为真 `※` |
| `※ str.isdecimal()` / `isnumeric()` / `isspace()` | 更严格的数字 / 空白判断 | 无参数 | `isdecimal` 只认十进制数字字符 |

### 4.4 对齐与补零

| 签名 | 作用与结论 | 参数说明 |
| --- | --- | --- |
| `str.zfill(width)` | 左侧补 `0` 到指定宽度 | `width`：目标总宽度；正负号会被保留在补零之前 |
| `str.ljust(width, fillchar=" ")` | 左对齐填充 | `fillchar` 必须是**长度为 1** 的字符 |
| `str.rjust(width, fillchar=" ")` | 右对齐填充 | 同上；`rjust(2, "0")` 是 `:02d` 的字符串版 |
| `str.center(width, fillchar=" ")` | 居中填充 | 同上 |
| `str.format(*args, **kwargs)` | 旧式格式化 | `{}` 按位置、`{0}` 按编号、`{name}` 按关键字；格式规范与 f-string 相同 |

## 5. 格式化输出（题 40 关键）

### 5.1 f-string 语法

| 项 | 内容 |
| --- | --- |
| 基本形式 | `f"{表达式}"`；`f"{表达式:格式规范}"`；`f"{表达式!转换:格式规范}"` |
| 转换 `!s` / `!r` / `!a` | 分别调用 `str()` / `repr()` / `ascii()` `※` |
| 格式规范结构 | `[[fill]align][sign][#][0][width][grouping][.precision][type]` |
| 对齐 `align` | `<` 左对齐、`>` 右对齐、`^` 居中、`=` 符号后填充（仅数值） |
| 前置 `0` | `0` 等价于 `fill="0"` 且 `align="="`；`:02d` = 宽度 2、不足左补 0 |
| `width` | 最小总宽度；不足时按对齐方式填充，超出不截断 |
| `.precision` | 对 `f` 是小数位数、对 `s` 是最大长度、对 `g` 是有效数字 |
| 分组 | `,` 千分位；`_` 下划线分隔 `※` |
| 嵌套 | `{v:{width}.{prec}f}` 可在格式规范里再放表达式 `※` |

### 5.2 常用类型码

| 类型码 | 含义 | 适用 |
| --- | --- | --- |
| `d` | 十进制整数 | `int`；`:02d` 是补零标准写法 |
| `f` / `F` | 定点小数 | `float`；`:.2f` 保留两位 |
| `e` / `E` / `g` / `G` | 科学计数 / 通用格式 `※` | `float` |
| `%` | 乘 100 后加百分号 `※` | `float` |
| `s` | 字符串 | `str`；补零需用 `zfill` 或 `:0>2` |
| `b` / `o` / `x` / `X` / `c` | 二 / 八 / 十六 / 大写十六 / 字符 `※` | `int` |

| 结论 | 内容 |
| --- | --- |
| `:02d` 只接受整数 | 对字符串用 `s.zfill(2)` 或 `f"{s:0>2}"` |
| `※` 补零不改写数值 | `f"{7:02d}"` 输出 `07`，变量 `7` 本身未变 |
| `※` 与 `print` 配合 | `print(f"{new_h:02d}:{new_m:02d}")`，比字符串拼接更不易漏零 |

## 6. 列表

### 6.1 索引、切片与操作

| 操作 | 结论 |
| --- | --- |
| `a[i]` | 下标从 `0` 开始；`a[-1]` 是最后一个；越界抛 `IndexError` |
| `a[i] = x` | 原地修改指定位置 |
| `a[start:stop:step]` | 左闭右开；省略时 `start=0`、`stop=len(a)`、`step=1`；返回**新列表**（浅拷贝） |
| `step < 0` | 反向取；此时 `start` 默认 `len-1`、`stop` 默认「开头之前」；`a[::-1]` 得到反转副本 |
| `del a[i]` / `del a[i:j]` | 删除元素或切片 `※` |
| `x in a` | 判断存在，线性时间；`a.count(x)` 统计次数 |
| `len(a)` | 元素个数 |
| `※ a * 3` / `a + b` | 重复 / 拼接，均返回新列表；嵌套列表的 `*` 是浅复制，易踩共享引用 |

### 6.2 列表方法（含参数）

| 签名 | 作用与结论 | 参数说明 | 返回值 / 注意 |
| --- | --- | --- | --- |
| `a.append(x)` | 追加**一个**元素 | `x`：任意对象，即使是列表也整体作为一个元素 | 返回 `None`，原地修改 |
| `a.extend(iterable)` | 逐个元素追加 | `iterable`：可迭代对象；字符串会被逐字符拆开 | 返回 `None`；与 `append` 的核心区别 |
| `a.insert(index, x)` | 插入 | 在 `index` **之前**插入；负数按 `len+index` 计，超界自动夹到两端 | 返回 `None` |
| `a.pop(index=-1)` | 删除并返回 | 默认删最后一个；`index` 越界抛 `IndexError` | 返回被删元素；空列表报错 |
| `a.remove(value)` | 按值删除 | 删除**第一个**匹配项；不存在抛 `ValueError` | 返回 `None` |
| `a.index(value, start=0, stop=len)` | 查下标 `※` | `start` / `stop` 限定搜索区间 | 找不到抛 `ValueError` |
| `a.count(value)` | 统计出现次数 | 按 `==` 比较 | 返回 `int`，不修改列表 |
| `a.reverse()` | 原地反转 | 无参数 | 返回 `None`；`reversed(a)` 才返回迭代器且不改原列表 |
| `a.sort(*, key=None, reverse=False)` | 原地排序 | 见 §7 | 返回 **`None`**，不能写 `a = a.sort()` |
| `a.copy()` | 浅拷贝 | 无参数 | 返回新列表；嵌套对象仍共享，深拷贝需 `copy.deepcopy` `※` |
| `a.clear()` | 清空 | 无参数 | 返回 `None`，保留列表对象本身 |

### 6.3 高频对比

| 对比 | 核心区别 |
| --- | --- |
| `append(x)` / `extend(it)` | 前者整体加一个元素，后者逐个展开 |
| `remove(x)` / `pop(i)` | 前者按**值**删且不返回值，后者按**下标**删并返回 |
| `reverse()` / `sort(reverse=True)` | 前者倒转现有顺序，后者按大小降序重排 |
| `sort()` / `sorted()` | 前者原地改、返回 `None`；后者返回新列表 |
| `a.copy()` / `a[:]` / `list(a)` | 三者都是浅拷贝 `※` |

## 7. 排序：`sort()` 与 `sorted()`

| 项 | `list.sort()` | `sorted()` |
| --- | --- | --- |
| 归属 | 列表**对象方法** | Python **内置函数** |
| 签名 | `a.sort(*, key=None, reverse=False)` | `sorted(iterable, *, key=None, reverse=False)` |
| 参数 `key` | 单参数函数，返回用于比较的键 | 同左；如 `key=len`、`key=str.casefold`、`key=lambda t: t[1]` |
| 参数 `reverse` | 布尔，`True` 为降序 | 同左 |
| 返回值 | `None` | **新列表** |
| 是否改原数据 | 是 | 否 |
| 稳定性 | 稳定（相等键保持原相对顺序） `※` | 稳定 `※` |

| 结论 | 内容 |
| --- | --- |
| 只能关键字传参 | 签名中的 `*` 表示 `key` / `reverse` 不能按位置传 |
| `key` 是函数不是值 | 传 `len` 而非 `len(x)`；传 `str.casefold` 而非 `x.casefold()` |
| 返回值陷阱 | `a = a.sort()` 会把 `a` 变成 `None` |
| `※` 多级排序 | `key=lambda t: (t[0], -t[1])` 用元组实现主 / 次关键字 |

## 8. 字典与集合

### 8.1 字典

| 签名 / 操作 | 作用与结论 | 参数说明 | 返回值 / 注意 |
| --- | --- | --- | --- |
| `d[key]` / `d[key] = v` | 取值 / 增改 | 键必须可哈希 | 键不存在时读抛 `KeyError` |
| `d.get(key, default=None)` | 安全取值 | `default`：键不存在时返回的值，默认 `None` | **不会**把键插入字典 |
| `d.setdefault(key, default=None)` | 取值并顺便建键 `※` | 键不存在时插入 `key: default` | 返回已有值或 `default`；适合计数 |
| `d.pop(key, default)` | 删除并取值 `※` | 不给 `default` 且键不存在抛 `KeyError` | 返回被删的值 |
| `d.popitem()` | 移除最后一项 `※` | 无参数；3.7+ 按 LIFO | 返回 `(key, value)` 元组 |
| `d.update(other, **kwargs)` | 批量更新 | 可传另一字典 / 键值对序列 / 关键字参数 | 返回 `None`，已存在的键被覆盖 |
| `d.keys()` / `values()` / `items()` | 视图 | 无参数 | 返回**动态视图**；`items()` 每项是 `(key, value)`，可 `for k, v in d.items()` |
| `key in d` | 判断**键**是否存在 | — | 不判断值；判值用 `v in d.values()` |
| `d.clear()` | 清空 | 无参数 | 返回 `None` |

### 8.2 集合

| 签名 / 操作 | 作用与结论 |
| --- | --- |
| `set(iterable)` | 由可迭代对象建集合，自动去重、无序、不可下标访问 |
| `add(x)` / `update(iterable)` | 加一个元素 / 加多个元素 |
| `remove(x)` / `discard(x)` | 前者元素不存在时抛 `KeyError`，后者静默 |
| `pop()` / `clear()` | 随机移除并返回一个元素 / 清空 |
| `union` / `intersection` / `difference` / `symmetric_difference` | 并 / 交 / 差 / 对称差，也可用对应运算符 |
| `issubset` / `issuperset` / `isdisjoint` | 子集 / 超集 / 无交集判断 |
| 查找效率 | `x in set` 平均 `O(1)`，`x in list` 为 `O(n)` |

## 9. 控制流、循环与函数

### 9.1 控制流

| 语法 | 结论 |
| --- | --- |
| `if / elif / else` | 条件后接冒号，靠缩进划分代码块 |
| `for x in iterable` | 逐个取元素；`for i in range(n)` 得到 `0..n-1` |
| `while condition` | 条件为真时重复；**必须**有使条件趋假的语句，否则死循环 |
| `break` | 立即跳出**最内层**循环；不影响外层 |
| `continue` | 跳过本轮剩余语句，直接进入下一轮 |
| `pass` | 占位，不执行任何操作 |
| `for ... else` / `while ... else` | `※` 仅当循环**未被 `break`** 时才执行 `else` |
| `try / except / finally` | `※` 捕获异常；`finally` 总会执行 |

### 9.2 函数

| 项 | 结论 |
| --- | --- |
| `def f(a, b=1, *args, c, **kwargs)` | 依次为必填位置参数、默认值参数、可变位置参数（元组）、仅关键字参数、可变关键字参数（字典） |
| 默认值陷阱 | 默认值只在定义时求值**一次**；不要用 `[]` / `{}` 作默认值 |
| `return` | 省略或 `return` 后无表达式时返回 `None`；可返回多个值（实为元组） |
| `lambda 参数: 表达式` | 匿名单行函数；常作 `key=` / `map()` 的参数 |
| 作用域 | `global` / `nonlocal` 声明后才能在函数内改写外层变量 |
| `if __name__ == "__main__":` | 主程序入口判断；作为模块导入时不执行 |

## 10. 三道编程题的结论与套路

### 10.1 题 38：判断单峰序列（One-Peaked Sequence）

| 项 | 内容 |
| --- | --- |
| 定义 | 存在 `i` 使 `a₁ < a₂ < … < aᵢ` 且 `aᵢ > aᵢ₊₁ > … > aₙ`；先**严格**递增到峰值，再**严格**递减；输入元素互不相同（distinct） |
| 边界结论 | `n ≤ 2` 恒输出 `Yes`；只递增、只递减都算单峰 |
| 核心套路 | 从 `i = 1` 起 `while i < n and a[i-1] < a[i]` 找第一个不再上升的位置 → 取 `peak = i - 1` → 检查峰后所有相邻对必须满足 `a[j] > a[j+1]` |
| 判非法条件 | 峰后出现 `a[j] <= a[j+1]` 即为 `No`；一次 `break` 即可退出 |
| 复杂度 | 单次扫描 `O(n)`，额外空间 `O(1)` |
| 常见错误 | 递增阶段写成 `<=`（必须严格）；只检查一次转折，漏掉「先降后升」如 `1 3 5 4 2 6` 这种两个峰；漏 `n ≤ 2`；用两个状态变量记录增减时逻辑写反 |
| 输出 | 严格匹配大小写 `Yes` / `No` |

### 10.2 题 39：不使用取模运算求余数

| 项 | 内容 |
| --- | --- |
| 限制 | `1 ≤ b ≤ 1000`；只能用**减法与比较**，禁止 `%` 运算符 |
| 核心套路 | `while a >= b: a -= b`；循环结束时的 `a` 就是余数 |
| 出口分支 | `a == 0` 输出 `Perfect`，否则输出 `a` 本身 |
| 边界结论 | `a < b` 时循环体一次都不执行，直接输出 `a`；必须保留该分支 |
| 常见错误 | 用了 `%` 直接判 0 分；循环没有趋假的更新导致死循环；忘记 `a < b` 分支 |
| 复杂度 | 减法次数 `O(a/b)`；`※` `a` 极大时可用倍增减法降到 `O(log(a/b))` |
| 输出 | 严格匹配 `Perfect` 的大小写 |

### 10.3 题 40：时间加 16 小时

| 项      | 内容                                                                                            |
| ------ | --------------------------------------------------------------------------------------------- |
| 输入格式   | 单行 24 小时制 `"HH:MM"`；先 `strip()` 去首尾空白                                                         |
| 拆分     | 必须 `s.split(":")`；`split()` 默认按空白，对 `"14:27"` **不会**拆开                                        |
| 统一单位   | `total = h * 60 + m + 16 * 60`，全部换算成分钟，避免小时与分钟混算                                              |
| 还原小时   | `new_h = (total // 60) % 24`，模 24 实现跨天循环                                                      |
| 还原分钟   | `new_m = total % 60`                                                                          |
| 输出格式   | `print(f"{new_h:02d}:{new_m:02d}")`；`:02d` 保证小时 / 分钟不足两位时补 0                                  |
| 常见错误   | 小时直接 `+16` 忘记模 24（`23+16=39`）；把分钟加进小时；字符串当整数参与算术导致 `"08"` 语义丢失；个位数忘记补 0（输出 `9:05` 而非 `09:05`） |
| 校验用例结论 | `14:27 → 06:27`；`23:59 → 15:59`；`08:00 → 00:00`                                               |

### 10.4 三题共用的做题套路

| 环节 | 结论 |
| --- | --- |
| 读入 | 先看输入是「一行一个数」「一行多个数」还是「带分隔符的字符串」，选对应的 `split` 形式 |
| 转换 | `input()` 一律返回 `str`，参与算术前必须 `int()` / `map(int, ...)` |
| 边界先行 | 在函数 / 分支最前面处理 `n ≤ 2`、`a < b`、空输入、单元素等特殊情况 |
| 主逻辑 | 单次线性扫描或带明确趋假条件的 `while`；一旦判定失败立即 `break` |
| 输出 | 用条件表达式或 f-string 一次成型；大小写、补零、分隔符必须逐字符对齐题目要求 |

## 11. 笔试速查表（指南附录「编程自测」）

| 项 | 内容 |
| --- | --- |
| 自测原题 | 边界条件（`n = 0, n = 1, n = 2`）记得处理吗？ |
| 对应考点 | 题 38 的 `n ≤ 2` 恒 `Yes`；题 39 的 `a < b`；空输入 / 单元素列表 |
| 标准读法 | `n = int(input())` 与 `a = list(map(int, input().split()))` 要分开读两行 |
| 标准输出 | `print("Yes" if is_one_peaked else "No")`；`print(f"{new_h:02d}:{new_m:02d}")` |

## 12. 高频易错点速查

| 陷阱 | 正确结论 |
| --- | --- |
| `input()` 结果直接做算术 | 返回 `str`，必须先 `int()` / `float()` |
| `sep` 与 `end` 混淆 | `sep` 管对象之间，`end` 管整行结尾 |
| `print()` 有返回值 | 返回 `None` |
| `map` / `range` / `zip` / `enumerate` / `reversed` 当列表用 | 都是迭代器（惰性、一次性），需要 `list()` 物化；`range` 例外，可重复迭代 |
| `a = a.sort()` | `sort()` 返回 `None`，会把 `a` 置空 |
| `strip()` 没生效 | `str` 不可变，必须写回 `s = s.strip()` |
| `strip("ab")` 当成去子串 | 是按**字符集合**去除首尾的 `a` 或 `b` |
| `split()` 用在 `"14:27"` | 默认按空白，不会按冒号拆；要 `split(":")` |
| `split("")` | 抛 `ValueError`；分隔符不能是空串 |
| `join` 传数字列表 | 元素必须全是 `str`，先 `map(str, ...)` |
| `remove` 与 `pop` 混用 | 前者按值删返回 `None`，后者按下标删并返回元素 |
| `remove` / `discard` 混用 | 集合中前者不存在时报错，后者不报错 |
| `//` 当成向零取整 | 是向下取整：`-7 // 3 = -3` |
| `%` 结果符号跟被除数 | 实际跟随**除数**：`-7 % 3 = 2` |
| `round(2.5)` 得 3 | 银行家舍入，结果为 `2` |
| `key in d` 判值 | 对 `dict` 判的是**键** |
| 用下标访问 `set` | 集合无序，不支持下标 |
| `while` 无更新语句 | 死循环；题 39 必须 `a -= b` |
| `break` 跳出所有循环 | 只跳出**最内层** |
| 默认值参数写 `[]` | 默认值只求值一次，会在多次调用间共享 |
| 三元表达式写反 | `A if cond else B`，`cond` 为真时取 `A` |
| `f"{s:02d}"` 用于字符串 | `d` 只接受整数；字符串用 `zfill(2)` 或 `:0>2` |
| 忘记输出补零 | 时间题必须 `:02d`，否则 `09:05` 变成 `9:5` |
| 忘记小时模 24 | `23 + 16` 必须 `% 24` 才回到 `15` |
| 输出大小写不严格 | `Yes` / `No` / `Perfect` 逐字符匹配，不能写成 `yes` / `YES` |
| 题 39 用了 `%` | 该题明确禁止取模运算符，用减法循环 |
| ※ 把 `is` 当 `==` | `is` 判对象身份，值比较一律用 `==` |
| ※ `and` / `or` 返回布尔 | 返回操作数本身，可用 `x or 默认值` |
| ※ 忽略 `EOFError` | `input()` 读到 EOF 会抛异常，多行读入优先 `sys.stdin.read().split()` |

## 13. 内置函数与对象方法归属速查

| 内置函数（直接调用） | 对象方法（`对象.方法()`） |
| --- | --- |
| `len(x)`、`sorted(x)`、`sum(x)`、`max(x)`、`min(x)` | `list.sort()`、`list.count()`、`list.reverse()` |
| `list(x)`、`tuple(x)`、`set(x)`、`dict(x)`、`str(x)`、`int(x)` | `str.split()`、`str.join()`、`str.strip()`、`str.replace()` |
| `map(f, it)`、`filter(f, it)`、`zip(*it)`、`enumerate(it, start)` | `dict.get()`、`dict.keys()`、`dict.values()`、`dict.items()` |
| `range(...)`、`reversed(x)`、`all(x)`、`any(x)`、`abs(x)`、`round(x, n)` | `set.add()`、`set.remove()`、`set.discard()` |
| `input(prompt)`、`print(*objects, sep, end, file, flush)` | `list.append()`、`list.extend()`、`list.pop()`、`list.insert()` |
