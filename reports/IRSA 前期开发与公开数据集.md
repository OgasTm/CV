# 公开数据够搭骨架，关键数据必须自己测

**结论先行：IRSA 的前期开发可以拆成五个阶段（0 治理与架构 → 1 物理台架 → 2 智能体离线评测 → 3 临床影子模式 → 4 干预与监管试点）。走到影子模式结束大约需要 30 个月、9–11 人年，核心团队 6–8 人【估算】。** 每个模块都能找到起步用的公开资源：MC 模拟有 OpenGATE 10，人体模型有 ICRP 体模，规则文本有美国 NRC 的公共领域文件，动力学先验有 Schütze、Kurth、Bailey 等人发表的汇总统计，原型剂量计可以直接买 Radiacode 103（US$269，BLE + MIT 许可的 Python 库），评测框架有 τ²-bench、HealthBench 和 AgentDojo（均为 MIT），端侧模型有 Gemma 4 和 Qwen3.5（Apache-2.0）。**但决定成败的三类数据，公开领域里都没有：** 患者佩戴式探测器在放射性药物治疗（RPT）后的逐患者原始时间序列；0.5 m 以内、多朝向的 177Lu 近场实测；放射防护患者对话基准。这些只能在阶段 1–3 自己采集、自己构建，或者向 Bailey、Levart、Kurth/Schütze 等作者索取。**许可证是第二条暗线。** 和 IRSA 最贴近的几项资源都是非商用或禁止演绎：开放的 177Lu 患者影像（Deep Blue）、POLYMOD 接触数据、UF/NCI 孕妇体模、PatientAgentBench、Levart 2025 原文。OpenRegulatory 模板已改为 CC BY-NC-SA，flutter_blue_plus 对营利用途收费并带构建期遥测。研究阶段这些都能用，但必须从第一天起分开登记"研究通道"和"产品通道"两类资产，否则将来商业化要返工。**最小可行原型只应包含五样东西：剂量计、半衰期滤波器、半解析剂量引擎、一个辖区的规则包、模板化表达加输出校验。** MC 代理模型留在离线台架上，人群数据和监管上报不进 v1。

> **阅读说明。** 标签含义如下。【估算】：本报告或研究笔记对工作量和周期的推算，没有文献依据。【推断】：推理得出的结论。【search-summary, verify】：只见过检索摘要，需要核对原文。【未核实】：许可证或细节本轮未能确认。【二手】：来自厂商博客、律所或媒体的转述。【预印本】：未经同行评审。许可证一律记录研究笔记在 2026-11-20 核对时的状态。"能否商用"是对许可证文字的工程解读，不构成法律意见，进入商业化之前需要法务复核。本报告沿用此前评估定下的设计原则：三模型对比、保形预测回退、数字只能来自引擎输出、自建 IRSA-Bench、默认保护隐私、v1 不做监管上报，试点顺序为德国院内 → 英美 177Lu 影子模式 → 日本。本报告不再重复那份评估的论证，只讨论怎么开发、用什么数据。

## 傻瓜版：IRSA 就是"量、算、查、说"四件事，外加一层隐私外壳

可以把 IRSA 想成患者出院后随身带着的"辐射管家"。下表用白话说明每块积木做什么，以及原型阶段最省事的做法。

