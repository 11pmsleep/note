> 来源链接：https://b23.tv/3V7FyVh

## 核心知识点（学习目标）
本视频围绕让AI Agent长时稳定工作的`Harness`工程方法，覆盖以下核心内容：
1.  AI Agent长时工作的核心瓶颈：上下文窗口耗尽导致输出质量下降、任务中断
2.  单智能体循环方案（Ralph）的原理、实现逻辑与使用流程
3.  多智能体协同长时工作方案的架构设计、工作流程与提示词设计方法
4.  多智能体方案的落地实践：PPT自动开发的完整流程与效果验证
5.  Harness工程的核心优化技巧

---

## 1. 背景：AI长时工作的核心瓶颈
当AI执行长周期复杂任务时，随着对话推进，上下文窗口占用量持续上升，会出现两类问题：
1.  模型理解负担加重，输出质量波动、稳定性下降
2.  上下文完全耗尽后，模型停止工作，任务直接中断
![](images/screenshot_000_d6a1e635-00e8-4115-8901-552970b8ce0e.jpg)*
![](images/screenshot_001_c7329584-3464-44ff-872b-c667f59aa6fc.jpg)*

---

## 2. 方案一：Ralph 单智能体循环方案
### 2.1 核心原理
Anthropic官方提出的基础解决思路：将大任务拆分为多个独立子任务，**每个子任务在全新的独立会话中完成**，通过重置上下文避免窗口耗尽，循环执行直到所有子任务完成。
![](images/screenshot_002_2d3ad067-fe99-466e-87cb-bb336fdc90eb.jpg)*
![](images/screenshot_003_e4a91483-0dc2-44f5-a896-a5569f13bb3f.jpg)*

### 2.2 Ralph的实现逻辑
Ralph项目的核心是通过Shell脚本维护任务循环，核心执行逻辑：
1.  通过`claude`命令行工具启动新会话，每轮会话读取统一的提示词
2.  AI执行完成后，通过本地文件同步任务进度，更新待办状态
3.  脚本检测任务是否全部完成，未完成则自动启动下一轮循环
![](images/screenshot_004_0fb83ed8-0e63-44f0-9e9b-e82910e0803e.jpg)*
![](images/screenshot_005_8e567548-b13a-40be-8083-172c810eaa14.jpg)*
![](images/screenshot_006_76cde25e-d1af-4031-94e3-8497ab6a4fc7.jpg)*

> 补充说明：`claude`命令行调用和Claude客户端GUI操作效果完全等价，命令行模式支持自动化调用，是Ralph实现无人值守循环的基础。
![](images/screenshot_007_0f2bd12b-8def-41f4-81c3-73908dec3bee.jpg)*
![](images/screenshot_008_46c96699-19c8-480c-b9c9-2f9203840e5e.jpg)*

### 2.3 同提示词执行不同任务的设计逻辑
每轮循环读取相同提示词，但执行不同任务的核心机制：
提示词要求AI每轮工作遵循固定流程：
1.  读取需求文档、进度文件，了解当前任务进度
2.  从待办任务中挑选优先级最高的任务执行
3.  完成后更新进度文件，标记对应任务完成
即**提示词逻辑不变，存储进度的本地文件内容变化，驱动AI执行不同任务**。
![](images/screenshot_009_17253f04-5b78-488f-9cb4-09bef5e34a62.jpg)*
![](images/screenshot_010_014bf227-bb85-46e7-8704-b028c10f16d3.jpg)*
![](images/screenshot_011_065bd87e-6ffa-4885-91bf-8790ad45ee3c.jpg)*

### 2.4 质量保障机制与局限性
Ralph原生的质量保障手段：
-  PRD需求文档提供完整任务上下文
-  进度文件同步全局任务状态
-  `AGENT.md`沉淀历史踩坑经验，避免重复错误
-  要求AI每轮任务完成后执行自测

**局限性**：AI自测效果有限，单智能体自开发自测试，产出质量存在明显上限。
![](images/screenshot_012_f09bd6bf-a04b-4e08-8dd6-0b718d938d97.jpg)*
![](images/screenshot_013_2b35a4ff-aaf2-4926-a343-db8fe9efb047.jpg)*

### 2.5 Ralph的三种使用方式
| 类型 | 说明 | 灵活度 |
|------|------|--------|
| 自写Shell脚本 | 完全自定义循环逻辑，适配个性化需求 | 最高 |
| 原生Ralph | 社区维护的基础实现方案 | 中等 |
| Ralph for Claude Code | 功能增强版，开箱即用，配套能力最完整 | 最低 |
![](images/screenshot_014_c2f288bd-bb9c-4698-adbd-9bdeec84de49.jpg)*
![](images/screenshot_015_000977e6-6985-4f60-a025-063018db4070.jpg)*

