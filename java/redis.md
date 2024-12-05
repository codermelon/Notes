# 基于Session实现登录

```java
public Result login(LoginFormDTO loginForm, HttpSession session) {
        // 校验手机号
        String phone = loginForm.getPhone();
        if(RegexUtils.isPhoneInvalid(phone)){
            return Result.fail("手机号格式错误");
        }
        // 校验验证码
        Object cacheCode = session.getAttribute("code");
        String code = loginForm.getCode();
        if(cacheCode == null || !cacheCode.toString().equals(code) ){
            // 不一致 报错
            return Result.fail("验证码不正确");
        }
        // 一致 根据手机号查询用户
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("phone",phone);
        User user = this.getOne(queryWrapper);
        // 判断用户是否存在
        if(user == null){
            // 不存在 直接创建用户
             user = createUserWithPhone(phone);
        }
        // 存在 保存用户信息到session
        UserDTO userDTO = new UserDTO();
        BeanUtil.copyProperties(user,userDTO);
        session.setAttribute("user",userDTO);
        // ok
        return Result.ok();
```

这里需要脱敏，不必要的信息不需要暴露出去
注意：BeanUtil.copyProperties(user,userDTO); 这里前面的是源，后面的是要复制过去的

# 添加拦截器实现登录

思考：后续业务增多，每次写业务都要重新获取session校验用户信息？

添加拦截器：所有请求先经过拦截器，把session保存在threadlocal中，由拦截器统一校验

![image-20241205122953672](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20241205122953672.png)

```java
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //1. 获取session
        HttpSession session = request.getSession();
        //2.获取session中的用户
        Object user = session.getAttribute("user");
        //3. 判断用户是否存在
        if (user == null){
            //4. 不存在，拦截
            response.setStatus(401);
            return false;
        }

        //5. 存在 保存用户信息到ThreadLocal
        UserHolder.saveUser((UserDTO) user);
        //6. 放行
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        //移除用户
        UserHolder.removeUser();
    }
}

```

注册拦截器：

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 登录拦截器
        registry.addInterceptor(new LoginInterceptor())
                .excludePathPatterns(
                        "/shop/**",
                        "/voucher/**",
                        "/shop-type/**",
                        "/upload/**",
                        "/blog/hot",
                        "/user/code",
                        "/user/login"
                );
    }
}
```

