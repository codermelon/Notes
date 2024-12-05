# Bug汇总

## 1.修改用户的时候后端拿不到值，调试的时候id为0

没有加注解@RequestBody

```java
 @PostMapping("/update")
    public BaseResponse<Integer> updateUser(@RequestBody User user,HttpServletRequest request){
        // 判断是否为空
        if(user == null){
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }
        // 判断权限
        User loginUser = userService.getLoginUser(request);
        int result = userService.updateUser(user,loginUser);
        return ResultUtils.success(result);
    }
```







