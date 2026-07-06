---
title: DataOps 与数据 CI/CD：dbt、SQLMesh 如何回答「改了表要重跑哪些」
date: 2026-06-12
---

先立个论点: **DataOps 真正的难点不在流程，而在「数据即状态」。** 交付物是持续变化的数据集，而不是可随时重建的代码，这让 DevOps 那套「改代码、跑测试、安全发布」的方法论在数据链路上集体失灵。

先对齐两个定义:

- **DevOps**: 一套旨在缩短从提交系统变更到将变更投入正常生产的时间，同时确保高质量的实践。[^1]
- **DataOps**: 数据生产者与数据消费者紧密合作，通过短暂且快速的部署周期，设计、开发、部署、观察和维护与消费者不断变化的需求和业务目标紧密匹配的新数据产品。[^2]

两者的差距，本质来自 Data Engineering 相较 Software Engineering 多出来的复杂度:

- **数据即状态**: 交付产物不是 SQL 代码，而是庞大且持续变化的数据集。代码可以随时重建，数据不能。
- **数据、代码、计算环境三者相互依赖**: 不只要管代码，还要管「这段代码 + 这份数据 + 这个环境」的组合；开发期所做的假设必须在生产环境中持续被验证。[^3]
- **数据链路的复杂性**: 改一张表会沿下游扩散出破坏性/非破坏性变更，需要字段级血缘才能判断影响半径。
- **复杂的系统集成**: 摄入侧涉及数据库、外部 API；离线链路牵扯 Airflow、YARN、Spark、Hive、S3 等一堆组件，任一环节都可能成为发布风险点。

正因如此，许多公司在数据 CI/CD 上做得很挣扎，原因集中在三点: [^4]

- 缺乏标准实践
- DevOps 专业知识有限
- 供应商能力有限

# 为什么数据 CI/CD 这么难

把场景具体化就能看清问题。假设只考虑日增、暂不考虑回填: 你在开发分支上改了链路中间的几张表，那么——

**开发环境和生产环境，到底需要重跑哪些表？**

这个看似简单的问题，正是数据 CI/CD 的核心矛盾。它同时牵扯四件事:

1. **变更影响分析**: 哪些下游会受影响？
2. **breaking 判定**: 这个改动是破坏性的（下游必须重算）还是非破坏性的（下游可复用旧数据）？
3. **重跑/回填成本**: 在大型数据集上，把一张表多跑几次就是真金白银。
4. **环境隔离**: 开发、预发、生产之间如何复用数据，而不是各自跑一遍。

下面的工具对比，核心就是看它们各自把上面这四条解决到什么程度。

# 现有工具对比

## [dbt](https://github.com/dbt-labs/dbt-core)

