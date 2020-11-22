# 【如果不想读文档的话】傻瓜式入门vue3





> 今天看完说唱新世代到了12点，一看反正时间已经这么完了就熬夜写了篇文档，后面有些困了...请多多担待
>
> 少说废话，直接开整

可任选一种创建方式

![1600550888325](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600550888325.png)

我个人是还有采用的脚手架

升级（即重装）

```bash
npm install -g @vue/cli # OR yarn global add @vue/cli
vue create hello-vue3
```

下面和原来不同的就是多了个选择版本的，选择vue3上手即可

##  一、重点

先以下面官网所列出的显著的为主（后三个先忽略）

![1600505937908](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600505937908.png)

### 1.1. composition API 

别不管 Options API 与composition API 比较，我们先直接上手composition API 

composition API 入口点`setup`函数

#### 创建响应式数据 `ref`与`reactive`

##### ref

ref函数接一个参数返回一个响应式的ref对象

直接看栗子

```js
<template>
  <div>
      {{num}}
  </div>
</template>
<script>
import { ref } from "vue";
export default {
  setup() {
    const num = ref(1);
    return { num };
  },
};
</script>
```

也即和vue2.x中的这种写法

```js
data(){
    return {
        num:1
    }
}
```

值得注意的是：ref返回的是一个对象，需要通过它的value属性拿到它的值

也即`num.value`。不过在模板中使用时他会自动解。也就是上面栗子中模板直接使用num，还有一种可自动解的情况先下面

另外现在就可以理解ref是用来把简单类型的数据进行响应式化的

##### reactive

则reactive就是将引用类型的数据进行响应式化

直接看栗子

```js
<template>
  <div>{{num}}{{obj.name}}</div>
</template>
<script>
import { ref, reactive } from "vue";
export default {
  setup() {
    const num = ref(1);
    const obj = reactive({
      name: "gxb",
      age: 18,
    });
    return { num, obj };
  },
};
</script>
```

上面的ref对象自动解套的另一种情况也即在这里它作为reactive函数参数对象的一属性

即:

```js
 const num = ref(1);
const obj = reactive({
      name: "gxb",
      age: 18,
      num
    });
```

值得注意的是：这里不要随意使用...语法或者结构，否则会丧失响应式性质，最后的工具函数那再写栗子介绍

##### readonly

这个函数，参数可以一个响应式或者普通的对象再或者是个ref。返回的是一个只读代理（深层的）

栗子

![1600540121924](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600540121924.png)

![1600540187092](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600540187092.png)

####  computed 与 watch 

##### computed

computed是一个函数，它需要传一个getter函数。其返回值是一个不可手动修改的ref对象

栗子

```js
<template>
  <div>{{num2}}</div>
</template>
<script>
import { ref, reactive, computed } from "vue";
export default {
  setup() {
    const num = ref(1);
    const obj = reactive({
      name: "gxb",
      age: 18,
      num,
    });
    const num2 = computed(() => num.value + 1);
    return { num, obj, num2 };
  },
};
</script>
```

注意不可修改

如

```js
 const num2=computed(()=>num.value+1)
    num2.value++
```

![1600536080863](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600536080863.png)

想要一个可以进行修改的，就需传一个具有get和set函数的对象

```js
<template>
  <div>{{num2.value}}</div>
</template>
<script>
import { ref, reactive, computed } from "vue";
export default {
  setup() {
    const num = ref(1);
    const obj = reactive({
      name: "gxb",
      age: 18,
      num,
    });
    const num2 = computed({
        get:()=>num,
        set:value=>num.value=value
    });
    
    num2.value=3
    return { num, obj, num2 };
  },
};
</script>
```

要注意的是，此时num2这个ref对象不会在自动解套了

##### watch 

**监听一个**

```js
<template>
  <div>{{num2.value}}</div>
</template>
<script>
import { ref, reactive, computed, watch } from "vue";
export default {
  setup() {
    const obj = reactive({
      name: "gxb",
      age: 18,
      num,
    });

    watch(
      () => obj.name,
      (name, preName) => {
        console.log(`new ${name}---old ${preName}`);
      }
    );
    setTimeout(() => {
      obj.name = "zhangsan";
    }, 1000);

    return { obj };
  },
};
</script>
```

![1600536735823](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600536735823.png)

第一参除了可以是上面那种有返回值的getter函数，也可以是一个ref对象

即

