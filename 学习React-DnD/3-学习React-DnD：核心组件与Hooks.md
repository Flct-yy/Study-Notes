---
theme: channing-cyan
---
上一篇我们完成了React-DnD的环境搭建，通过安装依赖和全局注入后端，让整个应用具备了拖放能力的基础。这一篇，我们将深入React-DnD的核心——那些支撑起拖放功能的组件和Hooks。从全局管理的DndProvider，到定义拖动源的useDrag、拖放目标的useDrop，每一个都至关重要。掌握它们，你就能轻松实现各种复杂的拖放场景。

在开始之前，先明确一个核心逻辑：React-DnD通过“组件提供上下文 + Hooks连接组件”的模式工作。DndProvider作为上下文提供者，为所有子组件传递拖放能力；而useDrag、useDrop等Hooks则负责将普通组件“改造”为拖动源或拖放目标，实现具体的交互逻辑。下面我们逐个拆解。

# 一、核心组件：拖放能力的“基石”

React-DnD的组件数量不多，但每一个都是构建拖放功能的关键。其中DndProvider是必用组件，DragPreviewImage则用于优化拖动体验，我们重点讲解这两个。

## 1. DndProvider：拖放上下文的“提供者”

如果把React-DnD的拖放能力比作“水电”，那么DndProvider就是“水电总闸”。它负责将拖放后端的能力注入到整个应用，让所有子组件都能共享这份能力。上一篇我们已经在入口文件中用过它，现在来深入理解它的核心作用和配置项。

### 核心作用

DndProvider的本质是一个React上下文（Context）的提供者，它会创建一个拖放上下文，并将后端（如HTML5Backend）的功能传递给所有子组件。这样一来，子组件通过useDrag、useDrop等Hooks就能直接获取拖放能力，无需单独配置后端。

如果不使用DndProvider包裹应用，后续编写拖动源或拖放目标时会直接报错——组件找不到拖放上下文，就像没接水电的房子无法使用电器一样。

### 关键配置项

DndProvider的配置项不多，但每一个都有明确的用途，其中backend是必填项，其他为可选项。

-   **backend（必填）** ：React-DnD的后端引擎，负责处理原生DOM事件（如鼠标拖动、悬停），并将其转化为React-DnD能识别的逻辑。我们开发PC端应用时，基本都使用官方提供的react-dnd-html5-backend；如果是移动端，则可以使用react-dnd-touch-backend。
-   **context（可选）** ：用于配置后端的上下文对象，具体用法取决于你使用的后端实现。一般情况下，使用默认配置即可，无需额外设置。
-   **options（可选）** ：用于配置后端的选项对象，同样依赖于后端实现。例如，某些后端支持配置拖动的延迟时间、触摸反馈等，都可以通过这个参数传递。

### 实战示例（回顾与强化）

在入口文件src/index.js中，我们用DndProvider包裹整个App组件，并注入HTML5Backend。这里再强调一下核心代码的逻辑：

```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
// 导入 DndProvider 和 HTML5Backend
import { DndProvider } from 'react-dnd';
import { HTML5Backend } from 'react-dnd-html5-backend';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  // 用DndProvider包裹App，注入后端
  <DndProvider backend={HTML5Backend}>
    <App />
  </DndProvider>
);
```

这里的关键是将HTML5Backend作为backend属性传递给DndProvider，这样整个应用的子组件都能使用拖放能力了。

## 2. DragPreviewImage：自定义拖动预览的“工具”

默认情况下，拖动元素时，浏览器会生成一个该元素的“快照”作为拖动预览。但在实际开发中，我们可能需要自定义预览效果（比如拖动时显示一个简化的图标，而不是整个元素），这时候就需要用到DragPreviewImage组件。

### 核心作用

DragPreviewImage组件用于将一张HTML图像元素（img）渲染为拖动时的预览效果，替代浏览器默认的预览快照。它需要配合useDrag钩子的拖动预览连接器使用。

### 关键配置项

DragPreviewImage只有一个必填配置项：connect。

-   **connect（必填）** ：拖动预览的连接器函数，来自useDrag钩子的返回值。它的作用是将自定义的预览图像与拖动操作关联起来，让浏览器在拖动时显示这张图像。

### 实战示例

