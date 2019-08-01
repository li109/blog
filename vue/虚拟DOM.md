# Vue2.0源码阅读笔记（六）：Virtual DOM
&emsp;&emsp;<br/>
## 一、挂载实例
&emsp;&emsp;*/src/core/instance/init.js*<br/>
```js
vm.$mount(vm.$options.el)
```
&emsp;&emsp;*vm.$mount* 方法在**只包含运行时**版本中是在 */src/platforms/web/runtime/index.js* 文件中添加到Vue原型的。<br/>
```js
Vue.prototype.$mount = function (el,hydrating) {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```
&emsp;&emsp;而在**完整版**中，*vm.$mount* 方法是在 */src/platforms/web/entry-runtime-with-compiler.js* 文件中添加到Vue原型的。<br/>
```js
const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (el,hydrating) {
  /**
   * 省略编译模板的代码...
   * 最终Vue实例vm的$options会添加两个属性：
   * 函数属性 render
   * 数组属性 staticRenderFns
  **/
  return mount.call(this, el, hydrating)
}
```
&emsp;&emsp;可以看到，完整版的 *$mount* 最终也是调用只包含运行时版本定义的 *$mount* 方法，只是在调用前将模板字符串编译渲染函数和静态根节点渲染函数而已。最终的 *$mount* 方法主要功能就是调用 *mountComponent* 方法，该方法定义在 */src/core/instance/lifecycle.js* 文件中。<br/>
```js
function mountComponent (vm,el,hydrating) {
  vm.$el = el
  /* 省略 render 函数不存在情况下的代码 */
  callHook(vm, 'beforeMount')

  let updateComponent
  /* 省略性能统计代码 */
  updateComponent = () => {
    vm._update(vm._render(), hydrating)
  }
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```
&emsp;&emsp;*mountComponent* 在实例挂载前调用生命周期 *beforeMount* 钩子函数数组中的函数，在实例挂载后调用生命周期 *mounted* 钩子函数数组中的函数。该方法核心代码是实例化一个渲染函数观察者对象，渲染函数观察者初次生成时与渲染函数中数据发生改变时均会调用 *updateComponent* 方法。<br/>
```js
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```
&emsp;&emsp;*updateComponent* 方法完成了从渲染函数到生成真实DOM并挂载的过程：*_render* 方法的功能是根据渲染函数生成虚拟DOM；*_update* 方法在首次渲染时将虚拟DOM转化成真实DOM并挂载，在数据更新时对比虚拟DOM的变化来对真实DOM进行更新。<br/>
## 二、生成虚拟DOM
&emsp;&emsp;*_render* 方法根据渲染函数生成虚拟DOM，该方法定义在 *src/core/instance/render.js* 文件中。<br/>
```js
Vue.prototype._render = function (): VNode {
  const vm: Component = this
  const { render, _parentVnode } = vm.$options

  if (_parentVnode) {
    vm.$scopedSlots = normalizeScopedSlots(
    _parentVnode.data.scopedSlots,
    vm.$slots,
    vm.$scopedSlots
    )
  

  // set parent vnode. this allows render functions to have access
  // to the data on the placeholder node.
  vm.$vnode = _parentVnode
  // render self
  let vnode
  try {
    // There's no need to maintain a stack because all render fns are called
    // separately from one another. Nested component's render fns are called
    // when parent component is patched.
    currentRenderingInstance = vm
    vnode = render.call(vm._renderProxy, vm.$createElement)
  } catch (e) {
    handleError(e, vm, `render`)
    // return error render result,
    // or previous vnode to prevent render error causing blank component
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production' && vm.$options.renderError) {
      try {
        vnode = vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e)
      } catch (e) {
        handleError(e, vm, `renderError`)
        vnode = vm._vnode
      }
    } else {
      vnode = vm._vnode
    }
  } finally {
    currentRenderingInstance = null
  }
  // if the returned array contains only a single node, allow it
  if (Array.isArray(vnode) && vnode.length === 1) {
    vnode = vnode[0]
  }
  // return empty vnode in case the render function errored out
  if (!(vnode instanceof VNode)) {
    if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
      warn(
          'Multiple root nodes returned from render function. Render function ' +
          'should return a single root node.',
          vm
      )
    }
    vnode = createEmptyVNode()
  }
  // set parent
  vnode.parent = _parentVnode
  return vnode
}
```
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
## 三、生成真实DOM
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
## 四、diff算法
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
## 五、总结
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>