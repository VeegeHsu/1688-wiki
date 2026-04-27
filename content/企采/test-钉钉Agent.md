---
tags: [test, 钉钉, Agent, 测试指南]
type: test
date: 2026-04-27
source: "raw/Readme-企采钉钉Agent测试.md"
---

# 企采&钉钉 Agent 测试指南

## 账号准备

免登实现逻辑：**钉钉账户 → 手机号 → 1688账号**

1. 申请测试账号权限，找测试账号owner绑定你本人手机号
2. 找涉月和中台@李晖(原昭)，绑定钉钉与借用的测试账号

### 可用测试账号

| 角色 | 账号 |
|------|------|
| 测试自营大店 | b工业品测试账号010（预发已配置） |
| 买家 | b交易内部测试账号001、b供威亚测试分销008、淘大店测试卖家002、b交易内部测试账号003 |

账号借用：https://tdbank.alibaba-inc.com/borrowAccount.html

## 快速工具

| 工具 | 说明 |
|------|------|
| 下单+支付+发货+确收 | mary案例46220 |
| 快捷支付（仅预发） | mary模板799 |
| 发送开票成功消息 | mary模板932（支持线上&预发） |
| 确认收货 | mary模板435（支持线上&预发） |
| 订单发货 | mary案例42472 |
| 申请退款 | mary案例41217 |
| 同意退款 | mary案例41386 |

## Agent 相关链接

| 环境 | 链接 |
|------|------|
| 线上agent | https://deap-agent.dingtalk.com/h5/index.html?publish_h5=1&code=e41b89bc-b533-4d00-8189-ce6e4855c7cd |
| 加入组织 | https://h5.dingtalk.com/invite-page/index.html?corpId=dingcbe374d160c2abbda1320dcb25e91351&inviteCode=o8Lu0XbH74TPkN2 |

## 测试步骤（预发）

### Step1：配置 host

```
# 基础host
140.205.215.168 login.taobao.com
140.205.215.168 member.1688.com
140.205.215.168 passport.1688.com
140.205.215.168 passport.taobao.com
# 金融host
140.205.215.168 pre2-jinrong.1688.com
140.205.215.168 rongzi.1688.com
# 开票host
140.205.215.168 lava.alibaba-inc.com
```

### Step2：免登进入预发 agent

1. 打开阿里钉预发环境（mac钉调试插件）
2. 免登（@sheyue 提供链接）
3. 进入预发agent链接（@songcheng 提供链接）

### Step3：配置测试商品（可选）

diamond配置：`offer_sku_config`，group=`imall-hub-gateway`

### Step4：完整链路

1. 审批节点选本人
2. 付款（agent内/1688端/快捷支付工具799）
3. 发货（mary案例42472，选预发环境）
4. 确认收货（agent内/1688端/工具435）
5. 开票：
   - 严选：登陆供应商后台发票中心录入
   - 自营：找@秉锐/@徐湘 操作自营预发开票
   - mock：mary模板932 发送开票成功消息
6. 报销（所有明细开票完成后勾选）

## 测试品数据

### 黑标商品

| 商家 | 商品 | skuId |
|------|------|-------|
| b工业品测试账号tp001 | 黑标测试商品（836337574038） | 5716102672677 |

### 自营商品（b工业品010大店）

| 商家 | 超链商品 | skuId | 备注 |
|------|---------|-------|------|
| xs001 | 企采TM二期xs001—测试品008（1001614414638，包邮） | 6156379962617 | 供货品：976636232777 |
| tp001 | 企采—下单—挂件—固定运费（1017896411067） | 6194396035008 | 固定运费0.01，6折 |

## 手机号绑定测试账号

https://tdbank.alibaba-inc.com/myAccount.html

## 关联

- [[biz-钉钉Agent]]
- [[biz-渠道履约]]
- [[test-飞猪Agent]]
