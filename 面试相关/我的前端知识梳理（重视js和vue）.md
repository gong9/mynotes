#  我的前端知识梳理（重视js和vue）（周三写完）

# 一、JS篇



## 1. js重点知识

这一块的知识非常重要，所以原来就进行过总结。故这里不再重复进行整理了（随着知识储备的上升感觉以前写的比较尴尬...但起码还是没有多大问题的）

### 1.1 作用域与闭包

作为js中重点知识，早先的时候已经拿出来整理过了，不过闭包结合调用栈与作用域的知识可理解的更深刻

[深入刨析闭包](https://juejin.im/post/6844904190783946760)

### 1.2 原型与继承

作为js中重点知识，早先的时候已经拿出来整理过了

[原型与继承全面刨析](https://juejin.im/post/6844904192063045639)

### 1.3 异步与event-lop

作为js中重点知识，早先的时候已经拿出来整理过了

[一起拿下异步](https://juejin.im/post/6844904200632008712)

[Event-Loop](https://juejin.im/post/6844904202334896135)



## 2. js执行机制

### 2.1 执行上下文与调用栈

#### 执行上下文

执行上下文：即代码的执行环境

它又分为三类

- 全局上下文
- 函数上下文
- Eval执行上下文（eval这种欺骗类型的知道就好了吧）

```js
let name='gxb'
function demo(){}
```

从这样一段简单的js代码说起

这段代码执行时，肯定是先创建一个全局的执行上下文

这个全局的执行上下文中，全局对象、this是必须的。并且这个this的指向就是全局对象

![1599385648099](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599385648099.png)

值得注意的是：此时的name变量里面放的是undefind

为啥呢？

因为每一个执行上下文均有两个阶段：创建阶段和执行阶段

在创建阶段所做的事如下：

- 创建全局对象
- 创建this，并使它指向全局
- 给变量和函数安排储存空间
- 默认给变量赋值为 undefined；将函数声明放入内存
- 创建作用域链

值的赋予在执行阶段（你现在理解变量提升的原理了吗）

函数执行上下文和全局执行上下文基本一致，不同之处如下

1. 首先创建的时机上
2. 再者创建的频率上，全局上下文只会被创建一次
3. 最后内容上，全局上下文开始里面是全局对象和this并且this指向全局对象；而函数上下文中创建的是参数对象arguments，this也得在运行时才能确定了

#### 调用栈

那么调用栈又是干啥的呢？

在js运行时，一个函数从堆内存拿出来，整成了函数上下文的模样。这个函数上下文放在哪呢？这确实是个问题吧

因为我们知道一个函数执行完毕后，它里面数据所占的那些地址是要被释放了的，并且有时候函数里面是要又套函数的它们之间的执行顺序已经内存释放顺序也是需要考虑的。

故想到可以采用一个栈的数据结构，来放置这些函数上下文，这就是调用栈

递归问题和闭包问题这里面（执行上下文、调用栈、作用域）可以得到充分的理解

### 2.2  垃圾回收机制

我们都知道js中的数据存储是这样的：简单数据类型放进栈内存，引用数据类型放进堆内存

堆栈这两种数据结构我们应该再熟悉不过了吧（前面小白有写过一篇实现的）

简单类型和引用类型的访问机制也是不同的，简单数据类型直接在栈内存就可以拿到数据，而引用类型在栈内存中拿到的仅是其数据在堆内存的地址

下面开始进行正题：垃圾回收

在js中，引擎会每隔一段时间就会对变量进行检查，如果发现这个变量不再被需要了，就会将这个变量所占的内存地址释放开来

js的垃圾回收机制，其实js的垃圾回收机制原理都是比较简单的

#### 引用计数法

这个引用，我们在js中可以理解成变量到实际内存的这种一对一的关系

如：

```js
const gxb={}
```

为了容易理解引用计数法，可以这样来（忽略堆栈内存）

![1599383775926](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599383775926.png)

即这块地址被gxb这个变量引用了，计个数1（再被别的变量引用再累计计数）

什么时候回收这块内存呢？即引用计数为0的时候

![1599383979290](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599383979290.png)

如上图，变量gxb又指向别的地址了，旧地址无人引用即计数为0 。这时候垃圾收集器就会把它释放掉



引用计数法已经被淘汰了，因为它有一个很大的缺点。看下面这个经典的栗子

```js
function demo{
    const a={}
    const b={}
    a.b=b
    b.a=a
}
```

函数进去调用栈之后，执行完毕，其作用域中的数据随着即被销毁（闭包那是函数执行完毕）

但是使用引用计数法，会造成这种情况

![1599384437136](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599384437136.png)

这两块地址没有被外界变量所引用，也即外界根本就访问不到这两块地址了，那留着它们属实是没有啥子用吧。

为了解决这个漏洞，标记清除法来了

#### 标记清除法

标记清除法也很容易理解，它的原理就是你能从外界访问到的内存即不需要回收的，访问不到的即要被干掉的

就比如上面那两块互相引用的，外界就没有对它们进行引用的变量，故可直接回收

## 3.ES6

### 3.1 模块化

**回顾知识**

它的大致发展路线是这样的：commonJS->AMD->CMD/UMD->ES6 Moudle

这里主要来看的是commonJS和ES6

#### 3.1.1 先来看看commonJS

commonJS的主要应用者是node，在node中每一个文件都是一个模块，都有自己的作用域。也就是说在一个文件中所定义的不管是变量、对象还是函数都是私有于这个文件的，外部是无法访问的

在commonJS的规范中，在每一个模块内部都有一个变量`moudle`，它代表着当前模块，这个moudle是一个对象。

我们先来打印一下这个moudel对象

![1597800574924](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1597800574924.png)

简单说一下这个module对象，在node内部提供了一个Module构造函数，每创建一个模块，便实例化一个module。或者这样说所有的模块都是Module的一个实例

它的一些属性解释

![1597800849379](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1597800849379.png)



#####  导出

上面我们知道了module对象有一个exports属性，这个属性是一个对外的接口。在别的文件中使用require加载这个模块，其实就是加载的这个属性的值

如:

```js
//main.js
const { name, getValue } = require('./test')

console.log(name)
console.log(getValue(1))


//test.js
let name = 'gxb'
function getValue(value) {
    return value + 1
}
module.exports = {
    name,
    getValue
}

```

进一步简便一下，来看exports变量

上面代码还可以写成这样

```js
let name = 'gxb'

function getValue(value) {
    return value + 1
}

exports.name = name
exports.getValue = getValue
```

怎么说呢？

其实也就是稍微为commonJS的导出做了一点简化，这个exports变量还是指的moudle的exports，相当于开始加了这个一句代码

```js
let exports=moudle.exports
```

这有个注意点，我想你一定注意过.

它是不能像开始moudle.exports那样直接给它赋予值的

```js
exports={
     name,
    getValue
}
```

为啥呢？

你开始你相当于`let exports=moudle.exports`,即你这个exports变量中只存储的是`moudle.exports`的引用。而加载是需要借助这个`moudle.exports`的。你另改变了exports变量，它之前的不就全废了吗

##### 导入

commonJS是利用require方法进行模块的加载。这个方法参数是模块的路径，后缀名默认为.js

它的基本功能是：读入并执行一个js文件，然后返回该模块的exports对象。

需要知道的是，使用require加载一个模块时，该模块代码的执行仅是发生在第一次加载时，后面都是直接从缓存中获取的

所有的缓存都保存在`require.cache`中,删除

```js
// 删除指定模块的缓存
delete require.cache[moduleName];//模块名是该模块的绝对路径

// 删除所有模块的缓存
Object.keys(require.cache).forEach(function(key) {
  delete require.cache[key];
})
```

##### 注意一下加载机制

CommonJS模块的加载机制是，导入的是导出值的拷贝。即我们外面拿到的仅是一份数据的拷贝，外面对数据的修改仍然影响不到模块内部

CommonJS的特点

- 所有模块仅运行在模块作用域中，不会污染全局作用域
- 模块可以多次加载，但是代码只会在第一次加载时运行一次然后其运行结果就被缓存了下来，后面再加载便是直接读取缓存结果了
- 模块的加载顺序是其在代码中出现顺序

#### 3.1.2 再来看ES6 Moudle

这个就比较简单了

使用

```js
//b.js

let name = 'gxb'

function demo() {
    console.log(111);
}
export { name, demo }


//a.js
import { name, demo } from './a'
demo()
```

或者默认导出，这样加载时我们就可以随意指定变量名了

```js
//b.js
export default function() {
    console.log(111)
}

//a.js
import demo from './a'
demo()
```

#### 3.1.3 两者的区别

- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用（重点）

- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

  

### 3.2 var、let、const

以前写过，加上比较简单就略了吧

### 3.3  symbol set  weakSet map weakMap

#### symbol

symbol被引入的原因：**防止对象属性名的重复**

symbol值由symbol函数生成，表示唯一、独一无二的值

```js
const s1 = Symbol()
const s2 = Symbol('s2')//参数是对此symbol值的描述
const s3 = Symbol.for('s3')
```

Symbol.for()与Symbol（）都是用于生成一个Symbol值。它们的不同之处在于使用Symbol.for()会在全局注册一下（就理解成全局有一个Symbol的hash表），拿上面的栗子来说：`const s3 = Symbol.for('s3')`这句话代码会先去hash表中查看有没有加s3的这么一个Symbol有的话就使用它，没有的话新建一个并放进hash表

这也就说下面的s3、s4是同一Symbol

```js
const s3 = Symbol.for('s3')
const s4 = Symbol.for('s3')
```

Symbol（）就无脑新建了，且不会去注册

值得注意的是当它作为对象属性时，这个属性是不会被for...in、for...of循环遍历到的（Object.keys、Object.getOwnPropertyNames等同样不行）

但是它也不是此对象的私有属性，可以通过Object.getOwnPropertySymbols拿到（Peflect.ownKeys也是可以的）

正是由于键为symbol属性的这个虽不是私有属性但是也不可被随随便便遍历到的特性，可以利用它做很多事情

**Symbol.keyFor用于获取已经注册的Symbol的描述（或叫键值）**

```js
const s4 = Symbol.for('s3')
console.log(Symbol.keyFor(s4))//s3
```

#### set与weakset

##### set

就是我们数据结构中的集合，也即成员不允许重复

```js
const s1=new Set()
const s2=new Set([1,2,3])//其构造函数的参数可以是其他具有iterator接口的数据结构


//相关方法
//add
//delete
//has
//clear
```

栗子

```js
const obj = {
    *[Symbol.iterator]() {
        yield 1
        yield 2
        yield 3
    }
}
const s2 =new  Set(obj)//Set { 1, 2, 3 }
```

常见应用：数组去重

```js
function dedupe(arr){
   return [...new Set(arr)]
}
```

**值得注意的是**：在set中NaN是等于NaN的

set的一些操作和集合一致，就不再赘述了

##### weakset

```js
const ws = new WeakSet()

//相关方法
//add
//delete
//has
```

与set的两点不同

1. 成员类型只能是对象
2. 弱引用

简单来说下弱引用吧，这里小白我用白话说一下吧。正常情况下一块内存只要外界有变量能够访问到它，那么这块地址就不会被垃圾回收。但是weakset不同，它自己做不了主。如果没有其他外界变量对它内部对象元素的引用它里面的东西还是会被垃圾回收的

且因为是弱引用，即不能保证其内部元素是否还存在。故不可遍历



#### map与weakmap

##### map

map是对对象结构的一种改进，传统对象结构的键只能是字符串，map可以是任意类型

```js
const m=new Map()
const obj={}
m.set(obj,{})
m.get(obj)//{}

其他相关操作
// has
// delete
```

或者直接new时，给构造函数传参

```js
const m1=new Map([[1,1],[2,2]])//保证每一项均是双元素，因为要一个为键一个为值

const obj = {
    *[Symbol.iterator]() {
        yield [1, 1]
        yield [2, 2]
    }
}
const m2 = new Map(obj)
```

map结构实现了迭代器接口，且其迭代器生成函数用的是  map.entries()  中的。

有了迭代器接口，故我们可以对他使用for...of，使用扩展运算符（这些东西本就是用的构造器）

**证明栗子**

```js
function* demo() {
    yield 1
    yield 2
    yield 3
}

for (const iterator of demo()) {
    console.log(iterator);//1,2,3
}
```

##### weakmap

```js
const wm=new WeakMap()
// set...
// get...
```

与map的区别基本和weakSet一样

1. 键必须是对象
2. 键的引用是弱类型

### 3.4  proxy与reflect

#### proxy

我不相信还有人不知道vue中的数据劫持...估计大家比我熟悉

它的作用类似于AOP吧

```js
const obj=new Proxy({},{
    get(target,name){
        return 'gxb'
    }
})
console.log(obj.name)//gxb
```

列举一下比较常用的拦截方法吧

- get
- set
- apply：拦截实例作为函数调用
- construct  ：拦截实例作为构造函数调用

值得注意的是：即使代理操作啥也不做，这个代理实例obj和目标对象也不是完全一致（this指向问题）

#### reflect

这个出来的目的是为了  将 Object 对象的一些明显属于语言内部的方法（ 比
如 Object.defineProperty ） ， 放到 Reflect 对象上  。我目前这里使用的也蛮少的以后在补充吧

### 3.5  iterator与for...of...

这个很简单，iterator的出现就是为不同的数据结构提供一种统一的访问机制（就是什么数组啦、set啦、map啦，有了一种统一的遍历机制）

迭代器本质是一个指针对象，通过next方法进行移动并返回一个节点对象。这个节点对象就是其指向的那个节点的信息，这个节点对象有两个属性value和done，一个保存节点值一个记录指向走到头没有

写一个迭代器生成器吧

```js
function demo() {
    return {
        next() {
            return { value: 0, done: false }
        }
    }
}
```

generator一个现成的迭代器生成函数

```js
function* demo() {
    yield 1
    yield 2
    yield 3
}
console.log(demo().next());//{ value: 1, done: false }
```

我们都知道，对象是没有iterator接口的。上面小白已有写过的栗子了

给它加上,这时候它就可以使用for...of进行遍历了。为啥呢？上面有过解释啊，for...of实际走的就是这个接口的迭代器嘛

```js
const obj = {
    *[Symbol.iterator]() {
        yield 1
        yield 2
        yield 3
    }
}
```

除了for...of走的是迭代器接口，还有一些别的也是走的它

如对数组和set的结构赋值、对数组的扩展运算符

栗子

```js
const arr = []
arr[Symbol.iterator] = function*() {

    yield 1
    yield 2
    yield 3

}
console.log(...arr)//1,2,3
```

开始看阮一峰老师的书，没有指定扩展运算符的范围。使我变得有点苦恼，因为对象也是可以使用扩展运算符的。但是对象是没有实现迭代器接口的

```js
const obj = {
    *[Symbol.iterator]() {
        yield 1
        yield 2
        yield 3
    },
    name: 'gxb'
}
const obj01 = {
    ...obj
}
console.log(obj01)

输出
{
  name: 'gxb',
  [Symbol(Symbol.iterator)]: [GeneratorFunction: [Symbol.iterator]]
}
```

可见对象的扩展语法走的另有别的东西（我现在也不知道它走的是啥...）

### 3.6  promise、generator、async

这一部分主要常用于异步，异步又作为js的重点。应该都熟悉，简单再重复写一点

promise应该不用再说了吧，至于generator上面栗子啥的也已经写的差不多了

至于对于`generator`函数的理解一般有两种

- 状态机
- 迭代器生成函数

就我来说，主要关注它走走停停的特性就好。

补充一点上面没有写过的知识点

1. 迭代器的next方法可以传参，参数被当做上一个yield的返回值
2. for...of拿不到return表达式后面的数据
3.   yield* 表达式， 用来在一个 Generator 函数里面执行另一个
   Generator 函数。

而async作为generator的语法糖，异步那也已经熟悉过了

### 3.7 其他扩展

像什么展开运算符、解构什么的就不写了吧

# 二、VUE篇

vue的总结按怎样的一个大纲开始的，这可真是一个令人头疼的问题啊。那么还是以自己的理解再去走一遍vue的常见面试题吧（虽然已经可能有无数人做过整理了，但是他们整理再好也不如你自己亲手走一遍）

## 1. 重点知识方面

### 1.1  生命周期

#### 1.1.1 单组件

![vue生命周期](C:\Users\T540P\Desktop\vue生命周期.png)



#### 1.1.2 父子组件

从创建到挂载，其钩子执行顺序为

![1599467071396](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599467071396.png)

子组件更新

这里其实有个坑，很多文章直接是这样写的

 子组件更新过程 ： 父 beforeUpdate -> 子 beforeUpdate -> 子 updated -> 父 updated 

但是上手一下，你仅改变子组件的一数据，它是不会触发父组件的更新钩子的

![宫小白822010028](C:\Users\T540P\Desktop\宫小白822010028.gif)

但是又一想，子组件它也应该属于父组件的一部分，其更新本就是应该触发父组件的更新钩子啊，唉其实不然

出现上述那种结论的前提是什么呢？

它的前提是子组件变化，被父组件监听到了故进而也造成了父组件中数据的变化（也即代码中子组件数据变化时用emit通知一下父组件）

![宫小白加油01](C:\Users\T540P\Desktop\宫小白加油01.gif)

父子组件的销毁和它一个样子

#### 1.1.3 比较常见的问题

##### 在哪个构造里面操作DOM

![1599468813162](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599468813162.png)

在这两个钩子中需要做好的事情是将已经编译好的模板真正的挂到浏览器上，故在mounted钩子中便可拿到最新的DOM了

##### 在哪个钩子里面调用异步请求

这个其实比上面的操作DOM需要的时机简单多了，一般来说发送异步请求均是为了获取服务器端的数据，这里主要的关注点就是数据的储存

即最起码data已经被初始化好了吧

故在  created、beforeMount、mounted 几个钩子中均可

为了可更快的获取数据一般均在created这个钩子中去处理异步请求

### 1.2  组件间的通信

#### 1.2.1 父到子

**法一：props**

这个要需要栗子？

**法二：refs**

父组件

```vue
<template>
  <div id="app">
   <test01 ref="test01Ref"></test01>
  </div>
</template>
<script>
import Test01 from './components/test01'
export default {
 mounted() {
   this.$refs.test01Ref.test='参数'
 },
  components: {
    Test01
  }
}
</script>
```

子组件

```vue
<template>
  <div>
    {{test}}
  </div>
</template>
<script>
export default {
  data() {
    return {
      test:''
    }
  },
}
</script>

```



法三：$children

上面父组件改动

```js
 mounted() {
   this.$children[0].test='参数'
 },
```

值得注意的是： $children 并不保证顺序，也不是响应式的 

#### 1.2.2 子到父

这个就是用emit了

#### 1.2.3 兄弟组件

##### 如有共同父辈

这个也比较简单。将要传的数据交给父辈

具体做法：在1组件中使用parent拿到父组件，使父组件派发一个事件，要要传的数据放进去。在2组件中再拿到其父组件监听到其刚才触发的事件并将数据拿过来

1组件

```js
 this.$parent.$emit('demo',数据)
```

2组件

```js
this.$parent.$on('demo',data => {
      console.log(data)
    })
```

##### 无共同父辈

使用事件总线

```js
class Bus {
    constructor() {
        this.callbacks = {}
    }
    on(name, fn) {

        this.callbacks[name] = fn

    }
    emit(name, args) {
        if (this.callbacks[name]) {
            this.callbacks[name](args);
        }
    }
}
Vue.prototype.$bus = new Bus()
```

#### 1.2.4 隔代组件

这就用到了这两东西

1. provide
2. inject

```js
祖先组件：
添加一个provide选项，provide选项可以是一个对象，也可以是一个返对象的函数
provide(){
    return{
      test:"参数"
    }
  },
子孙组件：
js内添加一个inject选项，该选项可以是一个字符串数组或对象
 inject:['test']
```

#### 1.2.5 vuex

vuex的相关知识，原来已经整理过了

[人家都在玩源码，你还在纠结vuex的使用...](https://juejin.im/post/6863661061103747085)

### 1.3  computed和watch的区别

#### computed

有缓存，只有依赖的数据发生了变化才会重新计算

#### watch

监听的属性一旦发生变化，就走一遍后面的回调

### 1.4 v-if与v-show的异同

v-if会选择性的渲染组件，v-show仅是显示和隐藏

### 1.5 vue的六大高级特性

像nextTick、插槽、混入、keep-alive等前面已经总结过了

链接：[总结vue的6大高级特性——及浅谈一下nextTick、keep-alive的原理](https://juejin.im/post/6864570298767769607#heading-21)

### 1.6 vuex和vue-router的使用及原理

[人家都在玩源码，你还在纠结vue-router的使用...](https://juejin.im/post/6863288289185988616)

[【姐妹篇】人家都在玩源码，你还在纠结vuex的使用...]( https://juejin.im/post/6863661061103747085 )

## 2.主要原理知识方面

### 2.1 vue的响应式原理实现

![1599641157200](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599641157200.png)

#### vue2.x  主要API Object.defineProperty 

getter时进行依赖收集，setter时进行触发更新

这里作为重要知识点，前面已经整理过了

[传送门：vue响应式实现&vue及react的diff算法](https://juejin.im/post/6844904200632008717)（ps：代码过多，加上写的比较早有一些疏漏没有更改，比如watcher中的异步更新应该用使用微任务，我使用的宏任务...）

这里在整理一下主要思路，主要流程就如小白所绘下图

![1599710878809](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599710878809.png)

Dep主要是收集watch对象和通知更新

```js
let id = 0

class Dep {
    constructor() {
        this.subs = []
        this.id=id++
    }

    // 订阅
    addSub(watcher) {
        this.subs.push(watcher)
    }

    // 发布
    notify() {
        this.subs.forEach(watcher => {
            watcher.update()
        })
    }
    
    //实现与watcher关联
    depend() {
        if (Dep.target) {
            Dep.target.addDep(this)
        }
    }
}

```

这里原来写的时候和图中稍微有一丁点不一样，即收集依赖时调用的是`dep.depend()`

这样走

![1599712526447](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599712526447.png)

**为什么还要搞这么复杂的一个依赖收集的流程呢？**

这我们回想一下vue中响应 式数据就明白了。vue中的响应式数据只要它发生了变动，其他用了这个数据的地方均会进行更新

还有只有getter是才会进行依赖的收集，故我们在组件中data定义了数据但是视图中没有进行使用也即没有触发getter，故改变这个数据是不会更新视图的（性能优化）

借助一下大佬的图

![1599713777015](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599713777015.png)

#### vue3的响应式原理

vue3的响应式原理与依赖相关见下

### 2.2 diff算法逻辑

这里作为重要知识点，前面已经整理过了

[传送门：vue响应式实现&vue及react的diff算法](https://juejin.im/post/6844904200632008717)

这里react和vue的diff都有涉及，react的较为简单还是vue的为主吧。之前整理中也有一点图解这里就不单独拿出来了

### 2.3 模本编译原理

这块知识点我前面都是落下了，现在简单补一下它的实现吧

它的主要流程其实比较简单，我们都知道模板编译完成的最终结果是生成一个渲染函数，后面这个渲染函数执行生成vnode然后经过diff之后挂载至页面

故这里的主要操作就是：模板——>渲染函数

模板中需要处理很多东西，像`指令`，`{{}}`等。这些东西都需要从里面找出来进行操作的。怎么操作一段源码中的一些数据呢？如果你看过我之前总结的webpack后面的手写部分或者是了解过抽象语法树，这里你也会第一时间给出答案。没错这里就是需要将模板转成抽象语法树，以方便我们可以结构化的以类似节点的形式操作这段模板源码中的部分数据

搞成AST之后的优化我没有去过多了解，故这里不会涉及到。我这块写作的重点就放在了模板->AST->render

开始正题，怎么拿这个模板的代码我想应该不用多说吧（比如有render函数用render，没有看看有没有template，没有再做html中找）...把精力放到重点吧

####  生成AST

模板

```html
 <div id="app" style="color:blueviolet;font-size: 30px;">
        我是{{name}}

        <span style="color: rgb(150, 70, 16)">{{age}}</span>
        <p class="demo" style="color:black;">怎么玩转vue呢？</p>

    </div>
```



在重写webpack的require那还比较简单，AST的生成可以借助第三方，但是vue模板这里就得我们自己动手写了。

那么下面主要就是字符串与正则相关的操作

首先来看生成的AST长啥样

![1600146793819](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600146793819.png)

基本结构

```js
tag:标签
attrs：{属性们...}
children:{
    孩子们...
}
parent:父亲
type：节点类型
```

接来来重点来了：生成AST的主要思路

还是主要靠正则

几个vue里面的正则，主要关注于有注释的

```js
//匹配属性  如 id="app" id='app' id=app
const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/;
const ncname = `[a-zA-Z_][\\-\\.0-9_a-zA-Z]*`;
const qnameCapture = `((?:${ncname}\\:)?${ncname})`;
// 匹配 标签开始如<div
const startTagOpen = new RegExp(`^<${qnameCapture}`);
// 匹配标签结束  如> />
const   startTagClose = /^\s*(\/?)>/;
//匹配  如 </div>
const endTag = new RegExp(`^<\\/${qnameCapture}[^>]*>`);
```

从`<`开始，匹配到标签开始并进行tag和属性保存的方法，匹配到第一个开始标签`<div`

![1600147839622](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600147839622.png)

如图，此时可以拿到tag（即标签是div），接着往后走去匹配属性（注意这的匹配规则是匹配完成之后即切割掉）

![1600148000314](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600148000314.png)

如图匹配到了这些属性数据，进行封装操作

注意只有匹配到了`</div>`才标明此标签操作完毕，故其里面的标签就均可做为它的子节点

来看这的主要代码

```js
function parseStartTag(template) {
    // 取到标签
    const start = template.match(startTagOpen)
    console.log(start)
    let end, attr

    // 进行拼接
    if (start) {
        const match = {
            tagName: start[1],
            attrs: []
        }
        template = advance(template, start[0].length)

        // 再看有无属性
        while (!(end = template.match(startTagClose)) && (attr = template.match(attribute))) {

            // 装属性
            match.attrs.push({
                name: attr[1],
                value: attr[3] || attr[4] || attr[5]
            })

            // 切割
            template = advance(template, attr[0].length)
        }

        // 结束了
        if (end) {
            template = advance(template, end[0].length)
            return { template, match }
        }
    }
}
```

再往下就是下面这种情况了，然后依次反复

![1600163261093](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600163261093.png)



还是得根据代码进行解释,看下面的这个主要函数，主要就是要处理三种情况。

1. 处理开始，此时要进行tag和属性数据的保存
2. 处理中间文本，此时要保存文本数据，并将此文本节点放到当前节点的孩子中
3. 处理结束

```js
// 将模板转成AST
export function parseHtml(template) {
    const typeMsg = {
        text: undefined,
        root: undefined,
        currentParent: undefined,
        stack: []
    }
    while (template) {
        let testEnd = template.indexOf('<')
        const endTagMatch = template.match(endTag)

        if (endTagMatch) {
            template = advance(template, endTagMatch[0].length)

            end(endTagMatch[1], typeMsg)
        } else if (testEnd === 0) {
            const { template: newTemplate, match } = parseStartTag(template)

            // 到这头部已经完成切割和收集
            template = newTemplate
            if (match) {
                start(match.tagName, match.attrs, typeMsg)
            }
        } else {
            typeMsg.text = template.substring(0, testEnd)
            template = advance(template, typeMsg.text.length)
            chars(typeMsg)
        }

    }

    return typeMsg.root
}
```



用于切割的advance方法

```js
function advance(template, n) {
    return template = template.substring(n)
}
```



三个开始中间结束的具体操作函数，这三个方法主要是树结构的构建

```js
function createAST(tagName, attrs) {
    return {
        tag: tagName,
        type: 1,
        children: [],
        attrs,
        parent
    }
}
```



```js
// 处理头部
function start(tagName, attrs, typeMsg) {
    const element = createAST(tagName, attrs)

    if (!typeMsg.root) {
        typeMsg.root = element
    }
    typeMsg.currentParent = element
    typeMsg.stack.push(element)
}

// 处理结尾
function end(tagName, typeMsg) {
    const element = typeMsg.stack.pop()
    typeMsg.currentParent = typeMsg.stack[typeMsg.stack.length - 1]

    if (typeMsg.currentParent) {
        element.parent = typeMsg.currentParent
        typeMsg.currentParent.children.push(element)
    }
}

// 处理中间文本
function chars(typeMsg) {
    typeMsg.text = typeMsg.text.trim()

    if (typeMsg.text.length > 0) {
        typeMsg.currentParent.children.push({
            type: 3,
            text: typeMsg.text
        })
    }
}
```



#### 转为render

这一步还是字符串拼接，即需要把上面得到的ast再搞成这个样子

```js
_c('div'),
    { id: "app", style: { "color": "blueviolet", "font-size": " 30px" } }
    , _v("我是" + _s(name)), _c('span'),
    { style: { "color": " rgb(150, 70, 16)" } }
    , _v(_s(age))
    , _c('p'),
    { class: "demo", style: { "color": "black" } }
    , _v("怎么玩转vue呢？")
```

_c:创建元素节点

_v:创建文本节点

_s:处理{{}}里面的数据

主要函数

```js

export function generate(node) {
    let code = `_c('${node.tag}'),
    ${node.attrs.length > 0
            ? `${formatProps(node.attrs)}`
            : undefined}
           ${node.children ? `,${formatChild(node.children)}` : ''}
            `
    return code

}
```

上面作为主要函数的同时也处理了拼完了`_c('div')`

下面是具体拼接函数

属性拼接：从ast中拿出当前节点属性，拼接成 `{ id: "app", style: { "color": "blueviolet", "font-size": " 30px" } }`这个样子

```js
// 拼接属性
function formatProps(attrs) {
    let attrStr = ''

    for (let i = 0; i < attrs.length; i++) {
        let attr = attrs[i]

        if (attr.name === 'style') {
            let styleAttrs = {}

            attr.value.split(';').map(item => {
                let [key, value] = item.split(':')
                styleAttrs[key] = value
            })
            attr.value = styleAttrs
        }

        attrStr += `${attr.name}:${JSON.stringify(attr.value)},`
    }

    return `{${attrStr.slice(0, -1)}}`

}
```



拼接子节点，这里主要是要处理文本节点

分为有无{{}}的情况，没有直接拼接_v()即可

有的话稍微复杂一点

拿下面这段为例

```js
我是{{name}}这是文字{{age}}
```

这里主要用到正则的可多次匹配`exec`方法

首先看看`/\{\{((?:.|\r?\n)+?)\}\}/g.exec('我是{{name}}这是文字{{age}}')`的返回值

返回值：` ["{{name}}", "name", index: 2, input: "我是{{name}}这是文字{{age}}", groups: undefined] `

通过此正则的捕获返回值可以拿到第一个{{}}出现的索引下标、{{}}的面的内容，同时通过这个返回值数组的第一项也可以拿到这个{{}}东西所占的字符长度

下面的操作和上面的生成AST是有些类似

首先第一次匹配到

![1600166031978](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600166031978.png)

将`我是`先push到一个临时数组容器中，然后拼接{{}}里面的东西，即拼接成_s(name)，再push到数组中去

下次循环指针移动，操作还是上面的操作数据片段篇push到临时容器中。即再把`这是文字`push进去，将age拼接好也push进去

![1600166170258](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600166170258.png)

即最后这个临时容器的数据是

```js
['我是','_s(name)','_s(age)']
```

当然可能还有这种情况:`我是{{name}}这是文字{{age}}还有蚊子`

那么只需出了循环再做一下判断即可，最后将容器的数据拼接成字符串

```js
// 拼接子节点

/**
 * 
 * 孩子有两种类型
 * 1. 元素节点
 * 2. 文本节点
 * 
 * 元素节点交给generate函数就可以了，这里主要就是处理文本
 */
//匹配{{}}
const defaultTagRE = /\{\{((?:.|\r?\n)+?)\}\}/g
function formatChild(children) {
    return children.map(child => generateChild(child)).join(',')
}
function generateChild(child) {

    switch (child.type) {
        case 1:

            return generate(child)

        // 主要逻辑：处理文本
        case 3:
            let text = child.text

            // 里面无{{}}，直接返回即可
            if (!defaultTagRE.test(text)) {
                return `_v(${JSON.stringify(text)})`;
            }

            //   下面是有{{}}的情况
            let match,
                index,
                lastIndex = defaultTagRE.lastIndex = 0,
                textArr = []
            while (match = defaultTagRE.exec(text)) {

                index = match.index

                // 把{{}}前面的文本截进去
                if (index > lastIndex) {
                    textArr.push(JSON.stringify(text.slice(lastIndex, index)))
                }

                // 拼接{{}}
                textArr.push(`_s(${match[1].trim()})`)

                // 把指针移动到{{}}的后面，以再次匹配后面的{{}}
                lastIndex = index + match[0].length
            }

            // 出循环时，lastIndex小于文本长度。即后面是有文本片段没有{{}}的情况，收集起来即可
            if (lastIndex < text.length) {
                textArr.push(JSON.stringify(text.slice(lastIndex)));
            }

            // 此时所有整理的数据均在textArr容器中，转为字符串即可
            // 即形式如：_v("你好，"+_s(name)),
            return `_v(${textArr.join('+')})`
    }
}
```

## 3. vue项目相关优化 

因为自己项目经验不足，所以也只能写一些自己做过的或者听说且比较重要的优化方案吧

### 3.1 从代码角度

1. 要写v-for的key，它的key主要用于diff。（可看一下上面的diff链接）

2. 活用v-show、v-if，用它们因根据使用场景。组件要进行频繁切换就用v-show，否则v-if
3. v-if，v-show不要一起用

### 3.2 从项目内容角度

1. 图片懒加载、路由懒加载组件。即不一次性全部请求，可加快资源返回速度；同时可能有些资源可能就用不到，也避免了请求浪费

2. 合理使用keep-alive缓存组件，对于一些使用频率非常高的组件可以把它缓存下来，避免了这个常用组件的反复创建与销毁

3. 图标尽量使用css图标，不用多余去请求图片资源了

4. 使用一些库（ui框架如elementUI）使用按需导入的形式
5. 及时释放组件资源（如绑定的事件等）

### 3.3 从打包角度

1. 像axios等插件、图片（大）可使用CDN引入（小图片可使用base64，虽加大一点打包体积但少了一次请求）

### 3.4 SPA的通病

单页面的通病肯定就是首屏问题，加一个 loading （小项目无所谓吧又感觉不到）

### 3.5 其他优化可使用webpack的配置来做了

链接：[webpack——从基础使用到手动实现]( https://juejin.im/post/6847009773448069128 )

## 4.  初尝vue3相关知识 

### 4.1 Composition API体验

![1600271328777](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600271328777.png)

### 4.2 手写一下reactive及简单实现其响应式

vue3的数据劫持是主要是利用了代理，注意这里是最简单。`像反复代理，数组的一些set操作等没有进行处理`(比如数组的push会走两遍set)

也就reactive的核心也就是代理，来简单实现以下吧。还是简单以拦截get、set为例。

vue3的数据劫持也是需要递归的，如我们reactive-一个有深度的对象。想比与vue2.x来说，它就是没有那么无脑递归

```js
function reactive(target) {
      return createReativeObject(target)
  }



  function createReativeObject(target) {
      //先判断其是否为对象
      if (isObject(target)) {

          let observed = new Proxy(target, {
              get(target, key, receiver) {

                  console.log('获取')
                  let res = Reflect.get(target, key, receiver)



                  return isObject(res) ? reactive(res) : res
              },
              set(target, key, value, receiver) {
                  console.log('设置')
                  let res = Reflect.set(target, key, value, receiver)
               
                  return res
              },
              
          })


          return observed
      }
      return target
  }
```

下面开始响应式的核心了，依赖的收集与派发更新

 vue3中effect算是这里的核心 ， 其会在 `mountComponent`、`doWatch`、`reactive`、`computed` 时被调用 

即这样

```js
const obj = reactive({ name: 'gxb' })
effect(() => {
    console.log(obj.name)
})
```

我们用reactive创建了一个响应式代理对象之后，后面跟着执行了一个副作用。这个副作用里面的回调会先进行调用，而后等到obj.name的数据发生了变化之后再次进行调用。即现在你可以这么理解，把副作用里面的回调先当成一个视图，视图首次先进行渲染，等到依赖的数据发生了变化之后进行更新。



来实现这个effect函数，这个函数的实现有些啰里啰嗦。下面还是以简化的形式进行编写。并且很烦的是源码中的变量老是和effect这个函数搞成一个名字...

```js
const stack = []

function effect(fn) {
    const effect = createReativeEffect(fn)
    effect()
}

function createReativeEffect(fn) {
    const effect = function() {
        return run(effect, fn)
    }
    return effect
}

function run(effect, fn) {

    stack.push(effect)
    fn()
    stack.pop()
}
```

源码简化之后的样子，effect函数中调用了一下createReativeEffect，createReativeEffect会返回一个函数，也即effect函数中的effect变量



这里的主要工作就是把一个effect放进准备好的栈中

最后推进去的是个啥呢？

即这个玩意

```js
ƒ () {
        return run(effect, fn)
    }
```

后面我们是一个属性数据可能对应多个这个玩意，属性发生了变化派发更新所做的事情也是靠调用了一下个玩意

我们的主要目前是属性发生了变动之后，最开始的副作用中的回调重新执行一遍。对应到这里正好是回调的执行是在run里面的。

至于为啥搞一个栈来存储这个玩意，和vue2.x中的watch栈那差不多。



接着看下面，fn回调执行了。肯定对触发数据的get方法。即这里也和vue2.x一样。get时进行依赖收集，set时进行派发更新

```js
function createReativeObject(target) {
      if (isObject(target)) {

          let observed = new Proxy(target, {
              get(target, key, receiver) {

                  console.log('获取')
                  let res = Reflect.get(target, key, receiver)

                  // 依赖收集
                  track(target, key)


                  return isObject(res) ? reactive(res) : res
              },
              set(target, key, value, receiver) {
                  console.log('设置')
                  let res = Reflect.set(target, key, value, receiver)
                  //派发更新
                  trigger(target, key)
                  return res
              },
              deleteProperty() {}
          })


          return observed
      }
      return target
  }
```

依赖收集使用track，派发更新使用track



先来实现track

先来看一下保存依赖的数据结构长啥样吧

```js
{
     target1:{     
     key:[effect,effect]
   }，
   	  target2:{     
     key:[effect,effect]
   	}
 
}
```

即这里需要一个三层的数据结构

因为target是对象，且为内存考虑。故最外层使用一个weakMap

里一层。key也可能是对象，故使用一个map吧

最层次，考虑到可能会做去重就使用set吧



下面操作就简单多了开整

```js
export function track(target, key) {
    先从effect栈中取对应effect
    const effect = stack[stack.length - 1]
    if (effect) {
        //创建好结构塞进去
        let depsMap = targetMap.get(target)

        if (!depsMap) {
            targetMap.set(target, depsMap = new Map)
        }
        let deps = depsMap.get(key)
        if (!deps) {
            depsMap.set(key, deps = new Set())
        }
        if (!deps.has(effect)) {
            deps.add(effect)
        }
    }
}
```

最开始是const obj = reactive({ name: 'gxb' })

也即保存成了这个样子

```js
{reactive({ name: 'gxb' }):{
    name:{effect}
}}
```

触发更新时，也即name属性数据发生了变化。将这个对应的effect取出来执行一下即可，也即最终还是走了其run里的回调。

```js
export function trigger(target, key) {
    const depsMap = targetMap.get(target)
    if (depsMap) {
        const deps = depsMap.get(key)
        if (deps) {
            deps.forEach(effect => {
                console.log(effect)
                effect()
            })
        }
    }
}
```

# 三、浏览器篇（补充知识）

浏览器的东西，我没有把它放到目前阶段所精读的...故只了解一下简单的

## 3.1 渲染引擎的工作（ Webkit ）

###  浅说主要流程

主要流程，如图

![1599791837921](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599791837921.png)

比较官方的

![1599793392614](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1599793392614.png)

我所画的流程是最开始看过一个修言大佬的文总结，不同文章的步骤流程有粗有细。再怎么变换最终现在阶段把握住最主要的即可吧

**最简化的流程介绍**

首先浏览器遇到html，解析html（生成DOM树）；遇到css解析css（生成CSSOM树）

DOM树与CSSOM树合并生成渲染树

值得注意的是：这个渲染树和开始的DOM树在节点结构上会有不同，渲染树只包括可视化的DOM节点，此时还要进行计算样式

再次注意的是：渲染树不包括节点位置及大小信息，故这是布局阶段所要做的事情

最终的绘制阶段就是使其显示在浏览器页面上了



最后再注意：这个步骤不是说就搞完html，再搞完css，再生成渲染树。它们是同步的，也即一边解析一边渲染

### 这里主要一个常见优化问题：重排与重绘

先说一下什么是重排与重绘

重排（回流）：操作引起DOM的几何尺寸的变化

重绘：操作只引发了样式改变

**为啥说这俩东西会影响渲染性能呢？**

其实知道了上面的东西，这里很好理解，触发了重排、重绘首先CSSOM需要更新。重绘还好说，只是样式发生了改变

它只需要跟新CSSOM树、更新渲染树再直接绘制即可

但是重排却不一样，因为一个节点的几何位置发生了改变除了会影响它本身的大小，还会影响它所在的环境（即它大了，占的位置多了，把它后面的东西不也挤跑了吗）

故重排需要从更新CSSOM树开始重新再走一遍流程

**故减少不必要的重排重绘（主要是重排）的发生，有助于优化渲染速度**



值得注意：还有一些操作可引发重排，节点操作、获取一些需要即时计算的值（ offsetTop 等）

## 

