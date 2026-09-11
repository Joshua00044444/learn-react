# learn-react

React 学习项目。目前包含一组入门核心概念的笔记和一个可交互的验证页面。
笔记中的 React 技术知识,来自 D 盘的 **DeepSeekMonitorWindows-1** 项目。

**官方教程(对照学习):** <https://zh-hans.react.dev/learn> —— React 中文文档「快速入门」,`react-val-study/` 里的验证页面即针对该文档的「更新界面」等小节编写。

## 目录结构

```
learn-react/
├── README.md                          ← 本文件(学习索引)
├── react-val-study/                   ← 所有可交互验证页面(双击打开,需联网加载 CDN)
│   ├── react-onclick-demo.html        ← 验证:onClick 传函数 vs 传执行结果
│   └── react-usestate-demo.html       ← 验证:useState 与「更新界面」
└── notes/
    ├── 01-JSX-是什么.md
    ├── 02-JSX-与-JS-的区别.md
    ├── 03-React-的本质.md
    ├── 04-onClick-传函数-还是-传执行结果.md
    ├── 05-useState-状态与更新界面.md
    └── 06-Hook-是什么.md
```

## 学习路径

按顺序读笔记,读完用 demo 页面动手验证:

1. **[JSX 是什么](notes/01-JSX-是什么.md)** — JSX 是 JS 的语法扩展,编译后是 `createElement` 调用;基础语法规则和常见注意点。
2. **[JSX 与 JS 的区别](notes/02-JSX-与-JS-的区别.md)** — JS 是语言,JSX 是扩展;浏览器不能直接跑 JSX,必须先编译。
3. **[React 的本质](notes/03-React-的本质.md)** — React 100% 是 JS,它约定的是"如何用 JS 构建 UI"的规则(声明式、单向数据流、状态驱动)。
4. **[onClick:传函数还是传执行结果](notes/04-onClick-传函数-还是-传执行结果.md)** — `{handleClick}` vs `{handleClick()}`,含实测数据。
5. **[useState:状态与更新界面](notes/05-useState-状态与更新界面.md)** — state 快照、批处理、函数式更新,含实测数据。
6. **[Hook 是什么](notes/06-Hook-是什么.md)** — 以 `use` 开头的特殊函数,函数组件挂接 React 特性的入口;两条调用铁律与自定义 Hook。

## 验证页面使用方法

`react-onclick-demo.html` 里有三张卡片,对应三种写法:

| 卡片 | 写法 | 行为 |
|------|------|------|
| ✅ 正确 | `onClick={handleClick}` | 渲染时只登记,点击才有反应 |
| ❌ 错误 | `onClick={handleClick()}` | 渲染时立刻弹 alert,点击无效 |
| 🧩 带参数 | `onClick={() => say('hi')}` | 箭头函数包一层,点击时带参执行 |

点每张卡片的 **🔄 重新挂载** 反复观察"渲染瞬间",日志面板会记录每次函数执行的时间和触发者。

`react-usestate-demo.html` 里有四张卡片,对应「更新界面」一节的四个问题:

| 卡片 | 问题 | 结论 |
|------|------|------|
| 📖 官方示例 | 两个 MyButton 共享计数吗? | 不共享,state 是组件实例私有的 |
| 📸 快照 | setCount 后立刻 alert(count) 是什么值? | 旧值,setCount 只是预约下次渲染 |
| ⚡ 批处理 | 一次事件连写 3 次 setState 加几次? | `setCount(count+1)`×3 只 +1;`setCount(c=>c+1)`×3 才 +3 |
| 🔬 对照 | 普通变量为什么存不住信息? | 每次渲染被重置,且改了不触发重渲染 |

## 核心结论速查

- JSX 是语法糖,最终都是纯 JS 在跑
- `{}` 里的表达式在**渲染时求值**——写 `()` 就当场调用,不写 `()` 只登记
- 传给事件的永远是"菜谱"(函数),不是"做好的菜"(执行结果)
