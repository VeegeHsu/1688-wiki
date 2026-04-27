---
tags: [tech, 天猫, 隐私面单, HSF, 校验脚本, 企业采购]
type: tech
date: 2026-04-28
source: "Obsidian Vault/1.Inbox/订单水平标校验方法总结.md"
---

# 订单水平标（_F_b_pry_o）校验方法

## 用途

校验天猫渠道履约单是否已打上**隐私面单标**（`_F_b_pry_o`）。

| 字段值 | 含义 |
|--------|------|
| `_F_b_pry_o = 1` | 已打隐私面单标（校验通过） |
| `_F_b_pry_o = 0` | 未打隐私面单标（校验失败） |
| 字段缺失 | 校验失败 |

## 适用范围

- 渠道：天猫（`channel_code = tmall`）
- 脚本分组：工业品
- 输入：履约单号（`fulfill_order_code`）

## 校验流程

```
输入 fulfill_order_code
  ↓
调用 OrderService.queryOrder 查询履约单详情（HSF）
  ↓
从返回结果中提取 attributes._F_b_pry_o 字段
  ↓
校验字段值是否为 1
  ↓
返回校验结果（空字符串=通过）
```

## HSF 接口信息

| 属性 | 值 |
|------|-----|
| 服务名 | `com.alibaba.shared.trade.service.OrderService` |
| 版本号 | `1.0.0` |
| 分组 | `DUBBO` |
| 方法名 | `queryOrder` |
| 参数类型 | `com.alibaba.shared.trade.param.order.QuerySingleOrderParam` |

**关键参数**：
```json
{
  "id": "<履约单号（Long）>",
  "needOrderEntries": true
}
```

**字段路径**：返回结果的 `attributes._F_b_pry_o`

## 输入参数格式

```json
{
  "sharding_key": "T20260220000010208272",
  "order_status": "buyerconfirm",
  "fulfill_order_code": "5106159073038424145",
  "out_biz_id": "5076697899416035816",
  "id": 13220747,
  "channel_code": "tmall"
}
```

## 校验结果说明

| 返回值 | 含义 |
|--------|------|
| `""`（空字符串） | 校验通过 |
| `"履约单号为空"` | 缺少 fulfill_order_code |
| `"履约单：xxx 查询结果为空"` | HSF 查询返回空 |
| `"履约单：xxx 缺少 _F_b_pry_o字段"` | attributes 中无该字段 |
| `"履约单：xxx _F_b_pry_o字段为0"` | 未打隐私面单标 |

## 关联

- [[biz-天猫通品]]
- [[test-天猫]]
