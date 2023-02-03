<h1 align="center">Vue学习笔记</h1>

## 脚手架文件结构
	node_modules
	public
		favicon.ico: 页签图标
		index.html: 主页面
	src
		assets: 存放静态资源
			logo.png
		components: 存放组件
			HelloWorld.vue
		App.vue: 汇总所有组件
		main.js: 入口文件
	.gitignore: git版本管理忽略的配置
	babel.config.js: babel的配置文件
	package.json: 应用包配置文件
	README.md: 应用描述文件
	package-lock.json: 包版本控制文件

## 关于不同版本的Vue
1. vue.js与vue.runtime.xxx.js的区别：  
	vue.js是完整版的Vue，包含核心功能+模板解析器  
	vue.runtime.xxx.js是运行版的Vue，只包含核心功能，没有模板解析器
2. 因为vue.runtime.xxx.js没有模板解析器，所以不能使用template配置项，需要使用render函数接收到的creatElement函数去指定具体内容
		
## vue.config.js配置文件
使用vue inspect > output.js可以查看到Vue脚手架的默认配置  
使用vue.config.js可以对脚手架进行个性化定制，详见：https://cli.vuejs.orz/zh
	
## ref属性
1. 被用来给元素或子组件注册引用信息（id的替代者）
2. 应用在html标签上获取的是真实DOM元素，应用在组件标签上是组件实例对象（vc）
3. 使用方式：  
	打标识：`<h1 ref="xxx">....</h1>` 或 `<School ref="xxx"></School>`  
	获取：`this.$refs.xxx`

## 配置项props
功能：让组件接收外部传过来的数据
1. 传递数据：`<Demo name="xxx"/>`
2. 接收数据：  
第一种(只接收)：`props: [name]`  
第二种(限制类型)：
``` javascript
props: {
	name: String
}
```
第三种(限制类型，限制必要性，指定默认值)：
``` javascript
props: {
	name: {
		type: String, //类型
		required: true, //必要性
		default: 'xxx' //默认值，不会与required同时存在
	}
}
```
> 备注：props是只读的，Vue底层监测对props的修改，若进行修改会发出警告，如果业务需求确实需要修改，那么可以复制props的内容到data中一份，然后去修改data中的数据

## mixin(混入)
功能：可以把多个组件共用的配置提取成一个混入对象  
使用方法：  
第一步定义混合：
``` javascript
{
	data(){...},
	methods: {...}
	...
}
```
第二步使用混入：  
1. 全局混入: `Vue.mixin(xxx)`  
2. 局部混入: `mixins: ['xxx']`

## 插件
功能：用于增强Vue  
本质：包含install方法的一个对象，install的第一个参数是Vue，第二个以后的参数是插件使用者传递的数据  
定义插件：
``` javascript
xxx.install = function (Vue, options) {
	//1.添加全局过滤器
	Vue.filter(...)
	
	//2.添加全局指令
	Vue.directive(...)
	
	//3.配置全局混入(合)
	Vue.mixin(...)
	
	//4.添加实例方法
	Vue.prototype.$myMethod = function () {...}
	Vue.prototype.$myProperty = xxx
}
```
使用插件: `Vue.use()`
	
## scoped样式
作用：让样式在局部生效，防止冲突  
写法：`<style scoped>`
	
## webStorage
1. 存储内容大小一般支持5MB左右（不同浏览器可能不一样）
2. 浏览器通过 `WIndow.sessionStorage` 和 `Window.localStorage` 属性实现本地存储机制
3. 相关API：  
	1. `xxxxxStorage.setItem('key','value');`  
		 接收一个键和值作为参数,把键值对添加到存储中,如果键名存在,则更新其对应的值
	2. `xxxxxStorage.getItem('key');`  
		 接收一个键名作为参数,返回键名对应的值
	3. `xxxxxStorage.removeItem('key');`  
		 接收一个键名作为参数,把该键名从存储中删除
	4. `xxxxxStorage.clear();`  
		 清空存储中所有数据

