## nacos


配置文件读取
${prefix}-${spring.profiles.active}.${file-extension} > ${prefix}.${file-extension} > ${prefix} > extension-configs > shared-configs

${prefix}-${spring.profiles.active}.${file-extension}
prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。

spring.profiles.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profiles.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}

file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。

默认配置文件读取按上面规则查找nacos的配置文件，配置文件如果不是properties就必须配置

extension-configs > shared-configs配置文件如果是别的格式需要在文件后加上这个后缀

