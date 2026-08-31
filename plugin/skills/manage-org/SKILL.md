---
name: manage-org
description: 组织与账号管理——用户（创建/批量创建/改信息/停用/删除/重置密码）、角色（权限配置）、部门（组织架构）。用户说「建账号/开账号/加角色/建部门/组织架构/重置密码/离职停用」时使用。
version: 1.0.0
allowed-tools:
  - mcp__agentic-clm__user__list
  - mcp__agentic-clm__user__create
  - mcp__agentic-clm__user__create_batch
  - mcp__agentic-clm__user__update
  - mcp__agentic-clm__user__delete
  - mcp__agentic-clm__user__reset_password
  - mcp__agentic-clm__role__list
  - mcp__agentic-clm__role__create
  - mcp__agentic-clm__role__update
  - mcp__agentic-clm__role__delete
  - mcp__agentic-clm__department__list
  - mcp__agentic-clm__department__create
  - mcp__agentic-clm__department__update
  - mcp__agentic-clm__department__delete
  - mcp__agentic-clm__entity__list
---

# 组织与账号管理工作流

## 1. 用户账号

- `user__list` 查用户（账号/姓名/电话/邮箱/角色/部门/主体）。
- `user__create` 建账号（账号/姓名/密码/角色/部门/主体）；Excel 名单批量导入 → `user__create_batch`。
- `user__update` 改信息/换角色/调部门/停用（离职）；`user__delete` 软删，**待办自动转交部门负责人或管理员**。
- `user__reset_password` 重置登录密码；**未指定新密码时生成随机密码，只直接告知用户转达**。

## 2. 角色

- `role__list` 查角色；`role__create` 建角色（编码/名称/数据范围/权限点）；`role__update` **权限点整组替换**（改权限前先 list 当前权限点）；`role__delete` 软删。
- 改角色权限会立即影响该角色所有成员，先向用户说明影响面。

## 3. 部门

- `department__list` 按主体查部门树（名称/上级/负责人）；`department__create` 建部门（支持批量）。
- `department__update` 改名/换负责人；`department__delete` 软删，**存在下级部门不可删**，先处理下级。

## 4. 前置约定

- 主体归属：用户/部门都挂在法人主体下，先 `entity__list` 确认主体，数据范围遵守当前用户可见主体。
- 所有写操作均触发 Claude Code 审批确认，等待批准后再继续。
