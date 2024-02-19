# 1. 快速上手

- 1.确保app注册 在settings.py中注册

![image.png](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image.png)

- 编写URL和视图函数的对应关系 urls.py

![image.png](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image%201.png)

- 编写视图函数 views.py

![image.png](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image%202.png)

- 启动django项目

# 2. 再写一个页面

主要是在urls.py和views.py中编写

![image.png](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image%203.png)

# 3. templates模版

- 返回字符串用httpresponse

- 返回一个html用render

![image.png](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image%204.png)

# 4. 静态文件

## 4.1 注意事项

开发过程中一般将图片，css，js当静态文件处理

必须放入static文件夹中，static中一般也要创建几个文件夹

![image.png](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image%205.png)

## 4.2 资源引入格式

![image.png](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image%206.png)

# 5. Django模板语法

```Python
from django.http import HttpResponse
from django.shortcuts import render

# Create your views here.

def index(request):
    return HttpResponse("欢迎使用")

def user_list(request):
    # app目录下创建一个templates文件夹
    return render(request,"user_list.html")

def user_add(request):
    return render(request,"user_add.html")

def tpl(request):
    name="潘杰"
    roles=["管理员","CEO","保安"]
    user_info={"name":"xiaopan","salary":1000,"role":"CTO"}
    data_list=[
        {"name": "xiaopan", "salary": 1000, "role": "jungle"},
        {"name": "xiaoming", "salary": 1000, "role": "top"},
        {"name": "uzi", "salary": 1000, "role": "ad"},
    ]
    return render(request,
                  "tpl.html",
                  {"n1":name,"n2":roles,"n3":user_info,"n4":data_list})
```

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>模板语法学习</title>
</head>
<body>
    <div>
        {{n1}}
    </div>
    <div>
        {{n2}}
    </div>
    <h1>访问数组内的元素</h1>
    <div>{{ n2.0 }}</div>
    <div>{{ n2.1 }}</div>
    <div>{{ n2.2 }}</div>
    <h1>循环打印数组内的元素</h1>
    <div>
        {% for item in n2 %}
            <spqn>{{ item }}</spqn>
        {% endfor %}
    </div>
<hr/>
    <h1>打印字典内的元素</h1>
    {{ n3.name }} {{ n3.salary }} {{ n3.role }}
    <h3>拿键</h3>
    <ul>
        {% for foo in n3.keys %}
        	<li>{{ foo }}</li>
        {% endfor %}
    </ul>
    <h3>拿值</h3>
    <ul>
        {% for foo in n3.values %}
        	<li>{{ foo }}</li>
        {% endfor %}
    </ul>
    <h3>键值</h3>
    <ul>
        {% for foo in n3.items %}
        	<li>{{ foo }}</li>
        {% endfor %}
    </ul>

    <hr/>
    {{ n4.0 }}
    {{ n4.1.name}}
    {% for foo in n4 %}
    	<div>{{ foo.name }} {{ foo.salary }} {{ foo.role }}</div>
    {% endfor %}
    <hr/>

    <h1>条件语句</h1>
    {% if n1 == "潘杰" %}
    	<h1>aaaa</h1>
    {% else %}
        <h2>bbbb</h2>
    {% endif %}

</body>
</html>
```

# 6. 简单的爬虫

```python
import requests
headers={
    "User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36 Edg/115.0.1901.203"
}
res = requests.get("https://movie.douban.com/j/search_tags?type=movie&tag=%E7%83%AD%E9%97%A8&source=index",headers=headers)
data_list=res.json()
print(data_list)
```

1. 导入requests
2. 加入UA
3. 发送请求



# 7. 请求和响应

```python
def something(request):
    #request是一个对象，封装了用户通过浏览器发送过来的所有数据
    # 1.获取用户请求方式
    print(request.method) # get

    # 2.在URL中传递值 /something/?n1=999&n2=999
    print(request.GET) # [18/Aug/2023 06:45:55] "GET /something/?n1=999&n2=999 HTTP/1.1" 200 12

    # 3.在请求体中提交数据
    print(request.POST)

    # 4.内容字符串返回给请求者
    # return HttpResponse("返回内容")

    # 5.返回页面，读取html的内容并且渲染返回
    # return render(request,
    #               "something.html",
    #               {"n1":"你好"})

    # 6.重定向
    # return redirect("https://www.baidu.com")
```

# 案例：用户登录

## 在urls.py和view.py中进行配置，创建login.html

**Urls.py:**

```python
# 用户登录
    path('login/',views.login)
```

**View.py**

```python
def login(request):
    if request.method == "GET":
        return render(request,"login.html")
    else:
        # 如果是post请求，获取用户提交的数据
        print(request.POST)
        username = request.POST.get("user") #获取提交的username
        password = request.POST.get("pwd") #获取提交的pwd
        # 进行校验
        if username == 'root' and password == "123":
            # return HttpResponse("登录成功")
            return redirect("https://www.bilibili.com/")
        else:
            return render(request,
                          "login.html",
                          {"error_msg":"用户名或密码错误"})