>备注:  
	1. sessionStorage存储的内容会随着浏览器窗口关闭而消失		
	2. localStorage存储的内容需手动清除才会消失		
	3. `xxxxxStorage.getItem('key')` 如果key对应的value获取不到,返回null		
	4. `JSON.parse(null)` 的结果依然是null

## 组件的自定义事件
1. 一种组件间的通信方式，适用于：子组件==>父组件
2. 使用场景：A是父组件，B是子组件，B想给A传数据，那么要在A中给B绑定自定义事件(事件的回调在A中)
3. 绑定自定义事件：  
	第一种：在父组件中：`<Demo @shijian="test" />` 或 `<Demo v-on:shijian="test" />`  
	第二种：在父组件中：
	``` javascript
		<Demo ref="demo" />
		......
		mounted() {
			this.$refs.xxx.$on('shijian', this.test函数)
		}
	```
	自定义事件只触发一次,可以使用 `once` 修饰符，或者 `$once` 方法
4. 触发自定义事件：`this.$emit('shijian', value)`
5. 解绑自定义事件：`this.$off('shijian')`
6. 组件上也可以绑定原生DOM事件，需要使用 `native` 修饰符
7. 通过 `this.$refs.xxx.$on('shijian', 回调)` 绑定自定义事件时，回调要么配置在methods中，要么用箭头函数，否则this指向会出问题

## 全局事件总线(GlobalEventBus)
	1.一种组件间通信的方式,适用于任意组件间通信
	2.安装全局事件总线:
		new Vue({
			...
			beforeCreate(){
				Vue.prototype.$bus = this //安装全局事件总线,$bus就是当前应用的vm
			},
			...
		})
	3.使用全局总线:
		① 接收数据:A组件想接收数据,则在A组件中给$bus绑定自定义事件,事件的回调留在A组件自身
			methods: {
				demo(data){...}
			},
			...
			mounted(){
				this.$bus.$on('xxx',this.demo)
			}
		② 提供数据: this.$bus.$emit('xxx',value)
	4.最好在beforeDestroy钩子中,用$off解绑当前组件用到的事件

## 消息订阅与发布(pubsub)
	1.一种组件间通信的方式,适用于任意组件间通信
	2.使用步骤
		① 安装pubsub: npm i pubsub-js
		② 引入: import pubsub from 'pubsub-js'
		③ 接收数据: A组件想接收数据,则在A组件中订阅消息,订阅的回调留在A组件自身
			methods: {
				demo(msgName,data){...}
			},
			...
			mounted(){
				this.pid = pubsub.subscribe('xxx',this.demo) //订阅消息
			}
		④ 提供数据: pubsub.publish('xxx',data) //发布消息
	3.最好在beforeDestroy钩子中,用pubsub.unsubscribe(this.pid)去取消订阅

## $nextTick
	1.语法: this.$nextTick(回调函数)
	2.作用: 在下一次DOM更新结束后执行其指定的回调
	3.什么时候用: 当改变数据后,要基于更新后的新DOM进行某些操作时,要在$nextTick所指定的回调函数中执行

## Vue封装的过度与动画
	1.作用: 在插入,更新或移除DOM元素时,在合适的时候给元素添加样式类名
	2.写法:
		准备好样式:
			元素进入的样式: 
				① v-enter: 进入的起点  //过度
				② v-enter-active: 进入的过程中  //动画 过度可选
				③ v-enter-to: 进入的终点  //过度
			元素离开的样式:
				① v-leave: 离开的起点  //过度
				② v-leave-active: 离开的过程中  //动画 过度可选
				③ v-leave-to: 离开的终点  //过度
			进起=离终  进终=离起
		使用<transition>包裹要过度的元素,并配置name属性:
			<transition name="hello" appear>  //appear表示加载完毕执行一次出现动画
				<h1 v-show="isShow">你好</h1>
			</transition>
		备注: 若有多个元素需要过度,则需使用<transition-group>,且每个元素都要指定唯一key值

