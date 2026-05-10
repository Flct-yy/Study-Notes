# 前端工程化 Mock 数据原理与实践

在前端工程化体系中，Mock 数据是连接“前端数据模型”与“后端 API 接口”的核心桥梁。前端开发需依赖特定数据模型组织页面与组件交互流程，而该数据模型又高度依赖后端提供的 API 接口——在后端实际 API 功能完成之前，若能获得与真实接口高度一致的模拟数据，就能提前处理前端交互中的数据逻辑，待后端开发完成进入联调阶段后，再无缝切换至真实后端服务接口，这正是 Mock 数据的核心价值所在。

当前前端工程化已进入 TypeScript 主流普及、现代架构（Next.js RSC、Edge Runtime、Monorepo）常态化的阶段，传统 Mock 方案（如停止维护的 Mock.js）已逐渐无法适配新时代需求。本文将从基础原理、主流方案、现代实践、架构适配等维度，结合通俗解读与专业实操，全面拆解 当前前端工程化中的 Mock 数据应用，帮你避开选型坑，实现 Mock 数据的规范化、高效化、安全化落地。

# 一、Mock 核心定位与业务价值

## 1.1 前端与后端的依赖困境

前端开发的核心逻辑围绕“数据模型”展开：页面渲染、组件交互、状态管理，均需依赖后端 API 返回的数据结构。若等待后端 API 完全开发完成再启动前端开发，会导致前后端开发串行，大幅延长项目迭代周期；若前端提前开发，又会因缺少真实数据支撑，无法调试交互逻辑、验证边界场景。

Mock 数据的核心作用，就是打破这种依赖困境，为前端提供“虚拟但高仿真”的接口返回，让前后端实现并行开发，同时提前覆盖异常场景，降低联调返工成本。

## 1.2 四类原始 Mock 实现方式对比

在工程化规范普及前，前端开发者常用的 Mock 方式各有优劣，结合当前现状，其适用场景已发生明显变化：

- **侵入式硬编码静态数据**：直接在业务代码中写入固定 JSON 数据，用于临时调试。优点是零配置、快速上手；缺点是侵入业务代码、可维护性差、切换真实接口时需逐行删除，仅适用于单个组件的临时调试，不符合工程化规范。
- **后端自建 Mock 服务**：后端在自身服务中，对未实现的接口返回 Mock 数据，前端请求直接指向后端服务。优点是 Mock 数据与真实接口逻辑、格式完全一致，联调时无需切换请求地址；缺点是依赖后端开发进度，无法实现前端独立开发，且后端需额外投入成本维护 Mock 逻辑，适用于后端开发进度较快、前后端协作紧密的团队。
- **本地规则 Mock 工具**：通过本地工具（如 Vite Plugin Mock）配置规则，生成 Mock 数据并拦截前端请求。优点是前端可独立控制、配置灵活；缺点是需前端维护 Mock 规则，跨团队协作时规则同步成本高，适用于中小型项目、前端独立开发场景。
- **接口平台独立 Mock**：借助接口管理工具（如 Apifox、YApi）提供的独立 Mock 服务，实现接口文档与 Mock 数据一体化。优点是规范、可协作、支持自动同步接口变更；缺点是需搭建/接入平台，配置成本略高，适用于中大型项目、多团队协作场景。

## 1.3 Mock 工程化核心价值

Mock 数据并非“临时凑数”，而是贯穿“开发-测试-联调-部署”全流程的工程化工具，其核心价值体现在三个方面：

- 解耦前后端依赖：实现前后端并行开发，前端无需等待后端接口，独立推进业务逻辑开发与调试，缩短项目整体迭代周期。
- 覆盖全场景测试：真实接口难以覆盖所有边界场景（如异常数据、权限拦截、网络延迟），Mock 可灵活模拟各类场景，帮助前端提前排查异常逻辑，提升代码健壮性。
- 降低协作成本：通过统一的 Mock 规范，提前与后端约定接口格式，避免联调时因数据格式不一致导致的返工；同时 Mock 规则可复用、可维护，减少重复开发成本。

# 二、Mock 三大核心评估维度

选择 Mock 方案、编写 Mock 规则时，需围绕“仿真度、易用性、灵活性”三个核心维度展开，这三个维度直接决定 Mock 模拟的效率与质量，也是 当前前端工程化 Mock 规范化的核心依据。

## 2.1 仿真度：Mock 数据的核心前提

Mock 数据需在接口定义的各个方面与后端实际接口保持一致，包括：请求参数（参数名、类型、必填项）、响应格式（code、message、data 结构）、数据类型（字符串长度、数字范围）、异常状态（错误码、错误信息）、请求头要求等。

