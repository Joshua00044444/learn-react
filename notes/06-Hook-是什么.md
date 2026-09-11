# Hook 是什么:函数组件「挂接」React 特性的入口

> 📌 本文的 React 技术知识,来自 D 盘的 **DeepSeekMonitorWindows-1** 项目。

其实你已经用过 Hook 了——上一篇的 `useState` 就是其中之一。**Hook 就是一批以 `use` 开头的特殊函数**,让函数组件能"挂接"(hook into) React 的内部能力:记状态、渲染后做事、缓存值……都从它们这儿来。

## 定义与名字由来

官方说法:Hook 是使用 React 各项特性的**入口函数**。名字取"钩子"之意——你的组件函数是普通 JS 函数,本身没有记忆、没有生命周期;调用 Hook 相当于把一根"钩子"**钩进 React 内部**,租用它替你保管的东西:

```jsx
import { useState, useEffect } from 'react';

function Demo() {
  const [count, setCount] = useState(0);   // 钩住「状态存储」
  useEffect(() => {                        // 钩住「渲染完成后」这个时机
    console.log('渲染完成');
  });
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

对照 03 篇的说法:组件是普通函数,Hook 是 React 预先备好的一组**自带内部记账的函数**——你在函数里调用它,它去 React 的账本上存取属于"这个组件这个位置"的数据。

## 常用 Hook 一览

| Hook | 挂接的能力 | 一句话 |
|---|---|---|
| `useState` | 状态存储 | 让组件「记住」信息(✅ 05 篇已学) |
| `useEffect` | 渲染后的副作用 | 渲染完之后做点事:写日志、改标题、发请求(demo 里已在用) |
| `useRef` | 跨渲染的盒子 | `{ current: ... }` 存个不触发重渲染的值/DOM 引用 |
| `useContext` | 跨层级读数据 | 不用一层层传 props |
| `useMemo` / `useCallback` | 缓存 | 记住计算结果/函数,避免不必要的重算 |
| `useReducer` | 状态存储(进阶) | state 复杂时,用"发动作"的方式更新 |
| 自定义 Hook(`useXxx`) | 逻辑复用 | 把上面这些组合起来,抽成可复用的函数 |

全部清单见官方文档:<https://zh-hans.react.dev/reference/react/hooks>

## 为什么会有 Hook(一句话历史)

React 16.8 之前,只有 class 组件能有 state 和生命周期,逻辑复用要靠高阶组件、render props 等层层嵌套的写法。Hook 出现后,**函数组件也能有 state**,而且逻辑可以抽成自定义 Hook 平铺直叙地复用——新代码基本都写函数组件 + Hook。

## 两条铁律(Hooks 规则)

1. **只在最顶层调用**:不要在循环、条件或嵌套函数里调用 Hook。
2. **只在 React 函数组件或自定义 Hook 里调用**:不要在普通 JS 函数、类组件里调用。

第 1 条的原因和 05 篇的「快照」一脉相承:React **靠调用顺序**把每个 `useState` 和它内部存的值对应起来——第一次渲染数到第 1 个 Hook 就存第 1 格,第 2 个存第 2 格;下次渲染必须按同样顺序再数一遍,账才对得上:

```jsx
// ❌ 条件里调 Hook:条件一旦不成立,后面的 Hook 全部错位一格
if (name !== '') {
  const [age, setAge] = useState(18);
}
const [count, setCount] = useState(0); // name 为空时,这行会被 React 当成「第 1 个 Hook」

// ✅ 永远顶层调用,顺序稳定;需要条件控制,放进 Hook 返回的 setter 或渲染逻辑里
const [age, setAge] = useState(18);
const [count, setCount] = useState(0);
```

配 `eslint-plugin-react-hooks` 可以在写错时直接报警。

## 自定义 Hook:以 use 开头的复用函数

把"多个 Hook 的组合逻辑"抽成一个以 `use` 开头的普通函数,就是自定义 Hook:

```jsx
function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  const toggle = () => setOn(!on);
  return [on, toggle];
}

function Panel() {
  const [open, toggle] = useToggle();   // 一个 Hook 顶一套 state + 逻辑
  return <button onClick={toggle}>{open ? '收起' : '展开'}</button>;
}
```

`use` 开头不是语法强制,而是约定——让人和 lint 都一眼认出"这函数里有 Hook,必须按规则调用"。

## 记忆口诀

> **Hook 是以 `use` 开头的租借点:state、副作用、缓存……都从 React 内部租。**
> 两条规则:**只在顶层调,只在组件(或自定义 Hook)里调**——顺序错一格,账本全错位。
