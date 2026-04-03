# 第二课：接口与类型别名

> 本课讲解如何定义对象结构，是 TypeScript 最核心的概念之一

---

## 目录

1. [接口基础](#1-接口基础)
2. [可选属性](#2-可选属性)
3. [只读属性](#3-只读属性)
4. [方法定义](#4-方法定义)
5. [类型别名](#5-类型别名)
6. [interface vs type](#6-interface-vs-type)
7. [函数类型](#7-函数类型)
8. [索引签名](#8-索引签名)
9. [接口继承](#9-接口继承)
10. [实战练习](#10-实战练习)

---

## 1. 接口基础

### 概念解释

接口（Interface）用于定义对象的**结构（shape）**。类似于 Go 的 `struct`，但更灵活。

### Go 对比

```go
// Go 中用 struct 定义对象结构
type Person struct {
    Name string
    Age  int
}

var p Person = Person{Name: "Alice", Age: 25}
```

### TypeScript 接口

```typescript
// 定义接口
interface Person {
    name: string
    age: number
}

// 方式一：显性类型注解 (Explicit Type Annotation)
// 明确要求变量必须严格遵循接口结构
let p: Person = {
    name: "Alice",
    age: 25
}

// 方式二：隐性类型推导 (Implicit Type Inference)
// 不写类型注解，由 TypeScript 根据右侧的值自动推断出类型
let p2 = {
    name: "Bob",
    age: 30
}
// p2 此时被自动推断为 { name: string; age: number }，满足 Person 接口结构

```

### 基本语法

```typescript
// interface 接口名 { 属性: 类型 }
interface User {
    id: number
    name: string
    email: string
    isActive: boolean
}

// 接口可以作为变量类型
let user: User

// 也可以作为函数参数类型
function greet(user: User): string {
    return `Hello, ${user.name}!`
}

// 还可以作为函数返回类型
function createUser(name: string, age: number): User {
    return {
        name,
        age,
        id: Math.random(),
        email: `${name.toLowerCase()}@example.com`,
        isActive: true
    }
}
```

### 真实案例

**文件：`src/textInputTypes.ts`**
```typescript
// Claude Code 中的接口定义
interface TextInputProps {
    readonly text: string
    readonly fullCommand: string
    readonly onHistoryUp?: () => void
    readonly onChange: (value: string) => void
}

// 使用接口
function handleInput(props: TextInputProps): void {
    console.log(props.text)
    props.onChange("new value")
}
```

---

## 2. 可选属性

### 概念解释

可选属性表示这个属性**可以有也可以没有**，类似于 Go 中使用指针类型（`*Type`，nil 表示不存在）。

### Go 对比

```go
// Go 中用指针表示可选
type Person struct {
    Name    string
    Age     *int  // 可选，用 nil 表示不存在
}

// 使用
age := 25
p := Person{Name: "Alice", Age: &age}
```

### TypeScript 可选属性

```typescript
// 用 ? 标记可选属性
interface Person {
    name: string
    age?: number  // 可选，可以不提供
}

// 使用
let p1: Person = { name: "Alice" }        // OK，age 省略
let p2: Person = { name: "Bob", age: 30 } // OK，提供了 age

// 访问可选属性
console.log(p1.age)  // undefined（未定义）
if (p1.age !== undefined) {
    console.log(p1.age)  // 安全访问
}
```

### 实际应用

```typescript
interface User {
    id: number
    name: string
    email: string
    phone?: string      // 可选：可能没有电话
    address?: string     // 可选：可能没有地址
    birthday?: Date     // 可选：可能不知道生日
}

function printUser(user: User): void {
    console.log(`ID: ${user.id}`)
    console.log(`Name: ${user.name}`)

    // 安全访问可选属性
    if (user.phone) {
        console.log(`Phone: ${user.phone}`)
    }

    // 使用可选链（后面会详细讲）
    console.log(`City: ${user.address?.split(',')[0]}`)
}
```

### 真实案例

**文件：`src/textInputTypes.ts`**
```typescript
interface TextInputProps {
    readonly text: string
    readonly fullCommand: string
    readonly onHistoryUp?: () => void    // 可选回调
    readonly onChange: (value: string) => void
}

// 省略可选属性
const props1: TextInputProps = {
    text: "hello",
    fullCommand: "hello world",
    // onHistoryUp 省略，因为是可选的
    onChange: (value) => console.log(value)
}
```

---

## 3. 只读属性

### 概念解释

只读属性表示属性**创建后不能修改**。这是 TypeScript 提供的编译时保护，Go 没有这个概念（但可以通过方法限制）。

### Go 对比

```go
// Go 没有原生的只读属性
// 只能用首字母大写（导出）和小写（不导出）来控制访问
type Person struct {
    Id   int    // 导出，可以访问
    name string // 不导出，外部无法访问
}
```

### TypeScript readonly

```typescript
interface Person {
    readonly id: number  // 只读属性
    name: string
}

let p: Person = {
    id: 1,
    name: "Alice"
}

p.name = "Bob"  // OK
p.id = 2        // 错误！Cannot assign to 'id' because it is a read-only property
```

### 实际应用

```typescript
interface Config {
    readonly apiUrl: string      // API 地址，创建后不能改
    readonly maxRetries: number  // 最大重试次数
    timeout: number             // 超时时间，可以动态调整
}

const config: Config = {
    apiUrl: "https://api.example.com",
    maxRetries: 3,
    timeout: 5000
}

config.timeout = 10000  // OK
config.apiUrl = "https://new-api.example.com"  // 错误！
```

### 真实案例

**文件：`src/types/textInputTypes.ts`**
```typescript
interface TextInputProps {
    readonly text: string           // 只读文本
    readonly fullCommand: string   // 只读完整命令
    readonly onChange: (value: string) => void  // 回调不应变
}

// 这保护了 text 和 fullCommand 不被意外修改
```

### readonly vs const

```typescript
// const 用于变量，readonly 用于对象属性
const MAX = 100           // 变量不可重新赋值
MAX = 200                 // 错误！

interface Point {
    readonly x: number
    readonly y: number
}

const origin: Point = { x: 0, y: 0 }
origin.x = 5  // 错误！即使 const 声明，属性仍是只读的
```

---

## 4. 方法定义

### 概念解释

接口可以定义方法，类似于 Go 的 struct 方法，但写在接口内部。

### Go 对比

```go
// Go 中方法写在 struct 外面
type Person struct {
    Name string
}

func (p Person) Greet() string {
    return "Hello, " + p.Name
}

p := Person{Name: "Alice"}
fmt.Println(p.Greet())
```

### TypeScript 方法

```typescript
interface Person {
    name: string
    age: number

    // 方法定义
    greet(): string

    // 带参数的方法
    introduce(name: string): string

    // 带返回值的方法
    getAgeInMonths(): number
}

let p: Person = {
    name: "Alice",
    age: 25,
    greet() {
        return `Hello, my name is ${this.name}`
    },
    introduce(otherName: string) {
        return `Hi ${otherName}, I'm ${this.name}!`
    },
    getAgeInMonths() {
        return this.age * 12
    }
}

console.log(p.greet())              // "Hello, my name is Alice"
console.log(p.introduce("Bob"))      // "Hi Bob, I'm Alice!"
console.log(p.getAgeInMonths())      // 300
```

### 方法与函数属性的区别

```typescript
interface Calculator {
    // 方法：每次调用 this 指向实例
    add(a: number, b: number): number

    // 函数属性：箭头函数，this 绑定到定义时的外层上下文
    subtract: (a: number, b: number) => number
}

let calc: Calculator = {
    add(a, b) {
        return a + b
    },
    subtract: (a, b) => a - b
}

calc.add(5, 3)        // 8，this === calc
calc.subtract(5, 3)  // 2，this !== calc（箭头函数 this 词法绑定）
```

### this 绑定详解

**方法**调用时，`this` 自动绑定到实例：

```typescript
const obj = {
    value: 42,
    getValue() {
        return this.value  // this 指向 obj
    }
}

obj.getValue()  // 42，this === obj
```

**箭头函数属性**的 `this` 是词法绑定，指向定义时的外层上下文：

```typescript
class Counter {
    count = 0

    // 方法：this 指向实例
    increment() {
        this.count++
    }

    // 箭头函数属性：this 始终指向实例（即使作为回调传递）
    log = () => {
        console.log(this.count)  // this 指向 Counter 实例
    }
}

const counter = new Counter()
setTimeout(counter.log, 1000)  // 1 秒后仍能正确访问 this.count
```

如果用普通函数作为回调，`this` 会丢失：

```typescript
setTimeout(counter.increment, 1000)  // this === undefined（严格模式）
```

### 何时选择哪种？

| 场景 | 推荐 |
|------|------|
| 需要访问实例属性 | 方法 |
| 作为回调传递且需要保留 `this` | 箭头函数属性 |
| 不需要访问 `this` | 两者皆可 |

---

## 5. 类型别名

### 概念解释

类型别名（Type Alias）为类型起一个名字，类似于 Go 的 `type` 声明。

### Go 对比

```go
// Go 中
type UserId int
type UserName string
type User struct {
    Id   UserId
    Name UserName
}
```

### TypeScript 类型别名

```typescript
// 给基本类型起别名
type ID = number
type Name = string
type Age = number

// 给对象类型起别名
type User = {
    id: ID
    name: Name
    age: Age
}

// 使用别名
let user: User = {
    id: 1,
    name: "Alice",
    age: 25
}
```

### 类型别名 vs 接口

```typescript
// 类型别名可以给任何类型起名
type Status = "pending" | "active" | "completed"
type Callback = (error: Error | null, result?: string) => void

// 接口只能描述对象结构
interface Config {
    url: string
    timeout: number
}

// 两者在描述对象时几乎可以互换
type Person1 = {
    name: string
}

interface Person2 {
    name: string
}
```

### 真实案例

**文件：`src/types/permissions.ts`**
```typescript
// 类型别名定义权限模式
type PermissionBehavior = 'allow' | 'deny' | 'ask'
type PermissionMode = 'acceptEdits' | 'bypassPermissions' | 'default' | 'dontAsk' | 'plan' | 'auto' | 'bubble'

// 使用
function checkPermission(mode: PermissionMode): PermissionBehavior {
    if (mode === 'bypassPermissions') {
        return 'allow'
    }
    return 'ask'
}
```

---

## 6. interface vs type

### 何时用 interface

```typescript
// 1. 对象需要被 implements（类实现）
interface Serializable {
    serialize(): string
}

class User implements Serializable {
    serialize() {
        return JSON.stringify(this)
    }
}

// 2. 对象可能被扩展（继承）
interface Animal {
    name: string
}

interface Dog extends Animal {
    breed: string
}
```

### 何时用 type

```typescript
// 1. 联合类型
type Result = Success | Failure

// 2. 交叉类型
type Extended = Base & Extra

// 3. 函数类型
type Handler = (event: Event) => void

// 4. 元组
type Pair = [string, number]

// 5. 映射类型
type Readonly<T> = {
    readonly [P in keyof T]: T[P]
}
```

### 对比总结

| 特性 | interface | type |
|------|-----------|------|
| 定义对象结构 | ✅ | ✅ |
| 被 implements | ✅ | ❌ |
| 扩展（extends）| ✅ | ❌ |
| 联合类型 | ❌ | ✅ |
| 交叉类型 | ❌ | ✅ |
| 函数类型 | ❌ | ✅ |
| 元组 | ❌ | ✅ |

### Claude Code 的风格

**文件：`src/Tool.ts`**
```typescript
// 使用 interface 定义工具的结构
interface Tool {
    readonly name: string
    readonly description: string
    call(args: unknown): Promise<ToolResult>
}

// 使用 type 定义联合类型
export type ValidationResult =
    | { result: true }
    | { result: false; message: string; errorCode: number }
```

---

## 7. 函数类型

### 概念解释

TypeScript 可以把函数定义为属性类型，类似于 Go 的函数类型。

### Go 对比

```go
// Go 中函数是头等公民
type Handler func(int) string

func process(h Handler) {
    result := h(42)
    fmt.Println(result)
}
```

### TypeScript 函数类型

```typescript
// type 函数签名 = (参数类型) => 返回类型
type Handler = (id: number) => string
type Callback = (error: Error | null, result?: string) => void
type Transform<T> = (value: T) => T

// 使用函数类型
let myHandler: Handler
myHandler = (id: number) => `Processing ${id}`

// 作为函数参数
function execute(handler: Handler, value: number): void {
    const result = handler(value)
    console.log(result)
}

execute(myHandler, 42)  // "Processing 42"

// 作为返回类型
function createGreeter(name: string): () => string {
    return () => `Hello, ${name}!`
}

const greeter = createGreeter("Alice")
console.log(greeter())  // "Hello, Alice!"
```

### 真实案例

**文件：`src/textInputTypes.ts`**
```typescript
// 回调函数类型
interface TextInputProps {
    // 可选的键盘事件回调
    readonly onHistoryUp?: () => void

    // 必需的值变化回调
    readonly onChange: (value: string) => void

    // 键盘事件回调
    readonly onKeyDown?: (event: KeyboardEvent) => void
}

// 使用
function handleInput(props: TextInputProps) {
    // 调用可选回调（需要检查是否存在）
    if (props.onHistoryUp) {
        props.onHistoryUp()
    }

    // 调用必需回调
    props.onChange("new value")
}
```

---

## 8. 索引签名

### 概念解释

索引签名允许用任意字符串或数字访问属性，类似于 Go 的 `map[string]interface{}`。

### Go 对比

```go
// Go 中用 map 表示键值对
data := map[string]interface{}{
    "name": "Alice",
    "age": 25,
}
```

### TypeScript 索引签名

```typescript
// 字符串索引签名：可以用任意字符串作为键
interface StringMap {
    [key: string]: string
}

let map: StringMap = {}
map["key1"] = "value1"
map["key2"] = "value2"

// 数字索引签名：可以用数字作为键（数组的原理）
interface NumberArray {
    [index: number]: string
}

let arr: NumberArray = ["a", "b", "c"]
console.log(arr[0])   // "a"
console.log(arr[1])   // "b"
```

### 混合索引签名

```typescript
// 同时支持字符串和数字键
interface Mixed {
    [key: string]: any
    length: number
    name: string
}

let m: Mixed = {
    length: 10,
    name: "test",
    customField: "any value"
}
```

### 实际应用

```typescript
// 用户可以添加任意属性
interface UserWithExtras {
    name: string
    age: number
    // 允许添加任意额外属性
    [key: string]: string | number | boolean
}

const user: UserWithExtras = {
    name: "Alice",
    age: 25,
    email: "alice@example.com",     // 额外属性
    isActive: true                  // 额外属性
}
```

---

## 9. 接口继承

### 概念解释

接口可以继承其他接口，类似于 Go 的嵌入（但语法不同）。

### Go 对比

```go
// Go 用嵌入实现继承
type Animal struct {
    Name string
}

type Dog struct {
    Animal          // 嵌入
    Breed string
}

dog := Dog{}
dog.Name = "Buddy"   // 通过嵌入访问
```

### TypeScript 继承

```typescript
// 单继承
interface Animal {
    name: string
}

interface Dog extends Animal {
    breed: string
}

let dog: Dog = {
    name: "Buddy",   // 继承自 Animal
    breed: "Labrador"
}

// 多继承
interface Printable {
    print(): void
}

interface Loggable {
    log(): void
}

interface PrintableLog extends Printable, Loggable {
    format(): string
}

let obj: PrintableLog = {
    print() {
        console.log("printing")
    },
    log() {
        console.log("logging")
    },
    format() {
        return "formatted"
    }
}
```

### 继承中的可选属性

```typescript
interface Base {
    id: number
    name?: string  // 可选
}

interface Extended extends Base {
    age: number
    // 继承来的 name 仍然是可选的
}

let e: Extended = {
    id: 1,
    age: 25
    // name 可以省略
}
```

### 覆盖属性

```typescript
interface Base {
    id: number
    name: string
}

// 以下两种覆盖都会导致冲突，子类的属性的类型必须能够 assign 给父类的同名属性：
// interface Extended extends Base {
//     name: number  // 错误！number 不能赋值给 string
// }
// interface Extended extends Base {
//     name: boolean  // 错误！boolean 不能赋值给 string
// }
```

---

## 10. 实战练习

### 练习 1：定义用户接口

```typescript
// 定义一个 User 接口，包含：
// - id: number (只读)
// - username: string
// - email: string
// - isActive: boolean
// - createdAt?: Date (可选)

// 创建两个用户实例，一个完整，一个省略可选属性

interface User {
    readonly id: number
    username: string
    email: string
    isActive: boolean
    createdAt?: Date
}

const user1: User = {
    id: 1,
    username: "alice",
    email: "alice@example.com",
    isActive: true
}

const user2: User = {
    id: 2,
    username: "bob",
    email: "bob@example.com",
    isActive: false,
    createdAt: new Date("2024-01-01")
}

console.log(user1, user2)
```

### 练习 2：接口继承

```typescript
// 定义 Animal 接口
// 定义 Dog 接口，继承 Animal，并添加 breed 属性
// 创建 Dog 实例

interface Animal {
    name: string
    age: number
}

interface Dog extends Animal {
    breed: string
}

const dog: Dog = {
    name: "Buddy",
    age: 3,
    breed: "Labrador"
}

console.log(`${dog.name} is a ${dog.breed} and is ${dog.age} years old`)
```

### 练习 3：函数类型接口

```typescript
// 定义一个函数类型 MathOperation
// type MathOperation = (a: number, b: number) => number

// 创建加法、减法、乘法实现
// 创建 calculate 函数，接收操作和两个数字，返回结果

type MathOperation = (a: number, b: number) => number

const add: MathOperation = (a, b) => a + b
const subtract: MathOperation = (a, b) => a - b
const multiply: MathOperation = (a, b) => a * b

function calculate(op: MathOperation, a: number, b: number): number {
    return op(a, b)
}

console.log(calculate(add, 10, 5))      // 15
console.log(calculate(subtract, 10, 5)) // 5
console.log(calculate(multiply, 10, 5)) // 50
```

---

## 本课总结

### 核心概念

| 概念 | 语法 | 说明 |
|------|------|------|
| 接口定义 | `interface Name { ... }` | 定义对象结构 |
| 可选属性 | `prop?: type` | 可以省略 |
| 只读属性 | `readonly prop: type` | 不可修改 |
| 方法 | `method(): returnType` | 对象的方法 |
| 类型别名 | `type Alias = ...` | 给类型起名 |
| 函数类型 | `(args) => returnType` | 函数的类型 |
| 索引签名 | `[key: string]: type` | 任意键访问 |
| 接口继承 | `interface A extends B` | 扩展接口 |

### 对比 Go

| Go | TypeScript |
|----|------------|
| `type Person struct{}` | `interface Person { }` |
| `*Type`（可选）| `prop?: type` |
| 无只读 | `readonly prop: type` |
| 方法写在 struct 外 | 方法写在 interface 内 |
| 嵌入 | `extends` |
| `type Handler func()` | `type Handler = () => void` |

---

## 下一步

学习 [第三课：函数](./lesson_03_functions.md)

---

*本课完*
