---
toc: true
title: Graphql
date: 2025-07-01 22:18:00
pic: https://graphql.cn/img/logo.svg
tags:
  - graphql
  - 后端
  - API
category:
  - 知识剖析
---

# GraphQL 全面知识体系、对照组与最优实践

---

## 一、什么是 GraphQL？

GraphQL 是一种 API 查询语言和服务端运行环境。它允许客户端声明所需数据的结构，服务端仅返回被请求的数据，解决了传统 REST API 的 over-fetching/under-fetching 问题。

- **核心理念**：客户端按需获取、单一数据入口、强类型 Schema。
- **典型应用场景**：前后端分离、大型项目数据聚合、复杂移动端应用。

---

## 二、知识体系（从全局到局部）

### 1. 全局视角

- **Schema 驱动开发**（SDL）：定义类型系统、Query/Mutation/Subscription。
- **统一入口**：所有请求通过 `/graphql`，不再区分多端多路由。
- **强类型 API**：所有数据结构、关系清晰可见。
- **聚合能力**：支持多服务聚合（如 BFF 层、中台层）。

### 2. 纵向分层与模块（以典型 React/Apollo 架构举例）

| 层级        | 责任                | 关键内容                        |
| ----------- | ------------------- | ------------------------------- |
| schema      | 类型定义            | type, query, mutation, input    |
| resolver    | 数据实现            | queryResolver, mutationResolver |
| api service | 外部服务/数据库访问 | DB 调用、REST 调用              |
| client      | 前端数据请求与缓存  | Apollo Client, urql, Relay      |
| UI 组件     | 渲染/组合数据       | React、Vue 组件                 |

---

## 三、核心语法与实用示例

### 1. Schema 定义（SDL）

```graphql
type User {
  id: ID!
  name: String!
  email: String!
}
type Query {
  users: [User!]!
  user(id: ID!): User
}
type Mutation {
  createUser(name: String!, email: String!): User
}
```

### 2. Resolver 实现

```js
const resolvers = {
  Query: {
    users: () => getUsersFromDB(),
    user: (_, { id }) => getUserById(id),
  },
  Mutation: {
    createUser: (_, { name, email }) => createUserInDB(name, email),
  },
};
```

### 3. 前端请求示例（Apollo Client with React）

```tsx
import { useQuery, gql } from "@apollo/client";
const USERS = gql`
  query {
    users {
      id
      name
      email
    }
  }
`;
function UserList() {
  const { data, loading } = useQuery(USERS);
  if (loading) return <p>Loading...</p>;
  return (
    <ul>
      {data.users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

---

## 四、横向对比：GraphQL vs REST vs gRPC

| 特性         | GraphQL       | REST        | gRPC                  |
| ------------ | ------------- | ----------- | --------------------- |
| API 入口     | 单一          | 多路由      | 多端点                |
| 数据粒度     | 客户端可控    | 固定        | 固定/流式             |
| 类型系统     | 强类型/自描述 | 弱类型      | 强类型(proto)         |
| 聚合能力     | 极强          | 通常弱      | 强（需配合 API 网关） |
| 前端开发体验 | 极佳          | 易冗余/缺失 | 工具依赖重            |
| 适合场景     | 复杂/大前端   | 简单/微服务 | 高性能内网服务        |

### 典型场景对照组

- **GraphQL**：多端一致、弹性聚合、强类型数据。
- **REST**：简单 CRUD、弱前端聚合需求。
- **gRPC**：高性能后端到后端，微服务内通讯。

---

## 五、纵向对比：GraphQL 各层分工

| 层级     | 关注点            | 最佳实践                                | 常见错误/反例                 |
| -------- | ----------------- | --------------------------------------- | ----------------------------- |
| schema   | 明确类型关系      | 仅暴露必要字段，合理设计 input/output   | 全部暴露表字段，前后端高耦合  |
| resolver | 关注数据聚合/业务 | 精简 resolver，按模块解耦，聚合多源数据 | resolver 逻辑臃肿，直接写 SQL |
| service  | 外部依赖          | 只暴露后端服务、隐藏内部实现            | resolver 直连 DB/微服务       |
| client   | 只取所需字段      | Fragment 复用、分页、懒加载、缓存       | 组件每次全量请求，未做缓存    |

---

## 六、最优与错误设计对照组

### 1. schema 层

**✅ 最优**

```graphql
type User {
  id: ID!
  name: String!
}
input UserInput {
  name: String!
  email: String!
}
type Mutation {
  createUser(input: UserInput!): User
}
```

**❌ 错误**

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  password: String!
  token: String!
  updatedAt: String!
}
# 错误点：全部表字段暴露，无安全边界
```

### 2. resolver 层

**✅ 最优**

```js
const resolvers = {
  Query: {
    users: (_, __, { dataSources }) => dataSources.userAPI.getUsers(),
  },
};
```

**❌ 错误**

```js
const resolvers = {
  Query: {
    users: async () => {
      const sql = "SELECT * FROM user";
      return db.query(sql);
    },
  },
};
// 错误点：resolver 层直接写 SQL，破坏解耦
```

### 3. client 层

**✅ 最优**

```tsx
const USER_FRAGMENT = gql`
  fragment UserFields on User {
    id
    name
  }
`;
const USERS = gql`
  query {
    users {
      ...UserFields
    }
  }
  ${USER_FRAGMENT}
`;
```

**❌ 错误**

```tsx
const USERS = gql`
  query {
    users {
      id
      name
      email
      password
      token
      updatedAt
    }
  }
`;
// 错误点：全部请求，无数据最小化，泄露敏感字段
```

---

## 七、最佳实践与建议

1. schema 明确，字段精简，合理用 input/output，禁止暴露敏感信息。
2. resolver 只聚合数据，不处理底层 SQL/业务逻辑，解耦 service 层。
3. client 只请求最小字段、善用 fragment、缓存、懒加载。
4. 采用类型生成工具（如 GraphQL Codegen）保持类型安全。
5. 配合权限与验证中间件，提升接口安全性。
6. 查询、变更、订阅权责清晰，避免业务混乱。
