# 一、基于注解管理bean

## 1.1 @Autowired注入

### 1.1.1 属性注入

- 先在配置文件中开启包扫描 bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    <!--开启组件扫描功能-->
    <context:component-scan base-package="com.melon"></context:component-scan>
</beans>
```

- 总体顺序：Controller-->Service-->Dao
- Service-->ServiceImpl
- Dao-->DaoImpl
- UserController中注入Service
- 在Service中注入UserDao



UserController:

```java
package com.melon.autowired.controller;

import com.melon.autowired.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

/**
 * ClassName: UserController
 * Package: com.melon.autowired.controller
 * Description:
 *
 * @Author melon
 * @Create 2024/3/3 17:39
 * @Version 1.0
 */
@Controller
public class UserController {
    @Autowired
    private UserService userService;

    public void add(){
        System.out.println("usercontroller...");
        userService.add();
    }
}

```



UserService：

```java
package com.melon.autowired.service;

/**
 * ClassName: UserService
 * Package: com.melon.autowired.service
 * Description:
 *
 * @Author melon
 * @Create 2024/3/3 17:38
 * @Version 1.0
 */
public interface UserService {
    public void add();
}

```



UserServiceImpl:

```java
package com.melon.autowired.service;

import com.melon.autowired.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * ClassName: UserServiceImpl
 * Package: com.melon.autowired.service
 * Description:
 *
 * @Author melon
 * @Create 2024/3/3 17:38
 * @Version 1.0
 */
@Service
public class UserServiceImpl implements UserService{

    @Autowired
    private UserDao userDao;
    @Override
    public void add() {
        System.out.println("service...");
        userDao.add();
    }
}
```



UserDao:

```java
package com.melon.autowired.dao;

/**
 * ClassName: UserDao
 * Package: com.melon.autowired.dao
 * Description:
 *
 * @Author melon
 * @Create 2024/3/3 17:39
 * @Version 1.0
 */

public interface UserDao {
    public void add();
}
```



UserDaoImpl:

```java
package com.melon.autowired.dao;

import org.springframework.stereotype.Repository;

/**
 * ClassName: UserDaoImpl
 * Package: com.melon.autowired.dao
 * Description:
 *
 * @Author melon
 * @Create 2024/3/3 17:39
 * @Version 1.0
 */
@Repository
public class UserDaoImpl implements UserDao{
    @Override
    public void add() {
        System.out.println("userdao...");
    }
}
```



### 1.1.2 set注入

- 修改controller中的注入
- 修改ServiceImpl中的注入
- UserController:

```java
		private UserService userService;
    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
```

- UserServiceImpl:

```java
    private UserDao userDao;
    @Autowired
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
```



### 1.13 构造方法注入

- UserController

```java
private UserService userService;
@Autowired
public UserController(UserService userService) {
    this.userService = userService;
}
```

- UserServiceImpl

```java
private UserDao userDao;
@Autowired
public UserServiceImpl(UserDao userDao) {
    this.userDao = userDao;
}
```

## 1.2 全注解开发

- 定义配置类，添加注解

- ```java
  @Configuration //这是一个配置类
  @ComponentScan("com.melon.autowired") // 打开包扫描
  ```

  ```java
  @Configuration
  @ComponentScan("com.melon.autowired")
  public class SpringConfig {
  }
  ```

- 测试类中进行修改

- ```java
  public class TestUserControllerAnno {
      public static void main(String[] args) {
          ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
          UserController controller = context.getBean(UserController.class);
          controller.add();
      }
  }
  ```



# 二、单元测试

## 2.1 整合junit5

1. 引入依赖
2. 添加配置文件
3. 添加Java类
4. 测试

- 引入依赖

```xml
<dependencies>
    <!--spring context依赖-->
    <!--当你引入Spring Context依赖之后，表示将Spring的基础依赖引入了-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>6.0.2</version>
    </dependency>

    <!--spring对junit的支持相关依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>6.0.2</version>
    </dependency>

    <!--junit5测试-->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.9.0</version>
    </dependency>

    <!--log4j2的依赖-->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.19.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j2-impl</artifactId>
        <version>2.19.0</version>
    </dependency>
</dependencies>
```

- 添加配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.melon.junit5"/>
</beans>
```

- 添加Java类

```java
package com.melon.junit5;

import org.springframework.stereotype.Component;

/**
 * ClassName: User
 * Package: com.melon.junit5
 * Description:
 *
 * @Author melon
 * @Create 2024/3/5 20:34
 * @Version 1.0
 */
@Component
public class User {
    public void run(){
        System.out.println("user...");
    }
}
```

- 测试

```java
@SpringJUnitConfig(locations = "classpath:bean.xml")
public class TestJunit5 {

    @Autowired
    private User user;

    @Test
    public void test(){
        System.out.println(user);
        user.run();
    }
}
```

# 三、事务

## 3.1 JdbcTemplate实现CRUD

1. 添加依赖
2. 创建jdbc.properties
3. 配置Spring的配置文件
4. 准备数据库与测试表
5. 创建测试类，整合JUnit，注入JdbcTemplate
6. 测试增删改查功能

- 添加依赖

```xml
<dependencies>
    <!--spring jdbc  Spring 持久化层支持jar包-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>6.0.2</version>
    </dependency>
    <!-- MySQL驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>
    <!-- 数据源 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.15</version>
    </dependency>
</dependencies>
```

- 创建jdbc.properties

```properties
jdbc.user=root
jdbc.password=pj123456
jdbc.url=jdbc:mysql://localhost:3306/spring?characterEncoding=utf8&useSSL=false
jdbc.driver=com.mysql.cj.jdbc.Driver
```

- Spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 导入外部属性文件 -->
    <context:property-placeholder location="classpath:jdbc.properties" />

    <!-- 配置数据源 -->
    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="username" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- 配置 JdbcTemplate -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!-- 装配数据源 -->
        <property name="dataSource" ref="druidDataSource"/>
    </bean>

</beans>
```

- 增删改查测试
  - 增删该都是相同的步骤

```java
@Test
public void testUpdate(){
    // 先写sql语句，然后将sql语句传入jdbcTemplate的update中
    String sql = "insert into t_emp values(null,?,?,?)";
    // Object[] parms = {"aaa",20,"男"};
    // int rows = jdbcTemplate.update(sql,parms);
    int rows = jdbcTemplate.update(sql,"ccc",40,"男");
    System.out.println(rows);
}
```

- 查！

  - 先注入jdbcTemplate

  - 写sql

  - 调用jdbcTemplate.queryForObject

  - `queryForObject`方法用于执行查询，并期望返回单个对象。这个方法接收三个参数：

    - 第一个参数是SQL查询字符串。
    - 第二个参数是`RowMapper`的实例，这里使用`BeanPropertyRowMapper`，它能够自动将查询结果的行映射到`Emp`类的实例。`BeanPropertyRowMapper<>(Emp.class)`指示Spring如何将查询结果的一行转换成一个`Emp`对象，基于列名和`Emp`类属性名的自动匹配。
    - 第三个参数是绑定到SQL查询中占位符的参数值，在这个例子中是数字`1`。这意味着查询会查找`id`等于1的`t_emp`表条目。

  - ```java
    // 注入jdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    // 查询
    // 1.返回对象
    @Test
    public void testSelect(){
        String sql = "select * from t_emp where id=?";
        Emp emp = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(Emp.class), 1);
        System.out.println(emp);
    }
    
    // 2.返回list集合
    @Test
    public void testSelectList(){
        String sql = "select * from t_emp";
        List<Emp> query = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(Emp.class));
        System.out.println(query);
    }
    ```