```

如果用户点击浏览器进行输入时，方法为GET，即method为get时，跳转到页面

当提交页面时，html表单会跳到login，此时method为post，打印用户信息

拿到用户提交的uesername和pwd进行校验

**Login.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
</head>
<body>
    <h1>用户登录</h1>
    <form method="post" action="/login/">
        {% csrf_token %}
        <input type="text" name="user" placeholder="用户名">
        <input type="password" name="pwd" placeholder="密码">
        <input type="submit" value="提交"/>
        <span style="color: red">{{ error_msg }}</span>
    </form>
</body>
</html>
```

## 常见错误

**Django常见问题 安全验证**

![image-20230818151154766](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230818151154766.png)

**解决：**

在form表单中加入

```html
{% csrf_token %}
```

# 8.数据库操作

- 数据库+pymysql

```python
import pymysql

# 1.连接MySQL
conn = pymysql.connect(host="127.0.0.1", port=3306, user='root', passwd="root123", charset='utf8', db='unicom')
cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)

# 2.发送指令
cursor.execute("insert into admin(username,password,mobile) values('wupeiqi','qwe123','15155555555')")
conn.commit()

# 3.关闭
cursor.close()
conn.close()
```

- Django 开发操作数据库，内部提供orm框架



## 8.1 安装第三方模块

```
pip3 install mysqlclient
```

### 出现的问题：安装失败，显示

![img](https://pj-typora.oss-cn-shanghai.aliyuncs.com/6be4fa706f85bbd9f6fec76505d45f29.png)

### 解决：

注意 此处对应的是python3.9

```
pip3 install mysqlclient==1.4.6
```

## 8.2 ORM

ORM可以帮助我们做两件事：

- 创建、修改、删除数据库中的表（不用你写SQL语句）。 【无法创建数据库】

- 操作表中的数据（不用写SQL语句）。

### 1. 自己创建数据库

- 启动MySQL服务

- 自带工具创建数据库

  ```mysql
  create database gx_day15 DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
  ```



![image-20230820132439532](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230820132439532.png)

### 2. django连接数据库

在settings.py文件中进行配置和修改。

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'gx_day15',  # 数据库名字
        'USER': 'root',
        'PASSWORD': 'pj123456',
        'HOST': 'localhost',  # 那台机器安装了MySQL
        'PORT': 3306,
    }
}
```





### 3.django操作表

- 创建表
- 删除表
- 修改表

**创建表：在models.py文件中**

![image-20230820133448065](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230820133448065.png)

```python
class UserInfo(models.Model):
    name = models.CharField(max_length=32)
    password = models.CharField(max_length=64)
    age = models.IntegerField()
```

相当于自动生成如下的sql语句

```sql
create table app01_userinfo(
    id bigint auto_increment primary key,
    name varchar(32),
    password varchar(64),
    age int
)
```

执行命令：

```
python3 manage.py makemigrations
python3 manage.py migrate
```

**注意：app需要提前注册。**

#### 3.1 执行python3 manage.py makemigrations出现的问题：

![image-20230820141422478](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230820141422478.png)

#### 3.2 解决方案：

在__init__.py中加入：

```python
import pymysql
pymysql.install_as_MySQLdb()
```



在表中新增列时，由于已存在列中可能已有数据，所以新增列必须要指定新增列对应的数据：

- 1，手动输入一个值。

- 设置默认值

  ```
  age = models.IntegerField(default=2)
  ```

- 允许为空

  ```
  data = models.IntegerField(null=True, blank=True)
  ```



以后在开发中如果想要对表结构进行调整：

- 在models.py文件中操作类即可。

- 命令

  ```
  python3 manage.py makemigrations
  python3 manage.py migrate
  ```

  

### 4.表中的数据

```python
# #### 1.新建 ####
# Department.objects.create(title="销售部")
# Department.objects.create(title="IT部")
# Department.objects.create(title="运营部")
# UserInfo.objects.create(name="武沛齐", password="123", age=19)
# UserInfo.objects.create(name="朱虎飞", password="666", age=29)
# UserInfo.objects.create(name="吴阳军", password="666")

# #### 2.删除 ####
# UserInfo.objects.filter(id=3).delete()
# Department.objects.all().delete()

# #### 3.获取数据 ####
# 3.1 获取符合条件的所有数据
# data_list = [对象,对象,对象]  QuerySet类型
# data_list = UserInfo.objects.all()
# for obj in data_list:
#     print(obj.id, obj.name, obj.password, obj.age)

# data_list = [对象,]
# data_list = UserInfo.objects.filter(id=1)
# print(data_list)
# 3.1 获取第一条数据【对象】
# row_obj = UserInfo.objects.filter(id=1).first()
# print(row_obj.id, row_obj.name, row_obj.password, row_obj.age)


