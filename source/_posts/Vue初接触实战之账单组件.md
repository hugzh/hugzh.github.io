title: Vue初接触实战之账单组件
date: 2016-08-13 11:20:36
tags: 
    - Vue 
    - MVVM
    - Vuex
categories:
    - 前端开发
---

Vue作为一个构建数据驱动web界面的库，是去年最火的MVVM风格库之一。Vue的用起来有Angular的影子，把很多自定义指令注入html，又吸收了React的优点和精华。比如与Vue的配套使用的Vuex就是从Redux和Flux中借鉴了不少思路。<!-- more -->
Vue从去年的下半年开始火起来，目前在Awesomes前端资源库的热度排行里面处于Top2的位置。得益于更加简洁的语法和易用性，Vue目前在社区的受欢迎度不在React之下。很多公司纷纷上手Vue了，我厂的前端团队也已经在使用Vue进行业务重构了。即将发布的Vue2.0版本也采用了和React一样的Virtual Dom技术,而且推出了前后端同构的服务端渲染框架vue-hackernews-2.0，这将会使得Vue在技术社区更加受追捧。目前[Vue2.0的RC文档](http://rc.vuejs.org/guide/)也已经出来了，可以预期，它的使用前景将会非常非常不错。

## 这里要讲的是啥？
这篇文章不是普及Vue的内部实现原理，也不是要比较MVVM三大法器Angular、React和Vue的优劣。这里主要是对自己最近上手学习Vue，进行SPA开发过程中的思路总结。希望通过展示一个分期账单组件，可以对像我一样的Vue初学者提供一丁点的帮助。因为是Vue新手,如有错误之处，也欢迎各位大神批评指正。

## Vuex
首先要介绍的是[Vuex](http://vuex.vuejs.org/en/index.html)这个神器。关于Vue的基本语法，可以直接打开[官方1.0文档](https://vuejs.org.cn/guide/)学习。当然，如果你想直接上手Vue2.0，也可以直接[访问这里](http://rc.vuejs.org/guide/)。
>Vuex 是一个专门为 Vue.js 应用所设计的集中式状态管理架构。它借鉴了 Flux 和 Redux 的设计思想，但简化了概念，并且采用了一种为能更好发挥 Vue.js 数据响应机制而专门设计的实现。

也就是说，Vuex将父组件与子组件之间的props传递，组件与组件之间的消息传递集中起来管理。在一个小型应用中，我们可能不会用到Vuex，这样会把原本很简单的任务复杂化了。但是，企业级的项目都是多条业务线交叉的，如果单纯使用Vue本身的组件通信，业务组件之间复杂的关系网会让项目后期的调试和Bug跟踪非常困难，尤其是当你在构建一个SPA项目的时候。
> 为了更好的解决在大型应用中状态的共用问题，我们需要对组件的 组件本地状态(component local state) 和 应用层级状态(application level state) 进行区分。应用级的状态不属于任何特定的组件，但每一个组件仍然可以监视(Observe)其变化从而响应式地更新 DOM。通过汇总应用的状态管理于一处，我们就不必到处传递事件。

Vuex的数据流模型如下图所示：
![vuex数据流模型](http://vuex.vuejs.org/en/vuex.png)
1. 用户在组件中的输入操作触发 action 调用；
2. Actions 通过分发 mutations 来修改 store 实例的状态；
3. Store 实例的状态变化反过来又通过 getters 被组件获知，组件获悉状态变更之后就是数据驱动的魔法——实时更新DOM状态。

需要注意的一点是，mutation本身是一个事件系统，通过定义事件来触发Store的状态变更。mutation里面定义的函数必须是同步函数，涉及到API调用的逻辑要放到Action进行，因为Action是可以定义异步函数的。

## Vue-route
介绍完Vuex，我们来说说[vue-router](http://router.vuejs.org/zh-cn/index.html)。vue-router是Vue官方提供的路由插件，通过vue-router配合Vuex，我们可以非常高效地开发大型SPA。vue-router最基本的作用是做SPA路由映射。
```
router.map({
    '/foo': {
        component: Foo
    },
    '/bar': {
        component: Bar
    }
})
```
上面的配置中，当访问路径"/foo"的时候，SPA就会在<router-view>挂载组件Foo，改变访问路径为 "/bar"，Bar组件就会切换到主视口。这就是一个最基本的SAP路由配置。

## 一步一步使用Vue构建S一个PA账单组件
在简单了解了Vuex和Vue-router的基本概念之后，我们可以进入实践环节。如果还没有完全理解清楚Vue的语法和Vuex数据流的概念，可以继续多看几次官方文档，尤其是对刚接触MVVM的人来说，可能要多看几次才能对数据驱动的编程理念有更好的理解。
### 1、项目目录
```
|-vue-demo
    |-node_modules

    |-order
        |-app
            |-components
                |-All.vue

                |-Latest.vue

                |-Nav.vue

                |-Order.vue
            |-vuex
                |-modules
                    |-orderList.js
                |-action.js

                |-mutation.js

                |-mutation-type.js

                |-store.js
            |-App.vue

            |-main.js

            |-router.js
        |-index.html

        |-webpack.config.js
    |-package.json

    |-README.md

```
这个是比较推荐的项目目录结构。其中order是我们要展示的账单组件的根目录，组件目录划分为入口index.html、webpack配置文件和app文件夹。在app目录内：

* App.vue - 组件的根节点;
* main.js - 组件入口;
* router.js - 路由相关配置;
* component - 组件根节点下的各个子组件;
* vuex - 顾名思义，数据流架构vuex相关的文件，包括action、mutation和store。

在大型项目中，组件数量多，涉及的数据流是比较复杂的。为了更好地管理store，我们又把store定义在不同的模块中，比如modules目录下的orderList就是账单列表相关的数据流。事件类型比较多的情况下，我们把事件名称定义在mutation-type中。
### 2、创建组件入口
组件入口需要做的事情包括创建Vue组件实例、挂载插件、配置路由，main.js如下:
```
import Vue from 'vue'
import VueRouter from 'vue-router'
import ConfigRouter from './router'
import App from './App.vue'
// import './style/order.css'
Vue.config.devtools = true

Vue.use(VueRouter)

// 创建vue-router实例
var router = new VueRouter()
//配置路由
ConfigRouter(router)

// 滚动到页面顶部
router.beforeEach(function() {
  window.scrollTo(0, 0)
})

//全路径匹配，防止出现404
router.redirect({
  '*': '/'
})
//启动APP
router.start(App, '#root')
```

### 3、使用vue-router配置SPA路由
账单组件的路由包括“最近7天待还”以及“全部待还”，router.js:

```
export default function(router) {
  router.map({
    '/': {
      component: require("./components/Latest.vue"),
      linkActiveClass: 'active'
    },
    '/latest': {
      component: require("./components/Latest.vue"),
      linkActiveClass: 'active'
    },
    '/all': {
      component: require("./components/All.vue"),
      linkActiveClass: 'active'
    }
  })
}
```

### 4、创建 Vuex Store
尝试列出组件用到的所有数据，在vuex目录下创建store.js文件：
```
// vuex/store.js
import Vue from 'vue'
import Vuex from 'vuex'
// 导入各个模块的初始状态和 mutations
import orderList from './modules/orderList'

Vue.use(Vuex)

export default new Vuex.Store({
  // 组合各个模块
  modules: {
    orderList
  }
})
```
由于我们把store拆分到不同的模块，所以创建store实例的时候需要引入orderList模块，它包括账单列表orders对象和当前被激活的账单对象activeOrder。需要定义的mutation只有一个"SHOW_DETAIL",当用户点击账单列表的某一个账单的时候，SHOW_DETAIL更新store的activeOrder，表示当前被展开的账单对象。modules/orderList.js如下：

```
// vuex/modules/index.js
import {
  SHOW_DETAIL,
} from '../mutation-types'

// 该模块的初始状态
const state = {
  orders: [{
    id: 'aedf9c25',
    tradeDate: '2016.04.08',
    name: 'Nike跑步鞋青年版',
    totalPrice: 888,
    repayDate: '2016.08.08',
    capital: '880',
    interest: '5.5',
    extra: '2.5',
    type: '消费',
    currentTerm: 1,
    totalTerms: 1
  }, {
    id: 'cves9chs',
    tradeDate: '2016.04.10',
    name: '支付宝提现',
    totalPrice: 773.5,
    repayDate: '2016.08.10',
    capital: '769',
    interest: '4.5',
    extra: '0',
    type: '现金',
    currentTerm: 1,
    totalTerms: 4
  }, {
    id: 'deef1d3g',
    tradeDate: '2016.05.15',
    name: '喜洋洋抱枕',
    totalPrice: 204.8,
    repayDate: '2016.08.15',
    capital: '203.5',
    interest: '1.3',
    extra: '0',
    type: '消费',
    currentTerm: 2,
    totalTerms: 3
  }],
  activeOrder: {}
}

// 相关的 mutations
const mutations = {
  [SHOW_DETAIL](state, id) {
    state.activeOrder = state.orders.find(function(ele) {
      return ele.id == id;
    });
  }
}

export default {
  state,
  mutations
}
```
mutation-types.js:
```
export const SHOW_DETAIL = 'SHOW_DETAIL'
```

###  5、Action
Actions 是组件内用来分发 mutations 的函数。第一个参数固定为store。在这里，当用户点击账单列表的某一个账单区域的时候，要调用dispatch('SHOW_DETAIL')。这里的demo只涉及到一个简单的用户事件，所以action.js也比较简单：
```
// index actions
export const showDetail = makeAction('SHOW_DETAIL')

function makeAction(type) {
  return ({
    dispatch
  }, ...args) => dispatch(type, ...args)
}
```

###  6、Vue组件
我们在第一步，创建入口文件main.js的时候import进来App.vue这个根组件：
```
import App from './App.vue'
...
router.start(App, '#root')
```
定义App.vue也很简单，创建一个Vue实例，把vuex的store注入到根实例即可，这样组件内的所有子组件都能访问到store：
```
// App.vue
<script>
  import Nav from './components/Nav.vue'
  import store from './vuex/store'

  export default {
    name: 'App',
    // 注入store到根组件
    store,
    data() {
      return {}
    },

    components: {
      //子组件order-nav
      'order-nav': Nav
    }
  }
</script>
<template>
  <div id="app">
    <order-nav></order-nav>
    <router-view></router-view>
  </div>
</template>
```

这里的order-nav是一个导航栏组件，包括“最近7天待还”和“全部待还”两个，由于"全部待还”的展示和“最近7天待还”基本一样，demo里面就没有再做实现，只提供一个占位方便展示router切换。Latest.vue就是最近7天待还子组件，它获取近7天待还账单列表，并在下一级子组件(order)中渲染组件列表。组件获取store的数据是通过getter来实现的。

```
// Latest.vue
<script>
  import { showDetail} from '../vuex/actions';
  import Order from './Order.vue';

  export default {
    name: 'Latest',

    data() {
      return {
      }
    },
    components:{
      'order':Order
    },
    vuex: {
     //解构函数，{orderList}对象指的是store的orderList模块的state.orderList
      getters: {
        orders: ({orderList}) => orderList.orders
      },
      //注入actoin
      actions: {
        showDetail
      },
      computed:{
      }
    }
  }
</script>
<template>
  <div class="container-fluid">
    <ul class="list-unstyled row">
      <order></order>
    </ul>
  </div>
</template>
```

Order.vue是这个账单组件中负责内容显示和用户事件分发的组件模块。Order组件中包括比较详细的Vue指令语法，比如v-for,v-show,track-by等。为了增强用户体验，Order组件在用户点击账单展示详情的时候，通过定义trasitoin属性达到简单的动画切换效果。

```
// Order.vue
<script>
  import {fold,showDetail} from '../vuex/actions'
  export default {
    name: 'Order',
    vuex: {
      getters: {
        orders: ({orderList}) => orderList.orders,
        activeOrder:({orderList})=>orderList.activeOrder
      },
      actions:{
        showDetail
      }
    }
  }
</script>
<template>
  <li  class="bill" v-for="order in orders" @click="showDetail(order.id)" track-by="id">
   <div>
      <h4>{{order.name}}</h4>
      <p>待还
      <span class="text-danger">{{order.totalPrice}}</span>元&nbsp;&nbsp;
      <span><small>[{{order.currentTerm}}/{{order.totalTerms}}]</small></span>
      </p>
    </div>
    <div class="bill-detail" v-show="order.id==activeOrder.id" transition="fade">
        <p>
        <div class="order-item">最后还款日期：{{order.repayDate}}</div>
        <div class="order-item">交易类型期：{{order.type}}</div>
        <div class="order-item">应还本金：{{order.capital}}元</div>
        <div class="order-item">应还利息：{{order.interest}}元</div>
        <div class="order-item">手续费：{{order.extra}}元</div>
        <div class="order-item">交易日期：{{order.tradeDate}}</div>
        </p> 
    </div>
  </li>
</template>

<style media="screen">
.bill {
  border-top:2px solid #e7e7e7;
  border-bottom: 2px solid #e7e7e7;
  margin: 5px; 
  padding: 10px;
  cursor: pointer;
}
.bill-detail {
  padding: 0 10px;
}
.order-item {
  display: inline-block;
  width: 45%;
}
/* 过渡效果 */
.fade-transition {
  transition: all .8s ease;
}
.fade-enter,
.fade-leave {
  height: 0;
  opacity: 0;
}
</style>

```
最后附上的是组件导航子组件，主要是负责router路由切换的Nav.vue:

```
// Nav.vue
<script>
  export default {
    name: 'nav',
    vuex: {
      getters: {
        current: ({orderList}) => orderList.activeOrder
      }
    }
  }
</script>
<template>
  <nav class="navbar navbar-default">
    <div class="container-fluid">
      <div class="navbar-header">
        <a class="navbar-brand" href="#" v-link="{ path: '/' }">Vue-订单demo</a>
      </div>
      <div class="collapse navbar-collapse">
        <ul class="nav navbar-nav">
          <li v-link="{ path: '/latest',activeClass:'active'}"><a href="#!">近7天待还 <span class="sr-only">(current)</span></a></li>
          <li v-link="{ path: '/all',activeClass:'active'}"><a href="#!"> 全部待还 </a></li>
        </ul>
      </div>
    </div>
  </nav>
</template>
```

至此，我们已经一步步实现了一个基于Vue+Vuex+vue-router搭建的SPA组件demo，如果大家还没学会，可以直接去把完整的demo看一遍，喜欢的话也麻烦给个star。[点击这里](https://github.com/hugzh/vue-demo)进入账单组件的github地址。


