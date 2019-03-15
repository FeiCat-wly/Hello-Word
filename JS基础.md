# JS基础

## [变量](变量类型和类型转换.md)

### JS 变量类型

JS中有 6 种原始值，分别是：
1. boolean
2. number
3. string
4. undefined
5. symble
6. null

引用类型：
1. 对象
2. 数组
3. 函数



### JS中使用typeof能得到哪些类型？
其中一个奇怪的 null，虽然是基本变量，但是因为设计的时候`null`是全0，而对象是`000`开头，所以有这个误判。
1. boolean
2. number
3. string
4. undefined
5. symble
6. **object**
7. function



### instanceof 能正确判断对象的原理是什么？
判断一个对象与构造函数是否在一个原型链上
```javascript
const Person = function() {}
const p1 = new Person()
p1 instanceof Person // true

var str = 'hello world'
str instanceof String // false

var str1 = new String('hello world')
str1 instanceof String // true
```



### 实现一个类型判断函数

1. 判断null
2. 判断基础类型
3. 使用`Object.prototype.toString.call(target)`来判断**引用类型**

注意： 一定是使用`call`来调用，不然是判断的Object.prototype的类型
之所以要先判断是否为基本类型是因为：虽然`Object.prototype.toString.call()`能判断出某值是：number/string/boolean，但是其实在包装的时候是把他们先转成了对象然后再判断类型的。 但是JS中包装类型和原始类型还是有差别的，因为对一个包装类型来说，typeof的值是object

```javascript
/**
 * 类型判断
 */
function getType(target) {
  //先处理最特殊的Null
  if(target === null) {
    return 'null';
  }
  //判断是不是基础类型
  const typeOfT = typeof target
  if(typeOfT !== 'object') {
    return typeOfT;
  }
  //肯定是引用类型了
  const template = {
    "[object Object]": "object",
    "[object Array]" : "array",
    "[object Function]": "function",
    // 一些包装类型
    "[object String]": "object - string",
    "[object Number]": "object - number",
    "[object Boolean]": "object - boolean"
  };
  const typeStr = Object.prototype.toString.call(target);
  return template[typeStr];
}
```



### 转Boolean
以下都为假值，其他所有值都转为 true，包括所有对象（空对象，空数组也转为真）。

 - false
 - undfined
 - null
 - ''
 - NaN
 - 0
 - -0



### 对象转基本类型
对象在转换基本类型时，会调用`valueOf`， 需要转成字符类型时调用`toString`。
```javascript
var a = {
  valueOf() {
    return 0;
  },
  toString() {
    return '1';
  }
}

1 + a           // 1
'1'.concat(a)   //"11"
```
也可以重写 `Symbol.toPrimitive` ，该方法在转基本类型时调用**优先级最高**。  [Symbol.toPrimitive](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toPrimitive) 指将被调用的指定函数值的属性转换为相对应的原始值。



### 类型转换

运算中其中一方为字符串，那么就会把另一方也转换为字符串
如果一方不是字符串或者数字，那么会将它转换为数字或者字符串
```javascript
1 + '1' // '11'
true + true // 2
4 + [1,2,3] // "41,2,3"
```

还需要注意这个表达式` 'a' + + 'b' `
```js
'a' + + 'b' // -> "aNaN"
```
因为 + 'b' 等于 NaN，所以结果为 "aNaN"，你可能也会在一些代码中看到过 + '1' 的形式来快速获取 number 类型。

