Hooks 是 React 16.8 推出的里程碑特性，核心目的是 **让函数组件拥有类组件的状态管理和生命周期能力**，彻底解决了函数组件无法维护状态、代码复用繁琐的痛点。其底层原理围绕「Hook 调用顺序」和「Hook 存储结构」展开，逻辑简洁但约束严格，是面试高频考点。


## 一、核心前提：为什么 Hooks 必须依赖固定调用顺序？

通俗理解：函数组件每次渲染（首次渲染/重渲染）都会从头到尾重新执行一遍，Hooks 要想“记住”每次渲染的状态，就必须保证每次执行时，调用的顺序完全一致——就像排队领号，每次排队的顺序不能乱，才能对应到自己的号码（状态）。

专业拆解：这是 Hooks 原理的基石，核心是「状态与 Hook 调用的一一对应」，依赖两个关键底层设计：

### 1. 底层存储结构：Hook 链表（核心！）

React 内部为每个组件的 Fiber 节点（组件的底层抽象表示，可理解为“组件的骨架”），维护了一个 **Hook 单向链表**，用于存储该组件所有 Hooks 的相关信息。

每个 Hook 本身是一个对象，官方简化结构：

```javascript
// 单个 Hook 节点的极简结构
const hook = {
  memoizedState: null, // 存储当前 Hook 的状态（如 useState 的值、useEffect 的回调）
  next: null, // 指向下一个 Hook 节点，串联成链表
  queue: null, // 存储该 Hook 的更新队列（如 setState 触发的新值）
};
```

补充说明：组件 Fiber 节点中，通过 `fiber.memoizedState` 指向 Hook 链表的头节点，后续每个 Hook 节点通过 `next` 依次连接，形成完整的链表结构（对应题干中 `fiber.memoizedState = { memoizedState, next }` 的极简模型）。

除了 Hook 链表，每个 Hook 节点还包含一个「更新队列」（queue），用于存储该 Hook 的待更新状态（比如 useState 调用 setXxx 时，新值会先存入 queue，等待组件重渲染时更新）。

### 2. 调用顺序的核心作用

React 无法通过“变量名”识别 Hooks，只能通过「调用顺序」匹配 Hook 链表中的节点，具体流程分两步：

1. **首次渲染**：函数组件执行时，会依次调用 useState、useEffect 等 Hooks，每调用一个 Hook，就创建一个对应的 Hook 节点，挂载到 Hook 链表的末尾，同时初始化 memoizedState（状态）和 queue（更新队列）。

2. **重渲染**：组件因 setState、props 变化等触发重渲染时，函数组件会再次执行，此时 React 会从 Hook 链表的头节点开始，按「与首次渲染完全相同的顺序」遍历链表，读取每个 Hook 节点的 memoizedState，从而保证“Hook 调用”与“状态”一一对应。

举个通俗例子：

```jsx
function Counter() {
  // 第1个 Hook：对应链表头节点，存储 count 状态
  const [count, setCount] = useState(0);
  // 第2个 Hook：对应链表第二个节点，存储 name 状态
  const [name, setName] = useState('React');
  return <button onClick={() => setCount(count+1)}>{count}-{name}</button>;
}
```

首次渲染时，count 对应链表第1个节点，name 对应第2个节点；重渲染时，依然按“先 count 后 name”的顺序读取，状态不会错乱。但如果破坏顺序（比如写在条件里），React 就无法匹配到正确的节点，直接报错。


## 二、核心 Hooks 原理拆解

重点拆解最常考的两个 Hooks：useState（基础）和 useEffect（高频），原理简化为“步骤化”，方便背诵。

### 1. useState 原理（最基础，必背）

通俗理解：useState 就像一个“带记忆的盒子”，第一次调用时放入初始值，之后每次调用，要么取出盒子里的当前值，要么通过 setXxx 替换盒子里的值，并且会通知组件重新渲染。

专业拆解：本质是「读取/更新 Hook 链表中对应节点的状态」，步骤分为3个阶段：

