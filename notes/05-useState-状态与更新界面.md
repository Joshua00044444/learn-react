# useState:组件如何「记住」信息并更新界面

> 📌 本文的 React 技术知识,来自 D 盘的 **DeepSeekMonitorWindows-1** 项目。

这是 React 官方文档快速入门(zh-hans.react.dev/learn)「更新界面」一节的核心:组件要"记住"一些信息(比如按钮被点了几次),就需要 state。

## 官方示例:计数器

```jsx
import { useState } from 'react';

function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      点了 {count} 次
    </button>
  );
}
```

`useState(0)` 返回一个数组,解构出两个东西:

- `count` —— **当前值**(这次渲染里的快照)
- `setCount` —— **更新函数**:调用它会「预约」一次带着新值的重渲染

`useState(0)` 里的 `0` 是初始值,只在组件**第一次**渲染时被采用。

## 为什么普通变量不行?

最容易的疑问:为什么不能 `let count = 0`,点击时 `count += 1`?

两个原因:

1. **组件函数每次渲染都会重新执行**。每次渲染,`let count = 0` 都从头跑一遍,上次改的值直接被重置——普通变量根本没有"跨渲染的记忆"。
2. **改普通变量不会通知 React**。React 的重渲染由 `setState` 等"预约"触发;直接改一个变量,React 毫不知情,界面纹丝不动。

state 的值存在 **React 内部**(组件对应的内部存储),所以重渲染时它能"记得"上一次的值。

## 关键机制一:state 是「快照」

```jsx
function handleClick() {
  setCount(count + 1);
  alert(count); // 弹的是旧值!
}
```

一次渲染期间,`count` 的值是固定的(渲染的"快照")。`setCount` 只做一件事:**预约下一次渲染**。它不会立刻改掉本次渲染里的 `count`,所以紧接着 `alert(count)` 看到的还是旧值。

新值要等下一次渲染才"生效",而且那时函数会重新执行、拿到新的快照。

## 关键机制二:一次事件里的多次 setState 会合并

```jsx
// ❌ 想加 3 次,结果只 +1
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

同一个事件处理器里的三次调用,**都基于同一个旧快照** `count`,React 合并处理后相当于只执行了一次 `setCount(旧值 + 1)`。

想基于"最新值"连续累加,用**函数式更新**:

```jsx
// ✅ +3:更新函数被排队,依次执行,每次拿到的是队列里的最新值
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

## state 是组件实例私有的

```jsx
function App() {
  return (
    <div>
      <MyButton />
      <MyButton />   {/* 两个按钮,各自独立的 count */}
    </div>
  );
}
```

同一个组件渲染出多份,每一份都有自己独立的 state——点第一个按钮,第二个的数字不会动。

## 实测验证(2026-09-10,react-val-study/react-usestate-demo.html)

做了四张卡片对照,带时间戳日志实测:

- **📖 官方示例**:两个 `MyButton` 各自计数,互不影响。
- **📸 快照**:`setCount` 后立刻 `alert(count)`,弹出的是旧值;日志显示「已预约,但本次快照仍为旧值」。
- **⚡ 批处理**:一次点击里连写三次 `setCount(count+1)` 只 +1;三次 `setCount(c=>c+1)` 才是 +3。
- **🔬 对照**:`clicks += 1` 改普通变量,界面无反应;再点 setN 触发重渲染,`clicks` 被重置回 0——两次证据都指向"必须用 state"。

## 记忆口诀

> **state 是「存进 React 的记忆」,普通变量是「每次渲染都失忆」。**
> `setCount` 是预约,不是立刻改;要连加,用 `c => c + 1`。
