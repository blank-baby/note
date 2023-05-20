



### 启动nacos服务端

管网下载nacos服务端，然后启动nacos，启动前需要做以下几个步骤

#### 1.建立数据库

在nacos文件中的conf中有一个nacos-mysql.sql文件，利用这个文件建立数据库

![image-20220308205641352](https://gitee.com/jiruixin/images/raw/master/images/image-20220308205641352.png)



#### 2.修改配置文件如下图所示

![image-20220308205823063](https://gitee.com/jiruixin/images/raw/master/images/image-20220308205823063.png)

#### 3. 启动nacos（单机启动）

修改startup.bat的MODE="standalone"保存

```cmd
set CUSTOM_SEARCH_LOCATIONS=file:%BASE_DIR%/conf/

rem 把这个修改为standalone
set MODE="standalone"
set FUNCTION_MODE="all"
set SERVER=nacos-server
set MODE_INDEX=-1
set FUNCTION_MODE_INDEX=-1
set SERVER_INDEX=-1
set EMBEDDED_STORAGE_INDEX=-1
set EMBEDDED_STORAGE=""
```

再点击startup.bat文件，如果是linux系统直接点击startup.sh文件。



### 注册微服务到nacos

1. 加载相应的jar包

```xml
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2. 在启动类上写上@EnableDiscoveryClient

3. 在配置文件中配置如下

   ```yml
   spring:
     application:
       name: springcloud-provider
   
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848
   ```

### nacos配置

1. 在nacos服务端新增配置信息，如下图所示

![image-20220309214950890](https://gitee.com/jiruixin/images/raw/master/images/image-20220309214950890.png)

2. 在客户端添加依赖

  ```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
  ```

3. 在springboot的资源文件中添加配置文件

   结构目录如下

![image-20220309215137204](https://gitee.com/jiruixin/images/raw/master/images/image-20220309215137204.png)



两个配置文件的内容为

application.yml配置文件

```yml
server:
  port: 3348
```

bootstrap.yml配置文件

```yml
spring:
  application:
    name: nacos-config

  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
      username: nacos
      password: nacos
      # 如果你在nacos上的配置文件不是Properties，在这需要指出配置文件的后缀
      # config:
        # file-extension: yaml
```

4. 在启动类的主类中测试

  ```java
@SpringBootApplication
public class Consumer3350 {

    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(Consumer3350.class, args);
        String userName = applicationContext.getEnvironment().getProperty("user.name");
        String userAge = applicationContext.getEnvironment().getProperty("user.age");
        System.err.println("user name :" +userName+"; age: "+userAge);
        // 系统可以打印出jiruixin和21  由此可见它获取到nacos服务器上的配置文件
    }
}
  ```



### 远程调用

#### Feign的使用

1. 添加依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 添加接口类

   ```java
   // @FeignClient(value = "应用名字")
   @FeignClient(value = "springcloud-provider")
   public interface StockClientFeign {
       // 和调用的微服务的方法一模一样
       @GetMapping(value = "/echo/{string}")
       public String echo(@PathVariable String string);
   
   }
   ```

3. 在启动类上添加注解@EnableFeignClients

  在需要调用的时候直接可以和以前一样使用

  ```java
@RestController
public class NacosController{

    // 和以前一样在这调用就行
    @Autowired
    private StockClientFeign stockClientFeign;

    @Value("${spring.application.name}")
    private String appName;

    @GetMapping("/echo/app-name")
    public String echoAppName(){
        return stockClientFeign.echo("appName");
    }
}
  ```

#### Dubbo使用



### 负载均衡

#### loadBalancerClient实现的负载均衡

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Consumer3346 {
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(Consumer3346.class,args);
    }
}

@RestController
public class NacosController{

    @Autowired
    private LoadBalancerClient loadBalancerClient;
    @Autowired
    private RestTemplate restTemplate;
    @Autowired
    private DiscoveryClient discoveryClient;

    @Value("${spring.application.name}")
    private String appName;

    @GetMapping("/echo/app-name")
    public String echoAppName(){
        // Access through the combination of LoadBalanceClient and RestTemplate
        // 第一种loadBalancerClient实现的负载均衡
        ServiceInstance serviceInstance = loadBalancerClient.choose("springcloud-provider");
        String path = String.format("http://%s:%s/echo/%s",serviceInstance.getHost(),serviceInstance.getPort(),appName);
        System.out.println("request path:" +path);
        return restTemplate.getForObject(path,String.class);
    }
}
```

#### 第二种

在RestTemplate加上注解@LoadBalanced

使用了@LoadBalanced有一个必须注意的点：**不能使用ip，应该使用应用名**

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Consumer3346 { 
    public static void main(String[] args) {
        SpringApplication.run(Consumer3346.class,args);
    }
    
    // Instantiate RestTemplate Instance
    // 在RestTemplate加上注解@LoadBalanced
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }  
}


@RestController
public class NacosController{

    @Autowired
    private RestTemplate restTemplate;

    @Value("${spring.application.name}")
    private String appName;

