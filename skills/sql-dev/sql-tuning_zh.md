# Oracle SQL 调优

## 概述

SQL 调优是通过理解 Oracle 基于成本优化器（CBO）如何生成和执行查询计划来提升查询性能的过程。有效的调优需要解读执行计划、理解连接方法和访问路径、管理优化器统计信息，以及掌握何时以及如何通过提示、Profile 或基线来影响优化器。

Oracle 的 CBO 会评估候选执行计划，并选择估计成本最低的方案——成本是 CPU、I/O 和内存的函数。当 CBO 做出错误选择时，通常是因为统计信息过期、缺失或具有误导性，或者查询结构阻止了高效的访问路径。

---

## 执行计划

### 生成执行计划

两种主要方法是 `EXPLAIN PLAN`（仅估算）和 `DBMS_XPLAN`（实际或估算）：

```sql
-- 仅估算计划（不执行）
EXPLAIN PLAN FOR
SELECT e.last_name, d.department_name
FROM   employees e
JOIN   departments d ON e.department_id = d.department_id
WHERE  e.salary > 10000;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY());
```

```sql
-- 通过 SQL*Plus autotrace 获取实际执行统计
SET AUTOTRACE ON
SELECT e.last_name, d.department_name
FROM   employees e
JOIN   departments d ON e.department_id = d.department_id
WHERE  e.salary > 10000;
```

```sql
-- 最佳方式：执行后从游标缓存获取实际计划
-- 首先执行查询，然后找到其 SQL_ID
SELECT sql_id, sql_text
FROM   v$sql
WHERE  sql_text LIKE '%last_name%employees%'
AND    sql_text NOT LIKE '%v$sql%';

-- 然后用实际行数获取计划
SELECT * FROM TABLE(
  DBMS_XPLAN.DISPLAY_CURSOR('abc12345xyz', 0, 'ALLSTATS LAST')
);
```

### 解读计划输出

`DBMS_XPLAN` 输出的关键列：

| 列 | 含义 |
|---|---|
| `Id` | 操作步骤编号；子步骤有缩进 |
| `Operation` | Oracle 正在执行的操作（TABLE ACCESS、INDEX RANGE SCAN 等） |
| `Name` | 被访问的对象 |
| `Rows`（E-Rows） | 估算行数——优化器的预测 |
| `A-Rows` | 实际返回的行数（需要 `ALLSTATS`） |
| `Bytes` | 估算的数据量 |
| `Cost (%CPU)` | 优化器成本估算和 CPU 占比 |
| `Time` | 估算的墙上时间 |
| `Starts` | 该步骤被执行了多少次 |

E-Rows 和 A-Rows 之间存在较大差异表明存在基数估算问题，这是大多数坏计划的根本原因。

### 常见访问路径

- **TABLE ACCESS FULL** — 读取段中的每个块。对大百分比检索高效；对高选择性查询效果差。
- **INDEX RANGE SCAN** — 在 B-tree 索引上遍历一系列值。适合有选择性的谓词。
- **INDEX UNIQUE SCAN** — 单次探查唯一索引。最有效的单行访问方式。
- **INDEX FAST FULL SCAN** — 使用多块 I/O 读取所有索引块；适用于仅索引查询。
- **INDEX SKIP SCAN** — 当前导列不在谓词中时用于复合索引。通常意味着需要更好的索引。

---

## 连接方法

Oracle 支持三种主要连接算法。优化器根据基数、可用索引和可用内存进行选择。

### 嵌套循环（NL）

当驱动行集较小且内表有良好索引时效果最佳。

```
NESTED LOOPS
  TABLE ACCESS FULL   EMPLOYEES      （驱动表，结果集小）
  INDEX RANGE SCAN    DEPT_ID_IDX    （每个驱动行执行一次内表查找）
```

- 内存需求低
- 如果驱动集很大，扩展性差
- 性能随驱动行数线性下降

### 哈希连接

最适合连接两个大集合且连接列上没有可用索引的情况。较小的表被哈希到内存中；较大的表被探测。

```
HASH JOIN
  TABLE ACCESS FULL   DEPARTMENTS    （构建端——较小的表）
  TABLE ACCESS FULL   EMPLOYEES      （探测端——较大的表）
```

- 需要 PGA 内存来存储哈希表
- 如果构建端无法放入 PGA，会溢出到临时表（很慢）
- 仅适用于等值连接

