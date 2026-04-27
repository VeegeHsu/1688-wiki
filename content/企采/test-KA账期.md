---
tags: [test, KA账期, 测试指南]
type: test
date: 2026-04-27
---

# KA账期测试指南（账期金融版）

## 造单方式

1. 接口下单
2. 浙建采渠道下单

## 测试准备

### 关键 MQ 消息（生意通）

| 消息类型 | Topic |
|---------|-------|
| 江苏银行账号消息 | `COUNTERPARTY_LINKED_ACCOUNT_OPEN_TOPIC` |
| 授信完结消息 | `RISK_AUDIT_NOTICE` |
| 来账消息 | `XPAYMENT_CENTER_FUNDS_TOPIC`（轻尘） |
| 投保消息（tag=INSURE） | `XPAYMENT_CONTRACT_INSURE_TOPIC` |

消费有问题时重新投递步骤：
1. 设置全局变量 `outOrderIndex` +1（如 03 改 04）
2. 修改"支付结果网关消息"-买家来账资金的 `amount`、`eventTime`
3. 修改"支付结果网关消息"-保险来账资金的 `amount`、`eventTime`

### 客户注册

1. loginId、社会信用代码调生意通注册（接口：`SytXPaymentAccountFacade~registerSyt`）→ 返回 88id（合作=停用，信用=未启用）
2. 监听客户江苏银行账号来账消息，回写江苏银行账号（合作=启用，信用=未启用）

### 流水归集

1. 执行流水认领任务，识别未认领流水
2. 将银行卡填写到客户档案中
3. 找轻尘 mock 来账信息

## 订单链路验证步骤

### （1）创建渠道订单后验证企采预占

### （2）接单前验证 syt 占用和采购单创建

登陆卖家账号，根据采购单号拼接 url：
```
https://pre-syt.1688.com/page/SYT/buyer-contract?draftNo={syt单号}&tracelog=header
```
反向验证：金融额度占用成功且采购单创建

### （3）确收验证预占转占用、同步 arm

### （4）确收验证保险订单明细生成、生意通投保

验证保险订单记录已写入，状态=等待投保：
```sql
SELECT * FROM qycg_mall_insurance_order 
WHERE relate_channel_sale_order_detail_code = '{子单号}';
```
关键接口：`SytFacadeImpl#confirmSytPurchaseOrder`

验证生意通投保成功：
- 日志：`SytInsuranceCreateListener consumeMessage message`
- 状态变为 init，外部保单号已记录

## 核销验证（未冻结核销）

### （1）明细金额全部核销完成

1. 验证 arm 已核销
2. 验证应收明细状态 == 已核销完成

### （2）执行生意通还款任务

任务：`syncRepayToSyt`
日志：`"syncRepayToSyt start, receivableBillItemNo"`
验证：应收明细 `repay_status == 成功`

## 逾期验证

### （1）Mock 逾期方式

- 方式一：账期天数改为 1 天，下单
- 方式二：找研发修改应收明细的 `overdue_date`

### （2）执行明细逾期检查任务

任务：supply-chain【ka账期】账单明细逾期时间检查
验证：
- 应收明细打逾期标签
- 明细维度逾期同步生意通
- 日志：`"syncOverdueStatusToSyt start，receivableBillItemNo: {}"`

### （3）执行客户维度逾期同步任务

任务：`CustomerInfoSyncScheduleProcessor`（imall-web）
日志：`"[CustomerInfoSyncScheduleProcessor.process]:start.id = {}"`
验证：客户企采侧冻结 + 生意通侧冻结 + 无法下单

### （4）执行理赔报案任务

任务：`AutoClaimApplyByOverdueTask`（supply-chain）
日志：`"AutoClaimApplyByOverdueTask process accountReceivableBillItemListForClaim: {}"`

## 逾期后核销（信用恢复）

1. 操作逾期订单明细全部核销 → 应收明细核销完成，`is_overdue == 0`，arm 同步核销
2. 执行生意通还款任务（同未冻结核销流程）
3. 执行客户维度逾期同步任务 → 验证企采侧恢复=启用，生意通侧解冻，可以下单

## 测试用例链接

测试用例：https://mytest.alibaba-inc.com/new/testPlan/testPlanDetail?testPlanId=15131

## 来源

- 原始资料：`raw/Readme-KA账期测试.md` + `raw/KA账期-业务逻辑.md`

## 关联

- [[biz-KA账期]]
- [[test-浙建采]]
