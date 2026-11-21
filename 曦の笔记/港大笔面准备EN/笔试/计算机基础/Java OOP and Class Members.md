# 1 Abstract Class（抽象类）
## 1.1 什么是抽象类？

抽象类（Abstract Class）是使用 `abstract` 关键字修饰的类。

它主要用于定义一类对象的共同属性和行为，并允许子类继承后实现具体功能。

语法：

```java
abstract class Animal {
    String name;

    // 抽象方法：只有声明，没有方法体
    abstract void makeSound();

    // 普通方法：有具体实现
    void eat() {
        System.out.println("Eating...");
    }
}
```

抽象类的特点：

1. 使用 `abstract` 关键字定义。
    
2. 可以包含抽象方法（Abstract Method）。
    
3. 可以包含普通方法（Concrete Method）。
    
4. 可以包含成员变量、构造方法等。
    
5. 不能直接创建抽象类的对象。
    
6. 子类继承抽象类后，需要实现其所有未实现的抽象方法，否则子类也必须声明为抽象类。
    

## 1.2 Abstract Method（抽象方法）

抽象方法只有方法声明，没有具体实现。

```java
abstract class Animal {
    abstract void makeSound();
}
```

子类需要实现抽象方法：

```java
class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Woof!");
    }
}
```

## 1.3 Concrete Method（具体方法）

具体方法是具有方法体、能够执行具体功能的方法。

```java
abstract class Animal {
    void eat() {
        System.out.println("Eating...");
    }
}
```

注意：抽象类不等于所有方法都是抽象方法。它可以同时包含抽象方法和具体方法。

# 2 Interface（接口）
## 2.1 什么是接口？

接口（Interface）是一种定义行为规范的机制。

它主要用于规定实现类应该具备哪些功能。

语法：

```java
interface Flyable {
    void fly();
}
```

实现接口：

```java
class Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("Flying...");
    }
}
```

## 2.2 接口的主要特点

1. 使用 `interface` 关键字定义。
    
2. 类通过 `implements` 关键字实现接口。
    
3. 一个类可以实现多个接口。
    
4. 接口不能直接实例化。
    
5. 接口中的普通抽象方法默认是 `public abstract`。
    
6. 接口中的变量默认是 `public static final`，必须初始化。
    

例如：

```java
interface Flyable {
    int MAX_SPEED = 100;

    void fly();
}
```

其中：

```java
int MAX_SPEED = 100;
```

实际上相当于：

```java
public static final int MAX_SPEED = 100;
```

## 2.3 Java 8 之后接口可以包含具体方法

在 Java 8 之前，接口主要通过抽象方法定义行为规范。

从 Java 8 开始，接口可以包含：

|Method Type|中文|是否有方法体|说明|
|---|---|---|---|
|Abstract Method|抽象方法|否|默认是 `public abstract`|
|Default Method|默认方法|是|使用 `default` 修饰|
|Static Method|静态方法|是|使用 `static` 修饰|

示例：

```java
interface MyInterface {

    // 抽象方法
    void test();

    // 默认方法
    default void show() {
        System.out.println("Default method");
    }

    // 静态方法
    static void print() {
        System.out.println("Static method");
    }
}
```

调用接口中的静态方法：

```java
MyInterface.print();
```

注意：

- 接口中的静态方法必须有具体实现。
    
- 接口中的静态方法通过接口名调用。
    
- 实现类不能通过普通的实例方法重写接口的静态方法。
    

## 2.4 Abstract Class vs Interface（抽象类与接口的区别）

| Comparison           | Abstract Class   | Interface                    |
| -------------------- | ---------------- | ---------------------------- |
|                      | 抽象类              | 接口                           |
| Definition           | `abstract class` | `interface`                  |
| Inheritance          | 使用 `extends`     | 使用 `implements`              |
| Multiple Inheritance | 一个类只能继承一个类       | 一个类可以实现多个接口                  |
| Instance Variables   | 可以有普通成员变量        | 字段默认是 `public static final`  |
| Constructor          | 可以有构造方法          | 不能定义普通构造方法                   |
| Abstract Methods     | 可以有              | 可以有                          |
| Concrete Methods     | 可以有              | 可以有 `default`、`static` 等具体方法 |
| Purpose              | 提供共同状态和行为        | 定义行为规范                       |

