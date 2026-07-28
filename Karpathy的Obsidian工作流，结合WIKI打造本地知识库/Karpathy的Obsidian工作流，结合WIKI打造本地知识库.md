> 来源链接：https://www.bilibili.com/video/BV1EVSfBpERw

## 学习目标
1. 掌握Andrej Karpathy提出的基于Obsidian+大模型的本地知识库搭建核心逻辑
2. 掌握Obsidian、Claudian插件、Obsidian Web Clipper的安装与全流程配置方法
3. 理解WIKI索引层的设计作用，掌握「原始数据-索引摘要-问答输出」三层知识存储架构
4. 掌握从碎片化互联网内容采集、AI自动整理到知识库问答的完整工作流
5. 理解本地个人知识库相对在线方案的核心优势与适用场景

## 1. 方案概述
![](images/screenshot_000_3ddfce46-740f-4283-981e-d5ddf80ca23d.jpg)
本方案由前特斯拉AI总监Andrej Karpathy提出，核心是利用Obsidian作为本地知识载体，结合Claude Code大模型能力，将日常浏览的碎片化互联网内容沉淀为可检索、可调用的个人本地知识库。
整个工作流分为三个核心阶段：
1. 数据采集：将所有有价值的原始内容统一存储
2. 数据整理：大模型自动为原始内容生成索引摘要，构建WIKI目录
3. 问答输出：基于本地知识库的内容进行提问，获取整合后的结构化答案
核心依赖工具：Obsidian（本地笔记载体）、Claude Code（大模型推理与操作能力）

## 2. 基础环境搭建
### 2.1 Obsidian 安装与库初始化
![](images/screenshot_001_2464933a-5ce3-4e8c-8b50-4cb4139be068.jpg)
1. 访问Obsidian官网下载对应操作系统的安装包，完成安装
2. 在本地任意位置创建空文件夹，作为知识库的根目录（示例命名为Note）
![](images/screenshot_002_bd39dc79-e67b-4359-960e-04276174af5a.jpg)
3. 打开Obsidian，选择「打开本地仓库」，选中刚才创建的文件夹，完成库的初始化
4. 在Obsidian设置中将界面语言调整为中文，方便操作

### 2.2 Claudian 插件安装与配置
Claudian是将Claude Code集成到Obsidian的第三方插件，可以让大模型直接读写、操作本地知识库的所有文件。
![](images/screenshot_003_ef5ab03c-2898-446a-b817-15ea65c795a4.jpg)
安装步骤：
1. 访问Claudian的Github项目页面，将项目地址提供给Claude Code，指令Claude为你的本地Obsidian库安装该插件
2. 安装完成后，打开Obsidian的设置，进入第三方插件页面，关闭安全模式，找到Claudian插件并启用
![](images/screenshot_004_0cc29e3f-bfb6-4f5a-a12e-479eaa542446.jpg)
3. 启用后Obsidian左侧边栏会出现Claudian入口，点击即可调用集成的Claude Code能力，支持直接改写本地文档、操作库内文件
![](images/screenshot_005_0bf326d7-f2f4-4126-9a1a-4cc4410f240d.jpg)

## 3. 第一阶段：碎片化数据采集
### 3.1 采集层设计思路
![](images/screenshot_006_a0ce05be-ed54-42f2-b054-285efe0aeb9f.jpg)
Karpathy的设计中，所有原始素材（包括文章、论文、代码仓库、数据集、图片等）都完整保存在`raw/`目录下，不对原始内容做任何修改，保证信息的完整性，为后续的索引整理提供完整的数据源。

### 3.2 采集工具：Obsidian Web Clipper
![](images/screenshot_007_9a377ca1-ccb6-4860-ad0d-bcfe152c1ea7.jpg)
使用Obsidian官方的浏览器插件Obsidian Web Clipper实现一键网页内容采集：
1. 搜索并下载插件，添加到Chrome/Edge等浏览器中
2. 浏览需要保存的网页时，点击浏览器右上角的插件图标
![](images/screenshot_008_9b5ab8c1-9358-44c8-bb3c-7e223ee8282d.jpg)
3. 原设计保存路径为`raw/`，由于插件默认路径为`Clippings/`，实操中可以直接使用该目录作为原始数据存储目录
4. 点击「Add to Obsidian」，网页内容会自动同步到本地Obsidian库的Clippings目录下，保留元信息（来源、作者、发布时间等）
![](images/screenshot_009_29da3453-d245-4d33-b3aa-204c942ad721.jpg)

### 3.3 采集流程与价值
![](images/screenshot_010_0a7edc98-e054-43ac-87ec-7d7045efe2e9.jpg)
替代传统的浏览器收藏夹方式，所有有价值的互联网内容（推文、技术文档、教程、问题解决方案等）都可以一键保存到本地，数据完全本地化，不会因为链接失效丢失内容，同时可以直接被大模型检索与处理。
对于单篇采集的内容，可以直接向Claude提问，例如要求总结内容中的Bug、给出修复方案、提炼核心观点，大模型会直接读取本地文件输出结果。
![](images/screenshot_011_3812a53c-5916-4399-b351-746608edecf4.jpg)

