

## 1.基础

### 1.实现多线程方式

> 继承Thread

- 使用jmc.exe分析线程
- 线程的执行顺序都是随机的
- 执行的start()方法顺序不代表执行线程run的顺序

```java
public class FirstImpl {

    public static void main(String[] args) {
        ThreadImpl1 threadImpl1 = new ThreadImpl1();
        threadImpl1.start();
        System.out.println("主线程");

    }
}

class ThreadImpl1 extends Thread{
    @Override
    public void run() {
        System.out.println("副线程跑起来");
    }
}
```

> 实现Runnable接口（推荐这种方法）



```java
//使用lambda表达式
public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(()->{
                System.out.println("副线程跑起来");
        });
        threadImpl1.start();
        System.out.println("主线程");
    }
}
```

不同点:

- 内部的执行逻辑不一样

> Callable<>

这个多线程实现以后可以有返回值

```java
package unsafe;

import java.util.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread myThread = new MyThread();
        FutureTask<String> stringFutureTask = new FutureTask<>(myThread);
        new Thread(stringFutureTask,"a").start();
        System.out.println(stringFutureTask.get());

    }
}


class MyThread implements Callable<String>{

    @Override
    public String call() throws Exception {
        System.out.println("--------");
        return "123";
    }
}

```

![image-20220104103938288](https://gitee.com/jiruixin/images/raw/master/images/image-20220104103938288.png)

注意点

- 有缓存
- 结果可能会堵塞，需等待
- get方法会堵塞，所以放在最后



### 2.Thrad的方法

#### 基础方法

> static Thread currentThread()

返回代码段正在被哪个线程调用的线程对象

```java
public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(()->{
                System.out.println(Thread.currentThread()+"线程跑起来");
        });
        threadImpl1.start();
        System.out.println(Thread.currentThread()+"线程");

    }
}
// 打印的结果
Thread[main,5,main]线程
Thread[Thread-0,5,main]线程跑起来

```

run方法和start方法的区别

- run--立即执行，不会启动新的线程
- start--不确定执行run方法的时机，但会启动新线程



> public static boolean holdsLock(Object obj)

该方法当线程对制定对象保持锁时返回true。

```java
package againget;

public class Main {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().holdsLock(Main.class));
        synchronized (Main.class){
            System.out.println(Thread.currentThread().holdsLock(Main.class));
        }
        System.out.println(Thread.currentThread().holdsLock(Main.class));
    }
}

```

![image-20211229095531293](https://gitee.com/jiruixin/images/raw/master/images/image-20211229095531293.png)





> isAlive

判断当前线程是否存活

```java
public class FirstImpl {
    public static void main(String[] args) {
        ThreadImpl1 threadImpl1 = new ThreadImpl1();
        System.out.println(threadImpl1.isAlive());
        threadImpl1.start();
        System.out.println(threadImpl1.isAlive());
       

    }
}

class ThreadImpl1 extends Thread{
    @Override
    public void run() {
        
    }
}

```

> static void sleep(long millis)

**哪个线程执行到这个代码段哪个线程就睡眠**

```java
// 线程睡眠1秒钟
Thread.sleep(1000)
```

面试题

```java
public class FirstImpl {
    public static void main(String[] args) {
        ThreadImpl1 threadImpl1 = new ThreadImpl1();
        threadImpl1.start();
        try{
            // 问题：threadImpl1这个线程会睡5秒钟吗
            // 答案：不会，谁执行这个代码谁睡5秒钟 记住 那个线程执行这个方法那个线程睡眠
            threadImpl1.sleep(1000 * 5)
        }catch(InterruptedException e){
            e.printStackTrace();
        }   
    }
}

class ThreadImpl1 extends Thread{
    @Override
    public void run() {
        
    }
}

```





>   StackTraceElement[] getStackTrace() 

返货当前线程的跟踪元素数组 StackTraceElement[]

```java
public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(()->{
            StackTraceElement[] stackTrace = Thread.currentThread().getStackTrace();
            for (int i = 0; i <stackTrace.length ; i++) {
                System.out.println(stackTrace[i].getClassName());
                System.out.println(stackTrace[i].getMethodName());
            }   
        });
        threadImpl1.start();
    }
}
// 打印的结果
java.lang.Thread
getStackTrace
    
ThreadImpl.SendImpl
lambda$main$0
    
java.lang.Thread
run
```

> static void dumpStack()

返回堆栈跟踪信息到标准错误流

```java
public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(()->{
           Thread.dumpStack();
        });
        threadImpl1.start();
    }
}
```

打印结果

![image-20211221234615084](https://gitee.com/jiruixin/images/raw/master/images/image-20211221234615084.png)



> static Map<Thread,StackTraceElement[]> getAllStackTraces() 





> long getId() 

获取当前线程的唯一标识符





> public static boolean interrupted()
>
> public boolean isInterrupted()

判断线程是否有被中断的标志位

相同点：都是检查线程是否有被中断的标志位

不同点：

- 一个静态方法 ，一个动态方法
- interrupted()方法执行完后会清除状态标志位为false的功能
-  isInterrupted()，不清除状态标志位



> void interrupt() 

**这种方法只是做了一个停止的标志位，并没有真正停止线程**

> 正常情况下使用interrupt

```java
package ThreadImpl;

import java.util.Map;

public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(new MyThread());

        threadImpl1.start();
        try {
            Thread.sleep(1);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadImpl1.interrupt();
        System.out.println("++++++++++++++");

    }
}


class MyThread implements Runnable{

    private int i = 0;

    @Override
    public void run() {
        for (int j = 0; j <1000 ; j++) {
            System.out.println(j);
        }
    }
}

```



执行结果

![image-20211223232321191](https://gitee.com/jiruixin/images/raw/master/images/image-20211223232321191.png)

分析结果：发现都执行了interrupt方法，该线程还是会接着执行。所以interrupt方法不能阻止线程中断。

需要配合interrupted()和isinterrupted()方法来中断线程

#### 停止线程

##### 1.第一种

> interrupt()+interrupted()/isinterrupted()+抛异常中断程序

```java
package ThreadImpl;

import java.util.Map;

public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(new MyThread());

        threadImpl1.start();
        try {
            Thread.sleep(1);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadImpl1.interrupt();
        System.out.println("++++++++++++++");

    }
}


class MyThread implements Runnable{

    private int i = 0;

    @Override
    public void run() {
        try {
            for (int j = 0; j <1000 ; j++) {
                if (Thread.interrupted()){
                    throw  new InterruptedException();
                }
                System.out.println(j);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
    }
}


```



执行结果

![image-20211223233240209](https://gitee.com/jiruixin/images/raw/master/images/image-20211223233240209.png)

分析结果：这种方式最好让try尽可能包裹全部代码，因为catch的方法结束后，他会接着执行try外面的代码。



##### 2.第二种

> interrupt()+interrupted()/isinterrupted()+return

```java
package ThreadImpl;

import java.util.Map;

public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(new MyThread());

        threadImpl1.start();
        try {
            Thread.sleep(1);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadImpl1.interrupt();
        System.out.println("++++++++++++++");

    }
}


class MyThread implements Runnable{

    private int i = 0;

    @Override
    public void run() {

        for (int j = 0; j <1000 ; j++) {
            if (Thread.interrupted()){
                return;
            }
            System.out.println(j);
        }
    }
}

```

执行结果：

![image-20211223233540323](https://gitee.com/jiruixin/images/raw/master/images/image-20211223233540323.png)

分析结果：这种方式也可以直接中断程序。

两种方式怎么选择

- 当具有多个条件判断的的时候，可以选择抛异常方法，这样可以统一在catch代码块中处理。
- 当只有一个条件判断的时候可以选择return



> 线程睡眠状态下用interrupt

这种唤醒线程的方法利用java的异常机制。

```java
public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(new MyThread());

        threadImpl1.start();
        try {
            Thread.sleep(1000 * 4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 本来threadImpl1线程睡1个小时，利用interrupt方法在直接唤醒
        threadImpl1.interrupt();
    }
}


class MyThread implements Runnable{

    @Override
    public void run() {
        try {
            Thread.sleep(1000 * 60 * 60);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

执行结果

![image-20211223215846653](https://gitee.com/jiruixin/images/raw/master/images/image-20211223215846653.png)



- 检验线程睡醒是否还会执行下面的代码

```java
public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(new MyThread());
        threadImpl1.start();
        try {
            Thread.sleep(1000 * 4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadImpl1.interrupt();
        System.out.println("++++++++++++++");

    }
}


class MyThread implements Runnable{

    @Override
    public void run() {
        try {
            Thread.sleep(1000 * 60 * 60);
            System.out.println("**********");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("---------------");
    }
}

```

执行结果：

![image-20211223222442934](https://gitee.com/jiruixin/images/raw/master/images/image-20211223222442934.png)

分析结果：

 interrupt() 方法会唤醒睡眠的线程，但是线程会直接跳转到catch代码块中，并且会接着执行下面的代码

##### 3.第三种

> void stop()

暴力停止线程，此方法现在已经作废，因为有可能影响业务的完整性。

```java
package ThreadImpl;

import java.util.Map;

public class SendImpl {
    public static void main(String[] args) {
        Thread threadImpl1 = new Thread(new MyThread());

        threadImpl1.start();
        try {
            Thread.sleep(1000 * 4);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadImpl1.stop();
        System.out.println("++++++++++++++");

    }
}


class MyThread implements Runnable{

    @Override
    public void run() {
        try {
            Thread.sleep(1000 * 60 * 60);
            System.out.println("**********");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("---------------");
    }
}

```

打印结果

![image-20211223223410925](https://gitee.com/jiruixin/images/raw/master/images/image-20211223223410925.png)

结果分析：线程停止





> void suspend() 

暂停线程。方法已经过期。stop方法会销毁线程对象，但是该方法只会暂停线程，并不会销毁对象。

> void resume() 

恢复线程

> static void yield()

让当前线程放弃CPU资源，让其他任务去占用CPU。但有可能线程放弃后会立马又获得CPU



> void  getPriority()

获取线程优先级

> void  setPriority(int newPriority)

优先级高的线程优先获取CPU，能得到更多的CPU执行的时间片。

- 线程分为1-10  超过这个数会抛异常
- 线程继承性  A线程启动B线程，A、B线程的优先级一样
- 高优先级的线程不是绝对先执行完。这两者没有依赖关系。



> void setDaemon(true)

当进程中没有非守护线程时就会自动销毁。最典型的就是GC垃圾回收器。

- 只有使用setDaemon(true)这个方法才会设置为守护线程
- 必须在start()方法之前设置为守护线程，不然会报错。

## 数据安全

> 什么时候数据在多线程并发的环境下会存在安全问题呢？

- 多线程并发
- 有共享数据
- 共享数据有修改的行为

满足以上3个条件之后，就会存在线程安全问题



> **Java中有三大变量？**

	实例变量：在堆中。
	
	静态变量：在方法区。
	
	局部变量：在栈中。
	
	以上三大变量中：
		局部变量永远都不会存在线程安全问题。
		因为局部变量不共享。（一个线程一个栈。）
		局部变量在栈中。所以局部变量永远都不会共享。
	
	实例变量在堆中，堆只有1个。
	静态变量在方法区中，方法区只有1个。
	堆和方法区都是多线程共享的，所以可能存在线程安全问题。
	
	局部变量+常量：不会有线程安全问题。
	成员变量：可能会有线程安全问题。

> 聊一聊，我们以后开发中应该怎么解决线程安全问题？

	是一上来就选择线程同步吗？synchronized
		不是，synchronized会让程序的执行效率降低，用户体验不好。
		系统的用户吞吐量降低。用户体验差。在不得已的情况下再选择
		线程同步机制。
	
	第一种方案：尽量使用局部变量代替“实例变量和静态变量”。
	
	第二种方案：如果必须是实例变量，那么可以考虑创建多个对象，这样
	实例变量的内存就不共享了。（一个线程对应1个对象，100个线程对应100个对象，
	对象不共享，就没有数据安全问题了。）
	
	第三种方案：如果不能使用局部变量，对象也不能创建多个，这个时候
	就只能选择synchronized了。线程同步机制。

> java的锁

乐观锁

- 版本号

- CAS



悲观锁

- synchronized
- Lock



## 2.对象及变量的并发访问

### 1.synchronized方法

*synchronized加在非静态方法上默认锁的是当前对象，加在静态方法上锁的是类对象。*

#### 1.synchronized注意点

- 方法内部的的变量不存在非线程安全，永远都是线程安全，这是因为方法内部的变量具有私有属性

- synchronized修饰的方法不影响其他异步方法的使用。

```java
package synchronizedThread;

public class MyObject {

    synchronized public void methodA(){
        System.out.println(Thread.currentThread().getName()+"方法A");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end");
    }

    public void methodB(){
        System.out.println(Thread.currentThread().getName()+"方法B");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end");
    }
}

//////////////////////////////////////////////////////////////////////////////
package synchronizedThread;

public class Thread1 implements Runnable {

    private MyObject myObject;

    public Thread1(MyObject myObject){
        this.myObject = myObject;
    }

    @Override
    public void run() {
        myObject.methodA();
    }
}

//////////////////////////////////////////////////////////////////////////////
package synchronizedThread;

public class Thread2 implements Runnable {

    private MyObject myObject;

    public Thread2(MyObject myObject){
        this.myObject = myObject;
    }

    @Override
    public void run() {
        myObject.methodB();
    }
}

//////////////////////////////////////////////////////////////////////////////
package synchronizedThread;

public class Run {
    public static void main(String[] args) {
        MyObject myObject = new MyObject();
        Thread thread1 = new Thread(new Thread1(myObject));
        thread1.setName("thread1");
        Thread thread2 = new Thread(new Thread2(myObject));
        thread2.setName("thread2");
        thread1.start();
        thread2.start();
    }
}

```

执行结果

![image-20211228221847497](https://gitee.com/jiruixin/images/raw/master/images/image-20211228221847497.png)

结论

- A线程有对象的Lock锁，B线程可以异步的方式调用该对象的非synchronized方法



#### 2.脏读

```java
package dirtyread;

public class PublicVar {
    public String name = "A";
    public String pw = "AA";

    synchronized public void setValue(String name,String pw){

        try {
            this.name = name;
            Thread.sleep(5000);
            this.pw = pw;
            System.out.println("setValue"+Thread.currentThread().getName()+"  name="+this.name+"  pw="+this.pw);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void getValue(){
        System.out.println("getValue"+Thread.currentThread().getName()+"  name="+name+"  pw="+pw);
    }
}

//////////////////////////////////////////////////////////
package dirtyread;

public class Thead1 implements Runnable {

    private PublicVar publicVar;

    public Thead1(PublicVar publicVar){
        this.publicVar = publicVar;
    }

    @Override
    public void run() {
        publicVar.setValue("B","BB");
    }
}

///////////////////////////////////////////////////////////////
package dirtyread;

import synchronizedThread.Thread1;

public class Run {

    public static void main(String[] args) throws InterruptedException {
        PublicVar publicVar = new PublicVar();
        Thread thread = new Thread(new Thead1(publicVar));
        thread.start();
        Thread.sleep(200);
        publicVar.getValue();
    }
}
```



执行结果

![image-20211228230309805](https://gitee.com/jiruixin/images/raw/master/images/image-20211228230309805.png)

分析问题：当一个线程在更改实例属性中的值时，在没有完成更改的时候又有另外一条线程去读取值。这样就造成了脏读线程。

解决：给操作的方法都加上同步锁。



#### 3.synchronized重入

线程在获得某个对象的锁时，在没有释放之前还能再次获取该锁。

```java
package againget;

public class MyObject {

    public synchronized void method1(){
        System.out.println("method1");
        method2();
    }

    public synchronized void method2(){
        System.out.println("method2");
    }

}
 
///////////////////////////////////////////////
package againget;

public class Thread1 implements Runnable {
    @Override
    public void run() {
        MyObject myObject = new MyObject();
        myObject.method1();
    }
}

///////////////////////////////////////////////////
package againget;

public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(new Thread1());
        thread.start();
    }
}
```

执行结果

![image-20211228231756908](https://gitee.com/jiruixin/images/raw/master/images/image-20211228231756908.png)



#### 4.synchronized可继承

```java
package extend;

public class Father {

    public int i =10;

    public synchronized void method1(){
        i--;
        System.out.println("father的i="+i);
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

}

/////////////////////////////////////////////
package extend;

public class Son extends Father {

    @Override
    public synchronized void method1() {

        while(i>0){
            i--;
            System.out.println("son的i="+i);

            try {
                Thread.sleep(100);
                super.method1();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

///////////////////////////////////////////////////
package extend;

public class Thread1 implements Runnable {
    @Override
    public void run() {
        Son myObject = new Son();
        myObject.method1();
    }
}

/////////////////////////////////////////////////
package extend;

public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(new Thread1());
        thread.start();
    }
}

```

执行结果

![image-20211228232732045](https://gitee.com/jiruixin/images/raw/master/images/image-20211228232732045.png)

结论：当存在父子继承关系时，子类完全可以通过锁重入进入到父类方法中



#### 6.出现异常，锁自动释放

```java
package exception;

public class Service {

    synchronized public void method(){
        try {
            System.out.println(Thread.currentThread().getName()+"begin");
            Thread.sleep(2000);
            System.out.println(1/0);
            System.out.println(Thread.currentThread().getName()+"end");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

///////////////////////////////////////////////////////////
package exception;

public class Thread1 implements Runnable {

    private Service service;

    public Thread1(Service service){
        this.service = service;
    }

    @Override
    public void run() {
           service.method();
    }
}

////////////////////////////////////////////////////////////
package exception;

public class Thread2 implements Runnable {

    private Service service;

    public Thread2(Service service){
        this.service = service;
    }

    @Override
    public void run() {
           service.method();
    }
}

//////////////////////////////////////////////////////////
package exception;

public class Run {

    public static void main(String[] args) {
        Service service = new Service();
        Thread thread = new Thread(new Thread1(service));
        Thread thread2 = new Thread(new Thread2(service));
        thread.start();
        thread2.start();
    }
}


```

执行结果

![image-20211229094552358](https://gitee.com/jiruixin/images/raw/master/images/image-20211229094552358.png)

结论：当线程执行的代码出现异常时会自动释放锁。



#### 7.重写方法不使用synchronized

重写方法如果不使用synchronized关键字，即时非同步方法，使用后变成同步方法。

### 2. 同步方法和同步代码块

相同点：都是能让线程同步执行

不同点

- 同步方法默认锁的是该对象，如果是静态方法，默认锁的是类对象。
- 同步代码块可以指定任意要锁定的对象

- 同步代码块锁定的对象中不能是this关键字

### 3.synchronized代码块

> 一边异步。一边同步

只有同步代码块中的代码是同步执行，代码块外面的代码还是异步执行

> 多个锁就是异步

同步代码块中锁定的对象如果不是同一个对象那也是异步。

> 调用方法是随机的

代码块中的代码是同步执行，但是调用的方法是随机的。

> 同步静态方法对所有实例都起作用

```java
package Thread2;

public class Service {
    synchronized public static void method1(){
        try {
            System.out.println(Thread.currentThread().getName()+"进入方法1");
            Thread.sleep(3000);
            System.out.println(Thread.currentThread().getName()+"进入方法2");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    synchronized public static void method2(){
        try {
            System.out.println(Thread.currentThread().getName()+"进入方法1");
            Thread.sleep(3000);
            System.out.println(Thread.currentThread().getName()+"进入方法2");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }


}

////////////////////////////////////////////////////////////////
package Thread2;

import com.sun.xml.internal.ws.api.ServiceSharedFeatureMarker;

public class Thread1 implements Runnable{

    private Service service;

    public Thread1(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.method1();
    }
}

/////////////////////////////////////////////////////////////////
package Thread2;

import exception.Run;

public class Thread2 implements Runnable {

    private Service service;

    public Thread2(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.method2();
    }
}

///////////////////////////////////////////////////////////////////
package Thread2;

public class Main {
    public static void main(String[] args) {

        Service service = new Service();
        Service service1 = new Service();

        Thread thread = new Thread(new Thread1(service));
        Thread thread1 = new Thread(new Thread2(service1));

        thread.start();
        thread1.start();
    }
}

```

![image-20211229104602363](https://gitee.com/jiruixin/images/raw/master/images/image-20211229104602363.png)

> 同步代码块锁定class对象对该对象的所有实例起作用



> String常量池特性与同步相关的问题

```java
package string;

public class Service {

    public void method1(String s){
        try {
            synchronized (s){
                while(true){
                    System.out.println(Thread.currentThread().getName()+"进入方法1");
                    Thread.sleep(500);
                }
        }

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

////////////////////////////////////////////////////////////////
package string;

public class Thread1 implements Runnable{

    private Service service;

    public Thread1(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.method1("AA");
    }
}

/////////////////////////////////////////////////////////////////
package string;

public class Thread2 implements Runnable {

    private Service service;

    public Thread2(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.method1("AA");
    }
}

////////////////////////////////////////////////////////////////
package string;

public class Main {
    public static void main(String[] args) {

        Service service = new Service();

        Thread thread = new Thread(new Thread1(service));
        Thread thread1 = new Thread(new Thread2(service));

        thread.start();
        thread1.start();
    }
}
```

![image-20211229105528097](https://gitee.com/jiruixin/images/raw/master/images/image-20211229105528097.png)

分析：出现这个值是因为String两个值都是"AA",持有相同的锁，造成线程不能执行。大多数情况下不用String作为锁对象。



> 死锁

```java
package dead;

public class MyThread implements Runnable {

    private String name;

    private Object lock1 = new Object();
    private Object lock2 = new Object();

    public void setFlag(String name){
        this.name = name;
    }



    @Override
    public void run() {
        if (name.equals("a")){
            synchronized (lock1){
                try{
                    System.out.println("name="+name);
                    Thread.sleep(1000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                synchronized (lock2){
                    System.out.println("按lock1--->lock2代码顺序执行");
                }
            }
        }
        if (name.equals("b")){
            synchronized (lock2){
                try{
                    System.out.println("name="+name);
                    Thread.sleep(1000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                synchronized (lock1){
                    System.out.println("按lock2---->lock1");
                }
            }
        }
    }
}

///////////////////////////////////////////////////////////////////////////////
package dead;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        MyThread myThread = new MyThread();
        myThread.setFlag("a");

        Thread thread = new Thread(myThread);
        thread.start();
        Thread.sleep(100);

        myThread.setFlag("b");
        Thread thread1 = new Thread(myThread);
        thread1.start();
    }
}
```

![image-20211229112036749](https://gitee.com/jiruixin/images/raw/master/images/image-20211229112036749.png)

出现死锁。在程序中死锁线程必须避免。



> 锁对象改变导致异步执行

```java
package chagelock;

public class Service {

    private String lokc = "123";

    public String getLokc() {
        return lokc;
    }

    public void setLokc(String lokc) {
        this.lokc = lokc;
    }

    public void method1(){

        try {
            synchronized (lokc){
                System.out.println(Thread.currentThread().getName()+"进入方法"+lokc);
                this.lokc = "456";
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName()+"离开方法");
        }

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

///////////////////////////////////////////////////////////////////
package chagelock;

public class Thread1 implements Runnable{

    private Service service;

    public Thread1(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.method1();
    }
}

/////////////////////////////////////////////////////////////////////
package chagelock;

public class Thread2 implements Runnable {

    private Service service;

    public Thread2(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.method1();
    }
}

///////////////////////////////////////////////////////////////////
package chagelock;

public class Main {
    public static void main(String[] args) throws InterruptedException {
        Service service = new Service();
        Thread thread = new Thread(new Thread1(service));
        thread.setName("A");
        Thread thread1 = new Thread(new Thread2(service));
        thread1.setName("B");
        thread.start();
        Thread.sleep(50);
        thread1.start();
    }
}

```

![image-20211229121036377](https://gitee.com/jiruixin/images/raw/master/images/image-20211229121036377.png)

锁对象改变导致代码异步执行



> 锁对象的属性值改变不会影响锁

### 4.volatile关键字

特性

- 可见性   

线程更改的数据别的线程可以马上看见

- 原子性

32位系统中，long或者double没有实现写原子性

64位系统中，原子性取决于具体的实现，在X86架构64位JDK版本中，写double或者long是原子的

**在volatile关键字不支持原子性**

- 禁止代码重排序

> 多线程可能出现的死循环

```java
package whilelook;

public class MyThread implements Runnable {

    private boolean isRunning = true;

    public boolean isRunning() {
        return isRunning;
    }

    public void setRunning(boolean running) {
        isRunning = running;
    }

    @Override
    public void run() {
        System.out.println("进入run");
        while(isRunning == true){
			// 这里不要放同步代码块或者同步方法
        }
        System.out.println("线程被停止了");
    }
}

////////////////////////////////////////////////
package whilelook;

public class Main {
    public static void main(String[] args) {

        try {
            MyThread myThread = new MyThread();
            Thread thread = new Thread(myThread);
            thread.start();
            Thread.sleep(1000);
            myThread.setRunning(false);
            System.out.println("已经赋值完毕");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }


    }
}
```

![image-20211229145433940](https://gitee.com/jiruixin/images/raw/master/images/image-20211229145433940.png)

明明已经执行完赋值操作了，但是子线程还是没有结束？

分析：启动子线程时，变量isRunning的值时true，代码虽然更新了变量isRunning的值，但是更新的是公共堆栈中的值，而子线程一直获取的是私有线程的变量值。所以程序一直没结束。



jvm内存图

![image-20211229154101754](https://gitee.com/jiruixin/images/raw/master/images/image-20211229154101754.png)



解决办法：用volatile修饰变量

``` java
// 在上面的代码中给这个变量加上volatile关键字
volatile private boolean isRunning = true
```

执行结果

![image-20211229145947689](https://gitee.com/jiruixin/images/raw/master/images/image-20211229145947689.png)

volatile变量强制让线程去主内存中读取值。



> synchronized代码块可以增加可见性

在**线程可能出现死循环**的例子中更改循环的代码如下

```java
package whilelook;

public class MyThread implements Runnable {

    private boolean isRunning = true;

    public boolean isRunning() {
        return isRunning;
    }

    public void setRunning(boolean running) {
        isRunning = running;
    }

    @Override
    public void run() {
        System.out.println("进入run");
        while(isRunning == true){
             // 以前这里是空，现在加上这个语句 
			synchronized ("aa"){
            }
        }
        System.out.println("线程被停止了");
    }
}
```

执行结果

![image-20211229151321951](https://gitee.com/jiruixin/images/raw/master/images/image-20211229151321951.png)

分析：同步代码块或者同步方法都会增加可见性



> volatile的i++非原子操作

```java
package volatilenotatomic;

public class MyThread extends Thread {

    volatile private static int count;

    private static void addCount() {

        for (int i = 0; i <100 ; i++) {
            count++;
        }
        System.out.println("count="+count);
    }

    @Override
    public void run() {
        addCount();
    }
}

////////////////////////////////////////////////////////////
package volatilenotatomic;

public class Main {

    public static void main(String[] args) {
        MyThread[] myThreads = new MyThread[100];
        for (int i = 0; i < 100; i++) {
            myThreads[i] = new MyThread();

        }
        for (int i = 0; i < 100; i++) {
            myThreads[i].start();
        }
    }
}

```

执行结果：

![image-20211229161720604](https://gitee.com/jiruixin/images/raw/master/images/image-20211229161720604.png)

说明volatile的count++不是原子性的

解决

- 添加synchronized关键字

对count++操作添加synchronized代码块（这里不多讲了）

- 使用Atomic原子类实现原子性

```java
package volatilenotatomic;

import java.util.concurrent.atomic.AtomicInteger;

public class MyThread extends Thread {

    private AtomicInteger count =  new AtomicInteger(0);
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName()+"    "+count.incrementAndGet());
        }
    }
}

/////////////////////////////////////////////////////////
package volatilenotatomic;

public class Main {

    public static void main(String[] args) {
        MyThread myThread = new MyThread();

        Thread thread = new Thread(myThread);
        thread.start();

        Thread thread1 = new Thread(myThread);
        thread1.start();

        Thread thread2 = new Thread(myThread);
        thread2.start();

    }
}

```

执行结果

![image-20211229164222030](https://gitee.com/jiruixin/images/raw/master/images/image-20211229164222030.png)

分析：解决了操作的原子性，但是原子类的结果输出具有随机性。出现这个问题的原因是incrementAndGet方法是原子性的，但是方法之间的调用不是原子性的。解决这种办法必须用同步。



### 5.禁止代码排序

volaile关键字前面的可以排序，关键字后面的可以排序，但是不能跨关键字重新排序

### 6.synchronized和volatile关键字对比

相同点

- 可见性
- 禁止代码排序

不同点

- 原子性

  synchronized是原子性，但是volatile不是原子性

使用场景

当一个线程的值被修改时，希望别的县城可以看见使用volatile。

当多个线程对一个值进行修改操作时为了避免非线程安全可以使用synchronized



## 3.线程间通信

### 1.wait/notify

#### 1.wait/notify机制

- 拥有相同锁的线程才可以实现wait/notify机制
- wait/notify机制必须获取到对象锁，没有获取到锁会抛异常。并且只能在代码块中执行。
- 执行wat的线程接到通知后选择的线程根据是执行wait的顺序确定。

- wait方法会让执行这个语句的线程处于等待，并且释放锁，知道接到通知或者中断。

- 执行notify方法的后不会马上释放锁，呈wait线程也不能马上执行，需要等到nofity线程完毕才可以获取该对象锁。
- 接到通知的线程完毕后没有notify语句，其他线程还是会是wait

#### 2.实现wait/notify机制

```java
package wait;

import java.util.ArrayList;
import java.util.List;

public class MyList {
    private static List list = new ArrayList();

    public static void add(){
        list.add("anyString");
    }

    public static int size(){
        return list.size();
    }
}

/////////////////////////////////////////////////////////////
package wait;

public class MyThread1 implements Runnable {

    private Object lock;

    public MyThread1(Object lock){
        this.lock = lock;
    }

    @Override
    public void run() {
        try {
             synchronized (lock){
                 if (MyList.size() != 5){
                     System.out.println("wait begin"+System.currentTimeMillis());
                     lock.wait();
                     System.out.println("wait end"+System.currentTimeMillis());
                 }
             }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

///////////////////////////////////////////////////////////
package wait;

public class MyThread2 implements Runnable {

    private Object lock;

    public MyThread2(Object lock){
        this.lock = lock;
    }

    @Override
    public void run() {

        try{
            synchronized (lock){
                for (int i = 0; i < 10; i++) {
                    MyList.add();
                    if (MyList.size() == 5){
                        lock.notify();
                        System.out.println("已经发出通知");
                    }
                    System.out.println("添加了"+(i+1)+"个元素");
                    Thread.sleep(1000);
                }
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

////////////////////////////////////////////////////////////////
package wait;

public class Run  {
    public static void main(String[] args) {
        Object o = new Object();
        Thread thread = new Thread(new MyThread1(o));
        thread.start();

        try {
            Thread.sleep(50);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        Thread thread1 = new Thread(new MyThread2(o));
        thread1.start();
    }
}

```

![image-20211229205529653](https://gitee.com/jiruixin/images/raw/master/images/image-20211229205529653.png)

注意点

- wait()方法可以使线程暂定，并且立即释放锁
- notify方法执行完毕才会释放锁，并且只能唤醒一个线程。
- notifyAll()方法执行后，会按照wait方法相反的顺序依次唤醒全部线程





#### 3.线程状态切换图

![image-20211229212945611](https://gitee.com/jiruixin/images/raw/master/images/image-20211229212945611.png)



#### 4.wait状态下的interrpt()

```java
package waitinterrupt;

public class Service {
    public void testMethod(Object lock){
        try {
            synchronized (lock){
                System.out.println("begin wait");
                lock.wait();
                System.out.println("wait end");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

///////////////////////////////////////////////////
package waitinterrupt;

public class MyThread implements Runnable {

    private Object lock;

    public MyThread(Object lock){
        this.lock = lock;
    }

    @Override
    public void run() {
        Service service = new Service();
        service.testMethod(lock);
    }
}

///////////////////////////////////////////////////////////
package waitinterrupt;

public class Run {
    public static void main(String[] args) {

        Object o = new Object();
        Thread thread = new Thread(new MyThread(o));

        thread.start();

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        thread.interrupt();
    }
}

```

执行结果

![image-20211229215123376](https://gitee.com/jiruixin/images/raw/master/images/image-20211229215123376.png)

睡眠下的interrpt和wait状态下的interrpt是一样的（目前的理解2021/12/29）

#### 5.notifyAll()

notifyAll唤醒取决于JVM的实现，不是所有的JVM都会按照正序、倒序、随机唤醒的。

```java
package notifyall;

import java.util.concurrent.atomic.AtomicInteger;

public class MyThread extends Thread {

    private Object lock;

    public MyThread(Object lock){
        this.lock  = lock;
    }

    @Override
    public void run() {
        {
            synchronized (lock){
                System.out.println(Thread.currentThread().getName()+"begin wait");
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"end wait");
            }
        }
    }
}

////////////////////////////////////////////////////////////
package notifyall;

public class MyThread2 implements Runnable {

    private Object lock;

    public MyThread2(Object lock){
        this.lock = lock;
    }

    @Override
    public void run() {
        synchronized (lock){
            lock.notifyAll();
        }
    }
}

/////////////////////////////////////////////////////////////
package notifyall;

public class Main {

    public static void main(String[] args) throws InterruptedException {

        Object o = new Object();
        for (int i = 0; i < 5 ; i++) {
            Thread thread = new Thread(new MyThread(o));
            thread.start();
        }
        Thread.sleep(2000);

        Thread thread = new Thread(new MyThread2(o));
        thread.start();

    }
}
```

执行结果

![image-20211229221051384](https://gitee.com/jiruixin/images/raw/master/images/image-20211229221051384.png)



#### 6.wait(long)

- 设置等待的时间，到时没有通知会自动唤醒
- 执行下面的代码需要重新获取锁

**不要让通知过早**

#### 7.生产者/消费者

通过wait/notify机制完成



#### 8.管道线程通信-字节流

#### 9.管道线程通信-字符流

### 2.join

#### 10 join()

不是静态方法，当前线程会等待调用join方法的线程执行完毕再往下执行。join方法内部通过wait实现

```java
package join;

public class Mythread implements Runnable{
    @Override
    public void run() {
        try{

            Thread.sleep(1000);
            System.out.println("join执行完毕");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

////////////////////////////////////////////////////
package join;

public class Run {
    public static void main(String[] args) throws InterruptedException {
        Mythread mythread = new Mythread();
        Thread thread = new Thread(mythread);
        thread.start();
        thread.join();
        System.out.println("等join执行完毕再执行");
    }
}

```

执行结果

![image-20211230102817365](https://gitee.com/jiruixin/images/raw/master/images/image-20211230102817365.png)

#### 11 join遇见interrupt

join内部是wait实现，所以和wait遇见interrupt一样

#### 12 join(long)

- 设置等待的时间，到时没有通知会自动唤醒
- 执行下面的代码需要重新获取锁

#### 13 join(long)、wait(long)、sleep(long)区别

 join()内部是wait()实现，所以他俩很想

 join(long)、wait(long)会释放锁，而sleep不会释放锁。



#### 14 join方法提前运行-出现意外

### 3.ThreadLocal

存储线程的私有变量。他是当前线程对象的和当前线程对象属性Map的桥梁

流程：数据---->ThreadLocal---->currentThread()----->Map



#### 1.隔离性

```java
package threadlocal;

public class Tools {
    public static ThreadLocal t1 = new ThreadLocal();
}

///////////////////////////////////////////////////
package threadlocal;

public class MyThread implements Runnable {

    @Override
    public void run() {
        Tools.t1.set("子线程");
        System.out.println(Tools.t1.get());
    }
}

//////////////////////////////////////////////////
package threadlocal;

public class Main {
    public  ThreadLocal t1 = new ThreadLocal();

    public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(new MyThread());
        thread.start();
        thread.join();
        System.out.println(Tools.t1.get());

    }
}

```

![image-20211230145841951](https://gitee.com/jiruixin/images/raw/master/images/image-20211230145841951.png)



#### 2.重写initialValue()

```java
package threadlocal2;

public class Tools {
    public static MyThreadLocal t1 = new MyThreadLocal();
}

//////////////////////////////////////////////////////////
package threadlocal2;

public class MyThreadLocal extends ThreadLocal {
    @Override
    protected Object initialValue() {
        return "初始值";
    }
}

////////////////////////////////////////////////
package threadlocal2;

public class Main {
    public  ThreadLocal t1 = new ThreadLocal();

    public static void main(String[] args) throws InterruptedException {
        System.out.println("主线程："+ Tools.t1.get());
    }
}


```

![image-20211230150526058](https://gitee.com/jiruixin/images/raw/master/images/image-20211230150526058.png)



### 4. InheritableThreadLocal

使用类InheritableThreadLocal可使子线程继承父线程的值。

类ThreadLocal不能实现值继承

> 子线程继承父线程的值

```java
package threadlocal3;

public class Tools {
    public static InheritableThreadLocal t1 = new InheritableThreadLocal();
}

////////////////////////////////////////////////////////////
package threadlocal3;

public class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println("子线程: "+ Tools.t1.get());
    }
}

///////////////////////////////////////////////////////////////
package threadlocal3;

public class Main {
    public  ThreadLocal t1 = new ThreadLocal();

    public static void main(String[] args) throws InterruptedException {
        Tools.t1.set("主线程设置的");
        Thread thread = new Thread(new MyThread());
        thread.start();
    }
}

```

执行结果

![image-20211230151839938](https://gitee.com/jiruixin/images/raw/master/images/image-20211230151839938.png)



> 子线程值更新，父线程还是旧值

```java
package threadlocal3;

public class Tools {
    public static InheritableThreadLocal t1 = new InheritableThreadLocal();
}

//////////////////////////////////////////////////////////////
package threadlocal3;

public class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println("子线程: "+ Tools.t1.get());
        Tools.t1.set("子线程设置的新值");
        System.out.println("子线程: "+ Tools.t1.get());
    }
}

//////////////////////////////////////////////////////////////
package threadlocal3;

public class Main {
    public  ThreadLocal t1 = new ThreadLocal();

    public static void main(String[] args) throws InterruptedException {
        Tools.t1.set("主线程设置的");
        Thread thread = new Thread(new MyThread());
        thread.start();
        thread.join();
        System.out.println("主线程: "+ Tools.t1.get());

    }
}

```

执行结果

![image-20211230153214025](https://gitee.com/jiruixin/images/raw/master/images/image-20211230153214025.png)



>父线程有新值，子线仍是旧值

```java
package threadlocal3;

public class Tools {
    public static InheritableThreadLocal t1 = new InheritableThreadLocal();
}

////////////////////////////////////////////
package threadlocal3;

public class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println("子线程: "+ Tools.t1.get());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("子线程: "+ Tools.t1.get());
    }
}

///////////////////////////////////////
package threadlocal3;

public class Main {
    public  ThreadLocal t1 = new ThreadLocal();

    public static void main(String[] args) throws InterruptedException {
        Tools.t1.set("主线程设置的");
        Thread thread = new Thread(new MyThread());
        thread.start();
        Thread.sleep(1000);
        Tools.t1.set("主线程设置的新值");
        System.out.println("主线程: "+ Tools.t1.get());

    }
}

```

执行结果

![image-20211230153730453](https://gitee.com/jiruixin/images/raw/master/images/image-20211230153730453.png)



> 子线程可以感知对象属性的变化

```java
package threadlocal4;

public class User {

    private String name;

    public User(String name){
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}

/////////////////////////////////////////////////
package threadlocal4;

public class Tools {
    public static InheritableThreadLocal t1 = new InheritableThreadLocal();
}

////////////////////////////////////////////////
package threadlocal4;

public class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println("子线程: "+ Tools.t1.get());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("子线程: "+ Tools.t1.get());
    }
}

/////////////////////////////////////////////
package threadlocal4;

public class Main {
    public  ThreadLocal t1 = new ThreadLocal();

    public static void main(String[] args) throws InterruptedException {
        User user = new User("主线程设置");
        Tools.t1.set(user);
        Thread thread = new Thread(new MyThread());
        thread.start();
        Thread.sleep(1000);
        user.setName("主线程设置的新值");
        Tools.t1.set(user);
        System.out.println("主线程: "+ Tools.t1.get());

    }
}

```

执行结果

![image-20211230155434739](https://gitee.com/jiruixin/images/raw/master/images/image-20211230155434739.png)



> 重写childValue()

- childValue方法只在创建子线程的时候执行一次

```java
package threadlocal4;

public class MyInheritable extends InheritableThreadLocal {
    
    @Override
    protected Object childValue(Object parentValue) {
        return "子线程加的  "+parentValue;
    }
}

//////////////////////////////////////////////////////////////
package threadlocal4;

public class Tools {
    public static MyInheritable t1 = new MyInheritable();
}

////////////////////////////////////////////////////////////////
package threadlocal4;

public class User {

    private String name;

    public User(String name){
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}

/////////////////////////////////////////////////
package threadlocal4;

public class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println("子线程: "+ Tools.t1.get());
    }
}

/////////////////////////////////////////////////
package threadlocal4;

public class Main {
    public  ThreadLocal t1 = new ThreadLocal();

    public static void main(String[] args) throws InterruptedException {
        User user = new User("主线程设置");
        Tools.t1.set(user);
        Thread thread = new Thread(new MyThread());
        thread.start();
    }
}
```

执行结果

![image-20211230160631643](https://gitee.com/jiruixin/images/raw/master/images/image-20211230160631643.png)



## 4.Lock的对象使用

### 1.ReentrantLock

```java
package lockobject1;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    private Lock lock = new ReentrantLock();

    public void MethodA(){
        lock.lock();
        for (int i = 0; i < 3; i++) {
            System.out.println(Thread.currentThread().getName()+"  "+i);
        }
        lock.unlock();
    }
}

/////////////////////////////////////////////////
package lockobject1;

public class MyThread implements Runnable{

    private Service service;

    public MyThread(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.MethodA();
    }
}

/////////////////////////////////////////////////////
package lockobject1;

public class Main {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(new Service());
        Thread thread = new Thread(myThread);

        Thread thread1 = new Thread(myThread);
        Thread thread2 = new Thread(myThread);

        thread.start();
        thread1.start();
        thread2.start();
    }
}

```

执行结果

![image-20211230163253978](https://gitee.com/jiruixin/images/raw/master/images/image-20211230163253978.png)

#### 2.Condition

控制并处理线程状态，可以让线程是wait状态，也可以让线程继续运行。

```java
package lockobject2;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    private Lock lock = new ReentrantLock();

    private Condition condition = lock.newCondition();

    public void MethodA(){
        lock.lock();
        try {
            System.out.println("执行await");
            condition.await();
            System.out.println("完成执行await");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        lock.unlock();
    }

    public void MethodB(){
        lock.lock();
        System.out.println("执行signal");
        condition.signal();
        System.out.println("执行signal结束");
        lock.unlock();
    }
}

///////////////////////////////////////////////////////
package lockobject2;

public class MyThread implements Runnable{

    private Service service;

    public MyThread(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.MethodA();
    }
}

///////////////////////////////////////////////////////
package lockobject2;

public class MyThread2 implements  Runnable {

    private Service service;

    public MyThread2(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.MethodB();
    }

}

/////////////////////////////////////////////////////
package lockobject2;

public class Main {
    public static void main(String[] args) throws InterruptedException {
        Service service = new Service();
        MyThread myThread = new MyThread(service);
        Thread thread = new Thread(myThread);
        Thread thread1 = new Thread(new MyThread2(service));

        thread.start();
        Thread.sleep(1000);
        thread1.start();

    }
}

```

![image-20211230214416969](https://gitee.com/jiruixin/images/raw/master/images/image-20211230214416969.png)

结论

- wait方法相当于Condition的await
- notify方法相当于Condition的signal
- notifyAll方法相当于Condition的signalAll

注意点

- **Condition对象可以创建多个，但是同一个Condition对象的signal方法可以唤醒它的await。**

### 生产者/消费者

```java

package lockobject3;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    private Lock lock = new ReentrantLock();

    private Condition condition = lock.newCondition();

    private List list = new ArrayList();

    public void pro(){
        while(true){
            lock.lock();
            if (list.size()==0){
                list.add(new Object());
                System.out.println("生产*************");
            }else{

                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            lock.unlock();
        }

    }

    public void con(){
        while(true){
            lock.lock();
            if (list.size()>0){
                list.remove(0);
                System.out.println("消费-------------");
            }else{
                condition.signal();
            }
            lock.unlock();
        }


    }
}

///////////////////////////////////////////////////////////////
package lockobject3;

import java.util.List;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

///////////////////////////////////////////////////////////////
package lockobject3;

public class Consumer implements Runnable{

    private Service service;

    public Consumer(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.con();
    }
}

/////////////////////////////////////////////////////////////////////
package lockobject3;

public class Main {
    public static void main(String[] args) {
        Service service = new Service();

        Thread thread = new Thread(new Produce(service));
        Thread thread1 = new Thread(new Consumer(service));

        thread.start();
        thread1.start();
    }
}

```

执行结果

![image-20211231093118647](https://gitee.com/jiruixin/images/raw/master/images/image-20211231093118647.png)



#### 公平锁和非公平锁

ReentrantLock默认是非公平锁。

公平锁采用先到先得的策略，每次获取锁之前都会检查队列里有没有排队等待锁的

非公平锁：采用“有机会插队”策略。

### 3.ReentrantLock的方法

> public int getHoldCount()

获得当前线程保持锁的个数，就是调用lock的次数。遇见unlock会减1。

```java
package lockobject4;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    private ReentrantLock lock = new ReentrantLock(true);
    public void pro(){

        lock.lock();
        lock.lock();
        lock.lock();
        lock.lock();
        System.out.println("当前锁的个数"+lock.getHoldCount());
        lock.unlock();
        System.out.println("执行一次unlock方法还剩的锁"+lock.getHoldCount());
    }
}

///////////////////////////////////////////////////////
package lockobject4;

public class Produce implements Runnable{

    private Service service;
    
    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

//////////////////////////////////////////////////////
package lockobject4;

import java.lang.reflect.Array;
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();
        Thread thread = new Thread(new Produce(service));
        thread.start();
    }
}

```

![image-20211231102018765](https://gitee.com/jiruixin/images/raw/master/images/image-20211231102018765.png)



> public final int getQueueLength()

返回等待获此锁的线程估计数

```java
package lockobject4;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    public ReentrantLock lock = new ReentrantLock(true);

    public void pro(){

        lock.lock();
        System.out.println(Thread.currentThread().getName()+"进入");
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //lock.unlock();
    }
}

///////////////////////////////////////////////////////
package lockobject4;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

////////////////////////////////////////////////////////
package lockobject4;

import java.lang.reflect.Array;
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();

        Thread[] threads = new Thread[5];

        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(new Produce(service));
        }

        for (int i = 0; i < 5; i++) {
            threads[i].start();
        }
        Thread thread = new Thread(new Produce(service));
        thread.start();
        Thread.sleep(1000);
        System.out.println("等待的线程"+service.lock.getQueueLength());
    }
}

```

![image-20211231102657490](https://gitee.com/jiruixin/images/raw/master/images/image-20211231102657490.png)



>public int getWaitQueueLength(Condition condition)

返回与此锁相关的给定条件的Condition的线程估计数

必须获取该锁的lock才能执行这个方法

```java
package lockobject4;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    public ReentrantLock lock = new ReentrantLock(true);

    public Condition condition = lock.newCondition();

    public void pro(){

        lock.lock();
        System.out.println(Thread.currentThread().getName()+"进入");
        try {
            System.out.println("处于等待状态的线程有"+lock.getWaitQueueLength(condition));
            condition.await();
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        lock.unlock();
    }

}

//////////////////////////////////////////////////////
package lockobject4;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

////////////////////////////////////////////////////
package lockobject4;

import java.lang.reflect.Array;
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();

        Thread[] threads = new Thread[5];

        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(new Produce(service));
        }

        for (int i = 0; i < 5; i++) {
            threads[i].start();
        }
        Thread thread = new Thread(new Produce(service));
        thread.start();
    }
}

```

![image-20211231103755390](https://gitee.com/jiruixin/images/raw/master/images/image-20211231103755390.png)



> public final boolean hasQueuedThread(Thread thread) 

查询指定线程是否在等待获取此锁

```java
package lockobject4;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    public ReentrantLock lock = new ReentrantLock(true);

    public Condition condition = lock.newCondition();

    private List list = new ArrayList();

    public void pro(){

        lock.lock();
        System.out.println(Thread.currentThread().getName()+"进入");
        try {

            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //lock.unlock();
    }
}

//////////////////////////////////////////////////////////////
package lockobject4;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

///////////////////////////////////////////////////////////////
package lockobject4;

        import java.lang.reflect.Array;
        import java.util.ArrayList;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();

        Thread[] threads = new Thread[5];

        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(new Produce(service));
        }
        for (int i = 0; i < 5; i++) {
            threads[i].start();
        }
        Thread.sleep(1000);
        System.out.println("线程4是否在等待此锁："+service.lock.hasQueuedThread(threads[4]));
    }
}
```

![image-20211231104329668](https://gitee.com/jiruixin/images/raw/master/images/image-20211231104329668.png)



> public final boolean hasQueuedThreads()

```java
package lockobject4;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    public ReentrantLock lock = new ReentrantLock(true);

    public Condition condition = lock.newCondition();

    private List list = new ArrayList();

    public void pro(){

        lock.lock();
        System.out.println(Thread.currentThread().getName()+"进入");
        try {

            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //lock.unlock();
    }
}

//////////////////////////////////////////////////////////////
package lockobject4;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

///////////////////////////////////////////////////////////////
package lockobject4;

        import java.lang.reflect.Array;
        import java.util.ArrayList;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();

        Thread[] threads = new Thread[5];

        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(new Produce(service));
        }
        for (int i = 0; i < 5; i++) {
            threads[i].start();
        }
        Thread.sleep(1000);
        System.out.println("是否有线程在等待此锁："+service.lock.hasQueuedThreads());
    }
}
```

![image-20211231104527962](https://gitee.com/jiruixin/images/raw/master/images/image-20211231104527962.png)



> public boolean hasWaiters

查询线程是否执行了condition的await()方法

```java
package lockobject4;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    public ReentrantLock lock = new ReentrantLock(true);

    public Condition condition = lock.newCondition();

    private List list = new ArrayList();

    public void pro(){

        lock.lock();
        System.out.println(Thread.currentThread().getName()+"进入");
        try {
            System.out.println("是否有线程执行了await:"+lock.hasWaiters(condition));
            condition.await();
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        lock.unlock();
    }
}

/////////////////////////////////////////////////////////////////
package lockobject4;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

///////////////////////////////////////////////////////////////////
package lockobject4;

        import java.lang.reflect.Array;
        import java.util.ArrayList;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();

        Thread[] threads = new Thread[5];

        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(new Produce(service));
        }

        for (int i = 0; i < 5; i++) {
            threads[i].start();
        }
        Thread.sleep(1000);
    }
}

```

![image-20211231105322017](https://gitee.com/jiruixin/images/raw/master/images/image-20211231105322017.png)



> public final boolean isFair()

判断锁是否为公平锁

```java
 ReentrantLock reentrantLock = new ReentrantLock(false);
 System.out.println("是否为公平锁:"+reentrantLock.isFair());
```

![image-20211231105657313](https://gitee.com/jiruixin/images/raw/master/images/image-20211231105657313.png)



> public boolean isHeldByCurrentThread()

```java
package lockobject4;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    public ReentrantLock lock = new ReentrantLock(true);

    public Condition condition = lock.newCondition();

    private List list = new ArrayList();

    public void pro(){

        System.out.println(lock.isHeldByCurrentThread());
        lock.lock();
        System.out.println(lock.isHeldByCurrentThread());
    
        lock.unlock();
    }
}

//////////////////////////////////////////////////////////
package lockobject4;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

/////////////////////////////////////////////////////////
package lockobject4;

        import java.lang.reflect.Array;
        import java.util.ArrayList;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();

        Thread[] threads = new Thread[5];

        for (int i = 0; i < 1; i++) {
            threads[i] = new Thread(new Produce(service));
        }

        for (int i = 0; i < 1; i++) {
            threads[i].start();
        }
    }
}

```

![image-20211231110408897](https://gitee.com/jiruixin/images/raw/master/images/image-20211231110408897.png)



> public boolean isLocked()

判断线程是否由线程保持，并没有释放



> public void lockInterruptibly() throws InterruptedException

某个线程尝试获取锁并且阻塞在lockInterruptibly()可以被中断

```java
package lockobject4;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    public ReentrantLock lock = new ReentrantLock(true);

    public Condition condition = lock.newCondition();

    public void pro(){
        try {
            System.out.println(Thread.currentThread().getName()+"进来前");
            lock.lockInterruptibly();
            System.out.println(Thread.currentThread().getName()+"进来后");
        } catch (Exception e) {
            e.printStackTrace();
        }
        //lock.unlock();
    }
}

////////////////////////////////////////////////
package lockobject4;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

/////////////////////////////////////////////////
package lockobject4;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();

        Thread thread = new Thread(new Produce(service));
        Thread thread2 = new Thread(new Produce(service));

        thread.start();
        Thread.sleep(500);

        thread2.start();
        Thread.sleep(500);

        thread2.interrupt();

    }
}

```

![image-20211231112949737](https://gitee.com/jiruixin/images/raw/master/images/image-20211231112949737.png)



> public void tryLock()

尝试能否获取该锁，如果不能继续往下执行其他的代码

> public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException    

如果线程在指定的时间能获取锁则返回true；否则返回false

>  boolean await(long time, TimeUnit unit) throws InterruptedException

具有自动唤醒的功能

> boolean awaitUntil(Date deadline) throws InterruptedException;

在指定的时间内结束等待

> void awaitUninterruptibly()

线程在等待的过程不允许被中断

```java
package lockobject4;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Service {

    public ReentrantLock lock = new ReentrantLock(true);

    public Condition condition = lock.newCondition();

    public void pro(){
        try {
            lock.lock();
            condition.await();
        } catch (Exception e) {
            e.printStackTrace();
        }
        lock.unlock();
    }
}

/////////////////////////////////////////////////////////
package lockobject4;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

////////////////////////////////////////////////////////
package lockobject4;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();
        Thread thread = new Thread(new Produce(service));
        thread.start();
        Thread.sleep(500);
        thread.interrupt();
    }
}

