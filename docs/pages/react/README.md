React 组件化思考

## 组件拆分法则

为什么需要组件拆分，组件拆分给我们带来好处？

在回答这个问题之前，我们先来看看，如果不拆分，把整个页面写在一个 Component 里面会遇到哪些问题？

- 性能问题：每次 state 的变更，将导致整个 App 的重新渲染。
- 可读性变差
- 复用性问题
- 测试
- 合作开发：代码冲突问题
- 第三方组件使用问题
- Api 或者 公共方法 抽离

## 错误的抽象 vs 代码拷贝

错误抽象导致的问题，原来的抽象越来越复杂，最后其实我们都不需要这个抽象了，后面的开发者也不敢破坏这个抽象

https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction

## 派生状态 

https://kentcdodds.com/blog/dont-sync-state-derive-it

- 状态维护在父组件
- 派生状态
- useReducer
- mobx 计算值
- useComputedValue fn 仅在计算时使用

## 致命的 useCallback/useMemo

https://bytedance.feishu.cn/docs/doccnKcSsW0lazRObCmw3GlGkmd

## useState

https://overreacted.io/zh-hans/making-setinterval-declarative-with-react-hooks/

## 为什么调用顺序对react重要

https://overreacted.io/zh-hans/why-do-hooks-rely-on-call-order/