### 2.6 Ralph使用三步流程
1.  **准备PRD需求文档**：清晰描述任务要求，放置在项目目录下
2.  **框架自动生成配套文件**：自动生成开发计划、设计规范、执行提示词等
3.  **运行Shell脚本启动循环**：AI自动执行任务，等待全部完成即可
![](images/screenshot_016_89c7265c-fcad-41cf-b340-b2b4b939998a.jpg)*
![](images/screenshot_017_ab5996ee-26f4-4a4b-ae7c-62c5c74a3f89.jpg)*
![](images/screenshot_018_7d98d80b-42ff-4778-90f0-6017bc42e78b.jpg)*

---

## 3. 方案二：多智能体协同长时工作方案
### 3.1 两种方案的共同本质
两种方案的核心逻辑都是**为每一段任务创建独立的上下文空间，避免主上下文耗尽**：
-  Ralph方案：通过bash循环+新建会话重置上下文，由脚本维护任务状态
-  多智能体方案：每个子Agent天然拥有独立上下文，由主Agent维护流程调度
![](images/screenshot_019_bf1a4fe0-3a30-429e-a442-88104e60daf2.jpg)*
![](images/screenshot_020_5f0441e0-56da-49bb-9b3a-b404b33d637c.jpg)*

### 3.2 多智能体方案整体架构
核心设计思路：
1.  **主Agent只调度不干活**：不负责开发、测试等具体工作，仅负责协调子Agent，保持自身上下文简洁，不会被任务内容占满
2.  子Agent分分工负责：设置计划Agent、开发Agent、多维度测试Agent等，每个子Agent拥有独立上下文，完成具体任务
![](images/screenshot_021_6f49849a-e14f-4c02-98db-f817feae92df.jpg)*
![](images/screenshot_022_bdb3d58c-ab93-4fc1-893e-299248e27c0e.jpg)*
![](images/screenshot_023_30fe9e3d-4a8b-4b4f-8c52-ddeaf8720ce2.jpg)*

### 3.3 方案实现两步法
1.  **设计工作流程**：明确各智能体职责、执行顺序、信息传递方式、异常处理逻辑
2.  **编写智能体提示词**：分别编写主Agent和子Agent的提示词，可先让AI辅助生成，再人工调整优化
![](images/screenshot_024_52f891c8-a8b3-4f2a-88ff-54372e1cf51c.jpg)*
![](images/screenshot_025_54d0645e-a12b-4dbd-bef4-562f6c266ea7.jpg)*

### 3.4 工作流程设计示例：PPT自动开发
#### 3.4.1 任务拆分粒度原则
粗粒度拆分（一次性交付大量任务）效果差，易出现布局、逻辑一致性问题；**细粒度拆分（一次开发1个页面，开发一个验证一个）**效果更稳定，测试也遵循同样粒度，一次验证少量页面效果更好。
![](images/screenshot_026_1f8a8be7-b391-49db-8173-a2e00bcf1171.jpg)*
![](images/screenshot_027_50cec589-66f5-4e2f-b7da-d7b73dbbac34.jpg)*

#### 3.4.2 完整工作流程
1.  **任务接收与计划阶段**：主Agent接收需求文档，将路径传递给计划子Agent；计划Agent完成开发计划后，仅将计划文件路径返回主Agent
2.  **任务开发阶段**：主Agent取出单个任务，启动开发子Agent执行；开发完成后仅将产出文件路径返回主Agent
3.  **测试阶段**：主Agent将开发产出路径提交给测试子Agent；测试完成后仅将测试报告路径返回主Agent，不返回报告详情
4.  **修正循环**：
    -  测试失败：通过`resume`参数唤醒**历史开发Agent**（保留开发上下文）修复bug，修复完成后唤醒**对应历史测试Agent**重测，最多循环3次
    -  测试成功：更新任务进度，启动下一个任务的开发
![](images/screenshot_028_1b221757-d8ad-4f7f-93ac-a0851c57985d.jpg)*
![](images/screenshot_029_7f9a3dce-f2e1-4de0-baa6-9cb948ac3ec8.jpg)*
![](images/screenshot_030_d21bd6e0-088f-420a-b081-ed38f5537b35.jpg)*
![](images/screenshot_031_142d5567-e934-4b73-84f0-4da714bd1055.jpg)*

