## spring源码

### 1. Spring扩展

####  1.bean生命周期的扩展

> 对属性资源的扩展，还没弄明白

```java
@Override
protected void initPropertySources(){
    
}
```





> BeanDefinitionRegistryPostProcessor

实现对BeanDefinition的处理，实现的功能

- 注册BeanDefinition
- 移除BeanDefinition
- 是否包含BeanDefinition
- 获取所有的BeanDefinition的名称
- 修改BeanDefinition

> BeanFactoryPostProcessor

主要是对BeanFactory的修改，不能注册BeanDefinition



> ImportBeanDefinitionRegistrar

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

> InstantiationAwareBeanPostProcessor

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



> @PostConstruct



> @Value  @Autowired

在AutowiredxxxBeanFactoryPostPocessor

> 





### 2.  springboot的包路径

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