#### （1）首次渲染时

1. 创建一个新的 Hook 节点，将传入的初始值（如 0）赋值给该节点的 memoizedState。

2. 将这个 Hook 节点挂载到组件 Fiber 的 Hook 链表末尾（通过 next 指针连接）。

3. 返回一个数组`[memoizedState, setXxx]`：第一个元素是当前状态，第二个元素是触发状态更新的函数（setXxx）。

#### （2）调用 setXxx 时（触发更新）

1. 将 setXxx 传入的新值，存入对应 Hook 节点的 queue（更新队列）中。

2. 触发组件重渲染（React 会标记该组件为“待更新”，进入调度流程）。

#### （3）重渲染时

1. 按首次渲染的顺序，找到该 Hook 节点，读取其 queue 中的新值，更新 memoizedState（将旧状态替换为新状态）。

2. 再次返回最新的 `[memoizedState, setXxx]`，保证组件渲染的是最新状态。

补充：setXxx 是异步的（React 会批量处理更新），这也是为什么有时候 setState 后，立即打印状态还是旧值——因为此时更新还未执行，需在 useEffect 中读取最新状态。

### 2. useEffect 原理（高频考点，重点记依赖对比）

通俗理解：useEffect 是“副作用处理器”，用于处理组件渲染之外的操作（如请求接口、操作 DOM、监听事件），它会在组件“渲染完成后”执行，并且可以控制“什么时候重新执行”。

专业拆解：核心是「依赖对比 + 异步执行」，避免副作用干扰组件渲染，步骤同样分3个阶段：

#### （1）首次渲染时

1. 创建一个 useEffect 对应的 Hook 节点，存储「副作用回调函数」和「依赖数组」（第二个参数）。

2. 组件渲染完成后（异步执行，不会阻塞 DOM 渲染），执行副作用回调函数。

#### （2）重渲染时

1. 读取该 Hook 节点中存储的「旧依赖数组」，与本次重渲染的「新依赖数组」进行**浅对比**（对比每一项的值，基本类型比值，引用类型比地址）。

2. 若依赖有变化：先执行上一次副作用的「清理函数」（useEffect 返回的函数），再执行本次的副作用回调，最后更新 Hook 节点中的“旧依赖数组”为新依赖。

3. 若依赖无变化：直接跳过副作用的执行（性能优化，避免不必要的重复操作）。

#### （3）组件卸载时

执行该 useEffect Hook 节点的清理函数，用于清除副作用（如取消接口请求、移除事件监听），避免内存泄漏。

补充：若 useEffect 没有第二个参数（依赖数组），则每次重渲染都会执行副作用和清理函数；若依赖数组为空（[]），则只在首次渲染和组件卸载时各执行一次。


## 三、关键约束的原理支撑（为什么 Hooks 不能写在条件里？）

核心结论（必背）：**Hooks 不能写在 if、for、while 等条件判断、循环中，也不能写在 return 之后**，本质是为了保证「Hook 调用顺序固定不变」，避免 Hook 链表节点匹配错位。

通俗解读：假设把 Hook 放在 if 条件里，当条件从 true 变为 false 时，重渲染时该 Hook 就不会被调用，导致后续的 Hook 调用顺序整体前移一位，React 按原顺序遍历链表时，就会匹配到错误的节点，进而导致状态错乱、报错。

专业拆解：

1. React 源码中，是通过「遍历索引」来定位 Hook 节点的（首次渲染时记录索引，重渲染时按索引匹配），一旦调用顺序被破坏，索引对应关系就会失效。

2. 举个反例：

```jsx
function WrongComponent() {
  const [count, setCount] = useState(0);
  // 错误：Hook 写在条件里
  if (count > 0) {
    const [name, setName] = useState('React'); // 条件为 false 时，该 Hook 不执行
  }
  return <button onClick={() => setCount(count+1)}>{count}</button>;
}
```

