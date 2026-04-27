---
tags: [biz, 支付宝积分商城, 锁价, 9折专享价]
type: biz
date: 2026-04-27
---

# 支付宝积分商城业务逻辑

## 背景

支付宝积分商城原有分销严选、找工厂等品，需逐步切换为企业采购商品。企采品在商城售卖需要**锁价**（价格与支付宝谈好）。

切流路径：`分销品 → 9折专享价活动 → 天天特卖品切9折活动`

## 品来源与接入方式

| 来源 | 活动 id | 接入方式 |
|------|---------|----------|
| 分销严选 | — | 工作台搜分销严选直接加入，不需审核 |
| 企采天天特卖 | `603830770153` | 活动报名 |
| 企采9折专享价 | `614564970153` | 活动报名（锁价） |

## 关键业务数据

| 字段 | 值 |
|------|-----|
| 9折专享价活动 id | `614564970153` |
| 商品公私域隐藏维表 | `cbu_industry.qycg_item_hide` |
| 公域隐藏标 | `330690` |

## HSF 接口（预发）

| 接口 | 所属应用 | 说明 |
|------|----------|------|
| `AlipayIntegralOfferService:1.0.0#getOffersByOfferIds` | kardel | 根据 offerId 批量查询商品信息 |
| `AlipayMallChannelService:1.0.0#queryQcOfferInfo` | imall-hub-gateway | 查询企采商品信息（面向支付宝侧） |
| `AlipayIntegralOfferService:1.0.0#isAddressAllow` | kardel | 判断收货地址是否在限售区域内 |
| `CreateOrderService:1.0.0#createOrder` | varyag | 创建订单 |

## 来源

- 原始资料：`raw/支付宝积分商城-业务逻辑.md`

## 关联

- [[biz-渠道商品体系]]
- [[biz-渠道履约]]