仿真度是决定 Mock 价值的首要因素——若 Mock 数据与真实接口差异较大，前端开发时需适配 Mock 数据，联调时又需修改适配真实接口，反而增加开发成本。当前，仿真度的实现已趋向全自动化，通过接口文档（OpenAPI/Swagger）自动同步 Mock 规则，结合 AI 校验，确保与接口定义完全对齐，大幅减少人工干预，但仍需人工校验关键业务逻辑。

## 2.2 易用性：提升开发效率的关键

高效的 Mock 方案需具备三大易用性特征：一是接口文档自动转换为 Mock 接口，无需手动编写大量规则；二是接口变更后，Mock 数据可自动同步，避免人工同步导致的遗漏；三是支持环境无缝切换，开发用 Mock、联调用真实接口，无需修改前端业务代码。

此外，当前的 Mock 方案还需深度适配现代构建工具（如 Vite 5、Webpack 5）、开发框架（如 React 19、Vue 3.5、Next.js 15），实现“零配置启动”“按需加载”“热更新”，进一步降低前端开发的配置成本。

## 2.3 灵活性：适配复杂业务场景的保障

实际接口调用中，会根据不同的调用方式（GET/POST/PUT/DELETE）、传入参数（query/body）、请求头（权限 token）等条件，返回不同的响应数据；前端也需根据这些差异，实现不同的交互逻辑（如登录成功/失败、数据为空/异常）。

因此，Mock 方案需具备足够的灵活性：支持根据请求参数动态返回数据、支持自定义脚本编写复杂业务逻辑、支持模拟网络延迟、错误状态码等异常场景，同时适配多环境（开发、测试、预发布）Mock 规则差异化配置，确保能覆盖前端开发中的所有业务场景。

# 三、Mock 底层核心原理

Mock 数据的核心原理是“拦截前端请求，模拟后端响应”——让前端发起的请求不真正发送到后端服务器，而是被 Mock 工具拦截，然后返回预设的模拟数据，整个过程对前端业务代码完全透明。

## 3.1 核心流程（文字流程图）

前端发起请求（如 axios.get('/api/user')）→ Mock 拦截器捕获请求 → 匹配预设的 Mock 规则（请求路径、方法、参数等）→ 匹配成功：返回预设 Mock 数据；匹配失败：转发至真实后端接口或返回异常提示 → 前端接收响应，执行后续业务逻辑（渲染页面、处理状态）。

## 3.2 两大拦截模型

- **前端层面拦截**：通过重写 XMLHttpRequest、fetch 或请求库（axios）的方法，实现请求拦截。例如 Vite Plugin Mock 已适配 Vite 5，优化了拦截性能，解决了与真实请求冲突的问题。优点是无需启动额外服务，配置简单；缺点是无法模拟真实跨域、网络延迟细节。
- **网络层面拦截**：基于 Service Worker 或独立 Mock 服务器，拦截所有网络请求，模拟真实 HTTP 响应。例如 MSW、Apifox 独立 Mock 服务均采用此方式，优化了 Safari 等浏览器的兼容性，支持更多边缘环境。优点是能模拟真实网络场景（跨域、延迟、异常状态），对业务代码无侵入；缺点是配置相对复杂，需关注边缘环境兼容边界。

## 3.3 数据生成机制

Mock 数据的生成主要分为三类，适配不同场景需求，2025-2026 年 AI 辅助生成成为重要补充能力（目前仍处于辅助阶段，未成为核心方式）：

- **静态生成**：手动编写固定 JSON 文件，Mock 工具直接读取并返回。适用于简单静态数据（如下拉选项），优点是直观，缺点是无法生成动态数据，复用性差。
- **动态模板生成**：通过 Mock 工具提供的语法规则，动态生成结构化数据，支持关联数据、随机数据。例如 MSW 的响应模板、faker.js v9.6.x的随机数据生成，支持多语言适配、业务场景模板库，适用于复杂业务场景，灵活性高、复用性强。
- **AI 辅助生成**：2025-2026 年的重要发展方向，可通过 AI 工具（如 Apifox AI 功能），根据接口文档、自然语言描述或真实业务数据样本，辅助生成符合业务逻辑的 Mock 数据，支持异常场景、边界值、关联数据的生成，可降低 Mock 规则编写成本，但仍需人工校验调整。

## 3.4 规则匹配逻辑与优先级

Mock 规则的核心是“请求特征匹配”，常见匹配维度包括：请求路径（精确/模糊/正则匹配）、请求方法（GET/POST 等）、请求参数（query/body）、请求头（如 Authorization）。

匹配优先级遵循“精确匹配 > 正则匹配 > 模糊匹配”，同时可通过配置优先级参数调整顺序，避免规则冲突。例如 MSW 中，可通过 `onRequest` 钩子自定义匹配逻辑，灵活调整优先级，适配复杂项目的规则管理需求。

# 四、主流 Mock 方案全解析（结合 当前 真实现状）