### 排序合并连接

两个输入都按连接键排序，然后合并。当输入已排序或哈希连接不可行时使用。

- 比哈希连接需要更多内存
- 如果数据已被索引预排序，可能比哈希连接更快

---

## 优化器提示（Hints）

提示是嵌入在 SQL 注释中的指令，用于指示优化器使用特定的计划元素。应谨慎使用——首先修复统计信息或索引。

### 语法

```sql
SELECT /*+ HINT_NAME(参数) */ col1 FROM table1;
```

### 常用提示

```sql
-- 强制使用索引
SELECT /*+ INDEX(e EMP_DEPT_IDX) */ *
FROM   employees e
WHERE  department_id = 30;

-- 强制全表扫描（绕过索引）
SELECT /*+ FULL(e) */ *
FROM   employees e
WHERE  status = 'A';

-- 控制连接方法
SELECT /*+ USE_NL(e d) */ e.last_name, d.department_name
FROM   employees e
JOIN   departments d ON e.department_id = d.department_id;

SELECT /*+ USE_HASH(e d) */ e.last_name, d.department_name
FROM   employees e
JOIN   departments d ON e.department_id = d.department_id;

-- 控制连接顺序（e 为第一/驱动表）
SELECT /*+ LEADING(e d) USE_NL(d) */ e.last_name, d.department_name
FROM   employees e
JOIN   departments d ON e.department_id = d.department_id;

-- 并行执行
SELECT /*+ PARALLEL(e, 4) */ COUNT(*) FROM employees e;

-- 禁用并行（OLTP 中有用）
SELECT /*+ NO_PARALLEL(e) */ * FROM employees e;

-- 基数提示（统计信息错误时）
SELECT /*+ CARDINALITY(e 100) */ *
FROM   employees e
WHERE  complex_function(salary) > 5000;
```

### 何时不使用提示

- 提示很脆弱——表或索引名称更改时会失效
- 它们绕过了优化器适应数据分布变化的能力
- 优先修复根本原因：收集更好的统计信息、添加索引或重写查询
- 使用 SQL Profile 和基线来稳定生产环境

---

## 收集和解读统计信息

Oracle 的 CBO 依赖对象统计信息。过期或缺失的统计信息是坏计划的頭号原因。

### 收集表和索引统计信息

```sql
-- 收集单个表的统计信息（所有列都生成直方图）
EXEC DBMS_STATS.GATHER_TABLE_STATS(
  ownname          => 'HR',
  tabname          => 'EMPLOYEES',
  estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE,
  method_opt       => 'FOR ALL COLUMNS SIZE AUTO',
  cascade          => TRUE   -- 同时收集索引统计
);

-- 收集整个 Schema 的统计信息
EXEC DBMS_STATS.GATHER_SCHEMA_STATS(
  ownname          => 'HR',
  estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE,
  method_opt       => 'FOR ALL COLUMNS SIZE AUTO',
  cascade          => TRUE
);
```

### 查看统计信息

```sql
-- 表级统计信息
SELECT table_name, num_rows, blocks, avg_row_len, last_analyzed
FROM   dba_tables
WHERE  owner = 'HR';

-- 列级统计信息和直方图
SELECT column_name, num_distinct, num_nulls, histogram, num_buckets,
       low_value, high_value, last_analyzed
FROM   dba_tab_col_statistics
WHERE  owner = 'HR'
AND    table_name = 'EMPLOYEES';

-- 检查过期统计信息（自上次分析后变化超过 10%）
SELECT owner, table_name, stale_stats
FROM   dba_tab_statistics
WHERE  owner = 'HR'
AND    stale_stats = 'YES';
```

### 直方图

直方图描述偏斜的列分布，对于非均匀数据的列至关重要。

```sql
-- 频率直方图（用于低基数列）
-- 当不同值数量 <= 254 时，Oracle 会自动使用 SIZE AUTO 创建这些直方图

-- 查看直方图桶
SELECT endpoint_number, endpoint_value
FROM   dba_histograms
WHERE  owner = 'HR'
AND    table_name = 'EMPLOYEES'
AND    column_name = 'DEPARTMENT_ID'
ORDER BY endpoint_number;
```

### 扩展统计信息（多列相关性）

当列相关时（例如 `city` 和 `state`），优化器可能会低估基数。扩展统计信息可以解决此问题。

