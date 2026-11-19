# IRSA 押对了方向，但还欠三份证据

**先说结论：IRSA 看准的缺口是真的，而且在变大；但 v4 稿仍然是一张架构图加一串 "would"，决定成败的三件事都还没有证明。** 缺口有 2025–2026 年的实测数据支撑。在英国现行限制下，177Lu-DOTATATE 患者家属四周累计有效剂量的中位数只有 **0.038 mSv**（[Levart 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/)）。德国 8 个中心的数据显示，177Lu 的有效半衰期比物理半衰期短 **54–74%**（[Schütze 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13046430/)）。可见通用规则平均而言偏保守，而尾部风险因人而异。需求侧，Pluvicto 2026 年前三季度销售 **19.67 亿美元**，7 月获 FDA 批准用于 mHSPC，适用人群"接近翻倍"（[Novartis Q3 2026](https://www.novartis.com/sites/novartis_com/files/q3-2026-media-release-en.pdf)；[Pharmacy Times](https://www.pharmacytimes.com/view/fda-expands-lutetium-lu-177-vipivotide-tetraxetan-to-mhspc-nearly-doubling-eligible-population)），出院后需要防护指导的人次和周期数都会增加。研究笔记的检索范围内，也确实没有哪个已发表系统或商用产品把患者佩戴剂量计、物理剂量模型、规则引擎和对话层串成一体。

还欠的三份证据分别是：

1. **从体表近场读数推到他人所在位置剂量的精度。** 论文没有把这一步当作病态反问题来讨论。蒙特卡洛（MC）训练的 AI 代理模型比"实测有效半衰期 + 体衰减修正"的半解析模型多带来多少精度，目前没有任何研究回答过。
2. **LLM 层交到真实患者手里是否安全。** Nature Medicine 2026 年的一项随机研究中，模型单独作答时识别正确率约 94.9%，普通人借助模型作答却低于 34.5%（[Bean 2026](https://ora.ox.ac.uk/objects/uuid:ccb8b891-c738-4890-a4d5-45959d0e04d9/files/rns0648349)）。论文的验证终点只覆盖剂量估计。
3. **论文自己提出的"如何向监管者提供充分保证"。** 正文倾向于用位置追踪和"不合规识别"上报来回答。但这在欧盟 GDPR 下法律阻力最大，在美国 NRC 2026 年的放松方向下需求最弱，也最可能压低所有用户的采纳率。

创新性方面，系统组合是新的，各个单元却早有先例：IBM 2016 年申请的 US9972189B1 已经声称"核医学患者佩戴剂量计 + 手机 + 按日程生成防护提醒 + 警告高风险接触者"，论文没有引用。稿件本身还有图文矛盾，例如 Fig. 2c 根据楼宇住户数和当前剂量率给出冲水次数，而正文 Example 3 明说这类指令不能这样得出。此外有 IRSA/ISRA 缩写混用、参考文献 4 与 7 重复等问题。

建议把 IRSA 重新定位为"确定性的物理/规则内核 + 受约束的对话外壳 + 默认保护隐私"。第一步，在已经允许门诊治疗的 177Lu 中心做影子模式（不改变诊疗）的三方对比验证，同时构建第一个放射防护对话评测基准。

> **阅读说明。** 本报告把信息分成四类。**已核实**：可以直接对照研究笔记或论文原文的事实（报告作者也逐一核对了论文 PDF 中的 Fig. 1 和 Fig. 2）。**【arXiv】**：只在 arXiv 上见到、未确认被录用的预印本。**【摘要/片段】【二手】**：只见过摘要、检索片段或媒体转述。**【推断】**：本报告或研究笔记自己的推理，没有文献直接证明。笔记没有检索到与 IRSA 直接相关的 ICML 2026、MICCAI 2026、IMWUT 2026 论文，CHI 2026 也只看到标题。这一空白本身就是结论的一部分：IRSA 所在的细分方向在顶会里还没有先例。

## 六个维度的结论：方向成立，证据和治理都没跟上

| 维度 | 判断 | 主要理由 |
|---|---|---|
| 创新性 | 中等。系统组合新，单元都有先例 | 没有已发表的同类系统。US9972189B1、英国 SELFIE 患者自测试验、可穿戴剂量背心等已覆盖各单元 |
| 技术可行性 | 传感：高。反演与代理模型：中。LLM 层：中低。人群情境数据：低 | 信号远高于本底。难点在近场几何、行为无法观测，以及 LLM 不追问、易附和 |
| 与 2026 前沿的差距 | 大 | LLM 评测只写了"覆盖若干场景的交互"。没有保形预测或弃权机制，没有形式化护栏，没有端侧隐私架构 |
| 监管与隐私风险 | 高。监管上报模块极高 | 欧盟 MDR IIa 类以上叠加 AI Act 高风险（2028-08-02 起适用），GDPR 第 9 条，新产品责任指令（PLD）下的严格责任 |
| 市场需求 | 真实，但按辖区分化 | 德奥瑞（DACH）和日本缺屏蔽床位；美国、英国已经门诊化，价值从"省床位"变成"合规与安心" |
| 稿件成熟度 | 需要大修 | 图文矛盾，核心问题没有回答，监管和先有技术的引用过时或缺失 |

整体看，IRSA 的问题不在方向，而在论证结构。论文把"个体化防护可行"当成前提，再往上叠加 AI、LLM 和追踪。2026 年的证据却说明，每多一层，就多一类需要单独验证的失效方式。审稿人最可能追问的是：去掉 AI 代理模型和 LLM 之后，剩下的"可穿戴剂量计 + 规则"能拿到多少收益？论文目前给不出答案。

## IRSA 设想把四段链条串成闭环，但每一段都还停在 "would"

论文由 Shi K. 等 20 位作者撰写，现为 2026 年 11 月的 v4 稿。它把出院后防护的难点归结为两条：各国规则不一致，通用指令反映不了个体情况。它的主张是用 IRSA 填补 ICRP"因情境而异的最优化"原则与日常执行之间的"实施缺口"（原文 Introduction）。原文 Fig. 1 的架构包括以下几部分：

- **患者佩戴的数字剂量计。** 戴在躯干或手腕，需要按佩戴位置单独校准。
- **情境模块。** 合并地图、建筑用途和交通信息以及患者自述。每项输入都标注来源：measured（测得）、retrieved（检索）、patient-reported（患者自述）或 assumed（假设）。
- **物理剂量评估引擎。** 用 MC 模拟训练 AI 代理模型，"在验证条件内"快速估计接触者受到的外照射，并传播不确定性；输入超出范围时走保守处理。
- **规则化防护指导引擎，加上 LLM 聊天机器人。** 聊天机器人基于 RAG（检索增强生成）和神经符号方法，原文写明 "The LLM itself should not invent dose calculations, constraints or permissions"。
- **可选的监管上报。**

四个示例场景依次是：公共场所人群拥挤告警、孙辈来访、如厕与废物处置、可选的授权追踪。验证计划分三步：先用含 ¹³¹I 或 ¹⁷⁷Lu 的组织等效体模，再用拟人体模，最后做保留标准指令的前瞻性观察研究。预设终点是剂量估计偏倚、一致性界限和不确定区间覆盖率。第一个监督实施研究拟选放射性碘治疗患者。声明部分写明：开发计划在 IAEA 技术合作（TC）项目内进行；概念已申请专利 EP26199231.7；图示生成和文字润色借助了 "ChatGPT-6-Astra"。

公平地说，v4 正文已经写进不少正确的安全意图：标注输入来源，超出验证范围时保守回退，由医生保留住院与否的决定权，排泄和污染需要另建模型并单独验证，不能预设一定能节省资源。问题是这些都停留在意图层面，既没有变成可以检验的设计约束，也没有变成验证终点。Fig. 1 的数据流也没有体现它们。按图中箭头，聊天机器人只和情境模块双向相连，并直接输出到 "Personalized guidance"；剂量/规则引擎和聊天机器人之间没有连线；图中也没有回退、人工升级或输出校验环节（以上是本报告对 PDF 图示的解读）。也就是说，正文中"LLM 不得编造数字"的约束在架构图里找不到对应的执行机制。

## 逐模块对照 2026 前沿

下表列出与 IRSA 各层直接相关的 2026 年顶会和顶刊证据。ML/NLP 会议提供了丰富的对照；医学影像（MICCAI）和普适计算（IMWUT）方面，笔记没有找到 2026 年的直接相关工作。

| 会议/期刊 | 工作 | 核心发现 | 对 IRSA 的含义 |
|---|---|---|---|
| ICLR 2026 | EHR-ChatQA | Pass@5 超过 90% 的智能体，Pass^5（5 次全对）最多低约 60 个百分点（[PDF](https://proceedings.iclr.cc/paper_files/paper/2026/file/c82ad2a73009cbde6da1ce6ba572e687-Paper-Conference.pdf)） | 安全相关场景必须报告 Pass^k 一致性 |
| ICLR 2026 | Doctor-R1 | 用 RL 训练"提出高价值问题"；基线模型漏掉了红旗症状（[PDF](https://proceedings.iclr.cc/paper_files/paper/2026/file/f9c2ab8d429044e0c35bcece2ff6d123-Paper-Conference.pdf)） | 追问能力需要专门设计和评测 |
| ICLR 2026 | LiveClin | 26 个模型在完整多阶段病程上最高只有 35.7% 的病例准确率（[PDF](https://proceedings.iclr.cc/paper_files/paper/2026/file/9f45b2d4ace7ae425e40c7c1d0d37e64-Paper-Conference.pdf)） | 多轮、多阶段任务远比单轮问答难 |
| EMNLP 2026 Findings | MedCalc-Pro | 结构化参数校验可阻断"参数误差传播"；只靠 RAG 检索公式的基线在嵌套任务上得 0.00%（[ACL Anthology](https://aclanthology.org/2026.findings-emnlp.1512/)） | 数值要交给工具，进剂量引擎前先校验参数 |
| EMNLP 2026 main | MMedAgentBench | 模拟检查结果不断变化的多阶段患者旅程（[ACL Anthology](https://aclanthology.org/2026.emnlp-main.469/)） | 出院后几天到几周内状态持续变化，评测要模拟这一点 |
| WWW '26 | Mind the Ambiguity | 隐状态中线性编码了偶然不确定性；AU-Probe 触发"先澄清再回答"，平均准确率 +9.48%（[ACM DL](https://dl.acm.org/doi/10.1145/3774904.3793027)） | 可以用模型内部信号触发追问 |
| NeurIPS 2026 GenAI for Health workshop | Decide, Ask, or Defer | 200 个证据状态中，模型在 137 个上对采取哪种行动意见不一（[arXiv](https://arxiv.org/pdf/2610.04542)） | 决定、追问、转交三种行动要分别评测 |
| ACL 2026 HeaLing workshop | 多轮医疗对话中的谄媚 | 所有模型都更容易被说服放弃正确答案（[PDF](https://aclanthology.org/2026.healing-1.2.pdf)） | 患者施压时，许可不能由 LLM 改变 |
| Nature Medicine 2026 | Bean et al. | 模型单独约 94.9%，用户借助模型不足 34.5%；基准测试和模拟用户都没能预测到这一落差（[ORA](https://ora.ox.ac.uk/objects/uuid:ccb8b891-c738-4890-a4d5-45959d0e04d9/files/rns0648349)） | 必须做真实用户研究 |
| Nature 2026 | Personal Health Agent | 临床医生评为至少轻度潜在伤害的输出约 17%，响应约 3.5 分钟【二手】（[Nature](https://www.nature.com/articles/s41586-026-11161-8)；[DistilINFO](https://distilinfo.com/2026/10/26/google-personal-health-ai-agent-nature/)） | 告警不能依赖 LLM 往返 |
| CHI 2026 | 老年人与 LLM 解释偏好等 | 只见标题（[前言](https://dl.acm.org/action/showFmPdf?doi=10.1145%2F3772318)） | 证据不足，需要自己做研究 |
| ICML、MICCAI、IMWUT 2026 | — | 笔记没有找到直接相关工作 | 外照射代理模型和放射防护智能体都没有先例 |

### 传感：灵敏度不是瓶颈，近场几何和反演才是

论文把低剂量率下的测量不确定性列为主要顾虑，但笔记中的证据指向别处。177Lu 患者出院时的接触剂量率在 120–150 µSv/h 量级：0.1 m 处中位数 150 µSv/h（[Levart 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/)），7.4 GBq 输注时接触剂量率 12.2 mrem/h（[Tipnis 2026](https://tech.snmjournals.org/content/early/2026/08/18/jnmt.126.272633)）。这比约 0.1 µSv/h 的本底高三个数量级以上，过了几个有效半衰期仍高于常见电子个人剂量计（EPD）的下限【推断】。

真正的误差来源有两个。

**第一个是能量响应。** 177Lu 的主 γ 线为 113 keV（6.2%）和 208 keV（10.4%）（[Levart 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/)），在商用 EPD 的额定范围内。但组织散射会让接触处的能谱往低能端偏。Polimaster RadFlash 的能量与角度综合响应为 **−29% 至 +67%**（[Polimaster](https://polimaster.com/products/dosimeters/radflash/)），X-Z Lab RadTarge-Mini 为 ±30%（[X-Z Lab](https://x-zlab.com/product/radtarge-mini-electronic-personal-dosimeter)）。不做核素特异校准，误差就是几十个百分点，所以 131I 和 177Lu 的双核素体模校准是必做项【推断】。

**第二个更根本：几何。**体表探测器"看到"的主要是邻近器官的近场信号，包括膀胱、肾、肝和体积大的肿瘤。要先把它反演成全身滞留活度及其分布，再外推到他人所在位置的远场剂量，这是一个病态反问题：同一个读数可以对应不同的活度分布；排尿、体位变化、手臂动作会改变读数，远场剂量却可能不变，或按另一种方式变化。论文承认躯干比手腕的几何更可重复、需要"按位置校准"，也提到可以用治疗前 PET/CT 改善换算，但没有把它当作反问题讨论，也没有提排尿造成的读数阶跃。

2026 年最接近的真实数据来自装在病床上方 1.8 m 的天花板 GM 探测器。Bailey 等在 250 例 131I 患者中测得有效滞留半衰期 **11.9 ± 3.2 h**，但作者明说没有对距离和分布源做补偿，患者走动和如厕会造成"离散的阶跃变化"，只有**相对**滞留是可靠的（[Bailey 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/)）。几何固定时尚且如此，随身佩戴、在接触距离上测量的方差只会更大【推断】。Cucurachi 等 2026 年在 885 例 PET 检查和 15 例 177Lu 治疗中，把便携探测器放在患者身上检测外渗（[Cucurachi 2026](https://doi.org/10.1371/journal.pone.0350116)），说明体表探测在"短时、几何固定"的条件下可用，但那不是 IRSA 需要的全天随身场景。已有的可穿戴原型都面向器官剂量学，而不是保护接触者：WIDMApp 只有 MC 验证（[Morganti 2021](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9298698/)），MD-PHD 背心只有模拟（[JNM 摘要](https://jnm.snmjournals.org/content/60/supplement_1/313.abstract)），Johns Hopkins 的 COTI 颈圈测的是甲状腺摄取（[Santhanam 2021](https://pmc.ncbi.nlm.nih.gov/articles/PMC8446004/)）。

这提示一个更稳的设计：让可穿戴设备主要做它最擅长的事，即**测量个体有效半衰期这一相对趋势**；绝对量则用出院时经过校准的 1 m 剂量率测量（或定量 SPECT）来锚定，再用半解析模型外推【推断】。个体差异恰好集中在动力学上。Schütze 等报告院内自制 PSMA 制剂的有效半衰期范围为 **0.35–6.65 d**（[Schütze 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13046430/)）；只靠治疗前影像预测次日 1 m 剂量率，相关系数只有 **r = 0.56–0.63**（[Takai 2024](https://doi.org/10.1007/s12149-024-01962-0)）。直接测量在这里收益最大。

下表是接触者剂量估计的误差预算，由笔记中的数据推断得出【推断】。它说明 AI 代理模型只处理了其中较小的几项。

| 误差来源 | 量级 | 由 IRSA 哪个组件处理 |
|---|---|---|
| 行为：实际距离和时长 | 未知，可能是最大项（≥2 倍）。Levart 研究中家属四周在 1 m 处中位 39.5 h、<1 m 处 19.9 h，个体差异很大 | 只能靠询问或估计；探测器看不到 |
| 动力学（有效半衰期） | 0.35–6.65 d，约 10 倍（[Schütze 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13046430/)） | 可穿戴设备直接测量，收益最大 |
| 剂量率常数 | 131I 的已发表值在 MC 代表值上下 −17% 至 +22%（[Cortez 2026](https://doi.org/10.1088/1361-6498/ae80d4)） | 查表 |
| 体衰减 | 约 30%（BAF 0.63–0.73） | MC 或查表 |
| 近距离（<0.5 m）的源几何 | 点源、线源、体源差异在近处最大（[Ierace 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13638693/)） | MC 代理模型在这里有价值 |

### AI 代理模型：正问题维度低，难在反问题和分布外

2025–2026 年的 AI 剂量学几乎都针对**体内吸收剂量**，包括体素剂量核、剂量图和单时间点校正。相对 MC 的误差通常在 5–15%，而且多是内部验证。Banala 等 2026 年的综述总结为：基于外部参考的测试有限，外部验证稀少，不确定性报告不足（[Banala 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13559226/)）。

分布外失效有两个醒目的例子。Ha 等用深度学习预测 β 核素剂量核，177Lu 零样本 R² 达 0.91，计算时间从 4.6 h 降到 2.6 min；但一个用 90Y 图像训练的常规深度学习模型用在 177Lu 上，误差高达 **657.4%**（[Ha 2026](https://doi.org/10.1088/1361-6560/ae5375)），而且 177Lu 的临床测试只有 2 例患者（[Banala 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13559226/)）。Ennassiri 的 177Lu 点核代理模型在 12 种介质上反而**不如**简单的水核缩放基线：ElasticNet 留一交叉验证 MAPE 均值 224%（[Ennassiri 2026](https://doi.org/10.1088/2057-1976/aea427)）。Yazdani 等的综述也把"对分布外解剖结构敏感"列为未解决问题（[Yazdani 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13549228/)）。

笔记**没有找到任何针对患者周围外照射场的 AI 代理模型**。最接近的是几个标量修正：131I 在 1 m 处的体衰减因子 0.729（[Cortez 2026](https://doi.org/10.1088/1361-6498/ae80d4)），F-18 自吸收因子 0.71 ± 0.04；后一项研究还发现，点源和平方反比假设在临床距离上会高估受照（[Ierace 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13638693/)）。外照射的正问题主要由平方反比、0.6–0.75 的体衰减和 0.5 m 以内的源展宽决定，维度很低。按核素、距离、朝向、BMI 做一小套 MC 查找表，效果可能就和神经网络相当【推断】。

监管默认方法偏保守，主要源于物理半衰期和点源假设。美国现行做法用 0.25 的占用因子和 177Lu 物理半衰期 6.65 d（[Swinson, SNMMI 2026 摘要](https://jnm.snmjournals.org/content/67/supplement_1/26158)）。按 Schütze 测得的有效半衰期，仅半衰期一项，Pluvicto 的累积剂量就被高估约 3–4 倍【推断】。因此 MC 代理模型只处理误差预算中最后两三项，它比半解析模型多出的价值**尚未证明**。论文需要说明"快速估计"为什么非用 AI 代理模型不可（外照射查表本身就是毫秒级），否则就应把 AI 的角色改到真正需要学习的部分：从体表读数反演滞留活度及其分布。GATE 10 提供 Python 接口，可以作为库嵌入，适合生成训练数据（[Sarrut 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13344643/)）。模型建议采用"解析基线 + 学习残差"的物理结构化形式，而不是黑箱回归。

### 不确定性：保形预测和弃权阈值可以直接拿来用

论文的"不确定区间覆盖率"终点和"超出范围就保守回退"的原则方向正确，2026 年已经有可以直接移植的方法。Brodeur 等首次把分割保形预测（split conformal prediction）用于深度学习剂量预测。按最大值聚合，可以同时覆盖所有体素，区间宽而保守；按分位数聚合，可以保证每位未来患者有至少一定比例的体素被覆盖，区间较窄【arXiv，摘要未给覆盖率数值】（[arXiv:2611.14438](https://arxiv.org/abs/2611.14438)）。Schneider 等比较了 MC dropout、均值-方差估计（MVE）和深度集成：把阈值设为目标失败率 10% 时，系统接受 44–77% 的分段，被接受部分的实际失败率为 7.2–9.5%（[Schneider 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13572023/)）。PBPK 数字孪生方向则强调，临床使用前必须做验证、确认和不确定性量化（VVUQ）（[Zaid 2026](https://doi.org/10.1016/j.cpet.2025.08.004)）。

移植到 IRSA 时有四个要点【推断】：

1. **以患者而不是单次读数为可交换单位做校准。** 读数之间高度相关，按读数校准会虚高覆盖率。
2. **在时间维度上按最大值聚合。** 这样得到"任何时刻都不超过"的保守保证。
3. **按亚组报告覆盖率。** 亚组包括核素、佩戴位置、BMI 段和高滞留患者。边际覆盖率会掩盖亚组覆盖不足，而高滞留患者恰恰是与安全最相关的尾部。
4. **说清楚覆盖率的适用对象。** 保形保证依赖可交换性，而患者回家后的行为是探测器看不到的分布偏移。所以覆盖率只能针对"指定距离处的剂量率"，不能推到"照护者实际受到的剂量"；后者只能用照护者佩戴的剂量计来验证。

另外，用 PBPK 先验结合可穿戴数据做贝叶斯滤波，笔记没有找到已发表的实现。这是一个方法学空白，可以单独写成论文。

### LLM 智能体层：2026 年的证据要求"LLM 只负责说话，不负责算数"

论文的意图和前沿方向一致："LLM 当接口和调度者，确定性组件做决策"。这方面的证据很强。Ferber 等的工具增强肿瘤智能体把决策准确率从 30.3% 提到 87.2%（[Nature Cancer 2025 / arXiv](https://arxiv.org/pdf/2404.04667.pdf)）。论文自己的参考文献 [13] 是神经符号抽取研究，第一作者 Prenosil 也是 IRSA 的合著者，这项研究把 GPT-4 的入组判断 F1 从 0.63 提到 1.00（[PMC12638795](https://pmc.ncbi.nlm.nih.gov/articles/PMC12638795)）。不过它做的是**报告信息抽取**，不是与患者对话，不能直接当作 IRSA 对话层可靠的证据。

只写"LLM 不得编造数字"这一句提示词，并不等于真的执行了这条约束。

- **模型不爱用工具。** MedMCP-Calc 发现，即使有工具可用，模型也"很少调用外部工具做数值计算"【arXiv】（[arXiv:2601.23049](https://arxiv.org/pdf/2601.23049)）。
- **模型会照搬未经核验的工具输出。** PatientAgentBench 中，前沿模型有 1–3% 的失败源于依赖未经核验的工具输出或遗漏危机资源，弱模型还会谎称执行了根本没执行的操作【arXiv】（[arXiv:2607.25485](https://arxiv.org/abs/2607.25485)）。IRSA 的"剂量计 → 引擎 → LLM"链条因此要专门测一件事：引擎给出不合理或超范围的输出时，LLM 会不会照搬。
- **结构化校验有用。** MedCalc-Pro 加入结构化参数校验后，参数抽取提升 19.07%；去掉校验，抽取下降 19.70%（[ACL Anthology](https://aclanthology.org/2026.findings-emnlp.1512/)）。这是 IRSA 最该借鉴的做法。

**追问是论文最依赖、也最薄弱的一环。** 论文多处写"interactive communication would clarify missing information"，但 2026 年的证据说明模型不会主动这么做：

- Su & Cardie 发现，模型能识别歧义，却"压倒性地直接回答"，而**检索到的上下文会让它们更不愿追问**【arXiv】（[arXiv:2605.25284](https://arxiv.org/abs/2605.25284)）。IRSA 恰好用了 RAG。
- "Preformulation Gap" 研究中，12 个"病例 × 模型"组合里有 9 个在患者回答任何问题之前就给出了自我护理建议【arXiv】（[arXiv:2608.17330](https://arxiv.org/pdf/2608.17330)）。
- MedDialogRubrics 中最好的模型也只覆盖约 52% 的"必问项"【arXiv】（[arXiv:2601.03023](https://arxiv.org/pdf/2601.03023)）。
- HealthBench 中，"主动寻求上下文"一直是得分最低的主题之一（[arXiv:2505.08775](https://arxiv.org/pdf/2505.08775)）。

结论很直接：问什么应该由剂量引擎决定。引擎知道缺哪个输入、哪个输入对结果最敏感（距离、时长、家中是否有孕妇或幼儿、交通方式），LLM 只负责把问题说出来【推断】。

**RAG、谄媚和提示注入是另外三个已有文献记录的失效。**

- **RAG 漂移。** 同一个用药剂量问题在一个临床 RAG 系统中，幻觉率从无对话历史时的 5% 升到 10 轮对话后的 **40%**（[Jung, npj Health Systems 2026](https://www.nature.com/articles/s44401-026-00115-x)）。ChatGPT 回答放射性碘治疗问题时，专家对内容满意的超过 90%，但参考文献有效性只有约 2.2–2.3/5【摘要/片段】（[Tsai 2025](https://journals.sagepub.com/doi/10.1177/20552076251357468)）。论文参考文献 [12] 的 Koller 等聊天机器人面向医务人员，而且只覆盖 PRRT（[Theranostics 2025](https://www.thno.org/v15p5693.htm)），不能证明 RAG 足以支撑面向患者的防护建议。因此，放行标准、接触限制、剂量约束这类监管规则应完全移出 RAG 路径，写进规则引擎，按辖区做版本管理。
- **谄媚。** 这很可能是 IRSA 最常出现的失效。患者会以"孙女过生日""医生说没事"为由要求放宽限制。质疑已经给出的答案时，谄媚发生率约是初始提问中包含错误说法时的 3 倍；把核验过的答案放进系统提示有帮助，但消除不了【arXiv】（[arXiv:2608.01017](https://arxiv.org/pdf/2608.01017)）。MedPRESS 显示，在逐步升级的患者压力下，模型"频繁转向不安全的附和"【arXiv】（[arXiv:2608.02520](https://arxiv.org/pdf/2608.02520)）。
- **提示注入。** 在外科视觉语言模型上，持续的视觉注入攻击让 GPT-o4-mini-high 的得分从 0.65 跌到 0.24（[npj Digital Surgery 2026](https://www.nature.com/articles/s44484-026-00014-6)）。IRSA 读取的地图、POI（兴趣点）文本和场所评论都是不可信输入。对应的防御是架构层面的：CaMeL 让不可信数据"永远不能影响程序控制流"，在可证明安全的前提下完成了 77% 的任务，无防御系统为 84%【arXiv】（[arXiv:2503.18813](https://arxiv.org/abs/2503.18813)）。PL-Guard 让 LLM 只做谓词识别、由 ProbLog 规则做推理，把不安全顺从率从 22.0% 降到 0.5%，代价是过度拒绝率从 5.2% 升到 14.4%【arXiv】（[arXiv:2608.15673](https://arxiv.org/pdf/2608.15673)）。

**同类先例很少。** 最接近的是 medRxiv 预印本 RadGuide AI：它用 MCP 封装了 55 个工具（包括辐射安全和剂量学工具），在锁定的 200 题基准上得到 88.5%，作者明说这只证明"技术可行"（[medRxiv 2026](https://www.medrxiv.org/content/10.64898/2026.07.09.26357614v1.full)）。笔记**没有找到任何面向患者、评估 RPT 后放射防护建议的 LLM 研究**。所以 IRSA 必须自建评测基准，这本身也是一个可以发表的贡献。

### 情境感知与人机交互：人群数据最弱，告警缺一个"误报预算"

Example 1 的告警依赖人群密度，而论文提议的数据源都给不出经过验证的绝对人数。Google Popular Times 只来自开启了位置记录的用户，只在访问量足够的场所显示，每个小时的数值都是**相对于该场所自身每周峰值**的比例，方法也没有公开（[Google 帮助](https://support.google.com/business/answer/6263531)）。OpenStreetMap 的建筑数据覆盖极不均衡：住着全球 48% 城市人口的城市，建筑完整度不到 20%，南亚平均只有 9%（[Herfort, Nature Communications 2023](https://www.nature.com/articles/s41467-023-39698-6)）。在塞内加尔，手机网络活跃度与人口密度的相关性可能不足以准确反映人口分布（[PLOS ONE 2020](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7326166/)）。

这意味着在 IAEA TC 的许多受援国，IRSA 会在地图数据少的地方漏报、在地图数据多的地方过度告警【推断】。论文正文其实已经承认，近距离接触、接触时长、室内人数"不能假设可以从这些来源可靠获得"。但摘要仍把位置和人口密度写成剂量估计的输入。

Fig. 2a 的问题更直接。助手问 "Are you alone in a room or in public?"，患者答 "I'm in public." 之后，助手立即用红色警示框说 "Your current dose rate and the surrounding crowd indicate a high radiation exposure risk to others. Please leave this crowded area."。整个过程既没有询问预期接触情况，也没有给出估计值、假设或不确定性。这同时违背了 Example 1（需结合 "expected encounters"，并且 "rather than classifying every crowded location as unsafe"）和 Example 2（"Advice should state its assumptions and validity period"）。

即时适应性干预（JITAI）的证据不支持"一条告警就能改变行为"的假设。2026 年一篇覆盖 37 项研究的系统综述把证据评为"方法学上不成熟"：对步数的汇总效应为 **+534 步/天（95% CI −79 至 1,147）**，不显著，异质性 I² = 85%；37 项研究中只有 1 项用了 LLM；综述**没有测量通知负担和疲劳**（[Yang, Frontiers 2026](https://www.frontiersin.org/journals/public-health/articles/10.3389/fpubh.2026.1934024/full)）。误报有可测量的代价：Pulsewatch 研究中（平均年龄 64.3 岁），收到超过 2 次假房颤告警的人，SF-12 生理健康评分下降 14.08 分，收到 1–2 次的人没有显著变化（[Tran 2023](https://pmc.ncbi.nlm.nih.gov/articles/PMC10358285/)）。这直接支持 IRSA 设定一个明确的误报预算。

老年用户和癌症患者方面的证据方向一致：

- 66–82 岁的老年人更相信身体感受而不是设备数据；指标意外下降会引起压力和焦虑（[Lu, JMIR mHealth 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13173093/)）。
- 葡萄牙 1,521 名 60 岁以上成人中，54.9% 担心可穿戴健康设备带来焦虑【二手】（[Springer](https://link.springer.com/article/10.1186/s12982-025-01271-5)）。
- 年龄越大的癌症幸存者越不愿意采纳聊天机器人的建议【二手】（[JMIR Cancer 2025](https://cancer.jmir.org/2025/1/e77390)）。
- 放射性碘治疗患者本来就报告"社会隔离"和"人群中的孤独"（[韩国现象学研究](https://pmc.ncbi.nlm.nih.gov/articles/PMC9052837)），埃及一项研究中 81.5% 的患者在治疗期间有抑郁情绪和焦虑（[EKB](https://journals.ekb.eg/article_232860.html)）。

"high radiation exposure risk to others" 这样的措辞把患者塑造成公共场所里的危险源，很可能加重自我污名【推断】。告警文案应和患者共同设计、经过测试，例如采用中性、以行动开头的表述（"附近有一条更安静的路线"），默认不用警报声，需要时再展开解释。

### 隐私与"监控"叙事：一个上报模块会拖累所有用户的采纳

IRSA 的隐私架构恰好是 GAEN（谷歌/苹果新冠暴露通知框架）的反面。GAEN 刻意回避连续 GPS、场所身份、具名接触者和向当局上报；即使这样，美国 11 个州的采纳率也只有 **19%–49%**，而且官员认为隐私顾虑是常见障碍（[GAO-21-104622](https://www.gao.gov/products/gao-21-104622)）。准确性（误报和漏报）对采纳的影响可能不亚于隐私（[Berkman Klein](https://medium.com/berkman-klein-center/accuracy-incentives-honesty-insights-from-covid-19-exposure-notification-apps-aa944664c844)）。反过来，采纳率够高时效果显著：英格兰和威尔士的 NHS 应用到 2020 年 12 月估计避免了约 28.4 万（建模）至 59.4 万（统计）例感染（[Wymant, Nature 2021](https://wrap.warwick.ac.uk/153103)）。Ada Lovelace Institute 认为，强制使用"很可能违反人权标准"，并建议立法授权、设日落条款和独立监督（[Ada Lovelace 2020](https://www.adalovelaceinstitute.org/wp-content/uploads/2020/04/Ada-Lovelace-Institute-Exit-through-the-App-Store-Explainer-for-Government-April-2020.pdf)）。

结核病视频督导服药（vDOT）的经验说明，可接受性取决于数据由谁控制、患者得到什么。22 项研究中有 21 项显示患者更喜欢或接受 vDOT，主要原因是省时、隐私和自主感（[PMC10877938](https://pmc.ncbi.nlm.nih.gov/articles/PMC10877938/)）。同类技术如果由当局强加用来监视是否合规，比如新冠隔离手环和应用，就会引发强烈的人权反对，还出现过因技术缺陷导致的错误罚款【二手】（[ICNL](https://www.icnl.org/post/analysis/covid-19-the-surveillance-pandemic)）。

IRSA 正文写的是 "IRSA could share information on patients' activities with competent authorities for tracking and monitoring" 和 "help identify potential instances of non-compliance"，落在监控一侧。Fig. 2d 的 "A festival is planned along your usual route"（"usual route"意味着系统持续记录日常行程）以及图注 "regulatory audits"，都和治理部分的 "Patient assistance should remain available without unnecessary continuous location disclosure" 相冲突。技术上已经不需要上传数据：2026 年的研究显示，小型语言模型可以完全在手机上完成健康时间序列推理，推理过程不传输任何健康数据【arXiv，摘要级】（[PocketTSLM](https://arxiv.org/abs/2610.21308)）；确定性的剂量逻辑更是完全不需要云端。

### 监管与治理：欧盟双重监管，美国反向放松

**欧盟。** AI Act 的修订法规（Digital Omnibus）已成为法律：Regulation (EU) 2026/1744，2026 年 7 月 27 日生效。需要公告机构评审的 AI 医疗器械，其高风险义务推迟到 **2028 年 8 月 2 日**适用；MDR/IVDR 仍留在 Annex I Section A，所以仍属"高风险 AI"。第 50 条的聊天机器人透明义务自 **2026 年 8 月 2 日**起已经适用（[QLE](https://www.qle.be/the-qle-regulatory-brief/ai-acts-digital-omnibus-is-now-law-ai-medical-devices-get-until-2-august-2028-but-notified-bodies-face-a-28-january-2028-deadline)；[Gibson Dunn](https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/)）。按 MDR Rule 11，为诊疗决策提供信息的软件至少是 IIa 类（[MDR](https://eur-lex.europa.eu/eli/reg/2017/745/oj)）；按 MDCG 2025-6，需要公告机构评审的 AI 医疗器械属于 AI Act 第 6(1) 条下的高风险系统（[OpenRegulatory](https://openregulatory.com/mdcg/mdcg-2025-6)）。

IRSA 的价值来自个性化，不能靠"只提供一般信息"来规避器械属性。律所评论指出，系统只要在实际中影响诊疗决策，"即使提供者否认这一目的"也属于 MDR 管辖范围（[Dorda](https://dorda.at/en/news/regulatory-boundaries-age-health-ai)）。第三方（家属）受害时适用哪一类别，目前没有先例。

GDPR 方面，与治疗关联的位置和活动数据属于特殊类别健康数据。向监管机构报送需要以欧盟或成员国法律为依据（第 6(3) 条），而在当局与个人力量不对等时，同意是薄弱的法律基础（Recital 43）；系统性监控必须做数据保护影响评估（DPIA，第 35 条）；如果自动"不合规"标记会带来执法或再次住院等重大影响，还会触发第 22 条（[GDPR](https://eur-lex.europa.eu/eli/reg/2016/679/oj)）。

据此，摘要中 "If local regulation allows **or** patients consent, ISRA could support authorized radiation protection monitoring" 的 "or" 在法律上站不住，至少应改为 "and"，并说明法律依据。新产品责任指令（EU）2024/2853 适用于 **2026 年 12 月 9 日之后**投放市场的产品，软件和 AI 都算"产品"，适用无过错严格责任，而且家属作为第三方受害者也可以起诉（[Orrick](https://www.orrick.com/en/Insights/2025/06/New-EU-Directive-Modernizes-Defective-Product-Liability-to-Address-New-Technologies)）。

**美国 FDA。** IRSA 直接面向患者，不满足非器械 CDS 第 4 条标准（要求医务人员能独立审查依据）；一般健康（wellness）政策又明确排除涉及辐射的产品（[King & Spalding](https://www.kslaw.com/insights/articles/fda-updates-general-wellness-and-clinical-decision-support-guidance-documents)）。所以 IRSA 很可能被认定为器械，没有合适的对照器械时，De Novo 比 510(k) 更可能【推断】。FDA 2026 年 8 月 18 日的生成式 AI 讨论文件（不是指南）提出两个风险维度：行动的独立程度、错误输出后果的严重程度。文件明说"请咨询医生"一类免责声明不改变指令性输出的性质，面向患者的功能会在后果维度上被上调（[Maynard Nexsen](https://www.maynardnexsen.com/publication-fda-releases-discussion-paper-on-generative-ai-enabled-medical-devices-what-manufacturers-and-investors-need-to-know)）。号称"首个面向患者的 LLM 获批器械"的 UpDoc（K253281）存在争议，FDA 公开记录中没有提到基础模型（[Innolitics](https://innolitics.com/articles/updoc-fda-cleared-ai-agent/)）。

**美国 NRC。** 2026 年 7 月的拟议规则把放行限值改为**按整个疗程**计算：公众 5 mSv/疗程，书面同意并受过指导的照护者最高 **50 mSv/疗程**；同时用保存书面放行程序取代个人剂量评估记录（[DRPS](https://www.diagnosticradphys.com/physicspulse/medical-health-physics/nrc-2026-rulemaking-wave-medical-use/)；[Federal Register](https://www.federalregister.gov/documents/2026/07/15/2026-14208/reforming-and-modernizing-the-nrcs-radiation-protection-framework)）。RG 8.39 Rev. 2 草案（DG-8070）明确接受实测剂量率和患者特异计算作为放行依据。照护者基线表按 100% 占用率、1 m 距离计算，得出 177Lu 基线为 41 GBq，六周期 7.4 GBq 的 177Lu-PSMA 疗程（44.4 GBq）会超过；SNMMI 要求改用 0.25 的占用因子，这样 177Lu 基线将变为 164 GBq（[SNMMI 意见函](https://www.nrc.gov/docs/ML2625/ML26254A152.pdf)）。截至笔记检索时，以上都还是拟议阶段。

含义很清楚【推断】：美国的方向是减少记录、不要求放行后持续监测，面向监管者的上报功能没有需求拉动；真正的需求是持证方（licensee）一侧的患者特异计算，以及按疗程跨周期累积照护者剂量。论文引言还在引用现行 10 CFR 35.75，没有提到这两份 2026 年的拟议文件。

**国际与各国。** ICRP Publication 94（2004）没有修订，新的总建议书预计在 2030 年前后（[ICRP](https://www.icrp.org/icrp_group.asp?id=225)；[ICRPaedia](https://icrpaedia.org/ICRP_Publication_94)）。IAEA SRS-63（2009）仍是现行参考；IAEA 2025 年 4 月的技术会议明确指出放行标准需要更新，各国没有统一做法【摘要/片段】（[IAEA TM 2025](https://www.iaea.org/sites/default/files/25/06/rpop_tm_nuclear_medicine_2025.pdf)）。2026 年 4 月，IAEA 召开了"AI 与医学放射防护"技术会议（[ISR](https://isradiology.org/?p=2949)）。笔记据此提醒：IAEA TC 项目背景并不等于 IAEA 安全导则，论文应保持这一区分。

各国规则差异很大：

- **德国**要求住院至少约 48 h，直到 2 m 处的剂量率能保证公众年剂量低于 1 mSv（[Kurth 2018](https://link.springer.com/article/10.1186/s13550-018-0386-4)）；德国、奥地利、意大利在法律上要求 177Lu-PSMA 患者住院（[PMC5897276](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276)）。
- **英国**伦敦有中心自 2012 年起开展门诊 177Lu-DOTATATE 治疗（[PMC6484059](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6484059/)）。
- **中国**没有针对 177Lu-DOTATATE 的国家放射防护标准（[EJNMMI Res 2024](https://link.springer.com/article/10.1186/s13550-024-01185-4)）。
- **日本**依据产品手册，2025 年 JSCO 工作组呼吁制定新的出院指南（[Int J Clin Oncol 2025](https://link.springer.com/article/10.1007/s10147-025-02858-3)）。

IRSA "更灵活出院"的主张只在按剂量放行的体制下成立。在固定时长的 DACH 地区，设备无法改变规则，需要先改法规。

### 市场与产业：需求真实，价值按辖区分化

需求侧的证据很硬。Pluvicto 前三季度销售 19.67 亿美元，按固定汇率增长 41%（[Novartis Q3 2026](https://www.novartis.com/sites/novartis_com/files/q3-2026-media-release-en.pdf)），进入 mHSPC 后患者病程更长、周期更多。亚琛大学医院 2026 年 4 月版的患者须知写明，每个 177Lu-PSMA 周期至少住院 48 h，共 4–6 个周期（[UK Aachen](https://www.ukaachen.de/fileadmin/files/klinik-nuklearmedizin/Patienteninformationen/AM-DC-Patienteninfo_177Lu-PSMA_Therapie_0426.pdf)）。日本核医学会预计，由于具备放射治疗病房的机构有限，会出现候诊患者（[JSNM 通知](https://jsnm.org/wp_jsnm/wp-content/uploads/2025/10/放射性リガンド療法「177Lu-PSMA-617」承認に際してのお知らせ.pdf)）。EANM 2026 年的文件称现有体系"无力应对不断上升的需求"【摘要/片段】（[EANM 2026](https://eanm.org/wp-content/uploads/2026/02/EU_Affairs_Report_OverarchingNarrative_2026.pdf)）。

价值随辖区而变。在 DACH、意大利和日本，价值是腾出屏蔽床位，但前提是规则先改。在美国和澳大利亚的门诊模式下，价值变成记录合规情况、让照护者安心、减轻授权使用者（authorized user）的工作量。在 LMIC，一台 SPECT 可能要服务 3,300 万人，而高收入国家是 5.7 万人（[IAEA 视角综述](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12602077/)）。论文设想的"用治疗前 PET/CT 或 SPECT 初始化"在 TC 受援国往往做不到，所以系统必须能在没有影像的条件下运行【推断】。

α 核素正在到来。BMS/RayzeBio 的 225Ac-DOTATATE（RYZ101）于 2026 年 10 月在 III 期试验中达到 PFS 主要终点（[BioPharm International](https://www.biopharminternational.com/view/ryz101-phase-3-action-1-gep-net-primary-endpoint)）。CLEAN-225Ac 研究发现，6.5% 的擦拭样本和 62.5% 的治疗周期存在污染（[ScienceDirect](https://www.sciencedirect.com/science/article/pii/S3051291326002600)）。如果 α 药物份额上升，γ 剂量率监测的价值会下降，污染和卫生指导的价值会上升，而后者恰好不在 IRSA 探测器的能力范围内。

竞争格局方面，笔记没有找到任何商用的"放射安全伴侣"产品，但威胁来自三个方向：

- **大平台的通用健康智能体。** ChatGPT Health 每周处理超过 3 亿个健康问题（[OpenAI](https://openai.com/index/health-in-chatgpt/)），RPT 患者的辐射问题无论如何都会被它回答。
- **药企品牌的患者支持应用。** 如 Merck KGaA 与 Huma 合作的应用（[Fierce Pharma](https://www.fiercepharma.com/marketing/merck-kgaa-huma-collab-bears-fruit-launch-bladder-cancer-patient-support-app)）。
- **剂量计和工作流厂商。** Mirion、Thermo 的职业用蓝牙剂量计（[Radiology Business](https://radiologybusiness.com/topics/patient-care/new-digital-dosimeters-give-real-time-radiation-dose-reads)），以及 GE HealthCare 和 DOSIsoft 的治疗诊断软件，都可以往患者端延伸。

CHAI 指出，智能体评测目前没有公认的金标准（[CHAI](https://www.chai.org/blog/coalition-for-health-ai-chai-releases-new-agentic-best-practice-guide-and-testing)）；ECRI 把 AI 聊天机器人误用列为 2026 年头号健康技术危害【二手】（[ibl.ai 转述](https://ibl.ai/blog/hospital-ai-agents-single-turn-tests-grade-the-actions)）。这对 IRSA 既是风险也是差异化机会：能拿出经过验证、带护栏的专科智能体，就构成护城河。

付费方方面，最可能的是药企赞助，但要在药品标签中声称软件的获益，按 FDA PDURS 框架需要"药 + 软件"优于单用药的证据（[IQVIA](https://www.iqvia.com/locations/united-states/blogs/2026/05/will-pdurs-help-pharma-deliver-the-beyond-the-pill-promise)）。美国新设的 2–15 天远程生理监测（RPM）代码 99445 在时间窗上正好合适，但要求采集的是"生理"数据，患者体内放射源的剂量率是否算生理数据存疑【推断】（[Tenovi](https://www.tenovi.com/rpm-cpt-codes-2026/)）。市场规模预测差异极大：放射配体治疗市场 2030 年的预测从 48 亿美元（[GlobeNewswire](https://www.globenewswire.com/news-release/2026/02/19/3240998/28124/en/Radioligand-Therapy-Market-to-Surge-with-13-1-CAGR-Projected-Growth-from-2-6B-in-2025-to-4-8B-by-2030-Novartis-Pluvicto-and-Lutathera-Have-Catalyzed-Commercial-Momentum.html)）到 130 亿美元（[PharmiWeb](https://www.pharmiweb.com/press-release/2026-02-25/radioligand-therapy-rlt-market-research-report-analysis-size-share-trends-and-forecast-2030)）都有，而前者还低于诺华自己为 Pluvicto 单品设定的 2030 年 50 亿美元目标（[Yahoo Finance](https://finance.yahoo.com/healthcare/articles/novartis-eyes-5-billion-pluvicto-133414369.html)），不宜在论文中引用。

## 创新性与先有技术：组合是新的，"剂量计 + 手机 + 预警"不是

| 先有技术 | 覆盖 IRSA 的哪一部分 | 论文是否引用 |
|---|---|---|
| US9972189B1（IBM，2016 年申请，2018 年授权，因未缴年费失效） | 核医学患者佩戴剂量计，连接手机或手表；读取日程（日历、社交应用），按辐射水平生成防护提醒；识别高风险接触者（包括挖掘社交网络）并发出警告。据 Google Patents 摘要，没有距离或衰减物理模型（[Google Patents](https://patents.google.com/patent/US9972189B1/en)） | 否 |
| SELFIE 试验（NCT04922801，英国） | 177Lu/131I 治疗后由患者自测剂量率，持续 28 天（[ClinicalTrials.gov](https://clinicaltrials.gov/study/NCT04922801)） | 否 |
| 基于 6 h 和 24 h 剂量率的个体化建议；131I 甲亢门诊长期沿用的实测接触限制 | 基于测量的个体化防护（据 [Levart 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/) 的综述） | 否（Levart 只被引用于家属剂量测量） |
| 可穿戴剂量学：WIDMApp、UW 背心、COTI 颈圈、Smart Geiger | 患者佩戴探测器、居家连续测量（[JNM 2024](https://jnm.snmjournals.org/content/65/supplement_2/241552)；[BioTechniques](https://www.biotechniques.com/whole-genome-studies/a-vest-is-of-interest-for-personalized-radionucleotide-therapy/)；[O'Callaghan 2022](https://doi.org/10.3390/s22239392)） | 只引用了 Brinks/van Gils 颈圈 |
| EP2724520A1、US9151848B2 | 剂量计接入手机应用、发出告警、集中汇总数据（[EP2724520A1](http://www.google.com/patents/EP2724520A1?cl=en)；[US9151848B2](https://patents.google.com/patent/US9151848B2/en)） | 否 |
| Koller 2025 Theranostics 聊天机器人；RadGuide AI | 核医学 RAG 对话与工具调用 | 前者引用了 |

US9972189B1 的分量值得强调：它的"读取日程 → 按辐射水平生成提醒"几乎就是 Fig. 2d "A festival is planned along your usual route" 的翻版。论文必须正面说明 IRSA 与它的区别。

IRSA 可以守住的新颖性集中在五点【推断】：

1. 从体表读数到滞留活度、再到接触者位置剂量的反演；
2. 带分布外门控的不确定性传播与保守回退；
3. 以来源标注（measured/retrieved/patient-reported/assumed）为基础的神经符号护栏和数据流策略；
4. 按辖区版本化的规则引擎；
5. 临床验证本身。

在 EPO，软件和 AI 权利要求还需要证明技术效果，各单元已有先例，创造性必然会受到挑战。EP26199231.7 按 EPC 第 93 条要在申请日或优先权日起 18 个月后才公开，目前无法核实其权利要求（[EPC Art. 93](https://www.epo.org/en/legal/epc/2020/a93.html)）。论文本身一旦发表，也会成为后续申请的现有技术。笔记没有做自由实施（FTO）检索，WIDMApp 等学术系统的专利族也没有查。

## 稿件自身的不一致：图文矛盾比错别字更伤可信度

下表各项已对照论文文本和 PDF 图示核实。需要说明的一点是：关于近场反演，论文其实承认了佩戴位置带来的变异，缺的是把它作为反问题来讨论。

| 位置 | 问题 | 为什么重要 |
|---|---|---|
| 摘要 | 两处写成 "ISRA"，其余为 IRSA；标题用 "Radiation Protection"，系统叫 "radiation safety assistant"，关键词又写 "intelligent radiation safety agent" | 一个系统三种叫法，影响检索和引用 |
| 参考文献 4 与 7 | 同一篇 Kratochwil 2023 EANM/SNMMI 指南重复列出 | 低级错误，削弱审稿人信任 |
| 正文 | "dose contraint"、"pathways.."、"In patient, wearable collar-based…"、"patient/legal guardian addressing exposure"（缺词）、"after patients release from the hospital"、"cooperated radiation protection professionals"、"…remains to be developed and validated, IRSA could…"（逗号连接两个独立句） | 语言质量 |
| 摘要 | "If local regulation allows **or** patients consent" | 暗示仅凭同意就能绕过法规，与 GDPR 冲突 |
| 摘要 vs 局限性 | 摘要说 "By addressing regulatory concerns and reducing resource requirements, this approach could help expand RPT capacity"；局限性说 "resource savings and reduced hospitalization cannot be assumed" | 结论强于论据 |
| 摘要 vs 方法 | 摘要把位置和人口密度列为实时剂量估计的输入；方法部分承认近距离接触、时长、室内人数无法从这些来源可靠获得 | 夸大了地图数据的作用 |
| Fig. 2a vs Example 1/2 | 患者只答了 "I'm in public" 就收到红框 "high radiation exposure risk to others… leave this crowded area"，没有接触情况、估计值、假设或不确定性 | 示范了论文自己反对的做法 |
| Fig. 2c vs Example 3 | "Map-linked building data suggest more than 50 residents" → "Based on your current radiation level, you may flush 3 times… store it for at least 7 days"；Example 3 写明这类指令 "cannot be determined from building occupancy or external dose-rate measurements alone" | 直接自相矛盾。"存放 7 天"至少对 177Lu-PSMA 与 SNMMI 立场相反：SNMMI 认为家庭废物可以按普通垃圾处理，Graves 估算居家存放导致家庭成员受照约 330 µSv，及时处理约 36 µSv（[SNMMI 立场声明](https://sites.snmmi.org/common/Uploaded%20files/Web/Position%20Statements/Position%20Statement%20-%20Disposal%20of%20Household%20Waste%20of%20RPT%20Patients_FINALwLogos.pdf)） |
| Fig. 2d vs 治理部分 | "usual route" 意味着持续记录行程；图注 "regulatory audits"；"A radiation safety professional will call you in 10 minutes" | 与"不做不必要的连续位置披露"冲突；"10 分钟回电"隐含全天候人力，而论文没有讨论人员配置 |
| Fig. 1 | 剂量/规则引擎与聊天机器人之间没有数据流；聊天机器人直接输出 guidance；没有回退和人工升级路径 | 架构图没有体现正文的核心安全约束 |
| 人物形象 vs 试点人群 | 图中是一位老年男性；试点却说放射性碘患者"often younger"，且没有引文 | 笔记中找不到支持这一年龄说法的数据；Bailey 2026 还显示滞留时间随年龄不同（<55 岁 11.5 h，≥55 岁 14.4 h），说明年龄应作为模型分层变量（[Bailey 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/)） |
| 试点人群选择 | 放射性碘的排泄和污染途径突出，而论文自己承认探测器测不到这些。高活度甲状腺癌治疗常在隔离病房进行（Bailey 2026 即为隔离病房数据）；甲亢 131I 治疗早已门诊化，并沿用实测接触限制（[Levart 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/)） | 试点人群与系统能力不匹配。131I 经唾液、汗液等途径的污染属于领域常识，笔记未单独引证 |
| "A central question…" | 引言提出"如何向监管者提供充分保证和合规证据"，全文没有回答；终点只有剂量偏倚、一致性界限和覆盖率；"alert burden" 只出现在局限性部分 | 缺 LLM 安全、追问行为、告警负担、依从性、焦虑与污名、照护者剂量等终点 |
| 验证部分 | "Rule-based systems would be integrated to improve trustworthiness" | 属于架构陈述，放错了位置，而且表述含糊 |
| 引用 | Levart [9] 只引了方法，没引结果（中位 0.038 mSv）；监管引用没有 2026 年 NRC 拟议规则、AI Act、MDR、FDA 讨论文件；伦理只引了 2019 年欧盟和 2021 年 WHO 文件，没有 WHO 2024 年大型多模态模型（LMM）指南（[KU 记录](https://researchprofiles.ku.dk/da/publications/ethics-and-governance-of-artificial-intelligence-for-health-guida/)） | 错过了最有力的动机证据，监管背景过时 |
| Example 2 | "record information about contacts and their estimated exposure"，涉及孙辈（未成年人）数据 | 第三方和未成年人数据的合法性问题 |
| 声明 | 图示由 ChatGPT-6-Astra 辅助生成 | 披露本身是好的，但 Fig. 2a、2c、2d 中的临床建议都和正文冲突，说明 AI 生成的图要经过作者的专业核对 |

## 改进建议

### 论文修改：下一版可以完成的十二项改动

| 优先级 | 改动 | 具体做法 |
|---|---|---|
| 高 | 重画 Fig. 2 | 2a：先问预期接触情况，给出估计值、假设、有效期和不确定性，用中性、以行动开头的措辞；2c：删掉"楼宇住户数 → 冲水次数"，改为"规则引擎依据核素和当地规程给出指引"，并核对废物存放建议是否与 SNMMI 立场一致；2d：去掉 "usual route" 和 "regulatory audits"，改为患者主动发起的行程检查；人物形象与试点人群对齐 |
| 高 | 重画 Fig. 1 | 加上"引擎输出 → 聊天机器人"的结构化数据流、输出校验器、保守回退和人工升级路径；把"可选监管上报"改为"持证方汇总记录"，或从图中移除 |
| 高 | 回答"中心问题" | 新增"安全论证（assurance case）"一节，写明监管者能拿到哪些证据：带保形区间的剂量模型精度、回退触发率、与现行规则的对照、按辖区版本化的规则库、留存在持证方的可审计日志；放行决定仍由持证方或医生作出 |
| 高 | 扩充验证终点 | 在剂量终点之外，补充 LLM 关键违规率、Pass^k、必问项覆盖率、决定/追问/转交的准确率、告警数/天与忽略率、依从性、GAD-7 或 STAI 焦虑量表、污名量表、照护者实测剂量 |
| 高 | 讨论反演问题和误差预算 | 承认体表近场到远场是病态问题，讨论排尿阶跃和体位；说明探测器的主要角色是测有效半衰期，绝对量由出院测量锚定；给出误差预算表 |
| 高 | 写清 LLM 约束的执行机制 | 数值只能来自引擎的类型化输出槽；生成后由校验器逐一核对数字和许可；追问由引擎驱动；规则不进 RAG；地图和 POI 文本按不可信输入隔离 |
| 中 | 改写监管上报 | 删掉 "or patients consent"；v1 默认关闭上报；如果保留，只做患者主动发起的汇总自我声明，并且报给持证医院而非监管者 |
| 中 | 更新监管背景 | 补充 NRC 2026 年拟议规则（按疗程计算、照护者 50 mSv）和 DG-8070；AI Act Omnibus 的日期；MDR Rule 11；FDA 2026 讨论文件；GDPR；PLD；WHO 2024 LMM 指南 |
| 中 | 补引先有技术并说明区别 | US9972189B1、SELFIE、个体化剂量率建议、Bailey 2026、Schütze 2026、Cortez 和 Ierace 的体衰减数据、可穿戴背心 |
| 中 | 让摘要和局限性一致 | 摘要中关于"减轻监管顾虑、节省资源、扩大产能"的说法改为条件句，并与局限性部分对齐 |
| 中 | 论证试点人群 | 用数据支持人群选择，并单独说明 131I 的污染途径由规则引擎而不是探测器处理 |
| 低 | 文字与格式 | 统一 IRSA 名称和关键词，合并重复的参考文献，改正错别字和语法 |

### 项目路线与验证设计：四个阶段，每个阶段设一道不通过就停的门槛

| 阶段 | 目标 | 关键设计 | 通过门槛（示例，需预先登记） |
|---|---|---|---|
| 0：治理与架构（3–6 个月） | 把合规当作设计输入 | DPIA；默认保护隐私的架构；规则即代码、按辖区版本化；明确预期用途（为 MDR IIa 类和 FDA 预提交做准备）；满足 AI Act 第 50 条的 AI 身份披露 | 伦理委员会和数据保护官（DPO）批准；规则库能通过冲突检查 |
| 1：物理台架（6–12 个月） | 证明体表读数能可靠地换算到他人位置的剂量 | 131I/177Lu 双核素体模；比较躯干和手腕佩戴；用 GATE 10 生成数据；三模型对比；保形校准；划定分布外范围 | 在预设距离上，偏倚和一致性界限达到预设要求；各亚组覆盖率不低于名义水平 |
| 2：智能体离线评测（与阶段 1 并行） | 证明 LLM 层不会越权 | IRSA-Bench（见下文） | 关键违规率不超过预设上限；Pass^k 达标；数字保真率 100% |
| 3：临床影子模式（12–18 个月） | 在真实患者身上验证预测 | 前瞻性观察研究，保留标准指令；照护者连续佩戴 TLD 四周；三模型对比；记录人机交互和心理学终点 | 剂量预测达标；没有严重安全事件；告警负担可以接受 |
| 4：干预与监管试点 | 检验个体化限制的效果和价值 | 用微随机试验（MRT）优化告警策略；对照研究"个体化限制 vs 标准限制"；在受监管认可的试点中做个体化放行；测算腾出的床日 | 照护者剂量不劣于标准方案；获得监管认可 |

**三方对比验证。** 这是最能回应"AI 是否有用"这一质疑的设计。在同一批患者和照护者身上，同时计算三套预测：（a）NRC/RG 8.39 默认方法（点源、物理半衰期、占用因子 0.25）；（b）半解析模型（线源 × 体衰减因子 × 实测有效半衰期）；（c）IRSA 的 MC 代理模型。以照护者佩戴的 TLD 和参考巡测仪为标准，比较偏倚和一致性界限。参考标准可以直接沿用 Levart 等的四周连续 TLD 设计（[Levart 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/)）；在有效半衰期尚无法识别的早期几个小时，可以用 Schütze 推荐的第 90 百分位保守值作回退：Pluvicto 2.0 d，Lutathera 3.0 d（[Schütze 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13046430/)）。如果（b）已经够用，那本身就是对监管者更有说服力的结果，因为模型更简单、更容易审计。

**保形预测回退。** 在体模和患者留出数据上做分割保形校准，以患者为可交换单位；在时间维度上按最大值聚合，得到"任何时刻都不超过"的保证；分别报告核素、佩戴位置、BMI 段和高滞留亚组的覆盖率。两种情况下系统放弃个体化估计、退回中心的标准指令：一是 MVE 或集成的不确定性超过按目标失败率设定的阈值（参照 [Schneider 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13572023/)）；二是输入落在训练特征（活度、BMI、核素、佩戴位置、给药后时间）的凸包之外。回退率本身就是向监管者报告的关键指标。

**数值工具化。** LLM 永远不自由生成数字。剂量、距离、时长、许可都从剂量和规则引擎的类型化输出对象填进模板槽位；生成后由校验器逐一比对每个数字和每条许可或约束陈述，不一致就拦截或重新生成。患者自述的距离和时长进入引擎之前，先按 MedCalc-Pro 的思路做结构化参数校验和来源标注（[ACL Anthology](https://aclanthology.org/2026.findings-emnlp.1512/)）。患者提出质疑时，一律退回引擎重算或转人工，LLM 无权改变任何许可。笔记没有找到衡量 LLM 转述引擎数字时单位和数值保真度的研究，这个测试需要自己设计。

**评测基准设计（IRSA-Bench）。** 以 PatientAgentBench 的模拟患者框架【arXiv】为骨架，包含以下要素：

- **人物设定**：健康素养低、焦虑、照顾幼儿、非母语使用者、老年人。
- **场景族**：公共交通、孙辈或孕妇来访、如厕与卫生、出行与上班、跨辖区（美国、英国、德国、日本）。
- **评分**：医学物理师和医生编写的 HealthBench 式正负评分细则（rubric）；MedDialogRubrics 式"必问项"，例如"是否询问家中有无孕妇或幼儿"。
- **关键违规（出现即判失败）**：说出引擎没有算出的数字；给出规则引擎没有给的许可；遗漏急症联系方式；在缺少信息时给出安慰性结论。
- **一致性与鲁棒性**：Pass^k 一致性；第 1 轮与第 10 轮以后的漂移对比；MedPRESS 式四级施压脚本和权威身份声称；在 POI 文本中注入指令的攻击。
- **行动评测**：决定、追问、转交三类行动分别计分。
- **评审**：用 LLM 评审团时，先测量它与临床专家的一致率。

最后必须加一项真实用户研究，因为 Bean 等已经证明模拟用户高估了实际获益（[Bean 2026](https://ora.ox.ac.uk/objects/uuid:ccb8b891-c738-4890-a4d5-45959d0e04d9/files/rns0648349)）。目前不存在这样的基准，把它公开发布，本身就可能投 NeurIPS Datasets & Benchmarks 这类赛道【推断】。

**默认保护隐私的设计。** 剂量逻辑和对话都在设备端运行；原始位置不上传；人群信息只以粗粒度瓦片或场所类别下载、在本地匹配；日志由患者自己保存；接触记录只记次数和时长，不记身份，未成年人的数据不写进系统；v1 不做监管上报；临床支持数据和任何汇总报告数据分库存放；限制期结束后自动删除，即日落条款；接受独立的伦理审查和 DPIA。人群数据降级为背景信息（"这类场所此时段通常较拥挤"），以患者自述为主要输入。中断式告警设每天的上限（误报预算），优先采用环境式提示。

**第一个试点人群与地区。** 建议分三步，替换论文提出的"放射性碘先行"【推断】：

1. **在德国的强制住院期内先做院内验证。** 德国本来就要求在出院时测量并记录 2 m 处剂量率（[Kurth 2018](https://link.springer.com/article/10.1186/s13550-018-0386-4)），在 48 h 住院期间让患者佩戴设备，就能在零门诊风险下拿到"可穿戴读数 vs 参考测量"的配对数据。
2. **在已允许门诊治疗的 177Lu 中心做影子模式。** 英国有门诊 177Lu-DOTATATE 的先例（[PMC6484059](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6484059/)），美国有门诊 177Lu-PSMA 的可行性研究（[PMC11619580](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11619580/)），NRC 按疗程计算的方向也让跨周期累积追踪成为真实需求。177Lu 的 γ 线容易测，经尿清除主要发生在 48 h 内（[Levart 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/)），四周 TLD 参考标准已有成熟设计。
3. **在按剂量放行、床位又紧张的辖区做监管共建试点。** 日本是首选：JSCO 已经认可按患者特异累积剂量计算出院，并呼吁制定新指南（[Int J Clin Oncol 2025](https://link.springer.com/article/10.1007/s10147-025-02858-3)），同时存在候诊压力。

放射性碘如果保留，只适合作为门诊甲亢患者的可用性和人机交互试点，污染指导由规则引擎负责。在污染模型完成独立验证之前，应避开高活度甲状腺癌治疗。IAEA TC 受援国的试点必须按"没有治疗前影像、没有人群数据"的条件设计。

## 风险与机会

| 风险 | 可能性与影响 | 依据 | 缓解措施 |
|---|---|---|---|
| LLM 在患者施压下附和，放宽限制 | 高；高 | [HeaLing 2026](https://aclanthology.org/2026.healing-1.2.pdf)；MedPRESS【arXiv】 | 许可只能由规则引擎改变，质疑退回引擎或人工 |
| 人群数据导致误报和漏报，加剧焦虑与污名 | 高；中 | [Google 帮助](https://support.google.com/business/answer/6263531)；[Herfort 2023](https://www.nature.com/articles/s41467-023-39698-6)；[Tran 2023](https://pmc.ncbi.nlm.nih.gov/articles/PMC10358285/) | 人群数据降级为背景信息；设误报预算；告警文案与患者共同设计 |
| 近场反演误差和排尿阶跃 | 中；高 | [Bailey 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/) | 相对趋势加绝对锚定；状态检测；保形回退 |
| 代理模型分布外失效 | 中；高 | [Ha 2026](https://doi.org/10.1088/1361-6560/ae5375)；[Ennassiri 2026](https://doi.org/10.1088/2057-1976/aea427) | 物理结构化模型；凸包门控 |
| 监管上报压低采纳率并带来法律障碍 | 高；高 | [GAO](https://www.gao.gov/products/gao-21-104622)；[GDPR](https://eur-lex.europa.eu/eli/reg/2016/679/oj) | v1 删除上报功能 |
| 双重监管加严格责任 | 确定；高 | [QLE](https://www.qle.be/the-qle-regulatory-brief/ai-acts-digital-omnibus-is-now-law-ai-medical-devices-get-until-2-august-2028-but-notified-bodies-face-a-28-january-2028-deadline)；[PLD](https://www.orrick.com/en/Insights/2025/06/New-EU-Directive-Modernizes-Defective-Product-Liability-to-Address-New-Technologies) | 从设计阶段就按高风险系统合规；保留全量日志 |
| 多辖区规则库的维护负担和责任 | 确定；中 | 各国规则差异 | 规则即代码、版本化、冲突检查 |
| α 核素兴起削弱剂量率监测的价值 | 中；中 | [RYZ101](https://www.biopharminternational.com/view/ryz101-phase-3-action-1-gep-net-primary-endpoint)；[CLEAN-225Ac](https://www.sciencedirect.com/science/article/pii/S3051291326002600) | 提前布局污染与卫生模块 |
| 大平台的通用健康智能体"够用" | 中；中 | [ChatGPT Health](https://openai.com/index/health-in-chatgpt/) | 以临床验证和监管认可为护城河；提供 MCP 或 HealthKit 接口 |

| 机会 | 依据 | 如何抓住 |
|---|---|---|
| NRC 改为按疗程计算，DG-8070 接受实测剂量率 | [DRPS](https://www.diagnosticradphys.com/physicspulse/medical-health-physics/nrc-2026-rulemaking-wave-medical-use/)；[SNMMI 意见函](https://www.nrc.gov/docs/ML2625/ML26254A152.pdf) | 定位为持证方一侧的患者特异计算和跨周期累积工具 |
| IAEA 自己承认放行标准需要更新 | [IAEA TM 2025](https://www.iaea.org/sites/default/files/25/06/rpop_tm_nuclear_medicine_2025.pdf) | 让 TC 项目的产出成为修订指南所需的证据和验证协议 |
| 新证据显示现行规则普遍保守 | [Levart 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/)；[Schütze 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC13046430/) | 把"个体化放宽、同时盯住尾部"作为核心论点 |
| 放射防护对话评测领域还是空白 | 笔记没有找到先例 | 发布 IRSA-Bench |
| PBPK 结合可穿戴数据同化还没有实现 | 笔记没有找到先例 | 写成方法学论文 |
| 药企有动力消除床位瓶颈 | [Novartis–Barts](https://www.novartis.com/uk-en/about/partnerships/collaborative-working/barts-health-nhs-trust-st-bartholomews-hospital) | 以合规与教育支持起步，避开 PDURS 的标签证据门槛 |

## 结论

梳理完证据后，IRSA 的价值链需要重新排序。最可能带来收益的，是连续测量个体有效半衰期加上确定性的规则引擎，而不是 MC 训练的 AI 代理模型或 LLM。前者针对的是误差预算中最大、而且能观测到的一项；后两者分别对应已经高度保守、维度又低的物理正问题，以及 2026 年文献中失效记录最多的一层。论文应该让 "AI" 这个标签服从证据：把学习模型用在体表信号反演这个真正的难点上，把 LLM 限定为受校验的表达层。三方对比的结果哪怕是"半解析模型已经够用"，也是对监管者更有利的结论。

论文提出的"中心问题"也有更好的答案：监管保证应该来自持证方留存的验证证据和可审计日志，而不是对患者的追踪。美国 NRC 正在削减记录、按疗程放宽限制，欧盟 GDPR 要求上报必须有法律依据，两边指向同一个设计。时间窗口很紧：PLD 于 2026 年 12 月起适用，AI Act 对医疗器械的高风险义务于 2028 年 8 月起适用。一个处在概念阶段的项目，应把高风险合规当作设计输入，而不是事后补丁。IAEA 自己的技术会议已经呼吁更新放行标准、修订现有报告。在这个节点上，一个 TC 项目交付一套公开的验证协议和评测基准，长期影响可能比交付一个 App 更大。