### 3.5 提示词设计
#### 3.5.1 主Agent提示词设计原则
核心要求：**只调度不干活，保持上下文整洁**
-  核心原则：不做开发、不做测试、不读取子Agent的产出内容，只接收文件路径和PASS/FAIL判定
-  日志记录：每个关键步骤写入日志，方便追踪进度和排查问题
-  Agent ID管理：通过文件系统探测获取子Agent ID，用于后续resume唤醒对应子Agent
-  流程控制：负责启动计划Agent、控制开发循环、修正循环、进度更新
![](images/screenshot_032_b1e28a57-c08f-433c-9f9e-2eea93ef314f.jpg)*
![](images/screenshot_033_a490e16c-2019-419b-a437-eba268920ff8.jpg)*
![](images/screenshot_034_45a3e03a-af38-46e9-9b66-1b80e1167243.jpg)*
![](images/screenshot_035_6fa74993-92a2-4eaa-9c2b-428de1f00196.jpg)*
![](images/screenshot_036_da3a5c1f-c85f-4a7a-8674-8de750659c50.jpg)*
![](images/screenshot_037_4cadab66-6b92-472e-a0f2-1a39ab6c5455.jpg)*

> 关键方法`resume`：调用子Agent时传入ID，可唤醒保留历史上下文的对应子Agent，无需重新启动新Agent，保留之前的开发/测试信息。若版本不支持，也可使用`SendMessage`方法替代。
![](images/screenshot_038_d021952e-1e86-4363-99f5-f47432e493fd.jpg)*
![](images/screenshot_039_7ffc71c1-6fd2-4c53-9ea0-89001b630167.jpg)*

#### 3.5.2 子Agent模板规范
Claude Code的子Agent通过markdown文件定义，分为两部分：
1.  **顶部配置**：定义智能体名称、可用工具、使用模型、权限模式、分配的Skill等
2.  **底部系统提示词**：定义智能体的工作方式、输入输出规范、决策原则
文件需要放置在`.claude/agents/`目录下，主Agent可通过名称调用生成对应子Agent。
![](images/screenshot_040_53f1d3c6-84e6-48db-ad7a-2f74b62d3088.jpg)*
![](images/screenshot_041_9a432b02-6dc3-4291-8c4e-cda00094bc6d.jpg)*
![](images/screenshot_042_2897f563-3b1e-4cb4-ab81-b46536ffd197.jpg)*
![](images/screenshot_043_0712d5bd-61b4-452e-905b-994fcf3e1c5c.jpg)*

#### 3.5.3 子Agent提示词的两个核心作用
| 作用类型 | 说明 | 性质 |
|----------|------|------|
| 衔接流程 | 规定工作前读取的文件、工作后写入的文件、仅返回路径给主Agent，适配当前工作流 | 流程特有 |
| 办好事情 | 规定使用的Skill、开发/测试规范、质量要求标准 | 通用要求 |

> 重要提示：不要迷信框架和固定流程，主Agent仅负责维护流程，**最终交付质量完全由子Agent的设计质量决定**。
![](images/screenshot_044_bbf2dbef-100d-4b73-b930-15729c9ce7dc.jpg)*
![](images/screenshot_045_0534e66a-d516-4881-9041-cbd39ee0c366.jpg)*
![](images/screenshot_046_921a9ce9-537c-4677-9141-ee0b92ccfffb.jpg)*
![](images/screenshot_047_89dd5337-6bb5-4034-a463-bed594315bc2.jpg)*

#### 3.5.4 子Agent设计示例
| 智能体类型 | 使用模型 | 职责 |
|------------|----------|------|
| 计划Agent | GLM-5.1 | 制定拆分后的开发计划 |
| 开发Agent | GLM-5.1 | 开发页面，修复bug，沉淀通用经验 |
| 布局测试Agent | GLM-4.7 | 页面布局合规性测试 |
| 美观测试Agent | GLM-4.7 | 页面美观度一致性测试 |
| 动画测试Agent | GLM-4.7 | 页面动画逻辑与时序测试 |

子Agent提示词的两个核心设计要点：
1.  **通过文件传递信息**：明确规定工作前读取的文件、完成后写入的文件，例如测试Agent开始前读取开发的代码文件，结束后将测试结果写入本地报告
2.  **严格限制输出格式**：仅向主Agent返回文件路径，不输出大段的开发/测试内容，避免占用主Agent的上下文空间
![](images/screenshot_048_74e28a37-c984-4c44-b4c1-a91d7c1eae66.jpg)*
![](images/screenshot_049_7a2da752-eaa8-4e08-a929-f5c9c4867e9a.jpg)*
![](images/screenshot_050_df420f80-c7a2-4558-8bd5-6b64bafec25d.jpg)*
![](images/screenshot_051_a4b8c093-b265-4dc1-86fa-f8dc86a0869a.jpg)*
![](images/screenshot_052_e2b61f57-7f33-4dc2-8e93-41d309f330f7.jpg)*
![](images/screenshot_053_9a05752a-ef3b-4a77-87de-8476e8244110.jpg)*

