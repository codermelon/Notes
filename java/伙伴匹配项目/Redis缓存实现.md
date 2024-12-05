# Redis缓存实现

## Redis数据结构

基本：

- String字符串类型：name:"panjie"
- List列表：names:["panjie","panjie1","panjie"]
- Set集合:names:["panjie","panjie1"]
- Hash: nameAge:{"panjie":1,"panjie2":2}
- Zset集合：names{panjie-1,panjie-2} 适合用来做排行榜

## 自定义序列化

```java
package com.yupi.yupao.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;

@Configuration
public class RedisTemplateConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        redisTemplate.setKeySerializer(RedisSerializer.string());
        return redisTemplate;
    }
}


```

 ## Java 操作 Redis

### Spring Data Redis

使用方式：

1）引入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.6.4</version>
</dependency>
```

2）配置redis

```yml
spring:
  # redis 配置
  redis:
    port: 6379
    host: localhost
    database: 0
```

##  实战

1）先注入RedisTemplate

```java
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
```

2）有缓存就直接读缓存

没有缓存就查询数据库并写入缓存

```java
@GetMapping("/recommend")
    public BaseResponse<Page<User>> recommendUsers(long pageSize, long pageNum, HttpServletRequest request) {
        User loginUser = userService.getLoginUser(request);
        String redisKey = String.format("yupao:user:recommend:%s", loginUser.getId());
        ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
        // 如果有缓存，直接读缓存
        Page<User> userPage = (Page<User>) valueOperations.get(redisKey);
        if(userPage!=null){
            return ResultUtils.success(userPage);
        }
        // 没有缓存，查数据库
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        userPage = userService.page(new Page<>(pageNum * pageSize,pageSize),queryWrapper);
        // 写缓存
        try {
            valueOperations.set(redisKey,userPage,30000, TimeUnit.MILLISECONDS);
        } catch (Exception e) {
            log.error("redis set key error",e);
        }
        return ResultUtils.success(userPage);
    }
```

## 缓存预热

解决的问题：

优点：

1. 第一个用户访问的还是很慢

缺点：

1. 预热的时机和时间如果错了，有可能缓存的数据不对或者太老
2. 占用额外的空间

### 怎么缓存预热

1. 定时触发
2. 模拟触发



### 定时任务实现

1. Spring Scheduler(spring boot默认整合了，推荐这种方式)
2. Quartz(独立于Spring存在的定时任务框架)
3. XL-Job之类的分布式任务调度平台（界面+sdk)

### 第一种方式

1. 主类开启@EnableScheduling
2. 给要定时执行的方法添加@Scheduling注解，通过cron表达式执行频率



## 控制定时任务的执行

方案：

1. 分离定时任务程序和主程序，只在1个服务器运行定时任务。（成本太大）
2. 写死配置，每个服务器都执行定时任务，但是只有ip符合配置的服务器才真实执行业务逻
   辑，其他的直接返回。成本最低；但是我们的ip可能是不固定的，把ip写的太死了
3. 动态配置，配置可以轻松的、很方便的更新。但是只有ip符合配置的服务器才真实执行业务逻辑
4. 分布式锁，只有抢到锁的服务器才能执行业务逻辑。坏处：增加成本；好处：不用手动配置，多少个服务器都一样



