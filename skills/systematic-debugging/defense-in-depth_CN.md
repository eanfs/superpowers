# 深度防御验证

## 概述

当你修复了一个由无效数据导致的缺陷时，在一个地方添加验证会让人觉得已经足够。但不同的代码路径、重构或 mock 都可能绕过这项单独的检查。

**核心原则：**在数据经过的每一层都进行验证。从结构上杜绝该缺陷。

## 为什么需要多层验证

单层验证：“我们修复了这个缺陷”
多层验证：“我们让这个缺陷变得不可能发生”

不同层可以捕获不同情况：
- 入口验证捕获大多数缺陷
- 业务逻辑捕获边界情况
- 环境防护阻止特定上下文中的危险操作
- 调试日志在其他层失效时提供帮助

## 四个层级

### 第 1 层：入口验证
**目的：**在 API 边界拒绝明显无效的输入

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### 第 2 层：业务逻辑验证
**目的：**确保数据对当前操作有意义

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### 第 3 层：环境防护
**目的：**防止特定上下文中的危险操作

```typescript
async function gitInit(directory: string) {
  // In tests, refuse git init outside temp directories
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### 第 4 层：调试工具
**目的：**记录取证所需的上下文

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## 应用此模式

发现缺陷时：

1. **追踪数据流**——错误值从哪里产生？在哪里使用？
2. **绘制所有检查点**——列出数据经过的每个位置
3. **在每一层添加验证**——入口、业务、环境、调试
4. **测试每一层**——尝试绕过第 1 层，验证第 2 层能捕获它

## 会话示例

缺陷：空的 `projectDir` 导致源代码中执行 `git init`

**数据流：**
1. 测试设置 → 空字符串
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init` 在 `process.cwd()` 中运行

**添加的四层防护：**
- 第 1 层：`Project.create()` 验证非空、存在且可写
- 第 2 层：`WorkspaceManager` 验证 projectDir 非空
- 第 3 层：`WorktreeManager` 在测试中拒绝在 tmpdir 之外执行 git init
- 第 4 层：执行 git init 前记录堆栈跟踪日志

**结果：**所有 1847 个测试均通过，无法再复现该缺陷

## 关键洞察

四层防护都是必要的。在测试期间，每一层都捕获了其他层遗漏的缺陷：
- 不同的代码路径绕过了入口验证
- mock 绕过了业务逻辑检查
- 不同平台上的边界情况需要环境防护
- 调试日志识别出了结构性误用

**不要止步于一个验证点。**在数据经过的每一层都添加检查。
