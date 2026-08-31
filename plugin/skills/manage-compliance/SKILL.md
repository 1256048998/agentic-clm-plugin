---
name: manage-compliance
description: 制度与标准条款管理——制度录入/检索/发布下发/签收/废止/删除，标准条款库维护，合同正文与条款库合规比对。用户说「发一条制度/发布管理制度/签收制度/加一条标准条款/合同条款合规核查」时使用。
version: 1.0.0
allowed-tools:
  - mcp__agentic-clm__policy__create
  - mcp__agentic-clm__policy__search
  - mcp__agentic-clm__policy__list
  - mcp__agentic-clm__policy__publish
  - mcp__agentic-clm__policy__ack
  - mcp__agentic-clm__policy__retire
  - mcp__agentic-clm__policy__delete
  - mcp__agentic-clm__clause__list
  - mcp__agentic-clm__clause__create
  - mcp__agentic-clm__clause__update
  - mcp__agentic-clm__clause__delete
  - mcp__agentic-clm__clause__compare
  - mcp__agentic-clm__contract__search
---

# 制度与条款管理工作流

## 1. 制度生命周期

- **录入**：用户粘贴制度正文/描述要求 → `policy__create`（LLM 拆分条文入库，供检索与审批判定引用）。
- **发布**：`policy__publish` 将草稿发布为现行（ACTIVE）并下发签收任务；**不填适用主体 = 全部法人主体下发**，先向用户确认范围。
- **签收**：`policy__ack` 是「当前 MCP 用户」对其适用范围内现行制度签收，明确操作主体。
- **废止**：`policy__retire` 现行 → 废止（REVOKED），废止后不再作为审批判定引用。
- **删除**：`policy__delete` 仅草稿/已废止可删，**现行制度先废止再删**。

## 2. 标准条款库

- `clause__list` 查条款库；`clause__create`/`update`（版本 +1）/`delete` 维护标准条款。
- 用户问「有哪些标准条款/引用哪条」时先 list 再答，不要凭空引用。

## 3. 合规核查

- 合同正文 vs 条款库 → `clause__compare`（覆盖全部正文和全部标准条款，逐条识别符合/偏离/缺失并给位置证据）。
- 检查 `coverage.complete`：false 时 UNKNOWN 不是缺失，说明失败片段并重试；只有 complete=true 的结果才会写条款引用留痕。
- 版本/改稿对比 → `contract__compare`（见 analyze-contract）。
- 核查前先用 `contract__search` 拿到合同真实编号（工具参数为 contractCode）。

## 4. 确认与边界

- 录入/发布/废止/删除均为写操作，等待 Claude Code 审批确认后再继续。
- 先 list 定位 id，禁止凭名称猜 id。
