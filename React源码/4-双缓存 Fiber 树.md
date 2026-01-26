四、双缓存 Fiber 树（面试必考）
你要知道

current Fiber Tree

workInProgress Fiber Tree

alternate 指针

核心一句话

React 同时维护两棵 Fiber 树
更新时在 workInProgress 上计算
提交后切换指针

面试官追问你可以这样接

这和浏览器的双缓冲渲染思想类似，避免边算边改 UI