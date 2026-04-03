# 第一课：TypeScript 类型基础

> 本课介绍 TypeScript 的基本类型系统

---

## 目录

1. [TypeScript 简介](#1-typescript-简介)
2. [类型注解 vs 类型推导](#2-类型注解-vs-类型推导)
3. [基本类型](#3-基本类型)
4. [数组](#4-数组)
5. [只读数组](#5-只读数组)
6. [元组](#6-元组)
7. [类型别名](#7-类型别名)
8. [枚举类型](#8-枚举类型)
9. [Any 和 Unknown](#9-any-和-unknown)
10. [Void 和 Never](#10-void-和-never)
11. [类型断言](#11-类型断言)
12. [实战练习](#12-实战练习)

---

## 1. TypeScript 简介

### 什么是 TypeScript？

TypeScript 是 JavaScript 的超集，由微软开发。它在 JavaScript 的基础上增加了**静态类型系统**。

```
JavaScript 代码  ──►  TypeScript 代码  ──►  JavaScript 编译输出
（.js 文件）      （.ts 文件）         （.js 文件）
```

### 为什么需要 TypeScript？

**Go 工程师应该能理解静态类型的好处：**

| 对比 | JavaScript（动态类型） | TypeScript（静态类型） |
|------|------------------------|------------------------|
| 类型检查 | 运行时才发现错误 | 编译时就发现错误 |
| IDE 支持 | 有限 | 强大的代码补全和重构 |
| 代码文档 | 依赖注释 | 类型本身就是文档 |
| 重构 | 容易引入 bug | 更安全的重构 |

### TypeScript 代码示例

```typescript
// 这是一个完整的 TypeScript 程序
function add(a: number, b: number): number {
    return a + b
}

const result = add(1, 2)  // 编译正确
const wrong = add("1", 2) // 编译错误！类型不对
```

---

## 2. 类型注解 vs 类型推导

### Go 的写法

```go
// Go 中类型写在变量名后面
var name string = "Alice"   // 显式类型
age := 30                   // 类型推导（:= 会自动推导类型）
```

### TypeScript 的写法

TypeScript 同样支持**显式注解**和**类型推导**：

```typescript
// 显式类型注解
let name: string = "Alice"

// 类型推导（TS 自动推断类型）
let age = 30          // TS 会推断 age 是 number 类型

// 推导后就不能赋值其他类型
age = "thirty"         // 错误！age 已经被推导为 number
```

### 基本语法对比

| Go | TypeScript | 说明 |
|----|------------|------|
| `var x int = 10` | `let x: number = 10` | 显式整型 |
| `x := 10` | `let x = 10` | 类型推导 |
| `const PI = 3.14` | `const PI = 3.14` | 常量（相似）|
| `var s string` | `let s: string` | 声明但未初始化 |

### 详细解释

```typescript
// ========== 显式类型注解 ==========
// 语法：let 变量名: 类型 = 值
let age: number = 25
let name: string = "Alice"
let isActive: boolean = true

// ========== 类型推导 ==========
// 如果初始化时没有指定类型，TypeScript 会自动推断
let count = 100        // 推断为 number
let message = "hello"  // 推断为 string
let flag = true        // 推断为 boolean

// ========== const 常量 ==========
// const 推导为更精确的字面量类型
const MAX_SIZE = 100   // 推导为 100（字面量类型），不是 number
const NAME = "Alice"   // 推导为 "Alice"（字面量类型），不是 string

// ========== 类型推导的局限性 ==========
let data = getValue()  // 如果 getValue() 返回 any，data 也会是 any
```

---

## 3. 基本类型

TypeScript 的基本类型与 Go 有些相似，但名称略有不同：

### 类型对照表

| Go 类型 | TypeScript 类型 | 说明 |
|---------|----------------|------|
| `string` | `string` | 字符串（相同）|
| `int`, `float64` | `number` | 数字（TS 不区分整数和浮点）|
| `bool` | `boolean` | 布尔（TS 用 boolean 不是 bool）|
| `nil` | `null` | 空值之一 |
| 无单独类型 | `undefined` | 未定义 |

### 代码示例

```typescript
// ========== 字符串 ==========
let firstName: string = "Alice"
let lastName: string = 'Bob'           // 单引号也可以
let fullName: string = `${firstName} ${lastName}`  // 模板字符串

// ========== 数字 ==========
let age: number = 25
let price: number = 99.99
let negative: number = -10
let hex: number = 0xff               // 十六进制
let binary: number = 0b1010          // 二进制
let octal: number = 0o12             // 八进制

// ========== 布尔 ==========
let isStudent: boolean = true
let hasPermission: boolean = false

// ========== 空值 ==========
let empty: null = null               // 明确表示"没有值"
let notYetDefined: undefined = undefined  // "未定义"

let name: string = null   // 错误！严格模式下（strictNullChecks 开启），null 不能赋值给 string
let nameOrNull: string | null = null  // 正确：可以用联合类型明确表示可能是字符串或 null
```

### 重要区别：null 和 undefined

在 TypeScript 中，`null` 和 `undefined` 是两个不同的概念：

```typescript
let a: number = undefined   // 错误！严格模式下（strictNullChecks 开启）不能赋值
let b: null = undefined     // 错误！严格模式下 null 和 undefined 不可互转

// 💡 提示：在 tsconfig.json 中开启 `strict: true` 会默认开启 `strictNullChecks: true`。这就迫使开发者必须去处理那些本该是空值的边缘情况，从而杜绝绝大多数的 "Cannot read property of undefined" 崩溃风险。这也是 TS 比起 JS 对 Go 开发者最有吸引力的地方。
let c: number | undefined = undefined  // 明确表示可能是数字或 undefined
let d: number | null = null            // 明确表示可能是数字或 null

```

---

## 4. 数组

### Go 的数组/切片

```go
// Go 中数组和切片
var nums []int = []int{1, 2, 3}
names := []string{"a", "b"}
```

### TypeScript 的数组

TypeScript 有两种定义数组的方式：

```typescript
// 方式一：类型[]
let numbers: number[] = [1, 2, 3, 4, 5]
let names: string[] = ["Alice", "Bob", "Charlie"]

// 方式二：Array<类型>
let scores: Array<number> = [98, 85, 92]
let fruits: Array<string> = ["apple", "banana", "orange"]

// 两种方式完全等价
```

### 数组的基本操作

```typescript
let arr: number[] = [1, 2, 3]

// 添加元素
arr.push(4)           // [1, 2, 3, 4]

// 删除最后一个元素
arr.pop()             // 返回 4, arr 现在是 [1, 2, 3]

// 遍历
arr.forEach(num => {
    console.log(num)
})

// map 转换
let doubled: number[] = arr.map(num => num * 2)

// filter 过滤
let evens: number[] = arr.filter(num => num % 2 === 0)

// reduce 聚合
let sum: number = arr.reduce((acc, num) => acc + num, 0)
```

### 真实案例：来自 Claude Code

**文件：`src/commands.ts`**
```typescript
// 定义一个字符串数组类型
const BUILTIN_COMMANDS: string[] = ["commit", "review", "compact", "mcp", "config", "doctor"]

// 使用数组
for (const name of BUILTIN_COMMANDS) {
    console.log(name)
}
```

---

## 5. 只读数组

### 概念解释

Go 中没有真正的只读数组，但可以通过切片创建不可变视图。TypeScript 有原生的 `readonly` 语法：

```typescript
// 普通数组 - 可以修改
let mutable: number[] = [1, 2, 3]
mutable.push(4)        // OK
mutable[0] = 10        // OK

// 只读数组 - 不能修改
let readonly: readonly number[] = [1, 2, 3]
// readonly.push(4)    // 错误！Property 'push' does not exist on type 'readonly number[]'
// readonly[0] = 10    // 错误！Index signature in type 'readonly number[]' only permits reading
```

### 语法对比

```typescript
// 普通数组
type MutableArray = number[]

// 只读数组（TS 特有）
type ReadonlyArray = readonly number[]
```

### 实际应用

```typescript
// 函数参数使用只读数组，表示"我只读取，不修改"
function printNames(names: readonly string[]): void {
    console.log(names.join(", "))
    // names.push("new")  // 错误！
}

// 调用
const allNames = ["Alice", "Bob", "Charlie"]
printNames(allNames)           // OK
printNames(["Solo", "Kiwi"])   // 也可以传字面量数组
```

### 真实案例

**文件：`src/textInputTypes.ts`**
```typescript
// 定义一个不可变的文本块数组
interface TextBlock {
    readonly type: string
    readonly text: string
}

// 函数接收只读数组
function processBlocks(blocks: readonly TextBlock[]): void {
    for (const block of blocks) {
        console.log(block.text)
    }
}
```

---

## 6. 元组

### 概念解释

元组（Tuple）是 TypeScript 特有的类型，表示**固定长度、已知类型的数组**。

### Go 的对比

Go 没有元组，但可以用结构体模拟：
```go
// Go 中模拟元组
type Pair struct {
    Name string
    Age  int
}
```

### TypeScript 元组

```typescript
// 定义一个元组：第一个是 string，第二个是 number
let person: [string, number] = ["Alice", 25]

// 访问元素
console.log(person[0])  // "Alice"
console.log(person[1])   // 25

// 赋值
person[0] = "Bob"       // OK
person[1] = 30          // OK

// 类型不匹配
person[0] = 123         // 错误！string 位置不能赋值为 number
// person[2] = true    // 错误！索引 2 超出元组定义的范围
// 注意：元组在静态类型层会对越界索引（如 tuple[5]）报错，但它运行时仍是普通 JS 数组，
// push()、pop() 等方法可以动态改变长度，TypeScript 编译无法检测。
// 要完全锁定长度（禁止任何修改），应使用 `readonly` 元组：
// const readonlyTuple: readonly [string, number] = ["hello", 42]
// readonlyTuple.push(100)  // Error: Property 'push' does not exist on readonly '[string, number]'
```

### 可选元组元素

```typescript
// 可选元素用 ? 标记（表示该位置可以有也可以没有）
let optionalTuple: [string, number?, boolean?]

optionalTuple = ["hello"]           // OK，只有第一个
optionalTuple = ["hello", 42]      // OK，有前两个
optionalTuple = ["hello", 42, true] // OK，三个都有
```

### 实际应用

```typescript
// 函数返回多个值（类似 Go 的多返回值）
function getUserInfo(): [string, number, string] {
    return ["Alice", 25, "Beijing"]
}

const [name, age, city] = getUserInfo()
console.log(name, age, city)

// 使用场景：表示键值对
let entry: [string, any] = ["key", { value: 123 }]
```

### 真实案例

**文件：`src/types/textInputTypes.ts`**
```typescript
// 键值对元组
type KeyHandler = [string, () => void]

// 命令历史记录，每条是 [命令字符串, 时间戳]
type HistoryEntry = [string, number]
```

---

## 7. 类型别名

### 概念解释

类型别名（Type Alias）就是给一个类型起个名字，类似于 Go 的 `type` 定义：

```go
// Go 中
type UserId int
type UserName string
```

### TypeScript 类型别名

```typescript
// 基本语法：type 别名 = 类型
type UserId = number
type UserName = string
type Age = number

// 给复杂类型起别名
type Point = {
    x: number
    y: number
}

type Callback = (data: string) => void
```

### 使用别名

```typescript
type ID = number
type Name = string
type User = {
    id: ID
    name: Name
    email: string
}

let user: User = {
    id: 1,
    name: "Alice",
    email: "alice@example.com"
}

// 别名也可以组合
type Users = User[]           // 用户数组
type UserOrNull = User | null  // 用户或空
```

### 真实案例

**文件：`src/types/permissions.ts`**
```typescript
// 给字符串字面量组合起别名
type PermissionBehavior = 'allow' | 'deny' | 'ask'
type PermissionMode = 'acceptEdits' | 'bypassPermissions' | 'default' | 'dontAsk' | 'plan'

// 使用别名
function checkPermission(mode: PermissionMode): PermissionBehavior {
    if (mode === 'bypassPermissions') {
        return 'allow'
    }
    return 'ask'
}
```

---

## 8. 枚举类型

### Go 的常量定义

```go
// Go 中用常量模拟枚举
const (
    Red   = "red"
    Blue  = "blue"
    Green = "green"
)
```

### TypeScript 枚举

```typescript
// 定义枚举
enum Color {
    Red,    // 0
    Blue,   // 1
    Green   // 2
}

// 使用枚举值
let c: Color = Color.Red
console.log(c)          // 0
console.log(Color[0])    // "Red"（反向映射）

// 枚举值从 1 开始
enum Status {
    Pending = 1,
    Active,
    Completed
}
console.log(Status.Active)  // 2
```

### 字符串枚举

```typescript
// 字符串枚举 - 更易读
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT"
}

let move: Direction = Direction.Up
console.log(move)  // "UP"
// ⚠️ 重要：字符串枚举没有反向映射！
// console.log(Direction["UP"])  // 这在字符串枚举上不存在，反向映射只是数字枚举的特性
```

### 异构枚举（混合）

```typescript
// 可以混合数字和字符串，但不推荐
enum BooleanLike {
    No = 0,
    Yes = "YES"
}
```

### 枚举的使用场景

```typescript
enum LogLevel {
    Debug = "DEBUG",
    Info = "INFO",
    Warn = "WARN",
    Error = "ERROR"
}

function log(level: LogLevel, message: string): void {
    if (level === LogLevel.Error) {
        console.error(`[${level}] ${message}`)
    } else {
        console.log(`[${level}] ${message}`)
    }
}

log(LogLevel.Info, "Application started")
log(LogLevel.Error, "Connection failed")
```

### 真实案例

**文件：`src/types/permissions.ts`**
```typescript
// 虽然 TS 有 enum，但 Claude Code 更多使用字面量联合类型
type PermissionBehavior = 'allow' | 'deny' | 'ask'

// 这比 enum 更简洁，也是现代 TS 的推荐做法
```

---

## 9. Any 和 Unknown

### Any（任意类型）

`any` 是 TypeScript 的"escape hatch"（逃逸舱），表示**任意类型**，关闭类型检查：

```typescript
let value: any = 123
value = "hello"        // OK
value = true           // OK
value = { name: "Alice" }  // OK

// any 类型可以像任意类型一样使用
value.foo()            // OK（编译不报错，但运行可能出错）
value.toUpperCase()    // OK
```

**⚠️ 警告：尽量少用 any，它会让你失去 TypeScript 的类型保护！**

```typescript
// 例子：滥用 any
function processData(data: any): any {
    return data.value  // 如果 data 没有 value，运行时才会报错
}

// 正确做法：使用具体类型或 unknown
function processDataSafe(data: unknown): string {
    if (typeof data === 'object' && data !== null && 'value' in data) {
        return (data as { value: string }).value
    }
    throw new Error("Invalid data format")
}
```

### Unknown（未知类型）

`unknown` 是 TypeScript 3.0 引入的类型，被称为"类型安全的 any"：

```typescript
let value: unknown = 123

// unknown 类型的值不能直接使用，必须先类型检查
if (typeof value === 'string') {
    console.log(value.toUpperCase())  // 只有在确认为 string 后才能用
}

// 可以赋值给其他类型（需要类型守卫）
let str: string = value  // 错误！unknown 不能直接赋值给其他具体类型
```

### 对比：any vs unknown

| 特性 | any | unknown |
|------|-----|---------|
| 可以赋值给任意类型 | ✅ | ❌（只能赋给 `any` 和 `unknown`）|
| 任意值可以赋值给它 | ✅ | ✅ |
| 可以调用任意方法 | ✅ | ❌ |
| 推荐使用 | ❌ | ✅ |

```typescript
// any - 不安全
let unsafe: any = getData()
unsafe.foo()  // 编译通过，运行时可能出错

// unknown - 安全
let safe: unknown = getData()
// safe.foo()  // 编译错误！必须先类型检查
if (typeof safe === 'object' && safe !== null) {
    // 安全地使用 safe
}
```

---

## 10. Void 和 Never

### Void（空）

`void` 表示**没有返回值**，类似于 Go 的 `func() {}`：

```typescript
// 没有返回值的函数
function logMessage(message: string): void {
    console.log(message)
    // 没有 return 语句，本质上返回 undefined
}

// void 函数可以 return undefined（或不写 return）——但不能 return null
function noReturn(): void {
    return undefined  // 合法
    // return null    // 错误！strict 模式下 null 不能赋给 void
}
```

### Never（永不返回）

`never` 表示**永远不会正常返回**，用于：

1. 函数总是抛出异常
2. 无限循环
3. 类型 narrowing（类型收窄）

```typescript
// 总是抛出异常的函数
function throwError(message: string): never {
    throw new Error(message)
}

// 无限循环（永不返回）
function infiniteLoop(): never {
    while (true) {
        // 处理事件
    }
}
```

### 实际应用

```typescript
// 高级用法：类型收窄
function processValue(value: string | number) {
    if (typeof value === 'string') {
        console.log(value.toUpperCase())  // value 在这里是 string
    } else {
        console.log(value.toFixed(2))      // value 在这里是 number
    }
}

// never 在 switch 中的应用
type Shape = { kind: 'circle'; radius: number } | { kind: 'square'; side: number }

function area(shape: Shape): number {
    switch (shape.kind) {
        case 'circle':
            return Math.PI * shape.radius ** 2
        case 'square':
            return shape.side ** 2
        default:
            // 如果 Shape 增加新类型但忘记处理，编译器会报错
            const _exhaustive: never = shape
            throw new Error(`Unknown shape: ${_exhaustive}`)
    }
}
```

---

## 11. 类型断言

### 概念解释

类型断言（Type Assertion）类似于 Go 的类型转换，但它是**编译时**的操作：

```typescript
// 语法：值 as 类型
let value: unknown = "hello"
let str: string = value as string

// 旧语法（不推荐）
let str2: string = <string>value
```

### 使用场景

```typescript
// 1. 从 unknown 中提取具体类型
function process(data: unknown): string {
    if (typeof data === 'string') {
        // 编译器已经知道是 string，不需要断言
        return data.toUpperCase()
    }
    if ((data as { name: string }).name) {
        return (data as { name: string }).name
    }
    return "unknown"
}

// 2. DOM 元素的类型断言
const input = document.getElementById("username") as HTMLInputElement
input.value = "alice"  // 不断言的话，TS 认为是 HTMLElement，没有 value 属性

// 3. JSON 解析结果
const json = '{"name": "Alice", "age": 25}'
const user = JSON.parse(json) as { name: string; age: number }
```

### 注意事项

```typescript
// 类型断言是"欺骗"编译器，要确保你知道自己在做什么
let value: any = "hello"
let num: number = value as number  // 编译通过，但 value 实际是字符串
console.log(num * 2)  // 运行时输出 NaN！

// 更安全的做法
if (typeof value === 'number') {
    let num: number = value
    console.log(num * 2)  // 安全
}
```

---

## 12. 实战练习

### 练习 1：声明各种类型的变量

```typescript
// 创建以下变量：
// 1. name: 字符串类型，值为 "Alice"
// 2. age: 数字类型，值为 25
// 3. isActive: 布尔类型，值为 true
// 4. scores: 数字数组，值为 [95, 87, 92]
// 5. user: 对象类型，有 name(string) 和 age(number) 属性

// 你的代码：
let name: string = "Alice"
let age: number = 25
let isActive: boolean = true
let scores: number[] = [95, 87, 92]
let user: { name: string; age: number } = { name: "Bob", age: 30 }

console.log(name, age, isActive, scores, user)
```

### 练习 2：实现数组操作

```typescript
// 给定数字数组，计算：
// 1. 总和
// 2. 平均值
// 3. 最大值
// 4. 只保留大于 60 的数

const numbers: number[] = [85, 42, 90, 55, 78, 62, 91]

// 你的代码：
const sum = numbers.reduce((acc, n) => acc + n, 0)
const avg = sum / numbers.length
const max = Math.max(...numbers)
const passed = numbers.filter(n => n > 60)

console.log(`总和: ${sum}, 平均: ${avg}, 最大: ${max}, 及格: ${passed}`)
```

### 练习 3：使用元组

```typescript
// 创建一个函数，返回用户名和年龄
// 使用元组类型作为返回类型

function getUser(): [string, number] {
    return ["Alice", 25]
}

// 解构使用
const [userName, userAge] = getUser()
console.log(`${userName} is ${userAge} years old`)
```

---

## 本课总结

### 核心概念

| 概念 | 说明 |
|------|------|
| 类型注解 | `let x: type = value` |
| 类型推导 | `let x = value`（自动推断）|
| 数组 | `type[]` 或 `Array<type>` |
| 只读数组 | `readonly type[]` |
| 元组 | `[type1, type2, ...]` |
| 类型别名 | `type Alias = existingType` |
| 枚举 | `enum Name { A, B, C }` |
| Any | 任意类型（慎用）|
| Unknown | 未知类型（安全）|
| Void | 无返回值 |
| Never | 永不返回 |
| 类型断言 | `value as Type` |

### 对比 Go

| Go | TypeScript |
|----|------------|
| `var x int` | `let x: number` |
| `x := 10` | `let x = 10` |
| `[]int{1,2,3}` | `[1, 2, 3]` 或 `Array<number>` |
| `type Person struct{}` | `type Person = { name: string }` 或 `interface Person {}` |
| `const (A=1; B=2)` | `enum { A = 1, B = 2 }` |

---

## 下一步

学习 [第二课：接口与类型别名](./lesson_02_interfaces.md)

---

*本课完*