```js
<template>
  <div></div>
</template>
<script>
import { ref, reactive, computed, watch } from "vue";
export default {
  setup() {
    const obj = reactive({
      name: "gxb",
      age: 18,
      num,
    });
    const num = ref(0);
    watch(num, (name, preName) => {
      console.log(`new ${name}---old ${preName}`);
    });
    setTimeout(() => {
      num.value = 2;
    }, 1000);

    return { obj };
  },
};
</script>
```

![1600536918765](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600536918765.png)

**监听多个**

即如下只要num或obj.name有一个发生变动就触发监听器处理回调

```js
<template>
  <div></div>
</template>
<script>
import { ref, reactive, computed, watch } from "vue";
export default {
  setup() {
    const obj = reactive({
      name: "gxb",
      age: 18,
      num,
    });
    const num = ref(0);
    watch([num, ()=>obj.name], ([newNum, newName], [oldNum, oldName]) => {
      console.log(`new ${(newNum)},${(newName)}---old ${(oldNum)},${oldName}`);
    });
    setTimeout(() => {
      num.value = 6;
    //   obj.name = "zhangsan";
    }, 1000);

    return { obj };
  },
};
</script>
```

#### 生命周期钩子

栗子

```js
import { onMounted, onUpdated, onUnmounted } from 'vue'

const MyComponent = {
  setup() {
    onMounted(() => {
      console.log('mounted!')
    })
    onUpdated(() => {
      console.log('updated!')
    })
    onUnmounted(() => {
      console.log('unmounted!')
    })
  },
}
```

对应2.x钩子

![1600537935047](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600537935047.png)

#### props和this

##### props

setup这个入口函数接收的第一个参数就是props

栗子

![1600538263455](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600538263455.png)

这里需要注意，不要随便进行解构

即图省事

```js
  props: {
    data: String,
  },
  setup({ data }) {
    console.log(data);
  }
```

解构会使其丧失响应式的

##### this

2.x中拿组件实例实例很好拿，一般就是直接this，但是setup则不同。

但是组件实例上有许多api我们还是要使用的

故setup的第二个参数是一个上下文对象

栗子:派发一个自定义事件

![1600538684446](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600538684446.png)

值得注意的是这个context只是选择性的暴露了一些属性，如上面的emit还有attrs、slots

#### 依赖注入与Refs

##### 依赖注入

和vue2.x的provide和inject基本一样

栗子

![1600539039884](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600539039884.png)

为图简单我用的一个组件举例

![1600539128387](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600539128387.png)

它们的响应式需要自己出来一下（如用ref）

##### Refs

如拿下面这个节点

![1600539375797](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600539375797.png)

```js
<template>
  <div ref="test">test</div>
</template>
<script>
import {
  ref,
  reactive,
  computed,
  watch,
  provide,
  inject,
  onMounted,
} from "vue";
export default {
  setup() {
    const test = ref(null);
    onMounted(() => {
      console.log(test.value);
    });

    return { test };
  },
};
</script>
```

![1600539576828](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600539576828.png)

#### 一些工具函数

先来写下破坏reactive生成的响应对象代理的栗子

首先正常写法

```js
![宫小白vue3-02](C:\Users\T540P\Desktop\宫小白vue3-02.gif)<template>
  <div ref="test">
    {{obj.age}}
    <button @click="obj.age++">add</button>
  </div>
</template>
<script>
import { ref, reactive, computed, watch, provide, inject, readonly } from "vue";
export default {
  props: {
    data: String,
  },
  setup(props, context) {
    console.log(props.data);
    context.emit("test");
    const obj = reactive({
      name: "gxb",
      age: 18,

    });

    return { obj };
  },
};
</script>
```

![宫小白vue3-02](C:\Users\T540P\Desktop\宫小白vue3-02.gif)

使用扩展语法

```js
<template>
  <div ref="test">
    {{age}}
    <button @click="age++">add</button>
  </div>
</template>
<script>
import { ref, reactive, computed, watch, provide, inject, readonly } from "vue";
export default {
  props: {
    data: String,
  },
  setup(props, context) {
    console.log(props.data);
    context.emit("test");
    const obj = reactive({
      name: "gxb",
      age: 18,

    });

    return { ...obj };
  },
};
</script>
```

![宫小白vue3-03](C:\Users\T540P\Desktop\宫小白vue3-03.gif)

解构出来的同样不行