结合当前前端工程化现状，以下是当前主流 Mock 方案的详细解析，包含真实版本、适用场景、优缺点及实操示例，帮你快速选型。

## 4.1 Mock.js：历史方案，仅作兼容参考

### 核心说明（关键提醒）

Mock.js 是早期主流的前端 Mock 工具，定义了 DTD（数据模板定义规范）和 DPD（数据占位符定义规范），核心能力包括 Ajax 请求拦截、动态数据生成、数据验证、模板导出。但该项目**已于 2021 年停止维护**（GitHub 仓库最后更新于 2021 年），存在安全风险且无法适配目前现代前端架构（如 Vite 5、React 19），**新项目严禁选用**，仅适用于历史老旧项目兼容维护，建议逐步迁移至现代方案。

### 核心能力（历史参考）

- DTD 规范：定义数据结构，如"'list|10'"生成 10 个元素的数组，"'id|+1': 1"生成自增 ID。
- DPD 规范：通过占位符生成随机数据，如"@cname"（中文名）、"@email"（邮箱）。
- 请求拦截：重写 XMLHttpRequest，支持指定 URL 和请求方法拦截。
- 数据验证：Mock.valid 方法验证真实接口返回值与 Mock 模板是否匹配。

## 4.2 faker.js：纯随机数据生成工具

### 核心定位

faker.js是专注于高质量随机数据生成的独立工具，不具备请求拦截能力，常与 MSW、Vite Plugin Mock 等工具结合使用，提升 Mock 数据的真实性，支持业务场景模板库（电商、医疗、金融等），支持多语言精准适配。可通过 `npm view @faker-js/faker versions` 查看最新稳定版本。

### 核心优势与实操示例

支持多语言、多场景数据生成（如用户信息、电商数据、地址、手机号等），可生成高度贴近真实业务的数据，支持自定义数据模板，适用于对数据真实性要求较高的场景（演示环境、测试环境、验收环境）。

```javascript
// 安装依赖
npm install @faker-js/faker@9.6 --save-dev

// 实操示例：生成用户数据（结合电商场景模板）
import { faker, commerce } from '@faker-js/faker';

// 生成单个用户数据（含电商关联信息）
const user = {
  id: faker.datatype.uuid(),
  name: faker.person.fullName(),
  age: faker.datatype.number({ min: 18, max: 60 }),
  email: faker.internet.email(),
  avatar: faker.image.avatar(),
  createTime: faker.date.past().toISOString(),
  favoriteProduct: commerce.productName(), // 新增电商场景字段
  memberLevel: faker.helpers.arrayElement(['普通会员', '黄金会员', '钻石会员'])
};

// 生成10条用户列表数据
const userList = Array.from({ length: 10 }, () => ({
  id: faker.datatype.uuid(),
  name: faker.person.fullName(),
  age: faker.datatype.number({ min: 18, max: 60 }),
  email: faker.internet.email(),
  memberLevel: faker.helpers.arrayElement(['普通会员', '黄金会员', '钻石会员'])
}));
```

## 4.3 JSON-Server：轻量 RESTful Mock 服务器

### 版本说明（关键提醒）

JSON-Server 截至目前，最新版本为 v1.0.0-beta.x（2024 年后持续迭代 beta 版本），无 v2.x 版本，当前版本优化了性能，支持 TypeScript 原生适配，新增动态规则配置、接口权限模拟等功能，解决了早期版本动态数据生成能力弱的问题。可通过 `npm view json-server versions` 查看最新版本。

### 核心定位与实操示例

基于 Node.js 的轻量级 Mock 服务器，可通过 JSON 文件或 TypeScript 配置文件快速搭建 RESTful 接口，支持分页、排序、过滤、权限模拟等高级查询，适合需要模拟真实后端接口的中小型项目、接口联调前的测试场景，适配 Node.js 18+ 版本。

```bash
// 1. 安装
npm install json-server@1.0.0-beta --save-dev

// 2. 创建 mock/db.ts（TypeScript 模拟数据库，当前主流）
export default {
  "users": [
    {"id": 1, "name": "张三", "age": 25, "email": "zhangsan@example.com", "memberLevel": "黄金会员"},
    {"id": 2, "name": "李四", "age": 30, "email": "lisi@example.com", "memberLevel": "普通会员"}
  ],
  "posts": [
    {"id": 1, "title": "前端工程化实践", "content": "Mock数据的使用技巧", "userId": 1}
  ]
}

// 3. 启动 Mock 服务器
npx json-server mock/db.ts --port 3001 --prefix /api --watch

// 4. 支持的高级查询（新增权限模拟）
// 分页：http://localhost:3001/api/users?_page=1&_limit=10
// 排序：http://localhost:3001/api/users?_sort=age&_order=desc
// 过滤：http://localhost:3001/api/users?age_gte=20&age_lte=30
// 权限模拟：http://localhost:3001/api/users?token=valid_token（需配置规则）
```

