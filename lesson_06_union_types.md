# 第六课：联合类型与区分联合

> 这是 TypeScript 最强大的模式之一，Claude Code 源码中大量使用

---

## 目录

1. [联合类型基础](#1-联合类型基础)
2. [类型收窄](#2-类型收窄)
3. [区分联合（Tagged Union）](#3-区分联合tagged-union)
4. [交叉类型](#4-交叉类型)
5. [工具类型](#5-工具类型)
6. [真实案例分析](#6-真实案例分析)
7. [实战练习](#7-实战练习)

---

## 1. 联合类型基础

### 概念解释

联合类型（Union Type）表示一个值可以是多种类型之一。用 `|` 分隔。

### Go 对比

Go 没有直接的联合类型，但可以用以下方式模拟：

```go
// 方式一：使用结构体 + 布尔标志
type Result struct {
    Value  any
    IsError bool
    Error   error
}

// 方式二：使用接口
type Any interface {
    isAny()
}
```

### TypeScript 联合类型

```typescript
// 基本语法：type = A | B | C
let value: string | number
value = "hello"   // OK
value = 123       // OK
// value = true   // 错误！

// 函数参数联合类型
function printId(id: number | string): void {
    console.log(`ID: ${id}`)
}

printId(123)      // OK
printId("abc")    // OK
```

### 字面量联合类型

```typescript
// 只能是特定的字符串
type Direction = "north" | "south" | "east" | "west"

let dir: Direction
dir = "north"   // OK
dir = "up"      // 错误！"up" 不是有效方向

// 只能是特定的数字
type Status = 200 | 301 | 404 | 500

function getStatus(): Status {
    return 200
}
```

### 实际应用

```typescript
// 处理多种输入类型
function processInput(input: string | number): string {
    if (typeof input === "string") {
        // 在这个分支，input 是 string
        return input.toUpperCase()
    } else {
        // 在这个分支，input 是 number
        return input.toFixed(2)
    }
}

console.log(processInput("hello"))  // "HELLO"
console.log(processInput(123.456))  // "123.46"
```

---

## 2. 类型收窄

### 概念解释

类型收窄（Narrowing）是 TypeScript 自动推断联合类型在特定分支下的具体类型。

### 常见收窄方式

#### typeof 收窄

```typescript
function example(x: string | number): void {
    if (typeof x === "string") {
        // x 是 string
        console.log(x.toUpperCase())
    } else {
        // x 是 number
        console.log(x.toFixed(2))
    }
}
```

#### 平等性收窄

```typescript
type Result =
    | { type: "success"; value: number }
    | { type: "error"; message: string }

function handle(result: Result): void {
    if (result.type === "success") {
        // result 是 { type: "success"; value: number }
        console.log(result.value)
    } else {
        // result 是 { type: "error"; message: string }
        console.error(result.message)
    }
}
```

#### instanceof 收窄

```typescript
class Dog {
    bark(): void { console.log("Woof!") }
}

class Cat {
    meow(): void { console.log("Meow!") }
}

function speak(animal: Dog | Cat): void {
    if (animal instanceof Dog) {
        animal.bark()
    } else {
        animal.meow()
    }
}
```

#### in 操作符收窄

```typescript
interface Fish {
    swim(): void
}

interface Bird {
    fly(): void
}

function move(animal: Fish | Bird): void {
    if ("swim" in animal) {
        animal.swim()
    } else {
        animal.fly()
    }
}
```

### 可辨识联合（Discriminated Unions）

这是最强大的收窄方式：

```typescript
// 用一个公共的 "type" 字段区分
interface Success {
    type: "success"  // 鉴别属性
    value: number
}

interface Error {
    type: "error"    // 鉴别属性
    message: string
}

type Result = Success | Error

function handle(result: Result): void {
    switch (result.type) {
        case "success":
            console.log(result.value)  // result 是 Success
            break
        case "error":
            console.error(result.message)  // result 是 Error
            break
    }
}
```

---

## 3. 区分联合（Tagged Union）

### 概念解释

区分联合（Discriminated Union）是 TypeScript 最强大的模式之一，类似于 Rust 的 `enum` 或 Go 的 if-err 模式，但更强大。

### Go 对比

Go 的典型模式：

```go
// Go 用返回 error 实现类似功能
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

result, err := divide(10, 0)
if err != nil {
    fmt.Println("Error:", err)
} else {
    fmt.Println("Result:", result)
}
```

### TypeScript 区分联合

```typescript
// 定义区分联合
type Result =
    | { type: "success"; data: string }
    | { type: "error"; error: string }
    | { type: "loading" }
    | { type: "idle" }

// 穷尽检查：TypeScript 会确保处理所有情况
function handleResult(result: Result): string {
    switch (result.type) {
        case "success":
            return `Data: ${result.data}`
        case "error":
            return `Error: ${result.error}`
        case "loading":
            return "Loading..."
        case "idle":
            return "Idle"
    }
    // 如果忘记处理某个 case，编译器会报错
}
```

### 真实案例

**文件：`src/query.ts`**
```typescript
// Claude Code 中的事件类型
export type CompactProgressEvent =
    | { type: 'hooks_start'; hookType: 'pre_compact' | 'post_compact' | 'session_start' }
    | { type: 'compact_start' }
    | { type: 'compact_end' }

// 使用
function handleEvent(event: CompactProgressEvent): void {
    switch (event.type) {
        case 'hooks_start':
            console.log(`Starting ${event.hookType} hooks`)
            break
        case 'compact_start':
            console.log("Compacting...")
            break
        case 'compact_end':
            console.log("Compaction complete")
            break
    }
}
```

### 嵌套区分联合

```typescript
// 更复杂的例子
type Node =
    | { type: "text"; value: string }
    | { type: "element"; tag: string; children: Node[] }
    | { type: "comment"; value: string }

function render(node: Node): string {
    switch (node.type) {
        case "text":
            return node.value
        case "element":
            const children = node.children.map(render).join("")
            return `<${node.tag}>${children}</${node.tag}>`
        case "comment":
            return `<!--${node.value}-->`
    }
}

// 使用
const ast: Node = {
    type: "element",
    tag: "div",
    children: [
        { type: "text", value: "Hello" },
        { type: "text", value: " World" }
    ]
}

console.log(render(ast))
// <div>Hello World</div>
```

### 区分联合的方法

```typescript
// 给区分联合添加方法
type Result<T> =
    | { type: "success"; data: T }
    | { type: "error"; message: string }

class ResultImpl<T> {
    constructor(private result: Result<T>) {}

    isSuccess(): boolean {
        return this.result.type === "success"
    }

    isError(): boolean {
        return this.result.type === "error"
    }

    getValue(): T | undefined {
        if (this.result.type === "success") {
            return this.result.data
        }
        return undefined
    }

    getError(): string | undefined {
        if (this.result.type === "error") {
            return this.result.message
        }
        return undefined
    }
}
```

---

## 4. 交叉类型

### 概念解释

交叉类型（Intersection Type）组合多个类型为一个。用 `&` 连接。

### Go 对比

Go 没有交叉类型，但可以用嵌入模拟：

```go
type A struct {
    FieldA string
}

type B struct {
    FieldB int
}

type AB struct {
    A
    B
}
```

### TypeScript 交叉类型

```typescript
interface A {
    a: string
}

interface B {
    b: number
}

// 交叉类型
type AB = A & B

const obj: AB = {
    a: "hello",
    b: 123
}
```

### 实际应用

```typescript
// 合并配置
type Config = {
    apiUrl: string
    timeout: number
}

type AuthConfig = {
    token: string
    refreshToken: string
}

type FullConfig = Config & AuthConfig

const config: FullConfig = {
    apiUrl: "https://api.example.com",
    timeout: 5000,
    token: "abc123",
    refreshToken: "xyz789"
}
```

### 交叉类型 vs 联合类型

```typescript
// 联合类型：A 或 B
type Union = A | B

// 交叉类型：A 和 B
type Intersection = A & B
```

### 真实案例

**文件：`src/Tool.ts`**
```typescript
// 工具类型组合
type Tool<
    Input extends AnyObject = AnyObject,
    Output = unknown,
    P extends ToolProgressData = ToolProgressData,
> = {
    readonly name: string
    readonly description: string
} & ToolCore<Input, Output, P>
```

---

## 5. 工具类型

### Record<K, V>

创建键值对类型：

```typescript
// 语法：Record<Keys, Values>
type UserRoles = Record<string, "admin" | "user" | "guest">

const roles: UserRoles = {
    alice: "admin",
    bob: "user",
    charlie: "guest"
}

// 键必须是 string，值必须是指定的三种之一
```

### Extract<T, U>

提取 T 中可以赋值给 U 的类型：

```typescript
type T = "a" | "b" | "c" | "d"
type U = "a" | "c"

type Extracted = Extract<T, U>
// "a" | "c"
```

### Exclude<T, U>

从 T 中排除可以赋值给 U 的类型：

```typescript
type T = "a" | "b" | "c" | "d"
type U = "a" | "c"

type Excluded = Exclude<T, U>
// "b" | "d"
```

### NonNullable<T>

移除 null 和 undefined：

```typescript
type T = string | null | undefined | number

type NonNull = NonNullable<T>
// string | number
```

### ReturnType<T>

获取函数返回类型：

```typescript
function getUser() {
    return { id: 1, name: "Alice" }
}

type User = ReturnType<typeof getUser>
// { id: number; name: string }
```

### Parameters<T>

获取函数参数类型：

```typescript
function doSomething(a: string, b: number, c: boolean) {}

type Params = Parameters<typeof doSomething>
// [a: string, b: number, c: boolean]
```

---

## 6. 真实案例分析

### 案例 1：ValidationResult

**文件：`src/Tool.ts`**
```typescript
// 验证结果类型
export type ValidationResult =
    | { result: true }
    | { result: false; message: string; errorCode: number }

// 使用
function validate(input: unknown): ValidationResult {
    if (typeof input !== "string") {
        return {
            result: false,
            message: "Input must be a string",
            errorCode: 400
        }
    }
    return { result: true }
}

const result = validate("hello")
if (result.result) {
    console.log("Valid!")
} else {
    console.error(`Error ${result.errorCode}: ${result.message}`)
}
```

### 案例 2：StreamEvent

```typescript
// 流事件类型
type StreamEvent =
    | { type: "start"; timestamp: number }
    | { type: "data"; content: string }
    | { type: "error"; message: string }
    | { type: "end"; duration: number }

// 事件处理
function handleStreamEvent(event: StreamEvent): void {
    switch (event.type) {
        case "start":
            console.log(`Stream started at ${event.timestamp}`)
            break
        case "data":
            console.log(`Received: ${event.content}`)
            break
        case "error":
            console.error(`Stream error: ${event.message}`)
            break
        case "end":
            console.log(`Stream ended after ${event.duration}ms`)
            break
    }
}
```

### 案例 3：状态机

```typescript
// HTTP 请求状态机
type RequestState<T> =
    | { status: "idle" }
    | { status: "loading" }
    | { status: "success"; data: T }
    | { status: "error"; error: Error }

class DataFetcher<T> {
    private state: RequestState<T> = { status: "idle" }

    async fetch(fn: () => Promise<T>): Promise<void> {
        this.state = { status: "loading" }

        try {
            const data = await fn()
            this.state = { status: "success", data }
        } catch (error) {
            this.state = { status: "error", error: error as Error }
        }
    }

    getState(): RequestState<T> {
        return this.state
    }
}

// 使用
const fetcher = new DataFetcher<string>()
fetcher.fetch(async () => {
    return "Hello, World!"
})
```

---

## 7. 实战练习

### 练习 1：实现 Result 类型

```typescript
// 实现一个 Result 类型，支持 success 和 error 两种状态

type Result<T, E = Error> =
    | { ok: true; value: T }
    | { ok: false; error: E }

function divide(a: number, b: number): Result<number, string> {
    if (b === 0) {
        return { ok: false, error: "Division by zero" }
    }
    return { ok: true, value: a / b }
}

const result = divide(10, 2)
if (result.ok) {
    console.log(`Result: ${result.value}`)
} else {
    console.error(`Error: ${result.error}`)
}
```

### 练习 2：实现事件系统

```typescript
// 实现一个简单的事件系统

type Event =
    | { type: "click"; x: number; y: number }
    | { type: "keydown"; key: string }
    | { type: "scroll"; deltaY: number }

function handleEvent(event: Event): string {
    switch (event.type) {
        case "click":
            return `Clicked at (${event.x}, ${event.y})`
        case "keydown":
            return `Pressed: ${event.key}`
        case "scroll":
            return `Scrolled ${event.deltaY > 0 ? "down" : "up"}`
    }
}

console.log(handleEvent({ type: "click", x: 100, y: 200 }))
console.log(handleEvent({ type: "keydown", key: "Enter" }))
console.log(handleEvent({ type: "scroll", deltaY: 100 }))
```

### 练习 3：实现状态机

```typescript
// 实现一个订单状态机

type OrderState =
    | { state: "pending" }
    | { state: "confirmed"; orderId: string }
    | { state: "shipped"; trackingNumber: string }
    | { state: "delivered" }
    | { state: "cancelled"; reason: string }

class Order {
    private status: OrderState = { state: "pending" }

    getStatus(): OrderState {
        return this.status
    }

    confirm(orderId: string): void {
        if (this.status.state !== "pending") {
            throw new Error("Can only confirm pending orders")
        }
        this.status = { state: "confirmed", orderId }
    }

    ship(trackingNumber: string): void {
        if (this.status.state !== "confirmed") {
            throw new Error("Can only ship confirmed orders")
        }
        this.status = { state: "shipped", trackingNumber }
    }

    deliver(): void {
        if (this.status.state !== "shipped") {
            throw new Error("Can only deliver shipped orders")
        }
        this.status = { state: "delivered" }
    }

    cancel(reason: string): void {
        if (this.status.state === "delivered") {
            throw new Error("Cannot cancel delivered orders")
        }
        this.status = { state: "cancelled", reason }
    }
}

// 测试
const order = new Order()
console.log(order.getStatus())  // { state: "pending" }

order.confirm("ORD-123")
console.log(order.getStatus())  // { state: "confirmed", orderId: "ORD-123" }

order.ship("TRK-456")
console.log(order.getStatus())  // { state: "shipped", trackingNumber: "TRK-456" }
```

---

## 本课总结

### 核心概念

| 概念 | 语法 | 说明 |
|------|------|------|
| 联合类型 | `A \| B` | A 或 B |
| 交叉类型 | `A & B` | A 和 B |
| 类型收窄 | `if (typeof x === "string")` | 缩小类型范围 |
| 区分联合 | `{ type: "a"; ... } \| { type: "b"; ... }` | 带鉴别属性的联合 |
| 穷尽检查 | switch/case | 确保处理所有情况 |
| Record | `Record<K, V>` | 键值对 |
| Extract | `Extract<T, U>` | 提取类型 |
| Exclude | `Exclude<T, U>` | 排除类型 |

### 对比 Go

| Go | TypeScript |
|----|------------|
| `error` 返回值 | `Result<T, E>` 联合类型 |
| if err != nil | `if (result.ok)` |
| 无 | 区分联合 + switch |
| struct 嵌入 | 交叉类型 `&` |

### 为什么区分联合如此重要？

```typescript
// Go 的问题：错误可能被忽略
result := divide(10, 0)
// result 可能是 0，error 被忽略

// TypeScript：必须处理所有情况
const result = divide(10, 0)
if (result.ok) {
    console.log(result.value)
} else {
    console.error(result.error)
}
// 忘记处理？编译器会警告！
```

---

## 下一步

学习 [第七课：异步编程](./lesson_07_async.md)

---

*本课完*
