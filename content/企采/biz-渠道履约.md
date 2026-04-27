---
tags: [biz, 渠道履约, 履约单]
type: biz
date: 2026-04-27
---

# 渠道履约

## 履约单权限规则

| 角色 | 可见范围 |
|------|----------|
| 大店（平台侧运营） | 可查看**所有**供应商的履约单 |
| 供应商 | 只能查看**各自**的履约单 |

## 测试入口

| 环境 | 地址 |
|------|------|
| 预发 | `https://pre-mychain.1688.com/#/performance-note-list` |
| 线上 | `https://mychain.1688.com/#/performance-note-list` |

测试账号：
- 大店：`b工业品测试账号010`
- 供应商1：`b工业品测试账号tp001`
- 供应商2：`xs找工厂测试账号001`

## 聚宝工具 sceneId（内部数据构造/查询平台）

| 场景 | sceneId |
|------|---------|
| 查询 specId | `2323` |
| 查询履约单 | `2337` |
| 生成履约单 | `2325` |
| 履约单发货 | `2347` |
| 履约单确认收货 | `2362` |
| 履约单发起逆向流程 | `2363` |
| 渠道订单测试 | `2381` |

## 核心接口

| 接口 | 参数 | 参数类型 |
|------|------|----------|
| `com.alibaba.china.imall.scm.domain.facade.order.OrderFacade:1.0.0@getOrderById~L` | `lvyueOrderId` | `java.lang.Long` |

## 来源

- 原始资料：`raw/渠道履约-业务逻辑.md`

## 关联

- [[biz-渠道商品体系]]
- [[biz-浙建采]]
- [[biz-天猫通品]]
- [[biz-钉钉Agent]]
- [[biz-飞猪Agent]]
- [[biz-KA账期]]
