# 第四课：泛型

> 本课讲解 TypeScript 泛型——类型系统最强大的特性之一

---

## 目录

1. [泛型简介](#1-泛型简介)
2. [泛型函数](#2-泛型函数)
3. [泛型接口](#3-泛型接口)
4. [泛型类](#4-泛型类)
5. [泛型约束](#5-泛型约束)
6. [多类型参数](#6-多类型参数)
7. [泛型工具类型](#7-泛型工具类型)
8. [条件类型](#8-条件类型)
9. [真实案例分析](#9-真实案例分析)
10. [实战练习](#10-实战练习)

---

## 1. 泛型简介

### 什么是泛型？

泛型（Generics）允许你编写**可复用**的代码，同时保持**类型安全**。

### Go 对比

Go 在 1.18 之前没有泛型，需要用 `interface{}` 和类型断言模拟：

```go
// Go 1.18 之前的模拟
func Min(a interface{}, b interface{}) interface{} {
    // 需要类型断言
}

// Go 1.18+ 泛型
func Min[T any](a T, b T) T {
    if a < b {
        return a
    }
    return b
}
```

### TypeScript 泛型

```typescript
// TypeScript 泛型
function min<T>(a: T, b: T): T {
    if (a < b) {
        return a
    }
    return b
}

console.log(min(1, 2))           // 1
console.log(min("a", "b"))       // "a"
console.log(min<number>(1, 2))  // 1（显式指定类型）
```

### 为什么需要泛型？

```typescript
// 问题：不使用泛型
function identity(arg: any): any {
    return arg
}

const num = identity(5)      // 返回 any，丢失类型信息
const str = identity("hello") // 返回 any，丢失类型信息

// num.toUpperCase()  // 错误！num 是 any，不知道有 toUpperCase

// 解决方案：使用泛型
function identity<T>(arg: T): T {
    return arg
}

const num = identity(5)        // T 被推断为 number
const str = identity("hello")  // T 被推断为 string

num.toFixed(2)  // OK！编译器知道是 number
str.toUpperCase()  // OK！编译器知道是 string
```

---

## 2. 泛型函数

### 基本语法

```typescript
// 语法：function 函数名<T>(参数: T): T { }
function firstElement<T>(arr: T[]): T | undefined {
    return arr[0]
}

const nums = [1, 2, 3]
const strs = ["a", "b", "c"]

console.log(firstElement(nums))  // 1
console.log(firstElement(strs))  // "a"
```

### 显式类型参数

```typescript
// TypeScript 可以自动推断类型
const result1 = firstElement([1, 2, 3])  // number | undefined

// 也可以显式指定
const result2 = firstElement<number>([1, 2, 3])  // number | undefined
```

### 多个类型参数

```typescript
// 两个泛型参数
function pair<T, U>(first: T, second: U): [T, U] {
    return [first, second]
}

const p1 = pair(1, "one")           // [number, string]
const p2 = pair(true, 42)           // [boolean, number]

// 类型推断
console.log(p1[0])  // number
console.log(p1[1])  // string
```

### 泛型函数类型

```typescript
// 定义泛型函数类型
type Transform<T> = (input: T) => T

// 使用
const double: Transform<number> = (n) => n * 2
const upper: Transform<string> = (s) => s.toUpperCase()

console.log(double(5))    // 10
console.log(upper("hi"))  // "HI"
```

---

## 3. 泛型接口

### 基本语法

```typescript
// 泛型接口
interface Container<T> {
    value: T
    getValue(): T
    setValue(value: T): void
}

const numContainer: Container<number> = {
    value: 42,
    getValue() { return this.value },
    setValue(v) { this.value = v }
}

console.log(numContainer.getValue())  // 42
numContainer.setValue(100)
console.log(numContainer.getValue())  // 100
```

### 多类型参数

```typescript
interface Dictionary<K extends string | number, V> {
    [key: K]: V
}

const dict: Dictionary<string, number> = {
    "one": 1,
    "two": 2
}

console.log(dict["one"])  // 1
```

### 真实案例

**文件：`src/Tool.ts`**
```typescript
// Claude Code 中的泛型工具定义
export type Tool<
    Input extends AnyObject = AnyObject,
    Output = unknown,
    P extends ToolProgressData = ToolProgressData,
> = {
    readonly name: string
    readonly description: string
    readonly input: () => Input
    readonly output: () => Output
    call(args: Input): Promise<ToolResult<Output>>
    // ...
}

// 使用
type ReadFileInput = { path: string }
type ReadFileOutput = { content: string }

interface ReadFileTool extends Tool<ReadFileInput, ReadFileOutput> {
    readonly name: "read_file"
}
```

---

## 4. 泛型类

### 基本语法

```typescript
// 泛型类
class Box<T> {
    private content: T

    constructor(initial: T) {
        this.content = initial
    }

    get(): T {
        return this.content
    }

    set(value: T): void {
        this.content = value
    }
}

const numberBox = new Box<number>(123)
console.log(numberBox.get())  // 123
numberBox.set(456)
console.log(numberBox.get())  // 456

const stringBox = new Box("hello")
console.log(stringBox.get())  // "hello"
```

### 真实案例

**文件：`src/utils/CircularBuffer.ts`**
```typescript
// Claude Code 中的环形缓冲区实现
export class CircularBuffer<T> {
    private buffer: T[]
    private head = 0
    private size = 0

    constructor(private capacity: number) {
        this.buffer = new Array(capacity)
    }

    push(item: T): void {
        this.buffer[this.head] = item
        this.head = (this.head + 1) % this.capacity
        if (this.size < this.capacity) {
            this.size++
        }
    }

    get(index: number): T | undefined {
        if (index < 0 || index >= this.size) {
            return undefined
        }
        return this.buffer[(this.head - this.size + index + this.capacity) % this.capacity]
    }

    getSize(): number {
        return this.size
    }

    isEmpty(): boolean {
        return this.size === 0
    }

    isFull(): boolean {
        return this.size === this.capacity
    }
}

// 使用
const buffer = new CircularBuffer<number>(3)
buffer.push(1)
buffer.push(2)
buffer.push(3)
console.log(buffer.get(0))  // 1
console.log(buffer.getSize())  // 3
```

### 类 vs 接口的泛型

```typescript
// 泛型接口
interface Pair<T, U> {
    first: T
    second: U
}

// 泛型类
class DataStore<T> {
    private data: T[] = []

    add(item: T): void {
        this.data.push(item)
    }

    get(index: number): T | undefined {
        return this.data[index]
    }

    getAll(): T[] {
        return [...this.data]
    }
}

// 接口可以继承泛型类，但类不能继承接口
```

---

## 5. 泛型约束

### 概念解释

泛型约束（Generic Constraint）限制泛型的范围，用 `extends` 关键字。

### 为什么需要约束？

```typescript
// 无约束：T 可以是任意类型
function getProperty<T, K>(obj: T, key: K) {
    return obj[key]  // 错误！T 上不存在属性 K
}

// 有约束：K 必须是 T 的键
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key]
}

const person = { name: "Alice", age: 25 }

console.log(getProperty(person, "name"))  // "Alice"
console.log(getProperty(person, "age"))   // 25
console.log(getProperty(person, "email")) // 错误！"email" 不在 person 中
```

### extends 约束

```typescript
// 约束 T 必须有 name 属性
interface HasName {
    name: string
}

function greet<T extends HasName>(entity: T): string {
    return `Hello, ${entity.name}!`
}

console.log(greet({ name: "Alice", age: 25 }))  // "Hello, Alice!"
console.log(greet({ name: "Bob" }))              // "Hello, Bob!"
// greet({ age: 30 })  // 错误！name 是必需的

// 约束 T 必须是对象类型
function process<T extends object>(value: T): void {
    console.log(JSON.stringify(value))
}

process({ a: 1 })      // OK
process([1, 2, 3])     // OK
// process("hello")    // 错误！string 不是 object
```

### 多重约束

```typescript
// 多个约束用 & 连接
interface Printable {
    print(): void
}

interface Serializable {
    serialize(): string
}

function process<T extends Printable & Serializable>(obj: T): void {
    obj.print()
    console.log(obj.serialize())
}
```

### 真实案例

**文件：`src/Tool.ts`**
```typescript
// 约束 Input 必须是对象类型
export type Tool<
    Input extends AnyObject = AnyObject,  // Input 必须是 AnyObject
    Output = unknown,
    P extends ToolProgressData = ToolProgressData,  // P 必须是 ToolProgressData
> = {
    readonly name: string
    readonly description: string
    readonly input: () => Input
    readonly output: () => Output
    // ...
}

// AnyObject 是什么？
type AnyObject = Record<string, unknown>
```

### keyof 约束

```typescript
// keyof 取出对象的所有键
interface User {
    id: number
    name: string
    email: string
}

type UserKeys = keyof User  // "id" | "name" | "email"

// 结合泛型
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key]
}

// 获取多个属性
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
    const result = {} as Pick<T, K>
    for (const key of keys) {
        result[key] = obj[key]
    }
    return result
}

const user = { id: 1, name: "Alice", email: "alice@example.com" }
console.log(pick(user, ["name", "email"]))
// { name: "Alice", email: "alice@example.com" }
```

---

## 6. 多类型参数

### 基本用法

```typescript
// 多个泛型参数
function map<T, U>(arr: T[], fn: (item: T) => U): U[] {
    return arr.map(fn)
}

const numbers = [1, 2, 3]
const strings = map(numbers, n => n.toString())
console.log(strings)  // ["1", "2", "3"]
```

### 类型参数默认值

```typescript
// 泛型参数可以有默认值
interface Response<T = any, E = Error> {
    success: boolean
    data?: T
    error?: E
}

// 使用默认
const r1: Response = { success: true }

// 指定第一个类型
const r2: Response<string> = { success: true, data: "hello" }

// 指定两个类型
const r3: Response<number, TypeError> = {
    success: false,
    error: new TypeError("failed")
}
```

### 泛型推断

```typescript
// TypeScript 可以推断多个类型参数
function apply<T, U>(value: T, fn: (t: T) => U): U {
    return fn(value)
}

const result = apply(5, n => n * 2)  // T=number, U=number, result=10
const str = apply("hello", s => s.toUpperCase())  // T=string, U=string, str="HELLO"
```

---

## 7. 泛型工具类型

### 内置工具类型

TypeScript 提供了一些内置的工具类型：

#### Partial<T> - 所有属性可选

```typescript
interface User {
    id: number
    name: string
    email: string
}

// 所有属性变成可选
type PartialUser = Partial<User>
// 等价于
type PartialUser = {
    id?: number
    name?: string
    email?: string
}

// 使用场景：更新用户时不需要全部字段
function updateUser(id: number, updates: Partial<User>): User {
    // 合并更新
    const existing = getUser(id)
    return { ...existing, ...updates }
}
```

#### Required<T> - 所有属性必需

```typescript
interface Config {
    url?: string
    timeout?: number
}

// 所有属性变成必需
type RequiredConfig = Required<Config>
// 等价于
type RequiredConfig = {
    url: string
    timeout: number
}
```

#### Readonly<T> - 所有属性只读

```typescript
interface User {
    name: string
    age: number
}

// 所有属性变成只读
type ReadonlyUser = Readonly<User>
// 等价于
type ReadonlyUser = {
    readonly name: string
    readonly age: number
}
```

#### Pick<T, K> - 选取属性

```typescript
interface User {
    id: number
    name: string
    email: string
    password: string
}

// 只选取部分属性
type UserPreview = Pick<User, "id" | "name">
// 等价于
type UserPreview = {
    id: number
    name: string
}
```

#### Omit<T, K> - 排除属性

```typescript
interface User {
    id: number
    name: string
    email: string
    password: string
}

// 排除某些属性
type PublicUser = Omit<User, "password">
// 等价于
type PublicUser = {
    id: number
    name: string
    email: string
}
```

### 真实案例

**文件：`src/utils/tasks.ts`**
```typescript
// Partial + Omit 组合使用
type TaskData = Omit<Task, 'id'>           // Task 去掉 id
updates: Partial<Omit<Task, 'id'>>          // 可选地更新任意 Task 属性（除 id）

// 这是常见的"部分更新"模式
function updateTask(id: string, updates: Partial<Omit<Task, 'id'>>): Task {
    const existing = getTask(id)
    return { ...existing, ...updates, id }
}
```

---

## 8. 条件类型

### 基本语法

```typescript
// T extends U ? X : Y
// 如果 T 是 U 的子类型，返回 X，否则返回 Y

type IsString<T> = T extends string ? true : false

type A = IsString<string>   // true
type B = IsString<number>   // false

// 有用的例子：提取函数返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never

function getUser() {
    return { id: 1, name: "Alice" }
}

type User = ReturnType<typeof getUser>  // { id: number; name: string }
```

### 分布式条件类型

```typescript
// 当 T 是联合类型时，条件类型会分布式处理
type ToArray<T> = T extends any ? T[] : never

type StrOrNumArray = ToArray<string | number>
// 等价于 ToArray<string> | ToArray<number>
// 结果是 string[] | number[]

// 提取联合类型的成员
type Extract<T, U> = T extends U ? T : never

type Colors = "red" | "green" | "blue"
type PrimaryColors = "red" | "blue"

type ExtractResult = Extract<Colors, PrimaryColors>  // "red" | "blue"
```

### 实战应用

```typescript
// 移除可选属性中的 undefined
type Defined<T> = T extends undefined ? never : T

type User = {
    id?: number
    name?: string
}

type DefinedUser = {
    [K in keyof User]: Defined<User[K]>
}
// { id: number; name: string }（可选变必需）
```

---

## 9. 真实案例分析

### 案例 1：CircularBuffer 完整实现

**文件：`src/utils/CircularBuffer.ts`**

```typescript
// 泛型类实现
export class CircularBuffer<T> {
    private buffer: T[]
    private head = 0
    private size = 0

    constructor(private capacity: number) {
        this.buffer = new Array(capacity)
    }

    push(item: T): void {
        this.buffer[this.head] = item
        this.head = (this.head + 1) % this.capacity
        if (this.size < this.capacity) {
            this.size++
        }
    }

    get(index: number): T | undefined {
        if (index < 0 || index >= this.size) {
            return undefined
        }
        return this.buffer[(this.head - this.size + index + this.capacity) % this.capacity]
    }

    getSize(): number {
        return this.size
    }

    isEmpty(): boolean {
        return this.size === 0
    }

    isFull(): boolean {
        return this.size === this.capacity
    }
}

// 使用泛型
const numBuffer = new CircularBuffer<number>(3)
numBuffer.push(1)
numBuffer.push(2)
numBuffer.push(3)
console.log(numBuffer.get(0))  // 1

const strBuffer = new CircularBuffer<string>(2)
strBuffer.push("a")
strBuffer.push("b")
console.log(strBuffer.get(0))  // "a"
```

### 案例 2：Tool 类型定义

**文件：`src/Tool.ts`**

```typescript
// Tool 是 Claude Code 的核心泛型类型
export type Tool<
    Input extends AnyObject = AnyObject,  // 默认是空对象
    Output = unknown,                       // 默认是 unknown
    P extends ToolProgressData = ToolProgressData,
> = {
    readonly name: string
    readonly description: string
    readonly input: () => Input
    readonly output: () => Output
    readonly tags?: readonly string[]
    call(args: Input): Promise<ToolResult<Output>>
}

// 使用示例
interface ReadFileInput extends AnyObject {
    path: string
}

interface ReadFileOutput {
    content: string
    lines: number
}

type ReadFileTool = Tool<ReadFileInput, ReadFileOutput>

// ReadFileTool 的实例需要实现：
// - name: string
// - description: string
// - input(): ReadFileInput
// - output(): ReadFileOutput
// - call(args: ReadFileInput): Promise<ToolResult<ReadFileOutput>>
```

### 案例 3：Result 类型

```typescript
// 模拟 Go 的 error 模式
type Result<T, E = Error> =
    | { ok: true; value: T }
    | { ok: false; error: E }

function divide(a: number, b: number): Result<number, string> {
    if (b === 0) {
        return { ok: false, error: "division by zero" }
    }
    return { ok: true, value: a / b }
}

const result = divide(10, 2)
if (result.ok) {
    console.log(result.value)  // 5
} else {
    console.error(result.error)
}
```

---

## 10. 实战练习

### 练习 1：实现泛型函数

```typescript
// 1. last - 返回数组最后一个元素
function last<T>(arr: T[]): T | undefined {
    return arr[arr.length - 1]
}

console.log(last([1, 2, 3]))    // 3
console.log(last(["a", "b"]))    // "b"
console.log(last([]))           // undefined

// 2. pipe - 函数组合
function pipe<T>(...fns: Array<(t: T) => T>): (t: T) => T {
    return (t: T) => fns.reduce((acc, fn) => fn(acc), t)
}

const process = pipe(
    (n: number) => n + 1,
    (n: number) => n * 2,
    (n: number) => n - 1
)

console.log(process(5))  // ((5 + 1) * 2) - 1 = 11
```

### 练习 2：实现泛型类

```typescript
// 创建一个 Stack<T> 类
class Stack<T> {
    private items: T[] = []

    push(item: T): void {
        this.items.push(item)
    }

    pop(): T | undefined {
        return this.items.pop()
    }

    peek(): T | undefined {
        return this.items[this.items.length - 1]
    }

    isEmpty(): boolean {
        return this.items.length === 0
    }

    size(): number {
        return this.items.length
    }
}

const stack = new Stack<number>()
stack.push(1)
stack.push(2)
stack.push(3)
console.log(stack.peek())  // 3
console.log(stack.pop())   // 3
console.log(stack.size())  // 2
```

### 练习 3：使用泛型约束

```typescript
// 实现一个 merge 函数，合并两个对象
function merge<T extends object, U extends object>(a: T, b: U): T & U {
    return { ...a, ...b }
}

const obj1 = { name: "Alice" }
const obj2 = { age: 25 }

const merged = merge(obj1, obj2)
console.log(merged)  // { name: "Alice", age: 25 }
```

### 练习 4：使用工具类型

```typescript
interface User {
    id: number
    name: string
    email: string
    password: string
    createdAt: Date
}

// 1. 创建 UpdateUser 类型（id 和 password 除外）
type UpdateUser = Partial<Omit<User, "id" | "password">>

// 2. 创建 PublicUser 类型（只读）
type PublicUser = Readonly<Pick<User, "id" | "name" | "email">>

// 使用
const update: UpdateUser = { name: "Bob" }
const publicUser: PublicUser = { id: 1, name: "Alice", email: "alice@example.com" }
// publicUser.name = "Bob"  // 错误！只读
```

---

## 本课总结

### 核心概念

| 概念 | 语法 | 说明 |
|------|------|------|
| 泛型函数 | `function fn<T>(x: T): T` | 类型参数化 |
| 泛型接口 | `interface Box<T> { value: T }` | 类型参数化 |
| 泛型类 | `class Box<T> { value: T }` | 类型参数化 |
| 泛型约束 | `T extends Constraint` | 限制类型范围 |
| keyof | `K extends keyof T` | 键约束 |
| Partial<T> | `Partial<User>` | 所有属性可选 |
| Pick<T, K> | `Pick<User, "id" | "name">` | 选取属性 |
| Omit<T, K> | `Omit<User, "password">` | 排除属性 |
| 条件类型 | `T extends U ? X : Y` | 条件映射 |

### 对比 Go

| Go | TypeScript |
|----|------------|
| `func Min[T any](a T, b T) T` | `function min<T>(a: T, b: T): T` |
| `T any`（无约束）| `T extends object` |
| 无内置工具类型 | `Partial<T>`, `Pick<T, K>` |
| 无条件类型 | `T extends U ? X : Y` |

### 泛型使用场景

| 场景 | 示例 |
|------|------|
| 数据结构 | `Stack<T>`, `Queue<T>`, `CircularBuffer<T>` |
| 函数 | `map<T, U>`, `filter<T>`, `reduce<T>` |
| API 响应 | `Result<T, E>`, `Response<T>` |
| 工具类型 | `Partial<T>`, `Readonly<T>` |
| 类组件 | `Component<Props>` |

---

## 下一步

学习 [第五课：类](./lesson_05_classes.md)

---

*本课完*