## 2.5 Java 的单继承与多接口实现

Java 中：

**一个类只能直接继承一个类，但可以实现多个接口。**

例如：

```java
abstract class Animal {
}

interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck extends Animal implements Flyable, Swimmable {

    @Override
    public void fly() {
        System.out.println("Flying");
    }

    @Override
    public void swim() {
        System.out.println("Swimming");
    }
}
```

这里：

- `Duck extends Animal`：继承一个抽象类。
    
- `Duck implements Flyable, Swimmable`：实现两个接口。
    

注意：抽象类本质上也是类，因此 Java 不允许一个类同时继承多个抽象类。

# 3 Static Keyword（static 关键字）
## 3.1 什么是 static？

`static` 是 Java 中的一个关键字，用于修饰类成员。

被 `static` 修饰的成员属于类（Class），而不是某个具体对象（Object）。

主要包括：

- Static Variable（静态变量）
    
- Static Method（静态方法）
    
- Static Block（静态代码块）
    
- Static Nested Class（静态嵌套类）
    

## 3.2 Static Variable（静态变量）

静态变量也称为类变量（Class Variable）。

它属于类，所有该类的实例共享同一份静态变量。

示例：

```java
class Student {
    static int count = 0;

    Student() {
        count++;
    }
}
```

测试：

```java
public class Main {
    public static void main(String[] args) {
        Student s1 = new Student();
        Student s2 = new Student();
        Student s3 = new Student();

        System.out.println(Student.count);
    }
}
```

输出：

```text
3
```

原因：

- `count` 是静态变量。
    
- 所有 Student 对象共享同一个 `count`。
    
- 每创建一个对象，`count` 就增加 1。
    

可以理解为：

```text
Student Class
    |
    |-- static int count = 3
    |
    |-- Object s1
    |-- Object s2
    |-- Object s3
```

注意：静态变量只有一份逻辑上的类级别变量，不会因为创建多个对象而为每个对象分别创建一份。

## 3.3 Instance Variable（实例变量）

实例变量属于对象，每创建一个对象，就有自己独立的实例变量。

示例：

```java
class Student {
    String name;
}
```

测试：

```java
Student s1 = new Student();
Student s2 = new Student();

s1.name = "Tom";
s2.name = "Jerry";

System.out.println(s1.name);
System.out.println(s2.name);
```

输出：

```text
Tom
Jerry
```

对比：

| Comparison       | Static Variable | Instance Variable |
| ---------------- | --------------- | ----------------- |
|                  | 静态变量            | 实例变量              |
| Belongs to       | Class           | Object            |
| Number of Copies | 类级别共享一份         | 每个对象各自一份          |
| Access           | 类名或对象名          | 对象名               |
| Lifetime         | 随类的生命周期管理       | 随对象生命周期管理         |

## 3.4 Static Method（静态方法）

静态方法属于类，而不是某个对象。

示例：

```java
class MathUtils {
    static int add(int a, int b) {
        return a + b;
    }
}
```

调用：

```java
int result = MathUtils.add(10, 20);
```

输出：

```text
30
```

静态方法的特点：

1. 可以直接通过类名调用。
    
2. 不需要创建对象。
    
3. 不能直接访问非静态成员变量。
    
4. 不能直接调用非静态方法。
    
5. 不能使用 `this` 和 `super`。
    

原因：

静态方法属于类，在没有创建对象时就可以调用；而非静态成员属于具体对象，需要通过对象访问。

## 3.5 Static Method vs Instance Method（静态方法与实例方法）

| Comparison              | Static Method | Instance Method |
| ----------------------- | ------------- | --------------- |
|                         | 静态方法          | 实例方法            |
| Belongs to              | Class         | Object          |
| Invocation              | 类名调用          | 对象调用            |
| Requires Object         | 不需要           | 需要              |
| Access Static Members   | 可以            | 可以              |
| Access Instance Members | 不能直接访问        | 可以              |
| Use `this`              | 不可以           | 可以              |