```

![image-20211231114426009](https://gitee.com/jiruixin/images/raw/master/images/image-20211231114426009.png)



分析：正常的线程执行await()方法可以被中断

把上面的代码  condition.await()   换成  condition.awaitUninterruptibly();

执行结果

![image-20211231114602486](https://gitee.com/jiruixin/images/raw/master/images/image-20211231114602486.png)

没有被中断



### 4.ReentrantReadWriteLock

ReentrantLock类的所有操作都是同步的。效率非常低

#### 1.读读共享

```java
package lockobject5;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class Service {

    public ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public void pro(){
        try {
            lock.readLock().lock();
            System.out.println(Thread.currentThread().getName()+"进来了");
        } catch (Exception e) {
            e.printStackTrace();
        }
        //lock.readLock().unlock();
    }
}

///////////////////////////////////////////////////////////
package lockobject5;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.pro();
    }
}

///////////////////////////////////////////////////////////
package lockobject5;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();

        Thread thread = new Thread(new Produce(service));
        Thread thread2 = new Thread(new Produce(service));
        thread.start();
        thread2.start();
    }
}

```

![image-20211231115431182](https://gitee.com/jiruixin/images/raw/master/images/image-20211231115431182.png)

结论：读锁是共享的

#### 2.读写互斥

```java
package lockobject5;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class Service {

    public ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public void read(){
        try {
            lock.readLock().lock();
            System.out.println("读锁进来了");
        } catch (Exception e) {
            e.printStackTrace();
        }
        //lock.readLock().unlock();
    }

    public void write(){
        try {
            lock.writeLock().lock();
            System.out.println("写锁进来了");
        } catch (Exception e) {
            e.printStackTrace();
        }
       //lock.readLock().unlock();
    }
}

