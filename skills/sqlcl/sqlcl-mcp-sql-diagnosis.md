# SQLcl MCP 实战指南 — AI 辅助 SQL 诊断

## 概述

本文档介绍如何通过 Oracle SQLcl MCP Server 连接 AI 助手，实现 SQL 诊断分析能力，通过 AI 辅助定位慢 SQL 问题并给出优化建议。

---

## 第一部分：SQLcl MCP Server 安装与配置

### 1.1 环境要求

| 组件 | 版本要求 |
|------|---------|
| SQLcl | **25.2.0 或更高**（MCP 从此版本引入） |
| JRE | 17 或 21 |
| Oracle 数据库 | 12c 及以上（12c/18c/19c/21c/23c/23ai） |
| AI 客户端 | Claude Desktop / Claude Code / VS Code + Cline 等支持 MCP 的工具 |

> ⚠️ Oracle 11g 不支持。11g 的 JDBC 协议与新版 SQLcl 不兼容。

---

### 1.2 安装 SQLcl 25.2+（Windows）

**方式一：下载安装（推荐）**

1. 访问 [Oracle SQLcl 下载页面](https://www.oracle.com/cn/database/sqldeveloper/technologies/sqlcl/)
2. 下载 Windows 版本 zip 包（约 200MB）
3. 解压到指定目录（如 `D:\sqlcl`）
4. 将 `D:\sqlcl\bin` 添加到系统 PATH 环境变量

**方式二：如果装了 SQL Developer，自带 SQLcl**

```
D:\app\oracle\Product\SQL Developer\sqldeveloper\sqlcl\bin\sql.exe
```

**配置 PATH（Windows）：**

1. 按 `Win + R`，输入 `sysdm.cpl`，回车
2. 高级 → 环境变量 → 系统变量 → 编辑 PATH
3. 添加 `D:\sqlcl\bin`（改成你的实际路径）
4. 确定保存

**验证安装：**

```cmd
sql -V
# 输出应为 25.2.0 或更高
```

---

### 1.3 保存数据库连接

MCP Server **不接受命令行传入凭据**，必须预先保存连接。

```cmd
# 启动 SQLcl（不连接）
sql /nolog
```

```sql
-- 保存连接（普通连接方式）
conn -save my_orcl -savepwd username/password@//hostname:1521/service_name

-- 验证连接已保存
conn -list
```

参数说明：
- `-save <name>` — 为连接指定一个名称
- `-savepwd` — 将密码加密存储到用户目录下 `\.dbtools`，**必须有此参数 MCP 才能使用该连接**

> ⚠️ 如果没有加 `-savepwd`，MCP Server 启动后会提示连接失败。
>
> Windows 上加密文件存储在：`C:\Users\你的用户名\.dbtools\`

---

### 1.4 启动 MCP Server

**基本启动：**
```cmd
sql -mcp
```

输出：
```
---------- MCP SERVER STARTUP ----------
MCP Server started successfully on Fri Jun 13 13:52:13 WEST 2025
Press Ctrl+C to stop the server
----------------------------------------
```

**指定 restrict 级别（安全加固）：**
```cmd
:: 默认 level 4（最严格），可调整为 level 3 获得更多 SQLcl 命令
sql -R 3 -mcp
```

Restrict 级别说明：

| 级别 | 阻止的命令 |
|------|-----------|
| `0` | 无限制 |
| `1` | 阻止 OS 命令（`host`、`!`、`$`） |
| `2` | 阻止文件写入（`save`、`spool`、`store`） |
| `3` | 阻止脚本执行（`@`、`@@`、`get`） |
| `4` | 阻止大量命令（默认） |

> Windows 下 MCP 使用 stdio 通信，不开网络端口，连接公司数据库走的是你本机的网络（包括 VPN）。

---

## 第二部分：配置 AI 客户端

### 2.1 Claude Desktop（Windows）

**找到配置文件：**
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

**编辑配置（Windows）：**

```json
{
  "mcpServers": {
    "oracle-sqlcl": {
      "command": "D:\\sqlcl\\bin\\sql.exe",
      "args": ["-mcp"],
      "type": "stdio"
    }
  }
}
```

> Windows 路径使用双反斜杠或正斜杠均可，JSON 中推荐双反斜杠。

**查找 sql 路径：**

在文件管理器中找到 `sql.exe` 的位置，或在命令提示符中运行 `where sql`。

**重启 Claude** 使配置生效。

---

### 2.2 Claude Code（命令行，Windows）

```cmd
# 添加 MCP 服务器
claude mcp add oracle-sqlcl D:\sqlcl\bin\sql.exe -- -mcp

# 验证是否添加成功
claude mcp list
```

或者手动创建 `.mcp.json`（项目级配置）：
```json
{
  "mcpServers": {
    "oracle-sqlcl": {
      "command": "D:\\sqlcl\\bin\\sql.exe",
      "args": ["-mcp"],
      "type": "stdio"
    }
  }
}
```

全局配置（所有项目生效），编辑 `~/.claude.json`：
```json
{
  "mcpServers": {
    "oracle-sqlcl": {
      "command": "D:\\sqlcl\\bin\\sql.exe",
      "args": ["-mcp"],
      "type": "stdio"
    }
  }
}
```

> Windows 下建议使用完整绝对路径。

---

---

## 第三部分：MCP 工具一览

SQLcl MCP Server 暴露以下 5 个工具：

| 工具 | 功能 |
|------|------|
| `list-connections` | 列出所有已保存的数据库连接 |
| `connect` | 连接到指定数据库 |
| `disconnect` | 断开当前连接 |
| `run-sql` | 执行 SQL 查询和 PL/SQL 代码块 |
| `run-sqlcl` | 执行 SQLcl 特有命令 |

**典型交互流程：**
```
1. list-connections       →  发现已保存的连接
2. connect("my_orcl")     →  建立数据库会话
3. run-sql("SELECT ...")  →  执行 SQL 进行分析
4. disconnect            →  结束会话
```

---

## 第四部分：AI 辅助 SQL 诊断分析

### 4.1 基本诊断流程

当 AI 通过 MCP 连接到 Oracle 后，可以执行以下诊断步骤：

**Step 1：查看当前 Schema 的表和索引**
```sql
SELECT table_name, num_rows, last_analyzed
FROM dba_tables
WHERE owner = 'YOUR_SCHEMA'
ORDER BY num_rows DESC;
```

**Step 2：查找全表扫描的 SQL**
```sql
SELECT s.sql_id, s.executions, p.object_name, p.operation, p.options,
       s.disk_reads / NULLIF(s.executions, 0) AS reads_per_exec
FROM v$sql s
JOIN v$sql_plan p ON s.sql_id = p.sql_id AND s.child_number = p.child_number
WHERE p.operation = 'TABLE ACCESS'
  AND p.options   = 'FULL'
  AND p.object_name NOT IN ('DUAL')
ORDER BY reads_per_exec DESC
FETCH FIRST 20 ROWS ONLY;
```

**Step 3：获取具体 SQL 的执行计划**
```sql
SELECT * FROM TABLE(
  DBMS_XPLAN.DISPLAY_CURSOR('sql_id_here', NULL, 'ALLSTATS LAST')
);
```

**Step 4：检查统计信息是否过期**
```sql
SELECT owner, table_name, num_rows, blocks, last_analyzed, stale_stats
FROM dba_tab_statistics
WHERE owner = 'YOUR_SCHEMA'
  AND stale_stats = 'YES'
FETCH FIRST 20 ROWS ONLY;
```

**Step 5：调用 SQL Tuning Advisor 自动分析**
```sql
DECLARE
  l_task_name VARCHAR2(100);
BEGIN
  l_task_name := DBMS_SQLTUNE.CREATE_TUNING_TASK(
    sql_id      => 'your_sql_id',
    scope       => DBMS_SQLTUNE.SCOPE_COMPREHENSIVE,
    time_limit  => 60,
    task_name   => 'auto_tune_task'
  );
  DBMS_SQLTUNE.EXECUTE_TUNING_TASK(task_name => l_task_name);
END;
/

SELECT DBMS_SQLTUNE.REPORT_TUNING_TASK('auto_tune_task') FROM dual;
```

---

### 4.2 AI 自动分析示例对话

```
用户：帮我分析这条 SQL 为什么慢
SQL: SELECT * FROM orders o, customers c
     WHERE o.customer_id = c.customer_id
     AND o.status = 'PENDING'
     AND c.region = 'EAST'
     ORDER BY o.created_date DESC;

AI（通过 MCP 执行）：
1. 先 EXPLAIN PLAN 看执行计划
2. 检查 ORDERS 和 CUSTOMERS 的统计信息
3. 检查相关索引是否存在
4. 发现 E-Rows 和 A-Rows 差距大
5. 给出优化建议
```

---

### 4.3 自动分析的范围与局限

| AI + MCP 能自动完成 | 需要人工确认后执行 |
|-------------------|-----------------|
| 生成 EXPLAIN PLAN | CREATE INDEX（高风险 DDL） |
| 调用 SQL Tuning Advisor | ACCEPT_SQL_PROFILE |
| 检查统计信息过期 | SPM baseline 的 ACCEPT |
| 查找缺失索引 | 改写 SQL 逻辑 |
| 分析连接方法和顺序 | 生产环境执行 |

## 第六部分：安全注意事项

### 最小权限原则

为 MCP 连接创建一个只读专用账户：

```sql
CREATE USER mcp_reader IDENTIFIED BY "StrongPwd123!";
GRANT CREATE SESSION TO mcp_reader;
GRANT SELECT ANY DICTIONARY TO mcp_reader;
-- 按需授予特定表的 SELECT 权限
-- 12c 不支持 DBA_AUTO_INDEX_* 视图，无需授权
```

> ⚠️ MCP 工具的权限完全取决于数据库用户权限。**绝对不要**用 DBA 账户连接 MCP。

### AI 生成的 SQL 安全审计

所有通过 MCP 执行的 SQL 都会被自动加上 LLM 标识注释：

```sql
/* LLM in use is [model-name] */ SELECT ...
```

此标记会出现在 `V$SQL` 和 AWR 报告中，方便审计和追溯。

---

## 相关技能文件

| 文件 | 说明 |
|------|------|
| [sqlcl-mcp-server.md](sqlcl-mcp-server.md) | SQLcl MCP Server 官方完整文档 |
| [sqlcl-basics.md](sqlcl-basics.md) | SQLcl 基础入门 |
| [explain-plan.md](../performance/explain-plan.md) | 执行计划深度解读 |
| [sql-tuning.md](../sql-dev/sql-tuning.md) | SQL 调优完整指南 |
| [optimizer-stats.md](../performance/optimizer-stats.md) | 统计信息管理 |
| [index-strategy.md](../performance/index-strategy.md) | 索引策略（含 Automatic Indexing 说明，19c+） |

---

## 附录：快速命令速查

```cmd
# ========== MCP 启动（Windows）==========
sql -mcp                    # 默认 restrict level 4
sql -R 3 -mcp               # 自定义 restrict level

# ========== AI 客户端配置后验证 ==========
claude mcp list             # 验证 MCP 服务器是否注册成功

# ========== MCP 连接后常用诊断 SQL ==========
-- 1. 查看 Top SQL（全表扫描）
SELECT sql_id, executions, buffer_gets, disk_reads,
       ROUND(elapsed_time/1000000,2) AS elapsed_sec
FROM v$sql
WHERE parsing_schema_name = 'YOUR_SCHEMA'
  AND command_type = 3  -- SELECT only
ORDER BY buffer_gets DESC
FETCH FIRST 20 ROWS ONLY;

-- 2. 获取执行计划
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR('sql_id', NULL, 'ALLSTATS LAST'));

