六、JS 为什么会阻塞渲染
你要能说清

JS 在渲染进程里执行

JS 可能修改 DOM / CSSOM

浏览器必须“停下来等 JS”

关键词

script 标签阻塞解析

async / defer 区别

preload / prefetch（知道即可）