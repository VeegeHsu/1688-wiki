---
tags: [test, 飞猪, Agent, 测试指南]
type: test
date: 2026-04-27
source: "raw/Readme-飞猪Agent测试.md"
---

# 飞猪 Agent 测试指南

## 账号信息

### 1688 测试账号

| 角色 | 账号 |
|------|------|
| 测试大店（已开通对公支付） | b工业品测试账号010（userId：2214243990343） |
| 常用买家 | b交易自动化测试001、b交易内部测试账号001、淘大店测试卖家002 |

账号借用：https://tdbank.alibaba-inc.com/borrowAccount.html  
密码格式：`域账号|域密码`（| 是竖线符号，非字母l）

### 飞猪账号

| 账号类型 | 值 |
|---------|-----|
| ebk账号 | 2219664915475（涉月ebk测试） |
| hid | 593854001554（盛苏测试植物园酒店） |

**登录逻辑**：飞猪ebk账户 → 选择酒店门店hid → 手机号登陆 → 选择1688账号

## 快速工具

| 工具 | 链接 |
|------|------|
| 快速下单+支付+发货+确收 | https://mary.alibaba-inc.com/case/detail?id=46220 |
| 快捷支付（仅预发） | https://mary.alibaba-inc.com/template/detail?id=799 |
| 发送开票成功消息 | https://mary.alibaba-inc.com/template/detail?id=932 |
| 订单确收 | https://mary.alibaba-inc.com/template/detail?id=435 |
| 订单发货 | https://mary.alibaba-inc.com/case/detail?id=42472 |
| 申请退款 | https://mary.alibaba-inc.com/case/detail?id=41217 |
| 同意退款 | https://mary.alibaba-inc.com/case/detail?id=41386 |

### HSF 工具（预发）

| 操作 | 接口 | 方法 |
|------|------|------|
| agent确收 | `com.alibaba.china.imall.scm.integration.ops.OpsService:1.0.0` | `confirmReceiveAgentOrder~java.lang.String` |
| 退款完结消息转发 | 同上 | orderId、refundNo、refundFee（分）、refundType（1售中 2售后） |
| 取消订单消息转发 | 同上 | `cancelAgentOrder~java.lang.String` |

## 测试品数据（b工业品测试账号010）

| 类型 | 超链商品ID | 备注 |
|------|-----------|------|
| 固定运费0.01 | 1017896411067 | 企采—下单—挂件 |
| 固定运费0.01 | 1032573143471 | 企采agent—太阳能等 |
| 阶梯运费0.02 | 1017721845149 | 企采—开关 |
| 大额包邮 | 998413180177 | 企采TM二期 |
| xs001包邮 | 1001614414638 | 企采TM二期 |

## 测试步骤

1. 下载飞猪开发包（iOS/Android，摩天轮平台）
2. 申请 ebk 账号（tdbank），绑定手机号到测试账号
3. 登陆飞猪ebk账号，在应用内扫码登陆1688
4. 商城测试品配置：diamond配置 `channel_config`，group=`imall-hub-gateway`
5. 发起采购 → 审批（可选本人）→ 付款（agent内/1688端/快捷支付工具）
6. 发货（工具 mary-42472 选预发环境）
7. 确认收货（agent内/1688端/工具435）
8. 开票：agent内勾选订单点击开票（自营：联系@秉锐/@徐湘 操作自营预发开票；mock可用工具932）
9. 报销：所有明细开票完成后勾选去报销

## 日志查询

- 订单支付成功消息 topic：`qycg_agent_msg`

## 开票 host（预发）

```
140.205.215.168 lava.alibaba-inc.com
```

## 关联

- [[biz-飞猪Agent]]
- [[biz-渠道履约]]
- [[biz-资金要素]]
- [[test-钉钉Agent]]