# #### 4.更新数据 ####
# UserInfo.objects.all().update(password=999)
# UserInfo.objects.filter(id=2).update(age=999)
# UserInfo.objects.filter(name="朱虎飞").update(age=999)
```

# 案例：用户管理

## 1. 展示用户列表

- url
- 函数
  - 获取所有用户信息
  - HTML渲染

```pyhton
def info_list(request):
    # 1.获取所有的用户信息
    UserInfo.objects.create(name="吴阳军", password="666")
    data_list = UserInfo.objects.all()
    print(data_list)
    return render(request,
                  "info_list.html",
                  {"data_list":data_list})
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>用户管理</h1>
    <table border="1">
        <thead>
            <tr>
                <th>ID</th>
                <th>姓名</th>
                <th>密码</th>
                <th>年龄</th>
            </tr>
        </thead>
        <tbody>
            {% for data in data_list %}
        	    <tr>
                    <th>{{ data.id }}</th>
                    <th>{{ data.name }}</th>
                    <th>{{ data.password }}</th>
                    <th>{{ data.age }}</th>
                </tr>
            {% endfor %}
        </tbody>
    </table>
</body>
</html>
```



## 2.添加用户

- url
- 函数
  - GET，看到页面，输入内容。
  - POST，提交 -> 写入到数据库。

```python
def info_add(request):
    if request.method == "GET":
        return render(request,"info_add.html")
    else:
        # 获取用户提交的数据
        user = request.POST.get("user")
        pwd = request.POST.get("pwd")
        age = request.POST.get("age")

        # 提交到数据库
        UserInfo.objects.create(name=user,password=pwd,age=age)

        # 自动跳转
        return redirect("/info/list/")
```



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>用户添加</h1>
    <form method="post">
        {% csrf_token %}
        <input type="text" name="user" placeholder="用户名">
        <input type="text" name="pwd" placeholder="密码">
        <input type="text" name="age" placeholder="年龄">
        <input type="submit" value="提交">
    </form>
</body>
</html>
```

## 3.删除用户

- url
- 函数



````
http://127.0.0.1:8000/info/delete/?nid=1
http://127.0.0.1:8000/info/delete/?nid=2
http://127.0.0.1:8000/info/delete/?nid=3

def 函数(request):
	nid = reuqest.GET.get("nid")
	UserInfo.objects.filter(id=nid).delete()
	return HttpResponse("删除成功")
````

# 9 项目实战

## 1. 创建项目

![image-20230823214213642](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230823214213642.png)

## 2. 创建app并注册

![image-20230823214520045](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230823214520045.png)

注册：

![image-20230823214750374](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230823214750374.png)

## 3. 设计表结构

```python
from django.db import models


class Department(models.Model):
    """ 部门表 """
    title = models.CharField(verbose_name='标题', max_length=32)


class UserInfo(models.Model):
    """ 员工表 """
    name = models.CharField(verbose_name="姓名", max_length=16)
    password = models.CharField(verbose_name="密码", max_length=64)
    age = models.IntegerField(verbose_name="年龄")
    account = models.DecimalField(verbose_name="账户余额", max_digits=10, decimal_places=2, default=0)
    create_time = models.DateTimeField(verbose_name="入职时间")

    # 无约束
    # depart_id = models.BigIntegerField(verbose_name="部门ID")
    # 1.有约束
    #   - to，与那张表关联
    #   - to_field，表中的那一列关联
    # 2.django自动
    #   - 写的depart
    #   - 生成数据列 depart_id
    # 3.部门表被删除
    # ### 3.1 级联删除
    depart = models.ForeignKey(to="Department", to_field="id", on_delete=models.CASCADE)
    # ### 3.2 置空
    # depart = models.ForeignKey(to="Department", to_field="id", null=True, blank=True, on_delete=models.SET_NULL)

    # 在django中做的约束
    gender_choices = (
        (1, "男"),
        (2, "女"),
    )
    gender = models.SmallIntegerField(verbose_name="性别", choices=gender_choices)
```

## 4. 在Mysql中生成表

- 工具连接生成数据库

```
create database gx_day16 DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

- 修改配置文件，连接mysql

![image-20230823221417266](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230823221417266.png)

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'gx_day16',  # 数据库名字
        'USER': 'root',
        'PASSWORD': 'root123',
        'HOST': 'localhost',  # 那台机器安装了MySQL
        'PORT': 3306,
    }
}
```

- django命令生成数据库

```python
python3 manage.py makemigrations
python3 manage.py migrate
```

**也可以**

![image-20230823222039144](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230823222039144.png)

### 执行python3 manage.py makemigrations出现的问题

在init_py中加入

![image-20230823222241151](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230823222241151.png)

## 5. 导入静态文件

![image-20230823222553313](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230823222553313.png)

## 6. 部门管理

### 6.1 部门列表展示

Urls.py中需要注意引入app01

**from app01 import views**

![image-20230824113629134](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824113629134.png)

![image-20230824113701434](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824113701434.png)

### 6.2 关联数据库并且展示数据

queryset作为参数传给render

![image-20230824114748222](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824114748222.png)

在depart_list中循环

![image-20230824114835331](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824114835331.png)

### 6.3 新建部门

#### 1. 链接跳转

![image-20230824115345382](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824115345382.png)

#### 2. 在视图中拿到数据

![image-20230824120750630](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824120750630.png)

#### 3. html中需要设置method

![image-20230824120844474](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824120844474.png)

### 6.4 删除部门

流程：用户点击删除，此时获取到当前的id，views中得到id后到数据库进行删除

#### 1. 链接跳转

