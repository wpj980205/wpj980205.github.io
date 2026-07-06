---
title: "Data Agent系列: Semantic Layer篇"
date: 2026-07-01
---

# overview
- Data Agent系列，旨在了解学术界/工业界/开源界的最新现状。
- 本文回答为什么Data Agent需要Semantic Layer？dbt与cube的看法又是怎样？

# TL;DR
- dbt Semantic Layer并非银弹
- 前期需要投入资源设计语义层
- 在schema简单、question复杂（治理良好、星型/星座范式）的情况下，表现非常优秀
- 若schema复杂，有多join（3NF等范式），dbt Semantic Layer的效果不如Text-to-SQL，但可以明确无法回答，而不是给出错误回答


# dbt's approach
## 231114 [data.world](https://data.world/): [A Benchmark to Understand the Role of Knowledge Graphs on Large Language Model's Accuracy for Question Answering on Enterprise SQL Databases](https://arxiv.org/pdf/2311.07509)
- result: 引入Knowledge Graph可大幅提升问答准确率

| Category                  | w/o KG (SQL) | w/ KG (SPARQL) | Improvement |
| ------------------------- | ------------ | -------------- | ----------- |
| All Questions             | 16.7%        | 54.2%          | 37.5%       |
| Low Question/Low Schema   | 25.5%        | 71.1%          | 45.6%       |
| High Question/Low Schema  | 37.4%        | 66.9%          | 29.5%       |
| Low Question/High Schema  | 0%           | 35.7%          | 35.7%       |
| High Question/High Schema | 0%           | 38.5%          | 38.5%       |

- benchmark设计
  - 保险行业企业级数据，包含
    - 数据库schema
    - 从报表生成到指标计算的SQL
    - 上下文层，包含ontology以及mappings，用于构建knowledge graph
  - 数据为3NF的Inmon风格
- architecture: Question/OWL Ontology -> SPARKQL Zero-shot Prompt -> GPT-4 -> SPARKQL -> Mapping/KG -> SQL -> SQL Database -> Answer

- 为什么不用现有benchmark包括Spider, WikiSQL, KaggleDBQA
  1. 问答系统在这些benchmark中已经表现出色，但这些benchmark的复杂度不够高。在企业级别几百个表的情况下问答系统的准确率未知
  2. 忽视企业运营和战略规划相关的问题，包括业务报告、指标、KPI相关的问题
  3. 缺乏业务上下文层，包括metadata, mappings, transformations, ontologies

