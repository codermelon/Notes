# Session共享

思考：为什么服务器A登陆后，请求发到服务器B，不认识该用户

原因：

1. 用户在A登录，所以session（用户登录信息）存在了A上
2. 结果请求B时，B没有用户信息，所以不认识

![image-20241008204650395](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20241008204650395.png)

# 如何共享存储

核心思想：把数据放到同一个地方去管理

1. Redis
2. MySQL
3. 文件服务器



# Session共享实现

(1)引入redis，能够操作redis

```xml

<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.6.4</version>
</dependency>

```

(2)引入spring-session和redis的整合，自动将session存储到redis中

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.session/spring-session-data-redis -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
    <version>2.6.3</version>
</dependency>

```

(3)修改spring-session存储配置

```yml
  # session 失效时间
  session:
    timeout: 86400
    store-type: redis
```

