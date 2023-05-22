# java基础

## 1.break  continue return

### 1.break

```java
for (int i = 0; i <5 ; i++) {
    System.out.println("第一层");
    for (int j = 0; j < 3; j++) {

        if (j==0) break;
        System.out.println("第二层");
    }
}
//直接终止第二层的for循环
```

### 2.continue

```java
for (int i = 0; i <5 ; i++) {
    System.out.println("第一层");
    for (int j = 0; j < 3; j++) {

        if (j==0) continue;
        System.out.println("第二层");
    }
}
//结束第二层i=0的循环，接着第二层的下一次循环
```

### 3.return

```java
for (int i = 0; i <5 ; i++) {
    System.out.println("第一层"+i);
    for (int j = 0; j < 3; j++) {
        if (j==0) return;
        System.out.println("第二层"+j);
    }
}
//直接结束第一层和第二层的循环
```

## 2.多态

### 1.前提：**需要两个类之间有继承关系**

### 2.基础语法

#### 1.向上转型

```java
public class arrayJava {
    public static void main(String[] args) {
        //这是向上转型（自动就可以）分类型 = new 子类型 
        animal a1 = new cat();
        a1.run();//可以通过编译，但是运行没有效果
        //  a1.eat();编译无法通过
    }
}

class animal{
    void move(){
        System.out.println("动物在运动");
    }
    void run(){
        System.out.println("动物在跑步");
    }
}

class cat extends animal{
    void move(){
        System.out.println("小猫在运动");
    }
    void eat(){
        System.out.println("小猫在吃东西");
    }
}


```

#### 2.向下转型

```java
public class arrayJava {
    public static void main(String[] args) {
        animal a1 = new cat();
        cat a2 = (cat)a1;
        //只有需要调用子类型中独有的方法才会向下转型
        a2.eat();
    }
}

class animal{
    void move(){
        System.out.println("动物在运动");
    }
    void run(){
        System.out.println("动物在跑步");
    }
}

class cat extends animal{
    void move(){
        System.out.println("小猫在运动");
    }
    void eat(){
        System.out.println("小猫在吃东西");
    }
}
```

#### 3. 向下转型经典错误

```java
public class arrayJava {
    public static void main(String[] args) {
        animal a1 = new cat();
        dog a2 = (dog)a1;
        a2.eat();
	    //报错误java.lang.ClassCastException
        //dog和cat没有继承关系无法转型，所以在运行期间直接报错
    }
}

class animal{
    void move(){
        System.out.println("动物在运动");
    }
    void run(){
        System.out.println("动物在跑步");
    }
}

class cat extends animal{
    void move(){
        System.out.println("小猫在运动");
    }
    void eat(){
        System.out.println("小猫在吃东西");
    }
}

class dog extends animal{
    void move(){
        System.out.println("小狗在运动");
    }
    void eat(){
        System.out.println("小狗在吃东西");
    }
}
```

#### 4.instanceof

```java
public class arrayJava {
    public static void main(String[] args) {
        animal a1 = new cat();
        //instanceof判断 a1引用所指向的对象是不是dog，如果是返回true 否则返回false
        if(a1 instanceof dog){
            System.out.println("a1引用的对象是小狗");
            dog a2 = (dog)a1;
            a2.eat();  
        }else if (a1 instanceof cat){
            System.out.println("a1引用的对象是小猫");
            cat a2 = (cat)a1;
            a2.eat();
        }
    }
}

class animal{
    void move(){
        System.out.println("动物在运动");
    }
    void run(){
        System.out.println("动物在跑步");
    }
}

class cat extends animal{
    void move(){
        System.out.println("小猫在运动");
    }
    void eat(){
        System.out.println("小猫在吃东西");
    }
}

class dog extends animal{
    void move(){
        System.out.println("小狗在运动");
    }
    void eat(){
        System.out.println("小狗在吃东西");
    }
}
```

### 3.多态的应用

#### 1.不是多态

```java
public class ArrayJava {
    public static void main(String[] args) {
        People people = new People();
        people.feed(new Dog());
		//每次有新的宠物都要去改People类的feed方法中参数对象
        //很麻烦
    }
}

class People{

    void feed(Dog dog){
        dog.eat();
    }

}
class Cat{
    void eat(){
        System.out.println("小猫在吃东西");
    }
}
class Dog{
    void eat(){
        System.out.println("小狗在吃东西");
    }
}
```

#### 2.使用多态

```java
public class ArrayJava {
    public static void main(String[] args) {
        People people = new People();
        people.feed(new Dog());
		//增加新宠物不用改变People中的代码，使用多态在编译期可以运行，在运行期也ok
    }
}

class People{
    void feed(Animal animal){
        animal.eat();
    }
}
class Animal{
    void eat(){
       
    }
}

class Cat extends Animal{
    void eat(){
        System.out.println("小猫在吃东西");
    }
}

class Dog extends Animal{
    void eat(){
        System.out.println("小狗在吃东西");
    }
}
```

## 3.数组

### 1.数组是参数

```java
//run是方法，参数是数组，如果传入静态数组需要
run(new int[]{1,2})
//run({1,2}) 这样传参是错误的
```

## 4.public protected private

### 1. 范围大小

- public > protected > 缺省 >private
- pubic和缺省才可以修饰类，其他的不可以

### 2.public

对所有的类都是可行

### 3. protected

对当前类、同一包、子孙类都可以，不可以让其他类访问

### 4. 缺省

- 修饰类时对于外部包的类不可以继承，只有public的类才可以被继承

- 修饰成员和方法时只有当前类和同一包下的可以访问

### 5. private

只有当前类可以访问

## 5.时间格式转换

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String time = sdf.format(new Date());
System.out.println(time);

// YYYY与yyyy
// yyyy使用的是日历年，我们应该使用日历年这种方式
// YYYY使用的是当前周所在的年份（Week of Year）。如果week（周）没有包含1月1日（没有跨年），那么week就属于当前所在的年如果week包含了1月1日（跨年了），那么week就属于新的一年。


作者：过气的程序员DZ
链接：https://www.jianshu.com/p/a4a307477c63
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

// MM与mm
// MM 表示月份 mm表示分钟

// DD与dd
// DD表示在当年中的天数  dd表示在当月中的天数

// HH与hh
// HH表示24小时计时法  hh表示12小时计时法

// SSS与ss
// SSS表示毫秒  ss表示秒
```