- 相关链接
  - [LLM + KG相关研究](https://github.com/RManLuo/Awesome-LLM-KG)
  - [企业级SQL schema](https://www.omg.org/spec/PC/1.0/About-PC)
  - [SQL DDL](https://www.omg.org/cgi-bin/doc?dtc/13-04-15.ddl)
  - [benchmark](https://github.com/datadotworld/cwd-benchmark-data)

## 231126 dbt: [Semantic Layer as the Data Interface for LLMs](https://roundup.getdbt.com/p/semantic-layer-as-the-data-interface)
- dbt复现了上面这篇论文
- result: dbt Semantic Layer在"高问题复杂度/低模式复杂度"的问题上有优势

![The questions attempted by the Semantic Layer with a 100% failure rate are the ones that required too many joins.](https://substackcdn.com/image/fetch/$s_!bfVJ!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F804230bc-2125-4abe-a3f3-55da48d32700_849x130.png)

- 相关链接
  - https://github.com/dbt-labs/semantic-layer-llm-benchmarking/tree/main/hex_notebook
  - [dbt Semantic Layer LLM Benchmarking Results 2023-11-23 - Google 表格](https://docs.google.com/spreadsheets/d/1n-o99KynLkgQu0QHLwmUVYk88QTfkbWgdors2nZFEXs/edit?gid=2130643231#gid=2130643231)

## 260407 [Semantic Layer vs. Text-to-SQL: 2026 Benchmark Update](https://docs.getdbt.com/blog/semantic-layer-vs-text-to-sql-2026)
- [TL;DR](https://docs.getdbt.com/blog/semantic-layer-vs-text-to-sql-2026?version=2.0&name=Fusion#tldr)

- 相关链接
  - https://github.com/dbt-labs/dbt-llm-sl-bench
  - https://dbt-labs.github.io/dbt-llm-sl-bench/
  - https://github.com/dbt-labs/dbt-agent-skills

## 240911 [Build, centralize, and deliver consistent metrics with the dbt Semantic Layer](https://www.getdbt.com/blog/build-centralize-and-deliver-consistent-metrics-with-the-dbt-semantic-layer)
- Bring consistency to your metrics
- Meet users where they are
  - 支持JDBC, GraphQL
- Empower downstream stakeholders to get their own answers
  - Providing definitions and other context about a metric as a key part of the development process.
  - Codifying the aggregation type and underlying calculation.
  - Dynamically rendering relevant dimensions and metrics.
- Optimize direct and indirect costs
- Centralize and simplify your code

- 相关链接
  - [Hex](https://learn.hex.tech/docs)
  - [ArrowFlight SQL](https://arrow.apache.org/blog/2022/02/16/introducing-arrow-flight-sql/)
  - [GraphQL](https://www.apollographql.com/docs/intro/benefits/)
  - [dbt Explorer](https://www.getdbt.com/blog/navigate-and-understand-your-dbt-cloud-projects-with-dbt-explorer)
  - [join navigation](https://docs.getdbt.com/docs/build/join-logic)

## 260504 [How the dbt Semantic Layer works](https://www.getdbt.com/blog/how-the-dbt-semantic-layer-works)
- dbt Semantic Layer的components
  1. 用MetricFlow规范定义语义模型及指标
  2. 语义层API与集成
  3. 查询生成
- Semantic models
  1. semantic model information包括name, desciption, model ref, default time dimension
  2. entities包括name, type(primary, foreign), expr
  3. measures包括数值列
  4. dimensions包含name, type, time_granularity
- Metrics
  - simple
  - cumulative
  - ratio
  - derived
  - conversion

![query generation](https://www.getdbt.com/_next/image?url=https%3A%2F%2Fcdn.sanity.io%2Fimages%2Fwl0ndo6t%2Fmain%2Fc70dc57d4236f309a62abdd5aed49f7b393f5746-2068x880.png%3Ffit%3Dmax%26auto%3Dformat&w=3840&q=75)

- 相关链接
  - [Jaffle Shop](https://github.com/dbt-labs/jaffle_shop)

# cube's approach
## 260428 [Why semantic layers make LLM analytics reliable: a paired benchmark across three frontier models](https://cube.dev/blog/why-semantic-layers-make-llm-analytics-reliable-a-paired-benchmark-across-three-frontier-models)
- 作者基于[Cleaned Contoso Retail数据集](https://www.kaggle.com/datasets/bhanuthakurr/cleaned-contoso-dataset)，使用Opus 4.7, Sonnet 4.6, GPT-5.4模型，发现添加语义层可以带来17-23个百分点的准确率提升

- 相关链接
  - [benchmark](https://github.com/cubedevinc/semantic-layer-benchmark)
  - 260408 [Semantic Layers for Reliable LLM-Powered Data Analytics: A Paired Benchmark of Accuracy and Hallucination Across Three Frontier Models](https://arxiv.org/pdf/2604.25149)

## 250108 [The Evolution of OLAP](https://cube.dev/blog/the-evolution-of-olap)
- OLAP的演变
- OLTP -> OLAP，满足分析、多维建模、性能优化需求
- OLAP -> MPP, Hadoop -> CloudDataWarehouse，满足水平扩展，降本需求
- 分析建模工作: OLAP -> CDW + BI工具，例如Looker的LookML, PowerBI的DAX，解决CDW不适合多维分析建模的问题
- BI工具内置内存OLAP引擎，解决CDW的性能不及OLAP的问题
- 未来: 解决BI工具造成多仓重复建设的问题

## 250206 [About Query Interfaces for OLAP Systems](https://cube.dev/blog/about-query-interfaces-for-olap-systems)
- MDX -> DAX
- SQL为universal semantic layer与BI工具间的理想通信协议，处于有利位置
- 但SQL是为OLTP设计，缺乏原生的多维原语，measure, aggregate
- DAX自上而下，指标计算需要自上而下，最后一层再聚合
- 而SQL自下而上，会导致二次聚合，结果失真
- 解决方案是: rewriting queries
- cube开源语义层基于Rust语言E-Graph理论完成查询重写

## 260521 [How Semantic SQL Works](https://cube.dev/blog/how-semantic-sql-works)
- 语义层的概念在90年代就已经有了
- 为什么不用表或者SEMANTIC_LAYER.md?
  - 评估问题: SQL自下而上，而metrics是自上而下的
  - 护栏问题: 防止大模型产生错误
- [[open-semantic-interchange]]在讨论语义层的查询接口应该如何设计。有没有比SQL更好的选择？
  - MDX已死; DAX只在微软生态中; 定制REST或GraphQL API需要每个消费者学习; 新的DSL理论上更简洁, 但现有工具和模型都不会使用
  - SQL不是多维查询的理论最佳语言，但每个编程语言、大模型都支持SQL
  - 从常规SQL到语义SQL (MEASURE函数)，语法改变小，但语义意义重大
- Semantic SQL是啥？
  - 区别在于评估模型
  - MEASURE外层的第一个GROUP BY是自上而下和自下而上的边界
- [E-Graphs](https://en.wikipedia.org/wiki/E-graph)是啥？
  - 语义层的query planner需要处理三个问题
    1. 聚合感知匹配
    2. 目标方言不兼容
    3. 生成SQL优化
  - 三种策略
    1. 先flatten
    2. 先简化sub-expression
    3. 用原子quarter重写，然后flatten

- 相关链接
  - https://github.com/egraphs-good/egg


# Other
## 250825 [From Metrics to Meaning: The Evolution of the Semantic Layer in the Age of Agentic AI](https://www.tellius.com/resources/blog/from-metrics-to-meaning-the-evolution-of-the-semantic-layer-in-the-age-of-agentic-ai)
- TL;DR
  - 语义层需要演变为上下文语义层：governed, ontology, KG, 记忆, LLM编排
- 语义层的不足之处

![short](https://cdn.prod.website-files.com/67fcfe6c0c7705918e4d79b1/68ac819db791a76c89c9885e_unnamed%20-%202025-08-25T113027.123.png)

- 上下文语义层(Contextual Semantic Layers)
  - Metrics
  - Ontology
  - Memory
  - KG
  - LLM Orchestration

![CSL](https://cdn.prod.website-files.com/67fcfe6c0c7705918e4d79b1/68ac80df2ede1092d263fd26_unnamed%20-%202025-08-25T111944.453.png)

![pitfall](https://cdn.prod.website-files.com/67fcfe6c0c7705918e4d79b1/68ac8520f99a9b503693a7fd_unnamed%20-%202025-08-25T114529.835.png)

- 相关
  - [A Benchmark to Understand the Role of Knowledge Graphs on Large Language Model's Accuracy for Question Answering on Enterprise SQL Databases](https://dl.acm.org/doi/10.1145/3661304.3661901?utm_source=chatgpt.com&__cf_chl_f_tk=rU3N4xEX3iKNWG9klYOJwEIxyj_pHzghp115XszdNV4-1782905074-1.0.1.1-ZGKWIgZdUMKOgI3byM9vjL459kwTfhMglXO7lVxnBrM)

## 251106 [Semantic Layer Architectures: Warehouse vs dbt vs Cube](https://www.typedef.ai/resources/semantic-layer-architectures-explained-warehouse-native-vs-dbt-vs-cube)
- 本文讲述了主流的语义层架构，并给出选型决策

- 指标口径统一是每个数据团队会遇到的问题，行业通过OLAP cubes, BI工具语义层、指标目录来解决
- 语义层的核心挑战
  - 建模问题
  - 执行问题
  - 性能问题
  - 治理问题
- 行业三种架构模式
  - warehouse-native
    - Snowflake Cortex Semantic View
      - pros: 运维简单; 原生性能; 统一治理; 使用方便
      - cons: 云平台绑定; BI工具支持少; 没有原始版本控制; 专有语法
    - Databricks Unity Catalog Metric Views
      - pros: 统一DS, BI, ML; Unity Catalog治理; 高性能; 领域驱动组织; 运维简单
      - cons: 云平台绑定; 专有语法; 连接限制; Unity Catalog要求; 时间粒度处理; 新功能文档少
  - transformation-layer: dbt MetricFlow
    - pros: 平台无关; 基于Git的治理; 统一工作流程; dbt生态系统; 无厂商锁定
    - cons: API依赖性; 查询延迟; 有限的缓存控制; dbt Cloud成本; 学习曲线
  - olap-acceleration: Cube.dev
    - pros: 极限性能; 成本节约; 高并发; 仓库无关; 开发者友好
    - cons: 额外基础设施; 预聚合管理; 过时数据风险; 存储开销; 建模复杂度

https://cube.dev/articles/best-semantic-layer-for-ai-and-bi-2026

## 260408 [Semantic Layers for Reliable LLM-Powered Data Analytics: A Paired Benchmark of Accuracy and Hallucination Across Three Frontier Models](https://arxiv.org/pdf/2604.25149)
- introduction部分很棒

