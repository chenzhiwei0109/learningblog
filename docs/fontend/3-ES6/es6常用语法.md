---
sidebarDepth: 2
---
# es6常用语法

## ES6作用域分类

- 全局作用域：在函数外部定义
  - **var a =123 和 a = 123的区别：**
    - 前者是**全局变量**，**无法使用delete a进行删除**,只是他会在window下生成一个属性
    - **未使用var定义**的变量会挂载到window，但是并不是全局变量,**可以使用delete删除**
- 函数作用域
  - 内部声明的变量无法被外部取到，但是函数内部可以通过词法作用域取到外部作用域的局部变量
  - 如果想被外部获取
    - return 局部变量
    - 闭包（根据词法作用域定义，）
- 块级作用域：{}就是块级作用域,let和const声明,
- 静态作用域(词法作用域，函数定义时确定的，函数变量查找根据定义时的词法作用域查找)
- 动态作用域(this的作用域是动态的，this指向查找是根据最近调用他的this,或者显示绑定的this,或者new绑定的this查找)
- **{}不是es5的作用域，变量提升会无视他,所以if和while语句无法阻碍变量提升**

## let&const

### let、const与var的区别

- let与const不会在window下创建属性。
- let与const创建的变量不能被重复声明
- let与const允许你声明一个作用域被限制在 [`块`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/statements/block)级中的变量。 [`var`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/statements/var)声明的变量只能是全局或者整个函数块的
- [`var`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/statements/var) 和 let、const 的不同之处在于后者是在编译时才初始化，也就是说他没有变量提升,如果使用`typeof`检测在暂存死区中的变量, 会抛出`ReferenceError`异常而不是undefined。class也没有变量提升

### const

- const定义时必须赋值 ；
- const不允许声明时不初始化;
- 如果是引用类型，只是指针冻结。需要通过Ojbect。freeze进行对象冻结

## 数组

### 遍历

#### es5遍历方法汇总

**方法1： for循环**

```js
for(let i = 0; i < arr.length ; i++){
	//continue;
	//break;
}
```

**方法2：forEach**

- 不支持break和continue
- 可以遍历NodeList 对象(queryselectall)

```js
aLi.forEach(li=>{
	....
})
```

**方法3:every**

- 内部回调函数返回值默认false,不继续遍历，如果是true继续遍历
- 如果每个都返回true,这个方法返回true

```js
arr.every(function(item){
	if(item === 2){
		return false  //停止遍历
	}
	console.log(item)
	return false
})
```

**方法4:for in**

以任意顺序遍历一个对象的除[Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)以外的[可枚举](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)属性(包括它的原型链上的可枚举属性 )

- 遍历可枚举的属性或者方法;
- key是一个字符串；
- 可以使用break和continue

```js
for(var key in object){
	console.log(key,object[key])
}
```

- 不建议遍历数组，数组如果添加了一个属性，也会被遍历

```
arr.a = 'xx';也会被遍历到
```

#### es6遍历方法汇总

**方法1：for of**

Iterator