```sql
-- 在列组上创建扩展统计信息
SELECT DBMS_STATS.CREATE_EXTENDED_STATS(
  ownname => 'HR',
  tabname => 'EMPLOYEES',
  extension => '(DEPARTMENT_ID, JOB_ID)'
) FROM dual;

-- 然后重新收集统计信息以填充它们
EXEC DBMS_STATS.GATHER_TABLE_STATS('HR', 'EMPLOYEES',
  method_opt => 'FOR ALL COLUMNS SIZE AUTO');
```

---

## 索引使用与策略

### 识别缺失索引

```sql
-- 从游标缓存中找出大表的全表扫描
SELECT s.sql_id, s.executions, p.object_name, p.operation, p.options,
       s.disk_reads / NULLIF(s.executions, 0) AS reads_per_exec
FROM   v$sql s
JOIN   v$sql_plan p ON s.sql_id = p.sql_id AND s.child_number = p.child_number
WHERE  p.operation = 'TABLE ACCESS'
AND    p.options   = 'FULL'
AND    p.object_name NOT IN ('DUAL')
ORDER BY reads_per_exec DESC NULLS LAST
FETCH FIRST 20 ROWS ONLY;
```

### 索引监控

```sql
-- Oracle 12.2+ — 检查索引使用统计
SELECT index_name, table_name, monitoring, used, start_monitoring, end_monitoring
FROM   v$object_usage
WHERE  table_name = 'EMPLOYEES';

-- 开始监控索引
ALTER INDEX hr.emp_name_idx MONITORING USAGE;
```

### 自动索引（19c+）

Oracle 19c+ 可以根据 SQL 工作负载分析自动识别、创建、验证和删除索引。详见 **`skills/performance/index-strategy.md` — 自动索引** 章节，了解配置（`DBMS_AUTO_INDEX`）、监控视图（`DBA_AUTO_INDEX_EXECUTIONS`、`DBA_AUTO_INDEX_IND_ACTIONS`）以及何时使用自动索引与手动索引。

### 不可见索引

在不影响生产计划的情况下测试新索引：

```sql
-- 创建不可见索引（暂不影响计划）
CREATE INDEX emp_salary_idx ON employees(salary) INVISIBLE;

-- 仅在当前会话中测试
ALTER SESSION SET optimizer_use_invisible_indexes = TRUE;
EXPLAIN PLAN FOR SELECT * FROM employees WHERE salary BETWEEN 5000 AND 8000;
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

-- 验证通过后设为可见
ALTER INDEX emp_salary_idx VISIBLE;
```

---

## SQL Profile 和 SQL Plan 基线

### SQL Profile

SQL Profile 存储特定查询的额外统计信息或校正因子，在不锁定计划的情况下改进优化器估算。

```sql
-- 使用 SQL Tuning Advisor 生成 Profile
DECLARE
  l_task_name VARCHAR2(30);
  l_sql_id    VARCHAR2(13) := 'abc12345xyz78';
BEGIN
  l_task_name := DBMS_SQLTUNE.CREATE_TUNING_TASK(
    sql_id      => l_sql_id,
    scope       => DBMS_SQLTUNE.SCOPE_COMPREHENSIVE,
    time_limit  => 60,
    task_name   => 'tune_task_1'
  );
  DBMS_SQLTUNE.EXECUTE_TUNING_TASK(task_name => l_task_name);
END;
/

-- 查看建议
SELECT DBMS_SQLTUNE.REPORT_TUNING_TASK('tune_task_1') FROM dual;

-- 如果建议接受 SQL Profile
EXEC DBMS_SQLTUNE.ACCEPT_SQL_PROFILE(
  task_name    => 'tune_task_1',
  replace      => TRUE
);
```

### SQL Plan 基线（SPM）

SQL Plan Management（SPM）捕获已知良好的计划并防止计划回退。

```sql
-- 从游标缓存捕获特定 SQL_ID 的基线
DECLARE
  l_count PLS_INTEGER;
BEGIN
  l_count := DBMS_SPM.LOAD_PLANS_FROM_CURSOR_CACHE(
    sql_id => 'abc12345xyz78'
  );
  DBMS_OUTPUT.PUT_LINE('已加载基线数: ' || l_count);
END;
/

-- 查看现有基线
SELECT sql_handle, plan_name, enabled, accepted, fixed, origin
FROM   dba_sql_plan_baselines
ORDER BY created DESC;

-- 演进未接受的计划（测试并提升更好的计划）
DECLARE
  l_report CLOB;
BEGIN
  l_report := DBMS_SPM.EVOLVE_SQL_PLAN_BASELINE(
    sql_handle => 'SQL_abc123',
    time_limit => 30,
    verify     => 'YES',
    commit     => 'YES'
  );
  DBMS_OUTPUT.PUT_LINE(l_report);
END;
/
```