![image-20230824121818611](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824121818611.png)

#### 2. 获得id并删除

![image-20230824121933971](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824121933971.png)

#### 3. 获取链接

![image-20230824122118318](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824122118318.png)

### 6.5 编辑部门

因为编辑部门的时候希望能够带着部门的信息显示

比如说此时部门是IT部门，点击编辑按钮，希望显示的是IT部门而不是空的

**思路：点击编辑的时候同时能够传入当前的id，通过ID来携带当前的部门值**

#### 1. 重点：获取id的方法

在urls.py中加入一个正则表达式

点击编辑然后通过url来匹配，获得当前的id

![image-20230824124200467](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824124200467.png)



![image-20230824125231886](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824125231886.png)

**显示默认信息：**

![image-20230824130807320](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824130807320.png)

#### 2. 提交修改的信息

![image-20230824130713227](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230824130713227.png)





## 7. 模板的继承

- 部门列表
- 添加部门
- 编辑部门



定义目版：`layout.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="{% static 'plugin...min.css' %}">
    {% block css %}{% endblock %}
</head>
<body>
    <h1>标题</h1>
    <div>
        {% block content %}{% endblock %}
    </div>
    <h1>底部</h1>
    
    <script src="{% static 'js/jquery-3.6.0.min.js' %}"></script>
    {% block js %}{% endblock %}
</body>
</html>

```

继承母版：

```html
{% extends 'layout.html' %}

{% block css %}
	<link rel="stylesheet" href="{% static 'pluxxx.css' %}">
	<style>
		...
	</style>
{% endblock %}


{% block content %}
    <h1>首页</h1>
{% endblock %}


{% block js %}
	<script src="{% static 'js/jqxxxin.js' %}"></script>
{% endblock %}
```

## 8. 用户管理

### 8.1 用户列表展示

```python
def user_list(request):
    queryset = models.UserInfo.objects.all()
    return render(request,
                  'user_list.html',
                  {'queryset':queryset})
```

![image-20230825161040634](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230825161040634.png)

### 8.2 新增用户（原始方法）

- 此处context相当于一个字典，在render中嵌套一个字典
- 为什么这么写？因为添加用户的时候希望性别和部门名称是从depart表中拿到的而不是自己去输入的
- redirect的时候”/user/list“第一个/不要忘记

```python
# 新建用户
def user_add(request):
    if request.method == "GET":
        context = {
            'gender_choices': models.UserInfo.gender_choices,
            'depart_list': models.Department.objects.all(),
        }
        return render(request,
                      "user_add.html",
                      context)
    # 获取用户数据
    user = request.POST.get("user")
    pwd = request.POST.get("pwd")
    age = request.POST.get("age")
    ac = request.POST.get("ac")
    ctime = request.POST.get("ctime")
    gender_id = request.POST.get("gd")
    depart_id = request.POST.get("dp")

    # 保存到数据库
    models.UserInfo.objects.create(name=user,password=pwd,
                                   age=age,account=ac,
                                   create_time=ctime,gender=gender_id,
                                   depart_id=depart_id)

    return redirect("/user/list/")
```

```html
<div class="form-group">
  <label>性别</label>
  <select class="form-control" name="gd">
    {% for item in gender_choices %}
      <option value="{{ item.0 }}">{{ item.1 }}</option>
    {% endfor %}
  </select>
</div>

<div class="form-group">
  <label>部门</label>
  <select class="form-control" name="dp">
    {% for item in depart_list %}
      <option value="{{ item.id }}">{{ item.title }}</option>
    {% endfor %}
  </select>
</div>
```

- 通过for循环遍历得到有的性别和部门



### 8.3 初识Form

#### 1. views.py

```python
class MyForm(Form):
    user = forms.CharField(widget=forms.Input)
    pwd = form.CharFiled(widget=forms.Input)
    email = form.CharFiled(widget=forms.Input)
    account = form.CharFiled(widget=forms.Input)
    create_time = form.CharFiled(widget=forms.Input)
    depart = form.CharFiled(widget=forms.Input)
    gender = form.CharFiled(widget=forms.Input)


def user_add(request):
    if request.method == "GET":
        form = MyForm() # 实例化
        return render(request, 'user_add.html',{"form":form})
```

#### 2.user_add.html

```html
<form method="post">
    {% for field in form%}
    	{{ field }}
    {% endfor %}
    <!-- <input type="text"  placeholder="姓名" name="user" /> -->
</form>
```

```html
<form method="post">
    {{ form.user }}
    {{ form.pwd }}
    {{ form.email }}
  	# 自动生成html标签
    <!-- <input type="text"  placeholder="姓名" name="user" /> -->
</form>
```



### 8.4 ModelForm（推荐）

#### 1. models.py

```python
class UserInfo(models.Model):
    """ 员工表 """
    name = models.CharField(verbose_name="姓名", max_length=16)
    password = models.CharField(verbose_name="密码", max_length=64)
    age = models.IntegerField(verbose_name="年龄")
    account = models.DecimalField(verbose_name="账户余额", max_digits=10, decimal_places=2, default=0)
    create_time = models.DateTimeField(verbose_name="入职时间")
    depart = models.ForeignKey(to="Department", to_field="id", on_delete=models.CASCADE)
    gender_choices = (
        (1, "男"),
        (2, "女"),
    )
    gender = models.SmallIntegerField(verbose_name="性别", choices=gender_choices)
```



