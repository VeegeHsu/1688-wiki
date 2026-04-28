---
tags: [tech, 数据库, 企业采购]
type: tech
date: 2026-04-28
source: ".claude/projects/memory/project_database.md"
---

# 企采数据查询入口

## 核心数据库

| 项目 | 数据库名 |
|------|---------|
| imall-web | `imall_operation` |

所有企采业务数据查询均在此库中，包括渠道订单、履约单、企采商城、天猫订单等。

## 使用方式

通过 `qycg-data-query` skill 查询，或直接对 `imall_operation` 执行 SQL，无需另建数据查询工具。

## 关联

- [[biz-渠道履约]]
- [[biz-渠道商品体系]]
- [[tech-履约单查询接口]]
