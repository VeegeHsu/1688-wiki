---
tags: [tech, 监控, 治理, 企业采购]
type: tech
date: 2026-04-27
source: "raw/企业采购监控治理.md"
---

# 企业采购监控治理

## 接口监控告警

| 接口 | 告警信息 | 归属业务 | 责任人 | Fix方案 |
|-----|---------|---------|-------|--------|
| `QycgMallHyperLinkAuditReadServiceImpl#getItemRegistrationDetail` | 调用发生未catch处理异常 | qiye caig | 作鸿 | 调用TPP重试失败才会告警，频率不算特别高，保留此告警方便排查tpp问题，不作处理 |

## 关联

- [[biz-渠道商品体系]]
- [[biz-钉钉Agent]]