////////////////////////////////////////////////////////////////////
package lockobject5;

public class Produce implements Runnable{

    private Service service;

    public Produce(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.read();
    }
}

///////////////////////////////////////////////////////////////////////////
package lockobject5;

public class Produce2 implements Runnable{

    private Service service;

    public Produce2(Service service){
        this.service = service;
    }

    @Override
    public void run() {
        service.write();
    }
}

////////////////////////////////////////////////////////////////
package lockobject5;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        Service service = new Service();

        Thread thread = new Thread(new Produce(service));
        Thread thread2 = new Thread(new Produce2(service));
        thread.start();
        thread2.start();
    }
}

```

![image-20211231115946796](https://gitee.com/jiruixin/images/raw/master/images/image-20211231115946796.png)

分析:当有读锁和写锁时，一个锁被获取，另外一个锁只能等待

### 4.结论

只有读读锁是异步的，其他的都是同步

## 5.定时器Timer



### 1.Timer的使用

#### 1.public void schedule(TimerTask task, Date time)

> 执行的时间晚于当前时间在未来执行

```java
package timer01;

import java.util.TimerTask;

public class MyTask extends TimerTask {

    @Override
    public void run() {
        System.out.println("任务执行了");
    }
}

////////////////////////////////////////
package timer01;

import java.util.Date;
import java.util.Timer;

