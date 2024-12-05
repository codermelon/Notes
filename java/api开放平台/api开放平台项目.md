# 模拟接口项目

项目名称：yuapi-interface

提供三个不同种类的模拟接口：

1. GET接口
2. POST接口（url传参）
3. POST接口（Restful）

## 调用接口

几种HTTP调用方式：

1. HttpClient
2. RestTemplate
3. 第三方库(OKHTTP、Hutool)

Hutool：https://hutool.cn/docs/#/



### 测试

```java
@RestController
@RequestMapping("/name")
public class NameController {

    @GetMapping("/")
    public String getNameByGet(String name){
        return "GET 你的名字是" + name;
    }

    @PostMapping("/")
    public String getNameByPost(@RequestParam String name){
        return "POST 你的名字是" +name;
    }

    @PostMapping("/user")
    public String getUsernameByPost(@RequestBody User user){
        return "POST 你的名字是" +user.getUsername();
    }
}

```

```java
public class YuApiClient {

    public String getNameByGet(String name){
        //可以单独传入http参数，这样参数会自动做URL编码，拼接在URL中
        HashMap<String, Object> paramMap = new HashMap<>();
        paramMap.put("name",name);
        String result= HttpUtil.get("http://localhost:8123/api/name/", paramMap);
        System.out.println(result);
        return result;
    }


    public String getNameByPost(@RequestParam String name){
        //可以单独传入http参数，这样参数会自动做URL编码，拼接在URL中
        HashMap<String, Object> paramMap = new HashMap<>();
        paramMap.put("name",name);
        String result= HttpUtil.post("http://localhost:8123/api/name/", paramMap);
        System.out.println(result);
        return result;
    }


    public String getUsernameByPost(@RequestBody User user){
        String json = JSONUtil.toJsonStr(user);
        HttpResponse httpResponse = HttpRequest.post("http://localhost:8123/api/name/user")
                .body(json)
                .execute();
        System.out.println(httpResponse.getStatus());
        System.out.println(httpResponse.body());
        return httpResponse.body();

    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        YuApiClient yuApiClient = new YuApiClient();
        String result1 = yuApiClient.getNameByGet("ppanjie");
        String result2 = yuApiClient.getNameByPost("panjie");
        User user = new User();
        user.setUsername("panjie");
        String result3 = yuApiClient.getUsernameByPost(user);
        System.out.println(result1);
        System.out.println(result2);
        System.out.println(result3);
    }
}
```

# API签名认证

本质：

1. 签发签名
2. 使用签名（校验签名）

为什么需要？

1. 保证安全性，不能随便一个人调用
2. 适用于无需保存登录态的场景。只认签名，不关注用户登录态

## 签名认证实现

通过http request header头传递参数。

1.参数1：accessKey：调用的标识userA，userB

- 服务器根据accessKey识别是那一个用户发起的请求，便于加载用户的密钥



2.参数2：secretKey：密钥 （**该参数不能放到请求头中**）

不能把密钥直接在服务器之间传递，有可能被拦截

- 配合用户请求的参数，通过签名算法生成签名



3.参数3：用户请求参数

- 用户需要传递给服务器的业务数据，例如查询条件



4.数4：sign

加密方式：对称加密、非对称加密、md5加密

用户参数+密钥 => 签名生成算法 (MD5、HMac、Sha1) => 不可解密的值

abc + abcdefgh => sasdadsadasdada

怎么知道这个签名对不对？

**服务器端用一模一样的参数和算法去生成签名，只要和用户传的一致，就表示一致。**



**怎么防重放？**

5.参数5：加nonce随机数，只能用一次

服务端要保存用过的随机数，用于验证是否已经使用过



6.参数6：加timestamp时间戳，校验时间戳是否过期

- 防止请求长期有效，从而限制攻击者重放合法请求



# 开发一个简单易用的SDK

项目名：yuapi-client-sdk

## 为什么需要Starter？

开发者只需要关心调用哪些接口、传递哪些参数，就跟调用自己写的代码一样简单。

开发starter的好处：开发者引入之后，可以直接在application.yml中写配置，自动创建客户端

## Starter开发流程

1.初始化环境依赖（一定要移除build）

spring-boot-configuration-processor的作用是自动生成配置的代码提示

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.16</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

2.编写配置类

