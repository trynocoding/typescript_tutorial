# 第八课：特殊操作符

> 本课讲解 TypeScript 特有的操作符

---

## 目录

1. [可选链（?.）](#1-可选链)
2. [空值合并（??）](#2-空值合并)
3. [非空断言（!）](#3-非空断言)
4. [类型断言（as）](#4-类型断言)
5. [typeof 操作符](#5-typeof-操作符)
6. [keyof 操作符](#6-keyof-操作符)
7. [in 操作符](#7-in-操作符)
8. [实战练习](#8-实战练习)

---

## 1. 可选链（?.）

### 概念解释

可选链（Optional Chaining）安全地访问嵌套属性，Go 无此语法。

### Go 对比

Go 需要手动检查：

```go
// Go 手动检查嵌套属性
if user != nil && user.Profile != nil && user.Profile.Avatar != nil {
    url := user.Profile.Avatar.URL
}
```

### TypeScript 可选链

```typescript
// 基本语法
const value = obj?.prop?.nested?.value

// 🚨 在强类型的 TS 中，即使有了 ?.，你也必须首先在接口中声明可能存在的结构，
// 否则 TS 编译器仍然会报 Property 'phone' does not exist 的深红报错！
interface UserProfile {
    profile?: {
        avatar?: {
            url: string
        }
        phone?: {
            number: string
        }
    }
}

// 假设从后端拿到了这样一份数据（可能 phone 缺失）
const user: UserProfile = {
    profile: {
        avatar: {
            url: "https://example.com/avatar.png"
        }
    }
}

// 安全访问
const url = user?.profile?.avatar?.url
console.log(url)  // "https://example.com/avatar.png"

// 访问不存在的路径（因为接口里写了 phone?，所以 TS 允许编译，并在运行时返回 undefined）
const missing = user?.profile?.phone?.number
console.log(missing)  // undefined（不会崩溃）
```

### 调用可选方法

```typescript
// 可选方法调用
obj?.method?.()

// 实际例子
// ⚠️ 错误：`?` 不能用于 let/const/var 变量声明
// const callback?: () => void  // 这是语法错误！

// 正确写法：
// 可选属性（`?`）只能用于对象属性或函数参数
let callback: (() => void) | undefined
callback?.()  // 安全调用，如果 callback 是 undefined，什么都不做
```

### 真实案例

**文件：`src/query.ts`**
```typescript
// Claude Code 中大量使用可选链
const lastAssistantMessage = assistantMessages.at(-1)
const textBlocks = lastAssistantMessage?.message.content.filter(...)

// 安全访问
const deletedTokens = Math.max(
    0,
    cumulativeDeleted - pendingCacheEdits.baselineCacheDeletedTokens ?? 0
)
```

### 与 && 对比

```typescript
// && 方式
const url1 = user && user.profile && user.profile.avatar && user.profile.avatar.url

// ?. 方式（更简洁）
const url2 = user?.profile?.avatar?.url

// 两者关键区别：
// && 会对所有 falsy 值（0, "", false, null, undefined）短路
// ?. 只对 null/undefined 短路

// 示例：使用可能为 null/undefined 的对象属性
interface Profile { avatar?: { url: string } }
interface UserProfile { profile?: Profile }

const loggedInUser: UserProfile | null = null
const guestUser: UserProfile = { profile: {} }  // 无头像

// && 方式：遇到 null 就短路
const avatarUrl1 = loggedInUser && loggedInUser.profile && loggedInUser.profile.avatar?.url
// ?. 方式：等价但更简洁
const avatarUrl2 = loggedInUser?.profile?.avatar?.url
// 两者结果相同： undefined

// 是否会短路的差异：
const score = 0
// && 方式：0 是 falsy，会短路，返回 0 而不是访问后面
const v1 = score && String(score)  // 0（数字）
// ?. 方式：0 不是 null/undefined，不短路，会继续访问属性
// （number 没有 foo 属性，下例仅为说明流程）
const arr = [1, 2, 3]
const v2 = score || arr  // arr（|| 对 falsy 短路）
const v3 = score ?? arr  // 0（?? 只对 null/undefined 短路，0 不短路）
```

---

## 2. 空值合并（??）

### 概念解释

空值合并（Nullish Coalescing）在值为 null/undefined 时提供默认值。

### Go 对比

Go 使用 `||`，但行为不同：

```go
// Go || 会检查所有 falsy 值
value := 0
result := value || 100  // 100（0 是 falsy）

// TypeScript ?? 只检查 null/undefined
let value = 0
let result = value ?? 100  // 0（?? 只处理 null/undefined）
```

### TypeScript ??

```typescript
// 基本用法
const a = null ?? "default"     // "default"
const b = undefined ?? "default" // "default"
const c = 0 ?? "default"        // 0
const d = "" ?? "default"       // ""
const e = false ?? "default"    // false

// 典型用法：设置默认值
function greet(name?: string): string {
    return "Hello, " + (name ?? "World") + "!"
}

console.log(greet())           // "Hello, World!"
console.log(greet("Alice"))    // "Hello, Alice!"
```

### 真实案例

**文件：`src/query.ts`**
```typescript
// 计算缓存删除的 token 数
const deletedTokens = Math.max(
    0,
    cumulativeDeleted - pendingCacheEdits.baselineCacheDeletedTokens ?? 0
)

// 如果 baselineCacheDeletedTokens 是 undefined，使用 0
```

### ?? vs ||

| 值 | `??` | `\|\|` |
|----|------|--------|
| `null` | 默认值 | 默认值 |
| `undefined` | 默认值 | 默认值 |
| `0` | 原始值 | 默认值 |
| `""` | 原始值 | 默认值 |
| `false` | 原始值 | 默认值 |
| `"hello"` | 原始值 | 原始值 |

```typescript
// 典型场景：设置配置默认值
const config = {
    port: 0,           // 用户没有配置
    host: "",          // 用户没有配置
    debug: false       // 用户没有配置
}

// 使用 ||（错误）
const port1 = config.port || 8080      // 8080（错误！0 是有效端口）
const host1 = config.host || "localhost" // "localhost"（可以接受）
const debug1 = config.debug || true    // true（错误！false 是有效值）

// 使用 ??（正确）
const port2 = config.port ?? 8080       // 0（正确）
const host2 = config.host ?? "localhost" // ""（空字符串）
const debug2 = config.debug ?? true      // false（正确）
```

### 链式使用

```typescript
// 可以链式使用
const value = a ?? b ?? c ?? "default"

// 典型场景
const userName = user?.name ?? user?.nickname ?? "Anonymous"
```

---

## 3. 非空断言（!）

### 概念解释

非空断言（Non-null Assertion）告诉 TypeScript 值不会是 null/undefined。

### Go 对比

Go 没有这个概念，但可以用类型断言模拟。

### TypeScript !

```typescript
// 语法：在值后面加 !
const value: string | null = getValue()

// 告诉 TypeScript 我确定它不是 null
console.log(value!.length)  // 安全（开发者保证）

// 但如果 value 实际是 null，运行时会报错！
```

### 使用场景

```typescript
// 1. DOM 元素（TypeScript 不知道元素存在）
const input = document.getElementById("username")!
// input.value = "Alice"  // 现在可以访问

// 2. 数组索引
const items: string[] = []
const first = items[0]!  // TypeScript 认为可能是 undefined

// 3. 类型收窄后的确定情况
function process(value: string | null): number {
    if (value !== null) {
        // 这里 TypeScript 知道 value 是 string
        // 但如果你确定某些情况不会发生：
        return value.length
    }
    return 0
}
```

### 危险提醒

```typescript
// ! 是危险的，可能导致运行时错误
let value: string | null = null
// ⚠️ 下行编译通过，但运行时抛出：
// TypeError: Cannot read properties of null (reading 'length')
console.log(value!.length)

// 优先使用安全的方式
if (value !== null && value !== undefined) {
    console.log(value.length)  // 安全
}

// 或者使用 ?
console.log(value?.length)  // 安全，返回 undefined
```

---

## 4. 类型断言

### 概念解释

类型断言（Type Assertion）告诉 TypeScript 值的具体类型，类似于 Go 的类型转换。

### Go 对比

```go
// Go 类型转换
var i interface{} = "hello"
// str := i.(string)  // 可能有 panic
str, ok := i.(string)  // 安全方式
```

### TypeScript as

```typescript
// 语法：value as Type
const value: unknown = "hello"
const str = value as string
console.log(str.toUpperCase())  // "HELLO"

// 或者用 <> 语法（不推荐，在 JSX 中会有歧义）
const str2 = <string>value
```

### 实际应用

```typescript
// 1. JSON 解析
const json = '{"name": "Alice", "age": 25}'
const data = JSON.parse(json) as { name: string; age: number }

// 2. DOM 类型断言
const input = document.getElementById("myInput") as HTMLInputElement
input.value = "Hello"

// 3. 接口实现
interface Config {
    url: string
}

const config = { url: "https://api.example.com" } as Config

// 4. 双重断言（不推荐）
const value: unknown = "hello"
const str = value as unknown as number  // 先转 unknown，再转 number
```

### 类型断言 vs 类型守卫

```typescript
// 类型守卫更安全
function isString(value: unknown): value is string {
    return typeof value === "string"
}

if (isString(value)) {
    console.log(value.toUpperCase())  // 安全
}

// 断言不检查，可能运行时出错
const str = value as string
console.log(str.toUpperCase())  // 如果 value 不是 string，运行时出错
```

---

## 5. typeof 操作符

### 概念解释

TypeScript/JavaScript 中的 `typeof` 有**两种完全不同的用途**，必须加以区分：

- **JavaScript `typeof` 运算符**：在**运行时**执行，返回值类型的字符串（如 `"string"`、`"number"`、`"object"`），用于类型守卫。
- **TypeScript `typeof` 类型查询**：在**编译时**执行，用于获取变量的 TypeScript 类型，只能在类型位置使用。

### Go 对比

Go 有 `reflect.TypeOf()`，属于运行时反射，更接近 JS 的 `typeof`。TypeScript 的类型层 `typeof` 是纯编译期概念，Go 没有等价物。

### JavaScript typeof（运行时，类型守卫）

```typescript
// 运行时 typeof：返回字符串，用于分支判断
function printValue(value: string | number | boolean) {
    if (typeof value === "string") {
        console.log(value.toUpperCase())  // TypeScript 知道这里是 string
    } else if (typeof value === "number") {
        console.log(value.toFixed(2))     // TypeScript 知道这里是 number
    } else {
        console.log(value)                // boolean
    }
}
```

### TypeScript typeof（编译时，类型查询）

```typescript
// 获取变量类型
let name = "Alice"
type NameType = typeof name  // string

// 获取函数返回类型
function getUser() {
    return { id: 1, name: "Bob" }
}

type UserFunc = typeof getUser    // () => { id: number; name: string }
type ReturnType = ReturnType<typeof getUser>  // { id: number; name: string }

// 获取对象字面量类型
const config = {
    port: 8080,
    host: "localhost",
    debug: false
}

type Config = typeof config
// {
//     port: number
//     host: string
//     debug: boolean
// }
```

### 结合 const

```typescript
// 使用 const 获取字面量类型
const ROUTES = {
    home: "/",
    about: "/about",
    contact: "/contact"
} as const

type Route = (typeof ROUTES)[keyof typeof ROUTES]
// "/" | "/about" | "/contact"
```

### 真实案例

**文件：`src/types/permissions.ts`**
```typescript
// 从常量推导类型
export const EXTERNAL_PERMISSION_MODES = [
    'acceptEdits',
    'bypassPermissions',
    'default',
    'dontAsk',
    'plan',
] as const

export type ExternalPermissionMode = (typeof EXTERNAL_PERMISSION_MODES)[number]
// "acceptEdits" | "bypassPermissions" | "default" | "dontAsk" | "plan"
```

---

## 6. keyof 操作符

### 概念解释

keyof 获取对象类型的所有键类型。

### Go 对比

Go 没有等价的操作符。

### TypeScript keyof

```typescript
// 获取类型的所有键
interface User {
    id: number
    name: string
    email: string
}

type UserKeys = keyof User  // "id" | "name" | "email"

// 遍历键
function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key]
}

const user: User = { id: 1, name: "Alice", email: "alice@example.com" }

getValue(user, "id")    // OK，返回 number
getValue(user, "name")  // OK，返回 string
getValue(user, "age")   // 错误！"age" 不在 User 中
```

### 实际应用

```typescript
// 通用 pick 函数
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
    const result = {} as Pick<T, K>
    for (const key of keys) {
        result[key] = obj[key]
    }
    return result
}

const user = { id: 1, name: "Alice", email: "alice@example.com" }

const picked = pick(user, ["id", "name"])
// { id: 1, name: "Alice" }

// 通用 omit 函数
function omit<T, K extends keyof T>(obj: T, keys: K[]): Omit<T, K> {
    const result = { ...obj }
    for (const key of keys) {
        delete result[key]
    }
    return result as Omit<T, K>
}

const withoutEmail = omit(user, ["email"])
// { id: 1, name: "Alice" }
```

---

## 7. in 操作符

### 概念解释

in 检查属性是否存在于对象中。

### Go 对比

Go 用 map 或反射检查。

### TypeScript in

```typescript
// 基本用法
interface Dog {
    bark(): void
}

interface Cat {
    meow(): void
}

function speak(animal: Dog | Cat): void {
    if ("bark" in animal) {
        animal.bark()
    } else {
        animal.meow()
    }
}

// 用于类型收窄
type Event =
    | { type: "click"; x: number }
    | { type: "keydown"; key: string }

function handle(event: Event): void {
    if ("x" in event) {
        console.log(`Click at ${event.x}`)
    } else {
        console.log(`Key: ${event.key}`)
    }
}
```

---

## 8. 实战练习

### 练习 1：可选链

```typescript
// 使用可选链安全访问嵌套对象
interface Company {
    name: string
    address?: {
        city?: string
        zip?: {
            code: string
        }
    }
}

const company: Company = {
    name: "Acme",
    address: {
        city: "Beijing"
        // zip 未定义
    }
}

// 安全获取城市
const city = company?.address?.city ?? "Unknown"
console.log(city)  // "Beijing"

// 安全获取邮编
const zip = company?.address?.zip?.code ?? "000000"
console.log(zip)  // "000000"
```

### 练习 2：空值合并

```typescript
// 实现配置获取函数
interface Config {
    port?: number
    host?: string
    debug?: boolean
    retries?: number
}

function getConfig(): Config {
    return {
        port: 0,
        host: "",
        debug: false,
        retries: undefined
    }
}

const config = getConfig()

// 使用 ?? 设置默认值
const port = config.port ?? 8080
const host = config.host ?? "localhost"
const debug = config.debug ?? false
const retries = config.retries ?? 3

console.log(`Server at ${host}:${port}, debug=${debug}, retries=${retries}`)
// Server at :0, debug=false, retries=3（使用了配置的实际值）
```

### 练习 3：类型守卫

```typescript
// 实现类型守卫
interface Cat {
    type: "cat"
    meow(): void
}

interface Dog {
    type: "dog"
    bark(): void
}

interface Fish {
    type: "fish"
    swim(): void
}

type Animal = Cat | Dog | Fish

function isCat(animal: Animal): animal is Cat {
    return animal.type === "cat"
}

function isDog(animal: Animal): animal is Dog {
    return animal.type === "dog"
}

function speak(animal: Animal): void {
    if (isCat(animal)) {
        animal.meow()
    } else if (isDog(animal)) {
        animal.bark()
    } else {
        animal.swim()
    }
}

speak({ type: "cat", meow: () => console.log("Meow!") })
speak({ type: "dog", bark: () => console.log("Woof!") })
```

### 练习 4：keyof + 泛型

```typescript
// 实现通用更新函数
function update<T, K extends keyof T>(
    obj: T,
    key: K,
    value: T[K]
): T {
    return { ...obj, [key]: value }
}

interface User {
    id: number
    name: string
    age: number
}

const user: User = { id: 1, name: "Alice", age: 25 }

const updated1 = update(user, "name", "Bob")
console.log(updated1)  // { id: 1, name: "Bob", age: 25 }

const updated2 = update(user, "age", 30)
console.log(updated2)  // { id: 1, name: "Alice", age: 30 }

// update(user, "name", 123)  // 错误！类型不匹配
```

---

## 本课总结

### 核心操作符

| 操作符 | 语法 | 说明 |
|--------|------|------|
| 可选链 | `obj?.prop?.nested` | 安全访问嵌套属性 |
| 空值合并 | `a ?? b` | null/undefined 时用 b |
| 非空断言 | `value!` | 断言非空（危险）|
| 类型断言 | `value as Type` | 强制类型转换 |
| typeof | `typeof value` | 获取运行时类型 |
| keyof | `keyof T` | 获取类型的所有键 |
| in | `"key" in obj` | 检查属性存在 |

### 对比 Go

| Go | TypeScript |
|----|------------|
| `if v != nil && v.Field != nil` | `v?.Field?.Nested` |
| `v != nil ? v : default` | `v ?? default` |
| `T(v)` 类型转换 | `v as T` |
| 无等价 | `typeof v` |
| 无等价 | `keyof T` |

### 安全建议

| 不安全 | 安全 |
|--------|------|
| `obj!.field` | `obj?.field` |
| `v as string` | 类型守卫 `isString(v)` |

---

## 下一步

学习 [第九课：模块系统](./lesson_09_modules.md)

---

*本课完*
