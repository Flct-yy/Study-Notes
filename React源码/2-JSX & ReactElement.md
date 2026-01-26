二、JSX & ReactElement（低成本高收益）
你要知道的点

JSX 编译后是什么

ReactElement 是什么数据结构

ReactElement ≠ DOM ≠ Fiber

面试够用理解
const el = <App name="x" />


≈

const el = {
  $$typeof: Symbol(react.element),
  type: App,
  props: { name: 'x' },
  key: null,
  ref: null
}

面试能这样说 ✔

JSX 只是语法糖
ReactElement 是一次“UI 描述快照”
真正参与更新的是 Fiber