public class Main {
    public static void main(String[] args) throws InterruptedException {
        MyTask myTask = new MyTask();
        Timer timer = new Timer();
        timer.schedule(myTask,new Date(System.currentTimeMillis()-1000));
     
    }
}
```

![image-20211231134243107](https://gitee.com/jiruixin/images/raw/master/images/image-20211231134243107.png)

分析:任务完成后没有程序没结束。

结果:她会启动任务线程，并且内部用while（true）一直执行计划。不会结束。这是程序不结束的原因

> cancel()

终止计数器，丢弃所有当前安排的任务

> 任务时间早于当前时间会立即执行



> 多个任务延迟

任务逐一按顺序执行，所以执行时间有可能和预期的不一样。

```java
package timer01;

import java.util.TimerTask;

public class MyTask extends TimerTask {

    @Override
    public void run() {
        try {
            Thread.sleep(1000);
            System.out.println("执行完任务的时间："+System.currentTimeMillis());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

///////////////////////////////////////////////////////////////////////////////
package timer01;

import java.util.Date;
import java.util.Timer;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        MyTask myTask = new MyTask();
        MyTask myTask2 = new MyTask();

        long l1 = System.currentTimeMillis()+1000;
        long l2 = System.currentTimeMillis()+3000;
        System.out.println("l1jihua:"+l1);
        System.out.println("l2jihua:"+l2);

        Timer timer = new Timer();
        timer.schedule(myTask,new Date(l1));
        timer.schedule(myTask2,new Date(l2));
   
    }
}

```

![image-20211231140036511](https://gitee.com/jiruixin/images/raw/master/images/image-20211231140036511.png)



因为任务1中执行了睡眠一秒，所以任务2比预计的晚了1秒左右



#### 2. public void schedule(TimerTask task, Date firstTime, long period)

> 计划晚于当前时间在未来执行

在指定的时间之后按照固定的间隔循环

```java
package timer01;

import java.util.TimerTask;

public class MyTask extends TimerTask {
    @Override
    public void run() {
        System.out.println("任务执行了");
    }
}

////////////////////////////////////////////////////
package timer01;

import java.util.Date;
import java.util.Timer;

public class Main {
    public static void main(String[] args) throws InterruptedException {
        MyTask myTask = new MyTask();

        long l1 = System.currentTimeMillis()+1000;
        Timer timer = new Timer();
        timer.schedule(myTask,new Date(l1),1000);
    }
}

```

![image-20211231140520681](https://gitee.com/jiruixin/images/raw/master/images/image-20211231140520681.png)



> 计划时间早于当前时间立即执行



> TimerTask的cancel()

TimerTask的cancel()方法会将自身从任务队列中消除，Timer的cancel是取消全部任务



> 间隔执行Task的算法

执行完一遍任务，把最后一个任务放在队列头再次执行

```java
package timer01;

import java.util.TimerTask;

public class MyTask extends TimerTask {