## 4. 第二阶段：WIKI索引层构建
### 4.1 索引层的设计意义
![](images/screenshot_012_1e78963b-71cb-469c-9eaa-904647233d9a.jpg)
当知识库数据量较小时，大模型可以直接加载读取所有Clippings目录下的文件，直接完成问答；但当知识库规模扩大，例如超过100篇文章、总字数超过40万字时，大模型的单次上下文窗口无法加载全部内容，检索效率会大幅下降。
![](images/screenshot_013_6a01a7d9-7305-4132-803e-dcb57ca01c19.jpg)
WIKI索引层的作用就是为大模型提供知识库的目录导航，相当于给知识库做了一层摘要索引，大模型不需要读取所有原始文件，只需要先读取索引，就可以定位到需要的内容，不需要搭建复杂的RAG、向量数据库系统，即可实现大数据量知识库的高效检索。
![](images/screenshot_014_39a9ea6b-7eba-415e-af29-cd4ced9a5112.jpg)

### 4.2 索引层架构设计
Karpathy并未公开对应的配置文件，实操中可以通过自定义`CLAUDE.md`规则文件，定义大模型的操作规范，完整的知识库三层结构如下：
![](images/screenshot_015_e2cf01d2-ad46-4fe8-b85f-00c1e5efb565.jpg)

| 层级 | 目录 | 作用 |
|------|------|------|
| 原始数据层 | `Clippings/` | 存储所有采集的原始内容，完整保留所有信息 |
| 索引层 | `wiki/` | 存储知识库的索引与摘要，是大模型检索的入口 |
| | `wiki/INDEX.md` | 知识库总目录，按主题分类罗列所有内容，是AI检索的第一入口 |
| | `wiki/articles/` | 每一篇原始内容的摘要，摘要中包含反向链接，指向Clippings中的原始文件 |
| 输出层 | `outputs/` | 存储所有问答的输出结果，反过来丰富知识库 |

![](images/screenshot_016_04763b4c-67b7-47cb-a00e-492c07f6ee3b.jpg)
`CLAUDE.md`作为知识库的说明书，大模型打开知识库后会首先读取该文件，遵循其中定义的所有操作规则。
![](images/screenshot_017_1ec44089-bc03-48dc-975a-207d93aef938.jpg)

### 4.3 自动更新规则
在`CLAUDE.md`中定义核心运行规则，实现全流程自动化：
![](images/screenshot_018_45882424-7936-4dd5-9c91-62c02fcf85dc.jpg)
1. 检索永远从`wiki/INDEX.md`开始，AI首先读取总目录理解知识库的结构与内容分布
2. 检索时优先读取`wiki/articles/`下的摘要，若摘要信息不足以回答问题，再通过摘要中的反向链接，打开`Clippings/`下的原始文件读取详细内容
3. 每次有新的内容保存到`Clippings/`目录后，Claude会自动为新内容生成摘要，存入`wiki/articles/`，同时自动更新`wiki/INDEX.md`的总目录
4. 禁止手动修改`wiki/`目录下的内容，所有索引的更新都由AI自动完成，保证索引的一致性
![](images/screenshot_019_0bc000b8-d382-42e7-a1a2-bb7aad33dbec.jpg)

### 4.4 工作流效果测试
![](images/screenshot_020_ec0020e8-3174-492f-af25-a35ce51ea93e.jpg)
以要求AI编写《Claude Code使用技巧》为例，大模型的自动执行流程：
1. 首先读取`wiki/INDEX.md`，理解知识库的整体结构
2. 扫描`wiki/articles/`下所有和Claude Code相关的摘要，筛选相关的内容
3. 对需要详细信息的部分，通过反向链接读取原始文件的完整内容
4. 整合所有25篇相关文档的内容，生成结构化、完整的使用技巧指南，存入outputs目录
![](images/screenshot_021_d72294ff-a6f2-48b1-90e6-f6d0d8e8d888.jpg)

## 5. 方案优势与总结
### 5.1 核心优势
![](images/screenshot_022_67d957b8-c2d2-471a-807e-e45fd4565843.jpg)
1. **数据安全性**：所有内容都保存在本地，和Notebook LM等在线知识库方案相比，没有隐私泄露风险，数据完全由个人掌控
2. **采集范围广**：支持采集全网的任意内容，包括网页、推文、代码仓库、文档等，都可以一键存入知识库，不断积累个人知识体系
3. **架构轻量**：不需要搭建复杂的向量数据库、RAG系统，仅通过文件目录+大模型本身的能力，就可以实现高效的知识库检索，维护成本低
![](images/screenshot_023_df3bedfd-3c4c-437c-8bd2-7487e798fb5e.jpg)

### 5.2 完整工作流闭环
整个知识库形成正向循环：
1. **采集**：日常浏览互联网内容时，通过Obsidian Web Clipper一键将有价值的内容保存到本地`Clippings/`目录
2. **整理**：Claude自动检测新增内容，生成摘要，更新WIKI索引目录，不需要人工手动整理
3. **问答**：通过Claudian向知识库提问，AI自动检索索引、定位内容，输出结构化的答案，答案存入`outputs/`，反过来丰富知识库的内容，形成个人的第二大脑

## 参考与资源
1. Andrej Karpathy 原推文地址：https://x.com/karpathy/status/2039805659526445956
2. Claudian Obsidian 插件 Github 仓库：https://github.com/YishenTu/claudian
3. Obsidian Web Clipper 官方下载页面
4. 知识库规则配置文件`CLAUDE.md`（可联系教程作者获取）