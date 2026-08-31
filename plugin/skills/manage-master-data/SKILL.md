---
name: manage-master-data
description: 主数据维护——相对方（供应商/客户档案、拉黑）、签约主体（子公司/孙公司）、字典（合同类型/单位/发票/付款条件/制度与条款分类）、审批上报规则。用户说「新增/录入/拉黑一个相对方、建一个主体、加一个字典选项、配上报规则」时使用。
version: 1.0.0
allowed-tools:
  - mcp__agentic-clm__counterparty__list
  - mcp__agentic-clm__counterparty__create
  - mcp__agentic-clm__counterparty__update
  - mcp__agentic-clm__counterparty__delete
  - mcp__agentic-clm__entity__list
  - mcp__agentic-clm__entity__create
  - mcp__agentic-clm__entity__update
  - mcp__agentic-clm__entity__delete
  - mcp__agentic-clm__dict__list
  - mcp__agentic-clm__dict__create
  - mcp__agentic-clm__dict__update
  - mcp__agentic-clm__dict__delete
  - mcp__agentic-clm__escalation_rule__list
  - mcp__agentic-clm__escalation_rule__create
  - mcp__agentic-clm__escalation_rule__update
  - mcp__agentic-clm__escalation_rule__delete
---

# 主数据维护工作流

## 1. 相对方（供应商/客户档案）

- `counterparty__list` 查库（名称/统一社会信用代码/类型/所属主体/拉黑状态/状态）。
- `counterparty__create` 录入新相对方；`counterparty__update` 改信息或拉黑/解除拉黑；`counterparty__delete` 软删（历史合同仍可追溯）。
- **同一相对方先查重**：录入前 list 确认不存在同名同代码，避免重复档案。

## 2. 签约主体（法人主体）

- `entity__list` 查主体树；`entity__create` 建子公司/孙公司；`entity__update` 改名称/信用代码/风险等级/上收阈值；`entity__delete` 软删，**有下级主体不可删**，先处理下级。
- 主体层级与数据范围相关：建主体前向用户确认父级归属。

## 3. 字典

- `dict__list` 查字典项（合同类型/明细行单位/发票类型/付款条件/制度分类/条款分类）。
- `dict__create` 加选项（如新增合同类型）；`dict__update` 改名称/编码/排序/停用；`dict__delete` **有子项时服务端拒绝，先删子项**。

## 4. 审批上报规则

- `escalation_rule__list` 查当前规则（仅数据范围内可管理）。
- `escalation_rule__create` 新增（父级为子孙配置：满足条件触发集团/上级审批或通知）；`update`/`delete` 修改/软删。
- **适用范围用显式开关**：规则适用对象不默认「空值=全部」，向用户确认明确的适用主体/类型/金额阈值后再配置。

## 5. 确认与边界

- 所有创建/修改/删除均触发 Claude Code 审批确认，等待批准后再继续。
- 先 list 查重、拿 id，禁止凭印象造数据。
