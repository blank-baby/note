## 1.web服务

### 1.容器

#### web容器

web容器（web服务器）主要有：Apache、IIS、Tomcat、Jetty、JBoss、webLogic等，而Tomcat、Jetty、JBoss、webLogic同时也是servlet容器，或者说他们还包含了servlet容器。没有servlet容器，你也可以用web容器直接访问静态页面，比如安装一个apache等，但是如果要显示jsp/servlet，你就要安装一个包含servlet容器的web容器。大多数servlet容器同时提供了web容器的功能，也就是说大多servelt容器可以独立运行你的web应用。

web容器是管理servlet（通过servlet容器），以及监听器(Listener)和过滤器(Filter)的。这些都是在web容器的掌控范围里

#### servlet容器

#### spring容器

#### springMVC容器

![image-20211225113531364](https://gitee.com/jiruixin/images/raw/master/images/image-20211225113531364.png)

- DispatchServlet对象里面包含了一个spring容器。这个spring容器一般是存放controller的bean

- 每个spring容器里面都可以设置一个spring父容器。这个容器一般存放的是service和dao层的bean

### 2.Servlet

servlet的3.0版本有了新特性，可以利用**ServletContainerInitializer**接口实现不用web.xml配置

没有这个特性之前我们需要配置web.xml文件

```xml
<web-app>
    <!--
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>

        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app-context.xml</param-value>
        </context-param>
	-->
    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```

上面配置的作用：

- 加载一个DispatcherServlet，并且给这个类做了一些配置

  

加载流程

- tomcat启动  
- 加载web.xml文件  
- 初始化spring容器（此步骤可省略） 
- 初始化DispatcherServlet（该类里面会有自己的spring容器）
- ....



3.0特性以后不再需要web.xml配置文件，只需要实现WebApplicationInitializer这个接口

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        //context.register(AppConfig.class);

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

上面代码的作用：

- 加载一个DispatcherServlet，并且给这个类做了一些配置



**启动流程（针对外部tomca+war的启动形式，不包括springboot内置tomcat启动）**

- tomcat启动 

- 加载SpringServletContainerInitializer这个类



第二个步骤很多小伙伴会有疑问?

为什么会去加载SpringServletContainerInitializer这个类？



这是tomcat实现的，tomcat利用java的spi机制去加载的**METAINF\services\javax.servlet.ServletContainerInitializer**文件里面的类

```
org.springframework.web.SpringServletContainerInitializer
```

以上是spring在web的包里实现的，见下图

![image-20211225095755412](https://gitee.com/jiruixin/images/raw/master/images/image-20211225095755412.png)

我们也可以按照以上的格式自己实现这个类，tomcat也会去调用。但有两点需要注意

- 定义的文件必须是**META-INF\services\javax.servlet.ServletContainerInitializer**这样
- 把需要加载的类全限定名写在里面
- 这个类必须实现ServletContainerInitializer接口

> @HandlesTypes({WebApplicationInitializer.class})

实现ServletContainerInitializer接口的类需要加上这个注解，注解里面的传入的类是一个接口，只要实现这个接口就会被调用

spring传入的是**WebApplicationInitializer.class**这个类，只要实现这个接口的类就会被调用。这个时候大家会发现上面的代码中的类就实现了该接口，onStartup方法就会被调用。而onStartup方法里面的做的事和web.xml配置文件做的事是一样的，所以才替代了web.xml配置



Springboot内嵌Tomcat

```
1.3.2. Servlet Context Initialization
Embedded servlet containers do not directly execute the servlet 3.0+ `javax.servlet.ServletContainerInitializer` interface or Spring’s `org.springframework.web.WebApplicationInitializer` interface. This is an intentional design decision intended to reduce the risk that third party libraries designed to run inside a war may break Spring Boot applications.

If you need to perform servlet context initialization in a Spring Boot application, you should register a bean that implements the `org.springframework.boot.web.servlet.ServletContextInitializer` interface. The single `onStartup` method provides access to the `ServletContext` and, if necessary, can easily be used as an adapter to an existing `WebApplicationInitializer`

// 翻译
嵌入式servlet容器不会直接执行servlet 3.0+ javax.servlet.ServletContainerInitializer接口或Spring的org.springframework.web.WebApplicationInitializer接口。这是一个有意的设计决策，目的是为了减少在战争中运行的第三方库可能破坏Spring Boot应用程序的风险。

如果你需要在Spring Boot应用程序中执行servlet上下文初始化，你应该注册一个实现org.springframework.boot.web.servlet.ServletContextInitializer接口的bean。onStartup方法提供了对ServletContext的访问，如果需要的话，可以很容易地用作现有WebApplicationInitializer的适配器。

```



![image-20211228104646343](https://gitee.com/jiruixin/images/raw/master/images/image-20211228104646343.png)



### 3.servlet、Listener、Filter

#### 1.注解加载

1.在bean上添加@ServletComponentScan(扫描路径)

一般在启动类上或者配置类上添加这个注解

添加对应的注解

- @WebServlet
- @WebFilter
- @WebListener

```java
@WebServlet(urlPatterns = "/myServlet")
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("-------------------------------");
        super.doGet(req, resp);
    }
    
}

// 添加扫描路径
@ServletComponentScan(basePackages = "com.health.manage.controller")
public class ApplicationConfig {

}
```

#### 2.配置类添加

```java
@Bean
public ServletRegistrationBean servletRegistrationBean(){
    ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean();
    servletRegistrationBean.setServlet(new MyServlet());
    servletRegistrationBean.addUrlMappings("/myServlet");
    return servletRegistrationBean;
}
```

#### 3. 通过ServletContextInitializer, ServletContextAware添加

使用其中任意一个都可以实现添加

```java
@Component
public class MyServletContainer implements ServletContextInitializer, ServletContextAware {
    // ServletContextInitializer的重写方法
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        servletContext.addServlet("myServlet",new MyServlet()).addMapping("/myServlet");
    }

    // ServletContextAware的重写方法
    @Override
    public void setServletContext(ServletContext servletContext) {
 		servletContext.addServlet("myServlet",new MyServlet()).addMapping("/myServlet");
    }
}
```



### SpringBoot的流程

### 配置环境

#### 创建容器

![image-20220120145214561](https://gitee.com/jiruixin/images/raw/master/images/image-20220120145214561.png)

#### 准备容器

```java
private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, Banner printedBanner) {
		context.setEnvironment(environment);
		/**
		 *  对容器和容器中的beanFactory做一些设置
		 *
		 */
		postProcessApplicationContext(context);
		/**
		 *  调用ApplicationContextInitializer的实现类
		 */
		applyInitializers(context);
		listeners.contextPrepared(context);
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// Add boot specific singleton beans
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		if (printedBanner != null) {
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		if (this.lazyInitialization) {
			context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
		}
		// Load the sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		/**
		 * 加载启动类到beanDefinition的Map
		 */
		load(context, sources.toArray(new Object[0]));
		listeners.contextLoaded(context);
	}
```

#### 刷新容器





## spring源码

### Spring的Bean的生命周期



#### 执行BeanFactoryPostProcessor

先执行BeanFactoryPostProcessor的子类BeanDefinitionRegistryPostProcessor（顺序：PriorityOrdered ---> Ordered ---> 普通的）

再执行BeanFactoryPostProcessor的实现类（顺序：PriorityOrdered ---> Ordered ---> 普通的）



#### 注册BeanPostProcessor

生成BeanPostProcessor的实现类为bean，再把它加到BeanPostProcessor的列表里，为生成其他的bean调用



#### LoadTimeWeaverAware

生成LoadTimeWeaverAware实现类的bean。

#### 实例化开始前

会调用的BeanPostProcessor的实例化前的方法，如果返回bean，则调用实例化后的方法直接返回bean

如果返回为null，反射生成对象，调用实例化后的方法，调用InstantiationAwareBeanPostProcessor的

postProcessProperties的方法，填充属性，进行Aware的回调，进行初始化前方法调用，调用InitializingBean的afterPropertiesSet方法

进行初始化后的方法调用。

### Spring扩展

####  1.bean生命周期的扩展

> 对属性资源的扩展，还没弄明白

```java
@Override
protected void initPropertySources(){
    
}
```

#### BootstrapRegistryInitializer(springboot)

- SpringApplication实例对象通过addInitializers()方法

#### EnvironmentPostProcessor(springboot)

对资源的扩展接口接口

#### ApplicationContextInitializer

在springboot准备容器这个方法中调用这个接口的实现类

![image-20220120112606741](https://gitee.com/jiruixin/images/raw/master/images/image-20220120112606741.png)



ApplicationContextInitializer通过三种方式可以加载

- java的SPI机制
- 配置文件指定全限定名
- SpringApplication实例对象通过addInitializers()方法

**springboot通过实现这个接口加载BeanFactoryPostProcessor到AbstractApplicationContext的**

**List<BeanFactoryPostProcessor> beanFactoryPostProcessors属性中**

#### BeanDefinitionRegistryPostProcessor

实现对BeanDefinition的处理，实现的功能

- 注册BeanDefinition
- 移除BeanDefinition
- 是否包含BeanDefinition
- 获取所有的BeanDefinition的名称
- 修改BeanDefinition

#### BeanFactoryPostProcessor

主要是对BeanFactory的修改，不能注册BeanDefinition



LoadTimeWeaverAware

在初始化之前先找到LoadTimeWeaverAware的实现类注册为bean

#### BeanPostProcessor

**BeanPostProcessor的实现类会被所有的bean调用**

##### InstantiationAwareBeanPostProcessor

实例化之前调用postProcessBeforeInstantiation，如果返回的为不为null，再调用postProcessAfterInstantiation，如果没调用postProcessBeforeInstantiation会直接调用

这是BeanPostProcessor的子类

```java
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    System.out.println(beanName);
    System.out.println("初始化之前");
    return bean;
}

@Override
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    System.out.println(beanName);
    System.out.println("初始化之后");
    return bean;
}

@Override
public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
    System.out.println(beanName);
    System.out.println("实例化之前");
    return null;
}

@Override
public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
    System.out.println(beanName);
    System.out.println("实例化之后");
    return false;
}
```

#### SmartInitializingSingleton

对单独的bean，在bean的生命周期结束之后调用实现该类的bean的afterSingletonsInstantiated方法

#### LifecycleProcessor

在完成容器刷新的时候调用



#### ApplicationRunner

在容器刷新完毕了会找实现了该接口的bean，然后调用它的run方法

可以容器刷新完毕之后需要执行某代码可以实现这个接口

#### CommandLineRunner

和上面的接口作用类似，区别在于调用方法时的参数不一样



#### 2. 对单独Bean的

- FactoryBean

FactoryBean的接口可以改变真是的Bean对象

```java
@Component
public class MyFactoryBean implements FactoryBean {
    @Override
    public Object getObject() throws Exception {
        return new Object();
    }

    @Override
    public Class<?> getObjectType() {
        return Object.class;
    }
}
```

- BeanNameAware
- ApplicationContextAware
- BeanFactoryAware
- BeanClassLoaderAware

```java
package com.bysj.extend;

// 利用这些接口可以在创建bean时获取一些bean的信息
@Component
public class MyBeanTend implements BeanNameAware, ApplicationContextAware, BeanFactoryAware,BeanClassLoaderAware {
   
    // BeanNameAware
    @Override
    public void setBeanName(String name) {
        System.out.println(name);
    }

    // ApplicationContextAware
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println(applicationContext);
    }

    // BeanFactoryAware
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println(beanFactory);
    }
    
    // BeanClassLoaderAware
    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println(classLoader);
    }
}

```

- DisposableBean
- InitializingBean

```java
// DisposableBean的方法
@Override
public void destroy() throws Exception {
    System.out.println("销毁中-------------");
}

// InitializingBean的方法
@Override
public void afterPropertiesSet() throws Exception {
    System.out.println("afterPropertiesSet");
}
```





> @PostConstruct  @PreDestroy 

@PostConstruct 相当于init-method的方法

@PreDestroy 销毁bean的方法

> @Value  @Autowired @Resource

@Value注册string的属性

@Autowired 注册bean的属性（这是ByType）

@Resource 注册bean的属性（先是ByName  然后ByType）

在AutowiredxxxBeanFactoryPostPocessor



#### ApplicationListener



### spring注册bean

#### ImportBeanDefinitionRegistrar

- 只能通过@Import来引入ImportBeanDefinitionRegistrar的实现类
- 通过注解的方式注册不了ImportBeanDefinitionRegistrar的实现类
- 会自动调用下面个方法

```java
// 回调次方法
default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry,BeanNameGenerator importBeanNameGenerator) {

    registerBeanDefinitions(importingClassMetadata, registry);
}

default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {}
```

### springboot的包路径

> ComponentScanAnnotationParser

通过启动类的包名获取扫描路径，把controller和service的bean扫描到容器里面

```java
Set<String> basePackages = new LinkedHashSet<>();
if (basePackages.isEmpty()) {
    basePackages.add(ClassUtils.getPackageName(declaringClass));
}
return scanner.doScan(StringUtils.toStringArray(basePackages));
```





## Mybatis-spring的源码

### 1.MapperScan

- 获取扫描的包路径
- 引入MapperScannerRegistrar这个类

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({MapperScannerRegistrar.class})
@Repeatable(MapperScans.class)
public @interface MapperScan {
    String[] value() default {};

    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    Class<? extends Annotation> annotationClass() default Annotation.class;

    Class<?> markerInterface() default Class.class;

    String sqlSessionTemplateRef() default "";

    String sqlSessionFactoryRef() default "";

    Class<? extends MapperFactoryBean> factoryBean() default MapperFactoryBean.class;

    String lazyInitialization() default "";
}
```

### 2.MapperScannerRegistrar

通过这个类注册MapperScannerConfigurer为BeanDefinition

```java
BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(MapperScannerConfigurer.class);

/**
	给builder对象添加各种属性的操作，这些代码已经省略
*/

registry.registerBeanDefinition(beanName, builder.getBeanDefinition());
```

### 3.MapperScannerConfigurer

- 保存着扫描Mapper的路径

```java
public class MapperScannerConfigurer implements BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware {
    private String basePackage;                        //Mapper的路径
   
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry){
    
        ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
        
        scanner.registerFilters();
        //ClassPathMapperScanner的方法
        scanner.scan(StringUtils.tokenizeToStringArray(this.basePackage, ",; \t\n"));
    }
}

```

###  4.ClassPathMapperScanner 

- ClassPathMapperScanner  extends ClassPathBeanDefinitionScanner

```java
 
// 这是ClassPathBeanDefinitionScanner的scan方法
public int scan(String... basePackages) {
     int beanCountAtScanStart = this.registry.getBeanDefinitionCount();
     this.doScan(basePackages);
     if (this.includeAnnotationConfig) {
         AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
     }

     return this.registry.getBeanDefinitionCount() - beanCountAtScanStart;
 }

// ClassPathMapperScanner的方法
public Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);
    if (beanDefinitions.isEmpty()) {
       // 省略......
    } else {
        // 这个方法会替换成动态Mybatis的FactoryBean
        this.processBeanDefinitions(beanDefinitions);
    }

    return beanDefinitions;
}


private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {
        GenericBeanDefinition definition;
        for(Iterator var3 = beanDefinitions.iterator(); var3.hasNext(); definition.setLazyInit(this.lazyInitialization)) {
            BeanDefinitionHolder holder = (BeanDefinitionHolder)var3.next();
            definition = (GenericBeanDefinition)holder.getBeanDefinition();
            String beanClassName = definition.getBeanClassName();
            
            // 设置成为MapperFactoryBean
            definition.setBeanClass(this.mapperFactoryBeanClass);
            definition.getPropertyValues().add("addToConfig", this.addToConfig);
}
```

### 5. 剩下的都交给FactoryBean创建过程



### 2. pom.xml()

### 3.yaml配置文件

- yaml配置格式(严格控制字符串)

  ```
  #
  name: jiruxin
  
  # duixiang
  student:
    name: jirui${random.int}
    age: 3
  
  student2: {name: jirui,age: 3}
  # shuzu
  pets:
    - cat
    - dog
    - pig
  
  pets1: [cat,dog,pig]
  ```

- 多环境配置application.yaml
  1. ./config/*.yaml
  2. ./*.yaml
  3. rec/main/resources/*.yaml
  4. rec/main/resources/config/*.yaml

## springboot的事务

### 开启事务

> 编程式事务

在程序中通过编程来开启事务

```java

```







> 声明式事务

在需要开启事务的方法中添加注解 @Transactional

```java
@Override
@Transactional
public void laoda() {
    laoer();
    // 在这假设抛物异常
}

public void laoer() {
 
    // 在这假设抛出异常
}
```



### 事务传播

| 事务传播行为类型 | 外部不存在事务 | 外部存在事务                                                 | 使用方式                                         |
| ---------------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| REQUIRED         | 开启新的事务   | 融合到外部事务中                                             | 适用增删改查                                     |
| SUPPORTS         | 不开启新的事务 | 融合到外部事务中                                             | 适用查询                                         |
| REQUIRES_NEW     | 开启新的事务   | 不用外部事务，创建新的事务                                   | 适用内部事务和外部事务不存在业务关联的情况如日志 |
| NOT_SUPPORTED    | 不开启新的事务 | 不用外部事务                                                 | 不常用                                           |
| NEVER            | 不开启新的事务 | 抛出异常                                                     | 不常用                                           |
| MANDATORY        | 抛出异常       | 融合到外部事物中                                             | 不常用                                           |
| NESTED           | 开启新的事务   | 融合到外部事务中，SavePoint机制，外层影响内层，内层不会影响外层 | 不常用                                           |

### 事务失效

1. 调用自身的方法
2. 抛出的异常不是RuntimeException和Error
3. 用在非public方法上
4. 异常被自己try catch
5. @Transactiona标记的类或者标记方法所在的类没有被spring管理
6. 数据库不支持事务



## springboot自动配置

### java的SPI机制

1. 新建一个接口类和该接口的实现类

```java
// 接口类
public interface Play {
    void play();
}


// 接口实现类
public class APlayer implements Play{
    public void play() {
        System.out.println("----------I am Duzhilin---------");
    }
}
```



2. 在resource文件下建立**META-INF/services**这样的目录，在该目录下新建文件，该文件命名方式是**接口的全限定名**，在文件里面填写实现该类的全限定名。

![image-20220422154658016](https://gitee.com/jiruixin/images/raw/master/images/202204221548270.png)

3. 编写测试

```java
public class Main {
    public static void main(String[] args) {
        ServiceLoader<Play> service = ServiceLoader.load(Play.class);
        for (Play player : service) {
            player.play();
        }
    }
}
```



### SprinMVC

#### 1.WebMvcConfigurer

mvc的配置，是一个接口

#### 2.WebMvcConfigurerAdapter

WebMvcConfigurer的空实现，已经被弃用

#### 3.WebMvcConfigurationSupport

实现了WebMvcConfigurer的一些方法

#### 4.WebMvcAutoConfiguration

springboot中对mvc的自动配置，当容器中有WebMvcConfigurationSupport这个类时会失效

