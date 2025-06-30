---
toc: true
title: Feature-Sliced Design
date: 2025-06-30 21:56:53
pic: https://feature-sliced.design/img/brand/logo-primary.png
tags:
  - 设计规范
  - WEB
  - react
category:
  - 设计规范
---

# Feature-Sliced Design（FSD）

## 一、什么是 Feature-Sliced Design

Feature-Sliced Design（简称 FSD）是一种前端架构设计方法论，专注于大型/中型前端项目的可维护性、可扩展性和可复用性。FSD 通过明确的分层和分割规则，将前端项目划分为不同的模块和责任层，优化项目结构和协作流程。

- **核心目标**：降低代码耦合、提升模块可复用性、支持多团队并行开发。
- **主要关注点**：关注业务功能（Feature）为核心进行切分，而非传统的页面/组件维度。

## 二、FSD 的知识体系

### 1. 全局视角（宏观设计）

- 责任分层（Layers）：明确每一层的作用和依赖方向。
- 分割原则（Slicing）：根据业务功能、领域、模块等进行横向切分。
- 依赖管理：上层依赖下层，下层不可依赖上层。
- 可扩展性、可测试性设计。

### 2. 主要分层（自顶向下）

| 层级            | 主要责任          | 依赖关系  | 典型内容                             |
| --------------- | ----------------- | --------- | ------------------------------------ |
| **app**         | 全局设置/依赖注入 | 最顶层    | Provider、路由、主题、国际化         |
| **processes**   | 业务流程整合      | app 之下  | 流程编排（如注册、登录流）           |
| **pages**       | 页面/路由         | processes | 页面结构，页面路由                   |
| **widgets**     | 复合 UI 部件      | pages     | 可复用的界面组合（如搜索+列表+分页） |
| **features**    | 独立业务功能块    | widgets   | 明确业务行为（如添加、编辑、过滤）   |
| **entities**    | 领域数据          | features  | 数据结构、领域模型、实体操作         |
| **shared**      | 通用基础能力      | 全层可用  | 公共工具函数、UI 组件、hooks         |
| **service-api** | 后端通信          | entities  | API 定义、数据访问、适配器           |

#### 结构图示例：

```
my-app/
  ├─ app/
  ├─ processes/
  ├─ pages/
  ├─ widgets/
  ├─ features/
  ├─ entities/
  ├─ shared/
  └─ service-api/
```

### 3. 局部视角（微观设计）

- **目录规范**：每一层内部按 slice/segment 细分（如 features/filter、widgets/userTable）。
- **API 规范**：每一层的暴露接口应通过 public API 目录统一管理。
- **内部依赖**：禁止跨 slice、跨层非规定依赖。

### 4. FSD 对比其他架构（横向对比）

| 架构范式   | 主要切分原则    | 优势                     | 劣势               |
| ---------- | --------------- | ------------------------ | ------------------ |
| FSD        | 以功能/领域划分 | 低耦合、高聚合、可复用性 | 学习曲线相对较高   |
| 按页面划分 | 页面为单位      | 结构简单、易上手         | 复用性差、耦合高   |
| 按组件划分 | 组件为单位      | 组件粒度灵活             | 业务复杂时结构混乱 |
| DDD        | 领域驱动设计    | 聚焦领域、业务分层清晰   | 与 UI 不够贴合     |

### 5. FSD 内部纵向对比

- 按层次责任区分，每层职责不可混淆，依赖方向从 app → service-api。
- features/widgets/entities/shared 四者分别代表不同的抽象层次和可复用度。

| 层级     | 复用性 | 关注点         | 依赖             |
| -------- | ------ | -------------- | ---------------- |
| app      | 全局   | 应用初始化     | 全部             |
| widgets  | 高     | UI 复合部件    | shared, entities |
| features | 中     | 业务逻辑块     | entities, shared |
| entities | 低     | 领域模型与数据 | shared           |
| shared   | 最高   | 基础工具       | -                |

## 三、实际目录与代码示例

### 1. 目录示例

```
my-app/
  ├─ app/
  │    ├─ providers.tsx
  │    └─ router.tsx
  ├─ pages/
  │    ├─ HomePage/
  │    └─ UserProfilePage/
  ├─ widgets/
  │    ├─ UserTable/
  │    └─ SearchBar/
  ├─ features/
  │    ├─ FilterUsers/
  │    └─ AddUser/
  ├─ entities/
  │    ├─ user/
  │    │    ├─ model.ts
  │    │    └─ api.ts
  │    └─ role/
  ├─ shared/
  │    ├─ ui/
  │    │    ├─ Button.tsx
  │    │    └─ Dialog.tsx
  │    ├─ hooks/
  │    └─ utils/
  └─ service-api/
       ├─ client.ts
       └─ user.ts
```

### 2. 代码片段对比

#### 示例 1：一个表单的 FSD 拆分

- **entities/user/model.ts**

```ts
export interface User {
  id: string;
  name: string;
  email: string;
}
```

- **entities/user/api.ts**

```ts
import { User } from "./model";
export function fetchUsers(): Promise<User[]> {
  // ...
}
```

- **features/AddUser/index.tsx**

```tsx
import { User } from "@/entities/user/model";
export function AddUserFeature() {
  // 只包含业务表单逻辑，UI 封装交给 widgets
}
```

- **widgets/UserForm/index.tsx**

```tsx
import { AddUserFeature } from "@/features/AddUser";
export function UserFormWidget() {
  // 负责整合多个 feature，比如校验、提交
}
```

- **pages/UserPage/index.tsx**

```tsx
import { UserFormWidget } from "@/widgets/UserForm";
export default function UserPage() {
  return <UserFormWidget />;
}
```

#### 示例 2：API 层

- **service-api/user.ts**

```ts
export async function getUserList() {
  // 具体 API 请求逻辑
}
```

### 3. 对照组示例

#### 传统页面-组件风格（对比）

```
src/
  ├─ components/
  │    ├─ UserForm.tsx
  │    └─ UserTable.tsx
  ├─ pages/
  │    └─ users.tsx
```

- 所有逻辑散落在组件内，难以复用和测试。

#### FSD 风格（最优设计）

- 领域模型、UI、业务逻辑清晰分层，易于维护与扩展。

## 四、FSD 最优设计建议

1. **严格按层和 slice 分割目录与文件，不可混用。**
2. **entities 层只负责数据和领域逻辑，不写 UI 和副作用。**
3. **features 层聚焦单一业务能力，避免复合型 feature。**
4. **widgets 层负责业务场景下的 UI 复合，鼓励组合 feature/entity 形成更高层复用。**
5. **shared 层只放通用能力，不要引入具体业务内容。**
6. **所有依赖严格遵循自顶向下单向依赖，不可反向。**
7. **统一使用 public API 进行各层暴露和复用，避免内部实现泄漏。**
8. **通过 TypeScript 类型隔离层间数据结构。**

## 五、常见问题与优化点

- 切忌偷懒全部丢 shared，shared 只做基础能力。
- 跨 slice 引用通过 public API，不直接耦合实现。
- 保持每个 slice 粒度均衡，避免过大或过小。
- 通过 storybook/vitest 等配合保证各层可独立开发和测试。

## 六、参考资料

- [feature-sliced.design 官方文档](https://feature-sliced.design/)
- [Feature-Sliced Design vs Atomic Design](https://feature-sliced.design/docs/concepts/motivation)
- [如何在中大型项目中落地 FSD](https://feature-sliced.design/docs/guides/monolith)
