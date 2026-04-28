---
tags: [test, 测试规范, 通用, playwright]
type: test
date: 2026-04-28
source: ".claude/projects/memory/feedback_screenshot_location.md"
---

# 测试截图存放规范

测试执行时截图统一按以下规则存放：

| 场景 | 存放路径 |
|------|---------|
| 有测试计划.md 时 | 与测试计划.md **同目录**下的 `screenshots/` 子文件夹 |
| 无测试计划.md（临时需求） | `/Users/linxiaobei/Documents/Obsidian Vault/assets/` |

> 保持截图与测试计划/报告的相对路径关系，方便 Obsidian 直接预览；临时截图统一收口到 assets 避免散落。

## 操作原则

每次调用 `playwright-cli` 截图前，先判断当前是否有测试计划.md，根据结果选择对应目录，不得随意存到其他位置。

## 关联

- [[test-traceId留痕规范]]
- [[test-用例工作流规范]]
