# 第十课：高级类型

> 本课讲解 TypeScript 的高级类型系统

---

## 目录

1. [映射类型](#1-映射类型)
2. [条件类型](#2-条件类型)
3. [infer 关键字](#3-infer-关键字)
4. [模板字面量类型](#4-模板字面量类型)
5. [品牌类型](#5-品牌类型)
6. [递归类型](#6-递归类型)
7. [真实案例分析](#7-真实案例分析)
8. [实战练习](#8-实战练习)

---

## 1. 映射类型

### 概念解释

映射类型通过已有的类型创建新类型。

### 基本语法

```typescript
// 基本映射
type Flags = "x" | "y" | "z"
type FlagsMap = { [K in Flags]: boolean }
// {
//     x: boolean
//     y: boolean
//     z: boolean
// }
```

### Partial 和 Required

```typescript
// Partial<T> - 所有属性变为可选
type User = {
    id: number
    name: string
    email: string
}

type PartialUser = Partial<User>
// {
//     id?: number
//     name?: string
//     email?: string
// }

// Required<T> - 所有属性变为必需
type RequiredUser = Required<User>
// {
//     id: number
//     name: string
//     email: string
// }
```

### Readonly 和 Mutable

```typescript
// Readonly<T> - 所有属性变为只读
type ReadonlyUser = Readonly<User>
// {
//     readonly id: number
//     readonly name: string
//     readonly email: string
// }

// Mutable<T> - 所有属性变为可变（自定义）
type Mutable<T> = {
    -readonly [K in keyof T]: T[K]
}
```

### Pick 和 Omit

```typescript
// Pick<T, K> - 选取部分属性
type UserPreview = Pick<User, "id" | "name">
// {
//     id: number
//     name: string
// }

// Omit<T, K> - 排除部分属性
type UserWithoutEmail = Omit<User, "email">
// {
//     id: number
//     name: string
// }
```

### 自定义映射

```typescript
// 将所有属性变为字符串类型
type Stringify<T> = {
    [K in keyof T]: string
}

type StringUser = Stringify<User>
// {
//     id: string
//     name: string
//     email: string
// }

// 将所有属性变为可选并添加默认值
type WithDefaults<T, Default> = {
    [K in keyof T]?: T[K] | Default
}

type UserWithDefaults = WithDefaults<User, "unknown">
// {
//     id?: number | "unknown"
//     name?: string | "unknown"
//     email?: string | "unknown"
// }
```

---

## 2. 条件类型

### 基本语法

```typescript
// 语法：T extends U ? X : Y
type IsString<T> = T extends string ? "yes" : "no"

type A = IsString<string>  // "yes"
type B = IsString<number>  // "no"
```

### 分布式条件类型

```typescript
// 当 T 是联合类型时，条件类型会分发
type ToArray<T> = T extends any ? T[] : never

type B = ToArray<string | number>  // string[] | number[]
```

### 实际应用

```typescript
// ReturnType - 获取函数返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never

function getUser() {
    return { id: 1, name: "Alice" }
}

type User = ReturnType<typeof getUser>
// { id: number; name: string }

// Parameters - 获取函数参数类型
type Parameters<T> = T extends (...args: infer P) => any ? P : never

function greet(name: string, age: number): void {}

type GreetParams = Parameters<typeof greet>  // [name: string, age: number]
```

### Never 在条件类型中

```typescript
// never 是所有类型的子类型
type NeverExample<T> = T extends string ? T : never

type Result = NeverExample<string | number | boolean>
// string | never | never
// = string

// 过滤联合类型
type ExtractString<T> = T extends string ? T : never

type Filtered = ExtractString<string | number | "hello">
// string | "hello"
```

---

## 3. infer 关键字

### 概念解释

infer 在条件类型中声明一个类型变量，可以在 true 分支中使用。

### 基本用法

```typescript
// 推断 Promise 的内部类型
type Awaited<T> = T extends Promise<infer U> ? U : T

type A = Awaited<Promise<string>>  // string
type B = Awaited<number>           // number

// 推断数组元素类型
type ElementType<T> = T extends (infer E)[] ? E : never

type C = ElementType<string[]>  // string
type D = ElementType<number[]>  // number
```

### 实际应用

```typescript
// 推断函数返回类型
type UnpackPromise<T> = T extends () => Promise<infer R> ? R : T

async function fetchUser() {
    return { id: 1, name: "Alice" }
}

type User = UnpackPromise<typeof fetchUser>
// { id: number; name: string }

// 推断构造函数的实例类型
type InstanceType<T> = T extends new (...args: any[]) => infer R ? R : T

class UserClass {
    constructor(public name: string) {}
}

type U = InstanceType<typeof UserClass>  // UserClass
```

---

## 4. 模板字面量类型

### 基本语法

```typescript
// 模板字符串
type EventName = `on${string}`
let name: EventName = "onClick"   // OK
let name2: EventName = "onChange" // OK
let name3: EventName = "click"    // 错误！
```

### 组合类型

```typescript
// 组合多个字面量
type CSSUnit = "px" | "em" | "rem"
type Property = `${string}: ${CSSUnit}`

let prop: Property = "font-size: px"  // OK
let prop2: Property = "width: em"    // OK
```

### 实际应用

```typescript
// 类型安全的事件名
type EventHandler<E extends string> = `on${E}`

type ButtonEvents = EventHandler<"click" | "focus" | "blur">
// "onclick" | "onfocus" | "onblur"

// 类型安全的路径
type Route = `/api/${string}`

let route: Route = "/api/users"    // OK
let route2: Route = "/api/orders/1" // OK
let route3: Route = "/users"       // 错误！
```

### 提取和转换

```typescript
// Extract event name from handler
type ExtractEvent<T> = T extends `on${infer E}` ? E : never

type Event = ExtractEvent<"onclick">  // "click"
type Event2 = ExtractEvent<"click">   // never
```

---

## 5. 品牌类型

### 概念解释

品牌类型（Branded Types）用于创建名义类型（Nominal Typing），防止类型混淆。

### 问题：结构类型的问题

```typescript
// TypeScript 使用结构类型（Structural Typing）
// 这导致两种不同的 ID 类型可以互换
type UserId = number
type OrderId = number

function getUser(id: UserId): User { return null! }
function getOrder(id: OrderId): Order { return null! }

const id: UserId = 1
getOrder(id)  // 错误！TypeScript 会检查，但不是强制的
```

### 解决方案：品牌类型

```typescript
// 用交叉类型创建品牌
type UserId = number & { readonly __brand: "UserId" }
type OrderId = number & { readonly __brand: "OrderId" }

// 创建品牌值的函数
function createUserId(id: number): UserId {
    return id as UserId
}

function createOrderId(id: number): OrderId {
    return id as OrderId
}

// 使用
const userId = createUserId(1)
const orderId = createOrderId(2)

getUser(userId)    // OK
getOrder(orderId)  // OK
getUser(orderId)   // 错误！OrderId 不能赋值给 UserId
```

### 真实案例

**文件：`src/types/ids.ts`**
```typescript
// Claude Code 中的品牌类型
export type SessionId = string & { readonly __brand: 'SessionId' }
export type AgentId = string & { readonly __brand: 'AgentId' }
export type ConversationId = string & { readonly __brand: 'ConversationId' }

// 创建函数
export function asSessionId(id: string): SessionId {
    return id as SessionId
}

export function asAgentId(id: string): AgentId {
    return id as AgentId
}

// 验证函数
export function toAgentId(s: string): AgentId | null {
    const pattern = /^agent:[a-zA-Z0-9]+$/
    return pattern.test(s) ? (s as AgentId) : null
}

// 使用
const sessionId: SessionId = asSessionId("sess_abc123")
const agentId: AgentId = asAgentId("agent_xyz")

// sessionId = agentId  // 错误！类型不兼容
```

---

## 6. 递归类型

### 基本语法

```typescript
// 递归类型定义
interface TreeNode<T> {
    value: T
    children: TreeNode<T>[]
}

// 使用
type BinaryTree = {
    value: number
    left?: BinaryTree
    right?: BinaryTree
}

const tree: BinaryTree = {
    value: 1,
    left: {
        value: 2,
        left: { value: 4 },
        right: { value: 5 }
    },
    right: { value: 3 }
}
```

### JSON 类型

```typescript
// 递归定义 JSON
type JSONValue =
    | string
    | number
    | boolean
    | null
    | JSONValue[]
    | { [key: string]: JSONValue }

interface JSONNode {
    [key: string]: JSONValue
}

const json: JSONNode = {
    name: "Alice",
    age: 25,
    hobbies: ["reading", "coding"],
    address: {
        city: "Beijing",
        zip: "100000"
    },
    married: null
}
```

---

## 7. 真实案例分析

### 案例 1：工具类型组合

**文件：`src/Tool.ts`**
```typescript
// 工具参数类型的构建
type DefaultableToolKeys =
    | "description"
    | "input"
    | "output"

type BuiltTool<D> = Omit<D, DefaultableToolKeys> & {
    [K in DefaultableToolKeys]-?: K extends keyof D
        ? undefined extends D[K]
            ? ToolDefaults[K]
            : D[K]
        : ToolDefaults[K]
}
```

### 案例 2：Zod 类型推断

**文件：`src/services/mcp/types.ts`**
```typescript
import { z } from 'zod'

// 定义 Schema
const McpServerConfigSchema = z.object({
    name: z.string(),
    command: z.string(),
    args: z.array(z.string()).optional(),
    env: z.record(z.string()).optional()
})

// 从 Schema 推断类型
export type McpServerConfig = z.infer<typeof McpServerConfigSchema>
// {
//     name: string
//     command: string
//     args?: string[]
//     env?: Record<string, string>
// }
```

### 案例 3：satisfies 操作符

**文件：`src/vim/types.ts`**
```typescript
// satisfies 验证类型但保留字面量推断
const OPERATORS = {
    motion: "move",
    change: "modify",
    delete: "remove"
} as const satisfies Record<string, string>

// 不使用 satisfies 时：
const WITHOUT_SATISFIES = {
    motion: "move",
    change: "modify"
}
// WITHOUT_SATISFIES.motion 的类型是 string

// 使用 satisfies 时：
const WITH_SATISFIES = {
    motion: "move",
    change: "modify"
} as const satisfies Record<string, string>
// WITH_SATISFIES.motion 的类型是 "move"（字面量）
```

---

## 8. 实战练习

### 练习 1：实现 Partial

```typescript
// 手动实现 Partial
type MyPartial<T> = {
    [K in keyof T]?: T[K]
}

interface User {
    id: number
    name: string
    email: string
}

type PartialUser = MyPartial<User>
// {
//     id?: number
//     name?: string
//     email?: string
// }
```

### 练习 2：实现 Pick

```typescript
// 手动实现 Pick
type MyPick<T, K extends keyof T> = {
    [P in K]: T[P]
}

type UserPreview = MyPick<User, "id" | "name">
// {
//     id: number
//     name: string
// }
```

### 练习 3：条件类型

```typescript
// 实现 UnwrapPromise
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T

type A = UnwrapPromise<Promise<string>>  // string
type B = UnwrapPromise<number>           // number
type C = UnwrapPromise<Promise<{ id: number }>>  // { id: number }
```

### 练习 4：品牌类型

```typescript
// 实现品牌类型
type Brand<T, B> = T & { readonly __brand: B }

type UserId = Brand<number, "UserId">
type OrderId = Brand<number, "OrderId">

function createUserId(id: number): UserId {
    return id as UserId
}

function createOrderId(id: number): OrderId {
    return id as OrderId
}

function getUserById(id: UserId): void {}
function getOrderById(id: OrderId): void {}

const uid = createUserId(1)
const oid = createOrderId(2)

getUserById(uid)   // OK
getOrderById(oid)  // OK
getUserById(oid)   // 错误！OrderId 不是 UserId
getOrderById(uid)  // 错误！
```

---

## 本课总结

### 核心概念

| 概念 | 语法 | 说明 |
|------|------|------|
| 映射类型 | `[K in keyof T]: T[K]` | 遍历类型属性 |
| Partial | `Partial<T>` | 所有属性可选 |
| Required | `Required<T>` | 所有属性必需 |
| Pick | `Pick<T, K>` | 选取属性 |
| Omit | `Omit<T, K>` | 排除属性 |
| 条件类型 | `T extends U ? X : Y` | 条件映射 |
| infer | `infer R` | 类型推断变量 |
| 模板字面量 | `` `${string}.js` `` | 字符串字面量类型 |
| 品牌类型 | `T & { __brand: B }` | 名义类型 |
| 递归类型 | `type JSON = ...JSON...` | 自引用类型 |

### 内置工具类型速查

| 类型 | 说明 |
|------|------|
| `Partial<T>` | 所有属性可选 |
| `Required<T>` | 所有属性必需 |
| `Readonly<T>` | 所有属性只读 |
| `Pick<T, K>` | 选取属性 |
| `Omit<T, K>` | 排除属性 |
| `Record<K, V>` | 键值映射 |
| `Exclude<T, U>` | 排除类型 |
| `Extract<T, U>` | 提取类型 |
| `NonNullable<T>` | 移除 null/undefined |
| `ReturnType<T>` | 函数返回类型 |
| `Parameters<T>` | 函数参数类型 |
| `InstanceType<T>` | 构造函数实例类型 |

### 对比 Go

| Go | TypeScript |
|----|------------|
| 无泛型工具类型 | `Partial<T>`, `Pick<T>` 等 |
| 无条件类型 | `T extends U ? X : Y` |
| 无模板字面量类型 | `` `${T}` `` |
| 无品牌类型 | `T & { __brand }` |
| 无递归类型 | `type JSON = ...JSON...` |

---

## 教程完成！

你现在已经掌握了阅读 Claude Code TypeScript 源码所需的基础知识。建议：

1. **复习重点**：
   - 区分联合（第六课）— Claude Code 最常用
   - 泛型约束（第四课）— Tool.ts 的基础
   - async/await（第七课）— query.ts 的核心

2. **开始阅读源码**：
   - 先读 `src/types/` 理解数据结构
   - 再读 `src/tools/` 理解工具系统
   - 最后读 `src/query.ts` 理解核心逻辑

3. **实践**：
   - 尝试编写简单的 TypeScript 代码
   - 使用 `tsc --noEmit` 检查类型错误
   - 在 IDE 中查看类型推断

祝你学习愉快！

---

*教程完*