```java
@Configuration
@ConfigurationProperties("yuapi.client")
@Data
@ComponentScan
public class YuApiClientConfig {

    private String accessKey;

    private String secretKey;

    @Bean
    public YuApiClient yuApiClient(){
        return new YuApiClient(accessKey,secretKey);
    }
}

```

- @ConfigurationProperties("yuapi.client")配置绑定，在application.yml中可以直接使用yuapi.client.accesskey和yuapi.client.secretkey
- @Bean注解将这个类添加到容器中，别的地方就可以直接调用（配合@configuration使用）
- @configuration的作用是告诉Spring这是一个配置类，里面可能包含Bean的含义

3.注册配置类，resour/META-INF/spring.factories

```
# spring boot starter
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.yupi.yuapiclientsdk.YuApiClientConfig
```

4.maven中进行打包

![image-20241124113141413](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20241124113141413.png)

5.记录pom.xml

```xml
<groupId>com.yupi</groupId>
<artifactId>yuapi-client-sdk</artifactId>
<version>0.0.1</version>
```

6.在新的项目中直接引入

```xml
<dependency>
    <groupId>com.yupi</groupId>
    <artifactId>yuapi-client-sdk</artifactId>
    <version>0.0.1</version>
</dependency>
```



# 功能开发

## 接口发布/下线功能

权限控制：仅管理员可操作

### 业务逻辑

发布接口：

1. 校验接口是否存在
2. 判断该接口是否可以调用
3. 修改数据库中的状态字段为1

下线接口：

1. 校验接口是否存在
2. 修改接口数据库中的状体字段为0



```java
/**
     * 发布接口
     * @param idRequest
     * @param request
     * @return
     */
    @PostMapping("/online")
    @AuthCheck(mustRole = "admin")
    public BaseResponse<Boolean> onlineInterfaceInfo(@RequestBody IdRequest idRequest,
                                                     HttpServletRequest request) {

        if(idRequest == null || idRequest.getId() <= 0){
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }
        Long id = idRequest.getId();
        // 判断是否存在
        InterfaceInfo oldInterfaceInfo = interfaceInfoService.getById(id);
        if(oldInterfaceInfo == null){
            throw new BusinessException(ErrorCode.NOT_FOUND_ERROR);
        }
        // 判断该接口是否可以调用
        com.yupi.yuapiclientsdk.model.User user = new com.yupi.yuapiclientsdk.model.User();
        user.setUsername("test");
        String username = yuApiClient.getUsernameByPost(user);
        if(StringUtils.isBlank(username)){
            throw new BusinessException(ErrorCode.PARAMS_ERROR,"接口验证失败");
        }
        // 仅本人或管理员可以修改
        InterfaceInfo interfaceInfo = new InterfaceInfo();
        interfaceInfo.setId(id);
        interfaceInfo.setStatus(InterfaceInfoStatusEnum.ONLINE.getValue());
        boolean result = interfaceInfoService.updateById(interfaceInfo);
        return ResultUtils.success(result);
    }

```

疑问解答：

为什么此处需要new InterfaceInfo();？

- 更新数据库中的记录时，通常只需要更新部分字段，而不需要更新整个记录
- 使用一个新对象，只设置需要更新的字段（id，status），避免无意中修改其他字段
- updateById通常是通用的更新办法，要求传递主键和需要更新的字段



## 申请签名

用户在注册成功时，自动分配accessKey、secretKey



{“Content-Type”:“application/json”}



## 在线调用

```java
/**
 * 调用接口
 * @param interfaceInfoInvokeRequest
 * @param request
 * @return
 */
@PostMapping("/invoke")
@AuthCheck(mustRole = "admin")
public BaseResponse<Object> invokeInterfaceInfo(@RequestBody InterfaceInfoInvokeRequest interfaceInfoInvokeRequest, HttpServletRequest request) {
    if (interfaceInfoInvokeRequest == null || interfaceInfoInvokeRequest.getId() <= 0) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR);
    }
    Long id = interfaceInfoInvokeRequest.getId();
    String userRequestParams = interfaceInfoInvokeRequest.getUserRequestParams();
    // 校验接口是否存在
    InterfaceInfo oldInterfaceInfo = interfaceInfoService.getById(id);
    if (oldInterfaceInfo == null) {
        throw new BusinessException(ErrorCode.NOT_FOUND_ERROR);
    }
    if (oldInterfaceInfo.getStatus() == InterfaceInfoStatusEnum.OFFLINE.getValue()) {
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "接口已关闭");
    }
    User loginUser = userService.getLoginUser(request);
    String accessKey = loginUser.getAccessKey();
    String secretKey = loginUser.getSecretKey();
    YuApiClient tempClient = new YuApiClient(accessKey, secretKey);
    Gson gson = new Gson();
    com.yupi.yuapiclientsdk.model.User user = gson.fromJson(userRequestParams, com.yupi.yuapiclientsdk.model.User.class);
    String usernameByPost = tempClient.getUsernameByPost(user);
    return ResultUtils.success(usernameByPost);
}
```