- **定位**: modern data stack 里 ELT 的 Transformation 层，支持模块化、测试、版本化、CI/CD、文档、血缘。
- **机制**: 代码仓库遵循 [Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow)（主分支 = 生产、可选 staging 分支、特性开发分支）。核心命令 `dbt build --select state:modified+` 会比较生产与开发环境，只针对被修改过的节点及其下游进行构建与测试，[CI/CD 详解见官方 guide](https://docs.getdbt.com/guides/set-up-ci?step=1)。
- **生态**: [dbt Project Evaluator](https://docs.getdbt.com/blog/align-with-dbt-project-evaluator?version=2.0&name=Fusion)（项目规范检查）、[SQLFluff](https://docs.sqlfluff.com/en/stable/)（SQL lint）。
- **短板**: `state` selector 是粗粒度的（「改了就连下游一起算」），即便用 [Defer](https://docs.getdbt.com/reference/node-selection/defer?version=2.0&name=Fusion) 复用上游，单张表在开发/生产里仍会被重复跑多次。这正是下面 SQLMesh 要解决的问题。

## [SQLMesh](https://github.com/SQLMesh/sqlmesh)

有人叫它 dbt 2.0。要理解它，先看它针对 dbt 的痛点开了什么药。

**dbt 的问题**: 回到上面那个「改了中间表要重跑哪些」的问题，dbt 通过 [state node selector](https://docs.getdbt.com/reference/node-selection/methods?version=2.0&name=Fusion#state) **粗略地**识别受影响范围，用户可以用 [Defer](https://docs.getdbt.com/reference/node-selection/defer?version=2.0&name=Fusion) 复用生产环境的上游表，避免在开发环境重跑上游。但即便如此，单张表在开发、生产环境里仍会被跑多次——这在大型数据集上是不可接受的。[^5]

**SQLMesh 的答案**: 受 Terraform 启发，用 **Virtual Data Environments** 对 model/表的状态做统一管理，再配合**列级血缘 + 变更影响分析**，做到两件事:

1. 判定一次改动是否 breaking，从而实现**任务级 skipping**（不该重跑的就不跑）；
2. 统一管理开发/预发/生产环境，让它们共享同一份物理数据。

除了这套核心状态管理，SQLMesh 还补齐了工程化体验:

1. **开发者体验**: VS Code 插件提供自动补全、悬停摘要；Linter 基于数据血缘检测错误 SQL；并提供 CLI 与 Notebook 入口。
2. **SQL 改进**: 用 macro 取代 dbt 的 Jinja 模板。
3. **CI/CD Bot**: 官方文档讲得一般，更推荐 [dagctl 的 blog](https://dagctl.io/blog/running-sqlmesh-in-production/#environment-strategy-and-deployment)。
4. **SQL 转译**: 基于 SQLGlot 实现跨方言。

**缺点**: 现有 dbt 项目无法无缝迁移，主要卡在 Virtual Data Environments 这层的实现不够灵活。

### Virtual Data Environments 的具体实现

它把环境管理拆成三层，全部映射到数据库原生对象上:

- **Physical Layer**: 用户不直接接触，落地为数据库的 **Table**。SQLMesh 计算 Fingerprint 作为 snapshot 版本号，并把它作为表名后缀。
- **Virtual Layer**: 对外暴露层，落地为 **View**。SQLMesh 通过改变 View 指向的 Table 版本来「更新」model——这就是为什么切换/发布几乎是零成本的。
- **Environments**: 落地为数据库的 **Database**，环境名作为库名后缀。

![virtual](https://cdn.prod.website-files.com/67f7cdf0feddc96ca194ff33/67f7cdf0feddc96ca195018d_virtual_envs_end_to_end.png)

### backfill 与 breaking 判定

这是 SQLMesh 相较 dbt 真正拉开差距的地方。

SQLMesh 用 model 的 [Fingerprint](https://sqlmesh.readthedocs.io/en/stable/concepts/architecture/snapshots/) 判断已有物理表能否复用，还是需要 backfill（回填重算）。当一个被直接修改的 model 在 [plan](https://sqlmesh.readthedocs.io/en/stable/concepts/plans/) 中被分类时:

- 判为 **non-breaking**（如新增一列）: 只回填该 model 自身，下游不回填、复用已有数据。
- 判为 **breaking**: 该 model 连同其下游一并回填。

对比 dbt: dbt 的 state selector 是粗粒度的，而 SQLMesh 借列级血缘把「改动是否破坏下游」判定到位，从而实现任务级 skipping，避免大表被无意义地重跑。

## [dbt-state](https://github.com/dbt-labs/dbt-state)

- **定位**: 2026 年 5 月底的新项目，目前还在 preview 阶段，且是收费功能。
- **机制**: 提供类似 dbt 与 SQLMesh 的状态管理，可替换 dbt 中的 `--defer`、`--state`。
- **判断**: 方向上是 dbt 官方对自身 state 粗粒度问题的正面回应，但成熟度与成本目前都还不明朗。

## [Databricks Repos](https://www.databricks.com/product/repos)

- **定位**: 比较通用的 Git 集成产品，把 Git 工作流搬进 Databricks，没有专门针对 Data Engineering 的变更/状态能力，也别指望它解决 breaking 判定。

## 小结对比

| 维度 | dbt Core | dbt-state | SQLMesh | Databricks Repos |
| --- | --- | --- | --- | --- |
| 变更识别 | state node selector（粗粒度） | 类 defer/state 的状态管理 | fingerprint + 列级血缘 | 无专用能力 |
| 任务跳过 | 受限，单表常多次重跑 | 改进 defer/state | 任务级 skipping | - |
| breaking 判定 | 无 | 无 | non-breaking 只回填自身，breaking 连下游 | - |
| 环境管理 | 手动/约定 | - | Virtual Data Environments | Git 仓库工作流 |
| 开发体验 | Jinja，CLI | 沿用 dbt 生态 | VS Code 插件/Linter/Notebook，macro | Notebook/Git UI |
| 成熟度/成本 | 开源成熟 | 2026 预览、收费 | 开源，迁移成本高 | 通用、非 DE 专用 |

# 选型建议

- **已有成熟 dbt 项目、暂不想迁移**: 留在 dbt，用 `state:modified+` + Defer 控制重跑，必要时关注 dbt-state 的正式版。
- **大型数据集、重跑成本敏感、想要任务级 skipping 和统一环境管理**: 选 SQLMesh，接受一定的迁移成本。
- **已在 Databricks 生态、主要诉求是 Git 工作流而非细粒度状态管理**: Databricks Repos 够用，但变更/状态管理仍要靠上层工具补齐。

[^1]: [DevOps - Wikipedia](https://en.wikipedia.org/wiki/DevOps)
[^2]: [DataOps guide: How to get started | dbt Labs](https://www.getdbt.com/blog/dataops-dbt-guide)
[^3]: [The Foundation of Modern DataOps with Databricks | DBSQL SME Engineering](https://medium.com/dbsql-sme-engineering/the-foundation-of-modern-dataops-with-databricks-68e36f5d72e8)
[^4]: [The Ultimate Guide to CI/CD for Data Engineering in Databricks | by Eduard Popa | Data Engineer Things](https://blog.dataengineerthings.org/the-ultimate-guide-to-ci-cd-for-data-engineering-in-databricks-fd14abdb680a)
[^5]: [Simplicity or Efficiency: How dbt Makes You Choose](https://www.tobikodata.com/blog/simplicity-or-efficiency-how-dbt-makes-you-choose)
