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

export default Demo;   // 页面里渲染 <Demo />:点一次按钮数字 +1,同时控制台打一行「渲染完成」
```

对照 03 篇的说法:组件是普通函数,Hook 是 React 预先备好的一组**自带内部记账的函数**——你在函数里调用它,它去 React 的账本上存取属于"这个组件这个位置"的数据。

## 常用 Hook 一览(按官方参考分类)

官方参考(<https://zh-hans.react.dev/reference/react>)把 Hook 按「挂接的能力」分成七类。日常 90% 的场景只用加粗的几个:

| 分类 | 包含的 Hook | 一句话 |
|---|---|---|
| State Hook | **`useState`**(✅ 05 篇)、`useReducer` | 让组件「记住」信息 |
| Context Hook | **`useContext`** | 跨层级读数据,不用一层层传 props |
| Ref Hook | **`useRef`**、`useImperativeHandle`(少用) | 跨渲染的「盒子」,存不触发重渲染的值 / DOM 引用 |
| Effect Hook | **`useEffect`**(变体:`useLayoutEffect`、`useInsertionEffect`) | 渲染之后与外部系统同步 |
| 性能 Hook | **`useMemo`**、**`useCallback`**、`useTransition`、`useDeferredValue` | 跳过不必要的计算 / 重渲染 |
| 其他 Hook | `useId`、`useSyncExternalStore`、`useActionState` 等 | 多为库作者准备 |
| 自定义 Hook | `useXxx` | 把上面这些组合起来复用(见下文) |

### State Hook:useState / useReducer

`useState` 已在 05 篇细学(快照、批处理、函数式更新)。当更新逻辑变复杂——多个值互相关联、同一种状态有很多种改法——用 `useReducer` 把「怎么改」集中到一个纯函数里,组件只负责「发动作」:

```jsx
import { useReducer } from 'react';

function reducer(state, action) {          // 「怎么改」全在这一个纯函数里
  switch (action.type) {
    case 'add': return { count: state.count + 1 };
    case 'sub': return { count: state.count - 1 };
    default:    return state;
  }
}

export default function Counter() {        // 组件只负责「发动作」
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  return (
    <button onClick={() => dispatch({ type: 'add' })}>
      点了 {state.count} 次
    </button>
  );
}
```

它和 05 篇的函数式更新 `setCount(c => c + 1)` 一脉相承:更新逻辑与事件处理器脱钩。操作种类多、state 是多字段对象时,`useReducer` 比一堆 setter 清晰。

**辨析:`{ type: 'add' }` 与 `{state.count}` 这两个花括号是两码事。**

`{ type: 'add' }` 是普通的 JS 对象字面量,术语叫 **action(动作)**。dispatch 把它原封不动交给 reducer 的第二个参数,正是 `switch (action.type)` 里读的那个对象。action 还可以携带数据:

```jsx
dispatch({ type: 'addBy', by: 5 });                        // 组件:宣布「加 5」这个动作
case 'addBy': return { count: state.count + action.by };   // reducer:从 action.by 读出数据
```

分工由此而来:组件只负责**宣布发生了什么**,**怎么改完全由 reducer 说了算**。`type` 的值随便起('add'、'increment' 都行),只要 dispatch 和 reducer 两边对得上。

`{state.count}` 则是 JSX 的嵌表达式语法(01 篇):花括号里放 JS 表达式,渲染时求值填进界面。`state` 是 `{ count: 0 }` 这个**对象**,所以要 `.count` 取出数字;不能直接写 `{state}`——React 不允许把对象渲染成文字,会报错 "Objects are not valid as a React child"。

> 一句话:**`{ type: 'add' }` 是递给 dispatch 的「工单」,`{state.count}` 是嵌进 JSX 的「显示值」——前者发出去,后者读出来。**

### Context Hook:useContext

解决 props 逐层透传(prop drilling):数据由祖先「提供」,任意深度的后代直接「读取」,中间层完全不用伸手。07 篇的状态提升要求「从公共父组件一层层传 props」;当层级太深、传递只是「路过」时,context 是官方给的第二条路。

**三步各自在做什么**:① 开一条「频道」,并给兜底默认值——头顶上没有 Provider 时读 `'light'`,不报错;② Provider 决定**广播什么**(`value`)、**广播给哪棵子树**(包住谁);③ 后代调频收听。①必须写在组件外(顶层),因为 `useContext` 靠**对象身份**对频道——传入的必须和 Provider 用的是同一个对象;写在组件内部的话,每次渲染都会造一条新频道,广播站和收音机就对不上号,惯例是单独一个文件再 export。

写全的示例(数据源是祖先的 state,①②③就是代码里的三处注释):

```jsx
import { createContext, useContext, useState } from 'react';

// ① 模块顶层(所有组件外面):开一条「频道」,'light' 是兜底默认值
const ThemeContext = createContext('light');

// ② 祖先组件:数据源是自己的 state;Provider 包住谁,谁就进入广播范围
function App() {
  const [theme, setTheme] = useState('light');       // ← value={theme} 的 theme 来自这
  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={() => setTheme('dark')}>切换</button>
      <Middle />                                     {/* 中间层一个 props 都不用传 */}
    </ThemeContext.Provider>
  );
}

