# 基础



## 数据

### 基本数据类型

Java 有8种基本数据类型

数字类型: int 、short、long、byte、double、float

布尔类型: boolean

字符类型: char

| 整型 | 范围 | 空间 |
| :--- | :--- | :--- |
| byte | -128~127 | 1字节 |
| short | -32768~32767 | 2字节 |
| int | -21亿~21亿 | 4字节 |
| long | -9亿亿~9亿亿 | 8字节 |

| 浮点型 | 范围 | 空间 |
| :--- | :--- | :--- |
| float | 精确小数点后6-7位 | 4字节 |
| double | 有效位15位 | 8字节 |

**关于浮点数的误差**

2.0-1.1等于多少等于0.89999999

2.0-0.1却等于0.1

原因：

浮点数值采用二进制表示，在二进制中无法精确表示 1/10

1.0 - 0.1 的计算结果截取时进位了，所以是0.9；而 2.0 - 1.1 的结算结果截取时没有进位

**浮点数的比较**

> 【强制】浮点数之间的等值判断，基本数据类型不能用来比较，包装数据类型不能用 equals 来判断。
>
> 说明：浮点数采用“尾数+阶码”的编码方式，类似于科学计数法的“有效数字+指数”的表示方式。二进 制无法精确表示大部分的十进制小数，具体原理参考《码出高效》。
>
> 参考:阿里巴巴开发规范嵩山版 \(四\) OOP 规约第9点

商业计算应该使用 BigDecimal 避免误差

```java
反例：
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
if (a == b) {
 // 预期进入此代码快，执行其它业务逻辑
 // 但事实上 a==b 的结果为 false
}
Float x = Float.valueOf(a);
Float y = Float.valueOf(b);
if (x.equals(y)) {
 // 预期进入此代码快，执行其它业务逻辑
 // 但事实上 equals 的结果为 false
}

正例：
(1) 指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的。
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
float diff = 1e-6f;
if (Math.abs(a - b) < diff) {
 System.out.println("true");
}
(2) 使用 BigDecimal 来定义值，再进行浮点数的运算操作。
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");
BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);
if (x.equals(y)) {
 System.out.println("true");
}
```

BigInteger 类 和 BigDecimal 类将在下一章节做具体介绍

> 任何货币金额，均以最小货币单位且整型类型来进行存储。
>
> 比如说人民币的最小单位是分，那假设一个商品的价格是1元钱，那就存到数据库的 price 字段，字段类型是 int 或者 bigint，值为 100，单位是分，也就是100分。前端显示 100/100即可
>
> 参考:阿里巴巴开发规范嵩山版 \(四\) OOP 规约第8点

**注意**

1. 使用 long 类型需要在数据后面加 L 否则会作为整形解析

### 复杂数据类型

#### 包装类型

基本数据类型对应的包装类型: Integer、Short、Long、Double、Float、Byte、Boolean、Character

