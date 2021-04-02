[When to break up a component into multiple components](https://kentcdodds.com/blog/when-to-break-up-a-component-into-multiple-components)

## React 组件拆分最佳实现

一个臃肿的 react 组件会带来哪些问题：

- 性能问题：每次 state 变成都会导致整个应用的重渲染。
- 代码复用性变差
- 状态变更将变得不可控，容易导致 bug
- 功能测试将变得复杂，很难将单独的功能，隔离出来做测试
- 合作开发会变得困难，很容易改出冲突
- 使用第三库与我们的目标不符，而且即使使用，也无法使用高阶组件
- 无法单独声明 api，api 在生命周期中到处都是

https://kentcdodds.com/blog/write-tests

https://kentcdodds.com/blog/dont-sync-state-derive-it

https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster

https://kentcdodds.com/blog/optimize-react-re-renders

一个小项目没有那么多重复的代码，比如设计团队才华横溢根本不认可组件化等等

React

## 组件拆分法则

为什么需要组件拆分，组件拆分给我们带来好处？

在回答这个问题之前，我们先来看看，如果不拆分，把整个页面写在一个 Component 里面会遇到哪些问题？

- 性能问题：每次 state 的变更，将导致整个 App 的重新渲染。
- 代码复兴用变差


状态提升，state 就近原则