// 中间层:透明管道,什么都不用接
function Middle() {
  return <DeepChild />;
}

// ③ 任意后代组件的函数体里:调频收听,拿到最近的 Provider 的 value
function DeepChild() {
  const theme = useContext(ThemeContext);            // 变量名随便起,叫 t 也合法
  return <p>当前主题:{theme}</p>;
}

export default App;   // 点「切换」→ DeepChild 直接换主题,中间层全程没碰 props
```

两个易混点:

- **`value={theme}`**:`value` 是 Provider 规定的属性名,必须叫 `value`;`{theme}` 是 JSX 嵌表达式,值来自祖先的 state。②里广播的 `theme` 和③里 `const theme = useContext(...)` 收到的 `theme` 是**两个毫不相干的变量**,只是恰好同名;③的名字随便起,叫 `t` 也合法。
- **中间层为什么不用传 props**:`Provider` 包住谁,谁就在广播范围里,夹多少层都无所谓——上面代码里的 `Middle` 是透明管道,demo 里对应 `<ThemeMiddle />`。凡是在 Provider **里面**的组件都能直接 `useContext` 收听,一层 props 都不用传。

> 一句话:**①造话筒、②插电广播、③谁要听谁收**——没有①就没有可共享的对象,没有②数据没有来源,没有③数据没人收。

### Ref Hook:useRef

**`useRef(初始值)` 返回一个「盒子」`{ current: 初始值 }`,React 保证它在组件整个生命周期里始终是同一个对象**——组件每次重渲染、函数从头执行、局部变量全部重来,盒子里的东西还在;而往盒子里放东西(`ref.current = 新值`)**不会通知 React**,界面不会因此重渲染。「既能跨渲染记住东西、又不惊动 React」,这就是它存在的理由。

**用途①:挂 DOM 引用,命令式操作**(按执行时机逐行走):

```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);    // ① 渲染时:造一个空盒子,先放 null 占位
                                    //   (此刻页面上还没有输入框,没东西可引用)

  function handleClick() {
    inputRef.current.focus();       // ④ 点击时:从盒子里取出 DOM,亲自让它聚焦
  }

  return (
    <>
      {/* ② 渲染时:React 看到 ref 属性,自动把这个 input 的
          真实 DOM 节点塞进盒子 → inputRef.current = input 的 DOM 对象 */}
      <input ref={inputRef} />
      {/* ③ 等待被点击 */}
      <button onClick={handleClick}>聚焦输入框</button>
    </>
  );
}
```

逻辑链:**页面渲染 → React 把 input 的 DOM 塞进盒子(②)→ 用户点按钮 → `handleClick` 执行 → 从盒子里取出 DOM 调 `.focus()`(④)→ 光标跳进输入框**。全程没有任何 setState——因为界面本来就不需要变,变的只是「浏览器焦点在哪」。

两个关键词:

- **`current`**:盒子的开口,塞进去、取出来的都走它。`useRef(null)` 的 `null` 只是出厂状态——第②步 React 会替你换成真实的 DOM。
- **「命令式操作 DOM」**:React 的常规套路是**声明式**——你只描述「界面长什么样」,改 DOM 的活 React 干;而 `.focus()` 是你亲自下场直接指挥 DOM(等价于原生 JS 的 `document.querySelector('input').focus()`)。聚焦、滚动、播放/暂停视频、量尺寸这类事 React 管不了,只能命令式,官方叫「脱围机制」(escape hatch)。

**用途②:存「不需要被看见」的值**(节选:`tick` 是别处定义的回调;完整效果见 react-hooks-demo 卡片 B):

```jsx
const timerRef = useRef(null);               // 存「不需要被看见」的值
timerRef.current = setInterval(tick, 1000);  // 改 .current 不触发重渲染!
```

`setInterval` 每次执行会返回一个**定时器编号**,想停必须 `clearInterval(编号)`,所以编号要记下来。放哪?普通变量——每次渲染被重置,编号丢失,停不掉定时器;state——编号界面又不显示,存它还引发无谓重渲染;ref 盒子——跨渲染记得住,又不惊动 React ✅。

与 useState 的分工:**界面要显示的用 state,界面不显示但要记住的用 ref**(定时器 id、上一次的值、DOM 节点)。注意:改 `ref.current` 不触发重渲染,渲染进行中也不要读写它。

### Effect Hook:useEffect

渲染提交到屏幕**之后**执行,用来和「组件之外的系统」同步:定时器、订阅、网络请求、直接改 DOM(react-usestate-demo 的日志面板就是它写的)。

```jsx
import { useState, useEffect } from 'react';

