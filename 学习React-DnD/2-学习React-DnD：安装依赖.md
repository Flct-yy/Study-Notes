上一篇我们用“待办清单”场景吃透了React-DnD的7个核心概念——从“拖动源”到“拖放后端”，每个角色的分工都清晰明了。这一篇就进入配置环节：把这些概念落地到代码里的第一步，就是搭建好环境。别担心，整个过程只有“安装依赖”和“全局注入”两步，新手也能一次成功。

## 一、环境配置核心逻辑：先备齐“工具”

回顾上一篇提到的“拖放后端”概念——它是处理原生DOM事件的“底层引擎”，而React-DnD的核心库需要和后端配合才能工作。所以环境配置的本质，就是“装齐核心库和后端”，再“把后端注入整个应用”，让所有组件都能用上拖放能力。

我们需要准备两个核心“工具”：

-   `react-dnd`：React-DnD的核心库，提供“拖动源”“拖放目标”等所有核心API。
-   `react-dnd-html5-backend`：PC端默认的拖放后端，把鼠标拖动、悬停等原生事件转化为React-DnD能识别的逻辑，开箱即用。

## 二、第一步：安装依赖（20秒完成）

### 1. 打开终端，定位项目目录

首先确保你已经创建了React项目（如果还没创建，先执行`npx create-react-app react-dnd-demo`），然后在终端进入项目根目录：

```bash
cd react-dnd-demo
```


### 2. 执行安装命令

根据你使用的包管理器选择对应命令，二选一即可：

```bash
# 使用npm安装
npm install react-dnd react-dnd-html5-backend

# 或使用yarn安装（需提前安装yarn）
yarn add react-dnd react-dnd-html5-backend
```

安装成功的标志：项目的`package.json`文件中，`dependencies`字段会新增这两个包及其版本号，比如：

```json
"dependencies": {
    "react-dnd": "^16.0.1",
    "react-dnd-html5-backend": "^16.0.1",
    // 其他依赖...  
    }
```

常见问题：如果安装失败，可能是这两个原因：

-   网络问题：可尝试切换npm镜像（`npm config set registry https://registry.npmmirror.com`）后重新安装。
-   React版本不兼容：react-dnd v16+对应React 18+，如果你的React是17及以下，需安装低版本（如`npm install react-dnd@15 react-dnd-html5-backend@15`）。

## 三、第二步：全局注入后端（1分钟完成）

安装完成后，需要让整个应用“认识”这个拖放后端——这就需要用上React-DnD的`DndProvider`组件，在项目入口文件中包裹整个应用，把后端注入进去。

### 1. 找到入口文件

React项目的默认入口文件是`src/index.js`，所有组件的渲染都从这里开始，在这里注入后端能确保所有子组件都能访问拖放上下文。

### 2. 编写注入代码

核心步骤有3个：导入DndProvider和HTML5Backend → 用DndProvider包裹App组件 → 传入backend属性指定后端。完整代码如下（重点已标注）：

```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
// 1. 导入React-DnD核心组件和HTML5后端
import { DndProvider } from 'react-dnd';
import { HTML5Backend } from 'react-dnd-html5-backend';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  // 2. 用DndProvider包裹整个App，注入后端
  <DndProvider backend={HTML5Backend}>
    <App />
  </DndProvider>
);
```

### 3. 关键说明：为什么要这么做？

这一步对应上一篇“拖放后端”的概念：

-   `DndProvider`就像一个“提供者”，负责把拖放后端的能力传递给所有子组件，让组件不用单独配置就能使用`useDrag`、`useDrop`等钩子。
-   如果不包裹DndProvider，后续写拖动源或拖放目标时会直接报错——组件找不到拖放上下文。

## 四、验证环境：确保配置生效

配置完成后别着急写功能，先启动项目验证环境是否正常。在终端执行启动命令：

```bash
npm start
```

如果出现这两种情况，说明环境配置成功：

1.  项目正常启动，浏览器打开后没有“React-DnD”相关的报错（如“Cannot find context”）。
1.  在`App.js`中尝试导入React-DnD的API，不会提示“模块找不到”，比如：

```js
// 在App.js中测试导入
import { useDrag } from 'react-dnd';

function App() {
  return <div className="App">环境配置成功！</div>;
}

export default App;
```
