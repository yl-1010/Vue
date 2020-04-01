- [Vue](#vue)
  * [Vue.js 运行机制全局概览](#vuejs---------)
      - [初始化及挂载](#------)
      - [编译](#--)
        * [Ⅰ.parse](#-parse)
        * [Ⅱ.optimize](#-optimize)
        * [Ⅲ.generate](#-generate)
      - [响应式](#---)
      - [Virtual DOM](#virtual-dom)
  * [教程](#--)
        * [Ⅰ.Prop](#-prop)
  * [API](#api)
    + [选项 / 数据](#-------)
        * [Ⅰ.data](#-data)
        * [Ⅱ.props](#-props)
        * [Ⅲ.propsData](#-propsdata)
        * [Ⅳ.computed](#-computed)
        * [Ⅴ.methods](#-methods)
        * [Ⅵ.watch](#-watch)
    + [选项 / DOM](#-----dom)
        * [Ⅰ.el](#-el)
        * [Ⅱ.template](#-template)
    + [选项 / 生命周期钩子](#-----------)
    
### Vue.js 运行机制全局概览

---
![image](https://note.youdao.com/yws/public/resource/ac2c1bd4cc84a9a9998a55bb2e8ef318/xmlnote/383094E023A94E5AA6B0C54DF3584E42/191)

##### 初始化及挂载

![image](https://note.youdao.com/yws/public/resource/ac2c1bd4cc84a9a9998a55bb2e8ef318/xmlnote/E04924C47A3F439D9751385E8C598B5B/197)
在 new Vue() 之后。 Vue 会调用 _init 函数进行初始化，也就是这里的 init 过程，它会初始化生命周期、事件、 props、 methods、 data、 computed 与 watch 等。其中最重要的是通过 Object.defineProperty 设置 setter 与 getter 函数，用来实现「响应式」以及「依赖收集」

初始化之后调用 $mount 会挂载组件，如果是运行时编译，即不存在 render function 但是存在 template 的情况，需要进行「编译」步骤。
##### 编译
compile编译可以分成 parse、optimize 与 generate 三个阶段，最终需要得到 render function。

![image](https://note.youdao.com/yws/public/resource/ac2c1bd4cc84a9a9998a55bb2e8ef318/xmlnote/32C1CFE08FAA479FB7E04EED8D999CA6/205)
###### Ⅰ.parse
parse 会用正则等方式解析 template 模板中的指令、class、style等数据，形成AST。
###### Ⅱ.optimize
optimize 的主要作用是标记 static 静态节点，这是 Vue 在编译过程中的一处优化，后面当 update 更新界面时，会有一个 patch 的过程， diff 算法会直接跳过静态节点，从而减少了比较的过程，优化了 patch 的性能。
######  Ⅲ.generate
generate 是将 AST 转化成 render function 字符串的过程，得到结果是 render 的字符串以及 staticRenderFns 字符串。

在经历过 parse、optimize 与 generate 这三个阶段以后，组件中就会存在渲染 VNode 所需的 render function 了。

##### 响应式
![image](https://note.youdao.com/yws/public/resource/ac2c1bd4cc84a9a9998a55bb2e8ef318/xmlnote/9E9339BB061E45BDB153433974A208C5/219)

这里的 getter 跟 setter 已经在之前介绍过了，在 init 的时候通过 Object.defineProperty 进行了绑定，它使得当被设置的对象被读取的时候会执行 getter 函数，而在当被赋值的时候会执行 setter 函数。

当 render function 被渲染的时候，因为会读取所需对象的值，所以会触发 getter 函数进行「依赖收集」，「依赖收集」的目的是将观察者 Watcher 对象存放到当前闭包中的订阅者 Dep 的 subs 中。形成如下所示的这样一个关系。

![image](https://note.youdao.com/yws/public/resource/ac2c1bd4cc84a9a9998a55bb2e8ef318/xmlnote/FC53E117B07B410D94FE3C4BD6A6D88A/223)

在修改对象的值的时候，会触发对应的 setter， setter 通知之前「依赖收集」得到的 Dep 中的每一个 Watcher，告诉它们自己的值改变了，需要重新渲染视图。这时候这些 Watcher 就会开始调用 update 来更新视图，当然这中间还有一个 patch 的过程以及使用队列来异步更新的策略


##### Virtual DOM
render function 会被转化成 VNode 节点。Virtual DOM 其实就是一棵以 JavaScript 对象（ VNode 节点）作为基础的树，用对象属性来描述节点，实际上它只是一层对真实 DOM 的抽象。最终可以通过一系列操作使这棵树映射到真实环境上。由于 Virtual DOM 是以 JavaScript 对象为基础而不依赖真实平台环境，所以使它具有了跨平台的能力，比如说浏览器平台、Weex、Node 等。


```
{
    tag: 'div',                 /*说明这是一个div标签*/
    children: [                 /*存放该标签的子节点*/
        {
            tag: 'a',           /*说明这是一个a标签*/
            text: 'click me'    /*标签的内容*/
        }
    ]
}
```
渲染后可以得到

```
<div>
    <a>click me</a>
</div>
```
![image](https://note.youdao.com/yws/public/resource/ac2c1bd4cc84a9a9998a55bb2e8ef318/xmlnote/18FF7FF4B75E417AA2A39DBFD7514FC4/239)

当数据变化后，执行 render function 就可以得到一个新的 VNode 节点，我们如果想要得到新的视图，最简单粗暴的方法就是直接解析这个新的 VNode 节点，然后用 innerHTML 直接全部渲染到真实 DOM 中。

### 教程
###### Ⅰ.Prop
prop 定义了这个组件有哪些可配置的属性，组件的核心功能也都是它来确定的。写通用组件时，props 最好用对象的写法，这样可以针对每个属性设置类型、默认值或自定义校验属性的值

```
<template>
  <button :class="'i-button-size' + size" :disabled="disabled"></button>
</template>
<script>
  // 判断参数是否是其中之一
  function oneOf (value, validList) {
    for (let i = 0; i < validList.length; i++) {
      if (value === validList[i]) {
        return true;
      }
    }
    return false;
  }

  export default {
    props: {
      size: {
        validator (value) {
          return oneOf(value, ['small', 'large', 'default']);
        },
        default: 'default'
      },
      disabled: {
        type: Boolean,
        default: false
      }
    }
  }
</script>
```
单向数据流：父级 prop 的更新会向下流动到子组件中，但是反过来则不行

prop验证：

```
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```


### API

---

#### 选项 / 数据

###### Ⅰ.data
- 类型：Object | Function
- 限制：组件的定义只接受 function
- 介绍： Vue 将会递归将 data 的属性转换为 getter/setter，
 Vue 实例也代理了 data 对象上所有的属性，因此访问 vm.a 等价于 vm.$data.a

```
var data = { a: 1 }

// 直接创建一个实例
var vm = new Vue({
  data: data
})
vm.a // => 1
vm.$data === data // => true

// Vue.extend() 中 data 必须是函数
var Component = Vue.extend({
  data: function () {
    return { a: 1 }
  }
})
注意，如果你为 data 属性使用了箭头函数，则 this 不会指向这个组件的实例，不过你仍然可以将其实例作为函数的第一个参数来访问。

data: vm => ({ a: vm.myProp })
```

###### Ⅱ.props
- 类型：Array<string> | Object
- 介绍：props 可以是数组或对象，用于接收来自父组件的数据。props 可以是简单的数组，或者使用对象作为替代，对象允许配置高级选项，如类型检测、自定义验证和设置默认值。

###### Ⅲ.propsData
- 类型：{ [key: string]: any }
- 限制：只用于 new 创建的实例中

```
var Comp = Vue.extend({
  props: ['msg'],
  template: '<div>{{ msg }}</div>'
})

var vm = new Comp({
  propsData: {
    msg: 'hello'
  }
})
```

###### Ⅳ.computed
- 类型：{ [key: string]: Function | { get: Function, set: Function } }
- 介绍：计算属性将被混入到 Vue 实例中。所有 getter 和 setter 的 this 上下文自动地绑定为 Vue 实例
```
computed: {
  aDouble: vm => vm.a * 2
}

```
！！！计算属性的结果会被缓存，除非依赖的响应式属性变化才会重新计算


```
var vm = new Vue({
  data: { a: 1 },
  computed: {
    // 仅读取
    aDouble: function () {
      return this.a * 2
    },
    // 读取和设置
    aPlus: {
      get: function () {
        return this.a + 1
      },
      set: function (v) {
        this.a = v - 1
      }
    }
  }
})
vm.aPlus   // => 2
vm.aPlus = 3
vm.a       // => 2
vm.aDouble // => 4
```
###### Ⅴ.methods
- 类型：{ [key: string]: Function }
- 详细：methods 将被混入到 Vue 实例中。可以直接通过 VM 实例访问这些方法，或者在指令表达式中使用。方法中的 this 自动绑定为 Vue 实例。

！！！不应该使用箭头函数来定义 method 函数

###### Ⅵ.watch
- 类型：{ [key: string]: string | Function | Object | Array }
- 详细：一个对象，键是需要观察的表达式，值是对应回调函数。值也可以是方法名，或者包含选项的对象。Vue 实例将会在实例化时调用 $watch()，遍历 watch 对象的每一个属性。

```
var vm = new Vue({
  data: {
    a: 1,
    b: 2,
    c: 3,
    d: 4,
    e: {
      f: {
        g: 5
      }
    }
  },
  watch: {
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
    // 方法名
    b: 'someMethod',
    // 该回调会在任何被侦听的对象的 property 改变时被调用，不论其被嵌套多深
    c: {
      handler: function (val, oldVal) { /* ... */ },
      deep: true
    },
    // 该回调将会在侦听开始之后被立即调用
    d: {
      handler: 'someMethod',
      immediate: true
    },
    e: [
      'handle1',
      function handle2 (val, oldVal) { /* ... */ },
      {
        handler: function handle3 (val, oldVal) { /* ... */ },
        /* ... */
      }
    ],
    // watch vm.e.f's value: {g: 5}
    'e.f': function (val, oldVal) { /* ... */ }
  }
})
vm.a = 2 // => new: 2, old: 1
```

!!!不应该使用箭头函数来定义 watcher 函数(箭头函数绑定父级作用域上下文)

####  选项 / DOM
###### Ⅰ.el
- 类型：string | Element
- 限制：只在用 new 创建实例时生效
- 提供一个在页面上已存在的 DOM 元素作为 Vue 实例的挂载目标。可以是 CSS 选择器，也可以是一个 HTMLElement 实例。在实例挂载之后，元素可以用 vm.$el 访问。
###### Ⅱ.template
- 类型：string
- 详细：一个字符串模板作为 Vue 实例的标识使用。模板将会 替换 挂载的元素。挂载元素的内容都将被忽略，除非模板的内容有分发插槽。
如果值以 # 开始，则它将被用作选择符，并使用匹配元素的 innerHTML 作为模板。常用的技巧是用 <script type="x-template"> 包含模板。
####  选项 / 生命周期钩子
- beforeCreate
- created
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeDestroy
- destroyed