```js
<template>
  <div ref="test">
    {{age}}
    <button @click="age++">add</button>
  </div>
</template>
<script>
import { ref, reactive, computed, watch, provide, inject, readonly } from "vue";
export default {
  props: {
    data: String,
  },
  setup(props, context) {
    console.log(props.data);
    context.emit("test");
    const obj = reactive({
      name: "gxb",
      age: 18,
    });
    const { age } = obj;
    return { age };
  },
};
</script>
```

这个的原理也很简单，reactive的内部原理是Proxy，它操作均在返回的代理实例上



下面开始介绍几个工具函数

1. unref，参数是一个ref则返回这个ref的value属性，否则返本身
2. toRef，给一个 reactive 对象的属性创建一个 ref 
3. toRefs， 把一个响应式对象转换成普通对象，该普通对象的每个 property 都是一个 ref  
4. isRef，判断一个值是否是ref
5. isProxy，判断一个对象是否是由 `reactive` 或者 `readonly` 方法创建的代理。
6.  isReactive，判断一个对象是否是由 `reactive` 创建的响应式代理 
7. isReadonly，判断一个对象是否是由 `readonly` 创建的只读代理。 

仅给2、3写个例子吧

toRef，即把reactive 对象上的一个属性变成ref

什么意思呢，还是看上面的破坏了响应式的栗子

修复一下(即可把它一个属性抽出来做成响应式的ref，且它们还是互相关联的)

```js
<template>
  <div ref="test">
    {{age}}
    <button @click="age++">add</button>
  </div>
</template>
<script>
import {
  ref,
  reactive,
  computed,
  watch,
  provide,
  inject,
  readonly,
  toRef,
} from "vue";
export default {
  props: {
    data: String,
  },
  setup(props, context) {
    const obj = reactive({
      name: "gxb",
      age: 18,
    });
    const age=toRef(obj, "age");
    watch(()=>obj.age,(newAge,oldAge)=>{
        console.log(newAge);
    })
    return { age };
  },
};
</script>
```

![宫小白vue3-04](C:\Users\T540P\Desktop\宫小白vue3-04.gif)

toRefs则是把这个对象里面的所有属性均整成ref，这个修复则更简单了

```js
<template>
  <div ref="test">
    {{age}}
    <button @click="age++">add</button>
  </div>
</template>
<script>
import {
  ref,
  reactive,
  computed,
  watch,
  provide,
  inject,
  readonly,
  toRef,
  toRefs
} from "vue";
export default {
  props: {
    data: String,
  },
  setup(props, context) {
    const obj = reactive({
      name: "gxb",
      age: 18,
    });
    const obj02=toRefs(obj);
    
    return { ...obj02 };
  },
};
</script>
```





### 1.2. Teleport 

传送门，顾名思义

场景：某一些组件中我们可能需要一个模态框的功能，然而虽然逻辑上来说这个模态框是属于该组件中的，但是实际操作一般需要把这个框子挂到body上。Teleport 组件就是帮我们处理这个问题的

来看栗子

假设组件中需要有一个模态框

```vue
<template>
    <div>
        <model></model>
    </div>
</template>
<script>
import Model from './model'
export default {
    components:{Model}
}
</script>
```

模态框组件

```vue
<template>
  <div>
    <button @click="flag=true">点击</button>
    <teleport to="body">
      <div v-if="flag">
        <div>模态框</div>
      </div>
    </teleport>
  </div>
</template>
<script>
import { ref } from "vue";
export default {
  setup() {
    const flag = ref(false);
    return { flag };
  },
};
</script>
```

即teleport组件的作用就是把teleport标签里面的元素传送到body上去

![宫小白vue3-01](C:\Users\T540P\Desktop\宫小白vue3-01.gif)

再看层级

![1600507806345](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600507806345.png)

### 1.3. Fragments 

这个的理解就更简单了

原来只能这样:即只允许存在一个最外层的父元素`div`

```vue
<template>
  <div>
    ...
  </div>
</template>
```

现在可多个

```vue
<template>
  <div>
    ...
  </div>
  <div>
    ...
  </div>
	...
</template>
```

### 1.4. Emits Component Option 

#### 1.4.1 自定义事件派发

这里的重点：即多了一个派发事件的选项`emits`

也就我们以后再次使用emit派发一个事件的时候需要把这此派发的事件名放到选项里

栗子:

```vue
<template>
  <div>
    <button @click="$emit('test')">点击</button>
  </div>
</template>
<script>
export default {
  emits: ["test"],
};
</script>
```

