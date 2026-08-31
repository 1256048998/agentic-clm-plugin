---
name: create-contract
description: 根据自然语言或上传文件创建合同——三种形态：结构化草稿、起草完整正文、批量建多份；并管理合同模板库（template__list/create/update/delete）。用户说「创建/新建/起草/根据文件建合同、把合同存成或修改模板」时使用。
version: 1.1.0
allowed-tools:
  - mcp__agentic-clm__dict__list
  - mcp__agentic-clm__entity__list
  - mcp__agentic-clm__counterparty__list
  - mcp__agentic-clm__user__list
  - mcp__agentic-clm__contract__create
  - mcp__agentic-clm__contract__draft
  - mcp__agentic-clm__contract__create_batch
  - mcp__agentic-clm__template__list
  - mcp__agentic-clm__template__create
  - mcp__agentic-clm__template__update
  - mcp__agentic-clm__template__delete
  - mcp__agentic-clm__file__read_chunk
---

# 创建合同工作流

按用户意图选择三种形态之一，遵循统一的前置查询 → 名称匹配 → 调用 → 确认汇报流程。

## 1. 判定形态

- **结构化草稿**（默认）：用户只说要素（名称/金额/相对方等），无正文要求 → `contract__create`，结果预填到「新建合同」页面，不落库。
- **起草正文**：用户要求「起草一份合同/帮我拟/写一份完整合同」→ `contract__draft`，生成条款式中文正文，落库为 v1 版本快照。
- **批量创建**：用户上传多个文件并要求「根据这些文件创建合同/批量建」→ `contract__create_batch`，数组元素与文件一一对应。
- **模板库**：查询 → `template__list`；新建 → `template__create`；修改正文/名称/启停 → `template__update`；删除 → `template__delete`。修改和删除前先 list 定位 id。

## 2. 前置查询（无论哪种形态都先做）

并行拉取四类参考数据：

1. `dict__list` → 合同类型字典（一级 + 二级，如「采购合同/设备采购」）
2. `entity__list` → 签约主体（法人主体）
3. `counterparty__list` → 相对方
4. `user__list` → 负责人

## 3. 名称匹配 id（禁止猜 id）

- 把用户给的相对方名称、主体名称、负责人姓名，在对应列表里**精确匹配**出 `id`。
- 匹配不上时：列出前 20 个可选项给用户确认后再调，绝不凭印象编造 id。
- 合同类型必须选到二级：`category` = 一级，`typeCode` = 二级（如「采购合同」「设备采购」）。

## 4. 调用工具并回填

- 用户没给名称时按场景自动生成（如「软件开发服务合同」+ 日期），不追问。
- 用户明确说「已填了一部分/帮我补齐/别覆盖我填的」时传 `fillMode: fill_blank`，否则默认 `fill_all`。
- **套用模板**：起草正文前先 `template__list` 查可用模板；命中时把模板名传给 `contract__draft` 的 `templateName`，骨架条款结构会被保留并替换占位信息。用户指定的模板名要精确匹配，查不到就如实告知并列出可选项。
- **长文件全文**：上传文件若提示 chunkCount>1，用 `file__read_chunk` 从 chunkIndex=1、count=6 开始，按 nextChunkIndex 继续到 coverage.complete=true 后再起草/总结；不得只根据首片预览推断全文。
- 附件：对话中上传的文件会随调用自动转挂为合同附件，无需特殊处理。

## 5. 汇报 + 确认

- **create 不落库**：明确告诉用户「已生成合同草稿建议并填充到新建合同页面，请确认后保存或提交审批」。
- **draft 落库**：报告合同编号、正文长度与「正文已存入版本 v1」。
- 写操作会触发 Claude Code 审批确认，等待用户批准后再认为完成。
