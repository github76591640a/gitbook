### Vue构造函数

#### 1. 绑定在Vue构造函数上面的属性和方法

​	了解Vue源码，我们首先需要找到一个切入点，打开package.json文件。这个文件里面script下面的dev属性执行命令：

```javascript
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev"
```

​	首先我们了解到，这个文件执行了scripts/config.js文件中的代码，现在我们打开此文件。 在这个文件当中我们能看到：

```javascript
'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  },
```

​	此文件引入了web/entry-runtime-with-compiler.js这个文件，那web是哪个目录呢，不要着急，这个文件中有aliases这个东西，就是他了，看aliases中，我们知道了，web是指向的src/platforms/web。直到现在我们终于找到了Vue这个构造函数，打开entry-runtime-with-compiler.js文件，我们发现没有定义，只是引入了。

```javascript
import Vue from './runtime/index'
```

​	这没得办法了， 我们只能接着找了，首先到这里我们需要了解一点，那就是通过import引入文件的时候是先执行被引入的文件，才转头执行此文件夹中的代码的。好既然这样我们就接着找吧，没得办法了。打开runtime/index文件，不难发现也不是Vue的源头，里面有一行这样的代码：

```javascript
import Vue from 'core/index'
```

​	又一次的扑空了，不要着急我们接着找core/index文件好了。

```javascript
import Vue from './instance/index'
```

​	打开文件我们发现又不是源头,而是上面这段代码。又是引入，源码真是把我们的耐心按在地上疯狂的摩擦。既然我们想了解源码，也没有办法了，那就接着找吧，打开./instance/index是下面这样的：

```javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue

```

​	到这个文件里面终于找到了Vue定义的地方， 想哭的有没有。真是源码虐我千百遍，我待源码如初恋(哈哈，皮一下)。言归正传，这个文件中有这个几个方法把Vue这个构造函数传进去了。那我们就点进去看看吧。

​	打开./init文件，找到调用的方法initMixin：

```javascript
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {}
}
```

​	上面没有写_init里面的代码，以后我们都会讲的现在我们只要知道这个方法只是将init方法挂载到Vue.prototype上面就好了。挂载到这个属性上面的方法将来是要直接再被挂载到new的实例上面的。这个方法就是定义Vue构造函数调用的方法。

​	接着我们打开./state，看stateMixin方法做了什么：

```javascript
const dataDef = {}
  dataDef.get = function () { return this._data }
  // props 的 Def
  const propsDef = {}
  propsDef.get = function () { return this._props }
  
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function () {
      warn(
        'Avoid replacing instance root $data. ' +
        'Use nested data properties instead.',
        this
      )
    }
    propsDef.set = function () {
      warn(`$props is readonly.`, this)
    }
  }
  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)

  Vue.prototype.$set = set
  Vue.prototype.$delete = del

  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    const vm: Component = this
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    options = options || {}
    options.user = true
    const watcher = new Watcher(vm, expOrFn, cb, options)
    if (options.immediate) {
      try {
        cb.call(vm, watcher.value)
      } catch (error) {
        handleError(error, vm, `callback for immediate watcher "${watcher.expression}"`)
      }
    }
    return function unwatchFn () {
      watcher.teardown()
    }
  }
```

​	这是stateMixin方法中的全部内容了， 不难看出，这个方法里面又往Vue.prototype上面挂载了几个属性，分别是：$data、$props、$set、$del、$watch。这几个属性看起来还是比较眼熟的把。还是我们现在不讲具体的代码，只说属性有什么，在什么地方。

​	打开./render文件，

```javascript
installRenderHelpers(Vue.prototype)

Vue.prototype.$nextTick = function (fn: Function) {}

Vue.prototype._render = function () {}
```

​	这个文件里面首先调用了一个installRenderHelpers函数，把Vue.prototype属性传进去了，接下来我们先看看这个函数都做了什么事情。

```javascript
export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
  target._d = bindDynamicKeys
  target._p = prependModifier
}
```

看到没有挂载了好多的属性，往prototype属性上面。现在终于知道new一个实例的时候为什么有那么多属性了把，哈哈。

​	老规矩，接下来打开./lifecycle文件，里面的lifecycleMixin函数是这样的：

```javascript
export function lifecycleMixin (Vue: Class<Component>) {
  Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}

  Vue.prototype.$forceUpdate = function () {}

  Vue.prototype.$destroy = function () {}
}
```

​	看到没有，又往Vue.prototype上面挂载了三个属性。话不多说，接着往下走。

​	打开./events文件，里面的代码是这样的：

```
export function eventsMixin (Vue: Class<Component>) {
	const hookRE = /^hook:/
  	Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {
    }

 	 Vue.prototype.$once = function (event: string, fn: Function): Component {}

	Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): 				Component {}

  	Vue.prototype.$emit = function (event: string): Component {}
    
  }
```

又是三个方法挂了上来，分别是:$on、$once、$off、$emit

下面就是instance.js中向Vue构造函数上面挂载的全部的属性了

