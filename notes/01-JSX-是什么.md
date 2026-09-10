# JSX 是什么

> 📌 本文的 React 技术知识,来自 D 盘的 **DeepSeekMonitorWindows-1** 项目。

**JSX(JavaScript XML)是一种 JavaScript 的语法扩展**,主要用于 React(以及 Vue、Solid 等框架)中,让你可以**在 JS 代码里直接写类似 HTML 的结构**来描述界面。

## 核心概念

浏览器本身不认识 JSX,它必须经过编译器(如 Babel、esbuild、Vite/SWC)转换成普通的 JavaScript 函数调用才能运行。

**你写的 JSX:**

```jsx
const element = <h1 className="title">Hello, {name}</h1>;
```

**编译后的结果(React 场景):**

```js
const element = React.createElement("h1", { className: "title" }, "Hello, ", name);
// 新版 React 17+ 也可以是 jsx("h1", {...}, ...) 的形式
```

也就是说,JSX 本质上是 `createElement` 函数调用的**语法糖**,最终生成一个描述界面的普通 JS 对象(称为"虚拟 DOM")。

## 主要语法规则

| 规则 | 示例 |
|------|------|
| 嵌入表达式用 `{}` | `<p>{1 + 2}</p>` → 显示 3 |
| 属性用驼峰命名 | `className`、`onClick`、`tabIndex` |
| 必须有唯一根节点 | 用 `<div>` 或 `<>...</>`(Fragment)包裹 |
| 标签必须闭合 | `<br />`、`<img src="..." />` |
| 小写开头 = 原生标签,大写开头 = 组件 | `<div>` vs `<MyButton>` |

## 典型用法

```jsx
function TodoList({ items }) {
  // {} 里可以放任意 JS 表达式:变量、函数调用、三元运算符等
  const count = items.length;

  return (
    <div>
      {/* JSX 里的注释写法 */}
      <h1>共 {count} 项</h1>

      {/* 条件渲染 */}
      {count === 0 ? <p>暂无数据</p> : null}

      {/* 列表渲染:用 map 生成元素数组,必须提供 key */}
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.text}</li>
        ))}
      </ul>

      {/* 事件处理:属性名是 onClick 而不是 onclick */}
      <button onClick={() => console.log("clicked")}>点我</button>
    </div>
  );
}
```

## 为什么用 JSX?

1. **声明式**:直接描述"界面长什么样",而不是一步步命令式地操作 DOM。
2. **类型安全**:配合 TypeScript(`.tsx` 文件)可以在编译期检查组件 props、标签拼写等错误。
3. **完整的 JS 能力**:它就是 JS,可以使用变量、函数、模块化等一切 JS 特性,不像模板语言那样受模板语法限制。

## 常见注意点

- `{}` 里**不能直接写 `if`/`for` 语句**(只能放表达式),条件逻辑可用三元运算符、`&&` 或在 JSX 外先算好。
- `class` 要写成 `className`,因为 `class` 是 JS 保留字。
- `style` 接收对象而非字符串:`style={{ color: 'red' }}`(外层 `{}` 是表达式,内层是对象字面量)。
- 渲染列表时忘了 `key` 会引发性能问题和警告。

> 一句话总结:**JSX = 让你用 HTML 的写法来构造 JS 对象,从而以声明式的方式描述 UI。**
