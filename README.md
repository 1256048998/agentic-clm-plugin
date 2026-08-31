# AgenticCLM · Claude Code 插件

把 AgenticCLM 合同全生命周期管理系统打包为 Claude Code / NoBuddy 插件：通过 MCP 暴露全部业务能力（当前生产环境 110 个工具，覆盖合同、审批、履约、合规、系统 5 个域），并附 5 个领域 agent 与 6 个工作流 skill，让客户端按任务自动匹配，而不是面对一堆工具临场摸索。

- 稳定版本：`v1.2.0`
- 正式服务：`https://agenticclm.657890.xyz`
- 生产 MCP：`https://agenticclm.657890.xyz/api/mcp`
- 公开发行仓库：`https://github.com/1256048998/agentic-clm-plugin`

## 结构

```
plugin/
├── .claude-plugin/plugin.json   # 插件清单
├── .mcp.json                    # MCP server（HTTP，标准浏览器 OAuth）
├── agents/                      # 5 个领域 agent（@ 可调）
│   ├── contract-assistant.md    #   合同
│   ├── approval-assistant.md    #   审批/风控
│   ├── fulfillment-assistant.md #   履约/预警/穿透
│   ├── compliance-assistant.md  #   制度/条款/合规
│   └── system-admin.md          #   组织/主数据/系统
└── skills/                      # 6 个工作流 skill（自动触发）
    ├── create-contract/         #   创建/起草/批量建合同
    ├── analyze-contract/        #   检索/统计/对比/预警/履约
    ├── manage-approval/         #   审批流/审批点/提审
    ├── manage-compliance/       #   制度/条款/合规核查
    ├── manage-master-data/      #   相对方/主体/字典/上报规则
    └── manage-org/              #   用户/角色/部门
```

## 连接方式

插件默认连接正式生产 MCP，无需填写服务器地址：

```text
https://agenticclm.657890.xyz/api/mcp
```

私有化部署可在启动 Claude Code / NoBuddy 前设置 `AGENTIC_CLM_URL`，只填写 HTTPS 源站，不包含 `/api/mcp`：

```bash
AGENTIC_CLM_URL=https://clm.example.com claude
```

私有服务端需将 `MCP_PUBLIC_URL` 与 `MCP_WEB_URL` 设置为同一正式 HTTPS 源站，并公开 `/api/mcp`、`/.well-known/*`、`/authorize`、`/token`、`/register` 与 `/revoke`。桌面客户端的 OAuth 回调地址使用 `http://127.0.0.1:<随机端口>/...` 或 `http://localhost:<随机端口>/...`。

普通用户和管理员都不需要复制 token。插件会在系统浏览器打开 AgenticCLM 登录页，并沿用登录账号所属租户、角色和数据权限。

## 安装

### 方式 A：通过公开 marketplace 安装（推荐）

```bash
/plugin marketplace add 1256048998/agentic-clm-plugin
/plugin install agentic-clm@agentic-clm
```

安装或升级后，在 Claude Code 中执行 `/reload-plugins`，再通过 `/mcp` 完成浏览器登录。

### 方式 B：本地路径安装（开发/私有化）

```bash
git clone https://github.com/1256048998/agentic-clm-plugin.git
/plugin install <克隆目录绝对路径>/plugin
```

### 方式 C：在 NoBuddy 中本地安装

1. 添加 marketplace：`https://github.com/1256048998/agentic-clm-plugin`，或选择克隆后的 `plugin/` 目录。
2. 公网版无需填写地址；私有部署时填写 AgenticCLM 的 HTTPS 源站并保存。
3. 点击“浏览器登录”，使用自己的 AgenticCLM 账号登录。返回 NoBuddy 显示“已连接”后，新建任务即可使用插件。

### 校验

```bash
claude plugin validate ./plugin            # 校验插件清单
claude plugin validate .                   # 校验 marketplace 清单
claude mcp list                            # 确认 agentic-clm 已连接
```

## 使用

- **领域 agent**：在输入框 `@contract-assistant` / `@approval-assistant` / `@fulfillment-assistant` / `@compliance-assistant` / `@system-admin` 显式调用，或让 Claude Code 按任务自动路由。
- **工作流 skill**：直接提需求即可自动触发，例如「帮我起草一份采购合同」「本月各相对方合同金额统计」「把这份合同提交审批」「发布一条采购管理制度」「给新员工开账号」。
- **权限一致**：普通账号只能执行其在 AgenticCLM 中已有权限允许的操作；集团管理员登录后才能创建用户、角色等集团级数据。
- **写操作确认**：创建/修改/删除/提审等写操作仍由宿主审批确认，系统侧同时落 AuditLog 审计。

## 安全说明

- OAuth access/refresh token 由宿主安全存储，不写回插件目录，也不暴露给 Agent Runtime；NoBuddy 只向运行时提供本机回环代理。
- `MCP_READ_TOKEN` / `MCP_WRITE_TOKEN` 继续保留给脚本、CLI 和无人值守系统集成，不作为普通用户安装步骤。
- MCP 通道写操作 `confirmed: true` 直接执行，人工确认由宿主的工具审批弹窗承担。
- 公开发行仓库只包含插件、marketplace、skills、agents、README 与许可证；AgenticCLM 主业务代码及任何部署密钥均不公开。
