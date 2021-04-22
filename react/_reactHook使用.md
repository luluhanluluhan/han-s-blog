# reactHook

## 什么是 react Hook?

16.8 的新增特性

## 什么作用？

在函数组件中使用 state
16.8 之前不行

## hook

钩子函数
非 class 的情况下，可以使用更多的 react 特性
完全可选
100%向后兼容
现在可用
没有计划从 react 中移除 class
对 react 概念理解没有影响

## 为什么使用 hook?

-   代码更简洁
-   上手更加简单
    -   react 上手不容易
        1. 生命周期难以理解，很难熟练应用
        2. redux 状态管理，概念多，难以理解，中文文档
        3. 高阶组件理解起来不容易、必须掌握
        4. 优秀的解决方案，都在 react 社区
-   hook 学习成本降低了，
    1. 生命周期可以不学
    2. 高阶组件不用学
    3. redux 也不在是必需品
    4. 开发体验非常好
        1. 可以让函数组件维护内部的状态

## react

    与vue的双向数据流不同的是。react是单向数据流的（自上而下的）
    state(props)组件重新渲染（父组件变化 -> 下边所有的组件都重新渲染）
    class render
    function 整个函数重新执行

### 函数组件

```js
function Component() {
	// 返回的是jsx结构
	return <div></div>;
}
```

hook 就是有状态的函数组件 state useState

### useState

每次渲染，函数都会重新执行，每当函数执行完毕，所有内存都会释放掉（函数式组件没法进行状态管理的原因）。
在函数内部，创建一个当前函数组件的状态，还提供了一个修改状态的方法。
useState(0) ; 返回的是一个数组 => [count, setCount]
useState 可以定义多次

### useEffect

    总会执行一些副作用，函数组件中，纯函数，props，固定的输入总是会得到固定的输出
    什么是副作用？
        没有发生在数据向视图转换过程中的逻辑
        只想渲染一个dom -> dom渲染完了，还想执行一段逻辑（副作用）
        ajax请求。访问原生dom对象，定时器等都是副作用。
        需要清除的副作用·不需要清除的
        hook之前，副作用都是不被允许的
    useEffect操作副作用， componentDidmount 相同大的用途，合成api
    useEffect(fn), 组件渲染到屏幕之后才会执行 返回一个清楚副作用的函数 不返回
    useEffect 不会阻塞浏览器的屏幕更新，组件渲染完成以后了，不需要同步的，
    需要同步的的话，useLayoutEffect()

```js
useEffect(() => {
    //ajax请求
    setTimeout(() =>{
        setCount(count + 1)
    }, 1500)
}, [])
// 会不停的循环加一
原因：
dom在渲染完成，副作用执行useEffect
在副作用执行的过程中，修改了count,state状态。
state改变 -> 引发重新渲染
无限循环
useEffect(fn, arr)
**arr** 浅比较
// 定义了第二个参数，就是不依赖props,state
数组中的变量变了才会执行


let [refresh, setRefresh] = useState()
useEffect(() => {
    //ajax
    setTimeout(() =>{
        setCount(count + 1)
    }, 1500)
    return clear()
}, [refresh])
handle() {
    setRefresh(!refresh)
}
```

    如何清除副作用？
        可以通过返回一个函数（组件卸载前执行），来清除副作用

### useContext

16 更新了 context api
context 用于爷孙组件传值
数据流是自上而下，一级一级的
useContext 使用了 context 的能力
顶层的组件

```js
import { createContext } from 'react';

export const context = createContext({});
// 顶层组件
export function ContextProvider(children) {
	const [count, setCount] = useState(10);
	const consVal = {
		count,
		setCount,
		add: () => setCount(count + 1),
		reduce: () => setCount(count - 1),
	};
	return <context.Provider value={consVal}>{children}</context.Provider>;
}
// 子组件
import { useContext } from 'react';
import { context, ContextProvider } from './ContextProvider';
function SubCount() {
	const { count = 0, add, reduce } = useContext(context);
	return (
		<div>
			<h1>我是subCount组件</h1>
			<p>{count}</p>
			<button onClick={add}>加</button>
			<button onClick={reduce}>减</button>
		</div>
	);
}
export default () => (
	<ContextProvider>
		<SubCount />
	</ContextProvider>
);
```

createContext 可以创建多个
redux

### useReducer

redux 差不多
useState 内部就是靠 useReducer 实现的
useState 的替代方案，（state, action） => newState
useReducer 接受三个参数，state dispatch
useReducer()

```js
useReducer接受三个参数，state dispatch
// 定义一个简单的reducer 第一个参数
import {useReducer} from 'react';
const reducer = (state, action) => {
    switch(action.type) {
        case 'add':
            return (...state, count: state.count + 1)
        case 'reduce':
            return (...state, count: state: count -1)
            defalut:
            return state;
    }
// 第二个参数 createStore()指定默认值
let initialState = {count: 10, name: 'reducer'}
// 第三个参数 把第二个参数当做参数执行
const init = initialCount => {
    return { count: initialCount.count + 2 };
}
export default function ReducerComponent() {
    const [state, dispatch] = useReducer(reducer, initialState, init)
    return <div>
        <button onClick={()=> dispatch({type: 'add'}))>加</button>
        <button onClick={()=> dispatch({type: 'reduce'}))>减</button>
    </div>
}
```

Store， Reducer ,Action redux 三大核心

### useRef

    访问dom节点
    16 Object.createRef 创建ref方法
    {
        current: ''
    }
    让input框直接获取一个焦点

```js
import {useRef} from 'react';
const refInput = useRef(null);
useEffect(() => {
    refInput.current.focus();
},[])
<input type="text" id="name" ref={refInput}>
```

### useMemo&useCallback 闭包 不能盲目使用，会占用内存

useMemo 把创建函数和依赖项数组作为参数传入 useMemo
useCallback 接受一个内联回调函数和一个依赖项数组

```js
    const memorized = useCallback(() => {
        return count
    }, [num])
    const memorized2 = useMemo(() => {
        return count
    }, [num])
    console.log(memorized(), memorized2());
    console.log('原始'， count)

```

### 自定义 hook 抽离公共代码

    逻辑功能相同的片段->封装成单独的函数来使用
    自定义hook，函数
    自定义hook中可以调用官方的hook
    use开头，表示只能在函数组件中使用
        use开头
        复用状态逻辑的方式，而不是复用state本身
        事实上hook每次调用的时候都有一个独立的state

```js
import { useState, useEffect } from 'react';
export function useNumber(params) {
	let [number, setNumber] = useState(2);
	useEffect(() => {
		setTimeout(() => {
			setNumber(number => number * 2);
		}, 1000);
	}, []);
	return [number, setNumber];
}

const [number, setNumber] = useNumber();
console.log(number);
```

## hook 使用规则

1. 只能在最顶层使用 hook
   不要在循环，嵌套函数中调用
2. 只在 react 函数中调用