### 优缺点

- 优点：零代码快速搭建，支持 RESTful 接口；可模拟真实 HTTP 请求，支持分页、排序、权限模拟等高级功能；可被 Postman 调用，方便前后端联调；支持 TypeScript 原生适配。
- 缺点：复杂业务逻辑实现难度较大；AI 动态生成能力较弱，需手动补充模板；不适合大型项目的复杂场景。

## 4.4 Vite Plugin Mock：Vite 生态专属本地化方案

### 核心定位

专为 Vite 5 构建工具设计的本地 Mock 插件，截至目前最新版本约为 v2.x，已适配 Vite 5 热更新机制，集成了请求拦截、动态数据生成能力，支持 TypeScript 原生适配，零配置启动，是当前 Vite 项目的首选本地化 Mock 方案（完全替代已停更的 Mock.js）。该插件无 AI 辅助生成功能及 proxy 配置项，相关表述为不实信息。可通过 `npm view vite-plugin-mock versions` 查看最新稳定版本。

### 实操示例（Vite 5 + React 19/Vue 3.5 通用）

```javascript
// 1. 安装依赖（适配 Vite 5，当前最新 v2.x 版本）
npm install vite-plugin-mock@2 mockjs --save-dev

// 2. 配置 vite.config.ts（Vite 5 配置，无 aiAssist 和 proxy 配置）
import { defineConfig } from 'vite';
import { viteMockServe } from 'vite-plugin-mock';
import react from '@vitejs/plugin-react'; // React 19 项目，Vue 3.5 项目替换为 @vitejs/plugin-vue

export default defineConfig({
  plugins: [
    react(),
    viteMockServe({
      enable: process.env.NODE_ENV === 'development', // 仅开发环境启用
      mockPath: './src/mock', // Mock 规则文件目录
      supportTs: true, // 支持 TypeScript
      watch: true // 监听 Mock 文件变化，自动更新
    })
  ]
});

// 3. 创建 src/mock/user.ts（Mock 规则文件）
import { MockMethod } from 'vite-plugin-mock';
import { faker, commerce } from '@faker-js/faker';

export default [
  {
    url: '/api/user/list',
    method: 'get',
    response: () => {
      // 动态生成10条用户数据（结合 faker.js v9.6.x 电商模板）
      return {
        code: 200,
        message: 'success',
        data: {
          list: Array.from({ length: 10 }, () => ({
            id: faker.datatype.uuid(),
            name: faker.person.fullName(),
            age: faker.datatype.number({ min: 18, max: 60 }),
            email: faker.internet.email(),
            memberLevel: faker.helpers.arrayElement(['普通会员', '黄金会员', '钻石会员']),
            favoriteProduct: commerce.productName()
          })),
          total: 100,
          page: 1,
          size: 10
        }
      };
    }
  },
  {
    url: '/api/login',
    method: 'post',
    response: (req) => {
      const { username, password } = req.body;
      if (username === 'admin' && password === '123456') {
        return {
          code: 200,
          message: '登录成功',
          data: { token: faker.datatype.string(32) }
        };
      }
      return { code: 401, message: '用户名或密码错误' };
    }
  }
] as MockMethod[];

// 4. 前端组件中使用（React 19 函数组件示例）
import { useEffect, useState } from 'react';
import axios from 'axios';

function UserList() {
  const [list, setList] = useState([]);
  useEffect(() => {
    axios.get('/api/user/list').then(res => setList(res.data.data.list));
  }, []);
  return (
    <ul>
      {list.map(user => (
        <li key={user.id}>{user.name}（{user.memberLevel}）</li>
      ))}
    </ul>
  );
}
```

## 4.5 MSW：当前主流 Mock 方案

### 版本说明

MSW（Mock Service Worker）截至目前，最新稳定版本为 v2.7.x（2025 年发布），无 v3 版本（v3 仍在讨论/开发中，未正式发布）。当前版本已优化 Edge Runtime 适配、支持 TypeScript 泛型强化、优化 Safari 等浏览器兼容性，解决了早期版本中边缘环境适配不足的问题。不存在规则分组与优先级可视化、AI 辅助生成 Mock 规则等功能，相关表述为不实信息。可通过 `npm view msw versions` 查看最新稳定版本。

### 核心定位与优势

基于 Service Worker 的网络层拦截工具，既能在浏览器端拦截请求，也能在 Node.js 18+、Edge Runtime 环境中使用（支持单元测试、Serverless 部署），对业务代码无侵入，支持动态规则、复杂业务逻辑，适配现代前端架构（Next.js 15、React 19、Vue 3.5），支持多环境 Mock 规则差异化配置。

