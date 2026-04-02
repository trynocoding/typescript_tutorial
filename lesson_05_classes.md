# 第五课：类

> 本课讲解 TypeScript 类的面向对象特性

---

## 目录

1. [类基础](#1-类基础)
2. [访问修饰符](#2-访问修饰符)
3. [参数属性](#3-参数属性)
4. [只读属性](#4-只读属性)
5. [接口实现](#5-接口实现)
6. [抽象类](#6-抽象类)
7. [继承](#7-继承)
8. [方法重写](#8-方法重写)
9. [静态成员](#9-静态成员)
10. [类作为类型](#10-类作为类型)
11. [实战练习](#11-实战练习)

---

## 1. 类基础

### Go 对比

Go 使用 struct + method 的组合，没有传统的类：

```go
type Person struct {
    Name string
    Age  int
}

func (p *Person) Birthday() {
    p.Age++
}

func (p Person) Greet() string {
    return "Hello, I'm " + p.Name
}

person := Person{Name: "Alice", Age: 25}
person.Birthday()
fmt.Println(person.Greet())
```

### TypeScript 类

TypeScript 有传统的 class 语法：

```typescript
class Person {
    name: string
    age: number

    constructor(name: string, age: number) {
        this.name = name
        this.age = age
    }

    // 方法
    greet(): string {
        return `Hello, I'm ${this.name}`
    }

    // 生日
    birthday(): void {
        this.age++
    }
}

// 创建实例
const person = new Person("Alice", 25)
console.log(person.greet())  // "Hello, I'm Alice"
person.birthday()
console.log(person.age)      // 26
```

### 类的组成

```typescript
class ClassName {
    // 属性（字段）
    property: type

    // 构造器
    constructor(params) {
        // 初始化
    }

    // 方法
    method(): returnType {
        // 实现
    }

    // getter/setter（可选）
    get property(): type { return this._property }
    set property(value: type) { this._property = value }
}
```

### 类与接口的关系

```typescript
// 接口定义结构
interface Greetable {
    greet(): string
}

// 类实现接口
class Person implements Greetable {
    constructor(public name: string) {}

    greet(): string {
        return `Hello, I'm ${this.name}`
    }
}
```

---

## 2. 访问修饰符

### Go 对比

Go 用首字母大小写控制访问：
- 大写 = 导出（公开）
- 小写 = 不导出（私有）

```go
type Person struct {
    Name   string  // 公开
    age    int     // 私有（不导出）
}
```

### TypeScript 访问修饰符

TypeScript 有三个访问修饰符：

| 修饰符 | 访问级别 | 说明 |
|--------|----------|------|
| `public` | 任意位置 | 公开访问（默认）|
| `private` | 仅类内 | 私有，外部不能访问 |
| `protected` | 类及子类 | 受保护，子类可访问 |

```typescript
class Person {
    public name: string      // 公开（默认）
    private age: number      // 私有
    protected id: number      // 受保护

    constructor(name: string, age: number, id: number) {
        this.name = name
        this.age = age
        this.id = id
    }

    getAge(): number {
        return this.age  // 类内可以访问 private
    }
}

const person = new Person("Alice", 25, 1)

console.log(person.name)     // OK：公开属性
// console.log(person.age)    // 错误！private 不能外部访问
// console.log(person.id)     // 错误！protected 不能外部访问
console.log(person.getAge()) // OK：通过方法访问
```

### 真实案例

**文件：`src/services/mcp/auth.ts`**
```typescript
class McpSession {
    // 私有属性
    private _metadata?: Awaited<ReturnType<...>>
    private readonly _accessToken?: string

    // 公开方法
    async getAccessToken(): Promise<string> {
        return await this.getAccessTokenImpl()
    }
}
```

---

## 3. 参数属性

### 概念解释

TypeScript 可以在构造器参数前加修饰符，自动创建并初始化属性。

### Go 对比

Go 没有这个语法，需要手动赋值：

```go
type Person struct {
    Name string
    Age  int
}

func NewPerson(name string, age int) *Person {
    return &Person{Name: name, Age: age}
}
```

### TypeScript 参数属性

```typescript
// 传统写法
class Person {
    name: string
    age: number

    constructor(name: string, age: number) {
        this.name = name
        this.age = age
    }
}

// 参数属性写法（更简洁）
class Person {
    constructor(
        public name: string,
        public age: number
    ) {}
}

// 等价于上面的写法！
```

### 带修饰符的参数属性

```typescript
class User {
    // public 参数
    constructor(
        public name: string,
        public email: string,
        // private 参数
        private password: string,
        // readonly 参数
        public readonly id: number
    ) {}
}

const user = new User("Alice", "alice@example.com", "secret123", 1)

console.log(user.name)    // "Alice"
console.log(user.email)   // "alice@example.com"
// console.log(user.password)  // 错误！private
console.log(user.id)     // 1
// user.id = 2  // 错误！readonly
```

### 真实案例

**文件：`src/utils/CircularBuffer.ts`**
```typescript
export class CircularBuffer<T> {
    // 使用参数属性
    constructor(private capacity: number) {
        // this.capacity 已经在参数中声明
        this.buffer = new Array(capacity)
    }

    private buffer: T[]
    private head = 0
    private size = 0

    // ...
}
```

---

## 4. 只读属性

### 基本用法

```typescript
class Config {
    // 只读属性必须在声明时或构造器中初始化
    constructor(
        public readonly apiUrl: string,
        public readonly maxRetries: number
    ) {
        // 只读属性只能在构造器中赋值
    }

    // 普通只读属性
    public readonly appName: string = "MyApp"

    // 方法不能修改只读属性
    changeUrl(newUrl: string): void {
        this.apiUrl = newUrl  // 错误！apiUrl 是 readonly
    }
}

const config = new Config("https://api.example.com", 3)
// config.apiUrl = "https://new-api.example.com"  // 错误！readonly
```

### readonly vs const

```typescript
// const 用于变量
// readonly 用于类的属性
class Example {
    const CONST_VALUE = 100  // 错误！类属性不能用 const

    readonly READONLY_VALUE = 100  // OK
}
```

---

## 5. 接口实现

### 概念解释

类可以实现一个或多个接口，用 `implements` 关键字。

### Go 对比

Go 使用隐式实现：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type MyReader struct {}

func (r MyReader) Read(p []byte) (n int, err error) {
    return 0, nil
}

// MyReader 自动实现 Reader 接口
```

### TypeScript implements

```typescript
// 定义接口
interface Printable {
    print(): void
}

interface Serializable {
    serialize(): string
}

// 实现单个接口
class Document implements Printable {
    constructor(public content: string) {}

    print(): void {
        console.log(this.content)
    }
}

// 实现多个接口
class Report implements Printable, Serializable {
    constructor(public title: string, public content: string) {}

    print(): void {
        console.log(`Title: ${this.title}\n${this.content}`)
    }

    serialize(): string {
        return JSON.stringify({ title: this.title, content: this.content })
    }
}
```

### 真实案例

**文件：`src/utils/ShellCommand.ts`**
```typescript
// 定义接口
interface ShellCommand {
    readonly command: string
    readonly args?: readonly string[]
    readonly cwd?: string
    readonly env?: Record<string, string>

    execute(): Promise<ShellCommandResult>
    kill(): void
}

// 实现接口
class ShellCommandImpl implements ShellCommand {
    constructor(
        public readonly command: string,
        public readonly args: readonly string[] = [],
        public readonly cwd?: string,
        public readonly env?: Record<string, string>
    ) {}

    async execute(): Promise<ShellCommandResult> {
        // 实现
        return { stdout: "", stderr: "", exitCode: 0 }
    }

    kill(): void {
        // 实现
    }
}

// 另一个实现
class AbortedShellCommand implements ShellCommand {
    readonly command = "aborted"
    readonly args = []
    readonly cwd = undefined
    readonly env = undefined

    async execute(): Promise<ShellCommandResult> {
        return {
            stdout: "",
            stderr: "Command was aborted",
            exitCode: 1
        }
    }

    kill(): void {}
}
```

---

## 6. 抽象类

### 概念解释

抽象类不能直接实例化，只能被继承。用 `abstract` 关键字定义。

### Go 对比

Go 没有抽象类的概念，用接口和组合实现类似功能。

### TypeScript 抽象类

```typescript
// 抽象类
abstract class Animal {
    constructor(public name: string) {}

    // 抽象方法 - 子类必须实现
    abstract speak(): string

    // 具体方法 - 子类可以直接使用
    move(): void {
        console.log(`${this.name} is moving`)
    }
}

// 抽象类不能直接实例化
// const animal = new Animal()  // 错误！

// 必须继承并实现抽象方法
class Dog extends Animal {
    speak(): string {
        return "Woof!"
    }
}

class Cat extends Animal {
    speak(): string {
        return "Meow!"
    }
}

const dog = new Dog("Buddy")
console.log(dog.speak())  // "Woof!"
dog.move()                // "Buddy is moving"
```

### 抽象类 vs 接口

| 特性 | 抽象类 | 接口 |
|------|--------|------|
| 实现方法 | ✅ 可以 | ❌ 不能 |
| 属性 | ✅ 可以 | ❌ 只能声明 |
| 多继承 | ❌ 不能 | ✅ 可以实现多个 |
| 实例化 | ❌ 不能 | ❌ 不能 |

```typescript
// 抽象类可以有实现的方法和属性
abstract class Base {
    abstract doSomething(): void

    // 具体方法
    log(): void {
        console.log("logging...")
    }
}

// 接口只能声明
interface IBase {
    doSomething(): void
    // 不能有实现
}
```

---

## 7. 继承

### Go 对比

Go 使用嵌入（Embedding）实现继承：

```go
type Animal struct {
    Name string
}

func (a *Animal) Speak() string {
    return "..."
}

type Dog struct {
    *Animal  // 嵌入
    Breed string
}

dog := &Dog{Animal: &Animal{Name: "Buddy"}, Breed: "Labrador"}
```

### TypeScript 继承

```typescript
// 基类
class Animal {
    constructor(public name: string) {}

    speak(): string {
        return "Some sound"
    }
}

// 继承
class Dog extends Animal {
    constructor(name: string, public breed: string) {
        super(name)  // 调用父类构造器
    }

    // 重写父类方法
    override speak(): string {
        return "Woof!"
    }
}

const dog = new Dog("Buddy", "Labrador")
console.log(dog.name)   // "Buddy"（从父类继承）
console.log(dog.breed)   // "Labrador"
console.log(dog.speak()) // "Woof!"（重写）
```

### super 调用

```typescript
class Cat extends Animal {
    constructor(name: string, public color: string) {
        super(name)  // 必须先调用 super
    }

    override speak(): string {
        const baseSound = super.speak()  // 调用父类方法
        return `${baseSound} Meow!`
    }
}
```

---

## 8. 方法重写

### 基本用法

```typescript
class Parent {
    greet(): string {
        return "Hello"
    }
}

class Child extends Parent {
    override greet(): string {  // override 关键字（TS 4.3+）
        return "Hi there!"
    }
}
```

### 保护成员

```typescript
class Base {
    private secret(): void {
        console.log("secret")
    }

    public expose(): void {
        this.secret()  // 可以访问 private 方法
    }
}

class Derived extends Base {
    public trySecret(): void {
        // this.secret()  // 错误！private 不能继承
    }
}
```

### 多层继承

```typescript
class A {
    value: number = 1
}

class B extends A {
    value: number = 2
}

class C extends B {
    value: number = 3
}

const c = new C()
console.log(c.value)  // 3（最子类覆盖）

// 访问父类
class D extends A {
    override value: number = 10

    showBase(): void {
        // 在 TypeScript 和 JavaScript 中，实例属性绑定在 this 上，而不是 prototype 上。
        // 所以使用 super 来读取父类属性是错误的（将返回 undefined）。只能用 super 调用方法。
        console.log(this.value)  // 10 （最子类覆盖）
    }
}
```

---

## 9. 静态成员

### 概念解释

静态成员属于类本身，不属于实例。用 `static` 关键字定义。

### Go 对比

Go 用包级变量和函数模拟：

```go
type Counter struct {
    count int
}

// 包级变量（类似静态）
var totalCounters = 0

func NewCounter() *Counter {
    totalCounters++
    return &Counter{count: 0}
}
```

### TypeScript 静态成员

```typescript
class Counter {
    static totalCount = 0  // 静态属性
    public count = 0      // 实例属性

    constructor() {
        Counter.totalCount++  // 访问静态成员
    }

    static getTotal(): number {  // 静态方法
        return Counter.totalCount
    }
}

const c1 = new Counter()
const c2 = new Counter()

console.log(c1.count)         // 0
console.log(c2.count)         // 0
console.log(Counter.totalCount)  // 2
console.log(Counter.getTotal())  // 2
```

### 实际应用

```typescript
class MathUtils {
    // 静态常量大写
    static readonly PI = 3.14159

    // 静态方法
    static add(a: number, b: number): number {
        return a + b
    }

    static average(arr: number[]): number {
        return arr.reduce((a, b) => a + b, 0) / arr.length
    }
}

console.log(MathUtils.PI)        // 3.14159
console.log(MathUtils.add(1, 2)) // 3
console.log(MathUtils.average([1, 2, 3])) // 2
```

---

## 10. 类作为类型

### 类本身是类型

```typescript
class Point {
    constructor(public x: number, public y: number) {}
}

class Person {
    constructor(public name: string) {}
}

// Point 既可以当类型用
const p: Point = new Point(1, 2)

// 也可以声明变量是某个类
let entity: typeof Point = Point
```

### 类的实例类型

```typescript
class User {
    name: string
    age: number
}

// 实例的类型是 User
const user: User = new User()

// 函数参数使用类类型
function createUser(factory: new () => User): User {
    return new factory()
}
```

---

## 11. 实战练习

### 练习 1：实现栈类

```typescript
class Stack<T> {
    private items: T[] = []
    private _size = 0

    constructor(private capacity: number = Infinity) {}

    push(item: T): void {
        if (this._size >= this.capacity) {
            throw new Error("Stack overflow")
        }
        this.items.push(item)
        this._size++
    }

    pop(): T {
        if (this.isEmpty()) {
            throw new Error("Stack underflow")
        }
        this._size--
        return this.items.pop()!
    }

    peek(): T {
        if (this.isEmpty()) {
            throw new Error("Stack is empty")
        }
        return this.items[this._size - 1]
    }

    isEmpty(): boolean {
        return this._size === 0
    }

    size(): number {
        return this._size
    }
}

// 测试
const stack = new Stack<number>(3)
stack.push(1)
stack.push(2)
stack.push(3)
console.log(stack.peek())  // 3
console.log(stack.pop())   // 3
console.log(stack.size())  // 2
```

### 练习 2：实现接口

```typescript
interface Shape {
    area(): number
    perimeter(): number
}

class Rectangle implements Shape {
    constructor(
        private width: number,
        private height: number
    ) {}

    area(): number {
        return this.width * this.height
    }

    perimeter(): number {
        return 2 * (this.width + this.height)
    }
}

class Circle implements Shape {
    constructor(private radius: number) {}

    area(): number {
        return Math.PI * this.radius ** 2
    }

    perimeter(): number {
        return 2 * Math.PI * this.radius
    }
}

const rect = new Rectangle(5, 3)
const circle = new Circle(2)

console.log(`Rectangle: area=${rect.area()}, perimeter=${rect.perimeter()}`)
console.log(`Circle: area=${circle.area().toFixed(2)}, perimeter=${circle.perimeter().toFixed(2)}`)
```

### 练习 3：抽象类

```typescript
abstract class Database {
    constructor(public connectionString: string) {}

    // 抽象方法
    abstract connect(): Promise<void>
    abstract query(sql: string): Promise<any[]>
    abstract disconnect(): Promise<void>

    // 具体方法
    async execute(sql: string): Promise<void> {
        await this.connect()
        try {
            await this.query(sql)
        } finally {
            await this.disconnect()
        }
    }
}

class MySQLDatabase extends Database {
    async connect(): Promise<void> {
        console.log(`Connecting to MySQL: ${this.connectionString}`)
    }

    async query(sql: string): Promise<any[]> {
        console.log(`Executing: ${sql}`)
        return []
    }

    async disconnect(): Promise<void> {
        console.log("Disconnecting from MySQL")
    }
}

const db = new MySQLDatabase("mysql://localhost:3306/mydb")
db.execute("SELECT * FROM users")
```

---

## 本课总结

### 核心概念

| 概念 | 语法 | 说明 |
|------|------|------|
| 类定义 | `class Name { }` | 包含属性、方法、构造器 |
| 访问修饰符 | `public/private/protected` | 控制访问级别 |
| 参数属性 | `constructor(public x: T)` | 自动创建属性 |
| 只读属性 | `readonly x: T` | 初始化后不可改 |
| implements | `class C implements I` | 实现接口 |
| extends | `class C extends P` | 继承父类 |
| abstract | `abstract class C` | 不能实例化 |
| override | `override method()` | 重写父类方法 |
| static | `static x = 1` | 属于类本身 |

### 对比 Go

| Go | TypeScript |
|----|------------|
| struct + method | class |
| 首字母大写/小写 | public/private/protected |
| 隐式接口实现 | implements 显式实现 |
| 嵌入实现继承 | extends |
| 无 | abstract class |
| 无 | override |

---

## 下一步

学习 [第六课：联合类型与区分联合](./lesson_06_union_types.md)

---

*本课完*
