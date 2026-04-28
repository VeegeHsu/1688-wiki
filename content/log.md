# 操作日志

---

## [2026-04-28] ingest | 导入数据库查询入口和traceId留痕规范

**来源**：`.claude/projects/memory/project_database.md`、`feedback_traceid_collection.md`

**新增 wiki/ 页面**（2个）：

| 页面 | 类型 | 说明 |
|------|------|------|
| `企采/tech-数据查询入口.md` | tech | imall_operation 数据库，企采全业务查询入口 |
| `通用/test-traceId留痕规范.md` | test（通用层） | traceId收集优先级、格式规范、写入位置 |

**更新**：`wiki/index.md`（技术知识表 + 通用层各增一行）

---

## [2026-04-28] ingest | 整合渠道对接业务知识

**来源**：`.claude/projects/memory/project_qycg_channels.md`

**新增 wiki/ 页面**（1个）：

| 页面 | 类型 | 说明 |
|------|------|------|
| `biz-渠道对接模式.md` | biz | 四渠道概览、两大对接模式（API/Agent）、核心表、channel_code 区分 |

**更新 wiki/ 页面**（1个）：
- `biz-渠道商品体系.md`：补充渠道商品价格链路 DB JOIN 详情、价格字段说明、已知坑

**更新**：`wiki/index.md`（业务知识表新增一行）

---

## [2026-04-28] ingest | 补充履约单查询接口文档

**来源**：`.claude/skills/hsf-generic-caller/interfaces/fulfillment/OrderFacade/interface-spec.md`

**新增 wiki/ 页面**（1个）：

| 页面 | 类型 | 说明 |
|------|------|------|
| `tech-履约单查询接口.md` | tech | OrderFacade.getOrderById HSF接口规格，含入参、出参、状态码、Long精度坑 |

**更新**：`wiki/index.md`（技术知识表新增一行）

---

## [2026-04-27] init | 知识库初始化

- 创建知识库：1688通用测试知识库
- 主题：1688 业务知识、技术知识、测试知识
- 初始化目录结构：`raw/`、`raw/assets/`、`wiki/`
- 创建 `wiki/index.md`（空索引模板）
- 创建 `wiki/log.md`（操作日志）
- 创建 `CLAUDE.md`（Schema 配置）

---

## [2026-04-28] restructure | wiki 多业务域目录重构

**变更**：从平铺结构迁移为子目录分域结构，支持多个一级业务线并行维护。

**目录变化**：
- 新增子目录：`wiki/通用/`、`wiki/企采/`、`wiki/跨境/`、`wiki/物流/`、`wiki/分销/`
- `test-用例设计方法论.md` → `wiki/通用/`（通用层）
- 所有 `biz-*`、`test-*`（渠道）、`tech-*` → `wiki/企采/`（企采业务层）
- `wiki/index.md` 更新为按业务域组织的多级索引
- `CLAUDE.md` 更新目录结构说明与页面分类规范

---

## [2026-04-27] ingest | 从 Obsidian 主 Vault 导入企采测试经验

**来源**：`~/Documents/Obsidian Vault/`（企业采购-业务知识库、1.Inbox）

**新增 raw/ 源文件**（8个）：
- `企采发票-状态逻辑总结.md`
- `企采结算-测试工具&逻辑总结.md`
- `企采结算对账.md`
- `企业采购资金要素逻辑.md`
- `企业采购监控治理.md`
- `qycg标签与类目.md`
- `测试用例生成经验.md`
- `订单水平标校验方法总结.md`

**新增 wiki/ 页面**（9个）：

| 页面 | 类型 | 说明 |
|------|------|------|
| `test-用例设计方法论.md` | test（通用层） | 10条可跨业务复用的用例设计经验 |
| `test-天猫.md` | test | 天猫渠道完整测试指南 |
| `test-飞猪Agent.md` | test | 飞猪Agent完整测试指南 |
| `test-钉钉Agent.md` | test | 钉钉Agent完整测试指南 |
| `test-浙建采.md` | test | 浙建采渠道完整测试指南 |
| `biz-发票.md` | biz | 发票状态机与类型枚举 |
| `biz-结算.md` | biz | 结算业务逻辑与测试工具 |
| `biz-资金要素.md` | biz | 飞猪资金表结构与金额一致性规则 |
| `tech-监控治理.md` | tech | 接口监控告警记录 |

**更新**：`wiki/index.md`（从空模板填充为完整索引）
