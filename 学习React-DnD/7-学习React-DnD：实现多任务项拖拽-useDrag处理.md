---
theme: channing-cyan
---

在完成任务项的选中与取消选中功能后，核心需求升级为“**选中状态下拖拽触发批量移动，未选中时保持单个拖拽**”，同时需实现“所有选中项同步透明”的视觉反馈。本文将基于React DnD的`useDrag`钩子，完整拆解这一功能的实现逻辑。

# 一、核心设计思路

多选拖拽的核心是“状态识别”与“数据联动”，需解决两个关键问题：

1.  **拖拽模式判断**：通过任务项的选中状态（`todo.selected`）和选中集合（`selectedTodos`），区分“单个拖拽”和“多选拖拽”。
2.  **批量状态同步**：拖拽任意一个选中项时，让所有选中项同步响应拖拽状态（如透明效果），而非仅拖拽触发项。

最终实现效果如示例所示：勾选多个任务项后，拖拽其中任意一项，所有选中项均变为透明并同步移动。

![alt text](SelectTodosDrag.gif)

# 二、关键技术点：React DnD useDrag 配置优化

拖拽功能的核心是`useDrag`钩子的配置，我们需要通过修改`item`（拖拽数据）和`isDragging`（拖拽状态判断）两个关键属性，实现模式切换与状态同步。

## 2.1 拖拽数据（item）：携带模式标识

拖拽时传递的数据需包含“当前任务信息”和“选中集合（可选）”，让接收方（`useDrop`）能识别拖拽模式。当任务项处于选中状态且存在选中集合时，携带`selectedTodos`标识为多选拖拽。
```js
    // TodoItem组件中useDrag的item配置
    item: () => {
      // 构建拖拽数据，区分单个/多选拖拽
      return {
        id: todo.id,               // 当前任务ID（必传，单个拖拽核心标识）
        index: index,              // 当前任务在列表中的原始索引
        selected: todo.selected,   // 当前任务的选中状态
        // 关键：仅选中状态且存在选中集合时，携带选中列表
        selectedTodos: todo.selected && selectedTodos ? selectedTodos : undefined
      };
    },
```
设计亮点：通过`selectedTodos`的“存在性”作为模式判断依据，无需额外定义“type”字段，简化数据结构的同时保证语义清晰。

## 2.2 拖拽状态（isDragging）：批量同步状态

默认的`isDragging`仅判断“当前项是否为拖拽触发项”，无法满足多选场景。我们需要自定义判断逻辑：**多选模式下，所有选中项均视为“正在拖拽”** ，从而同步视觉效果。
```js
    // TodoItem组件中useDrag的isDragging配置
    isDragging: (monitor) => {
      const item = monitor.getItem();
      // 安全校验：避免拖拽数据为空时的异常
      if (!item) return false;

      // 场景1：单个拖拽（无选中集合或当前项未选中）
      if (!item.selectedTodos || !item.selected) {
        // 仅当拖拽数据ID与当前项ID一致时，视为拖拽中
        return item.id === todo.id;
      }

      // 场景2：多选拖拽（存在选中集合且当前项已选中）
      // 检查当前项是否在选中集合中，是则视为拖拽中
      return item.selectedTodos.some(
        selectedTodo => selectedTodo && selectedTodo.id === todo.id
      );
    },
```
逻辑拆解：

*   安全校验：优先判断`item`是否存在，避免`monitor.getItem()`返回空值导致的报错。
*   单个拖拽判断：无`selectedTodos`时，沿用默认逻辑（ID匹配即视为拖拽中）。
*   多选拖拽判断：通过`some()`方法遍历选中集合，只要当前项在集合中，就标记为“拖拽中”，实现批量状态同步。

# 三、视觉效果联动：选中项透明化

通过`isDragging`返回的布尔值，直接关联组件样式，实现“拖拽中选中项透明”的视觉反馈，这也是提升用户体验的关键一步。
```js
    // TodoItem组件样式与拖拽状态联动
    import { useDrag } from 'react-dnd';

    export default function TodoItem({ todo, index }) {
      // 省略useDrag其他配置...
      const [{ isDragging }, drag] = useDrag({
        item: /* 上文配置 */,
        isDragging: /* 上文配置 */,
        // 其他属性（如end：拖拽结束后的排序逻辑）
      });

      // 拖拽状态样式：选中项拖拽时透明化
      const todoItemStyle = {
        opacity: isDragging ? 0.5 : 1, // 核心视觉反馈
        transition: 'opacity 0.2s ease', // 平滑过渡提升体验
        // 其他基础样式（如padding、border等）
      };

      return (
        <div className={`todo-item${isDragging ? ' isDragging' : ''}`} ref={divRef}>
            {/* 其他 */}
        </div>
      );
    };
```
效果说明：当拖拽任意一个选中项时，所有选中项的`isDragging`均变为`true`，`opacity`同步变为0，实现示例中的透明效果；拖拽结束后，`isDragging`重置为`false`，样式恢复正常。

# 四、功能总结

本次实现通过修改`useDrag`的核心配置，仅用“数据携带标识+状态批量判断”的轻量方案，完成了单个/多选拖拽的无缝切换，同时通过样式联动提升了用户体验。核心亮点：

*   无侵入式设计：不修改原有单个拖拽逻辑，通过条件判断扩展多选能力。
*   状态自洽：依赖选中集合动态计算状态，避免局部状态与全局状态的不一致。
*   视觉反馈清晰：选中项同步透明，让用户明确感知批量拖拽的作用范围。
