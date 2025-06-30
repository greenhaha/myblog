---
toc: true
title: Feature-Sliced Design（2）
date: 2025-06-30 21:56:53
pic: https://feature-sliced.design/img/brand/logo-primary.png
tags:
  - 设计规范
  - WEB
  - react
category:
  - 设计规范
---

# FSD 各层级最优与错误对照组

本节面向团队实践和规范推进，结合实际项目代码示例，对每一层提供“最优做法（正确）”和“常见反面案例（错误）”对比，直观展现分层价值和常见问题。

---

## 1. app 层（全局配置/入口）

**最优做法（正确）**

```tsx
// app/providers.tsx
import { ChakraProvider } from "@chakra-ui/react";
import { AppRouter } from "./router";

export function AppProviders({ children }: { children: React.ReactNode }) {
  return (
    <ChakraProvider>
      <AppRouter>{children}</AppRouter>
    </ChakraProvider>
  );
}
```

**反面案例（错误）**

```tsx
// src/App.tsx
import { ChakraProvider } from "@chakra-ui/react";
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { UserTableWidget } from "../widgets/UserTable";
import { FilterUsersFeature } from "../features/FilterUsers";

export default function App() {
  // 全部混在入口，页面与Provider无分离
  return (
    <ChakraProvider>
      <BrowserRouter>
        <FilterUsersFeature />
        <UserTableWidget />
      </BrowserRouter>
    </ChakraProvider>
  );
}
```

- 错误点：入口包含页面/业务渲染，职责混乱。

---

## 2. pages 层（页面/路由）

**最优做法（正确）**

```tsx
// pages/UserPage/index.tsx
import { UserTableWidget } from "@/widgets/UserTable";
import { FilterUsersFeature } from "@/features/FilterUsers";

export default function UserPage() {
  return (
    <div>
      <FilterUsersFeature />
      <UserTableWidget />
    </div>
  );
}
```

**反面案例（错误）**

```tsx
// pages/UserPage/index.tsx
import { useState, useEffect } from "react";

export default function UserPage() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch("/api/users")
      .then((r) => r.json())
      .then(setUsers);
  }, []);
  return (
    <div>
      <input /> {/* 筛选功能直接写在页面 */}
      <table>
        <tbody>
          {users.map((u) => (
            <tr key={u.id}>
              <td>{u.name}</td>
              <td>{u.email}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

- 错误点：业务逻辑、状态、副作用、UI 全写进页面，违背单一职责。

---

## 3. widgets 层（复合 UI 场景）

**最优做法（正确）**

```tsx
// widgets/UserTable/index.tsx
import { useUsers } from "@/entities/user";
import { UserRowFeature } from "@/features/UserRow";

export function UserTableWidget() {
  const { users } = useUsers();
  return (
    <table>
      <tbody>
        {users.map((u) => (
          <UserRowFeature key={u.id} user={u} />
        ))}
      </tbody>
    </table>
  );
}
```

**反面案例（错误）**

```tsx
// widgets/UserTable/index.tsx
export function UserTableWidget({ users }) {
  // 仅做纯展示，不组合 feature/entity，丢失复用性
  return (
    <table>
      <tbody>
        {users.map((u) => (
          <tr key={u.id}>
            <td>{u.name}</td>
            <td>{u.email}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

- 错误点：组件成为“死表格”，没有能力组合 feature/entity，难以扩展。

---

## 4. features 层（单一业务能力）

**最优做法（正确）**

```tsx
// features/FilterUsers/index.tsx
import { useUserFilter } from "@/entities/user";
export function FilterUsersFeature() {
  const { filter, setFilter } = useUserFilter();
  return (
    <input
      value={filter}
      onChange={(e) => setFilter(e.target.value)}
      placeholder="筛选用户"
    />
  );
}
```

**反面案例（错误）**

```tsx
// features/FilterUsers/index.tsx
export function FilterUsersFeature({ users, onFilter }) {
  // 直接操作父组件状态，业务能力耦合外部
  return (
    <input
      onChange={(e) =>
        onFilter(users.filter((u) => u.name.includes(e.target.value)))
      }
    />
  );
}
```

- 错误点：feature 不聚焦自身业务，状态管理混乱、耦合父组件。

---

## 5. entities 层（领域数据/模型）

**最优做法（正确）**

```ts
// entities/user/model.ts
export interface User {
  id: string;
  name: string;
  email: string;
}
// entities/user/api.ts
import { fetchUserList } from "@/service-api/user";
export async function getUsers() {
  return await fetchUserList();
}
```

**反面案例（错误）**

```ts
// entities/user/model.ts
export const UserModel = {};
// model 只做变量导出，无类型定义，缺少实体语义

// entities/user/api.ts
export async function getUsers() {
  // 直接 fetch，无适配
  return await fetch("/api/users").then((r) => r.json());
}
```

- 错误点：缺少类型、缺少对 service-api 的抽象和适配。

---

## 6. shared 层（通用能力）

**最优做法（正确）**

```tsx
// shared/ui/Button.tsx
import { Button as ChakraButton, ButtonProps } from "@chakra-ui/react";
export function Button(props: ButtonProps) {
  return <ChakraButton {...props} />;
}
```

**反面案例（错误）**

```tsx
// shared/ui/Button.tsx
export function Button({ label }) {
  // 没有类型、props受限，不能扩展
  return <button>{label}</button>;
}
```

- 错误点：不利用已有能力、类型定义不全。

---

## 7. service-api 层（后端通信）

**最优做法（正确）**

```ts
// service-api/user.ts
export async function fetchUserList() {
  const res = await fetch("/api/users");
  return res.json();
}
```

**反面案例（错误）**

```ts
// service-api/user.ts
export async function fetchUserList() {
  // 缺乏错误处理、未适配返回值结构
  return fetch("/api/users");
}
```

- 错误点：没有数据解析/错误处理，未抽象出前端可用格式。

---

## 总结

- FSD 强调每层**只做本层职责**，各层通过 public-api 协作，避免耦合与责任重叠。
- 错误分层的常见症状：职责混乱、类型不明、状态乱入、无复用、不可单测。
