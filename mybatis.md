# ssm-mybatis

## 1. mapper绑定

### 1. 对mapper进行绑定

```xml
<mappers>
        <mapper resource="cn/runningone/dao/user/UserMapper.xml"/>
</mappers>
```

### 2.对mapper进行绑定

```xml
<mappers>
        <mapper class="cn.runningone.dao.user.UserMapper"/>
</mappers>
```

- 接口和它的mapper配置文件必须同名
- 接口和它的mapper配置文件必须在同一包下

### 3 .对mapper进行绑定

```xml
<mappers>
    <package name="cn.runningone.dao"/>
</mappers>
    
```

- 接口和它的mapper配置文件必须同名
- 接口和它的mapper配置文件必须在同一包下

## 2. 实体类起别名

### 1.第一种(类少的时候使用)

```xml
<typeAliases>
        <typeAlias type="cn.runningone.pojo.User" alias="User"/>
</typeAliases>
    
```

### 2.第二种(类多的时候使用)

```xml
<typeAliases>
        <package name="cn.runningone.pojo"/>
</typeAliases>
```

- 该方法的类别名默认是类的类名首字母小写
- 或者给在实体类上加@Alias()

## 3. Mapper传递参数

### 1.序号传

### 2.@Param





# ssm-spring

## 1. Ioc创建对象

### 1.无参数构造

IOC容器调用类的无参数构造器构造函数，再利用set方法给对象配置属性。

```xml
<bean id="myUser" class="cn.runningone.pojo.User">
        <property name="userName" value="jiruixin"/>
</bean>
```

### 2.有参数构造

直接利用容器的有参构造方法来创建对象。

```xml
<!-- 第一种根据index参数下标设置 -->
<bean id="myUser" class="cn.runningone.pojo.User">
   <!-- index指构造方法 , 下标从0开始 -->
   <constructor-arg index="0" value="kuangshen2"/>
</bean>
<!-- 第二种根据参数名字设置 -->
<bean id="myUser" class="cn.runningone.pojo.User">
   <!-- name指参数名 -->
   <constructor-arg name="name" value="kuangshen2"/>
</bean>
<!-- 第三种根据参数类型设置(不推荐使用) -->
<bean id="myUser" class="cn.runningone.pojo.User">
   <constructor-arg type="java.lang.String" value="kuangshen2"/>
</bean>
```

## 2.起别名

### 1.alias标签起别名

```xml
<!--设置别名：在获取Bean的时候可以使用别名获取-->
<alias name="userT" alias="userNew"/>
```

### 2.name属性起别名

```xml
<!--bean就是java对象,由Spring创建和管理-->

<!--
   id 是bean的标识符,要唯一,如果没有配置id,name就是默认标识符
   如果配置id,又配置了name,那么name是别名
   name可以设置多个别名,可以用逗号,分号,空格隔开
   如果不配置id和name,可以根据applicationContext.getBean(.class)获取对象;

class是bean的全限定名=包名+类名
-->
<bean id="hello" name="hello2 h2,h3;h4" class="com.kuang.pojo.Hello">
   <property name="name" value="Spring"/>
</bean>
```

## 3.依赖注入

### 1. 通过有参构造器注入

```xml
<!-- 第一种根据index参数下标设置 -->
<bean id="myUser" class="cn.runningone.pojo.User">
   <!-- index指构造方法 , 下标从0开始 -->
   <constructor-arg index="0" value="kuangshen2"/>
</bean>
<!-- 第二种根据参数名字设置 -->
<bean id="myUser" class="cn.runningone.pojo.User">
   <!-- name指参数名 -->
   <constructor-arg name="name" value="kuangshen2"/>
</bean>
<!-- 第三种根据参数类型设置(不推荐使用) -->
<bean id="myUser" class="cn.runningone.pojo.User">
   <constructor-arg type="java.lang.String" value="kuangshen2"/>
</bean>
```



### 2.set注入

**使用set方法注入在类中必须有set的方法**

#### 1常量注入

```xml
<bean id="myUser" class="cn.runningone.pojo.User">
        <property name="userName" value="jiruixin"/>
</bean>
```