    @GetMapping("/echo/app-name")
    public String echoAppName(){
        
        // 不能使用ip，应该使用应用名
        return restTemplate.getForObject("http://springcloud-provider/echo/"+appName,String.class);
    }
}
```

###  更改负载均衡策略

官方文档指出：自定义的负载均衡配置类**不能放在 @componentScan 所扫描的当前包下及其子包下**，否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，也就是说我们达不到特殊化定制的目的了；

1. 自定义策略类如下图所示

```java
public class CustomerLoadBalancerRule extends AbstractLoadBalancerRule {
    public Server choose(ILoadBalancer lb, Object key){
        if (lb == null) {
            return null;
        } else {
            Server server = null;

            while(server == null) {
                if (Thread.interrupted()) {
                    return null;
                }

                List<Server> upList = lb.getReachableServers();
                List<Server> allList = lb.getAllServers();
                int serverCount = allList.size();
                if (serverCount == 0) {
                    return null;
                }

                server = (Server)upList.get(0);
                if (server == null) {
                    Thread.yield();
                } else {
                    if (server.isAlive()) {
                        return server;
                    }

                    server = null;
                    Thread.yield();
                }
            }

            return server;
        }
    }

    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {

    }

    @Override
    public Server choose(Object o) {
        return this.choose(this.getLoadBalancer(), o);
    }
}


```

2. 设置的层级结构如下图所示。 用配置类把刚才的策略类定义成Bean。也可以把自带的策略类注册成bean。

  

![image-20220309105159938](https://gitee.com/jiruixin/images/raw/master/images/image-20220309105159938.png)

3. 在启动类上加上注解

```java
@RibbonClient(name = "应用者名称",configuration = 配置类的类文件)
// 这是我的配置
@RibbonClient(name = "springcloud-provider",configuration = MyselfRule.class)
```

### Sentinel

启动sentinel控制台

```bash
java -Dserver.port=8849 -Dcsp.sentinel.dashboard.server=localhost:8849 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
```

### 注册微服务到Sentinel

1. 加载相应的jar包

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

2. 在配置文件中添加如下配置

```yml
spring:
  application:
    name: springcloud-sentienl
    sentinel:
      transport:
        port: 9999 #写一个没有用过的端口号
        dashboard: localhost:8849  # 这是控制台的端口号
```

注意点：**Sentinel是懒加载，需要别人调用它才会注册到Sentinel。**



> @SentinelResource("资源名")

指定资源在控制台中的名字

```java
@RestController
public class EchoController {

    @GetMapping(value = "/echo/{string}")
    // 指定了在控制台的名字
    @SentinelResource("Sentinel-provider-1")
    public String echo(@PathVariable String string) {
        return "Hello sentinel " + string + "1111";
    }
}
```

#### Sentinel控制台

> 油脂类型

QPS: 每秒请求数量达到阈值时限流

线程数：线程数达到阈值时限流

![image-20220311100545144](https://gitee.com/jiruixin/images/raw/master/images/image-20220311100545144.png)



> 流控效果

直接： 当指定资源达到阈值直接限流

关联： **当关联的资源达到阈值限流指定的资源**（适合做资源的让步）

​       举例：指定资源为A,关联资源为B，当访问B请求达到到阈值时，对A资源进行限流

![image-20220311101055158](https://gitee.com/jiruixin/images/raw/master/images/image-20220311101055158.png)



> 流控效果

- 快速失败（默认）: 直接失败，抛出异常，不做任何额外的处理，是最简单的效果 
- Warm Up：它从开始阈值到最大QPS阈值会有一个缓冲阶段，一开始的阈值是最大QPS阈值的 1/3，然后慢慢增长，直到最大阈值，适用于将突然增大的流量转换为缓步增长的场景。
- 排队等待：让请求以均匀的速度通过，单机阈值为每秒通过数量，其余的排队等待； 它还会让设 置一个超时时间，当请求超过超时间时间还未处理，则会被丢弃



> sentinel配置到nacos

sentinel读取不到nacos的配置，原来是在配置文件中的用户名和密码没有生效，升级2.2.1到2.2.2版本。





### GateWay

> 普通gateway

#### 1.添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

#### 2.添加配置文件

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: provider
          uri: http://localhost:3344
          predicates:
            - Path=/**
```

> 整合nacos

#### 1.添加依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

#### 2.添加配置文件



```yml

spring:
  application:
    name: springcloud-gateway

  cloud:
    gateway:
      routes:
        - id: provider
          # 直接使用nacos的注册名字
          uri: lb://springcloud-provider
          predicates:
            - Path=/**

    nacos:
      discovery:
        server-addr: localhost:8848
        username: nacos
        password: nacos
       

```

直接开启自动注册

```yml
server:
  port: 3351

spring:
  application:
    name: springcloud-gateway

  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
    nacos:
      discovery:
        server-addr: localhost:8848
        username: nacos
        password: nacos

    sentinel:
      transport:
        port: 9999
        dashboard: localhost:8849

```

注意点：**在调用的时候需要在路由的端口后面加上应用的名字**

举例：

原来的路由是：http://localhost:3351/echo/11

现在的路由是：http://localhost:3351/springcloud-provider/echo/11



#### 3.主类添加注解@EnableDiscoveryClient

> gateway整合sentinel

#### 1.添加依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>

<!--整合sentinel和gateway-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>
```

#### 2.编写配置文件

```yml
spring:
  application:
    name: springcloud-gateway
  cloud:
    gateway:
      routes:
        - id: provider
          uri: lb://springcloud-provider
          predicates:
            - Path=/**
    nacos:
      discovery:
        server-addr: localhost:8848
        username: nacos
        password: nacos

    sentinel:
      transport:
        port: 9999
        dashboard: localhost:8849
```