注意：这里如果你派发的是一个原生事件，且没有把此事件放进emits选项中，其父组件的监听会被触发两次

```vue
<template>
  <div>
    <button @click="$emit('click')">点击</button>
  </div>
</template>
<script>
export default {
//   emits: ["click"],
};
</script>
```

![1600508878313](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600508878313.png)

#### 1.4.2 v-model

vue3中的`v-model`,所借助的属性是 `modelValue`  所借助的事件是 `update:modelValue `(且3中把sync移出掉了)

栗子

父组件

```vue
<template>
  <div id="nav">
    {{data}}
    <test05 v-model="data"></test05>
  </div>
</template>
<script>
import { ref } from "vue";
import Test05 from "./components/test05";
export default {
  components: { Test05 },
  setup() {
    const data=ref('gxb')
    return {data};
  },
};
</script>
```

子组件

```vue
<template>
  <div>
    <input type="text" :value="modelValue" @input="$emit('update:modelValue',$event.target.value)" />
  </div>
</template>
<script>
export default {
    props:{
        modelValue:String
    },
    emits:['update:modelValue']
}
</script>
```

自定义属性名，vue2.x中可通过 model选项  指定传过来的属性名和指定本次v-model要利用的事件。 vue3中也可自定属性

栗子

父组件(即在v-model后面指定绑定)

```js
<test05 v-model:foo="data"></test05>
```

子组件

```vue
<template>
  <div>
    <input type="text" :value="foo" @input="$emit('update:foo',$event.target.value)" />
  </div>
</template>
<script>
export default {
    props:{
        foo:String
    },
    emits:['update:foo']
}
</script>
```

一个组件中可写多个v-model指令

栗子：

父组件

```vue
 <test01 v-model:foo="a" v-model:bar="b"></test01>
```

子组件

```vue
<template>
  <div>
    <input type="text" :value="foo" @input="$emit('update:foo',$event.target.value)" />
    <input type="text" :value="bar" @input="$emit('update:bar',$event.target.value)" />
  </div>
</template>
<script>
export default {
  props: {
    foo: String,
    bar: String,
  },
  emits: ["update:foo", "update:bar"],
  setup(props) {
    return {};
  },
};
</script>
```

### 1.5. createRendererAPI 

自定义渲染器字如其名，它的主要功能是我们可以自定义 Virtual DOM 到DOM的方式，看了好3、4个大佬的栗子都是用canvas画了图。自己先不写了

## 二、其他

###  2.1 Global API

在vue2.x中没有应用的概念，vue2.x中的所谓的“app”也只不过是一个通过Vue构造出的实例。但是2.x中的一些全局API（像mixins、use、 component等 ）是直接在Vue构造函数中的

也即如果下面还有使用new Vue的“应用”，这些全局API很容易造成污染

下面来看一眼vue3的入口文件

```vue
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'

createApp(App).use(store).use(router).mount('#app')
```

现在有了一个createApp，这个方法就返回一个应用程序的实例

拿component写个栗子

```vue
import { createApp, h } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'

createApp(App)
    .component('test06', {
        render() {
            return h('div', {}, '全局组件')
        }
    })
    .use(store)
    .use(router)
    .mount('#app')
```

其他API相应改变，如官网

![1600512935838](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600512935838.png)

#### Global API Treeshaking

官网是以 Vue.nextTick() ，这个全局API来举例的

这个摇树是什么玩意呢？

![1600543520369](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600543520369.png)

通过这句话可以理解，即这些个API不分青红皂白就写死在vue的构造函数上，如果我们在应用中本就没有对这些个API进行使用。那么打包时把这些东西也打进去是不是浪费性能同时也增大了打包体积呢



故vue3中，nextTick的使用也是需要从vue中导入一下的

```js
import { nextTick } from 'vue'

nextTick(() => {
  ...
})
```

其他受影响的API

![1600543795916](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600543795916.png)

### 2.2 Template Directives

#### v-model

v-model上面已经写了，去掉了`.sync`，使用v-model进行了统一

#### v-if、v-for优先级问题

在2.x是v-for优先级高，在3.0中v-if的优先级高

### 2.3 Components

#### 函数式组件

因为在vue3中函数式组件现在在性能上的提升可以忽略不计，还是推荐使用状态组件。

并且这里函数式组件只能通过纯函数进行声明，只能接受props和context（也是emit、slots、attrs）

简单搞个例子

这里偷个懒吧，把官网的栗子拿过来