[JS类型转换规则总结](https://blog.csdn.net/qq_37746973/article/details/82491282)

[JS隐射类型转换](https://blog.csdn.net/qq_37746973/article/details/81010057)



### "a common string"为什么会有length属性

通过字面量的方式创建：var a = 'string';，这时它就是基本类型值；通过构造函数的方式创建：var a = new String('string');这时它是对象类型。

基本类型是没有属性和方法的，但仍然可以使用对象才有的属性方法。这时因为在对基本类型使用属性方法的时候，后台会隐式的创建这个基本类型的对象，之后再销毁这个对象



### == 操作符

对于 == 来说，如果对比双方的类型不一样的话，就会进行类型转换

判断流程：
1. 首先会判断两者类型是否相同。相同的话就是比大小了
2. 类型不相同的话，那么就会进行类型转换
3. 会先判断是否在对比 null 和 undefined，是的话就会返回 true
4. 判断两者类型是否为 string 和 number，是的话就会将字符串转换为 number
```js
1 == '1'
      ↓
1 ==  1
```
5. 判断其中一方是否为 boolean，是的话就会把 boolean 转为 number 再进行判断
```js
'1' == true
        ↓
'1' ==  1
        ↓
 1  ==  1
```
6. 判断其中一方是否为 object 且另一方为 string、number 或者 symbol，是的话就会把 object 转为原始类型再进行判断
```js
'1' == { a: 'b' }
        ↓
'1' == '[object Object]'
```
7. 两边都是对象的话，那么只要不是同一对象的不同引用，都为false

注意，只要出现NaN，就一定是false，因为就连NaN自己都不等于NaN
对于NaN，判断的方法是使用全局函数 `isNaN()`



### === 操作符
不转类型，直接判断类型和值是否相同。
但是 NaN === NaN 还是false



### {} 等于true还是false
```js
var a = {};

a == true // -> ?
a == false // -> ?
```
答案是两个都为false
因为 a.toString() -> '[object Object]' -> NaN



### 1 与 Number(1)有什么区别
```js
var a = Number(1) // 1
var b = new Number(1)  // Number {[[PrimitiveValue]]: 1}
typeof (a) // number
typeof (b) // object
a == b // true
```

 - var a = 1 是一个常量，而 Number(1)是一个函数
 - new Number(1)返回的是一个对象
 - a==b 为 true 是因为所以在求值过程中，总是会强制转为原始数据类型而非对象，例如下面的代码:

```js
typeof 123 // "number"
typeof new Number(123) // "object"
123 instanceof Number // false
(new Number(123)) instanceof Number // true
123 === new Number(123) // false
```


### console.log(!!(new Boolean(false))输出什么 [易混淆]

true
布尔的包装对象 Boolean 的对象实例，对象只有在 null 与 undefined 时，才会认定为布尔的 false 值，布尔包装对象本身是个对象，对象->布尔 都是 true，所以 new Boolean(false)其实是布尔的 true，看下面这段代码:

```js
if(new Boolean(false)){
    alert('true!!');
}
```

只有使用了 valueOf 后才是真正的转换布尔值，与上面包装对象与原始资料转换说明的相同:

```js
!!(new Boolean(false))  //true
(new Boolean(false)).valueOf() //false
```


### 如何判断一个数据是不是Array

 - `Array.isArray(obj)`
   - ECMAScript 5种的函数，当使用ie8的时候就会出现问题。
 - `obj instanceof Array`
   - 当用来检测在不同的window或iframe里构造的数组时会失败。这是因为每一个iframe都有它自己的执行环境，彼此之间并不共享原型链，所以此时的判断一个对象是否为数组就会失败。此时我们有一个更好的方式去判断一个对象是否为数组。
 - `Object.prototype.toString.call(obj) == '[object Array]'`
   - 这个方法比较靠谱
 - `obj.constructor === Array `
   - constructor属性返回对创建此对象的函数的引用


### Object.prototype.toString
如果是原始类型，他会将原始类型包装为引用类型，然后调用对应方法

```js
function dd(){}
var toString = Object.prototype.toString;
toString.call(dd);          //[object Function]
toString.call(new Object);  //[object Object]
toString.call(new Array);   //[object Array]
toString.call(new Date);    //[object Date]
toString.call(new String);  //[object String]
toString.call(Math);        //[object Math]
toString.call(undefined);   //[object Undefined]
toString.call(null);        //[object Null]
toString.call(123)          //[object Number]
toString.call('abc')        //[object String]
```



### obj.toString() 和Object.prototype.toString.call(obj)
同样是检测对象obj调用toString方法，obj.toString()的结果和Object.prototype.toString.call(obj)的结果不一样，这是为什么？

这是因为toString为Object的原型方法，而Array ，function等类型作为Object的实例，都重写了toString方法。不同的对象类型调用toString方法时，根据原型链的知识，调用的是对应的重写之后的toString方法（function类型返回内容为函数体的字符串，Array类型返回元素组成的字符串.....），而不会去调用Object上原型toString方法（返回对象的具体类型），所以采用obj.toString()不能得到其对象类型，只能将obj转换为字符串类型；因此，在想要得到对象的具体类型时，应该调用Object上原型toString方法。

## [this](this.md)

# this

### this的指向有哪几种情况？
this代表函数调用相关联的对象，通常页称之为执行上下文。

1. 作为函数直接调用，非严格模式下，this指向window，严格模式下，this指向undefined；
2. 作为某对象的方法调用，this通常指向调用的对象。
3. 使用apply、call、bind 可以绑定this的指向。
4. 在构造函数中，this指向新创建的对象
5. 箭头函数没有单独的this值，this在箭头函数创建时确定，它与声明所在的上下文相同。

### 如果对一个函数进行多次 bind，那么上下文会是什么呢？

```js
let a = {}
let fn = function () { console.log(this) }
fn.bind().bind(a)() // => ?
```
不管我们给函数 bind 几次，fn 中的 this 永远由第一次 bind 决定，所以结果永远是 window。
```js
// fn.bind().bind(a) 等于
let fn2 = function fn1() {
  return function() {
    return fn.apply()
  }.apply(a)
}
fn2()
```

### 多个this规则出现时，this最终指向哪里？
首先，new 的方式优先级最高，接下来是 bind 这些函数，然后是 obj.foo() 这种调用方式，最后是 foo 这种调用方式，同时，箭头函数的 this 一旦被绑定，就不会再被任何方式所改变。
![this](../img/this.png)

## [函数](函数.md)

# 函数

### 箭头函数 => 中this

箭头函数不会创建自己的this,**它只会从自己的作用域链的上一层继承this**


### JS运行分三步：
语法分析（通篇扫描是否有语法错误），预编译（发生在函数执行的前一刻），解释执行（一行行执行）。


### 预编译执行分五步：
一、创建AO对象（Activation Object  执行期上下文）

二、找形参和变量声明，将变量和形参名作为AO属性名，值为undefined.
    变量声明提升（变量放到后面也不会报错，只是未定义类型）如：console.log(a);var a=10;结果undenfined;

三、将实参值和形参统一（传参）

四、在函数体里面找到函数声明{函数声明整体提升（相当于放到程序最前面）}

五、值赋予函数体，执行（声明函数和变量的部分直接不看了）


### 函数作用域[[scope]]

每个javascript函数都是一个对象，对象中有的属性可以访问，有的不能，这些属性仅供javascript引擎存取，如[[scope]]。

[[scope]]就是函数的作用域，其中存储了执行期上下文的集合。

**执行期上下文**： 当函数执行时，会创建一个称为执行期上下文的内部对象（AO）。一个执行期上下文定义了一个函数执行时的环境，函数每次执行时对应的执行期上下文都是独一无二的，所以多次调用一个函数会导致创建多个执行期上下文，当函数执行完毕，它所产生的执行上下文被销毁。


### 作用域链
`[[scope]]`中所存储的执行期上下文对象的集合，这个集合呈链式链接，我们称这种链式链接为作用域链。查找变量时，要从作用域链的顶部开始查找。Activation Object（AO）到Global Object（GO）。

![function](../img/function.png)
![a定义](../img/define.png)
![a run](../img/run.png)
![b define](../img/bdefine.png)
![b run](../img/brun.png)



### 闭包

当内部函数被保存到外部时，将会生成闭包。生成闭包后，内部函数依旧可以访问其所在的外部函数的变量。

闭包问题的解决方法：立即执行函数、let

详细解释：

当函数执行时，会创建一个称为**执行期上下文的内部对象（AO）**，执行期上下文定义了一个函数执行时的环境。

函数还会获得它所在作用域的**作用域链**，是存储函数能够访问的所有执行期上下文对象的集合，即这个函数中能够访问到的东西都是沿着作用域链向上查找直到全局作用域。

函数每次执行时对应的执行期上下文都是独一无二的，当函数执行完毕，函数都会失去对这个作用域链的引用，JS的垃圾回收机制是采用引用计数策略，如果一块内存不再被引用了那么这块内存就会被释放。

但是，当闭包存在时，即内部函数保留了对外部变量的引用时，这个作用域链就不会被销毁，此时内部函数依旧可以访问其所在的外部函数的变量，这就是闭包。


先看两个例子，两个例子都打印5个5
```js
for (var i = 0; i < 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 100)
}
```

```js
function test() {
   var a = [];
   for (var i = 0; i < 5; i++) {
         a[i] = function () {
            console.log(i);
         }
   }
   return a;
}

var myArr = test();
for(var j=0;j<5;j++)
{
   myArr[j]();
}
```


解决方法： 使用立即执行函数
```js
for (var i = 0; i < 5; i++) {
   ;(function(i) {
      setTimeout(function timer() {
         console.log(i)
      }, i * 100)
   })(i)
}
```
```js
function test(){
   var arr=[];
   for(i=0;i<10;i++)
   {
      (function(j){
         arr[j]=function(){
         console.log(j);
         }
      })(i)
   }
   return arr;
}

var myArr=test();
for(j=0;j<10;j++)
{
   myArr[j]();
}
```


### 闭包-封装私有变量
```js
function Counter() {
   let count = 0;
   this.plus = function () {
      return ++count;
   }
   this.minus = function () {
      return --count;
   }
   this.getCount = function () {
      return count;
   }
}

const counter = new Counter();
counter.puls();
counter.puls();
console.log(counter.getCount())
```


### 作用域与变量声明提升

 - 在 JavaScript 中，函数声明与变量声明会被 JavaScript 引擎隐式地提升到当前作用域的顶部 
 - 声明语句中的赋值部分并不会被提升，只有名称被提升
 - 函数声明的优先级高于变量，如果变量名跟函数名相同且未赋值，则函数声明会覆盖变量声明
 - 如果函数有多个同名参数，那么最后一个参数（即使没有定义）会覆盖前面的同名参数


### 构造函数，new时发生了什么？

```javascript
   var obj  = {}; 
   obj.__proto__ = Base.prototype;
   Base.call(obj);  
```

1. 创建一个新的对象 obj;
2. 将这个空对象的__proto__成员指向了Base函数对象prototype成员对象
3. Base函数对象的this指针替换成obj, 相当于执行了Base.call(obj);
4. 如果构造函数显示的返回一个对象，那么则这个实例为这个返回的对象。 否则返回这个新创建的对象



### 函数参数是对象会发生什么问题？

```javascript
function test(person) {
  person.age = 26
  person = {
    name: 'yyy',
    age: 30
  }

  return person
}
const p1 = {
  name: 'hy',
  age: 25
}
const p2 = test(p1)
console.log(p1) // -> {name: "hy", age: 20}
console.log(p2) // -> {name: "yyy", age: 30}
```

`person = {}` 这一步操作就将应用与原来的分离了
![地址改变](../img/addressChange.png)



### JavaScript 中，调用函数有哪几种方式？

 - 方法调用模式 Foo.foo(arg1, arg2);
 - 函数调用模式 foo(arg1, arg2);
 - 构造器调用模式 (new Foo())(arg1, arg2);
 - call/applay 调用模式 Foo.foo.call(that, arg1, arg2);
 - bind 调用模式 Foo.foo.bind(that)(arg1, arg2)();

## [对象](对象.md)

# 对象

### JS中有那些内置对象
 - 数据封装类对象 
   - String、Array、Object、Boolean、Number
 - 其他对象
   - Math、Date、RegExp、Error、Function、Arguments
 - ES6 新增对象
   - Promise、Map、Set、Symbol、Proxy、Reflect


### 数组Array对象常用方法
 - concat()	
 - join()	
 - pop()	
 - push()	
 - shift()	
 - unshift()
 - slice()	切片
 - splice()	在某位置删除与添加
 - reverse()	
 - sort()	
 - toString()	
 - valueOf()


### String对象常用方法
 - concat()
 - indexOf()
 - match()
 - replace()
 - search()
 - slice()
 - split()
 - toUpperCase()


### 对象：
 - object.hasOwnProperty(prop);
 - object.propertyIsEnumerable(prop);
 - object.valueOf();
 - object.toString();
 - object.toLocaleString();
 - Class.prototype.isPropertyOf(object);

## [原型链与继承](原型链与继承.md)

### 创建对象的方法
 - 字面量创建
 - 构造函数创建
 - Object.create()

```js
var o1 = {name: 'value'};
var o2 = new Object({name: 'value'});

var M = function() {this.name = 'o3'};
var o3 = new M();

var P = {name: 'o4'};
var o4 = Object.create(P)
```


### 原型
- JavaScript 的所有对象中都包含了一个 `__proto__` 内部属性，这个属性所对应的就是该对象的原型
 - JavaScript 的函数对象，除了原型 `__proto__` 之外，还预置了 prototype 属性
 - 当函数对象作为构造函数创建实例时，该 prototype 属性值将被作为实例对象的原型 `__proto__`。

![原型](../img/prototype.png)

### 原型链

任何一个实例对象通过原型链可以找到它对应的原型对象，原型对象上面的实例和方法都是实例所共享的。

一个对象在查找以一个方法或属性时，他会先在自己的对象上去找，找不到时，他会沿着原型链依次向上查找。

注意： 函数才有prototype，实例对象只有有__proto__， 而函数有的__proto__是因为函数是Function的实例对象


### instanceof原理
判断实例对象的__proto__属性与构造函数的prototype是不是用一个引用。如果不是，他会沿着对象的__proto__向上查找的，直到顶端Object。


### 判断对象是哪个类的直接实例
使用`对象.construcor`直接可判断


### 构造函数，new时发生了什么？

```javascript
   var obj  = {}; 
   obj.__proto__ = Base.prototype;
   Base.call(obj);  
```

1. 创建一个新的对象 obj;
2. 将这个空对象的__proto__成员指向了Base函数对象prototype成员对象
3. Base函数对象的this指针替换成obj, 相当于执行了Base.call(obj);
4. 如果构造函数显示的返回一个对象，那么则这个实例为这个返回的对象。 否则返回这个新创建的对象


### 类
类的声明
```js
// 普通写法
function Animal() {
  this.name = 'name'
}

// ES6
class Animal2 {
  constructor () {
    this.name = 'name';
  }
}
```

## 继承

### 借用构造函数法
在构造函数中 使用`Parent.call(this)`的方法继承父类属性。

原理： 将子类的this使用父类的构造函数跑一遍 

缺点： Parent原型链上的属性和方法并不会被子类继承

```js
function Parent() {
  this.name = 'parent'
}

function Child() {
  Parent.call(this);
  this.type = 'child'
}
```

### 原型链实现继承

原理：把子类的prototype（原型对象）直接设置为父类的实例

缺点：因为子类只进行一次原型更改，所以子类的所有实例保存的是同一个父类的值。
当子类对象上进行值修改时，如果是修改的原始类型的值，那么会在实例上新建这样一个值；
但如果是引用类型的话，他就会去修改子类上唯一一个父类实例里面的这个引用类型，这会影响所有子类实例

```js
function Parent() {
  this.name = 'parent'
  this.arr = [1,2,3]
}

function Child() {
  this.type = 'child'
}

Child.prototype = new Parent();
var c1 = new Child();
var c2 = new Child();
c1.__proto__ === c2.__proto__
```


### 组合继承方式
组合构造函数中使用call继承和原型链继承。

原理： 子类构造函数中使用`Parent.call(this);`的方式可以继承写在父类构造函数中this上绑定的各属性和方法； 
使用`Child.prototype = new Parent()`的方式可以继承挂在在父类原型上的各属性和方法

缺点：  父类构造函数在子类构造函数中执行了一次，在子类绑定原型时又执行了一次

```js
function Parent() {
  this.name = 'parent'
  this.arr = [1,2,3]
}

function Child() {
  Parent.call(this);
  this.type = 'child'
}

Child.prototype = new Parent();
```


### 组合继承方式 优化1：
因为这时父类构造函数的方法已经被执行过了，只需要关心原型链上的属性和方法了
```js
Child.prototype = Parent.prototype;
```
缺点：
 - 因为原型上有一个属性为`constructor`，此时直接使用父类的prototype的话那么会导致 实例的constructor为Parent，即不能区分这个实例对象是Child的实例还是父类的实例对象。
 - 子类不可直接在prototype上添加属性和方法，因为会影响父类的原型


注意：这个时候instanseof是可以判断出实例为Child的实例的，因为instanceof的原理是沿着对象的__proto__判断是否有一个原型是等于该构造函数的原型的。这里把Child的原型直接设置为了父类的原型，那么: 实例.__proto__ === Child.prototype === Child.prototype


### 组合继承方式 优化2 - 添加中间对象【最通用版本】：

```js
function Parent() {
  this.name = 'parent'
  this.arr = [1,2,3]
}

function Child() {
  Parent.call(this);
  this.type = 'child'
}

Child.prototype = Object.create(Parent.prototype); //提供__proto__
Child.prototype.constrctor = Child;
```
Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__

## [正则](正则.md)

# 正则表达式

### 创建正则表达式
1. 使用一个正则表达式字面量
```js
const regex = /^[a-zA-Z]+[0-9]*\W?_$/gi;
```
2. 调用RegExp对象的构造函数
```js
const regex = new RegExp(pattern, [, flags])
```

### 特殊字符

 - ^ 匹配输入的开始
 - $ 匹配输入的结束
 - \* 0次或多次  {0，} 
 - \+ 1次或多次  {1，}
 - ? 
   - 0次或者1次 {0,1}。
   - 用于先行断言
   - 如果紧跟在任何量词 *、 +、? 或 {} 的后面，将会使量词变为非贪婪
     - 对 "123abc" 用 /\d+/ 将会返回 "123"，
     - 用 /\d+?/,那么就只会匹配到 "1"。
 - . 匹配除换行符之外的任何单个字符
 - (x)  匹配 'x' 并且记住匹配项
 - (?:x)  匹配 'x' 但是不记住匹配项
 - x(?=y)  配'x'仅仅当'x'后面跟着'y'.这种叫做正向肯定查找。
 - x(?!y)  匹配'x'仅仅当'x'后面不跟着'y',这个叫做正向否定查找。
 - x|y  匹配‘x’或者‘y’。
 - {n}  重复n次
 - {n, m}  匹配至少n次，最多m次
 - [xyz]   代表 x 或 y 或 z
 - [^xyz]  不是 x 或 y 或 z
 - \d  数字
 - \D  非数字
 - \s  空白字符，包括空格、制表符、换页符和换行符。
 - \S  非空白字符
 - \w  单词字符（字母、数字或者下划线）  [A-Za-z0-9_]
 - \W  非单字字符。[^A-Za-z0-9_]
 - \3  表示第三个分组
 - \b   词的边界
   - /\bm/匹配“moon”中得‘m’；
 - \B   非单词边界


### 使用正则表达式的方法

 - exec	一个在字符串中执行查找匹配的RegExp方法，它返回一个数组（未匹配到则返回null）。
 - test	一个在字符串中测试是否匹配的RegExp方法，它返回true或false。
 - match	一个在字符串中执行查找匹配的String方法，它返回一个数组或者在未匹配到时返回null。
 - search	一个在字符串中测试匹配的String方法，它返回匹配到的位置索引，或者在失败时返回-1。
 - replace	一个在字符串中执行查找匹配的String方法，并且使用替换字符串替换掉匹配到的子字符串。
 - split	一个使用正则表达式或者一个固定字符串分隔一个字符串，并将分隔后的子字符串存储到数组中的String方法。


## 练习

匹配结尾的数字
```js
/\d+$/g
```

统一空格个数
字符串内如有空格，但是空格的数量可能不一致，通过正则将空格的个数统一变为一个。
```js
let reg = /\s+/g
str.replace(reg, " ");
```

判断字符串是不是由数字组成
```js
str.test(/^\d+$/);
```

电话号码正则
 - 区号必填为3-4位的数字
 - 区号之后用“-”与电话号码连接电话号码为7-8位的数字
 - 分机号码为3-4位的数字，非必填，但若填写则以“-”与电话号码相连接
```js
/^\d{3,4}-\d{7,8}(-\d{3,4})?$/
```

手机号码正则表达式
正则验证手机号，忽略前面的0，支持130-139，150-159。忽略前面0之后判断它是11位的。
```js
/^0*1(3|5)\d{9}$/
```

使用正则表达式实现删除字符串中的空格
```js
funtion trim(str) {
  let reg = /^\s+|\s+$/g
  return str.replace(reg, '');
}
```

限制文本框只能输入数字和两位小数点等等
```js
/^\d*\.\d{0,2}$/
```

只能输入小写的英文字母和小数点，和冒号，正反斜杠(：./\)
```js
/^[a-z\.:\/\\]*$/
```

替换小数点前内容为指定内容
例如：infomarket.php?id=197 替换为 test.php?id=197
```js
var reg = /^[^\.]+/;
var target = '---------';
str = str.replace(reg, target)
```

只匹配中文的正则表达式
```js
/[\u4E00-\u9FA5\uf900-\ufa2d]/ig
```

返回字符串的中文字符个数
先去掉非中文字符，再返回length属性。
```js
function cLength(str){
  var reg = /[^\u4E00-\u9FA5\uf900-\ufa2d]/g;
  //匹配非中文的正则表达式
  var temp = str.replace(reg,'');
  return temp.length;
}
```

正则表达式取得匹配IP地址前三段
只要匹配掉最后一段并且替换为空字符串就行了
```js
function getPreThrstr(str) {
  let reg = /\.\d{1,3}$/;
  return str.replace(reg,'');
}
```

匹配<ul>与</ul>之间的内容
```js
/<ul>[\s\S]+?</ul>/i
```


用正则表达式获得文件名
c:\images\tupian\006.jpg
可能是直接在盘符根目录下，也可能在好几层目录下，要求替换到只剩文件名。
首先匹配非左右斜线字符0或多个，然后是左右斜线一个或者多个。
```js
function getFileName(str){
  var reg = /[^\\\/]*[\\\/]+/g;
  // xxx\ 或是 xxx/
  str = str.replace(reg,'');
  return str;
}
```


绝对路径变相对路径
"http://23.123.22.12/image/somepic.gif"转换为："/image/somepic.gif"
```js
var reg = /http:\/\/[^\/]+/;
str = str.replace(reg,"");
```


用户名正则
用于用户名注册，，用户名只 能用 中文、英文、数字、下划线、4-16个字符。
```js
/^[\u4E00-\u9FA5\uf900-\ufa2d\w]{4,16}$/
```


匹配英文地址
规则如下:
包含 "点", "字母","空格","逗号","数字"，但开头和结尾不能是除字母外任何字符。
```js
/^[a-zA-Z][\.a-zA-Z,0-9]*[a-zA-Z]$/
```


正则匹配价格
开头数字若干位，可能有一个小数点，小数点后面可以有两位数字。
```js
/^\d+(\.\d{2})?$/
```


身份证号码的匹配
身份证号码可以是15位或者是18位，其中最后一位可以是X。其它全是数字
```js
/^(\d{14}|\d{17})(X|x)$/
```


单词首字母大写
每单词首字大写，其他小写。如blue idea转换为Blue Idea，BLUE IDEA也转换为Blue Idea
```js
function firstCharUpper(str) {
  str = str.toLowerCase();
  let reg = /\b(\w)/g;
  return str.replace(reg, m => m.toUpperCase());
}
```


正则验证日期格式
yyyy-mm-dd格式
4位数字，横线，1或者2位数字，再横线，最后又是1或者2位数字。
```js
/^\d{4}-\d{1,2}-\d{1,2}$/
```


去掉文件的后缀名
www.abc.com/dc/fda.asp 变为 www.abc.com/dc/fda
```js
function removeExp(str) {
  return str.replace(/\.\w$/,'')
}
```


验证邮箱的正则表达式
开始必须是一个或者多个单词字符或者是-，加上@，然后又是一个或者多个单词字符或者是-。然后是点“.”和单词字符和-的组合，可以有一个或者
多个组合。
```js
/^[\w-]+@\w+\.\w+$/
```


正则判断标签是否闭合
例如：<img xxx=”xxx” 就是没有闭合的标签；
<p>p的内容，同样也是没闭合的标签。

标签可能有两种方式闭合，<img xxx=”xxx” /> 或者是<p> xxx </p>。
```js
/<([a-z]+)(\s*\w*?\s*=\s*".+?")*(\s*?>[\s\S]*?(<\/\1>)+|\s*\/>)/i
```


正则判断是否为数字与字母的混合
不能小于12位，且必须为字母和数字的混
```js
/^(([a-z]+[0-9]+)|([0-9]+[a-z]+))[a-z0-9]*$/i
```


将阿拉伯数字替换为中文大写形式
```js
function replaceReg(reg,str){
  let arr=["零","壹","贰","叁","肆","伍","陆","柒","捌","玖"];
  let reg = /\d/g;
  return str.replace(reg,function(m){return arr[m];})
}
```


去掉标签的所有属性
<td style="width: 23px; height: 26px;" align="left">***</td>
变成没有任何属性的
<td>***</td>
思路：非捕获匹配属性，捕获匹配标签，使用捕获结果替换掉字符串。正则如下：
```js
/(<td)\s(?:\s*\w*?\s*=\s*".+?")*?\s*?(>)/
```

## [事件队列](事件队列.md)

### 为什么JavaScript是单线程？
JavaScript语言的一大特点就是单线程，也就是说，同一个时间只能做一件事。那么，为什么JavaScript不能有多个线程呢？这样能提高效率啊。

JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。


### Event Loop
参考地址:[Event Loop 这个循环你晓得么？(附 GIF 详解)-饿了么前端](https://zhuanlan.zhihu.com/p/41543963)



### 任务队列的本质

 - 所有同步任务都在主线程上执行，形成一个**执行栈**（execution context stack）。
 - 主线程之外，还存在一个”**任务队列**”（task queue）。只要异步任务有了运行结果，就在”任务队列”之中放置一个事件。
 - 一旦”执行栈”中的所有同步任务执行完毕，系统就会读取”任务队列”，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
 - 主线程不断重复上面的第三步。


### 异步任务
 - setTimeOut、setInterval
 - DOM 事件
 - Promise


### JavaScript 实现异步编程的方法？

 - 回调函数
 - 事件监听
 - 发布/订阅
 - Promises 对象
 - Async 函数[ES7]


### 关于 setTimeOut、setImmediate、process.nextTick()的比较

#### setTimeout()
将事件插入到了事件队列，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。
当主线程时间执行过长，无法保证回调会在事件指定的时间执行。
浏览器端每次setTimeout会有4ms的延迟，当连续执行多个setTimeout，有可能会阻塞进程，造成性能问题。

#### setImmediate()
事件插入到事件队列尾部，主线程和事件队列的函数执行完成之后立即执行。和setTimeout(fn,0)的效果差不多。
服务端node提供的方法。浏览器端最新的api也有类似实现:window.setImmediate,但支持的浏览器很少。

#### process.nextTick()
插入到事件队列尾部，但在下次事件队列之前会执行。也就是说，它指定的任务总是发生在所有异步任务之前，当前主线程的末尾。
大致流程：当前”执行栈”的尾部–>下一次Event Loop（主线程读取”任务队列”）之前–>触发process指定的回调函数。
服务器端node提供的办法。用此方法可以用于处于异步延迟的问题。
可以理解为：此次不行，预约下次优先执行。

## [DOM](DOM.md)



## [BOM](BOM.md)


## 常见问题

### eval 是做什么的？

eval 的功能是把对应的字符串解析成 JS 代码并运行

 - eval不安全，若有用户输入会有被攻击风险
 - 非常耗性能（先解析成 js 语句，再执行）



### ['1', '2', '3'].map(parseInt) 答案是多少？

答案 [1, NaN, NaN]

map会给函数传递3个参数： (elem, index, arry)

parseInt接收两个参数(sting, radix)，其中radix代表进制。省略 radix 或 radix = 0，则数字将以十进制解析

因此，map 遍历 ["1", "2", "3"]，相应 parseInt 接收参数如下

```js
parseInt('1', 0);  // 1
parseInt('2', 1);  // NaN
parseInt('3', 2);  // NaN
```


### [严格模式的限制](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode/Transitioning_to_strict_mode)

 - 变量必须声明后再使用
 - 函数的参数不能有同名属性，否则报错
 - 不能使用 with 语句
 - 不能对只读属性赋值，否则报错
 - 不能使用前缀 0 表示八进制数，否则报错
 - 不能删除不可删除的属性，否则报错
 - 不能删除变量 delete prop，会报错，只能删除属性 delete global[prop]
 - eval 不会在它的外层作用域引入变量
 - eval 和 arguments 不能被重新赋值
 - arguments 不会自动反映函数参数的变化
 - 不能使用 arguments.callee
 - 不能使用 arguments.caller
 - 禁止 this 指向全局对象
 - 不能使用 fn.caller 和 fn.arguments 获取函数调用的堆栈
 - 增加了保留字（比如 protected、static 和 interface）


### Ajax
Asynchronous Javascript And XML 异步传输+js+xml

 - 创建 XMLHttpRequest 对象,也就是创建一个异步调用对象
 - 建一个新的 HTTP 请求,并指定该 HTTP 请求的方法、URL 及验证信息
 - 设置响应 HTTP 请求状态变化的函数
 - 发送 HTTP 请求
 - 获取异步调用返回的数据

```js
function ajax(url, handler){
  var xhr;
  xhr = new XMLHttpRequest();

  xhr.onreadystatechange = function() {
    if(xhr.readyState == 4 && xhr.status == 200) {
      handler(xhr.responseXML);
    }
  }
  xhr.open('GET', url, true);
  xhr.send();
}
```


### Javascript 垃圾回收方法

标记清除（mark and sweep）

 - 这是 JavaScript 最常见的垃圾回收方式，当变量进入执行环境的时候，比如函数中声明一个变量，垃圾回收器将其标记为“进入环境”，当变量离开环境的时候（函数执行结束）将其标记为“离开环境”
 - 垃圾回收器会在运行的时候给存储在内存中的所有变量加上标记，然后去掉环境中的变量以及被环境中变量所引用的变量（闭包），在这些完成之后仍存在标记的就是要删除的变量了

引用计数(reference counting)

 - 在低版本 IE 中经常会出现内存泄露，很多时候就是因为其采用引用计数方式进行垃圾回收。引用计数的策略是跟踪记录每个值被使用的次数，当声明了一个 变量并将一个引用类型赋值给该变量的时候这个值的引用次数就加 1，如果该变量的值变成了另外一个，则这个值得引用次数减 1，当这个值的引用次数变为 0 的时 候，说明没有变量在使用，这个值没法被访问了，因此可以将其占用的空间回收，这样垃圾回收器会在运行的时候清理掉引用次数为 0 的值占用的空间

参考链接 [内存管理-MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)



### 哪些操作会造成内存泄漏？

 - JavaScript 内存泄露指对象在不需要使用它时仍然存在，导致占用的内存不能使用或回收
 - 未使用 var 声明的全局变量
 - 闭包函数(Closures)
 - 循环引用(两个对象相互引用)
 - 控制台日志(console.log)
 - 移除存在绑定事件的 DOM 元素(IE)


### 为什么要使用模块化？都有哪几种方式可以实现模块化，各有什么特点？

模块化可以给我们带来以下好处

 - 解决命名冲突
 - 提供复用性
 - 提高代码可维护性

实现模块化方式：
 - 立即执行函数
 - AMD 和 CMD
 - CommonJS
 - ES Module


### setTimeout、setInterval
常见的定时器函数有 `setTimeout`、`setInterval`、`requestAnimationFrame`，但setTimeout、setInterval并不是到了哪个时间就执行，**而是到了那个时间把任务加入到异步事件队列中**。

因为 JS 是单线程执行的，如果某些同步代码影响了性能，就会导致 setTimeout 不会按期执行。

而setInterval可能经过了很多同步代码的阻塞，导致不正确了，可以使用setTimeout每次获取Date值，计算距离下一次期望执行的时间还有多久来动态的调整。

[requestAnimationFrame](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame) 自带函数节流功能，基本可以保证在 16.6 毫秒内只执行一次（不掉帧的情况下），并且该函数的延时效果是精确的，没有其他定时器时间不准的问题



### cookie，localStorage，sessionStorage，indexDB

|     特性     |                   cookie                   |       localStorage       | sessionStorage |         indexDB          |
| :----------: | :----------------------------------------: | :----------------------: | :------------: | :----------------------: |
| 数据生命周期 |     一般由服务器生成，可以设置过期时间     | 除非被清理，否则一直存在 | 页面关闭就清理 | 除非被清理，否则一直存在 |
| 数据存储大小 |                     4K                     |            5M            |       5M       |           无限           |
| 与服务端通信 | 每次都会携带在 header 中，对于请求性能影响 |          不参与          |     不参与     |          不参与          |

从上表可以看到，`cookie` 已经不建议用于存储。如果没有大量数据存储需求的话，可以使用 `localStorage` 和 `sessionStorage` 。对于不怎么改变的数据尽量使用 `localStorage` 存储，否则可以用 `sessionStorage` 存储。

对于 `cookie`，我们还需要注意安全性。

|   属性    |                             作用                             |
| :-------: | :----------------------------------------------------------: |
|   value   | 如果用于保存用户登录态，应该将该值加密，不能使用明文的用户标识 |
| http-only |            不能通过 JS 访问 Cookie，减少 XSS 攻击            |
|  secure   |               只能在协议为 HTTPS 的请求中携带                |
| same-site |    规定浏览器不能在跨域请求中携带 Cookie，减少 CSRF 攻击     |