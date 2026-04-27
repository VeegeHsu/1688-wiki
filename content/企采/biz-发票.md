---
tags: [biz, 发票, 状态机, 企业采购]
type: biz
date: 2026-04-27
source: "raw/企采发票-状态逻辑总结.md"
---

# 企采发票业务逻辑

## 发票状态流转

| 前端显示状态 | DB 状态（apply_record） | DB 状态（invoice_info） | 可操作点 |
|------------|----------------------|----------------------|---------|
| 未开票 | — | — | 开票 |
| 开票中 | PENDING | — | 撤销 |
| 已开票 | EFFECT | EFFECT | 发票下载、退票 |
| 退票中 | RETURNING | EFFECT | 发票下载 |
| 退票驳回 | RETURN_REJECT | EFFECT | 发票下载 |
| 已退票 | RETURN_FIN | RED_RUSH | 红票下载、重新开票 |
| 已撤销（开票中→撤销） | WITHDRAW | PENDING | 重新开票 |
| 已撤销（退票中→撤销） | WITHDRAW | EFFECT | 发票下载、退票 |
| 开票异常 | REJECT | REJECT | 无操作点 |

## 状态枚举值

### 发票申请状态（qycg_mall_invoice_apply_record）

```java
PENDING("待决状态"),
EFFECT("生效状态"),
REJECT("回绝状态"),
ERROR("错误状态"),
RETURNING("退票中"),
WITHDRAW("已撤销"),
RETURN_FIN("已退票"),
RETURN_REJECT("退票驳回");
```

### 发票信息状态（qycg_mall_invoice_info）

```java
PENDING("PENDING", "待决状态"),
EFFECT("EFFECT", "生效状态"),
REJECT("REJECT", "回绝状态"),
RED_RUSH("RED_RUSH", "红冲状态");
```

## 发票类型枚举

| 类型码 | 名称 | byte值 |
|-------|------|-------|
| TAX | 营业税 | 0 |
| VATAX_COMM | 增值税普通发票 | 1 |
| VATAX_SPEC | 增值税专用发票 | 2 |
| PROFORMA | 形式发票 | 3 |
| RECEIPT | 收款凭据 | 4 |
| FD_EINVOICE_VAT_COMM | 数字化电子普票 | 7 |
| FD_EINVOICE_VAT_SPEC | 数字化电子专票 | 8 |

## 核心术语

| 术语 | 定义 |
|------|------|
| **销项发票** | 你开给买家的发票（你是销售方） |
| **进项发票** | 供应商开给你的发票（你是购买方） |

## 关联

- [[biz-渠道履约]]
- [[biz-结算]]
- [[test-天猫]]
- [[test-钉钉Agent]]