当 count 从 0 变为 1 时，首次执行 if 里的 useState，Hook 链表有2个节点；当 count 再变为 0 时，if 条件不成立，该 Hook 不执行，重渲染时只调用1个 useState，React 按顺序遍历链表时，会试图读取第二个节点（不存在），直接抛出错误：“Hooks must be called in the exact same order in every render”。


## 四、核心总结

1. 核心结构：每个组件 Fiber 节点维护一个 Hook 单向链表，每个 Hook 节点存储 memoizedState（状态）、next（下一个节点）、queue（更新队列），靠「fiber.memoizedState」指向链表头节点。

2. 核心逻辑：首次渲染创建 Hook 节点并初始化，重渲染按固定顺序读取/更新节点状态；useState 负责状态的读取与更新，useEffect 基于依赖对比控制副作用的执行时机。

3. 核心约束：Hooks 必须在函数组件顶层调用，本质是保证调用顺序固定，避免 Hook 链表节点匹配错位，导致状态错乱。


## 五、面试常考问题及标准回答

### 1. 请说说 React Hooks 的核心原理？

标准回答：Hooks 的核心是「固定调用顺序 + Hook 链表存储」。React 为每个组件的 Fiber 节点维护一个 Hook 单向链表，每个 Hook 节点存储状态（memoizedState）、下一个节点（next）和更新队列（queue）；首次渲染时，按顺序创建 Hook 节点并挂载到链表，初始化状态；重渲染时，按相同顺序遍历链表，读取/更新对应节点的状态，保证 Hook 调用与状态一一对应。其核心目的是让函数组件拥有状态管理和生命周期能力。

### 2. 为什么 Hooks 不能写在条件判断、循环里？

标准回答：因为 Hooks 依赖「固定的调用顺序」来匹配 Hook 链表中的节点。React 无法通过变量名识别 Hooks，只能按调用顺序遍历链表、匹配状态；若写在条件/循环里，会导致组件重渲染时，Hook 调用顺序发生变化，React 无法匹配到正确的链表节点，进而导致状态错乱、抛出错误。

### 3. useState 的原理是什么？setXxx 是同步还是异步？

标准回答：useState 本质是操作组件 Fiber 节点上的 Hook 链表——首次渲染创建 Hook 节点，初始化状态并返回 [状态, setXxx]；调用 setXxx 时，将新值存入该 Hook 的更新队列，触发组件重渲染；重渲染时，按顺序读取更新队列中的新值，更新状态并返回最新结果。setXxx 是异步的，React 会批量处理更新，避免频繁渲染，因此直接在 setXxx 后打印状态，可能得到旧值。

### 4. useEffect 的依赖数组作用是什么？依赖为空（[]）和不写依赖有什么区别？

标准回答：依赖数组的作用是「控制 useEffect 副作用的执行时机」，React 会通过浅对比新旧依赖数组，判断是否执行副作用。区别：① 不写依赖数组：每次组件重渲染，都会执行副作用和清理函数；② 依赖数组为空（[]）：只在组件首次渲染时执行一次副作用，组件卸载时执行一次清理函数，相当于类组件的 componentDidMount 和 componentWillUnmount。

### 5. Hook 链表的结构是什么？fiber.memoizedState 作用是什么？

标准回答：单个 Hook 节点的结构是 { memoizedState: 状态值, next: 下一个 Hook 节点, queue: 更新队列 }，多个 Hook 节点通过 next 指针串联成单向链表。fiber.memoizedState 的作用是「指向该组件 Hook 链表的头节点」，React 通过它遍历整个 Hook 链表，读取和更新各个 Hook 的状态。

### 6. 函数组件重渲染时，Hooks 是如何“记住”上一次的状态的？

标准回答：因为状态存储在组件 Fiber 节点的 Hook 链表中，而非函数组件的局部变量（局部变量每次渲染都会重新初始化）。重渲染时，函数组件重新执行，React 会从 Hook 链表的头节点开始，按与首次渲染相同的顺序，读取每个 Hook 节点的 memoizedState，从而“记住”上一次的状态。