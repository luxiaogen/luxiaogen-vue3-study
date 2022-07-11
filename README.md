# 一、创建Vue3.0工程
## 1.使用 vue-cli 创建

官方文档：https://cli.vuejs.org/zh/guide/creating-a-project.html#vue-create

```bash
## 查看@vue/cli版本，确保@vue/cli版本在4.5.0以上
vue --version
## 安装或者升级你的@vue/cli
npm install -g @vue/cli
## 创建
vue create vue_test
## 启动
cd vue_test
npm run serve
```

## 2.使用 vite 创建

官方文档：https://v3.cn.vuejs.org/guide/installation.html#vite

vite官网：https://vitejs.cn

- 什么是vite？—— 新一代前端构建工具。
- 优势如下：
  - 开发环境中，无需打包操作，可快速的冷启动。
  - 轻量快速的热重载（HMR）。
  - 真正的按需编译，不再等待整个应用编译完成。
- 传统构建 与 vite构建对比图


```bash
## 创建工程
npm init vite-app <project-name>
## 进入工程目录
cd <project-name>
## 安装依赖
npm install
## 运行
npm run dev
```

# 二、常用 Composition API
官方文档：https://v3.cn.vuejs.org/guide/composition-api-introduction.html、
## 1. 拉开序幕的 setup
1. 理解：Vu3.0中的一个新的配合项，值为一个函数
2. `setup`是所有的**Composition API(组合API)**，表演的舞台
3. 组件中所用到的：数据、方法等等，均要配置在`setup`中
4. `setup`函数的两种返回值：
   - 若返回一个对象，则对象中的属性、方法，在模板中均可以直接使用。(!!!!)
   - <span style="color:#aad">若返回的是渲染函数：则可以自定义渲染内容(了解)</span>
5. 注意点：
   1. 尽量不要与Vue2.x配置混用
     - Vue2.x配置(data、methods、computed..)中**可以访问到**setup中属性、方法
     - 但是setup中不能访问到Vue2.x的配置(data、methods、computed..)
     - 如果有重名，setup优先
   2. setup不能是`async`函数，因为返回值不再是`return`的对象，而是`promise`
      模板看不到`return`对象中的属性(后期也可返回一个Promise实例，但需要Suspense和异步组件的配合)
6. 示例代码：
```js
import { ref } from 'vue';
export default {
  name: 'App',
  // 此处只是测试一下setup，暂时不考虑响应式的问题
  setup() {
    // 数据
    let name = '张三'
    return {name}
  },
}
```
## 2. ref 函数
- 作用：定义一个响应式数据
- 语法：`const xxx = ref(initValue)`
  - 创建一个包含响应式数据的引用对象(`reference对象`，简称为ref对象)
  - JS中操作数据：`xxx.value`
  - 模板中读取数据：不需要.value，直接`{{xxx}}`
- 备注：
  - 接收的数据可以是：基本类型、也可以是对象类型
  - 基本类型的数据：响应依然是靠：`Object.defineProperty()`的`get`和`set`完成的
  - 对象类型的数据：内部 **求助** 了Vue3.0 中的一个新函数 ——— `reactive` 函数。
- 示例代码：
```js
setup() {
  let name = ref('张三')
  let job = ref({
    type: '后端工程师',
    salary: '3k'
  })
  function changeInfo() {
    name.value = '李四',
    job.value.type = '后端小子'
    job.value.salary = '2.5k'
  }
  return { name,job, changeInfo }
},
```


## 3. reactive 函数
- 作用：定义一个 **对象类型** 的响应式数据(基本类型不要用它，用`ref`函数)
- 语法：`const 代理对象 = reactive(源对象)`接收一个对象(或数组)，返回一个 **代理对象**(Proxy的实例对象,简称为proxy对象)
- `reactive`定义的响应式是"深层次的"
- 内部基于 ES6 的 Proxy 实现的，通过代理对象操作源对象内部数据进行操作。

## 4. Vue3.0中的响应式原理
### 1. Vue2.x的响应式
- 实现原理：
  - 对象类型：通过`Object.defineProperty()`对属性的读取、修改进行拦截(数据劫持)。
  - 数组类型：通过重写更新数组的一系列方法来实现拦截。（对数组的变更方法进行了包裹）
- 存在问题：
  - 新增属性、删除属性，界面不会更新
  - 直接通过下标修改属性，界面不会自动更新
### 2.Vue3.0的响应式
- 实现原理：
  - 通过`Proxy(代理)`：拦截对象中任意属性的变化，包括：属性值的读取、属性的添加、属性的删除等。
  - 通过`Reflect(反射)`: 对源对象的属性进行操作。
  - MDN文档
    - `Proxy`：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
    - `Reflect`：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect
