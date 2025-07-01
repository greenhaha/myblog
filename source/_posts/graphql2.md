---
toc: true
title: Graphql（二）
date: 2025-07-01 22:18:00
pic: https://graphql.cn/img/logo.svg
tags:
  - graphql
  - 后端
  - API
category:
  - 知识剖析
---

# FSD × GraphQL 分层结构、规范与 CRUD 最佳实践

本指南面向中大型 React 前端项目，结合 FSD（Feature-Sliced Design）与 GraphQL，从分层结构、目录规范、代码模板到错误对照组，给出最优落地方案。内容包含完整 CRUD（增删改查）场景，每层示例通俗易懂，便于团队参考与实践。

---

## 一、全局项目结构（FSD × GraphQL）

```
my-app/
  ├─ app/              # 全局配置、Provider、路由
  ├─ pages/            # 页面路由容器
  ├─ widgets/          # 场景复合UI
  ├─ features/         # 单一业务能力
  ├─ entities/         # 领域模型/GraphQL hooks
  ├─ shared/           # 通用UI、工具、hooks
  └─ service-api/      # GraphQL client配置、API调用适配
```

---

## 二、CRUD 典型业务（以 User 管理为例）

### 1. GraphQL schema 层（后端或 mock 层）

**正确（最优）**

```graphql
# service-api/graphql/schema.graphql

type User {
  id: ID!
  name: String!
  email: String!
}
input CreateUserInput {
  name: String!
  email: String!
}
input UpdateUserInput {
  id: ID!
  name: String
  email: String
}
type Query {
  users: [User!]!
  user(id: ID!): User
}
type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}
```

**错误对照**

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  password: String! # ❌敏感字段直接暴露
}
# Mutation参数不规范，未使用input类型，导致后续扩展难
```

---

### 2. GraphQL resolver 层（后端或 mock 层）

**正确（最优）**

```js
const resolvers = {
  Query: {
    users: (_, __, { dataSources }) => dataSources.userAPI.getUsers(),
    user: (_, { id }, { dataSources }) => dataSources.userAPI.getUserById(id),
  },
  Mutation: {
    createUser: (_, { input }, { dataSources }) =>
      dataSources.userAPI.createUser(input),
    updateUser: (_, { input }, { dataSources }) =>
      dataSources.userAPI.updateUser(input),
    deleteUser: (_, { id }, { dataSources }) =>
      dataSources.userAPI.deleteUser(id),
  },
};
```

**错误对照**

```js
const resolvers = {
  Query: {
    users: () => db.query("SELECT * FROM users"), // ❌直接SQL操作
  },
  Mutation: {
    createUser: (_, { name, email }) => {
      // ❌参数无 input 类型，字段混乱
      return db.insert({ name, email });
    },
  },
};
```

---

### 3. service-api 层（前端 GraphQL client 配置）

**正确（最优）**

```ts
// service-api/client.ts
import { ApolloClient, InMemoryCache, HttpLink } from "@apollo/client";
export const apolloClient = new ApolloClient({
  link: new HttpLink({ uri: "/graphql" }),
  cache: new InMemoryCache(),
});
```

**错误对照**

```ts
// service-api/client.ts
// ❌重复 new client，每个页面都配置一遍，缓存无共享
```

---

### 4. entities 层（领域模型/数据 hooks）

**正确（最优）**

```ts
// entities/user/model.ts
export interface User {
  id: string;
  name: string;
  email: string;
}

// entities/user/api.ts
import { gql, useQuery, useMutation } from "@apollo/client";
import { apolloClient } from "@/service-api/client";

export const USERS_QUERY = gql`
  query {
    users {
      id
      name
      email
    }
  }
`;
export function useUsers() {
  return useQuery(USERS_QUERY);
}

export const CREATE_USER = gql`
  mutation ($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;
export function useCreateUser() {
  return useMutation(CREATE_USER);
}
// updateUser/deleteUser 省略，模式一致
```

**错误对照**

```ts
// entities/user/api.ts
// ❌每次请求手写 fetch('/graphql', ...)，无封装、易出错
// ❌请求参数写死，UI 直接拼接 GraphQL 字符串
```

---

### 5. features 层（业务能力，如表单、列表操作）

**正确（最优）**

```tsx
// features/CreateUserForm/index.tsx
import { useState } from "react";
import { useCreateUser } from "@/entities/user/api";
export function CreateUserForm() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [createUser, { loading }] = useCreateUser();
  const handleSubmit = async (e) => {
    e.preventDefault();
    await createUser({ variables: { input: { name, email } } });
    setName("");
    setEmail("");
  };
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit" disabled={loading}>
        Create
      </button>
    </form>
  );
}
```

**错误对照**

```tsx
// features/CreateUserForm/index.tsx
// ❌直接写 fetch('/graphql', { body: JSON.stringify({ ... }) })，无封装
// ❌状态管理、请求都耦合在页面
```

---

### 6. widgets 层（复合 UI 场景，如 User 管理面板）

**正确（最优）**

```tsx
// widgets/UserPanel/index.tsx
import { useUsers } from "@/entities/user/api";
import { CreateUserForm } from "@/features/CreateUserForm";

export function UserPanelWidget() {
  const { data, loading } = useUsers();
  if (loading) return <p>Loading...</p>;
  return (
    <div>
      <CreateUserForm />
      <ul>
        {data?.users.map((u) => (
          <li key={u.id}>
            {u.name} ({u.email})
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**错误对照**

```tsx
// widgets/UserPanel/index.tsx
// ❌UI与状态管理全部混杂（useState+fetch），不复用 entities/features 能力
```

---

### 7. pages 层（页面拼装）

**正确（最优）**

```tsx
// pages/UserManagePage/index.tsx
import { UserPanelWidget } from "@/widgets/UserPanel";
export default function UserManagePage() {
  return <UserPanelWidget />;
}
```

**错误对照**

```tsx
// pages/UserManagePage/index.tsx
// ❌全部直接写页面（状态、请求、UI 混杂），不利于复用和测试
```

---

### 8. shared 层（通用能力）

**正确（最优）**

```tsx
// shared/ui/Button.tsx
import { Button as ChakraButton, ButtonProps } from "@chakra-ui/react";
export function Button(props: ButtonProps) {
  return <ChakraButton {...props} />;
}
```

**错误对照**

```tsx
// shared/ui/Button.tsx
// ❌ props 类型定义缺失，样式、行为固定死，不可复用
```

---

## 三、总结与实践建议

1. **schema 只暴露安全字段，合理 input/output 拆分**。
2. **resolver 只聚合业务，无数据源直连，无安全泄漏**。
3. **service-api 封装 Apollo client 全局实例，统一管理**。
4. **entities 封装 domain hooks，前端各层只通过 hooks 调用**。
5. **features 聚焦单一业务能力，禁止状态/副作用杂糅**。
6. **widgets 横向组合 features/entities，做场景复合**。
7. **pages 只做页面拼装，不承载业务/副作用/数据**。
8. **shared 只放通用能力，不涉足业务与领域数据**。
