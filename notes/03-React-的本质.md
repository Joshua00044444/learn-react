# React 的本质:一套"如何更好地使用 JS"的规则

> 📌 本文的 React 技术知识,来自 D 盘的 **DeepSeekMonitorWindows-1** 项目。

**React 在语法层面 100% 是 JavaScript,它没有发明任何新语言**——你写的每一个 React 程序,最终都是纯 JS 在浏览器里跑。React 的价值不在于新语法,而在于它在纯 JS 之上**约定了一套运行规则**。

## React 概念 → JS 本质对照表

| React 里的概念 | 本质上就是 |
|---|---|
| 组件 `<Profile />` | 一个普通的 JS 函数 |
| JSX | 语法糖,编译成 `createElement()` 函数调用 |
| props | 函数的入参(一个对象) |
| state(`useState`) | 闭包里存的变量 + 一套更新机制 |
| 事件 `onClick={fn}` | 传给元素的函数属性 |
| ref | 一个普通对象 `{ current: ... }` |
| 组件组合 | 就是函数互相调用 |

例如:

```jsx
function Greet({ name }) {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{name}: {count}</button>;
}
```

剥掉 JSX 糖衣后,全都是 JS 基础:**函数、对象参数、闭包、箭头函数、模板表达式**。

## React 约定的核心规则

| React 的规则 | 约束的是什么 |
|---|---|
| 用组件拆分界面 | 界面代码该怎么组织(函数化、可复用) |
| 声明式描述 | 你只说"界面长什么样",别手动改 DOM |
| 数据单向流动 | props 往下传、事件往上传,数据流向可预测 |
| state 驱动渲染 | 状态变了 → React 自动重新渲染,你不用操作 DOM |
| Hooks 规则 | 逻辑怎么复用、怎么挂在组件生命周期上 |

## 对比:没有 React(命令式) vs 有 React(声明式)

**原生 JS,命令式——你自己一步步操作 DOM:**

```js
document.getElementById("count").innerText = count;
box.classList.add("active");
```

**React,声明式——你只声明界面长什么样,更新交给 React:**

```jsx
<div id="count">{count}</div>
```

能力上没有任何新增——还是同一个 JS、同一套 DOM。变化的是**谁负责"怎么更新"**:以前你自己手动改,现在你按 React 的规则声明,它替你干脏活。

## 同理类推

- **Vue** 也是 JS,只是另一套规则(模板语法 + 响应式)
- **Next.js** 是在 React 规则之上再叠一层规则(路由、渲染时机)
- **DOM API** 本身也是浏览器提供的一套规则

## 学习分层

> **JS 是语言和材料,框架是使用材料的"章法"。**
> 章法会过时会更换,材料功底(闭包、异步、原型、事件循环)才是跟着你走的。
>
> 学 React = 学 JS 基础 + 学 React 的运行规则。看文档时把"新概念"都翻译成"它在约定什么",学起来会快很多。