#### 2. views.py

```python
class MyForm(ModelForm):
    xx = form.CharField*("...")
    class Meta:
        model = UserInfo
        fields = ["name","password","age","xx"]


def user_add(request):
    if request.method == "GET":
        form = MyForm()
        return render(request, 'user_add.html',{"form":form})
```

#### 3.user_add.html

```html
<form method="post">
    {% for field in form%}
    	{{ field }}
    {% endfor %}
    <!-- <input type="text"  placeholder="姓名" name="user" /> -->
</form>
```

```html
<form method="post">
    {{ form.user }}
    {{ form.pwd }}
    {{ form.email }}
    <!-- <input type="text"  placeholder="姓名" name="user" /> -->
</form>
```

#### 实例：使用ModelForm添加用户

```python
class UserModelForm(forms.ModelForm):
    class Meta:
        model = models.UserInfo
        fields = ["name","password","age","account","create_time","depart","gender"]
```

1. 先导入forms
2. 定义一个类：Userform
3. class Meta固定的
4. model = models.UserInfo 表明需要和哪张表进行关联
5. fields 制定这张表里面需要的字段

```python
def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # 循环找到所有的插件，添加了class="form-control"
        for name, field in self.fields.items():
            # if name == "password":
            #     continue
            field.widget.attrs = {"class": "form-control", "placeholder": field.label}
```

1. `def __init__(self, *args, **kwargs):` 这是一个构造函数，它将在创建这个类的实例（对象）时被调用。`*args`和`**kwargs`是用来接收任意数量的位置参数和关键字参数的特殊形式。
2. `super().__init__(*args, **kwargs)` 这一行调用了父类的构造函数，以确保父类的初始化代码也被执行。这是一种常见的做法，以便在子类中扩展父类的行为。
3. `for name, field in self.fields.items():` 这一行使用一个循环来迭代表单类的`fields`属性的每一个字段。`self.fields`似乎是一个字典，其中包含了表单中的字段名和对应的字段对象。
4. `field.widget.attrs = {"class": "form-control", "placeholder": field.label}` 这行代码为当前迭代的字段添加了一些HTML属性。在Django中，表单字段会渲染成HTML元素，这些元素可以拥有各种属性。在这里，对于每个字段，代码向其`widget`属性中的`attrs`字典添加了两个属性：
   - `"class": "form-control"`：这个属性会在HTML中添加一个CSS类，通常用于对字段进行样式化，这里是添加了名为`"form-control"`的类。
   - `"placeholder": field.label`：这个属性会将字段的标签（label）用作表单元素的占位符，当用户没有输入内容时会显示这个占位符。

```python
def user_model_form_add(request):
    if request.method == "GET":
        form = UserModelForm()
        return render(request,
                      "user_model_form_add.html",
                      {"form":form})
    # 用户post提交数据
    form = UserModelForm(data=request.POST)
    # 如果数据合法保存到数据库
    if form.is_valid():
        form.save()
        return redirect("/user/list/")
    # 检验失败的话显示错误信息
    return render(request,
                  "user_model_form_add.html",
                  {"form": form})
```

- form = UserModelForm() 创建一个UserModelForm的实例
- form = UserModelForm(data=request.POST) 把用户提交的POST数据传给UserModelForm的实例form

```python
# 新建用户 使用ModelForm（推荐）
from django import forms

class UserModelForm(forms.ModelForm):
    class Meta:
        model = models.UserInfo
        fields = ["name","password","age","account","create_time","depart","gender"]

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # 循环找到所有的插件，添加了class="form-control"
        for name, field in self.fields.items():
            # if name == "password":
            #     continue
            field.widget.attrs = {"class": "form-control", "placeholder": field.label}
            
def user_model_form_add(request):
    if request.method == "GET":
        form = UserModelForm()
        return render(request,
                      "user_model_form_add.html",
                      {"form":form})
    # 用户post提交数据
    form = UserModelForm(data=request.POST)
    # 如果数据合法保存到数据库
    if form.is_valid():
        form.save()
        return redirect("/user/list/")
    # 检验失败的话显示错误信息
    return render(request,
                  "user_model_form_add.html",
                  {"form": form})
```

### 8.5 编辑用户

1. def user_edit(request,nid): 此处需要传入nid，因为编辑的时候需要带入原来的信息
2. row_object = models.UserInfo.objects.filter(id=nid).first() 根据ID去数据库获取要编辑的那一行数据
3. form = UserModelForm(instance=row_object) instance=表示创建表单的时候需要自动填充数据
4. 如果请求方式为post
   - 先执行找到那个数据 row_object = models.UserInfo.objects.filter(id=nid).first()
   - form = UserModelForm(data=request.POST,instance=row_object)，如果没有instance=row_object，那么编辑数据就会变成新增数据，因为form.save()表示保存，这样写表示更新到找到的那一行数据