下面的例子中，我们创建一个可拖动的任务卡片，拖动时显示一张自定义的预览图片：

```js
import React from 'react';
import { useDrag } from 'react-dnd';
import { DragPreviewImage } from 'react-dnd';
// 导入自定义预览图片
import taskPreview from './task-preview.png';

function TaskCard({ id, title }) {
  // 从useDrag中获取拖动预览连接器
  const [, dragSourceRef, dragPreviewRef] = useDrag({
    type: 'TASK',
    item: { id, title }
  });

  return (
    <div ref={dragSourceRef} style={{ padding: 16, border: '1px solid #ccc', margin: 8 }}>
      {/* 关联自定义预览图片 */}
      <DragPreviewImage connect={dragPreviewRef} src={taskPreview} />
      {title}
    </div>
  );
}

export default TaskCard;
```

这里的核心逻辑是：从useDrag的返回值中获取dragPreviewRef（拖动预览连接器），然后将其传递给DragPreviewImage的connect属性，同时通过src属性指定自定义预览图片的路径。这样，拖动TaskCard时，就会显示taskPreview.png这张图片作为预览效果。

# 二、核心Hooks：拖放交互的“实现者”

如果说组件是React-DnD的“骨架”，那么Hooks就是“肌肉”——它们负责实现具体的拖放交互逻辑。React-DnD提供了多个实用Hooks，其中useDrag（定义拖动源）、useDrop（定义拖放目标）是最常用的两个，useDragLayer和useDragDropManager则用于更复杂的场景。

## 1. useDrag：让组件成为“拖动源”

useDrag是将普通React组件转化为“拖动源”的核心钩子。通过向它传递一个规范对象（spec），我们可以声明性地描述拖动源的类型、拖动的数据、拖动过程中的回调等。

### 基本用法：参数与返回值

useDrag的用法可以总结为“传入spec配置，返回三个核心对象”，具体如下：

```js
const [collectedProps, dragSourceRef, dragPreviewRef] = useDrag(spec, deps);
```

#### 参数说明

-   **spec（必填）** ：规范对象或返回规范对象的函数，用于配置拖动源的核心逻辑。这是useDrag的核心，我们后面会详细拆解其成员。
-   **deps（可选）** ：依赖关系数组，用于备忘录化（类似React的useMemo）。如果spec是函数，deps默认是空数组；如果spec是对象，deps默认是包含spec的数组。当deps中的值发生变化时，useDrag会重新计算spec配置。

#### 返回值说明

useDrag返回一个包含三个元素的数组，每个元素都有明确的用途：

-   **collectedProps（索引0）** ：从collect函数中收集的属性对象。collect函数用于从拖放监控器（monitor）中获取拖动状态（如是否正在拖动），并将其转化为组件的props。如果没有定义collect函数，返回空对象。
-   **dragSourceRef（索引1）** ：拖动源的连接器函数，必须绑定到组件的DOM元素上。它的作用是告诉React-DnD“哪个元素是可拖动的”，如果不绑定，组件将无法被拖动。
-   **dragPreviewRef（索引2）** ：拖动预览的连接器函数，用于关联自定义的拖动预览（如DragPreviewImage组件）。如果不需要自定义预览，可以忽略它。

### 核心：spec规范对象详解

spec对象是useDrag的灵魂，它定义了拖动源的所有行为。其中type和item是必填项，其他为可选项。

#### 必填成员

-   **type（必填）** ：字符串或符号（Symbol），用于标识拖动源的类型。只有注册了相同类型的拖放目标（useDrop），才会对该拖动源的拖放操作做出反应。这是React-DnD实现“拖动源与目标匹配”的核心机制。例如，我们可以将任务卡片的type设为'TASK'，将任务列表的accept设为'TASK'，这样任务卡片就能拖放到任务列表中。
-   **item（必填）** ：描述拖动数据的对象，或返回该对象的函数。这是拖动源传递给拖放目标的“核心数据”，也是两者之间唯一的通信桥梁。 如果是对象，应只包含拖放目标需要的最小数据（如id、名称），避免传递复杂引用（比如整个组件实例），否则会导致拖动源和目标过度耦合。
-   如果是函数，会在拖动操作开始时执行，并返回上述对象。如果返回null，拖动操作会被取消。