### 实操示例（React 19 + TypeScript + MSW v2.7.x）

```typescript
// 1. 安装依赖（MSW 当前最新稳定版 v2.7.x）
npm install msw@2.7 --save-dev

// 2. 创建 src/mocks/handlers.ts（请求处理器）
import { http, HttpResponse } from 'msw';
import { faker, commerce } from '@faker-js/faker';

// 定义接口类型（TypeScript 类型安全，泛型强化）
interface User {
  id: string;
  name: string;
  age: number;
  email: string;
  memberLevel: '普通会员' | '黄金会员' | '钻石会员';
  favoriteProduct?: string;
}

interface UserListResponse {
  code: number;
  message: string;
  data: {
    list: User[];
    total: number;
    page: number;
    size: number;
  };
}

export const handlers = [
  // 模拟用户列表接口（GET），无 group 参数
  http.get('/api/user/list', () => {
    const list: User[] = Array.from({ length: 10 }, () => ({
      id: faker.datatype.uuid(),
      name: faker.person.fullName(),
      age: faker.datatype.number({ min: 18, max: 60 }),
      email: faker.internet.email(),
      memberLevel: faker.helpers.arrayElement(['普通会员', '黄金会员', '钻石会员']),
      favoriteProduct: commerce.productName()
    }));

    const response: UserListResponse = {
      code: 200,
      message: 'success',
      data: { list, total: 100, page: 1, size: 10 }
    };

    return HttpResponse.json(response, { status: 200 });
  }),

  // 模拟登录接口（POST），支持动态参数校验
  http.post<{ username: string; password: string }, UserListResponse>('/api/login', async ({ request }) => {
    const { username, password } = await request.json();
    if (username === 'admin' && password === '123456') {
      return HttpResponse.json({
        code: 200,
        message: '登录成功',
        data: { 
          list: [],
          total: 0,
          page: 1,
          size: 10,
        }
      });
    }
    return HttpResponse.json(
      { code: 401, message: '用户名或密码错误', data: { list: [], total: 0, page: 1, size: 10 } },
      { status: 401 }
    );
  })
];

// 3. 创建 src/mocks/server.ts（Node.js 环境，用于单元测试）
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// 4. 创建 src/mocks/browser.ts（浏览器环境，用于开发，适配 Edge Runtime）
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);

// 5. 入口文件 src/index.tsx 引入（仅开发环境）
if (process.env.NODE_ENV === 'development') {
  import('./mocks/browser').then(({ worker }) => {
    worker.start({
      onUnhandledRequest: 'bypass' // 未匹配的请求转发至真实接口
    });
  });
}

// 6. 单元测试中使用（src/components/UserList.test.tsx）
import { server } from '../mocks/server';
import { render, screen, waitFor } from '@testing-library/react';
import UserList from './UserList';

// 测试前启动 Mock 服务
beforeAll(() => server.listen());
// 测试后重置规则
afterEach(() => server.resetHandlers());
// 测试结束关闭服务
afterAll(() => server.close());

test('渲染用户列表', async () => {
  render(<UserList />);
  await waitFor(() => {
    expect(screen.getByText(/用户列表/i)).toBeInTheDocument();
  });
});
```

## 4.6 接口平台方案：YApi / Apifox 一体化 Mock

适用于中大型项目、多团队协作场景，实现“接口文档-Mock-测试-联调”全流程一体化，无需前端维护 Mock 规则，由接口文档自动同步 Mock 数据，结合 AI 辅助生成，是当前企业级项目的主流选择之一。

### 4.6.1 YApi：开源接口管理 + Mock

开源接口管理工具，已升级适配 OpenAPI 3.1 文档，支持导入导出 TypeScript 类型，自动生成 Mock 接口，支持自定义 Mock 规则、数据模板复用、团队协作管理、权限控制。其 Mock 功能可结合第三方 AI 工具辅助生成规则，可根据接口描述生成符合业务逻辑的 Mock 数据，支持动态脚本编写，可根据请求参数返回不同数据，适配复杂业务场景；同时支持接口版本控制，接口文档更新后，Mock 数据自动同步，适配目前企业级团队协作需求。

### 4.6.2 Apifox：全流程一体化工具

集成接口设计、Mock、测试、联调于一体，其 Mock 功能支持 AI 辅助生成 Mock 数据（基于接口文档、自然语言描述或真实数据样本）、自定义 Mock 规则、模拟异常场景（网络延迟、错误状态码、权限拦截）；支持与真实接口无缝切换，联调时无需修改前端请求地址，只需切换数据源即可；支持 Mock 规则版本控制、多环境差异化配置，支持与 CI/CD 流程深度集成，实现 Mock 环境自动化部署；支持 TypeScript 类型自动生成，与前端项目无缝衔接。需注意，“Apifox 5.0”“AI 2.0”为营销术语，非明确技术版本，实际功能以官方最新版本为准。

