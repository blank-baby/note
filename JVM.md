# JVM

## 1 编译期

.javac把.java的文件编译成.class文件

## 2. 运行期

利用java运行.class的二进制文件

### 2.1JVM图

- 方法区:静态变量（static）  常量（final） 、类信息（class）（构造方法、接口定义）、运行时的常量池存在方法区
- 栈:8大基本类型、对象的引用、实例的方法
- 堆：具体的实例对象（对象和数组）。
- 本地方法栈：带native的方法。
- 执行引擎：去执行字节码。
- 本地方法接口：native的方法就是调用的这个接口

![image-20210204110650344](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210204110650344.png)

### 2.2 类加载

- 启动类根加载器
- 扩展类加载器
- 应用程序系统类加载器

双亲委派机制：

1. 类加载器有加载类的请求。
2. 委托给父类的加载器加载，一直向上委托直到根加载器。
3. 如果根加载器有就去加载这个类，没有就会抛出异常给子类，通知子类加载器进行加载。
4. 重复3的步骤。

![image-20210204110811698](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210204110811698.png)

