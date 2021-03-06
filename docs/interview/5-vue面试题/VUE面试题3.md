# vue面试题3
[[toc]]
[TOC]
## 响应式的好处

通过getter和setter,修改数据就可以自动更新视图。开发者更多关注于业务逻辑而不是DOM操作

![image-20200920214545035](../../.vuepress/public/assets/img/image-20200920214545035.png)

## vue的优(特点)缺点

⾸先Vue最核⼼的两个特点，响应式和组件化。 

**响应式：**这也就是vue.js最⼤的优点，通过MVVM思想实现数据的双向绑定，通过虚拟DOM让我们可以⽤数据

来操作DOM，⽽不必去操作真实的DOM，提升了性能。且让开发者有更多的时间去思考业务逻辑。 

**组件化：**把⼀个单⻚应⽤中的各个模块拆分到⼀个个组件当中，或者把⼀些公共的部分抽离出来做成⼀个可复 

⽤的组件。所以组件化带来的好处就是，提⾼了开发效率，⽅便重复使⽤，使项⽬的可维护性更强。 

**虚拟DOM，**当然，这个不是vue中独有的。 

缺点：

vue2里基于对象的方式，很难从新添加新的对象属性，不能通过vm.a = xx来创建新的对象。                        

## 如何理解MVVM

MVVM指的是Model,View和ViewModel。vue使用mvvm的思想实现数据驱动视图，所谓的model就是指的我们的数据data,js对象。View就是指的我们的视图层，DOM，这两者之间通过ViewModel，也就是我们的vue进行连接，当和视图层关联的model层的数据发生变化时，就会执行DOM渲染，当监听到view层的事件变化，也会修改DOM层的数据，从而实现view层的DOM修改。vue的组件化采用的这种mvvm思想也是他和传统组件化的一个很大区别。因为传统的组件化只是静态渲染，更新依然需要开发者执行相应的dom操作。这样做的好处是让开发者更加关注业务逻辑而不是一些兼容性问题，如何优化dom渲染等等的。

![image-20200614141052141](D:/czw-project/learningblog/docs/.vuepress/public/assets/img/image-20200614141052141.png)

## 监听data变化的核心api是什么？

vue使用了Object.defineProperty这个API，来操作对象的属性描述符。这个方法可以精准的添加或者是修改对象的属性，属性描述符主要是分为了存取描述符和数据描述符，存取描述符就是我们的这个**getter**和**setter**,而属性描述符则是一个具有**value**和**writable**属性。这两类属性描述符还会有**configurable**和**enumerable**两个属性。代表该属性的描述符能否被改变删除和能否被枚举。但是这个方法是会破坏对象的原有结构，使得对象本身的结构不稳定，而v8引擎希望你对象的结构越稳定越好，所以这个方法的性能并不是那么好。

```js
const data = {};
const name = 'zhangsan';
Object.defineProperty(data,"name",{
	get:function(){
		return name;
	},
	set:function(newVal){
		name = newVal
	}
})
```

## Object.DefineProperty缺点是什么？

- 深度监听需要递归到底，计算量很大，所以数据结构设计时尽量不要嵌套太多层

- 无法监听到新增或者是删除属性

  ```js
  vm.c = 'a'//无法监听
  delete vm.b
  ```

- 无法原生监听数组，需要特殊处理

- ```js
  Object.defineProperty 对数组和对象的表现一直，并非不能监控数组下标的变化，vue2.x中无法通过数组索引来实现响应式数据的自动更新是vue本身的设计导致的，不是 defineProperty 的锅。
  Object.defineProperty 和 Proxy 本质差别是，defineProperty 只能对属性进行劫持，所以出现了需要递归遍历，新增属性需要手动 Observe 的问题。
  Proxy 作为新标准，浏览器厂商势必会对其进行持续优化，但它的兼容性也是块硬伤，并且目前还没有完整的polifill方案。
  ```

## vue如何深度监听data?缺点是什么？

```js
const data = {
  name: 'zs',
  age: 20
}
observer(data);
function observer(target) {
  if (typeof target !== 'object' || target === null) {
    //不是对象或者数组
    return target
  }
  for (let key in target) {
    // Reactive:反应
    defineReactive(target, key, target[key])
  }
}
function defineReactive(data, key, value) {
  observer(value);
  Object.defineProperty(data, key, {
    get() {
      //dep.addSub(Dep.target) 添加观察者
      return value
    },
    set(newVal) {
      if (newVal !== value) {
       	observer(value);
        value = newValue;
        //触发更新
        dep.notify()
      }
    }
  })
}
```