    @Override
    public void run() {
        System.out.println("任务1执行了");
    }
}

/////////////////////////////////////////////////
package timer01;

import java.util.TimerTask;

public class MyTask2 extends TimerTask {

    @Override
    public void run() {
        System.out.println("任务2执行了");
    }
}

///////////////////////////////////////////////////////
package timer01;

import java.util.TimerTask;

public class MyTask3 extends TimerTask {

    @Override
    public void run() {
        System.out.println("任务3执行了");
    }
}

//////////////////////////////////////////////////////////
package timer01;

import java.util.Date;
import java.util.Timer;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        MyTask myTask = new MyTask();
        MyTask2 myTask2 = new MyTask2();
        MyTask3 myTask3 = new MyTask3();

        long l1 = System.currentTimeMillis()+1000;

        Timer timer = new Timer();
        timer.schedule(myTask,new Date(l1),1000);
        timer.schedule(myTask2,new Date(l1),1000);
        timer.schedule(myTask3,new Date(l1),1000);
    }
}

```

![image-20211231142712396](https://gitee.com/jiruixin/images/raw/master/images/image-20211231142712396.png)



> Timer类中cancel有时会失效

Timer类的cancel没有抢到queue锁



#### 3.public void schedule(TimerTask task, long delay)

在延迟的基础上执行一次任务

#### 4.public void schedule(TimerTask task, long delay, long period)

在延迟的时间上循环执行任务



#### 5.schedule和scheduleAtFixedRate对比

相同点

- 在没有延迟的情况下（没有延时指执行任务的时间超过设置的间隔时间）

执行下次循环的时间为   上次执行任务的时间+设置的间隔时间

- 在有延迟的情况下

执行下次循环的时间为   上次任务时间结束后开始本次的任务



不同点

scheduleAtFixedRate具有追赶性、而schedule没有

```java
package timer01;