```python
def user_edit(request,nid):
    # 根据ID去数据库获取要编辑的那一行数据
    row_object = models.UserInfo.objects.filter(id=nid).first()
    # 创建表单时自动填充数据
    if request.method == "GET":
        form = UserModelForm(instance=row_object)
        return render(request,
                      "user_edit.html",
                      {'form':form})

    form = UserModelForm(data=request.POST,instance=row_object)
    if form.is_valid():
        form.save()
        return redirect('/user/list/')

    return render(request,
                  "user_edit.html",
                  {"form": form})
```

## 9. 靓号管理

### 9.1 表结构

```python
class PrettyNum(models.Model):
    """ 靓号表 """
    mobile = models.CharField(verbose_name="手机号", max_length=11)
    # 想要允许为空 null=True, blank=True
    price = models.IntegerField(verbose_name="价格", default=0)

    level_choices = (
        (1, "1级"),
        (2, "2级"),
        (3, "3级"),
        (4, "4级"),
    )
    level = models.SmallIntegerField(verbose_name="级别", choices=level_choices, default=1)

    status_choices = (
        (1, "已占用"),
        (2, "未使用")
    )
    status = models.SmallIntegerField(verbose_name="状态", choices=status_choices, default=2)
```

### 9.2 新增号码

#### 1. 手机号格式-两种方法：

方法一：

1. 先导入手机号格式检查的包 from django.core.validators import RegexValidator
2. 定义格式：mobile = forms.CharField(
           label="手机号",
           validators=[RegexValidator(r'^1[3-9]\d{9}$', '手机号格式错误'), ],
       )
3. 利用modelform创建表单

```python
from django.core.validators import RegexValidator
from django.core.exceptions import ValidationError
class PrettyModelForm(forms.ModelForm):
    # 验证
    # 方式一：正则表达式，检查手机号格式
    mobile = forms.CharField(
        label="手机号",
        validators=[RegexValidator(r'^1[3-9]\d{9}$', '手机号格式错误'), ],
    )
    class Meta:
        model = models.PrettyNum
        fields = ["mobile","price","level","status"]

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        for name, field in self.fields.items():
            field.widget.attrs = {"class": "form-control", "placeholder": field.label}
    # 验证：
    # 方式二：
    def clean_mobile(self):
        txt_mobile = self.cleaned_data["mobile"]
        if len(txt_mobile)!=11:
            raise ValidationError("格式错误")
        return txt_mobile
```

方法二：

- 导入from django.core.exceptions import ValidationError

- 前提是定义了字段名，比如此处就是在fields中定义了mobile
- def clean_后面自动生成这个字段名

#### 2. 添加手机号时不能重复：

在上面的方法二中的函数中定义一个exists函数，判断是否已存在

```python
def clean_mobile(self):
    txt_mobile = self.cleaned_data["mobile"]
    exists = models.PrettyNum.objects.filter(mobile="txt_mobile").exists()
    if exists:
        raise ValidationError("手机号已存在")
    return txt_mobile
```

### 9.3 编辑号码

```python
def clean_mobile(self):
        txt_mobile = self.cleaned_data["mobile"]
        exists = models.PrettyNum.objects.exclude(id=self.instance.pk).filter(mobile=txt_mobile).exists()
        if exists:
            raise ValidationError("手机号已存在")
        return txt_mobile
```

这一段主要用来处理编辑号码时，排除自己的id对应的号码，不能和其他号码重合

```python
class PrettyEditModelForm(forms.ModelForm):
  	# 主要处理当前手机号不允许修改，disable=true
    # mobile = forms.CharField(disabled=True,label="手机号")
    class Meta:
        model = models.PrettyNum
        fields = ["mobile","price","level","status"]

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        for name, field in self.fields.items():
            field.widget.attrs = {"class": "form-control", "placeholder": field.label}

    def clean_mobile(self):
        txt_mobile = self.cleaned_data["mobile"]
        exists = models.PrettyNum.objects.exclude(id=self.instance.pk).filter(mobile=txt_mobile).exists()
        if exists:
            raise ValidationError("手机号已存在")
        return txt_mobile
      
def pretty_edit(request,nid):
    row_object = models.PrettyNum.objects.filter(id=nid).first()
    if request.method == "GET":
        form = PrettyEditModelForm(instance=row_object)
        return render(request,
                  "pretty_edit.html",
                  {"form":form})

    form = PrettyEditModelForm(data=request.POST,instance=row_object)
    if form.is_valid():
        form.save()
        return redirect('/pretty/list/')

    return render(request,
                  "pretty_edit.html",
                  {"form": form})
```

### 9.4 查找号码-手机号

```python
# 查询mobile=19999999991，id=12的结果
models.PrettyNum.objects.filter(mobile="19999999991",id=12)
# 上下两个结果相同，只是用了字典
# 先定义一个字典
data_dict = {"mobile":"19999999991","id":123}
models.PrettyNum.objects.filter(**data_dict)
```