#### 2 Bean注入

使用ref标签

```xml
<bean id="addr" class="cn.runningone.Address">
     <property name="address" value="重庆"/>
</bean>
 
<bean id="student" class="cn.runningone.user">
    <property name="name" value="小明"/>
    <property name="address" ref="addr"/>
</bean>
```

#### 3 数组注入

```xml
<bean id="student" class="com.kuang.pojo.Student">
    <property name="name" value="小明"/>
    <property name="address" ref="addr"/>
    <property name="books">
        <array>
            <value>西游记</value>
            <value>红楼梦</value>
            <value>水浒传</value>
        </array>
    </property>
</bean>

```

#### 4List注入

```xml
<property name="hobbys">
    <list>
        <value>听歌</value>
        <value>看电影</value>
        <value>爬山</value>
    </list>
</property>
```

#### 5 Map注入

```xml
<property name="card">
    <map>
        <entry key="中国邮政" value="456456456465456"/>
        <entry key="建设" value="1456682255511"/>
    </map>
</property>
```

#### 6.set注入

```xml
<property name="games">
     <set>
         <value>LOL</value>
         <value>BOB</value>
         <value>COC</value>
     </set>
 </property>
```

#### 7.null注入

```xml
<property name="wife"><null/></property>
```

#### 8.**Properties注入**

```xml
<property name="info">
    <props>
        <prop key="学号">20190604</prop>
        <prop key="性别">男</prop>
        <prop key="姓名">小明</prop>
    </props>
</property>
```

### 3其他方式注入依赖

#### 1.p标签

```xml
导入约束 : xmlns:p="http://www.springframework.org/schema/p"

<!--P(属性: properties)命名空间 , 属性依然要设置set方法-->
<bean id="user" class="cn.runningone.pojo.User" p:name="jiruixin" p:age="18"/>
```

#### 2c标签

```xml
导入约束 : xmlns:c="http://www.springframework.org/schema/c"
<!--C(构造: Constructor)命名空间 , 属性依然要设置set方法-->
<bean id="user" class="cn.runningone.pojo.User" c:name="jiruixin" c:age="18"/>
```

## 4bean装配

装配有三种方式

- 在xml中显式配置；
- 在java中显式配置；
- 隐式的bean发现机制和自动装配。

以上都是xml装配，这里主要讲第三种装配：bean自动化装配

### 1.xml配置自动装配

通过set方法注入

#### 1 autowire byName

```xml
<bean id="user" class="com.kuang.pojo.User" autowire="byName">
    <property name="str" value="qinjiang"/>
</bean>
```

当一个bean节点带有 autowire byName的属性时。

1. 将查找其类中所有的set方法名，例如setCat，获得将set去掉并且首字母小写的字符串，即cat。
2. 去spring容器中寻找是否有此字符串名称id的对象。
3. 如果有，就取出注入；如果没有，就报空指针异常。

#### 2.autowire byType

使用autowire byType首先需要保证：同一类型的对象，在spring容器中唯一。如果不唯一，会报不唯一的异常。

### 2注解自动装配

- 需提前在配置文件把bean交给IOC管理。

- 可以无set方法，通过反射机制注入。

#### 1.@Autowired

- 通过类型寻找匹配的属性注入，加上@Qualifier可以根据name匹配。

- @Qualifier不能单独使用。

#### 2.@Resource

1. 先按name属性来匹配
2. 然后是byname
3. 最后是bytype

#### 3.包扫描加注解

须在bean配置文件中加上

```xml
<context:component-scan base-package="com.kuang.pojo"/>
```

须在类上加注解@Component，代表把这个类加入到IOC中。为了更好的进行分层，Spring可以使用其它三个注解，功能一样，目前使用哪一个功能都一样。

- @Controller：web层
- @Service：service层
- @Repository：dao层

### 3.java类配置bean

```java
@Configurable //代表这是配置类
@ComponentScan("cn.runningone.pojo")  //启动包扫描
@Import() //引入其他的配置类
public class Config_Spring {
   //和配置文件中的bean标签一样
   @Bean
   public Dog dog(){
       return new Dog();
  }

}
```

