# 第三课：函数

> 本课讲解 TypeScript 函数的声明和使用

---

## 目录

1. [函数声明](#1-函数声明)
2. [函数类型](#2-函数类型)
3. [箭头函数](#3-箭头函数)
4. [参数类型](#4-参数类型)
5. [返回值类型](#5-返回值类型)
6. [可选参数](#6-可选参数)
7. [默认参数](#7-默认参数)
8. [剩余参数](#8-剩余参数)
9. [函数重载](#9-函数重载)
10. [this 类型](#10-this-类型)
11. [实战练习](#11-实战练习)

---

## 1. 函数声明

### Go 对比

```go
// Go 函数
func Add(a int, b int) int {
    return a + b
}

func main() {
    result := Add(1, 2)
    fmt.Println(result)
}
```

### TypeScript 函数声明

```typescript
// 基本语法：function 函数名(参数: 参数类型): 返回类型 { }
function add(a: number, b: number): number {
    return a + b
}

// 调用
const result = add(1, 2)
console.log(result)  // 3

// 无返回值函数
function greet(name: string): void {
    console.log(`Hello, ${name}!`)
}

greet("Alice")  // "Hello, Alice!"
```

### 函数表达式

```typescript
// 把函数赋值给变量
const add = function(a: number, b: number): number {
    return a + b
}

// 调用
console.log(add(1, 2))  // 3
```

---

## 2. 函数类型

### 概念解释

函数类型描述函数的签名（参数和返回值类型）。

### Go 对比

```go
// Go 函数作为参数
type Transform func(int) int

func double(n int) int {
    return n * 2
}

func transform(n int, fn Transform) int {
    return fn(n)
}

result := transform(5, double)  // 10
```

### TypeScript 函数类型

```typescript
// 语法：type 函数类型 = (参数类型) => 返回类型
type Transform = (n: number) => number
type BinaryOp = (a: number, b: number) => number
// 可选参数 result?: string 表示参数可以省略；
// result: string | undefined 表示必须传 undefined 值（两者不同）
type Callback = (err: Error | null, result?: string) => void

// 使用函数类型
function apply(n: number, fn: Transform): number {
    return fn(n)
}

const double = (n: number): number => n * 2
const square = (n: number): number => n * n

console.log(apply(5, double))   // 10
console.log(apply(5, square))   // 25
```

### 真实案例

**文件：`src/textInputTypes.ts`**
```typescript
// 回调函数类型
interface TextInputProps {
    // 带参数的回调
    readonly onKeyDown?: (event: KeyboardEvent) => void

    // 无参数回调
    readonly onHistoryUp?: () => void

    // 返回 void 的回调
    readonly onChange: (value: string) => void
}
```

---

## 3. 箭头函数

### 概念解释

箭头函数（Arrow Function）是 TypeScript/JavaScript 的简洁函数语法，Go 没有这个概念。

### 基本语法

```typescript
// 普通函数
function add(a: number, b: number): number {
    return a + b
}

// 箭头函数 - 单表达式
const addArrow = (a: number, b: number): number => a + b

// 箭头函数 - 多行函数体
const addMultiLine = (a: number, b: number): number => {
    const sum = a + b
    return sum
}
```

### 与 Go 的对比

```go
// Go 没有箭头函数，只能用普通函数
add := func(a, b int) int { return a + b }
```

### 箭头函数的各种形式

```typescript
// 管理简单：简单记录一下箭头函数的括号规则：
// - 单参数 + 无类型注解：可以省略括号  n => n * 2
// - 单参数 + 有类型注解：括号必须加  (n: number) => n * 2
// - 无参数：必须加  () => ...
// - 多参数：必须加  (a, b) => ...

// 1. 单参数，有类型注解（必须加括号）
const double2 = (n: number): number => n * 2

// 2. 单参数，无类型注解（可以省略括号，不推荐）
// const double3 = n => n * 2  // 开启 noImplicitAny 时报错

// 3. 上下文可以推导时，自然地省略括号和类型
const doubled = [1, 2, 3].map(n => n * 2)

// 4. 有函数体必须加括号
const process = (n: number): number => {
    const doubled = n * 2
    return doubled + 1
}

// 5. 无参数
const getRandom = (): number => Math.random()

// 6. 多参数
const add = (a: number, b: number): number => a + b

// 7. 多行箭头函数
const greet = (name: string): void => {
    const message = `Hello, ${name}!`
    console.log(message)
}
```

### 实际应用

```typescript
// 数组的 map
const numbers = [1, 2, 3, 4, 5]
const doubled = numbers.map((n: number): number => n * 2)
console.log(doubled)  // [2, 4, 6, 8, 10]

// 数组的 filter
const evens = numbers.filter((n: number): boolean => n % 2 === 0)
console.log(evens)  // [2, 4]

// 数组的 reduce
const sum = numbers.reduce((acc: number, n: number): number => acc + n, 0)
console.log(sum)  // 15

// 链式调用
const result = numbers
    .filter(n => n % 2 === 0)    // [2, 4]
    .map(n => n * 10)             // [20, 40]
    .reduce((a, b) => a + b, 0)  // 60

console.log(result)
```

---

## 4. 参数类型

### 显式参数类型

```typescript
// TypeScript 要求明确指定参数类型
function greet(name: string): void {
    console.log(`Hello, ${name}!`)
}

greet("Alice")     // OK
greet(123)        // 错误！Argument of type 'number' is not assignable to parameter of type 'string'
```

### 多参数函数

```typescript
function calculate(
    a: number,
    b: number,
    operation: "add" | "sub" | "mul" | "div" // 💡 推荐：使用字面量联合类型，而不是宽泛的 string
): number {
    switch (operation) {
        case "add": return a + b
        case "sub": return a - b
        case "mul": return a * b
        case "div": return a / b
        default:
            // 穷尽性检查：如果日后新增操作类型却忘记处理，这里编译报错
            const _exhaustive: never = operation
            throw new Error(`Unknown operation: ${_exhaustive}`)
    }
}

console.log(calculate(10, 5, "add"))  // 15
console.log(calculate(10, 5, "mul"))  // 50
```

### 对象参数

```typescript
// 使用解构赋值
function createUser({ name, age, email }: {
    name: string
    age: number
    email: string
}): { name: string; age: number; email: string; id: number } {
    return {
        name,
        age,
        email,
        id: Math.random()
    }
}

const user = createUser({
    name: "Alice",
    age: 25,
    email: "alice@example.com"
})
```

---

## 5. 返回值类型

### 显式返回类型

```typescript
// 虽然 TypeScript 可以推断，但显式声明是好的实践
function add(a: number, b: number): number {
    return a + b
}

// 无返回值
function log(message: string): void {
    console.log(message)
}

// 返回 never（永不返回）
function fail(message: string): never {
    throw new Error(message)
}
```

### 为什么需要返回类型注解？

```typescript
// 1. 帮助编译器
function ambiguous(a: number) {
    if (a > 0) return "positive"
    return false  // 类型推断会发现这里有问题
}

// 2. 文档作用
function parseDate(str: string): Date | null {
    const date = new Date(str)
    return isNaN(date.getTime()) ? null : date
}

// 3. 防止错误
// 在严格模式（开启 noUncheckedIndexedAccess）下，TypeScript 会强制认为数组访问可能越界：
function getFirstElementSafe(arr: number[]): number | undefined {
    return arr[0]
}
```

---

## 6. 可选参数

### 概念解释

可选参数表示参数可以省略，用 `?` 标记。类似于 Go 中使用指针或切片模拟。

### Go 对比

```go
// Go 不支持可选参数，用可变参数模拟
func greet(name string, args ...string) {
    fmt.Print("Hello, " + name)
    for _, arg := range args {
        fmt.Print(" " + arg)
    }
    fmt.Println()
}

greet("Alice")
greet("Bob", "how", "are", "you")
```

### TypeScript 可选参数

```typescript
// 用 ? 标记可选参数，可选参数必须在必需参数之后
function greet(name: string, greeting?: string): string {
    if (greeting) {
        return `${greeting}, ${name}!`
    }
    return `Hello, ${name}!`
}

console.log(greet("Alice"))                  // "Hello, Alice!"
console.log(greet("Bob", "Hi"))               // "Hi, Bob!"

// 可选参数类型是 T | undefined
// 模拟用户数据
const users = [
    { id: 1, name: "Alice", deleted: false },
    { id: 2, name: "Bob", deleted: true }
]

function getUser(id: number) {
    return users.find(u => u.id === id)
}

function findUser(id: number, includeDeleted?: boolean): string | undefined {
    const user = getUser(id)
    if (!user) return undefined
    if (!includeDeleted && user.deleted) return undefined
    return user.name
}

const name1 = findUser(1)              // string | undefined
const name2 = findUser(2)              // undefined（已删除）
const name3 = findUser(2, true)        // string | undefined（包含已删除用户）
```

### 实际应用

```typescript
// 可选参数链式调用
function createURL(
    base: string,
    path?: string,
    query?: string,
    fragment?: string
): string {
    let url = base
    if (path) url += `/${path}`
    if (query) url += `?${query}`
    if (fragment) url += `#${fragment}`
    return url
}

console.log(createURL("https://api.example.com"))
// "https://api.example.com"

console.log(createURL("https://api.example.com", "users"))
// "https://api.example.com/users"

console.log(createURL("https://api.example.com", "users", "age=25"))
// "https://api.example.com/users?age=25"
```

---

## 7. 默认参数

### 概念解释

默认参数在调用时不传值时使用默认值。

### Go 对比

```go
// Go 不支持默认参数，用函数内部判断模拟
func greet(name string) string {
    if name == "" {
        name = "World"
    }
    return "Hello, " + name
}
```

### TypeScript 默认参数

```typescript
// 语法：param: type = defaultValue
function greet(name: string = "World"): string {
    return `Hello, ${name}!`
}

console.log(greet())            // "Hello, World!"
console.log(greet("Alice"))     // "Hello, Alice!"

// 带类型的默认参数
function createUser(
    name: string,
    age: number = 18,
    email: string = "unknown@example.com"
): { name: string; age: number; email: string } {
    return { name, age, email }
}

console.log(createUser("Alice"))
// { name: "Alice", age: 18, email: "unknown@example.com" }

console.log(createUser("Bob", 30))
// { name: "Bob", age: 30, email: "unknown@example.com" }
```

### 默认参数与可选参数的区别

| 特性 | 默认参数 | 可选参数 |
|------|----------|-----------|
| 语法 | `param: type = value` | `param?: type` |
| 隐式值 | `undefined` | `undefined` |
| 位置限制 | 建议在必需参数**后** | 必须在必需参数后 |
| 传 `undefined` | 使用默认值 | 跳过 |

> ⚠️ **反模式警告**：虽然 TypeScript 允许默认参数放在必需参数之前，
> 例如 `function f(a: number = 0, b: number)`，但当想要触发默认值时必须显式传 `undefined`：`f(undefined, 5)`。
> 这是非常反直觉的设计，应视为**反模式**。实际开发中应始终将默认参数放在必需参数之后。

```typescript
// 默认参数可以在任意位置
function createUser(
    name: string = "Anonymous",
    age: number = 0
): void { }

// 可选参数必须在必需参数后
// function invalid(name?: string, age: number) {} // 错误！

// 传 undefined 使用默认值
function test(a: number = 10): number {
    return a
}

console.log(test())          // 10（使用默认值）
console.log(test(undefined))  // 10（使用默认值）
console.log(test(5))         // 5（覆盖默认值）
```

---

## 8. 剩余参数

### 概念解释

剩余参数允许函数接受任意数量的参数，类似于 Go 的可变参数。

### Go 对比

```go
// Go 可变参数
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

fmt.Println(sum(1, 2, 3, 4, 5))  // 15
```

### TypeScript 剩余参数

```typescript
// 用 ...rest 收集剩余参数
function sum(...numbers: number[]): number {
    return numbers.reduce((acc, n) => acc + n, 0)
}

console.log(sum(1, 2, 3, 4, 5))  // 15
console.log(sum())               // 0

// 配合其他参数使用
function collectArgs(a: number, ...rest: string[]): void {
    console.log(`First: ${a}`)
    console.log(`Rest: ${rest.join(", ")}`)
}

collectArgs(1, "a", "b", "c")
// First: 1
// Rest: a, b, c
```

### 实际应用

```typescript
// 格式化字符串
function format(template: string, ...values: unknown[]): string {
    return template.replace(/\{(\d+)\}/g, (_, index) => {
        const value = values[parseInt(index)]
        return value != null ? String(value) : ""
    })
}

console.log(format("Hello, {0}! You have {1} messages.", "Alice", 5))
// "Hello, Alice! You have 5 messages."

// 合并对象
function merge<T extends object>(base: T, ...others: Partial<T>[]): T {
    const result = { ...base }
    for (const other of others) {
        Object.assign(result, other)
    }
    return result
}

const defaults = { name: "Anonymous", age: 0, email: "" }
const user1 = merge(defaults, { name: "Alice" })
const user2 = merge(defaults, { name: "Bob", age: 30 })

console.log(user1)  // { name: "Alice", age: 0, email: "" }
console.log(user2)  // { name: "Bob", age: 30, email: "" }
```

---

## 9. 函数重载

### 概念解释

函数重载（Function Overload）允许一个函数对**不同的参数类型组合**提供不同的类型签名，让调用方获得精确的类型推断。

> **与 Go 的区别**：Go 本身不支持函数重载，且 Go 的"多返回值"（`return value, err`）是完全不同的概念——多返回值是让一个函数返回多个值；而函数重载是让同一个函数对不同的*输入类型*有不同的*返回类型*。两者解决的问题不同，不要混淆。

### Go 对比

Go 本身不支持函数重载，但可以用接口和可变参数模拟。

### TypeScript 函数重载

```typescript
// 重载签名
function parse(input: string): string[]      // 字符串转字符串数组
function parse(input: number): number[]      // 数字转数字数组
function parse(input: string | number): string[] | number[] {
    // 实现只有一个
    if (typeof input === "string") {
        return input.split("")
    }
    return [input]
}

console.log(parse("hello"))  // ["h", "e", "l", "l", "o"]
console.log(parse(123))     // [123]

function findOne(
    items: string[],
    query: string
): string | undefined

function findOne(
    items: number[],
    query: number
): number | undefined

function findOne(
    items: (string | number)[],
    query: string | number
): string | number | undefined {
    return items.find(item => item === query)
}

const names = ["Alice", "Bob", "Charlie"]
const ages = [25, 30, 35]

console.log(findOne(names, "Bob"))    // "Bob"
console.log(findOne(ages, 30))        // 30
console.log(findOne(names, "David"))  // undefined

// 💡 提示：在现代 TypeScript 中，这种重载场景通常更推荐使用泛型（下节课介绍）：
// function findOne<T>(items: T[], query: T): T | undefined
```

### 真实案例：接口方法重载

**文件：`src/Tool.ts`**

上面是**函数重载**（一个函数多个签名），下面是**可选方法**（接口中用 `?` 标记的可选成员）：

```typescript
// 工具接口有两种调用方式
interface Tool {
    // 调用工具，参数根据工具定义
    call(args: unknown): Promise<ToolResult>

    // callSync 是可选的同步调用方法（用 ? 标记，非方法重载）
    callSync?(args: unknown): ToolResult
}

// 使用
async function invokeTool(tool: Tool, args: unknown): Promise<ToolResult> {
    const result = await tool.call(args)
    return result
}
```

---

## 10. this 类型

### 概念解释

箭头函数没有自己的 `this`，它继承外层的 `this`。

### Go 对比

Go 没有 `this` 关键字，方法接收者就是第一个参数。

### TypeScript 中的 this

#### 显式指定 this 参数

TypeScript 允许在函数声明时，将 `this` 作为第一个"伪"参数提供类型。这与 Go 语言的方法接收者（Receiver）非常相似！

```go
// Go 方法接收者
func (u *User) Greet() string {
    return "Hello, " + u.Name
}
```

```typescript
// TypeScript 中显式声明 this 类型（编译后会自动消除）
function greetUser(this: { name: string }): string {
    return `Hello, ${this.name}!`
}

// greetUser() // 错误：必须要提供正确的 this 上下文
console.log(greetUser.call({ name: "Alice" })) // "Hello, Alice!"

const user = { name: "Bob", greet: greetUser }
console.log(user.greet()) // "Hello, Bob!"
```

#### 类中的 this

```typescript
// 推荐：使用 class 定义带 this 的函数
class Person {
    name: string

    constructor(name: string) {
        this.name = name
    }

    greet() {
        return `Hello, ${this.name}!`
    }
}

const p = new Person("Alice")
console.log(p.greet())  // "Hello, Alice!"

// 箭头函数的 this：捕获定义时外层的 this
const person = {
    name: "Bob",
    // ⚠️ 错误示范：箭头函数没有自己的 this，它会继承外层作用域（通常是 window 或 undefined）
    greet: () => {
        // 这里的 this 不是 person 对象！
        // return `Hello, ${this.name}!` // TypeScript 也会报错或提示
        return `Hello, unknown!`
    }
}
console.log(person.greet())  // "Hello, unknown!"

// 正确做法：用普通方法而非箭头函数
const person2 = {
    name: "Bob",
    greet() {
        return `Hello, ${this.name}!`
    }
}
console.log(person2.greet())  // "Hello, Bob!"
```

### 类中的 this

```typescript
class Counter {
    count: number = 0

    increment() {
        this.count++
        return this
    }

    decrement() {
        this.count--
        return this
    }

    getCount() {
        return this.count
    }
}

const counter = new Counter()
counter.increment().increment().decrement()
console.log(counter.getCount())  // 1
```

---

## 11. 实战练习

### 练习 1：基础函数

```typescript
// 创建以下函数：

// 1. add - 两数相加
const add = (a: number, b: number): number => a + b

// 2. isEven - 判断偶数
const isEven = (n: number): boolean => n % 2 === 0

// 3. greet - 打招呼，默认 "Hello"
const greet = (name: string, greeting: string = "Hello"): string =>
    `${greeting}, ${name}!`

console.log(add(5, 3))              // 8
console.log(isEven(4))               // true
console.log(greet("Alice"))          // "Hello, Alice!"
console.log(greet("Bob", "Hi"))      // "Hi, Bob!"
```

### 练习 2：数组操作函数

```typescript
// 创建以下高阶函数：

// 1. filterEven - 过滤偶数
const filterEven = (nums: number[]): number[] =>
    nums.filter(n => n % 2 === 0)

// 2. mapToString - 转为字符串数组
const mapToString = (nums: number[]): string[] =>
    nums.map(n => n.toString())

// 3. compose - 函数组合
const compose = <T>(f: (n: T) => T, g: (n: T) => T): ((n: T) => T) =>
    (n: T) => f(g(n))

const double = (n: number) => n * 2
const addOne = (n: number) => n + 1

const doubleThenAddOne = compose(addOne, double)
console.log(doubleThenAddOne(5))  // 11 (5*2+1)
```

### 练习 3：可选参数和默认参数

```typescript
// 创建 createUser 函数：
// - name: 必需
// - email: 可选，默认 "unknown@example.com"
// - age: 可选，默认 0
// - isActive: 可选，默认 true

interface User {
    name: string
    email: string
    age: number
    isActive: boolean
}

function createUser(
    name: string,
    email: string = "unknown@example.com",
    age: number = 0,
    isActive: boolean = true
): User {
    return { name, email, age, isActive }
}

console.log(createUser("Alice"))
// { name: "Alice", email: "unknown@example.com", age: 0, isActive: true }

console.log(createUser("Bob", "bob@example.com"))
// { name: "Bob", email: "bob@example.com", age: 0, isActive: true }
```

### 练习 4：剩余参数

```typescript
// 创建 sumAll 函数，接受任意数量的数组，返回所有元素的总和

function sumAll(...arrays: number[][]): number {
    return arrays.flat().reduce((acc, n) => acc + n, 0)
    // TypeScript 从初始值 0 推断 acc 和 n 为 number，无需显式注解
}

console.log(sumAll([1, 2], [3, 4], [5]))  // 15
console.log(sumAll([10, 20]))             // 30
console.log(sumAll())                      // 0
```

---

## 本课总结

### 核心概念

| 概念 | 语法 | 说明 |
|------|------|------|
| 函数声明 | `function fn() {}` | 普通函数 |
| 函数表达式 | `const fn = function() {}` | 赋值给变量 |
| 箭头函数 | `const fn = () => {}` | 简洁语法 |
| 函数类型 | `(a: A) => B` | 描述函数签名 |
| 可选参数 | `fn(a?: T)` | 可以省略 |
| 默认参数 | `fn(a: T = default)` | 有默认值 |
| 剩余参数 | `fn(...args: T[])` | 可变参数 |
| 函数重载 | `fn(): A; fn(): B` | 多签名 |
| 返回 never | `fn(): never` | 永不返回 |

### 对比 Go

| Go | TypeScript |
|----|------------|
| `func Add(a, b int) int` | `function add(a: number, b: number): number` |
| `func() int`（无参数）| `() => number` |
| 不支持默认参数 | `fn(a = 10)` |
| 可变参数 `...int` | `...numbers: number[]` |
| 无箭头函数 | `const fn = () => {}` |

---

## 下一步

学习 [第四课：泛型](./lesson_04_generics.md)

---

*本课完*
