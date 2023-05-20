# JavaBean（实体类）

Jav aBean有特定的写法

- 必须要有一个无参构造
- 属性必须私有化
- 必须对应的get/set方法

一般用来和数据的字段做ORM

- 表              类
- 字段          属性
- 行记录      对象

## session

1. 用户第一次请求服务器时，服务器端会生成一个sessionid
2. 服务器端将生成的sessionid返回给客户端，通过set-cookie
3. 客户端收到sessionid会将它保存在cookie中，当客户端再次访问服务端时会带上这个sessionid
4. 当服务端再次接收到来自客户端的请求时，会先去检查是否存在sessionid，不存在就新建一个sessionid重复1,2的流程，如果存在就去遍历服务端的session文件，找到与这个sessionid相对应的文件，文件中的键值便是sessionid，值为当前用户的一些信息
5. 此后的请求都会交换这个 Session ID，进行有状态的会话。