# 五、现代新生代 Mock 工具栈

除上述主流方案外，2025-2026 年涌现出一批适配现代前端架构的新生代 Mock 工具，针对特定场景提供更优解决方案，以下是核心工具解析：

## 5.1 Mirage JS：前端模拟后端服务（现状说明）

专注于模拟完整的后端服务逻辑，截至目前，最新版本为 v0.1.48（2022 年后几乎停更），无 v2 版本，相关 v2 版本及 AI 辅助生成功能表述为不实信息。该工具支持数据持久化、关联查询、自定义路由，可适配 React 19、Vue 3.5 等现代框架，适用于部分复杂业务场景（如电商、管理系统），但其停更状态不建议新项目首选。

## 5.2 Hono + Zod：TypeScript 类型安全 Mock

Hono 当前最新版本为 v4.7.x，是轻量、高性能的 Web 框架，Zod v3（当前最新 v3.24.x）是 TypeScript 类型验证工具，两者结合可实现“类型安全的 Mock 服务”，适配 Edge Runtime、Serverless 架构，支持 Vercel、Cloudflare 等边缘部署。通过 Zod 定义接口参数和响应类型，Hono 提供路由和请求处理，既能作为本地 Mock 服务，也能部署为独立 Mock 服务器，可结合第三方 AI 工具辅助生成类型和 Mock 规则，是当前 TypeScript 项目、边缘环境项目的高端选择之一。可通过 `npm view hono versions`、`npm view zod versions` 分别查看两者最新稳定版本。

## 5.3 WireMock v3 / Prism v5：接口文档驱动 Mock

两者均为“接口文档驱动”的 Mock 工具，最新分别升级至 v3、v5 版本，支持导入 OpenAPI 3.1 文档，自动生成 Mock 接口，确保 Mock 数据与接口文档完全一致，支持 TypeScript 类型同步。其中 WireMock v3 侧重模拟真实 HTTP 服务，新增故障注入、延迟模拟、权限校验等功能，适配复杂测试场景；Prism v5 侧重接口文档验证，同时提供 Mock 功能，支持接口契约测试，适合对接口规范要求严格的企业级团队。

# 六、TypeScript 类型安全 Mock 工程化

TypeScript 已成为前端工程化主流，Mock 数据的类型安全成为核心需求——避免因 Mock 数据与真实接口类型不一致，导致联调时出现类型错误。以下是 TypeScript 项目中 Mock 数据的最佳实践：

## 6.1 Mock 数据类型约束

通过 TypeScript 接口、泛型定义请求参数和响应数据类型，结合 Zod 进行 runtime 校验，确保 Mock 数据符合类型规范，避免类型错误，示例如下：

```typescript
// src/types/api.ts（定义接口类型，结合 Zod 校验）
import { z } from 'zod';

// 定义请求参数类型及校验规则
export const LoginParamsSchema = z.object({
  username: z.string().min(3).max(20),
  password: z.string().min(6).max(20)
});
export type LoginParams = z.infer<typeof LoginParamsSchema>;

// 定义响应数据类型及校验规则
export const LoginResponseSchema = z.object({
  code: z.number(),
  message: z.string(),
  data: z.object({
    token: z.string(),
    userInfo: z.object({
      id: z.string(),
      name: z.string()
    })
  })
});
export type LoginResponse = z.infer<typeof LoginResponseSchema>;

// src/mocks/handlers.ts（Mock 规则类型约束 +  runtime 校验）
import { http, HttpResponse } from 'msw';
import { LoginParamsSchema, LoginResponseSchema, LoginParams, LoginResponse } from '../types/api';
import { faker } from '@faker-js/faker';

export const handlers = [
  http.post<LoginParams, LoginResponse>('/api/login', async ({ request }) => {
    const params = await request.json();
    //  runtime 校验请求参数
    const validatedParams = LoginParamsSchema.safeParse(params);
    if (!validatedParams.success) {
      return HttpResponse.json(
        { code: 400, message: '参数格式错误', data: { token: '', userInfo: { id: '', name: '' } } },
        { status: 400 }
      );
    }
    if (validatedParams.data.username === 'admin' && validatedParams.data.password === '123456') {
      const response = {
        code: 200,
        message: '登录成功',
        data: {
          token: faker.datatype.string(32),
          userInfo: { id: faker.datatype.uuid(), name: faker.person.fullName() }
        }
      };
      // 校验 Mock 响应数据类型
      LoginResponseSchema.parse(response);
      return HttpResponse.json(response);
    }
    return HttpResponse.json(
      { code: 401, message: '用户名或密码错误', data: { token: '', userInfo: { id: '', name: '' } } },
      { status: 401 }
    );
  })
];
```

## 6.2 OpenAPI 自动生成 TS 类型 + Mock 接口

