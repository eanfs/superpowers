# 基于条件的等待

## 概述

不稳定的测试经常用任意延迟来猜测时机。这会造成竞态条件：测试在速度较快的机器上通过，但在负载较高时或 CI 中失败。

**核心原则：**等待你真正关心的实际条件，而不是猜测它需要多长时间。

## 何时使用

```dot
digraph when_to_use {
    "测试使用 setTimeout/sleep？" [shape=diamond];
    "是否在测试时序行为？" [shape=diamond];
    "记录需要超时的原因" [shape=box];
    "使用基于条件的等待" [shape=box];

    "测试使用 setTimeout/sleep？" -> "是否在测试时序行为？" [label="yes"];
    "是否在测试时序行为？" -> "记录需要超时的原因" [label="yes"];
    "是否在测试时序行为？" -> "使用基于条件的等待" [label="no"];
}
```

**适用场景：**
- 测试包含任意延迟（`setTimeout`、`sleep`、`time.sleep()`）
- 测试不稳定（有时通过，在负载较高时失败）
- 并行运行时测试超时
- 等待异步操作完成

**不适用场景：**
- 测试实际的时序行为（防抖、节流间隔）
- 使用任意超时时，始终记录原因

## 核心模式

```typescript
// ❌ BEFORE: Guessing at timing
await new Promise(r => setTimeout(r, 50));
const result = getResult();
expect(result).toBeDefined();

// ✅ AFTER: Waiting for condition
await waitFor(() => getResult() !== undefined);
const result = getResult();
expect(result).toBeDefined();
```

## 快速模式

| 场景 | 模式 |
|----------|---------|
| 等待事件 | `waitFor(() => events.find(e => e.type === 'DONE'))` |
| 等待状态 | `waitFor(() => machine.state === 'ready')` |
| 等待数量 | `waitFor(() => items.length >= 5)` |
| 等待文件 | `waitFor(() => fs.existsSync(path))` |
| 复杂条件 | `waitFor(() => obj.ready && obj.value > 10)` |

## 实现

通用轮询函数：
```typescript
async function waitFor<T>(
  condition: () => T | undefined | null | false,
  description: string,
  timeoutMs = 5000
): Promise<T> {
  const startTime = Date.now();

  while (true) {
    const result = condition();
    if (result) return result;

    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`Timeout waiting for ${description} after ${timeoutMs}ms`);
    }

    await new Promise(r => setTimeout(r, 10)); // Poll every 10ms
  }
}
```

参见本目录中的 `condition-based-waiting-example.ts`，其中包含实际调试会话中的完整实现，以及特定领域的辅助函数（`waitForEvent`、`waitForEventCount`、`waitForEventMatch`）。

## 常见错误

**❌ 轮询过快：**`setTimeout(check, 1)`——浪费 CPU
**✅ 修复：**每 10ms 轮询一次

**❌ 没有超时：**如果条件永远无法满足，循环会永远运行
**✅ 修复：**始终设置超时，并提供清晰的错误信息

**❌ 数据过时：**在循环之前缓存状态
**✅ 修复：**在循环内调用 getter，以获取最新数据

## 任意超时何时是正确的

```typescript
// Tool ticks every 100ms - need 2 ticks to verify partial output
await waitForEvent(manager, 'TOOL_STARTED'); // First: wait for condition
await new Promise(r => setTimeout(r, 200));   // Then: wait for timed behavior
// 200ms = 2 ticks at 100ms intervals - documented and justified
```

**要求：**
1. 首先等待触发条件
2. 基于已知时序（而不是猜测）
3. 添加注释解释原因

## 真实影响

来自调试会话（2025-10-03）：
- 修复了 3 个文件中的 15 个不稳定测试
- 通过率：60% → 100%
- 执行时间：快 40%
- 不再有竞态条件
