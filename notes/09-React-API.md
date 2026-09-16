# 内置的 React API:Hooks 和组件之外的第三类导出

06 篇梳理了官方参考里的**内置 Hook**(`useXxx` 那批),08 篇过完了**内置组件**(`<Fragment>` 那批);`react` 包的导出还剩第三类——**既不是 Hook、也不是组件的裸 API**。官方参考页(<https://zh-hans.react.dev/reference/react/apis>)把「除了 Hooks 和 Components 之外」的导出都收在这里,开篇点名五个最常用的:

> `createContext` 创建 context(配 `useContext`);`lazy` 延迟加载组件;`memo` 在 props 没变时跳过重渲染(配 `useMemo`/`useCallback`);`startTransition` 把更新标记为不紧急(类似 `useTransition`);`act` 在测试中包住渲染和交互,确保断言前更新已完成。

参考页侧边栏一共登记了 12 个,按「成熟度」分三档,先给全景:

| API | 状态 | 一句话 | 本篇 |
|---|---|---|---|
| `createContext` | ✅ 稳定 | 开一条 Context「频道」,给子树广播数据 | 有 demo |
| `lazy` | ✅ 稳定 | 组件延迟到首次渲染才加载,配 `Suspense` | 有 demo |
| `memo` | ✅ 稳定 | props 浅比较没变就跳过重渲染 | 有 demo |
| `startTransition` | ✅ 稳定 | 把不急的更新标成「可让路」的过渡 | 有 demo |
| `use` | ✅ 稳定(React 19) | 渲染中读 Promise / Context,还能写进条件 | 有 demo |
| `act` | ✅ 稳定(React 19) | 测试专用:断言前强制把更新跑完 | 有 demo |
| `captureOwnerStack` | ✅ 稳定(仅开发构建) | 拿到「谁渲染了它」的组件栈 | 有 demo |
| `cache` | ✅ 稳定(Server Components) | 同一次服务端渲染内缓存函数结果 | 纯文档 |
| `cacheSignal` | ✅ 稳定(Server Components) | 拿一个「本次渲染结束就中止」的 AbortSignal | 纯文档 |
| `addTransitionType` | 🧪 实验通道 | 给过渡打类型标记,配合 ViewTransition 做动画 | 纯文档 |
| `experimental_taintObjectReference` | 🧪 实验通道 | 服务端「弄脏」整个对象,禁止它流向客户端 | 纯文档 |
| `experimental_taintUniqueValue` | 🧪 实验通道 | 服务端「弄脏」某个具体值(密钥/token)防外泄 | 纯文档 |

后五个不配浏览器 demo:`cache`/`cacheSignal` 只在 Server Components 的服务端渲染里生效,实验通道的三个要用 experimental 构建才有——但概念都在下面讲清楚。前七个每一个都有完整可跑示例 + 逻辑链路,并能在 demo 页(`react-val-study/react-built-in-api-demo.html`)亲手点出来。

## `createContext`:开一条广播频道

06 篇用「频道/广播/收音机」讲过 `useContext` 三步走,这里补上 06 篇没展开的三个细节,示例给完整版:

```jsx
import { createContext, useContext, useState } from 'react';

// ① 必须写在组件外(模块顶层):useContext 靠「对象身份」对频道,
//    写在组件里每次渲染都会造新频道,广播站和收音机就对不上了
const ThemeContext = createContext('light'); // 参数 = 头顶没有 Provider 时的默认值

function ThemedButton() {
  // ③ 后代收听:头顶最近一个 Provider 的 value;一路都没有 → 默认值
  const theme = useContext(ThemeContext);
  return <button className={theme}>当前主题:{theme}</button>;
}

export default function App() {
  const [theme, setTheme] = useState('dark'); // 广播的内容存在祖先的 state 里

  return (
    <div>
      {/* 这个按钮在 Provider 外面:读到的是默认值 'light' */}
      <ThemedButton />

      {/* ② 祖先用 Provider 广播:属性名必须叫 value
           React 19 起还能直接写 <ThemeContext value={theme}> */}
      <ThemeContext.Provider value={theme}>
        <ThemedButton />
        <button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
          切换主题
        </button>
      </ThemeContext.Provider>
    </div>
  );
}
```

逻辑链路:

- ① 首次渲染:页面上有**两个** `ThemedButton`——Provider 外的那个向上找不到 Provider,`useContext` 返回 `createContext('light')` 的默认值;Provider 里的那个返回 `'dark'`。**同一个组件,读到什么取决于它挂在哪**,这就是 Context 的核心。
- ② 点「切换主题」→ `setTheme('light')` → Provider 的 `value` 变了 → React 把 **Provider 包住的所有消费组件**重渲染一遍(里的按钮变成 light),Provider 外的那颗纹丝不动。
- ③ 为什么 `value={theme}` 要接 state:Context 只负责「把值送到哪里」,不负责「值从哪来、怎么变」——值的更新还得靠 state + `setState`。Context 是广播喇叭,state 是广播稿。

两个容易踩的坑:

- **默认值只是兜底,不是初始值**。它只在「头顶没有 Provider」时生效,而且这种情况下 React **不会**因为值的含义变化通知你重渲染。想表达「暂时没有数据」,惯例是 `createContext(null)` 然后自己判空,别拿一个像真数据的默认值糊弄。
- **传对象时 value 的引用要稳**:`value={{ theme, setTheme }}` 每次渲染都是新对象,所有消费者跟着陪跑;要用 `useMemo(() => ({ theme, setTheme }), [theme])` 包一层。

## `lazy`:首次渲染才加载,加载完有缓存

```jsx
import { Suspense, lazy, useState } from 'react';

// ⚠️ 必须写在模块顶层(组件外)。写在组件渲染里的话,
// 每次渲染都会造一个「新组件类型」,React 会把子树卸载重挂、重新加载
const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));

export default function DocPage() {
  const [show, setShow] = useState(false);

  return (
    <div>
      <button onClick={() => setShow(true)}>显示预览</button>
      {show && (
        <Suspense fallback={<p>⏳ 正在加载预览组件……</p>}>
          <MarkdownPreview />
        </Suspense>
      )}
    </div>
  );
}
```

逻辑链路:

- ① `lazy(加载函数)` 接受一个返回 Promise 的函数;**组件第一次渲染到 `<MarkdownPreview/>` 时**,React 才去调用它——这就是「延迟加载」,首屏不用背着这个模块。
- ② Promise 还没 resolve → 组件「挂起」→ 最近的 `<Suspense>` 用 `fallback` 顶班(08 篇的顶班机制);
- ③ 加载函数必须 resolve 成 **`{ default: 组件 }`**——和 ES 模块的 default 导出对齐,`import()` 拿到的天生就是这个形状;
- ④ resolve 后 React 重试渲染,真组件**整体替换** fallback。
- ⑤ **结果有缓存**:同一个 `lazy` 组件第二次渲染到时,加载函数不会被再次调用,直接用上次的模块,连 fallback 都不会闪——不用你写任何 `loaded` 标记。

Promise 被 reject(比如网络断了)会抛错,由错误边界(ErrorBoundary)接住——`lazy` 管「等多」,错误边界管「等砸了」。

## `memo`:props 浅比较没变,就跳过重渲染

父组件一渲染,所有子组件默认跟着渲染——哪怕收到的 props 一个字没变。`memo` 给子组件加一道「浅比较」闸门:

```jsx
import { memo, useCallback, useMemo, useState } from 'react';

function CountBadge({ count }) {
  // 没包 memo:父组件渲染,它一定陪跑
  return <span>计数:{count}</span>;
}

const MemoBadge = memo(function MemoBadge({ label, onReset }) {
  console.log('MemoBadge 渲染了');
  return <button onClick={onReset}>{label}</button>;
});

export default function App() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);

  // 正确姿势:对象用 useMemo 稳引用,函数用 useCallback 稳引用
  const config = useMemo(() => ({ color: 'blue' }), []);   // 依赖空 → 永远同一个对象
  const handleReset = useCallback(() => setCount(0), []);  // 依赖空 → 永远同一个函数

  return (
    <div>
      <CountBadge count={count} />
      {/* ❌ memo 失效写法:每次渲染都是新对象新函数
      <MemoBadge label="重置" config={{ color: 'blue' }} onReset={() => setCount(0)} />
      */}
      {/* ✅ 生效写法 */}
      <MemoBadge label="重置" config={config} onReset={handleReset} />
      <button onClick={() => setCount(count + 1)}>count + 1</button>
      <button onClick={() => setOther(other + 1)}>无关 state + 1</button>
    </div>
  );
}
```

逻辑链路(点「无关 state + 1」最说明问题):

- ① 点「无关 state + 1」→ `setOther` → `App` 重渲染 → `CountBadge` 陪跑(memo 不包没办法);
- ② 轮到 `MemoBadge`:React 把**这次的 props 和上次的 props 逐个做 `Object.is` 浅比较**——`label` 是同一个字符串、`config` 是 `useMemo` 留下的同一个对象、`onReset` 是 `useCallback` 留下的同一个函数 → 全等 → **直接复用上一次的渲染结果,组件函数根本不执行**(console 里没有新日志);
- ③ 如果按注释里的失效写法传「裸对象 + 裸箭头函数」,每次渲染都是新引用,浅比较必然不等,`memo` 等于白包——这就是官方说「memo 通常与 `useMemo`/`useCallback` 配合使用」的原因。

三个补充:

- `memo(组件, 比较函数)` 的第二个参数可以自定义比较,**返回 `true` 表示相等(跳过渲染)**;默认行为就是浅比较,大多数时候不需要自己写。
- 浅比较只比一层:`props.obj.a` 变了但 `props.obj` 引用没变,memo 是感知不到的(这正是 `useMemo` 要解决的另一半问题)。
- **别无脑全包**:memo 本身有比较成本,只给「渲染贵 + 经常被无关更新牵连 + props 常常其实没变」的组件包。

## `startTransition`:把不急的更新标记为「可让路」

React 把状态更新分成两档:**急的**(打字、悬停、点击——必须立刻反馈)和**缓的**(大列表过滤、切换视图——晚零点几秒无感)。`startTransition` 把包在里面的更新划到「缓」那一档:

```jsx
import { startTransition, useState } from 'react';

function FilterList({ items }) {
  const [text, setText] = useState('');   // 急:输入框自己的值,永远同步更新
  const [query, setQuery] = useState(''); // 缓:列表用的值,装进过渡里慢慢来

  function onChange(e) {
    setText(e.target.value);        // 急更新:立即执行,输入框跟手
    startTransition(() => {         // 缓更新:可打断、可让路
      setQuery(e.target.value);     // 就算要渲染几百毫秒,也不拖住下一次打字
    });
  }

  const rows = items.filter((it) => it.includes(query));
  return (
    <div>
      <input value={text} onChange={onChange} />
      <ul>{rows.map((it) => <li key={it}>{it}</li>)}</ul>
    </div>
  );
}

export default function App() {
  const items = [];
  for (let i = 0; i < 20000; i++) items.push('条目-' + i); // 一份足够大的数据
  return <FilterList items={items} />;
}
```

逻辑链路:

- ① 用户每敲一个键 → `onChange` 执行 → `setText` 是急更新,输入框**立刻**显示新字符;
- ② `setQuery` 被包进 `startTransition`:React 知道这个更新「不急」,渲染大列表时**随时可以被打断**——用户又敲了一个键,React 会扔掉渲染到一半的旧结果,直接按最新的 `query` 重来;
- ③ 净效果:输入框永远跟手,列表慢半拍自己追,连打 N 个键中间状态可能被整体跳过。不加过渡的话,每个键都要同步等一次大渲染,打字就是一卡一卡的。

和两位亲戚的关系:

- **`useTransition`**(06 篇)= `startTransition` + 一个 `isPending` 布尔值,用来显示「加载中」转圈;不提示就用 `startTransition`。
- **`useDeferredValue`**(06 篇)是按「值」缓的写法(`const query = useDeferredValue(text)`),不用拆两个 state;效果类似,选一种顺手的。

## `use`:渲染中读 Promise / Context,还能写进条件

React 19 新 API,名字最短,身兼两职:

```jsx
import { Suspense, use, createContext, useContext } from 'react';

const ThemeContext = createContext('light');
const dataPromises = {}; // ⚠️ 铁律:promise 必须缓存!在渲染里现造新 promise 会无限挂起

function fetchData(which) {
  if (!dataPromises[which]) {
    dataPromises[which] = fetch('/api/' + which).then((r) => r.json());
  }
  return dataPromises[which];
}

function DataView({ which }) {
  // use 不是 Hook,可以写在 if 里:which 不同,读的 promise 也不同
  if (which === 'a') {
    const a = use(fetchData('a'));   // promise 没就绪 → 组件在这里「挂起」
    return <p>{a.title}</p>;
  }
  const b = use(fetchData('b'));
  return <p>{b.title}</p>;
}

export default function Page() {
  return (
    <Suspense fallback={<p>⏳ 数据加载中……</p>}>
      <DataView which="a" />
    </Suspense>
  );
}

// use 读 Context 与 useContext 完全等价:
// const theme = use(ThemeContext);  ≈  const theme = useContext(ThemeContext);
```

逻辑链路:

- ① `use(promise)`:promise 还没 resolve → 组件挂起 → `Suspense` 顶班;resolve 后 React 重试,`use` 直接返回数据。相当于把「`useEffect` 里 fetch + `isLoading` state + 条件渲染」三件套压成一行,加载态交给 `Suspense`。
- ② **promise 必须在渲染外创建或缓存**(模块级变量、`cache()`、`useMemo`):要是在渲染里写 `use(fetch(...))`,每次渲染都造一颗新 promise,永远等不到同一颗 resolve,子树无限挂起。
- ③ `use(Context)`:和 `useContext` 等价;**区别是 `use` 可以写在 if/循环/`&&` 后面**——`useContext` 是 Hook,受「只能在顶层无条件调用」约束,`use` 不是 Hook,条件调用合法。上面的 `DataView` 换成 `useContext` 就得拆成两个组件。

一个分工说明:在 **Server Components**(服务端组件)里读异步数据是直接 `await`,根本不用挂起;`use` 是这个能力在**客户端组件**里的对应物。

## `act`:测试专用,断言前把更新跑完

先建立一个事实:点击触发的 `setState` **不是同步改 DOM**,只是「预约」一次更新。测试里如果你「点击后立刻断言」,读到的多半还是旧 DOM。`act` 就是来解决这个的:

```jsx
// 测试环境全局配置(Jest/Vitest 的 setup 文件里写一次):
globalThis.IS_REACT_ACT_ENVIRONMENT = true;

import { act } from 'react'; // React 19 起 act 从 'react' 导出
import { createRoot } from 'react-dom/client';
import Counter from './Counter.js';

test('点击后计数 +1', async () => {
  const container = document.createElement('div');
  const root = createRoot(container);

  await act(async () => {
    root.render(<Counter />);   // 挂载
  });

  const btn = container.querySelector('button');

  // ❌ 不用 act:点击后立刻断言 → 更新只是被「预约」,DOM 还是旧的
  // btn.click();
  // expect(btn.textContent).toContain('1');  // 实际读到 0,挂

  // ✅ 用 act:act 会把里面的更新(渲染 + Effect)全部跑完才放行
  await act(async () => {
    btn.click();
  });
  expect(btn.textContent).toContain('1'); // 稳过

  root.unmount();
});
```

逻辑链路:

- ① `btn.click()` → `setCount(1)` → React **预约**更新(调度器排进队列),`click()` 这行代码返回时 DOM 还没变;
- ② `await act(async () => { … })`:act 执行回调后,**强制 React 把队列里的更新、连带 Effect 全部执行完**,才让 `await` 往下走;
- ③ 断言执行时 DOM 一定是最终状态 → 测试变成确定性的,不再「偶尔挂、偶尔过」。

三个实用说明:

- **`IS_REACT_ACT_ENVIRONMENT = true` 必须设**:不设的话 React 会在控制台告警「当前不是测试环境却用了 act」。Jest/Vitest + Testing Library 的模板都帮你配好了。
- **平时你几乎不手写 act**:`@testing-library/react` 的 `render`、`fireEvent`、`userEvent` 内部已经把 act 包好了,知道它在干嘛即可。
- React 18 及更早要从 `react-dom/test-utils` 导入;React 19 起才从 `react` 直接导出。

## `captureOwnerStack`:拿到「谁渲染了它」的组件栈

排查「这个报错的组件到底从哪渲染进来的」时,把**组件层谱系**打到日志/错误上报里:

```jsx
import { captureOwnerStack } from 'react';

function ErrorReporter({ error }) {
  // 必须在「渲染期间」调用(渲染函数体、Effect 里都行)
  const stack = captureOwnerStack();
  reportToServer(error, stack);
  return null;
}

// stack 的形状(一段类似错误堆栈的字符串,从近到远列祖先):
//     at MidLayer (src/App.js:12)
//     at OwnerStackDemo (src/App.js:30)
//     at App (src/App.js:55)
```

要点:

- 栈里是 **owner 链**——「谁把它渲染出来的」那一串祖先组件,**调用它的组件自己不在栈里**。注意 owner 是「谁创建了这些元素」,不是 DOM 父子链,两者在 `children` 传参等场景下会不一致。
- **仅开发构建有内容**;生产构建返回 `null`——别把它当线上功能用,它的定位是开发期调试和测试环境里的错误报告。
- 最典型的用途:全局错误处理里附带 owner 栈,收到报错一眼定位是哪条组件链渲染出来的。

## `cache` / `cacheSignal`:Server Components 的按请求缓存

这两个服务于 **Server Components**(服务端组件,渲染只发生在服务器上),浏览器 demo 验不了,但概念要知道:

```jsx
// ⚠️ 以下代码运行在服务器(Server Components 环境),不是浏览器
import { cache, cacheSignal } from 'react';
import { db } from './database.js';

// cache(函数) 返回一个「按请求缓存」的版本:
// 同一次渲染请求里,相同参数只真的查一次库
const getUser = cache(async (id) => {
  const signal = cacheSignal(); // 本次渲染作用域的 AbortSignal
  return db.users.find(id, { signal });
});

export default async function Profile({ userId }) {
  // 页面里两个组件都调 getUser(userId),也只打一次数据库——
  // cache 会把第一次的结果直接喂给第二次
  return (
    <section>
      <UserCard data={await getUser(userId)} />
      <UserStats data={await getUser(userId)} />
    </section>
  );
}
```

- **`cache` 解决「一次请求内重复干活」**:缓存粒度是单次服务端渲染请求,请求结束自动清空(所以不会像全局缓存那样跨用户串数据)。没有它,同一份数据要靠一层层传 props 才能不重复查。
- **`cacheSignal` 解决「白干的活要及时止损」**:返回一个 `AbortSignal`,当前这次渲染被放弃(用户关页面、请求被丢弃)时它会中止,传给 `fetch`/数据库驱动就能停掉已经不需要的查询。
- 提醒:它们**不在客户端生效**,纯浏览器页面里调了也没意义。

## `addTransitionType` 与两个 `experimental_taint*`:实验通道速览

侧边栏里带 🧪 烧瓶图标的三个,活在 **experimental 构建**里,API 随时可能改,这里只建立概念、别写进生产代码:

**`addTransitionType`**:写在 `startTransition` 回调里,给这次过渡打个「类型标签」,配合同样处于实验阶段的 `<ViewTransition>` 组件,让不同类型的过渡播放不同的进出动画:

```jsx
import { startTransition, addTransitionType } from 'react'; // experimental 构建

function switchTab(tab) {
  startTransition(() => {
    addTransitionType('switch-tab'); // 给这次过渡贴标签
    setActiveTab(tab);
  });
}
```

**`experimental_taintObjectReference` / `experimental_taintUniqueValue`**:服务端数据安全网。在数据出口处把敏感内容「弄脏」,此后无论代码怎么不小心,只要它试图被传进客户端组件,React 直接抛错:

```jsx
// 服务器数据层(experimental 构建)
import { experimental_taintObjectReference, experimental_taintUniqueValue } from 'react';

export async function getUser(id) {
  const user = await db.users.find(id);
  // 弄脏整个用户对象:谁把它整个传给客户端组件,立刻报错
  experimental_taintObjectReference('不能把完整用户记录传给客户端', user);
  return user;
}

export async function getToken(user) {
  // 弄脏单个值:哪怕有人把 token 复制进别的对象再传,照样报错
  experimental_taintUniqueValue('token 不允许离开服务端', user, user.token);
  return user;
}
```

`taintObjectReference` 管「整个对象别出门」;`taintUniqueValue` 管「这个具体值(密钥、token、身份证号)别出门」,连被复制出去都能追到。定位是**纵深防御的最后一道网**——它防的是「不小心」,防不了蓄意,查询时该不查的敏感字段还是别查。

## 实测验证(2026-09-16,react-val-study/react-built-in-api-demo.html)

七张卡片,带时间戳日志实测(React 19.2 development 构建):

- **📡 createContext**:点 Provider 外的按钮,`useContext` 读到默认值 `light`;点 Provider 里的按钮读到 `dark`;点「切换」后日志:`setTheme('light') → Provider 的 value 变了 → 包住的组件自动重渲染,Provider 外的按钮纹丝不动`——默认值与广播范围肉眼可证。
- **📦 lazy**:点「新建一个 lazy 并挂载」,日志依次是 `① 工厂被调用(累计第 1 次) → fallback 顶班 1.5 秒 → ③ Promise resolve({default: 组件}),真组件整体替换`;点「卸载再挂载」,工厂**没有被再次调用**、无 fallback——lazy 的缓存实证。
- **🧊 memo**:父组件 +1 三次后点「报数」:`普通 4 次 / memo-失效 4 次 / memo-生效 1 次`(4 = 首次挂载 + 3 次更新)。包了 memo 但吃裸对象/裸函数的照样陪跑,props 用 `useMemo`/`useCallback` 稳住的纹丝不动——浅比较规则一次看全。
- **🕊️ startTransition**:两个输入框分别逐键输入「096」,Profiler 日志打出每次列表渲染实际耗时约 **80ms**;普通模式一个 setState 连输入框一起拖,过渡模式输入框同步更新、列表更新被包进 `startTransition` 慢半拍(连打时中间渲染可被打断合并)。
- **🪝 use**:点「读 B」,日志 `① 给 B 建 promise(缓存) → Suspense 顶班 → ③ resolve,use 返回数据`;再点「读 A」(A 早已 resolve)**秒出**、不再建 promise——promise 缓存铁律正反两面都验了。勾掉条件开关,`MaybeReader` 这次渲染根本没调用 `use(Context)`——条件调用,普通 Hook 做不到。
- **🧪 act**:迷你测试两条断言同屏对照——不用 act,点击后立刻断言读到 `"act 计数:0"`,**FAIL**(更新只是被预约);用 act 包住同样的点击再断言,读到 `"act 计数:1"`,**PASS**。
- **🧬 captureOwnerStack**:页面一渲染就有输出,栈从近到远列出祖先:`at MidLayer → at OwnerStackDemo → at App`,调用它的 `DeepChild` 自己不在栈里;仅开发构建有内容,生产返回 `null`。

⚠️ 验证页沿用 08 篇的环境:**esm.sh 的 React 19.2 development 构建**(import map + ESM,需要联网;双击报 CORS 错就用本地静态服务打开本目录)。act 的 `IS_REACT_ACT_ENVIRONMENT` 行为、captureOwnerStack 的输出都依赖开发构建。

## 记忆口诀

> **Context 开频道,lazy 搭 Suspense;memo 靠稳引用,transition 分急缓;use 读 Promise,act 保断言。**
> 默认值是兜底;lazy 只加载一次(必须 `{default}`);memo 只浅比较(对象函数要 useMemo/useCallback);过渡不阻塞输入(要转圈用 useTransition);use 不是 Hook(能写进 if,promise 必须缓存);act 只活在测试里(IS_REACT_ACT_ENVIRONMENT);captureOwnerStack 仅开发有(栈里只有祖先没有自己);cache/cacheSignal 属于服务端,带 🧪 的三个先观望。
