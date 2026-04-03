# 第九课：模块系统

> 本课讲解 TypeScript 的模块导入导出机制

---

## 目录

1. [模块基础](#1-模块基础)
2. [命名导出](#2-命名导出)
3. [默认导出](#3-默认导出)
4. [导入语法](#4-导入语法)
5. [类型导入](#5-类型导入)
6. [重导出](#6-重导出)
7. [命名空间](#7-命名空间)
8. [实战练习](#8-实战练习)

---

## 1. 模块基础

### Go 对比

Go 使用 package 和 import：

```go
package utils

import "fmt"

func Add(a, b int) int {
    return a + b
}
```

```go
package main

import "utils"

func main() {
    result := utils.Add(1, 2)
}
```

### TypeScript 模块

每个 `.ts` 文件都是一个模块，文件内的导出对其他文件可见：

```typescript
// math.ts - 模块
export function add(a: number, b: number): number {
    return a + b
}

export const PI = 3.14159
```

```typescript
// main.ts - 使用模块
import { add, PI } from "./math.js"

console.log(add(1, 2))  // 3
console.log(PI)         // 3.14159
```

### 文件即模块

```typescript
// types.ts
export interface User {
    id: number
    name: string
}

// interfaces 和 classes 都是独立的模块成员
export class Animal {
    constructor(public name: string) {}
}
```

---

## 2. 命名导出

### 基本语法

```typescript
// 方式一：直接在声明前加 export
export const APP_NAME = "MyApp"
export function greet(name: string): string {
    return `Hello, ${name}!`
}
export class User {
    constructor(public name: string) {}
}

// 方式二：先定义，最后统一导出
const VERSION = "1.0.0"
interface Config {
    debug: boolean
}

export { VERSION, Config }
```

### as 重命名导出

```typescript
// 导出时重命名
export { User as UserClass }

// 导入时重命名
export { UserClass as User }
```

---

## 3. 默认导出

### 基本语法

```typescript
// 一个文件只能有一个 default export
export default class Application {
    run() {
        console.log("Running...")
    }
}

// 导入 default
import Application from "./app.js"
const app = new Application()
app.run()
```

### default 导出 vs 命名导出

```typescript
// default 导出
export default class Dog {
    speak() { return "Woof!" }
}

// 命名导出
export class Cat {
    speak() { return "Meow!" }
}
```

### 常见模式

```typescript
// utils.ts - 多个工具函数
export default class Utils {
    static formatDate(date: Date): string {
        return date.toISOString()
    }
}

// 同时导出命名
export function helper() { }
```

---

## 4. 导入语法

### 基本导入

```typescript
// 导入命名导出
import { add, subtract } from "./math.js"

// 导入 default
import App from "./app.js"

// 导入所有
import * as math from "./math.js"
console.log(math.add(1, 2))
console.log(math.subtract(5, 3))
```

### as 重命名

```typescript
// 导入时重命名
import { User as UserModel } from "./models.js"

// 导入 default 并重命名
import { default as App } from "./app.js"
```

### 组合导入

```typescript
// 同时导入 default 和命名
import React, { useState, useEffect } from "react"

// 导入模块和特定成员
import MyClass, { helper } from "./utils.js"
```

### 副作用导入

```typescript
// 只执行模块，不导入任何绑定
import "./polyfills.js"

// 用于初始化全局状态等
```

### 真实案例

**文件：`src/query.ts`**
```typescript
// 从其他模块导入
import { queryLoop } from './queryLoop.js'
import type { CanUseToolFn } from './hooks/useCanUseTool.js'
import type { ToolResultBlockParam, ToolUseBlock } from '@anthropic-ai/sdk/resources/index.mjs'
```

---

## 5. 类型导入

### import type

TypeScript 3.8 引入了纯类型导入：

```typescript
// 只导入类型，编译后完全移除
import type { User, Config } from "./types.js"

// 普通导入也会导入类型
import { User, type Config } from "./types.js"
```

### 为什么需要类型导入？

```typescript
// types.ts
export interface User {
    name: string
}

// user.ts - 只需要类型
import type { User } from "./types.js"

// 编译后，类型会被完全移除
// 这减少了运行时代码体积
```

### 内联类型导入

```typescript
// TypeScript 4.5+ 支持内联
import { type User, type Config } from "./types.js"
```

### 真实案例

**文件：`src/query.ts`**
```typescript
// 纯类型导入（只导入类型信息）
import type {
    ToolResultBlockParam,
    ToolUseBlock,
} from '@anthropic-ai/sdk/resources/index.mjs'

// 运行时导入（实际使用值）
import { queryLoop } from './queryLoop.js'
```

### export type

```typescript
// 只导出类型
export type { User, Config }

// 导出类/接口的纯类型签名
export class InternalService {
    secret: string = "123"
}
// 完全合法！这相当于只向外暴漏了 InternalService 的结构约束
// 外部可以定义 `let s: InternalService`，但永远无法执行 `new InternalService()`
export type { InternalService } 
```

---

## 6. 重导出

### 从其他模块导出

```typescript
// 重新导出（re-export）
export { add, subtract } from "./math.js"

// 重命名导出
export { add as addNumbers } from "./math.js"

// 导出所有
export * from "./utils.js"
```

### 实际应用

```typescript
// index.ts - 统一导出
export { User } from "./User.js"
export { Product } from "./Product.js"
export { Order } from "./Order.js"

// 使用
import { User, Product, Order } from "./models/index.js"
```

### barrel file 模式

```typescript
// models/index.ts
export { User } from "./User.js"
export { Product } from "./Product.js"
export { Order } from "./Order.js"
export { default as Database } from "./Database.js"
```

---

## 7. 命名空间

### 基本语法

```typescript
// 命名空间（早期 TypeScript 语法，现在推荐用模块）
namespace Validation {
    export interface Rules {
        required?: boolean
        minLength?: number
    }

    export function validate(value: string, rules: Rules): boolean {
        if (rules.required && value.length === 0) {
            return false
        }
        if (rules.minLength && value.length < rules.minLength) {
            return false
        }
        return true
    }
}

// 使用命名空间
const rules: Validation.Rules = { required: true, minLength: 5 }
const isValid = Validation.validate("hello", rules)
```

### namespace vs module

| 特性 | namespace | module |
|------|-----------|--------|
| 语法 | `namespace Name {}` | `export/` |
| 编译目标 | 全局/模块 | ES 模块 |
| 推荐度 | 旧（TS 1.5+）| 现代 |
| 使用场景 | 全局类型声明 | 代码模块化 |

---

## 8. 实战练习

### 练习 1：创建模块

```typescript
// math.ts
export function add(a: number, b: number): number {
    return a + b
}

export function subtract(a: number, b: number): number {
    return a - b
}

export function multiply(a: number, b: number): number {
    return a * b
}

export function divide(a: number, b: number): number | null {
    if (b === 0) return null
    return a / b
}

export const PI = 3.14159
export const E = 2.71828
```

```typescript
// main.ts
import { add, subtract, multiply, divide, PI, E } from "./math.js"

console.log(add(5, 3))       // 8
console.log(subtract(5, 3))  // 2
console.log(multiply(5, 3))  // 15
console.log(divide(5, 0))    // null
console.log(divide(5, 2))    // 2.5
console.log(PI)               // 3.14159
```

### 练习 2：类型导入

```typescript
// types.ts
export interface User {
    id: number
    name: string
    email: string
}

export interface Product {
    id: number
    name: string
    price: number
}
```

```typescript
// repository.ts - 只需要类型
import type { User, Product } from "./types.js"

class UserRepository {
    async findAll(): Promise<User[]> {
        return []
    }

    async findById(id: number): Promise<User | null> {
        return null
    }
}

class ProductRepository {
    async findAll(): Promise<Product[]> {
        return []
    }
}

export { UserRepository, ProductRepository }
```

### 练习 3：默认导出

```typescript
// logger.ts
export default class Logger {
    log(message: string): void {
        console.log(`[LOG] ${message}`)
    }

    error(message: string): void {
        console.error(`[ERROR] ${message}`)
    }

    warn(message: string): void {
        console.warn(`[WARN] ${message}`)
    }
}
```

```typescript
// app.ts
import Logger from "./logger.js"

const logger = new Logger()
logger.log("Application started")
logger.error("Something went wrong")
```

---

## 本课总结

### 核心概念

| 概念 | 语法 | 说明 |
|------|------|------|
| 命名导出 | `export const x = 1` | 导出命名成员 |
| 默认导出 | `export default Class` | 每个文件一个 |
| 命名导入 | `import { x } from "mod"` | 导入命名成员 |
| 默认导入 | `import x from "mod"` | 导入 default |
| 类型导入 | `import type { T }` | 仅导入类型 |
| 重导出 | `export { x } from "mod"` | 再导出 |
| namespace | `namespace N {}` | 旧式模块（不推荐）|

### 对比 Go

| Go | TypeScript |
|----|------------|
| `package` | 每个文件是独立模块 |
| `import "package"` | `import from "module"` |
| `package.Func` | `export function Func()` |
| 无 | `export default` |
| 无 | `import type` |

### 模块化最佳实践

```typescript
// ✅ 推荐：一个文件一个导出
// User.ts
export class User { }

// ✅ 推荐：使用 index.ts 汇总
// index.ts
export { User } from "./User.js"
export { Product } from "./Product.js"

// ✅ 推荐：显式导入需要的成员
import { User } from "./User.js"

// ❌ 不推荐：使用 namespace
namespace Utils { }

// ❌ 不推荐：default 和命名混合（混乱）
export default class Foo { }
export class Bar { }
```

---

## 下一步

学习 [第十课：高级类型](./lesson_10_advanced.md)

---

*本课完*
