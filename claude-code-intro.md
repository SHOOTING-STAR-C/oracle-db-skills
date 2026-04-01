# Claude Code 入门指南

## 概述

本文档介绍 Claude Code 的安装方法，以及 Agent、MCP、Skills 三个核心概念，帮助你快速上手 AI 辅助 Oracle 数据库开发。

---

## 一、Claude Code 简介

Claude Code 是 Anthropic 官方推出的命令行工具，为开发者提供 AI 辅助编程能力。你可以通过自然语言与代码库交互、完成代码编写和调试、执行 git 操作、搜索代码库等。

---

## 二、安装 Claude Code

### 系统要求

| 项目 | 要求 |
|------|------|
| 操作系统 | Windows（需 Git Bash）、macOS、Linux |
| Node.js | 18.x 或更高（通过 npm 安装时需要） |
| Git Bash | Windows 必须（用于运行 Claude Code） |
| 内存 | 推荐 8GB+ |
| 磁盘 | 200MB 可用空间 |
| 网络 | 需要访问 Anthropic API |

### Windows 安装步骤

1. **安装 Node.js**
   - 下载 [Node.js 官网安装包](https://nodejs.org/)，LTS 版本即可
   - 安装后打开命令提示符验证：`node -v` 和 `npm -v`

2. **安装 Git Bash**
   - 下载 [Git for Windows](https://git-scm.com/download/win)
   - 安装时选择 "Use Git from Bash Only"
   - 记住 Git 安装路径（如 `C:\Program Files\Git`）

3. **配置环境变量**
   - Win + R → `sysdm.cpl` → 高级 → 环境变量
   - 新建系统变量：
     - 变量名：`CLAUDE_CODE_GIT_BASH_PATH`
     - 变量值：`C:\Program Files\Git\bin\bash.exe`

3. **安装 Claude Code**
   - 下载 [Claude Code CLI](https://github.com/anthropics/claude-code/releases) 或通过 npm 安装：
   ```cmd
   npm install -g @anthropic-ai/claude-code
   ```

4. **验证安装**
   ```cmd
   claude --version
   ```

5. **首次登录**
   ```cmd
   claude
   ```
   首次运行会提示输入 API Key，按提示操作即可。

### macOS / Linux 安装

```bash
npm install -g @anthropic-ai/claude-code
# 或
brew install claude-code
```

---

## 三、核心概念

### 3.1 Agent（智能体）

**Agent 是 AI 的"分身"**，是一个能够自主思考、规划并执行任务的智能程序。在 Claude Code 中，Agent 能帮你完成复杂的多步骤任务。

**核心能力：**
- 理解自然语言描述的目标
- 将大任务分解为小步骤
- 自主决定调用什么工具
- 在失败时自动调整策略

**类比理解：**
- 你给 Agent 一个目标（如"帮我优化这条 SQL"）
- Agent 会自动分析问题、制定计划、执行操作、验证结果
- 就像一个经验丰富的数据库工程师坐在你旁边工作

**Agent 与普通聊天机器人的区别：**

| 特性 | 普通聊天 | Agent |
|------|---------|-------|
| 任务完成 | 只能回答问题 | 能自动执行并交付结果 |
| 工具调用 | 需要手动指定 | 自动判断并调用所需工具 |
| 多步骤任务 | 需要一步步引导 | 自动分解并连贯执行 |
| 上下文感知 | 只看当前对话 | 深度理解整个代码库 |

---

### 3.2 MCP（Model Context Protocol，模型上下文协议）

**MCP 是一种通信协议**，让 AI 能够与外部工具和数据源安全交互。

**解决的问题：**
- AI 模型（如 Claude）本身无法访问你的数据库、文件系统
- MCP 通过标准化的接口，让 AI 安全地"调用"外部工具
- 就像给 AI 装上了"插头"，可以连接各种数据源

**MCP 的工作原理：**

```
你的项目代码库
      ↓
Claude Code（AI 客户端）
      ↓  stdio 通信
MCP Server（SQLcl MCP Server）
      ↓  JDBC/ODBC
Oracle 数据库
```

**为什么需要 MCP？**
- AI 无法直接读你的数据库——需要 MCP Server 作为桥梁
- SQLcl MCP Server 让 AI 能够执行 SQL、查询数据
- Claude Code 通过 MCP 调用 SQLcl，操作 Oracle 数据库

**一个比喻：**
- AI 是"大脑"，能思考和分析
- MCP 是"手和眼睛"，能接触外部世界
- 没有 MCP，AI 只能空想；有了 MCP，AI 才能动手做事

---

### 3.3 Skills（技能库）

**Skills 是 Oracle 数据库知识库的独立文档**，每个文件涵盖一个具体主题，包含 explanation、examples、version notes 和 sources。

**本仓库的 Skills 结构：**

```
skills/
├── admin/          # 运维：备份恢复、用户管理、日志
├── appdev/         # 应用开发：JDBC、JSON、Python/Node.js
├── architecture/   # 架构：RAC、CDB/PDB、Exadata
├── design/         # 设计：ERD、 partitioning
├── devops/         # DevOps：迁移、CI/CD、utPLSQL
├── features/       # 特性：高级队列、调度器、物化视图
├── migrations/     # 迁移：从 PG、MySQL、SQL Server 等迁入
├── monitoring/     # 监控：告警、AWR、top SQL
├── ords/           # ORDS、REST API、OAuth2
├── performance/    # 性能：AWR、执行计划、索引、统计信息
├── plsql/          # PL/SQL：包、游标、集合、调试
├── security/       # 安全：权限、VPD、TDE、审计
├── sql-dev/        # SQL 开发：模式、函数、注入防范
├── sqlcl/          # SQLcl：命令、MCP、Liquibase
└── containers/     # 容器：OCR 镜像、Helm、Operator
```

**为什么需要 Skills？**
- 给 AI 提供了 Oracle 数据库的权威参考文档
- AI 在回答问题时直接从 Skills 中读取，不再"凭空编造"
- 每个 Skill 都有官方文档来源，保证准确性

**Skills 如何工作？**
- Claude Code 配置了 `oracle-db-skills` 技能
- 当你询问 Oracle 相关问题时，AI 会自动检索相关 Skill 文件
- Skill 文件包含该主题的完整知识：语法、示例、最佳实践

---

## 四、三者关系

```
┌─────────────────────────────────────────────────┐
│                   Claude Code                    │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│  │  Agent   │    │   MCP    │    │  Skills  │   │
│  │ (行动者) │    │ (桥梁)   │    │ (执行方案) │   │
│  └────┬─────┘    └────┬─────┘    └────┬─────┘   │
│       │               │               │          │
│       │     ┌─────────┴─────────┐      │          │
│       └─────┤  SQLcl MCP Server ├─────┘          │
│             └─────────┬───────┘                  │
│                   Oracle DB                       │
└─────────────────────────────────────────────────┘
```

- **Agent** — 负责思考和决策，像一个经验丰富的工程师
- **MCP** — 负责通信，连接 AI 与外部工具和数据
- **Skills** — 负责执行方案，提供 Oracle 领域具体任务的操作指南和最佳实践

**典型工作流程示例：**

```
你："帮我分析这条 SQL 为什么慢"

Agent 接收任务
    ↓
Agent 读取 Skills 中的 sql-tuning.md、执行计划文档
    ↓
Agent 通过 MCP 调用 SQLcl，执行 SQL 分析
    ↓
Agent 整合结果，给出优化建议
```

---

## 五、快速上手

### 1. 配置 Skills（已有本仓库）

Claude Code 已配置加载 `oracle-db-skills` 知识库，可直接询问 Oracle 相关问题。

### 2. 配置 MCP（连接 Oracle 数据库）

```json
// ~/.claude.json 或项目 .mcp.json
{
  "mcpServers": {
    "oracle-sqlcl": {
      "command": "D:\\software\\sqlcl\\bin\\sql",
      "args": ["-mcp"],
      "type": "stdio"
    }
  }
}
```

### 3. 开始使用

```cmd
# 直接提问
claude "帮我查看 Oracle 数据库的版本"

# 分析慢 SQL
claude "分析 v$sql 中最耗资源的 SQL"

# 执行数据库操作
claude "帮我创建一个只读用户用于 MCP 连接"
```

---

## 六、相关文档

| 文档 | 说明 |
|------|------|
| [sqlcl-mcp-server.md](sqlcl/sqlcl-mcp-server.md) | SQLcl MCP Server 完整配置指南 |
| [sqlcl-mcp-sql-diagnosis.md](sqlcl/sqlcl-mcp-sql-diagnosis.md) | AI 辅助 SQL 诊断实战 |
| [sqlcl-basics.md](sqlcl/sqlcl-basics.md) | SQLcl 基础入门 |
| [explain-plan.md](performance/explain-plan.md) | 执行计划深度解读 |
| [sql-tuning.md](sql-dev/sql-tuning.md) | SQL 调优完整指南 |

---

## Sources

- [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)
- [MCP 官方协议规范](https://modelcontextprotocol.io/)
- [Oracle SQLcl 下载](https://www.oracle.com/tools/downloads/sqlcl-downloads.html)