```js
new Proxy(data, {
	// 拦截读取属性值
    get (target, prop) {
    	return Reflect.get(target, prop)
    },
    // 拦截设置属性值或添加新属性
    set (target, prop, value) {
    	return Reflect.set(target, prop, value)
    },
    // 拦截删除属性
    deleteProperty (target, prop) {
    	return Reflect.deleteProperty(target, prop)
    }
})

proxy.name = 'tom'   
```

## 5. reactive 对比 ref
- 从定义数据角度对比：
  - ref用来定义：基本类型的数据
  - reactive用来定义：对象(或数组)类型数据
  - 备注：ref也可以用来定义对象(或数组)类型数据，它内部会自动通过`reactive`转为代理对象


## 6. setup的两个注意点
- `setup`执行的时机
  - 在`beforeCreate`之前执行一次，`this`是`undefined`
- `setup`的参数
  - `props`：值为对象，包含：组件外部传递过来，且组件内部声明接收了属性
  - `context`：上下文对象
    - `attrs`：值为对象，包含：组件外部传递过来，没有在`props`配置中声明的属性，相当于`this.$attrs`
    - `slots`：收到的插槽内容，相当于`this.$slots`
    - `emit`：分发自定义事件的函数，相当于`this.$emit`


## 7. 计算属性与监视
### 1. computed 函数
- 与Vue2.x中`computed`配置功能一致
- 写法
```js
import {computed} from 'vue'

setup(){
    ...
		//计算属性——简写
    let fullName = computed(()=>{
        return person.firstName + '-' + person.lastName
    })
    //计算属性——完整
    let fullName = computed({
        get(){
            return person.firstName + '-' + person.lastName
        },
        set(value){
            const nameArr = value.split('-')
            person.firstName = nameArr[0]
            person.lastName = nameArr[1]
        }
    })
}
```
### 2. watch 函数
- 与Vue2.x中的`watch`配置功能一致
- `ref`定义的数据 默认不开启 深度监视
- 两个小"坑"
  - 监视`reactive`定义的响应式数据时：`oldValue`无法正确获取、强制开启了深度监视(`deep`配置失效)
  - 监视`reactive`定义的响应式数据中某个属性(值为对象)时：`deep`配置有效
```js
//情况一：监视ref定义的响应式数据
watch(sum,(newValue,oldValue)=>{
	console.log('sum变化了',newValue,oldValue)
},{immediate:true})

//情况二：监视多个ref定义的响应式数据
watch([sum,msg],(newValue,oldValue)=>{
	console.log('sum或msg变化了',newValue,oldValue)
}) 

/* 情况三：监视reactive定义的响应式数据
			若watch监视的是reactive定义的响应式数据，则无法正确获得oldValue！！
			若watch监视的是reactive定义的响应式数据，则强制开启了深度监视 
*/
watch(person,(newValue,oldValue)=>{
	console.log('person变化了',newValue,oldValue)
},{immediate:true,deep:false}) //此处的deep配置不再奏效

//情况四：监视reactive定义的响应式数据中的某个属性
watch(()=>person.job,(newValue,oldValue)=>{
	console.log('person的job变化了',newValue,oldValue)
},{immediate:true,deep:true}) 

//情况五：监视reactive定义的响应式数据中的某些属性
watch([()=>person.job,()=>person.name],(newValue,oldValue)=>{
	console.log('person的job变化了',newValue,oldValue)
},{immediate:true,deep:true})

//特殊情况
watch(()=>person.job,(newValue,oldValue)=>{
    console.log('person的job变化了',newValue,oldValue)
},{deep:true}) //此处由于监视的是reactive素定义的对象中的某个属性，所以deep配置有效
```

### 3. watchEffect 函数
- `watch`的套路是：既要指明监视的属性，也要指明监视的回调
- `watchEffect`的套路是：不用指明监视哪个属性，监视的回调中 **用到哪个属性，那就监视哪个属性。**
- `watchEffect`有点像`computed`：
  - 但`computed`注重的就算出来的值(回调函数的返回值)，所以必须要写返回值
  - 而`watchEffect`更注重的是过程(回调函数的函数体)，所以不用写返回值
```js
// watchEffect所指定的回调中用到的数据只要发生变化，则直接重新执行回调。
watchEffect(()=>{
    const x1 = sum.value
    const x2 = person.age
    console.log('watchEffect配置的回调执行了')
})
```

## 8. 生命周期
<div style="border:1px solid black;width:380px;float:left;margin-right:20px;"><strong>vue2.x的生命周期</strong><img src="https://cn.vuejs.org/images/lifecycle.png" alt="lifecycle_2" style="zoom:33%;width:1200px" /></div><div style="border:1px solid black;width:510px;height:985px;float:left"><strong>vue3.0的生命周期</strong><img src="https://v3.cn.vuejs.org/images/lifecycle.svg" alt="lifecycle_2" style="zoom:33%;width:2500px" /></div>
- Vue3.0可以继续使用Vue2.x中的生命周期钩子，但有两个被更名：
  - `beforeDestory`改名为`beforeUnmounted`
  - `destory`改名为`unmounted`