#### 可选成员（回调与配置）

**end(item, monitor)（可选）** ：拖动操作结束时触发的回调函数，无论拖动是否成功（比如拖到目标后释放，或拖到无效区域释放），都会执行。
-   item：拖动的核心数据（与spec.item一致）。

-   monitor：拖放监控器，用于获取拖动状态（如monitor.didDrop()可以判断拖放是否被目标接受，monitor.getDropResult()可以获取目标返回的结果）。

-   常用场景：拖动结束后更新数据（如将任务从“待办”列表移到“已办”列表）。

**canDrag(monitor)（可选）** ：用于判断当前组件是否允许被拖动。返回true则允许拖动，返回false则禁止。 monitor：拖放监控器，可以通过monitor.getItem()获取拖动数据，结合组件props判断是否允许拖动（如某些任务卡片不允许被拖动）。

注意： 不能在该函数中调用monitor.canDrag()，否则会导致死循环。

**isDragging(monitor)（可选）** ：用于自定义“是否正在拖动”的判断逻辑。默认情况下，只有启动拖动的组件会被视为“正在拖动”。 常用场景：当有多个相同类型的组件时（如多个任务卡片），通过item.id与组件props.id对比，确保只有当前拖动的组件显示“拖动中”的样式。

示例：return monitor.getItem().id === props.id;

**collect(monitor, props)（可选）** ：收集函数，用于从监控器中获取拖动状态，并转化为组件的props。返回的对象会作为useDrag的第一个返回值（collectedProps）传递给组件。 
 
常用监控器方法：monitor.isDragging()（是否正在拖动）、monitor.getInitialClientOffset()（拖动开始时的鼠标位置）等。

示例：(monitor) => ({ isDragging: monitor.isDragging() })

### useDrag实战：可拖动的任务卡片

结合上面的知识点，我们实现一个完整的可拖动任务卡片组件，包含拖动状态判断、拖动结束回调等功能：

```js
// DraggableTask.jsx
import React from 'react';
import { useDrag } from 'react-dnd';

// 定义拖动类型（建议用Symbol避免冲突）
export const TASK_TYPE = Symbol('TASK');

function DraggableTask({ id, title, onDragEnd }) {
  const [collectedProps, dragSourceRef] = useDrag({
    // 拖动类型
    type: TASK_TYPE,
    // 拖动数据（只传递必要的id和title）
    item: () => ({ id, title }),
    // 拖动结束回调
    end: (item, monitor) => {
      // 判断拖放是否被目标接受
      if (monitor.didDrop()) {
        // 获取目标返回的结果（如目标列表的id）
        const dropResult = monitor.getDropResult();
        // 调用父组件方法更新数据
        onDragEnd(item.id, dropResult.listId);
      }
    },
    // 收集拖动状态
    collect: (monitor) => ({
      isDragging: monitor.isDragging()
    }),
    // 只有id为偶数的任务可以拖动（示例）
    canDrag: () => id % 2 === 0
  });

  // 根据拖动状态设置样式（拖动时半透明）
  const cardStyle = {
    padding: 16,
    border: '1px solid #ccc',
    margin: 8,
    opacity: collectedProps.isDragging ? 0.5 : 1,
    cursor: collectedProps.isDragging ? 'grabbing' : 'grab'
  };

  return <div ref={dragSourceRef} style={cardStyle}>{title}</div>;
}

export default DraggableTask;
```

这个例子中，我们实现了以下功能：

-   只有id为偶数的任务卡片可以被拖动（canDrag配置）。
-   拖动时卡片显示半透明效果（通过collect获取isDragging状态，动态设置opacity）。
-   拖动结束后，根据拖放结果调用父组件的onDragEnd方法更新数据（end回调）。

## 2. useDrop：让组件成为“拖放目标”

useDrag负责“发起”拖放，useDrop则负责“接收”拖放——它将普通组件转化为“拖放目标”，用于接收拖动源传递的数据，并处理拖放相关的逻辑（如悬停、接收拖放）。

### 基本用法：参数与返回值

useDrop的用法与useDrag类似，都是“传入spec配置，返回核心对象”，具体如下：

```js
const [collectedProps, dropTargetRef] = useDrop(spec, deps);
```

