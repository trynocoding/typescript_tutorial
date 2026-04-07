# 番外篇：异步编程前置知识铺垫

> 本篇是第七课的前导读物。如果你对 Promise、async/await 感到陌生，请先通读本篇，再回头看第七课，你会发现所有东西都水到渠成。

---

## 目录

1. [函数是一等公民：先理解这个，其他都好说](#1-函数是一等公民先理解这个其他都好说)
2. [回调函数：一切的根源](#2-回调函数一切的根源)
3. [从"回调地狱"到 Promise：历史演化的必然](#3-从回调地狱到-promise历史演化的必然)
4. [手术刀式解剖 Promise 语法](#4-手术刀式解剖-promise-语法)
5. [async/await：语法糖的本质](#5-asyncawait语法糖的本质)
6. [Generator：yield 的直觉建立](#6-generatoryield-的直觉建立)
7. [AsyncGenerator：把前面所有东西串起来](#7-asyncgenerator把前面所有东西串起来)
8. [一张图总结所有关系](#8-一张图总结所有关系)

---

## 1. 函数是一等公民：先理解这个，其他都好说

在 Go 里，函数当然也可以作为参数传递，但在 JavaScript/TypeScript 中，**函数和数字、字符串一样，是一个普通的值**。这个概念在 TS 异步编程里无处不在。

```typescript
// 函数可以赋值给变量
const greet = function(name: string): string {
    return `Hello, ${name}`
}

// 函数可以作为参数传入另一个函数
function runTwice(fn: (name: string) => string, name: string) {
    console.log(fn(name))
    console.log(fn(name))
}

runTwice(greet, "Alice")
// Hello, Alice
// Hello, Alice
```

### 箭头函数：让传递函数更简洁

箭头函数 `=>` 是普通函数的简写形式，它们**功能完全等价**（先忽略 `this` 的区别）：

```typescript
// 普通函数定义
function add(a: number, b: number): number {
    return a + b
}

// 箭头函数，完全等价
const addArrow = (a: number, b: number): number => {
    return a + b
}

// 单行箭头函数，可省略 return 和大括号
const addShort = (a: number, b: number): number => a + b

// 单参数时可省略小括号
const double = (n: number): number => n * 2
```

你在第七课里会看到大量 `() => { ... }` 这样的写法，这只是"定义了一个匿名函数并立刻作为参数传入"。

---

## 2. 回调函数：一切的根源

### 什么是回调

"回调"（callback）就是**把一个函数传给另一个函数，让它在某个时机调用你传入的函数**。这是 JS 处理异步的最原始方式。

```typescript
// setTimeout 接收一个"回调函数"作为第一个参数
// 它会在 1000ms 后"回来调用"这个函数
setTimeout(function() {
    console.log("1秒后执行")
}, 1000)

// 用箭头函数写法更简洁（完全等价）
setTimeout(() => {
    console.log("1秒后执行")
}, 1000)
```

剖析一下 `setTimeout` 的签名：

```typescript
// 伪代码：setTimeout 的函数签名
function setTimeout(
    callback: () => void,   // 第一个参数：一个无参数、无返回值的函数
    delay: number           // 第二个参数：延迟毫秒数
): void // 注：实际环境会返回一个 timerId (数字或对象)，用于清除定时器
```

> **关键直觉**：`() => { ... }` 不是"立刻执行的代码块"，它是"定义了一个函数，交给 setTimeout 保管，等时机到了再执行"。

### 为什么需要回调？

因为 JS 是单线程的。如果直接"等"1秒，整个程序就僵死了（网页会卡住）。所以 JS 改了一个思路：

> "我不等，我告诉你完成后该做什么（回调），然后我继续干别的事。"

```typescript
console.log("第1行：立刻执行")

setTimeout(() => {
    console.log("第3行：1秒后执行")
}, 1000)

console.log("第2行：立刻执行")

// 实际输出顺序：
// 第1行：立刻执行
// 第2行：立刻执行
// 第3行：1秒后执行
```

---

## 3. 从"回调地狱"到 Promise：历史演化的必然

### 回调地狱是什么

当异步操作需要嵌套时，代码会横向失控：

```typescript
// 想象：先登录，再获取用户信息，再获取用户的帖子，再……
login(username, password, (user) => {
    fetchProfile(user.id, (profile) => {
        fetchPosts(profile.id, (posts) => {
            fetchComments(posts[0].id, (comments) => {
                // 嵌套到这里，已经不知道自己在哪了
                console.log(comments)
            })
        })
    })
})
```

这就是臭名昭著的**"回调地狱"（Callback Hell）**：

- 代码向右延伸，无法阅读
- 错误处理极其繁琐（每层都要检查错误）
- 没法"等待多个异步同时完成"

### Promise 是解决方案

Promise 是 ES2015（ES6）引入的语言特性，它把"未来才会有结果的值"包装成一个**对象**，让你可以用链式调用代替嵌套调用。

---

## 4. 手术刀式解剖 Promise 语法

### 4.1 创建一个 Promise

```typescript
const p = new Promise<string>((resolve, reject) => {
    // 这里是执行器函数（executor）
    // 它会"立刻同步执行"
    
    if (/* 成功 */ true) {
        resolve("成功结果")   // 调用 resolve，Promise 状态变为 Fulfilled
    } else {
        reject(new Error("失败原因"))  // 调用 reject，状态变为 Rejected
    }
})
```

**逐行拆解：**

| 代码片段 | 含义 |
|---------|------|
| `new Promise<string>(...)` | 创建一个 Promise 对象，泛型 `<string>` 说明成功时的值类型 |
| `(resolve, reject) => { ... }` | 这是"执行器函数"，是你传给 Promise 构造函数的回调 |
| `resolve` | 是一个函数，调用它 = 宣告"成功了，结果是…" |
| `reject` | 是一个函数，调用它 = 宣告"失败了，原因是…" |

> **关键直觉**：`resolve` 和 `reject` 不是你定义的，是 Promise 内部创建好传给你的两个"内部遥控器"，你决定什么时候按哪个。

### 4.2 Promise 的三种状态（是状态机！）

```
           resolve()
Pending ─────────────► Fulfilled（已兑现）
   │
   │     reject()
   └──────────────────► Rejected（已拒绝）
```

- **Pending**：初始状态，还没有结果
- **Fulfilled**：`resolve(value)` 被调用，状态永久固化，不可再改变
- **Rejected**：`reject(reason)` 被调用，状态永久固化

```typescript
// 状态一旦确定，就永远不变
const p = new Promise<number>((resolve, reject) => {
    resolve(1)      // ✅ 状态变为 Fulfilled，值为 1
    resolve(2)      // ❌ 无效！状态已经固化（注意：JS 会静默忽略，不会像 Go 抛出 panic）
    reject(new Error("x"))  // ❌ 无效！状态已经固化（静默忽略）
})
```

### 4.3 消费 Promise：.then() / .catch() / .finally()

```typescript
const p: Promise<string> = new Promise((resolve) => {
    setTimeout(() => resolve("hello"), 1000)
})

p
  .then((result) => {       // result 的类型自动推断为 string
      console.log(result)      // "hello"
      return result.length     // ★ then 里 return 的值，会变成下一个 then 的输入
  })
  .then((len) => {          // len 的类型自动推断为 number
      console.log("长度:", len)
  })
  .catch((error) => {       // 捕获链上任意位置的错误
      console.error("出错了:", error)
  })
  .finally(() => {          // 无论成功或失败，都执行
      console.log("清理完毕")
  })
```

**`.then()` 的关键规则（很多人在这里卡住）：**

`.then(fn)` 中 `fn` 的返回值决定了下一个 `.then` 接收什么：

```typescript
Promise.resolve(1)
    .then(n => n + 1)           // 返回普通值 2  ─► 下一个 then 得到 2
    .then(n => Promise.resolve(n * 10))  // 返回 Promise<20>  ─► Promise 被"展开"，下一个 then 得到 20
    .then(n => {
        console.log(n)           // 20
    })
```

> **规则**：如果 `.then` 的回调返回的是一个 Promise，下一个 `.then` 会等待它完成，拿到其内部值——Promise 不会嵌套成 `Promise<Promise<T>>`，而是自动"拍平（flatten）"。

### 4.4 一些容易混淆的细节

**执行器函数是同步的：**

```typescript
console.log("1")

new Promise<void>((resolve) => {
    console.log("2")  // ← 这行是同步执行的！！
    resolve()
})

console.log("3")

// 输出：1, 2, 3（不是 1, 3, 2）
```

但 `.then()` 里的回调是异步的（微任务队列）：

```typescript
console.log("1")

Promise.resolve().then(() => {
    console.log("3")  // ← .then 的回调是异步的，放入微任务队列
})

console.log("2")

// 输出：1, 2, 3
```

**Promise.resolve() 快速创建已完成的 Promise：**

```typescript
// 这两种写法完全等价
const p1 = new Promise<number>((resolve) => resolve(42))
const p2 = Promise.resolve(42)

// 用于测试或快速返回值时很方便
function getConfig(): Promise<{debug: boolean}> {
    return Promise.resolve({ debug: true })
}
```

---

## 5. async/await：语法糖的本质

### 5.1 什么是"语法糖"

语法糖（Syntactic Sugar）是指：**一种让代码更好写、更好读的简化写法，但底层完全等价于已有语法**。

`async/await` 就是 Promise 的语法糖。理解这一点至关重要：**用 async/await 写的代码，编译器会把它翻译成用 Promise 写的等价代码**。

### 5.2 async 关键字

`async` 加在函数前，做了**两件事**：

1. 让函数的返回值自动包装进 Promise
2. 允许函数体内使用 `await`

```typescript
// 不用 async，手动包装 Promise
function getName(): Promise<string> {
    return Promise.resolve("Alice")
}

// 用 async，等价写法
async function getName(): Promise<string> {
    return "Alice"           // ← 这个 string 被自动包进 Promise<string>
}

// 二者调用方式完全相同
getName().then(name => console.log(name))
```

**async 函数的返回值规则：**

```typescript
async function demo(): Promise<string> {
    return "hello"         // 相当于 return Promise.resolve("hello")
}

async function demo2(): Promise<string> {
    return Promise.resolve("hello")  // 如果已经是 Promise，不会双重包装
}

async function demo3(): Promise<never> {
    throw new Error("oops")  // 相当于 return Promise.reject(new Error("oops"))
}
```

### 5.3 await 关键字

`await` 只能在 `async` 函数内使用，它做了**一件事**：

> **等待一个 Promise 完成，然后"解包"出它的值。**

```typescript
// 不用 await（Promise 链式写法）
function getUser(id: number): Promise<string> {
    return fetchUser(id).then(user => {
        return user.name
    })
}

// 用 await（等价，但像同步代码一样流畅）
async function getUser(id: number): Promise<string> {
    const user = await fetchUser(id)   // ← 等待 Promise 完成，解包出 User 对象
    return user.name                   // ← 直接用解包后的值
}
```

**`await` 的类型推断：**

```typescript
async function demo() {
    const p: Promise<number> = Promise.resolve(42)

    const n = await p   // n 的类型：number（不是 Promise<number>！）
    //         ↑
    // await 把 Promise<T> 解包成 T
}
```

### 5.4 错误处理的对应关系

| Promise 写法 | async/await 写法 |
|----|-----|
| `.then(fn)` | `await` 后直接写代码 |
| `.catch(fn)` | `try { ... } catch(e) { ... }` |
| `.finally(fn)` | `try { ... } finally { ... }` |

```typescript
// Promise 链式写法
fetchUser(1)
    .then(user => user.name)
    .catch(err => console.error(err))
    .finally(() => console.log("done"))

// async/await 等价写法
async function main() {
    try {
        const user = await fetchUser(1)
        console.log(user.name)
    } catch (err) {
        console.error(err)
    } finally {
        console.log("done")
    }
}
```

### 5.5 经典误区：串行 vs 并行

**错误写法（串行，慢）：**

```typescript
async function wrong() {
    const user1 = await fetchUser(1)   // 等 fetchUser(1) 完成再请求 fetchUser(2)
    const user2 = await fetchUser(2)   // 两个请求顺序执行，总时间翻倍
    return [user1, user2]
}
```

**正确写法（并行，快）：**

```typescript
async function correct() {
    // 先同时发出两个请求，再一起等待
    const [user1, user2] = await Promise.all([fetchUser(1), fetchUser(2)])
    return [user1, user2]
}
```

> `await` 只是"等待单个 Promise"。要并行等多个，先用 `Promise.all` 把它们合并成一个 Promise，再 `await` 这一个。

---

## 6. Generator：yield 的直觉建立

在学 AsyncGenerator 之前，必须先理解普通 Generator。

### 6.1 Generator 是"可暂停的函数"

普通函数从头跑到尾，期间无法暂停。Generator 函数（用 `function*` 声明）可以在 `yield` 处暂停，把控制权还给调用方，等调用方准备好了再继续。

```typescript
// function* 声明一个 Generator 函数
function* counter(): Generator<number> {
    console.log("开始")
    yield 1              // ← 暂停在这里，把 1 返回给调用方
    console.log("继续")
    yield 2              // ← 再次暂停，把 2 返回给调用方
    console.log("结束")
    // 函数执行完毕
}
```

**调用 Generator 函数不会立即执行！** 它返回一个迭代器对象：

```typescript
const gen = counter()   // ← 不执行任何代码，只创建迭代器

console.log("调用 next 之前")
console.log(gen.next())  // { value: 1, done: false }  ← 执行到第一个 yield
console.log(gen.next())  // { value: 2, done: false }  ← 执行到第二个 yield
console.log(gen.next())  // { value: undefined, done: true }  ← 执行到函数结束

// 控制台完整输出：
// 调用 next 之前
// 开始
// { value: 1, done: false }
// 继续
// { value: 2, done: false }
// 结束
// { value: undefined, done: true }
```

> **关键直觉**：每次调用 `.next()` 就像"放行一步"。Generator 是一个"按需执行"的机器，你不拉，它就不动。

### 6.2 用 for...of 自动消费

手动调用 `.next()` 太麻烦，`for...of` 可以自动帮你调用：

```typescript
function* range(start: number, end: number): Generator<number> {
    for (let i = start; i <= end; i++) {
        yield i
    }
}

// for...of 自动调用 next()，当 done: true 时停止
for (const n of range(1, 5)) {
    console.log(n)  // 1, 2, 3, 4, 5
}
```

---

## 7. AsyncGenerator：把前面所有东西串起来

`AsyncGenerator` = Generator（可暂停）+ Promise（异步等待）

### 7.1 语法对比

```typescript
// 普通 Generator（同步）
function* gen(): Generator<number> {
    yield 1
    yield 2
}

// Async Generator（异步）：加 async* 
async function* asyncGen(): AsyncGenerator<number> {
    await delay(100)   // ← 可以在内部 await 一个 Promise
    yield 1
    await delay(100)
    yield 2
}
```

| 特性 | Generator | AsyncGenerator |
|------|-----------|---------------|
| 声明 | `function*` | `async function*` |
| 产出 | `yield value` | `yield value`（value 可以是 Promise）|
| 内部等待 | 不支持 `await` | 支持 `await` |
| 返回类型 | `Generator<T>` | `AsyncGenerator<T>` |
| 消费 | `for...of` | `for await...of` |

### 7.2 完整示例：逐步理解

```typescript
// ① 定义一个辅助延迟函数
function delay(ms: number): Promise<void> {
    return new Promise<void>(resolve => setTimeout(resolve, ms))
}

// ② 定义异步生成器
async function* fetchLogs(ids: number[]): AsyncGenerator<string> {
    for (const id of ids) {
        await delay(500)                      // 异步等待（模拟网络请求）
        yield `Log entry #${id}`             // 产出一个值
    }
}

// ③ 消费：for await...of
async function main() {
    for await (const log of fetchLogs([1, 2, 3])) {
        //                ↑
        // 每次迭代：
        // - 调用 fetchLogs 内部的 .next()
        // - 等待 await delay(500) 完成
        // - 等待 yield 产出的值（如果是 Promise，会自动解包）
        // - log 拿到解包后的 string
        console.log(log)
    }
}

main()
// 每隔500ms输出一行：
// Log entry #1
// Log entry #2
// Log entry #3
```

### 7.3 AsyncGenerator 的"拉模型"直觉

**关键**：AsyncGenerator 是**拉模型**（消费者驱动）。

```
消费者（for await）         生成器（async function*）

    请求下一个值 ─────────────► 开始执行
                               await delay(500)   ← 等待异步
                               yield "Log #1"     ← 暂停，把值送回来
    收到 "Log #1" ◄────────────
    处理值...
    准备好了，请求下一个 ──────► 继续执行
                               await delay(500)
                               yield "Log #2"
    收到 "Log #2" ◄────────────
    ...
```

不像 Go 的 Channel（生产者主动 push），AsyncGenerator 是"你不来拿，我就不动"。

---

## 8. 一张图总结所有关系

```
回调函数 (callback)
    │
    │ 解决：嵌套地狱、错误处理困难
    ▼
Promise                 ─── new Promise((resolve, reject) => {...})
    │                   ─── .then() / .catch() / .finally()
    │                   ─── Promise.all() / .race() / .allSettled()
    │
    │ 语法糖（编译器翻译）
    ▼
async / await           ─── async function 自动包装返回值为 Promise
    │                   ─── await 把 Promise<T> 解包成 T
    │                   ─── try/catch 对应 .catch()
    │
Generator               ─── function* / yield / .next()
    │                   ─── for...of 消费
    │
    │ 交叉融合
    ▼
AsyncGenerator          ─── async function* / yield / await
                        ─── for await...of 消费
```

### 用一句话概括每个概念

| 概念 | 一句话 |
|------|--------|
| **回调函数** | "完成后帮我调用这个函数" |
| **Promise** | "给你一张承诺单，未来会有结果，成功或失败" |
| **async** | "这个函数返回 Promise，你可以在里面写 await" |
| **await** | "等这个 Promise 完成，把结果解包给我" |
| **Generator** | "我能暂停自己，让你按需取值" |
| **AsyncGenerator** | "我既能暂停，又能在暂停前做异步操作" |

---

## 现在回头读第七课

读完本篇后，再回到第七课，你会对以下代码一目了然：

```typescript
// 第七课 §2 代码
function fetchUser(id: number): Promise<{ id: number; name: string }> {
    return new Promise((resolve, reject) => {
        // ① executor 同步执行
        setTimeout(() => {
            // ② 1秒后，异步调用 resolve
            if (id > 0) {
                resolve({ id, name: `User ${id}` })
                // ③ Promise 状态变为 Fulfilled，值被存储
            } else {
                reject(new Error("Invalid user ID"))
            }
        }, 1000)
    })
    // ④ new Promise(...) 执行器同步执行完毕，但 setTimeout 还没触发
    // ⑤ 返回一个处于 Pending 状态的 Promise 给调用方
}
```

```typescript
// 第七课 §3 代码
async function getUser(id: number): Promise<string> {
    // ① async 函数，内部可以用 await
    const user = await fetchUser(id)
    // ②     ↑ await 把 Promise<{id,name}> 解包成 {id,name}
    // ③ user 的类型是 { id: number; name: string }，不是 Promise！
    return user.name
    // ④ 返回的 string 被 async 自动包进 Promise<string>
}
```

```typescript
// 第七课 §4 代码
async function* readLines(lines: string[]): AsyncGenerator<{line: string; lineNum: number}> {
    for (let i = 0; i < lines.length; i++) {
        await new Promise<void>(resolve => setTimeout(resolve, 100))
        // ↑ 每次 yield 前先等 100ms（异步操作）
        yield { line: lines[i], lineNum: i + 1 }
        // ↑ 产出当前行，暂停等待消费者调用 next()
    }
}

// for await...of 是消费者，它每次调用 .next() 触发一次 yield
for await (const { line, lineNum } of readLines(["Hello", "World"])) {
    console.log(`${lineNum}: ${line}`)
}
```

---

## 快速自测

在继续读第七课之前，先测试自己是否真的掌握了：

1. `new Promise((resolve, reject) => { ... })` 中的 `resolve` 和 `reject` 是谁定义的？
2. `.then()` 的回调如果返回一个 `Promise`，下一个 `.then()` 收到的是什么？
3. `async function` 和普通 `function` 的主要区别是什么？
4. `await somePromise` 和 `somePromise.then(...)` 有什么关系？
5. `for await...of` 和 `for...of` 的区别是什么？

**参考答案（展开查看）：**

1. `resolve` 和 `reject` 是 **Promise 内部创建并注入**给执行器的两个函数，你只负责调用它们。
2. 下一个 `.then()` 会等待这个 `Promise` 完成，收到其**内部值**（不是嵌套 Promise）。
3. `async function` 返回值**自动包装为 Promise**，且函数体内可以使用 `await`。
4. `await` 是 `Promise.then()` 的**语法糖**，等价但写法更接近同步代码。
5. `for await...of` 用于消费 **AsyncGenerator**，会自动 `await` 每次迭代产生的 Promise；`for...of` 用于消费同步 Generator 或可迭代对象。

---

> 恭喜！现在请回头阅读 **第七课：异步编程**，游刃有余地享受它吧。
>
> 👉 [第七课：异步编程](./lesson_07_async.md)

---

*番外篇完*