import java.util.TimerTask;

public class MyTask extends TimerTask {

    @Override
    public void run() {
        System.out.println("任务执行了："+System.currentTimeMillis());
    }
}

///////////////////////////////////////////////////////////
package timer01;

import java.util.Date;
import java.util.Timer;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        MyTask myTask = new MyTask();

        long l1 = System.currentTimeMillis()-20000;
        System.out.println("当前时间"+System.currentTimeMillis());
        System.out.println("计划时间"+l1);
        Timer timer = new Timer();
        timer.scheduleAtFixedRate(myTask,new Date(l1),1000);
    }
}

```

![image-20211231145327442](https://gitee.com/jiruixin/images/raw/master/images/image-20211231145327442.png)



把 上面代码 timer.scheduleAtFixedRate(myTask,new Date(l1),1000)  换成 timer.schedule(myTask,new Date(l1),1000)

执行结果如下

![image-20211231145455344](https://gitee.com/jiruixin/images/raw/master/images/image-20211231145455344.png)

scheduleAtFixedRate会去弥补之前的定时任务功能。

## 6.单例模式与多线程

### 1.延迟加载

立即加载/饿汉模式

```java
package oneobject;

public class MyObject {
    private static MyObject myObject = new MyObject();

    private MyObject(){

    }
    
    private static String name;
    private static String pw;

    // 这个方法没有加锁，可能导致非线程安全问题
    public static MyObject getInstance(){
        name = "从不同的服务器取值";
        pw = "从不同的服务器取值";
        return myObject;
    }

}

```

### 2.延迟加载

延迟加载/懒汉模式

```java
package oneobject;

public class MyObject {
    private static MyObject myObject;

    private MyObject(){

    }

    
    public static MyObject getInstance(){
       if (myObject != null){
           
       }else{
           try {
               // 在这里可能出现非线程安全问题
               Thread.sleep(1000);
               myObject = new MyObject();
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }
    }
}

```

synchronized和同步代码块都会使代码效率特别低下。

### 3. DCL机制解决单例



```java
package oneobject;

public class MyObject {
    // 让线程都可见这个变量，并且禁止排序
    private volatile static MyObject myObject;

    private MyObject(){

    }

    public static MyObject getInstance(){
       try{
           if (myObject !=null){
           }else{
               Thread.sleep(1000);
               synchronized (MyObject.class){
                   if (myObject == null){
                       myObject = new MyObject();
                   }
               }
           }
       }catch (InterruptedException e){
           e.printStackTrace();
       }
       return myObject;
    }

}

///////////////////////////////////////////////
package oneobject;

public class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}

////////////////////////////////////////////////
package oneobject;

public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(new MyThread());
        Thread thread2 = new Thread(new MyThread());
        Thread thread3 = new Thread(new MyThread());

        thread.start();
        thread2.start();
        thread3.start();
    }
}

```

![image-20220101100703192](https://gitee.com/jiruixin/images/raw/master/images/image-20220101100703192.png)



### 4.静态内置类实现单例

```java
package oneobject2;

public class MyObject {
    private volatile static MyObject myObject;

    private static class MyObjectHandler{
        private static MyObject myObject = new MyObject();
    }

    private MyObject(){

    }

    public static MyObject getInstance(){
       return MyObjectHandler.myObject;
    }
}

///////////////////////////////////////////////
package oneobject;

public class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}

////////////////////////////////////////////////
package oneobject;

public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(new MyThread());
        Thread thread2 = new Thread(new MyThread());
        Thread thread3 = new Thread(new MyThread());

        thread.start();
        thread2.start();
        thread3.start();
    }
}

```

![image-20220101110647612](https://gitee.com/jiruixin/images/raw/master/images/image-20220101110647612.png)



### 5.代码块实现单例

```java
package oneobject3;

import java.io.ObjectStreamException;
import java.io.Serializable;

public class MyObject implements Serializable {

    private volatile static MyObject myObject;

    static {
        myObject = new MyObject();
    }

    public static MyObject getInstance(){
       return myObject;
    }
}

////////////////////////////////////////////////////////////
package oneobject3;

public class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println(MyObject.getInstance().hashCode());
    }
}

//////////////////////////////////////////////////////////////
package oneobject3;

public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(new MyThread());
        Thread thread2 = new Thread(new MyThread());
        Thread thread3 = new Thread(new MyThread());

        thread.start();
        thread2.start();
        thread3.start();
    }
}

```

![image-20220101113040166](https://gitee.com/jiruixin/images/raw/master/images/image-20220101113040166.png)

### 6.序列化

```java
package oneobject3;

import java.io.ObjectStreamException;
import java.io.Serializable;

public class MyObject implements Serializable {


    private volatile static MyObject myObject = new MyObject();

    private MyObject(){

    }

    public static MyObject getInstance(){
       return myObject;
    }
    // 这段代码会让反序列化的对象和序列化对象是一样的
    // protected Object readResolve() throws ObjectStreamException{
    //     return MyObject.myObject;
    // }
}

//////////////////////////////////////////////////////////////////
package oneobject3;

import java.io.*;

public class Main {
    public static void main(String[] args) {
       try{
           MyObject myObject = MyObject.getInstance();
           System.out.println("序列化myobject="+myObject.hashCode());
           FileOutputStream outputStream = new FileOutputStream(new File("myobject-file.txt"));
           ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
           objectOutputStream.writeObject(myObject);
           objectOutputStream.close();
           outputStream.close();
       }catch (FileNotFoundException e){
           e.printStackTrace();
       } catch (IOException e) {
           e.printStackTrace();
       }

       try{
           FileInputStream fileInputStream = new FileInputStream(new File("myobject-file.txt"));
           ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);
           MyObject o = (MyObject)objectInputStream.readObject();
           objectInputStream.close();
           fileInputStream.close();
           System.out.println("反序列化myobject="+o.hashCode());
       } catch (FileNotFoundException e) {
           e.printStackTrace();
       } catch (IOException e) {
           e.printStackTrace();
       } catch (ClassNotFoundException e) {
           e.printStackTrace();
       }

        System.out.println();
    }
}

```

![image-20220101154350926](https://gitee.com/jiruixin/images/raw/master/images/image-20220101154350926.png)

把上面注释的代码打开

执行结果如下

![image-20220101154549964](https://gitee.com/jiruixin/images/raw/master/images/image-20220101154549964.png)

注意：序列化和反序列化的操作如果放在两个class中执行会产生新的对象。因为在class中执行相当于创建了两个jvm虚拟机。如果保证序列化对象是单例的必须在一个jvm中操作。

### 7.枚举类型实现单例



## 7.拾遗增补

### 1.线程状态

- NEW                       线程对象实例化，但是没有start
- RUNNABLE             线程正在执行
- TERMINATED          线程销毁
- TIMED_WAITING     线程睡眠是这种状态
- BLOCKED               线程在等待锁的时候是这种状态
- WAITING                 线程执行了wait后所处的状态

### 2.线程组

- 线程组中可以有线程对象
- 可以有线程组
- 组中也可以有线程

#### 1.一级关联

父对象中有子对象，但是子对象没有子孙对象

```java
public class Thread1 implements Runnable{
    
    @Override
    public void run() {

    }
}

//////////////////////////////////////////////
package threadgroup;

public class Main {
    public static void main(String[] args) {

        Thread1 thread1 = new Thread1();
        Thread1 thread2 = new Thread1();

        ThreadGroup tg = new ThreadGroup("纪的线程组");
        Thread thread = new Thread(tg, thread1);
        Thread thread3 = new Thread(tg, thread2);

        thread.start();
        thread3.start();

        System.out.println(tg.activeCount());
        System.out.println(tg.getName());
    }
}

```

![image-20220102102119654](https://gitee.com/jiruixin/images/raw/master/images/image-20220102102119654.png)

#### 2.多级关联

父对象中有子对象，子对象中有子孙对象

父线程组-------子线程组-----------子孙对象

#### 3.自动归属

在当前线程创建的线程组没有指定父线程组时会自动归属到当前线程所在的线程组。

```java
package threadgroup;

public class Main {
    public static void main(String[] args) {
        System.out.println("创建线程组前："+Thread.currentThread().getThreadGroup().activeGroupCount());
        ThreadGroup threadGroup = new ThreadGroup("纪念");
        System.out.println("创建线程组后："+Thread.currentThread().getThreadGroup().activeGroupCount());
    }
}

```

![image-20220102103539474](https://gitee.com/jiruixin/images/raw/master/images/image-20220102103539474.png)



####  4.获取父线程组

> getParent().

- 根父线程组为system，再取父线程组就会报空异常

#### 5.添加线程组

在创建线程组时指定父线程

#### 6.批量停止线程

```java
// 线程组的interrupt方法可以中断线程组中的所有正在运行的线程
Thread.currentThread().getThreadGroup().interrupt();
```

#### 7.递归与非递归获取组内对象

```java
package threadgroup;

public class Main {
    public static void main(String[] args) {

        ThreadGroup a = Thread.currentThread().getThreadGroup();
        ThreadGroup b = new ThreadGroup(a, "B");
        ThreadGroup c = new ThreadGroup(b, "c");
        ThreadGroup[] threadGroups = new ThreadGroup[Thread.currentThread().getThreadGroup().activeGroupCount()];

        // 把true换成false就是非递归
        Thread.currentThread().getThreadGroup().enumerate(threadGroups,true);

        for (int i = 0; i < threadGroups.length; i++) {
            if (threadGroups[i] != null){
                System.out.println(threadGroups[i].getName());
            }
        }
    }
}


