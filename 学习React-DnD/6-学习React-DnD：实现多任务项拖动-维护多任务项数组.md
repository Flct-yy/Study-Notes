# 一、功能承接与需求概述

上一篇文档中，我们已完成单个任务项的拖动排序功能。本次迭代的核心需求是实现**多个任务项的批量选中**，具体需达成以下目标：

-   扩展状态管理，支持存储选中的任务项集合
-   新增选中、取消选中任务项的操作逻辑
-   在任务项组件中关联单选框与选中状态的交互

# 二、核心状态扩展（TodoProvider）

要实现多任务选中与拖拽，首先需要在全局状态中新增“选中任务集合”字段，并同步更新上下文供组件调用。

## 2.1 初始化状态修改

在`initialState`中添加`selectedTodos`数组，用于存储当前选中的任务项，同时确保任务项模型的完整性：

```js
const initialState = {
  todos: [
    { id: 1, text: 'Learn React', completed: false, selected: false },
    { id: 2, text: 'Build a Todo App', completed: false, selected: false },
    { id: 3, text: 'Build a Demo', completed: true, selected: false },
    { id: 4, text: 'Fix a Bug', completed: false, selected: true },

  ],
  // 当前选中的任务数组
  selectedTodos: [{ id: 4, text: 'Fix a Bug', completed: false, selected: true }],
};
```


## 2.2 上下文同步更新

将新增的`selectedTodos`状态注入上下文，确保子组件能获取到选中任务集合：

```js
// TodoProvider内部的上下文值配置
const contextValue = {
  todos: state.todos,
  selectedTodos: state.selectedTodos,
  ...actions,
};
```

# 三、核心操作逻辑实现（Reducer）

基于需求新增两个核心操作类型：`ADD_SELECT_TODOS`（添加选中任务）、`REMOVE_SELECT_TODOS`（移除选中任务），以下是完整实现逻辑。

## 3.1 操作类型常量定义（建议单独维护）

```js
// ActionTypes.js
export const ActionTypes = {
  // 原有操作类型...
  ADD_SELECT_TODOS: 'ADD_SELECT_TODOS',
  REMOVE_SELECT_TODOS: 'REMOVE_SELECT_TODOS',
};
```

## 3.2 添加选中任务（ADD_SELECT_TODOS）

通过任务ID查找目标任务，确保任务存在且未被选中后，添加到`selectedTodos`集合中，避免重复选中：

```js
// ADD_SELECT_TODOS
case ActionTypes.ADD_SELECT_TODOS:
  {
    // 找到要添加的todo对象
    const todoToAdd = state.todos.find(todo => todo.id === action.payload.id);
    // 确保todo存在且尚未被选中（避免重复添加）
    if (todoToAdd && !state.selectedTodos.some(todo => todo.id === todoToAdd.id)) {
      return {
        ...state,
        selectedTodos: [...state.selectedTodos, todoToAdd],
      };
    }
    return state;
  }
```

## 3.3 移除选中任务（REMOVE_SELECT_TODOS）

根据任务ID从`selectedTodos`集合中过滤掉目标任务：

```js
// REMOVE_SELECT_TODOS
case ActionTypes.REMOVE_SELECT_TODOS:
  return {
    ...state,
    selectedTodos: state.selectedTodos.filter(todo => todo.id !== action.payload.id),
  };
```

# 四、任务项组件交互（TodoItem）

在TodoItem组件中，通过Ref绑定单选框，并在单选框状态变化时，触发选中/取消选中的操作，实现视图与状态的同步。

## 4.1 组件核心逻辑

```js
import { useContext, useRef } from 'react';
import useTodoContext from '@/context/TodoContext/useTodoContext';

const TodoItem = ({ todo }) => {
  const { addSelectTodos, removeSelectTodos } = useTodoContext();
  // 用Ref绑定单选框元素，便于后续操作（如主动获取状态）
  const checkboxRef = useRef(null);

  // 单选框状态变化处理函数
  const handleCheckboxChange = (e) => {
    toggleSelected(todo.id);
    if (e.target.checked) {
      // 选中：调用添加选中任务的action
      addSelectTodos(todo.id);
    } else {
      // 取消选中：调用移除选中任务的action
      removeSelectTodos(todo.id);
    }
  };

  return (
    <div className={`todo-item${isDragging ? ' isDragging' : ''}`} ref={divRef}>
      <div className="todo-item-content">
        <input
          type="checkbox"
          id={`todo-${todo.id}`}
          checked={todo.selected}
          onChange={handleCheckboxChange}
          className="todo-checkbox"
          ref={checkboxRef}
        />
      </div>
      {/* 原有单个任务拖拽相关逻辑 */}
    </div>
  );
};

export default TodoItem;
```

注意：仅保留必需内容，不包含TodoItem组件的所有内容。

## 4.2 关键交互说明

-   **Ref绑定**：通过`checkboxRef`可在需要时主动控制单选框（如全选/取消全选功能），提升组件灵活性。
-   **状态双向绑定**：单选框的`checked`属性直接绑定任务项的`selected`状态，确保视图与全局状态一致。
-   **操作触发**：状态变化时通过上下文获取的`addSelectTodos`和`removeSelectTodos`方法更新全局状态，实现跨组件状态同步。

# 五、配套Action函数实现

为了让组件更便捷地调用上述操作，需在TodoProvider中定义对应的action函数，并注入上下文：

```js
const actions = {
  // 原有action...
  
    // 添加选中任务
    addSelectTodos: (id) => {
      dispatch({
        type: ActionTypes.ADD_SELECT_TODOS,
        payload: { id },
      });
    },

    // 移除选中任务
    removeSelectTodos: (id) => {
      dispatch({
        type: ActionTypes.REMOVE_SELECT_TODOS,
        payload: { id },
      });
    },
    
};
```

# 六、测试要点

1.  单个任务选中/取消：勾选单选框后，`selectedTodos`应同步增减，任务项`selected`状态正确。
1.  多个任务选中：选中多个任务后，`selectedTodos`应包含所有选中项，无重复数据。