vue2.x

```js
// Vue 2 Functional Component Example
export default {
  functional: true,
  props: ['level'],
  render(h, { props, data, children }) {
    return h(`h${props.level}`, data, children)
  }
}
```

vue3,区别除了下面所要说的h函数变化问题，还有上面提到的参数问题，再一个变化就是去掉了 `functional: true,`

```js
import { h } from 'vue'

const DynamicHeading = (props, context) => {
  return h(`h${props.level}`, context.attrs, context.slots)
}

DynamicHeading.props = ['level']

export default DynamicHeading
```

单文件形式对比

2.x

```js
// Vue 2 Functional Component Example with <template>
<template functional>
  <component
    :is="`h${props.level}`"
    v-bind="attrs"
    v-on="listeners"
  />
</template>

<script>
export default {
  props: ['level']
}
</script>
```

3.0,区别去掉了`functional`,监听器放进了$attrs且可删除

```js
<template>
  <component
    v-bind:is="`h${props.level}`"
    v-bind="$attrs"
  />
</template>

<script>
export default {
  props: ['level']
}
</script>
```

#### 异步组件

原来异步组件咋整的呢

```js
const asyncPage = () => import('./NextPage.vue')
```

或者带选项的

```js
const asyncPage = {
  component: () => import('./NextPage.vue'),
  delay: 200,
  timeout: 3000,
  error: ErrorComponent,
  loading: LoadingComponent
}
```



但是vue3中不一样了，这里有一个新的API ` defineAsyncComponent `用来显示定义异步组件

也即

```js
const asyncPage = defineAsyncComponent(() => import('./NextPage.vue'))
```

或

```js
const asyncPageWithOptions = defineAsyncComponent({
  loader: () => import('./NextPage.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

细心看也可看出`component`改成了`loader`

还有一点不同的是，2.x中函数参数中可接收`resolve,reject`,3.0则不可，但是必须要返回一个Promise

### 2.4 Render Function

#### 渲染函数的改变

即原来的h函数是这样的

```vue
export default {
  render(h) {
    return h('div')
  }
}
```

而现在h函数则需要从vue的再导入进来

其实我上面有一个栗子已经用到了，再拿过来一次

```vue'
import { createApp, h } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'

createApp(App)
    .component('test06', {
        render() {
            return h('div', {}, '全局组件')
        }
    })
    .use(store)
    .use(router)
    .mount('#app')
```

还有一个属性的变动，直接拿官网的栗子吧

2.x 中的节点属性格式

```js
{
  class: ['button', 'is-outlined'],
  style: { color: '#34495E' },
  attrs: { id: 'submit' },
  domProps: { innerHTML: '' },
  on: { click: submitForm },
  key: 'submit-button'
}
```

在3.0中，这些属性不再被嵌套，被展平了(这看起来更像DOM节点上的东西了吧)

```js
{
  class: ['button', 'is-outlined'],
  style: { color: '#34495E' },
  id: 'submit',
  innerHTML: '',
  onClick: submitForm,
  key: 'submit-button'
}
```

#### 插槽方面

废掉了`$scopedSlots`，使用`$slots`

vue2.x中，一个组件使用渲染函数拿插槽是这样的

```vue
<script>
export default {
    render(h) {
        return h('div',{},this.$scopedSlots.default)
    },
}
</script>
```

vue3.x中则是这样的

```vue
<script>
import {h} from 'vue'
export default {
    props:{
        data:String
    },
    render() {
        return h('div',{},this.$slots.default())
    },
}
</script>
```

![1600532426696](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600532426696.png)

### 2.5 Custom Elements

#### 自定义元素白名单

如一些特殊的组件，我们要特殊用处的希望vue的编译忽略

栗子

直接往组件中放一个为注册过的组件

```vue
 <test08></test08>
```

![1600549345967](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1600549345967.png)

不希望出现这个错就把它放进白名单里

使用构建工具版本

```js
rules: [
  {
    test: /\.vue$/,
    use: 'vue-loader',
    options: {
      compilerOptions: {
        isCustomElement: tag => tag === 'test08'
      }
    }
  }
  // ...
]
```

 运行时编译版本 

```js
const app = Vue.createApp({})
app.config.isCustomElement = tag => tag === 'test08'
```

#### is只能用在`<component>`上

但是 `<component :is="componentId"></component> `除了这个玩意，我们别的地方可能也要需求需要用到is怎么办呢

故vue3中推出了v-is指令