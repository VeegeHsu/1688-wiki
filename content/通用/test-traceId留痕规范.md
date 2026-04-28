---
tags: [test, traceId, 测试规范, 通用]
type: test
date: 2026-04-28
source: ".claude/projects/memory/feedback_traceid_collection.md"
---

# 测试执行 traceId 留痕规范

## 核心原则

测试执行中凡能拿到 traceId 的，**必须收集并记录到报告的 Case 信息表格，禁止留空**。

> traceId 是线上/预发问题排查的核心留痕手段，后续查验 RPC 链路时依赖它。

## 来源优先级

| 优先级 | 来源 | 说明 |
|--------|------|------|
| 1 | HSF 接口返回值中的 `traceId` 字段 | 最直接，优先使用 |
| 2 | SLS 日志检索提取 | 接口无返回时，从日志中找 |
| 3 | 无法获取时写明检索路径 | 禁止直接留空 |

## 无法获取时的格式

```
待补充（可在 {logstore名} 以 {关键词} 检索）
```

## 写入位置

每个 Case 信息表格中单独一行：

```markdown
| **Trace ID** | ${值} |
```

## 适用范围

已固化到 test-executor（Step 3 收集规则）和 test-report-generator（Case 模板信息表格），自动化测试流程中自动执行。

## 关联

- [[test-用例设计方法论]]