## Vue脚手架配置代理
	方法一
		在vue.config.js中添加如下配置
			devServer:{
				proxy:"http://localhost:5000"
			}
		说明
			1.优点：配置简单，请求资源时直接发给前端(8080)即可。
			2.缺点：不能配置多个代理，不能灵活的控制请求是否走代理。
			3.工作方式：若按照上述配置代理，当请求了前端不存在的资源时，那么该请求会转发给服务器（优先匹配前端资源）
	方法二
		编写vue.config.js配置具体代理规则
		module.exports = {
			devServer:{
				proxy:{
					'/api1':{    //匹配所有以'/api1'开头的请求路径
						target:'http://localhost:5000',  //代理目标的基础路径
						changeOrigin:true,
						pathRewrite:{'/api1':''}
					},
					'/api2':{    //匹配所有以'/api2'开头的请求路径
						target:'http://localhost:5001',/代理目标的基础路径
						changeOrigin:true,
						pathRewrite:{'/api2':''}
					}					
				}
			}
		}
		changeOrigin设置为true时，服务器收到的请求头中的host为：localhost:5000
		changeOrigin设置为falsef时，服务器收到的请求头中的host为：localhost:8080
		changeOriging默认值为true
		说明
			1.优点：可以配置多个代理，且可以灵活的控制请求是否走代理。
			2.缺点：配置略微繁琐，请求资源时必须加前缀。