- Vue3.0也提供了 `Composition API` 形式的生命周期钩子，与Vue2.x中钩子对应关系如下：
  - - `beforeCreate`===>`setup()`
  - `created`====>`setup()`
  - `beforeMount` ===>`onBeforeMount`
  - `mounted`====>`onMounted`
  - `beforeUpdate`===>`onBeforeUpdate`
  - `updated` ====>`onUpdated`
  - `beforeUnmount`====>`onBeforeUnmount`
  - `unmounted` ===>`onUnmounted`

## 9. 自定义hook函数
- 什么是 hook? -- 本质就是一个函数，把`setup`函数中使用的Composition API 进行了封装。
- 类似于Vue2.x中`mixin`（混合）
- 自定义hook的优势：复用代码，让`setup`中的逻辑更清楚易懂。

## 10. toRef
- 作用：创建一个`ref`对象，其`value`值指向另一个对象中的某个属性
- 语法：`const name = toRef(person, 'name')`\
- 应用：要将响应式对象中的某个属性单独提供给外部使用时。
- 扩展：`toRefs`与`toRef`功能一致，但可以批量创建多个`ref`对象，语法：`toRefs(person)`


# 三、其它的 Composition API
## 1. shallowReactive 与 shallowRef
- `shallowReactive`：只处理对象最外层的响应式(浅响应式)
- `shallowRef`：只处理基本数据类型的响应式，不进行对象的响应式处理。
- 什么时候使用？
  - 如果有一个对象数据，结构比较深，但变化时只是外层属性变化 ==> `shallowReactive`
  - 如果有一个对象数据，后续功能不会修改该对象中的属性，而是生成新的对象来替换 ==> `shallowRef`

## 2. readonly 与 shallowReadonly
- `readonly`：让一个响应式数据变为只读的(深只读)。
- `shallowReadonly`：让一个响应式数据变为只读的(浅只读)。
- 应用场景：不希望数据被修改时，而且这个数据可能是别人传给你的。

## 3. toRaw 与 markRaw
- `toRaw`：
  - 作用：将一个由`reactive`生成的响应式对象转为普通对象。
  - 使用场景：用于读取响应式对象对应的普通对象，对这个普通对象的所有操作，不会引起页面更新。
- `markRaw`
  - 作用：标记一个对象，使其永远不会再成为响应式对象。
  - 应用场景：
    1. 有些值不应该设置为响应式的，例如复杂的第三方类库等。(axios..)
    2. 当渲染具有不可变数据源的大列表时，跳过响应式转换可以提高性能。

## 4. customRef
- 作用：创建一个自定义的`ref`，并对其依赖项跟踪和更新触发进行显示控制。
- 实现防抖效果：
```js
<template>
	<input type="text" v-model="keyword">
	<h3>{{keyword}}</h3>
</template>

<script>
	import {ref,customRef} from 'vue'
	export default {
		name:'Demo',
		setup(){
			// let keyword = ref('hello') //使用Vue准备好的内置ref
			//自定义一个myRef
			function myRef(value,delay){
				let timer
				//通过customRef去实现自定义
				return customRef((track,trigger)=>{
					return{
						get(){
							track() //告诉Vue这个value值是需要被“追踪”的
							return value
						},
						set(newValue){
							clearTimeout(timer)
							timer = setTimeout(()=>{
								value = newValue
								trigger() //告诉Vue去更新界面
							},delay)
						}
					}
				})
			}
			let keyword = myRef('hello',500) //使用程序员自定义的ref
			return {
				keyword
			}
		}
	}
</script>
```


  
## 5. provide 与 inject
<img src="https://v3.cn.vuejs.org/images/components_provide.png" style="width:300px" />

----
- 作用：实现 **祖与后代组件间**通信
- 套路：父组件有一个 `provide` 选项来体用数据，后代组件有一个 `inject` 选项来开始使用这些数据
- 具体写法：
1. 祖组件间中
```js
setup(){
	......
    let car = reactive({name:'奔驰',price:'40万'})
    provide('car',car)
    ......
}
```
2. 后代组件中
```js
setup(props,context){
	......
    const car = inject('car')
    return {car}
	......
}
```

## 6. 响应式数据的判断
- `isRef()`：检查一个值是否为 `ref` 对象
- `isReactive()`：检查一个对象是否是由 `reactive` 创建的响应式代理
- `isReadonly()`：检查一个对象是否是由 `readonly` 创建的只读代理
- `isProxy()`：检查一个对象是否是由 `reactive` 或者 `readonly` 方法创建的代理