当前，通过 OpenAPI 文档（后端提供），使用工具（如 openapi-typescript-codegen，当前最新 v0.29.x，无 v10 版本）自动生成 TS 接口类型和 Zod 校验规则，再结合 MSW、Apifox 等工具，自动生成 Mock 规则，实现“接口文档 → TS 类型 → Zod 校验 → Mock 接口”全自动化，避免人工编写导致的类型不一致，提升开发效率。可通过 `npm view openapi-typescript-codegen versions` 查看该工具最新稳定版本。

```bash
// 安装自动生成工具
npm install openapi-typescript-codegen@0.29 --save-dev

// 生成 TS 类型和 Zod 校验规则（package.json 新增脚本）
"scripts": {
  "generate:types": "openapi-typescript-codegen --input ./openapi.json --output ./src/types/api --client axios --zod"
}

// 执行脚本，自动生成 TS 接口类型和 Zod 校验规则
npm run generate:types
```

## 6.3 前后端类型对齐最佳实践

- 后端提供 OpenAPI 3.1 文档，前端通过工具自动生成 TS 类型和 Zod 校验规则，确保前后端类型一致。
- Mock 规则严格遵循生成的 TS 类型和 Zod 校验规则，通过 runtime 校验避免出现类型不匹配的 Mock 数据。
- 联调前，通过 MSW 的类型验证功能 + Zod 校验，检查真实接口返回值与 TS 类型是否一致，提前发现问题。
- 新增前后端类型同步机制，后端接口变更后，自动同步 OpenAPI 文档，前端自动更新 TS 类型和 Mock 规则，无需人工干预。

# 七、现代前端架构适配

随着 Next.js 15 RSC、Edge Runtime、Monorepo 等现代架构的普及，Mock 方案需针对性适配，避免出现兼容性问题，以下是核心场景的适配方案（基于真实现状）：

## 7.1 Next.js 15 RSC / App Router 下的 Mock 处理

Next.js 15 的 App Router 进一步优化 React Server Components（RSC），服务端组件无法访问浏览器 API（如 Service Worker），因此 MSW 的浏览器端拦截方案无法直接使用，当前主流适配方案如下：

- 方案1：使用 MSW v2.7.x Node.js 版本，在 Next.js 15 中间件中拦截请求，模拟服务端 Mock，支持 Edge Runtime 部署。
- 方案2：使用 Hono v4.7.x + Zod 搭建独立 Mock 服务器，服务端组件直接请求 Mock 服务器，避免浏览器 API 依赖，适配 Serverless 架构。
- 方案3：通过 Next.js 15 的 app/api 目录配置自定义 Mock 路由（非官方内置 Mock 功能，仅为普通 API 路由），无需额外安装工具，适用于简单场景。

## 7.2 Edge Runtime 边缘环境 Mock

Vercel、Cloudflare 等 Edge Runtime 环境不支持 Node.js 核心模块，传统 Mock 工具（如 JSON-Server）无法使用，当前主流适配方案如下：

- 方案1：MSW v2.7.x（支持 Edge Runtime 原生适配），无需修改代码，直接部署为边缘 Mock 服务。
- 方案2：Hono v4.7.x + Zod，轻量高性能，适配 Edge Runtime，支持 TypeScript 类型安全，可部署为独立 Mock 服务器。
- 方案3：Apifox 云端 Mock 服务，无需本地部署，直接调用云端 Mock 接口，适配 Serverless 架构。

## 7.3 Monorepo 多项目共享 Mock 规则

在 Turborepo、Nx 等 Monorepo 架构中，多个项目可能共用一套接口规范，当前主流实践是：将 Mock 规则、TS 接口类型、Zod 校验规则抽取为独立共享包，可结合第三方 AI 工具辅助生成功能，供所有子项目复用，确保 Mock 规则的一致性，减少重复开发成本，同时支持多项目 Mock 规则差异化配置。

```plain
monorepo/
├── packages/
│   ├── mock-shared/      # 共享 Mock 包
│   │   ├── src/
│   │   │   ├── handlers/ # 共享 Mock 规则
│   │   │   ├── types/    # 共享 TS 接口类型 + Zod 校验规则
│   │   │   └── ai/       # 第三方 AI 辅助生成配置
│   │   └── package.json
├── apps/
│   ├── app1/             # 子项目1（复用 mock-shared，支持差异化配置）
│   └── app2/             # 子项目2（复用 mock-shared，支持差异化配置）
```

# 八、工程化规范、最佳实践与避坑

## 8.1 Mock 目录结构规范

无论使用哪种 Mock 方案，统一的目录结构能提升可维护性，以下是 TypeScript 项目的推荐目录结构（新增第三方 AI 辅助、Zod 校验相关目录）：