拆箱装箱详解参考 [https://www.cnblogs.com/dolphin0520/p/3780005.html](https://www.cnblogs.com/dolphin0520/p/3780005.html)

Java 大部分的基本类型都实现了**常量池**的技术

> 什么是常量池技术

Integer 缓存源码

```java
/**
*此方法将始终缓存-128 到 127（包括端点）范围内的值，并可以缓存此范围之外的其他值。
*/
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

**应用场景：**

1. Integer i1=40；Java 在编译的时候会直接将代码封装成 Integer i1=Integer.valueOf\(40\);，从而使用常量池中的对象。
2. Integer i1 = new Integer\(40\);这种情况下会创建新的对象。

```java
  Integer i1 = 40;
  Integer i2 = new Integer(40);
  System.out.println(i1 == i2);//输出 false，因为创建了新的对象
```

```java
  Integer i1 = 40;
  Integer i2 = 40;
  Integer i3 = 0;
  Integer i4 = new Integer(40);
  Integer i5 = new Integer(40);
  Integer i6 = new Integer(0);

  System.out.println("i1=i2   " + (i1 == i2));      //true 指向常量池中的缓存对象
  System.out.println("i1=i2+i3   " + (i1 == i2 + i3));//true 自动拆箱，如果有运算符 比较值
System.out.println("i1=i2+i3   " + (i1.equals(i2 + i3));//true 先拆箱，数值加完后装箱，调用inValue再比较
  System.out.println("i1=i4   " + (i1 == i4));    // new新的对象，false
  System.out.println("i4=i5   " + (i4 == i5)); // flase 
  System.out.println("i4=i5+i6   " + (i4 == i5 + i6));// true 值比较
  System.out.println("40=i5+i6   " + (40 == i5 + i6));// true 值比较
```

**注意**

1. 如果 == 号两侧没有运算符，则比较的是对象，如果有运算符则会触发拆箱调用 intValue 比较值

#### 大数值类

**BigInteger** 

为了处理任意精度的整数运算

```java
//将常用类型转换为 BigInteger 类型
BigInteger a = new BigInteger("211111111111111111111111111111111");
BigInteger b = new BigInteger("9999999999999999999999999999");


//常用方法
a.add(b)   //+
a.subtract(b) //-
a.multiply(b) //*
a.divide(b) // ÷
```

**BigDecimal**

为了解决基本浮点类型精度丢失的问题

```java
//基本类型转为 BigDecimal 类型
BigDecimal a = new BigDecimal("2.0");
BigDecimal b = BigDecimal.valueOf(1.0);//实际还是内部调用了toString（）方法
```

> 11.【强制】禁止使用构造方法 BigDecimal\(double\)的方式把 double 值转化为 BigDecimal 对象。 说明：BigDecimal\(double\)存在精度损失风险，在精确计算或值比较的场景中可能会导致业务逻辑异常。
>
> 参考：阿里巴巴开发规范嵩山版 \(四\) OOP 规约第12点

如：BigDecimal g = new BigDecimal\(0.1f\);

实际的存储值为：0.10000000149 正例：优先推荐入参为 String 的构造方法，或使用 BigDecimal 的 valueOf 方法，此方法内部其实执行了 Double 的 toString，而 Double 的 toString 按 double 的实际能表达的精度对尾数进行了截断。

```java
BigDecimal recommend1 = new BigDecimal("0.1");
BigDecimal recommend2 = BigDecimal.valueOf(0.1);
```

BigDecimal 数值比较应该使用 Compareto方法

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");
BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);
if (x.compareTo(y) == 0) {
 System.out.println("true");
}
```

> 10.【强制】如上所示 BigDecimal 的等值比较应使用 compareTo\(\)方法，而不是 equals\(\)方法。 说明：equals\(\)方法会比较值和精度（1.0 与 1.00 返回结果为 false），而 compareTo\(\)则会忽略精度。
>
> 参考：阿里巴巴开发规范嵩山版 \(四\) OOP 规约第10点

### 常量

常量 final 关键字修饰

类常量 static final 关键词修饰 其他类的方法也可以使用这个常量

### 字符串

> 23.【推荐】循环体内，字符串的连接方式，使用 StringBuilder 的 append 方法进行扩展。
>
> 说明：下例中，反编译出的字节码文件显示每次循环都会 new 出一个 StringBuilder 对象，然后进行 append 操作，最后通过 toString 方法返回 String 对象，造成内存资源浪费。
>
> 参考：阿里巴巴开发规范嵩山版 \(四\) OOP 规约第23点

```java
反例： String str = "start"; 
for (int i = 0; i < 100; i++) {
    str = str + "hello"; 
}
```

StringBuilder 和 StringBuffer ，String 的区别

|  | StringBuilder | StringBuffer | String |
| :--- | :--- | :--- | :--- |
| 线程 | 单线程，线程不安全 | 多线程同步，线程安全 |  |
| 效率 | 效率高 | 效率低 |  |
|  |  |  | 不可变字符串 |

StringJoiner

## 方法（函数）

### 按值调用

按值调用：方法接受调用者提供的值

引用调用：接受调用者提供的变量的地址

例1：

```java
public class functionDemo {
    public static void main(String[] args) {
        int a = 10, b = 20;
        swap(a, b);
        System.out.println(a);
        System.out.println(b);
    }

    private static void swap(int a, int b) {
        int tmp = b;
        b = a;
        a = tmp;
        System.out.println(a);
        System.out.println(b);
    }
}
```

输出

```java
20
10
10
20
```

用于a,b是 int 类型，是基本数据类型。因此通过方法是==不能修改基本数据类型的参数==

例2：

```java
public class functionDemo2 {
    public static void main(String[] args) {
        Employee e1 = new Employee(10);
        Employee e2 = new Employee(20);
        swap(e1, e2);
        System.out.println(e1.salary);
        System.out.println(e2.salary);
    }
    private static void swap(Employee x, Employee y) {
        Employee tmpEmployee = x;
        x = y;
        y = tmpEmployee;
        System.out.println(x.salary);
        System.out.println(y.salary);
    }
}
```

输出

```java
20
10
10
20
```

x，y 被初始化为两个员工对象的引用的拷贝，方法交换的是两个拷贝。方法结束后，参数变量 x , y 被丢弃了。原来的员工变量 e1 , e2 仍然引用方法调用之前引用的对象。

例3：

```java
public class functionDemo2 {
    public static void main(String[] args) {
        Employee employee = new Employee(10);
        System.out.println(employee.salary);
        change(employee);
        System.out.println(employee.salary);
    }

    private static void change(Employee x) {
        x.salary = 20;
        System.out.println(x.salary);
    }
```

结果

```java
10
20
20
```

![image-20210222214730959](https://gitee.com/qqggzwm/blog-img/raw/master/img/20210222214731.png)

1. x被初始化为 employee 的拷贝，这里是一个对象的引用。
2. change 方法应用于这个对象的引用。x 和 employee 同时引用的那个 Employee 对象的薪资提高了200%。
3. 方法结束后虽然参数 x 不再使用，但是 employee 对象仍然引用那个被提高200%薪资的对象。

### 重载重写

overload 重载 —— 方法名一致，参数不同

> 方法的名称和参数称为方法的签名，返回值不是签名的一部分。因此不存在两个方法名一致，参数一致，返回值不一致。

overwrite 重写——继承父类的方法，方法体内的实现不同

## 数组

数组是引用类型

```java
int[] oldArr = newArr;
// 修改旧数组的值，新数组下标为5的数值也会变
oldArr[5] = 100;
```

![image-20210222205216035](https://gitee.com/qqggzwm/blog-img/raw/master/img/20210222205223.png)

原因：数组是oldArr 和 newArr 引用的是同一个数组

**排序**

Arrays.sort\(\)方法==优化的快速排序==

![img](https://gitee.com/qqggzwm/blog-img/raw/master/img/20210202194605.png)

**拷贝**

反例：

```java
int[]  arr  = barr;
```

正例：

```java
Arrays.copyOf(oldArr,oldArr.length)

int[] barr = Arrays.copyOf(new int[] { 1, 2, 3 }, 3);
```

## Java面向对象

### 对象和类

**成员变量和局部变量的区别**

1. 语法上看成员变量属于类，局部变量属于方法；成员变量可以被public,privte,static 等修饰符修饰，而局部变量是不能被这些修饰符修饰的。
2. 从变量在内存的位置来看。如果是成员变量被 static 修饰，是属于类的，否则是属于对象的。对象变量存储在堆内存，局部变量存储在栈内存。
3. 从变量的生命周期来看，如果是被 static 修饰的成员变量，是属于类的，对象未被创建也是存在的。如果是未被 static 修饰的成员变量随着对象的创建而产生。局部变量则是随着方法的调用结束而消失的。
4. 成员变量如果没有赋值则会被赋上类型的默认值，局部变量不会赋初始值。
5. 
**创建一个对象用什么运算符?对象实体与对象引用有何不同?**

new 运算符，new 创建对象实例（对象实例在堆内存中），对象引用指向对象实例（对象引用存放在栈内存中）。一个对象引用可以指向 0 个或 1 个对象（一根绳子可以不系气球，也可以系一个气球）;一个对象可以有 n 个引用指向它（可以用 n 条绳子系住一个气球）。

### 面向对象三大特性

封装，继承，多态

### 构造器

1. 构造器与类同名
2. 构造器可以无参也可以有1个或多个参数
3. 可以有一个以上的构造器，如果没有显示的构造器，默认无参构造
4. 构造器无返回值
5. 构造器伴随着 new 操作一起调用
6. 构造器内部不能定义和实际域（成员变量）重名的局部变量

**构造器的作用**

对数据域进行参数的初始化赋值操作

**调用子类构造方法构造器之前先调用父类构造器的作用**

子类的构造器不能访问父类的私有域，必须利用父类的构造器进行初始化，即通过super 对超类的构造器进行调用

### 修饰符final 、static

**final** 关键字一般用于不可变类

**static** 修饰的方法和变量是属于类的，而不是属于类对象的

```java
class Employee{

    private static int id = 1;
    private int salary;

    public static int getId(){
        return id;
    }

}
```

即使没有一个雇员对象，雇员的id也是存在的，因为==id是属于类的，不是属于对象的==

获取 id 属性

```java
// 无需对象
Employee.getId();

// getId方法也可以省略static修饰符，改用对象调用方法(不推荐阿里巴巴规范)
public int getId(){return id;}

new Employee().getId();
```

> 【强制】避免通过一个类的对象引用访问此类的静态变量或静态方法，无谓增加编译器解析成 本，直接用类名来访问即可
>
> 参考：阿里巴巴开发规范嵩山版 \(四\) OOP 规约第1点

### 其他重要的知识

**== 和 equals 方法 的区别**

1. 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象\(==基本数据类型比较的是值，引用数据类型比较的是内存地址==\)。
2. equals\(\): 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：
   * 情况 1：类没有覆盖 equals\(\) 方法。则通过 equals\(\) 比较该类的两个对象时，等价于通过“==”比较这两个对象。
   * 情况 2：类覆盖了 equals\(\) 方法。一般，我们都覆盖 equals\(\) 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true \(即，认为这两个对象相等\)。

例：

```java
    String str = "999"; 
    String str1 = "999"; // 同一个对象引用
    System.out.println(str == str1); // true

    String str2 = new String("666");
    String str3 = new String("666"); // 创建新的对象
    System.out.println(str2 == str3); // false
```

说明：

1. String 类型是实现了常量池技术的，当创建一个新的字符串之前会先在常量池中查找和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。
2. 而new String 是之间创建对象的，没有使用常量池技术的。
3. String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。

**equals 方法和 hashCode方法**

Object 类的 equals 方法

**说说为什么重写了 equals 方法后要重写 hashCode 方法**

## 集合

![image-20210301160910792](https://gitee.com/qqggzwm/blog-img/raw/master/img/20210301160917.png)

