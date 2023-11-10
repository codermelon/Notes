# 1. Vue-模板语法

- 插值语法
- 指令语法v-bind

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>模板语法</title>
    <script type="text/javascript" src="../js/vue.js"></script>
  </head>
  <body>
    <div id="root">
      <h1>插值语法</h1>
      <h3>你好,{{name}}</h3>
      <hr/>
      <h1>指令语法</h1>
      <a v-bind:href="school.url.toUpperCase()">我要去{{school.name}}</a>
      <a :href="school.url">我要去硅谷2</a>
    </div>
    <script type="text/javascript">
      Vue.config.productionTip = false
      new Vue({
        el:'#root',
        data:{
          name:'panjie',
          school:{
            name:'硅谷',
            url:'http://www.atguigu.com'
          }
        }
      })
    </script>
  </body>
</html>
```

# 2. 数据绑定

- v-model：value 双向绑定
- V-model:value 可以简写为 v-model

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>数据绑定</title>
    <script type="text/javascript" src="../js/vue.js"></script>
  </head>
  <body>
    <div id="root">
      单项数据绑定：<input type="text" v-bind:value="name"><br/>
      双向数据绑定：<input type="text" v-model:value="name"><br/>
      双向数据绑定：<input type="text" v-model="name"><br/>
      <!--v-model只能应用于表单元素（如：input,select）上-->
      <!--v-model:value可以简写为v-model-->
    </div>
    <script type="text/javascript">
      new Vue({
        el:'#root',
        data:{
          name:'硅谷'
        }
      })
      Vue.config.productionTip = false
    </script>
  </body>
</html>
```



# 3. el与data的两种写法

## 3.1 el的两种写法 

- 直接new出来

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript" src="../js/vue.js"></script>
  </head>
  <body>
    <div id="root">
        <h1>你好，{{name}}</h1>
    </div>
    <script type="text/javascript">
      Vue.config.productionTip = false
      new Vue({
        el:'#root',
        data:{
          name:'微软'
        }
      })
    </script>
  </body>
</html>
```

- Const v = new Vue

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <script type="text/javascript" src="../js/vue.js"></script>
    </head>
    <body>
      <div id="root">
          <h1>你好，{{name}}</h1>
      </div>
      <script type="text/javascript">
        Vue.config.productionTip = false
        const v=new Vue({
          data:{
            name:'微软'
          }
        })
        v.$mount('#root')
      </script>
    </body>
  </html>
  ```

![image-20230903225529149](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230903225529149.png)

## 3.2 data的两种写法

**第一种：**对象式

```html
new Vue({
  el:'#root',
  data:{
    name:'微软'
  }
})
```

**第二种**：函数式

```html
      new Vue({
        el:'#root',
        data(){
          return{
            name:'微软'
          }
        }
      })
```

- 目前两种方法都可以，但后面学到组件时，必须使用函数式

# 4. MVVM模型

![image-20230903230339898](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230903230339898.png)

# 5. 数据代理

## 5.1 回顾object.defineProperty

```html
<script type="text/javascript">
  let person = {
    name: '张三',
    sex: '男',
  }
  Object.defineProperty(person,'age',{
    value: 18
  })
  console.log(person)
</script>
```

控制台可以输出

![image-20230904224553856](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230904224553856.png)

但是name，sex 和 age的颜色不一样

**这种方法输出的age是不可以被枚举的，如果想被枚举需要加上属性**

```html
Object.defineProperty(person,'age',{
  value: 18,
  enumerable: true //控制属性是否可以枚举
})
```

- 这种方法定义的age不可修改



但是如果写成这种，age是可以被枚举出来的

```
<script type="text/javascript">
  let person = {
    name: '张三',
    sex: '男',
    age: 18,
  }
  console.log(person)
```

- 此处的age可以修改



重点：getter和setter方法

```html
<script type="text/javascript">
  let number = 18
  let person = {
    name: '张三',
    sex: '男',
    // age: 18,
  }
  Object.defineProperty(person,'age',{
    // value: 18,
    // enumerable: true, //控制属性是否可以枚举
    // writable: true, //控制属性是否可以修改
    // configurable:true, //控制属性是否可以被删除

    //当有人读取person的age属性时，get函数（getter）就会被调用，且返回值就是age的值
    get(){
      return number
    },
    //当有人修改person的age属性时，set函数（setter）就会被调用，且会收到修改的具体值
    set(value){
      console.log('有人修改了age属性，且值是',value)
      let number =value
    }
  })
  console.log(person)
</script>
```



## 5.2 数据代理

- 想通过obj2管理obj的数据
- 当访问obj2的x元素时，返回的是obj x的元素值

```html
<script type="text/javascript">
  let obj = {x:100}
  let obj2 = {y:200}

  Object.defineProperty(obj2,'x',{
    get(){
      return obj.x
    },
    set(value){
      obj.x = value
    }
  })
</script>
```

## 5.3 Vue中的数据代理