```plain
src/
├── mock/                # Mock 根目录
│   ├── index.ts         # Mock 入口文件（启动/配置）
│   ├── handlers/        # Mock 规则处理器（按业务模块划分）
│   │   ├── user.ts      # 用户模块
│   │   └── order.ts     # 订单模块
│   ├── templates/       # 共享 Mock 数据模板
│   ├── ai/              # 第三方 AI 辅助生成配置
│   └── validators/      # Zod 校验规则
├── types/               # TS 类型目录
│   └── api.ts           # 接口类型定义
└── api/                 # 接口请求封装
```

## 8.2 环境隔离：避免生产环境误引入

生产环境误引入 Mock 代码，会导致请求被拦截、接口异常，是最常见的踩坑点，当前推荐实践是：

- 通过环境变量（NODE_ENV、VITE_ENV 等）判断，仅在开发、测试、预发布环境引入 Mock 代码，生产环境自动忽略。
- 使用构建工具（Vite 5、Webpack 5）配置，生产环境打包时自动剔除 Mock 相关代码，避免冗余，同时通过 ESLint 规则禁止生产环境引入 Mock 模块。
- 新增 Mock 环境校验机制，上线前通过 CI/CD 流程检查，确保生产环境未引入 Mock 代码，避免线上事故。
- 示例（Vite 5 配置）：在 vite.config.ts 中，通过 enable 配置仅开发、测试环境启用 Mock 插件，同时配置构建剔除逻辑。

## 8.3 Mock 复用与模板抽离

避免每个接口重复编写 Mock 数据，抽离公共模板，结合第三方 AI 工具辅助生成，提升复用性，同时支持模板参数化，适配不同场景需求：

- 公共模板抽离：将高频使用的 Mock 数据结构（如用户信息、分页响应、异常响应）抽离为独立模板函数，通过参数传递动态调整数据，减少重复代码。
- 模板参数化：设计可配置的模板，支持传入参数（如数据条数、状态码、字段值），适配不同接口的需求，例如分页模板可传入 total、page、size 等参数，动态生成对应分页数据。
- 跨模块复用：将公共模板、工具函数抽取至共享目录（如 src/mock/templates），供所有业务模块调用，同时结合第三方 AI 工具辅助生成模板，提升模板编写效率。

## 8.4 常见踩坑点

结合当前前端工程化实践，总结 Mock 使用过程中最易踩坑的场景及解决方案，帮助开发者规避风险、提升效率：

- 踩坑点1：Mock 规则与真实接口同步不及时 → 解决方案：绑定接口文档（OpenAPI/Swagger），开启自动同步功能，后端接口变更后，前端 Mock 规则自动更新；同时建立接口变更通知机制，确保前后端同步知晓。
- 踩坑点2：生产环境误引入 Mock 代码 → 解决方案：严格通过环境变量隔离，结合构建工具剔除生产环境 Mock 代码，新增 ESLint 规则禁止生产环境引入 Mock 相关模块，CI/CD 流程添加 Mock 引入校验。
- 踩坑点3：Mock 数据与真实接口类型不一致 → 解决方案：基于 TypeScript 接口和 Zod 校验，强制 Mock 数据符合类型规范，联调前通过校验工具检查真实接口返回值与类型是否匹配。
- 踩坑点4：复杂业务逻辑 Mock 难度大 → 解决方案：优先选用支持自定义脚本的 Mock 方案（如 MSW、Hono+Zod），拆分复杂逻辑为多个简单模板，结合 AI 辅助生成核心逻辑的 Mock 规则，减少人工编写成本。
- 踩坑点5：版本号过时导致工具使用异常 → 解决方案：避免硬编码版本号（或标注版本查询方式），使用时通过 npm 命令查询最新稳定版，例如 `npm view 工具名 versions`，确保工具版本与项目架构适配。

## 8.5 Mock 工程化落地建议

结合当前前端工程化现状，针对不同规模项目，给出 Mock 方案落地建议，兼顾效率与规范：

- 小型项目（单人/小团队，Vite 架构）：优先选用 Vite Plugin Mock + faker.js，零配置启动，快速实现本地 Mock，无需额外搭建服务，降低配置成本。
- 中型项目（多团队协作，TypeScript 为主）：选用 MSW + Zod，实现类型安全 Mock，结合 OpenAPI 自动生成类型和 Mock 规则，规范团队协作流程。
- 大型项目（企业级，多架构适配）：选用 Apifox/YApi 一体化平台 + Hono+Zod 边缘 Mock 服务，实现接口文档、Mock、测试、联调全流程一体化，适配 Monorepo、Edge Runtime 等现代架构。

同时，建议定期梳理 Mock 规则，删除废弃接口的 Mock 配置，更新工具版本，结合团队反馈优化 Mock 规范，确保 Mock 工具持续为项目提效，而非增加维护成本。