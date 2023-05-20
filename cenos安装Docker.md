## 1.cenos安装Docker

### 1.删除原来的docker

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 2.安装yum-utils

```shell
sudo yum install -y yum-utils
```

### 3.安装阿里云镜像

```shell
 sudo yum-config-manager \
    --add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 4.安装Docker

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

### 5.启动Docker

systemctl start docker

### 6.配置阿里云镜像加速

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://71q1eamg.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 2.开机启动docker和镜像

### 1.设置开机启动docker

```bash
# 启动docker
systemctl start docker

# 设置docker自动启动
systemctl enable docker.service

# 检查自动启动是否成功
systemctl list-unit-files | grep enable | grep docker
# 出现这个代表成功
docker.service                                enabled 
```

### 2.设置docker的镜像自动启动

```bash
# 开启镜像
docker start d5c440e6d44f

# 设置镜像自动启动
docker update --restart=always d5c440e6d44f
```



## 3.docker部署mysql集群（一主一从）

### 1.开启主机mysql镜像

```bash
docker run --name mysql-master -e MYSQL_ROOT_PASSWORD=123456  -p 3306:3306 --privileged=true -v /home/mysql/mysql-master/log:/var/log/mysql -v /home/mysql/mysql-master/data:/var/lib/mysql -v /home/mysql/mysql-master/conf:/etc/mysql/conf.d -d mysql:5.7

```

### 2.进入master conf 目录 新建my.cnf

```bash
# my.cnf 内容
[mysqld] 
## 设置server_id，同一局域网中需要唯一 
server_id=101
## 指定不需要同步的数据库名称 
binlog-ignore-db=mysql
## 开启二进制日志功能 
log-bin=mall-mysql-bin
## 设置二进制日志使用内存大小（事务） 
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed,statement,row） 
binlog_format=mixed
## 二进制日志过期清理时间。默认值为0，表示不自动清理。 
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。 
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致 
slave_skip_errors=1062
```



###  3.启动镜像

```bash
docker restart mysql-master
```



### 4. 进入镜像

```bash
docker exec -it mysql-master /bin/bash
```

### 5.master 容器实例内创建用户同步数据

```bash
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```



###  6. 查看主机状态

```bash
show master status;
```

![image-20220509221343683](https://gitee.com/jiruixin/images/raw/master/images/202205092213779.png)



### 7.开启从机镜像

```bash
docker run --name mysql-slave -e MYSQL_ROOT_PASSWORD=123456  -p 3307:3306 --privileged=true -v /home/mysql/mysql-slave/log:/var/log/mysql -v /home/mysql/mysql-slave/data:/var/lib/mysql -v /home/mysql/mysql-slave/conf:/etc/mysql/conf.d -d mysql:5.7
```

### 8.在宿主机从数据库配置文件目录创建my.cnf

```bash
# 文件内容
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql 
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
log-bin=mysql-slave1-bin
## 设置二进制日志使用内存大小（事务
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed,statement,row
binlog_format=mixed
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。 
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致 
slave_skip_errors=1062
## relay_log配置中继日志 
relay_log=mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志 
log_slave_updates=1
## slave设置为只读（具有super权限的用户除外） 
read_only=1
```



### 9. 启动从机镜像

```bash
docker restart mysql-slave
```

### 10 进入到从机mysql

```bash
docker exec -it mysql-slave /bin/bash
mysql -uroot -p123456
```



### 11. 在从机中配置主的信息

```bash
change master to master_host='192.168.0.201', master_user='slave', master_password='123456', master_port=3306, master_log_file='master-bin.000008', master_log_pos=617, master_connect_retry=30;

# 参数说明
master_host：主数据库的IP地址； 
master_port：主数据库的运行端口； 
master_user：在主数据库创建的用于同步数据的用户账号； 
master_password：在主数据库创建的用于同步数据的用户密码； 
master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数； 
master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数； 
master_connect_retry：连接失败重试的时间间隔，单位为秒。 

