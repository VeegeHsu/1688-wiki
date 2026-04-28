---
tags: [tech, HSF, 履约单, 企业采购]
type: tech
date: 2026-04-28
source: ".claude/skills/hsf-generic-caller/interfaces/fulfillment/OrderFacade/interface-spec.md"
---

# 履约单查询接口（OrderFacade）

## 基本信息

| 字段 | 值 |
|------|----|
| 服务名 | `com.alibaba.china.imall.scm.domain.facade.order.OrderFacade` |
| 版本号 | `1.0.0` |
| 分组 | `""`（空字符串） |
| 方法名 | `getOrderById` |

> ⚠️ `methodName` 只填 `getOrderById`，不能加 `~L` 或 `~java.lang.Long` 后缀，否则无返回。

## 入参

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| orderId | `java.lang.Long` | ✅ | 1688 履约单订单 ID |

**调用示例**：
```json
paramTypes: ["java.lang.Long"]
paramObjs:  ["5111564077001028736"]
```

> ⚠️ **Long 精度坑**：orderId 超过 JS 安全整数上限（2^53），必须用**字符串**传递，数字形式会精度丢失。

## 出参关键字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `orderId` | Long | 订单 ID |
| `orderStatus` | String | 订单状态（见状态码表） |
| `loginId` | String | 买家账号 |
| `createTime` | String | 创建时间（ISO 8601） |
| `gmtCompleted` | String | 完成时间，未完成时为 null |
| `logisticsStatus` | Integer | 物流状态 |
| `paidSumPayment` | Integer | 实付总金额（分） |
| `sumPayment` | Integer | 应付总金额（分） |
| `sumProductPayment` | Integer | 商品总金额（分） |
| `divisionCode` | String | 收货地区码 |
| `orderEntryInfoModelList` | Array | 子单列表 |

### 子单字段（orderEntryInfoModelList 元素）

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | Long | 子单 ID |
| `offerId` | Long | 1688 商品 offer ID |
| `productName` | String | 商品名称 |
| `quantity` | Integer | 购买数量 |
| `amount` | Integer | 子单总金额（分） |
| `discountPrice` | Integer | 折扣后单价（分） |
| `payStatus` | Integer | 支付状态 |
| `entryStatusStr` | String | 子单状态（见状态码表） |
| `logisticsStatus` | Integer | 物流状态 |
| `postFee` | Integer | 运费（分） |
| `refundFee` | Integer | 退款金额（分） |
| `refundId` | Long/null | 退款单 ID |

## 订单状态码

| `orderStatus` | 含义 |
|---------------|------|
| `WAIT_SELLER_SEND` | 待卖家发货 |
| `WAIT_BUYER_CONFIRM` | 待买家确认收货 |
| `SUCCESS` | 交易成功 |
| `CLOSED` | 交易关闭 |

## 调用成功示例

```json
{
  "orderId": 5111564077001028736,
  "orderStatus": "WAIT_SELLER_SEND",
  "loginId": "b工业品测试账号010",
  "createTime": "2026-04-23T17:05:22+08:00",
  "logisticsStatus": 1,
  "paidSumPayment": 4,
  "sumPayment": 4,
  "orderEntryInfoModelList": [
    {
      "id": 5111564077001028736,
      "offerId": 973919095109,
      "productName": "企采TM二期tp001—测试品003",
      "quantity": 2,
      "amount": 4,
      "entryStatusStr": "waitsellersend"
    }
  ]
}
```

## 关联

- [[tech-订单水平标校验]]
- [[biz-渠道履约]]
- [[test-天猫]]