export default function TitleCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `点了 ${count} 次`;   // 副作用:改浏览器标签页标题
    return () => { /* 清理:下次执行前 / 卸载时调用 */ };
  }, [count]);                             // 依赖数组:count 变了才重新执行

  return <button onClick={() => setCount(count + 1)}>点我({count})</button>;
}
```

依赖数组三种写法:**不写** = 每次渲染后都执行;**`[]`** = 只在挂载后执行一次;**`[count]`** = count 变了才执行(且先执行上一次的清理函数)。官方提醒:能由 props/state 直接算出来的东西,不要用 effect。

### 性能 Hook:useMemo / useCallback

缓存,跳过不必要的重算与重渲染:

```jsx
import { useMemo, useState } from 'react';

function heavySum(n) {                     // 假装很贵的计算:1+2+…+n
  let s = 0;
  for (let i = 1; i <= n; i++) s += i;
  return s;
}

export default function Calc() {
  const [n, setN] = useState(100);
  const total = useMemo(() => heavySum(n), [n]);  // n 不变 → 直接用上次结果,不重算

  return (
    <>
      <div>1 加到 {n} = {total}</div>
      <button onClick={() => setN(n + 1)}>n + 1</button>
    </>
  );
}
```

`useCallback`(节选:写在父组件里,`id` 来自父组件的 state;常用于把回调传给子组件时,避免子组件无谓重渲染):

```jsx
const onSave = useCallback(() => save(id), [id]);  // id 不变 → 返回同一个函数
```

`useMemo` 缓存「计算结果」;`useCallback` 缓存「函数本身」(相当于 `useMemo(() => fn, deps)`)。另两个性能 Hook:`useTransition` 把更新标记为「可打断、不阻塞输入」,`useDeferredValue` 让某个值的更新「慢半拍」——入门先混个脸熟。

### 其他 Hook(先认识)

`useId`(生成稳定 id)、`useSyncExternalStore`(订阅外部 store)、`useDebugValue`(调试)等多为库作者准备;React 19 又新增了 `useActionState`、`use` 等。用到时再查[官方参考](https://zh-hans.react.dev/reference/react)。

## 为什么会有 Hook(一句话历史)

React 16.8 之前,只有 class 组件能有 state 和生命周期,逻辑复用要靠高阶组件、render props 等层层嵌套的写法。Hook 出现后,**函数组件也能有 state**,而且逻辑可以抽成自定义 Hook 平铺直叙地复用——新代码基本都写函数组件 + Hook。

## 两条铁律(Hooks 规则)

1. **只在最顶层调用**:不要在循环、条件或嵌套函数里调用 Hook。
2. **只在 React 函数组件或自定义 Hook 里调用**:不要在普通 JS 函数、类组件里调用。

第 1 条的原因和 05 篇的「快照」一脉相承:React **靠调用顺序**把每个 `useState` 和它内部存的值对应起来——第一次渲染数到第 1 个 Hook 就存第 1 格,第 2 个存第 2 格;下次渲染必须按同样顺序再数一遍,账才对得上:

```jsx
// 说明片段(不可直接运行):假设 name 是组件里已有的一个 state,这里只演示 Hook 的调用位置

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
import { useState } from 'react';

function useToggle(initial = false) {    // 自定义 Hook:复用「开/关」逻辑
  const [on, setOn] = useState(initial);
  const toggle = () => setOn(!on);
  return [on, toggle];
}

export default function Panel() {        // 普通组件,像用官方 Hook 一样用它
  const [open, toggle] = useToggle();    // 一个 Hook 顶一套 state + 逻辑
  return <button onClick={toggle}>{open ? '收起' : '展开'}</button>;
}
```

`use` 开头不是语法强制,而是约定——让人和 lint 都一眼认出"这函数里有 Hook,必须按规则调用"。

## 实测验证(2026-09-11,react-val-study/react-hooks-demo.html)

五张卡片对照,带时间戳日志实测:

- **🎛️ State Hook**:`useState` 计数器与 `useReducer` 计数器并排,`dispatch({type:'add'})` 和 `setCount` 效果一致——「发动作」只是把更新逻辑挪进了 reducer 纯函数。
- **📦 Ref Hook**:`inputRef.current.focus()` 命令式聚焦输入框;`hiddenRef.current` 连加点到几十,界面纹丝不动,直到 `setShown` 触发重渲染才「现形」——改 ref 不触发重渲染,值却一直都在。
- **⏱️ Effect Hook**:挂载时 `[]` 的 effect 只跑一次;count 变化时「先执行上一次的清理、再执行本次」;🔄 重新挂载(key 强制)能看到完整的「卸载清理 → 重新挂载」过程。
- **🌳 Context Hook**:父组件切主题,孙子组件用 `useContext` 直接读到新值,中间层一个 props 都没传。
- **⚡ 性能 Hook**:无关 state 触发重渲染,普通计算每次都重算(累计次数上涨),`useMemo([n])` 无动于衷;n 翻倍(依赖变了)才真算——缓存生效的直接证据。

## 记忆口诀

> **Hook 是以 `use` 开头的租借点:state、副作用、缓存……都从 React 内部租。**
> 两条规则:**只在顶层调,只在组件(或自定义 Hook)里调**——顺序错一格,账本全错位。
> 常用五件套:**useState 记、useRef 藏、useEffect 事后跑、useContext 空降、useMemo/useCallback 省重算。**
