# 前端

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.23-front-web-qa&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2023-02-22

## 1、vue和react的生命周期差异

Vue 和 React 是两个非常流行的前端框架，它们都有各自的生命周期方法，用于在组件的不同阶段执行特定的代码。

Vue 生命周期分为八个阶段，依次为：

1. beforeCreate：组件实例被创建之初，数据观测和事件配置之前被调用。
2. created：组件实例已经完全创建，数据观测和事件配置完成之后被调用。
3. beforeMount：组件被挂载到 DOM 之前被调用。
4. mounted：组件被挂载到 DOM 后被调用。
5. beforeUpdate：组件更新之前被调用，发生在虚拟 DOM 重新渲染和打补丁之前。
6. updated：组件更新之后被调用，发生在虚拟 DOM 重新渲染和打补丁之后。
7. beforeDestroy：组件销毁之前被调用。
8. destroyed：组件销毁之后被调用。

React 生命周期分为十个阶段，依次为：

1. constructor：组件实例化时被调用。
2. static getDerivedStateFromProps：组件接收到新的 props 时被调用，用于更新 state。
3. render：组件渲染时被调用。
4. componentDidMount：组件被挂载到 DOM 后被调用。
5. shouldComponentUpdate：组件更新前被调用，用于判断是否需要重新渲染组件。
6. getSnapshotBeforeUpdate：组件更新前被调用，用于获取更新前的快照。
7. componentDidUpdate：组件更新后被调用。
8. componentWillUnmount：组件被销毁前被调用。
9. componentDidCatch：组件渲染时发生错误时被调用。
10. static getDerivedStateFromError：组件渲染时发生错误时被调用，用于更新 state。

> 需要注意的是，React 16 版本以后，推荐使用 static getDerivedStateFromProps 和 getSnapshotBeforeUpdate 替代 componentWillReceiveProps、componentWillUpdate 和componentWillMount。此外，React 17 版本开始废弃 componentWillReceiveProps、componentWillUpdate 和componentWillMount 这三个生命周期方法。


## 2、vue和react的通信机制差异

Vue和React都是流行的前端框架，它们都提供了一些机制来支持组件间的通信。下面是它们的通信机制：

**Vue：**

1、Props和Events：父组件通过props向子组件传递数据，子组件通过emit触发事件来向父组件传递数据。

2、$emit和$on：通过$emit方法触发一个自定义事件，并通过$on方法监听该事件，实现组件间通信。

3、$refs：可以使用$refs属性获取子组件的引用，然后通过这个引用调用子组件的方法或者访问子组件的属性。

4、Vuex：Vue提供了一个状态管理库Vuex，通过它可以在组件之间共享状态。

**React：**

1、Props：父组件通过props向子组件传递数据，子组件通过回调函数将数据返回给父组件。

2、Context：React提供了Context API来实现跨组件的数据传递，可以通过创建一个Context对象，在需要传递数据的组件中使用Provider提供数据，然后在需要访问数据的组件中使用Consumer消费数据。

3、Refs：可以使用Refs属性获取子组件的引用，然后通过这个引用调用子组件的方法或者访问子组件的属性。

4、Redux：Redux是React的状态管理库，通过它可以在组件之间共享状态。

总的来说，Vue和React的通信机制都是基于props和事件的，同时也提供了一些额外的机制来实现更复杂的通信需求。其中，Vue使用$emit/$on和Vuex来实现组件间通信，而React使用Context API和Redux来实现组件间通信。