```

执行结果

![image-20220102105446627](https://gitee.com/jiruixin/images/raw/master/images/image-20220102105446627.png)



把上面的代码按注释要求换成非递归的执行结果

![image-20220102105714246](https://gitee.com/jiruixin/images/raw/master/images/image-20220102105714246.png)

#### 8.enumerate

复现当前线程中的线程组及其子组中的每个线程到指定数组中

#### 9.SimpleDateFormat非线程安全

```java
package dateformat;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class MyThread implements Runnable {

    private SimpleDateFormat sdf;
    private String dateString;

    public MyThread(SimpleDateFormat sdf,String dateString){
        this.sdf = sdf;
        this.dateString = dateString;
    }

    @Override
    public void run() {
        try {
            Date parse = sdf.parse(dateString);
            String s = sdf.format(parse).toString();
            if (!s.equals(dateString)){
                System.out.println(dateString+"======="+s);
            }
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}

//////////////////////////////////////////////////////////////////////////////////
package dateformat;

import java.text.SimpleDateFormat;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        String[] strings = {"2000-01-01", "2000-01-02", "2000-01-03", "2000-01-04", "2000-01-05"};

        Thread[] threads = new Thread[strings.length];
        for (int i = 0; i < strings.length; i++) {
            threads[i] = new Thread(new MyThread(sdf,strings[i]));
        }

        for (int i = 0; i < strings.length; i++) {
            threads[i].start();
        }


    }
}

```

执行结果

![image-20220102112156916](https://gitee.com/jiruixin/images/raw/master/images/image-20220102112156916.png)



出现问题，转换不对应

解决问题

> 工具类

```java
package dateformat;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateTools {
    public static Date parse(String format, String dataString) throws ParseException {
        return new SimpleDateFormat(format).parse(dataString);
    }

    public static String format(String format,Date date){
        return new SimpleDateFormat(format).format(date).toString();
    }
}

/////////////////////////////////////////////////////////////////////////////////////////////
package dateformat;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class MyThread implements Runnable {

    private SimpleDateFormat sdf;
    private String dateString;

    public MyThread(SimpleDateFormat sdf,String dateString){
        this.sdf = sdf;
        this.dateString = dateString;
    }

    @Override
    public void run() {
        try {
            Date parse = DateTools.parse("yyyy-MM-dd",dateString);
            String s = DateTools.format("yyyy-MM-dd",parse);
            if (!s.equals(dateString)){
                System.out.println(dateString+"======="+s);
            }
        } catch (ParseException e) {
            e.printStackTrace();
        }

    }
}

///////////////////////////////////////////////////////////////
package dateformat;

import java.text.SimpleDateFormat;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        String[] strings = {"2000-01-01", "2000-01-02", "2000-01-03", "2000-01-04", "2000-01-05"};

        Thread[] threads = new Thread[strings.length];
        for (int i = 0; i < strings.length; i++) {
            threads[i] = new Thread(new MyThread(sdf,strings[i]));
        }

        for (int i = 0; i < strings.length; i++) {
            threads[i].start();
        }
    }
}


```

![image-20220102113059879](https://gitee.com/jiruixin/images/raw/master/images/image-20220102113059879.png)

没有出现就是全部转换正切

> ThreadLocal

```java
package dateformat2;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateTools {

    private static ThreadLocal<SimpleDateFormat> t1 = new ThreadLocal<>();

    public static SimpleDateFormat getSimpleDateFormat(String format){
        SimpleDateFormat sdf = null;
        sdf = t1.get();

        if (sdf ==null){
            sdf = new SimpleDateFormat(format);
            t1.set(sdf);
        }

        return sdf;
    }
}

////////////////////////////////////////////////////////////////
package dateformat2;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class MyThread implements Runnable {

    private SimpleDateFormat sdf;
    private String dateString;

    public MyThread(SimpleDateFormat sdf,String dateString){
        this.sdf = sdf;
        this.dateString = dateString;
    }

    @Override
    public void run() {
        try {
            Date parse =
                    DateTools.getSimpleDateFormat("yyyy-MM-dd").parse(dateString);

            String s = DateTools.getSimpleDateFormat("yyyy-MM-dd").format(parse).toString();
            if (!s.equals(dateString)){
                System.out.println(dateString+"======="+s);
            }
        } catch (ParseException e) {
            e.printStackTrace();
        }

    }
}

/////////////////////////////////////////////////////////////////////////////////////
package dateformat2;

import java.text.SimpleDateFormat;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        String[] strings = {"2000-01-01", "2000-01-02", "2000-01-03", "2000-01-04", "2000-01-05"};

        Thread[] threads = new Thread[strings.length];
        for (int i = 0; i < strings.length; i++) {
            threads[i] = new Thread(new MyThread(sdf,strings[i]));
        }

        for (int i = 0; i < strings.length; i++) {
            threads[i].start();
        }
    }
}

```

![image-20220102114252942](https://gitee.com/jiruixin/images/raw/master/images/image-20220102114252942.png)



### 3.线程异常

> setUncaughtExceptionHandler

```java
package exception;

public class Thread1 implements Runnable {

    private Service service;

    public Thread1(){

    }

    public Thread1(Service service){
        this.service = service;
    }

    @Override
    public void run() {
           service.method();
    }
}

////////////////////////////////////////////////////////
package exception;

public class Run {

    public static void main(String[] args) {

        Thread thread = new Thread(new Thread1());
        thread.setName("A");
        thread.setUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
            @Override
            public void uncaughtException(Thread t, Throwable e) {
                System.out.println(t.getName()+"出现异常");
            }
        });
        thread.start();
    }
}

```

![image-20220103093211437](https://gitee.com/jiruixin/images/raw/master/images/image-20220103093211437.png)



> setDefaultUncaughtExceptionHandler

Thread的内部静态方法

```java
package exception;

public class Thread1 implements Runnable {

    private Service service;

    public Thread1(){

    }

    public Thread1(Service service){
        this.service = service;
    }

    @Override
    public void run() {
           service.method();
    }
}

///////////////////////////////////////////////////
package exception;

public class Run {

    public static void main(String[] args) throws InterruptedException {

        Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
            @Override
            public void uncaughtException(Thread t, Throwable e) {
                System.out.println(t.getName()+"出现异常");
            }
        });
        Thread thread = new Thread(new Thread1());  
        thread.start();
        Thread thread2 = new Thread(new Thread1());
        Thread.sleep(100);
        thread2.start();
    }
}

```

![image-20220103093543862](https://gitee.com/jiruixin/images/raw/master/images/image-20220103093543862.png)



### 4.线程组异常

```java
package exception;

public class Mygroup extends ThreadGroup {
    public Mygroup(String name) {
        super(name);
    }

    @Override
    public void uncaughtException(Thread t, Throwable e) {
        //super.uncaughtException(t, e);
        System.out.println(t.getThreadGroup()+"线程组中线程"+t.getName()+"出现异常");
    }
}


///////////////////////////////////////////////////
package exception;

public class Thread1 implements Runnable {

    private Service service;

    public Thread1(){

    }

    public Thread1(Service service){
        this.service = service;
    }

    @Override
    public void run() {
           service.method();
    }
}

///////////////////////////////////////////////////
package exception;

import timer01.MyTask2;

public class Run {

    public static void main(String[] args) throws InterruptedException {

        Mygroup mygroup = new Mygroup("a线程");
        Thread thread = new Thread(mygroup,new Thread1());
        thread.start();
    }
}
```

![image-20220103095918852](https://gitee.com/jiruixin/images/raw/master/images/image-20220103095918852.png)

注意点

- 重写线程组的异常捕获时不要在线程对象的内部的run方法进行任何的catch操作
- 线程组内一个线程出现异常不影响其他线程的运行

### 5.线程捕获优先性

线程对象的异常最高------Thread的静态方法异常捕获其次-----------线程组的异常捕获最低





## JUC编程

### 线程安全的方式

#### 1.锁机制（线程堵塞）

通过让线程堵塞的方式，让线程同步从而保证线程安全

- synchronized关键字

- Lock

#### 2.CAS（比较并交换）机制

通过比较值来决定是否更新值。



### 1.ArrayList不安全

报错java.util.ConcurrentModificationException

```java
package unsafe;

import java.util.ArrayList;
import java.util.UUID;

public class Main {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();

        for (int i = 0; i <= 20; i++) {
            new Thread(()->{
               list.add(UUID.randomUUID().toString().substring(0,5));
               System.out.println(list);
            },String.valueOf(i)).start();
        }

    }
}

```



![image-20220104100200326](https://gitee.com/jiruixin/images/raw/master/images/image-20220104100202044.png)

解决问题

> Vector

```java
package unsafe;

        import java.util.ArrayList;
        import java.util.List;
        import java.util.UUID;
        import java.util.Vector;

public class Main {
    public static void main(String[] args) {
        // 换成Vector 底层是syn语句
        List<String> list = new Vector<>();
        for (int i = 0; i <= 20; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(list);
            },String.valueOf(i)).start();
        }

    }
}

```

> 工具类转换

```java
package unsafe;

import java.util.*;

public class Main {
    public static void main(String[] args) {
        // 工具类变成安全
        List<String> list = Collections.synchronizedList(new ArrayList<>());
        for (int i = 0; i <= 20; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(list);
            },String.valueOf(i)).start();
        }

    }
}

```

> CopyOnWriteArrayList

```java
package unsafe;

import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;

public class Main {
    public static void main(String[] args) {
        
        // Vector底层是syn语句，效率很低。但是CopyOnWriteArrayList是在写入时复制一份
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i <= 20; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(list);
            },String.valueOf(i)).start();
        }

    }
}

```

### 2.set线程不安全

报错与ArrayList一样的

解决问题

```java
// 第一种
Collections.synchronizedSet();
// 第二种
CopyOnWriteArraySet<>(); 
```

### 3.HashMap不安全

报错与ArrayList一样的

解决问题

```java
ConcurrentHashMap<>()
```

### 4. CountDownLatch

```java
package unsafe;

import java.util.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 6 ; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName());
                // 减1操作
                countDownLatch.countDown();
            }).start();
        }
        // 等待计算器为0，再往下执行
        countDownLatch.await();
        System.out.println("---------");
    }
}

```

### 5.CyclicBarrier

```java
package unsafe;