**for...of语句**在[可迭代对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/iterable)（包括 [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Array)，[`Map`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Map)，[`Set`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)，[`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String)，[`TypedArray`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)，[arguments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/arguments) 对象等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句,forEach不能遍历他们。

```js
for(let val of arr){
	console.log(item)
}
```

### 伪数组转化数组

**伪数组:**

**对象按照索引方式存储数据，并且具备length属性:{0:'a',1:'b',length:2}**

arguements、nodelist、htmlcollection

**转换**

es5写法

```js
let args = [].slice.call(arguements)
大坑：es6废弃arguements,eslint会出错
```

es6写法

```js
Array.prototype.from(document.querySelectAll('img'));
```

**Array.from语法**

```js
//伪数组，map函数 ，函数体内部this的指向
Array.form(arrayLike,mapFn,thisArg)

```

初始化并填充默认值

```js
let array = Array(5);
```

进行foreach遍历的时候，需要判空的。无法实现

```js
let array = Array.from({length:5},function(){return 1})
```

### 填充替换或者创建数组

**es5生成新数组**

- var arr = Array(5)
- var arr = []

**es6生成新数组**

- Array.from

- Array.prototype.of

  es6允许你快速把多个元素放入一个数组

  ```js
  let array = Array.of(1,2,8,6,5)  //[1,2,8,6,5]
  array = Array.of(1)   //[1]
  ```

- Array.prototype.fill

  替换、填充

  ```js
  Array.fill(value[,start[,end]) 
  //start默认0 end默认数组最后一个元素，如果不填，默认所有数组元素都是被填充
  ```

  初始化数组，然后赋值1

  ```js
  let array = Array(5);
  array = array.fill(1);
  
  ```

  ```js
  let array = [1,2,3,4,5];
  array.fill(8,2,4) //[1,2,8,8,5]
  
  ```

### 查找元素

#### es5查找

filter方法

- 筛选满足条件的**所有**元素，返回一个数组或者空数组
- 缺点：数组很大，只想知道是否存在，不合适

```js
//筛选值为3的数组 
arr.filter(item=>item===3);   //[3]  //
//筛选偶数
arr.filter(item=>item%2 ===0);

```

#### es6查找

find方法

- 不关注返回的所有值
- 找到就返回满足条件的数组
- 筛选满足条件的**第一个值**,找到就不再找

```
array.find(item=>item ===2)// 返回查找到的数据,找不到返回undefined

```

findIndex方法

- 返回查找到的第一个元素的索引

```
arr = [1,2,3,]
arr.findIndex(item=>item===2) //1

```

## 类

### 声明类

#### es5声明类

```
let Phone = function(name){
	this.name = name;
}
Phone.prototype.type = function(){
	console.log('手机')
}
let xiaomi = new Phone('小米');
xiaomi.constructor.prototype.type = ...//修改原型的方法  

```

#### es6声明类

```
class Animal {
	constructor (type){
		this.type = type
	}
	eat(){//挂载到原型上
		console.log('aa')
	}
}

```

区别:

- Class 类不存在变量提升

- class内部会启用严格模式

- class的所有方法都是不可枚举的

- class 必须使用new调用

- class 内部无法重写类名

  ```
       // es5 
      function Bar() {
          Bar = 'Baz';
          this.bar = 42;
      }
      var bar = new Bar();
      console.log(bar);// Bar {bar: 42}
      console.log(Bar);// 'Baz'
  
      // es6
      class Foo {
          constructor(){
              this.foo = 42;
              Foo = 'Fol'; // Uncaught TypeError: Assignment to constant variable.
          }
      }
      let foo = new Foo();
      Foo = 'Fol';// it's ok
  
  ```

- class 的继承有两条继承链

  一条是： 子类的__proto__ 指向父类

  另一条： 子类prototype属性的__proto__属性指向父类的prototype属性.

  es6的子类可以通过__proto__属性找到父类，而es5的子类通过__proto__找到Function.prototype

### 属性读写getter和setter

存取描述符数据描述符。

**属性只读**

```js
class Animal {
	constructor (type){
		this.type = type
	}
	eat(){//挂载到原型上
		console.log('aa')
	}
    get age(){
        return 4
    }
}
let dog = new Animal('dog');
dog.age  //  4
dog.age = 5  //  4

```

**属性可写**

```js
class Animal {
	constructor (type){
		this.type = type
	}
	eat(){//挂载到原型上
		console.log('aa')
	}
    get age(){
        return 5
    }
    set age(val){
        
    }
}
let dog = new Animal('dog');
dog.age  //  4
dog.age = 5  //  4

```

**私有属性_age**

```js
let _age = 4;
class Animal {
	constructor (type){
		this.type = type
	}
	eat(){
		console.log('aa')
	}
    get age(){
        return _age; 
    }
    set age(val){
        if(val <50){
            _age = val;
        }
    }
}
let dog = new Animal('dog');
dog.age  //  4
dog.age = 5  //  set会拦截你的赋值操作

```

### 静态方法

只能通过类.或者类['方法名']访问。而不腻通过实例访问

es5访问静态方法

```
function Animal(type){
	this.type = type
}
Animal.walk = function(){
	console.log(walk)
}

```

es6访问静态方法

```js
class Animal {
	constructor (type){
		this.type = type
	}
	eat(){
		console.log('aa')
	}
    static walk(){
        console.log(1)
    }
}
Animal.walk()

```

### 继承

**es5继承**

```js
let Animal = function(type){
	this.type = type
}
Animal.prototype.eat = function(){
	Animal.walk()
}
Animal.walk = function(){
	console.log('我在走路')
}

```

```js
let Dog = function(color){
	 //执行animal的构造函数，改变Animal里运行时的this指向Dog的实例，继承了Animal的属性
    Animal.call(this,'dog')
    this.color = color
}
Dog.prototype = Object.create(Animal.prototype)//重写原型对象
Dog.prototype.constructor = Dog;

```

**es6继承**

```js
class Animal {
	constructor (type){
		this.type = type
	}
	eat(){
		console.log('aa')
	}
    static walk(){
        console.log(1)
    }
}


```

super放在构造函数第一行

```
class Dog extends Animal{
	//显示，隐式
	constructor(type){
		super(type)
		this.age = 2
	}
}
let dog = new Dog('dog')
dog.eat() //方法自动继承

```

## 函数

### 函数参数默认值

**es5默认值**

```js
function f(x,y,z){
	if(y===undefined){
		y =7
	}
	console.log(x,y,z)
}
f(1,,3)

```

**es6默认值**

有默认值的放后面

使用参数默认值时，函数不能有同名参数,否则报错

```js
function f(x,y=7,z=42){
	console.log(x,y,z) 
}
f(1,undefined,43)  //1 7 43
f()  //undefined 7 2

```

**默认值可以是表达式**

```js
function f(x,y=7,z = x+y){  
	console.log(x,y,z)
}
f(1,undefined,undefined)// 1,7,8

```

```js
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}

foo() // 100

x = 100;
foo() // 101

```

//优先选择自己括号内的，且是从左往右

```js
let x = 99;
function foo(x=1,p = x + 1) {
  console.log(p);
}

foo() // 2

x = 100;
foo() //2

```

arguements:告诉你当前函数的外部传递参数是什么，所以和默认值无关

```
function f(x,y=7,z = x+y){  
	console.log(arguements.length);
}
f(1,undefined,undefined)//3

```

length :获取没有默认值的参数个数,不是外部传递的参数个数。

```
function f(x,y=7,z = x+y){ 
	f.length //获取到没有默认值的参数个数  //1,只有x没有默认值。
	console.log(arguements.length);
}
f(1,2,3)

```

```
function f(x,y){
	console.log(f.length)
}
f(1)  //2

```

### rest参数(rest parameter)

es5不确定参数

```
//获取不确定参数求和
function sum(){
	let num = 0;
	Array,prorotype.forEach.call(arguements,function(item){
		num+=item*1
	})
	//Array.from(arguements).forEach(item=>num+=item*1)
	return num;
}
sun(1,2,3,4,5)

```

es6

剩余参数,获取函数被执行的参数

```
function sum(...rest){
	return rest.reduce((sum,item)=>sum+item,0)
}

```

```
function sum(1,2,...rest){

}

```

### rest参数逆运算

es5

```
function sum(x,y,z){

}
sum.apply(null,[1,2,3])

```

```
function a(x,y,z){
}
a(...[1,2,3])

```

### 箭头函数

```js
let hellow = name => {return {name}}
let obj = hellow('小明');

```

 箭头函数的this是闭合词法环境的this,this不会变化。

```
let test = {
	name:'1',
	say:()=>{
		console.log(this.name)
	}
}

```

pppps:webpack构建时执行eval,会把外层作用域指向空对象，正常的外层函数this指向window。

#### **this**

> 在[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)中，`this`与封闭词法环境的`this`保持一致。也就是`this`被设置为他被创建时的环境，在全局代码中，它将被设置为全局对象。

在箭头函数出现之前，每一个新函数根据它是被如何调用的来定义这个函数的this值：

- 如果是该函数是一个构造函数，this指针指向一个新的对象
- 在严格模式下的函数调用下，this指向undefined
- 如果是该函数是一个对象的方法，则它的this指针指向这个对象

在ECMAScript 3/5中，通过将`this`值分配给封闭的变量，可以解决`this`问题。

```js
function Person() {
  var self = this;
  self.age = 0;

  setInterval(function growUp() {
    //  回调引用的是`that`变量, 其值是预期的对象. 
    self.age++;
  }, 1000);
}

```

```js
var obj = {
  bar: function() {
    var x = (() => this);
    return x;
  }
};

var fn = obj.bar();

console.log(fn() === obj); // true

var fn2 = obj.bar;   //隐式丢失，function(){var x = (()=>this);return x}这个函数内部this指向fn2所在作用域的this。
console.log(fn2()() == window); // true

```

#### 通过 call 或 apply 调用

由于 箭头函数没有自己的this指针，通过 `call()` *或* `apply()` 方法调用一个函数时，只能传递参数,不能绑定this,他们的第一个参数会被忽略,这种现象对于bind方法同样成立

```js
var adder = {
  base : 1,
    
  add : function(a) {
    var f = v => v + this.base;
    return f(a);
  },

  addThruCall: function(a) {
    var f = v => v + this.base;
    var b = {
      base : 2
    };
            
    return f.call(b, a);
  }
};

console.log(adder.add(1));         // 输出 2
console.log(adder.addThruCall(1)); // 仍然输出 2

```

#### 不绑定`arguments`

#### 不能作为构造函数

#### 没有`prototype`属性。

#### 解析顺序

```js
let callback;

callback = callback || function() {}; // ok

callback = callback || () => {};      
// SyntaxError: invalid arrow-function arguments

callback = callback || (() => {});    // ok

```

#### 其他

```js
//因为仅有一个返回，return 及括号（）也可以省略
var Add = (i=0)=> ()=> (++i);
// 递归
var fact = (x) => ( x==0 ?  1 : x*fact(x-1) );
fact(5);       // 120
```

## 对象

### 属性增强写法

**es5声明方式**

```js
let x = 1;
let z = 3;
let b = function (){
    console.log(this.x)
};
let obj = {
	x:x,
	y:2
    a:function(){
        //...
    },
    b:b
};
obj[z] = 5;

```

**es6声明方式**

- 允许添加异步方法generator

```js
let x = 1;
let b = function (){
    console.log(this.x)
};
let z = 3;
let obj = {
	x,//外部同名变量
	y:2,
    [z+x]: 6,  // 这里可以是任何的变量或者表达式  z+x = 4
    a(){
        
    },
    * asy(){  //generator函数
        
    },
    b// 外部同名函数
}
//结果
obj= {
    x:1;
    y:2,
    3:6
}

```

### 对象复制Object.assign

`Object.assign`方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）。

**重名会覆盖**

```js
const target = {b:1};
const source = {b:4,a:1};
Object.assign(target,source)  
target // {b:4,a:1}

```

**只能拷贝一层**

只能实现浅拷贝，引用类型的拷贝

```
const target = {};
const source = {a:{b:1}};
Object.assign(target,source);
console.log(target) // {a:{b:1}};

```

由于`undefined`和`null`无法转成对象，所以如果它们作为参数，就会报错。

如果该参数不是对象，则会先转成对象，然后返回。

如果非对象参数出现在源对象的位置（即非首参数），那么处理规则有所不同。首先，这些参数都会转成对象，如果无法转成对象，就会跳过。这意味着，如果`undefined`和`null`不在首参数，就不会报错。

```
let obj = {a: 1};
Object.assign(obj, undefined) === obj // true
Object.assign(obj, null) === obj // true

```

其他类型的值（即数值、字符串和布尔值）不在首参数，也不会报错。但是，除了字符串会以数组形式，拷贝入目标对象，其他值都不会产生效果。

```javascript
const v1 = 'abc';
const v2 = true;
const v3 = 10;

const obj = Object.assign({}, v1, v2, v3);
console.log(obj); // { "0": "a", "1": "b", "2": "c" }

```

## 解构赋值

### 数组解构

Array Destructure

**需要加let**

```js
let arr = ['从','啊'];
let [a,b] = arr;  //a='从',b = '啊'
```

```js
let arr = ['a','b','c','d']
let [firstName,,thirdName] = arr // firstName = 'a'  thirdName = 'c'
```

```
let [firstName,,thirdName] = new Set([1,2,3,4]);  //firstName =1 thirdName = 3
```

**对象属性，不需要加let**

```js
let user = {name:'zs',age:11};
[user.name,user.age] = [1,2];    //user={name:1,age:2}
```

**循环赋值**

采用隐式赋值   显示索引 ar[1]

```
let user = {name:'zs',age:11};
for( let [key,val] of Object.entries(user)){  //user变成可遍历成kv对的形式
	console.log(key,bal)
}

```

![1586685045479](../../.vuepress/public/assets/img/1586685045479.png)

```
let arr = [1,2,3,4,5,6,7,8,9];
let [firtsName,curName,...rest] = arr;  // rest = [3,4,5,6,7,8,9]
```

```
let arr = [1,2,3,4]
console.log(...arr) //1 2 3 4
```

**-数据量不够，解构赋值是undefined**

### 对象解构

对象赋值必须要这样

```js
let obj = {
	title:'menu',
	width:100,
	height:200
}
let {width,title,height} = options 
```

**别名**

```js
let obj = {
	title:'menu',
	width:100,
}
let {width,title:title2,height:height2=130} = obj  
console.log(width,title2,height2)
```

**剩余的，也会变成对象**

![1586685495163](../../.vuepress/public/assets/img/1586685495163.png)

![1586685500467](../../.vuepress/public/assets/img/1586685500467.png)

### 扁平化flat

```js
let options = {
  size: {
    width: {
      size: {
        width: 200
      }
    },
    height: 200
  },
  items: ['Cake', 'Donut'],
  extra: true
}

let { size: { width: width2, height }, items: [, item2], extra } = options
console.log(width2, height, item2, extra)  //items和size都不存在。因为他们不是具体的，

```

```
size: {width: 200} Object 200 "Donut" true
```