## 接口调用次数统计

需求：

1. 用户每次调用接口成功，次数+1
2. 给用户分配或者用户自主申请接口调用次数

业务流程：

1. 用户调用接口
2. 修改数据库，调用次数 + 1



```java
@Override
public boolean invokeCount(long interfaceInfoId, long userId) {
    if(interfaceInfoId <= 0 || userId <= 0){
        throw new BusinessException(ErrorCode.PARAMS_ERROR);
    }
    UpdateWrapper<UserInterfaceInfo> updateWrapper = new UpdateWrapper<>();
    updateWrapper.eq("interfaceInfoId",interfaceInfoId);
    updateWrapper.eq("userId",userId);
    updateWrapper.setSql("leftNum = leftNum - 1, totalNum = totalNum + 1");
    return this.update(updateWrapper);
}
```

- 此处可以优化，加锁，一个用户可能瞬间添加



**问题：**

如果每个接口的方法都写调用次数 + ，是不是比较麻烦?

**解决：**

使用AOP切面的有优点：独立于接口，在每个接口调用后统计次数 + 1

缺点：只存在于单个项目中，如果每个团队都要开发自己的模拟接口，那么都要写一个切面



## 网关

![image-20241125161212482](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20241125161212482.png)

### 背景问题

>以下网址详细解释

https://www.codefather.cn/course/1790979723916521474/section/1790987201920151554?type=#heading-21

- 用户希望调用某个接口，面临多个问题：
  - 先调用接口A，然后接口A再调用统计次数
  - 接着调用接口B，再去统计接口次数

### 网关的设计作用

1. 统一管理接口
   - 用户直接调用网关，而不是直接调用具体接口
   - 网关负责根据用户的请求地址找到对应的接口
2. 隐藏具体实现
   - 用户不需要关心接口由哪个团队开发
   - 用户只需要提供需求
3. 自动统计调用次数
   - 网关在处理每个请求时，会自动对接口调用次数进行统计

### 网关作用

**路由**

- 将请求分发到正确的后端服务，实现请求的路径转发。

**负载均衡**

- 在多个后端实例中分配请求，优化资源使用，提升系统性能。

**统一鉴权**

- 负责用户身份验证和权限校验，确保服务安全。

**跨域**

- 解决浏览器的跨域访问限制，允许合法的跨域请求。
- 网关统一处理跨域，不用在每个项目里单独处理

**统一业务处理（缓存）**

- 提供通用的业务逻辑处理（如缓存机制），减少重复计算，提升响应速度。

**访问控制**

- 限制特定用户或 IP 访问服务，防止非法请求。

**发布控制**

- 支持灰度发布、版本控制等功能，方便新功能逐步上线。
- 可以先给新街口分配20%的流量，老接口80%

**流量染色**

- 为特定请求流量打标签，用于调试或监控。

**接口保护**

- **限制请求：** 防止接口被过度调用。
- **信息脱敏：** 对敏感信息进行隐藏或加密处理。
- **降级（熔断）：** 在服务故障时自动降级以保护系统。
- **限流：** 使用算法（如令牌桶、漏桶）控制流量，避免过载。
- **超时设置：** 限制请求处理时间，避免资源被长期占用。

**统一业务处理**

- 把一些每个项目中都要做的通用逻辑放到上层（网关），统一处理，比如本项目的次数统计

**统一鉴权**

- 判断用户是否有权限进行操作，无论访问什么接口，我都统一去判断权限，不用重复写

**访问控制**

- 黑白名单。限制DDOS IP

**统一日志**

- 集中记录请求和响应的日志，便于监控和排查问题。

**统一文档**