缺点：

- 深度监听，需要递归到底，一次性计算量大。

  ```
  data{
  	info:{
  		address:{
  			a:{
  				
  			}
  		}
  	}
  }
  ```

  如果这个对象非常大，页面初始化时就卡死了。

- 无法监听新增属性/删除属性(Vue.set Vue.delete解决,3.0不用)

  ```
  data.x = '100' 新增属性监听不到
  delete data.name 删除属性监听不到
  ```

## 如何实现数组监听？

为了防止污染全局原型。重写了个自己的原型。

Vue用Object.create方法创建了一个对象，并让他的原型指向数组原型，

然后在这个对象内部重写了push,

```js
const arrayPhoto = Object.create(Array.prototype);
['push','pop','shift','unshift','splice','reverse','sort'].forEach(method=>{
	arrayPhoto[method] = function(){
		updateView();//触发视图更新
		Array.prototype[method].call(this,...rest)
	}
})
```

监听判断

```js
function observer(target) {
  if (typeof target !== 'object' || target === null) {
    //不是对象或者数组
    return target
  }
  //
  if(Array.isArray(target)){
  	target.__proto__ = arrPhoto
  }
  for (let key in target) {
    defineReactive(target, key, target[key])
  }
}
```

## 虚拟DOM与diff算法

- DOM操作十分耗费性能
- JQ操作DOM,可以自行控制DOM操作时间，手动调整
- Vue和React是数据驱动视图，如何有效控制DOM操作?

**解决方案-vdom**

- 有一定复杂度，想减少计算次数很难
- 把计算转为JS计算
- 用JS模拟DOM结构，计算出最小的变更，操作DOM，最大成本避免无意义操作

![1584941250405](D:/czw-project/learningblog/docs/.vuepress/public/assets/img/1584941250405.png)

## diff算法是什么?

- diff就是对比的意思，比如linux的diff命令，git diff等等

- 两颗树做diff，比如这里的vdom diff

![1584943219806](D:/czw-project/learningblog/docs/.vuepress/public/assets/img/1584943219806.png)

**传统树diff的时间复杂度O(n^3) 不可用的时间复杂度**

- 第一,遍历tree1,第二,遍历tree2
- 第三,排序

- 1000个节点 计算一亿次，算法不可用

**优化时间复杂度到O(n)**

- 只同层比较

  ![1584943824584](D:/czw-project/learningblog/docs/.vuepress/public/assets/img/1584943824584.png)

- tag不相同，就删掉重建，不再深度比较

  ![1584943835019](D:/czw-project/learningblog/docs/.vuepress/public/assets/img/1584943835019.png)

  

- tag和key,两者都相同，就是相同节点，不再深度比较

## with语法

![1584949707022](D:/czw-project/learningblog/docs/.vuepress/public/assets/img/1584949707022.png)

![1584950519690](D:/czw-project/learningblog/docs/.vuepress/public/assets/img/1584950519690.png)

- template-complier把模板编译为render函数
- 执行render函数生产vnode(patch函数渲染出vnode)

## vue模板被编译成什么?

核心要点：

**通过vue-template-complier,模板会被转化成render 函数，这个函数执行后生成vNode。基于vnode再执行patch和diff**

**使用webpack或者cli vue-loader会在开发环境下编译模板，产生的代码全是render函数的形式**

```
npm init -y
npm i vue-template-compiler --save
```

```
index.js
const compiler = require('vue-template-complier')
const template = '<xx>x<xx>'
```

```js
<div id="div1" class="container">
        <img :src="imgUrl"/>
</div>
with (this) {
  return _c('div',
    { staticClass: "container", attrs: { "id": "div1" } },
    [_c('img', { attrs: { "src": imgUrl } })]
  )
}
```

条件:把ifelse变成三元表达式

```
const template = `
    <div>
        <p v-if="flag === 'a'">A</p>
        <p v-else>B</p>
    </div>
```

```
with(this){return _c('div',[(flag === 'a')?_c('p',[_v("A")]):_c('p',[_v("B")])])}
```

循环

```
const template = `
    <ul>
        <li v-for="item in list" :key="item.id">{{item.title}}</li>
    </ul>
```

```js
with(this){
    return _c('ul',
              _l(
        		(list),
                    function(item){
                        return _c('li',{key:item.id},[_v(_s(item.title))])
                    }
    			)
              ,0
             )
}
tag 属性，子元素
通过_l方法传入一个list数组，并传入函数，针对数组每一项返回一个li元素。

```

