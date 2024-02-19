# 入门案例

**入门案例步骤：**

- 引入spring相关依赖

```java
<!--spring context依赖-->
<!--当你引入Spring Context依赖之后，表示将Spring的基础依赖引入了-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.0.2</version>
</dependency>

<!--junit5测试-->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.3.1</version>
</dependency>
```

- 创建类，定义属性和方法

```java
public class User {
    public void add(){
        System.out.println("add...");
    }
}
```

- 按照spring要求创建配置文件

```java
<!--完成user对象创建
    bean标签
        id属性：唯一标识
        class属性：要创建对象所在类的全路径（包名称+类名称）
-->
<bean id="user" class="com.melon.User"></bean>
```

- 在spring配置文件配置相关信息

```java
public void testUser(){
    //加载spring配置文件，对象创建
    ApplicationContext context =
            new ClassPathXmlApplicationContext("bean.xml");

    //获取创建的对象
    User user = (User)context.getBean("user");
    System.out.println("1:"+user);

    //使用对象调用方法进行测试
    System.out.print("2:");
    user.add();
}
```

- 进行最终测试