# JSX 与 JS 的区别

> 📌 本文的 React 技术知识,来自 D 盘的 **DeepSeekMonitorWindows-1** 项目。

一句话:**JS 是编程语言本身,JSX 是在 JS 基础上扩展出来的"方言"**——它让你能在 JS 代码里写 HTML 样式的标签,但它不是标准的 JS,浏览器无法直接运行。

## 1. 本质区别

**标准 JS(浏览器能直接跑):**

```js
const name = "Hedy";
console.log("Hello " + name);
```

**JSX(JS 里混入了标签语法,浏览器不认识):**

```jsx
const element = <h1>Hello {name}</h1>;
```

第二段代码如果直接交给浏览器会**报语法错误**,必须先经过编译器(Babel / esbuild / SWC)转换成纯 JS:

```js
const element = React.createElement("h1", null, "Hello ", name);
```

所以 JSX 的依赖链是:**JSX →(编译)→ JS →(浏览器执行)**。

## 2. 主要区别对比

| | JS | JSX |
|---|---|---|
| 身份 | 标准 ECMAScript 语言 | JS 的语法扩展(React 推广) |
| 浏览器能否直接运行 | ✅ 能 | ❌ 不能,需先编译 |
| 文件后缀 | `.js` | `.jsx`(TypeScript 则是 `.tsx`) |
| 能写什么 | 变量、函数、循环……纯逻辑 | 逻辑 + 类似 HTML 的 UI 结构 |
| 属于谁的语法 | 语言规范 | 框架生态(React/Vue/Solid 等都可用) |

## 3. 具体语法差异举例

**只有 JSX 能写的:**

```jsx
// 标签直接出现在代码里 —— 这是 JSX 独有的
const box = (
  <div className="card">
    <img src="a.png" alt="" />
    <p>{count > 0 ? "有数据" : "空的"}</p>
  </div>
);
```

**换成纯 JS 就必须用函数调用来描述同样的东西:**

```js
// 纯 JS 写法:没有 JSX 时只能这样
const box = React.createElement(
  "div",
  { className: "card" },
  React.createElement("img", { src: "a.png", alt: "" }),
  React.createElement("p", null, count > 0 ? "有数据" : "空的")
);
```

两种写法**运行结果完全一样**,JSX 只是后一种的"糖"。

## 4. 容易混淆的点

- **JSX ≠ HTML**:虽然长得像,但它是 JS 的一部分,规则遵循 JS——属性用驼峰(`className`、`onClick`)、必须闭合、要编译。
- **React 不强制用 JSX**:你完全可以只用 `.js` + `React.createElement` 写 React,只是没人这么干,因为太难读。
- **后缀是有实际影响的**:比如 Vite 默认配置下,`.js` 文件里写 JSX 会直接编译报错,必须把文件改名成 `.jsx`(或 `.tsx`)。

## 5. 记忆方式

> **JSX = JS + 描述界面的标签语法。**
> 逻辑部分照写 JS,界面部分用标签声明,编译器把标签翻译成函数调用,最终还是纯 JS 在跑。
