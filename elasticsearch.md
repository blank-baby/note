-冯Elasticsearch

## 基本概念

> 索引(名词)

一个 *索引* 类似于传统关系数据库中的一个 *数据库* ，是一个存储关系型文档的地方。 *索引* (*index*) 的复数词为 *indices* 或 *indexes*



> 索引(动词)

*索引一个文档* 就是存储一个文档到一个 *索引* （名词）中以便被检索和查询。这非常类似于 SQL 语句中的 `INSERT` 关键词，除了文档已存在时，新文档会替换旧文档情况之外。

>文档

一个索引中的一个数据理解为文档。相当于一个数据库中的一条数据

> 对象 和文档

通常情况下，我们使用的术语 *对象* 和 *文档* 是可以互相替换的。不过，有一个区别： 一个对象仅仅是类似于 hash 、 hashmap 、字典或者关联数组的 JSON 对象，对象中也可以嵌套其他的对象。 对象可能包含了另外一些对象。在 Elasticsearch 中，术语 *文档* 有着特定的含义。它是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。

**字段的名字可以是任何合法的字符串，但 不可以包含英文句号(.)。**

> 文档元数据

 三个必须的元数据元素如下：

- **`_index`**

  文档在哪存放

  这个名字必须小写，不能以下划线开头，不能包含逗号

- **`_type`**

  文档表示的对象类别

  一个 `_type` 命名可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号， 并且长度限制为256个字符

- **`_id`**

  文档唯一标识

  *ID* 是一个字符串，当它和 `_index` 以及 `_type` 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创建一个新的文档，要么提供自己的 `_id` ，要么让 Elasticsearch 帮你生成





## 分词器

### 1.es的分词介绍

> 创建索引时进行的分词

在索引里放入数据时，es会对这些数据进行分词，然后进行倒序排序记录。如果分词器不设置的话就是默认的standard分词器。分词器类型在创建索引时可以进行设置



> 在对某个词进行搜索进行的分词

在对某个词进行match搜索时，es会对这个词进行分组。如果不设置分词器种类，会采用写数据时用的分词器种类。



### 2.分词器类型

> standard
>
> 单词按每个单词分组。汉字按每个汉字分组，不会进行词语的组合分组。



> ik_smart
>
> 按正常的词语分组



> ik_max_word
>
> 对词进行最细粒度的分词



### 3.检测分词

> 检测分词类型的分词效果

用户可以想要查看某个类型的分词器对某个词的分词效果，可以使用如下命令。

```
GET _analyze
{
  "analyzer": "分词的类型", 
  "text":"要检测的内容"
}
```

返回的结果是

```
{
  "tokens" : [
    {
      "token" : "中",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "国",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "共",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "产",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "党",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "<IDEOGRAPHIC>",
      "position" : 4
    }
  ]
}
```



> 检测某个索引的字段的分词效果

如果想要查看某个字段在写入数据时进行的分词可采用如下命令

```
GET /索引名字/_analyze
{
  "field": "该索引的字段",
  "text": "写入该索引的内容"
}
```





## 索引

### 1.创建索引

命令格式： put  /索引名字

setting 设置的该索引的分词器

mappings 设置的索引的字段和类型

```
PUT /rsp
{
    "settings" : {
        "index" : {
            "analysis.analyzer.default.type": "ik_max_word"
        }
    },
    "mappings": {
        "properties": {
          "name":{
            "type":"text"
          },
          "des":{
            "type":"keyword"
          }
        }
  	}
}
```



## docker部署es

### 1. 拉取镜像

```bash
docker pull elasticsearch:7.14.2

启动镜像
docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx1024m" elasticsearch:7.14.2

```



> linux配置es

```bash
查看容器的日志
docker logs -f 容器id

进入docker容器
docker exec -it 容器id /bin/bash

docker cp /复制的文件 容器id:/容器位置
echo 'http.cors.enabled: true' >> elasticsearch.yml
echo 'http.cors.allow-origin: "*"' >> elasticsearch.yml

//创建索引
curl -H "Content-Type: application/json" -XPUT http://localhost:9200/rsp1 -d'{
    "settings" : {
        "index" : {
            "analysis.analyzer.default.type": "ik_max_word"
        }
    }
   
}'

```

springboot 打包项目

```
mvn clean package -Dmaven.test.skip=true


docker-compose up -d
```

nginx 启动vue  配置跨域请求

```
location / {
	root   /usr/share/nginx/html;
	try_files $uri $uri/ /index.html last;

	index  index.html index.htm;
}

location /prod-api {
	rewrite  ^.+/prod-api/?(.*)$ /$1 break;
	proxy_pass http://49.233.2.176:8085/rsp;
}

```



