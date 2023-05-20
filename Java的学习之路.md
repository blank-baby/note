JDK

java和javax都是Java的API包

java是核心包

javax的x是extension的意思，也就是扩展包

重要的类包

- java.applet 提供了需要创建一个小程序和用来跟其他小程序交流上下文的类；
- **java.awt 包含了所有用于创建用户界面和绘制图形和图像的类；**
-  java.beans 包含了beans（基于JavaBean架构组件）开发相关的类；
-  **java.io：关于Java输入输出编程的相关类和接口；**
-  **java.lang：包含了Java语言的核心类，如String/Math/System/Thread等，使用该包下面的类是不需要显式的导入的；**
-  **java.math 提供了执行任意精度整数算法（BigInteger）和任意精度小数算法的类；**
- **java.net：关于Java网络编程的一些类和接口；**
- java.nio 定义了缓冲器，它是数据容器，并且提供其他NIO包的概述；
- java.rmi 提供了RMI包，java的不同虚拟机之间的远程调用；
- java.security 提供了安全框架的类和接口；
-  **java.sql：定义了Java进行JDBC数据库编程的相关类和接口；**
-  **java.text：Java格式化相关的类；**
-  **java.time 日期、时间、时刻和时间段的主要API；**
- **java.util：包含了大量的工具类，如集合框架的，日期处理等；**



这些包都jdk里面都是编辑好的class文件



