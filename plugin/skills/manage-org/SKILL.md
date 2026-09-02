---
name: manage-org
description: 组织与账号管理——用户（创建/批量创建/改信息/停用/删除/重置密码）、角色（权限配置）、部门（组织架构）。用户说「建账号/开账号/加角色/建部门/组织架构/重置密码/离职停用」时使用。
version: 1.0.0
allowed-tools:
  - mcp__agentic-clm__user__list
  - mcp__agentic-clm__user__create
  - mcp__agentic-clm__user__create_batch
  - mcp__agentic-clm__user__provision_entity_admin
  - mcp__agentic-clm__user__reset_delegated_admin_password
  - mcp__agentic-clm__user__manage_delegated_admin
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
- 普通建账号仅限当前公司。为下级公司创建公司管理员必须用 `user__provision_entity_admin`：公司管理员可逐级向自己的严格下级委派，目标管理员固定为该主体 `ENTITY` 数据范围。
- 下级管理员忘记密码用 `user__reset_delegated_admin_password`；启用、停用或软删除用 `user__manage_delegated_admin`。停用/删除会使旧会话失效并转交未完成审批待办。
- `user__update` 改信息/换角色/调部门/停用（离职）；`user__delete` 软删，**待办自动转交部门负责人或管理员**。
- `user__reset_password` 重置登录密码；**未指定新密码时生成随机密码，只直接告知用户转达**。

## 2. 角色

- `role__list` 查角色；`role__create` 建角色（自动归属当前公司，默认 `LOCAL` 本公司级）；只有下级上报场景才显式使用 `CROSS_ENTITY` 跨公司级。
- `CUSTOM` 数据范围用 `customEntityNames` 选择公司；只能选操作者已有范围内的公司。`role__update` 的权限点为**整组替换**，改权限前先 list 当前值；`role__delete` 软删。
- 持有 `system:role` 的用户可管理本公司角色，但新角色的权限点与数据范围必须是自身授权的子集。
- 改角色权限会立即影响该角色所有成员，先向用户说明影响面。

## 3. 部门

- `department__list` 按主体查部门树（名称/上级/负责人）；`department__create` 建部门（支持批量）。
- `department__update` 改名/换负责人；`department__delete` 软删，**存在下级部门不可删**，先处理下级。

## 4. 前置约定

- 当前公司是普通写入边界：用户/部门等只允许维护当前公司；下级可见数据仅供查询。上报合同由审批流程处理，不得直接修改下级源数据。
- 管理员委派只能沿法人树向下，禁止同级、上级和跨分支授权；每次委派都要确认并审计。
- 所有写操作均触发 Claude Code 审批确认，等待批准后再继续。
