# react17 新 API + 原理深入

## react 15 的架构

### reconciler 找出更新的组件

负责找到有变化的组件

### renderer

把有变化的组件渲染到页面上

### react 15 架构是递归的，一个长任务，会导致阻塞用户后续交互，会卡顿

### react 16 fiber 架构做法

1. 执行异步的调度任务会在宏任务中执行，这样可以保证，不会让用户失去响应
2. react 16 对所的更新都做了一个优先级的绑定，当出现多个需要同时处理的任务的时候，可以中断低优先级更新，执行高优先级更新，这就是 scheduler 模块，来调度任务的优先级
   链表式的：方便打断和复原
   比如说：