- 提供接口的统一文档，方便开发者理解和调用。

### 网关分类

1. 全局网关：作用是负载均衡、请求日志，不和逻辑业务绑定
2. 业务网关：会有一些业务逻辑，作用是将请求转发到不同的业务/项目/接口/服务

>推荐文章：https://blog.csdn.net/qq_21040559/article/details/122961395

### 网关实现

1. Nginx（全局网关）、Kong网关，编程成本相对高
2. Spring Cloud Gateway 性能高、可以用Java代码来写逻辑，适合学习

> 网关技术选型：https://zhuanlan.zhihu.com/p/500587132



## Spring Cloud Gateway

> 官网：https://spring.io/projects/spring-cloud-gateway/

>https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway.html

### 核心概念

路由（根据什么条件，转发请求到哪里）

断言：一组规则、条件，用来确定如何转发路由

过滤器：对请求进行一系列的处理，比如添加请求头、添加请求参数



请求流程：

1. 客户端发起请求
2. Handler Mapping：根据断言，去将请求转发到对应的路由
3. Web Handler：处理请求（一层层经过过滤器）
4. 实际调用服务

![image-20241126154416773](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20241126154416773.png)

### 两种配置方式

1. 配置式
2. 编程式

### 建议开启日志

```text
logging:
  level:
    org:
      springframework:
        cloud:
          gateway: trace
```

### 断言

1. After在 xx时间之后
2. Before在 xx 时间之前
3. Between在xx时间之间
4. 请求类别
5. 请求头（包含cookie）
6. 查询参数
7. 客户端地址
8. **权重**--- 根据权重实现发布设置

### 过滤器

基本功能：对请求头、请求参数、响应头的增删改查

1. 添加请求头
2.  添加请求参数
3. 添加响应头
4. 降级
5. 限流
6. 重试

引入：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```



**添加请求头、请求参数、响应头：**

此处当你访问http://localhost:8090/api/...时会自动映射到http://localhost:8123/api/...

```yaml
server:
  port: 8090

