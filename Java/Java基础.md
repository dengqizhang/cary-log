# Java 基础

Date：2024-12-03

作者：Cary

## 一，数据类型

### 基本数据类型：

- 整数类型：byte(1 字节)、short（2 字节）、int（4 字节）、long（8 字节）

- 浮点类型: float（4 字节）、double（8 字节）

- 布尔类型：boolean

- 字符类型：char（2 字节）

### 引用数据类型：

- 引用数据类型： 类(class)，接口(interface)

## 二，数组与集合

数组的特点有固定大小，只能存储同类型元素等，在开发中很少使用，一般使用集合来代替。

### 主要集合接口

#### Collection 接口

这是集合框架的根接口，代表一组对象的集合，它主要有两个子接口，list 和 set。

#### list 接口

代表一个有序的集合，可以包含重复的元素，实现类有 ArrayList 等。

#### Set 接口

代表一个不包含重复元素的集合，常见的实现类有，HashSet 等。

#### Map 接口

代表一个键值对的集合，用于存储键值映射关系，常见的实现类有，HashMap 等。可以通过键来获取对应值。

### 常见集合类

#### ArrayList

基于动态数组实现，有序集合。

#### HashSet

使用哈希表实现，无序集合，并且拥有唯一元素特性。

#### HashMap

使用哈希表实现，提供快速的键值查找，无需集合

## 三，面向对象

面向对象编程（OOP）是一种编程范式，Java 是一种纯粹的面向对象语言。面向对象编程的核心概念包括封装、继承、多态。

### 方法

方法包含参数，返回值，方法名，方法体。

```
public 返回体 方法名(参数1,参数2){
    //方法体
}
```

### 重载

一个类中有多个方法具有相同的名字，但参数列表不同，编译器会根据参数的类型和数量来决定调用哪个方法。

```
public void Fun1(String params){
//方法体
}
```

```
public void Fun1(String params,String params2){
//方法体
}
```

### 封装

将数据和操作数据的方法都封装在一个类中，通过访问修饰符来控制对数据的访问权限

- private 私有,只能在当前类内部被访问

- protected 同包中的类可以访问，此外，不同包中的子类也可以访问

- public 可以被任何类访问

### 继承(extends)

允许一个类继承另一个类的属性和方法，子类可以扩展父类的功能，也可以重写父类的方法。使用场景在 Saas 的租户等功能，通常会有一个基类定义公共属性，实体类去继承基类。

```
// 父类
class Animal {
    private String name;
    public Animal(String name) {
        this.name = name;
    }
    public void eat() {
        System.out.println(name + " is eating.");
    }
    public String getName() {
        return name;
    }
}

// 子类
class Dog extends Animal {
    public Dog(String name) {
        super(name); // 调用父类的构造函数
    }
    public void bark() {
        System.out.println(getName() + " is barking.");
    }
}
```

### 多态

多态是指同一个行为有多种不同的表现形式，通过方法重写或重载以及父类引用指向子类来实现。

```
interface Shape {
    double calculateArea();
}

class Circle implements Shape {
    private double radius;
    public Circle(double radius) {
        this.radius = radius;
    }
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

class Rectangle implements Shape {
    private double length;
    private double width;
    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    @Override
    public double calculateArea() {
        return length * width;
    }
}

public class Main {
    public static void main(String[] args) {
        Shape shape1 = new Circle(5.0);
        Shape shape2 = new Rectangle(4.0, 6.0);
        System.out.println(shape1.calculateArea()); // 计算圆的面积
        System.out.println(shape2.calculateArea()); // 计算矩形的面积
    }
}
```

### 抽象类(abstract)

抽象类不能被实例化，它可以包含抽象方法(没有方法体的方法)和具体方法，抽象类的目的是为了给子类提供一个通用模板。

### 接口 （Interface）

接口是一种完全抽象的类型，它只包含方法签名和常量，用于定义行为规范，一个类可以实现多个接口。

### 枚举

枚举是一种特殊的类，用于定义一组有限的常量。枚举可以有属性和方法。

### 泛型 < E >

泛型允许在定义类，接口，和方法时使用类型参数，使代码通用和类型安全。

### 注解

注解可以在不改变代码结构和不影响运行逻辑的基础上，增加额外的功能，例如注入容器，日志记录，接口鉴权等。

### 异常处理

可以捕获程序运行期间出现的异常，异常可以被捕获和处理。

### 多线程

#### 1，线程的状态

关于线程的执行状态有：

- 新建状态
- 就绪状态
- 运行状态
- 阻塞状态
- 死亡状态

#### 2，创建线程的方式

线程实现一共有两种方式

第一种继承 java.lang.Thread,重写 run 方法,然后通过 start()方法启动线程。

第二种方法是通过实现 Runnable 接口，但是最后还是需要 new Thread, 并且调用 Thread 的 start()启动线程。

所以本质意义来说，创建线程只有一种方式，通过 Thread 的 start 方法，其他的方法都是在花式调用它。

#### 3，线程的同步

在多线程业务场景下，可能会导致多个线程同时访问共享资源，会导致读取脏数据等并发问题。

在这种情况下可以使用同步锁来保证同一时间只能有一个线程访问资源。

#### 4，线程的通讯

java 的 Object 类中有 wait、notify、notifyAll 方法可以实现线程中间的通讯，一个线程在等待某个条件满足时，可以调用 wait 方法进入等待状态。另一个线程在条件满足时，可以调用 notyfy 或 notifyAll 方法通知等待的线程。

#### 5，线程池

通过线程池可以管理线程，可以重复利用已经创建的线程，减少线程的创建和销毁，提高性能。

java 提供了`java.util.concurrent.Executors`工厂类来创建不同的线程池。

#### 6，反射

反射是一种在运行时动态获取类的信息、访问类的成员变量、方法和构造函数，并调用它们的机制。反射提供了一种强大的方式来实现动态编程，使得程序可以在运行时根据不同的条件执行不同的操作。

## Java8 特性

### StreamAPI

流可以优化代码结构，处理数据更简单。

流可以接收一个数据源并进行聚合操作

#### 流和 Collection 有什么不同？

1，流的操作会返回对象本身，这样操作可以串联成管道，如同流式风格。

2，内部迭代，For-Each 的方式是在集合外部进行迭代，流是通过访问者模式实现了内部迭代。

**实例**：

```
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
```

### Lambda 表达式

Lambda 表达式允许将函数当成参数传递给某个方法

语法如下

```
(parameters) -> expression
或
(parameters) ->{ statements; }
```

parameters 是参数列表，expression 或 { statements; } 是 Lambda 表达式的主体。如果只有一个参数，可以省略括号；如果没有参数，也需要空括号。

实例：

```
// 使用 Lambda 表达式计算两个数的和
MathOperation addition = (a, b) -> a + b;

// 调用 Lambda 表达式
int result = addition.operation(5, 3);
System.out.println("5 + 3 = " + result);

```

**为什么要使用 Lambda 表达式？**

1，代码会变得更简洁

2， 函数式编程支持

3，可以捕获外部作用域的变量

### 新日期时间 API

### 接口默认方法

声明接口默认方法语法如下：

```
public interface Vehicle {
   default void print(){
      System.out.println("我是一辆车!");
   }
}
```