import java.util.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args){

        CyclicBarrier cyclicBarrier = new CyclicBarrier(6, () -> {
            System.out.println("操作成功");
        });
        for (int i = 0; i < 6 ; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName());
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

### 6.Semaphore

```java
package unsafe;

import java.util.concurrent.Semaphore;

public class Main {
    public static void main(String[] args){

        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"进来了");
                    Thread.sleep(1000);
                    System.out.println(Thread.currentThread().getName()+"出去了");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            }).start();
           
        }
    }
}

```

**semaphore.acquire()获得资源，如果资源已经使用完了，就等待资源释放后再进行使用！**

**semaphore.release()释放，会将当前的信号量释放+1，然后唤醒等待的线程！**

作用： 多个共享资源互斥的使用！ 并发限流，控制最大的线程数！



### 7.BlockingQueue

| 方式         | 抛出异常 | 不会抛出异常 | 阻塞等待 | 超时等待                  |
| ------------ | -------- | ------------ | -------- | ------------------------- |
| 添加         | add      | offer        | put      | offer（timenum,timeUnit） |
| 移出         | remove   | poll         | take     | poll（timenum,timeUnit）  |
| 判断队首元素 | element  | peek         | -        | -                         |

### 8.SynchronousQueue

同步队列没有容量，也可以视为容量为 1 的队列；进去一个元素，必须等待取出来之后，才能再往里面放入一个元素；

put 方法和 take 方法；

Synchronized 和 其他的BlockingQueue 不一样 它不存储元素；

put了一个元素，就必须从里面先take出来，否则不能再put进去值！

并且SynchronousQueue 的take是使用了lock锁保证线程安全的。


### 9.线程池

#### 1.三大方法

```java
ExecutorService threadPool = Executors.newSingleThreadExecutor();
ExecutorService threadPool2 = Executors.newFixedThreadPool(5);
ExecutorService threadPool3 = Executors.newCachedThreadPool();
```

#### 2.七大参数

```java
public ThreadPoolExecutor(int corePoolSize,                     // 核心线程池大小
                          int maximumPoolSize,                  // 最大的线程池大小  一般是电脑的核数
                          long keepAliveTime,                   // 超时了没有人调用就会释放
                          TimeUnit unit,       				   // 超时单位
                          BlockingQueue<Runnable> workQueue,    // 阻塞队列
                          ThreadFactory threadFactory,		   // 线程工厂 创建线程的 一般不用动
                          RejectedExecutionHandler handler      // 拒绝策略
                         ) {// 代码省略}
  
```

推荐这种new ThreadPoolExecutor类的方式创建线程池，这样能根据配置充分利用资源。

#### 3.四大拒绝策略

```java
new ThreadPoolExecutor.AbortPolicy             // 该拒绝策略为：银行满了，还有人进来，不处理这个人的，并抛出异常
new ThreadPoolExecutor.CallerRunsPolicy():     // 该拒绝策略为：哪来的去哪里 main线程进行处理
new ThreadPoolExecutor.DiscardPolicy():        // 该拒绝策略为：队列满了,丢掉异常，不会抛出异常。
new ThreadPoolExecutor.DiscardOldestPolicy()： // 该拒绝策略为：队列满了，尝试去和最早的进程竞争，不会抛出异常
```

```java
package unsafe;

import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) throws InterruptedException {

        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
                2,
                2,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.CallerRunsPolicy()
        );
        try {
            for (int i = 0; i < 6; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            threadPool.shutdown();
        }
    }
}
```

#### 4.设置线程池的大小

CPU 密集型：电脑的核数是几核就选几；选择maximunPoolSize 的大小

I/O密集型：

在程序中有15个大型任务，io十分占用资源；I/O 密集型就是判断我们程序中十分耗 I/O 的线程数量，大约是最大 I/O 数的一倍到两倍之间。



### 10四大接口

#### 1.Function 函数型接口

![在这里插入图片描述](https://gitee.com/jiruixin/images/raw/master/images/20201216213723923.png)



#### 2. Predicate 断定型接口

![在这里插入图片描述](https://gitee.com/jiruixin/images/raw/master/images/20201216213803195.png)

#### 3.Suppier 供给型接口

![在这里插入图片描述](https://gitee.com/jiruixin/images/raw/master/images/20201216213832570.png)

#### 4.Consumer 消费者接口

![image-20220104121613867](https://gitee.com/jiruixin/images/raw/master/images/image-20220104121613867.png)

### 11.Stream流式计算

```java
package com.sjmp.demo13Stream;

import java.util.Arrays;
import java.util.List;


public class StreamDemo {
    public static void main(String[] args) {
        User user1 = new User(1, "a", 21);
        User user2 = new User(2, "b", 22);
        User user3 = new User(3, "c", 23);
        User user4 = new User(4, "d", 24);
        User user5 = new User(5, "e", 25);

        List<User> list = Arrays.asList(user1, user2, user3, user4, user5);
        // lambda、链式编程、函数式接口、流式计算
        list.stream()
                .filter(user -> {return user.getId()%2 == 0;})
                .filter(user -> {return user.getAge()>23;})
                .map(user -> {return user.getName().toUpperCase();})
                .sorted((u1,u2)->{return u2.compareTo(u1);})
                .limit(1)
                .forEach(System.out::println);
    }
}
```

### 12.CAS

CAS(compare and swap)翻译为中文是比较并交换

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存地址里面的值和A的值是一样的，那么就将内存里面的值更新成B。CAS是通过无限循环来获取数据的，若果在第一轮循环中，a线程获取地址里面的值被b线程修改了，那么a线程需要自旋，到下次循环才有可能机会执行。

CAS在更新值是会比较内存里面的值是否和预期的值是一样的，如果一样才会更新为新值。

```java
package unsafe;

import java.util.concurrent.atomic.AtomicInteger;

public class Main {
    static AtomicInteger atomicInteger = new AtomicInteger(0);
    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            new Thread(()->{
                while (atomicInteger.get()<500){
                    System.out.println(Thread.currentThread().getName()+": "+atomicInteger.getAndIncrement());
                }
            }).start();
        }

    }
}

```

CAS：比较当前工作内存中的值 和 主内存中的值，如果这个值是期望的，那么则执行操作！如果不是就一直循环，使用的是自旋锁。

**缺点：**

- 循环会耗时；
- 一次性只能保证一个共享变量的原子性；
- 它会存在ABA问题

> 什么是ABA?

```java
package unsafe;

import java.util.concurrent.atomic.AtomicInteger;

public class Main {

    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicInteger = new AtomicInteger(0);

        // 线程1的操作
        new Thread(()->{
            if ( atomicInteger.compareAndSet(0,1) &&
                    atomicInteger.compareAndSet(1,0)){
                System.out.println("对0进行了修改");
            }
        }).start();

        Thread.sleep(100);
        // 线程2想要获取的值为0
        // 此时就会与ABA问题，虽然线程2的预期值是0，但是被修改过的0，已经不是原来的0，这就是ABA问题
        new Thread(()->{
            if ( atomicInteger.compareAndSet(0,1)){
                System.out.println("修改成功");
            }
        }).start();
    }
}
```

> 解决ABA问题 -原子引用

就是使用了**乐观锁**。带版本号的原子操作！

Integer 使用了对象缓存机制，默认范围是 -128~127，推荐使用静态工厂方法 valueOf 获取对象实例，而不是 new ，因为 valueOf 使用了缓存，而 new 一定会创建新的对象分配新的内存空间。

![在这里插入图片描述](https://gitee.com/jiruixin/images/raw/master/images/20201216214808262.png)

```java
package unsafe;

import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;

public class Main {

    public static void main(String[] args) throws InterruptedException {
        AtomicStampedReference<Integer> atomicInteger = new  AtomicStampedReference<>(0,0);
        int stamp = atomicInteger.getStamp();
        // 线程1的操作
        new Thread(()->{
            if ( atomicInteger.compareAndSet(0,1,atomicInteger.getStamp(),atomicInteger.getStamp()+1) &&
                    atomicInteger.compareAndSet(1,0,atomicInteger.getStamp(),atomicInteger.getStamp()+1)){
                System.out.println("对0进行了修改");
            }

        }).start();

        Thread.sleep(100);
        // 线程2想要获取的值为0
        // AtomicStampedReference增加了版本号，所以只要更改了值版本号会变，就不会导致ABA的问题了。
        new Thread(()->{
            if ( atomicInteger.compareAndSet(0,1,stamp,stamp+1)){
                System.out.println("修改成功");
            }else {
                System.out.println("更新失败，最新版本号为"+atomicInteger.getStamp());
            }
        }).start();
    }
}
```

![image-20220104151928285](https://gitee.com/jiruixin/images/raw/master/images/image-20220104151928285.png)

### 13.自旋锁

```java
package unsafe;

import java.util.concurrent.atomic.AtomicReference;

public class MySpinlock {

    // 默认
    // int 0
    // thread null
    AtomicReference<Thread> atomicReference=new AtomicReference<>();

    //加锁
    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName()+"===> mylock");

        //自旋锁
        while (!atomicReference.compareAndSet(null,thread)){
            System.out.println(Thread.currentThread().getName()+" ==> 自旋中~");
        }
    }

    //解锁
    public void myUnlock(){
        Thread thread=Thread.currentThread();
        System.out.println(thread.getName()+"===> myUnlock");
        atomicReference.compareAndSet(thread,null);
    }

}

///////////////////////////////////////////////////////////////
package unsafe;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;
import java.util.concurrent.locks.ReentrantLock;

public class Main {

    public static void main(String[] args) throws InterruptedException {

        // 使用CAS实现自旋锁
        MySpinlock spinlockDemo=new MySpinlock();
        new Thread(()->{
            spinlockDemo.myLock();
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                spinlockDemo.myUnlock();
            }
        },"t1").start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{
            spinlockDemo.myLock();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                spinlockDemo.myUnlock();
            }
        },"t2").start();
    }
}
```

![image-20220104151842005](https://gitee.com/jiruixin/images/raw/master/images/image-20220104151842005.png)



### 14.ForkJoin



### 15. 异步回调

#### 1.没有返回值的异步回调

```java
package unsafe;


import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        CompletableFuture<Void> future = CompletableFuture.runAsync(()->{
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"......");
        });
		
        System.out.println("-------------------------");
        // 等待执行完毕获取结果再往下执行
        System.out.println(future.get());// 获取执行结果
        System.out.println("*************************");
    }
}
```

#### 2.有返回值的异步回调

```java
package unsafe;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // int i = 10/0;
            return 1024;
        });

        System.out.println(completableFuture.whenComplete((t, u) -> {
		   // success 回调
            System.out.println("t=>" + t);//正常的返回结果
            System.out.println("u=>" + u);//抛出异常的错误信息
        }).exceptionally((e) -> {
            // error 回调
            System.out.println(e.getMessage());
            return 404;
        }).get());
    }
}
```

## JUC里面重要的类



### TimeUnit和sleep

提高程序的可读性，底层还是sleep



### LongAdder

```java
public class LongAdderThread extends Thread {
    private LongAdder count =  new LongAdder();
    
    @Override
    public void run() {
        {
            for (int i = 0; i < 100; i++) {
               count.add(1);
            }
        }
        System.out.println(Thread.currentThread().getName()+"    "+count.sum());
    }
}


public class Main {

    public static void main(String[] args) {
   
        LongAdderThread myThread = new LongAdderThread();

        Thread thread = new Thread(myThread);
        thread.start();

        Thread thread1 = new Thread(myThread);
        thread1.start();

        Thread thread2 = new Thread(myThread);
        thread2.start();

        Thread thread3 = new Thread(myThread);
        thread3.start();

        Thread thread4 = new Thread(myThread);
        thread4.start();
    }
}


```

### LongAccumulator

```java
public class LongAccumulatorThread extends Thread {

    private LongAccumulator count =  new LongAccumulator((long left, long right)->{
        return left * right;
    },0);

    @Override
    public void run() {
        {
            for (int i = 0; i < 100; i++) {
               count.accumulate(10);
            }
        }
        System.out.println(Thread.currentThread().getName()+"    "+count.get());
    }
}


public class Main {

    public static void main(String[] args) {
        LongAccumulatorThread myThread = new LongAccumulatorThread();

        Thread thread = new Thread(myThread);
        thread.start();

        Thread thread1 = new Thread(myThread);
        thread1.start();

        Thread thread2 = new Thread(myThread);
        thread2.start();

        Thread thread3 = new Thread(myThread);
        thread3.start();

        Thread thread4 = new Thread(myThread);
        thread4.start();
    }
}

```

#### CopyOnWriteArrayList

加锁操作

- 增、删、改

无锁操作

- 查询

CopyOnWriteArrayList的*查询和迭代*操作是弱一致性。



### LockSupport

LockSupport默认情况和使用它的线程有一个许可证，默认情况下该线程的不持有许可证。

在没有获取许可的情况下，调用LockSupport.park()的线程会被挂起

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("start");
        LockSupport.park();
        System.out.println("end");
    }
}

//打印结果
//start
```

如果想让挂起的线程返回

>  让挂起线程获取许可

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("start");
        //获取线程许可
        LockSupport.unpark(Thread.currentThread());
        LockSupport.park();
        System.out.println("end");
    }
}

//结果
//start
//end
```

> 使用interrupt()方法

```java

```





## 9. 线程的JVM内存

### 1.单线程内存图

```java
package unsafe;

public class Main {

    public static void main(String[] args) throws InterruptedException, ExecutionException {
    	method1(10);    
    }
    
    private static void method1(int x){
        int y = x + 1;
        Object m = method2();
        System.out.println(m);
    }

    private static Object method2(){
        Object n = new Object();
        return n;
    }
}
```

内存图(执行到 Object n = new Object(); )

![image-20220104181702874](https://gitee.com/jiruixin/images/raw/master/images/image-20220104181702874.png)





![image-20220104181702874.png](https://github.com/blank-baby/images_gitee/blob/master/images/image-20220104181702874.png)









### 2. 多线程内存图