# 4 Method Overriding vs Method Hiding（方法重写与方法隐藏）

这是本题非常重要的考点。

## 4.1 Method Overriding（方法重写）

方法重写是指子类重新实现父类中继承而来的实例方法。

要求：

- 子类方法名相同。
    
- 参数列表相同。
    
- 返回类型相同或满足协变返回类型要求。
    
- 子类方法不能降低访问权限。
    
- 通常使用 `@Override` 注解。
    

示例：

```java
class Parent {
    void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    @Override
    void show() {
        System.out.println("Child");
    }
}
```

测试：

```java
Parent p = new Child();
p.show();
```

输出：

```text
Child
```

这是因为 Java 的实例方法支持运行时多态（Runtime Polymorphism），实际执行哪个方法取决于对象的实际类型。

## 4.2 Method Hiding（方法隐藏）

静态方法不能被真正重写。

如果子类定义了与父类同名、参数列表相同的静态方法，这种情况称为方法隐藏（Method Hiding）。

示例：

```java
class Parent {
    static void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    static void show() {
        System.out.println("Child");
    }
}
```

测试：

```java
Parent p = new Child();
p.show();
```

输出：

```text
Parent
```

原因：

静态方法属于类，调用哪个方法主要取决于**引用变量的声明类型**，而不是对象的实际类型。

如果写成：

```java
Child.show();
```

则输出：

```text
Child
```

## 4.3 Overriding vs Hiding

| Comparison           | Method Overriding     | Method Hiding        |
| -------------------- | --------------------- | -------------------- |
|                      | 方法重写                  | 方法隐藏                 |
| Applicable To        | 实例方法                  | 静态方法                 |
| Keyword              | 不需要特殊关键字              | `static`             |
| Binding              | Dynamic Binding（动态绑定） | Static Binding（静态绑定） |
| Depends On           | 对象的实际类型               | 引用变量的声明类型（或调用时指定的类名） |
| Runtime Polymorphism | 支持                    | 不支持                  |

**考试重点：**

- Instance Methods（实例方法）可以被重写。
    
- Static Methods（静态方法）不能被真正重写，只能被隐藏。
    
- `@Override` 不能用于重写父类静态方法。
    

# 5 Important English Terms（重要英文专有名词）

| English                           | 中文         |
| --------------------------------- | ---------- |
| Object-Oriented Programming (OOP) | 面向对象编程     |
| Abstract Class                    | 抽象类        |
| Abstract Method                   | 抽象方法       |
| Concrete Method                   | 具体方法       |
| Interface                         | 接口         |
| Implementation                    | 实现         |
| Implement an Interface            | 实现接口       |
| Encapsulation                     | 封装         |
| Inheritance                       | 继承         |
| Polymorphism                      | 多态         |
| Multiple Inheritance              | 多重继承       |
| Single Inheritance                | 单继承        |
| Multiple Interfaces               | 多个接口       |
| Static Keyword                    | static 关键字 |
| Static Variable                   | 静态变量       |
| Class Variable                    | 类变量        |
| Instance Variable                 | 实例变量       |
| Static Method                     | 静态方法       |
| Instance Method                   | 实例方法       |
| Static Block                      | 静态代码块      |
| Method Overriding                 | 方法重写       |
| Method Hiding                     | 方法隐藏       |
| Runtime Polymorphism              | 运行时多态      |
| Dynamic Binding                   | 动态绑定       |
| Static Binding                    | 静态绑定       |
| Access Modifier                   | 访问修饰符      |
| Constructor                       | 构造方法       |
| Default Method                    | 默认方法       |
| Multiple Implementation           | 多接口实现      |

# 6 Practice Question（例题）

### Question: Abstract Classes, Interfaces and Static Keyword

**English Question:**

Which of the following statements about abstract classes, interfaces and the `static` keyword is correct?

**中文题目：**

关于抽象类、接口和 `static` 关键字，以下说法正确的是：

**Options:**

A) A class can implement multiple interfaces and inherit multiple abstract classes.

B) Static methods can be overridden.