```python
models.PrettyNum.objects.filter(id=12)       # 等于12
models.PrettyNum.objects.filter(id__gt=12)   # 大于12
models.PrettyNum.objects.filter(id__gte=12)  # 大于等于12
models.PrettyNum.objects.filter(id__lt=12)   # 小于12
models.PrettyNum.objects.filter(id__lte=12)  # 小于等于12

data_dict = {"id__lte":12}
models.PrettyNum.objects.filter(**data_dict)
```

```python
models.PrettyNum.objects.filter(mobile="999")               # 等于
models.PrettyNum.objects.filter(mobile__startswith="1999")  # 筛选出以1999开头
models.PrettyNum.objects.filter(mobile__endswith="999")     # 筛选出以999结尾
models.PrettyNum.objects.filter(mobile__contains="999")     # 筛选出包含999

data_dict = {"mobile__contains":"999"}
models.PrettyNum.objects.filter(**data_dict)
```

```python
    data_dict = {}
    search_data = request.GET.get("q","")
    if search_data:
        data_dict["mobile__contains"] = search_data

    queryset = models.PrettyNum.objects.filter(**data_dict)
    return render(request,
                  "pretty_list.html",
                  {"search_data":search_data,"queryset":queryset})
```

- 定义一个空字典
- 从浏览器中得到q对应的值，即我们输入的值为search_data
- 如果这个值不为空，就把这个值存到定义好的字典当中
- 然后从数据库中去查找mobile_contains对应的值是否存在，并存为queryset

### 9.5 分页

```python
queryset = models.PrettyNum.objects.all()

queryset = models.PrettyNum.objects.filter(id=1)[0:10]


# 第1页
queryset = models.PrettyNum.objects.all()[0:10]

# 第2页
queryset = models.PrettyNum.objects.all()[10:20]

# 第3页
queryset = models.PrettyNum.objects.all()[20:30]
```

```python
data = models.PrettyNum.objects.all().count()
data = models.PrettyNum.objects.filter(id=1).count()
```

- 分页的逻辑和处理规则
- 封装分页类
  - 从头到尾开发
  - 写项目用【pagination.py】公共组件。

def pretty_list(request):

```python
data_dict = {}
search_data = request.GET.get("q","")
if search_data:
    data_dict["mobile__contains"] = search_data

# 在浏览器中输入的page的值，默认为1
page = int(request.GET.get("page",1))
pagesize = 10
start = (page - 1) * pagesize
end = page * pagesize

# 数据总条数
total_count = models.PrettyNum.objects.filter(**data_dict).count()
total_page_count,div = divmod(total_count,pagesize)
if div:
    total_page_count+=1
# 总页码
queryset = models.PrettyNum.objects.filter(**data_dict)[start:end]
# 计算出显示前5页后5页
plus=5
if total_page_count<= 2 * plus + 1:
    # 数据库中的数据比较少，都没有达到11页
    start_page=1
    end_page=total_page_count + 1
else:
    # 数据库中的数据比较多 大于11页

    # 当前页<5时，不显示负数，从1开始显示
    if page <= plus:
        start_page = 1
        end_page = 2 * plus + 1 + 1
    else:
        # 当前页>5
        # 当前页+5 > 总页面
        if (page+plus)>total_page_count:
            start_page = total_page_count-2*plus
            end_page = total_page_count+1
        else:
            start_page = page-plus
            end_page = page+plus + 1
# 页码
page_str_list=[]

# 首页
page_str_list.append('<li><a href="?page={}">首页</a></li>'.format(1))

# 上一页
if page > 1:
    prev = '<li><a href="?page={}">上一页</a></li>'.format(page-1)
else:
    prev = '<li><a href="?page={}">上一页</a></li>'.format(1)
page_str_list.append(prev)

# 页面
for i in range(start_page,end_page):
    if(i==page):
        ele = '<li class="active"><a href="?page={}">{}</a></li>'.format(i,i)
    else:
        ele = '<li><a href="?page={}">{}</a></li>'.format(i, i)
    page_str_list.append(ele)

# 下一页
if page < total_page_count:
    prev = '<li><a href="?page={}">下一页</a></li>'.format(page + 1)
else:
    prev = '<li><a href="?page={}">下一页</a></li>'.format(total_page_count)
page_str_list.append(prev)

# 尾页
page_str_list.append('<li><a href="?page={}">尾页</a></li>'.format(total_page_count))
# 搜索
search_string = """
            <li>
                <form style="float: left;margin-left: -1px" method="get">
                    <input name="page"
                           style="position: relative;float:left;display: inline-block;width: 80px;border-radius: 0;"
                           type="text" class="form-control" placeholder="页码">
                    <button style="border-radius: 0" class="btn btn-default" type="submit">跳转</button>
                </form>
            </li>
            """
page_str_list.append(search_string)

page_string = mark_safe("".join(page_str_list))
return render(request,
              "pretty_list.html",
              {"search_data":search_data,"queryset":queryset,"page_string":page_string})
```



- 小Bug，搜索 + 分页情况下。

  ```
  分页时候，保留原来的搜索条件
  
  http://127.0.0.1:8000/pretty/list/?q=888
  http://127.0.0.1:8000/pretty/list/?page=1
  
  http://127.0.0.1:8000/pretty/list/?q=888&page=23
  ```

  

## 10. 时间插件

