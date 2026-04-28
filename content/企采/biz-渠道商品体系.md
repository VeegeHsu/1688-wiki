---
tags: [biz, 渠道商品, 超链商品, 通品]
type: biz
date: 2026-04-27
---

# 渠道商品体系

## 三层映射结构

```
供应商商品
    ↓ 铺品
托管供货中心产品（超链产品）
    ↓ 转化
企采超链商品（面向买家）
```

关键清退逻辑：
- 原链下架 → 超链下架
- 原链 SKU 下架 → 超链对应 SKU 下架
- 超链清退：只解除超链与供货中心产品的关系，**供应商商品与产品的关系不解除**
- 企采超链商品**不支持无 SKU 商品**

## 通品铺品模式

| 模式 | 说明 | 典型渠道 |
|------|------|----------|
| SKU 模式 | 渠道 SKU ↔ 企采 SKU（1:1 映射） | 浙建采 |
| SPU 模式 | 一个 SPU 对应多个 SKU | 商越 |

- 支持**自动类目映射**（关键词匹配）和**手动类目映射**
- 类目映射失败时需人工处理

## 变更感知（MetaQ）

| 来源 | 监听方式 |
|------|----------|
| 内部系统 | 监听**领域事件** |
| 外部渠道 | 监听 **MetaQ 消息变更** |

## 下架类型与规则

| 类型 | 触发方 | 说明 |
|------|--------|------|
| 渠道商品下架 | 渠道侧 | 渠道采购员可上架 |
| 超链主动下架 | 运营 | 若品有在通渠道且状态正常，**不允许操作** |
| 超链被动下架 | 系统 | 底层问题或改价时先下再上 |

关键约束：通品后运营操作渠道品下架 → 无法再由运营在渠道平台上架，只能由**渠道采购员**操作上架；超链下架时，所有已通渠道同步下架。

## 商品不可售判断

以下情况商品置灰，不可加入采购单：
- 商品已下架
- 地区限售 / 买家限售
- 不支持在线交易

> 已在采购草稿中的商品若变成不可售，预览下单时提示用户。同钉钉Agent的商品不可售逻辑一致。

## 渠道商品价格链路（DB JOIN）

**完整路径**：`channel_offer` → `channel_offer_item` → `product_item` → `supply_product_item`

**关键字段映射**（三表同值，是串联关键）：

| JOIN 条件 | 说明 |
|-----------|------|
| `order_detail.item_id` = `channel_offer.qc_product_id` | 从订单找渠道商品 |
| `order_detail.sku_id` = `channel_offer_item.qc_product_sku_id` = `product_item.id` | SKU 串联 |
| `order_detail.out_sku_id` = `channel_offer_item.channel_sku_id` | 天猫侧 SKU ID |
| `product_item.supply_sku_id` = `supply_product_item.supply_sku_id` | 供货价入口 |

**价格字段**：

| 字段 | 含义 |
|------|------|
| `channel_offer_item.channel_price` | 渠道售价（分）= `order_detail.sale_price` |
| `supply_product_item.supply_price` | 含税供货价（分，不含运） |
| `order_detail.purchase_price` | 含税含运费供货价（分）= 供应商结算价 |

**天猫定价规则**：天猫渠道默认包邮，运营定价时将运费计入 `channel_price`（买家 `out_post_fee=0`）。  
平均运费无独立字段，从订单反推：`purchase_price − supply_price`。

**已知坑**：
- `channel_offer.offer_id` 在测试商品中为 **NULL**，不能用天猫商品 ID 直接匹配。
- 预发 `channel_code` 是 `tmall_test`，不是 `tmall`；用 `tmall` 查永远 0 行。

---

## 铺品接口

| 接口 | 说明 |
|------|------|
| `com.alibaba.china.imall.cg.api.service.QycgMallChannelSyncSpiService#syncCreateChannelOffer:1.0.0` | 渠道商品铺品（创建渠道 Offer） |

## 来源

- 原始资料：`raw/渠道商品-业务逻辑.md`

## 关联

- [[biz-渠道履约]]
- [[biz-天猫通品]]
- [[biz-浙建采]]
- [[biz-支付宝积分商城]]
- [[biz-钉钉Agent]]