-- 3. 检查统计信息过期
SELECT owner, table_name, stale_stats, last_analyzed
FROM dba_tab_statistics
WHERE owner = 'YOUR_SCHEMA'
  AND stale_stats = 'YES';

-- 4. 创建调优任务
DECLARE
  l_task VARCHAR2(100);
BEGIN
  l_task := DBMS_SQLTUNE.CREATE_TUNING_TASK(
    sql_id => 'your_sql_id',
    time_limit => 60,
    task_name => 'tune_001'
  );
  DBMS_SQLTUNE.EXECUTE_TUNING_TASK(l_task);
END;
/
SELECT DBMS_SQLTUNE.REPORT_TUNING_TASK('tune_001') FROM dual;

-- 5. 加载 SPM baseline（12c/11g 支持）
DECLARE
  l_count PLS_INTEGER;
BEGIN
  l_count := DBMS_SPM.LOAD_PLANS_FROM_CURSOR_CACHE(sql_id => 'your_sql_id');
END;
/
```

---

## Sources

- [Using the Oracle SQLcl MCP Server](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/25.4/sqcug/using-oracle-sqlcl-mcp-server.html)
- [Oracle SQL Tuning Advisor](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgsql/sql-tuning-advisor.html)
- [Automatic Indexing in Oracle 19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/auto-indexing.html)
- [DBMS_SPM — SQL Plan Management](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_SPM.html)
- [DBMS_AUTO_INDEX](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_AUTO_INDEX.html)
