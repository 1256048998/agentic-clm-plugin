---
name: analyze-contract
description: 合同检索、统计、对比与风险监控——按关键词查台账、按相对方/类型/主体统计金额、版本/改稿对比、到期/质保/付款预警、履约穿透溯源。用户说「查一下/搜合同、本月统计、对比两份、快到期/逾期、这单走到哪一步」时使用。
version: 1.0.0
allowed-tools:
  - mcp__agentic-clm__contract__search
  - mcp__agentic-clm__contract__stats
  - mcp__agentic-clm__contract__compare
  - mcp__agentic-clm__contract__alerts
  - mcp__agentic-clm__reminder__list
  - mcp__agentic-clm__contract__trace
  - mcp__agentic-clm__group__overview
  - mcp__agentic-clm__obligation__list
---

# 合同检索与风险分析工作流

## 1. 检索台账

- 用户报编号/名称/相对方关键词 → `contract__search`（返回最近 10 条：编号/名称/类型/金额/状态/相对方）。
- 精确拿 id：后续对比、改、提审前先 search 定位真实 id。

## 2. 金额统计

- 「本月各相对方/各类型/各主体合同金额统计、占比、汇总」→ `contract__stats`（数据库聚合，杜绝心算）。
- 前端已渲染图表，回复只给结论：最大相对方、占比、未填金额数；**不要**输出 ASCII 图表或重复明细表。

## 3. 合同对比

- 版本对比或与对方改稿对比 → `contract__compare`；工具按条款覆盖全文并返回字符位置与 `coverage`。
- 对比后突出金额/期限/责任变化，按风险排序；若 `coverage.complete=false`，明确列出失败片段并建议重试，禁止称“全文无风险”。

## 4. 风险监控

- 「快到期/逾期/该付款了」→ `contract__alerts`（合同到期/质保到期/付款节点三类，含剩余/逾期天数），可按 `type` 过滤。
- 「哪些合同快到期/有没有到期提醒」→ `reminder__list`（30 天窗口可调）。
- 回复按紧急度组织：已逾期优先于即将到期，标注剩余/逾期天数、相对方、主体。

## 5. 履约穿透

- 「这笔单据走到哪了/三单为什么不匹配」→ `contract__trace`（需合同/单据编号或穿透链 ID，缺入口先 search）。
- 「集团整体情况」→ `group__overview`（三单匹配统计、待处理差异数、资金池规模）。
- 「有哪些履约义务/里程碑/谁负责/哪些逾期」→ `obligation__list`；写入和完成动作交给履约助手并等待确认。

## 边界

本 skill 保持只读分析；若用户要求认领、差异销号或付款预检，切换履约助手并使用 `fulfillment__*` 写工具，等待确认后执行。