C) Static variables have only one copy in memory, shared by all instances.

D) Interfaces can contain concrete implementations of static methods.

**Answer: C**

### 解析

**A) 错误**

一个类可以实现多个接口，但只能直接继承一个类，包括抽象类。

```java
class A extends B implements C, D {
}
```

其中 `B` 可以是普通类或抽象类，`C` 和 `D` 是接口。

**B) 错误**

静态方法不能被真正重写（Overriding），只能被隐藏（Hiding）。

**C) 正确**

静态变量属于类，而不是某个具体对象。

同一个类的所有实例共享同一份静态变量。

**D) 正确性需要注意题目中的 Java 版本。**

Java 8 开始，接口允许定义具有具体实现的静态方法。

例如：

```java
interface Test {
    static void show() {
        System.out.println("Hello");
    }
}
```

因此，按照 Java 8 及之后的规则，D 也是正确的。

**重要：原题给出的答案 C 并不严谨。**

如果题目没有限定 Java 8 之前，那么 C 和 D 都正确。若按照 Java 7 及以前的规则，D 才是错误的。

# 7 Inheritance：继承什么、不继承什么

**题目方向：** 关于类的继承，下列说法正确的是？（选项常涉及多继承、private 可见性、构造器是否被调用）

**知识点：Inheritance**

## 7.1 子类到底继承了什么

|成员|是否被继承|说明|
|---|---|---|
|public / protected 字段与方法|✅ 继承|直接可用|
|private 字段与方法|⚠️ 存在于对象中，但**不可直接访问**|内存里确实有（父类部分必须初始化），但编译器禁止子类直接引用；只能通过父类的 public/protected 方法间接操作|
|**构造器**|❌ **不继承**|构造器不是成员方法；子类必须自己声明，或依赖默认构造器|
|static 成员|⚠️ 共享但**不参与多态**|属于类，通过类名访问；子类同名 static 方法是**隐藏**，不是重写|
|final 方法|✅ 继承但**不能重写**|—|

## 7.2 访问修饰符与可见性

|修饰符|本类|同包|子类（不同包）|任何地方|
|---|---|---|---|---|
|private|✅|❌|❌|❌|
|（default）|✅|✅|❌|❌|
|protected|✅|✅|✅|❌|
|public|✅|✅|✅|✅|

**protected 的细节（易错）：** 子类**跨包**时，只能通过「子类类型的引用」访问 protected 成员，不能通过父类类型的引用访问。

## 7.3 单继承、多实现

|语言|类的继承|接口|
|---|---|---|
|Java / C#|**单继承**|多实现|
|C++|**多继承**|用纯虚函数模拟接口|
|Python|**多继承**（MRO / C3 线性化）|抽象基类 ABC|

**所以「一个类可以继承多个类」这句话，只在 C++/Python 成立。** 题目未指明语言时，按 Java/C# 规则作答。

## 7.4 继承与多态的关系

不是「有了继承就不需要多态」，而是**继承是多态的前提**：

```java
Parent p = new Child();   // 向上转型
p.show();                 // 若 Child 重写了，运行期调用 Child 的版本
```

没有继承（或接口实现），就没有「父类型引用指向子类型对象」，也就没有运行期动态分派。

**所属知识点：** 计算机基础 / Java OOP / 继承。

# 8 Initialization Order（初始化顺序）

**题目方向：** 给出父类、子类的静态块、实例块、构造器，问输出顺序。

**知识点：Initialization Order**

## 8.1 固定顺序

```text
父类 static 块  →  子类 static 块  →  父类实例初始化块  →  父类构造器
                                        →  子类实例初始化块  →  子类构造器
```

一句话：**先父后子，先静态后实例。**

## 8.2 为什么 static 最先

static 属于**类**，在类加载/初始化阶段执行，与对象无关；而 `{ }` 实例块和构造器属于**对象**，在 new 时才跑。**谁属于类谁先跑。**

## 8.3 为什么父类在子类之前

子类构造器第一行隐式调用 `super()`，所以「父类部分先初始化完，才能初始化子类部分」——对象是一个整体，父类部分是它的地基。

