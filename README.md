# TypeScript 入门教程：Go 工程师视角

> 目标：让你能读懂 Claude Code TypeScript 源码

---

## 教程目录

| 课程 | 主题 | 优先级 | 预估时长 |
|------|------|--------|----------|
| [第一课](./lesson_01_types.md) | 类型基础 | 🔴 必须掌握 | 20分钟 |
| [第二课](./lesson_02_interfaces.md) | 接口与类型别名 | 🔴 必须掌握 | 25分钟 |
| [第三课](./lesson_03_functions.md) | 函数 | 🔴 必须掌握 | 20分钟 |
| [第四课](./lesson_04_generics.md) | 泛型 | 🔴 必须掌握 | 30分钟 |
| [第五课](./lesson_05_classes.md) | 类 | 🟡 重要 | 25分钟 |
| [第六课](./lesson_06_union_types.md) | 联合类型与区分联合 | 🔴 必须掌握 | 35分钟 |
| [第七课](./lesson_07_async.md) | 异步编程 | 🔴 必须掌握 | 30分钟 |
| [第八课](./lesson_08_operators.md) | 特殊操作符 | 🟡 重要 | 15分钟 |
| [第九课](./lesson_09_modules.md) | 模块系统 | 🟡 重要 | 20分钟 |
| [第十课](./lesson_10_advanced.md) | 高级类型 | 🟢 进阶 | 40分钟 |

---

## Go vs TypeScript 快速对照表

| 概念 | Go | TypeScript |
|------|-----|------------|
| 类型定义 | `type Foo struct{}` | `interface Foo {}` |
| 方法绑定 | `func (f Foo) Method(){}` | `Method() {}`（写在类内）|
| 泛型 | `func Foo[T any](t T) {}` | `function Foo<T>(t: T) {}` |
| 空值 | `nil`（只能用 pointer） | `null` 和 `undefined` |
| 异步 | `goroutine` + `channel` | `Promise` + `async/await` |
| 错误处理 | `return err` | `throw` 或返回 `Promise.reject` |

---

## 推荐学习路径

### 零基础入门（推荐顺序）

1. **第一课** - 先理解 TS 的类型系统
2. **第二课** - 理解如何定义对象结构
3. **第三课** - 学会声明和使用函数
4. **第六课** - 🔴 **重点**：联合类型和区分联合（Claude Code 最常用的模式）
5. **第四课** - 泛型（Tool.ts 等核心文件的基础）
6. **第七课** - 异步编程（query.ts 的核心）
7. **第五课** - 类（如果需要维护面向对象代码）
8. **第八、九课** - 语法糖和模块系统
9. **第十课** - 进阶内容（按需查阅）

### 快速查阅（按需查看）

- 想知道某个 TS 语法怎么写 → 对应课程
- 读懂 Claude Code 具体例子 → 课程内的「真实案例」板块
- 某个类型工具怎么用 → 第十课「工具类型」

---

## 环境准备

### 1. 安装 Node.js（包含 npm）

```bash
# macOS
brew install node

# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Windows
# 从 https://nodejs.org 下载安装
```

验证安装：
```bash
node --version  # 应显示 v20.x 或更高
npm --version   # 应显示 10.x 或更高
```

### 2. 安装 TypeScript 编译器

```bash
# 全局安装
npm install -g typescript

# 验证安装
tsc --version  # 应显示 6.x
```

### 3. 运行示例代码

```bash
# 创建项目目录
mkdir ts-tutorial && cd ts-tutorial

# 初始化 npm 项目
npm init -y

# 安装 TypeScript
npm install typescript --save-dev

# 创建 tsconfig.json
npx tsc --init

# 创建示例文件 hello.ts
echo 'const greeting: string = "Hello, TypeScript!"; console.log(greeting);' > hello.ts

# 编译并运行
npx tsc hello.ts && node hello.js
```

---

## 课程特色

### 1. Go 对比
每个概念都提供 Go 对比，帮助你用已知的知识理解 TS。

### 2. 真实案例
每个重要概念都提供来自 Claude Code 源码的真实案例，标注文件路径。

### 3. 可运行示例
代码块都可以复制运行，配有详细注释。

### 4. 新手友好
从零开始，逐步深入，每个概念都有清晰解释。

---

## 如何使用本教程

1. **从头阅读**：按顺序学习各课程
2. **边学边练**：复制运行每个代码示例
3. **对照源码**：学完后阅读 Claude Code 对应源文件
4. **查漏补缺**：遇到不懂的语法回来查阅

---

## 下一步

开始学习 [第一课：类型基础](./lesson_01_types.md)

---

*教程基于 Claude Code 源码分析生成*