![image-20230904231415878](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230904231415878.png)

![image-20230904232451795](https://pj-typora.oss-cn-shanghai.aliyuncs.com/image-20230904232451795.png)

1.Vue中的数据代理：

​	通过vm对象来代理data对象中属性的操作（读/写）

2.Vue中数据代理的好处：

​	更加方便的操作data中的数据，不然都得{{_data.name}}

3.基本原理：

​	通过Object.defineProperty()把data对象中所有属性添加到vm上。

​	为每一个添加到vm上的属性，都指定一个getter/setter。

​	在getter/setter内部去操作（读/写）data中对应的属性。

# 6. 事件处理

## 6.1 事件的基本使用

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>事件的基本使用</title>
		<!-- 引入Vue -->
		<script type="text/javascript" src="../js/vue.js"></script>
	</head>
	<body>
		<!-- 
				事件的基本使用：
							1.使用v-on:xxx 或 @xxx 绑定事件，其中xxx是事件名；
							2.事件的回调需要配置在methods对象中，最终会在vm上；
							3.methods中配置的函数，不要用箭头函数！否则this就不是vm了；
							4.methods中配置的函数，都是被Vue所管理的函数，this的指向是vm 或 组件实例对象；
							5.@click="demo" 和 @click="demo($event)" 效果一致，但后者可以传参；
		-->
		<!-- 准备好一个容器-->
		<div id="root">
			<h2>欢迎来到{{name}}学习</h2>
			<!-- <button v-on:click="showInfo">点我提示信息</button> -->
			<button @click="showInfo1">点我提示信息1（不传参）</button>
			<button @click="showInfo2($event,66)">点我提示信息2（传参）</button>
		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

		const vm = new Vue({
			el:'#root',
			data:{
				name:'尚硅谷',
			},
			methods:{
				showInfo1(event){
					// console.log(event.target.innerText)
					// console.log(this) //此处的this是vm
					alert('同学你好！')
				},
				showInfo2(event,number){
					console.log(event,number)
					// console.log(event.target.innerText)
					// console.log(this) //此处的this是vm
					alert('同学你好！！')
				}
			}
		})
	</script>
</html>
```

## 6.2 事件修饰符

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>事件修饰符</title>
		<!-- 引入Vue -->
		<script type="text/javascript" src="../js/vue.js"></script>
		<style>
			*{
				margin-top: 20px;
			}
			.demo1{
				height: 50px;
				background-color: skyblue;
			}
			.box1{
				padding: 5px;
				background-color: skyblue;
			}
			.box2{
				padding: 5px;
				background-color: orange;
			}
			.list{
				width: 200px;
				height: 200px;
				background-color: peru;
				overflow: auto;
			}
			li{
				height: 100px;
			}
		</style>
	</head>
	<body>
		<!-- 
				Vue中的事件修饰符：
						1.prevent：阻止默认事件（常用）；
						2.stop：阻止事件冒泡（常用）；
						3.once：事件只触发一次（常用）；
						4.capture：使用事件的捕获模式；
						5.self：只有event.target是当前操作的元素时才触发事件；
						6.passive：事件的默认行为立即执行，无需等待事件回调执行完毕；
		-->
		<!-- 准备好一个容器-->
		<div id="root">
			<h2>欢迎来到{{name}}学习</h2>
			<!-- 阻止默认事件（常用） -->
			<a href="http://www.atguigu.com" @click.prevent="showInfo">点我提示信息</a>

			<!-- 阻止事件冒泡（常用） -->
			<div class="demo1" @click="showInfo">
				<button @click.stop="showInfo">点我提示信息</button>
				<!-- 修饰符可以连续写 -->
				<!-- <a href="http://www.atguigu.com" @click.prevent.stop="showInfo">点我提示信息</a> -->
			</div>

			<!-- 事件只触发一次（常用） -->
			<button @click.once="showInfo">点我提示信息</button>

			<!-- 使用事件的捕获模式 -->
			<div class="box1" @click.capture="showMsg(1)">
				div1
				<div class="box2" @click="showMsg(2)">
					div2
				</div>
			</div>

			<!-- 只有event.target是当前操作的元素时才触发事件； -->
			<div class="demo1" @click.self="showInfo">
				<button @click="showInfo">点我提示信息</button>
			</div>

			<!-- 事件的默认行为立即执行，无需等待事件回调执行完毕； -->
			<ul @wheel.passive="demo" class="list">
				<li>1</li>
				<li>2</li>
				<li>3</li>
				<li>4</li>
			</ul>

		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

		new Vue({
			el:'#root',
			data:{
				name:'尚硅谷'
			},
			methods:{
				showInfo(e){
					alert('同学你好！')
					// console.log(e.target)
				},
				showMsg(msg){
					console.log(msg)
				},
				demo(){
					for (let i = 0; i < 100000; i++) {
						console.log('#')
					}
					console.log('累坏了')
				}
			}
		})
	</script>
</html>
```