## 8.4 示例

```java
class Parent {
    static { System.out.print("1"); }
    { System.out.print("2"); }
    Parent() { System.out.print("3"); }
}
class Child extends Parent {
    static { System.out.print("4"); }
    { System.out.print("5"); }
    Child() { System.out.print("6"); }
}
public class Main {
    public static void main(String[] args) { new Child(); }
}
```

输出：

```text
142356
```

## 8.5 构造器链的两个细节

- 子类构造器第一行若不是 `super(...)`，编译器自动插入**无参** `super()`；
- 若父类**没有无参构造器**（只写了带参的），子类必须**显式**写 `super(参数)`，否则编译错误；
- `this(...)` 与 `super(...)` 不能同时出现在第一行。

**所属知识点：** 计算机基础 / Java OOP / 对象初始化。

# 9 Polymorphism 的三条边界

**题目方向：** 哪些方法能被重写？字段会不会多态？

**知识点：Polymorphism 的边界**

## 9.1 不能重写的情况

|成员|能否重写|实际发生的是|
|---|---|---|
|static 方法|❌|**隐藏（hiding）**，编译期绑定|
|final 方法|❌|编译报错|
|private 方法|❌（子类不可见）|子类写同名方法是**新方法**|
|构造器|❌|不属于可继承成员|
|**字段（field）**|❌|**字段不多态**：编译期看引用类型|

## 9.2 字段不多态（最容易错的一条）

```java
class Parent { int x = 1; }
class Child extends Parent { int x = 2; }

Parent p = new Child();
System.out.println(p.x);   // 输出 1，不是 2
```

原因：字段的访问在**编译期**就按「引用变量的声明类型」确定了，不走动态分派。**只有方法才多态。**

## 9.3 overload vs override 再对照

|对比|Overload（重载）|Override（重写）|
|---|---|---|
|位置|同一个类（或父子类）|子类重写父类|
|签名|方法名同、**参数不同**|方法名 + 参数**完全相同**|
|绑定时机|**编译期**（静态绑定）|**运行期**（动态分派）|
|返回类型|可不同|必须相同或协变|
|能否多态|❌|✅|

**所属知识点：** 计算机基础 / Java OOP / 多态。

# 10 英文例题

**（1）继承**

Which of the following statements about class inheritance is correct?

A. A subclass can extend multiple superclasses in Java.

B. Private members of the superclass are directly accessible inside the subclass.

C. A subclass inherits the fields and methods of its superclass and can override them to change behavior.

D. The superclass constructor is never invoked when a subclass object is created.

E. Inheritance makes polymorphism unnecessary.

**答案：C**

**解析：** A 错——Java 类只能单继承（接口才可多实现），只有 C++/Python 支持多继承类；B 错——private 成员对子类不可见；D 错——子类构造器会（显式或隐式）先调用父类构造器；E 错——继承正是多态的基础。

**（2）static**

Which of the following statements about the `static` keyword is correct?

A. A static method can be overridden.

B. A static variable has a single copy in memory shared by all instances.

C. A new copy of a static variable is created for each object.

D. A static method can directly access the instance variables of its class.

E. Static variables are stored on the stack.

**答案：B**

**解析：** static 属于类，整个类只有一份。A 错——static 方法只能被**隐藏**，不能重写（编译期绑定）；C 与 B 互斥；D 错——static 方法里没有 `this`；E 错——static 变量在方法区/元空间（Java）或静态存储区（C/C++），不在栈上。

**（3）初始化顺序**

```java
class Parent {
    static { System.out.print("1"); }
    { System.out.print("2"); }
    Parent() { System.out.print("3"); }
}
class Child extends Parent {
    static { System.out.print("4"); }
    { System.out.print("5"); }
    Child() { System.out.print("6"); }
}
new Child();
```

What is the output?

A. 123456

B. 142356

C. 132456

D. 142536

E. 124356

**答案：B**

**解析：** 父静态(1) → 子静态(4) → 父实例块(2) → 父构造(3) → 子实例块(5) → 子构造(6)，即 142356。

**所属知识点：** 计算机基础 / Java OOP / 综合应用。