## 插槽
	1.作用：让父组件可以向子组件指定位置插入html结构，也是一种组件间通信的方式，适用于父组件==>子组件。
	2.分类：默认插槽、具名插槽、作用域插槽
	3.使用方法：
		① 默认插槽：
			父组件中：
				<Category>
					<div>html结构1</div>
				</Category>
			子组件中：
				<template>
					<div>
						<!-- 定义插槽 -->
						<slot>插槽默认内容..</slot>
					</div>
				</template>
		② 具名插槽：
			父组件中：
				<Category>
					<template slot="center">
						<div>html结构1</div>
					</template>
					
					<template v-slot:footer>
						<div>html结构2</div>
					</template>
				</Category>
			子组件中：
				<template>
					<div>
						<!-- 定义插槽 -->
						<slot name="center">插槽默认内容..</slot>
						<slot name="footer">插槽默认内容..</slot>
					</div>
				</template>
		③ 作用域插槽：
			理解：数据在组件的自身，但根据数据生成的结构需要组件的使用者来决定。(games数据在Category组件中，但使用数据所遍历出来的结构由App组件决定)
			具体编码：
				父组件中：
					<Category>
						<template scope="scopeData">
							<!-生成的是ul列表-->
							<ul>
								<li v-for="g in scopeData.games"key="g">{{g}}</li>
							</ul>
						</template>
					</Category>
					
					<Category>
						<template slot-scope="scopeData">
							<!-- 生成的是h4标题 -->
							<h4 v-for="g in scopeData.games"key="g">{{g}}</h4>
						</template>
					</Category>
				子组件中：
					<template>
						<div>
							<slot :games="games"></slot>
						</div>
					</template>
					<script>
						export default {
							name:'Category',
							props:['title'],
							//数据在子组件自身
							data(){
								return {
									games:['红色警戒'，‘穿越火线'，'劲舞团'，'超级玛丽']
								}
							}
						}
					</script>
		v-slot:aaa            具名插槽 可简写为#aaa [aaa]可以动态指定参数
		v-slot="bbb"          作用域插槽 可简写为v-slot:default="bbb"  ==> #default="bbb"
		v-slot:item="ccc"     具名作用域 可简写为#item="ccc"

## Vuex
### 1.概念
&emsp;在Vue中实现集中式状态(数据)管理的一个Vue插件，对Vue应用中多个组件的共享状态进行集中式管理(读/写)，也是一种组件间的通信方式，且适用于任意组件间通信。  
### 2.何时使用？  
&emsp;1.多个组件依赖于同一状态(数据)  
&emsp;2.来自不同组件的行为需要变更同一状态(数据)  
&emsp;(多个组件需要共享数据时)
### 3.搭建Vuex环境
1. 创建文件：`src/store/index.js`
``` javascript
	import Vue from 'vue'
	import Vuex from 'vuex'
	
	Vue.use(Vuex)
	
	//相应组件中的动作
	const actions = {}
	//操作数据(state)
	const mutations = {}
	//存储数据
	const state = {}
	
	export default new Vuex.Store({
		actions,
		mutations,
		state
	})
```
2. 在 `main.js` 中创建vm时传入 `store` 配置项
``` javascript
	import store from './store'

	new Vue({
		render: h => h(App),
		store
	}).$mount('#app')
```

### 4.基本使用
1. 初始化数据、配置 `actions`、配置 `mutations`，操作文件 `store.js`
``` javascript
	import Vue from 'vue'
	import Vuex from 'vuex'

	Vue.use(Vuex)

	//相应组件中的动作
	const actions = {
		// 直接转发不进行逻辑处理的可直接通过mutations处理
		jia(context, value) {
			console.log('actions')
			context.commit('JIA', value)
		},
		jiaOdd(context, value) {
			console.log('actions')
			console.log('处理')
			context.dispatch('demo1', value) // 逻辑处理的数据过多可以交给另一个action处理
		},
		demo1(context, value){
			console.log('处理')
			if (context.state.sum % 2) {
				context.commit('JIA', value)
			}
		}
	}
	//操作数据(state)
	const mutations = {
		JIA(state, value) {
			console.log('mutations')
			state.sum += value
		}
	}
	//存储数据
	const state = {
		sum: 0
	}

	export default new Vuex.Store({
		actions,
		mutations,
		state
	})
```
2. 组件中读取vuex中的数据：`$store.state.sum`
3. 组件中修改vuex中的数据：`$store.dispatch('actions中的方法名', 数据)` 或 `$store.commit('mutations中的方法名', 数据)`，如果直接修改state中的数据，即 `$store.state.xxx=0`，则会导致状态不能被调试工具跟踪到，如果开启了严格模式 `strict: true`，则会报错
> 备注：若没有网络请求或其他业务逻辑，组件也可以越过actions，即不写 `dispatch`，直接编写 `commit`

### 5.getters的使用
1. 概念：当state中的数据需要经过加工后再使用时，可以使用getters加工。
2. 在 `store.js` 中追加 `getters` 属性
``` javascript
	······
	const getters = {
		bigNum(state) {
			return state.sum * 10
		}
	}
	
	export default new Vuex.Store({
		······
		getters
	})
```
3. 组件中读取数据：`$store.getters.bigNum`

### 6.四个map方法的使用
1. **mapState方法：**用于帮助映射 `state` 中的数据为计算属性
``` javascript
computed: {
	// 借助mapState生成计算属性，从state中读取数据。（对象写法）
	...mapState({he: 'sum',xuexiao:'school',xueke:'subject'}),
	
	// 借助mapState生成计算属性，从state中读取数据。（数组写法）
	...mapState(['sum','school','subject'])
}
```
2. **mapGetters方法：**用于帮助映射 `getters` 中的数据为计算属性
``` javascript
computed: {
	// 借助mapGetters生成计算属性，从getters中读取数据。（对象写法）
	...mapGetters({bigSum: 'bigSum'}),
	
	// 借助mapGetters生成计算属性，从getters中读取数据。（数组写法）
	...mapGetters(['bigSum'])
}
```
3. **mapActions方法：**用于帮助生成与 `actions` 对话的方法，即：包含 `$store.dispatch(xxx)` 的函数
``` javascript
methods: {
	// 借助mapActions生成对应方法，方法中会调用dispatch去联系actions。（对象写法）
	...mapActions({incrementOdd: 'jiaOdd',incrementWait: 'jiaWait'}),
	
	// 借助mapActions生成对应方法，方法中会调用dispatch去联系actions。（数组写法）
	...mapActions(['jiaOdd','jiaWait'])
}
```
4. **mapMutations方法：**用于帮助生成与 `mutations` 对话的方法，即：包含 `$store.commit(xxx)` 的函数
``` javascript
methods: {
	// 借助mapMutations生成对应方法，方法中会调用commit去联系mutations。（对象写法）
	...mapMutations({increment: 'JIA',decrement: 'JIAN'}),
	
	// 借助mapMutations生成对应方法，方法中会调用commit去联系mutations。（数组写法）
	...mapMutations(['JIA','JIAN'])
}
```
> 备注：mapActions与mapMutations使用时，若需要传递参数，需要在模板中绑定好参数，否则参数是事件对象（默认的$event）

### 7.模块化+命名空间
1. 目的：让代码更好维护，让多种数据分类更加明确。
2. 修改 `store.js`
``` javascript
	const countAbout = {
		namespaced: true,//开启命名空间
		actions: { ... },
		mutations: { ... },
		state: {x:1},
		getters: {
			bigSum(state) {
				return state.sum * 10
			}
		}
	}

	const personOption = {
		namespaced: true,//开启命名空间
		actions: { ... },
		mutations: { ... },
		state: {x:1},
		getters: {
			bigSum(state) {
				return state.sum * 10
			}
		}
	}
	
	const store = new Vuex.store({
		modules: {
			countAbout,
			personAbout: personOption
		}
	})
```
3. 开启命名空间后，组件中读取state数据：
``` javascript
//方式一：直接读取
this.$store.state.personAbout.personList
//方式二：借助mapState读取
...mapState('countAbout',['sum','school','subject'])
```
4. 开启命名空间后，组件中读取getters数据：
``` javascript
//方式一：直接读取
this.$store.getters['personAbout/firstPersonName']
//方式二：借助mapGetters读取
...mapGetters('countAbout',['bigSum'])
```
5. 开启命名空间后，组件中调用dispatch：
``` javascript
//方式一：直接调用
this.$store.dispatch('personAbout/addPersonWang',person)
//方式二：借助mapActions
...mapActions('countAbout',{incrementOdd: 'jiaOdd',incrementWait: 'jiaWait'})
```
6. 开启命名空间后，组件中调用commit：
``` javascript
//方式一：直接调用
this.$store.commit('personAbout/ADD_PERSON',person)
//方式二：借助mapMutations
...mapMutations('countAbout', {increment: 'JIA',decrement: 'JIAN'})
```

## 路由 更多内容查看官方文档
1. 理解：一个路由（route）就是一组映射关系（key-value），多个路由需要路由器（router）进行管理。
2. 前端路由：key是路径，value是组件。

### 1.基本使用
1. 安装vue-router，命令：`npm i vue-router`，vue2版本：`npm i vue-router@3`
2. 应用插件：`Vue.use(VueRouter)`
3. 编写router配置项：
``` javascript
	// 引入VueRouter
	import VueRouter from 'vue-router'
	// 引入组件
	import About from '../components/About'
	import Home from '../components/Home'

	//创建router实例对象，管理路由规则
	const router = new VueRouter({
		routes: [
			{
				path: '/about',
				component: About
			},
			{
				path: '/home',
				component: Home
			}
		]
	})

	export default router;
```
4. 实现切换（active-class可配置高亮样式）
``` html
<router-link to="/home" active-class="active">HOME</router-link>
```
5. 指定展示位置
``` html
<router-view></router-view>
```

### 2.几个注意点
1. 路由组件通常放在 `pages` 文件夹，一般组件通常放在 `compontents` 文件夹。
2. 通过切换，“隐藏”了的路由组件，默认是被销毁掉的，需要的时候再去挂载。
3. 每个组件都有自己的 `$route` 属性，里面存储着自己的路由信息。
4. 整个应用只有一个router，可以通过组件的 `$router` 属性获取到。

### 3.多级路由（嵌套路由）
1. 配置路由规则，使用children配置项：
``` javascript
routes: [{
	path: '/about',
	component: About
}, {
	path: '/home',
	component: Home,
	children: [{ //配置子路由
		path: 'news',//不能加'/'
		component: News
	}, {
		path: 'message',
		component: Message
	}]
}]
```
2. 跳转（要写完整路径）：
``` html
<router-link to="/home/news">NEWS</router-link>
```

### 4.路由的query参数
1. 传递参数：
``` html
	<!-- query参数，to的字符串写法 -->
	<router-link :to="'/home/message/detail?id='+i+'&title='+i">跳转</router-link>
	<!-- 模板语法，使用反引号 -->
	<router-link :to="`/home/message/detail?id=${i}&title=${i}`">跳转</router-link>

	<!-- query参数，to的对象写法 -->
	<router-link :to="{
		path: '/home/message/detail',
		query: {
			id: i,
			title: i
		}
	}">跳转</router-link>
```
2. 接收参数：
``` javascript
$route.query.id
$route.query.title
//注意是route不是router
```

### 5.命名路由
1. 作用：可以简化路由的跳转。
2. 使用：
	1. 给路由命名：
	``` javascript
	{
		path: '/home',
		component: Home,
		children: [{
			path: 'news',
			component: News
		}, {
			path: 'message',
			component: Message,
			children: [{
				path: 'detail',
				name: 'xiangqing',//给路由命名
				component: Detail
			}]
		}]
	}
	```
	2. 简化跳转：
	``` html
	<!-- 简化前 -->
	<router-link to="/home/message/detail">跳转</router-link>
	<!-- 简化后 -->
	<router-link :to="{name: 'xiangqing'}">跳转</router-link>
	<!-- 简化配合传参 -->
	<router-link :to="{
		name: 'xiangqing',
		query: {
			id: i,
			title: i
		}
	}">跳转</router-link>
	```

### 6.路由的params参数
1. 配置路由，声明接收params参数
``` javascript
{
	path: '/home',
	component: Home,
	children: [{
		path: 'news',
		component: News
	}, {
		path: 'message',
		name: 'xiaoxi',
		component: Message,
		children: [{
			path: 'detail/:id/:title',//使用占位符声明接收params参数
			name: 'xiangqing',
			component: Detail
		}]
	}]
}
```
2. 传递参数
```	html
	<!-- 模板语法，params参数，to的字符串写法 -->
	<router-link :to="`/home/message/detail/${i}/你好`">跳转</router-link>
				
	<!-- params参数，to的对象写法 -->
	<router-link :to="{
		name: 'xiangqing',
		params: {
			id: i,
			title: i
		}
	}">跳转</router-link>
```
> 特别注意：路由携带params参数时，若使用to的对象写法，则不能使用path配置项，必须使用name配置！
3. 接收参数：
```	javascript
$route.params.id
$route.params.title
```

### 7.路由的props配置
作用：让路由组件更方便的收到参数
```	javascript
{
	path: 'detail/:id/:title',//使用占位符声明接收params参数
	name: 'xiangqing',
	component: Detail,
	
	//第一种写法，值为对象，对象中的所有key-value都会以props的形式传给Detail组件。
	props: {a:'1',b:'2'},
	
	//第二种写法，值为布尔值，若布尔值为真，就会把该路由组件收到的所有params参数，以props的形式传给Detail组件。
	props: true,
	
	//第三种写法，值为函数，perams和query两种写法都可以
	props($route){
		return {
			id: $route.query.id,
			title: $route.query.title
		}
	},
	props({query}){
		return {id: query.id, title: query.title}
	},
	//解构赋值连续写法
	props({query: {id,title}}){
		return {id, title}
	}
}
```

### 8. `<router-link>` 的replace属性
1. 作用：控制路由跳转时操作浏览器历史记录的模式
2. 浏览器的历史记录有两种写入方式：分别为 `push` 和 `replace`，`push` 是追加历史记录，`replace` 是替换当前记录。路由跳转时默认为 `push`
3. 如何开启 `replace` 模式：`<router-link replace ......>ABOUT</router-link>`

### 9.编程式路由导航
1. 作用：不借助 `<router-link>` 实现路由跳转，让路由跳转更加灵活
2. 具体编码：
``` javascript
//$router的两个API
	this.$router.push({
		name: 'xiangqing',
		params: {
			aaa: 'aaa'
		}
	})
	
	this.$router.replace({
		name: 'xiangqing',
		params: {
			aaa: 'aaa'
		}
	})
	
	this.$router.forward() //前进
	this.$router.back() //后退
	this.$router.go(num) //可前进也可后退
```

### 10.缓存路由组件
1. 作用：让不展示的组件保持挂载，不被销毁。
2. 具体编码：
``` html
<keep-alive include="News">
	<router-view></router-view>
</keep-alive>
```
>备注：include属性表示缓存的路由组件，同组件中的name属性的值，不写则表示该标签内的所有组件都被缓存，缓存多个使用数组 `:include="['xxx','xxx']"`

### 11.两个新的生命周期钩子
1. 作用：路由组件所独有的两个钩子，用于捕获路由组件的激活状态，被 `keep-alive` 缓存的组件激活/失活时调用
2. 具体名字：
	1. `activated` 路由组件被激活时触发
	2. `deactivated` 路由组件失活时触发

### 12.路由守卫
1. 作用：对路由进行权限控制
2. 分类：全局守卫，独享守卫，组件内守卫，只有全局守卫分前后
3. 全局守卫：
``` javascript
	//全局前置路由守卫————初始化时调用、每次路由切换之前调用
	router.beforeEach((to, from, next) => {
		console.log('前置路由守卫', to, from)
		
		// document.title = to.meta.title || '初始'
		
		// if (to.path === '/home/news' || to.path === '/home/message') {
		// if (to.name === 'xinwen' || to.name === 'xiaoxi') {
		if (to.meta.isAuth) { //判断当前路由是否需要进行权限控制
			if (localStorage.getItem('school') === 'xuexiao') { //权限控制具体规则
				next() //放行
			} else {
				console.log('学校名错误')
			}
		} else {
			next() //放行
		}
	})
	
	//全局后置路由守卫————初始化时调用、每次路由切换之后调用
	router.afterEach((to, from) => {
		console.log('后置路由守卫', to, from)
		document.title = to.meta.title || '初始' //修改网页title
	})
```
4. 独享守卫：
``` javascript
{
	path: 'news',
	name: 'xinwen',
	component: News,
	meta: {isAuth: true,title: '新闻'},
	beforeEnter: (to, from, next) => {
		//... 同上方全局路由守卫的配置
		//独享守卫没有后置，可以配合全局后置守卫使用
	}
}
```
5. 组件内守卫：
``` javascript
//进入守卫：通过路由规则进入该组件时调用
beforeRouteEnter(to, from, next) {
	if (to.meta.isAuth) { //判断是否需要鉴权
		if (localStorage.getItem('school') === 'xuexiao') {
			next()
		} else {
			console.log('学校名错误')
		}
	} else {
		next()
	}
},
//离开守卫：通过路由规则离开该组件时调用
beforeRouteLeave(to, from, next) {
	next()
}
```

### 13.路由器的两种工作模式
1. 对于url来说，什么是hash值？———— #及其后面的内容就是hash值
2. hash值不会包含在HTTP请求中，即：hash值不会带给服务器
3. hash模式：
	1. 地址中永远带着#号，不美观。
	2. 若以后将地址通过第三方手机app分享，若app校验严格，则地址会被标记为不合法。
	3. 兼容性较好。
4. history模式：
	1. 地址干净，美观。
	2. 兼容性和hash模式相比略差
	3. 应用部署上线时需要后端人员支持，解决刷新页面服务端404的问题

<h1 align="center">123</h1>
<font color="red">123</font>

## 其他
在index.html中的
`<title><%= htmlWebpackPlugin.options.title %></title>`
读取的是package.json中的项目名