![image-20220106130955254](https://gitee.com/jiruixin/images/raw/master/images/image-20220106130955254.png)



解压这个文件就会看到这些class文件



## 1. Java的类型转换

   



| 类型   |         | 字节 |                                                             |      |
| ------ | ------- | ---- | ----------------------------------------------------------- | ---- |
| 整型   | byte    | 1    | -128-127                                                    |      |
| 整型   | short   | 2    | -32768  -  32767                                            |      |
| 整型   | int     | 4    | -2,147,483,648  -  2,147,483,647                            |      |
| 整型   | long    | 8    | -9,223,372,036,854,775,808   -   9,223,372,036,854,775,8087 |      |
| 字符型 | char    | 2    |                                                             |      |
| 浮点型 | float   | 4    |                                                             |      |
| 浮点型 | double  | 8    |                                                             |      |
| 布尔型 | boolean |      |                                                             |      |
|        |         |      |                                                             |      |

​	由低到高

​    byte,short,char->int->long->float->double

​    强制转换: 由高到低

​    自动转换: 由低到高

​    **注意点：**

	1. 不能对布尔类型转换
	2. 不能转换成不相干的类型
	3. 在高容量在低容量转换时是强制转换
	4. 转换涉及内存溢出的问题。

### 类型转换

第一条：八种基本数据类型中，除 boolean 类型不能转换，剩下七种类型之间都可以进行转换；

第二条：**如果整数型字面量没有超出 byte,short,char 的取值范围，可以直接将其赋值给byte,short,char 类型的变量**；

第三条：小容量向大容量转换称为自动类型转换，容量从小到大的排序为：
byte < short(char) < int < long < float < double，其中 short和 char 都占用两个字节，但是char 可以表示更大的正整数；

第四条：大容量转换成小容量，称为强制类型转换，编写时必须添加“强制类型转换符”，但运行时可能出现精度损失，谨慎使用；

第五条：**byte,short,char 类型混合运算时，先各自转换成 int 类型再做运算**；

第六条：**多种数据类型混合运算，各自先转换成容量最大的那一种再做运算**；



### 1.int  String  Integer的转换

```java
// int 与 Integer
Integer i = Integer.valueOf(int);
int i = Integer.intValue();
// int 与 String
String s = String.valueOf(int);
int s = Integer.parseInt(s); 
// Integer 与 String  
String s = String.valueOf(Integer);   
Integer s = Integer.valueOf(s); 

// 包装类的valueOf()方法会变成该包装类。
// 静态方法
xxx i = xxx.valueOf();

// 包装类的xxxValue()方法会变成xxx的基本类型
// 实例方法
xxx i = yyy.xxxValue();

// 包装类的parsexxx方法会变成xxx的基本数据类型.主要用来解析字符串。
// 静态方法
xxx i = yyy.parseXxx(String s);
```

### 2.String的方法

#### 1.静态方法

> copyValueOf

```java
char[] a = {'a','b','c'};
// 变字符数组为字符串
String s = String.copyValueOf(a);
System.out.println(s);
// 变字符数组为字符串(字符数组  起始位置  转换个数) 
System.out.println(String.copyValueOf(a, 1, 2));
```

> join

```java
// 连接字符串(连接符，要连接的字符串)
String s = "a";
System.out.println(String.join("-", s,s,s,s,s,s));
```

> format

```java
String str=String.format("Hi,%s今年%d", "小超",20);
System.out.println(str);
```

#### 2.实例方法

```java
// 返回字符串中指定索引的字符的Unicode代码点
"1a".codePointAt(0);

// 全部变小写
"AAAA".toLowerCase();
// 全部变大写
"aaaa".toUpperCase();

// 追加指定的字符串到后面
"1a".concat("22");
// 是否按自定的字符开头
"1a".startWith("1")
// 是否指定的字符串结尾    
"1a".endsWith("a");
// 查询第一次出现字符的位置    
"1a".indexOf("a")
// 查询最后一次出现字符的位置       
"1a".lastIndexOf("a");

// 按索引切割字符串
"01234567".substring(2,4);

// 按字符分割字符串
"0-1-2-3".split("-");

// 这两个区别
// replace的参数是char和CharSequence,即可以支持字符的替换,也支持字符串的替换
// replaceAll的参数是regex,即基于规则表达式的替换,比如,可以通过replaceAll("\\d", "*")把一个字符串所有的数字字符都换成星号;
// 以上两个方法会全部替换。但是replaceFirst只会替换第一个
"0-1-2-3".replace('-', '*');
"0-1-2-3".replaceAll("-", "");
"0-1-2-3".replaceFirst("-", "");

// 去除字符串开头和结尾的空格
"  0-1-2-3  ".trim();

// 字符串变成字符数组
"0-1-2-3".toCharArray();

// 返回索引处的字符
"0-1-2-3".charAt(1);
```

### 3.StringBuffer

底层是字符数组，初始容量是16

```java
StringBuffer s = new StringBuffer();

// 添加任何东西都会变成字符
s.append("11111111");

// 确保容量的最小值为这个数，如果小于这个数会扩容
s.ensureCapacity(30);

// 插入到当前的字符中
insert()
    
// 删除对应的字符 
delete();

// 当前的容量
s.capacity();
 
// 返回指定索引的字符
s.charAt(2);

// 返回指定索引字符的Unicode代码点
s.codePointAt(1);

// 返回第一次出现字符的索引
s.indexOf("1",1);

// 返回最后一次出现字符的索引
s.lastIndexOf("3");

// 在指定的索引替换字符串
s.replace(1,2,"45");

// 从指定位置切割字符串，返回一个新的字符串String
s.substring(3);

// 字符串翻转
s.reverse();

// 赋值指定的字符串到字符数组
// 字符串开始索引  结束索引  接收的字符数组  开始接收的索引
StringBuffer s = new StringBuffer("12345");
char[] chars = new char[5];
s.getChars(0,5,chars,0);
for (int i = 0; i < chars.length ; i++) {
    System.out.println(chars[i]);
}
```

### 4.StrungBuilder

方法与StringBuffer一样，但是不保证同步。

### 5.以上三个字符串类的比较

- StringBuffer是线程安全，可以不需要额外的同步用于多线程中;

- StringBuilder是非同步,运行于多线程中就需要使用着单独同步处理，但是速度就比StringBuffer快多了;
- String是字符串常量，StringBuffer与StringBuilder是字符串变量；
- StringBuffer与StringBuilder两者共同之处:可以通过append、indert进行字符串的操作。

> 执行速度

StringBuilder > StringBuffer > String

> 使用场景

String：适用于少量的字符串操作的情况

StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况

StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况



### 移位操作

> 左移 高位丢弃，地位用0补齐

```java
public class Main {
    public static void main(String[] args) {
        // << 左移操作
        int i = 65535;
        System.out.println(Integer.toBinaryString(i));
        System.out.println(Integer.toBinaryString(i << 4));
    }
}
// 打印结果
//1111111111111111
//11111111111111110000
```

> 右移 高位用**符号位**补齐，低位丢弃

如果是正数用0补齐

```java
public class Main {
    public static void main(String[] args) {
        int i = 65535;
        System.out.println(Integer.toBinaryString(i));
        System.out.println(Integer.toBinaryString(i >> 4));
    }
}

// 打印结果
//1111111111111111
//111111111111       （前面的0被省略）
//0000111111111111   （真实是这个样子）
```

如果是负数用1补齐

```java
public class Main {
    public static void main(String[] args) {
        int i = -65535;
        System.out.println(Integer.toBinaryString(i));
        System.out.println(Integer.toBinaryString(i >> 4));
    }
}

// 打印结果
//11111111111111110000000000000001
//11111111111111111111000000000000
```

> 无符号移动   高位用0补齐，低位丢弃

```java
public class Main {
    public static void main(String[] args) {
        int i = -65535;
        System.out.println(Integer.toBinaryString(i));
        System.out.println(Integer.toBinaryString(i >> 4));
    }
}

// 打印结果
//11111111111111110000000000000001
//1111111111111111000000000000         （前面的0被省略）
//00001111111111111111000000000000     （真实的样子）
```



## 集合

### 数组

```java
// Array全部是静态方法，通过这个类可以查询数组里面的值，但是不可以对数据的容量进行改变。
int[] nums = {1,3,5,9};
System.out.println(Array.get(nums, 2));
System.out.println(Array.getInt(nums,0));
System.out.println(Array.getDouble(nums,1));


// Arrays全部是静态方法，通过这个类可以查询数组里面的值，目前不可以对数据进行容量改变。
int[] nums = {1,3,5,9};
System.out.println(Arrays.toString(nums));  //打印 1359
Arrays.fill(nums,0);//对原来的对象进行操作，不会产生新对象。
System.out.println(Arrays.toString(nums));//打印0000


// list<Object>可以动态给列表添加元素。并且可以变成数组用toArray方法。
List<Object> list = new ArrayList<Object>();
int[] nums = {1,3,5,9};
list.add("我是");
list.add("纪锐鑫");
list.add(1);
for (Object list1:list) {
    System.out.println(list1.getClass());
    System.out.println(list1);
}
// 结果：
我是
class java.lang.String
纪锐鑫
class java.lang.String
1
class java.lang.Integer
    
    
// ArrayList类可以动态给列表添加元素。并且可以变成数组用toArray方法。
ArrayList<Object> list = new ArrayList<Object>();
list.add("我是");
list.add("纪锐鑫");
list.add(1);
Object[] objects = list.toArray();
for (int i = 0; i <objects.length ; i++) {
    System.out.println(objects[i]);
    System.out.println(objects[i].getClass());
}
// 结果：
我是
class java.lang.String
纪锐鑫
class java.lang.String
1
class java.lang.Integer
```



### 集合和数组区别

![img](https://gitee.com/jiruixin/images/raw/master/images/1201956-20190514103822436-1884438617.png)





### Collection类图

Collection<E> extends Iterable<E>

Collection的一级子类

![image-20220106220958007](https://gitee.com/jiruixin/images/raw/master/images/image-20220106220958007.png)



红色为重要的子类

### List主要的类

| name                 | 底层结构 | 线程安全           | 有序性 | 唯一   |
| -------------------- | -------- | ------------------ | ------ | ------ |
| ArrayList            | 数组     | 不安全             | 有序   | 不唯一 |
| LinkedList           | 双向链表 | 不安全             | 有序   | 不唯一 |
| Vector               | 数组     | 安全(synchronized) | 有序   | 不唯一 |
| CopyOnWriteArrayList | 数组     | 安全               | 有序   | 不唯一 |

### Set主要的类

| name                | 直接父类                               | 底层结构 | 线程安全 | 有序性 | 唯一 |
| ------------------- | -------------------------------------- | -------- | -------- | ------ | ---- |
| HashSet             | Set                                    | HashMap  | 不安全   | 无序   | 唯一 |
| TreeSet             | AbstractSet、NavigableSet（SortedSet） | TreeMap  | 不安全   | 有序   | 唯一 |
| LinkedHashSet       | Set、HashSet                           | HashMap  | 不安全   | 有序   | 唯一 |
| CopyOnWriteArraySet |                                        |          | 安全     |        |      |
| SortedSet(接口)     | Set                                    |          |          |        |      |
| Graph(接口)         | Set                                    |          |          |        |      |



### Queue主要的类

![image-20220107110750785](https://gitee.com/jiruixin/images/raw/master/images/image-20220107110750785.png)

| name                             | 直接父类    | 底层结构 | 线程安全 | 有序性 | 唯一   |
| -------------------------------- | ----------- | -------- | -------- | ------ | ------ |
| ConcurrentLinkedQueue            | Queue       |          | 安全     | 有序   | 不唯一 |
| Deque                            | Queue(接口) |          |          |        |        |
| BlockingQueue                    | Queue(接口) |          |          |        |        |
| PriorityQueue                    |             | 数组     | 不安全   | 有序   | 不唯一 |
| ArrayDeque(双端队列，里面是数组) |             | 数组     | 不安全   | 有序   | 不唯一 |
|                                  |             |          |          |        |        |



### Map的主要类

![](https://gitee.com/jiruixin/images/raw/master/images/image-20220107153205511.png)





| name                  | 直接父类                             | 底层结构         | 线程安全 | 有序性 |
| --------------------- | ------------------------------------ | ---------------- | -------- | ------ |
| HashMap               | Map                                  | 数组+链表/红黑树 | 不安全   | 无序   |
| HashTable             | Map                                  | 数组+链表        | 安全     | 无序   |
| TreeMap               | AbstractMap、NavigableMap(SortedMap) | 红黑树           | 不安全   | 有序   |
| LinkedHashMap         | Map                                  | 数组+链表/红黑树 | 不安全   |        |
| SortedMap             | Map(接口)                            |                  | 不安全   |        |
| ConcurrentMap         | Map(接口)                            |                  | 安全     |        |
| ConcurrentHashMap     | ConcurrentMap                        | 数组+链表/红黑树 | 安全     | 无序   |
| ConcurrentSkipListMap | ConcurrentMap                        | 单向链表         | 安全     | 无序   |



> HashMap和HashTable的区别

- HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null

- HashTable键和值都不可以为null
- HashTable线程安全，HashMap线程不安全





### 集合的遍历

> collection

- foreach遍历
- 迭代器遍历
- 转成数组遍历
- 变成流遍历

```java
Collection<Object> objects = new ArrayList<>();

// foreach遍历
for (Object o:objects) {
}

// 迭代器遍历
Iterator<Object> iterator = objects.iterator();

// 转成数组遍历
Object[] objects1 = objects.toArray();

// 变成流计算
```

> map遍历

只遍历key

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();

// 第一种
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
    System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
}

// 第二种。再遍历set(上面三种)就可以拿到key
Set<Integer> objects = map.keySet();

// 第三种 map的forEach，参数k是key，v是value
map.forEach((k,v)->{
});

```

只遍历value

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
 
// 第一种
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
    System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
}

// 第二种。再遍历Collection(上面四种)就可以拿到key
Collection<Integer> values = map.values();

// 第三种 map的forEach，参数k是key，v是value
map.forEach((k,v)->{
});
```

遍历key和value

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();

// 第一种
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
    System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
}

// 第二种。再遍历set(上面四种)就可以拿到key和value
Set<Integer> objects = map.keySet();

// 第三种 map的forEach，参数k是key，v是value
map.forEach((,v)->{
});

```

迭代器遍历踩坑：在迭代器遍历的时候不可以直接用**集合对象**进行删除等操作

### 集合之间的转换

> List与set互转

```java
// List转Set
ArrayList<Object> list = new ArrayList<>();
HashSet<Object> set = new HashSet<>(list);

// Set转List
ArrayList<Object> objects = new ArrayList<>(set);
```



### 集合和数组之前的转换

> Collection与数组互转

```java
Collection<Object> objects = new ArrayList<>();

// Collection的子类都可以转成数组
Object[] objects1 = objects.toArray();

// 数组转List
List<Object> objects2 = Arrays.asList(objects1);

// 数组转Set
Set<Object> staffsSet = new HashSet<>(Arrays.asList(objects1));
```



## 规范

### 命名规范

#### is前缀

【强制】**POJO 类中的任何布尔类型的变量，都不要加 is 前缀**，否则部分框架解析会引起序列 化错误。 说明：在本文 MySQL 规约中的建表约定第一条，表达是与否的值采用 is_xxx 的命名方式，所以，需要在 设置从 is_xxx 到 xxx 的映射关系。 版本号 制定团队 更新日期 备注 1.6.0 阿里巴巴与 Java 社区开发者 2020.04.22 泰山版，首次发布错误码统一方案 Java 开发手册 2/57 反例：定义为基本数据类型 Boolean isDeleted 的属性，它的方法也是 isDeleted()，框架在反向解析的时 候，“误以为”对应的属性名称是 deleted，导致属性获取不到，进而抛出异常。



包名采用小写并且单数形式

避免在子父类的成员变量之间、或者不同代码块的局部变量之间采用完全相同的命名， 使可读性降低

在常量与变量的命名时，表示类型的名词放在词尾，以提升辨识度。

**接口类中的方法和属性不要加任何修饰符号（public 也不要加）**，保持代码的简洁 性，并加上有效的 Javadoc 注释。尽量不要在接口里定义变量，如果一定要定义变量，确定 与接口方法相关，并且是整个应用的基础常量

各层命名规约： 

A) Service/DAO 层方法命名规约 

 1） 获取单个对象的方法用 get 做前缀。

 2） 获取多个对象的方法用 list 做前缀，复数结尾，如：listObjects。

 3） 获取统计值的方法用 count 做前缀。 

 4） 插入的方法用 save/insert 做前缀。

 5） 删除的方法用 remove/delete 做前缀。

 6） 修改的方法用 update 做前缀。 

B) 领域模型命名规约 

 1） 数据对象：xxxDO，xxx 即为数据表名。

 2） 数据传输对象：xxxDTO，xxx 为业务领域相关的名称。 Java 开发手册 4/57 

 3） 展示对象：xxxVO，xxx 一般为网页名称。 

 4） POJO 是 DO/DTO/BO/VO 的统称，禁止命名成 xxxPOJO。



**在 long 或者 Long 赋值时，数值后使用大写的 L，不能是小写的 l**



不同逻辑、不同语义、不同业务的代码之间插入一个空行分隔开来以提升可读性。 说明：任何情形，没有必要插入多个空行进行隔开。



任何货币金额，均以最小货币单位且整型类型来进行存储



浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals 来判断。

![image-20220117195721300](https://gitee.com/jiruixin/images/raw/master/images/image-20220117195721300.png)



关于基本数据类型与包装数据类型的使用标准如下： 

1） 【强制】所有的 POJO 类属性必须使用包装数据类型。 

2） 【强制】RPC 方法的返回值和参数必须使用包装数据类型。 

3） 【推荐】所有的局部变量使用基本数据类型。



定义 DO/DTO/VO 等 POJO 类时，不要设定任何属性默认值。

## 基础点



重写

- 方法名相同，参数类型相同
- 子类返回类型等于父类方法返回类型，
- 子类抛出异常小于等于父类方法抛出异常，
- 子类访问权限大于等于父类方法访问权限



字面值

```java
/**
Java表达式转型规则由低到高转换：
1、所有的byte,short,char型的值将被提升为int型；

2、如果有一个操作数是long型，计算结果是long型；

3、如果有一个操作数是float型，计算结果是float型；

4、如果有一个操作数是double型，计算结果是double型；

5、被final修饰的变量不会自动的改变类型,当两个final修饰的变量操作时,结果会根据左边变量的类型而转换
 */
byte b1=1,b2=2,b3,b6,b8;
final byte b4=4,b5=6,b7;
// 会报错 因为b1 b2运算会转成int计算，而左边是byte，int无法自动转成byte
//b3=(b1+b2);  /*语句1*/

//不会报错 因为b4 b5都是常量，因此计算结果会根据左边的变量定义
b6=b4+b5;    /*语句2*/

//会报错 b1会转成int类型，因此还是需要强制成byte
//b8=b1+b4;  /*语句3*/

//会报错
//b7=(b2+b5);  /*语句4*/
//System.out.println(b3+b6);
```



### 访问权限

| 访问控制符 | 任何地方 | 保外类 | 包内 | 类内 |
| ---------- | -------- | ------ | ---- | ---- |
| public     | OK       | OK     | OK   | OK   |
| protected  | NO       | OK     | OK   | OK   |
| 无         | NO       | NO     | OK   | OK   |
| private    | NO       | NO     | NO   | OK   |



### 对象的多态

父类引用子类的实例化，多态和static没有关系。

```java
 Object object = new Son1();
class Person1{

    public void eat(){
        System.out.println("|这是人类的吃饭");
    }
}

class Son1 extends Person1{
    public void read()
        System.out.println("这是儿子在读书");
    }

    public void eat(){
        System.out.println("这是人类的吃饭");
    }
}
```

- **main方法、静态方法、构造方法不能重写**
- **接口只能继承接口，并且可以多继承，不能继承抽象类和实体类。**



### 参数传递

**java的参数传递就是值复制的过程。**

传基本数据类型和String都不会改变原来的值。

传引用类型会把当前变量的值传过去。因为引用类型的变量保存的是对象的地址，所以传递的也是对象的引用。这样形参和实参都可以对对象的属性进行改变。



### 对象的强制转化



父类的属于高   子类的属于低

```java
package cn.runningone;

public class Learn14 {
    public static void main(String[] args) {

        //用父类接收子类可以  
        Object object = new Son1();
        //父类引用子类如果调用子类独有的方法需要强制转换
        //父类引用子类如果调用复写的方法优先子类的方法
        ((Son1) object).read();
        
        //用子类接受父类需要强转
        Son1 son = (Son1) new Person1();  
        //子类引用接收父类实例化，在编译期间没事，运行会报ClassCastException 这个错误
        //X instanceof Y    X Y之间有父子关系
        System.out.println(object instanceof Object);  //对
        System.out.println(object instanceof Person1); //对
        System.out.println(object instanceof Son1);	   //对
        System.out.println(object instanceof Father1); //错
    }
}

class Person1{

    public void eat(){
        System.out.println("|这是人类的吃饭");
    }
}

class Father1 extends Person1{
    public void walk(){
        System.out.println("这是人类的行走");
    }
}

class Son1 extends Person1{
    public void read()
        System.out.println("这是儿子在读书");
    }
}

```

### super

```java
//如果子类和父类中有一样的方法或者属性，需要使用super才可以调用到父类的方法或者属性
class father{
    public String name="father";

    public void eat(){
        System.out.println("这是父类");
    }
}

class son extends father{
    private String name="son";

    public void eat(){
        System.out.println("这是子类");
    }

    public void drink(){
        eat();
        super.eat();
        System.out.println(name);
        System.out.println(super.name);
    }

}
class test{
    public static void main(String[] args){
        son son1 = new son();
        son1.drink();

    }
}

```

上面的代码的运行结果

```
这是子类
这是父类
son
father
```

### 静态代码块

静态代码块，在虚拟机加载类的时候就会加载执行，而且只执行一次；
非静态代码块，在创建对象的时候（即new一个对象的时候）执行，每次创建对象都会执行一次.

相同点：都是在JVM加载类时且在构造方法执行之前执行，在类中都可以定义多个，一般在代码块中对一些static变量进行赋值。
不同点：静态代码块在非静态代码块之前执行(静态代码块—>非静态代码块—>构造方法)。　　　　
静态代码块只在第一次类加载时执行一次，之后不再执行，而非静态代码块在每new一次就执行一次。非静态代码块可在普通方法中定义(不过作用不大)；而静态代码块不行。



### 泛型

- 代表一种位置元素
  - E    Element
  - T    Type
  - K    Key
  - V    Value
- 必须在类名之后或者方法返回值之前
- 只具备执行Object方法的能力

> List、List<Object>、List<?> 区别

> <? extends T>   <? super T>

```java
//泛型的上下边界<? extends 父类>
class Father{
    
}

class Son extends Father{
    
}

class A<T>{
    
}
class B{
    public static void main(String[] args){
        A<Son> = new A<son>;
        //如果没有上届 无法引用后面的对象
        A<? extends Father> = new A<Father>;
        
    }
}
```

### this

- this是一个变量，一个引用。this指向自己的对象的一个变量。保存当前对象的地址
- this只能用在实例方法中。谁调用实例方法。不能用在static修饰的方法中。因为this指向当前对象，static在加载时还没有对象，所以无法修饰static的方法

### native

- native修饰的方法是代表Java的作用达不到了，区调用底层的C语言库
- 会进到jvm的本地方法栈。
- JNI(本地方法接口)：扩展Java的使用。

### Prin()   write()

- 当输出为字符转时没有区别

- 当输出整形时print会把整数变成字符再输出，而write直接对整数输出。

  

### equal  ==

- equal()方法比较的是两个引用变量的内容
- ==  比较的是引用变量的保存的地址值。

### IO

- ```java
  FileInputStream是从指定的文件输入到内存中，如果参数制定的文件不存在会报错
  FileOutputStream 是从内存中读取数据写入到制定文件，没有参数中的文件会自动创建
  ```

### Java的反射

![image-20210201092043017](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210201092043017.png)

![image-20210201094148558](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210201094148558.png)

![image-20210201101328698](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210201101328698.png)

```
Java总有Class对象的类型
class：外部类 
interface
[]：数组
enum:枚举
annotation：注解
peimitive type：基本类型
void
```

![image-20210201104017928](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210201104017928.png)  



