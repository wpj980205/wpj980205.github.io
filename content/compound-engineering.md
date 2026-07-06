---
title: Compound Engineering如何积攒复利，让后续工作更简单？
date: 2026-06-26
---

# reference
- [Guide - Compound Engineering - Every](https://every.to/guides/compound-engineering)
- https://github.com/everyinc/compound-engineering-plugin

# Writing Motivation
- 记录最早在Agent工程方向启蒙我的Compound Engineering。

# Overview
- 本文先介绍Why，然后依次对[Every](https://every.to/), [Vinci Rufus](https://www.vincirufus.com/)的文章进行简述。
- Every是Compound Engineering的提出者，深度介绍了如何使用该方法自动构建他们的应用Cora。
- Vinci Rufus将Compound Engineering作为理论根基，和其他理论进行对比，并与时俱进发展到多agent编排。
- 各级标题即文章出处，标题下的文本是我的精简版，有点像个人note而不是blog了。

# Why Compounding Engineering?
- 这里引入一篇260109的文章[认知重建：Speckit 用了三个月，我放弃了——走出工具很强但用不好的困境 - 腾讯技术工程](https://zhuanlan.zhihu.com/p/1993009461451831150)。我也是从这篇文章中得知Compound Engineering的。
- 作者探索AI编程工程化方法论，以解决规范共识、工程标准化、提示词复用问题。
- 作者研究[Spec Kit](https://github.com/github/spec-kit)与[OpenSpec](https://github.com/Fission-AI/openspec)后碰壁。speckit假设需求是清晰、可一次性规划的，而企业真实需求是动态、多方博弈、持续变化的。openspec虽然有Delta机制，但人工介入成本高。而且他们都更适用于流程从0开始的情况，缺乏历史经验。
- 作者从第一性原理出发，发现speckit, openspec会忽视过程，导致执行中的知识价值丢失。每一次的迭代花费时间相同，迭代并没有变简单，无法形成Compound。
- 作者认为其解决了知识管理问题，并结合Compound Engineering与[Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)，自己设计了一套AI工程工具，这里不做展开。

# Every介绍Compounding Engineering
- 我认为翻译为**复利工程**更贴切。
- 核心理念：每一个工程单元都应该让后续的单位变得更简单，而不是更难，所以被称为复利。
- 接下来我逐篇总结[Every](https://every.to/)关于Compounding Engineering的文章。
- (btw, [Every](https://every.to/)的插图真的很棒！)
- 更完整的Guide以及coding agent plugin见本文置顶的reference
- Guide中还包含一些AI时代个人需要做出的想法改变(关键词`Beliefs to`)、核心原则、最佳实践。这里不赘述。

## 250818 Kieran Klaassen发表的文章[My AI Had Already Fixed the Code Before I Saw It](https://every.to/source-code/my-ai-had-already-fixed-the-code-before-i-saw-it)
- Claude Code从前3个月的code review中吸取经验并自动应用，还捕捉了作者的taste。
- 作者将其称为Compounding Engineering，构建自我改进的系统，每一次迭代让下一次迭代更好更快。
- 作者为Cora(AI管理邮件)构建一个frustration detector察觉用户的不满，并自动提交改进报告。
- 传统软件工程方法包括
  - TDD
- AI软件工程方法包括
  - 工作流程被编码进`CLAUDE.md`
  - 将生产错误转化为永久修复: 通过agent自动调查崩溃、从系统日志中重现问题
  - 录制协作工作会议，Claude提取架构决策、创建一致的标准
  - 建立具有不同专业知识的评审agent
  - agent自动生成可视化文档(带图表、流程图、界面示意图的技术的文档)
  - 并行化反馈解决方案

### 复利工程行动指南
- Step 1: Teach through work
  - `CLAUDE.md`中用大白话记录taste
  - `llms.txt`中有高层架构决策——设计原则、系统级规则 (笔者注: 这里反模式了。[llms.txt](https://llmstxt.org/)应该提供结构化内容目录。架构决策文档保存在`docs/architecture_decision_records.md`中更合适。)
- Step 2: Turn failures into upgrades
  - 程序出错时。传统工程师会解决眼前的问题。而复利工程师负责添加测试、更新规则、编写evaluation
- Step 3: Orchestrate in parallel
  - 作者的[Warp](https://github.com/warpdotdev/warp)看起来像任务控制中心，左做planning，中做delegating，右做reviewing
- Step 4: Keep context lean but yours
  - 作者建议不要抄网上的ultimate CLAUDE.md files
  - 应该反应自己的代码库、模式、辛苦获得的教训
- Step 5: Trust the process, verify output
  - 本能是微观管理和审查每一句。但应该通过测试、评估、抽查来进行验证。

## 251211 Dan Shipper与Kieran Klaassen发表的文章[Compound Engineering: How Every Codes With Agents](https://every.to/chain-of-thought/compound-engineering-how-every-codes-with-agents) + 260313 [Compound Engineering Camp: Every Step, From Scratch](https://every.to/source-code/compound-engineering-camp-every-step-from-scratch)

### 复利工程loop
- Plan
  - plan前可brainstorm
  - plan应该无需人运行
  - Plan占工作的70%
  - agent查看代码库以及commit history，了解结构、最佳实践、构建方式
  - 交付为一个文档，作为本地文件或者Github上的issue。内容为feature的目标、拟议架构、代码的具体写法、研究来源列表、成功标准
- Work
  - trick是适用MCP，例如Playwright, XcodeBuildMCP
  - 生成PR
- Review / Access
  - 传统方法包括linter, unit tests
  - AI方法包括code review agents包括Claude, Codex, Friday, Charlie
- Compound
  - 记录之前学到的内容——bugs、潜在性能问题、新的解决方法
  - 在上下文被压缩前compound

- 不同步骤使用不同模型
   - brainstorm: 快模型Claude Haiku 4.5, Gemini 2.5 Flash
   - planning: Opus
   - implementation: Codex
   - code review: Gemini

### limitations
- 开发者决定构建、处理什么，描述“好”是什么

## 260529 [Compound Engineering Gets an Upgrade](https://every.to/p/compound-engineering-gets-an-upgrade)
- loop从4步拓展为了8步
  - ideate
  - brainstorm
  - plan
  - work
  - review
  - polish
  - compound
  - repeat


# Vinci Rufus关于Compound Engineering的看法
## 260120 [How the Ralph Loop Plays a Key Role in Compound Engineering](https://www.vincirufus.com/en/posts/ralph-loop-compound-engineering-future-software-development/)
- [Ralph Loop](https://ghuntley.com/ralph/)每次迭代产生一个新的agent instance，以解决[Context Rot](https://www.trychroma.com/research/context-rot)问题。
- `AGENTS.md`在Compound Engineering和Ralph Loop中都很重要，其产生飞轮效应。
- Ralph Loop需要反馈，包括
  - Type checking
  - Automated tests
  - CI
  - Browser verification
- Ralph Loop成功的关键要素是任务尽可能小，防止超出上下文。
- Ralph Loop改变了软件开发的经济学。工程师成为编排者，专注于架构、反馈环、质量门槛。Geoffrey Huntley认为[软件开发已死，而软件工程非常重要](https://linearb.io/dev-interrupted/podcast/inventing-the-ralph-wiggum-loop)。
- Steve Yegge的[Gas Town](https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04)依赖Molecular Expression of Work细粒度拆分工作，代表Compound Engineering的终极体现。

## 260122 [Compound Engineering vs Traditional Software Engineering - Why Linear Teams Can't Keep Up](https://www.vincirufus.com/en/posts/compound-engineering-vs-traditional-software-engineering/)
- 传统软件工程是线性，复利工程是指数级
- 复利工程还消除了传统软件工程中线性依赖带来的协调成本。
- 举例计算AI+2名工程师 > 30个工程师。
- 传统软件工程还有哪些方面出色吗？
  - 需要人类创造力的新颖问题
  - 政治与组织导航
  - 指导与知识转移
  - 分布式决策

## 260217 [Antfarm Patterns - Orchestrating Specialized Agent Teams for Compound Engineering](https://www.vincirufus.com/en/posts/antfarm-patterns-orchestrating-specialized-agent-teams/)
- [Antfarm](https://github.com/snarktank/antfarm)是多智能体工作流协调开源框架。
- Antfarm每个步骤都有干净的上下文。除了git和进度文件，没有共享内存。没有Context Rot。
- Antfarm的自带`feature-dev`工作流就是Compound Engineering的体现。

- AI干得好的
  - 简单直接 参数清晰
  - 通过可复现步骤修复bug
  - 生成已知边缘case的测试
  - 文档更新
- AI干得不行的
  - 探索性工作: 缺少上下文
  - 复杂架构决策: 需要人类判断
  - 训练分布之外的新问题
  - 需要真正创造力而不是模式匹配完成的事情
- 最佳平衡点是 well-defined, bounded tasks
- 多智能体编排是实现真正大规模复利工程的唯一途径
  - 横向扩展
  - 全天候工作
  - 质量一致
  - 廉价迭代
- 工程师摆脱写样板、基础测试、审查变更的低杠杆工作。需要设计、协调、改进agent系统的工程师。

## 260426 [Compound Engineering v3 and the Rise of Agentic Software Delivery](https://www.vincirufus.com/en/posts/compound-engineering-v3/)
- Compounding Engineering v3发布
- 命名空间统一
- 可追溯性: 通过ID贯穿始终
- 跨Harness可移植性
- 更好的Review
- 自我诊断代理
- next: 考虑交付的agent

## 260426 [What Is Agentic Engineering - The Complete Guide to AI-First Software Development](https://www.vincirufus.com/en/posts/agentic-engineering-building-systems-where-ai-agents-do-the-work/)
- Why: 2026年，采用Agentic Engineering智能体工程的团队比实用单一agent的团队快5-10倍

### Evolution: Solo Coder -> Agent Ochestrator
- write code: `产出 = 打字速度 × 知识 × 可用小时数`
- AI as Pair Programmer: `产出 = AI生成速度 × prompt质量 × review能力`
- Design Systems Where Agents Work: `产出 = agents数量 × Agent专业能力 × 反馈循环质量 × 复利速率`

### Architecture
- specialized agents: `Researcher → Architect → Builder → Tester → Reviewer → Documenter`
- 遵从单一责任原则，保持可调试性
- 专业agent会在自己的岗位上变得更好
- agents间的显式约定
- 可观察状态
- 复利反馈循环

### [Ralph Loop实战](https://www.vincirufus.com/en/posts/agentic-engineering-building-systems-where-ai-agents-do-the-work/#the-ralph-loop-agentic-engineering-in-action)

### 智能体工程的四大支柱
1. 显式约定
2. 可观察状态
3. 幂等工作流
4. 组合优于巧技

### [Get Started](https://www.vincirufus.com/en/posts/agentic-engineering-building-systems-where-ai-agents-do-the-work/#the-ralph-loop-agentic-engineering-in-action)


## 260428 [Agentic Engineering Patterns- The Foundation of AI-Native Development](https://www.vincirufus.com/en/posts/agentic-engineering-patterns/)
### agent harness
- agent lifecycle
- tool registration
- workflow composition
- state management

### content engineering
- schema-first
- context hierarchy
- prompt template
- learning capture
- multi-modal content
  - interaction diagram
  - accessibility notes
- agent improvement
- knowledge graph

### compound engineering

### [Getting Started](https://www.vincirufus.com/en/posts/agentic-engineering-patterns/#implementation-guide-getting-started)

### [Tools and Resources](https://www.vincirufus.com/en/posts/agentic-engineering-patterns/#implementation-guide-getting-started)


## 260429 [Compound Engineering - The Complete Guide to 300-700% Faster Development](https://www.vincirufus.com/en/posts/compound-engineering/)
- vibe coding能够把编码速度提高30%-70%，而复利工程能够实现300-700%的生产力提升。
- 需要转变软件开发思维，随着代码生成速度变快，反馈循环、指令护栏、端到端测试决定工程战略成败。
- 复利工程等式: `生产力 = 编码速度 × 反馈质量 × 迭代频率`

### 工程师的思维转变
- Code Writer -> System Orchestrator
- Get It Right -> Fail Fast and Correct
- From Manual Verification -> Automated Guardrails

### The Feedback Loop
- how quickly you can 
  - Validate correctness
  - Test funtionality
  - Verify integration
  - Deploy safely
  - Get user feedback
- 快速反馈架构
  - 大规模TDD
  - 并行测试执行
  - 增量部署 (canary)
  - observability驱动开发

### Guardrails
- Precision-Productivity Tradeoff: 平衡前期澄清成本以及后期重新生成的时间, token成本
- 指令护栏构建
  - 规范即代码
  - 类型约束即护栏
  - 带有上下文的prompt模版
- 验证优先的做法

### Testing Frameworks
- 单元测试验证组件，端到端测试系统
- 测试金字塔的转变
  - 更多的E2E覆盖
  - 更快的E2E执行
  - 并行E2E执行
- 如何打造Testing Harness
  - 快速单元测试
  - 并行集成测试
  - 优先级的E2E测试
  - 基于属性的测试
  - 性能基准测试
- CI/CD作为复利工程基础设施
  - 亚秒级运行：并行、增量测试
  - 即时回滚: 功能开关、蓝绿发布
  - 自动验证：linting, type checking, 安全扫描
  - 可观测性hook: 指标仪表盘、异常检测
  - 反馈集成：测试结果指导提示优化

### [Practical Framework](https://www.vincirufus.com/en/posts/compound-engineering/#implementing-compound-engineering-a-practical-framework)

### [The Economics of Compound Engineering](https://www.vincirufus.com/en/posts/compound-engineering/#the-economics-of-compound-engineering)

# What's next?
- Compound Engineering vs [[harness-engineering]]
- 用这套方法做一个项目

