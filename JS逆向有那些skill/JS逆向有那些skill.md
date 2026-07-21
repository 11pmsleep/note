> 来源链接：https://b23.tv/0adL3Iq

## 1. 学习目标
完成本内容学习后，可掌握以下知识点：
1.  理解AI辅助JS逆向的核心价值，以及AI Skill对传统人工逆向的效率提升逻辑
2.  掌握3个主流JS逆向开源Skill仓库的定位、核心能力与适用场景
3.  理解基于MCP工具的AI逆向工作流，涵盖源码定位、Hook调试、算法还原全流程
4.  了解Web端与App端AI逆向的差异，以及静态分析+动态分析的组合分析策略
5.  掌握JS逆向Skill的安装方式，可快速配置AI逆向工作环境

---

## 2. JS逆向AI Skill概述
传统JS逆向依赖人工阅读混淆代码、断点调试、手动还原加密逻辑，AI逆向Skill本质是将资深逆向工程师的经验沉淀为标准化规则，赋予AI专业逆向分析能力，可自动完成加密算法还原、代码混淆还原、反爬对抗等工作，大幅降低逆向技术门槛。
![](images/screenshot_000_83473fca-0b0b-43d9-9566-c279f7601f22.jpg)

---

## 3. 核心JS逆向Skill仓库详解
### 3.1 hello_js_reverse_skill（Web端逆向分析与接口签名还原）
该仓库是面向Web逆向分析与接口签名还原场景的成熟Skill，围绕`camoufox-reverse` MCP构建单工作流：先通过Camoufox反检测浏览器完成网络捕获、源码定位、Hook调试与检测验证，再落地到Node.js或Python实现算法还原与自动化调用。
![](images/screenshot_001_0af8519d-f51c-41fc-b5f7-e99db2ac521a.jpg)

#### 3.1.1 核心能力
1.  **双语言算法还原**：支持Node.js（`crypto` / `crypto-js`）与Python（`hashlib` / `pycryptodome`）双路径实现算法还原
2.  **加密算法覆盖**：支持MD5、SHA、AES、DES、RSA、HMAC、Base64等常见加密算法的纯法复现
3.  **混淆代码还原**：覆盖OB混淆、控制流扁平化、eval打包、自定义VM/SVM等主流混淆类型的分析策略
4.  **虚拟机保护追踪**：通过Hook栈、日志、源码级插桩四板斧，从行为层面追踪JSVMP虚拟机保护的签名生成链路，对主流反爬厂商的JS保护均有效
5.  **Cookie归因分析**：整合HTTP Set-Cookie与`document.cookie`日志，一次性解析Cookie的完整生成逻辑
6.  **反爬分桶决策**：区分签名型、行为型、混淆三类反爬场景，内置顶层决策框架，避免触发`observer effect`被反爬检测
7.  **动态Cookie破解**：支持eval加密、app加强、浏览器指纹等多种Cookie生成机制的逆向破解
8.  **WASM逆向**：支持WebAssembly加密模块的加载、环境补全和调用分析
9.  **协议层对抗**：支持TLS指纹检测、HTTP/2强制、频率限制等协议层防护的绕过
10. **反检测能力**：基于Camoufox C++引擎实现指纹级伪装，可绕过Cloudflare等主流反爬检测
11. **单工作流闭环**：统一使用`camoufox-reverse`完成源码搜索、Hook注入、函数追踪、网络分析、请求拦截与反检测验证

#### 3.1.2 设计规范
该Skill通过标准化定义约束AI行为，保障逆向分析的合规性与准确性：
1.  设定AI为专业全栈逆向工程师的角色，明确授权边界与行为规则
2.  定义最高优先级硬约束Checklist：激活Skill后，第一次调用任何MCP工具之前，必须复述Checklist，跳过该步骤视为违规
3.  对核心MCP工具做分类索引，定义标准化浏览器连接流程
![](images/screenshot_002_983693df-008b-450d-bea8-e3db88954a26.jpg)

#### 3.1.3 安装方式
- **推荐方式**：直接向AI编程助手（Cursor、Claude Code、CodeX等）发送安装指令，AI会自动完成下载、配置，后续JS逆向任务中会自动调用该Skill
- **手动安装**：将仓库克隆到对应AI工具的skills目录下即可
![](images/screenshot_003_16870378-4bbe-4fb1-986d-c5c29b7e5c00.jpg)

---

### 3.2 js-reverse-automation--skill（JS逆向自动化分析）
该仓库结合`chrome-devtools-mcp`的能力与Skill规范，实现基于`JSRPC+Flask+autoDecoder`方案的前端JS逆向自动化分析，大幅提升JS逆向的落地效率。
![](images/screenshot_004_c4151599-10f6-4bd5-a1a2-1eb4dcc57a2a.jpg)

