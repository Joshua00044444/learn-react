# onClick:传函数,还是传执行结果?

> 📌 本文的 React 技术知识,来自 D 盘的 **DeepSeekMonitorWindows-1** 项目。

这是 React 入门最经典的坑:`onClick={handleClick}` 和 `onClick={handleClick()}` 差一个字,含义完全不同。

## 核心:加不加 `()`,是两个东西

```js
handleClick    // 函数本身(一个"东西",可以递给别人)
handleClick()  // 立刻执行这个函数(一个"动作",当场就发生)
```

- 不带括号 = **把函数递过去**:"这是遥控器,你拿着"
- 带括号 = **当场执行**:"我现在就按"

## 两种写法的真实行为

**✅ 正确:`<button onClick={handleClick}>`**

意思是:"React,这个函数给你存着,**等用户点击的时候再调用它**"。
时间线:渲染时只是登记 → 用户点击 → React 调用 `handleClick` → 弹出 alert。

**❌ 错误:`<button onClick={handleClick()}>`**

`handleClick()` **在渲染的那一刻就立刻执行了**——页面一打开,alert 就弹出来,根本没等人点。而它返回的值(`undefined`)才被交给 `onClick`,所以之后用户真去点按钮,什么也不会发生。

> **onClick 要的是"函数",不是"函数执行的结果"。**

## 为什么 `()` 必然在渲染时执行?(求值顺序)

JSX 就是普通 JS。`<button onClick={handleClick()}>` 编译后大致是:

```js
React.createElement("button", { onClick: handleClick() }, "点我");
```

JS 要创建属性对象 `{ onClick: handleClick() }`,就必须**先求值** `handleClick()` 才知道往里存什么——这一"算",函数就跑了。`onClick` 属性名只是存放位置的标签,管不了表达式什么时候执行。

用纯 JS 类比:

```js
const a = foo;   // a 是函数本身,foo 不执行
const b = foo(); // 为了得到值,foo 立刻执行
```

**不是 onClick 决定执行时机,而是你递给它的是"函数"还是"执行结果"决定执行时机。**

## 需要传参数怎么办?

不能写 `handleClick('hi')`(会立刻执行),用箭头函数包一层:

```jsx
<button onClick={() => handleClick('hi')}>
```

`{() => handleClick('hi')}` 传的仍然是一个**函数**(箭头函数本身没被执行),React 存着它,点击时才调用,它内部再带参数调 `handleClick`。

## 实测验证(2026-09-10,react-onclick-demo.html)

做了一个三卡片对比页面(本目录 `react-onclick-demo.html`),带时间戳日志实测:

```
❌ 错误卡片:
[21:25:57.854] 手动重新挂载组件,触发一次渲染 ↓
[21:25:57.855] handleClick 执行!触发者:渲染过程(不是点击!)
→ 点击「重新挂载」后仅 1 毫秒,handleClick 就在渲染中被执行;
→ 之后再点「点我」按钮,日志零新增(onClick 里是 undefined)。

✅ 正确卡片:
[21:25:05] 组件挂载(onClick={handleClick},函数未执行)
[21:25:29] handleClick 执行!(由用户点击触发)   ← 挂载 24 秒后才有第一条
→ 渲染时不执行,每次都精确对应一次真实点击。
```

**验证方法**:打开 demo 页面,点红色错误卡片的「🔄 重新挂载」——alert 会在你没碰大按钮的情况下自己弹出来;再随便点大按钮,永远没反应。

## 记忆口诀

> **传给事件的永远是"菜谱",不是"做好的菜"。**
> 不带括号是递菜谱;需要带参数时,用箭头函数把菜谱包一层再递过去。