```javascript
// ./init.js
Vue.prototype._init_ = fn

// ./state.js
Vue.prototype.$data
Vue.prototype.$props
Vue.prototype.$set
Vue.prototype.$del
Vue.prototype.$watch

// ./event.js
Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {}
Vue.prototype.$once = function (event: string, fn: Function): Component {}
Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {}
Vue.prototype.$emit = function (event: string): Component {}

// ./lifecycle.js
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
Vue.prototype.$forceUpdate = function () {}
Vue.prototype.$destroy = function () {}

// ./render.js
Vue.prototype._o = markOnce
Vue.prototype._n = toNumber
Vue.prototype._s = toString
Vue.prototype._l = renderList
Vue.prototype._t = renderSlot
Vue.prototype._q = looseEqual
Vue.prototype._i = looseIndexOf
Vue.prototype._m = renderStatic
Vue.prototype._f = resolveFilter
Vue.prototype._k = checkKeyCodes
Vue.prototype._b = bindObjectProps
Vue.prototype._v = createTextVNode
Vue.prototype._e = createEmptyVNode
Vue.prototype._u = resolveScopedSlots
Vue.prototype._g = bindObjectListeners

Vue.prototype.$nextTick = function (fn: Function) {}
Vue.prototype._render = function (): VNode {}
```

#### 2. 向Vue构造函数上挂全局的方法	

​	既然isntance.js文件查看完毕了， 下面我们接着往会看，源码里面还做了哪些事情。打开core/index.js

```javascript
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'
initGlobalAPI(Vue)

// 是否是服务器渲染
Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

// 服务器渲染
Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

Vue.version = '__VERSION__'
```

​	上面就是这个文件做的所有事情了，首先我们看一下initGlobalAPI函数做了什么事情呢？通过函数的名称，我们不难发现，这个函数就是给Vue这个构造函数添加全局api的，其实事实也是如此。经过查看我们发现这个函数在./global-api/index中。好的，打开这个文件：

```javascript
export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  // 2.6 explicit observable API
  Vue.observable = <T>(obj: T): T => {
    observe(obj)
    return obj
  }

  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue

  extend(Vue.options.components, builtInComponents)

  initUse(Vue)
  initMixin(Vue)
  initExtend(Vue)
  initAssetRegisters(Vue)
}
```

​	看这个函数体，首先这个函数给Vue这个构造函数添加了一个config属性，指向一个config对象，并且这个属性在开发模式下面不允许修改。

​	然后给Vue.util属性添加了四个属性，上面的注释意思是这四个属性，没有在Vue官方的全局api当中，如果你想要使用也是可以的。

​	接着想Vue构造函数中挂载了set、del、nextTick、observable、options。

​	下面调用的四个方法你要是点进去会发现，他们只是给Vue挂载了use、mixin、extend、component、

 directive、filter

​	至此，直接挂到Vue构造函数上面的方法我们总结一下：

```javascript
// 一个只读的config属性
Vue.config
// 在util上挂了四个建议没事别瞎用的属性
Vue.util.warn
Vue.util.extend
Vue.util.mergeOptions
Vue.util.defineReactive
// 还有将Vue剩余的全局方法挂到Vue的构造函数上面
Vue.set
Vue.del
Vue.nextTick
Vue.observable
Vue.optiobs
Vue.use
Vue.mixin
Vue.extend
Vue.component
Vue.directive
Vue.filter
Vue.version
```

​	在new Vue()的时候，执行了this._init()方法， 我们要知道这个方法里面都往实例对象上面挂载了什么属性

```javascript
// 首先将_uid属性挂载到Vue实例上去
vm._uid = uid++
// 然后挂载判断对象是否是Vue实例对象的属性
vm._isVue = true
// 将实例化传进去的options和Vue构造函数options属性合并，挂载到Vue实例上面去
vm.$options = mergeOptions()
// 挂载监听对vm操作的属性_renderProxy
vm._renderProxy = vm/new Proxy(vm)
// 定义一个指向自己的属性_self
vm._self = vm

// 执行initLifeCycle
vm.$parent = parent // 将实例的$parent属性指向
vm.$root = parent ? parent.$root : vm // 指向最顶层的Vue实例

vm.$children = [] // 先定义一个空的$children属性
vm.$refs = {} // 定义一个空的$refs属性

vm._watcher = null
vm._inactive = null
vm._directInactive = false
vm._isMounted = false
vm._isDestroyed = false
vm._isBeingDestroyed = false

/**
* initEvents
**/
vm._events = Object.create(null)
vm._hasHookEvent = false

/**
* initRender
**/
vm._vnode = null
vm._staticTrees = null
```



#### 3.  把剩余的文件看完

​	跟踪Vue的时候， 我们还剩下两个文件。runtime/index 和entry-runtime-with-compiler。其实这个时候Vue有的东西，已经被我们列的七七八八了。还有一些零碎的东西，我们看到了再说。

​	那么好，我们先看runtime/index文件吧:

```javascript

```



​	