**事件**

```html
<button @click="clickHandler">submit</button>
```

```js
with(this){return _c('button',{on:{"click":clickHandler}},[_v("submit")])}
```

_c创建button,

里面有个属性on,给她传一个click,并参数是clickHandler

_v是*createTextVNode*，创建文本

 ***v-model***

```
<input type="text" v-model="name">
```

```js
with(this){return _c('input',{directives:[{name:"model",rawName:"v-model",value:(name),expression:"name"}],attrs:{"type":"text"},domProps:{"value":(name)},on:{"input":function($event){if($event.target.composing)return;name=$event.target.value}}})}

```

挂载Input事件，把当前input的$event.target.value赋值给this.name  value显示时显示name的变量

## vue组件使用render代替template

```js
Vue.component('heading',{
	render:function(creatElement){
		return creatElement(
			'h'+this.level,    h1标签
            [
                creatElement('a'),{  子元素a标签
                    attrs:{
                        name:'headerId',    属性
                        href:'#'+'headerId'
                    }
                },'this is a tag0')   他有个tag标签
            ]
		)
	}
})

```

- 有些复杂情况不能用template,可以考虑用render函数写。
- React一直用render

## 总结

- with语法
- 模板到render函数，再到vnode,再到渲染更新
- vue组件可以用render代替template

## 组件渲染/更新过程

- template->ast->render->virtuldom -> UI
- render->virtuldom ->Ui

![1584956825238](D:/czw-project/learningblog/docs/.vuepress/public/assets/img/1584956825238.png)

- 初次渲染过程：
  - 第一template解析为render函数(或者是开发环境已经完成，vue-loader，拿到的就是成品的rendr函数)
  - 第二触发响应式，监听data属性,geter setter 
  - 第三,执行render函数生成vnode ,执行patch(elem,vnode);
    - 模板内没有用到的data属性的改变不会触发渲染，因为和视图无关
- 更新过程：
  - 修改data，触发setter(此前getter里已经被监听)
  - 重新直接render函数，生成newVnode
  - patch(vnode,newvnode)
    - patch的diff算法会算出最小差异
- 异步渲染
  - $nextTick
    - vue组件是异步渲染，nextTick等DOM渲染完进行回调，页面渲染时会把data数据整合，多次data修改只渲染一次
  - 汇总data的修改，一次性渲染视图。
  - 减少DOM操作次数，提高性能。

## vue3为什么使用proxy

https://www.jb51.net/article/171869.htm

## vue首屏渲染解决方案

输入URL时

- 使用CDN加速请求，可能在全国各地放置一些CDN服务器。

  我的网站买的是青岛的CDN，所以山东境内看的可能比较快。

  我在黑龙江上学时，哈尔滨好像有个叫红房的地方，每年优酷都会提供数千万上亿的资金在这个地方放置他们的CDN服务器。

- 缓存

  - 强缓存
    - 不向服务器发信息
    - 响应头 cache-control :max-age:31536000  (秒 一年31536000秒)。在一年内不需要向服务器发送请求，他的优先级更高
    - 响应头 Expires 一个日期，时间戳
    - cache-control 设置了max-age会使得expires失效
  - 协商缓存
    - `Last-Modified`:

- HTTP2

- 预加载

渲染流程

- SSR
- SSG
  - 请求之前生成的静态页面。
  - Gatsby
  - Gridsome
  - 不专门做SSG

## vue双向数据绑定

```html
<body>
    姓名：<span id="spanName"></span>
    <br>
    <input type="text" id="inpName">

    <!-- IMPORT JS -->
    <script>
        let obj = {
            name: ''
        };
        let newObj = {
            ...obj
        };
        Object.defineProperty(obj, 'name', {
            get() {
                return newObj.name;
            },
            set(val) {
                newObj.name = val;
                observe();
            }
        });

        function observe() {
            spanName.innerHTML = newObj.name;
        }
        inpName.oninput = function () {
            obj.name = this.value;
        };
    </script>
</body>
```

```html
<body>
    姓名：<span id="spanName"></span>
    <br>
    <input type="text" id="inpName">

    <!-- IMPORT JS -->
    <script>
        let obj = {
            name: ''
        };
        obj = new Proxy(obj, {
            get(target, prop) {
                return target[prop];
            },
            set(target, prop, value) {
                target[prop] = value;
                observe();
            }
        });

        function observe() {
            spanName.innerHTML = obj.name;
        }
        inpName.oninput = function () {
            obj.name = this.value;
        };
    </script>
</body>
```

