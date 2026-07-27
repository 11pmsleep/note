> 来源链接：https://b23.tv/9UUp3LZ

# Harness Engineering 核心知识与学习指南

## 学习目标
完成本内容学习后，你将能够：
1. 准确阐述Harness Engineering的核心定义与设计目标，厘清其与提示词工程、上下文工程的演进与包含关系
2. 掌握Anthropic、OpenAI、LangChain三家机构的Harness落地方案与适用场景
3. 理解Harness技术的生命周期与长期价值，建立AI应用领域工程能力的核心认知
4. 掌握面向新手的Harness科学学习路径

## 1. Harness的起源与核心定义
Harness是AI Agent领域近期兴起的工程概念，暂无统一中文翻译，区别于此前的MCP、上下文工程、Skill等概念，其核心聚焦于Agent可靠性提升。
![](images/screenshot_000_ee71db52-1fb8-44f5-8f63-76e3599740d7.jpg)
![](images/screenshot_001_5bb8d930-f119-46a8-b50c-39266d1c3082.jpg)

### 1.1 概念起源
Harness概念最早来自行业大牛关于Agent使用的研究文章，核心出发点是：构建一套机制，保障Agent不会重复犯同类错误。Harness Engineering并非单一技术或工具，而是所有用于提升Agent可靠性的技巧、框架、机制的集合。
该概念提出后获得行业广泛共鸣，后OpenAI发布纯AI生成百万行生产级代码的相关研究，证实了Harness的工程价值，推动该概念快速爆火。
![](images/screenshot_002_188c8f64-95e3-4ccb-b29e-2c82f62bbc94.jpg)
![](images/screenshot_003_4ca52e25-3c53-447e-be51-0b8dbe9224ae.jpg)
![](images/screenshot_004_63cbeb0e-5c06-46c5-a499-13bd323108ed.jpg)

### 1.2 核心目标
当前Agent的发展现状为：能力已经足以落地到实际生产场景，但可靠性不足，无法完全脱离人工管控稳定执行任务。Harness的核心目标就是通过各类工程手段，弥补Agent的能力缺陷，提升其执行任务的稳定性与可靠性。
常见的基础Harness技巧包括：将Agent的错误教训沉淀到文档避免重犯、开发前先让Agent输出任务计划经人工校验后再执行等。
![](images/screenshot_005_59fa70a0-25bb-4f61-8223-da1a838b7a78.jpg)
![](images/screenshot_006_08b0ec44-989c-45e6-8e78-a0b041095248.jpg)

## 2. Harness行业落地实践案例
![](images/screenshot_007_5b7ad25f-e91b-43c0-8a25-587bbeb168e7.jpg)

### 2.1 Anthropic的Harness方案
Anthropic围绕Agent的落地痛点，提出了两套不同场景的Harness架构：

#### 2.1.1 长任务拆解+Agent接力架构
针对Agent上下文窗口有限、长任务执行效果随上下文累积持续下降的痛点，提出拆分+接力的解决方案：
1. 将复杂大任务拆解为多个独立小任务
2. 每个小任务由独立的Coding Agent执行，每个Agent拥有独立的上下文空间
3. 每个Agent启动时，通过命令获取任务进展、编码规范等必要信息，执行完成后输出工作日志传递给下一个Agent
该方案的代价是会消耗更多Token，重复读取编码规范等公共信息。
![](images/screenshot_008_a3b79725-e373-47d6-a484-279f40d5b4df.jpg)
![](images/screenshot_009_323f50dd-d9c4-47c9-9a02-8bdc99060054.jpg)
![](images/screenshot_010_ea253fd6-4aa3-44ca-9a9a-adce1cf8e46f.jpg)

#### 2.1.2 生成器-评估器分离架构
针对Agent自评估偏差（即Agent容易误判自己的输出质量，宣称任务已完成但实际存在大量问题）的痛点，提出运动员与裁判分离的架构：
1. 生成器Agent：负责生成代码、输出任务成果
2. 评估器Agent：负责独立验证生成成果的质量，提出优化意见
3. 两者多轮交互迭代，直到输出符合质量要求
为保障评估效果，评估器有三项配套设计：
- 预设明确的评估规范，覆盖设计质量、原创性、工艺水准、功能性等维度
- 集成Playwright工具，支持实际操作页面做精准验证
- 支持智能方向调整：如果多轮迭代后效果无提升，则判定当前技术路径不可行，切换优化方向
![](images/screenshot_011_be84209a-207b-4afc-9b10-094cb71a88e9.jpg)
![](images/screenshot_012_c2fc9643-1cd1-4207-ae5c-fbb20053fc18.jpg)

#### 2.1.3 两种架构的适用场景对比
Anthropic的两套Harness模式对应不同的任务类型：
1. 线性执行模式：按部就班完成任务清单，适合需求明确、边界清晰的标准化任务
2. 循环迭代模式：通过生成-评估-修改的多轮攻防持续提升质量，适合探索性、创新性的非标准化任务
![](images/screenshot_013_532ca78a-be2c-4972-8fee-4e3e78f5a424.jpg)
![](images/screenshot_014_ef4a917b-4242-4604-bdf0-27b59c00bbe8.jpg)

