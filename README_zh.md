# Oracle DB Skills

**仓库地址：** https://github.com/krisrice/oracle-db-skills
**版本：** 1.0.0

Oracle DB Skills 是一个精心整理的 Oracle 数据库实践指南库，包含 100+ 篇有据可查的操作指南，涵盖：SQL 和 PL/SQL 开发、性能调优、安全、管理、监控、架构、DevOps、迁移、SQLcl、ORDS 及 Oracle 特有功能。每篇指南都包含可操作的示例、最佳实践、常见陷阱、出处以及明确的 19c 和 26ai 版本兼容性说明。

## 版本覆盖标准

- 涉及版本特定行为的技能文件必须包含名为 `## Oracle Version Notes (19c vs 26ai)` 的章节。
- 以 Oracle Database 19c 作为基线兼容目标，除非另有说明。
- 明确标注需要更高版本的功能，并在实际情况下提供 19c 兼容的替代方案。

## GitHub 规则集

- 默认分支规则集定义存储在 `.github/rulesets/main.json` 中。
- 应用规则集命令：
  - `export GITHUB_TOKEN=<具有仓库管理权限的令牌>`
  - `./scripts/apply-github-ruleset.sh krisrice oracle-db-skills`

---

## 分类概览

| 分类 | 文件数 | 路径 |
|------|--------|------|
| [数据库设计与建模](#数据库设计与建模) | 4 | `skills/design/` |
| [SQL 开发](#sql开发) | 5 | `skills/sql-dev/` |
| [性能与调优](#性能与调优) | 7 | `skills/performance/` |
| [应用开发](#应用开发) | 14 | `skills/appdev/` |
| [安全](#安全) | 6 | `skills/security/` |
| [管理](#管理) | 6 | `skills/admin/` |
| [监控与诊断](#监控与诊断) | 5 | `skills/monitoring/` |
| [架构与基础设施](#架构与基础设施) | 5 | `skills/architecture/` |
| [DevOps 与 CI/CD](#devops-与-cicd) | 5 | `skills/devops/` |
| [迁移到 Oracle](#迁移到-oracle) | 14 | `skills/migrations/` |
| [PL/SQL 开发](#plsql开发) | 12 | `skills/plsql/` |
| [Oracle 特有功能](#oracle特有功能) | 6 | `skills/features/` |
| [SQLcl](#sqlcl) | 8 | `skills/sqlcl/` |
| [ORDS (Oracle REST Data Services)](#ords-oracle-rest-data-services) | 10 | `skills/ords/` |
| [框架](#框架) | 9 | `skills/frameworks/` |

---

## 数据库设计与建模

`skills/design/`

| 文件 | 说明 |
|------|------|
| `erd-design.md` | 实体关系设计、规范化（1NF–5NF）、Oracle 命名约定、保留字 |
| `data-modeling.md` | 逻辑建模 vs 物理建模、星型/雪花型 schema、ODS、SCD 类型 |
| `partitioning-strategy.md` | 范围、列表、哈希、复合分区，分区裁剪，本地索引 vs 全局索引 |
| `tablespace-design.md` | 容量规划、bigfile vs smallfile、ASSM vs MSSM、生产环境布局模式 |

---

## SQL开发

`skills/sql-dev/`

| 文件 | 说明 |
|------|------|
| `sql-tuning.md` | 执行计划、优化器提示、SQL Profile、计划基线 |
| `sql-injection-avoidance.md` | 绑定变量、DBMS_ASSERT、安全动态 SQL 模式 |
| `pl-sql-best-practices.md` | BULK COLLECT/FORALL、异常处理、游标管理、包结构 |
| `sql-patterns.md` | 窗口函数、CTE、CONNECT BY、PIVOT/UNPIVOT、MERGE、MODEL 子句 |
| `dynamic-sql.md` | EXECUTE IMMEDIATE、DBMS_SQL、一次解析多次执行、注入防护 |

---

## 性能与调优

`skills/performance/`

| 文件 | 说明 |
|------|------|
| `awr-reports.md` | AWR 报告生成与解读、关键章节、基线、瓶颈定位 |
| `ash-analysis.md` | 活动会话历史，实时 vs 历史分析，ASH 报告生成 |
| `explain-plan.md` | DBMS_XPLAN、解读执行计划、autotrace、识别坏计划 |
| `index-strategy.md` | B-tree、位图、函数索引、复合索引、不可见索引；重建 vs 合并 |
| `optimizer-stats.md` | DBMS_STATS、直方图、扩展统计信息、待发布统计、增量统计 |
| `wait-events.md` | 常见等待事件、诊断查询、各类事件的处理方法 |
| `memory-tuning.md` | SGA 组件、PGA 管理、AMM vs ASMM、 advisory 视图 |

---

## 应用开发

`skills/appdev/`

| 文件 | 说明 |
|------|------|
| `connection-pooling.md` | UCP、DRCP、池大小调整、连接验证、JDBC/Python/Node.js 示例 |
| `transaction-management.md` | ACID 特性、保存点、自治事务、分布式事务 |
| `locking-concurrency.md` | MVCC、SELECT FOR UPDATE、NOWAIT/SKIP LOCKED、死锁避免 |
| `sequences-identity.md` | 序列缓存、identity 列、UUID 替代方案、间隙行为 |
| `json-in-oracle.md` | 原生 JSON 类型、JSON_VALUE/QUERY/TABLE、JSON Duality Views (23c) |
| `xml-in-oracle.md` | XMLType 存储、XQuery、XMLTable、XML 索引、XMLDB 仓库 |
| `spatial-data.md` | SDO_GEOMETRY、空间索引、SDO_RELATE、坐标系统 |
| `oracle-text.md` | CONTEXT/CTXCAT 索引、CONTAINS、模糊/词干搜索、HIGHLIGHT/SNIPPET |
| `sql-property-graph.md` | SQL 属性图 DDL、GRAPH_TABLE、MATCH 模式、量化路径 (23ai+) |
| `python-oracledb.md` | python-oracledb 驱动、thin/thick 模式、绑定变量、池化、异步 |
| `java-oracle-jdbc.md` | JDBC thin 驱动、UCP、PreparedStatement、批量操作、Spring Boot |
| `nodejs-oracledb.md` | node-oracledb 驱动、async/await、池化、结果集、LOB |
| `dotnet-oracle.md` | ODP.NET 托管驱动、EF Core、数组绑定、OracleParameter |
| `golang-oracle.md` | godror 驱动、database/sql 接口、命名绑定、REF CURSOR |

---

## 安全

`skills/security/`

| 文件 | 说明 |
|------|------|
| `privilege-management.md` | 最小权限、角色、DBMS_PRIVILEGE_CAPTURE、避免 PUBLIC 授权 |
| `row-level-security.md` | VPD/FGAC、DBMS_RLS、应用上下文、所有策略类型 |
| `data-masking.md` | Oracle Data Redaction (DBMS_REDACT)、完全/部分/正则/随机脱敏 |
| `auditing.md` | 统一审计、CREATE AUDIT POLICY、细粒度审计 (DBMS_FGA) |
| `encryption.md` | TDE、Oracle Wallet 配置、表空间/列加密、密钥轮换 |
| `network-security.md` | SSL/TLS、sqlnet.ora 加密、网络包 ACL、监听器加固 |

---

## 管理

`skills/admin/`

| 文件 | 说明 |
|------|------|
| `backup-recovery.md` | RMAN 架构、备份集 vs 镜像副本、增量备份、恢复场景 |
| `dataguard.md` | 物理/逻辑备用库、Data Guard Broker、切换 vs 故障切换、Active Data Guard |
| `rman-basics.md` | 常用 RMAN 命令、通道配置、压缩、加密、报告 |
| `undo-management.md` | Undo 大小、UNDO_RETENTION、ORA-01555 原因与预防、Undo Advisor |
| `redo-log-management.md` | 日志大小、archivelog 模式、多路复用、切换频率监控 |
| `user-management.md` | CREATE USER、profile、密码策略、代理认证、CDB/PDB 用户 |

---

## 监控与诊断

`skills/monitoring/`

| 文件 | 说明 |
|------|------|
| `alert-log-analysis.md` | 告警日志位置、关键 ORA- 错误、自动监控模式 |
| `adrci-usage.md` | ADR 仓库、adrci 命令、IPS 打包、事件关联 |
| `health-monitor.md` | DBMS_HM 健康检查、SQL Tuning Advisor、Segment Advisor、Memory Advisor |
| `space-management.md` | 表空间监控、HWM、SHRINK SPACE vs MOVE、LOB 空间、临时空间 |
| `top-sql-queries.md` | V$SQL/V$SQLAREA、按资源统计的 Top SQL、AWR Top SQL、V$SQL_MONITOR |

---

## 架构与基础设施

`skills/architecture/`

| 文件 | 说明 |
|------|------|
| `rac-concepts.md` | Cache Fusion、GCS/GES、服务、节点亲和性、RAC 等待事件、TAF/FCF |
| `multitenant.md` | CDB/PDB 架构、克隆、插入/拔出、资源管理、应用容器 |
| `oracle-cloud-oci.md` | ATP、ADW、Base Database Service、ExaCS、连接方式、免费套餐 |
| `exadata-features.md` | Smart Scan、存储索引、HCC 压缩、IORM、offload 监控 |
| `inmemory-column-store.md` | IMCS 架构、对象填充、Join Groups、In-Memory 聚合、AIM |

---

## DevOps 与 CI/CD

`skills/devops/`

| 文件 | 说明 |
|------|------|
| `schema-migrations.md` | Liquibase 和 Flyway 与 Oracle、版本化 vs 可重复迁移、CI/CD 流水线 |
| `online-operations.md` | DBMS_REDEFINITION、在线索引重建/创建、ALTER TABLE ONLINE |
| `edition-based-redefinition.md` | EBR 零停机部署、editioning 视图、跨 edition 触发器 |
| `database-testing.md` | utPLSQL 框架、断言、模拟、代码覆盖率、GitHub Actions 集成 |
| `version-control-sql.md` | DBMS_METADATA DDL 提取、git 结构、漂移检测、幂等授权 |

---

## 迁移到 Oracle

`skills/migrations/`

| 文件 | 说明 |
|------|------|
| `migrate-postgres-to-oracle.md` | 数据类型映射、SQL 语法差异、SERIAL→identity、psql vs sqlplus |
| `migrate-mysql-to-oracle.md` | AUTO_INCREMENT、LIMIT→FETCH、存储过程转换、mysqldump 到 Oracle |
| `migrate-redshift-to-oracle.md` | MPP vs Oracle、分布/排序键、COPY 命令、WLM→Resource Manager |
| `migrate-sqlserver-to-oracle.md` | T-SQL→PL/SQL、TRY/CATCH→EXCEPTION、链接服务器→DBLink、SSMA 指南 |
| `migrate-db2-to-oracle.md` | DB2 SQL 语法、REORG→MOVE、RUNSTATS→DBMS_STATS、LOCATE vs INSTR |
| `migrate-sqlite-to-oracle.md` | 类型亲和性、AUTOINCREMENT、pragmas、从嵌入式到企业级扩展 |
| `migrate-mongodb-to-oracle.md` | 文档→关系型、JSON Duality Views、聚合管道→SQL |
| `migrate-snowflake-to-oracle.md` | VARIANT/OBJECT→JSON、QUALIFY→窗口函数、Time Travel→Flashback |
| `migrate-teradata-to-oracle.md` | BTEQ→SQL*Plus、多集表、QUALIFY、TPT→SQL*Loader |
| `migrate-sybase-to-oracle.md` | 链式/非链式事务、RAISERROR→RAISE_APPLICATION_ERROR、BCP→SQL*Loader |
| `oracle-migration-tools.md` | SQL Developer Migration Workbench、AWS SCT、ora2pg、Oracle ZDM、GoldenGate |
| `migration-assessment.md` | 迁移前检查清单、复杂度评分、风险矩阵、工作量估算 |
| `migration-data-validation.md` | 行数、ORA_HASH 指纹识别、对账报告、漂移检测 |
| `migration-cutover-strategy.md` | 切换阶段、并行运行、go/no-go 标准、回滚计划、干系人沟通 |

---

## PL/SQL开发

`skills/plsql/`

| 文件 | 说明 |
|------|------|
| `plsql-package-design.md` | 规范 vs 主体、公开/私有 API、初始化块、ACCESSIBLE BY、重载 |
| `plsql-error-handling.md` | 异常层次结构、PRAGMA EXCEPTION_INIT、FORMAT_ERROR_BACKTRACE、自治日志 |
| `plsql-performance.md` | 上下文切换、BULK COLLECT/FORALL、管道函数、RESULT_CACHE、PRAGMA UDF |
| `plsql-collections.md` | 关联数组、嵌套表、VARRAY、集合方法、SQL 中的 TABLE() |
| `plsql-cursors.md` | 隐式/显式游标、游标 FOR 循环、REF CURSOR、SYS_REFCURSOR、泄漏防护 |
| `plsql-dynamic-sql.md` | EXECUTE IMMEDIATE、DBMS_SQL、一次解析多次执行、注入防护 |
| `plsql-security.md` | AUTHID DEFINER vs CURRENT_USER、注入向量、DBMS_ASSERT、安全编码检查清单 |
| `plsql-debugging.md` | DBMS_OUTPUT、DBMS_APPLICATION_INFO、SQL Developer 调试器、PLSQL_WARNINGS、DBMS_TRACE |
| `plsql-unit-testing.md` | utPLSQL、测试包、断言、模拟、CI 集成、代码覆盖率 |
| `plsql-patterns.md` | TAPI 模式、自治事务日志、管道函数、对象类型 |
| `plsql-compiler-options.md` | PLSQL_OPTIMIZE_LEVEL、本机编译 vs 解释执行、条件编译、PLSQL_CCFLAGS |
| `plsql-code-quality.md` | 命名约定、Trivadis 指南、反模式、审查检查清单、PL/SQL Cop |

---

## Oracle特有功能

`skills/features/`

| 文件 | 说明 |
|------|------|
| `advanced-queuing.md` | AQ/事务事件队列、DBMS_AQ/DBMS_AQADM、传播、JMS、TEQ (21c) |
| `dbms-scheduler.md` | 作业、调度、链、事件驱动调度、窗口、监控 |
| `virtual-columns.md` | GENERATED ALWAYS AS、虚拟列索引、分区键、限制 |
| `materialized-views.md` | COMPLETE/FAST/FORCE 刷新、ON COMMIT、MV 日志、查询重写 |
| `database-links.md` | 固定/连接/共享链接、分布式 DML、两阶段提交、安全风险 |
| `oracle-apex.md` | APEX 架构、认证、ORDS 集成、REST API、CI/CD 部署 |

---

## SQLcl

`skills/sqlcl/`

| 文件 | 说明 |
|------|------|
| `sqlcl-basics.md` | 安装、连接（TNS/Easy Connect/wallet）、与 SQL*Plus 的主要区别 |
| `sqlcl-scripting.md` | JavaScript 引擎（Nashorn/GraalVM）、script 命令、Java 互操作、自动化示例 |
| `sqlcl-liquibase.md` | 内置 Liquibase、lb generate-schema、lb update/rollback、CI/CD 集成 |
| `sqlcl-formatting.md` | SET SQLFORMAT 模式（CSV、JSON、XML、INSERT、LOADER）、COLUMN、SPOOL |
| `sqlcl-ddl-generation.md` | DDL 命令、隐藏存储子句、全 schema 提取、版本控制 |
| `sqlcl-data-loading.md` | LOAD 命令加载 CSV/JSON、列映射、日期格式、错误处理 |
| `sqlcl-cicd.md` | 无头/非交互模式、退出码、wallet 连接、GitHub Actions/GitLab CI |
| `sqlcl-mcp-server.md` | MCP 服务器设置、连接 Claude/AI 助手到 Oracle、可用工具、安全 |

---

## ORDS (Oracle REST Data Services)

`skills/ords/`

| 文件 | 说明 |
|------|------|
| `ords-architecture.md` | 部署模型（Jetty/Tomcat/WebLogic/OCI）、请求路由、模块层次结构 |
| `ords-installation.md` | 安装 ORDS、`ords config set`、基于 wallet 的凭证存储、ATP/ADW 的 mTLS |
| `ords-auto-rest.md` | ORDS.ENABLE_SCHEMA/OBJECT、端点模式、JSON 过滤语法、分页 |
| `ords-rest-api-design.md` | DEFINE_MODULE/TEMPLATE/HANDLER、源类型、隐式绑定参数、CRUD 示例 |
| `ords-authentication.md` | OAuth2 客户端凭证和授权码流程、JWT 验证、角色映射 |
| `ords-pl-sql-gateway.md` | 从 REST 调用 PL/SQL、REF CURSOR、APEX_JSON、错误处理、CLOB/BLOB |
| `ords-file-upload-download.md` | BLOB 上传/下载、多部分表单数据、Content-Type/Content-Disposition |
| `ords-metadata-catalog.md` | OpenAPI 3.0 生成、Swagger UI/Postman 集成、元数据视图 |
| `ords-security.md` | HTTPS 强制执行、通过 `ords config set` 配置 CORS、基于 wallet 的密钥、请求验证 |
| `ords-monitoring.md` | 日志配置、请求日志、连接池监控、错误诊断 |

---

## 框架

`skills/frameworks/`

| 文件 | 说明 |
|------|------|
| `sqlalchemy-oracle.md` | SQLAlchemy ORM/Core Oracle 方言、引擎设置、模型、序列、批量操作 |
| `django-oracle.md` | Django ORM Oracle 后端、设置、迁移、空字符串/NULL 怪癖 |
| `pandas-oracle.md` | read_sql、to_sql、分块读取、通过 executemany 批量加载、dtype 映射 |
| `spring-data-jpa-oracle.md` | Spring Data JPA + Hibernate Oracle 方言、@SequenceGenerator、本地查询、PL/SQL |
| `mybatis-oracle.md` | MyBatis mapper XML、#{} 绑定、动态 SQL、CALLABLE 语句、序列 |
| `typeorm-oracle.md` | TypeORM 实体、QueryBuilder、迁移、NestJS 集成 |
| `sequelize-oracle.md` | Sequelize 模型定义、字段映射、序列钩子、事务 |
| `dapper-oracle.md` | Dapper Query<T>、DynamicParameters、OUT 参数、多映射 |
| `gorm-oracle.md` | GORM 模型、BeforeCreate 序列钩子、作用域、事务 |

---

## 目录结构

```
oracle-db-skills/
├── README.md
├── README_zh.md            # 中文版说明文档
├── skills-index.md          # 所有文件的完整检查清单及完成状态
└── skills/
    ├── admin/               # 管理
    ├── appdev/              # 应用开发
    ├── architecture/         # 架构与基础设施
    ├── design/              # 数据库设计与建模
    ├── devops/              # DevOps 与 CI/CD
    ├── features/            # Oracle 特有功能
    ├── migrations/          # 迁移到 Oracle
    ├── monitoring/          # 监控与诊断
    ├── frameworks/          # 语言框架（SQLAlchemy、Django、Spring 等）
    ├── ords/                # Oracle REST Data Services
    ├── performance/         # 性能与调优
    ├── plsql/               # PL/SQL 开发
    ├── security/            # 安全
    ├── sql-dev/             # SQL 开发
    └── sqlcl/               # SQLcl
```
