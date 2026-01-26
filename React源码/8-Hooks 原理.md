八、Hooks 原理（面试高频）
必须掌握的 3 个点

hooks 按顺序存

链表结构

为什么不能写在条件里

极简模型（够面试）
fiber.memoizedState = {
  memoizedState,
  next
}

面试这样说就稳了

Hooks 依赖调用顺序
React 通过链表 + 当前指针读取状态
条件调用会破坏顺序