### 2.2 OpenAI的Harness实践
OpenAI通过Harness机制实现了纯AI生成百万行生产级代码，其核心技巧包括：
1. **代码仓维护**：及时清理过时内容、补充必要信息，AI的所有认知完全基于代码仓的内容，不在代码仓内的信息对AI而言不存在
2. **AI友好的日志构建**：日志面向AI阅读设计，而非面向人类开发者，采用AI易理解的结构构建，支持AI自行排查问题
3. **渐进式披露**：不在Agent.md（项目说明文件）中写入所有细节，仅将其作为索引目录，让AI根据需求自行查找对应信息，避免上下文冗余
![](images/screenshot_015_d46ad347-4c76-4bf7-9639-88411a8c5e7d.jpg)
![](images/screenshot_016_5f03da7c-13d2-4f4b-94c7-008ab3df759c.jpg)

### 2.3 LangChain的Harness实践
LangChain在不更换底层模型的前提下，仅通过Harness技巧，就将其Deep Agent在编码榜单的排名从30名外提升至第5名，核心技巧包括：
1. **AI优化AI**：完整记录Agent的执行轨迹（包括决策过程、工具调用、错误节点等），将轨迹输入大模型，让AI自行提炼优化方案
2. **中间件Hook机制**：在Agent执行的全流程关键节点插入校验逻辑，例如在Agent判定任务结束时，自动校验任务实际完成度，如果未完成则强制提示Agent继续执行，避免Agent提前终止任务
![](images/screenshot_017_e2e4f354-95f1-44c0-baac-f9353a430418.jpg)
![](images/screenshot_018_b223b0dc-3ab3-4e20-864b-ac1bfe54b55c.jpg)
![](images/screenshot_019_96979dd3-8162-4a4e-b757-797f4050fbe1.jpg)

## 3. Harness与相关工程概念的演进关系
Harness与提示词工程、上下文工程一脉相承，都是面向AI应用的工程技巧，随人类对大模型的应用深度逐步演进，三者为包含关系，而非并列关系。
![](images/screenshot_020_140e431a-eed7-4cc9-9bd3-587905127c81.jpg)

| 概念 | 发展阶段 | 核心目标 | 典型技巧 |
|------|----------|----------|----------|
| 提示词工程 | 第一代：早期简单对话阶段 | 优化与大模型的单轮/多轮对话效果 | 角色设定、Few-shot示例、结构化提示词 |
| 上下文工程 | 第二代：Agent辅助阶段 | 合理编排大模型的上下文空间 | 记忆压缩、Skill渐进式披露、多轮对话管理 |
| Harness | 第三代：当前Agent独立执行阶段 | 构建大模型运行的全链路外层框架，保障可靠性 | 评估器、Hook机制、记忆管理、中间件 |
![](images/screenshot_021_f4a9bbb8-2f78-41be-b649-882784370b48.jpg)
![](images/screenshot_022_cc1c999a-a126-4424-82c1-c4cb2e4bd630.jpg)
![](images/screenshot_023_6b7efd67-65bb-408f-a2a7-76f4046f2645.jpg)

无论概念如何迭代，AI应用的核心始终是工程能力。工程能力的差距是企业与个人AI应用能力的核心差距：低工程能力者会判定需求无法实现，高工程能力者会探索多种可行方案尝试落地。对于绝大多数开发者，AI应用的核心能力就是模型应用的工程能力，而非底层大模型研发能力。
![](images/screenshot_024_cb9dcb93-9b0c-4a4a-932b-c39b877deba1.jpg)
![](images/screenshot_025_0c81f74d-289c-4b51-bbed-da76b4c609b8.jpg)

## 4. Harness的发展前景
### 4.1 Harness不会被模型升级淘汰
针对“模型升级后Harness是否会过时”的疑问，核心结论为：
1. 模型能力升级确实会淘汰部分旧的Harness技巧，但更强的模型会被应用到更复杂的场景，产生新的痛点与问题，进而催生新的Harness技巧
2. 个别Harness技巧可能随模型迭代被淘汰，但Harness Engineering作为提升Agent可靠性的工程方法论，在可见的未来会长期存在
3. 并非所有开发者与企业都能使用最强模型，小模型、旧模型的落地场景中，Harness的价值更为突出
![](images/screenshot_026_9bd9575c-a2e3-4d02-94d8-60e0253cfef3.jpg)
![](images/screenshot_027_81aff7dd-35bf-417a-9dc4-98587e8bb760.jpg)
![](images/screenshot_028_0cb38a49-d10d-429a-b967-68aeb839476e.jpg)

## 5. Harness学习路径
Harness是面向实际问题的坑位解决技巧，没有实际Agent使用经验、未踩过相关痛点的新手，直接啃教程无法理解其价值，因此推荐分三步学习：
1. **Step1 先去用**：下载Claude Code，购买Coding Plan，凭直觉直接使用Agent完成实际开发任务，无需提前学习技巧
2. **Step2 去踩坑**：主动体验Agent的各类问题：重复犯错、输出不符合预期、提前终止任务、承诺修改但实际不调整等，积累真实的痛点体验
3. **Step3 系统学习**：带着实际踩坑的经验再学习Harness相关技巧，对应解决自己遇到的问题，即可快速理解掌握
![](images/screenshot_029_3b993745-ccc5-4be2-aaae-077506800fa8.jpg)
![](images/screenshot_030_6941ecec-bfb2-45bf-ba10-fd5452d4ac1b.jpg)
![](images/screenshot_031_a07de0d5-fe69-43ac-b6cf-2003351dcf41.jpg)

## 参考资料
1. 首次提出Harness Engineering概念的行业大牛研究文章
2. Anthropic. (2024). *Effective Harnesses for Long-running Agents*
3. Anthropic. Harness design for long-running application development 相关研究
4. OpenAI. 纯AI生成生产级代码相关研究报告
5. LangChain. (2025). *Improving Deep Agents with Harness Engineering*