spring:
  cloud:
    gateway:
      routes:
        - id: add_request_header_route
          uri: http://localhost:8123
          predicates:
            - Path=/api/**
          filters:
            - AddRequestHeader=melon, shuaige
            - AddRequestParameter=name, dog
```

**降级**

先引入包：

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
        </dependency>
```



```yaml
server:
  port: 8090

spring:
  cloud:
    gateway:
      routes:
        - id: add_request_header_route
          uri: http://localhost:8123
          predicates:
            - Path=/api/**
          filters:
            - AddRequestHeader=melon, shuaige
            - AddRequestParameter=name, dog
            - name: CircuitBreaker
              args:
                name: myCircuitBreaker
                fallbackUri: forward:/fallback
        - id: yupi-fallback
          uri: https://www.codefather.cn/course
          predicates:
            - Path=/fallback
```

当访问localhost:8090的时候不能访问，触发。就转到fallback



# 网关开发

## 要用的特性

1. 路由（转发请求到模拟接口项目）
2. 统一鉴权（access Key，secretKey）
3. 统一业务处理（每次请求接口后，接口调用次数+1）
4. 访问控制（黑白名单）
5. 流量染色（记录请求是否为网关来的）
6. 统一日志（记录每次的请求和响应日志）

## 业务逻辑

1. 用户发送请求到API网关
2. 请求日志
3. （黑白名单）
4. 用户鉴权（判断ak，sk）
5. 请求的模拟接口是否存在
6. **请求转发，调用模拟接口**
7. 响应日志
8. 调用成功，接口调用次数+1
9. 调用失败，返回错误码

## 具体实现

### **1.请求转发**

使用前缀匹配断言路由器

所有路径为：/api/** 的请求进行转发，转发到http://localhost:8123/api/**

比如请求网关：http://localhost:8090/api/name/get?name=dog

转发到：http://localhost:8123/api/name/get?name=dog

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: api_route
          uri: http://localhost:8123
          predicates:
            - Path=/api/**
```

### 2.编写业务逻辑

使用GlobalFilter，全局请求拦截处理

因为网关项目没有引入Mybatis等操作数据库的类库，如果该操作较为复杂，可以由backend增删改查项目提供接口，我们直接调用



### 问题

预期是等模拟接口调用完成，才记录响应日志、统计调用次数

但现实是chain.filter方法立刻返回了，直到filter过滤器return后才调用了模拟接口

原因是：chain.filter是个异步操作

解决方案：利用response装饰者，增强原有response的处理能力

参考博客：https://blog.csdn.net/qq_19636353/article/details/126759522



# RPC

问题：网管项目比较纯净，没有操作数据库的包，并且还要调用我们之前写过的代码？复制粘贴维护麻烦

理想：直接请求到其他项目的方法

## 怎么调用其他项目的方法？

1. 复制代码
2. HTTP请求（提供一个接口）
3. RPC
4. 把公共的代码打个jar包，其他项目引用（客户端SDK）

## HTTP请求

1. 提供方开放一个接口（地址、请求方法、参数、返回值）
2. 调用方法使用HTTP CLient之类的代码去发送HTTP请求

### 示例

使用RESTful API调用远程服务

假设有两个项目：

服务A：提供了一个RESTful API，可以查询用户信息

服务B：需要从服务A获取用户信息，服务B使用HTTP请求调用服务A



步骤1：服务A--提供RESTful API，可以查询用户信息

```java
@RestController
@RequestMapping("/users")
public class UserController {

    // 模拟从数据库中查询用户
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        // 假设用户数据如下
        User user = new User(id, "dog", "dog@example.com");
        return ResponseEntity.ok(user);
    }
}

```
步骤2：服务B--使用HTTP请求调用服务A

```java
@Service
public class UserService {

    private final RestTemplate restTemplate;

    public UserService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public User getUserFromServiceA(Long userId) {
        // 服务 A 的 URL
        String url = "http://localhost:8080/users/" + userId;

        // 发起 GET 请求并返回 User 对象
        return restTemplate.getForObject(url, User.class);
    }public User getUserFromServiceA(Long userId) {
        // 服务 A 的 URL
        String url = "http://localhost:8080/users/" + userId;

        // 发起 GET 请求并返回 User 对象
        return restTemplate.getForObject(url, User.class);
    }
}
```

步骤3--配置RestTemplate和调用

在服务B中调用

```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

## RPC

作用：像调用本地方法一样调用远程方法

和直接HTTP调用的区别：

1. 对开发者更加透明，减少了很多沟通成本
2. RPC向远程服务器发送请求时，未必要使用HTTP协议，比如还可以用TCP/IP，性能更高。（内部服务更适用）

RPC调用模型：

![image-20241129112953992](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20241129112953992.png)



## Dubbo框架（RPC实现）

> 官网：https://cn.dubbo.apache.org/zh-cn/overview/mannual/java-sdk/quick-start/starter/

### 示例项目学习

zookeeper注册中心：通过内嵌的方式运行，更方便

zookeeper就是图中的注册中心

最先启动注册中心，先启动服务提供者，再启动服务消费者

### 整合运用

1. backend项目作为服务提供者，提供三个方法：
   - 实际情况应该是去数据库中查是否已经分配给用户
   - 从数据库中查询模拟接口的存在，以及请求方法是否匹配
   - 调用成功，接口调用次数+1 invokeCOunt
2. gateway项目作为服务调用者，调用这3个办法



### 整合Nacos

跟着官方文档来！

建议使用Nacos！https://cn.dubbo.apache.org/zh-cn/overview/mannual/java-sdk/reference-manual/registry/nacos/

### 

#### 报错1：端口冲突

![image-20241203110129762](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20241203110129762.png)



服务自动绑定22222端口，但是绑定失败

原因：因为在配置文件中写的都是-1，两个项目都随机生成相同的端口了



# Todo

1. 完成网关业务逻辑
2. 开发管理员分析的功能
3. 上线



可以复用的操作：

1. 

1. 实际情况应该是去数据库中查是否已经分配给用户密钥
   1. 先根据accessKey判断用户是否存在，查到accessKey
   2. 对比secretKey和用户传的加密后的secretKey是否一致
2. 从数据库中查询模拟接口是否存在，以及请求方法是否匹配
3. 调用成功，接口调用次数 + 1 invokeCount



# 公共服务

目的是让方法、实体类在多个项目间复用



1. 实际情况应该是去数据库中查是否已经分配给用户密钥
2. 从数据库中查询模拟接口是否存在，以及请求方法是否匹配
3. 调用成功，接口调用次数 + 1 invokeCount

