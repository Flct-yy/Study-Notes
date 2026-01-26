五、Render 阶段 vs Commit 阶段（必考）
Render 阶段（可中断）

beginWork

completeWork

生成 effects（flags）

👉 不碰 DOM

Commit 阶段（不可中断）

beforeMutation

mutation

layout

👉 真正操作 DOM

面试一句话总结（背）

Render 阶段是纯计算，可以被打断
Commit 阶段是副作用执行，必须一次完成