```

### 12 在从数据库中开启主从同步

```bash
start slave;
```

### 13查看主从同步状态

```bash
show slave status \G;
```

![image-20220509221950661](https://gitee.com/jiruixin/images/raw/master/images/202205092219653.png)

### 到这完成主从复制

## 4.springboot的和MySQL集群

### 1.springboot自带的数据源路由完成

#### 1.配置多个数据源

```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    druid:
      master:
        enabled: true
        url: jdbc:mysql://127.0.0.1:3306/health_management?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
        username: root
        password: 123456
      slave:
        enabled: true
        url: jdbc:mysql://192.168.0.201:3306/health_management?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
        username: root
        password: 123456

      initialSize: 5
      minIdle: 10
      maxActive: 20
      maxWait: 6000
      # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
      timeBetweenEvictionRunsMillis: 60000
      # 配置一个连接在池中最小生存的时间，单位是毫秒
      minEvictableIdleTimeMillis: 300000
      # 配置一个连接在池中最大生存的时间，单位是毫秒
      maxEvictableIdleTimeMillis: 900000
      # 配置检测连接是否有效
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      pool-prepared-statements: true
      max-pool-prepared-statement-per-connection-size: 20
      filters: stat,wall
      stat-view-servlet:
        enabled: true
        login-username: admin
        login-password: admin
        url-pattern: /druid/*
        allow: localhost

```

#### 2. 增加配置类

```java
package com.health.manage.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;
import com.alibaba.druid.spring.boot.autoconfigure.properties.DruidStatProperties;
import com.alibaba.druid.util.Utils;
import com.health.manage.database.DataSourceType;
import com.health.manage.database.DruidProperties;
import com.health.manage.database.DynamicDataSource;


import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.servlet.*;
import javax.sql.DataSource;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

/**
 * druid 配置多数据源
 *
 * @author ruoyi
 */
@Configuration
public class DruidConfig
{
    @Bean
    @ConfigurationProperties("spring.datasource.druid.master")
    @ConditionalOnProperty(prefix = "spring.datasource.druid.master", name = "enabled", havingValue = "true")
    public DataSource masterDataSource(DruidProperties druidProperties)
    {
        DruidDataSource dataSource = DruidDataSourceBuilder.create().build();
        return druidProperties.dataSource(dataSource);
    }

    @Bean
    @ConfigurationProperties("spring.datasource.druid.slave")
    @ConditionalOnProperty(prefix = "spring.datasource.druid.slave", name = "enabled", havingValue = "true")
    public DataSource slaveDataSource(DruidProperties druidProperties)
    {
        DruidDataSource dataSource = DruidDataSourceBuilder.create().build();
        return druidProperties.dataSource(dataSource);
    }

    @Bean(name = "dynamicDataSource")
    @Primary
    public DynamicDataSource dataSource(DataSource masterDataSource,DataSource slaveDataSource)
    {
        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put(DataSourceType.MASTER.name(), masterDataSource);
        targetDataSources.put(DataSourceType.SLAVE.name(), slaveDataSource);
        return new DynamicDataSource(masterDataSource, targetDataSources);
    }

}

```

#### 3.添加配置类

![image-20220509181747545](https://gitee.com/jiruixin/images/raw/master/images/202205091817999.png)

#### 4.使用Aop来切换数据源

添加到方法上就可以动态切换数据源

```java
@DataSource(DataSourceType.SLAVE)
```



###  2.shardingjdbc完成数据源切换

#### 1.yml的配置文件

```yml
server:
  port: 8090
  servlet:
    context-path: /cluster

spring:
  shardingsphere:
    props:
      sql:
        show: true
    datasource:
      names: master,slave   #对应下面主从库
      master:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.0.201:3306/health_management?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
        username: root
        password: 123456
      slave:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.0.201:3307/health_management?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
        username: root
        password: 123456
    config:
      masterslave:
        load-balance-algorithm-type: round_robin  #负载 轮询，当你有多个从库或者主库时
        name: ms
        master-data-source-name: master           #设置主库
        slave-data-source-names: slave            #设置从库

  redis:
    cluster:
      nodes: 192.168.0.203:6379,192.168.0.203:6380,192.168.0.203:6381,192.168.0.203:6382,192.168.0.203:6383,192.168.0.203:6384
      max-redirects: 3

    password: 123456
    # 连接超时时间
    timeout: 6s
    jedis:
      pool:
        # 连接池中的最小空闲连接
        min-idle: 0
        # 连接池中的最大空闲连接
        max-idle: 8
        # 连接池的最大数据库连接数
        max-active: 8
        # #连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1ms

