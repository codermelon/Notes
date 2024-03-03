# 一、 基于注解管理bean

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