#### 参数说明

-   **spec（必填）** ：规范对象或返回规范对象的函数，用于配置拖放目标的核心逻辑，是useDrop的核心。
-   **deps（可选）** ：依赖关系数组，作用与useDrag的deps一致，用于备忘录化。

#### 返回值说明

useDrop返回一个包含两个元素的数组：

-   **collectedProps（索引0）** ：从collect函数中收集的属性对象，与useDrag的collectedProps类似，用于获取拖放状态（如是否有元素悬停在目标上）。
-   **dropTargetRef（索引1）** ：拖放目标的连接器函数，必须绑定到组件的DOM元素上，告诉React-DnD“哪个元素是拖放目标”。

### 核心：spec规范对象详解

useDrop的spec对象与useDrag类似，但核心关注点是“接收拖放”，其中accept是必填项。

#### 必填成员

-   **accept（必填）** ：用于指定当前拖放目标可以接受的拖动源类型，与useDrag的type对应。它可以是字符串、符号，也可以是包含多个类型的数组。 
    -   示例：`accept: TASK_TYPE`（接受类型为TASK_TYPE的拖动源）。
    -   示例：`accept: [TASK_TYPE, PROJECT_TYPE]`（接受两种类型的拖动源）。

#### 可选成员（回调与配置）

**drop(item, monitor)（可选）** ：当兼容类型的拖动源在目标上**释放时**触发的回调函数，是处理拖放逻辑的核心。
-   item：拖动源传递的核心数据（与useDrag的spec.item一致）。
-   monitor：拖放监控器，可以通过monitor.isOver({ shallow: true })判断是否是直接悬停（而非嵌套目标）。
-   返回值：可以返回一个对象，该对象会作为拖放结果，通过monitor.getDropResult()传递给拖动源的end回调。
-   常用场景：接收拖动的任务数据，将其添加到当前列表中。

**hover(item, monitor)（可选）** ：当拖动源悬停在目标上时**持续**触发的回调函数（即使鼠标不动也会触发）。 常用场景：实现“拖入时高亮目标”“拖动排序”等交互（如在列表中拖动任务时，调整任务的位置）。

注意：即使canDrop返回false，该函数也会触发，可以通过monitor.canDrop()判断当前是否允许接收拖放。

**canDrop(item, monitor)（可选）** ：用于判断当前目标是否允许接收该拖动源的数据。返回true则允许，返回false则禁止。 示例：根据拖动源的id判断是否允许接收（如禁止将任务拖放到自己所在的列表）。

注意：不能在该函数中调用monitor.canDrop()。

**collect(monitor, props)（可选）** ：收集函数，用于从监控器中获取拖放状态，转化为组件的props。 常用监控器方法：monitor.isOver()（是否有元素悬停）、monitor.canDrop()（是否允许接收拖放）等。

示例：(monitor) => ({ isOver: monitor.isOver(), canDrop: monitor.canDrop() })

### useDrop实战：可接收任务的列表

结合useDrag的任务卡片，我们实现一个可接收任务的列表组件，包含悬停高亮、接收任务等功能：

```js
// TaskList.jsx
import React from 'react';
import { useDrop } from 'react-dnd';
import { TASK_TYPE } from './DraggableTask';

function TaskList({ id, title, tasks, onAddTask }) {
  const [collectedProps, dropTargetRef] = useDrop({
    // 接受TASK_TYPE类型的拖动源
    accept: TASK_TYPE,
    // 接收拖放时的回调
    drop: (item, monitor) => {
      // 调用父组件方法，将任务添加到当前列表
      onAddTask(id, item);
      // 返回拖放结果，传递给拖动源
      return { listId: id };
    },
    // 悬停时的回调
    hover: (item) => {
      // 可以在这里实现拖动排序逻辑（如调整任务在列表中的位置）
      console.log(`任务 ${item.id} 悬停在 ${title} 列表上`);
    },
    // 收集拖放状态
    collect: (monitor) => ({
      isOver: monitor.isOver(),
      canDrop: monitor.canDrop()
    })
  });

  // 根据悬停状态和是否允许拖放设置样式
  const listStyle = {
    padding: 16,
    border: collectedProps.isOver && collectedProps.canDrop 
      ? '2px solid #2196F3' 
      : '1px solid #eee',
    margin: 16,
    minHeight: 200
  };

  return (
    <div ref={dropTargetRef} style={listStyle}>
      <h3>{title}</h3>
      {tasks.map(task => (
        <div key={task.id} style={{ padding: 8, borderBottom: '1px solid #eee' }}>
          {task.title}
        </div>
      ))}
    </div>
  );
}

export default TaskList;
```