| 积木 | 白话解释 | 原型阶段最省事的做法 |
|---|---|---|
| **量**（剂量计） | 患者胸前或手腕戴一个小型探测器，手机通过蓝牙定时读数。它最擅长测"药排得多快"，也就是个体有效半衰期。绝对剂量水平靠出院时医院用校准仪器在 1 m 处测的那一次读数锚定 | 买 2–3 台 Radiacode 103，用开源的 `cdump/radiacode` 库读数据（[GitHub](https://github.com/cdump/radiacode)） |
| **算**（剂量引擎） | 把"排得多快 + 出院读数 + 体型 + 家人离多近、待多久"换算成"家人会受到多少剂量"。三套算法同时跑、互相对照：NRC 默认公式、半解析模型、MC 训练的 AI 代理模型 | 先写半解析模型，再用 GATE 10 离线生成训练数据（[OpenGATE](https://github.com/OpenGATE/opengate)） |
| **不确定性**（保形预测） | 代理模型不只给一个数，还给一个"有 90% 把握落在这里"的区间。输入超出训练范围时，自动退回保守算法 | MAPIE 或 crepes（BSD 许可）（[MAPIE](https://github.com/scikit-learn-contrib/MAPIE)） |
| **查**（规则引擎） | 把算出的数字和当地法规比对，得出"几天内不要和 5 岁以下孩子近距离接触"这类结论。规则写成带版本号、可测试的代码，不交给 LLM 去检索 | 用 json-logic 或 OPA/Rego 写第一个辖区的规则包（[json-logic-js](https://github.com/jwadhams/json-logic-js)） |
| **说**（LLM 对话层） | 只做两件事：把引擎的结论翻译成患者听得懂的话；在引擎缺信息时，按引擎的要求追问（"家里有孕妇吗？"）。它不得自己生成任何数字，校验器逐个核对文本里的数字和许可，对不上就换成固定模板 | 先只用模板，之后再接 Gemma 4 E4B 等端侧模型做对比（[Gemma 4](https://huggingface.co/google/gemma-4-E4B-it)） |
| **情境**（情境模块） | 了解患者在哪类场所、和谁接触多久。以患者自述为主，地图和人口数据只作背景 | 开放的 POI 和建筑底图离线放在手机里，不调用在线地图 API |
| **外壳**（隐私） | 计算和对话默认都在手机上完成，位置不上传，v1 不向监管机构上报 | 端侧审计日志，用哈希链防篡改 |

五个阶段可以用一句话概括。时间是估算，门槛的具体数值要在阶段 0 预先登记。

| 阶段 | 白话目标 | 时间【估算】 | 过关才继续；不过关怎么办 |
|---|---|---|---|
| 0 治理与架构 | 先写清楚"做什么、不做什么"，把合规当作设计输入 | 第 0–4 月 | 预期用途冻结、DPIA 证明端侧处理可行、许可证登记里没有 AGPL 或 BSL 组件。不过关就不写产品代码 |
| 1 物理台架 | 在实验室里用体模证明"身上的读数能换算成别人受到的剂量" | 第 3–12 月 | 不过关：调整佩戴位置或传感器；如果代理模型不比半解析模型强，就把它从产品路径中拿掉 |
| 2 智能体离线评测 | 在电脑里用模拟患者反复考对话层，直到不出错 | 第 9–18 月 | 不过关：关掉 LLM，只用模板输出 |
| 3 临床影子模式 | 真实患者佩戴设备，但 App 的建议不给患者看，事后和照护者佩戴的 TLD 对账 | 第 18–30 月 | 不过关：不进入干预试点 |
| 4 干预与监管试点 | 让建议真正影响行为，走认证 | 第 30 月以后 | 超出前期开发范围 |

## 开发拆解：约 30 个月、9–11 人年走到影子模式终点

笔记没有找到任何关于学术界 SaMD 团队规模或工作量的同行评审数据。现有的指南只说学术团队通常人少、流动性大、软件工程背景参差不齐，所以只需落实 IEC 62304 的一个精简子集，变更管理交给固定员工负责（[Hauschild 2022](https://pmc.ncbi.nlm.nih.gov/articles/PMC9685384/)）。下面的拆解是研究笔记依据法规要求和工程范围推导出来的，**所有人月数字都是估算**。咨询公司给出的外部成本可作参照：MDR IIa 类软件认证的外部花费约 €50–150k，技术文档备齐后还要等 3–9 个月的公告机构审核（[OpenRegulatory 成本](https://openregulatory.com/questions/how-much-does-medical-device-compliance-cost)；[OpenRegulatory 周期](https://openregulatory.com/questions/how-long-does-medical-device-compliance-take)）。英国有来源估计，首次为 II 类软件取得 CE 标志最高约需 £250,000（[Health Innovation Network](https://healthinnovationnetwork.com/projects/what-are-the-timelines-and-costs-for-medical-device-regulation/)）。这些数字来自咨询方，可能偏向其自身利益。

### 五个阶段的工作包、交付物和门槛

| 阶段与时间 | 工作包 | 交付物 | 主要角色 | 人月【估算】 | 通过/停止门槛 |
|---|---|---|---|---|---|
| **0 治理与架构**（第 0–4 月） | WP0.1 预期用途与声明矩阵（做什么、不做什么）；WP0.2 MDR Rule 11 预分类、AI Act 高风险判定；WP0.3 DPIA（GDPR 第 35 条）与端侧隐私架构；WP0.4 开发计划，决定安全等级；WP0.5 风险管理计划与初版危害日志（ISO 14971 + BS/AAMI 34971 + LLM 危害 + IEC 81001-5-1 威胁模型）；WP0.6 SOUP 与许可证登记；WP0.7 规则即代码的 schema，以及首个辖区规则包（附法规原文出处）；WP0.8 仓库、CI、DVC、MLflow 脚手架；WP0.9 发出数据与许可申请（见"关键数据空白"一节）【推断】 | D0.1 预期用途与声明；D0.2 DPIA；D0.3 架构与 SOUP 清单；D0.4 风险计划与危害日志 v0；D0.5 规则包 v0.1 及测试向量 | PI/临床负责人、法规/QA、医学物理师、DPO | 18–24 | **G0**：预期用途冻结为 v1；没有缺少缓解思路的"不可接受"危害；DPIA 证明端侧处理可行；许可证登记干净，交付路径中没有 AGPL 或 BUSL 组件 |
| **1 物理台架**（第 3–12 月） | WP1.1 半解析模型；WP1.2 GATE 10 MC 计算（131I 和 177Lu 体模，多种几何与屏蔽）；WP1.3 代理模型训练、保形校准与 OOD 检测；WP1.4 体模实测，参考巡测仪和 EPD；WP1.5 剂量计表征（能量响应、线性、角响应、BLE 数据包完整性）；WP1.6 三模型对比（半解析、代理、MC、实测）；WP1.7 动力学先验与合成读数轨迹生成器【推断】 | D1.1 物理验证报告；D1.2 代理模型卡与覆盖率报告；D1.3 剂量计表征报告 | 医学物理师、ML 工程师、嵌入式/移动工程师 | 约 36 | **G1**：相对体模实测达到预先登记的精度目标；留出几何上的经验保形覆盖率不低于名义水平；预定义的 OOD 用例都能触发回退。**停止或简化**：如果代理模型不优于半解析模型，产品路径只保留半解析模型；如果体表读数追踪不到有效半衰期趋势，先改传感器方案，再考虑进入阶段 3【推断】 |
| **2 智能体离线评测**（第 9–18 月） | WP2.1 编排器、类型化工具、校验器与模板回退；WP2.2 IRSA-Bench 场景语法（家庭构成、核素、语言、地图文本中的对抗注入、模糊回答、红旗症状）；WP2.3 指标（槽位保真、经校验后的数字幻觉率目标为 0、追问是否恰当、可读性、抗注入、升级转人工的召回率）；WP2.4 按 IEC 62366-1 做一轮形成性可用性研究；WP2.5 端侧 LLM 与纯模板对比 | D2.1 IRSA-Bench v1（公开发布，CC BY/Apache）；D2.2 智能体验证报告；D2.3 形成性可用性报告；D2.4 冻结的临床研究版本及验证证据 | LLM/评测工程师、HCI 研究员、移动工程师 | 30–36 | **G2**：基准测试中没有任何未经校验的数字到达界面；注入测试全部通过；与危害相关的任务没有关键使用错误；风险文件表明剩余风险对观察性研究可以接受。**停止**：任何一条不满足，LLM 层就关闭，只输出模板【推断】 |
| **3 临床影子模式**（第 18–30 月） | 按 ISO 14155:2026 或国家对 MDR 第 82 条的要求撰写临床研究方案（CIP）；伦理审批，按要求通知主管机关；中心启动与培训；招募；照护者 TLD/EPD 的物流；数据监查，AE 与器械缺陷处理；分析（预测与实测照护者剂量的一致性、区间校准、规则引擎与临床医嘱的一致性、App 可靠性含 BLE 断档统计） | D3.1 CIP 与伦理批件；D3.2 临床研究报告；D3.3 更新后的风险文件与上市后监督类经验总结 | 临床研究护士/协调员、PI、医学物理师 | 24–36，另加临床费用 | **G3**：预设的一致性和校准终点达标；没有严重的器械相关事件；BLE 数据完整率高于阈值；确定法规路线（法定制造商、公告机构、2028 年 AI Act 就绪） |
| **4 干预与监管试点**（第 30 月以后） | 超出前期开发范围。需要有 ISO 13485 资质的法定制造商、公告机构沟通计划，并满足 2028-08-02 起适用的 AI Act 附件 I 义务 | — | — | 另计；外部认证花费约 €50–150k 以上 | — |

以上合计约 **108–132 人月，即 9–11 人年**；笔记较宽的口径是 8–12 人年、30–36 个月。如果剂量计硬件不用现成产品，还要再加 0.5 FTE 的嵌入式工程师【估算】。

阶段 3 的人群和地区沿用既定的试点顺序。第一步在德国强制住院的 48 h 内做院内配对：德国本来就要求出院时测量 2 m 处剂量率（[Kurth 2018](https://link.springer.com/article/10.1186/s13550-018-0386-4)），这一步不涉及门诊风险。第二步到已允许门诊治疗的英国或美国 177Lu 中心做影子模式（英国门诊先例见 [PMC6484059](https://pmc.ncbi.nlm.nih.gov/articles/PMC6484059/)，美国门诊可行性研究见 [PMC11619580](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11619580/)），以照护者连续佩戴四周的 TLD 作为参考标准。第三步到日本做监管共建，日本 JSCO 已认可按患者特异累积剂量计算出院（[Int J Clin Oncol 2025](https://link.springer.com/article/10.1007/s10147-025-02858-3)）。研究笔记的阶段 3 草案写的是"131I 甲状腺和 177Lu 两个队列"，这一点和既定原则冲突：在污染模型独立验证之前，应避开高活度 131I 甲状腺癌治疗，131I 只适合在门诊甲亢患者中做可用性试点【推断】。

### 团队：6–8 人，多数兼职，变更管理交给固定员工

| 角色 | 投入【估算】 | 职责 |
|---|---|---|
| PI/临床负责人（核医学） | 0.1–0.2 FTE | 预期用途、临床问题、伦理 |
| 医学物理师（辐射防护/剂量学） | 1.0 FTE | 剂量模型、体模台架、规则内容、MC |
| ML 工程师 | 阶段 1–2 为 1.0 FTE，之后 0.5 | 代理模型、保形预测、OOD、半衰期估计 |
| 移动/BLE 工程师（兼固件对接） | 阶段 1 起 1.0 FTE；硬件自研时另加嵌入式 0.5 FTE | App、BLE、端侧引擎移植 |
| LLM/评测工程师 | 阶段 1–3 为 1.0 FTE | 编排器、校验器、IRSA-Bench、红队测试 |
| 法规/QA 负责人 | 0.3–0.5 FTE，可与机构共享或外聘 | 精简 QMS、风险文件、62304/14971/62366/81001-5-1、与 DPO 合作 DPIA |
| HCI/可用性研究员 | 0.5 FTE | 62366-1 形成性研究，多语言健康素养 |
| 临床研究护士/协调员 | 阶段 3–4 为 0.5–1.0 FTE | 招募、TLD 物流、AE 报告 |
| DPO/信息安全 | 按需 | 数据保护与安全评审 |

### 最小可行原型：一条可审计的确定性链路，LLM 可有可无

最小可行原型（MVP）要回答的问题只有一个：在不靠 AI 代理模型、也不靠 LLM 的情况下，"可穿戴读数 + 规则"能不能给出可审计、可校验的个体化建议。代理模型和 LLM 都是在这条链路上叠加的增量，需要用阶段 1、2 的数据证明自己的价值【推断】。

| MVP 包含 | MVP 不包含（以及原因） |
|---|---|
| Radiacode 103 + 手机端 BLE 采集，同时记录剂量率和能谱 | 自研剂量计硬件：先用现成设备验证信号，再决定是否自研 |
| 半衰期估计：状态空间滤波器（filterpy），先验取自 Schütze、Kurth、Bailey、Turku 的发表数据 | App 内的 MC 代理模型：先在阶段 1 离线完成三模型对比，再决定是否上车 |
| 出院时 1 m 校准读数作为绝对锚点 | 人群拥挤度和实时人流：官方 API 拿不到，误报风险高 |
| NRC 默认基线 + 半解析剂量引擎（线源 × 体衰减 × 实测有效半衰期） | 位置追踪和监管上报：v1 明确不做 |
| 一个辖区的规则包（json-logic，带版本号和测试向量） | 把真实患者数据发给云端 LLM：阶段 0–3 都禁止 |
| 模板化表达 + 数字/许可校验器 + 哈希链审计日志 | 多辖区并行、225Ac、排泄和污染剂量建模（卫生指导由规则引擎给出） |
| 可选：端侧 LLM 表达层，放在校验器之后，模板始终兜底 | 公开上架应用商店：阶段 3 通过 TestFlight、Play 封闭测试或 MDM 分发 |

### 技术栈：几乎每个组件都有宽松许可的选项

| 层 | 首选 | 备选 | 许可证 | 要点 |
|---|---|---|---|---|
| 半衰期与贝叶斯先验 | filterpy（端侧在线更新） | PyMC、NumPyro（离线层级先验） | MIT；Apache-2.0 | 只有滤波器需要移植到手机（[PyPI filterpy](https://pypi.org/project/filterpy/)） |
| 保形与 OOD | MAPIE、crepes | puncc；pytorch-ood | BSD-3、BSD；MIT；Apache-2.0 | 避开 Alibi Detect（BUSL）；TorchCP 是 LGPL，不要静态链接进 App（[PyPI mapie](https://pypi.org/project/mapie/)） |
| MC | OpenGATE 10 + radot | OpenMC、PHITS 交叉校验 | LGPL-3.0；MIT；JAEA 许可 | 只作为离线数据生成工具使用，不链接进产品（[opengate](https://pypi.org/project/opengate/)） |
| 影像与分割 | TotalSegmentator（开放任务）、PyTomography | PyTheranostics | Apache-2.0；MIT；许可证有冲突 | 部分 TotalSegmentator 子任务需要商业许可（[TotalSegmentator](https://github.com/wasserth/TotalSegmentator)） |
| 规则即代码 | 用 Catala 或 YAML DSL 撰写，编译成 json-logic 在端侧运行 | 服务器端用 OPA/Rego | Apache-2.0；MIT | App 中避开 OpenFisca（AGPL）（[Catala](https://github.com/CatalaLang/catala)） |
| 编排与护栏 | 确定性状态机 + Pydantic 类型化槽位 + Guardrails AI 校验器 | NeMo Guardrails、LlamaFirewall、LLM Guard | MIT；Apache-2.0；MIT | 不可信文本按 CaMeL 的思路隔离（[CaMeL](https://github.com/google-research/camel-prompt-injection)） |
| 结构化输出 | Outlines、XGrammar | llguidance | Apache-2.0；MIT | 小模型在语法约束下才能稳定输出 JSON |
| 工具暴露 | 进程内调用；需要服务边界时用 MCP 2026-07-28 无状态协议 | — | MCP Python SDK MIT | 用 `outputSchema` 写明单位，用 OpenTelemetry 传递追踪信息（[MCP changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog.md)） |
| 端侧模型 | 模板基线 + Gemma 4 E4B 或 Qwen3.5 小尺寸 | Apple Foundation Models、Gemini Nano（系统模型，无法锁定版本） | Apache-2.0 | 运行时可选 LiteRT-LM、llama.cpp、MLC、ExecuTorch |
| 评测 | Inspect AI + τ²-bench `irsa` 域 + HealthBench 评分细则格式 + AgentDojo `irsa` 套件 | promptfoo（CI 回归）、Ragas/DeepEval | 全部 MIT 或 Apache-2.0 | 安全关键项要求 Pass^k 全部安全 |
| 移动端 | Kotlin Multiplatform + Kable + SwiftUI/Compose | Flutter + flutter_reactive_ble | Apache-2.0；BSD | 不用 flutter_blue_plus（[FBP LICENSE](https://raw.githubusercontent.com/chipweinberger/flutter_blue_plus/master/LICENSE.md)） |
| 数据与实验管理 | DVC + MLflow | — | Apache-2.0 | 登记 MC 宏、几何、种子、GATE/Geant4 版本 |
| 医院导出 | FHIR R4，与 IPS/EPS 对齐 | HAPI FHIR、android-fhir、fhir.resources | Apache-2.0；BSD | 暂不采用 R6，它还没有定稿 |

### 影子模式的法规路径：欧盟走 MDR 第 82 条，美国争取 NSR

在欧盟，"IRSA 与标准诊疗并行运行，输出不给患者看，事后与照护者 TLD 和临床判断对比"这种影子模式研究，最可能定性为 MDR **第 82 条**临床研究，即不以符合性评价为目的的研究。它适用国家法律，需要伦理委员会意见，**器械本身不需要 CE 标志**（[Fimea](https://fimea.fi/en/medical-devices/investigations-with-devices/clinical-evaluation-and-clinical-investigations)；[BfArM](https://www.bfarm.de/EN/Medical-devices/Tasks/Clinical-investigations-and-performance-studies/Clinical-investigations/Notification-other/_node.html)）。如果打算把这批数据放进将来的 CE 技术文档，就应从一开始按第 62 条执行。院内器械豁免（第 5(5) 条）不适用：MDCG 2023-1 明确写道，患者在医疗机构之外使用的 App 不属于院内器械（[MDCG 2023-1](https://health.ec.europa.eu/system/files/2023-01/mdcg_2023-1_en.pdf)）。ISO 14155 第 4 版已于 2026-03-23 发布，新增了剩余风险评估、临床事件委员会等要求（[NSF](https://www.nsf.org/life-science-regulatory-news/the-fourth-edition-of-iso-14155-was-published-on-march-23-2026-and-replaces-the-2020-version)）。按 MDCG 2025-6，MDR 临床研究同时算作 AI Act 第 60 条的真实世界测试，不需要另走沙盒程序（[Hogan Lovells](https://www.hoganlovells.com/en/publications/mdcg-published-new-guidance-on-the-interplay-between-the-mdr-ivdr-and-the-ai-act)）。在美国，无创、观察性、不改变诊疗的软件研究，是"非重大风险（NSR）"判定的典型候选，适用简化 IDE 要求，只需 IRB 监督（[UCSF](https://compliance.ucsf.edu/your-study-ide-exempt-or-non-significant-risk-nsr)）。FDA 2026-01-06 修订的 CDS 指南主要面向医护人员，面向患者的辐射防护建议大概率仍属于器械功能【推断】（[DLA Piper](https://www.dlapiper.com/insights/publications/2026/01/fda-updates-its-clinical-decision-support-and-general-wellness-guidances-key-points)）。工程上的直接要求是：研究版本必须冻结（构建哈希固定），附带风险分析、验证报告和面向研究者的使用说明，App 内要有器械缺陷上报按钮和审计日志。

### 设计历史：从第一天起维护十份 git 原生文件

精简 QMS 的核心是一套随代码演进的设计历史。研究笔记建议保留以下十份"活文件"【推断】。（1）预期用途声明与用户画像，可同时用于 MDR 附件 II、IEC 62366-1 使用规范和 AI Act 第 13 条。（2）2–3 页的软件开发计划，写明安全等级。（3）带编号、可追溯到测试的需求清单。（4）架构说明和 SOUP/OTS 清单，记录版本、许可证和已知异常，端侧 LLM 和 ML 模型也要列入。（5）风险管理文件：ISO 14971 危害日志 + BS/AAMI 34971 的 ML 危害（数据集偏移、OOD、过度信任）+ LLM 危害（编造数字、来自地图文本的提示注入）+ 81001-5-1 威胁模型。（6）数据管理计划和数据集说明书，按 GMLP 原则记录来源、训练/测试独立性和可复现性。（7）代理模型的模型卡。（8）CI 自动生成、按发布标签归档的验证记录。（9）可用性记录。（10）发布说明，包括剩余异常和签名提交的变更日志。安全等级方面，按现行 IEC 62304 必须假设软件失效概率为 1，所以 **B 级是现实的底线**；如果危害分析认为软件失效可能导致孕妇等接触者受到严重伤害，C 级也说得通，这个判断要在阶段 0 定下来并写进风险文件（[Johner](https://blog.johner-institute.com/iec-62304-medical-software/iec-62304-2nd-edition-all-areas-of-application-and-changes/)）。LLM 表达层的失效可以被校验器检出，这是把它排除在安全关键路径之外的最有力论据，也可能借此为该部分争取较低的过程严格度（隔离论证，需与公告机构确认）【推断】。

## 模块 × 数据/工具总表：每块积木都有起步资源，但没有一块是现成的

下表把 IRSA 的模块和最重要的公开资源对应起来。"缺什么"一列就是阶段 1–3 必须自己补的部分，详细清单见后文各小节。

| 模块 | 需要什么 | 现成可用的首选资源 | 公开领域缺什么 | 主要许可风险 |
|---|---|---|---|---|
| 剂量计硬件与校准 | 带 BLE 和开放接口的 γ 探测器；177Lu/131I 能量响应；校准源 | Radiacode 103 + `cdump/radiacode`（MIT）；IEC 61526:2024；137Cs/133Ba | Radiacode 在 113/208/364 keV 的能量和角响应测试；认证 EPD 的公开 BLE 协议 | Open Gamma Detector 与 G4Radiacode 为 GPL-3.0；两项居家剂量计专利需做 FTO |
| 读数反演与动力学 | 有效半衰期先验、双指数结构、排尿阶跃模型 | Schütze 2026、Kurth 2018、Monserrat Fuertes 2023、Bailey 2026、Turku 2024、Levart 2019 | 逐患者原始剂量率时间序列；佩戴式探测器真实轨迹 | Fele-Paranj PBPK 代码无许可证 |
| 外照射剂量引擎 | Γ 常数、衰减系数、积累因子、转换系数、体衰减因子 | ICRP 107/116/74、NIST XCOM、RG 8.39 Rev. 1、Bellamy 2022 | 177Lu 的开放 Γ 表（需购买 Smith & Stabin）；开放的现代积累因子表 | ICRP 数据可免费下载，再分发需许可；ANS-6.4.3 需付费 |
| MC 与代理模型 | MC 引擎、人体与接触者体模、外照射训练数据 | OpenGATE 10 + radot；ICRP 110/143/145；UF/NCI；TotalSegmentator；EURADOS 数据集 | "活度分布 → 接触者位置剂量场"的 177Lu 数据集 | UF/NCI 体模仅限非商用；Deep Blue 为 CC BY-NC；EGSnrc 为 AGPL |
| 不确定性与 OOD | 保形回归、预测分布、OOD 检测 | MAPIE、crepes、pytorch-ood | 外照射代理模型的公开覆盖率基准 | Alibi Detect 为 BUSL；TorchCP 为 LGPL |
| 规则引擎 | 各辖区放行与限制条款、规则运行时 | 10 CFR 35.75、RG 8.39 Rev. 1、DG-8061/DG-8070、SSK、BAG、JSNM/MHLW；json-logic、OPA、Catala | DG-8070 全文；德国和日本现行条款的综合文本 | IAEA/ICRP 文本再利用需许可；OpenFisca 为 AGPL |
| 情境：接触先验 | 谁、多近、多久 | Gambia 2022 接触调查（CC BY）；Levart 2025 日记汇总；NRC SECY-18-0015 场景 | 同时记录真实距离和日常行为的家庭数据 | POLYMOD 与 SocioPatterns 为 CC BY-NC |
| 情境：地点与人口 | POI 类别、建筑、人口密度 | Overture、Foursquare OS Places、Google/Microsoft 建筑、GHSL/WorldPop/LandScan | 官方渠道的实时人流 | ODbL 共享相同方式；Google Maps 内容不可缓存 |
| 情境：户型与动作 | 房间间距、同床同室、如厕/卧床/乘车事件 | Swiss Dwellings（CC BY）；DHS hv216；Capture-24；ExtraSensory | 带家具位置的户型；LMIC 户型；佩戴辐射探测器时的动作数据 | CubiCasa5K 为 CC BY-NC-SA；ZInD 仅限学术用途 |
| LLM 知识库 | 可入库的指南和法规文本 | NRC/eCFR（公共领域）；EANM/SNMMI 177Lu-PSMA 指南（CC BY 4.0） | 已获授权的患者语言材料 | IAEA、ICRP、HPS、SNMMI、NHS 文本均需许可 |
| 评测与模拟器 | 对话基准、模拟患者、LLM 评审 | τ²-bench、HealthBench、AgentDojo、Inspect AI、MediQ | 放射防护患者对话基准；评审与专家一致性数据 | PatientAgentBench 为 CC BY-NC；MedPRESS 仅限研究；MedCalc-Bench 为 CC BY-SA |
| 护栏与工具层 | 校验器、结构化输出、注入隔离、MCP | Guardrails AI、Outlines、CaMeL、LlamaFirewall、MCP SDK | 医疗场景下的提示注入基准数据 | PurpleLlama 顶层仓库为 Llama 3.2 社区许可 |
| 开源模型 | 端侧/自托管、多语言 | Gemma 4、Qwen3.5（Apache-2.0） | 放射防护沟通场景的模型评测 | MedGemma 适用 HAI-DEF 条款；Aya 为 CC BY-NC；Llama 4 为自定义许可 |
| 移动端与 BLE | 跨平台框架、BLE 库、后台保活、FHIR | KMP + Kable；Nordic BLE 库；HAPI FHIR | 辐射防护或患者剂量学的 FHIR IG | flutter_blue_plus 营利用途收费并带遥测 |
| 标准与模板 | 生命周期、风险、可用性、网络安全、AI | IEC 62304、ISO 14971、BS/AAMI 34971、IMDRF N88；OpenSaMD 模板（CC BY 4.0） | IEC 62304 第 2 版（尚未发布） | OpenRegulatory 现行模板为 CC BY-NC-SA |

## 分模块详细清单

### 物理与影像数据：开放的 177Lu 多时间点患者影像只有 2 例

开放的 177Lu 患者 SPECT/CT 多时间点数据只有一套：SNMMI 剂量学挑战赛使用的 2 例 177Lu-DOTATATE 数据，托管在 Deep Blue。它包括投影、重建、时间积分活度（TIA）图、VOI、平面全身像和治疗前 Ga-68 PET。挑战赛共有 178 人报名，收到 119 份提交（[Uribe JNM 2021](https://jnm.snmjournals.org/content/62/Supplement_3/36S)）；参赛者结果的离散度约为器官 20%、肿瘤超过 25%（[JNM 2026;67(8):1198](https://jnm.snmjournals.org/content/67/8/1198)）。PSMA PET 数据量大，**PSMA-PET-CT-Lesions 共 597 例、117 GB**，在 TCIA 上是 CC BY 4.0，可以商用。笔记**没有找到**开放的 177Lu-PSMA SPECT/CT、131I SPECT/CT、225Ac 影像，也没有找到规模可观的 SSTR-PET 数据集。FNIH PDIB 项目表示将来会发布多时间点 SPECT/CT 数据，但目前还没有（[JNM 2026](https://jnm.snmjournals.org/content/67/8/1198)）。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| Lu-177 DOTATATE Multi-Time Point SPECT/CT（Dewaraja & Van） | 重建 SPECT 及配准 CT，治疗后一周内 4 个时间点，Siemens Intevo，DICOM | 2 例 × 4 个时间点 | CC BY-NC 4.0，不可商用（**冲突**：动力学笔记读到的 DataCite rights 字段为空，落地页返回 403） | Deep Blue 公开下载 | [doi:10.7302/0n8e-rz46](https://doi.org/10.7302/0n8e-rz46) | 反演先验、MC 源分布、验证（由 4 个时间点求有效半衰期） | 按非商用处理，并在落地页人工确认许可证 |
| 同系列另 5 个记录：投影与 µ 图、TIA 图、治疗前影像（含 Ga-68 PET）、器官与病灶 VOI、多时间点平面全身像 | 同上，配套数据 | 各 2 例 | CC BY-NC 4.0 | Deep Blue | [ctcw-6s13](https://doi.org/10.7302/ctcw-6s13)；[9nct-bk44](https://doi.org/10.7302/9nct-bk44)；[vqmy-g059](https://doi.org/10.7302/vqmy-g059)；[vhrh-qg23](https://doi.org/10.7302/vhrh-qg23)；[y4xd-s758](https://doi.org/10.7302/y4xd-s758) | 平面全身像时间序列最接近"躯干探测器读数"，可用来模拟佩戴式读数；TIA 图可作 MC 累积源 | 文件大小因 Cloudflare 拦截无法读取 |
| Lu-177 患者 CT 与勾画（Lim & Dewaraja） | Intevo 低剂量 CT，放射科医生勾画的器官和病灶 | 例数未知 | CC BY-NC 4.0 | Deep Blue | [doi:10.7302/864r-tb45](https://doi.org/10.7302/864r-tb45) | 反演（患者几何） | 低剂量 CT，图像质量有限 |
| DblurDoseNet 虚拟患者数据集 | 14 例 Ga-68 DOTATATE PET/CT 作活度图；SIMIND 模拟的 177Lu SPECT；MC 剂量率真值 | 14 例虚拟患者 | CC BY-NC 4.0 | Deep Blue | [doi:10.7302/ykz6-cn05](https://doi.org/10.7302/ykz6-cn05) | 代理模型（配对训练数据） | 体内吸收剂量率，不是外照射 |
| MRTDosimetry（EMPIR 15HLT06/19SIP01） | EURAMET MRT 剂量学项目数据，含用于 177Lu 研究的四器官体模 | 未说明 | CC BY 4.0（物理笔记，据 DataCite）；动力学笔记标为【未核实】 | OSF 公开 | [doi:10.17605/OSF.IO/69NGE](https://doi.org/10.17605/OSF.IO/69NGE) | 验证、定量 SPECT 校准 | 只有体模，没有患者 |
| 177Lu-DOTATATE 空间分辨率研究的派生数据（Ogino、Daisaki 等，2026） | 四器官体模在 1、4、24、40、72、144 h 的无噪声活度真值，128³ 体素 | 6 个时间点，1.2 MB | 数据 CC BY 4.0，代码 Apache-2.0 | Zenodo | [doi:10.5281/zenodo.24343109](https://doi.org/10.5281/zenodo.24343109)（动力学笔记记为 [24343110](https://doi.org/10.5281/zenodo.24343110)，可能一个是概念 DOI、一个是版本 DOI） | 合成器单元测试的已知真值 | 2026-11-18 发布，论文尚未经同行评审 |
| 比利时 8 家医院 13 台 SPECT/CT 的 Lu-177 体模扫描（Claeys 等） | 均匀圆柱与 NEMA 体模，活度可溯源至基准 | 13 台系统 | CC BY 4.0 | Zenodo；KU Leuven RDR | [zenodo.17897043](https://doi.org/10.5281/zenodo.17897043)；[V5KQMX](https://doi.org/10.48804/V5KQMX) | 验证（不同扫描仪之间的定量不确定度） | 只有体模 |
| CZT 相机 177Lu 剂量学处理数据（Banda，比勒陀利亚大学） | GE StarGuide 体模校准与回收系数；9 例回顾性患者的处理结果 | 4.57 MB | CC BY 4.0 | UP Research Data | [doi:10.25403/upresearchdata.32744928](https://doi.org/10.25403/upresearchdata.32744928) | 验证 | 没有患者图像 |
| 177Lu-SPECT 深度学习部分容积校正的 MC 数据集（Leube 等） | 随机活度分布与 MC 模拟 SPECT 配对 | 未说明 | CC BY 4.0 | Zenodo | [zenodo.8282567](https://doi.org/10.5281/zenodo.8282567) | 代理模型、反演 | 合成数据 |
| Radboud 唾液腺癌 177Lu-PSMA 试点 | 研究数据，293 个文件 | 15 MB | 受限访问 | 向 Radboud 申请 | [doi:10.34973/60e0-k133](https://doi.org/10.34973/60e0-k133) | 验证（待定） | 内容不可见 |
| Vivli：I-131 tositumomab 多中心药代与剂量学 | 临床试验数据包 | 未说明 | Vivli 数据使用协议 | vivli.org 申请 | [doi:10.25934/00000313](https://doi.org/10.25934/00000313) | 验证（131I 全身滞留） | 抗体动力学与 NaI 及小分子 RPT 不同 |
| **PSMA-PET-CT-Lesions**（LMU；autoPET III/IV 训练数据） | 378 名男性的 597 例全身 PSMA PET/CT，人工 3D 病灶掩膜 | 117.08 GB，1,791 个序列 | **TCIA 上为 CC BY 4.0，可商用；Tübingen FDAT 镜像标为 CC BY-NC 4.0（冲突）** | TCIA Data Retriever 或 IDC | [TCIA](https://www.cancerimagingarchive.net/collection/psma-pet-ct-lesions/)；[FDAT 镜像](https://doi.org/10.57754/fdat.gabc0-3hv03) | 反演先验（177Lu-PSMA 摄取模式）、配对 CT | 用 TCIA 副本并留存许可证记录；v2（2026-02-26）修正了 5 例掩膜 |
| FDG-PET-CT-Lesions | 501 例阳性 + 513 例阴性对照 | 约 1,014 例 | DataCite 同时列出 CC BY 4.0 和 TCIA 受控访问政策（**冲突**） | TCIA | [TCIA](https://www.cancerimagingarchive.net/collection/fdg-pet-ct-lesions/) | 患者体型 CT | 商用前在 TCIA 页面确认许可证 |
| DEEP-PSMA（Peter MacCallum） | 100 例 177Lu-PSMA 治疗前 mCRPC 患者，PSMA 与 FDG PET/CT 配对，全肿瘤负荷勾画 | 100 例 | CC BY-NC 4.0；1 例样例为 CC BY | 大概率需申请（未确认） | [zenodo.15281784](https://doi.org/10.5281/zenodo.15281784)；[样例](https://doi.org/10.5281/zenodo.18150034) | 与 177Lu-PSMA 最匹配的先验队列 | 访问方式未确认 |
| autoPET/CT IV | 全身 PET/CT 病灶分割，纵向 CT | 基于上述数据集 | 随底层数据集 | grand-challenge 注册 | [autoPET IV](https://autopet-iv.grand-challenge.org/) | 预训练分割模型 | 纵向 CT 未注明许可证 |
| **TotalSegmentator CT v3.0.0** | 1,939 例临床 CT，117 个结构 | 大，提供镜像 | CC BY 4.0 | Zenodo | [zenodo.6802613](https://doi.org/10.5281/zenodo.6802613) | 患者几何、接触者体模 | 不是 RPT 专用数据；另有 MRI 版 1,296 例（[zenodo.11367004](https://doi.org/10.5281/zenodo.11367004)） |
| TotalSegmentator 软件 | "total" 等开放任务；部分子任务（如身高体重预测）需许可 | PyPI v2.18.0 | 开放任务 Apache-2.0；授权子任务商用需另购许可 | `pip install totalsegmentator` | [GitHub](https://github.com/wasserth/TotalSegmentator) | 患者几何 | 用 `totalseg_info --list-tasks` 查看每个任务的许可证 |

### 体模与核数据：ICRP 能免费下载，但不能随意再分发

接触者体模的最短路径是：成人用 ICRP 110 体素和 ICRP 145 网格体模，儿童用 ICRP 143，孕妇和不同 BMI 用 UF/NCI 体模库。ICRP 的原话是"保留所有出版物的版权，即使免费获取的出版物，再利用也需许可"（[ICRP](https://www.icrp.org/page.asp?id=5)）。因此**部署出去的代理模型只能带训练好的权重，不能带体模文件**【推断】。ICRP 145 自带 Geant4、PHITS、MCNP6 格式的输入文件，用 PHITS 或 Geant4/GATE 生成可调姿态的外照射训练数据成本最低。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| ICRP 110 成人参考体素体模 | 参考男性、女性 | 2 个 | 免费下载；再利用需 ICRP 许可 | icrp.org | [ICRP 110](https://www.icrp.org/publication.asp?id=ICRP%20Publication%20110) | MC 患者体、成人接触者 | 产品内再分发需许可 |
| ICRP 145 成人网格体模（MRCP） | 提供 PHITS、MCNP6、Geant4 格式 | 2 个 | 同上 | icrp.org | [ICRP 免费列表](https://www.icrp.org/page.asp?id=5)；[文件结构](https://icrp.org/admin/P145-Webinar_C-Choi.pdf) | MC；网格适合模拟坐姿、拥抱等姿态 | Geant4 肺气道文件夹标注为待更新 |
| ICRP 143 儿童参考体素体模 | 新生儿、1、5、10、15 岁，男女各一 | 10 个 | 免费下载；再利用需许可 | icrp.org | [ICRP 免费列表](https://www.icrp.org/page.asp?id=5) | 儿童接触者 | 体素分辨率限制小器官 |
| ICRP 156 儿童网格体模 | P143 的网格版，2025 年发布 | 10 个 | **尚不免费**，需购买 | 购买 | [ICRP 156](https://www.icrp.org/page.asp?id=502) | 儿童接触者 | ICRP 仍以 P143 作为主要参考 |
| **UF/NCI 混合体模库** | 参考体型、362 个体型相关体模（儿童和成人）、8 个孕妇体模，.nii.gz | 370 个以上 | **按 NCI 软件转让协议，仅限非商用研究**；商用需经 NCI 技术转让中心 | 申请表 | [NCI 体模库](https://dceg.cancer.gov/tools/radiation-dosimetry-tools/phantoms-library)；[申请入口](https://ncidose.github.io/portal/request-access/)；[用户手册](https://github.com/ncidose/ncidosetools/wiki/PHANTOM-User-Manual) | 按身高体重选最近体模；BMI 变化与孕妇接触者；解剖变异 | 体模数量来自 GitHub wiki |
| UF 孕妇系列（文献） | 6 个孕周（15–38 周）× 3 个身高百分位；另有 6 孕周 × 5 体重百分位版本 | — | 经 UF/NCI | — | [AAPM 2019](https://w3.aapm.org/meetings/2019AM/programInfo/programAbs.php?aid=44481) | 孕妇接触者 | 哪些包含在 NCI 下载集中未确认 |
| MIDA（IT'IS/FDA） | 仅头颈部 | 1 个 | 除手续费外免费，需签许可协议 | DOI 申请 | [IT'IS MIDA](https://itis.swiss/virtual-population/regional-human-models/mida-model) | 用处很小 | 不是全身模型 |
| XCAT（Duke） | 4D 解剖体模软件 | — | 收费许可；脑部学术版首年 $400，全身版价格不公开 | 申请 | [XCAT](https://cvit.duke.edu/resource/xcat-phantom-program/) | 解剖变异 | 价格需询问 |

核数据层的参考数据都可以免费获取。积累因子是例外：ANSI/ANS-6.4.3 是付费且已撤销的标准，同样的数据可以通过 RSICC DLC-129 获取。NRC RG 8.39 Rev. 1（2020 年 4 月）只列出 I-131 的 Γ（2.2 R·cm²/(mCi·h)），**没有 Lu-177 和 Ac-225 这两行**（[RG 8.39 Rev. 1](https://www.nrc.gov/docs/ML1923/ML19232A081.pdf)），所以 IRSA 的"NRC 默认基线"必须从文献取 177Lu 的 Γ，并记录取值依据。Bellamy 等在 230 例 177Lu 患者中测得，紧随输注后 1 m 处的平均值为 **0.0037 mSv/h/GBq**，理论裸源值为 0.007636，即患者平均只有裸源 Γ 的 **48%**，且剂量率与 BMI 负相关（Spearman R = −0.35）（[Bellamy 2022](https://pmc.ncbi.nlm.nih.gov/articles/PMC9667277/)）。这直接给出了半解析模型中体衰减因子的先验。按笔记的换算，I-131 约为 0.059 µSv·m²/(MBq·h)，177Lu 理论值约为 0.0076 µSv·m²/(MBq·h)，相差约 8 倍【推断，需对照 Smith & Stabin 核对】。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| ICRP 107 衰变数据 | 发射能量与产额、β 能谱 | 1 部出版物 | 免费下载；ICRP 版权 | icrp.org | [ICRP 107](https://www.icrp.org/publication.asp?id=ICRP%20Publication%20107) | 核数据（177Lu、131I、225Ac 链的光子谱） | 原始文件随 CD 发行 |
| RadData（CRAN） | ICRP 107 的整理版 | — | 包 GPL-2；数据声明只允许"教育、研究和非营利用途" | `install.packages("RadData")` | [CRAN](https://ftp.fau.de/cran/web/packages/RadData/index.html) | 核数据 | **数据有非营利限制** |
| radioactivedecay | 衰变链计算 | PyPI 0.6.1 | 代码 MIT；PyPI 同时带 "Other/Proprietary" 分类，可能对应内置的核数据 | pip | [GitHub](https://github.com/radioactivedecay/radioactivedecay) | 核数据 | 内置 ICRP-107 数据的再分发条款未核实 |
| NuDat 3（NNDC） | ENSDF 能级、衰变、半衰期 | — | 美国政府数据服务（条款未查） | 网页 | [NuDat 3](https://www.nndc.bnl.gov/nudat3/) | 核数据 | — |
| IAEA LiveChart | ENSDF 衰变数据，可下载 | — | 条款未查 | 网页 | [LiveChart](https://www-nds.iaea.org/livechart/) | 核数据 | 自动抓取被 Cloudflare 拦截 |
| NIST XCOM / ESTAR | 光子截面与衰减系数；电子阻止本领 | — | NIST 公共数据 | 网页 | [XCOM](https://physics.nist.gov/PhysRefData/Xcom/html/xcom1.html)；[ESTAR](https://physics.nist.gov/PhysRefData/Star/Text/ESTAR.html) | 半解析模型（组织、铅、混凝土、玻璃的 µ） | — |
| ICRP 116 外照射转换系数 | 注量 → 有效剂量与器官剂量，AP/PA/LAT/ROT/ISO 等几何 | — | 免费下载；ICRP 版权 | icrp.org | [ICRP 116](https://www.icrp.org/publication.asp?id=ICRP%20Publication%20116) | 剂量引擎（按接触者姿态换算） | 补充文件格式未检查 |
| ICRP 74 | 含 H*(10) 等运行实用量的转换系数 | — | 免费下载 | icrp.org | [ICRP 免费列表](https://www.icrp.org/page.asp?id=5) | 与巡测仪读数对比 | — |
| NRC RG 8.39 Rev. 1 | I-131 Γ 及放行表；1 m 处占用因子 0.25；假设 1 R = 10 mSv | — | 美国政府作品 | ADAMS PDF | [ML19232A081](https://www.nrc.gov/docs/ML1923/ML19232A081.pdf) | NRC 基线 | 没有 Lu-177 和 Ac-225 的 Γ |
| Smith & Stabin 2012，Health Phys 102(3) | 1,100 多种核素的裸源照射率常数和铅 HVL/TVL | — | 期刊版权，付费 | 期刊 | [doi:10.1097/HP.0b013e318235153a](https://doi.org/10.1097/HP.0b013e318235153a) | NRC 基线（177Lu、225Ac 子体）；铅屏蔽 | 本轮没有提取具体数值 |
| Bellamy 2022 实测 177Lu 患者"有效 Γ" | 230 例，1 m 前方电离室测量 | 230 例 | PMC 开放获取 | 期刊 | [PMC9667277](https://pmc.ncbi.nlm.nih.gov/articles/PMC9667277/) | 半解析模型（随 BMI 变化的体衰减）、验证 | 没有数据可用性声明 |
| 177Lu-DOTATATE 自吸收 Γ（SNMMI 2015 摘要） | 实测值平均为空气中数值的 63% | 摘要 | — | JNM 增刊 | [JNM 2015;56(S3):1246](https://jnm.snmjournals.org/content/56/supplement_3/1246) | 半解析模型 | 【search-summary, verify】 |
| Arasi 2026：年龄分组体模中 I-131 外照射剂量率常数 | MCNPX 输入与计数，7 个 ORNL 程式化体模 × 4 个方向 | — | CC BY 4.0 | Zenodo | [zenodo.23863742](https://doi.org/10.5281/zenodo.23863742) | 半解析（131I 方向相关体衰减）、交叉验证 | 对应论文在审 |
| ANSI/ANS-6.4.3-1991 积累因子 | 0.015–15 MeV，最高 40 mfp，含 G-P 拟合系数 | — | 付费，已撤销 | ANSI | [ANSI](https://webstore.ansi.org/standards/ansi/ansians1991-1534182) | 半解析（屏蔽场景） | — |
| RSICC DLC-129 / ORNL/RSIC-49/R1 | 与 ANS-6.4.3 相同的数据 | — | RSICC 分发条款（未查） | RSICC 申请 | [RSICC](https://rsicc.ornl.gov/codes/dlc/dlc1/dlc-129.html) | 半解析 | 通常需注册 |

### MC 工具、代理模型与保形库：外照射代理训练集只能自己生成

生成患者周围外照射训练数据，OpenGATE 10 最合适：基于 Geant4 的 Python 接口，LGPL-3.0，PyPI 10.1.2 版于 2026-11-09 发布，自带 `opengate_dose_rate` 工具（[PyPI opengate](https://pypi.org/project/opengate/)）。GATE 负责人新推出的 radot 工具箱（LGPL-3.0）把它封装成 RPT 工作流，其中 p142 验证病例同时覆盖 Lu-177、I-131 和含衰变链的 Ac-225（[radot](https://github.com/dsarrut/radot)；[zenodo.23809698](https://doi.org/10.5281/zenodo.23809698)）。要改的只有一处：把计分从体内体素移到患者周围的空气或接触者网格上【推断】。**笔记没有找到任何公开数据集提供 177Lu 的"患者活度分布 → 接触者位置外照射场"配对。** 能用来交叉核对外照射 MC 的开放参考只有两套：EURADOS-EANM 数据集（Tc-99m、F-18、I-131，CC BY 4.0）和 Arasi 的 I-131 数据。EURADOS 研究中所有实测值都落在模拟值的 95% 置信区间内，各程序之间的差异在 1 m 处小于 8%，在 30 cm 和 50 cm 处为 5–16%（[Struelens 2024](https://pmc.ncbi.nlm.nih.gov/articles/PMC11035505/)）。IRSA 可以照这个协议为 177Lu 重做一遍。Ennassiri 2026 发现，简单 ML 代理模型在跨组织点核预测上表现不佳（[zenodo.21790169](https://doi.org/10.5281/zenodo.21790169)），这正支持了"先有半解析基线，再谈代理模型"的三模型对比设计。

| 名称 | 功能 | 规模/版本 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **OpenGATE 10**（`opengate`） | Geant4 MC 的 Python 接口，支持体素/网格体模、剂量计分器、径迹长度估计 | v10.1.2（2026-11-09） | LGPL-3.0，满足 LGPL 义务即可商用 | `pip install opengate` | [文档](https://opengate-python.readthedocs.io/)；[GitHub](https://github.com/OpenGATE/opengate) | MC、代理模型训练数据（主力） | 首次运行时下载 Geant4 数据 |
| radot（D. Sarrut） | RPT 剂量学 Python 库与命令行，集成 TotalSegmentator，读取 OpenGATE 剂量率图 | 早期开发阶段 | LGPL-3.0（物理笔记）；动力学笔记写"未检查" | `pip install radot` | [GitHub](https://github.com/dsarrut/radot) | 反演、MC 封装 | PyPI 包未确认 |
| Geant4 | GATE 与 TOPAS 的底层工具包 | — | Geant4 Software License v1.0（宽松，自定义） | 源码 | [GitHub](https://github.com/Geant4/geant4) | MC | 全文未审读 |
| OpenTOPAS | 参数文件驱动的 Geant4 封装 | — | MIT 风格 | GitHub | [OpenTOPAS](https://github.com/OpenTOPAS/OpenTOPAS) | MC 备选 | 主要面向粒子治疗 |
| OpenMC | 中子与光子输运 MC | — | MIT | pip/conda | [GitHub](https://github.com/openmc-dev/openmc) | 许可最宽松的 MC 备选 | 光子组织剂量学应用不如 Geant4 成熟 |
| EGSnrc | 电子与光子 MC | — | **AGPL-3.0**，网络使用即触发源码公开 | GitHub | [GitHub](https://github.com/nrc-cnrc/EGSnrc) | 仅作离线基准 | 不要用于 SaaS 后端 |
| GGEMS | GPU MC（OpenCL） | — | **GPL-3.0** | GitHub | [GitHub](https://github.com/GGEMS/ggems) | 快速生成数据（离线） | GPL 传染性 |
| PHITS（JAEA） | 通用 MC，可直接读取 ICRP 145 | — | 免费；经 NEA 数据库或 JAEA 授权，仅限和平用途，需出口管制审查 | 申请 | [How to get PHITS](https://phits.jaea.go.jp/howtoget.html) | 用 ICRP 145 体做交叉校验 | 不开源，按机构授权 |
| FLUKA、PENELOPE/penEasy、MCNP6、SIMIND | 通用或 SPECT MC | — | 【未核实】（站点被拦截或未访问） | 各自渠道 | [FLUKA](https://fluka.cern/)；[SIMIND](https://www.msf.lu.se/en/research/simind-monte-carlo-program) | 基准 | MCNP6 通常受出口管制 |
| PyTomography | GPU SPECT/PET 重建 | v4.0.0 | MIT | PyPI | [GitHub](https://github.com/PyTomography/PyTomography) | 重建 Deep Blue 投影数据 | — |
| PyTheranostics（QURIT） | 核医学图像处理与剂量学 | v0.3.0 | **冲突：GitHub 为 Apache-2.0，PyPI 为 MIT**，两者都宽松 | PyPI | [GitHub](https://github.com/qurit/PyTheranostics) | SPECT/CT → TIA 流程 | 再分发前要澄清 |
| OpenDose | ICRP 110 成人体模的 SAF 与 S 值，1,252 种核素 | — | 文章 CC BY 4.0；**数据库许可证未说明** | opendose.org | [JNM 2020](https://jnm.snmjournals.org/content/61/10/1514) | 器官级核对（内照射） | 再分发前确认条款 |
| MIRDcalc（MIRDsoft） | 器官与肿瘤内照射剂量学，v1.26（2026-10-23） | 1,224 种同位素 | 免费；2025-12-16 修订的使用条款"包含类似 MIT 许可的元素" | mirdsoft.org 注册 | [mirdsoft.org](https://mirdsoft.org/) | 内照射合理性检查 | 运行在 Excel 中 |
| IDAC-Dose 2.1 | 基于 ICRP 的内照射软件 | — | 免费；© IDAC Dose AB，未找到商业再利用条款 | idac-dose.org | [idac-dose.org](https://www.idac-dose.org/) | 内照射核对 | 引擎用于商业产品 QDOSE |
| DblurDoseNet 代码 | SPECT + 密度 → 剂量率图的 CNN | — | MIT | GitHub | [GitHub](https://github.com/ZongyuLi-umich/DblurDoseNet) | 代理模型架构模板 | 内照射 |
| radot p142 验证数据 v1.2 | 躯干 CT、器官标签、GATE 10 剂量率及不确定度（Lu-177、Y-90、I-131、含衰变链的 Ac-225） | 119 MB | CC BY 4.0 | Zenodo | [zenodo.23809698](https://doi.org/10.5281/zenodo.23809698) | 参考 MC 配置 | 早期版本标错了 I-131 活度，v1.2 已修正 |
| Ennassiri 2026：13 种介质中的 177Lu 径向点核 | GATE 10 脚本、材料库，以及简单 ML 代理模型基准 | — | CC BY 4.0 | Zenodo | [zenodo.21790169](https://doi.org/10.5281/zenodo.21790169) | 代理模型泛化的阴性结果基准 | 毫米尺度 β |
| 径向点核重采样到笛卡尔网格 | 体素点核与 DVK 生成的代码和数据 | — | CC BY 4.0 | Zenodo | [zenodo.7596345](https://doi.org/10.5281/zenodo.7596345) | 内照射 | — |
| **EURADOS-EANM 外照射剂量率数据集** | 5 种 MC 程序、ICRP 成人体素体模，Tc-99m、F-18、I-131 实测对比 | — | CC BY 4.0 | Zenodo | [zenodo.8414338](https://doi.org/10.5281/zenodo.8414338) | 外照射验证协议模板 | 没有 177Lu；文件内容未检查 |

| 名称 | 功能 | 版本 | 许可证/商用 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|
| **MAPIE** | 分割、CV+、jackknife+、CQR、时间序列 EnbPI 保形 | v1.6.0 | BSD-3 | [GitHub](https://github.com/scikit-learn-contrib/MAPIE) | 不确定性 | EnbPI 适合连续可穿戴读数 |
| **crepes** | 保形回归与保形预测系统，输出完整预测分布 | v0.9.1 | BSD | [GitHub](https://github.com/henrikbostrom/crepes) | 不确定性 | 可直接输出"儿童累计剂量超过 1 mSv 的概率" |
| puncc（DEEL） | 回归、分类、异常检测的保形库 | v0.9.3 | MIT（PyPI 分类） | [PyPI](https://pypi.org/project/puncc/) | 不确定性 | LICENSE 文件未找到 |
| TorchCP | PyTorch 保形工具箱 | v1.2.1 | **LGPL-3.0** | [GitHub](https://github.com/ml-stat-Sustech/TorchCP) | 不确定性 | 研究代码中可用，避免静态链接进 App |
| pytorch-ood | OOD 检测 | v0.4.0 | Apache-2.0 | [PyPI](https://pypi.org/project/pytorch-ood/) | OOD 回退 | — |
| Alibi Detect | 漂移与 OOD 检测 | v0.13.0 | **BUSL-1.1** | [PyPI](https://pypi.org/project/alibi-detect/) | 避免使用 | 生产用途不是开源许可 |
| filterpy | Kalman/UKF 滤波 | v2.0.0 | MIT | [PyPI](https://pypi.org/project/filterpy/) | 半衰期端侧估计 | — |
| PyMC / NumPyro | 层级贝叶斯 | 6.4.0 / 0.22.0 | Apache-2.0 | [PyMC](https://pypi.org/project/pymc/)；[NumPyro](https://pypi.org/project/numpyro/) | 离线群体先验 | — |
| DVC / MLflow | 数据版本与实验追踪 | 3.67.1 / 3.18.0 | Apache-2.0 | [DVC](https://pypi.org/project/dvc/)；[MLflow](https://pypi.org/project/mlflow/) | 全流程 | MLflow 的 LICENSE 文件本轮未打开 |
| Synthea | 合成患者生成器 | — | Apache-2.0 | [GitHub](https://github.com/synthetichealth/synthea) | 合成患者背景 | — |
| SDV | 合成表格数据 | v2.2.0 | **BUSL-1.1** | [PyPI](https://pypi.org/project/sdv/) | 避免使用 | 用场景语法代替 |

笔记也没有找到开源、经过验证的 RPT 患者外照射半解析模型（点源或线源加自吸收），团队需要按文献自行实现，这是阶段 1 的 WP1.1。

### 患者动力学与排泄：只有汇总统计，没有逐患者原始曲线

笔记检查过的临床剂量率研究都写"数据可按合理请求提供"，**没有一项公开逐患者的外照射剂量率时间序列**。现在能用的是三类东西：发表的汇总统计；一张逐患者的早期/晚期有效半衰期表（Monserrat Fuertes 2023，n = 20，CC BY）；以及 Deep Blue、MRTDosimetry 等基于影像的开放数据。用它们构建先验和合成数据模拟器已经足够，但不足以用真实的佩戴式轨迹验证滤波器。

Schütze 等的德国 8 中心研究最重要，它直接说明拟合窗口会让估计产生偏倚。只用 ≤48 h 数据拟合时，有效半衰期为 Pluvicto 1.30 d、Lutathera 1.40 d、院内 PSMA 1.45 d、院内 RRT 1.97 d；用到 168 h 时变为 1.60、1.89、2.10、2.85 d（[Schütze 2026](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)）。作者建议至少纳入一个末期数据点，并在早期出院时用群体衰减常数补充拟合，这正好支持"贝叶斯先验 + 可穿戴更新"的设计。

| 化合物（≤168 h 拟合） | 均值 ± SD（d） | 中位数（d） | 范围（d） | 第 90 百分位（d） | 推荐的保守值（d） |
|---|---|---|---|---|---|
| Lutathera | 1.89 ± 1.14 | 1.86 | 0.66–3.66 | 2.59 | 3.0 |
| 院内 PSMA | 2.10 ± 0.98 | 2.04 | 0.35–6.65 | 3.08 | 3.5 |
| 院内 RRT | 2.85 ± 1.12 | 2.72 | 0.90–5.82 | 3.88 | 4.0 |
| Pluvicto | 1.60（均值） | — | — | — | 2.0 |

数据来源：[Schütze 2026](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)。摘要把最后一列称为"第 90 百分位"，但它高于正文中的第 90 百分位（例如院内 PSMA 3.5 vs 3.08 d），应理解为向上取整后的推荐值。常被引用的"0.35–6.65 d"只是院内 PSMA ≤168 h 亚组的范围，不代表全部 177Lu。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **Schütze 2026**，Nuklearmedizin 65:60–68 | 8 个科室、210 组剂量率测量（2 m 处），四类化合物，平均活度 7149 ± 522 MBq | 仅汇总 | 文章 CC BY 4.0；全文未见数据可用性声明 | PMC 开放获取 | [PMC13046430](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML) | 有效半衰期先验；拟合窗口偏倚测试 | 补充表未打开 |
| **Kurth 2018**，EJNMMI Res 8:32 | 50 例 177Lu-PSMA-617，6.3 ± 0.5 GBq；2 m 处 2、4、24、48、72 h 剂量率；25 例测排泄 | 汇总表 | CC BY 4.0；数据按请求提供 | PMC | [PMC5897276](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/) | 177Lu-PSMA 群体曲线、排泄先验 | 只有 25 例有 72 h 数据 |
| **Monserrat Fuertes 2023**，EJNMMI Phys | 20 例 177Lu-DOTATATE，每例 6 次全身剂量率；表 2 列出逐患者早期与晚期有效半衰期 | 20 行 | CC BY 4.0；原始数据按请求提供 | PMC | [PMC10477145](https://pmc.ncbi.nlm.nih.gov/articles/PMC10477145/) | 早期窗口与晚期窗口估计的验证；双指数慢相先验 | 没有原始剂量率值 |
| **Bailey 2026**，AOJNMB 14(1):1–9 | 250 例 131I 甲状腺消融；天花板 GM 探测器距床 1.8 m，**每分钟采样，CSV 输出** | 汇总 + 示例曲线 | CC BY 3.0；**无数据可用性声明** | PMC | [PMC12854189](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/) | 伪影模型（体位、排泄、淋浴）；131I 先验 | 设备为商用产品，作者声明了利益冲突 |
| Turku 2024，EJNMMI Phys | 110 例 DTC（135 次治疗），天花板远程剂量率计 | 汇总 | CC BY 4.0；按请求提供 | PMC | [PMC11604915](https://pmc.ncbi.nlm.nih.gov/articles/PMC11604915/) | 带协变量的 131I 先验（BMI、GFR、糖尿病） | 仅住院阶段 |
| **Levart 2019**，EJNMMI Phys 6:7 | T0 时 0.1 m 处 407 µSv/h、1 m 处 20 µSv/h；双指数有效半衰期 4.7 ± 1.4 h 和 87.2 ± 14.7 h；出院时滞留率（门诊实测 38%，预测 63%） | 16+18+15 例 | CC BY 4.0 | PMC | [PMC6484059](https://pmc.ncbi.nlm.nih.gov/articles/PMC6484059/) | 验证（0.1 m 和 1 m 点）、双指数结构 | 与 Levart 2025 是两篇不同的论文（见矛盾裁定一节） |
| Remy 2008，JNM 49(9):1445 | 254 例全身滞留、30 例尿排泄；rhTSH 组约 10.5 h，停药组约 15.8 h | 汇总 | 期刊 | 期刊 | [JNM 表格](https://jnm.snmjournals.org/content/49/9/1445/tab-figures-data) | 131I 先验 | 【search-summary, verify】 |
| 131I 尿排泄有效半衰期（RPD 2013） | 前 24 h 为 18.4 ± 1.8 h；48–120 h 为 64 ± 2.7 h | 汇总 | 期刊 | PubMed | [PubMed 23535081](https://pubmed.ncbi.nlm.nih.gov/23535081/) | 131I 排泄 | 【search-summary, verify】 |
| 131I 甲亢 | 体内负荷有效半衰期约 111.4 h | 汇总 | — | PMC | [PMC3003283](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3003283/) | 甲亢参数集 | 【search-summary, verify】 |
| Iori 2026：外渗实时检测 | 18F/68Ga 注射和 177Lu-DOTATATE 给药期间的便携 γ 探测器计数率 | 0.5 MB | CC BY 4.0 | Zenodo | [zenodo.19251295](https://doi.org/10.5281/zenodo.19251295) | 真实探测器计数率，用于测试阶跃检测 | 只覆盖输注期间的几分钟 |
| SNMMI 挑战赛结果论文与社区代码 | 2021 增刊；2025 JNM 66(3):441；2026 分割篇（figshare CC BY）；carluri 代码（MIT）；luisacbscs 代码（无许可证） | — | 各异 | 开放 | [JNM 66(3):441](https://jnm.snmjournals.org/content/66/3/441)；[figshare](https://doi.org/10.6084/m9.figshare.c.8432063)；[carluri](https://github.com/carluri/snmmi_dosimetry_challenge) | 时间积分方法离散度 | 仓库久未维护 |

排泄数据方面，Kurth 2018 表 2 是最好的开放表格：177Lu-PSMA-617 累计排出比例在 2 h 为 30%、4 h 为 50%、12 h 为 70%、24 h 为 76%、48 h 为 77%、72 h 为 78%，主要经尿排出，2 h 的个体范围为 13–42%（[Kurth 2018](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/)）。由此可以推断，回家后可穿戴设备主要看到的是慢相，阶跃幅度较小；阶跃检测测试应当用"第 1 天大幅阶跃、第 2 天以后小幅阶跃"的合成数据【推断】。笔记没有找到 177Lu-DOTATATE 或 131I 的逐次排尿活度数据。

先验模型方面，177Lu **没有 ICRP 参考模型**，ICRP 128 只覆盖诊断用途（【search-summary, verify】，[SAGE 列表](https://uk.sagepub.com/en-gb/mst/icrp-publication-128/book248515)）。131I 的标准先验是 ICRP 137（OIR Part 3）碘系统模型和 Leggett 2010 生理模型，系数可以通过免费的 ICRP OIR 数据查看器获得（[ICRP 137](https://icrp.org/page.asp?id=374)；[OIR Viewers](https://icrp.org/page.asp?id=505)；[Leggett 2010](https://www.nrc.gov/docs/ML1102/ML110200086.pdf)）。PSMA 的 PBPK 模型谱系是 Kletting 2016 → Begum 2019 → Fele-Paranj 2024。Fele-Paranj 的模型有 150 多个 ODE，文章为 CC BY 4.0，SBML 文件放在 GitHub 上，但 `alifele/Computational-Physics` 和镜像 `qurit/PSMA-PBPK-Model-Matlab` **都没有许可证文件**（[PMC10803696](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC10803696/fullTextXML)；[alifele](https://github.com/alifele/Computational-Physics)；[qurit](https://github.com/qurit/PSMA-PBPK-Model-Matlab)）。可穿戴设备测到的是全身滞留与几何的卷积，150 个 ODE 对这个信号来说参数过多。实用的先验是 2–3 个房室：血液/快速肾清除、慢速肿瘤/肾滞留、带排尿事件的膀胱储存室。参数对齐 Kurth 的排泄曲线和 Schütze、Monserrat Fuertes 的有效半衰期分布；Fele-Paranj 的 SBML 只在离线时用来生成训练曲线，再分发需作者许可【推断】。

### 剂量计硬件与校准：Radiacode 是唯一同时具备 BLE 和开放 API 的现成选项

**Radiacode 10x 是笔记找到的唯一一款同时具备 BLE、成熟的 MIT 许可 Python 库和实时剂量率/能谱接口的设备。** 2026-11-20 官方商店售价 **US$269**（原价 $319），产品页没有任何计量或医疗认证声明（[Radiacode 103](https://www.radiacode.com/products/radiacode-103)；[cdump/radiacode](https://github.com/cdump/radiacode)）。经过认证的 EPD（Polimaster、Thermo、Kromek、Mirion）都有蓝牙，但笔记**没有找到公开 API**。推荐的原型路径是：Radiacode 103 作为患者佩戴传感器；手机网关读数，或通过 ESP32 BLE 桥、Web Bluetooth 读数；同时记录能谱，因为 113/208 keV 与 364 keV 窗口可以核实核素；用 G4Radiacode 模拟佩戴几何；体模和试点工作中，用一台 IEC 61526 认证 EPD 做对照【推断】。IRSA 的主要任务是测相对有效半衰期，所以比绝对能量响应更重要的是**随时间的稳定性和线性**。绝对水平由出院时校准过的 1 m 测量提供。

| 设备 | 探测器/规格 | 价格 | 接口与开放程度 | 认证 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **Radiacode 103**（另有 101/102/103G/110） | CsI(Tl) 约 1 cm³；剂量率 0.01–1000 µSv/h，±15%（137Cs）；灵敏度 30 cps/(µSv/h)；662 keV 分辨率 8.4% | US$269（促销），原价 $319 | BLE + USB；`cdump/radiacode` MIT，PyPI 0.4.0，提供 `data_buf()`、`spectrum()`、`energy_calib()` | 无认证声明 | [产品页](https://www.radiacode.com/products/radiacode-103)；[库](https://github.com/cdump/radiacode)；[PyPI](https://pypi.org/project/radiacode/) | 佩戴传感器原型；能谱核实核素 | 规格来自检索片段【search-summary, verify】；能量下限有 20 keV、30 keV、100 keV 三种说法互相冲突；电池续航未核实 |
| Radiacode 生态 | radiacode-tools（MIT）、ha-radiacode（MIT）、RadiaCode ESP32 BLE（MIT）、radiacode.js（MIT，Web Bluetooth）、Open-RadiaCode-Android（无许可证）、**G4Radiacode（GPL-3.0，晶体的 Geant4 数字孪生）** | — | 开源 | — | [GitHub 搜索](https://github.com/search?q=radiacode&type=repositories) | ESP32 桥、浏览器 App、能量/角响应模拟 | 社区维护 |
| Radiacode Zero | BLE 5.2，0.5 s 刷新，IP64 | 未核实 | Python 库支持未核实 | — | [规格](https://radiacode.com/docs/de/zero/device/tech-specs) | 更小的外形 | 探测器与量程未核实 |
| Open Gamma Detector | NaI(Tl) + SiPM + Raspberry Pi Pico | DIY | **GPL-3.0**，OSHWA 认证；USB 串口，BLE 未见文档 | 无 | [GitHub](https://github.com/OpenGammaProject/Open-Gamma-Detector) | 可完全修改的参考设计 | Pico ADC 非线性；NaI 易潮解、体积大 |
| GQ GMC-500/600/800 | GM 管 | 未核实 | 串口协议 GQ-RFC1801 公开；pygmc 支持 GMC-800 | 无 | [RFC1801](https://www.gqelectronicsllc.com/download/GQ-RFC1801.txt)；[pygmc](https://github.com/ctoth/pygmc) | 低成本台架参考 | 无 BLE；GM 管低能过响应；pygmc 许可证未查 |
| Kromek D3S | CsI(Tl) 16 cm³，237 g | 未核实 | 只能连预配对的手机 App，**无公开 BLE API** | 安防市场 PRD | [Kromek](https://www.kromek.com/prds/) | 高灵敏度参考 | 需要厂商 SDK |
| Polimaster RadFlash / PoliSimeter ER / PM1703MO-II BT | RadFlash：30 keV–3 MeV，蓝牙，IEC 61526:2024；PoliSimeter ER：20 keV–10 MeV | 未核实 | 厂商 App，**无公开 API** | IEC 61526（厂商声明） | [RadFlash](https://polimaster.com/products/dosimeters/radflash/)；[PoliSimeter ER](https://polimaster.com/products/dosimeters/polisimeter-er/) | 认证对照仪；日后产品化的路线 | Polimaster 承接定制开发 |
| Mirion Instadose+ / VUE | 积分式剂量计，按键触发 BLE 上传 | 服务订阅模式 | 专有算法，无开发者 API | 职业剂量监测服务 | [Mirion](https://www.mirion.com/products/medical/occupational-dosimetry/dosimetry-services/whole-body-area-dosimeters/on-demand-dosimetry-products/instadose-wireless-dosimeter) | 不适合流式剂量率 | — |
| Thermo RadEye PRD-ER4 | 10 nSv/h–250 µSv/h；蓝牙为选配 | 未核实 | 无公开 API | 专业 PRD | [Thermo](https://www.thermofisher.com/order/catalog/product/425067127) | 低剂量率参考 | 能量范围有 58 keV–6 MeV 和 60 keV–1.3 MeV 两种说法 |
| Technonia Smart Geiger Pro | 插在手机上的传感器 | 低成本 | 手机 App | — | [Sensors 2022](https://www.mdpi.com/1424-8220/22/23/9392) | 低成本患者侧 131I 测量的先例 | 规格未获取 |
| Better Geiger S-1、Atom Fast、Safecast bGeigie Nano | — | 未核实 | BLE/协议未核实 | — | [S-1 手册](https://manuals.online/better-geiger/measuring-instrument/s-1/user-manual/139oq)；[AtomConnect](https://www.gammaspectacular.com/blue/software-downloads/atomconnect)；[bGeigie](https://github.com/Safecast/bGeigieNanoKit) | 本底记录等 | 均未核实 |

校准方面，**IEC 61526:2024（第 4 版）**是现行的有源个人剂量计型式试验标准，已有含 AMD1:2026 的合订本；IEC 62387 覆盖无源积分系统，按第 1 版表格，Hp(10) 的强制光子范围为 80 keV–1.25 MeV（[IEC 61526](https://webstore.iec.ch/publication/66825)；[EVS 合订本](https://www.evs.ee/en/iec-61526-2024-amd1-2026-csv-consolidated)；[IEC 62387](https://webstore.iec.ch/publication/66492)）。177Lu 的光子线（113 keV 约 6%，208 keV 约 11%）正好落在未补偿 GM 管和小闪烁体偏离 137Cs 校准最大的区域（[NNDC 177Lu](https://www.nndc.bnl.gov/ensnds/177/Lu/adopted.pdf)）。133Ba（356 keV）在 **SPECT/CT** 校准中可以替代 131I（364 keV）（[Tran-Gia 2023](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10665282/)），但在活度计中不行：同等活度下 Capintec 对 133Ba 的响应约为 131I 的 3 倍（[NIST](https://www.nist.gov/node/1095576)）。笔记建议的校准协议【推断】分三步：先用 137Cs 把计数率校准到空气比释动能或 H*(10)；再用 133Ba 和密封或瓶装 177Lu 测能量响应比（57Co 的 122 keV 可能可以替代 113 keV 线，但未核实）；最后在灌注 177Lu 或 131I 的拟人体模上表征佩戴几何，以认证 EPD 和校准巡测仪为参考。设备需要在约 0.1–200 µSv/h 当量范围内保持线性。笔记**没有找到** Radiacode 在 177Lu 能量下的能量或角响应评估。O'Callaghan 2022 评估了活度、光子能量、患者体型、传感器位置和运动对 Smart Geiger Pro 读数的影响（CC BY 4.0），可以作为方法模板（[Sensors 2022](https://www.mdpi.com/1424-8220/22/23/9392)）。

本底扣除方面，**Safecast** 为 CC0，不需要 API 密钥，有完整归档（2020 年时压缩约 4.5 GB）（[Safecast](https://safecast.org/data/)；[AWS 注册表](https://registry.opendata.aws/safecast/)）。**EURDEP** 有约 5000 个欧洲站点的小时均值公开地图，但没有确认有通用的批量下载接口（[JRC](https://remon.jrc.ec.europa.eu/About/Rad-Data-Exchange)）。实际上更好的本底是"同一设备、同一地点"的基线，在给药前或信号回到平台后记录；治疗后头几天，患者自身信号比本底高几个数量级【推断】。

已有的患者佩戴式剂量计原型都**没有公开硬件或代码**：华盛顿大学的 PODD 背心（16 个 GAGG+SiPM 探头，BLE，2021 年报告半衰期估计误差在 4% 以内【search-summary, verify】）（[UW](https://people.ece.uw.edu/hauck/Radiology/)）；罗马的 WIDMApp 多通道服装（[PMC9298698](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9298698/)）；O'Callaghan 的 Smart Geiger Pro；意大利 CNR 的甲状腺患者无线设备【search-summary, verify】；Bailey/Globalsonics 的商用天花板 GM。另有两项专利需要做自由实施（FTO）审查：US 12377289（个体化剂量学设备）和 US 12011248（多探测器居家剂量学服装）（[US 12377289](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/12377289)；[US 12011248](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/12011248)）。

### 情境与接触行为：调查数据有时长、没有距离，传感器数据有距离阈值、没有行为

接触先验是 IRSA 误差预算中最大、也最难观测的一项。日记式接触调查（POLYMOD，以及 Zenodo 上 48 项同格式调查）提供时长分档、是否有身体接触、场所和年龄，**但没有距离**。传感器数据集（SocioPatterns、Copenhagen、Reality Mining）提供精细的时长，但只有固定的接近阈值或原始 RSSI。只有暴露通知类 BLE 数据集（MIT PACT、MITRE、NIST）有带标注的真实距离，可它们没有行为数据。IRSA 因此只能拼接：用调查数据回答"谁、多久"，再为每类场所假设距离，然后对照辐射防护领域的照护者研究做校准。笔记建议的映射是"身体接触 ≈ <0.3 m，交谈 ≈ 0.5–1.5 m"，这只是待校准的假设【推断】。

校准目标中，NRC 的文献综述最直接：RG 8.39 默认 1 m 处占用因子为 0.25；如果患者每天独睡 8 h，最大占用因子约 0.7；扣除约 4 h 的其他事务后，**合理上限约 0.5**；住拖车或小公寓的患者很难做到 1 m 处 0.25；距离从 1 m 缩到 0.5 m，剂量率升至 4 倍（[NRC ML17279B142](https://www.nrc.gov/docs/ML1727/ML17279B142.pdf)）。同一综述中，养老院场景假设一人坐在距床沿约 30 cm 处，甲亢治疗后占用因子超过 0.2（每天 4.8 h）、甲状腺癌治疗后超过 0.25（每天 6 h）就会超过 1 mSv。**Levart 2025** 是最好的实测行为目标：12 名家属四周内在 1 m 处的中位时间为 39.5 h（0–194.3 h），<1 m 为 19.9 h（0–229.8 h）；限制解除后的第 4 周，1 m 处中位 1.6 h/天（0–7.0），<1 m 处 0.5 h/天（0–14.6）（[PMC12595148](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/)）。第 4 周的中位数约相当于占用因子 0.07，远低于默认的 0.25，但最大值超过了默认值。所以场景模拟器要复现的是**重尾、因人而异的分布**，而不是单一的占用因子【推断】。Levart 的限制清单（禁止同床 16 天、按年龄分组的儿童限制、一周内公共交通不超过 1 h）可以直接映射为 IRSA 的场景库。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| POLYMOD（Mossong 2008） | 单日日记；身体接触或交谈；时长分 5 档；家/工作/学校/休闲/交通/其他；5 岁年龄组 | 7,290 人，97,904 次接触，8 个欧洲国家 | **CC BY-NC 4.0**，不可商用 | Zenodo，R `socialmixr`/`contactsurveys` | [zenodo.3874557](https://zenodo.org/records/3874557) | 接触先验 | 没有距离；家中约 75% 的接触是身体接触 |
| Zenodo "Social contact data" 社区（48 项调查） | 同一 schema；含秘鲁、津巴布韦、越南、赞比亚/南非、中国、泰国、索马里兰、冈比亚等 LMIC 调查，以及荷兰 70 岁以上、EPICURUS 老年人、比利时家庭调查 | 48 项 | **逐项核对**；**冈比亚 2022 为 CC BY 4.0**，POLYMOD 为 CC BY-NC | R `contactsurveys::download_survey()` | [清单](https://cran.nics.utk.edu/cran/web/packages/contactsurveys/vignettes/contactsurveys.html)；[冈比亚](https://zenodo.org/records/13101862) | 接触先验（LMIC、老年人） | 多数记录的许可证未核实 |
| CoMix（2020–2023） | 疫情期间的重复面板日记 | 欧洲 20 国左右 | 未逐项核对 | Zenodo | [清单](https://cran.nics.utk.edu/cran/web/packages/contactsurveys/vignettes/contactsurveys.html) | "被要求减少接触"情景的类比 | 受疫情扭曲 |
| BBC Pandemic | 英国分场所、分类型的混合矩阵 | 40,177 人 | 未说明；仅有矩阵 | medRxiv 附件 | [medRxiv](https://www.medrxiv.org/content/10.1101/2020.02.16.20023754v2.external-links) | 英国矩阵 | 无个体记录 |
| Prem 2021 合成接触矩阵 | 年龄 × 场所矩阵 | 177 个地区 | **未确认** | GitHub、Zenodo | [PMC8354454](https://pmc.ncbi.nlm.nih.gov/articles/PMC8354454) | 无调查国家的默认先验 | 是模型推算，不是观测；使用更正后的版本 |
| SocioPatterns 数据包（Génois & Barrat） | 6 个面对面接触网络 + 6 个共处网络 | 66.7 MB | **CC BY-NC 4.0** | Zenodo | [zenodo.1117884](https://zenodo.org/record/1117884) | 接触时长分布 | 接近是阈值，不是距离；含医院病房、小学等部署 |
| 马拉维农村村庄（Ozella 2021） | 可穿戴接近传感器，户内与户间面对面接触 | 86 人，13 天 | 文章 CC BY 4.0；**数据许可证与托管未确认** | 待查 | [EPJ Data Science](https://link.springer.com/article/10.1140/epjds/s13688-021-00302-w) | 最接近 TC 受援国家庭的开放类比 | 需向作者索取 |
| Copenhagen Networks Study | 每 5 分钟一次蓝牙 RSSI，以及通话、短信 | 706 人，4 周 | 文章 CC BY 4.0；数据文件许可证未确认 | Figshare | [Figshare](https://figshare.com/articles/dataset/The_Copenhagen_Networks_Study_interaction_data/7267433) | RSSI 作为距离代理的实验 | 研究对象为年轻学生 |
| MIT Reality Mining | 蓝牙邻近、通话、基站 | 约 100 人 | 未确认 | IEEE DataPort | [CRAWDAD](https://ieee-dataport.org/open-access/crawdad-mitreality) | 优先级低 | 年代久远 |
| NIST TC4TL 与 MIT PACT 数据集 | 手机 RSSI 序列及距离标签（1.2/1.8/3.0/4.5 m） | 1.5 万训练文件等 | 各仓库许可证未核实 | MIT PACT GitHub | [NIST TC4TL](https://www.nist.gov/node/1617526)；[PACT](https://mitll.github.io/PACT/datasets.html) | 如果将来用手机间 BLE 感知同伴距离 | 只有手机对手机，没有行为 |
| NIST BLE RSSI 接近检测（mds2-2827） | 人体遮挡与不同墙体材料下的 RSSI 及实测间距 | ZIP | NIST 开放许可 | 公开 | [data.gov](https://catalog.data.gov/dataset/bluetooth-low-energy-ble-received-signal-strength-indicator-rssi-data-for-proximity-detect) | 人体与墙体对距离感知的影响 | — |
| ATUS（美国时间利用调查） | 24 h 日记，2003–2025 | 全国代表性样本 | BLS 免费下载 | bls.gov | [BLS](https://www.bls.gov/tus/data.htm) | 在家、通勤时长 | 共处文件未确认 |
| IPUMS Time Use（ATUS/AHTUS/MTUS） | 协调后的多国日记 | 多国 | 研究用途免费；商用与再分发条款未确认 | ipums.org | [IPUMS](https://www.ipums.org/projects/ipums-time-use) | 跨国时间预算 | — |
| HETUS（Eurostat） | 2010 轮 17 国科研用文件 | 17 国 | **仅限认证研究机构**；HETUS 2020 微观数据最早 2027 年开放 | Eurostat 申请 | [Eurostat](https://ec.europa.eu/eurostat/web/microdata/harmonised-european-time-use-surveys) | 欧洲聚合先验 | 访问慢 |
| UKTUS 2014–15（SN 8128） | 8 岁以上日记 | 英国 | UK Data Service 最终用户许可；**商用需数据所有者批准** | 注册 | [b2find](https://b2find.eudat.eu/dataset/686f6564-b83e-5fe4-a7e9-24367928fa2e) | 英国先验 | — |
| 中国第二次（2018）、第三次（2024）时间利用调查 | 2018：20,226 户、48,580 人；只发布汇总 | 全国 | 仅汇总，未找到微观数据 | 国家统计局公报 | [2018 公报](https://www.stats.gov.cn/xxgk/sjfb/zxfb2020/201901/t20190125_1768278.html)；[2024 No.1](https://www.stats.gov.cn/english/PressRelease/202411/t20241115_1957435.html) | 中国弱先验 | — |
| 照护时长 | 美国癌症照护者平均 32.9 h/周，32% 超过 41 h/周；CanCORS 终末期 24.5 h/周 | — | 报告 | 公开 | [NAC 2016](https://www.caregiving.org/wp-content/uploads/2022/11/CancerCaregivingReport_FINAL_June-17-2016.pdf)；[PMC4479404](https://pmc.ncbi.nlm.nih.gov/articles/PMC4479404) | "依赖型患者"场景的上下界 | 照护时长不等于在 1 m 内的时间 |
| NRC SECY-18-0015 文献综述；NUREG-1556 Vol. 9 Rev. 3 | 占用因子推导与场景；医疗许可证项目指南（2019，ML19256C219） | — | 美国公共领域 | ADAMS | [ML17279B142](https://www.nrc.gov/docs/ML1727/ML17279B142.pdf)；[NUREG-1556 Vol. 9](https://www.nrc.gov/reading-rm/doc-collections/nuregs/staff/sr1556/v9/index.html) | 场景参数、校准目标 | 附录 U 原文未获取 |
| **Levart 2025**，EJNMMI Phys | 12 名家属，LiF TLD 连续佩戴 4 周，中位有效剂量 0.038 mSv（0.014–0.111），附分时段的距离与时长日记 | 12 人 | **CC BY-NC-ND 4.0**；数据按请求提供 | PMC | [PMC12595148](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/) | 端到端验证；行为校准 | 由此推导的表格只能引用，不能作为改编表格再分发 |
| 其他照护者研究 | Levart 2019：不加限制时伴侣剂量估计 6.2 mSv（2.0–17.9）；坦桑尼亚 131I 建模中依赖型患者按 10 cm 计；Graves 病门诊家属平均 0.258 mSv；2025 年日本 177Lu-PSMA 研究中 7 例患者有 6 例家中受照为 0 µSv | — | 各异 | 期刊 | [PMC6484059](https://pmc.ncbi.nlm.nih.gov/articles/PMC6484059)；[VJOL](https://vjol.info.vn/mostc/article/download/136456/111618/)；[ScienceDirect](https://www.sciencedirect.com/science/article/pii/S0378603X18301748)；[Ann Nucl Med 2025](https://link.springer.com/10.1007/s12149-025-02079-8) | 场景参数 | 日本研究为【search-summary, verify】 |

### 地理与人口：宽松许可的全球图层齐全，实时人流拿不到

全球开放图层对 LMIC 的覆盖不错，也大多允许商用。**Google Open Buildings** 有 18 亿栋建筑，可选 CC BY 4.0 或 ODbL；**Microsoft Global ML Building Footprints** 为 CDLA-Permissive-2.0；Overture 的 Places 主题以 CDLA-Permissive-2.0 为主；**Foursquare OS Places** 为 Apache 2.0；GHSL、WorldPop、LandScan 三套人口栅格都是 CC BY 4.0。实时人流**不能通过 Google 官方 API 获取**，替代方案要么是非官方抓取，要么是商业服务（[Google Business 帮助](https://support.google.com/business/answer/6263531?hl=en)）。这进一步支持把人群数据降为背景信息。为了默认保护隐私，可以在手机里放一个离线小包：Overture/Foursquare 的 POI 类别、建筑底图和一套人口栅格。这样不用把位置发给服务器，也能区分"家、交通、工作场所、公共场所"。Google Maps 的内容按条款不能这样缓存【推断】。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| Overture Places | 汇集 Meta、Microsoft、Foursquare、AllThePlaces 的 POI | 全球 | 主要为 CDLA-Permissive-2.0；Foursquare 来源为 Apache 2.0；AllThePlaces 为 CC0 | 云端 Parquet | [Overture 署名说明](https://docs.overturemaps.org/attribution/) | 场所类别（端侧查找） | 按来源署名 |
| Overture Buildings / Transportation / Base / Divisions | 合并 OSM、Microsoft、Google、Esri 等 | 全球 | **ODbL**（衍生数据库需共享相同方式） | 公开 | [同上](https://docs.overturemaps.org/attribution/) | 场所、户型（用建筑底面积估计住宅大小） | ODbL 义务 |
| Overture Addresses | 各国开放地址 | 因国而异 | 混合 | 公开 | [同上](https://docs.overturemaps.org/attribution/) | 场所 | LMIC 覆盖稀疏【推断】 |
| Foursquare OS Places | 1 亿多个 POI，22 个核心属性，每月更新 | 全球 | Apache 2.0 | Hugging Face（需留联系方式） | [HF](https://huggingface.co/datasets/foursquare/fsq-os-places) | 场所 | OSM 社区反映有已关闭多年的过时地点 |
| Microsoft Global ML Building Footprints | Bing 影像 2014–2024 提取的建筑多边形 | 介绍写 14 亿，FAQ 写 9.99 亿（**数字不一致**）；最近更新 2026-11-12 | **GitHub 为 CDLA-Permissive-2.0；在 Overture 建筑主题内按 ODbL 发布（冲突）** | GeoJSONL、Planetary Computer | [GitHub](https://github.com/microsoft/GlobalMLBuildingFootprints) | 场所、户型 | 直接从 GitHub 获取可保留 CDLA 条款 |
| Google Open Buildings v3 | 置信度 ≥0.65 的 18 亿栋建筑检测结果 | 非洲、南亚、东南亚、拉美；多边形 178 GB | **CC BY 4.0 或 ODbL，由用户选择** | CSV、Earth Engine、HDX | [Open Buildings](https://sites.research.google/gr/open-buildings/) | 户型（LMIC 住宅面积先验） | 不含冲突地区 |
| OpenStreetMap | 道路、POI、建筑、交通 | 全球，LMIC 完整度不均 | ODbL | 公开 | [OSM 版权](https://www.openstreetmap.org/copyright) | 场所 | 本轮未重新核实 |
| GHSL GHS-POP R2023A | 1975–2030 多期人口栅格 | 全球 | CC BY 4.0 | JRC、AWS、Earth Engine | [AWS](https://registry.opendata.aws/ghs-pop/) | 背景人口密度 | 居住（夜间）人口 |
| WorldPop | 100 m 国家栅格；1 km 全球拼接 | 全球，侧重 LMIC | CC BY 4.0 | 公开 | [licence.txt](https://worldpop-public-data.soton.ac.uk/GIS/Population/Individual_countries/TZA/United_Republic_of_Tanzania_100m_Population/licence.txt) | 背景人口密度 | 模型估算 |
| LandScan Global | **环境（24 h 平均）人口**，约 1 km，2000–2024 | 全球 | CC BY 4.0（经二手目录确认） | ORNL、GEE 社区目录 | [ORNL](https://www.ornl.gov/news/public-release-ornl-global-population-distribution-data-aids-humanitarian-support) | 白天暴露背景（更贴切） | 未在 ORNL 条款页核实 |
| Meta HRSL | 约 30 m 人口 | 多国 | 据称 CC BY 4.0【未核实】 | HDX | [HDX](https://data.humdata.org/organization/meta) | 背景 | 页面返回 403 |
| Mobility Database | GTFS/GTFS-RT/GBFS 目录 | 6,000 多个 feed，99 个以上国家 | 每个 feed 许可证不同【未核实】 | API | [mobilitydatabase.org](https://mobilitydatabase.org/about) | 通勤时长与交通工具 | 非正规公交多数没有 GTFS |
| Google Popular Times | 相对繁忙度 | 视场所而定 | **官方 Places API 不提供**，只能靠非官方抓取，可能违反条款 | — | [populartimes](https://github.com/m-wrzr/populartimes) | 放弃 | — |
| BestTime.app | 商用繁忙度预测 | — | 商用 API，条款未查 | 付费 | [BestTime](https://blog.besttime.app/popular-times-api-alternative-for-developers/) | 可选 | 非官方数据 |
| Google Maps Platform 缓存条款 | Place ID 可缓存；部分 API 的经纬度最多缓存 30 天 | — | 限制性条款 | — | [Maps 服务条款（2024 存档）](https://cloud.google.com/maps-platform/terms/maps-service-terms/index-20240422) | 场所 | 现行 Places 存储条款未确认 |

### 户型与可穿戴动作：Swiss Dwellings、Capture-24 和 ExtraSensory 各补一块

在户型数据中，笔记只确认了一个**宽松许可**的大型数据集：**Swiss Dwellings v3**（CC BY 4.0，45,176 套公寓，含马桶、浴缸等设施）。CubiCasa5K 是 CC BY-NC-SA 4.0，ZInD 只限学术非商用，RPLAN、Structured3D、HouseExpo 的许可证未确认。这些数据都来自高收入国家或东亚。LMIC 的拥挤程度可以用 DHS 的"每间卧室人数"（hv216）做代理指标。实用做法是：从 Swiss Dwellings 中抽样得到卧室到客厅、卧室到卫生间的距离分布；再按 Google/Microsoft 建筑底面积和 DHS hv216 缩小面积；DHS 显示每间卧室 3 人以上时，强制使用同床或同室场景【推断】。同床伴侣的距离需要假设（0.1–0.3 m），这与 Levart 的 0.1 m 睡眠假设和 NRC 的 30 cm 床边场景一致。

动作数据方面，**Capture-24**（CC BY，151 人自由生活状态下的腕部加速度计数据，100 Hz）最适合识别睡眠、坐、走。**ExtraSensory** 是唯一确认带明确的**如厕**（2,655 例/43 人）和**淋浴**标签的开放数据集，同时有"躺下""睡觉""在车里""在公交上"等标签。它注明"免费使用，须引用"，但没有写明正式许可证（[ExtraSensory](http://extrasensory.ucsd.edu/)）。这些标签可以作为上下文标志：如厕可能对应排泄造成的剂量率阶跃，卧床对应几何变化和同床可能，乘车对应交通场景。它们不能直接当剂量估计用。**所有这些数据集都没有佩戴辐射探测器**，体位和读数之间的关系仍需自己测量或模拟【推断】。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **Swiss Dwellings v3.0.0** | 2D 几何：墙、门窗、房间、卫生间、厨房、马桶、浴缸；附视野、日照、噪声模拟 | 45,176 套公寓，约 37 万个房间，931.9 MB | CC BY 4.0 | Zenodo | [zenodo.7788422](https://zenodo.org/records/7788422) | 房间间距分布 | 只有瑞士住宅，没有家具 |
| Modified Swiss Dwellings（MSD） | ML 就绪的衍生版本 | — | "许可证信息不可用" | TU Delft | [doi](https://doi.org/10.4121/e1d89cb5-6872-48fc-be63-aadd687ee6f9) | 户型 | 许可证不明 |
| CubiCasa5K | 带多边形标注的户型图 | 5,000 张，5.5 GB | **CC BY-NC-SA 4.0** | Zenodo | [zenodo.2613548](https://zenodo.org/records/2613548) | 户型 | 非商用且需共享相同方式 |
| ZInD（Zillow） | 户型图 + 360° 全景 | — | **仅限学术非商用**；代码 Apache | GitHub 申请 | [GitHub](https://de.github.com/zillow/zind) | 户型 | 美国住宅 |
| RPLAN / Structured3D / HouseExpo | 亚洲真实户型 / 合成住宅 / SUNCG 衍生 | 8 万以上 / 3,500 套 / — | **未确认** | 申请表 | [综述 arXiv 2504.09694](https://arxiv.org/pdf/2504.09694) | 户型 | 许可证未核实 |
| DHS 拥挤度（hv216） | 每间卧室人数，≥3 人记为拥挤 | 众多 LMIC 调查 | DHS 注册（条款未查） | DHS Program | [DHS 论坛](https://userforum.dhsprogram.com/pdf.php?msg=19811) | LMIC 同床同室先验 | 各次调查定义可能不同 |
| World Bank `H_SLEEPROOMS` 等 | 国家调查中的卧室数变量 | — | 各目录 | 微观数据库 | [World Bank](https://microdata.worldbank.org/index.php/catalog/3161/variable/H/H_SLEEPROOMS) | 同上 | 异质性大 |
| **Capture-24** | Axivity AX3 腕部加速度计，100 Hz；200 多种活动标签 | 151 人，每人约 24 h，6.4 GB | CC BY | ORA | [ORA](https://ora.ox.ac.uk/objects/uuid:99d7c092-d865-4a19-b096-cc16440cd001)；[GitHub](https://github.com/OxWearables/capture24) | 睡眠、坐、走的伪影模型 | 戴在手腕而非躯干；英国成年人 |
| **ExtraSensory** | 手机 + 手表传感器；如厕、淋浴、睡觉、躺下、乘车、公交标签 | 60 人，30 多万条一分钟样本 | 免费，须引用；**没有正式许可证** | 公开下载 | [extrasensory.ucsd.edu](http://extrasensory.ucsd.edu/) | 如厕、卧床、乘车事件检测；交通情境 | 标签为自报；再分发条款不明 |
| PAMAP2 | 3 个 IMU + 心率，18 种活动 | 9 人 | CC BY 4.0 | UCI | [UCI 231](https://archive.ics.uci.edu/dataset/231) | 姿态与步态 | 实验室协议 |
| WISDM | 手机 + 手表加速度计与陀螺仪，20 Hz，18 种活动 | 51 人，295 MB | CC BY 4.0 | UCI | [UCI 507](https://archive.ics.uci.edu/dataset/507/wisdm+smartphone+and+smartwatch+activity+and+biometrics+dataset) | 姿态 | 脚本化动作 |
| HHAR | 多款设备的加速度计与陀螺仪 | 6 种活动 | CC BY 4.0 | UCI | [UCI 344](https://archive.ics.uci.edu/dataset/344/heterogeneity+activity+recognition) | 设备异质性（TC 国家手机型号杂） | 脚本化动作 |

### LLM 知识语料：只有 NRC 文本和 CC BY 指南可以放心入库

**目前没有公开且有许可证的放射防护患者对话数据集或基准**，IRSA-Bench 必须自建。不经许可就可以改动并再分发的只有三类：美国 NRC 和 eCFR 文本（公共领域）、CC BY 4.0 的 EANM/SNMMI 177Lu-PSMA 指南、CC BY 研究论文。IAEA 和 ICRP 文件可以免费阅读，但再利用需要许可；HPS "Ask the Experts" 有 15,000 多个问答，最贴近真实患者的提问方式，但保留所有权利。由于 IAEA TC 是项目资助方，正式向 IAEA 申请许可（包括翻译）大概比较容易，应尽早提出【推断】。各辖区的阈值差异很大：美国 §35.75 是 5 mSv/1 mSv，德国 I-131 出院要求 2 m 处 3.5 µSv/h，瑞士要求 1 m 处 <10 µSv/h，日本另有一套放行条件。这正说明阈值应写进版本化的规则引擎，而不是交给 RAG 检索。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| 10 CFR 35.75（eCFR） | 他人剂量不太可能超过 5 mSv 即可放行；超过 1 mSv 需书面指导；哺乳中断指导 | 1 节 | 美国政府作品，无版权 | eCFR API | [eCFR §35.75](https://www.ecfr.gov/current/title-10/chapter-I/part-35/subpart-C/section-35.75) | 规则引擎、知识库 | 仅适用美国 |
| NUREG-1556 Vol. 9 Rev. 3 | 剂量计算方法与活度表（2019，ML19256C219） | 1 份报告 | 公共领域 | ADAMS | [PDF](https://www.nrc.gov/docs/ML1925/ML19256C219.pdf) | 规则引擎 | nrc.gov 拒绝脚本下载，需用浏览器 |
| NRC RG 8.39 Rev. 1 | 现行最终版（2020-04-07） | 1 份指南 | 公共领域 | Federal Register | [FR 2020-07307](https://www.govinfo.gov/content/pkg/FR-2020-04-07/html/2020-07307.htm) | 规则引擎（现行版） | — |
| NRC DG-8061 | **2023 年草案**：按核素的给药活度与实测剂量率阈值、哺乳阈值 | 1 份草案 | 公共领域 | Federal Register | [FR 2023-08418](https://www.govinfo.gov/content/pkg/FR-2023-04-21/html/2023-08418.htm) | 规则版本漂移测试 | 草案不代表 NRC 正式立场 |
| NRC DG-8070 | **2026 年草案**（RG 8.39 Rev. 2），91 FR 51749，案卷 NRC-2026-3730 | 1 份草案 | 公共领域 | Federal Register / ADAMS | [SNMMI 意见函 ML26254A152](https://www.nrc.gov/docs/ML2625/ML26254A152.pdf)；[OSF k4e2q](https://osf.io/k4e2q/) | 规则引擎（未来版本） | **本轮未取得全文**（见矛盾裁定） |
| IAEA SRS-63《放射性核素治疗后的患者放行》 | 放行标准、患者指导、照护者剂量约束 | 89 页 | IAEA 版权，再利用需许可，通常收费 | 免费 PDF | [Pub1417](https://www-pub.iaea.org/MTCD/Publications/PDF/Pub1417_web.pdf) | 知识库、IAEA 基线规则 | 据前期分析修订版在准备中（未再核实） |
| IAEA SSG-46 | 医疗照射防护要求，含治疗、放行、照护者 | 340 页 | IAEA 版权 | 免费 PDF | [PUB1775](https://www-pub.iaea.org/MTCD/Publications/PDF/PUB1775_web.pdf) | 知识库、规则引擎 | — |
| IAEA RPOP 治疗核医学患者页面 | 面向患者的 FAQ：出院卡通常携带 4 周；照护者每次治疗 ≤5 mSv；儿童 <1 mSv/年 | 网页 | IAEA 网站内容受版权保护 | 网页 | [RPOP](https://iaea.org/resources/rpop/health-professionals/nuclear-medicine/therapeutic-nuclear-medicine/patients) | 患者语言、基准题目种子 | 自动抓取被拒，应正式申请 |
| ICRP 94、103、128、140 | 核心参考文件 | 4 份 | **可免费阅读，不可免费再利用**；联系 annals@icrp.org | 免费下载 | [ICRP](https://icrp.org/page.asp?id=5) | 内部知识库；剂量系数走引擎 | 除最新三卷外均已免费 |
| **EANM/SNMMI 177Lu-PSMA 联合指南**（EJNMMI 2023） | 隔离 48–72 h；门诊约观察 2 h；避免与孕妇、婴儿长时间近距离接触；避孕 3 个月 | 1 篇 | **CC BY 4.0，可商用；但前言写着"未提供这些服务的机构不得复制或修改"（冲突）** | PMC | [PMC10317889](https://pmc.ncbi.nlm.nih.gov/articles/PMC10317889/) | 知识库、软性规则、基准 | 再分发前向 EANM 澄清；EANM 其他指南的许可证未逐一核对 |
| SNMMI 患者须知 | 患者语言的防护措施 | 几十份 | ©SNMMI，未找到再利用条款 | 网页/PDF | [I-131 须知](https://sites.snmmi.org/AM/Patients/Fact-Sheets/Guidelines-for-Patients-Receiving-Radioiodine-I-131-Treatment.aspx) | 需许可后入库 | 条款不明 |
| NHS 信托手册（UCLH、GSTT、UHS、UH Bristol、CUH 等） | 177Lu-PSMA、131I 患者手册 | 10 份以上 | 各信托持有版权，未确认 OGL | 网页/PDF | [UCLH](https://uclh.nhs.uk/patients-and-visitors/patient-information-pages/lutetium-177-psma-radionuclide-therapy)；[GSTT](https://guysandstthomas.nhs.uk/health-information/lutetium-therapy-neuroendocrine-tumours/after-having-lutetium-therapy) | 基准（英国场景、"请咨询您的治疗团队"类题目） | 手册常不写具体天数 |
| 美国医院 Pluvicto 手册（UChicago、MSK 等） | 例如与他人保持 3 英尺 3 天，与儿童和孕妇 7 天 | 数份 | 机构版权 | PDF | [UChicago](https://www.uchicagomedicine.org/-/media/pdfs/adult-pdfs/conditions-and-services/cancer/prostate-cancer/pluvicto-patient-guide.pdf) | 跨辖区冲突测试 | 只作测试用例 |
| 德国 SSK 2004 / Richtlinie；瑞士 BAG Wegleitung 2025 | 德国：I-131 出院 ≤250 MBq，对应 2 m 处 3.5 µSv/h；瑞士：48 h 后 1 m 处 <10 µSv/h | 文件 | 官方文件，再利用条款未查 | PDF | [SSK 2004](https://ssk.de/fileadmin/documents/de/2004/2004-12-17_Ermittlung_Vorbelastung.pdf)；[BAG](https://www.bag.admin.ch/dam/de/sd-web/vsVYUlb8caju/V2_250624_Strahlenschutz_Wegleitung_Radionuklidtherapie_DE.pdf) | 德国、瑞士规则包 | 德国现行综合文本未核实 |
| 日本 JSNM《Lu-177-DOTA-TATE 适正使用手册》+ MHLW 通知 | 与 15 岁以下儿童或孕妇同住、无法与家人保持 2 m 睡眠距离、失禁、同一公共交通需 ≥2 h 等情形不得放行 | 1 份手册 + 通知 | 开放获取文章，许可证未查 | PMC、MHLW | [PMC5852188](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5852188/)；[MHLW](https://www.mhlw.go.jp/content/10800000/001161279.pdf) | 日本规则包、日语题目 | 现行附件阈值未获取 |
| **HPS "Ask the Experts"** | 15,000 多个问答，22 个类别，含"核医学患者问题""妊娠与辐射" | >15,000 | 作者或 HPS 版权，**转载须获许可**；未找到文本挖掘条款 | 网页 | [ATE](https://hps.org/publicinformation/ate/) | 基准（最真实的照护者、妊娠、宠物、出行提问） | **未获书面许可不得批量抓取**；回答跨越数十年，以美国为中心 |
| HPS 面向医护人员的文件 | 立场声明与资料 | 几十份 | 原样使用需注明 HPS；大幅修改后需去掉 HPS 字样 | 网页 | [HPS](https://hps.org/physicians/documents/) | 知识库 | 注意文件日期 |
| Koller 等 MedChat（Theranostics 2025） | 面向医护人员的 RAG 聊天机器人；197 篇论文作上下文；英德双语评测题 | 70 行（约 35 题 × 英/德） | 文章 CC BY 4.0；**仓库无 LICENSE** | GitHub | [piakoller/MedChat](https://github.com/piakoller/MedChat)；[thno.org](https://www.thno.org/v15p5693.htm) | 基准种子、文献列表 | 与 IRSA 同一课题组，可直接索取许可；题目偏 PRRT |
| Bilgin 2024（Front Oncol） | 177Lu-PSMA 最常见的 12 个患者问题，GPT-4 有 16.6%、Bard 有 29.1% 的回答被评为错误或部分正确 | 12 题 | CC BY | 文章 | [PMC11272524](https://pmc.ncbi.nlm.nih.gov/articles/PMC11272524/) | 基准种子、基线错误率 | 规模很小 |
| `hiroki-rad/radiation_consult_qa` | 日语诊断影像辐射咨询问答 | 243 行；姊妹集 12,117 行 | **未声明许可证** | HF | [HF](https://huggingface.co/datasets/hiroki-rad/radiation_consult_qa) | 日语语气模板、反例 | 来源不明，疑似 LLM 生成；不是治疗场景 |
| `ZhJiHo/radiation_domain_dataset`、`dkqkqhdms/nuclear_manual` | 内容未查 | 77,878 行 / <1K | 无许可证 | HF | [HF](https://huggingface.co/datasets/ZhJiHo/radiation_domain_dataset) | 暂不可用 | — |
| 其他（核医学技术 MCQ、日本核医学专科考试、日本第一种放射线取扱主任者考试、RadGuide AI、LLMPatientSafety） | 考试或评测研究 | — | 数据未发布或无许可证 | 论文 | [PubMed 40345820](https://pubmed.ncbi.nlm.nih.gov/40345820/)；[RadGuide AI](https://doi.org/10.64898/2026.07.09.26357614)；[LLMPatientSafety](https://github.com/rachellea/LLMPatientSafety) | 背景；评分 schema 参考 | RadGuide AI 无公开数据或代码 |

### 评测基准与模拟器：骨架可以用 MIT 组件拼出来，放射防护内容必须自写

IRSA-Bench 的每个组成部分都有宽松许可的模板。要做一个将来可以商用的基准，最干净的底座是：**τ²-bench（MIT）+ HealthBench 评分细则格式（MIT）+ AgentDojo 注入套件（MIT）**，全部在 **Inspect AI（MIT）**里运行（[τ²-bench](https://github.com/sierra-research/tau2-bench)；[HealthBench](https://huggingface.co/datasets/openai/healthbench)；[AgentDojo](https://github.com/ethz-spylab/agentdojo)；[Inspect](https://github.com/UKGovernmentBEIS/inspect_ai)）。具体做法【推断】：在 τ²-bench 中新增 `irsa` 域，策略取规则引擎的输出，工具包括剂量引擎和辖区查询，数据库保存患者和家庭状态；模拟患者借鉴 MedDialogRubrics 的"原子事实"约束和 MediQ 的"隐藏信息"模式，每个模拟家庭有一张事实卡（年龄、是否怀孕、是否同床、通勤时长、宠物、工作），模拟患者只能依据事实卡回答；"必问项"由规则引擎需要哪些事实来决定。数字评分借鉴 MedCalc-Bench 的容差带和 MedMCP-Calc 的工具选择评分，但"数字只能来自引擎"这一项**零容差**。每个场景运行 k 次（例如 k = 4–8），安全关键题目要求 k 次全部安全。PatientAgentBench（CC BY-NC）、MedPRESS（受限访问，仅限研究）、Q4Dx（文章 CC BY-NC-ND，仓库无许可证）和 MedDialogRubrics（无许可证）只能借鉴设计，不能打包它们的数据。**目前没有任何基准覆盖"辖区切换"和"规则版本漂移"**，这是 IRSA-Bench 的新内容：NRC Rev. 1 对 DG-8061/DG-8070，德国对瑞士对日本。

| 名称 | 测什么 | 规模 | 许可证/商用 | 获取方式 | 链接 | IRSA 用法 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **HealthBench**（OpenAI） | 多轮健康对话，按医生编写的评分细则打分 | 5,000 段对话，48,562 条评分标准 | **MIT**（代码和数据） | HF；simple-evals；已有 Inspect 移植 | [HF](https://huggingface.co/datasets/openai/healthbench)；[simple-evals](https://github.com/openai/simple-evals) | 复制评分细则 schema，写 IRSA 专用条目 | 评审是 LLM，需要用物理师打分来校准 |
| MedDialogRubrics | 基于指南的"必问项"评分细则；模拟患者锚定原子事实 | 5,200 个合成病例，6 万多条评分标准 | **未声明** | HF | [HF](https://huggingface.co/datasets/AQ-MedAI/MedDialogRubrics) | 借鉴方法，不用数据 | 许可证不明 |
| MediQ | 交互式追问；专家系统不确定时先问再答 | 见论文 | **CC BY 4.0** | GitHub | [mediQ](https://github.com/stellalisy/mediQ) | 隐藏信息模式 | README 本轮未能读取 |
| AgentClinic | 医生、患者、检查三方智能体；可注入偏见 | 215 + 120 例；另有 MIMIC-IV 套件 | **MIT** | GitHub | [AgentClinic](https://github.com/SamuelSchmidgall/AgentClinic) | 患者偏见设定 → 谄媚脚本 | MIMIC-IV 需 PhysioNet 资质 |
| PatientAgentBench（Amazon） | 面向患者的智能体 + 有状态工具沙盒 + LLM 评审团 | 生成器 + 样例 | **CC BY-NC 4.0，不可商用** | GitHub | [GitHub](https://github.com/amazon-science/PatientAgentBench) | 架构模板，自行重写，不要 fork | 1,200 个场景的说法是二手信息 |
| MedAgentBench（Stanford） | 智能体在 FHIR EHR 上执行任务 | 300 个任务 | **MIT** | GitHub + Docker | [GitHub](https://github.com/stanfordmlgroup/MedAgentBench) | 把确定性后端暴露为工具并对状态变化评分 | v2 只有二手描述 |
| **τ²-bench**（Sierra） | 工具-智能体-用户交互，受领域策略约束；Pass^k；6 种语言 | 900 个任务 | **MIT** | GitHub；Inspect 移植 | [τ²-bench](https://github.com/sierra-research/tau2-bench) | 最适合 fork：新增 `irsa` 域 | 多语言有利于覆盖 IAEA 成员国 |
| MedCalc-Bench | 55 种计算器，小数容差 ±5% | 1,100 测试 / 10,543 训练 | v1.2 **CC BY-SA 4.0**；v1.0 CC BY 4.0（受限访问） | HF | [v1.2](https://huggingface.co/datasets/ncbi/MedCalc-Bench-v1.2) | 容差带评分 | 共享相同方式：衍生数据集也必须是 CC BY-SA |
| MedMCP-Calc（ACL 2026） | 通过 MCP 选择并调用医学计算器 | 118 个场景 | 代码 **MIT** | GitHub | [GitHub](https://github.com/SPIRAL-MED/MedMCP-Calc) | MCP 工具设计与工具选择评分 | EHR 据称源自 MIMIC-IV |
| MedCalc-Pro、EHR-ChatQA、Q4Dx、Preformulation Gap、CV-MedBench | 结构化校验；先澄清再执行；部分信息下的追问效率；标准化患者；不确定性 | — | 未核实或无 LICENSE；Q4Dx 文章 CC BY-NC-ND | GitHub/HF | [MedCalc-Pro](https://github.com/easymoney7774/MedCalc-Pro)；[EHR-ChatQA](https://github.com/glee4810/EHR-ChatQA)；[MedQDx](https://github.com/MaiWert/MedQDx) | 方法参考 | 许可证不明 |
| Decide, Ask, or Defer；Knowing but Not Showing | 决定、追问、转交三类行动评分；三分类回应判别器 | 200 个证据状态 | 未找到代码 | arXiv | [2610.04542](https://arxiv.org/abs/2610.04542)；[2605.25284](https://arxiv.org/abs/2605.25284) | 三类行动分别计分 | 只是方法模板【预印本】 |
| AmbigQA；ClarQ/ClariQ；MedDialog；MTSamples | 歧义分类；澄清问题；医患对话；病历 | — | CC BY-SA 3.0；无许可证；未知；来源有风险 | HF/GitHub | [AmbigQA](https://huggingface.co/datasets/sewon/ambig_qa)；[ClarQ](https://github.com/vaibhav4595/ClarQ) | 风格参考，优先级低 | MedDialog 不要用于商业 |
| **MedPRESS** | 患者施压下的多轮谄媚，5 轮递进 | 600 段对话 | HF 受限访问，"仅限研究" | HF | [HF](https://huggingface.co/datasets/samanjoy2/medpress_dataset) | 复制 5 轮递进结构 | 按非商用处理 |
| SycEval；Why LLMs Give In；SycoEval-EM；Inspect `sycophancy`/`mask` | 反驳类型分类；身份权威 × 证据水平；急诊谄媚；通用谄媚 | — | 无仓库 / 未查 / 未查 / MIT | arXiv、Inspect | [SycEval](https://arxiv.org/abs/2502.08177v4)；[Inspect sycophancy](https://github.com/UKGovernmentBEIS/inspect_evals/tree/main/src/inspect_evals/sycophancy) | 施压脚本中加入"我是护士"等身份声明 | — |
| **AgentDojo** | 工具型智能体的间接提示注入 | 97 个用户任务，629 个安全测试用例 | **MIT** | GitHub；Inspect 移植 | [AgentDojo](https://github.com/ethz-spylab/agentdojo) | 写 `irsa` 套件：在地图/POI 描述、网页手册、患者上传的 PDF 中注入指令 | 可扩展的环境，不是静态测试集 |
| InjecAgent | 间接注入 | 1,054 个用例 | **无 LICENSE** | GitHub | [InjecAgent](https://github.com/uiuc-kang-lab/InjecAgent) | 攻击模板 | 默认版权 |
| MedSafetyBench | 有害医疗请求 | 1,800 条示范 + 74,374 条请求 | MIT，但 README 写"仅限研究"（冲突） | GitHub | [GitHub](https://github.com/AI4LIFE-GROUP/med-safety-bench) | 拒答与过度拒答检查 | 把"仅限研究"视为使用请求 |

| 工具 | 作用 | 许可证 | 链接 | 备注 |
|---|---|---|---|---|
| **Inspect AI** + inspect_evals | 评测框架；已移植 HealthBench、τ²、AgentDojo、sycophancy、mask、agentharm | MIT | [Inspect](https://github.com/UKGovernmentBEIS/inspect_ai)；[inspect_evals](https://github.com/UKGovernmentBEIS/inspect_evals) | 没有 MedCalc-Bench 和 MediQ 的移植 |
| simple-evals；OpenAI evals | HealthBench 参考评审；旧版评测库 | MIT | [simple-evals](https://github.com/openai/simple-evals) | 2025 年 7 月起不再更新新模型 |
| promptfoo | 回归测试与红队测试，可接 CI | MIT | [promptfoo](https://github.com/promptfoo/promptfoo) | 用于护栏的每夜回归 |
| DeepEval | pytest 风格的 LLM 指标 | Apache-2.0 | [DeepEval](https://github.com/confident-ai/deepeval) | — |
| Ragas | RAG 忠实度等指标 | Apache-2.0 | [Ragas](https://github.com/explodinggradients/ragas) | 只评解释性文字，数字交给校验器；仓库可能已迁移 |
| Tonic Validate | Koller 等使用的 RAG 评测工具 | 未查 | [tonic-validate](https://github.com/TonicAI/tonic-validate) | 可复现 Koller 的评测 |

### 护栏、规则引擎、结构化输出与 MCP：全部有 MIT 或 Apache 选项

当前的安全智能体模式都指向同一个设计：确定性编排器掌控流程；引擎以类型化 schema 暴露为工具；不可信文本（地图/POI 文本、用户自由文本）交给一个**无工具权限的隔离模型**处理，这是 CaMeL 的做法；每条输出在显示前都要校验，每一步都要留痕。CaMeL 在 AgentDojo 上以可证明的安全性完成 77% 的任务，不设防时为 84%（[arXiv 2503.18813](https://arxiv.org/abs/2503.18813)）。CHAI 2026 年 7 月的智能体指南明确指出，很多工作流应当是确定性的"技能"而不是智能体（[CHAI](https://www.chai.org/blog/coalition-for-health-ai-chai-releases-new-agentic-best-practice-guide-and-testing)）。IRSA 的参考架构【推断】由六部分组成：确定性状态机编排器，负责槽位填充和引擎驱动的追问；类型化工具，包括 `dose_engine.predict`、`rules.evaluate(jurisdiction, version, facts)`、`half_life.update`、`dosimeter.read`；隔离 LLM，只能输出带 `untrusted` 来源标签的类型化事实；表达 LLM，只接收已校验的槽位；输出校验器，确定性地核对每个数字、单位、时长和限制，失败就退回模板；端侧哈希链审计日志，同时满足 AI Act 第 12 条日志要求，也可作为日后 ISO 14155 的源数据。

| 工具 | 功能 | 许可证 | 链接 | IRSA 用法 | 注意事项 |
|---|---|---|---|---|---|
| NeMo Guardrails | Colang 编写的输入、输出、对话、检索护栏 | Apache-2.0 | [GitHub](https://github.com/NVIDIA-NeMo/Guardrails) | 对话护栏：先追问再建议，限定话题 | — |
| **Guardrails AI** | 输出校验器（schema、正则、自定义），可要求重答 | Apache-2.0 | [GitHub](https://github.com/guardrails-ai/guardrails) | 拒绝任何不在引擎输出集合中的数字 | Hub 上的校验器许可证各自不同 |
| LlamaFirewall | PromptGuard、AlignmentCheck、CodeShield | **MIT**；PurpleLlama 顶层仓库为 **Llama 3.2 社区许可** | [GitHub](https://github.com/meta-llama/PurpleLlama/tree/main/LlamaFirewall) | 扫描 POI/地图和 RAG 文本 | PromptGuard 权重适用 Llama 许可 |
| LLM Guard | 输入输出扫描（注入、PII、毒性） | MIT | [GitHub](https://github.com/protectai/llm-guard) | 护栏 | — |
| Presidio | PII 检测与匿名化 | MIT | [GitHub](https://github.com/microsoft/presidio) | 记录日志或评测前去除标识 | — |
| **CaMeL** 参考代码 | 基于能力标签的不可信数据隔离 | Apache-2.0 | [GitHub](https://github.com/google-research/camel-prompt-injection) | 隔离地图/POI 文本 | 研究原型 |
| gpt-oss-safeguard | 开放权重的策略分类器 | Apache-2.0 | [HF](https://huggingface.co/openai/gpt-oss-safeguard-20b) | 自定义策略，例如识别"给出了出院决定" | 可自托管 |
| Llama Guard 4 | 安全分类器 | Llama 许可 | [HF](https://huggingface.co/meta-llama/Llama-Guard-4-12B) | 护栏 | 有许可限制 |
| **json-logic-js**（Python 移植 json-logic-qubit） | 可移植的 JSON 规则格式 | MIT | [GitHub](https://github.com/jwadhams/json-logic-js)；[PyPI](https://pypi.org/project/json-logic-qubit/) | 端侧规则运行时；签名的 JSON 规则包 | 体积小，可嵌入 |
| **Catala** | 文学式法律代码编译器，可编译到多种目标语言 | Apache-2.0 | [GitHub](https://github.com/CatalaLang/catala) | 规则撰写，保留与法规原文的对应 | — |
| **OPA/Rego** | 声明式策略引擎，带版本的策略包和决策日志 | Apache-2.0 | [GitHub](https://github.com/open-policy-agent/opa) | 服务器端规则、可审计的决策日志 | — |
| Drools；ProbLog | 规则引擎；概率逻辑 | Apache-2.0 | [Drools](https://github.com/apache/incubator-kie-drools)；[ProbLog](https://github.com/ML-KULeuven/problog) | 备选；家庭事实未知时的规则 | — |
| OpenFisca | 规则即代码 | **AGPL-3.0** | [PyPI](https://pypi.org/project/openfisca-core/) | 只作参考，不要放进 App | AGPL |
| **Outlines**；XGrammar；llguidance | 约束解码到 JSON Schema、正则或语法 | Apache-2.0；Apache-2.0；MIT | [Outlines](https://github.com/dottxt-ai/outlines)；[XGrammar](https://github.com/mlc-ai/xgrammar)；[llguidance](https://github.com/guidance-ai/llguidance) | LLM 只输出带槽位引用的模板，数字由引擎填 | — |
| Pydantic | 类型化数据模型 | MIT | [PyPI](https://pypi.org/project/pydantic/) | 槽位 schema | — |
| **MCP**（规范 2026-07-28；Python SDK；TS SDK） | 无状态协议，`outputSchema` 支持 JSON Schema 2020-12，`_meta` 中传递 OpenTelemetry 追踪上下文 | 项目从 MIT 迁往 Apache-2.0；Python SDK 为 MIT；TS SDK 正在迁移 | [changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog.md)；[Python SDK](https://github.com/modelcontextprotocol/python-sdk) | 需要服务边界时暴露引擎 | 两种许可都宽松 |

### 开源模型：Gemma 4 和 Qwen3.5 是许可风险最低的默认选择

截至 2026 年 11 月，有多个强而且许可宽松的开放权重模型家族。**Gemma 4**（2026 年 3 月，Apache-2.0，支持 140 多种语言，E2B/E4B 面向端侧）和 **Qwen3.5/Qwen3.8-27B**（Apache-2.0，201 种语言和方言）是 IAEA TC 多国部署场景下风险最低的默认选择（[Gemma 4](https://huggingface.co/google/gemma-4-E4B-it)；[Qwen3.5-9B](https://huggingface.co/Qwen/Qwen3.5-9B)）。MedGemma 经过医学调优，但适用 Google 的 HAI-DEF 条款：允许商用，但有附加条件，禁止"无执照行医"，并要求在适用时取得监管授权（[HAI-DEF 条款](https://developers.google.com/health-ai-developer-foundations/terms)；[禁止用途](https://developers.google.com/health-ai-developer-foundations/prohibited-use-policy)）。在 IRSA 的设计中，数字和决定只来自确定性的引擎和规则，所以医学微调没有指令遵循、结构化输出和多语言质量重要【推断】。所有候选模型都应在开启约束解码的条件下跑 IRSA-Bench。

| 模型 | 尺寸/日期 | 许可证 | 能否商用 | 多语言/端侧 | 链接 | 注意事项 |
|---|---|---|---|---|---|---|
| **Gemma 4**（E2B、E4B、12B、26B-A4B、31B） | 2026-03 至 06 | Apache-2.0 | 能 | 140 多种语言；E2B/E4B 面向端侧 | [HF](https://huggingface.co/google/gemma-4-E4B-it) | 端侧最简单的选择 |
| Gemma 3n（E2B/E4B） | 2025-06 | Gemma 使用条款 | 能，受条款和禁止用途约束 | LiteRT 端侧 | [HF](https://huggingface.co/google/gemma-3n-E4B-it) | 已被 Gemma 4 取代 |
| **Qwen3.5**（0.8B–35B-A3B）/ Qwen3.8-27B | 2026-02 / 2026-08 | Apache-2.0 | 能 | 201 种语言，原生 262K 上下文 | [Qwen3.5-9B](https://huggingface.co/Qwen/Qwen3.5-9B)；[Qwen3.8-27B](https://huggingface.co/Qwen/Qwen3.8-27B) | 部分部署需考虑数据主权政策 |
| Qwen3 | 2025-04 | Apache-2.0 | 能 | — | [HF](https://huggingface.co/Qwen/Qwen3-8B) | — |
| gpt-oss 20B/120B | 2025-08 | Apache-2.0 | 能 | 20B 可在单卡或笔记本上运行 | [HF](https://huggingface.co/openai/gpt-oss-20b) | — |
| Mistral Small 3.1/3.2（24B）、Small 4（119B） | 2025、2026-01 | Apache-2.0 | 能 | 欧盟厂商 | [HF](https://huggingface.co/mistralai/Mistral-Small-3.2-24B-Instruct-2506) | — |
| Phi-4 系列 | 2024-12 至 2025-04 | MIT | 能 | 小，适合边缘设备 | [HF](https://huggingface.co/microsoft/phi-4) | 多语言能力未查 |
| Apertus（Swiss AI）8B/70B，v1.5 | 2025-09 至 2026-07 | Apache-2.0 | 能 | 数据和配方都开放 | [HF](https://huggingface.co/swiss-ai/Apertus-v1.5-8B) | 语言数量未核实 |
| MedGemma 4B/27B；MedGemma 1.5 4B | 2025、2026-01 | HAI-DEF 条款 | 有条件 | 医学文本与图像；4B 可端侧 | [HF](https://huggingface.co/google/medgemma-1.5-4b-it) | 现行条款只读到检索片段，决策前要重读原文 |
| MeditronFO（EPFL） | 2026-05 | OLMo 版为 Apache-2.0；Gemma 版适用 Gemma 条款 | 视底座而定 | 临床微调，流程可审计 | [HF](https://huggingface.co/EPFLiGHT/OLMo-2-32B-MeditronFO) | — |
| Llama 4 Scout/Maverick | 2025-04 | Llama 4 社区许可，人工审核放行 | 有条件 | — | [HF](https://huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct) | 许可证原文本轮未读 |
| Aya Expanse；Tiny Aya | 2024-10；2026-02 | **CC BY-NC 4.0** | **不能** | 多语言 | [HF](https://huggingface.co/CohereLabs/tiny-aya-global) | 只用于研究或评测 |

端侧运行时都是宽松许可：llama.cpp（MIT）、MLC LLM（Apache-2.0）、ExecuTorch（BSD）、LiteRT/LiteRT-LM（Apache-2.0）（[llama.cpp](https://github.com/ggml-org/llama.cpp)；[LiteRT-LM](https://github.com/google-ai-edge/LiteRT-LM)）。系统自带模型免下载，但无法替换或锁定版本：iOS 26 的 Foundation Models 框架只在支持 Apple Intelligence 的设备上可用，Gemini Nano 通过 ML Kit GenAI 调用，具体设备支持未从官方文档核实（[Apple 文档](https://developer.apple.com/documentation/foundationmodels)；【二手】[DField](https://www.dfieldsolutions.com/blog/on-device-llms-2026)）。随 App 打包开放小模型的好处是版本可控，模型哈希可以写进 SOUP 清单。云端方面，Anthropic 第一方 API 的 `inference_geo` 只接受 `"global"` 和 `"us"`，静态存储只有 `"us"`（[Anthropic 数据驻留](https://platform.claude.com/docs/en/manage-claude/data-residency)），要在欧盟境内处理，需要走 Bedrock/Vertex 的欧盟端点、Azure EU 数据区，或者用端侧/自托管模型。成本可以忽略：按 2k 输入、300 输出 token 计，一轮表达在 Haiku 5.5 上约 $0.0004，在 Sonnet 5.5 上约 $0.007；每位患者约 50 轮，合计约 2–35 美分（[Anthropic 定价](https://platform.claude.com/docs/en/about-claude/pricing)，价格需在官网实时核对）【推断】。原型阶段的建议是：模板渲染作为始终可用的基线；端侧模型作为主力；欧盟托管的云端模型只用于合成数据上的 IRSA-Bench 对比。阶段 0–3 不得把真实患者数据发给云端 LLM。

### 移动端与 BLE 栈：原生 BLE 层加共享内核，避开 flutter_blue_plus

医疗级 BLE 流式 App 最稳妥的做法是**原生 BLE 层加共享内核**：iOS 用 CoreBluetooth，Android 用 companion device API 和 `connectedDevice` 前台服务。后台 BLE 在两个系统上都可行，但有硬性限制。iOS 上用户强制退出 App 后，状态恢复不会重新拉起 App，除非使用 **AccessorySetupKit（iOS 26+）**（[Apple 论坛 840468](https://developer.apple.com/forums/thread/840468)；[TN3115](https://developer.apple.com/documentation/technotes/tn3115-bluetooth-state-restoration-app-relaunch-rules)）。Android 14 要求每个前台服务声明类型，长连接推荐用 CompanionDeviceManager（[Android FGS](https://developer.android.com/about/versions/14/changes/fgs-types-required)；[Companion](https://developer.android.com/develop/connectivity/bluetooth/companion-device-pairing)）。由此得出几条工程要求【推断】：GATT 服务要带序号、时间戳、设备端缓冲和电池/健康特征；半衰期估计器要显式处理缺失数据；界面要显示数据"陈旧度"；数据超过阈值时间未更新时，引擎拒绝给出基于时间的建议。应用商店方面，Apple 审核指南 1.4.1 对可能给出不准确数据的医疗 App 审查更严，5.1.3 禁止把健康数据用于广告或数据挖掘【二手】（[ConductAtlas](https://conductatlas.com/platform/apple/apple-app-store-review-guidelines/healthcare-and-medical-app-requirements/)）。Google Play 要求所有开发者完成健康类 App 声明，**封闭测试轨道也不例外**（[Google Play](https://support.google.com/googleplay/android-developer/answer/13996367)）。阶段 3 应通过 TestFlight、Play 封闭测试或机构 MDM 分发。

| 名称 | 作用 | 许可证/商用 | 链接 | 注意事项 |
|---|---|---|---|---|
| Kotlin Multiplatform + **Kable** | 跨平台 BLE（Android、iOS、macOS、JS、JVM），协程 API | Apache-2.0 | [Kable](https://github.com/JuulLabs/kable) | 推荐：共享内核 + SwiftUI/Compose 原生 UI |
| Compose Multiplatform | 跨平台 UI | Apache-2.0 | [GitHub](https://github.com/JetBrains/compose-multiplatform) | — |
| Flutter + **flutter_reactive_ble**（Signify） | Flutter 方案的 BLE 库 | BSD-3；BSD | [Flutter](https://github.com/flutter/flutter)；[flutter_reactive_ble](https://github.com/PhilipsHue/flutter_reactive_ble) | 团队偏好 Flutter 时选它 |
| React Native + react-native-ble-plx | RN 方案 | MIT；Apache-2.0 | [ble-plx](https://github.com/dotintent/react-native-ble-plx) | — |
| Nordic Android-BLE-Library / IOS-BLE-Library | 原生 BLE 封装 | BSD-3 | [Android](https://github.com/NordicSemiconductor/Android-BLE-Library)；[iOS](https://github.com/NordicSemiconductor/IOS-BLE-Library) | — |
| **flutter_blue_plus** | Flutter BLE | **FlutterBluePlus License v1.6**：个人、注册非营利组织和认证教育机构免费；**任何营利用途（包括开发、测试、评估）都要付费**；禁止重新授权；**构建时可能发送许可遥测** | [LICENSE](https://raw.githubusercontent.com/chipweinberger/flutter_blue_plus/master/LICENSE.md) | 大学可豁免，但衍生公司的全部开发都需要商业许可；遥测必须写进 DPIA |
| HAPI FHIR；Google Android FHIR SDK；fhir.resources | FHIR R4 导出 | Apache-2.0；Apache-2.0；BSD | [HAPI](https://github.com/hapifhir/hapi-fhir)；[android-fhir](https://github.com/google/android-fhir)；[fhir.resources](https://pypi.org/project/fhir.resources/) | 与 IPS STU2 和 HL7 Europe EPS 对齐；R6 预计 2026 年底到 2027/2028 年定稿 |

### 医疗器械软件标准与模板：按现行标准做，别等 IEC 62304 第 2 版

现行的标准组合是：IEC 62304:2006+A1:2015、ISO 14971:2019（ML 部分补充 BS/AAMI 34971:2023）、IEC 62366-1、EN IEC 81001-5-1:2022。AI 方面有 IMDRF GMLP N88（2025 年 1 月定稿）、FDA PCCP 最终指南（2024 年 12 月）、FDA AI 生命周期指南（仍是 2025 年 1 月的草案）和 MDCG 2025-6。Regulation (EU) 2026/1744 把 MDR 产品的 AI Act 高风险义务推迟到 **2028-08-02** 适用（[QLE](https://www.qle.be/the-qle-regulatory-brief/ai-acts-digital-omnibus-is-now-law-ai-medical-devices-get-until-2-august-2028-but-notified-bodies-face-a-28-january-2028-deadline)）。IEC 62304 第 2 版**尚未发布**，发布时间各方说法不一：Johner 认为 2026 年 7 月工作组计划的是 CD2，FDIS 不早于 2028 年；Critical Software 则称 FDIS 已于 2026 年 5 月开始，2027 年发布（[Johner](https://blog.johner-institute.com/iec-62304-medical-software/iec-62304-2nd-edition-all-areas-of-application-and-changes/)；[Critical Software](https://asd.criticalsoftware.com/en/newsroom/iec-62304-edition-2-changes-august-2026)）。应当按"尚未发布、日期不确定"处理。

| 名称 | 状态（2026-11） | 许可/获取 | 链接 | IRSA 用法 | 注意事项 |
|---|---|---|---|---|---|
| IEC 62304:2006+A1:2015 | 现行；第 2 版处于 CD 阶段 | 付费标准 | [Johner](https://blog.johner-institute.com/iec-62304-medical-software/iec-62304-2nd-edition-all-areas-of-application-and-changes/) | 软件生命周期；安全等级以 B 级为底线 | 第 2 版草案用两级"过程严格度"取代 A/B/C |
| ISO 14971:2019 + BS/AAMI 34971:2023 | 现行 | 付费 | [BSI](https://knowledge.bsigroup.com/products/application-of-iso-14971-to-machine-learning-in-artificial-intelligence-guide) | 风险文件，含 ML 危害 | — |
| IEC 62366-1 | 现行（2026 年修订状态未查） | 付费 | — | 可用性工程 | — |
| IEC 82304-1 | 第 2 版状态未查 | 付费 | — | 健康软件产品 | — |
| EN IEC 81001-5-1:2022 | 现行；ISH1:2025 | 付费 | [Complife](https://www.complifegroup.com/2026/02/10/cybersecurity-regulatory-mdr-ivdr/) | 网络安全与威胁模型 | 是否已在 OJEU 中列为协调标准未确认 |
| IMDRF N88 GMLP；N93 草案 | 2025-01 定稿；2026 年草案 | 免费 | [N88](https://imdrf.org/sites/default/files/2025-02/IMDRF_AIML%20WG_GMLP_N88%20Final.pdf)；[N93](https://www.imdrf.org/sites/default/files/2026-04/IMDRF%20AIML%20WG%20N93%20DRAFT%202026%20-%20Technical%20Framework%20for%20Artificial%20Intelligence%20Life%20Cycle%20Management.pdf) | 数据与模型文档 | 定稿版新增"可复现结果" |
| FDA PCCP 最终指南 | 2024-12 | 免费 | [FR](https://www.govinfo.gov/content/pkg/FR-2024-12-04/html/2024-28361.htm) | 内部预先规定代理模型重训和规则包更新 | 有检索摘要说 2025 年 8 月定稿，没有一手来源支持 |
| FDA AI 器械软件生命周期指南 | **仍为草案**（2025-01），FY2027 优先定稿 | 免费 | [FDA](https://fda.gov/regulatory-information/search-fda-guidance-documents/artificial-intelligence-enabled-device-software-functions-lifecycle-management-and-marketing)；[RAPS](https://www.raps.org/resource/fda-s-device-center-releases-guidance-agenda-for-fy-2027.html) | 美国申报准备 | — |
| FDA CDS 与 General Wellness 指南 | 2026-01-06 修订 | 免费 | [FDA Law Blog](https://www.thefdalawblog.com/2026/01/a-busy-day-in-the-cdrh-neighborhood-updates-to-the-cds-and-general-wellness-guidance-documents/) | 判断是否属于器械 | 面向患者的功能大概率仍属器械【推断】 |
| MDCG 2025-6；Regulation (EU) 2026/1744 | 2025-06；2026-07 生效 | 免费 | [EC](https://health.ec.europa.eu/latest-updates/mdcg-2025-6-faq-interplay-between-medical-devices-regulation-vitro-diagnostic-medical-devices-2025-06-19_en)；[EUR-Lex](https://eur-lex.europa.eu/eli/reg/2026/1744/oj/eng) | MDR 与 AI Act 合并成一套技术文档 | 公告机构须在 2028-01-28 前申请 AI Act 指定 |
| MDCG 2023-1 | 现行 | 免费 | [PDF](https://health.ec.europa.eu/system/files/2023-01/mdcg_2023-1_en.pdf) | 确认院内豁免不适用 | — |
| ISO 14155:2026 | 2026-03-23 发布 | 付费 | [ISO](https://www.iso.org/standard/14155) | 阶段 3 的 CIP | EN 版过渡安排见 EVS |
| CHAI 智能体最佳实践指南 + T&E 框架 v1.0 | 2026-07-29 | 公开；T&E 仓库许可证未找到 | [CHAI](https://www.chai.org/blog/coalition-for-health-ai-chai-releases-new-agentic-best-practice-guide-and-testing) | 自主边界、红旗升级、同意范围对象 | — |
| Hauschild 2022 学术软件生命周期指南 | 已发表 | 开放获取 | [PMC9685384](https://pmc.ncbi.nlm.nih.gov/articles/PMC9685384/) | 精简版 62304 的最小文档集 | 没有工作量数据 |
| OpenRegulatory templates | 现行 | **CC BY-NC-SA 4.0** | [LICENSE](https://raw.githubusercontent.com/openregulatory/templates/master/LICENSE.md) | 只用于内部学习 | 不要带进将来公司会复用的交付物 |
| **OpenSaMD/templates**（早期分叉） | 早期快照 | **CC BY 4.0**，保留声明即可商用 | [LICENSE](https://raw.githubusercontent.com/OpenSaMD/templates/master/LICENSE.md) | 将来公司要继承的 QMS 文档以此为底 | 内容极简，可导入 Formwork eQMS |

## 三组矛盾的裁定：Levart 是两篇论文，DG-8070 是 2026 年草案

**Levart：2019 年和 2025 年确实是两篇不同的论文，物理笔记"2025 年那篇不存在"的说法是错的。** 两篇的作者有重叠（Levart、Kalogianni、Mulholland），2025 年那篇来自伦敦 King's College Hospital；两篇的内容和许可证都不同。第一篇是 Levart D, Kalogianni E, Corcoran B, Mulholland N, Vivian G，EJNMMI Phys 2019;6:7（doi:10.1186/s40658-019-0243-1），CC BY 4.0。它报告 177Lu-DOTATATE 患者在 0.1 m 和 1 m 处的剂量率、双指数有效半衰期（4.7 h 和 87.2 h）、出院时滞留率，以及据此推导的接触限制。文中估计，不加限制时伴侣剂量约为 6.2 mSv（[PMC6484059](https://pmc.ncbi.nlm.nih.gov/articles/PMC6484059/)）。第二篇是 Levart D, Wicks S, Heraghty N, Kalogianni E, Mulholland N，EJNMMI Phys 2025（DOI 10.1186/s40658-025-00776-x），**CC BY-NC-ND 4.0**，也就是 IRSA 论文参考文献中引用的 "EJNMMI Physics 2025;12(1):94"。它是照护者 TLD 研究：12 名家属连续佩戴 4 周，中位有效剂量 0.038 mSv，附有分时段的距离和时长日记（[PMC12595148](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/)）。物理笔记的 Crossref 查询声称"任何年份都没有 Levart 署名的 177Lu 论文"，可它自己又从 PMC 引用了 2019 年那篇，可见查询本身失败了；动力学笔记和情境笔记都通过 Europe PMC 读到了 2025 年论文的全文。两篇在工程上用途不同：2019 年那篇进入剂量引擎和动力学先验，表格可在署名后改编再利用；2025 年那篇作为端到端验证和行为校准的目标，只能引用，不能把改编后的表格再分发，个体数据需向作者索取。

**NRC 指南草案：DG-8061 是 2023 年草案，DG-8070 是 2026 年草案，本轮没有拿到后者全文。** 现行最终版是 RG 8.39 Rev. 1（2020-04-07）（[FR 2020-07307](https://www.govinfo.gov/content/pkg/FR-2020-04-07/html/2020-07307.htm)）。DG-8061 是 Rev. 2 的 2023 年草案，2023-04-21 公告征求意见，案卷 NRC-2023-0086（[FR 2023-08418](https://www.govinfo.gov/content/pkg/FR-2023-04-21/html/2023-08418.htm)）。DG-8070 是 2026 年重新发布的草案：91 FR 51749（2026-08-11），案卷 NRC-2026-3730，意见征集于 2026-09-10 截止。两份独立来源提到了它：SNMMI 的意见函（ADAMS ML26254A152），以及一份 2026 年 OSF 研究方案，后者写明 RG 8.39 正以 DG-8070 修订（[SNMMI 意见函](https://www.nrc.gov/docs/ML2625/ML26254A152.pdf)；[OSF k4e2q](https://osf.io/k4e2q/)）。LLM 语料笔记没有检索到 DG-8070，就推测它是 DG-8061 的误写，这个推测站不住；DRPS 的综述仍引用 DG-8061，说明二手来源的更新有滞后。按 SNMMI 意见函的转述，DG-8070 的照护者表 3 按 50 mSv、1 m 处 100% 占用计算，177Lu 基线为 41 GBq，131I 为 3.2 GBq；超过基线时，持证方可以改用实测剂量率、筛查标准或患者特异计算；SNMMI 要求把占用因子改为 0.25，这样 177Lu 基线会变成 164 GBq。工程上的处理办法是：Rev. 1 作为"现行"规则包；DG-8061 和 DG-8070 作为"草案"规则包，只用于版本漂移测试，界面上永远不能把草案当作现行规定；拿到 DG-8070 原文后，再核对 SNMMI 转述的数值。同样处于拟议阶段的，还有 NRC 2026 年 7 月"按整个疗程计算"的放行限值改革（[Federal Register 2026-14208](https://www.federalregister.gov/documents/2026/07/15/2026-14208/reforming-and-modernizing-the-nrcs-radiation-protection-framework)）。

**笔记中的许可证冲突和数据不一致保留如下，不作取舍：**

| 项目 | 说法 A | 说法 B | 处理建议 |
|---|---|---|---|
| PSMA-PET-CT-Lesions | TCIA：CC BY 4.0 | Tübingen FDAT 镜像：CC BY-NC 4.0 | 只用 TCIA 副本，并存档许可证页面快照 |
| PyTheranostics | GitHub LICENSE：Apache-2.0 | PyPI 元数据：MIT | 两者都宽松；向维护者确认，确认前按 Apache-2.0 保留 NOTICE【推断】 |
| EANM/SNMMI 177Lu-PSMA 指南 | 文章 CC BY 4.0 | 前言称非服务提供方"不得复制或修改" | 先改写并引用；逐字再分发前向 EANM 书面澄清 |
| Microsoft 建筑底图 | GitHub：CDLA-Permissive-2.0 | Overture 建筑主题内：ODbL | 需要 CDLA 条款时直接从 GitHub 获取 |
| Deep Blue 177Lu-DOTATATE | 物理笔记据 DataCite：CC BY-NC 4.0 | 动力学笔记：DataCite rights 字段为空，落地页 403 | 按非商用处理，人工核对落地页 |
| MRTDosimetry（OSF） | 物理笔记据 DataCite：CC BY 4.0 | 动力学笔记：未核实 | 在 OSF 页面核实 |
| FDG-PET-CT-Lesions | CC BY 4.0 | TCIA/NIH 受控访问政策 | 商用前在 TCIA 页面确认 |
| MedSafetyBench | MIT | README："仅限研究" | 视为仅限研究 |
| radot | 物理笔记：LGPL-3.0（读过 LICENSE） | 动力学笔记：未检查 | 按 LGPL-3.0 处理 |
| Ogino/Daisaki 派生数据 DOI | zenodo.24343109 | zenodo.24343110 | 大概是概念 DOI 与版本 DOI 的区别，引用时写明版本 |
| IEC 62304 第 2 版时间表 | Johner/VDE：计划 CD2，FDIS 不早于 2028 年 | Critical Software：FDIS 始于 2026 年 5 月，2027 年发布 | 按尚未发布处理 |
| Radiacode 能量下限 | 20 keV | 30 keV（指示）/ 100 keV（剂量） | 自行做能量响应测试 |
| Thermo PRD-ER4 能量范围 | 58 keV–6 MeV | 60 keV–1.3 MeV | 以现行厂商目录为准并实测 |
| Microsoft 建筑数量 | 14 亿 | 9.99 亿 | 不影响使用，引用时注明来源 |
| Schütze 保守有效半衰期 | 摘要称"第 90 百分位" | 推荐值高于正文中的第 90 百分位 | 理解为向上取整的推荐值 |
| PurpleLlama 与 LlamaFirewall | LlamaFirewall 子目录：MIT | 顶层仓库：Llama 3.2 社区许可 | 只取子目录，权重另按 Llama 许可 |

## 关键数据空白：七类数据必须自采，十余项必须索取

### 必须自己采集或生成的七类数据

以下数据在公开领域都没有，而且恰好对应 IRSA 最核心的科学问题。它们也是 TC 项目最可能留下的长期资产。如果打算将来公开这些数据，阶段 3 的知情同意和 DPIA 就必须从一开始写入数据共享条款，事后很难补【推断】。

| 数据 | 为什么公开领域没有 | 在哪个阶段采集 | 怎么采集 |
|---|---|---|---|
| 1. 佩戴式探测器在 RPT 后的逐患者原始时间序列 | 笔记检查的所有临床研究都只给"按请求提供"；Zenodo、Figshare、Deep Blue 上都没有（例如 [Kurth 2018](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/)、[Turku 2024](https://pmc.ncbi.nlm.nih.gov/articles/PMC11604915/)） | 阶段 1（体模）；阶段 3 第一步（德国 48 h 住院期配对 2 m 参考测量） | Radiacode 同时记录剂量率和能谱，与校准巡测仪、出院测量配对 |
| 2. 0.5 m 以内、多朝向的 177Lu 近场实测 | 已有来源只覆盖 0.1 m、1 m、2 m 的单点，体衰减和线源假设最关键的近距离区间没有数据（[Levart 2019](https://pmc.ncbi.nlm.nih.gov/articles/PMC6484059/)；[Struelens 2024](https://pmc.ncbi.nlm.nih.gov/articles/PMC11035505/)） | 阶段 1 | 在灌注 177Lu 的拟人体模上按 EURADOS 协议测 30/50/100 cm，朝向包括面对面和并排 |
| 3. CsI 小探测器在 113/208/364 keV 的能量与角响应 | 没有发表过 Radiacode、Kromek 或 GQ 在这些能量下的独立测试 | 阶段 1 | 137Cs、133Ba、瓶装 177Lu 实测，加 G4Radiacode 模拟 |
| 4. 体位、排尿与佩戴读数之间的关系 | 动作数据集里没有辐射探测器；Bailey 的阶跃归因于排泄只是推测，并未验证（[Bailey 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/)） | 阶段 1 体模 + 阶段 3 | 手机 IMU 记录事件标志，同时记录读数阶跃 |
| 5. "活度分布 → 接触者位置剂量场"的 177Lu 外照射训练集 | 现有 AI 剂量学数据几乎都是体内剂量；外照射 MC 参考只有 EURADOS（无 177Lu）和 Arasi（I-131） | 阶段 1 | GATE 10 在 ICRP、UF/NCI、TotalSegmentator 人体上计算，计分网格放在接触者位置 |
| 6. 同时记录真实距离和行为的家庭接触数据 | 调查没有距离，传感器没有行为；Levart 2025 的日记只发表了汇总值 | 阶段 3 | 照护者连续佩戴 TLD 4 周并记日记（沿用 Levart 设计），可加照护者 EPD |
| 7. 放射防护患者对话基准，以及 LLM 评审与专家的一致性数据 | 在 HF 和 GitHub 上检索放射性碘、治疗诊断、放射性药物、剂量学、保健物理、镥等关键词，都没有相关结果 | 阶段 2 | IRSA-Bench；由医学物理师标注一个子集，用来校准 LLM 评审 |

此外还有几项缺口优先级较低，可以留到后期：LMIC 户型和"无治疗前影像"条件下的先验；端侧 LLM 在表达任务上的延迟和电量；225Ac 的外照射数据（只找到 225Ac-lintuzumab 的结果：接触处中位 2.0 µSv/h，1 m 处 0.3 µSv/h，n = 19）（[PubMed 34750237](https://pubmed.ncbi.nlm.nih.gov/34750237)）。

### 需要向作者或机构索取的数据与许可

| 对象 | 要什么 | 为什么 | 依据 | 优先级【推断】 |
|---|---|---|---|---|
| Bailey 等（AOJNMB 2026） | 250 例 131I 的 1 分钟分辨率 CSV | 唯一的连续固定探测器滞留数据，可用来训练伪影模型；论文没有数据可用性声明 | [PMC12854189](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/) | 高 |
| Levart 等（2025） | 个体 TLD 剂量与距离/时长日记 | 端到端验证和行为分布；文章为 CC BY-NC-ND，数据按请求提供 | [PMC12595148](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/) | 高 |
| Schütze 等（8 中心）/ Kurth 等 | 逐患者 2 m 剂量率序列 | 有效半衰期先验与拟合窗口偏倚的个体级验证 | [Schütze 2026](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)；[Kurth 2018](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/) | 高（与德国试点中心同一网络） |
| Monserrat Fuertes 等 | 原始全身剂量率 | 早期与晚期窗口的个体对照 | [PMC10477145](https://pmc.ncbi.nlm.nih.gov/articles/PMC10477145/) | 中 |
| Koller 等（MedChat） | 仓库许可证（目前没有 LICENSE） | 英德双语种子题目和 RAG 流程；与 IRSA 同一课题组，最容易拿到 | [MedChat](https://github.com/piakoller/MedChat) | 高，成本低 |
| Fele-Paranj/Rahmim（PBRPK） | SBML 代码许可证 | 离线生成全身和膀胱曲线；两个仓库都没有许可证 | [alifele](https://github.com/alifele/Computational-Physics)；[qurit](https://github.com/qurit/PSMA-PBPK-Model-Matlab) | 中 |
| HPS（ATE 编辑） | 把改写后的问题题干用作基准提示的书面许可 | 真实照护者提问的最好来源；没有文本挖掘条款 | [HPS 编辑](https://hps.org/editor)；[HPS 免责声明](https://hps.org/disclaimer/) | 中 |
| ICRP（annals@icrp.org） | 体模文件和出版物文本的再利用许可 | 即使免费获取的出版物，再利用也需许可 | [ICRP](https://icrp.org/page.asp?id=5) | 高（MC 依赖体模） |
| IAEA（经 TC 渠道，NUCLEUS） | SRS-63、SSG-46、RPOP 文本的再利用与翻译许可 | IAEA 版权，通常收费；TC 资助身份可能便于获得 | [IAEA 目录说明](https://www-pub.iaea.org/MTCD/publications/PDF/PublicationsCatalogue.pdf)；[NUCLEUS](https://nucleus.iaea.org/) | 高 |
| EANM；SNMMI；各 NHS 信托 | 指南再分发澄清；患者须知和手册的入库许可 | 条款冲突或不明 | [PMC10317889](https://pmc.ncbi.nlm.nih.gov/articles/PMC10317889/)；[SNMMI 须知](https://sites.snmmi.org/AM/Patients/Fact-Sheets/Guidelines-for-Patients-Receiving-Radioiodine-I-131-Treatment.aspx) | 中 |
| NCI（UF/NCI 体模） | 软件转让协议；如需商用，联系技术转让中心 | 唯一的孕妇和 BMI 系列体模 | [申请入口](https://ncidose.github.io/portal/request-access/) | 高 |
| Peter MacCallum（DEEP-PSMA）；Radboud；Vivli | 访问权限 | 177Lu-PSMA 先验队列；受限数据 | [DEEP-PSMA](https://deep-psma.grand-challenge.org/)；[Radboud](https://doi.org/10.34973/60e0-k133)；[Vivli](https://doi.org/10.25934/00000313) | 中/低 |
| Ozella 等（马拉维村庄） | 传感器原始数据 | 最接近 TC 受援国家庭的接触数据 | [EPJ Data Science](https://link.springer.com/article/10.1140/epjds/s13688-021-00302-w) | 中 |
| MedDialogRubrics、InjecAgent、EHR-ChatQA、MedQDx、LLMPatientSafety 作者 | 许可证 | 仓库没有许可证或未声明 | 见评测一节 | 低 |
| 付费或注册渠道 | Smith & Stabin 2012 的 Γ 值（购买）；RSICC DLC-129（注册）；PHITS（机构许可）；XCAT（询价） | 177Lu 的 Γ 和积累因子没有开放替代 | [Smith & Stabin](https://doi.org/10.1097/HP.0b013e318235153a)；[RSICC](https://rsicc.ornl.gov/codes/dlc/dlc1/dlc-129.html) | 高（Γ）/ 中 |

### 用合成数据补哪些缺口，补不了哪些

| 缺口 | 合成方法【推断】 | 依据数据 | 局限 |
|---|---|---|---|
| 佩戴式读数轨迹 | 每种化合物的有效半衰期用对数正态先验；双指数结构；显式的"早期拟合偏倚"项；按 Kurth 排泄曲线设置排尿阶跃；按 Bailey 的伪影分类设置三种模式：体位造成可回复的乘性电平变化，排泄造成不可逆的下降阶跃，淋浴或离开造成长时间低值 | [Schütze](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)、[Kurth](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/)、[Monserrat Fuertes](https://pmc.ncbi.nlm.nih.gov/articles/PMC10477145/)、[Bailey](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/)、[Turku](https://pmc.ncbi.nlm.nih.gov/articles/PMC11604915/) | 佩戴式探测器的距离伪影大多会消失，取而代之的是体型和位置伪影，后者未经检验 |
| 躯干探测器读数 | 用 Deep Blue 的平面全身像时间序列和 TIA 图，模拟躯干探测器在 4 个时间点的读数；用 G4Radiacode 模拟晶体响应 | [y4xd-s758](https://doi.org/10.7302/y4xd-s758)；[9nct-bk44](https://doi.org/10.7302/9nct-bk44) | 只有 2 例，且为 CC BY-NC |
| 外照射场 | GATE 10 计算，照搬 radot p142 的配置，把计分改到接触者网格 | [radot p142](https://doi.org/10.5281/zenodo.23809698)；[EURADOS](https://doi.org/10.5281/zenodo.8414338) | 必须用体模实测（缺口 2）验证 |
| 家庭场景 | 接触调查给时长 + 时间利用给时间预算 + Swiss Dwellings 按 DHS 缩放得到户型 + 按 Levart 分布做重尾抽样 | [冈比亚](https://zenodo.org/records/13101862)；[Swiss Dwellings](https://zenodo.org/records/7788422)；[Levart 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/) | 距离仍然靠假设 |
| 对话场景 | 结构化场景语法（家庭构成、核素、活度、地图情境、对抗性地图文本）；用 Synthea 生成患者背景；不用 SDV（BUSL） | [Synthea](https://github.com/synthetichealth/synthea) | 模拟用户会高估真实获益，必须另做真实用户研究 |
| 真实提问措辞 | HPS 拒绝授权时，按 ATE 的类别主题人工改写题干 | [ATE 索引](https://hps.org/publicinformation/ate/) | 措辞的真实性打折扣 |

## 商用红线：七类资产必须和产品通道隔离

| 类别 | 资产 | 研究阶段 | 商业化时 |
|---|---|---|---|
| **非商用 / 学术专用 / 仅限研究** | Deep Blue 全系列（含 DblurDoseNet 数据集）；DEEP-PSMA；PSMA 数据的 FDAT 镜像；UF/NCI 体模（STA）；RadData 数据；POLYMOD；SocioPatterns；ZInD；PatientAgentBench；MedPRESS；MedSafetyBench（"仅限研究"）；Aya/Tiny Aya；TotalSegmentator 授权子任务；HETUS/UKTUS 微观数据 | 可用于内部研究与验证 | 产品不得用其训练或打包；需另签协议 |
| **禁止演绎** | Levart 2025（CC BY-NC-ND）；Q4Dx 文章（CC BY-NC-ND） | 可引用、可复算 | 不得再分发改编后的表格 |
| **共享相同方式** | MedCalc-Bench v1.2（CC BY-SA 4.0）；AmbigQA（CC BY-SA 3.0）；OSM 和 Overture 建筑/交通（ODbL）；CubiCasa5K（CC BY-NC-SA）；OpenRegulatory 现行模板（CC BY-NC-SA） | 可用 | 衍生数据集或文档须以相同许可发布；含 NC 的两项完全不能商用 |
| **强 copyleft** | OpenFisca、EGSnrc（AGPL-3.0）；GGEMS、Open Gamma Detector、G4Radiacode（GPL-3.0）；RadData 包（GPL-2） | 只作离线基准或数据生成工具 | 不得进入分发的 App 或服务端 |
| **弱 copyleft** | OpenGATE、radot、TorchCP（LGPL-3.0） | 可用 | 作为离线工具或可替换的动态库使用；仅用于生成数据的工具不对 App 施加义务 |
| **源码可见但非开源** | SDV、Alibi Detect（BUSL-1.1） | 尽量不用 | 生产用途不允许 |
| **自定义或附条件** | FlutterBluePlus License（营利用途收费并带遥测）；MedGemma（HAI-DEF）；Gemma 3n（Gemma 条款）；Llama 4、Llama Guard 4、PurpleLlama 顶层仓库（Llama 许可）；Geant4 License（宽松，自定义）；PHITS（和平用途、出口管制）；MIRDcalc 条款；IDAC-Dose 版权；Google Maps 条款；IAEA、ICRP、HPS、SNMMI、NHS 版权；ExtraSensory（须引用，无正式许可） | 按条款使用 | 逐项法务审查 |
| **没有许可证（默认保留所有权利）** | Fele-Paranj PBPK；MedChat；MedDialogRubrics；InjecAgent；EHR-ChatQA；MedQDx；LLMPatientSafety；ClarQ/ClariQ；`hiroki-rad`、`ZhJiHo` 数据集；Open-RadiaCode-Android；MedDialog（未知）；Prem 矩阵（未确认）；MSD | 只能阅读和借鉴思路 | 不得复制，必须先取得授权 |
| **专利** | US 12377289；US 12011248 | — | 部署前做 FTO 审查 |

研究和商业化之间的隔离，靠的是一套从阶段 0 起就执行的规则【推断】。第一，SOUP/数据登记表加一列"通道"，只能填"研究"或"产品"，G0 门槛检查交付路径中没有 AGPL、GPL 或 BUSL 组件。第二，任何会离开团队的工件，包括 App 二进制、模型权重、规则包和公开基准，只能由产品通道资产生成：随产品发布的代理模型只用宽松许可数据或自有数据训练；非商用数据只用于内部研究和验证报告。至于在公司内部用非商用数据做评测算不算"商业用途"，笔记没有给出依据，需要法务确认。第三，copyleft 的 MC 工具只当作离线数据生成器使用，生成的模拟数据和工具本身分开登记。第四，体模只交付训练好的权重，不交付文件。第五，规则包只编码阈值并注明法规出处；IAEA 和 ICRP 的文字先改写，同时申请许可。第六，IRSA-Bench 用自写内容加 MIT 框架构建，以 CC BY/Apache 发布，不收录任何非商用数据。第七，QMS 文档以 OpenSaMD 的 CC BY 4.0 快照为底；BLE 用 Kable 或 flutter_reactive_ble；任何会联网回传的 SDK 都要先过 DPIA 和 SOUP 审查。

## 前 90 天行动清单

| 周次 | 目标 | 具体动作 | 产出 |
|---|---|---|---|
| 第 1–2 周 | 启动与登记 | 搭建仓库、CI、DVC、MLflow；建立分研究/产品通道的 SOUP 与许可证登记表；起草预期用途与声明矩阵；订购 2–3 台 Radiacode 103，借用 IEC 61526 认证 EPD 和校准巡测仪；发出第一批索取函（Bailey CSV、Levart 2025 原始数据、Schütze/Kurth、Monserrat Fuertes、MedChat 许可证、PBPK 许可证） | D0.1 草稿、D0.3 骨架 |
| 第 3–4 周 | 开放数据落地 | 下载 Deep Blue 全系列、TCIA 版 PSMA-PET-CT-Lesions、TotalSegmentator v3、EURADOS 和 Arasi 数据集、radot p142、ICRP 107/110/116/74/143/145、NIST XCOM；提交 UF/NCI STA；发出第二批许可函（ICRP、IAEA、HPS、EANM、SNMMI）；购买 Smith & Stabin，确定并记录 177Lu 的 Γ 取值；实现 NRC 默认基线 | 数据清单与数据集说明书 v0 |
| 第 5–6 周 | 动力学与合成器 | 搭建有效半衰期先验库（Schütze、Kurth、Monserrat Fuertes、Bailey、Turku、Levart 2019）；写合成轨迹生成器（排尿阶跃、Bailey 伪影）；filterpy 状态空间原型；半解析引擎 v0，用 Bellamy 的 BMI 体衰减作先验 | 合成数据集 v0；引擎黄金测试向量 |
| 第 7–8 周 | 台架首测与 BLE | Radiacode 用 137Cs 测线性、用 133Ba 测能量响应，条件允许时加测瓶装 177Lu；BLE 采集 App 原型（KMP + Kable 或原生实现），包含序号和缓冲；GATE 10 先复现 EURADOS 的 I-131 几何来验证流程，再在 ICRP 110 上计算 177Lu 接触者网格；完成 DPIA 草案和危害日志 v0 | D1.3 首批表征数据；D0.2、D0.4 草稿 |
| 第 9–10 周 | 规则与智能体骨架 | 用 json-logic 写规则包 v0.1（美国 §35.75 + RG 8.39 Rev. 1；德国 SSK）及测试向量；建立 DG-8061/DG-8070 草案包（DG-8070 待全文）；确定性编排器 + Pydantic 槽位 + Guardrails 校验器 + 模板渲染 + 哈希链审计日志 | D0.5；可演示的端到端确定性链路 |
| 第 11–13 周 | IRSA-Bench 雏形与 G0 | 在 Inspect 中搭 τ²-bench `irsa` 域和事实卡家庭模拟器；与物理师起草 HealthBench 格式的评分细则；写第一批 AgentDojo 注入用例；在合成数据上以患者为单位做第一轮保形校准（MAPIE/crepes）；召开 G0 评审；与德国中心做伦理预沟通，并就第 82 条问询主管机关 | G0 决议；IRSA-Bench v0；阶段 1 计划冻结 |

## 推荐入门套件：每个模块先拿这几样

| 模块 | 首选资源（1–3 个） | 理由 |
|---|---|---|
| 剂量计硬件 | Radiacode 103 + `cdump/radiacode`；G4Radiacode（离线）；一台 IEC 61526 认证 EPD 作对照 | 唯一现成的 BLE + 开放 API 组合；有数字孪生；有认证参考（[Radiacode](https://www.radiacode.com/products/radiacode-103)；[RadFlash](https://polimaster.com/products/dosimeters/radflash/)） |
| 动力学先验 | Schütze 2026；Kurth 2018；Monserrat Fuertes 2023（131I 加 Bailey 2026） | 覆盖化合物差异、排泄曲线和早期/晚期偏倚（[Schütze](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)） |
| 剂量引擎 | Bellamy 2022；NIST XCOM；ICRP 116 + 74 | 体衰减先验、衰减系数、接触者剂量与巡测仪量的换算（[Bellamy](https://pmc.ncbi.nlm.nih.gov/articles/PMC9667277/)） |
| MC 与代理模型 | OpenGATE 10 + radot；ICRP 110/145/143；EURADOS 数据集 | 主力引擎、参考人体、外照射验证协议（[opengate](https://pypi.org/project/opengate/)） |
| 不确定性与 OOD | MAPIE；crepes；pytorch-ood | 都是 BSD 或 Apache；crepes 可以直接给出超限概率 |
| 规则引擎 | 10 CFR 35.75 + RG 8.39 Rev. 1；json-logic；Catala（或服务器端 OPA） | 公共领域文本加可移植运行时（[eCFR](https://www.ecfr.gov/current/title-10/chapter-I/part-35/subpart-C/section-35.75)） |
| 情境：接触 | 冈比亚 2022（CC BY 4.0）；Levart 2025 汇总值；NRC SECY-18-0015 场景 | 可商用的 LMIC 先验加校准目标（[冈比亚](https://zenodo.org/records/13101862)） |
| 情境：地理 | Overture Places；Google Open Buildings；LandScan | 宽松许可，LMIC 覆盖好，可端侧离线使用 |
| 户型与动作 | Swiss Dwellings；DHS hv216；Capture-24 + ExtraSensory | 唯一宽松许可的大型户型数据；带如厕和睡眠标签 |
| 知识库 | eCFR/NRC 文本；EANM/SNMMI 177Lu-PSMA 指南（向 EANM 澄清后）；IAEA SRS-63（获许可后） | 可再分发的核心来源 |
| 评测 | τ²-bench；HealthBench；AgentDojo（都在 Inspect AI 中运行） | 全部 MIT，可直接扩展出 `irsa` 域（[τ²-bench](https://github.com/sierra-research/tau2-bench)） |
| 护栏 | Guardrails AI；Outlines；CaMeL 模式 | 数字校验、约束解码、不可信文本隔离 |
| 模型 | 模板渲染；Gemma 4 E4B；Qwen3.5 小尺寸 | 模板保底，两种 Apache-2.0 多语言模型做对比 |
| 移动端 | KMP + Kable；Nordic BLE 库；HAPI FHIR | 宽松许可，后台 BLE 可控，导出可用 FHIR R4 |
| 标准与模板 | Hauschild 2022；OpenSaMD 模板（CC BY 4.0）；IMDRF N88 | 精简生命周期、可商用模板、GMLP 原则（[Hauschild](https://pmc.ncbi.nlm.nih.gov/articles/PMC9685384/)） |

## 结论

公开资源盘点下来，IRSA 前期开发的瓶颈不在工具，而在三类只有 IRSA 自己能产生的数据：佩戴式探测器的原始轨迹、0.5 m 以内的 177Lu 近场实测、放射防护对话基准。工具层几乎全部有 MIT 或 Apache 选项，而真正贴近 IRSA 的数据（Deep Blue、DEEP-PSMA、UF/NCI 孕妇体模、Levart 2025、PatientAgentBench）偏偏大多带非商用或禁止演绎条款。这个不对称说明，一个 IAEA TC 项目最有价值、最持久的产出，可能是把这三类数据以宽松许可公开，而不是 App 本身。前提是阶段 3 的知情同意从第一天就写进数据共享条款。

MVP 的定义也由此清楚了：第一个可交付版本应当是一条不含 AI 的确定性链路，即"读数 → 半衰期 → 半解析剂量 → 规则 → 模板"。代理模型和 LLM 各自要在 G1、G2 用数据证明自己优于这条基线，才能进入产品路径。按这个顺序推进，阶段 1 的三模型对比无论结果如何都有价值：如果半解析模型已经够用，产品更简单、更容易审计；如果代理模型确实更好，它的优势也就有了可以发表的证据。
