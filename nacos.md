## nacos


### 配置文件
#### 配置文件优先级
```yaml
读取主配置文件的规则（优先级依次往下）
1. ${prefix}-${spring.profiles.active}.${file-extension} 
2. ${prefix}.${file-extension}
3. ${prefix} 

读取其他配置文件
4. extension-configs 
5. shared-configs
```
#### 注意点

1. ${prefix}-${spring.profiles.active}.${file-extension}
prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。

2. spring.profiles.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profiles.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}

3. file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。

4. 默认配置文件读取按上面规则查找nacos的配置文件，配置文件如果不是properties就必须配置