这个例子中，我们实现了以下功能：

-   列表只接受TASK_TYPE类型的拖动源（accept配置）。
-   当任务卡片悬停在列表上且允许拖放时，列表边框变为蓝色高亮（通过collect获取isOver和canDrop状态）。
-   接收任务卡片后，调用父组件的onAddTask方法将任务添加到当前列表，并返回列表id给拖动源。

## 3. 其他实用Hooks：应对复杂场景

除了useDrag和useDrop，React-DnD还提供了两个用于复杂场景的Hooks：useDragLayer和useDragDropManager。

### useDragLayer：自定义全局拖动层

当需要实现超越单个组件的拖动预览（如拖动时显示一个覆盖整个页面的提示）时，useDragLayer就派上用场了。它可以创建一个独立于拖动源和目标的“全局拖动层”，不受其他组件的样式影响。

#### 核心用法

useDragLayer只接收一个必填参数collect（收集函数），返回从collect函数中获取的属性对象。collect函数的作用是从监控器中获取拖动状态，用于渲染拖动层的内容。

```js
// CustomDragLayer.jsx
import React from 'react';
import { useDragLayer } from 'react-dnd';

function CustomDragLayer() {
  // 收集拖动状态
  const { item, isDragging } = useDragLayer(monitor => ({
    item: monitor.getItem(),
    isDragging: monitor.isDragging()
  }));

  // 没有拖动时不渲染
  if (!isDragging) return null;

  // 拖动时显示自定义提示
  return (
    <div style={{
      position: 'fixed',
      zIndex: 9999,
      pointerEvents: 'none',
      left: 0,
      top: 0,
      width: '100%',
      height: '100%',
      display: 'flex',
      alignItems: 'center',
      justifyContent: 'center'
    }}>
      <div style={{ padding: 20, backgroundColor: 'rgba(0,0,0,0.7)', color: 'white' }}>
        正在拖动：{item.title}
      </div>
    </div>
  );
}

export default CustomDragLayer;
```

这个自定义拖动层会在拖动时显示一个居中的提示框，显示当前拖动的任务标题，且不会影响其他组件的交互（pointerEvents: 'none'确保点击事件能穿透到下层组件）。

### useDragDropManager：获取拖放管理器实例

DragDropManager是React-DnD的核心单例对象，包含了拖放系统的状态、监控器、后端等核心资源。useDragDropManager钩子用于获取这个实例，一般用于自定义后端或高级扩展场景（如手动触发拖放事件）。

#### 基本用法

```js
import { useDragDropManager } from 'react-dnd';

function AdvancedComponent() {
  const manager = useDragDropManager();
  // 可以通过manager获取监控器、后端等资源
  const monitor = manager.getMonitor();

  // 高级用法：手动监听拖动状态变化
  React.useEffect(() => {
    const unsubscribe = monitor.subscribeToStateChange(() => {
      console.log('拖放状态变化：', monitor.isDragging());
    });
    return () => unsubscribe();
  }, [monitor]);

  return <div>高级扩展组件</div>;
}
```

对于大多数普通开发场景，我们很少会直接使用useDragDropManager，除非需要深度定制React-DnD的行为。


# 三、总结

这一篇我们深入讲解了React-DnD的核心组件和Hooks，核心要点总结如下：

-   **组件**：DndProvider是基础（提供拖放上下文），DragPreviewImage用于自定义预览。
-   **Hooks**：useDrag定义拖动源，useDrop定义拖放目标，两者通过type和accept匹配；useDragLayer用于全局预览，useDragDropManager用于高级扩展。
-   **核心逻辑**：拖动源通过item传递数据，拖放目标通过drop接收数据，两者通过monitor实现状态通信。

掌握了这些内容，你已经能实现大多数常见的拖放场景了。