---
tags: [biz, 资金, 飞猪, ODPS, 数据表, 企业采购]
type: biz
date: 2026-04-27
source: "raw/企业采购资金要素逻辑.md"
---

# 企采飞猪资金要素逻辑

> 面向 AI 自动化测试，描述企采飞猪Agent业务中资金相关的表结构、字段含义、数据关联关系和验证规则。

## 支付链路

| 支付方式 | 链路描述 |
|---------|---------|
| 储蓄罐支付 | 飞猪储蓄币扣减 → 通知1688交易对公支付 → 通知飞猪对公打款 |
| 支付宝支付 | 直接唤起支付宝收银台完成支付 |

**关键关系**：一笔企采采购单对应 **2笔飞猪支付单**：
- `channel = 1`：现金/支付宝支付单
- `channel = 2`：储蓄罐支付单

**异常场景**：储蓄罐支付成功但未打款给1688时，channel=2 状态为**支付成功(1)**，channel=1 状态仍为**待支付(0)**。

## 数据表定义

### 企采方 — 采购单表

**表名**：`odps.cbu_op_platform.s_agent_purchase_order_imall_operation`

| 字段 | 含义 | 取值说明 |
|------|------|---------|
| `env` | 环境 | `staging`=预发，`prod`=线上 |
| `is_deleted` | 软删除 | `0`=有效，`1`=已删除 |
| `purchase_order_no` | 企采采购单号 | **主键** |
| `order_status` | 采购单状态 | 见下方枚举 |
| `channel_code` | 渠道码 | 飞猪固定为 `fliggy` |
| `total_price` | 货品总金额 | 单位：分 |
| `total_actual_post_fee` | 总运费 | 单位：分 |
| `total_actual_pay_amount` | 实际付款总金额 | 单位：分 |

**采购单状态枚举（order_status）**：

| 状态值 | 含义 |
|--------|------|
| `PENDING_PAYMENT` | 待付款 |
| `ORDERING` | 下单中（储蓄罐扣款成功后） |
| `PROCUREMENT_IN_PROGRESS` | 采购中（1688支付完成后） |
| `WAITING_INVOICE` | 待开票 |
| `INVOICING` | 开票中 |
| `INVOICE_FAILED` | 开票失败 |
| `COMPLETED` | 已完成（终态） |
| `CANCELLED` | 已取消（终态） |

### 企采方 — 采购明细表

**表名**：`odps.cbu_op_platform.s_agent_purchase_order_detail_imall_operation`

| 字段 | 含义 |
|------|------|
| `purchase_order_no` | 企采采购单号（外键） |
| `purchase_order_detail_no` | 企采采购明细单号 |
| `detail_status` | 采购明细状态 |
| `main_order_id` | 1688主订单ID |
| `actual_pay_amount` | 货品金额（分） |
| `main_order_post_fee` | 运费（分） |

### 飞猪方 — 支付单表

**表名**：`trip_ods.s_hotel_merchant_fund_pay_order_hfr_hour`

| 字段 | 含义 | 取值说明 |
|------|------|---------|
| `pay_order_no` | 支付单号 | **主键** |
| `out_order_no` | 企采采购单号（外键） | 关联 `purchase_order_no` |
| `channel` | 支付渠道 | `1`=现金/支付宝，`2`=储蓄罐 |
| `total_amount` | 需付金额 | |
| `paid_amount` | 实际支付金额 | |
| `refunded_amount` | 累计退款金额 | |
| `status` | 状态 | `0`=待支付，`1`=支付成功，`2`=失败，`3`=关闭 |

### 飞猪方 — 退款表

**表名**：`trip_ods.s_hotel_merchant_refund_order_hfr_hour`

| 字段 | 含义 | 取值说明 |
|------|------|---------|
| `refund_no` | 退款单号 | **主键** |
| `pay_order_no` | 关联支付单号（外键） | |
| `out_order_no` | 企采采购单号 | |
| `refund_amount` | 本次退款金额 | |
| `channel` | 退款渠道 | `1`=现金，`2`=储蓄罐 |
| `status` | 状态 | `0`=处理中，`1`=退款成功，`2`=失败，`3`=关闭 |

## 表关联关系

```
企采采购单 ─────────┬────── 企采采购明细（1:N，飞猪目前1:1）
(purchase_order_no) │          ├── main_order_id → 1688主订单
                    │
                    └────── 飞猪支付单（1:2，固定两笔）
                    │       (out_order_no = purchase_order_no)
                    │          ├── channel=1（现金/支付宝）→ 退款单（0~N笔）
                    │          └── channel=2（储蓄罐）→ 退款单（0~N笔）
```

**通用过滤条件**：
- 企采表：`env = 'prod' AND is_deleted = 0 AND channel_code = 'fliggy'`
- 飞猪表：`product_type = 1`

## 金额一致性规则

### 企采内部一致性

| 规则 | 等式 |
|------|------|
| 采购单自身 | `total_price + total_actual_post_fee = total_actual_pay_amount` |
| 明细汇总=采购单 | `SUM(明细.actual_pay_amount) + SUM(明细.main_order_post_fee) = 采购单.total_actual_pay_amount` |

### 异常判断

| 异常场景 | 判断条件 | 风险等级 |
|---------|---------|---------|
| 采购单取消但储蓄罐未退 | 采购单 `CANCELLED` + channel=2 `status=1` + 退款单不存在或失败 | **高** |
| 储蓄罐扣款成功未打款 | channel=2 `status=1` + channel=1 `status=0` | 高 |
| 采购单与明细金额不一致 | total_actual_pay_amount ≠ SUM(明细金额+运费) | 高 |
| 采购单无飞猪支付单 | 非PENDING_PAYMENT状态的采购单在飞猪侧找不到支付单 | 高 |

## 关联

- [[biz-飞猪Agent]]
- [[biz-渠道履约]]
- [[test-飞猪Agent]]