---

## 最佳实践

- **始终检查实际行数 vs 估算行数。** 基数不匹配达到 10 倍或以上几乎总是会产生坏计划。
- **对易变表定期收集统计信息。** 对高 DML 表使用统计信息作业或 `DBMS_SCHEDULER` 进行夜间收集。
- **使用 `AUTO_SAMPLE_SIZE`。** 它以极低的成本提供接近 100% 的准确度，使用 Oracle 的增量统计信息算法。
- **避免在 WHERE 子句中对索引列使用函数调用。** 必要时使用函数索引。
- **参数化查询。** 字面量值会阻止计划共享并用硬解析填充共享池。
- **生产前验证计划变更。** 使用不可见索引和 SPM 基线来无风险测试。
- **避免 `SELECT *`。** 只检索需要的列；宽投影会阻止仅索引访问路径。

---

## 常见错误

| 错误 | 问题 | 修复 |
|---|---|---|
| 在索引列上调用函数：`WHERE UPPER(name) = 'SMITH'` | 使索引失效 | 创建函数索引：`CREATE INDEX ... ON t(UPPER(name))` |
| 隐式类型转换：`WHERE emp_id = '100'`（emp_id 是 NUMBER） | 阻止索引使用 | 显式匹配数据类型 |
| 批量加载后统计信息过期 | CBO 使用错误的基数 | 在大量 DML 后运行 `GATHER_TABLE_STATS` |
| 在应用代码中使用提示 | 脆弱；重命名时会失效 | 改用 SQL Profile/基线 |
| 在 `ORDER BY` 前使用 `ROWNUM` | 先返回无序行再过滤 | 使用 `FETCH FIRST N ROWS ONLY` 或子查询 |
| 过度索引 | 减慢 DML；浪费空间 | 监控索引使用情况；删除未使用的索引 |
| 忽略 `TEMP` 表空间溢出 | 哈希/排序溢出会降低性能 | 增加 PGA 或重写以减少中间结果集 |

---

## Oracle 特定说明

- `OPTIMIZER_ADAPTIVE_PLANS` 参数（12c+ 默认 ON）允许 Oracle 根据实际行数在执行过程中切换连接方法。这有帮助，但也可能导致令人惊讶的计划变更。
- `OPTIMIZER_STATISTICS_ADVISOR`（12.2+）在维护窗口期间自动运行，并推荐统计信息收集的变更。
- 在 Exadata 上，计划中的 `CELL_OFFLOAD_PROCESSING` 表示 Smart Scan 已激活——由于存储单元处理，Exadata 上的全表扫描可能非常快。
- `/*+ GATHER_PLAN_STATISTICS */` 提示无需 autotrace 即可向任何查询添加 `A-Rows` 和 `A-Time`——在应用代码测试中很有用。

---

## Oracle 版本说明（19c vs 26ai）

- 本文件中的基线指导适用于 Oracle Database 19c，除非明确标注了更高的最低版本要求。
- 标记为 21c、23c 或 23ai 的功能应视为 Oracle Database 26ai 可用功能；对于混合版本环境，保留 19c 兼容的替代方案。
- 对于双支持环境，请在 19c 和 26ai 中都测试语法和包行为，因为默认设置和弃用在不同版本更新中可能有所不同。

## 另请参阅

- [Explain Plan — 执行计划分析](../performance/explain-plan.md) — 解读和理解执行计划

## 参考来源

- [Oracle Database 19c SQL 调优指南 (TGSQL)](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgsql/)
- [Oracle Database 19c SQL 语言参考 (SQLRF)](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/)
- [DBMS_XPLAN — Oracle Database 19c PL/SQL 包和类型参考](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_XPLAN.html)
- [DBMS_SQLTUNE — Oracle Database 19c PL/SQL 包和类型参考](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_SQLTUNE.html)
- [DBMS_SPM — Oracle Database 19c PL/SQL 包和类型参考](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_SPM.html)
