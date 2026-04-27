---
tags: [biz, 钉钉Agent, 采购单, 推品, 智能报销]
type: biz
date: 2026-04-27
---

# 钉钉 Agent 业务逻辑

## 采购单核心规则

| 规则 | 说明 |
|------|------|
| 草稿逻辑 | 存在最近一次未提交的采购草稿 → 新商品加入草稿；否则新建采购单 |
| SKU 上限 | 采购明细最多 **20 个 SKU** |
| 主订单上限 | 采购单创建订单最多 **20 笔**主订单 |
| 关闭采购 | 待支付时可关闭 → 采购单取消，对应合约单/订单同步关闭 |
| 定时巡检 | 定时任务巡检订单是否全部关闭，全部关闭则更新采购单为已取消 |

## 采购单状态流转

```
草稿
  ↓ 提交审批
审批中
  ├─ 审批通过 → 待支付
  └─ 审批驳回 → 取消

待支付
  ├─ 储蓄罐扣减成功 → 下单中
  └─ 关闭采购 → 取消

下单中
  └─ 监听 1688 订单已付款消息 → 采购中

采购中 → 已完成
```

## 三方比价（2026-03 315优化）

| 场景 | 行为 |
|------|------|
| 相同商品 + 相同数量 | **不重新触发**比价 |
| 修改 SKU 或修改数量 | **重新触发**比价 |

结果共享：同组织内共享；不同组织间不共享。

| 状态 | 可选行动点 |
|------|-----------|
| 比价失败 / 异常 | 重新比价 / 立即下单 / 查看历史记录 |
| 比价成功 | 立即下单 / 查看历史记录 |

## 推品链路架构（2026-04-22 升级）

三条并行链路：

| 链路 | 协议 | 核心能力 |
|------|------|----------|
| **链路一：A2A 推品** | http（A2A） | 意图识别 + H5 卡片推品 |
| **链路二：流式查询** | mtop（流式） | 多意图分发：问答 / 订单查询 |
| **链路三：商卡查询** | HSF | 商品信息查询（独立链路） |

**链路一负责「把 H5 卡片放到用户面前」，链路二是这张卡片「开口说话」。**
流式说话只能由站在用户面前的前端容器发起，后端 A2A 无法代劳（会退化为批式等待）。

## 智能报销业务逻辑（2026-04-20）

### 报销入口

**正确入口**：对账中心页面 → 勾选已开票完成订单 → 点击**申请报销**按钮。
空选时申请报销按钮**置灰**（非弹出提示）。

### 报销单状态枚举

| 对象 | 状态 | 说明 |
|------|------|------|
| agent 报销单 | `REIMBURSING` | 创建成功后初始状态 |
| | `COMPLETED` | OA 审批通过 |
| | `CANCELLED` | 报销作废 |
| 采购子单报销状态 | `PENDING_REIMBURSEMENT` | 初始 / 作废后回到此状态 |
| | `REIMBURSING` | 报销单创建后同步 |
| | `COMPLETED` | 报销单 COMPLETED 后同步 |

> **作废后**：采购子单回到 `PENDING_REIMBURSEMENT`，**不是报销失败**，可重新从对账中心发起报销。

### 创建报销同步调用链路

```
对账中心勾选已开票订单 → 申请报销
  ↓
展示企业报销模版列表 → 用户选择确认
  ↓
【同步】createAgentReimburse：
  ① 创建 agent 报销单（存 processcode）
  ② 发票附件同步上传至钉盘
  ↓ 若①②均成功
报销单 REIMBURSING，采购子单 REIMBURSING
  ↓
【异步】源神填写 OA 报销单（根据 processcode 自动填写）
  → 完成后：详情页展示【跳转 OA 审批单】按钮
```

**易错点**：发票附件上传钉盘在 createAgentReimburse 同步执行（不在「创建OA报销审批单」节点下）。

## Mock 工具（预发）

| 工具 | 说明 |
|------|------|
| `com.alibaba.china.imall.api.service.TestService:1.0.0#testAgentPushV2` | Mock 发消息接口 |
| 所属应用 | imall-hub-gateway，envType: pre |

Diamond 配置：dataId=`agent_mock_config`，group=`imall-hub-gateway`

## 核心 HSF 接口

| 接口 | 应用 | 说明 |
|------|------|------|
| `AgentPurchaseOrderAppService.createPurchaseOrder` | imall-hub-gateway | 创建采购单 |
| `AgentPurchaseOrderAppService.createOrder` | imall-hub-gateway | 创建订单 |

关键日志：`AgentPurchaseOrderAppServiceImpl.createPurchaseOrder start`

## 来源

- 原始资料：`raw/钉钉Agent-业务逻辑.md`
- 测试指南：`raw/Readme-企采钉钉Agent测试.md` → [[test-钉钉Agent]]

## 关联

- [[biz-飞猪Agent]]
- [[biz-淘宝企业购]]
- [[biz-渠道商品体系]]
- [[biz-渠道订单开票]]
- [[test-钉钉Agent]]
