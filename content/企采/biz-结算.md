---
tags: [biz, 结算, 测试工具, 企业采购]
type: biz
date: 2026-04-27
source: "raw/企采结算-测试工具&逻辑总结.md"
---

# 企采结算业务逻辑与测试工具

## 业务核心逻辑

交易侧按时间周期聚合订单明细数据，为一个账单：

**一笔结算单 → n笔账单 → 一笔账单可有多笔订单（主单维度的明细）**

## 测试工具

### 测试入口

| 环境 | 地址 |
|------|------|
| 万彩平台线上 | https://mychain.1688.com/#/statements |
| 预发 | https://pre-mychain.1688.com/#/statements?__mtop_subdomain__=wapa |
| 付款预发影子页面 | https://pre-shadow-financial-portal.caijing.alibaba-inc.com/smartpay/kp-invoice |

### 测试账号（大店：b工业品测试账号010）

- 买家：淘大店测试买家002、分销大b测试001（托管通企采方式）
- 供应商：b工业品测试账号tp001
  - 测试商品（无运费）：881234885006；（带运费）：891024984555

### 测试工具链接

| 工具 | 链接 |
|------|------|
| 聚宝数据构造（动态门槛红包） | https://1688dacu.alibaba-inc.com/index#dataSceneListManage?sceneId=2024 |
| 聚宝发货工具 | https://1688dacu.alibaba-inc.com/index#/dataSceneListManage?sceneId=1143 |
| 生成银行卡号 | https://ddu1222.github.io/bankcard-validator/bcBuilder.html |

## 预发测试配置

### 1. 修改预发订单售后期结束时间

- diamond配置入口：https://mse.alibaba-inc.com/pre/diamond/configlist/configedit?DataId=com.alibaba.cbu.shellcenter.config
- 配置项：`afterSaleExpiredDay`

### 2. 预发交易侧出账单时间周期

- 配置入口：https://ctf.alibaba-inc.com/world-1688/ABkkl3brqp
- 汇总周期：30min/1小时（按小时产出）

### 3. 将预发下查询的 cycleType 改成4

- 配置入口：https://mse.alibaba-inc.com/pre/diamond/configlist/configdetail?dataId=com.alibaba.china.scm.account.bill.config

### 正式链路-新履约单出账配置

- `settledays`：s，表示+xx分钟（测试写300s=5min）
- `upsertAccountPeriodMs`：ms，表示出账时间gap（测试写600000ms=10min）

## 智付影子测试链路

1. 结算单处于待结算状态，需用其他平台（京东）开出的真实数电专票
2. 去数据库修改结算单金额与发票金额保持一致
3. 发票中心验证成功 → 结算单状态变为**结算中**
4. 进入智付影子链路（https://pre-shadow-financial-portal.caijing.alibaba-inc.com/smartpay/kp-invoice）
5. 在"待匹配发票"中，筛选1688企采发票，根据发票号找对应上传发票，点击人工处理
6. 匹配确认 → 进入付款链路 → 关联单据号搜索 → 核对金额 → 点击"付款"→ "线下支付"

## 数据库表

```sql
-- 站内超链链路供应商侧订单
cbu_wireless_quality.s_tc_biz_order_cn_for_settle（供货价：f_sum_op）

-- 渠道链路供应商侧订单
cbuods.s_tc_biz_order_cn

-- 交易账单表
tradedev1688.s_settle_write_off_bill_cbu_trade_unit_app

-- 企业采购侧结算单数据
s_cbu_trade_qycg_commission_info_seller
```

```sql
-- 账单维度查询
SELECT *
FROM tradedev1688.s_settle_write_off_bill_cbu_trade_unit_app
WHERE ds = MAX_PT('tradedev1688.s_settle_write_off_bill_cbu_trade_unit_app')
AND settle_scene_code = 'QYCG_MALL_SETTLEMENT'
AND id = {账单ID};

-- 明细维度（根据账单号查询所有明细订单）
SELECT *
FROM wxbdw.s_cbu_trade_misc_app_commission_info_seller
WHERE ds = MAX_PT('wxbdw.s_cbu_trade_misc_app_commission_info_seller')
AND biz_type = 'QYCG_MALL_SETTLEMENT'
AND fee_code = 'supplier_goods_fee'
AND write_off_bill_id = {账单ID};
```

## 关联

- [[biz-发票]]
- [[biz-渠道履约]]
- [[biz-天猫通品]]