#### 3.5.5 经验沉淀机制
要求开发Agent修复bug后，将通用性经验写入`lessons-learned.md`，经验写入遵循三个原则：
1.  原则性>数值性：写"为什么错"，而非"改了什么值"
2.  模式级>页面级：写"哪种布局模式容易犯这个错"
3.  可迁移>可复制：确保经验在其他项目也能适用
经验的两个核心作用：
1.  当前项目循环中，后续开发可以复用之前踩坑的经验，减少重复错误
2.  项目完成后用于优化完善Skill，提升后续项目的开发质量
![](images/screenshot_054_b8888aab-25b8-4ae6-8879-18779877adb0.jpg)*
![](images/screenshot_055_d9b8938a-2904-499c-a2db-a7c46fadd648.jpg)*
![](images/screenshot_056_c96cce02-2c69-4c30-8b29-cb71e4c7c1ee.jpg)*
![](images/screenshot_057_a8aeed91-f674-480c-b768-6a556251066e.jpg)*

### 3.6 Skill配置
可以为开发、测试Agent分配专属Skill，相当于子智能体的专业能力背书，是提升产出质量的关键。推荐的通用前端类Skill：
1.  `frontend-slides`：HTML演示文稿制作
2.  `diagram-design`：技术图表/架构图/流程图生成
3.  `huashu-design`：高保真原型/交互Demo设计
4.  `frontend-design`：通用前端界面设计
5.  `ui-ux-pro-max`：UI/UX全链路设计
![](images/screenshot_058_e4b7bd2c-e331-4d5b-8f5c-b2554f82d40f.jpg)*
![](images/screenshot_059_09cfd3f5-0469-4d1a-a657-7c3c199fb089.jpg)*
![](images/screenshot_060_fc29525c-dbf5-46fa-ba4b-97e14439eedd.jpg)*

### 3.7 运行流程
完成所有智能体设计后，只需向Claude Code提交需求文档和主智能体提示词，即可启动自动循环，AI会自动完成所有任务，过程中无需人工干预。
![](images/screenshot_061_1c051f00-d37b-4aa8-a884-a5b787226059.jpg)*
![](images/screenshot_062_1d098dd0-91dc-4792-9943-a1e10608d6b2.jpg)*

---

## 4. 实践效果演示
以PPT自动开发场景为例：
1.  运行过程：部分页面一次性通过测试，部分页面需要多轮迭代修正，整个过程可持续运行数小时，理论上可无限运行，只要网络和账号状态正常
2.  产出结果：完整的PPT HTML文件、开发日志、经验沉淀文档、测试报告、设计指南等全套产出
3.  经验验证：首批页面需要多轮迭代，后续页面借助沉淀的经验可以实现一次性通过，验证了多智能体方案记录经验、持续自优化的有效性。
![](images/screenshot_063_686b6824-2321-41ce-a6b5-35af38afce97.jpg)*
![](images/screenshot_064_23eafaea-3ab2-4e4e-a0ad-ef44f13616f7.jpg)*
![](images/screenshot_065_6db230ed-f02f-4315-b864-4b0632d162c4.jpg)*
![](images/screenshot_066_2832dcd9-d1dd-40d8-8c34-2a62df9436e7.jpg)*

---

## 5. Harness 长时工作技巧总结
| 序号 | 技巧 | 核心做法 |
|------|------|----------|
| 1 | 重置上下文是关键 | 用claude命令或Agent机制为每段任务创建独立上下文 |
| 2 | 多智能体=设计流程+写提示词 | 先梳理清楚业务流程，再让AI辅助生成提示词，人工调整优化 |
| 3 | 主Agent只协调不干活 | 主Agent不参与具体任务，保持主上下文整洁 |
| 4 | 用文件传递消息 | 明确设计每个智能体工作的输入文件、输出文件规范 |
| 5 | 独立Agent测试 | 拆分开发和测试角色，比开发自测试的效果更好 |
![](images/screenshot_067_0109fabf-7f5b-4641-9cf5-f45acf3f80e7.jpg)*

---

## 参考资源
1.  Ralph 开源项目官方仓库
2.  Anthropic Harness 工程官方文档
3.  Claude Code Agent 开发官方规范
4.  本次演示配套的子Agent模板、提示词示例文件