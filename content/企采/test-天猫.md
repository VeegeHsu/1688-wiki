---
tags: [test, 天猫, 渠道测试, 测试指南]
type: test
date: 2026-04-27
source: "raw/Readme-天猫订单测试.md"
---

# 天猫渠道测试指南

## 测试账号

| 角色 | 账号 |
|------|------|
| 天猫买家 | b供威亚测试分销008 |
| 极速退款平台垫付买家 | b交易内部测试账号001 |
| 天猫大店 | 淘大店测试卖家002 |
| 1688测试供应商 | b工业品测试账号tp001、xs找工厂测试账号001 |

账号借用：https://tdbank.alibaba-inc.com/borrowAccount.html

### 凭据（预发）

| 角色 | 账号 | 密码 |
|------|------|------|
| 天猫买家 | b供威亚测试分销008 | zhuaifei.xx\|Hsu20020223gk$ |
| 内部工具账号 | zhuaifei.xx@alibaba-inc.com | Hsu20020223gk$ |
| 支付宝支付密码 | — | a7118609 |

## 测试品数据

**非白名单品**：天猫 974901526968，原链 975921645322

### b工业品测试账号tp001

| 原链品状态 | 原链ID | 天猫ID |
|---------|--------|--------|
| 正常（有运费） | 973919095109 | 974440018128 |
| 正常（包邮） | 975286818547 | 974764159302 |
| 下架 | 974766094420 | 976117992043 |
| sku删除&下架（黑色）&修改（白色） | 975472965387 | 974245895110 |
| 删除 | 974862810452 | 976200608164 |

### xs找工厂测试账号001

| 原链品状态 | 原链ID | 天猫ID |
|---------|--------|--------|
| 正常（有运费） | 976267684036 | 975607653138 |
| 正常（包邮） | 976636232777 | 975989733361 |
| 下架 | 975611281626 | 975607653146 |
| 删除 | 975611789624 | 975607653151 |
| sku删除&下架 | 975611281839 | 974384151131 |

## 环境配置

### hosts 文件

```
/Users/linxiaobei/hosts-configs/pre-tmall-rule1.txt
```

包含天猫交易、逆向、发票、会员、支付等全链路 hosts。

### 自动化脚本

| 脚本 | 路径 | 说明 |
|------|------|------|
| 环境切换 | `/Users/linxiaobei/scripts/open-alibaba-env.js` | Chromium hosts 注入 |
| 录制脚本 | `/Users/linxiaobei/scripts/recorded_script.py` | 含账号密码，仅参考 |

### 关键 host（交易，建议全开）

```
140.205.215.168 pre1-jinrong.1688.com
jinrong.1688.com pre1-jinrong.1688.com
140.205.215.168 bao.1688.com
140.205.173.181 member.1688.com
140.205.173.181 member.china.alibaba.com
```

## 操作入口

| 操作 | 链接 |
|------|------|
| 天猫小二审核退款申请（PC预发） | https://qn.wapa.taobao.com/home.htm/trade-platform/tp/sold |
| 天猫小二同意地址修改（改单服务） | https://qn.wapa.taobao.com/home.htm/seller-change-address-sku/ |

> 千牛客户端：添加 `wapa` 即可打开预发环境。

## 关键测试步骤

1. 天猫下单 → 约5s后供应商后台可见履约单（若没有，找研发重试创建渠道订单/履约单）
2. 供应商pc后台发货 或 商家工作台发货（支持对一主多子全部发货 / 对子单发货）
3. 买家确收

## DB 查询

- 数据库：`imall_operation`（DMS database_id = 1606956）
- 主单表：`qycg_mall_channel_sale_order`
- 子单表：`qycg_mall_channel_sale_order_detail`
- 履约单表：`qycg_mall_fullfill_order`（注意双 l）

## 金额公式

| 场景 | 公式 | 验证字段 |
|------|------|------|
| 买家实付（含红包） | out_paid_amount = out_paid_whithout_coupon_fee + out_coupon_fee | 主单 |
| 红包全额抵扣时 | out_paid_whithout_coupon_fee=0，out_coupon_fee=sale_price×qty | 主单 |
| 供应商结算金额 | total = purchase_price × quantity | 子单 |
| 履约单价格 | = purchase_price（含运供货价） × quantity | 子单+fulfill_order |

> **字段拼写 typo**：`out_paid_whithout_coupon_fee`（含多余 h，非 without），金额单位为分。

## 库存验证工具

itemtools（商品门户360）：`https://itemtools.taobao.com/item/search?itemId={天猫商品ID}`

## 自动化下单已知坑（脚本下单速查）

### 坑1：登录按钮 click 返回 exit code 1，但实际已登录成功

`playwright-cli click` 点击登录按钮后命令以 exit code 1 退出，看似失败。

**原因**：点击触发页面跳转，playwright-cli 把导航中断误报为失败，属于工具自身 bug。

**处理方式**：不要立即重试或重新登录。先执行 `playwright-cli snapshot` 检查当前 URL 和页面账号名，若已显示账号名说明登录实际成功，直接继续后续步骤。

---

### 准则：预发环境切换失败时必须停止，不得擅自降级

Step 1 环境切换脚本执行失败（报错退出、找不到 hosts 配置等）时，**必须立即停止流程**，向用户说明失败原因，让用户决策是否重试或放弃。

**原因**：用户指定预发环境可能是测试用途，擅自继续等于在错误环境制造垃圾数据。

---

### 坑2：确认订单页提交按钮文案是"立即支付"而非"提交订单"

天猫预发确认订单页实际按钮文案为**"立即支付￥0"**，而非 SKILL.md 中描述的"提交订单￥0"。

**处理方式**：用 `grep -B3 "提交订单"` 无结果时，改用 `grep -i "立即支付\|查看付款"` 定位按钮，再找其父容器 ref 点击。

## 关联

- [[biz-天猫通品]]
- [[biz-渠道履约]]
- [[biz-渠道商品体系]]
- [[test-浙建采]]
