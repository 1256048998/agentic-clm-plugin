---
name: manage-approval
description: 审批体系搭建、合同提审与待办处理——创建/发布审批流与 AI 审批点，提交、通过、驳回、转办、取消、加签和催办。用户说「建审批流/加审批点/提交或处理审批/管控报告」时使用。
version: 1.0.0
allowed-tools:
  - mcp__agentic-clm__approval_flow__list
  - mcp__agentic-clm__approval_flow__create
  - mcp__agentic-clm__approval_flow__publish
  - mcp__agentic-clm__approval_flow__disable
  - mcp__agentic-clm__approval_flow__delete
  - mcp__agentic-clm__approval_point__list
  - mcp__agentic-clm__approval_point__create
  - mcp__agentic-clm__approval_point__update
  - mcp__agentic-clm__approval_point__distribute
  - mcp__agentic-clm__contract__submit_approval
  - mcp__agentic-clm__contract__search
  - mcp__agentic-clm__control__report
  - mcp__agentic-clm__contract__judgement
  - mcp__agentic-clm__approval__my_tasks
  - mcp__agentic-clm__approval__handle
  - mcp__agentic-clm__approval__transfer
  - mcp__agentic-clm__approval__cancel
  - mcp__agentic-clm__approval__add_signer
  - mcp__agentic-clm__approval__urge
---

# 审批管理工作流

## 1. 搭建审批流

- 用户描述「部门负责人审批 → AI 风控判定」这类需求 → `approval_flow__create`（生成节点树草稿）。
- 发布前先 `approval_flow__list` 确认流程、适用主体×合同类型范围 → `approval_flow__publish`（版本 +1，发布后按主体×类型匹配生效）。
- 修改流程先 `disable` 再重建或改草稿，**已发布流程删除前必须先停用**。

## 2. 配置 AI 审批点

- 自然语言要求 → `approval_point__create`（AI 内部解析为确定性条件 + 语义检查，创建即生效）。
- 收紧/放开/停用/启用 → `approval_point__update`。
- 集团模板下发 → `approval_point__distribute`，**不填主体 = 全集团下发**，先向用户确认范围。

## 3. 提交合同审批

- `contract__submit_approval`：传合同 id，不传 id 时传名称自动查找。
- 前置校验：合同为草稿态；该签约主体×合同类型已配置对应审批流；无匹配流时提示先配置。

## 4. 管控视角

- 「各主体触发/拦截/绕行统计」→ `control__report`（≈绕行率是管控重点）。
- 「这份合同为什么被拦截/提示」→ `contract__judgement`（审批要求 + 命中证据 + 拦截/提示结果）。

## 5. 处理审批待办

- 先 `approval__my_tasks` 查询真实 taskId；通过/驳回用 `approval__handle`，只能处理分配给当前账号的待办。
- 转办用 `approval__transfer`；取消我发起的审批用 `approval__cancel`；加签/催办分别用 `approval__add_signer` / `approval__urge`。
- 驳回与取消会让合同退回草稿，确认卡中应明确影响。

## 6. 确认与边界

- 创建/发布/停用/删除/提审均为写操作，等待 Claude Code 审批确认后再继续。
- 先 list 拿 id 再改/停/删，禁止凭名称猜 id。
