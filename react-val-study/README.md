# 验证 Demo 与笔记对照表

这个文件夹里放的全是 **React 验证 demo**(浏览器里现场编译 JSX,无需构建工具)。
每个 demo 都对应 `../notes/` 下的一篇笔记:先看笔记懂概念,再打开 demo 亲手点一遍。

| 验证页面 | 对应笔记 | 验证什么 |
|---|---|---|
| [react-onclick-demo.html](./react-onclick-demo.html) | [04-onClick-传函数-还是-传执行结果.md](../notes/04-onClick-传函数-还是-传执行结果.md) | `onClick={fn}` 传函数本身,不是调用结果;写 `fn()` 为什么会出事 |
| [react-usestate-demo.html](./react-usestate-demo.html) | [05-useState-状态与更新界面.md](../notes/05-useState-状态与更新界面.md) | state 存在 React 内部、`setXxx` 触发重渲染;state 跟着组件实例走 |
| [react-hooks-demo.html](./react-hooks-demo.html) | [06-Hook-是什么.md](../notes/06-Hook-是什么.md) | `useState` / `useRef` / `useEffect` / `useContext` / 性能类 Hook 的现场对照 |
| [react-sharing-state-demo.html](./react-sharing-state-demo.html) | [07-组件间共享数据.md](../notes/07-组件间共享数据.md) | 状态提升:两个组件共享同一份数据;复制 props 为什么必然失步 |
| [react-built-in-components-demo.html](./react-built-in-components-demo.html) | [08-内置组件.md](../notes/08-内置组件.md) | `<Fragment>` 不留 DOM 节点 / `<Profiler>` 计时 / `<StrictMode>` 双跑 / `<Suspense>` 顶班 / `<Activity>` 隐藏不销毁(React 19.2,esm.sh 引入) |

没有对应 demo 的笔记(纯概念,没什么可点的):

- [01-JSX-是什么.md](../notes/01-JSX-是什么.md)
- [02-JSX-与-JS-的区别.md](../notes/02-JSX-与-JS-的区别.md)
- [03-React-的本质.md](../notes/03-React-的本质.md)

## 怎么打开

直接**双击任意 HTML 文件**用浏览器打开即可(页面从 CDN 加载 React,需要联网)。
卡片区 + 时间戳日志:先读卡片顶部的代码和说明,再点按钮,对照下方日志看结论。
