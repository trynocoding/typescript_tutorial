# 第七课：异步编程

> 本课讲解 TypeScript 的异步编程模式

---

## 目录

1. [异步编程简介](#1-异步编程简介)
2. [Promise 基础](#2-promise-基础)
3. [async/await](#3-asyncawait)
4. [异步生成器](#4-异步生成器)
5. [错误处理](#5-错误处理)
6. [并行执行](#6-并行执行)
7. [真实案例分析](#7-真实案例分析)
8. [实战练习](#8-实战练习)

---

## 1. 异步编程简介

### Go 对比

Go 使用 goroutine 和 channel 实现并发：

```go
// Go 的并发模式
ch := make(chan string)

go func() {
    result := doWork()
    ch <- result  // 发送结果
}()

result := <-ch  // 接收结果

// 或者用 WaitGroup
var wg sync.WaitGroup
wg.Add(1)
go func() {
    defer wg.Done()
    doWork()
}()
wg.Wait()
```

### TypeScript 异步模型

TypeScript/JavaScript 使用 Promise + async/await：

```typescript
// Promise：表示一个异步操作的最终结果
const promise = new Promise<string>((resolve, reject) => {
    setTimeout(() => {
        resolve("Hello")
    }, 1000)
})

// 等待 Promise 完成
promise.then(result => {
    console.log(result)  // "Hello"
})
```

### JavaScript 事件循环

JavaScript 是单线程的，但通过事件循环实现异步：

```typescript
console.log("1")

setTimeout(() => {
    console.log("3")  // 异步，最后执行
}, 0)

console.log("2")

// 输出：1, 2, 3
```

---

## 2. Promise 基础

### 概念解释

Promise 表示一个异步操作的最终结果。它有三种状态：
- **Pending**（待定）：初始状态
- **Fulfilled**（已兑现）：操作成功
- **Rejected**（已拒绝）：操作失败

### Go 对比

Go 用 channel 和 error：

```go
// Go 的模式
result, err := doWork()
if err != nil {
    // 处理错误
    return err
}
// 使用 result
```

### TypeScript Promise

```typescript
// 创建 Promise
const promise = new Promise<string>((resolve, reject) => {
    // 异步操作
    setTimeout(() => {
        const success = true
        if (success) {
            resolve("Done!")  // 成功
        } else {
            reject(new Error("Failed"))  // 失败
        }
    }, 1000)
})

// 处理结果
promise
    .then(result => {
        console.log(result)  // "Done!"
    })
    .catch(error => {
        console.error(error)  // Error: Failed
    })
    .finally(() => {
        console.log("Cleanup")  // 总是执行
    })
```

### Promise 状态转换

```
Pending ──► Fulfilled ──► (resolved)
    │
    └──► Rejected ──► (rejected)
```

### 创建 Promise

```typescript
// 模拟 API 调用
function fetchUser(id: number): Promise<{ id: number; name: string }> {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (id > 0) {
                resolve({ id, name: `User ${id}` })
            } else {
                reject(new Error("Invalid user ID"))
            }
        }, 1000)
    })
}

// 使用
fetchUser(1)
    .then(user => console.log(user))
    .catch(err => console.error(err))
```

### Promise 链式调用

```typescript
// 链式调用
fetchUser(1)
    .then(user => fetchPosts(user.id))           // 返回 Promise<Post[]>
    .then(posts => posts.filter(p => p.published)) // 过滤
    .then(posts => posts.map(p => p.title))        // 转换
    .then(titles => console.log(titles.join("\n")))
    .catch(err => console.error(err))
```

### 静态方法

```typescript
// Promise.resolve - 创建已解决的 Promise
const resolved = Promise.resolve({ id: 1, name: "Alice" })

// Promise.reject - 创建已拒绝的 Promise
const rejected = Promise.reject(new Error("Oops"))

// Promise.all - 等待所有 Promise 完成
const results = await Promise.all([
    fetchUser(1),
    fetchUser(2),
    fetchUser(3)
])
console.log(results)  // [User1, User2, User3]

// Promise.race - 返回第一个完成的
const first = await Promise.race([
    fetchUser(1),
    fetchUser(2)
])
```

---

## 3. async/await

### 概念解释

async/await 是 Promise 的语法糖，让异步代码看起来像同步代码。

### Go 对比

Go 本身就是同步语法风格：

```go
// Go
result, err := doWork()
if err != nil {
    return err
}
use(result)
```

### TypeScript async/await

```typescript
// async 函数自动返回 Promise
async function getUser(id: number): Promise<string> {
    // await 等待 Promise 完成
    const user = await fetchUser(id)
    return user.name
}

// 调用 async 函数
getUser(1)
    .then(name => console.log(name))
    .catch(err => console.error(err))

// 或者用 await（只能在 async 函数中）
async function main() {
    try {
        const name = await getUser(1)
        console.log(name)
    } catch (error) {
        console.error(error)
    }
}
```

### 详细语法

```typescript
// async 函数声明
async function fetchData(): Promise<string> {
    return "data"
}

// async 箭头函数
const fetchDataAsync = async (): Promise<string> => {
    return "data"
}

// await 只能在 async 函数中使用
async function process() {
    // 等待 Promise
    const result = await fetchData()
    return result.toUpperCase()
}
```

### 实际应用

```typescript
// 模拟数据库查询
interface User {
    id: number
    name: string
}

interface Post {
    id: number
    userId: number
    title: string
}

const users: User[] = [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" }
]

const posts: Post[] = [
    { id: 1, userId: 1, title: "Hello" },
    { id: 2, userId: 2, title: "World" }
]

async function getUserById(id: number): Promise<User | null> {
    return new Promise(resolve => {
        setTimeout(() => {
            const user = users.find(u => u.id === id)
            resolve(user || null)
        }, 100)
    })
}

async function getPostsByUserId(userId: number): Promise<Post[]> {
    return new Promise(resolve => {
        setTimeout(() => {
            const userPosts = posts.filter(p => p.userId === userId)
            resolve(userPosts)
        }, 100)
    })
}

// 组合查询
async function getUserWithPosts(userId: number) {
    const user = await getUserById(userId)
    if (!user) {
        return null
    }

    const userPosts = await getPostsByUserId(userId)
    return { user, posts: userPosts }
}

async function main() {
    const result = await getUserWithPosts(1)
    if (result) {
        console.log(`${result.user.name}'s posts:`)
        result.posts.forEach(p => console.log(`- ${p.title}`))
    }
}

main()
```

---

## 4. 异步生成器

### 概念解释

异步生成器（Async Generator）是 TypeScript/ES2018 引入的特性，可以产生异步的值序列。

### Go 对比

Go 用 channel 实现流式数据：

```go
// Go 流式处理
func generate(nums ...int) <-chan int {
    ch := make(chan int)
    go func() {
        for _, n := range nums {
            ch <- n
        }
        close(ch)
    }()
    return ch
}

for n := range generate(1, 2, 3) {
    fmt.Println(n)
}
```

### TypeScript AsyncGenerator

```typescript
// 异步生成器函数
async function* fetchPages(url: string): AsyncGenerator<string> {
    let page = 0
    while (true) {
        const response = await fetch(`${url}?page=${page}`)
        const text = await response.text()

        if (text === "") {
            break  // 没有更多数据
        }

        yield text  // yield 返回当前页
        page++
    }
}

// 使用 for await...of
async function main() {
    for await (const page of fetchPages("/api/data")) {
        console.log(`Page: ${page}`)
    }
}

main()
```

### 生成器基础

```typescript
// 普通生成器
function* numberGenerator(): Generator<number> {
    yield 1
    yield 2
    yield 3
}

const gen = numberGenerator()
console.log(gen.next())  // { value: 1, done: false }
console.log(gen.next())  // { value: 2, done: false }
console.log(gen.next())  // { value: 3, done: false }
console.log(gen.next())  // { value: undefined, done: true }

// for...of 自动调用 next()
for (const num of numberGenerator()) {
    console.log(num)  // 1, 2, 3
}
```

### AsyncGenerator 完整示例

```typescript
// 模拟逐行读取文件
async function* readLines(
    lines: string[]
): AsyncGenerator<{ line: string; lineNum: number }> {
    for (let i = 0; i < lines.length; i++) {
        // 模拟异步延迟
        await new Promise(resolve => setTimeout(resolve, 100))
        yield { line: lines[i], lineNum: i + 1 }
    }
}

async function main() {
    const lines = ["Hello", "World", "TypeScript"]

    for await (const { line, lineNum } of readLines(lines)) {
        console.log(`${lineNum}: ${line}`)
    }
}

main()
// 输出：
// 1: Hello
// 2: World
// 3: TypeScript
```

### 真实案例

**文件：`src/query.ts`**
```typescript
// Claude Code 的 query 函数是异步生成器
export async function* query(
    params: QueryParams,
): AsyncGenerator<
    | StreamEvent
    | RequestStartEvent
    | Message
    | TombstoneMessage
    | ToolUseSummaryMessage,
    Terminal
> {
    // ... 初始化

    // yield 产生各种事件
    yield {
        type: "request_start",
        requestId: params.requestId
    }

    // 循环处理
    while (true) {
        const event = yield* queryLoop(params, consumedCommandUuids)

        if (event.type === "terminal") {
            return event.terminal  // 完成
        }
    }
}

// 使用
for await (const event of query(params)) {
    switch (event.type) {
        case "request_start":
            console.log("Request started")
            break
        case "message":
            console.log("Message:", event.message)
            break
        // ...
    }
}
```

---

## 5. 错误处理

### Try-Catch

```typescript
async function fetchData(): Promise<string> {
    throw new Error("Network error")
}

async function main() {
    try {
        const data = await fetchData()
        console.log(data)
    } catch (error) {
        console.error("Error:", error.message)
    } finally {
        console.log("Cleanup")  // 总是执行
    }
}

main()
```

### 错误类型检查

```typescript
async function process() {
    try {
        const result = await mightFail()
    } catch (error) {
        if (error instanceof TypeError) {
            console.error("Type error:", error.message)
        } else if (error instanceof RangeError) {
            console.error("Range error:", error.message)
        } else if (error instanceof Error) {
            console.error("Error:", error.message)
        }
    }
}
```

### 链式调用的错误处理

```typescript
// Promise.catch 会捕获之前的错误
fetchUser(1)
    .then(user => fetchPosts(user.id))
    .then(posts => posts[0].title)
    .catch(error => {
        console.error("Failed:", error.message)
        return "Default title"  // 恢复
    })
    .then(title => console.log(title))
```

---

## 6. 并行执行

### Promise.all

等待所有 Promise 完成：

```typescript
async function fetchUsers(): Promise<User[]> {
    const responses = await Promise.all([
        fetch("/api/user/1"),
        fetch("/api/user/2"),
        fetch("/api/user/3")
    ])

    return Promise.all(responses.map(r => r.json()))
}
```

### Promise.allSettled

所有 Promise 都完成后返回结果（不抛出）：

```typescript
const results = await Promise.allSettled([
    fetch("/api/1"),
    fetch("/api/2"),
    fetch("/api/invalid")  // 会失败
])

results.forEach((result, i) => {
    if (result.status === "fulfilled") {
        console.log(`User ${i}:`, result.value)
    } else {
        console.error(`User ${i} failed:`, result.reason)
    }
})
```

### Promise.race

返回第一个完成的结果：

```typescript
const result = await Promise.race([
    fetch("/api/fast"),
    new Promise((_, reject) =>
        setTimeout(() => reject(new Error("Timeout")), 5000)
    )
])
```

### Promise.any

返回第一个成功的结果：

```typescript
const result = await Promise.any([
    fetch("/api/1").then(r => r.json()),
    fetch("/api/2").then(r => r.json()),
    fetch("/api/3").then(r => r.json())
])
```

---

## 7. 真实案例分析

### 案例：McpSession 认证

**文件：`src/services/mcp/auth.ts`**
```typescript
class McpSession {
    private _metadata?: Awaited<ReturnType<...>>

    async getAccessToken(): Promise<string> {
        return await this.getAccessTokenImpl()
    }

    private async getAccessTokenImpl(): Promise<string> {
        // 检查缓存
        if (this._metadata?.result?.ok) {
            const token = this._metadata.result.value?.tokens?.access_token
            if (token) {
                return token
            }
        }

        // 获取新 token
        const result = await this.client.getOAuthTokens(...)
        if (!result.ok) {
            throw new Error(`Auth failed: ${result.error}`)
        }

        return result.value.tokens.access_token
    }
}
```

### 案例：流式处理

```typescript
// 模拟流式 API 响应
async function* streamResponses(
    messages: string[]
): AsyncGenerator<string> {
    for (const msg of messages) {
        await new Promise(resolve => setTimeout(resolve, 500))
        yield msg.toUpperCase()
    }
}

async function processStream() {
    console.log("Starting...")

    for await (const response of streamResponses(["hello", "world", "test"])) {
        console.log("Received:", response)
    }

    console.log("Done!")
}

processStream()
// Starting...
// Received: HELLO (after 500ms)
// Received: WORLD (after 500ms)
// Received: TEST (after 500ms)
// Done!
```

---

## 8. 实战练习

### 练习 1：Promise 基础

```typescript
// 实现一个 delay 函数，返回一个 Promise
function delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
}

// 使用 delay
async function run() {
    console.log("Start")
    await delay(1000)
    console.log("After 1 second")
    await delay(1000)
    console.log("After 2 seconds")
}

run()
```

### 练习 2：并行请求

```typescript
interface User {
    id: number
    name: string
}

const userIds = [1, 2, 3]

// 使用 Promise.all 并行获取用户
async function fetchUsersParallel(ids: number[]): Promise<User[]> {
    const promises = ids.map(id =>
        new Promise<User>(resolve => {
            setTimeout(() => {
                resolve({ id, name: `User ${id}` })
            }, Math.random() * 1000)
        })
    )

    return Promise.all(promises)
}

async function main() {
    console.log("Fetching users...")
    const users = await fetchUsersParallel(userIds)
    console.log("All users:", users)
}

main()
```

### 练习 3：异步生成器

```typescript
// 实现一个计数器异步生成器
async function* countUp(
    start: number,
    end: number
): AsyncGenerator<number> {
    for (let i = start; i <= end; i++) {
        await new Promise(resolve => setTimeout(resolve, 100))
        yield i
    }
}

async function main() {
    console.log("Counting up:")
    for await (const num of countUp(1, 5)) {
        console.log(num)
    }
    console.log("Done!")
}

main()
```

### 练习 4：错误恢复

```typescript
// 实现带重试的请求
async function fetchWithRetry<T>(
    fn: () => Promise<T>,
    maxRetries: number = 3
): Promise<T> {
    let lastError: Error

    for (let i = 0; i < maxRetries; i++) {
        try {
            return await fn()
        } catch (error) {
            lastError = error as Error
            console.log(`Attempt ${i + 1} failed: ${lastError.message}`)
            if (i < maxRetries - 1) {
                await delay(1000 * (i + 1))  // 指数退避
            }
        }
    }

    throw new Error(`All ${maxRetries} attempts failed`)
}

// 测试
async function unreliableRequest(): Promise<string> {
    if (Math.random() < 0.7) {
        throw new Error("Random failure")
    }
    return "Success!"
}

async function main() {
    try {
        const result = await fetchWithRetry(unreliableRequest, 3)
        console.log("Result:", result)
    } catch (error) {
        console.error("Final failure:", error)
    }
}

main()
```

---

## 本课总结

### 核心概念

| 概念 | 语法 | 说明 |
|------|------|------|
| Promise | `new Promise((resolve, reject) => {})` | 异步操作封装 |
| then/catch | `promise.then().catch()` | 处理结果/错误 |
| async | `async function fn()` | 声明异步函数 |
| await | `await promise` | 等待 Promise |
| AsyncGenerator | `async function* gen()` | 异步生成器 |
| yield | `yield value` | 产生值 |
| for await...of | `for await (x of gen)` | 遍历异步序列 |
| Promise.all | `Promise.all([p1, p2])` | 并行等待 |
| Promise.race | `Promise.race([p1, p2])` | 返回第一个完成 |
| Promise.allSettled | `Promise.allSettled([...])` | 所有完成后 |

### 对比 Go

| Go | TypeScript |
|----|------------|
| goroutine | Promise + async |
| channel | AsyncGenerator + yield |
| `<-ch` | `for await...of` |
| sync.WaitGroup | Promise.all |
| err != nil | try/catch + if (result.ok) |
| select | Promise.race |

### Promise 状态

```
Pending ──► Fulfilled
    │
    └──► Rejected
```

### AsyncGenerator 流程

```
async function* gen() {
    yield 1      ──► { value: 1, done: false }
    yield 2      ──► { value: 2, done: false }
    return       ──► { value: undefined, done: true }
}
```

---

## 下一步

学习 [第八课：特殊操作符](./lesson_08_operators.md)

---

*本课完*