```



#### 2.pom文件的导包

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>4.0.0-RC1</version>
</dependency>
```

>如果遇到如下图错误

![image-20220512180411856](https://gitee.com/jiruixin/images/raw/master/images/202205121804494.png)

在pom文件中添加如下依赖

```xml

<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>28.0-jre</version>
</dependency>
```



> 如果遇到如下问题

![image-20220512180721208](https://gitee.com/jiruixin/images/raw/master/images/202205121807586.png)

降低sharding-jdbc-spring-boot-starter报的版本



#### 3.正常使用就会自动读写分离



## 5.docker 部署redis

```bash
# 设置linux的时间
timedatectl set-timezone Asia/Shanghai
```

### redis基本文档

#### 1.redis基本命令

```bash
连接redis
redis-cli

验证密码
auth 密码

查看信息
info replication

查看所有值
keys *
```

#### 2.redis的持久化

>  RDB持久化

在制定的时间间隔内，执行制定次数的写操作，将redis的数据保存到硬盘上

redis.conf配置文件

```bash
# 900秒内有1次更改就保存内容到硬盘
save 900 1
```



> AOF持久化

以持久化的方式把写操作记录到日志里，每次redis重启时会根据日志的写操作再执行一下

redis.conf配置文件（默认不开启）

```bash
appendonly yes
```







### 2.集群和主从区别

> 集群模式

redis的主从模式，指的是针对多台redis实例时候，只存在一台主服务器master，提供读写的功能，同时存在依附在这台主服务器的从服务器slaver，只提供读服务，且数据和主服务器保持一致。主服务器只能有一台，从服务器可以有多台，而且可以存在级联模式(从服务器下面也挂载从服务器)。主从的存在是为了分散访问量，提高访问可读性，同事保证数据的冗余和备份。

> 主从模式

redis集群模式，指的是针对多个redis实例，去中心化，去中间件，集群中的每个节点都是平等的关系，都是对等的，每个节点都保存各自的数据和整个集群的状态，并且每个节点都和其他所有节点连接，而且这些连接保持活跃，这样就保证了我们只需要连接集群中的任意一个节点，就可以获取到其他节点的数据。

redis集群没有统一的ip入口，客户端与redis节点直连,不需要中间proxy层，客户端访问任意一个节点，都可以进入集群，访问集群的所有数据。节点内部使用PING-PONG机制来互相通信，当集群添加一个节点的时候，只需要连接集群中任意一个节点即可，不需要连接所有节点。

当超过半数的master节点检测某个master节点失效时(通行超时)，该节点才会失效，进行故障转移。因此必须要有3个以上的master才可以创建出集群。

如果一个master进行故障转移的时候，发现没有slaver来使用，那么整个集群fails。当整个集群有一半的master出现故障，无论有没有slaver，整个集群fails。

通过 redis-trib.rb create --replicas 1来创建集群。

**哈希槽：**

哈希槽的存在，是为了能快速找到key所在的节点。一个redis集群包含2^16=16384个哈希槽，会分配给各个节点。可以通过槽的分配来控制不同节点的数据量和请求数。哈希槽通过顺时针来分片，即一个节点顺时针到下一个几点之间的槽属于下一个节点。

每一个key，通过公式slot=CRC16（key）/16384来计算属于哪个槽。

扩容的时候，只需要动一个节点的数据即可。eg.A->B(顺时针)，中间插入C节点，那么只修要迁移B的数据，重新分配到B和C中即可。

每一个redis实例节点都会保持元数据对应的哈希槽信息，数据会不断的同步。



### 3.redis的主从复制

#### 1.redis的配置的文件

```bash
# bind 127.0.0.1  #注释掉这部分，添加bind 0.0.0.0 
# 使redis可以外部访问
bind 0.0.0.0 

# 端口号
port 6379 

# 将本机访问保护模式设置no
protected-mode no  

# aof持久化
appendonly yes 

# 开启集群
cluster-enabled no

# 配置文件 
cluster-config-file nodes.conf 

# 集群超时时间
cluster-node-timeout 15000 

slaveof 192.168.0.201 6379

masterauth 123456

```

#### 2.创建三个redis的镜像（一从二主）

> 创建镜像命令

```bash
docker run -d -it -p 6379:6379 -v /home/redis/redis-6379/conf/redis.conf:/etc/redis/redis-6379/redis.conf -v /home/redis/redis-6379/data:/data/redis-6379 --privileged=true --restart always --name redis-6379 redis redis-server /etc/redis/redis-6379/redis.conf --requirepass 123456;

docker run -d -it -p 6380:6379 -v /home/redis/redis-6380/conf/redis.conf:/etc/redis/redis-6380/redis.conf -v /home/redis/redis-6380/data:/data/redis-6380 --privileged=true --restart always --name redis-6380 redis redis-server /etc/redis/redis-6380/redis.conf --requirepass 123456

docker run -d -it -p 6381:6379 -v /home/redis/redis-6381/conf/redis.conf:/etc/redis/redis-6381/redis.conf -v /home/redis/redis-6381/data:/data/redis-6381 --privileged=true --restart always --name redis-6381 redis redis-server /etc/redis/redis-6381/redis.conf --requirepass 123456

```



> 主机和从机配置

```bash
# 主机配置
bind 0.0.0.0
port 6379
protected-mode no
appendonly yes
cluster-enabled no
cluster-config-file nodes.conf
cluster-node-timeout 15000

masterauth 123456


# 从机配置
bind 0.0.0.0
port 6379
protected-mode no
appendonly yes
cluster-enabled no
cluster-config-file nodes.conf
cluster-node-timeout 15000
slaveof 192.168.0.201 6379
masterauth 123456

slave-announce-ip 192.168.0.201
slave-announce-port 6380
```





#### 3.创建哨兵

> 创建哨兵命令

```bash
docker run -p 26379:26379 --restart=always --name sentinel-26379 -v/home/redis/redis-6379/conf/sentinel-26379.conf:/etc/redis/sentinel.conf -v /home/redis/redis-6379/data/sentinel-26379-data:/data -d redis redis-sentinel /etc/redis/sentinel.conf

docker run -p 26380:26380 --restart=always --name sentinel-26380 -v/home/redis/redis-6380/conf/sentinel-26380.conf:/etc/redis/sentinel.conf -v /home/redis/redis-6380/data/sentinel-26380-data:/data -d redis redis-sentinel /etc/redis/sentinel.conf

docker run -p 26381:26381 --restart=always --name sentinel-26381 -v/home/redis/redis-6381/conf/sentinel-26381.conf:/etc/redis/sentinel.conf -v /home/redis/redis-6381/data/sentinel-26381-data:/data -d redis redis-sentinel /etc/redis/sentinel.conf
```

> 哨兵配置

```bash
port 26380

dir "/data"

logfile "sentinel-26380.log"

sentinel monitor mymaster 192.168.0.201 6379 2

sentinel down-after-milliseconds mymaster 10000

sentinel failover-timeout mymaster 60000

sentinel auth-pass mymaster 123456               
```



### 4.redis集群

> 判断当前节点是否下线

整个集群中的所有的master节点参与，超过半数的master与该节点通信超时就算下线



> 一下两种情况将导致整个集群不可用

- 某一个master集群下线，并且该主节点没有slave节点

- 集群中超过半数的master节点下线



#### 1.配置文件

```bash
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes-6383.conf
cluster-node-timeout 5000
cluster-announce-ip 192.168.0.203
cluster-announce-port 6383
cluster-announce-bus-port 16383
appendonly yes
masterauth 123456
```



#### 2.创建镜像

```bash
docker run -d -it -p 6379:6379 -p 16379:16379 -v /home/redis/redis-6379/conf/redis.conf:/etc/redis/redis-6379/redis.conf -v /home/redis/redis-6379/data:/data/redis-6379 --privileged=true --restart always --name redis-6379 redis redis-server /etc/redis/redis-6379/redis.conf --requirepass 123456;

docker run -d -it -p 6382:6379 -v /home/redis/redis-6382/conf/redis.conf:/etc/redis/redis-6382/redis.conf -v /home/redis/redis-6382/data:/data/redis-6382  --restart always --name redis-6382 redis redis-server /etc/redis/redis-6382/redis.conf --requirepass 123456;


docker run -d -it -p 6383:6379 -v /home/redis/redis-6383/conf/redis.conf:/etc/redis/redis-6383/redis.conf -v /home/redis/redis-6383/data:/data/redis-6383  --restart always --name redis-6383 redis redis-server /etc/redis/redis-6383/redis.conf --requirepass 123456;

docker run -d -it -p 6384:6379 -v /home/redis/redis-6384/conf/redis.conf:/etc/redis/redis-6384/redis.conf -v /home/redis/redis-6384/data:/data/redis-6384  --restart always --name redis-6384 redis redis-server /etc/redis/redis-6384/redis.conf --requirepass 123456;

docker run -d -it -p 6384:6379 -v /home/redis/redis-6384/conf/redis.conf:/etc/redis/redis-6384/redis.conf -v /home/redis/redis-6384/data:/data/redis-6384  --restart always --name redis-6384 redis redis-server /etc/redis/redis-6384/redis.conf --requirepass 123456;
```

#### 3.进入任意一个redis的镜像

```bahs
redis-cli --cluster create 192.168.0.203:6379 192.168.0.203:6380 192.168.0.203:6381 192.168.0.203:6382 192.168.0.203:6383 192.168.0.203:6384 --cluster-replicas 1 -a 123456
```



####  4.进入集群的命令

在进入集群时记得-c

```bash
redis-cli -c -a 123456
```



## 6.springboot和redis集群

### 1.添加pom文件依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 2.编写配置文件

```yml
spring:
  redis:
    cluster:
      nodes: 192.168.0.203:6379,192.168.0.203:6380,192.168.0.203:6381,192.168.0.203:6382,192.168.0.203:6383,192.168.0.203:6384
      max-redirects: 3

    password: 123456
    # 连接超时时间
    timeout: 6s
    jedis:
      pool:
        # 连接池中的最小空闲连接
        min-idle: 0
        # 连接池中的最大空闲连接
        max-idle: 8
        # 连接池的最大数据库连接数
        max-active: 8
        # #连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1ms
```

### 3.编写测试文件

```java
@RestController
public class UserController {

    @Autowired
    private RedisTemplate redisTemplate;

 	// baocunredis的值
    @GetMapping("/redis/save/{key}/{value}")
    public AjaxResult isFilled(@PathVariable("key")String key, @PathVariable("value")String value){
        AjaxResult ajax = AjaxResult.success();
        redisTemplate.opsForValue().set(key, value);
        ajax.put("data",redisTemplate.opsForValue().get(key));
        return ajax;
    }

    // 获取redis的值
    @GetMapping("/redis/get/{key}")
    public AjaxResult isFilled(@PathVariable("key")String key){
        AjaxResult ajax = AjaxResult.success();
        ajax.put("data",redisTemplate.opsForValue().get(key));
        return ajax;
    }
}
```

## 7.Rsp项目部署

拷贝命令

```bash
拷贝命令
scp root@49.233.2.176:/home/myElasticSearch.tar /home/

docker save -o 打包名字.tar 镜像名字:版本号
docker load -i 打包名字.tar
```



### 1.创建数据库

```bash
docker run --name mysql -e MYSQL_ROOT_PASSWORD=jrx123654@qq  -p 3306:3306 -v /home/mysql/logs:/var/log/mysql -v /home/mysql/data:/var/lib/mysql -v /home/mysql/conf:/etc/mysql/conf.d -d mysql:5.7 
```



### 2.创建elasticsearch镜像

#### 1.运行elasticsearch镜像

```bash
docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.14.2
```

#### 2.拷贝分词器到镜像的插件中

```bash
docker cp elasticsearch-analysis-ik-7.14.2.zip elasticsearch:/usr/share/elasticsearch/plugins
```

#### 3.配置elasticsearch

```yml
cluster.name: "docker-cluster"
network.host: 0.0.0.0

加上这两句话可以跨域
http.cors.enabled: true
http.cors.allow-origin: "*"
```

#### 4.重启elasticsearch镜像

#### 5.用head创建表结构rsp1

#### 6.用程序导入数据

### 3.上传前端和后端程序

#### 1.配置nginx

#### 2.启动docker-compose

```bash
docker-compose up -d
```