#### 3.2.1 适用场景
1.  需要快速落地前端签名/加密参数逆向
2.  需要将JS逆向逻辑封装为可复用的工程代码
3.  需要与Burp Suite配合完成抓包、改包操作

#### 3.2.2 设计思路
针对JS逆向中常用的远程调用方案（如JSRPC+Mitmproxy、JSRPC+Flask等），将人工定位加密函数、编写注册代码、编写Python调用代码等繁琐操作，通过MCP与Skill技术赋能AI自动完成，实现从半自动化到高自动化的跨越：操作人员仅需下达分析指令，最终完成Burp配置即可走完JS逆向全流程。
![](images/screenshot_005_e5e76f31-c445-4440-ba6d-c5bd983d1091.jpg)

#### 3.2.3 核心能力
1.  基于MCP连接真实浏览器，触发并跟踪JS加密/签名完整链路
2.  自动定位`sign`、`enc`、`token`等关键参数的生成入口
3.  自动生成JSRPC注入与注册代码
4.  自动生成Python Flask代理代码
5.  输出Burp `autoDecoder`对接说明，支持端到端联调
6.  集成`AntiDebug.Breaker`的11项反调试对抗能力
![](images/screenshot_006_dc17605b-a4dc-468b-bc12-63dedebb877a.jpg)

---

### 3.3 reverse-skills（通用逆向工程技能集）
该仓库是通用逆向工程分析技能集，支持40+种AI编程工具，专为`IDA-NO-MCP`设计，可从IDA导出反编译结果后交由AI编程工具完成分析，主要覆盖安卓端逆向，也支持其他平台逆向场景。
![](images/screenshot_007_16780589-88b7-4b58-9aaa-f9ee5c802506.jpg)

#### 核心包含技能
| 技能名 | 功能说明 |
|--------|----------|
| `rev-symbol` | 从导出表/导入表或反编译代码分析函数符号 |
| `rev-struct` | 从反编译函数重建数据结构 |
| `rev-frida` | 使用现代Frida API生成动态插桩脚本 |
| `rev-unicorn-debug` | 使用Unicorn引擎模拟执行和调试指定代码片段 |
| `rev-dex-dumper` | 从运行中的Android应用内存中dump DEX文件，实现整体加固脱壳 |
| `rev-ildb-dump` | 从Unity IL2CPP构建中提取C#符号地址，生成IDA/Ghidra导入脚本 |
| `rev-ildapython` | 提供IDAPython/IDALib脚本参考，涵盖调试、内存操作、反编译、混淆辅助、批量分析等场景 |

---

## 4. AI辅助逆向的核心逻辑与优势
### 4.1 双分析模式
AI逆向结合两种分析模式覆盖全场景需求：
1.  **静态分析**：AI具备极强的代码阅读能力，可直接对解压后的安装包源码（无论是否经过混淆）进行静态分析，快速梳理代码逻辑链路
2.  **动态分析**：通过MCP工具调用浏览器/运行环境，执行代码并捕获运行时行为，获取运行时参数与逻辑，解决仅静态分析无法覆盖的运行时依赖场景

### 4.2 技术价值
传统逆向依赖人工逐行读代码、断点调试，AI逆向将该过程大幅简化，复用成熟逆向工程师的经验沉淀，大幅降低逆向技术门槛，同时提升分析效率。AI辅助逆向是具备高可行性的技术方向，未来会衍生出更多AI逆向的实现方案。
![](images/screenshot_008_bdefaefe-48c4-451d-ba7d-6203634a9fe2.jpg)

---

## 5. 安装使用说明
本视频介绍的所有Skill均无需手动完成安装配置操作，只需向AI助手下达安装对应Skill的指令，AI即可自动完成全流程安装与环境配置。详细操作步骤可参考往期对应安装教程。

---

## 6. References
本次介绍的相关开源仓库与工具如下：
1.  `js-url-auto-reverse`: https://github.com/14u9h/js-url-auto-reverse
2.  `hello_js_reverse_skill`: https://github.com/WhiteNightShadow/hello_js_reverse_skill
3.  `js-reverse-automation--skill`: https://github.com/Fausto-404/js-reverse-automation--skill
4.  `agent-skills-library`: https://github.com/christophacham/agent-skills-library
5.  `reverse-skills`: https://github.com/P4nda0s/reverse-skills
6.  `JSReverser` MCP工具: https://github.com/NoOne-hub/JSReverser-MCP