三、Fiber（重点，必须）
必须掌握的 5 个字段（不用全背）
fiber = {
  type,
  key,
  stateNode,
  child,
  sibling,
  return,
  alternate,
  flags
}

你要理解的不是“字段”，而是：

为什么是链表结构

为什么 child / sibling / return

为什么可以中断

面试回答模板（记住）

Fiber 是一个可中断的工作单元
用链表而不是递归树，是为了：

支持时间切片

支持优先级

支持中断和恢复