## 5.代理

- 静态代理
- 动态代理

## 6.AOP

首先需要导入一个包

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

### 1.Spring API实现

业务代码和实现类

```java
public interface UserService {

   public void add();

   public void delete();

   public void update();

   public void search();

}
public class UserServiceImpl implements UserService{

   @Override
   public void add() {
       System.out.println("增加用户");
  }

   @Override
   public void delete() {
       System.out.println("删除用户");
  }

   @Override
   public void update() {
       System.out.println("更新用户");
  }

   @Override
   public void search() {
       System.out.println("查询用户");
  }
}
```

然后去写我们的增强类 , 我们编写两个 , 一个前置增强 一个后置增强

```java
public class Log implements MethodBeforeAdvice {

   //method : 要执行的目标对象的方法
   //objects : 被调用的方法的参数
   //Object : 目标对象
   @Override
   public void before(Method method, Object[] objects, Object o) throws Throwable {
       System.out.println( o.getClass().getName() + "的" + method.getName() + "方法被执行了");
  }
}
public class AfterLog implements AfterReturningAdvice {
   //returnValue 返回值
   //method被调用的方法
   //args 被调用的方法的对象的参数
   //target 被调用的目标对象
   @Override
   public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
       System.out.println("执行了" + target.getClass().getName()
       +"的"+method.getName()+"方法,"
       +"返回值："+returnValue);
  }
}
```

spring的文件中注册 , 并实现aop切入实现 , 注意导入约束 .

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

   <!--注册bean-->
   <bean id="userService" class="cn.runningone.service.UserServiceImpl"/>
   <bean id="log" class="cn.runningone.log.Log"/>
   <bean id="afterLog" class="cn.runningone.log.AfterLog"/>

   <!--aop的配置-->
   <aop:config>
       <!--切入点 expression:表达式匹配要执行的方法-->
       <aop:pointcut id="pointcut" expression="execution(* cn.runningone.service.UserServiceImpl.*(..))"/>
       <!--执行环绕; advice-ref执行方法 . pointcut-ref切入点-->
       <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
       <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
   </aop:config>

</beans>
```

### 2自定义 实现Aop

目标业务类不变依旧是userServiceImpl,写自己的切入类

```java
public class DiyPointcut {

    public void before(){
        System.out.println("---------方法执行前---------");
    }
    public void after(){
        System.out.println("---------方法执行后---------");
    }

}
```

```xml
<!--第二种方式自定义实现-->
<!--注册bean-->
<bean id="diy" class="cn.runningone.config.DiyPointcut"/>

<!--aop的配置-->
<aop:config>
    <!--第二种方式：使用AOP的标签实现-->
    <aop:aspect ref="diy">
        <aop:pointcut id="diyPonitcut" expression="execution(* cn.runningone.service.UserServiceImpl.*(..))"/>
        <aop:before pointcut-ref="diyPonitcut" method="before"/>
        <aop:after pointcut-ref="diyPonitcut" method="after"/>
    </aop:aspect>
</aop:config>
```

### 3.注解实现

编写的自己的注解类

```java
@Aspect
public class AnnotationPointcut {
    @Before("execution(* cn.runningone.service.UserServiceImpl.*(..))")
    public void before(){
        System.out.println("---------方法执行前---------");
    }

    @After("execution(* cn.runningone.service.UserServiceImpl.*(..))")
    public void after(){
        System.out.println("---------方法执行后---------");
    }

    @Around("execution(* cn.runningone.service.UserServiceImpl.*(..))")
    public void around(ProceedingJoinPoint jp) throws Throwable {
        System.out.println("环绕前");
        System.out.println("签名:"+jp.getSignature());
        //执行目标方法proceed
        Object proceed = jp.proceed();
        System.out.println("环绕后");
        System.out.println(proceed);
    }
}
```

在Spring配置文件中注册bean

```xml
<!--第三种方式:注解实现-->
<bean id="annotationPointcut" class="com.kuang.config.AnnotationPointcut"/>
<aop:aspectj-autoproxy/>
```

#  Springmvc





