```
<link rel="stylesheet" href="static/plugins/bootstrap-3.4.1/css/bootstrap.css">
<link rel="stylesheet" href="static/plugins/bootstrap-datepicker/css/bootstrap-datepicker.css">


<input type="text" id="dt" class="form-control" placeholder="入职日期">



<script src="static/js/jquery-3.6.0.min.js"></script>
<script src="static/plugins/bootstrap-3.4.1/js/bootstrap.js"></script>
<script src="static/plugins/bootstrap-datepicker/js/bootstrap-datepicker.js"></script>
<script src="static/plugins/bootstrap-datepicker/locales/bootstrap-datepicker.zh-CN.min.js"></script>


<script>
    $(function () {
        $('#dt').datepicker({
            format: 'yyyy-mm-dd',
            startDate: '0',
            language: "zh-CN",
            autoclose: true
        });

    })
</script>
```

## 11.ModelForm和BootStrap

- ModelForm可以帮助我们生成HTML标签。

  ```python
  class UserModelForm(forms.ModelForm):
      class Meta:
          model = models.UserInfo
          fields = ["name", "password",]
  
  form = UserModelForm()
  ```

  ```html
  {{form.name}}      普通的input框
  {{form.password}}  普通的input框
  ```

- 定义插件生成样式

  ```python
  class UserModelForm(forms.ModelForm):
      class Meta:
          model = models.UserInfo
          fields = ["name", "password",]
          widgets = {
              "name": forms.TextInput(attrs={"class": "form-control"}),
              "password": forms.PasswordInput(attrs={"class": "form-control"}),
              "age": forms.TextInput(attrs={"class": "form-control"}),
          }
  ```

  ```python
  class UserModelForm(forms.ModelForm):
      name = forms.CharField(
          min_length=3,
          label="用户名",
          widget=forms.TextInput(attrs={"class": "form-control"})
      )
  
      class Meta:
          model = models.UserInfo
          fields = ["name", "password", "age"]
  ```

  ```html
  {{form.name}}      BootStrap的input框
  {{form.password}}  BootStrap的input框
  ```

- 重新定义的init方法，批量设置

  ```python
  class UserModelForm(forms.ModelForm):
      class Meta:
          model = models.UserInfo
          fields = ["name", "password", "age",]
  
      def __init__(self, *args, **kwargs):
          super().__init__(*args, **kwargs)
          
          # 循环ModelForm中的所有字段，给每个字段的插件设置
          for name, field in self.fields.items():
  			field.widget.attrs = {
                  "class": "form-control", 
                  "placeholder": field.label
              }
  ```

  ```python
  class UserModelForm(forms.ModelForm):
      class Meta:
          model = models.UserInfo
          fields = ["name", "password", "age",]
  
      def __init__(self, *args, **kwargs):
          super().__init__(*args, **kwargs)
          
          # 循环ModelForm中的所有字段，给每个字段的插件设置
          for name, field in self.fields.items():
              # 字段中有属性，保留原来的属性，没有属性，才增加。
              if field.widget.attrs:
  				field.widget.attrs["class"] = "form-control"
  				field.widget.attrs["placeholder"] = field.label
              else:
                  field.widget.attrs = {
                      "class": "form-control", 
                      "placeholder": field.label
                  }
  ```

  ```python
  class UserEditModelForm(forms.ModelForm):
      class Meta:
          model = models.UserInfo
          fields = ["name", "password", "age",]
  
      def __init__(self, *args, **kwargs):
          super().__init__(*args, **kwargs)
          
          # 循环ModelForm中的所有字段，给每个字段的插件设置
          for name, field in self.fields.items():
              # 字段中有属性，保留原来的属性，没有属性，才增加。
              if field.widget.attrs:
  				field.widget.attrs["class"] = "form-control"
  				field.widget.attrs["placeholder"] = field.label
              else:
                  field.widget.attrs = {
                      "class": "form-control", 
                      "placeholder": field.label
                  }
  ```

  

- 自定义类

  ```python
  class BootStrapModelForm(forms.ModelForm):
      def __init__(self, *args, **kwargs):
          super().__init__(*args, **kwargs)
          # 循环ModelForm中的所有字段，给每个字段的插件设置
          for name, field in self.fields.items():
              # 字段中有属性，保留原来的属性，没有属性，才增加。
              if field.widget.attrs:
  				field.widget.attrs["class"] = "form-control"
  				field.widget.attrs["placeholder"] = field.label
              else:
                  field.widget.attrs = {
                      "class": "form-control", 
                      "placeholder": field.label
                  }
  ```

  ```python
  class UserEditModelForm(BootStrapModelForm):
      class Meta:
          model = models.UserInfo
          fields = ["name", "password", "age",]
  ```

  



### 操作

- 提取公共的类

  ![image-20211126175803303](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20211126175803303.png)
  ![image-20211126175826579](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20211126175826579.png)

  

- ModelForm拆分出来
  ![image-20211126175852716](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20211126175852716.png)

- 视图函数的归类
  ![image-20211126175927378](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20211126175927378.png)

  ![image-20211126175946996](/Users/panjie/Documents/typora/assets/image-20211126175946996.png)
