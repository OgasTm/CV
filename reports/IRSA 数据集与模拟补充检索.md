# 新资源补齐物理与规则，患者轨迹仍须自测

**结论先行：本轮补充检索把上轮报告留下的大部分"物理参数"和"许可证"缺口补上了，但 IRSA 最核心的三类数据仍然没有任何公开来源：出院后佩戴式探测器的逐患者原始轨迹、0.5 m 以内的 177Lu 近场开放实测、放射防护患者对话基准。** 新发现集中在四处。第一，177Lu 的剂量率常数不必再购买 Smith & Stabin：Oumano 2025（CC BY）转载了 0.181 R·cm²/(mCi·h)（[Oumano 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12059296/)），Peplow 2020 的 DOE 接受稿可在 OSTI 免费获取，给出 1 m 处有效剂量率 5.18 µSv·h⁻¹·GBq⁻¹（ICRP 107 数据）（[OSTI 1607209](https://www.osti.gov/biblio/1607209)）。但不同来源用的物理量不同，取值最多相差约 1.8 倍，IRSA 必须先固定一个量【本报告计算】。第二，佩戴读数的正向仿真链已能全部用开源工具搭起来（G4Radiacode、MIT 许可的 Gamma Dose Converter、Sandia 的 InterSpec/SpecUtils/PyRIID）。MRTDosimetry 四器官体模在 1–144 h 六个时间点的影像已以 CC BY 发布到 Zenodo，是"读数 → 滞留活度"反演最干净的已知真值试验台（[zenodo.18891044](https://doi.org/10.5281/zenodo.18891044)）。第三，日本 MHLW 的患者退出通知（著作权法第 13 条 + PDL1.0）和英国 ARSAC 2026 年 7 月版指南（OGL v3.0）是首批可以直接编码、可以商用再分发的官方规则文本。前者还附带一个现成的"规则版本漂移"测试用例：新增 177Lu-PSMA 1 m 处 16 µSv/h 的修订（[MHLW](https://www.mhlw.go.jp/content/10800000/001527013.pdf)；[GOV.UK](https://www.gov.uk/government/publications/arsac-notes-for-guidance)）。第四，相关论文给出了 IRSA 必须先打败的简单基线。Shimizu 2026 用治疗前 SSTR 显像的洗脱率预测 6 h 能否出院，AUC 0.929；Wakabayashi 2025 预测第 1 天剂量率，RMSE 2.24/3.47 µSv/h；Ebrahimi 2018 用神经网络识别家属剂量 >1 mSv，AUC 0.957。评测方面，2026 年多项真人对照研究显示，LLM 模拟用户过度合作、风格单一，会高估智能体表现（[arXiv 2603.11245](https://arxiv.org/abs/2603.11245)）。因此 IRSA-Bench 必须加一个真人校准子集。对此前提出的约 18 个月压缩方案，这些发现让第 3 个月的演示和第 6–9 个月的物理证据更有把握；"基准与临床数据完全脱钩"一条则需要修正，基准应分合成 v0 和真人校准 v1 两步做，总工作量大致仍在 5–7 人年【估算】。许可证仍是暗线：PyTheraTwin 本身是 MIT，但它依赖的 PyCNO 没有许可证，SIMIND 仅限研究用途；CASAS（CC BY）和南非 88 户家庭接触数据（CC0）可以商用，马拉维、津巴布韦、MITRE、AMASS、Habitat 人形资产则不行。

> **阅读说明。** 本报告是《IRSA 前期开发与公开数据集》（2026-11-20）的补充。这里只收录本轮（2026-11-23 核对）新增的资源，以及上轮条目的状态更新：许可证厘清、缺口补上、说法更正。上轮已收录的条目不再展开，提到时只简述其身份。标签含义如下。【推断】：研究笔记或本报告的推理。【估算】：工作量与周期估计，没有文献依据。【本报告计算】：本报告做的单位换算或简单算术。【search-summary, verify】【search-level, verify】：只见过检索摘要，需要核对原文。【abstract-only】：只有会议摘要，没有同行评审论文。【title-only】：只确认了题录，没有读摘要。【snippet】：来自检索片段或新闻报道。【未核实】：许可证或细节本轮未能确认。【预印本】：未经同行评审。"能否商用"是对许可证文字的工程解读，不构成法律意见。为避免各笔记模块编号互相冲突，表格中的"服务模块"一律用中文名称：读数反演、剂量引擎（NRC 点源基线 / 半解析 / MC）、MC 训练与代理模型、不确定性、验证、接收者体模、核数据、动力学先验、合成轨迹、接触先验、户型、事件检测、场景引擎、规则引擎、知识库、评测基准、模拟器、评审、多语言。

## 执行摘要：十一项新资源改变了开发路径，五类核心数据仍是空白

下表按对 IRSA 开发的影响排序，列出本轮最重要的新发现。

| # | 新发现 | 关键事实 | 对 IRSA 的意义 | 链接 |
|---|---|---|---|---|
| 1 | **177Lu 剂量率常数有了开放来源** | Smith & Stabin 的 177Lu 照射率常数 0.181 R·cm²/(mCi·h)、铅 HVL 0.542 mm，经 Oumano 2025（CC BY）转载。Peplow 2020 的 1 m 有效剂量率常数：177Lu 为 4.92–5.18 µSv·h⁻¹·GBq⁻¹，131I 为 54.7–54.8 µSv·h⁻¹·GBq⁻¹ | 上轮"必须购买"的缺口关闭，NRC 点源基线第 1 周即可实现。但各来源相差约 1.8 倍，必须先定物理量 | [PMC12059296](https://pmc.ncbi.nlm.nih.gov/articles/PMC12059296/)；[OSTI 1607209](https://www.osti.gov/biblio/1607209) |
| 2 | **佩戴读数正向仿真链全部开源** | Gamma Dose Converter（MIT，2026-08）用 Geant4 响应矩阵按 Casanovas 2016 方法把能谱换算为 H*(10)；InterSpec、SpecUtils 为 LGPL-2.1；PyRIID 为 BSD-3；GuideRadPROS 有源剂量计能量/角响应测试数据为 CC BY 4.0 | 读数反演的仿真工具不必自写。Radiacode 本身的能量和角响应仍须实测 | [zenodo.21789847](https://doi.org/10.5281/zenodo.21789847)；[InterSpec](https://github.com/sandialabs/InterSpec)；[zenodo.18133170](https://doi.org/10.5281/zenodo.18133170) |
| 3 | **宽松许可的点核代码与代理模型训练原则** | rad_point_kernel（MIT，PyPI 1.8.0，2026-05-01）带 MC 推导的积累修正。PTNO 证明：MC 噪声无偏时，最小二乘的最优解不变；预算固定时，多而嘈杂的场景胜过少而收敛的场景 | 半解析引擎的附加屏蔽项可直接调用；MC 预算按"多场景、低统计"分配 | [rad_point_kernel](https://github.com/fusion-energy/rad_point_kernel)；[arXiv 2609.40090](https://arxiv.org/abs/2609.40090) |
| 4 | **近场几何与体衰减有了定量文献** | Li 2024：PHITS 模拟 177Lu-PSMA 成人男性体模，1 m 内与实测的 rRMSD 为 0.21–0.25，前后方向高于左右。Dewji：131I 点源公式在 1 m 处高估 1.9–3.3 倍 | 给出半解析和代理模型相对实测的现实误差目标，以及点源偏倚的先验 | [PMID 38537256](https://pubmed.ncbi.nlm.nih.gov/38537256)；[OSTI 1213312](https://www.osti.gov/servlets/purl/1213312) |
| 5 | **已知真值的多时间点体模 + 第一例开放 177Lu-PSMA 患者投影** | MRTDosimetry 四器官 177Lu 体模在 1/4/24/40/72/144 h 的平面像、原始与重建 SPECT 和 CT，CC BY。另有一名 177Lu-PSMA-617 患者的 2 床位 SPECT 投影 + CT，CC BY | 反演链可在接触患者之前做端到端测试；真实 PSMA 摄取模式可作 MC 源 | [zenodo.18891044](https://doi.org/10.5281/zenodo.18891044)；[zenodo.14172228](https://doi.org/10.5281/zenodo.14172228) |
| 6 | **数字孪生管线与宽松动力学栈** | PyTheraTwin（MIT）：CT → TotalSegmentator → PBPK 时间活度曲线 → SIMIND SPECT → OpenGATE 剂量。tellurium（Apache-2.0）、PySB（BSD-2）、Chi（BSD-3）。topas-rpt（MIT）按正确时间结构抽样完整衰变链 | 合成患者可以起步；产品通道要替换 PyCNO（无许可）和 SIMIND（仅限研究） | [PyTheraTwin](https://github.com/qurit/PyTheraTwin)；[tellurium](https://github.com/sys-bio/tellurium)；[topas-rpt](https://github.com/bertoletlab/topas-rpt) |
| 7 | **可引用的排泄先验** | Lutathera 说明书：5/24/48 h 累计尿排 44%/58%/65%。de Bakker 2023：快排组与慢排组 48 h 排出 75% vs 36%，前 20 h 半排时间 4.4 h vs 8.4 h，eGFR 是协变量。Pluvicto 说明书：终末半衰期 41.6 h（CV 68.8%） | 排尿阶跃生成器可以全部用开放汇总数据参数化 | [Lutathera PI](https://www.novartis.com/us-en/sites/novartis_us/files/lutathera.pdf)；[PMC10547615](https://pmc.ncbi.nlm.nih.gov/articles/PMC10547615/)；[Pluvicto PI](https://www.novartis.com/us-en/sites/novartis_us/files/pluvicto.pdf) |
| 8 | **可直接编码的官方规则文本** | 日本 MHLW：I-131 放行条件为 500 MBq 或 1 m 处 30 µSv/h；Lu-177 SSTR 为 18 µSv/h；修订新增 Lu-177 PSMA 16 µSv/h；依据著作权法第 13 条和 PDL1.0 可再利用。英国 ARSAC NfG 2026 年 7 月版为 OGL v3.0，表 7.1 规定 I-131 治疗后 4 个月内避孕 | 首个可测试的规则包不必等 IAEA 许可，而且自带版本漂移用例 | [MHLW PDF](https://www.mhlw.go.jp/content/10800000/001527013.pdf)；[ARSAC NfG](https://assets.publishing.service.gov.uk/media/6a510b75c3d64d94cacab7aa/ARSAC_NfG_July_2026.pdf) |
| 9 | **可商用的家庭行为数据** | CASAS：189 户，2007–2024，含户型图，CC BY 4.0。南非 SA-S-HTS：88 户，CC0。TotTag 用 UWB 以 1 Hz 测距，没有哪对亲子在 ≤3 ft 内的时间超过 12 h 观察时段的 16% | 场景引擎有了如厕/卧床时刻先验和近距离时长的上界 | [zenodo.15708568](https://zenodo.org/records/15708568)；[SocioPatterns](https://sociopatterns.org/datasets/south-african-households-contact-networks/)；[PMC8907335](https://pmc.ncbi.nlm.nih.gov/articles/PMC8907335) |
| 10 | **评测模板，以及"模拟-真实差距"的证据** | Cancer-Myth（CC BY 4.0）：前沿模型纠正错误前提的比例不超过 43%。DischargeBench：话题清单 + 模拟理解 + 声明级安全审计。Sim2Real：451 名真人对比 31 个模拟器，模拟器"过度合作、风格单一" | IRSA-Bench 的结构有现成模板，但分数必须经真人校准 | [Cancer-Myth](https://huggingface.co/datasets/Cancer-Myth/Cancer-Myth)；[arXiv 2609.20827](https://arxiv.org/abs/2609.20827)；[arXiv 2603.11245](https://arxiv.org/abs/2603.11245) |
| 11 | **必须超越的基线和问题陈述证据** | Shimizu 2026 AUC 0.929；Wakabayashi 2025 RMSE 2.24/3.47 µSv/h；Ebrahimi 2018 AUC 0.957；Zhang 2025 SVM 准确率 88.04%（n = 1,044）。Zahner 2026：23 国 102 家中心的出院后防护指导差异很大 | 预注册的比较对象；论文的动机陈述 | [Shimizu](https://doi.org/10.1097/mnm.0000000000002110)；[PMC12174230](https://pmc.ncbi.nlm.nih.gov/articles/PMC12174230/)；[Zahner](https://doi.org/10.2967/jnumed.126.273336) |

**仍然缺失的部分同样清楚。** 第一，出院后**居家**佩戴式探测器的逐患者轨迹：Gallicchio 2021 只在院内防护病房测量，而且没有公开数据（[PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/)）；SELFIE 登记为已完成、目标 50 人，但截至 2026-11-23 既没有贴出结果，也没有找到论文（[NCT04922801](https://clinicaltrials.gov/study/NCT04922801)）。第二，177Lu 近场的开放实测数据：Li 2024 论文付费，没有发布数据；EURADOS WG13 的 177Lu-PSMA 案例只有会议幻灯片。第三，Radiacode 在 113/208/364 keV 下的独立能量和角响应表征。第四，影像数据：131I SPECT/平面患者影像、225Ac 患者影像、成规模的 SSTR PET 队列、多时间点 177Lu-PSMA 患者数据，DataCite、Zenodo、TCIA 和 Hugging Face 都确认没有。第五，任何语言的放射防护患者对话数据集。此外还缺：RPT 患者与家属之间用传感器实测的距离；有许可证的 177Lu-DOTATATE、131I、225Ac PBPK 代码；RPT 旁观者几何下的有效剂量/H*(10) 比值；OpenDose 数据库许可证。

## 上轮缺口盘点：许可证大多已厘清，临床与近场的开放数据仍未补上

| 上轮缺口 | 本轮状态 | 依据与说明 | 链接 |
|---|---|---|---|
| 佩戴式探测器在 RPT 后的逐患者原始时间序列 | **仍缺** | Gallicchio 2021（n = 20，131I）只在院内测量，没有数据；SELFIE 未发表。可用 MRTDosimetry 六时间点体模平面像作为已知真值的代用试验台 | [PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/)；[zenodo.18891044](https://doi.org/10.5281/zenodo.18891044) |
| 0.5 m 以内、多朝向的 177Lu 近场实测 | **部分补**（只有文献，没有开放数据） | Li 2024：1 m 内 rRMSD 0.21–0.25，模拟/实测比 0.82–1.19，前后 > 左右。Sheikhzadeh 2025：0.5 m 与 1 m 剂量率之比约 1.4–1.5，远低于平方反比的 4（笔记计算）。EURADOS WG13 只有幻灯片【search-level, verify】 | [IOP](https://iopscience.iop.org/article/10.1088/1361-6498/ad3835)；[PMID 40434013](https://pubmed.ncbi.nlm.nih.gov/40434013/)；[Covens](https://smrd2025.efomp.org/wp-content/uploads/2025/12/Covens.pdf) |
| CsI 小探测器在 113/208/364 keV 的能量与角响应 | **仍缺** | 没有独立的同行评审表征，只有一篇博客估计 FWHM 7–10%。GuideRadPROS（CC BY）可作商用 EPD 偏差的先验。两个相关的 2026 年 Zenodo 记录为受限访问 | [Radiacode 100 系列](https://radiacode.com/100-series?lang=en)；[zenodo.18133170](https://doi.org/10.5281/zenodo.18133170) |
| 体位、排尿与佩戴读数之间的关系 | **部分补**（工具和先验，没有实测） | 姿态变形体模（Yeom 2019、MPPD）；如厕时刻先验（CASAS）；排泄分组（de Bakker 2023）。体位与读数的关系仍未测过 | [PMID 39230434](https://pubmed.ncbi.nlm.nih.gov/39230434/)；[zenodo.15708568](https://zenodo.org/records/15708568) |
| 177Lu"活度分布 → 接触者位置剂量场"训练集 | **仍缺**（生成条件改善） | 没有公开数据集；有人建了一个仓库，描述为 177Lu/161Tb/212Pb/225Ac 患者周围的 H*(10) MC，但 2026-11-23 时仓库是空的。开放的患者 SPECT、PyTheraTwin、topas-rpt 和 PTNO 让自行生成更容易 | [lucasfffaria 仓库](https://github.com/lucasfffaria/Monte-Carlo-Dosimetry-of-Therapeutic-Radioligands) |
| 同时记录真实距离和行为的家庭接触数据 | **部分补** | TotTag 有 UWB 连续测距，但数据许可证不明；UC Berkeley 痴呆照护研究逐秒测距，但数据未公开；SocioPatterns 家庭数据只有 1–1.5 m 阈值；都不是 RPT 人群 | [PMC8907335](https://pmc.ncbi.nlm.nih.gov/articles/PMC8907335)；[PMC9771049](https://pmc.ncbi.nlm.nih.gov/articles/PMC9771049/) |
| 放射防护患者对话基准；评审与专家一致性数据 | **仍缺** | Hugging Face 检索"radioiodine""nuclear medicine""radiation oncology"均无问答数据。可借用 Cancer-Myth 结构和多项评分细则方法 | [Cancer-Myth](https://huggingface.co/datasets/Cancer-Myth/Cancer-Myth) |
| 177Lu 的开放 Γ（上轮写需购买 Smith & Stabin） | **已补** | Oumano 2025（CC BY）转载 Smith & Stabin 数值；Peplow 2020 OSTI 免费；NRC 起草稿列 0.00527【search-summary, verify】 | [PMC12059296](https://pmc.ncbi.nlm.nih.gov/articles/PMC12059296/)；[OSTI 1607209](https://www.osti.gov/biblio/1607209)；[ML21348A111](https://www.nrc.gov/docs/ML2134/ML21348A111.pdf) |
| 开放的现代积累因子表 | **部分补** | rad_point_kernel 用 MC 推导积累修正，绕开 ANS 表；JAERI-M 86-071/88-198 在 INIS 有记录，免费全文未确认；Phy-X/PSD 免费但需注册；仍没有 CC 许可的 EPDL/ENDF 积累因子表 | [INIS](https://inis.iaea.org/records/apt99-7p625)；[rad_point_kernel](https://github.com/fusion-energy/rad_point_kernel) |
| FLUKA、PENELOPE/penEasy、MCNP6、SIMIND 许可【未核实】 | **大部分已补** | MCNP6.3 源码受出口管制；FLUKA 注册需审批；SIMIND 仅限研究、教育和培训；MC-GPU 为公共领域；penEasy 许可证文本仍未读到 | 见物理层一节 |
| radot 的 PyPI 包未确认 | **已更正：不在 PyPI** | 2026-11-23 PyPI JSON 和 simple 索引均返回 404，但 README 仍写 `pip install radot` | [radot](https://github.com/dsarrut/radot) |
| OpenDose 数据库许可证 | **仍缺** | CC BY 4.0 只覆盖期刊文章，没有找到数据库条款页；opendose.polsl.pl 与 opendose.org 的关系未核实 | [JNM 2020](https://jnm.snmjournals.org/content/61/10/1514) |
| MRTDosimetry 许可证（两份笔记说法不一） | **已补**（Zenodo 镜像） | Zenodo 镜像为 CC BY 4.0，标注 `isPartOf` OSF 69NGE | [zenodo.18891044](https://doi.org/10.5281/zenodo.18891044) |
| 131I SPECT/CT、225Ac 影像、SSTR PET 队列 | **仍缺**（225Ac 只有体模） | TCIA 共 6,140 个 PT 序列，唯一的 PSMA 集合是 PSMA-PET-CT-Lesions，没有 SSTR/DOTATATE 集合；225Ac 只有 Polson 体模 | [NBIA API](https://services.cancerimagingarchive.net/nbia-api/services/v1/getSeries?Modality=PT)；[zenodo.14166469](https://doi.org/10.5281/zenodo.14166469) |
| Fele-Paranj PBPK 无许可证 | **仍缺** | 其 Python/SBML 移植 PyCNO 同样没有许可证；可用 tellurium/PySB 按已发表方程重写 | [PyCNO](https://github.com/qurit/PyCNO) |
| 177Lu-DOTATATE 和 131I 的逐次排尿数据 | **部分补**（只有累计汇总） | Lutathera 说明书给出累计曲线；de Bakker 2023 给出 PSMA 快/慢分组；仍没有逐次排尿数据 | [Lutathera PI](https://www.novartis.com/us-en/sites/novartis_us/files/lutathera.pdf) |
| 225Ac 外照射数据 | **部分补**（模型值） | MHLW 2024 研究报告中的 225Ac 有效剂量率常数 0.03059 µSv·m²/(MBq·h) 是模型输入，不是实测【search-level, verify】；Peplow 只有母体值 | [MHLW 报告](https://mhlw-grants.niph.go.jp/system/files/report_pdf/202421004A-buntan2.pdf) |
| 接触数据许可证（CoMix、Prem、Copenhagen、MIT PACT、马拉维、Meta HRSL） | **已补** | 详见许可证一节 | 见许可证一节 |
| 户型数据（LMIC 和家具位置） | **部分补** | CASAS 户型图（美国）；ProcTHOR/AI2-THOR 程序化住宅（Apache-2.0）；LMIC 户型仍缺 | [zenodo.15708568](https://zenodo.org/records/15708568)；[ProcTHOR](https://github.com/allenai/procthor) |
| HPS "Ask the Experts" 许可 | **已核实：仍受限** | 没有文本挖掘条款；robots.txt 允许抓取，但抓取许可不是再利用许可；转载需作者或 HPS 许可 | [HPS 免责声明](https://hps.org/disclaimer/) |
| IAEA 文本再利用条款 | **部分补** | 出版物版权页已核实（通常收取版税，非商用复制和翻译"欢迎并逐案审议"）；网站使用条款页仍被 Cloudflare 拦截 | [SSG-46 PDF](https://www-pub.iaea.org/MTCD/Publications/PDF/PUB1775_web.pdf) |
| JSNM Lu-177-DOTATATE 手册许可证 | **已补：CC BY** | Europe PMC 许可证字段 | [PMC5852188](https://pmc.ncbi.nlm.nih.gov/articles/PMC5852188/) |
| 日本、德国现行条款的综合文本 | **日本已补，德国部分补** | MHLW 通知合订本（最后修订 R3.8.19）加 PSMA 修订（只读到广岛市转发件）；德国 Richtlinie 部分章节已被取代，没有找到 BMUV 合订本 | [MHLW](https://www.mhlw.go.jp/content/10800000/001527013.pdf)；[umwelt-online](https://umwelt-online.de/recht/energie/strahlen/rl/medstrschrl.htm) |
| NRC DG-8070 全文 | **本轮未检索，仍缺** | 本轮笔记没有涉及 | — |
| 意大利 CNR 甲状腺患者无线设备【search-summary, verify】 | **已补：即 Gallicchio 2021** | IEEE JTEHM，n = 20，r = 0.96 | [PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/) |
| Google Places 现行存储条款 | **已补** | 经纬度最多缓存 30 天；Maps Platform 条款禁止处理受 HIPAA 管辖的健康信息和"高风险活动" | [Maps 服务条款](https://cloud.google.com/maps-platform/terms/maps-service-terms) |
| NHS 信托手册许可证 | **本轮未复查** | — | — |
| Synthea 的核医学模块 | **仍缺** | 模块代码中检索 iodine、lutetium、radioiodine、PSMA、nuclear medicine 均为 0 命中 | [Synthea modules](https://github.com/synthetichealth/synthea/tree/master/src/main/resources/modules) |
| 德国核医学治疗床位数据 | **部分补** | Zippel 2021：全国约 234,000 个治疗日，储备约 19,000 床日（约 63 张床） | [doi:10.1055/a-1351-0030](https://doi.org/10.1055/a-1351-0030) |

## 物理层：177Lu 剂量率常数不再需要购买，佩戴读数的正向仿真链可以全用开源工具拼出

### 剂量率常数：同一核素最多差 1.8 倍，先定物理量再定数值

上轮报告写到，NRC RG 8.39 Rev. 1 没有 177Lu 一行，只能购买 Smith & Stabin 2012。本轮找到两条免费路径。**Oumano 2025（JACMP，CC BY）表 4 转载了 Smith & Stabin 的数值**：177Lu 照射率常数 0.181 R·cm²/(mCi·h)，铅 HVL 0.542 mm、TVL 2.11 mm；131I 为 2.2 R·cm²/(mCi·h)，铅 HVL 2.74 mm（[PMC12059296](https://pmc.ncbi.nlm.nih.gov/articles/PMC12059296/)）。**Peplow 2020 的 DOE 接受稿可在 OSTI 免费下载**。它定义的"特征 γ 剂量常数"是 1 m 处的有效剂量率（ICRP 116，AP 几何），既不是空气比释动能，也不是 H*(10)。用 ICRP 107 发射数据计算，177Lu 为 5.176 × 10⁻⁶ mSv·h⁻¹·MBq⁻¹，131I 为 5.483 × 10⁻⁵（[OSTI 1607209](https://www.osti.gov/biblio/1607209)）。按 RG 8.39 的 1 R = 10 mSv 约定，Smith & Stabin 的 0.181 换算为 0.00489 mSv·m²/(GBq·h)；按空气比释动能换算约为 0.0043 mGy·m²/(GBq·h)。上轮引用的 Bellamy 2022 理论裸源值是 0.007636 mSv/h/GBq，**因此 177Lu 的"NRC 基线"随取值不同可相差约 1.8 倍（0.0043–0.0076）**（笔记计算）。研究笔记建议采用 Peplow 的 ICRP 107 有效剂量值（5.18 µSv·h⁻¹·GBq⁻¹），因为它和 ICRP 116 的接收者剂量学一致；其余取值作为敏感性分析。接收者剂量用有效剂量 E，与巡测仪验证数据比较时用 H*(10)【推断】。

这些新数值也修正了上轮报告的一个推断。上轮写的是"I-131 约 0.059、177Lu 理论值约 0.0076 µSv·m²/(MBq·h)，相差约 8 倍"，但这两个数来自不同的物理量。如果都取 Peplow ICRP 107 的数值，131I/177Lu 约为 10.6 倍；都取 Smith & Stabin 的数值，约为 12.2 倍【本报告计算】。225Ac 的情况更复杂。Peplow 表中 Ac-225 的 2.796 × 10⁻⁶ 只是母体本身，**低估了光子输出**，链平衡时的 Γ 要按分支比把 221Fr 和 213Bi 的贡献加起来；213Bi 的数值本轮没有提取。日本 MHLW 2024 研究报告用的 225Ac 有效剂量率常数是 0.03059 µSv·m²/(MBq·h)【search-level, verify】（[MHLW 报告](https://mhlw-grants.niph.go.jp/system/files/report_pdf/202421004A-buntan2.pdf)），约为 Peplow 母体值的 11 倍【本报告计算】。这与"包含子体"的解释相符，但未经核实。最后，笔记没有找到 RPT 患者旁观者几何下有效剂量与 H*(10) 的比值；EURADOS-EANM 把"多种近距离构型下每单位活度的有效剂量率"列为后续工作，截至 2026-11-23 没有找到后续论文（[Struelens 2024](https://pmc.ncbi.nlm.nih.gov/articles/PMC11035505)）。

| 来源 | 物理量 | 177Lu | 131I | 其他核素 | 177Lu 折合 mSv·m²/(GBq·h) | 许可/获取 | 链接 |
|---|---|---|---|---|---|---|---|
| Smith & Stabin 2012，经 Oumano 2025 表 4 转载 | 1 cm 处照射率常数，R·cm²/(mCi·h)；f 因子；铅 HVL/TVL | 0.181；f = 0.957；Pb HVL 0.542 mm，TVL 2.11 mm | 2.2；f = 0.963；Pb HVL 2.74 mm，TVL 9.93 mm | Tc-99m 0.795；F-18 5.7 | 0.00489（按 1 R = 10 mSv）；约 0.0043 mGy（按空气比释动能）（笔记计算） | Oumano 文章 CC BY；原文付费，RADAR 网站有副本 | [PMC12059296](https://pmc.ncbi.nlm.nih.gov/articles/PMC12059296/)；[RADAR](https://www.doseinfo-radar.com/Exposure_Rate_Constants_and_Lead_Shielding_Values%204.pdf) |
| Peplow 2020，表 4（SCALE 6.2.3，1,264 种核素） | 1 m 处有效剂量率（ICRP 116 AP），mSv·h⁻¹·MBq⁻¹ | 4.918 × 10⁻⁶ | 5.468 × 10⁻⁵ | — | 0.00492 | DOE 接受稿，OSTI 免费 | [OSTI 1607209](https://www.osti.gov/biblio/1607209)；[PDF](https://www.osti.gov/servlets/purl/1607209) |
| Peplow 2020，表 5（ICRP 107，1,192 种核素） | 同上 | **5.176 × 10⁻⁶** | **5.483 × 10⁻⁵** | Ac-225 2.796 × 10⁻⁶（仅母体）；Fr-221 4.278 × 10⁻⁶；Ra-223 2.107 × 10⁻⁵ | 0.00518（推荐默认值【推断】） | 同上 | 同上 |
| NRC 供 ACMUI 审议的起草稿（RCD-21-181） | 外照射剂量率常数，mSv·m²·GBq⁻¹·h⁻¹ | 0.00527；同表引 Smith & Stabin 0.00489、Peplow 0.00518 | — | — | 0.00527 | 美国政府文件【search-summary, verify】 | [ML21348A111](https://www.nrc.gov/docs/ML2134/ML21348A111.pdf) |
| Bellamy 2022（上轮已收录） | 理论裸源值 | 0.007636 mSv/h/GBq | — | — | 0.0076 | PMC 开放获取 | [PMC9667277](https://pmc.ncbi.nlm.nih.gov/articles/PMC9667277/) |
| MHLW 2024 研究报告（202421004A） | 225Ac 有效剂量率常数，模型输入 | — | — | Ac-225 0.03059 µSv·m²/(MBq·h)；计划最大活度 10 MBq（PSMA-617）和 20 MBq（PSMA-R2） | — | 政府报告【search-level, verify】 | [MHLW 报告](https://mhlw-grants.niph.go.jp/system/files/report_pdf/202421004A-buntan2.pdf) |

体衰减先验方面，本轮补上了 131I 的对应值。Dewji 的 OSTI 手稿引用 Al-Haj 2007 的实测体屏蔽因子 **0.46 ± 0.13**（n = 311；女性 0.45，男性 0.50），以及 Yi 2013 的 1 m 处计算/实测比 1.79 ± 0.54（范围 0.82–4.13）（[OSTI 1213312](https://www.osti.gov/servlets/purl/1213312)）。它们在 131I 中的作用，相当于 Bellamy 的 48% 在 177Lu 中的作用；两组数加起来，可以界定 NRC 点源基线与半解析模型之间的差距【推断】。Sheikhzadeh 2025 在 47 例 177Lu-PSMA 患者中测得，给药后 5 min 时 0.5 m 和 1 m 处分别为 34.6 ± 9 和 24.6 ± 5 µSv/h，1 h 峰值为 48.5 ± 15 和 31.6 ± 8 µSv/h，照护者有效剂量为 20–1170 µSv（[PMID 40434013](https://pubmed.ncbi.nlm.nih.gov/40434013/)）。两个距离的比值只有约 1.4–1.5，远低于平方反比给出的 4（笔记计算）。可见在 1 m 以内，点源加平方反比对 177Lu 是失效的，而且失效不只来自体衰减，**半解析模型必须使用线源或体源**【推断】。

### 体模姿态与人体包络：能直接放进产品的只有 SMPL-Body、MakeHuman 和 BodyParts3D

在 ICRP 网格体模的姿态变形方面，汉阳大学 C.H. Kim 课题组走在前面：Yeom 2019 发表了行走、坐、弯腰、跪、蹲五种非站立姿态的剂量系数；Han 2025 发布了同时调整姿态和身高的 GUI 工具 MPPD，但没有找到下载条款。ORNL/NRC 的 PiMAL 是唯一有公开分发渠道、四肢可活动的体模工具，但它绑定 MCNP，解剖结构是程式化的。在辐射物理之外的人体形状和动作数据中，SMPL、SMPL-X、AMASS 都只限研究用途，部分 MPI 许可还禁止用它们训练商用网络。不经谈判就能放进产品的，只有 **SMPL-Body 子集（CC BY 4.0）、MakeHuman 导出模型（CC0）和 BodyParts3D（现行 README 写 CC BY 4.0）**。笔记的建议是：用 ICRP 或 UF/NCI 体模训练，只交付权重；端侧需要渲染人体或摆放探测器时，用 MakeHuman 或 SMPL-Body 的体表包络；AMASS 和完整 SMPL 只在离线研究中使用【推断】。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **MPPD**（Mesh-type Phantom Posture Deformer；Han H 等，Health Phys 2025） | 对 ICRP 成人 MRCP 及配套体模库做姿态变形的 GUI，可同时处理身高和姿态 | — | 摘要未说明 | **未找到下载页** | [PMID 39230434](https://pubmed.ncbi.nlm.nih.gov/39230434/) | 接收者体模（坐、卧）；MC 训练数据的姿态增广；患者坐卧时探测器位置变化 | 需联系汉阳大学；底层 MRCP 文件受 ICRP 再利用限制 |
| Yeom 等 2019，PMB 64:075018 | MRCP 变形为 5 种非站立姿态；29 个器官/组织和有效剂量的剂量系数，0.01 MeV–10 GeV，AP/PA/LLAT/RLAT/ROT | 5 种姿态 | 期刊版权（IOP） | 付费 | [doi:10.1088/1361-6560/ab0917](https://doi.org/10.1088/1361-6560/ab0917) | 接收者体模（宽束近似下的坐、卧姿修正） | 主要百分比差异未取得；同一团队的会议论文报告两种姿态间有效剂量差 <4%【search-summary, verify】（[KNS 2019](https://kns.org/files/pre_paper/41/19S-354-한혜진.pdf)） |
| **PiMAL**（ORNL/NRC） | 设定姿态、生成 MCNP 输入并计算剂量的 GUI；v4.1.0 有男女程式化体模，四肢可活动；NRC RIL 2025-03 记录 v6 | — | 所读来源未写明许可；美国政府资助，经 NRC RAMP 安装 | GitLab 落地页；NRC RAMP | [NUREG/CR-7243](https://www.nrc.gov/reading-rm/doc-collections/nuregs/contract/cr7243)；[GitLab](https://gitlab.com/RCD-1/pimal) | MC（坐、站姿患者与接收者，如 Dewji 2015） | 依赖 MCNP（出口管制）；程式化解剖；v6 许可证未读到 |
| 患者形变网格体模（Carter 等，Med Phys 2021） | 把参考网格体模按患者体表轮廓或"轮廓 + 器官"变形 | 概念验证 | 期刊版权 | 付费 | [doi:10.1002/mp.14784](https://doi.org/10.1002/mp.14784) | 读数反演（由身高体重或体表扫描生成患者个体化发射体） | 未发布代码 |
| 由体表几何生成体积体模（MICCAI 类论文，2018） | 由 2.5D 深度传感器的体表生成类 CT 体模，在 2,045 例全身 CT 上测试 | — | Springer 版权 | 付费 | [Springer](https://link.springer.com/chapter/10.1007/978-3-030-00928-1_20) | 读数反演（由手机体表扫描生成体模） | 【search-summary, verify】 |
| **SMPL**（MPI-IS） | 参数化体形与姿态模型 | — | **SMPL-Body 子集（无形状混合）为 CC BY 4.0**；完整 SMPL 商用需经 Max Planck Innovation 或 Meshcapade，最低商用档约 €1,500/年 | 注册 | [SMPL 许可](https://smpl.is.tue.mpg.de/license.html)；[Meshcapade](https://meshcapade.com/assets/body-models) | 接收者体模、读数反演（体表形状与探测器摆放） | 价格来自检索摘要 |
| SMPL-X | 身体 + 手 + 面部 | — | 仅限学术研究；禁止未经许可的商用、色情、军事、监控用途 | 注册 | [SMPL-X 许可](https://smpl-x.is.tue.mpg.de/modellicense.html) | 接收者体模 | — |
| AMASS | SMPL 参数化的大型动作捕捉库（坐、走、拥抱等序列） | 大型 | **非商用科研**；商用请求发至 ps-license@tue.mpg.de | 注册 | [AMASS 许可](https://amass.is.tue.mpg.de/license.html) | 读数反演（由真实动作合成探测器朝向时间序列）、MC 训练 | 相关 MPI 许可（SPEC、MANO）禁止用模型训练商用网络（[SPEC 许可](https://spec.is.tue.mpg.de/license.html)） |
| **MakeHuman / MPFB2** | 参数化人体网格生成器；Blender 插件 MPFB2 | — | 代码 AGPL；**官方未修改版本导出的模型为 CC0**；MPFB2 为 GPLv3，资产为 CC0 | 免费下载 | [MakeHuman 许可](https://static.makehumancommunity.org/about/license.html)；[MPFB](https://extensions.blender.org/add-ons/mpfb/) | 产品可用的任意姿态体表包络 | 只有体表，没有器官 |
| **BodyParts3D**（DBCLS） | 解剖结构多边形模型 | — | 现行 README 写 **CC BY 4.0**；较早来源写 CC BY-SA 2.1 JP（**冲突**） | 公开下载 | [README](https://dbarchive.biosciencedbc.jp/data/bodyparts3d/LATEST/README_e.html) | 接收者体模（器官网格） | 以官方许可页为准 |
| Z-Anatomy | 基于 BodyParts3D 的 Blender 解剖图谱 | — | **CC BY-SA 4.0** | GitHub | [Z-Anatomy](https://github.com/Z-Anatomy/Models-of-human-anatomy) | 接收者体模 | 共享相同方式；据论坛帖已不活跃 |

### 探测器响应与能谱工具：从 CsI 能谱到 H*(10) 的链路已经开源

在患者佩戴式探测器的仿真方面，已发表的工作仍然很少：WIDMApp（上轮已收录）、伊朗 IRD 床旁碘剂量计（用 MC 设计，固定在床旁而不是佩戴在身上）和华盛顿大学背心（上轮已收录）。**没有任何论文模拟过 RPT 患者身上佩戴单个 CsI 能谱仪的情形**，也没有论文报告以人体为背散射体和衰减体时佩戴式探测器的角响应。分析工具则已经成熟。笔记据此拼出一条完全开放的正向链路【推断】：G4Radiacode 或 GATE 生成晶体响应矩阵 R(E, θ)；Gamma Dose Converter 按 Casanovas 方法求 G(E)，把能谱换算成 H*(10)；体模 MC（GATE/PHITS）给出各源器官在探测器位置的光子注量谱；PyRIID 或 becquerel 生成并拟合合成能谱；SpecUtils 负责文件读写；InterSpec 用于人工核对。其中 BSD/MIT/LGPL 组件可以进入产品，GPL 的 G4Radiacode 只能留作离线工具。笔记还推断，**能谱观测量可能比剂量率更适合作反演输入**：例如 208 keV 峰与低能散射连续谱之比，应当随源深度和体内衰减路径变化。这一点没有在患者身上验证过，但它支持在 Radiacode 上记录完整能谱【推断】。

| 名称 | 内容 | 规模/版本 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **InterSpec**（Sandia） | 基于峰的能谱分析：核素识别、活度与**屏蔽拟合**、源龄、**剂量率计算器**；可打开 CsI 等多数格式；有 Windows/macOS/Linux/iOS/Android 版本 | — | **LGPL-2.1**（读过 LICENSE） | GitHub releases | [InterSpec](https://github.com/sandialabs/InterSpec) | 读数反演（113/208/364 keV 窗口分析、带屏蔽的活度拟合）、验证 | 基于峰，不模拟人体几何 |
| **SpecUtils**（Sandia） | C++ 库，有 Python 和 Java 绑定；读取 100 多种能谱文件格式，可写 13 种 | — | **LGPL-2.1** | GitHub | [SpecUtils](https://github.com/sandialabs/SpecUtils) | 读数反演的数据摄取（Radiacode CSV/XML → N42） | — |
| **PyRIID**（Sandia；PyPI `riid`） | 合成 γ 能谱生成，以及用于核素检测、识别、定量的 ML 模型 | PyPI v2.1.0（2024-08-09），Python 3.10–3.12 | **BSD-3-Clause** | `pip install riid` | [GitHub](https://github.com/sandialabs/PyRIID) | 读数反演（随源重新分布、散射变化的类 Radiacode 能谱）、MC 训练 | 依赖探测器"种子"；不用 GADRAS 能否生成种子未核实 |
| **becquerel**（LBNL） | 能谱、刻度、拟合、核数据访问 | PyPI v0.7.0（2025-10-06） | **BSD-3 风格的 LBNL 许可；PyPI 分类写 "Other/Proprietary"（冲突）** | `pip install becquerel` | [GitHub](https://github.com/lbl-anp/becquerel) | 读数反演（能量刻度漂移、峰拟合） | 再分发前通读 LICENSE.txt |
| **Gamma Dose Converter**（Woodbridge、Fearn） | 由单能 Geant4 运行构建探测器响应矩阵，按 Casanovas 2016 和 ICRP 74 导出能谱 → H*(10) 转换因子 | 0.2 MB，Zenodo 2026-08-04 | **MIT** | Zenodo | [doi:10.5281/zenodo.21789847](https://doi.org/10.5281/zenodo.21789847) | **读数反演、验证：用与能量无关的 G(E) 把 Radiacode 能谱换算成 H*(10)，不再依赖厂商的 137Cs 刻度** | 新发布、体量小、未经同行评审 |
| G4Radiacode（上轮已收录） | Radiacode CsI(Tl) 晶体的 Geant4 数字孪生 | — | **GPL-3.0**（本轮读过 LICENSE，确认） | GitHub | [G4Radiacode](https://github.com/lucamors/G4Radiacode) | 113/208/364 keV 和不同角度的响应矩阵 | 只作离线工具 |
| GADRAS / GADRAS-DRF（Sandia） | 计算 γ 和中子探测器输出的参考探测器响应函数代码 | v18.6 手册；v18.8 | **不开放**；2014 年 Sandia 资料称"将通过 RSICC 发布"，此前未在国家实验室以外提供 | 未核实 | [OSTI 1163837](https://www.osti.gov/biblio/1163837) | 参考方法 | 当前申请资格和出口管制状态未找到 |
| radioactivedecay（上轮已收录，许可更新） | 衰变链计算（如 225Ac → 221Fr → 213Bi 光子源项随时间变化） | PyPI v0.6.1（2025-01-09） | **MIT**（LICENSE："Copyright (c) 2020-2024 Japan Atomic Energy Agency & contributors"）；上轮写的 PyPI "Other/Proprietary" 分类可能对应内置数据 | PyPI | [GitHub](https://github.com/radioactivedecay/radioactivedecay) | 核数据 | 默认衰变数据集（是否为 ICRP 107）未核实 |
| IRD 碘剂量计（Hariri Tabrizi 等，MBEC 2020） | 用 MC 推导系统规格；用已知 131I 活度的模拟剂量刻度，用 MIRD 方法验证；探测器固定在离床一定距离处 | 体外评估 | 期刊 | 付费 | [doi:10.1007/s11517-020-02129-5](https://doi.org/10.1007/s11517-020-02129-5) | 读数反演（131I 的"以模拟定刻度"先例） | 【search-summary, verify】 |

| 数据集 | 内容 | 规模 | 许可证 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **22NRM07 GuideRadPROS WP3**（Vlahović、Siiskonen、Kržanović） | 个人和场所有源剂量计的性能测试：稳定性、过载、剂量率非线性、**响应随光子能量和入射角的变化** | 0.1 MB | **CC BY 4.0** | Zenodo（2026-01-02） | [doi:10.5281/zenodo.18133170](https://doi.org/10.5281/zenodo.18133170) | 验证（对照 EPD 的响应）；小探测器能量与角响应先验 | 未检查测了哪些设备，大概率不含 Radiacode |
| 合成城市 γ 能谱（Bandstra 等，RADAI 项目） | 模拟 2×4×16″ NaI(Tl) 在模拟城市中行驶的训练/验证/测试能谱，含 K/U/Th、137Cs、雨后氡子体、宇宙射线本底 | 2.3 GB（HDF5） | **CC0** | 公开 | [doi:10.7941/D1XC97](https://doi.org/10.7941/D1XC97) | 读数反演（本底模型、ML 预训练） | 大体积 NaI，不是 1 cm³ CsI |
| 现场 NaI(Tl) 能谱 + 实验室能谱 + PHITS 模拟（Huynh、Truong） | 关键词含角响应、全谱分析、PHITS | — | 元数据写 CC BY 4.0 | **受限访问** | [Zenodo 24126454](https://zenodo.org/records/24126454) | 读数反演（PHITS 探测器响应验证示例） | 文件不可见 |
| 紧凑 CsI(Tl)-SiPM γ 成像的复现数据与代码（Alharbi） | — | — | 元数据写 CC BY 4.0 | **受限访问** | [Zenodo 23580810](https://zenodo.org/records/23580810) | 读数反演（紧凑 CsI 响应） | 文件和说明不可见 |
| Radiacode 社区能谱 | App 把能谱、频谱图和轨迹存在本地"库"中，可导出 .csv/.xml；官网有同位素能谱库；论坛有"Spectrogram Library" | — | 未声明 | App 导出 | [导出说明](https://radiacode.com/docs/en/software/android/spectrum/export-and-import-of-spectra)；[论坛](https://space.radiacode.com/spectrogram-library-78) | 读数反演（合理性检查用能谱） | **没有成批、带许可的归档**；Zenodo 检索"radiacode"只有 4 项野外巡测 |
| Radiacode 100 系列厂商参数 | CsI(Tl) 约 1 cm³，137Cs 处 FWHM 9.5%，30 cps/(µSv/h)；称立方晶体使灵敏度与朝向无关 | — | — | — | [产品页](https://radiacode.com/100-series?lang=en) | 读数反演的角响应先验 | 营销说法；只有一篇博客评测估计 FWHM 7–10%（[tspi.at](https://www.tspi.at/2025/06/14/radiacode102.html)） |

### MC 代码许可与点核代码：MCNP 受出口管制，MC-GPU 属于公共领域

上轮标为【未核实】的几款 MC 代码，本轮基本查清。对 IAEA TC 项目最要紧的是 **MCNP6.3 源码受出口管制**：RSICC 的 MCNP 6.3.1 公告据称只向美国人员和 10 CFR 810 附录 A 所列国家的公民分发【search-summary, verify】，许多 TC 受援国因此难以使用。笔记的结论是，可以跨 TC 国家移植的 MC 组合仍然是 GATE/Geant4 + OpenMC + PHITS，MC-GPU（公共领域）和 SIMIND（免费、仅限研究）作为可选补充【推断】。点核方面，现在有一小批开放的 Python 代码，但**没有一款能把人体当作分布式、自吸收的源来建模**，所以半解析引擎仍要自己写；体内自吸收更适合用 MC 训练数据推导的衰减因子表，而不是水中的 G-P 积累因子【推断】。

| 代码 | 状态 | 许可证/商用 | 获取方式 | 链接 | IRSA 用途 | 注意事项 |
|---|---|---|---|---|---|---|
| **MC-GPU**（FDA/CDRH/OSEL/DIDSR，A. Badal） | 投影成像、CBCT、乳腺摄影/DBT 等多个版本 | **公共领域**（17 U.S.C. §105）；免责声明允许免费使用、修改和销售，衍生作品须附声明 | GitHub | [DIDSR/MCGPU](https://github.com/DIDSR/MCGPU) | MC 训练（GPU 光子输运，可改造用于快速外照射注量） | 为 X 射线成像能量和几何而建；改造用于放射性核素源只是笔记的建议，还没有人做过 |
| **SIMIND**（Lund，M. Ljungberg） | 下载页列出 v9.0（标注 "08/10/2026"）和 v8.0，有 Windows、macOS（Intel/arm64）、Linux 版 | 没有命名许可；免责声明称"仅用于研究、教育和培训"，未经临床验证；**商用未提及** | 免费下载 | [SIMIND 下载](https://www.msf.lu.se/en/research/simind-monte-carlo-program/downloads) | 读数反演的 SPECT 先验；MC 训练 | 日期格式有歧义（10 月 8 日还是 8 月 10 日）；产品管线使用前需问 Lund |
| **FLUKA**（CERN & INFN） | 注册需审批（最多两个工作日）；个人可把二进制库用于学术或教育；源码只提供给持有 CERN 机构许可的学术机构用户 | 旧版许可：内部科研、非商用、**非军事**用途免版税；商用另有试用和商业许可 | fluka.cern 注册 | [注册 FAQ](https://fluka-forum.web.cern.ch/t/important-registration-and-package-download-faq/3857) | 剂量引擎 MC 基准 | 现行许可页未读到 |
| **MCNP6.3**（LANL；RSICC CCC-870） | 两个包：源码 + 可执行文件 + 文档，或仅可执行文件 | RSICC 许可；**源码受出口管制**，申请需说明理由 | RSICC 申请 | [RSICC CCC-870](https://rsicc-docs.ornl.gov/abstract/M/MCNP--18.html) | 剂量引擎 MC 基准；PiMAL 需要它 | 分发对象限制见上【search-summary, verify】 |
| **PENELOPE 2018**（NEA-1525）/ **penEasy** | PENELOPE 经 NEA 数据库分发；penEasy v2024-03-13 可从 UPC 下载 | penEasy 下载页**没有写许可证**；PENELOPE 按 NEA 数据库规则（成员国） | UPC zip；NEA 申请 | [penEasy](https://inte.upc.edu/en/downloads/peneasy)；[NEA-1525](https://www.oecd-nea.org/tools/abstract/detail/nea-1525) | MC 基准 | 需查看 zip 内的许可证 |

| 点核代码 | 功能 | 许可证/商用 | 获取方式 | 链接 | IRSA 用途 | 注意事项 |
|---|---|---|---|---|---|---|
| **rad_point_kernel**（fusion-energy） | 中子和光子点核屏蔽：透射比、注量、有效剂量；单层和多层；MC 推导的积累修正；Shin–Ishii 等解析拟合 | **MIT**（© 2026） | `pip install rad_point_kernel`，PyPI v1.8.0（2026-05-01） | [GitHub](https://github.com/fusion-energy/rad_point_kernel) | 半解析引擎（患者与接收者之间的墙、门、铅围裙等屏蔽） | 平板几何，不处理体内分布源 |
| **ZapMeNot**（A. Ford） | 点源 + 半无限平板屏蔽；基准结果打印与 MicroShield 的百分比差 | **GPL-3.0** | GitHub（alpha，无发布版），不在 PyPI | [ZapMeNot](https://github.com/AlanFord/ZapMeNot) | 半解析代码的 MicroShield 回归测试 | 只作测试工具，不链接进产品；积累方法未说明 |
| **pyshield-smr**（A. Allan） | 一维平板点核，Taylor 两项积累（"ANS-6.4.3 风格"）、XCOM 衰减、ICRP 74 H*(10)；带数据哈希的 QA 清单，README 称 254 项测试通过 | **MIT** | GitHub | [pyshield-smr](https://github.com/theonlyallana/pyshield-smr) | 参考实现模式（数据哈希、H*(10) 系数） | 自称"不是持证安全论证工具"；系数出处未核实 |
| POKER（Chen 等，NET 2025） | 三维点核辐射场表征，支持分层和深穿透屏蔽 | 期刊开放获取（许可未查） | 代码可得性未确认 | [ScienceDirect](https://www.sciencedirect.com/science/article/pii/S1738573324006612) | 半解析设计参考 | — |
| 3DGShield | 任意源和屏蔽几何的三维点核，Python 2.7 | 未说明 | 未确认 | [ResearchGate](https://www.researchgate.net/publication/351397786_3DGShield_A_new_3D_gamma_ray_shielding_code_for_arbitrary_source_and_shield_geometry_based_on_point_kernel_technique) | 参考 | Python 版本老旧 |
| MIGSHIELD / GPU 点核 | 交互式和 GPU 点核包，面向虚拟环境 | 未说明 | 未确认 | [ResearchGate](https://www.researchgate.net/publication/338250080_MIGSHIELD_A_new_model-based_interactive_point_kernel_gamma_ray_shielding_package_for_virtual_environment) | 实时剂量可视化思路 | 【search-summary, verify】 |
| PyPI 上的 `pyshield` | **与本领域无关**：一个 Python 代码混淆器 | MIT | — | [PyPI](https://pypi.org/project/pyshield/) | 避免误装 | 重名 |

### AI 代理模型：多跑嘈杂场景胜过少跑收敛场景，但没有一篇放出代码

2025–2026 年其他领域有几项相关的代理模型工作，**全部没有发布代码或训练数据**，也没有一篇针对核医学患者对旁人的外照射。最有用的是方法上的启示。PTNO 给出一条可以直接写进阶段 1 计划的预算规则；Choi 2026 的点源 → 器官剂量 DNN 覆盖了问题的"接收者一半"，可以和"活度分布 → 光子场"的患者一半串联起来，或者端到端训练一个代理模型【推断】。笔记据此提出的 MC 训练配方是：在（生物分布、BMI、姿态、接收者位置）空间上跑大量低统计 GATE 运行；在对数空间中学习单位活度剂量率；用 POD/PCA 基线和半解析模型作竞争预测器；在留出的收敛运行和实测上做保形校准，以 Li 2024 的 rRMSD 约 0.2 作现实性检查。外推到训练能量范围以外是已知弱点，**225Ac（218/440 keV）应当有自己的 MC 数据**，不要从 177Lu/131I 外推【推断】。

| 引用 | 方法与关键发现 | 代码/数据 | 对 IRSA 的用途 |
|---|---|---|---|
| Cao 等，"PTNO"，arXiv 2609.40090（2026-09-30），CC BY 4.0 | 神经算子直接用**嘈杂、低成本的 MC 标签**学习粒子输运解算子。MC 标签无偏，因此平方损失与收敛解上的损失有相同的最小值点。对场景数 M、每次渲染样本数 N、每场景渲染次数 K 的预算研究显示，**多而嘈杂的场景胜过少而收敛的场景**；能处理跨数量级的高动态范围 | 摘要未说明 | 阶段 1 的 MC 预算规则：把 GATE 算力分给大量低统计的患者/接收者构型；用对数目标处理接触距离到 3 m 的动态范围（[arXiv](https://arxiv.org/abs/2609.40090)） |
| Choi 等，NET 2026;58:104579 | 约 **54,000 个 Geant4 病例**训练的 DNN 预测点源照射下的器官和有效剂量；**5,000 例用时 0.76 s**；残差部分与 MC 统计噪声有关【search-summary, verify；全文 403】 | 未找到 | 接收者一侧的代理模板（[ScienceDirect](https://www.sciencedirect.com/science/article/pii/S1738573326004675)） |
| Pal Chowdhury 等，NIMB 2026;570:165909 | CNN 模拟 FRIB 屏蔽中的中子衰减，几乎即时输出出射注量和有效剂量率；屏蔽优化"几分钟"完成，精度"合理" | OSTI 页面无代码链接 | 附加屏蔽层的代理模式（[OSTI 3001618](https://www.osti.gov/pages/biblio/3001618-surrogate-modeling-monte-carlo-radiation-transport-convolutional-neural-networks-shielding-optimization)） |
| 归属不明的 XGBoost 代理（R² = 0.995，乘性误差约 4.8%，留一能量外推到未见低能时"更有挑战"） | 出现在一条指向 NIMB 摘要页的检索片段中，但 Pal Chowdhury 的 OSTI 记录只提到 CNN | — | 支持留一核素/留一能量的保形测试；**归属未核实**（[ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S0168583X2500299X)） |
| Son、Tan、Zhang，arXiv 2606.27624（2026-06-26） | 物理引导的 ML 用"物理启发模型张量"表示未知障碍物下的衰减 γ 注量；在随机域中用 MC 输运评估 | 未说明 | 读数反演的类比：沿任意路径的读数、未知衰减（患者身体）下反推源强（[arXiv](https://arxiv.org/abs/2606.27624)） |
| Tan、Son、Zhang，arXiv 2505.07088（2025-05-11） | 归一化 POD 降阶重建移动探测器数据的 γ 场，并自适应选测点 | 未说明 | 按姿态和生物分布建立患者剂量场 POD 基，作为便宜、可解释的代理基线（[arXiv](https://arxiv.org/abs/2505.07088)） |
| BNCT 治疗室门屏蔽 ANN（2024）；FLUKA 训练的 60Co 辐照器 ANN（Radiation 2026）；PWR 严重事故有效剂量 DNN（2025） | 屏蔽设计与源构型 → 剂量场的代理先例 | 未找到 | 结构与 IRSA 的"器官分布 → 剂量场"相同（[PMC11208447](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11208447/)；[doi:10.3390/radiation6030024](https://doi.org/10.3390/radiation6030024)） |
| 核系统代理模型与数字孪生综述（arXiv 2210.00074） | 训练样本不足是核系统深度代理模型的主要障碍 | — | 支持 PTNO 式"多而嘈杂"的策略（[arXiv](https://arxiv.org/pdf/2210.00074)） |

## 影像与数字孪生：第一例开放 177Lu-PSMA 患者投影出现，131I 和 225Ac 患者影像仍为零

### 新增影像数据：CC BY 的 177Lu 积木变多了，商用比 Deep Blue 宽松

上轮报告中唯一的开放 177Lu 多时间点患者数据是 Deep Blue 的 2 例 177Lu-DOTATATE，许可证是 CC BY-NC。本轮新增的几项都是 **CC BY 4.0，带署名即可商用**。其中最有用的三块是：一名 177Lu-PSMA-617 患者的单次采集（真实 PSMA 摄取模式和真实投影）；MRTDosimetry 六时间点体模系列（已知真值的动力学）；177Lu 和 225Ac 的重复体模扫描（噪声模型）（[zenodo.14172228](https://doi.org/10.5281/zenodo.14172228)；[zenodo.18891044](https://doi.org/10.5281/zenodo.18891044)；[zenodo.14226289](https://doi.org/10.5281/zenodo.14226289)）。MRTDosimetry 在 1–144 h 的平面像，是"单探测器读数 → 滞留活度"反演最干净的开放试验台，因为每个时间点的真值活度都已知。笔记据此提出一条完整链路：t 时刻活度 → 模拟佩戴读数 → 反演 → 有效半衰期，每一步都有真值，许可证为 CC BY【推断】。SSTR 方面，开放路线只剩模拟：Lund 的 XCAT 68Ga-SSTR/177Lu-PRRT 模拟影像（无许可证，需先询问），或上轮收录的 DblurDoseNet 14 例 PET 衍生体模（CC BY-NC）。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **SlicerSPECTRecon Tutorial Data**（Polson，2024） | (i) **一名接受 177Lu-PSMA-617 治疗的患者**的 DICOM：2 个床位的 SPECT 投影 + CT（304 个 DICOM 文件）；(ii) 灌注 177Lu 的 Jaszczak 体模的 SIMIND 模拟（3 个能窗 + 衰减图） | 95.9 MB zip，319 个条目 | **CC BY 4.0** | Zenodo | [doi:10.5281/zenodo.14172228](https://doi.org/10.5281/zenodo.14172228) | 读数反演的活度分布先验（重建真实 177Lu-PSMA 分布）；MC 源分布 | **单患者、单时间点**，没有动力学；采集时间未在元数据中说明 |
| **PyTomography SPECT Tutorial data v2.1**（Polson；Zubal 部分由 H. Wei，KCL） | `SPECT.zip` 含：`Lu177-PSMA-GEDisc` 患者（双床位投影、304 层 CT、分割、重建）；`Lu177-NEMA-SymT2`；`Ac225-NEMA-SymT2`（投影、CT、PSF 算子）；`SIMIND-Jaszak`（lu177、ac225、**fr221、bi213、tl209**、pb212、bi212、tl208、y90 及衰减图）；Tc-99m 心脏与 NEMA。另有 Zubal 体模的模拟 177Lu-DOTATATE SPECT 及模拟 68Ga-DOTATATE PET 先验 | 1,455 MB，7 个文件 | **CC BY 4.0** | Zenodo（v2.1，2026-11-07） | [doi:10.5281/zenodo.24040979](https://doi.org/10.5281/zenodo.24040979) | 活度分布先验；MC 训练；225Ac 子体能窗建模 | `Lu177-PSMA-GEDisc` 患者看来与 SlicerSPECTRecon 是同一人（都是 304 个 CT 文件、两个床位，笔记推断），**不要重复计数**；没有 I-131 文件夹 |
| **53 Repeated Acquisitions of a Lu-177 NEMA Phantom**（Polson，2024） | 53 次重复 SPECT 采集；6 个球（37–10 mm），10:1，初始 0.89 MBq/mL；128×128、4.8 mm，96 投影 × 15 s，加 CT | 128.8 MB | **CC BY 4.0** | Zenodo | [doi:10.5281/zenodo.14226289](https://doi.org/10.5281/zenodo.14226289) | 不确定性（噪声与重复性） | 重复之间的时间间隔未说明 |
| **Ac-225 Scans: 34 Repeated Phantom Acquisitions**（Polson，2024） | 圆柱体模，3 个球（60/28/22 mm），10:1，初始 1.37 kBq/mL；96 投影 × 60 s，加 CT | 88.6 MB | **CC BY 4.0** | Zenodo | [doi:10.5281/zenodo.14166469](https://doi.org/10.5281/zenodo.14166469) | 225Ac 附加模块（低计数统计与能窗选择） | 只有体模 |
| **MRTDosimetry 四器官体模影像**（Robinson、Denis-Bacelar、Tran-Gia、Tipping、Bardiès、Kayal） | 灌注 177Lu 的四器官体模在 **1、4、24、40、72、144 h** 的 GE DICOM：CT、平面像、原始 SPECT、重建 SPECT | 172.3 MB zip，875 个条目 | **CC BY 4.0** | Zenodo（OSF 69NGE 的镜像） | [doi:10.5281/zenodo.18891044](https://doi.org/10.5281/zenodo.18891044)；论文 [doi:10.1016/j.ejmp.2023.102583](https://doi.org/10.1016/j.ejmp.2023.102583) | **估计器验证（已知真值的多时间点序列；平面像可代表全身滞留）**、读数反演、验证 | 上传者注明需用 7zip 解压，不要用 unzip |
| 177Lu 临床前 SPECT 方案影像（Tosoni，2026） | EJNMMI Physics 一篇临床前 SPECT 优化论文的影像 | 323.7 MB | CC BY 4.0 | Zenodo | [doi:10.5281/zenodo.19336881](https://doi.org/10.5281/zenodo.19336881) | 边缘 | 只有临床前数据 |
| DPM-RPT 与 GATE MC 剂量学对比数据（Gupta & Dewaraja，Deep Blue 2026） | 8 例患者中 4 例的影像，用于比较"四种疗法"中 DPM-RPT 与 GATE 的体素剂量学；SNMMI 2026 摘要，全文在准备中 | 未说明 | **DataCite 元数据中没有许可证**；Dewaraja 的其他 Deep Blue 数据集为 CC BY-NC 4.0 | Deep Blue | [doi:10.7302/me7k-ph50](https://doi.org/10.7302/me7k-ph50) | MC 训练、验证（MC 交叉核对） | 元数据没有写明是哪四种疗法 |
| **Lund XCAT 模拟 177Lu-DOTATATE SPECT 与 68Ga-SSTR PET 影像**（Gustafsson 等） | 改进 XCAT 患者模型上的 MC 模拟影像 | 未说明 | **无许可证**："按原样提供，不作任何保证"，要求引用论文 | 公开链接（lu.box.com） | [Lund 资源页](https://www.msf.lu.se/en/research/resources)；[论文 doi:10.1186/s40658-026-00899-9](https://doi.org/10.1186/s40658-026-00899-9) | 活度分布先验、MC 训练（逼真的模拟 SSTR 队列） | 文件未打开；再分发前询问作者 |
| lu177-resolution-dosimetry（Ogino/Daisaki 2026 分析代码） | 由 MRTDosimetry 数据和派生数据复现全部图表 | 0.2 MB | MIT | Zenodo | [doi:10.5281/zenodo.24355541](https://doi.org/10.5281/zenodo.24355541) | 读数反演、MC 训练 | 记录写明代码在 LLM 协助下编写、由作者验证 |
| simind-python-connector v2.0.2（Porter、Varzakis、Thielemans） | SIMIND 的 Python 驱动，带 STIR/SIRF 和 PyTomography 适配器 | 2.1 MB | **Apache-2.0**（LICENSE 文件）；GitHub API 显示 NOASSERTION（冲突） | Zenodo/GitHub | [doi:10.5281/zenodo.24110015](https://doi.org/10.5281/zenodo.24110015) | MC 训练（脚本化虚拟 SPECT） | SIMIND 需另行安装 |
| YazdanSalimi/simind-pipeline | "完整 SPECT 模拟管线" | — | MIT（GitHub API） | GitHub | [repo](https://github.com/YazdanSalimi/simind-pipeline) | MC 训练 | 内容未检查 |
| Ennassiri & Touti 2026 数据（游离 221Fr 抬高 225Ac 定量 SPECT 活度估计） | 能窗响应矩阵与原始 MC 计数（OpenGATE 10.0.3，Siemens Intevo，ME/HE 准直器） | 可见文件 0 个 | 声明 CC BY 4.0，但**访问"受限"** | 需申请 | [doi:10.5281/zenodo.22055021](https://doi.org/10.5281/zenodo.22055021) | 225Ac 子体平衡修正 | 不申请无法下载 |

| 检索或条目 | 结果 | 链接 | 注意事项 |
|---|---|---|---|
| TCIA PT 序列清单（NBIA API，2026-11-23） | 44 个集合共 6,140 个 PT 序列；唯一的 PSMA 集合是 PSMA-PET-CT-Lesions（597）；**没有 SSTR/DOTATATE 集合** | [NBIA API](https://services.cancerimagingarchive.net/nbia-api/services/v1/getSeries?Modality=PT) | 确认上轮缺口 |
| MSKCC DOTATATE PET/CT 神经内分泌肿瘤分割 nnU-Net 模型 | 用 915 例 DOTATATE PET/CT 训练；**影像不公开**；模型标为 CC BY 4.0 | [MSK 数据目录 11202](https://datacatalog.mskcc.org/dataset/11202) | 【search-level, verify】；只能当分割工具，不是数据 |
| 131I 影像检索（DataCite、Zenodo、Deep Blue） | DataCite 12 条命中，都不是患者影像；Deep Blue 19 条记录只有 Lu-177 和 Y-90 | [DataCite 查询](https://api.datacite.org/dois?query=titles.title:(131I%20OR%20I-131%20OR%20radioiodine)) | **131I 患者影像仍为零** |
| Hugging Face（SPECT、PSMA、lutetium、nuclear medicine、PET/CT、dosimetry） | 没有 RPT 专用数据，只有通用 PET/CT 集（如 `dacthai2807/pet-ct`，CC BY-NC-SA 4.0） | [HF API](https://huggingface.co/api/datasets?search=PET/CT) | 对 IRSA 无用 |
| IAEA CRP E23005"分子放疗剂量学"（2017–2024）；后续 CRP E24027"RPT 审核体系"（2025-04-25 启动） | **没有找到可下载数据** | [E23005](https://iaea.org/projects/crp/e23005)；[E24027 新闻](https://www.iaea.org/newscenter/news/new-crp-audit-system-for-radiopharmaceutical-therapy) | 【search-level, verify】；IAEA 页面抓取返回 HTTP 402 |
| 意大利 ENEA-INMRI ILC-2 全国比对；荷比德 8 家医院 32 台活度计比对 | 前者只有 8 家报告了 177Lu；后者显示随系统、容器和几何差异很大；均无公开数据 | [Spagnuolo 2024](https://iris.uniroma1.it/retrieve/9dd0f962-30a2-4e05-be1f-2d9df56b79bd/Spagnuolo_National%20programme_2024.pdf)；[EJNMMI Phys](https://link.springer.com/10.1186/s40658-018-0213-z) | 【search-level, verify】 |
| EURADOS WG13 177Lu-PSMA-617 案例（Covens，EFOMP SMRD 2025 幻灯片） | 1 m 处周围剂量当量率实测与模拟对比；幻灯片称近距离姿态对结果影响很大 | [Covens 幻灯片](https://smrd2025.efomp.org/wp-content/uploads/2025/12/Covens.pdf) | 【search-level, verify】；没有同行评审的 177Lu 产出 |
| 未有效检索的仓库 | Kaggle（API 需认证）、Synapse（未查询）；grand-challenge.org 和 PhysioNet 的 API 忽略关键词参数 | — | 列入下一步检索清单 |

### 数字孪生与虚拟试验代码：PyTheraTwin 最接近，但依赖链卡在两处许可

QURIT/Rahmim–Uribe 课题组现在有两个公开仓库。**PyTheraTwin（MIT，2026-11-17 推送）**是一条端到端的"治疗诊断数字孪生"管线：CT → TotalSegmentator → PBPK 时间活度曲线 → 合成病灶 → SIMIND SPECT + PyTomography 重建 → OpenGATE 剂量学（[PyTheraTwin](https://github.com/qurit/PyTheraTwin)）。它是 IRSA 合成轨迹需求（每个虚拟患者的器官 TAC 和配对影像）最接近的开放实现。但商用链条被两个依赖卡住：PBPK 引擎 PyCNO 没有许可证，SIMIND 带"仅限研究"免责声明。TC 项目原型现在就可以用；任何要分发的东西，都要用 tellurium 或 PySB 按已发表的 Kletting/Fele-Paranj 方程重写 PBPK，或取得 QURIT 的书面许可【推断】。**没有任何课题组发布合成队列**：640 名虚拟患者的 PBPK 研究只有预印本；一个独立的 MIT 仓库用公开 PET/CT 建了 PSMA 虚拟队列，但只发布了网络权重（CC BY-NC）。这些数字孪生也都没有模拟 IRSA 需要的全身层面：出院后的滞留活度、膀胱充盈与排尿、体外探测器信号。这一层仍要自己加，例如在 PyCNO/Kletting 的 TAC 上叠加带排尿事件的膀胱房室，再接一个到躯干佩戴探测器的正向模型【推断】。samkiwi18 的结果（一次晚期 SPECT 加贝叶斯更新优于网络先验）与 IRSA"群体先验 + 校准的 1 m 读数 + 可穿戴更新"的设计同构，它的 MIT 代码可以作为更新步骤的模板【推断】。

| 名称 | 功能 | 许可证/商用 | 状态 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|
| **qurit/PyTheraTwin** | 结合 CT 解剖/分割、PBPK 动力学和基于物理的 SPECT 模拟/重建，生成患者特异的治疗诊断数字孪生，用于"虚拟治疗诊断试验"；有浏览器 UI 和 CLI | **MIT**（"Copyright (c) 2026 Qurit"） | 2026-11-17 推送 | [GitHub](https://github.com/qurit/PyTheraTwin) | 合成轨迹（每个虚拟患者的器官 TAC）；活度分布先验、MC 训练（配对活度图与 SPECT）；MC 剂量 | 依赖 SIMIND（商用条款不清）和 PyCNO（无许可证）；代码中"bladder"只出现在 ROI 命名和选项 JSON 中，**是否有膀胱排尿模型未确认** |
| **qurit/PyCNO** | Fele-Paranj 2024 PBRPK PSMA 模型的 Python 实现，由 SimBiology 导出为 SBML：原始模型、精简关键器官模型、PK/PD 耦合模型 | **无许可证**（LICENSE 返回 404） | 2026-10-14 推送；README 警告会有破坏性变更 | [GitHub](https://github.com/qurit/PyCNO)；[模型论文](https://link.springer.com/article/10.1186/s41181-023-00236-w) | 动力学先验、合成轨迹（PSMA 全身 PBPK） | 默认版权；代码检索没有找到"bladder"或"urine"（索引可能跳过大型 SBML） |
| samkiwi18/lu177-psma-predictive-dosimetry | 用公开 PSMA PET/CT 加 **Kletting 2016 PBPK** 构建虚拟队列；GATE 10 MC 剂量；SwinUNETR 预测体素级动力学参数及不确定度；6 种单次 SPECT 贝叶斯更新方法；在 Deep Blue 的 2 例上测试。1–7 d 的一次 SPECT 把 TIA 中位绝对对数误差从 0.36 降到 0.14–0.18 | **代码 MIT；权重、数据和派生材料 CC BY-NC 4.0** | 2026-11-22 推送；3 次提交；无论文 | [GitHub](https://github.com/samkiwi18/lu177-psma-predictive-dosimetry) | 动力学先验（群体先验的贝叶斯更新逻辑可直接迁移）、不确定性 | 不含数据，只发布 `model.pth`；个人项目，未经评审 |
| Yenipertiwi16/VP-NLME-STP-Dosimetry | 177Lu-PSMA-617 单时间点剂量学的虚拟患者非线性混合效应建模（R + NONMEM） | **无许可证** | 2026-08-03 | [GitHub](https://github.com/Yenipertiwi16/VP-NLME-STP-Dosimetry) | 动力学先验（NLME 群体先验） | NONMEM 是商业软件 |
| UNHSAILLab/rl_for_theranostics | PBPK、神经 ODE/PINN 与 RL 笔记本，含 Lu-DOTATATE 计算机模拟分析和一份 UBC PSMA PBPK Matlab 模型副本 | 仓库 Apache-2.0 | 最后推送 2024-03-31 | [GitHub](https://github.com/UNHSAILLab/rl_for_theranostics) | 合成轨迹（玩具 PBPK） | **许可歧义**：捆绑的 UBC 模型原仓库没有许可证 |
| Borealis-BioModeling/radioligand-therapy-model | PySB 编写的半机制 RLT PK/PD 模型 | MIT | 2024-10-17 | [GitHub](https://github.com/Borealis-BioModeling/radioligand-therapy-model) | 合成轨迹 | 通用模型，未针对特定配体拟合 |
| **bertoletlab/topas-rpt** | OpenTOPAS 4.3.0 扩展：按正确时间结构抽样完整分支链的衰变，并把房室结合动力学引入源项；接受 DICOM 活度图 | **MIT**；Zenodo [10.5281/zenodo.22651399](https://doi.org/10.5281/zenodo.22651399) | 2026-11-13；锁定 Geant4 11.3.2 | [GitHub](https://github.com/bertoletlab/topas-rpt) | MC 剂量与训练（**带动力学的 225Ac 衰变链外照射场**） | 需给 OpenTOPAS 打补丁；方法论文针对 211At；225Ac 的验证未说明 |
| qurit/LesionStudio；qurit/xcat-sims | PET/SPECT 合成病灶插入；XCAT 淋巴系统扩展与 PET 模拟 | MIT；**无许可证**（XCAT 升级版向 Duke 申请） | 2026-07 / 2026-09 | [LesionStudio](https://github.com/qurit/LesionStudio)；[xcat-sims](https://github.com/qurit/xcat-sims) | MC 训练 | xcat-sims 面向 PET/淋巴瘤，不是 RPT |
| callmesobhan/rlt-sparse-dosimetry；RadioNuclide_Uncertainty_Lab；SJTUzhou 单时间点体素调整 | 稀疏测量下的 TIA 与红骨髓剂量学；MIRD 剂量学的 MC 传播与 Sobol 敏感性；单时间点剂量学调整 | MIT；Apache-2.0；MIT | 2026 | [repo 1](https://github.com/callmesobhan/rlt-sparse-dosimetry)；[repo 2](https://github.com/callmesobhan/RadioNuclide_Uncertainty_Lab)；[SJTUzhou](https://github.com/SJTUzhou/Voxel-wise-Adjustment-of-Single-Time-Point-Dosimetry-for-Lu177-PSMA-RPT) | 动力学先验、不确定性 | 自称"研究代码，寻求评审"；SJTUzhou 未检查 |
| lucasfffaria/Monte-Carlo-Dosimetry-of-Therapeutic-Radioligands | 描述为 177Lu、161Tb、212Pb、225Ac 患者周围 H*(10) 和有效剂量的 MC 评估 | 无许可证 | **2026-11-23 仓库为空** | [GitHub](https://github.com/lucasfffaria/Monte-Carlo-Dosimetry-of-Therapeutic-Radioligands) | 观察项（外照射，含 225Ac） | 目前没有可用内容 |
| Synthea（上轮已收录，本轮核查模块） | 有肺、乳腺、结直肠、退伍军人肺癌和前列腺癌模块，以及甲减模块；**没有任何核医学模块**；辐射内容只有外照射放疗 | Apache-2.0 | 2026-11-20 推送 | [modules](https://github.com/synthetichealth/synthea/tree/master/src/main/resources/modules) | 合成患者背景（年龄、BMI、肾功能代理指标、家庭） | 需用 Generic Module Framework 自写 RPT 模块，例如在退伍军人前列腺癌模块里加"177Lu-PSMA 周期"状态（按 Pluvicto 说明书每 6 周 1 次、共 6 次）【推断】 |

治疗诊断数字孪生（TDT）的方法文献在 2024–2026 年快速增加，但**没有一篇确认发布了代码或队列**：Abdollahi 等 Theranostics 2024 给出定义和路线图（[PMC11209714](https://pmc.ncbi.nlm.nih.gov/articles/PMC11209714)）；Zaidi 等 JNM 2025 讨论验证、确认与互操作（[PMC12051771](https://pmc.ncbi.nlm.nih.gov/articles/PMC12051771/)）；Strigari 等 JNM 2025 讨论伦理、法规和联邦学习（[JNM](https://jnm.snmjournals.org/content/66/5/748)）；Zaid 等 arXiv 2509.21697 讨论 PBPK 的 VVUQ（[arXiv](https://arxiv.org/abs/2509.21697)）。Abdollahi 等 arXiv 2510.21054 用 PBPK 模拟了 **640 名虚拟患者**、15,360 条肿瘤和器官 TAC，Cu-64 PSMA-617 PET 预测肿瘤剂量的 MAPE 低至 8%，但没有确认代码或数据是否发布（[arXiv](https://arxiv.org/abs/2510.21054)）。Glatting 谱系的 177Lu-DOTATATE PBPK（Stenvall 2024，12 例 NET 患者；Jiménez-Franco 2021）发表了方程和参数，但没有找到代码【search-level, verify】（[LUP](https://lup.lub.lu.se/record/059f169f-fa5b-4d3f-81b5-e6d4046bcf0a)；[JNM](https://jnm.snmjournals.org/content/62/1/92)）。DosiTest 计划向临床科室分发 GATE 生成的 177Lu SPECT/CT，但没有找到公开下载【search-level】（[PubMed 33957577](https://pubmed.ncbi.nlm.nih.gov/33957577/)）。

## 动力学与排泄：说明书和 de Bakker 2023 让排尿阶跃生成器有了可引用参数

### 软件栈：tellurium、PySB、Chi 可以嵌入 App，GPL 工具留在离线

本轮按许可证宽松程度把 PBPK 和房室建模软件排了一遍。**可以嵌入要分发的 App 的宽松组合是：tellurium（Apache-2.0）或 PySB（BSD-2）负责模型定义和模拟，Chi（BSD-3）负责层级贝叶斯先验和逐患者更新**（[tellurium](https://github.com/sys-bio/tellurium)；[PySB](https://github.com/pysb/pysb)；[Chi](https://github.com/DavAug/chi)）。Open Systems Pharmacology（PK-Sim/MoBi/OSPSuite-R）是 GPL-2.0 加澄清附录，mrgsolve 是 GPL（≥2），nlmixr2 和 rxode2 是 GPL（≥3），这些只适合作离线先验生成工具【推断】。现成的放射性药物模型仍然稀少：PSMA PBRPK 有 SBML/Python 版本但没有许可证；177Lu-DOTATATE 的 PBPK 发表了但没有代码；没有找到带子体房室的 225Ac PBPK。

| 工具 | 用途 | 许可证 | 版本/活跃度 | 链接 | IRSA 适配 |
|---|---|---|---|---|---|
| **PK-Sim / MoBi**（Open Systems Pharmacology） | 全身 PBPK；多尺度模型编辑 | **GPL-2.0 加澄清附录**（GitHub 显示 NOASSERTION） | 2026-11 推送 | [PK-Sim](https://github.com/Open-Systems-Pharmacology/PK-Sim)；[MoBi](https://github.com/Open-Systems-Pharmacology/MoBi) | 离线生成合成轨迹先验；只在分发代码时触发 copyleft |
| OSPSuite-R | OSP 模拟的 R 接口 | GPL-2（同一附录） | 2026-11-21 推送 | [GitHub](https://github.com/Open-Systems-Pharmacology/OSPSuite-R) | 批量虚拟人群运行 |
| **tellurium** | Python 中的 SBML/Antimony 建模（libRoadRunner 引擎） | **Apache-2.0** | PyPI 2.2.13.1 | [GitHub](https://github.com/sys-bio/tellurium) | 运行 PyCNO/Fele-Paranj SBML；搭建带排尿事件的膀胱房室模型 |
| libRoadRunner | 高性能 SBML 模拟器 | **未核实**（PyPI 许可字段为空） | PyPI 2.11.0 | [PyPI](https://pypi.org/project/libroadrunner/) | — |
| **PySB** | Python 规则式系统生物学建模 | **BSD-2-Clause** | PyPI 1.17.0 | [GitHub](https://github.com/pysb/pysb) | Borealis RLT PK/PD 模型用的就是它 |
| **Chi**（`chi-drm`） | Python 中的 PKPD 建模与模型引导精准给药（贝叶斯推断） | **BSD-3-Clause** | PyPI 1.0.3 | [GitHub](https://github.com/DavAug/chi) | 动力学先验（层级先验、逐患者贝叶斯更新） |
| mrgsolve | R 中的 ODE PopPK/PD 与 QSP 模拟 | GPL（≥2） | CRAN 2.0.1 | [GitHub](https://github.com/metrumresearchgroup/mrgsolve) | 快速虚拟人群模拟（离线） |
| nlmixr2 / rxode2 | R 中的非线性混合效应估计 / ODE 求解 | GPL（≥3） | CRAN 7.0.1 / 5.1.8 | [nlmixr2](https://github.com/nlmixr2/nlmixr2) | 拿到患者数据后拟合群体有效半衰期和房室先验（离线） |
| **PKPDsim**（InsightRX） | R 中的 ODE PK-PD 模拟 | **MIT** | CRAN 1.4.1 | [GitHub](https://github.com/InsightRX/PKPDsim) | mrgsolve 的宽松替代 |
| BioModels（EBI） | SBML 模型库 | — | — | [BioModels](https://www.ebi.ac.uk/biomodels/) | **未检索成功**（搜索 API 返回非 JSON） |

### 参数来源：从药品说明书到 de Bakker 的快/慢两组

**药品说明书给出了经监管审评的群体参数。** Lutathera 美国说明书（2024-10 修订）写明：清除率 4.5 L/h（CV 31%），终末半衰期 71 ± 28 h，"5 小时内累计排出 44%，24 小时内 58%，48 小时内 65%"，尿中放射性可检出长达 30 天（[Lutathera PI](https://www.novartis.com/us-en/sites/novartis_us/files/lutathera.pdf)）。Pluvicto 美国说明书（2026-07 修订）给出终末消除半衰期 41.6 h（CV 68.8%）和清除率 2.04 L/h（CV 31.5%），只说"主要经肾排出"，**没有尿排百分比**（[Pluvicto PI](https://www.novartis.com/us-en/sites/novartis_us/files/pluvicto.pdf)）。41.6 h 与上轮收录的 Kurth 有效半衰期 40.5 ± 9.6 h 一致（[Kurth 2018](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/)），CV 68.8% 支持较宽的对数正态先验【推断】。**de Bakker 2023（Radboud，CC BY 4.0）是 177Lu-PSMA-617 尿排动力学最好的新来源**：25 例患者（28 个周期）收集 24 h 尿液，另有 9 例在 7–42 d 的 35 份样本；前 20 h 的排出半时间总体 4.9 h，快排组（n = 19）4.4 h，慢排组（n = 6）8.4 h；预测 48 h 排出 67 ± 18%（快组 75 ± 10%，慢组 36 ± 10%）；eGFR 50–65 mL/min 比 >65 排得慢（p < 0.01）；尿中 177Lu 可测到第 18 天（[PMC10547615](https://pmc.ncbi.nlm.nih.gov/articles/PMC10547615/)）。笔记据此把排尿阶跃生成器的混合权重定为快组约 76%、慢组约 24%（25 例中 19 例，笔记计算）。再加上 Kurth 2018 的 PSMA-617 累计曲线和 Schütze/Monserrat Fuertes 的晚期有效半衰期，生成器可以完全用开放、可引用的汇总数据参数化；逐患者原始数据仍需向 de Bakker/Radboud、Kurth、Schütze、Bailey 索取【推断】。

| 药物 | 来源 | 关键内容 | 许可/获取 | 链接 | 注意事项 |
|---|---|---|---|---|---|
| 177Lu-DOTATATE（Lutathera） | 美国说明书（2024-10） | 累计尿排 5/24/48 h：**44%/58%/65%**；终末半衰期 71 ± 28 h；CL 4.5 L/h；14 天内排出 >99% | 公开说明书 | [Lutathera PI](https://www.novartis.com/us-en/sites/novartis_us/files/lutathera.pdf) | 只有群体均值 |
| 177Lu-PSMA-617（Pluvicto） | 美国说明书（2026-07） | 终末半衰期 **41.6 h（CV 68.8%）**；CL 2.04 L/h（CV 31.5%）；肌酐清除率下降时 AUC 上升；CLcr < 54 mL/min 未研究 | 公开说明书 | [Pluvicto PI](https://www.novartis.com/us-en/sites/novartis_us/files/pluvicto.pdf) | **没有尿排百分比**；欧盟 SmPC 未查 |
| 177Lu-PSMA-617 | **de Bakker 等，EJNMMI 2023;50(12):3572** | 快/慢两组；48 h 排出 75 ± 10% vs 36 ± 10%；eGFR 效应；尿中可测至第 18 天；0–8 h 污染的皮肤当量剂量 50–145 mSv | CC BY 4.0；逐患者曲线只在图中，数据"按合理请求提供" | [PMC10547615](https://pmc.ncbi.nlm.nih.gov/articles/PMC10547615/) | 给出排尿模型的双峰先验 |
| 177Lu-PSMA（PBPK） | Abdollahi、Fele-Paranj、Rahmim，Cancers 2024;16:3120 | PBPK 参数对 TAC 衍生量的影响 | 开放 | [doi:10.3390/cancers16183120](https://doi.org/10.3390/cancers16183120) | 参数表未提取 |
| 177Lu-PSMA-I&T（PSA 的 PopPK/PD）；177Lu/68Ga-HA-DOTATATE 半生理 PopPK；68Ga-(HA-)DOTATATE PBPK | EJNMMI 2024 附加文件 | 个体拟合结果、拟合优度图、患者信息表 | CC BY 4.0 | [figshare 26711045](https://doi.org/10.6084/m9.figshare.26711045)；[26612495](https://doi.org/10.6084/m9.figshare.26612495)；[26566100](https://doi.org/10.6084/m9.figshare.26566100) | 多为图，不是参数表；第一项是 PSA 动力学而非活度 |
| 177Lu-DOTATATE（PBPK） | Stenvall 2024；Jiménez-Franco 2021 | 全身 PBPK 结构与参数在论文中 | 期刊 | 见数字孪生一节 | 【search-level, verify】；没有代码 |
| **225Ac-PSMA-I&T（子体动力学）** | **Liubchenko 等，EJNMMI 2024;51(8):2504**（LMU） | 5 例，7.7 ± 0.2 MBq；24 h 和 48 h SPECT/CT，能窗 440 keV（213Bi）、218 keV（221Fr）、78 keV。尿中 213Bi/225Ac 比 0.98 ± 0.15（24 h）→ 1.08 ± 0.09（48 h）。肾有效半衰期 27 ± 6 h（213Bi）、24 ± 6 h（221Fr）、23 ± 7 h（78 keV），病灶约 38–39 h；肾内 213Bi 与 221Fr 差异显著（p = 0.0078） | CC BY 4.0；数据请联系通讯作者 | [PMC11178588](https://pmc.ncbi.nlm.nih.gov/articles/PMC11178588/) | 没有全身有效半衰期；检索摘要称肾剂量 0.17 ± 0.06 Sv/MBq（RBE 5）【search-level】 |
| 225Ac 载体（通用） | Yskaa91/AlphaRadPharmacokinetics | 225Ac 载体 PK 与剂量学的 R 程序 | GPL-3.0 | [GitHub](https://github.com/Yskaa91/AlphaRadPharmacokinetics) | 2017 年后未更新 |
| 131I-NaI | — | 除上轮的 ICRP 137 / Leggett 2010 外，**没有新的开放参数集** | — | — | 缺口 |

本轮还对开放仓库做了一次负面检查：DataCite 按标题检索（剂量率 + 患者 + 177Lu/131I）和（尿/排泄/滞留 + 177Lu/131I/225Ac），只返回上轮已知的 OSF 研究方案和 Maulidina 131I 数据（[DataCite 查询](https://api.datacite.org/dois?query=titles.title:(urine%20OR%20urinary%20OR%20excretion%20OR%20retention)%20AND%20titles.title:(177Lu%20OR%20lutetium%20OR%20131I%20OR%20radioiodine%20OR%20225Ac))）。一份 2025/26 年 EJNMMI Phys 红骨髓剂量学论文的 CC BY 附加文件看起来相关，打开后发现内容是按骨体积分数列出的能量吸收分数，不是患者 TAC（[figshare 31023349](https://doi.org/10.6084/m9.figshare.31023349)）。**逐患者的全身滞留、逐次排尿活度或连续 1 m 剂量率表，本轮一项都没有新找到。**

### 225Ac：外照射很小，污染和卫生才是主要问题

对 225Ac 附加模块来说，外照射问题不大：按 MHLW 的 Γ（约 0.031 µSv·m²/(MBq·h)）和 ≤10–20 MBq 的给药量，加上上轮收录的 225Ac-lintuzumab 实测值（1 m 处 0.3 µSv/h），**IRSA 对 225Ac 的主要功能应当是污染和卫生指导**，驱动因素是尿中 213Bi 过量和失禁【推断】。动力学先验可以借用 Liubchenko 的肾和病灶有效半衰期（约 24–39 h），辐射物理则按 9.9 d 的母体计算。佩戴式 γ 探测器看到的主要是 218 keV（221Fr）和 440 keV（213Bi）线，计数率非常低；Polson 的 225Ac 体模数据和 PyTomography 的子体投影，是测试计数统计和能窗选择仅有的开放数据【推断】。

| 条目 | 数据 | 许可/获取 | 链接 | IRSA 用途 | 注意事项 |
|---|---|---|---|---|---|
| [225Ac]Ac-SibuDAB，JNM 2025;66(8):1239 | 反冲 213Bi 迅速进入尿液；20 min 尿样中 213Bi 相对平衡值**净过量 281%**；血中未检出 213Bi | 期刊 | [PubMed 40473466](https://pubmed.ncbi.nlm.nih.gov/40473466/) | 卫生模块的尿污染源项 | 【search-level, verify】 |
| [225Ac]Ac-PSMA-I&T I 期，JNM 2026;67(7):1109 | 推荐 II 期剂量 10 MBq；剂量学基于全身平面像加血、尿样本 | 期刊 | [JNM](https://jnm.snmjournals.org/content/67/7/1109) | 若补充材料有尿表，可作动力学先验 | 【search-level, verify】；补充材料未查看 |
| 2026 年住院污染研究（225Ac/177Lu-PSMA-I&T 串联；225Ac-DOTATOC） | 病房擦拭：6 次治疗中 4 次有污染；治疗后 128 份样本中 **12 份（9.4%）**阳性，主要在地面和马桶座，其次是水龙头；尿失禁时风险更高 | 未确定 | 检索结果指向 [JNM node 128470](https://jnm.snmjournals.org/node/128470) | 225Ac 居家卫生指导 | **【search-level, verify】**；与 Dahlke 2025 摘要和"CLEAN-225Ac"的关系见矛盾清单 |
| MHLW 2024 研究报告（202421004A） | 模型输入：Ac-225 Γ 0.03059 µSv·m²/(MBq·h)；模型输出：照护者每次治疗约 0.32 mSv，公众约 0.16 mSv/年（PSMA-617） | 政府报告 | [MHLW 报告](https://mhlw-grants.niph.go.jp/system/files/report_pdf/202421004A-buntan2.pdf) | 225Ac 的 NRC 式基线 Γ | 【search-level, verify】；是模型结果，不是实测 |
| 225Ac 子体再分布（小鼠，PSMA-I&T） | MIRDcalc 分析：假设 221Fr 摄取和 213Bi 原位衰变时，未结合子体再分布使肾和唾液腺剂量分别增至 1.3 和 2.5 倍；另一项小鼠研究未发现肿瘤中子体再分布 | 期刊 | 经 [PMC11178588](https://pmc.ncbi.nlm.nih.gov/articles/PMC11178588/) 汇总 | 子体模型的不确定范围 | 临床前 |
| topas-rpt | 带时间分辨衰变链和结合动力学的 MC | MIT | [GitHub](https://github.com/bertoletlab/topas-rpt) | 模拟 225Ac 衰变链外照射场随时间的变化 | 225Ac 验证未说明 |
| Radboud [225Ac]Ac-DOTA-MSC3（临床前放射生物学） | 临床前 CAIX 肿瘤研究数据 | CC0 | [doi:10.34973/pzkn-kw44](https://doi.org/10.34973/pzkn-kw44) | 边缘 | 小鼠模型 |

## 家庭行为与仿真：CASAS 和南非 CC0 家庭数据可以商用，真实距离仍只有 UWB 小样本

### 智能家居与睡眠：如厕和卧床时刻有了大样本先验，但都是环境传感器

本轮最强的新开放资源是 **CASAS 在 Zenodo 上的 2025 年发布（CC BY 4.0）**：189 户社区家庭，2007–2024，PIR 和门磁传感器按房间命名（Bathroom、Bedroom、Bed），部分家庭有活动标签，并附**户型图**（[zenodo.15708568](https://zenodo.org/records/15708568)）。它和欧洲 4 国 62 名独居老人的智能家居数据（CC BY 4.0，359 天）都足够大，可以估计每晚去卫生间的次数和时刻（[zenodo.14243470](https://zenodo.org/records/14243470)）。患者何时下床排尿，同时关系到污染和探测器读数的阶跃。但这两套都是环境传感器，只能作先验，不能用来训练手机或可穿戴设备上的分类器【推断】。可穿戴方面的新开放数据只有 SPHERE 脚本化数据（腕部加速度计 + BLE RSSI 房间定位）和 Walch 睡眠数据；带明确如厕/淋浴标签的开放可穿戴数据集，仍然只有上轮收录的 ExtraSensory。**这些数据集都没有佩戴辐射探测器。**

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **CASAS 自由生活数据**（WSU） | 天花板 PIR + 门磁，按房间命名；部分家庭有外部标注的活动标签；**floorplans.zip** | **189 户，2007–2024**；3.0 GB | **CC BY 4.0** | 公开下载 | [Zenodo 15708568](https://zenodo.org/records/15708568) | 事件检测（卫生间/卧床时刻、夜间如厕）、户型、接触先验（多住户家庭共处时长） | Zenodo 页面没有列出标签词表；多住户时无法区分谁触发了传感器；美国住宅 |
| CASAS 原始数据（49 户）；Aruba/Cairo/Milan/Tulum 标注集；脚本化多住户；"分离与交织""有无活动错误"脚本集 | 无标注环境数据；4 户标注活动；多名住户分别或协作完成的脚本化 ADL；脚本化 ADL/iADL | 49 户 1.5 GB；4 户 33 MB；实验公寓 | **CC BY 4.0** | 公开 | [16343844](https://zenodo.org/records/16343844)；[17180309](https://zenodo.org/records/17180309)；[15713378](https://zenodo.org/records/15713378)；[15713190](https://zenodo.org/records/15713190) | 事件检测；接触先验（两人同屋） | 脚本化任务的卫生间内容少 |
| **SPHERE house scripted v2.0**（Bristol） | 腕部三轴加速度计经 BLE 传到室内接收器；**各接收器的 RSSI（室内定位）**；环境和视频衍生数据 | v2.0（2022-11-04） | **CC BY 4.0** | data.bris（本轮返回 503） | [data.bris](https://data.bris.ac.uk/data/dataset/2h0wyctxrd69j2oqccsi45hy1p) | 事件检测（腕部动作 + 房间位置）；接触先验（RSSI 判断同屋） | 单一房屋的脚本化数据；SPHERE 100 户研究的获取途径未找到 |
| **TIHM**（Surrey，Sci Data 2023） | 各房间 PIR + 门磁；**床垫下睡眠垫（睡眠、上下床）**，1 分钟采样；生理指标；标注的健康事件 | **56 名痴呆患者**，平均 50 天，共 2,803 天；6 MB | Zenodo API：**CC BY 4.0**；同一记录的检索摘要称条款限于研究和患者获益、**禁止商用**（**冲突**） | 公开（Zenodo + GitHub 笔记本） | [Zenodo 7622128](https://zenodo.org/records/7622128)；[PMC10492790](https://pmc.ncbi.nlm.nih.gov/articles/PMC10492790) | 事件检测（老年人离床、夜间活动）、验证 | 商用前通读 Zenodo 描述 |
| 欧洲 4 国智能家居环境数据 | PIR、门磁、温湿度、光照、CO2、VOC、声音（2–5 min）；含卫生间、卧室 | **62 名 65 岁以上独居者**（丹麦、葡萄牙、罗马尼亚、瑞士），359 天，516 MB | **CC BY 4.0** | 公开 | [Zenodo 14243470](https://zenodo.org/records/14243470) | 事件检测（独居老人如厕时刻先验） | **没有活动标签**；瑞士没有 PIR/门磁数据 |
| Orange4Home（Orange Labs/Inria） | 236 个数据源；20 类活动、493 个现场标注实例，含**"如厕""淋浴""小睡"** | 1 人，约 180 h（2017） | **未命名许可证**；须引用 Cumin 等 UCAmI 2017 | **邮件申请**（MySQL 导出） | [amiqual4home](https://amiqual4home.inria.fr/orange4home/) | 事件检测 | 单人；再分发条款未知 |
| ARAS；MARBLE；MuRAL；UCAmI Cup 2018 | 多住户环境传感器（ARAS 2 户各 1 个月；MARBLE 智能手表 + 环境，最多 4 人同时，无如厕/睡眠；MuRAL 带住户身份和自然语言注释；UCAmI 单人 24 类活动） | 小 | **均未声明许可证** | 公开或申请 | [ARAS](https://www.cmpe.boun.edu.tr/aras)；[MARBLE](https://doi.org/10.13130/RD_UNIMI/VGLD0Y)；[MuRAL](https://arxiv.org/abs/2504.20505)；[UJAmI](https://ceatic.ujaen.es/ujami/en/repository) | 接触先验（多住户归属）、事件检测 | 需作者确认 |
| Kasteren 合成智能家居数据 | 马尔可夫模型生成的日常活动序列（原始数据含睡眠、淋浴、如厕标签） | 1 MB | 合成版 **CC BY 4.0**；原始数据的托管和许可未找到 | Zenodo | [Zenodo 12686003](https://zenodo.org/records/12686003) | 场景引擎（活动序列种子） | 合成文件仍在完善中 |
| UCI 506；AgeInPlace-Synth；AgeSafe-SynthHome；2026 年老年居家护理多模态预印本 | 环境传感器 HAR；合成老年人智能家居；10 人约 1 个月多模态 | — | 未核实 | UCI；IEEE DataPort；Preprints | [UCI 506](https://archive.ics.uci.edu/dataset/506/human+activity+recognition+from+continuous+ambient+sensor+d)；[AgeInPlace](https://ieee-dataport.org/documents/ageinplace-synth-v1-calibrated-longitudinal-smart-home-sensor-dataset-older-adults-living) | 事件检测、场景引擎 | 只见检索结果 |
| Walch 等腕戴设备运动、心率与 PSG 睡眠标签 | Apple Watch 原始加速度 + PPG 心率，PSG 睡眠标签 | — | **Open Data Commons Attribution v1.0** | PhysioNet | [sleep-accel 1.0.0](https://physionet.org/content/sleep-accel/1.0.0/) | 事件检测（由腕部动作判断入睡、卧床） | — |
| DREAMT | Empatica E4 腕部信号 + PSG，专家睡眠分期 | v2.2.0 | **PhysioNet 受限健康数据许可 + DUA** | 需资质 | [PhysioNet DREAMT](https://physionet.org/content/dreamt/) | 事件检测 | 受限 |

"HomeSense"数据集本轮没有找到。

### 家庭接近度与测距：南非和肯尼亚是 CC0，但只有 1–1.5 m 阈值

SocioPatterns 现在托管了**三套非洲家庭数据**：南非 SA-S-HTS（88 户，CC0）、肯尼亚农村（5 户，CC0）、马拉维农村（86 人/28 户，CC BY-NC-SA）。南非 PHIRST 数据（60 户、307 人、3 轮各 10–14 天）放在 GitHub 上，但**没有许可证文件**。这些数据都用**约 1–1.5 m 的信号衰减阈值、20 s 分辨率**，不是连续距离；对 IRSA 来说，它们回答的是"白天在约 1.5 m 内待了多久"，对应"<1–1.5 m"暴露档。马拉维、肯尼亚和意大利婴儿家庭研究都没有夜间数据，所以同床时间仍要靠假设（例如上轮 Levart 2025 使用的 0.1 m × 8 h）【推断】。**笔记找到的家庭成员之间连续实测距离，只来自 UWB 可穿戴设备 TotTag**（1 Hz 测距，亲子场景），而它的数据许可证不明。2026 年新出的婚礼 UWB 徽章数据集是 CC BY 4.0，但阈值是 1.5 m，场景也不是家庭。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **南非家庭（SA-S-HTS，NICD + SocioPatterns）** | 家庭成员近距离接触：连续 20 s 区间内至少交换 1 个数据包（约 1.5 m 内） | **88 户**（Klerksdorp、Soweto），每户约 2 周，2020-11 至 2021-08；70,271 条边 | **CC0**；引用 Kleynhans 等 eLife 2023 | 公开（GitHub） | [SocioPatterns](https://sociopatterns.org/datasets/south-african-households-contact-networks/)；[Netzschleuder](https://networks.skewed.de/net/sp_south_african_households) | 接触先验（LMIC 城市家庭内按年龄的接触时长） | 跟踪的是有成人 SARS-CoV-2 指示病例的家庭，行为可能受疾病影响 |
| 肯尼亚农村家庭（Kilifi HDSS） | 白天 7:00–20:00 面对面接触；年龄、性别、时长 | 5 户，各连续 3 天（2012）；网络 **47 个节点**，32,643 次接触 | **CC0 1.0**（Netzschleuder 上游许可）；Figshare 补充 PDF 为 CC BY 4.0 | 公开 | [Netzschleuder](https://networks.skewed.de/net/sp_kenyan_households)；[Figshare](https://figshare.com/collections/Quantifying_social_contacts_in_a_household_setting_of_rural_Kenya_using_wearable_proximity_sensors/3697636/1) | 接触先验 | Figshare 描述写 **75 人**，网络文件只有 47 个节点（**未解决**）；没有夜间数据 |
| 马拉维农村村庄（Mdoliro，Dowa） | 衰减阈值 −75 dBm（约 1–1.5 m），20 s；夜间和传感器静止时段已删除 | **86 人/28 户**；2019-12 至 2020-01；102,293 条边 | **CC BY-NC-SA**（Netzschleuder 列 v3.0）→ 非商用、共享相同方式 | 公开下载 | [SocioPatterns](https://sociopatterns.org/datasets/contact-patterns-in-a-village-in-rural-malawi/) | 接触先验（LMIC 户内与户间时长） | **上轮缺口已解决**；没有夜间数据 |
| PHIRST SocioPatterns（南非 2018） | 可穿戴接近徽章，约 1.5 m，20 s；家庭类型、户主性别、个人年龄/性别/就业 | **60 户、307 人**；3 轮（2、4、6 月），各 10–14 天 | **仓库没有许可证文件**（默认保留所有权利）；论文 PLOS ONE 2026 | GitHub `Data/` | [GitHub](https://github.com/lorenzodallamico/PHIRST_SocioPatterns) | 接触先验（按家庭类型、儿童照护、季节） | 使用前向作者索取许可 |
| 意大利婴儿家庭（Ozella 2018） | RFID 面对面接触，约 1–1.5 m；排除夜间 | 16 户有 <6 个月婴儿的家庭，55 人；5,958 次接触 | **数据可得性未找到** | — | [PMC5991752](https://pmc.ncbi.nlm.nih.gov/articles/PMC5991752) | 接触先验（母婴接触占比、祖父母来访） | 婴儿接触主要来自母亲 |
| 里昂医院病房（Vanhems 2013） | 20 s 面对面接触，角色分护士/患者/医生/行政 | 46 名医护、29 名患者 | **CC BY-NC-SA 3.0** | 公开 | [Netzschleuder](https://networks.skewed.de/net/sp_hospital) | 接触先验（住院或门诊场景） | 非商用 |
| i-Bird 长期护理机构 | 患者与员工 RFID 接近 | 329 名患者、261 名员工 | 获取未确认 | — | [PMC5786108](https://pmc.ncbi.nlm.nih.gov/articles/PMC5786108) | 接触先验（养老院场景） | 可能受限 |
| **TotTag**（Vanderbilt/UCSD） | **UWB 飞行时间测距 + BLE 发现**，1 Hz；1/3/6 ft 处真值标准差约 0.3–1.6 ft；室内视距 20–30 m，可穿透石膏墙和人体 | 案例：2 个家庭、2 个周末日的 12 h 窗口；**没有哪对亲子在"触手可及"（≤3 ft）距离内的时间超过当天的 16%**；可行性预印本 125 个家庭、7 天 | 数据与脚本在 GitHub `vanderbiltsealab/tottag_methods`（**无许可证文件**）；文章再利用限"非商用学术研究"；固件 `lab11/socitrack` 为 **Apache 2.0** | GitHub | [Salo 2022, PMC8907335](https://pmc.ncbi.nlm.nih.gov/articles/PMC8907335)；[socitrack](https://github.com/lab11/socitrack) | **验证（唯一找到的家庭内连续实测距离）**、接触先验 | 研究对象是儿童而非患者；公开样本极小；洗澡时摘下 |
| SociTrack（Berkeley） | BLE + UWB 可穿戴；所有佩戴者之间的成对距离可达 **16 Hz**，99 百分位精度 **14.8 cm** | 系统论文 | 固件 Apache 2.0 | — | [Biri 等](https://lab11.eecs.berkeley.edu/content/pubs/biri20socitrack.pdf) | 照护者距离试点的硬件选项 | 没有公开数据集 |
| 婚礼鸡尾酒会接触网（2026） | **UWB** 徽章每 5 s 检测 1.5 m 内的其他徽章，合并为 10 s 区间；有关系组标签 | 95 名成人；7,213 次接触，2,760 对 | **CC BY 4.0** | 公开 | [Zenodo 20430824](https://zenodo.org/records/20430824) | 接触先验（"家庭聚会"场景中亲属与非亲属的接触时长） | 阈值数据；户外社交活动 |
| 住宅 UWB 信道冲激响应数据（Bocus & Piechocki 2022） | 被动（无设备）UWB CIR，用于家中定位和活动识别 | 约 1.6 h 标注 | 未查 | Figshare | [PMC9587989](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9587989/) | 事件检测（无设备房间活动）、户型 | 没有可穿戴、没有人际距离 |
| **LocoVR**（ICLR 2025） | VR 中采集的 **7,000 多条双人轨迹**，含社交运动行为 | 130 多个室内住宅环境 | **MIT** | GitHub | [LocoVR](https://github.com/kt2024-hal/LocoVR) | 场景引擎（双人户内距离轨迹）、户型 | VR，不是真实住宅；短距离目标导向行走，不是全天作息 |

### 合成人群、ABM 与具身模拟器：代码多数宽松，人形资产多数非商用

要搭一个可复用、商用安全的场景引擎，宽松选项有：**SPW 合成人群（CC BY 4.0 数据，含家庭、日常活动和地点）、IDM 的 SynthPops/Covasim/Starsim（MIT）、PopulationSim（BSD-3）、pyEpiabm（BSD-3）、Mesa（Apache 2.0）**。OpenABM-Covid19、JUNE、CovidSim 是 GPL-3；FRED v5 只限非商用；RTI 2010 年美国合成人群只对非商用免费。**这些流行病 ABM 都不建模距离**，只把接触表示为每天在家庭/工作/社区层中的成员关系，IRSA 必须在上面加一层距离和时长模型。笔记建议的组合是【推断】：SPW 或 SynthPops 生成家庭 → ATUS 式作息（上轮）→ 在 CASAS/Swiss Dwellings 户型上分配房间 → Mesa 智能体带坐标 → 输出每个家庭成员的距离-时间轨迹；Covasim/Starsim 提供户外层（工作、学校、社区、交通）而不带 copyleft 义务。谁和谁同床、每天在 <1 m 待多少小时，必须用 SocioPatterns 家庭时长和 Levart/NRC 数值来校准。

具身模拟器的代码大多宽松（VirtualHome MIT、Habitat-lab/sim MIT、BEHAVIOR-1K/OmniGibson MIT、iGibson MIT、ProcTHOR 和 AI2-THOR Apache 2.0），但**人形资产和动作数据多数是非商用**：Habitat 人形化身（CC BY-NC / NC-SA，互相冲突）、HSSD 场景（CC BY-NC 4.0）、BEHAVIOR 数据包（单独许可，加密）、AMASS（非商用科研）、BABEL（仅限学术）、HumanML3D（派生自 AMASS）。VirtualHome 是唯一围绕**多智能体家庭活动程序**构建的模拟器。这些工具都不能直接输出全天的家庭距离-时间轨迹，要输出，就得给两个智能体编写日常程序并记录位置，这是开发任务，不是现成数据【推断】。商用路线应选 VirtualHome 代码 + ProcTHOR/AI2-THOR 住宅，避开 HSSD、Habitat 人形、BEHAVIOR 资产和 AMASS/BABEL 派生数据。AMASS 类动捕数据更适合模拟躺、弯腰、坐如何改变探测器与源的几何，而不是作接触先验【推断】。

| 名称 | 类型 | 距离/时长 | 许可证 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|
| **SPW**（UVA Biocomplexity） | 合成人群：人口学属性、**组织成家庭**、**分配日常活动**和地点 | 无距离 | **CC BY 4.0** | [Zenodo 社区](https://zenodo.org/communities/synpop_world) | 场景引擎（家庭构成 + 作息） | 21 个 Zenodo 记录（美国州、印度邦）；模型较粗 |
| SPEW（CMU） | R 包，生成住户、学校、工作场所 | 无距离 | **未找到许可证文件** | [arXiv 1701.02383](https://arxiv.org/pdf/1701.02383) | 场景引擎 | 似已停止维护【推断】 |
| RTI 美国合成家庭人口（2010）/ RTI SynthPop | 地理定位的美国家庭和个人 | 无距离 | **非商用免费**；2019 版数据代码 CC BY-NC-SA 4.0 | [RTI](https://www.rti.org/impact/rti-us-synthetic-household-populationtm) | 场景引擎 | 只有美国 |
| **SynthPops**（IDM） | 带家庭、学校、工作场所接触层的合成人群 | 无距离 | **MIT** | [GitHub](https://github.com/InstituteforDiseaseModeling/synthpops) | 场景引擎 | 开发似停于 2021 年【推断】 |
| **PopulationSim**（ActivitySim） | 按控制总量做列表平衡的人口合成 | — | **BSD-3** | [GitHub](https://github.com/ActivitySim/populationsim) | 场景引擎 | 需要种子微观数据（如 IPUMS） |
| synthpop（R） | 敏感微观数据的合成版本（披露控制），不是空间家庭生成器 | — | GPL-2 \| GPL-3 | [CRAN](https://cran.r-project.org/web/packages/synthpop/) | 合成患者家庭调查微观数据以便共享 | 用途不同 |
| **Covasim** / **Starsim**（IDM） | 家庭、学校、工作、社区层 / 通用多疾病网络 ABM | 只有每层接触数 | **MIT** | [Covasim](https://github.com/InstituteforDiseaseModeling/covasim)；[Starsim](https://github.com/starsimhub/starsim) | 场景引擎（层结构） | Covasim 层列表来自检索摘要 |
| OpenABM-Covid19（Oxford BDI） | **家庭网络**（成员每天互动）、职业网络、随机网络（交通、购物、聚会） | 每日互动；无距离 | **GPL-3** | [GitHub](https://github.com/BDI-pathogens/OpenABM-Covid19) | 场景引擎（家庭/交通层映射到 IRSA 场景） | copyleft |
| JUNE（Durham/UCL） | 按普查构建的虚拟英格兰（约 5,300 万人）；家庭、学校、工作、**养老院** | 按时间步作息；无距离 | **GPL-3** | [GitHub](https://github.com/IDAS-Durham/JUNE) | 场景引擎（养老院和家庭场景） | 英国输入；copyleft |
| FRED v5（Pittsburgh） | 合成人群 ABM | 无距离 | **Pitt EULA：仅限非商用、教育或研究** | [GitHub](https://github.com/PublicHealthDynamicsLab/FRED) | 场景引擎 | 限制性强 |
| CovidSim（Imperial）/ **pyEpiabm**（Oxford） | 家庭、学校、工作、空间社区 / 其 Python 重写 | 无距离 | **GPL-3** / **BSD-3** | [covid-sim](https://github.com/mrc-ide/covid-sim)；[epiabm](https://github.com/SABS-R3-Epidemiology/epiabm) | 场景引擎 | — |
| **Mesa** | 通用 Python ABM 框架，带网格/连续空间 | 可建模距离 | **Apache 2.0** | [GitHub](https://github.com/projectmesa/mesa) | **场景引擎（带坐标的户内智能体模型）** | 模型要自己写 |
| AgentSense（2025）；MoSen（2021）；程序化家庭活动生成器（2023）；Patterns-of-Life（2026） | LLM 生成人设 → VirtualHome 作息（250 天、22 种户型）；由单人轨迹合成 2–5 人多住户行为；模板化家中运动；城市尺度 OSM 移动 | — | 未查 | [AgentSense](https://arxiv.org/pdf/2506.11773v1)；[MoSen](https://ar5iv.labs.arxiv.org/html/2101.00235)；[2311.15400](https://arxiv.org/abs/2311.15400v1)；[2601.01219](https://arxiv.org/pdf/2601.01219v2) | 场景引擎、事件检测 | 只见检索结果 |
| **VirtualHome**（MIT/Toronto） | "多智能体家庭模拟器"；活动写成程序；Unity 可执行文件 | 记录位置即可得距离 | 代码 **MIT**；论文记录 CC BY-NC-SA 4.0（仅论文） | [GitHub](https://github.com/xavierpuigf/virtualhome) | 场景引擎（两个智能体的脚本化作息 → 位置时间序列） | Unity 资产许可未确认 |
| Habitat 3.0（Meta） | 人形化身 + 机器人在家中 | — | habitat-lab/sim **MIT**；**habitat_humanoids**：YAML 卡写 **CC BY-NC-SA 4.0**，正文写化身 **CC BY-NC 4.0**（**冲突**），动作文件按 SMPL Body Motion File 许可；**HSSD 场景 CC BY-NC 4.0** | [habitat-lab](https://github.com/facebookresearch/habitat-lab)；[HF humanoids](https://huggingface.co/datasets/ai-habitat/habitat_humanoids) | 场景引擎、户型 | 化身和场景非商用 |
| BEHAVIOR-1K / OmniGibson；iGibson | 1,000 项家务任务；交互式家庭场景 | — | 代码 **MIT**；BEHAVIOR 数据包单独许可且**加密**（资产多来自 ShapeNet/TurboSquid） | [BEHAVIOR-1K](https://github.com/StanfordVL/BEHAVIOR-1K)；[iGibson](https://github.com/StanfordVL/iGibson) | 户型（带家具场景） | 需 NVIDIA Isaac Sim |
| **ProcTHOR / AI2-THOR**（AI2） | 程序化生成的住宅；智能体导航 | — | **Apache 2.0** | [procthor](https://github.com/allenai/procthor)；[ai2thor](https://github.com/allenai/ai2thor) | 户型（程序化带家具户型）、场景引擎 | 人形化身支持未确认 |
| HumanML3D；BABEL | 14,616 段动作与 44,970 条文本描述（源自 HumanAct12 + AMASS）；AMASS 帧级动作标签 | — | 仓库 MIT，底层动作继承 AMASS 条款【推断】；BABEL **仅限学术** | [HumanML3D](https://github.com/EricGuo5513/HumanML3D)；[BABEL](https://babel.is.tue.mpg.de/) | 事件检测（"躺在沙发上""坐在马桶上"等文本到动作） | 单人片段 |

### 照护者实测距离：只有一项研究在家中连续测了病人与照护者的距离

**只有 UC Berkeley 的痴呆研究在家中连续测量了病人与家庭照护者之间的距离**：27 对痴呆或 MCI 患者与同住家庭照护者，双方佩戴 BLE 手表（电池最长 4 个月），每户 3 个插电式蓝牙接收器，**逐秒测距 6 个月**。照护者与患者的距离从前 3 个月到后 3 个月增加（t = 20.67，p < 0.001），距离增加越多，照护者孤独感上升越多（r = 0.46，p = 0.03）。但论文没有报告绝对距离或近距离时长，数据可得性未说明（[PMC9771049](https://pmc.ncbi.nlm.nih.gov/articles/PMC9771049/)）。俄勒冈 EVALUATE-AD 用墙装 PIR 记录"同屋时间"，只发表了单例（照护者自报每周照护 106 h）（[PMC8272983](https://pmc.ncbi.nlm.nih.gov/articles/PMC8272983)）。癌症照护的传感研究（BESI-C、智能手机 GPS）没有测量接近度（[BESI-C 方案](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6928698/)；[JMIR Form Res 2025](https://formative.jmir.org/2025/1/e59161)）。**没有找到任何放射性药物治疗后用传感器测量照护者距离的研究。** 因此 IRSA 的行为校准仍要依靠上轮的 Levart 2025 日记和 NRC 场景值，以 SocioPatterns 家庭时长和 TotTag 的 ≤16% 上界作交叉核对【推断】。Berkeley 的设计（BLE 手表 + 少量插电接收器，给双方提供房间级位置）是可以直接复用的试点方案，再加 UWB（TotTag/SociTrack 类，固件 Apache-2.0）就能得到亚米级距离（见开发计划一节）。

## 对话层：日本 MHLW 通知是第一份可直接编码的开放法规文本，模拟器分数必须用真人校准

### 官方法规文本：英日两国可商用再分发，加拿大只限非商用，IAEA 仍需许可

上轮报告只确认了美国 NRC/eCFR 文本（公共领域）和少数 CC BY 指南可以放心入库。本轮新增了两个可以再分发、带署名即可商用的辖区。**日本 MHLW 通知《放射性医薬品を投与された患者の退出について》是本轮找到的最"可编码"的官方文本**：它按核素和适应证给出明确的活度阈值和剂量率阈值，有与学会方案挂钩的条件，还有一次新修订（新增 Lu-177 PSMA 1 m 处 16 µSv/h）。修订前后对照表就是一个现成的**规则版本漂移**测试用例，可以和上轮的 NRC Rev. 1 对 DG-8061 配对使用（[MHLW PDF](https://www.mhlw.go.jp/content/10800000/001527013.pdf)；[广岛市转发件](https://www.city.hiroshima.lg.jp/_res/projects/default_project/_page_/001/039/664/250926bessi.pdf)）。依据是日本著作权法第 13 条：国家机关的告示、训令、通达不受著作权保护（[NDL 讲义](https://ndl.go.jp/jp/library/supportvisual/docs/kouza/r7_kouza04-2.pdf)）；MHLW 网站内容按 PDL1.0 使用，须注明出处，修改处要标明，不得冒充政府作品（[MHLW 利用规约](https://www.mhlw.go.jp/chosakuken/index.html)）。**英国 ARSAC《Notes for Guidance》2026 年 7 月版适用 OGL v3.0**，表 7.1–7.4 给出治疗后避孕期、哺乳中断、有效暴露时间和最大有效半衰期（[GOV.UK](https://www.gov.uk/government/publications/arsac-notes-for-guidance)）。英国的出院/放行数值则在 IPEM MDGN 第 2 版里，那是商业出版物。加拿大没有数值放行阈值，靠持证方针对 5 mSv 照护者约束做剂量评估，所以加拿大辖区配置应调用剂量引擎计算照护者剂量，而不是查表。这正好可以作为 IRSA-Bench"辖区切换"的对照用例【推断】。

| 辖区 | 文件（本轮找到的现行版本） | 已核实的关键内容 | 许可/再利用 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|
| **日本** | MHLW 通知（H10.6.30 医薬安発第 70 号，最后修订 R3.8.19 医政地発 0819 第 1 号，合订本），加上新增 177Lu-PSMA 的修订（广岛市转发的新旧对照表，文件日期 2025-09-26） | 剂量约束：公众 1 mSv/年，照护者每次 5 mSv。按活度放行：Sr-89 200 MBq、**I-131 500 MBq**、Y-90 1,184 MBq。按剂量率放行：**I-131 1 m 处 30 µSv/h**。患者特异情形：按学会方案门诊消融 I-131 1,110 MBq；Ra-223 每次 12.1 MBq（每疗程 72.6）；**Lu-177 SSTR NET 1 m 处 18 µSv/h**（7.4 GBq，每 8 周 1 次 ×4）。放行记录保存 2 年。修订新增 **Lu-177 PSMA mCRPC 1 m 处 16 µSv/h**（7.4 GBq，每 6 周 1 次 ×6） | **著作权法第 13 条（通达不受著作权保护）+ PDL1.0**；可商用，注明出处 | [MHLW PDF 001527013](https://www.mhlw.go.jp/content/10800000/001527013.pdf)；[广岛市 PDF](https://www.city.hiroshima.lg.jp/_res/projects/default_project/_page_/001/039/664/250926bessi.pdf) | **规则引擎（日本）**、知识库、评测基准（PSMA 修订前后的版本漂移） | PSMA 修订的正式编号和日期未在 mhlw.go.jp 上确认，只读到地方政府转发件；JSNM 2025-07-17 PSMA 手册未打开，许可证未查 |
| 日本 | JSNM 认可的《Lu-177-DOTA-TATE 适正使用手册》英文版（Ann Nucl Med 2018） | 内容见上轮 | **CC BY**（Europe PMC 许可字段） | [PMC5852188](https://pmc.ncbi.nlm.nih.gov/articles/PMC5852188/) | 知识库、规则引擎 | 上轮"许可证未查"现已解决；只有日文原版是正式文本；JRIAS 的日文 v1.2 许可未查 |
| **英国** | ARSAC *Notes for Guidance*，**2026 年 7 月版**（87 页，页面 2026-07-14 更新） | 表 7.1 治疗后避孕期：131I 碘化物（所有甲状腺适应证，≤6,000 MBq）**4 个月**；131I-mIBG（7,500 MBq）4 个月；89Sr 24 个月；153Sm-EDTMP 1 个月。表 7.2–7.4：哺乳中断、有效暴露时间、最大有效半衰期。2026 年 7 月的更新涉及 IR(ME)R 2024 修订和 2027-01-01 起的许可流程。出院标准只作为课程条目出现，没有数值 | **Open Government Licence v3.0**（"另有说明者除外"）；可商用，须署名 | [GOV.UK 页面](https://www.gov.uk/government/publications/arsac-notes-for-guidance)；[NfG PDF](https://assets.publishing.service.gov.uk/media/6a510b75c3d64d94cacab7aa/ARSAC_NfG_July_2026.pdf) | 规则引擎（英国妊娠/哺乳表）、知识库 | 属于建议而非法定文本（2020 版措辞）；放行数值在 IPEM MDGN |
| 英国 | IPEM *Medical and Dental Guidance Notes* 第 2 版（IOP 2024），含"患者使用放射性物质后离院"一章；ARSAC 2024-11 会议纪要把 NfG §7.23 与 MDGN 第 2 版对齐 | 放行章节内容未取得 | **商业出版物，受版权保护**（电子版约 £99） | [检索摘要](https://indiepubs.co.uk/products/medical-and-dental-guidance-notes-second-edition-1755102643066)；[ARSAC 纪要](https://assets.publishing.service.gov.uk/media/6989d892d3f57710b50a9bd3/7_November_2024_ARSAC_meeting_minutes.pdf) | 规则引擎（需许可） | **不开放**；购买许可，或在法律意见下按合理使用编码其中数值 |
| **加拿大** | CNSC"核物质的治疗用途"指南页（2024-07-15 修改）；REGDOC-1.6.1 | 持证方的政策须涵盖照护者和患者须知及患者放行；他人剂量可能超过 1 mSv，因此需要剂量评估；照护者约束"通常为 5 mSv"；须知可以是信息单或钱包卡 | Government of Canada 条款：除另有说明外，可为**非商业**目的复制；商业再分发需书面许可 | [CNSC](https://www.cnsc-ccsn.gc.ca/eng/nuclear-substances/therapeuticuseofnuclearsubstances/)；[canada.ca 条款](https://www.canada.ca/en/transparency/terms.html) | 规则引擎（加拿大，调用剂量引擎）、知识库（非商用） | **没有数值放行阈值** |
| **澳大利亚** | ARPANSA RPS 14.2（2008）、RPS 4（放行建议）、RPS C-5（2019） | ARPANSA 计划重写安全指南（检索摘要） | 据检索结果，RPS F-1、C-1 Rev 1 为 **CC BY 3.0 AU**；版权页称各出版物条款不同 | [RPS 14.2](https://www.arpansa.gov.au/regulation-and-licensing/regulatory-publications/radiation-protection-series/guides-and-recommendations/rps14-2)；[RPS 4 PDF](https://www.arpansa.gov.au/sites/default/files/legacy/pubs/rps/rps4.pdf) | 规则引擎（澳大利亚）、知识库 | **【未核实】**：arpansa.gov.au 返回 503 或超时；RPS 4、14.2、C-5 的许可未逐一核对 |
| **德国** | *Richtlinie Strahlenschutz in der Medizin*（GMBl 2011，2014 修订）；Fachkunde 条款被 BMUV 2024-12-05 通函废止并移入 FK-RL Medizin；人员配置规则（2.1.2）被 2025-07-18 的 BMUV 指南宣布失效 | 没有找到完整的新版本；放行标准（≤250 MBq I-131 / 2 m 处 3.5 µSv/h）见上轮 | **§5 UrhG**：法律、条例、官方公告不受保护（第 1 款）；其他为公众知情发布的官方作品也不受保护，但仍须遵守禁止修改（§62）和注明出处（§63）（第 2 款） | [umwelt-online 合订本](https://umwelt-online.de/recht/energie/strahlen/rl/medstrschrl.htm)；[§5 UrhG](https://www.gesetze-im-internet.de/urhg/__5.html) | 规则引擎（德国）、知识库 | 属于第 1 款还是第 2 款未定，按第 2 款处理：注明出处、不改原文；umwelt-online 是私人汇编，**没有找到 BMUV 托管的合订本** |
| **IAEA** | SRS-63、SSG-46（上轮） | 出版物版权页："使用 IAEA 出版物全部或部分文字须取得许可，通常需签版税协议。非商业复制和翻译的申请受到欢迎，逐案审议" | 需许可；非商用复制和翻译"欢迎" | [SSG-46 PDF](https://www-pub.iaea.org/MTCD/Publications/PDF/PUB1775_web.pdf) | 知识库（取得许可后） | 网站使用条款页仍被 Cloudflare 拦截（403/402） |

| 指南或材料 | 许可证（Europe PMC 等，2026-11-23） | 链接 | 对 IRSA 的结论 |
|---|---|---|---|
| EANM 良性甲状腺疾病放射性碘治疗指南（2023） | **CC BY** | [PMC10542302](https://pmc.ncbi.nlm.nih.gov/articles/PMC10542302/) | 可入库（署名） |
| EANM 剂量学委员会 177Lu 标记 SSTR 和 PSMA 配体建议（2022） | **CC BY** | [PMC9015994](https://pmc.ncbi.nlm.nih.gov/articles/PMC9015994) | 可入库 |
| EANM/EANO/RANO/SNMMI 脑膜瘤治疗诊断联合指南（2024） | **CC BY** | [PMC11445317](https://pmc.ncbi.nlm.nih.gov/articles/PMC11445317/) | 可入库 |
| IAEA/EANM/SNMMI PRRNT 实践指导（2013） | **CC BY-NC** | [PMC3622744](https://pmc.ncbi.nlm.nih.gov/articles/PMC3622744) | IAEA 非商用部署可用，商用不可 |
| EANM/IAEA/SNMMI SSTR 靶向放射性核素治疗联合指导（2026，*The EANM Journal*，Elsevier，doi:10.1016/j.eanmj.2026.100017） | **未核实** | [检索摘要](https://iro.uiowa.edu/esploro/outputs/journalArticle/Joint-EANM-IAEA-and-SNMMI-practical/9985147077202771) | 它取代了 2013 版的 177Lu-DOTATATE 内容，入库前先查许可 |
| EANM DTC 放射性碘指南（2008）；EANM/SNMMI RAIU 与甲状腺显像（2019） | **非开放获取** | [doi:10.1007/s00259-008-0883-1](https://doi.org/10.1007/s00259-008-0883-1)；[Springer](https://link.springer.com/10.1007/s00259-019-04472-8) | 只能引用、改写 |
| SNMMI/EANM DTC 程序标准（JNM 63(6):15N 节选版） | 未查 | [JNM](https://jnm.snmjournals.org/content/63/6/15N) | 待查 |
| HPS "Ask the Experts" | 索引称"超过 15,000 个问题"、22 个类别；"专家无法提供医疗建议"。转载政策：有署名作者的条目版权归作者，否则归 HPS，须直接向版权方取得许可；robots.txt 允许抓取（Crawl-delay 10） | [HPS ATE](https://hps.org/publicinformation/ate/)；[HPS 免责声明](https://hps.org/disclaimer/) | **没有文本挖掘条款**；抓取许可不是再利用许可；改写的问题主题风险较低，但需法务审查 |
| SNMMI 患者材料 | 网站材料除另有说明外受版权保护【检索摘要】 | [SNMMI 法律声明](https://communities.snmmi.org/legal-disclaimer) | 先申请许可，未获许可前只当作问题主题 |

研究笔记还提出一个未核实的问题：欧盟 DSM 指令第 3–4 条或日本著作权法第 30-4 条这类法定文本与数据挖掘例外，能否覆盖"用 HPS/IAEA/SNMMI 文本建内部 RAG 索引"？这需要法务审查，而且即使适用，也不能替代在回答中再分发文本所需的许可【推断】。

### 数据集：放射防护患者问答仍为零，Cancer-Myth 是最好的开放模板

Hugging Face、GitHub 和 PhysioNet 上**仍然没有任何带开放许可的放射防护或核医学治疗患者问答数据集**；检索"radioiodine""radiation oncology""nuclear medicine"只返回基础设施统计和影像数据。可用的开放资源都在相邻领域。**Cancer-Myth（CC BY 4.0）是 IRSA"虚假安慰/错误前提"题目最好的开放模板**：585 个由 3 名血液肿瘤科医生核查、带错误前提的癌症患者问题；GPT-5、Gemini-2.5-Pro、Claude-4-Sonnet 等前沿模型纠正错误前提的比例都不超过 43%；加入预防性提示后升到 80%，但把 150 题无错误前提对照集（Cancer-Myth-NFP）中的 41% 误判为有问题（[arXiv 2504.11373](https://arxiv.org/abs/2504.11373)；[HF](https://huggingface.co/datasets/Cancer-Myth/Cancer-Myth)）。NFP 对照集说明了怎样衡量"过度纠正"的代价，这对 IRSA 很重要，因为 IRSA 也不能凭空编造限制【推断】。多语言方面没有捷径：放射防护题目必须先用英文写好再专业翻译，或者用各目标语言原生编写；现有多语言医学数据集只能用来筛查候选 LLM 的医学语体，或校准多语言评审（[MedExpQA](https://huggingface.co/datasets/HiTZ/MedExpQA)；[GerMedIQ](https://huggingface.co/datasets/justn-hfnbtzr/GerMedIQ)）。

| 名称 | 内容 | 规模 | 许可证/商用 | 获取方式 | 链接 | 服务模块 | 注意事项 |
|---|---|---|---|---|---|---|---|
| **Cancer-Myth**（USC，arXiv 2504.11373） | 带错误前提的癌症患者问题，3 名血液肿瘤科医生核查；另有无错误前提对照集 | 585 + 150 题 | **CC BY 4.0**（HF 标签和数据卡），可商用 | HF（`split='validation'`） | [HF](https://huggingface.co/datasets/Cancer-Myth/Cancer-Myth)；[代码](https://github.com/Bill1235813/cancer-myth) | 评测基准：例如"我的 I-131 辐射一天就没了，今晚可以抱孙子吧？" | **代码仓库没有 LICENSE 文件**；衍生集 `shenranw/CancerMyth-TPQ` 标 Apache-2.0，内容未查 |
| "The Patient Is Always Right?"（arXiv 2610.34013，2026-10） | 9 个模型；患者坚信某个错误诊断，按信念与证据的距离分级；模型平均 **23.2%** 采纳患者信念，有时连虚构疾病也采纳；"罕见病声称者"人设达 **43.0%** | 论文 | 未发现数据发布 | arXiv | [arXiv](https://arxiv.org/abs/2610.34013) | 评测基准：按与引擎输出的距离给信念分级；权威人设 | 只有方法 |
| MedPRESS 新 HF 仓库 `samanjoy2/MedPRESS_Benchmark` | 与上轮相同的基准，新增第二个仓库 | — | 新仓库**没有许可证标签**；上轮记录的原仓库为受限访问、仅限研究 | HF | [HF](https://huggingface.co/datasets/samanjoy2/MedPRESS_Benchmark) | 评测基准 | 两个都按非商用处理 |
| `geodesic-puria/medical-sycophancy`（及 -severe、-egregious）；`mli5/medquad-sycophancy` | 医疗谄媚提示 | — | **无许可证标签** | HF | [HF](https://huggingface.co/datasets/geodesic-puria/medical-sycophancy) | 不使用 | 来源不明 |
| HealthBench Professional 红队子集（`type = red_teaming`） | 面向临床医生的对抗题 | 约占 525 题的三分之一 | **MIT** | HF | [HF 卡](https://huggingface.co/datasets/openai/healthbench-professional) | 评审方法参考 | 面向医生而非患者；OpenAI 要求不要公开发布样例 |
| LaymanRRG；BioLaySumm 2025 LaymanRRG 开源赛道 | 胸片报告的通俗语言翻译；放射报告与通俗报告配对 | 报告级 50,000 行；150,454 / 10,000 / 10,537 | **HF 无许可证标签**；源自 MIMIC-CXR，大概率适用 PhysioNet 资质条款 | HF | [LaymanRRG](https://huggingface.co/datasets/LaymanRRG/LaymanRRG-report-level)；[BioLaySumm](https://huggingface.co/datasets/BioLaySumm/BioLaySumm2025-LaymanRRG-opensource-track) | 评审（可读性和通俗用语参考） | 不是放射防护内容；商用不明 |
| ROND（Radiation Oncology NLP Database，arXiv 2401.10995） | 逻辑推理（100 道人工标注题）、问答、摘要等放疗 NLP 任务 | — | **Apache-2.0**（LICENSE 文件，本轮核对）；检索摘要称许可问题联系作者（可能冲突） | GitHub | [GitHub](https://github.com/zl-liu/Radiation-Oncology-NLP-Database) | 知识库检索测试 | 唯一找到的开放许可放疗问答集；面向专业人员 |
| RadLIT-9；NuclearQAv2 | 放射学教育检索基准（含 45 题物理/核医学子集）；约 1,240 道**核工程**问答 | — | 未查 | HF；arXiv | [RadLIT-9](https://huggingface.co/datasets/matulichpt/radlit9-benchmark)；[NuclearQAv2](https://arxiv.org/html/2606.27047) | 知识库检索测试 | NuclearQAv2 与医学无关，列出以免误用 |

| 多语言数据集 | 语言 | 内容/规模 | 许可证（HF 标签） | 链接 | IRSA 用途 | 注意事项 |
|---|---|---|---|---|---|---|
| **MedExpQA**（HiTZ） | 英、西、法、意 | 带金标准解释的医学考试问答 | **CC BY 4.0** | [HF](https://huggingface.co/datasets/HiTZ/MedExpQA) | 多语言评审校准；西、法语质量检查 | 考试题，不是对话 |
| **GerMedIQ**（ACL SRW 2025） | 德 | 39 名非患者对 116 个问诊问题的 4,524 条**模拟**回答，18 个 LLM 的回答，以及 LLM 评审与人工评审对比 | **CC BY 4.0** | [HF](https://huggingface.co/datasets/justn-hfnbtzr/GerMedIQ) | 模拟器（德语外行回答风格）；评审（有人工与 LLM 评审对照） | 规模小；本轮找到的最好德语患者侧资源 |
| `elcooooo/patient-simulator-german` | 德 | 受约束的患者模拟器对话：模拟患者只能从目录中逐字选答，4,000 训练 / 500 验证 | **无许可证标签** | [HF](https://huggingface.co/datasets/elcooooo/patient-simulator-german) | 模拟器设计思路（目录约束，类似原子事实） | 不要随产品发布 |
| **MediQAl**；FrenchMedMCQA | 法 | 法语医学问答；法语药学选择题 | **CC BY 4.0**；**Apache-2.0**（qanastek 副本），DrBenchmark 副本无标签 | [MediQAl](https://huggingface.co/datasets/ANR-MALADES/MediQAl)；[FrenchMedMCQA](https://huggingface.co/datasets/qanastek/frenchmedmcqa) | 多语言（法） | 镜像之间许可不同 |
| MMedBench | 英、中、日、法、俄、西 | 带理由的多语言医学选择题 | 原版 **CC BY-NC 4.0**；学生重传版标 CC BY 4.0 | [原版](https://huggingface.co/datasets/Henrychur/MMedBench) | 仅作多语言评测（非商用） | **不要相信重传版的标签** |
| **CMB；Huatuo26M-Lite；ApolloCorpus**（FreedomIntelligence） | 中（Apollo 为多语言，含西、法、阿、中） | 中文医学基准；问答对；多语言医学语料 | **Apache-2.0** | [CMB](https://huggingface.co/datasets/FreedomIntelligence/CMB)；[Huatuo26M-Lite](https://huggingface.co/datasets/FreedomIntelligence/Huatuo26M-Lite) | 多语言（中、阿） | Huatuo 回答来自在线问诊网站，来源未查 |
| JMedBench；`japan-ai-official/igakuqa-subset-curated` | 日 | 日语医学选择题/NER 等；日本医师国家考试精选子集 | 无标签；**Apache-2.0** 标签 | [JMedBench](https://huggingface.co/datasets/Coldog2333/JMedBench)；[IgakuQA 子集](https://huggingface.co/datasets/japan-ai-official/igakuqa-subset-curated) | 仅作日语评测 | 考试题版权不因标签而解决 |
| **AfriMed-QA v2** | 英（非洲医疗情境） | 泛非医学问答 | **CC BY 4.0**（镜像为 CC BY-SA 4.0） | [HF](https://huggingface.co/datasets/afrimedqa/afrimedqa_v2) | IAEA TC 受援国的地区公平性检查 | 不是放射防护 |
| **Global-MMLU**（CohereLabs） | 42 种语言 | 含医学科目的 MMLU 译本 | **Apache-2.0** | [HF](https://huggingface.co/datasets/CohereLabs/Global-MMLU) | 快速筛查候选 LLM 的语言能力 | 通用知识 |
| KorMedMCQA；阿拉伯语问诊集（Shifaa、arabic_medical_dialogue）；MedArabiQ | 韩；阿 | 韩国执照考试；阿拉伯语问诊对话；阿拉伯语医学问答 | **CC BY-NC 2.0**；Apache-2.0/MIT 标签；无标签 | [KorMedMCQA](https://huggingface.co/datasets/sean0042/KorMedMCQA)；[Shifaa](https://huggingface.co/datasets/Ahmed-Selem/Shifaa_Arabic_Medical_Consultations)；[MedArabiQ](https://huggingface.co/datasets/qimma/QA_MedArabiQ) | 仅作评测 | 阿拉伯语问诊数据常为抓取所得，上传者的标签不一定有效 |

### 患者模拟器与"模拟-真实差距"：模拟器太合作，会抬高分数

两款医学模拟器发布了代码：**PatientSim**（NeurIPS 2025 D&B；人设维度包括性格、语言能力、病史回忆、认知混乱，共 37 种组合）和 **DischargeBench**（基于人设的出院教育）。两者的临床档案都来自 MIMIC，适用 PhysioNet 资质许可；PatientSim 的代码仓库**没有 LICENSE 文件**，所以应当重写而不是 fork（[PatientSim](https://github.com/dek924/PatientSim)；[PhysioNet](https://physionet.org/content/persona-patientsim/)）。**DischargeBench 是与 IRSA 最接近的任务类比**：LLM"出院教育者"给模拟患者讲解，评分包括话题清单（对应必须说到的内容）、模拟理解（对应复述确认）和声明级安全审计（对应数字/声明校验器）；8 个模型中最好的 TCS 0.70、SCS 0.72；不信任型和焦虑型人设让所有模型的理解分都下降（[arXiv 2609.20827](https://arxiv.org/abs/2609.20827)）。

2026 年几项把模拟器和真人对比的研究结论一致：**LLM 模拟器过度合作、风格单一、过早交出信息，从而抬高智能体分数。** "Mind the Sim2Real Gap"首次用真人完整跑了 τ-bench：451 名参与者、165 个任务，对比 31 个 LLM 模拟器；模拟器缺乏挫败感和模糊性，像"简单模式"，智能体成功率高于真人基线，而通用能力更强的模型并不模拟得更像（[arXiv 2603.11245](https://arxiv.org/abs/2603.11245)）。"Lost in Simulation"在美国、印度、肯尼亚、尼日利亚做用户研究，智能体成功率随所用的模拟用户 LLM 变化最多 9 个百分点，对 AAVE 和印度英语使用者的代理保真度最差（[arXiv 2601.17087](https://arxiv.org/abs/2601.17087)），这直接关系到 IAEA 多国部署。"The Average Patient"发现模拟的精神科患者变异度比真人小 1.5–11 倍（[arXiv 2611.11679](https://arxiv.org/abs/2611.11679)）。笔记据此给出 IRSA 模拟器的设计【推断】：抽样生成家庭事实卡（原子事实，上轮已提出）；按 PatientSim 的人设维度重写健康素养、回忆、焦虑/不信任和语言能力；采用 DischargeBench 式指标；加入脚本化摩擦（隐瞒、自相矛盾、"我已经问过护士了"）；在试点子集上报告经真实照护者或标准化病人对话校准过的分数；按语言和方言分别报告。

| 名称 | 模拟什么/研究设计 | 规模 | 许可证/商用 | 链接 | IRSA 用途 | 注意事项 |
|---|---|---|---|---|---|---|
| **PatientSim**（Kyung 等，arXiv 2505.17818） | 基于 MIMIC-ED/MIMIC-IV 档案的急诊患者；人设维度：性格、语言能力、病史回忆、认知混乱；测了 8 个 LLM，Llama 3.3 70B 经 **4 名临床医生**验证 | **170** 个档案 | 数据：**PhysioNet 资质健康数据许可 1.5.0** + DUA + CITI 培训；代码：**无 LICENSE** | [GitHub](https://github.com/dek924/PatientSim)；[PhysioNet](https://physionet.org/content/persona-patientsim/) | 模拟器：语言能力 → 健康素养；回忆 → "记不清给药活度"；混乱 → 老年照护者 | 托管 LLM 只支持 Vertex AI 或 Azure OpenAI，本地用 vLLM；重写而不是 fork |
| **DischargeBench**（arXiv 2609.20827） | 出院教育者教人设化模拟患者（性格、教育、**健康素养**、回忆）；指标：对话质量、话题清单分（TCS）、模拟理解分（SCS）、声明级安全审计；LLM 评审经医生标注验证 | **320** 例，覆盖 16 个 ICD 章 | PhysioNet 资质许可；代码许可未找到 | [arXiv](https://arxiv.org/abs/2609.20827)；[PhysioNet](https://physionet.org/content/mimic-iv-ext-dischargebench/1.0.0/) | **最接近的任务类比** | 评审与医生一致性只基于 20 个共同病例，医生之间一致性的 95% CI（[-0.09, 0.49]）包含 0【检索摘要】 |
| 对话式健康 AI 风险评估的患者模拟框架（JMIR AI 2026） | 三类档案：医学（All of Us EHR）、语言（健康素养梯度）、行为（聚类）；用于抗抑郁药决策辅助：熟练读写者首位概念检索 81.9%，读写能力有限者 47.6% | 论文 | 代码/许可未查 | [JMIR AI](https://ai.jmir.org/2026/1/e100772) | 模拟器：健康素养语言档案；81.9% vs 47.6% 是现成指标 | 只证明模拟器能区分人设，**不证明**它像真人 |
| LLM 模拟出院小结患者理解（多模型研究） | 按教育、性别、就医频率设人设，对比 99 名一般人群 | 99 人 | 文章 OA | [PMC13690339](https://pmc.ncbi.nlm.nih.gov/articles/PMC13690339) | 模拟器验证设计 | LLM 能逼近众数子组的回答，但逼近不了离散度；作者建议不要用它替代人工反馈 |
| **Mind the Sim2Real Gap**（arXiv 2603.11245，COLM 2026） | 451 名真人、165 个 τ-bench 任务 vs 31 个模拟器；提出 User-Sim Index | — | 数据未查 | [arXiv](https://arxiv.org/abs/2603.11245) | **IRSA-Bench 校准的关键引用**：基于 τ² 的 IRSA-Bench 若不经真实照护者校准，会高估 Pass^k | — |
| **Lost in Simulation**（arXiv 2601.17087） | 美、印、肯、尼日利亚用户研究，τ-Bench 零售域；成功率随模拟用户 LLM 变化最多 9 pp；难题被低估、中等题被高估 | — | 数据未查 | [arXiv](https://arxiv.org/abs/2601.17087) | 多国部署中模拟器的方言和地区偏差 | — |
| *realsim*："Synthetic Users, Real Differences"（arXiv 2605.02624） | 16 个领域 1K 段真实任务对话，在 8 个维度上与模拟对话做分布级比较；模拟器缺少真实的"沟通摩擦" | 1K 段对话 | 许可未查 | [arXiv](https://arxiv.org/abs/2605.02624) | 用 realsim 维度对照试点记录审查 IRSA 照护者模拟器 | — |
| "The Average Patient"（arXiv 2611.11679，2026-11） | 17 个模型 vs 8 个人类样本；模拟患者变异度小 1.5–11 倍，几乎所有症状都给中等分；提示、人设、更大模型和推理只能减轻、不能消除 | — | 未查 | [arXiv](https://arxiv.org/abs/2611.11679) | 人设模拟器会坍缩成"平均照护者"；要显式抽样事实卡（如"家中无儿童"），不能只靠人设提示 | — |
| MIMESIS；AutoSim | 训练出来的用户模拟器；AutoSim 的排名与 NIST iKAT 一致，但检索 nDCG 可高估 +0.136 | — | 未查 | [MIMESIS](https://arxiv.org/pdf/2610.09484)；[AutoSim](https://arxiv.org/pdf/2610.28054) | 排名一致不代表绝对分数一致 | 检索摘要 |

### 评分细则与评审一致性：评审之间一致不等于评对了

本轮新增的评测方法可以拼成一条 IRSA-Bench 评分细则管线【推断】：先由指南或规则引擎的原子事实自动起草逐题细则（检索增强的智能体细则生成，arXiv 2601.15161）；把条目写成布尔判断（Adaptive Precise Boolean，arXiv 2503.23339）；只保留能把专家偏好的回答排在被拒回答之前的细则（arXiv 2604.24710：20 名临床医生为 823 个病例写了 1,646 条细则，LLM 与临床医生的排序一致性 τ 0.42–0.46，临床医生之间为 0.38–0.43，LLM 细则成本约低 1,000 倍）；由 ≥3 名专家分 3 个阶段裁定（HealthBench Professional）；**只报告评审与专家的一致性，不只报告评审之间的一致性**。最后一条来自一项预注册的负面结果：改写争议最大的细则条款后，评审之间的 Krippendorff α 上升 0.069，与专家评分的一致性却没有变化（Kendall τ −0.007）；在 HealthBench 元评测上，两项指标都没有达到预注册标准（[arXiv 2610.13303](https://arxiv.org/abs/2610.13303)）。

| 引用 | 设计与关键数字 | 数据/许可 | IRSA 用途 |
|---|---|---|---|
| **HealthBench Professional**（arXiv 2604.27470，2026-04-30） | 临床医生与"ChatGPT for Clinicians"的真实对话，3 类用途；细则由 **≥3 名医生分 3 阶段**撰写和裁定；从 15,079 题池中挑选，难题富集约 3.5 倍；约三分之一为红队题；每题都收集了医生回答；共 525 题【search-summary】 | **MIT**；字段含 `rubric_items`、`type`（good_faith/red_teaming）、`physician_response`；有防污染标记串；**没有发布官方外部评分器** | 照搬 good_faith/red_teaming 划分和 3 阶段裁定（由医学物理师参与）；每题收集一份 RPO 参考回答（[HF](https://huggingface.co/datasets/openai/healthbench-professional)） |
| Epoch AI 对 HealthBench Professional 的评审 | 结论"Flawed"：来自真实提示的题目可能含糊、欠规定，他人撰写的细则可能带偏见 | — | 尽量让 IRSA 题目能由引擎核对、有确定的预期结果（[Epoch AI](https://epoch.ai/benchmarks/healthbench-professional/review)） |
| 823 个病例的病例特异细则（arXiv 2604.24710，投 JAMIA） | 见上文 | 数据未查 | 细则验证协议；LLM 起草、专家核查的成本论据（[arXiv](https://arxiv.org/abs/2604.24710)） |
| 检索增强的智能体细则生成（arXiv 2601.15161） | 检索权威证据 → 拆成原子事实 → 加入交互约束 → 生成逐题细则；临床意图一致性 50.20%（HealthBench）和 31.90%（LLMEval-Med）；比 GPT-4o 细则胜率 +7.8 pp；按细则修改的回答提升 9.2% | 代码未查 | 从 EANM/MHLW 等指南的原子事实生成必须说/不得说细则草稿（[arXiv](https://arxiv.org/abs/2601.15161)） |
| Adaptive Precise Boolean 细则（Google，arXiv 2503.23339） | 用一小组有针对性的是/否问题替代少量复杂条目；在代谢健康领域验证 | — | 适合引擎可核对的题目（例如"是否与引擎输出一致地说明了 7 天内避免与儿童接触？"）（[arXiv](https://arxiv.org/abs/2503.23339)） |
| "Agreement Without Alignment"（arXiv 2610.13303，预注册） | 见上文；共 129,770 次评审请求 | — | **注意**：评审之间高度一致不是有效性证据 |
| DischargeBench、PhysAssistBench、MedRealMM 的评审验证 | PhysAssistBench 在 3,685 条细则上评审与标注者一致 95.37%（单一目标模型）；MedRealMM 发现评审与医生的一致性接近评审之间的一致性，正向条目较低 | — | 按条目类型（正向/负向）分别报告一致性（[PhysAssistBench](https://arxiv.org/pdf/2606.18613)；[MedRealMM](https://arxiv.org/pdf/2607.09142)）【检索摘要】 |
| "Artificial Authority"（叙述性综述，2026） | 医疗中 LLM 评审的验证策略 | OA | 背景引用（[PMC12837895](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12837895/)） |

## 注释书目：IRSA 先要打败 AUC 0.929 和 RMSE 2.2 µSv/h 这类简单基线

本节按主题列出上轮报告和背景评估都没有用过的 2018–2026 年论文。每篇都核对过 Europe PMC/PubMed 题录；除非另有标注，读的是摘要。"数据/代码"一列写"未查"，表示没有打开全文查找数据可用性声明；除非明确写出，没有一篇存放了逐患者数据或代码。

### 必须超越的现有基线：五项简单模型已经做得不差

**这是本轮对 IRSA 验证设计影响最大的一组文献。** 用治疗前影像预测出院剂量率，已经做到 AUC 约 0.93 或 RMSE 约 2–3.5 µSv/h（177Lu-DOTATATE）；用简单统计模型预测 131I 放行和家属剂量，也有大样本或高 AUC 的结果。IRSA 的"可穿戴更新贝叶斯估计"在宣称附加价值之前，应当先与这些更简单的基线比较，并且要胜过它们【推断】。这些基线用的是治疗前显像（洗脱率、SPECT 总摄取），IRSA 本身不一定有这些数据，所以院内配对测量阶段应顺带采集这些协变量，既可放进先验，也可作为预注册的比较对象【推断】。Kranert 2022 发现剂量率与滞留活度之间没有普适的相关关系，这又说明不经患者特异校准、直接由剂量率推活度是不可靠的。

| 引用 | 设计/n/核素 | 关键数字 | 数据/代码 | 对 IRSA 的用途 |
|---|---|---|---|---|
| **Shimizu H 等**，*Nucl Med Commun* 2026;47(5):503–509（[doi:10.1097/mnm.0000000000002110](https://doi.org/10.1097/mnm.0000000000002110)） | 回顾性，**n = 26**，177Lu-DOTATATE 首周期（日本，1 m 处 18 µSv/h 标准） | 26 例中 14 例（53.8%）6 h 达标；治疗前 SSTR 显像全身**洗脱率** 57.6% vs 35.0%（p < 0.001），**AUC 0.929**；以 53.5% 为界，灵敏度 92.9%、特异度 91.7%；人口学、eGFR、活度无差异 | 付费 | 贝叶斯有效半衰期先验的治疗前协变量；"预计当日可出院"标志；**预注册比较基线** |
| **Wakabayashi N 等**，*Ann Nucl Med* 2025;39(7):696–706（[PMC12174230](https://pmc.ncbi.nlm.nih.gov/articles/PMC12174230/)） | 回顾性，**36 例 111 个周期**，177Lu-DOTATATE（北海道） | 给药后第 1 天 1 m 处剂量率中位 **8.0（4.0–26.0）µSv/h**；预测因子为 SPECT/CT 总摄取和单位体重活度，肌酐清除率和 ALBI 评分无预测价值；由治疗前 111In-喷曲肽 SPECT/CT 得到的预测方程 **RMSE 2.24 µSv/h**（建模）、**3.47 µSv/h**（后续测试病例） | CC BY；数据未查 | IRSA 自己的第 1 天剂量率预测必须达到或优于的误差；先验协变量清单 |
| **Han S 等**，*PLoS One* 2021;16(5):e0251627（[PMC8139448](https://pmc.ncbi.nlm.nih.gov/articles/PMC8139448/)） | 回顾性，**n = 215** 例 DTC 住院 131I 消融（韩国） | 放行时（第 2 天）1 m 处剂量当量率 **1.0–48.9 µSv/h**；双指数模型预测第 6–8 天剂量率，**除 1 例外实测值均 ≤ 预测值**（模型偏保守）；不需要额外的白天限制；与 ATA 按活度的规则相比，允许乘公共交通的时间更长，夜间限制显著更短 | CC BY；数据未查 | 现成的"先预测、再用后续实测核对"验证设计；131I 放行剂量率范围 |
| **Ebrahimi M 等**，*J Radiol Prot* 2018;38(1):422–433（[doi:10.1088/1361-6498/aa9b9b](https://doi.org/10.1088/1361-6498/aa9b9b)） | **52 例甲状腺癌患者、99 名家属**，TLD，131I（伊朗） | 实测剂量均值 0.45、中位 0.28、范围 **0.1–3.64 mSv**；13 个输入的前馈神经网络 **MSE 0.142**，识别 >1 mSv 的 **AUC 0.957**，所有 >1 mSv 的家属都被识别 | 付费；未见数据/代码声明 | 最早的放行前预测照护者剂量的 ML 先例；IRSA 代理模型的基线；13 个输入特征可复用 |
| **Zhang C 等**，*Front Physiol* 2025;16:1599657（[PMC12510937](https://pmc.ncbi.nlm.nih.gov/articles/PMC12510937/)） | 回顾性，**n = 1,044** 例 DTC，131I（南京 2022–2024），SVM | 48 h 滞留 <400 MBq 者有效半衰期 10.35 h（男）、9.64 h（女），>400 MBq 者 14.07 h、13.47 h；"48 h 可出院"SVM **准确率 88.04%、精确率 96.89%** | CC BY；数据未查 | 大队列基准与 131I 有效半衰期先验（400 MBq 是中国标准） |
| Liu F 等，*EJNMMI Res* 2024;14:119（[PMC11604894](https://pmc.ncbi.nlm.nih.gov/articles/PMC11604894/)） | 前瞻性，**n = 30** 例 GEP-NEN，177Lu-DOTATATE（中国），27 例完成 4 周期 | 0–6 h 剂量率双指数拟合后推算公众剂量；保证 **4 周期总剂量**在限值内的最早日常接触时间：**成年家属 ≥24 h，>10 岁儿童 ≥48 h，3–10 岁儿童 ≥144 h，同事 ≥192 h** | CC BY-NC-ND；数据未查 | 按接收者类别的时间规则；IAEA TC 情境下的中国监管基准 |
| Kranert WT 等，*Nuklearmedizin* 2022;61(2):111–119（[doi:10.1055/a-1697-8126](https://doi.org/10.1055/a-1697-8126)） | 140 例患者 478 个周期，177Lu-PSMA-617（法兰克福） | 34% 的患者一年内接受 3–5 个周期；72 h 出院时，德国 2 m 处 1 mSv/年限值在每年 3 周期时有 **10%**、4 周期时有 **15%** 的患者超标；1 m 处平均比剂量率 **0.00462 µSv/(h·MBq)**；**剂量率与滞留活度之间没有普适相关** | 付费 | 规则引擎的年度多周期剂量台账；比剂量率先验；反对不经个体校准由剂量率推活度 |
| Chandra A 等，*Appl Radiat Isot* 2023;198:110878（[doi:10.1016/j.apradiso.2023.110878](https://doi.org/10.1016/j.apradiso.2023.110878)） | 放射性药物设施环境 γ 剂量率时间序列上的随机森林 | 过程检测**精确率 82.35%、特异度 97.11%、平衡准确率 91%、F1 82%**；专家标注由 k-means 聚类辅助 | 付费 | 可穿戴轨迹事件检测（排尿、淋浴、接近变化）和降低误报的方法模板 |

### 问题陈述：102 家中心的出院指导各不相同

**Zahner 等 2026 年 JNM 调查是 IRSA"个性化、规则带版本号"指导最有力的近期论据**：23 个国家 102 家治疗诊断中心对治疗前和核心治疗步骤高度一致，但治疗后防护措施差异很大，包括距离要求、防护措施持续多久、出院后交通、家庭与卫生规则、社交（[doi:10.2967/jnumed.126.273336](https://doi.org/10.2967/jnumed.126.273336)）。同一批文献还指出，有三个剂量项常被放行计算漏掉：限制期内的剂量、回家路上的剂量、一年多个周期。三者合起来可以把"合规"的结论翻过来，IRSA 的引擎应当默认把它们都算进去【推断】。

| 引用 | 设计/n/核素 | 关键数字 | 数据/代码 | 对 IRSA 的用途 |
|---|---|---|---|---|
| **Zahner J、Unterrainer LM、Czernin J、Calais J、… Benz MR**，*J Nucl Med* 2026（[doi:10.2967/jnumed.126.273336](https://doi.org/10.2967/jnumed.126.273336)，PMID 42988564） | 国际调查，23 国 **102 家**中心，177Lu-PSMA | 见上文；作者据此制作了**患者信息模板**；新闻报道补充称问卷 152 题、2025 年 1 月发给 186 家中心、门诊 65%/住院 35%【snippet】（[AuntMinnie](https://www.auntminnie.com/clinical-news/molecular-imaging/nuclear-medicine/article/16424445/what-are-patients-being-told-before-during-and-after-lu177-rpt)） | 模板是研究产出；许可和补充材料未查 | 规则引擎与语料的种子（先查许可）；问题陈述；"告诉了患者什么"检查表的结构 |
| Chipiga L 等，*J Radiol Prot* 2025;45(1)（[doi:10.1088/1361-6498/adba70](https://doi.org/10.1088/1361-6498/adba70)） | 建模研究：177Lu-PSMA-617、177Lu-DOTATATE、Na131I、131I-mIBG、153Sm-oxabifor | 比较两类放行标准：只计物理衰变（单次与疗程内多次给药），以及物理衰变加生物排出（有效半衰期），分别在每疗程 1 mSv 和 5 mSv 约束下；估算公共交通乘客与工作人员、化粪池的剂量；结论是放行标准必须纳入交通照射 | 付费 | "回家路上"场景模板（公交、火车的乘客占用）；按疗程计的剂量台账 |
| Banci Buonamici F 等，*Phys Med* 2025;131:104922（[doi:10.1016/j.ejmp.2025.104922](https://doi.org/10.1016/j.ejmp.2025.104922)） | 电子表格模型（连续照射，带时间/距离修正），用已发表场景验证（佛罗伦萨） | 如果把"限制期"**内**的剂量算进去，约束可能根本无法满足；长途返程，尤其是公共交通，**可能需要推迟出行数天**；忽略这些项会误判约束已满足 | 付费；表格未见 | 引擎应积分限制期和回程中的剂量；好的引擎测试用例 |
| Otis G，*Health Phys* 2020;119(6):766–771（[doi:10.1097/hp.0000000000001308](https://doi.org/10.1097/hp.0000000000001308)） | 解析方法 | 给出**分次或间隔很近的给药**的分段有效剂量函数；除非排出时间远短于给药间隔，把每次给药当作相同会过度限制 | — | 分次方案和 225Ac/177Lu 串联治疗的引擎逻辑 |
| Prasad K 等，*EJNMMI Res* 2024;14:70（[PMC11291838](https://pmc.ncbi.nlm.nih.gov/articles/PMC11291838/)） | 儿科门诊脑室内 131I-omburtamab，925–1850 MBq（MSKCC） | 0.3 m 处 0.16–0.46 µSv/h/MBq，1 m 处 0.03–0.08 µSv/h/MBq；患者特异的防护持续 1–10 天，最长的也**比通用法规指导短约 78%** | CC BY | 证明患者特异参数能大幅缩短限制时间；儿科患者场景 |
| Mench A 等，*J Nucl Med* 2025;66(7):990–994（[doi:10.2967/jnumed.124.268522](https://doi.org/10.2967/jnumed.124.268522)） | 工作流程论文（OHSU）：一名 177Lu-PSMA 患者治疗后 48 h，姑息门诊的员工反应惊慌 | 两个工具：**电子病历标志**和**实体腕带**，腕带链接到为非专科医护准备的辐射安全网页 | — | IRSA 可以提供数字版（二维码或钱包卡）和急诊就诊用的"医护模式" |
| Underwood J 等，*Health Phys* 2021;121(2):160–165（[doi:10.1097/hp.0000000000001425](https://doi.org/10.1097/hp.0000000000001425)） | 美国 Lutathera 放行计算 | 结论：Lutathera 可门诊给药；给出安全和出行限制 | 付费 | 规则引擎的美国基线指令集 |
| Forwood N 等，*Phys Eng Sci Med* 2022;45(1):9–12【title-only】；Badawy MK、Anderson A，*J Med Radiat Sci* 2023;70(2):103–105【title-only】 | 观点："放射性核素治疗住院应是例外而非常规"；关于照护者的评论 | — | CC BY（后者） | IAEA TC 叙事的政策框架；澳大利亚照护者剂量约束背景（[doi:10.1007/s13246-022-01104-8](https://doi.org/10.1007/s13246-022-01104-8)；[PMC10258637](https://pmc.ncbi.nlm.nih.gov/articles/PMC10258637/)） |

### 外照射几何与近场 MC：131I 的姿态场景齐全，177Lu 只有一篇

用体模模拟近距离几何的研究**主要针对 131I**：Han 2013（UF 混合体模）覆盖面对面、**母亲抱新生儿**、1 岁儿童站在患者腿上；Dewji 2015（ORNL PIMAL + MCNP6 + Leggett 碘生物动力学）覆盖站姿和**坐乘公共交通**的姿态；Taghvaee 2018（MIRD 体模）覆盖**患者卧床、照护者站在床边**。177Lu 方面，体模 MC 模拟患者体外三维场的只找到 Li 2024 一篇；没有 177Lu 抱孩子或同床的 MC 研究，也没有 225Ac/223Ra 的体模 MC。**这些研究都没有存放代码或数据。** 笔记据此定出 IRSA 的场景集：面对面 0.1–3 m；并排坐；坐在后面；并排躺 0.3 m；把婴儿抱在胸前；儿童坐在腿上。所有这些都已有 131I 的发表结果，177Lu 一个都没有，IRSA 的 MC 训练数据将是第一套 177Lu 对应物【推断】。

| 引用 | 设计/n/核素 | 关键数字 | 数据/代码 | 对 IRSA 的用途 |
|---|---|---|---|---|
| **Li H、Liu H、Zhang W、Lin X、Li Z、Zhuo W**，*J Radiol Prot* 2024;44(2)（[doi:10.1088/1361-6498/ad3835](https://doi.org/10.1088/1361-6498/ad3835)；[PMID 38537256](https://pubmed.ncbi.nlm.nih.gov/38537256)） | PHITS 模拟 177Lu-PSMA 成人男性体模的体外三维场，用实测核对 | 1 m 内模拟/实测比 **0.82–1.19**；骨盆、腹部、胸部横断面拟合剂量率函数的相对 RMS 偏差 **0.21、0.25、0.23**；前后 > 左右；源高度处最高；放行检查需要**多方向、多高度**测量 | 付费；未见存放 | **与 IRSA 177Lu MC 最接近的已发表类比**；rRMSD 约 0.2 是半解析/代理模型相对实测的现实目标；佩戴位置和读数修正的依据 |
| Han EY、Lee C、McGuire L、Brown TL、Bolch WE，*Med Phys* 2013（[doi:10.1118/1.4812425](https://doi.org/10.1118/1.4812425)；[PMID 23927361](https://pubmed.ncbi.nlm.nih.gov/23927361/)） | UF 混合体模 + MC；暴露于成年男/女 131I 患者的儿童和成人家属 | 器官 S 值和单位时间积分活度的有效剂量；场景：5 个距离的面对面站立；**成年女患者抱新生儿**；**1 岁儿童站在患者腿上** | 未见存放；付费 | 抱孩子场景的模板；IRSA 131I 代理模型的直接对照 |
| Dewji S 等，"Estimated dose rates to members of the public from external exposure to patients with 131I thyroid treatment"，*Med Phys* 2015（[doi:10.1118/1.4915084](https://doi.org/10.1118/1.4915084)；[OSTI 1185433](https://www.osti.gov/biblio/1185433)） | 含组织衰减和碘生物动力学的有效剂量率，与 RG 8.39 假设比较；正常（峰值甲状腺摄取约 27%）、DTC（5%）、甲亢（80%）三种情形 | 场景含 10 cm 面对面站立，患者坐在公众前面、后面或旁边；**患者坐在公众后面**时坐乘公共交通的剂量率最高【search-summary, verify】 | 未见存放 | 131I 分姿态几何集，IRSA 场景库可照搬 |
| Dewji S 等，"Assessment of the point-source method…"（*Health Physics*，OSTI 接受稿） | 两个 PIMAL 体模，MCNP6，径迹长度注量 × 比释动能因子，Leggett 2010 碘模型 | 结论：点源从 **10 到 300 cm** "一贯高估"，300 cm 以外与 PIMAL 收敛；站立甲亢患者 1 m 处高估约 **1.9** 倍，站立 DTC 患者 1 m、8 h 处高估 **3.3** 倍；引用 Al-Haj 2007 体屏蔽因子 0.46 ± 0.13（n = 311）。经 Dewji 转述的 Han 2013：1 m 面对面点源高 1.6（甲亢）和 2.2（DTC）倍；两体模**并排躺 0.3 m** 时，甲状腺、腹部、全身源分别高 26、29、23 倍 | 美国政府资助的手稿在 OSTI 免费；无数据或代码 | 131I 的半解析体衰减先验（对应 177Lu 的 Bellamy 48%）；验证；**手稿内部有矛盾，见矛盾清单**（[OSTI 1213312](https://www.osti.gov/servlets/purl/1213312)） |
| Aziz L、Dewji S，*Radiat Prot Dosimetry* 2022;198(6):311–324（[doi:10.1093/rpd/ncac049](https://doi.org/10.1093/rpd/ncac049)） | 年龄分组体模 + 排尿生物动力学的 MC；131I | 与 RG 8.39 点源法相比，8 h 时剂量率被高估约 **3 倍**（DTC，5% 摄取）和约 **2 倍**（甲亢，80% 摄取） | 付费 | 按年龄的接收者系数（儿童接触者）；量化 NRC 点源基线有多保守 |
| Taghvaee S 等，*J Cancer Res Ther* 2018（[PMID 30488857](https://pubmed.ncbi.nlm.nih.gov/30488857/)） | 一个 MCNP4C 输入中两个 MIRD 体模；都站立，或患者卧床、照护者站在旁边 | 患者卧床、照护者站在患者头部附近时剂量最高；最大有效剂量 **0.2 mSv**【数字 search-summary, verify】 | 无 | 床边/卧姿场景 |
| Bellamy M 等，*J Radiol Prot* 2023;43(3)（[doi:10.1088/1361-6498/ace4d4](https://doi.org/10.1088/1361-6498/ace4d4)；与 Bellamy 2022 不同） | ICRP 体素体模 MC + 电离室验证；131I 病房 | 紧挨照护者的屏蔽剂量最低，但只覆盖很小的面积；放在患者尾侧的屏蔽在大面积上有适度降低；标准宽度屏蔽只把剂量降约 **4 倍** | CC BY（Europe PMC） | 校准任何居家"屏蔽"建议；代理训练集的 MC 方法 |
| PHITS + ICRP 成年男性 MRCP，131I 病房护士剂量 | — | 护士照射在 131I 给药后 1–13 h 内达峰【search-summary, verify】 | 未知 | MRCP 在 PHITS 中的先例（[ResearchGate](https://www.researchgate.net/publication/403400290_RADIATION_DOSE_ESTIMATION_FOR_NUCLEAR_MEDICINE_NURSES_USING_THE_MONTE_CARLO-BASED_PHITS_SIMULATION_CODE)） |
| 同床，131I：一项 JNM 研究在枕头上放 TLD 代表睡在 33 cm 外的伴侣 | 实测 | 2.7 GBq 时 3–5 天 0.15–0.18 mSv；4.1 GBq 时 0.28–0.37 mSv；作者认为 13–18 天的睡眠限制过严 | 无 | "伴侣同床"场景的验证【search-summary, verify；作者和年份未确认】（[JNM node 34438](https://jnm.snmjournals.org/node/34438)） |
| 密歇根大学致 *Thyroid* 的信 | — | 限制 8 天后每天与孩子同睡 8 h，单此一项就可能"大幅超过 5 mSv"；建议 ≥2 周、可能 4 周内不与孩子同睡或午睡 | — | 儿童同睡最坏情况【search-summary, verify】（[Deep Blue PDF](https://deepblue.lib.umich.edu/bitstream/2027.42/90468/1/thy-2E2011-2E0181-2Ers.pdf)） |
| 阿曼门诊甲亢研究（SQUMJ） | TLD | 家属剂量 <1 mSv；一名 7 岁儿童与父母同床；支持门诊治疗至 800 MBq | — | 验证【search-summary, verify】（[PMC3074783](https://pmc.ncbi.nlm.nih.gov/articles/PMC3074783)） |
| Meo SL 等，*Nucl Med Commun* 2008 | 90Y-Zevalin 患者（n = 8） | MC 与电离室空气比释动能率 4.2 ± 0.1 vs 4.4 ± 0.8 µGy/h | 无 | 只是 MC-实测一致性的例子（轫致辐射，时间早于 2015）（[PMID 18987532](https://pubmed.ncbi.nlm.nih.gov/18987532/)） |

### 照护者实测剂量：实测普遍远低于保守计算

儿科 177Lu（Lu 2025：1 m 处计算 0.44 mSv，实测 0.245 mSv）和 131I-MIBG（Willegaignon 2018：潜在剂量 11.9 mSv，实测 1.88 mSv）都显示，实测照护者剂量远低于保守计算。这支持 IRSA 的主张，即个性化可以在不突破约束的前提下放宽限制；但这些都是小样本、单中心研究【推断】。Kadhim 2020 发现照护者剂量与活度无关，说明行为起主导作用；Brudecki 2020 发现家属甲状腺 131I 与吃患者做的饭有关，揭示了一条物理引擎会漏掉的内污染途径。

| 引用 | 设计/n/核素 | 关键数字 | 数据/代码 | 对 IRSA 的用途 |
|---|---|---|---|---|
| Hartmann H 等，*RöFo* 2023;195:605–612（[doi:10.1055/a-2015-0475](https://doi.org/10.1055/a-2015-0475)） | **147 例放射性碘治疗患者及家属**，测量 14 天，分析 88 组数据（德累斯顿） | 指环剂量计：家属平均 **0.75 mSv**，患者 **10 mSv**（0.1–50 mSv）；睡眠时佩戴 OSL 和 TLD 结果相近，但 **OSL 的重复性更好** | 付费 | IRSA 照护者验证组的剂量计选择（OSL 优于 TLD）；睡眠期剂量测量可行 |
| Kadhim AA 等，*Radiat Prot Dosimetry* 2020;190(2):208–216（[doi:10.1093/rpd/ncaa092](https://doi.org/10.1093/rpd/ncaa092)） | 非癌症（甲亢）131I 患者的照护者，185–1110 MBq，TLD 佩戴 12–28 天 | 照护者平均 **0.15 ± 0.15 mSv**；与活度、离医院距离、照护者年龄、教育、交通方式都无关；高于癌症患者的照护者 | 付费 | 甲亢照护者先验；行为主导，支持行为感知的指导 |
| Brudecki K 等，*Radiat Environ Biophys* 2020;59(3):559–564（[PMC7368868](https://pmc.ncbi.nlm.nih.gov/articles/PMC7368868/)） | **20 名家属**，3–72 岁，全身谱仪测甲状腺；131I（波兰） | 20 人中 11 人甲状腺可测到 131I，**9–1140 Bq**；最高有效剂量 **218 µSv**，甲状腺当量剂量 **2.4 mSv**；与住房条件和**吃患者做的饭**有关 | CC BY | 内污染途径的证据；支持"不要给别人做饭"规则 |
| Willegaignon J 等，*Clin Nucl Med* 2018;43(8):572–578（[doi:10.1097/rlu.0000000000002158](https://doi.org/10.1097/rlu.0000000000002158)） | **23 名神经母细胞瘤儿童**，131I-MIBG 8.55 ± 1.69 GBq（巴西） | 有效半衰期 **23 ± 5 h**；1 m 处最大潜在剂量 **11.9 ± 3.4 mSv**，97% 在 120 h 内；实测照护者剂量（n = 22）**1.88 ± 1.85 mSv** | 付费 | 儿科患者、父母照护者的极端情形；实测约为计算潜在剂量的 1/6 |
| Lu Z 等，*Clin Nucl Med* 2025（[doi:10.1097/rlu.0000000000005763](https://doi.org/10.1097/rlu.0000000000005763)） | **23 名 3–13 岁神经母细胞瘤儿童**，177Lu-DOTATATE 3.687 ± 1.545 GBq；8 个时间点剂量率；照护者 TLD（中国） | 双指数有效半衰期 **1.4 ± 0.8 h 和 52.3 ± 18.6 h**；1 m 处估计最大累积剂量 **0.442 ± 0.174 mSv**；实测照护者 **0.245 ± 0.063（0.150–0.390）mSv** | 付费 | 近距离照护的 177Lu 计算-实测配对；儿科 177Lu 动力学先验 |
| Bhatia N 等，*World J Nucl Med* 2023;22(3):203–207（[PMC10581751](https://pmc.ncbi.nlm.nih.gov/articles/PMC10581751/)） | **134 例门诊**（102 例甲亢、32 例残余消融），131I（印度） | 5 cm（颈部和胃部）及 1 m 处照射率，0/1/2 h；颈部甲亢 vs 消融：0 h 6.9 vs 22.27 mR/h，2 h 41.75 vs 48.90 mR/h | CC BY | LMIC 门诊的早期 131I 先验（IAEA TC） |
| Ravichandran R 等，*Indian J Nucl Med* 2022;37(1):108–111【title-only】 | 关于高剂量 131I 后照护者杂散照射的通信 | — | CC BY-NC-SA | 背景（阿曼）（[PMC9037879](https://pmc.ncbi.nlm.nih.gov/articles/PMC9037879/)） |

### α 核素、污染、排泄与废物：马桶是三种核素共同的热点

近期病房表面研究一致发现，**马桶是 177Lu 的主要污染热点**，177Lu-DOTATATE 治疗后病房残留污染远低于 131I（Ideguchi 2026）；225Ac 擦拭阳性也集中在马桶座和地面（Dahlke 2025）。177Lu-PSMA 的唾液源项约为 177Lu-DOTATOC 的 4 倍（Schedeit 2025）。污水中的 177Lu 剂量可以忽略（德黑兰模型约 8.7 µSv/年），但废物储存时间取决于**生产路线**：含 177mLu 的产品超过 120 天仍高于本底，无载体产品约第 77 天降到本底（Tsuchiya 2026）。对 α 核素，外照射可以忽略，风险在于唾液、汗液、尿液和失禁造成的内污染；剂量率可穿戴设备对 225Ac 帮助不大，有用的是污染行为指导【推断】。据此，IRSA 的污染指导可以简短地以马桶为中心：坐着排尿、冲两次水、擦马桶座、筛查失禁；177Lu-PSMA 再加约 2 天的唾液规则【推断】。**没有找到任何试验或前后对照研究检验过卫生指导在患者家中是否有效**，这是 IRSA 前瞻性队列可以填补的缺口。

| 引用 | 设计/n/核素 | 关键数字 | 数据/代码 | 对 IRSA 的用途 |
|---|---|---|---|---|
| Wanke C 等，*Z Med Phys* 2021;31(1):58–64（[doi:10.1016/j.zemedi.2020.09.002](https://doi.org/10.1016/j.zemedi.2020.09.002)） | 德国多中心，门诊 **223Ra**；1 m 和 2 m 剂量率、家中擦拭、唾液和汗液样本 | 注射后 10–20 min 外照射 **1 m 处中位 <0.080 µSv/h**（最大 0.208）；唾液中位 **27 Bq/g**（12.6–110）；前 24 h 汗液中位 **0.121 Bq/cm²**；家中卫生间和厨房污染中位 0.026–0.215 mBq/cm²（最大 2.1）；家属照射估计**远低于 1 mSv** | 付费 | 225Ac 指导的模板：α 核素外照射可忽略，指导应聚焦卫生和污染；唾液和汗液源项 |
| Pratt BE 等，*Nucl Med Commun* 2018;39(2):125–130（[doi:10.1097/mnm.0000000000000783](https://doi.org/10.1097/mnm.0000000000000783)） | 6 例 × 2 次 223Ra；排泄物计数 + 全身探头 | 累计排出 8 h 2.6%，**48 h 39%**；约 1 周中位 70%，6 周 86%；6 周骨滞留 11–60% | 付费 | α 核素卫生时段；尿与粪的比例在未读的全文中 |
| Katzdobler L 等，*Radiat Environ Biophys* 2026;65(2):741–756（[doi:10.1007/s00411-025-01193-5](https://doi.org/10.1007/s00411-025-01193-5)） | 223Ra 后呼出 219Rn 的 ICRP 生物动力学建模 | 输注后约 1.5 min 室内 219Rn 达峰，**每 Bq 输注 5.8 × 10⁻² Bq**；219Rn 加气载子体 ≤1.2 × 10⁻¹ Bq/Bq | 付费 | α 衰变链模块的吸入途径（可能很小，但已量化）；照护者剂量结果在摘要中被截断 |
| **Dahlke PM 等**，*J Nucl Med* 2025;66(suppl 1):251831【abstract-only】（[JNM](https://jnm.snmjournals.org/node/128470)） | 6 名男性（3 例 225Ac/177Lu-PSMA-I&T 串联，3 例 225Ac-DOTATOC），225Ac 5.66–8.56 MBq；176 份擦拭（Charité） | 6 次治疗中 **4 次**有污染；治疗后 128 份擦拭中 **12 份（9.4%）**阳性；部位为水龙头、马桶座、地面；0.2–274.2 Bq（中位 20.6）；尿失禁最严重的两人中位 92.2 Bq；阳性多在 24–48 h；门把手、遥控器、手机无污染 | 摘要 | 支持 IRSA 对 225Ac 设**失禁筛查问题**；与 CLEAN-225Ac 的关系见矛盾清单 |
| Schmitl S 等，*J Nucl Med* 2025;66(8):1239–1244（[doi:10.2967/jnumed.125.269655](https://doi.org/10.2967/jnumed.125.269655)） | 回顾性，**n = 15**，[225Ac]Ac-SibuDAB，每 8 周 6 MBq（维也纳） | 3 例测了 213Bi 分布，给药后"尿中立即出现大量净 213Bi 过量"【snippet】 | 付费 | 支持 225Ac 早期严格的如厕卫生 |
| Jokar N 等，*Semin Nucl Med* 2026;56(6):1231–1247（[doi:10.1053/j.semnuclmed.2026.09.002](https://doi.org/10.1053/j.semnuclmed.2026.09.002)） | 综述：靶向 α 治疗的安全、剂量学、辐射防护与监管 | 反冲与子体再分布、污染控制、废物处置、按辖区的规则；区分既定做法与机构自选做法 | 付费 | 225Ac 模块的框架参考 |
| Ideguchi R 等，*Ann Nucl Med* 2026;40(7):776–785（[doi:10.1007/s12149-026-02183-3](https://doi.org/10.1007/s12149-026-02183-3)） | **20 间 177Lu-DOTATATE 病房 vs 31 间 131I 病房**（长崎） | 177Lu 残留污染显著低于 131I；177Lu 的高周围剂量**只在床边和淋浴间**，高表面污染**只在马桶内** | 付费 | 居家热点图先验（马桶优先）；177Lu 的清洁建议可以比 131I 轻 |
| Karo C 等，*Health Phys* 2019;117(4):419–425（[doi:10.1097/hp.0000000000001063](https://doi.org/10.1097/hp.0000000000001063)） | 131I 隔离病房（3.7–5.5 GBq），每次出院后监测 3 周 | 职业风险可忽略；监测点含马桶 U 形弯、水槽、枕头、门把手 | 付费 | 131I 居家清洁的污染点清单 |
| Okuhata K 等，*Ann Nucl Med* 2025;39(9):966–972（[doi:10.1007/s12149-025-02064-1](https://doi.org/10.1007/s12149-025-02064-1)） | 疏水氟聚氨酯膜涂马桶的台架 + 临床测试；177Lu-DOTATATE | 接触角 >90° 的膜使计数降低 **92.1–97.4%**；涂膜马桶冲水后表面污染比未涂膜低 **62.2%**，降到本底以下 | 付费 | 失禁患者或有幼儿家庭的"马桶垫/涂层"选项；证明尿液飞溅污染真实可测 |
| Schedeit C 等，*Clin Oral Investig* 2025;29(5):241（[PMC11991938](https://pmc.ncbi.nlm.nih.gov/articles/PMC11991938/)） | 唾液采样至 45–69 h；**177Lu-PSMA-I&T n = 14**，177Lu-DOTA-TOC n = 6（伯尔尼） | 唾液峰值 4 h（PSMA）vs 2 h（DOTATOC）；PSMA 中位峰值约**高 4 倍**；生物半衰期 **14 h vs 8.5 h**；平均 AUC **895 vs 96 kBq·h/ml** | CC BY | 共用餐具、接吻、看牙的唾液源项；PSMA 唾液约 2 天内偏高 |
| Tsuchiya J 等，*Ann Nucl Med* 2026;40(12):1564–1569（[doi:10.1007/s12149-026-02267-0](https://doi.org/10.1007/s12149-026-02267-0)） | 每种生产路线 6 瓶 + 177Lu-DOTATATE 7.4 GBq × 4 后患者尿液废物；1 m 处 NaI 测至 120 天 | 直接路线（含 177mLu）的药瓶和尿液**超过 120 天**仍可测；无载体药瓶和尿液分别在**第 73.5 天和第 77.0 天**降到本底 | 付费 | 给出尿布、护垫、造口袋的储存或处置建议前，IRSA 需知道产品是有载体还是无载体 |
| Prevot S 等，*EJNMMI Phys* 2023;10:3（[PMC9829929](https://pmc.ncbi.nlm.nih.gov/articles/PMC9829929/)） | 前 **65 次 Lutathera 治疗**的废物（第戎） | 177mLu 占活度 0.3%，半衰期 **152 d（104–205）**；部分使用的药瓶 5 年后、其他废物 3 年后可处置 | CC BY | 废物衰变计算的 177mLu 定量项 |
| Deevband M 等，*Asia Ocean J Nucl Med Biol* 2026:130–134（[PMC13438261](https://pmc.ncbi.nlm.nih.gov/articles/PMC13438261/)） | 德黑兰市政污水中 177Lu 的模型 | 公众剂量 **8712.9 nSv/年**（约 8.7 µSv/年）；177Lu 不需要化粪池或衰变池 | CC BY 3.0 | 支持"正常使用马桶即可"的公众信息 |
| Puspita TD 等，*J Radiol Prot* 2023;43(3)（[doi:10.1088/1361-6498/ace6ad](https://doi.org/10.1088/1361-6498/ace6ad)） | 131I 治疗期间医院污水处理厂 HPGe 采样（西爪哇） | 衰变池 **95.9 × 10⁶ Bq/m³**；与其他污水混合后 472,680 Bq/m³，低于 10⁷ Bq/m³ 清洁解控水平 | CC BY | LMIC 的 131I 污水情境（IAEA TC） |
| Ryu CJ，*Nucl Med Commun* 2025;46(3):187–192（[PMC11792983](https://pmc.ncbi.nlm.nih.gov/articles/PMC11792983/)） | 低剂量 131I（1.11 GBq）治疗废物 | 降到容许限以下的天数：内容器 6、外容器 1、治疗用塑料 5 | CC BY-NC-ND | 131I 废物的简单衰变储存时长 |
| Schomäcker K 等，*Int J Mol Sci* 2026;27(2):590（[PMC12841383](https://pmc.ncbi.nlm.nih.gov/articles/PMC12841383/)） | 综述 + 自有患者与小鼠数据 | 131I 治疗后呼出总活度极少，但 **>95% 为挥发性有机形态**，能逃过标准过滤器 | CC BY | 通风建议保持温和（低优先级） |
| Kumar SA 等，*Clin Nucl Med* 2026【title-only】；Chevalier B 等，*J Nucl Med* 2026；Al Saidi H 等，*J Nucl Med Technol* 2026【title-only】 | 177Lu-DOTATATE 治疗后显像中的尿液污染"环形征"；血液透析 131I 患者经验；终末期肾病 131I | — | — | 皮肤和衣物尿污染示意；IRSA 应排除或标记透析患者（[doi:10.1097/rlu.0000000000005990](https://doi.org/10.1097/rlu.0000000000005990)；[doi:10.2967/jnumed.125.271707](https://doi.org/10.2967/jnumed.125.271707)） |

### 旅行、妊娠、火化与急诊：LMIC 合乘交通是 IAEA 场景的新用例

| 引用 | 设计/n/核素 | 关键数字 | 数据/代码 | 对 IRSA 的用途 |
|---|---|---|---|---|
| Mongane MS 等，*Ann ICRP* 2026（[doi:10.1177/01466453251411730](https://doi.org/10.1177/01466453251411730)） | **62 例甲亢 131I 患者**乘合乘患者交通车 4–9 h（南非自由州） | 病房剂量 **387 ± 185 µSv**（18.6 ± 6.9 h）；**0.3 m 处 217.2 ± 45.8 µSv/h**，**1 m 处 25.4 ± 5.4 µSv/h**；不考虑社会经济因素就按 25 µSv/h 或 555 MBq 放行门诊患者，可能抬高乘客剂量 | 付费 | 契合 IAEA TC 的 LMIC 合乘交通场景；邻座 0.3–1 m 几何 |
| Kashima A 等，*Kaku Igaku* 2026;63(1):31–34（[doi:10.18893/kakuigaku.tr.2601](https://doi.org/10.18893/kakuigaku.tr.2601)） | 7 年审计，**6,132 例**核医学（诊断）出行前筛查 | 1.29%（79 例）计划国际出行（非疫情期 2.24%）；62% 在预约时告知，38% 当天才说；措施：改期 11 次、换药 1 次、紧急证明 3 份；无机场事件报告 | 付费 | 引导流程应尽早询问出行计划并生成出行证明 |
| Sillanpaa J、Yuan J，*Health Phys* 2026;131(3):173–178（[doi:10.1097/hp.0000000000002084](https://doi.org/10.1097/hp.0000000000002084)） | 2005–2024 年美国 NRC 意外胎儿照射事件报告 | 39 起，28 起为治疗；**71.8% 涉及 131I**；治疗事件平均胎儿剂量 **260 mSv**（20–860）；28 起治疗事件中 21 起做过妊娠试验 | 公开 NRC 报告 | 妊娠护栏和治疗后受孕建议（2 起在给药后受孕）；这些是患者本人怀孕，不是接触者 |
| Sozio S 等，*EJNMMI Rep* 2026【title-only】 | 131I 治疗前妊娠检测建议的文献综述 | — | OA | 治疗前妊娠检查的规则来源（[PMC13654911](https://pmc.ncbi.nlm.nih.gov/articles/PMC13654911/)） |
| Yu NY 等，*JAMA* 2019;321(8):803–804【title-only】 | 研究通信：放射性药物治疗患者火化后的放射性污染（Mayo Clinic Arizona） | 细节未取得 | — | 2018–2026 年主要火化文献；引用数字前先读全文（[PMC6440253](https://pmc.ncbi.nlm.nih.gov/articles/PMC6440253/)） |
| Nakagawa K 等，*Surg Case Rep* 2026;12(1)（[PMC13283750](https://pmc.ncbi.nlm.nih.gov/articles/PMC13283750/)） | 病例报告：**177Lu-DOTATATE 后第 6 天**急诊腹腔镜阑尾切除 | **1 m 处 1.1 µSv/h**；术野 20 cm 处 8.4–12.2 µSv/h；尿液 5 cm 处 0.2 µSv/h；手术安全完成 | CC BY | "计划外就医或手术"场景；支持"不要延误急诊"的医护信息 |

酒店住宿后 RPT 的同行评审研究、177Lu 机场报警的频率、孕妇接触者剂量的直接测量、177Lu-PSMA 火化场剂量学系列，本轮都没有找到。

### 远程监测、数字教育与患者体验：患者要的是"有保障的早出院"

**Gallicchio 2021 证实了上轮无法核实的"意大利 CNR 设备"**：20 例 DTC 住院患者佩戴无线可穿戴设备，与防护病房内的固定"家用设备"对比，约 56 h 内约 45,580 次读数（每小时 53 次），按 2 h 平均，r = 0.96（p < 0.0001），Bland–Altman 显示一致（[PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/)）。第 2 天末佩戴读数 47.08 µSv/h，固定设备 24.96 µSv/h（差异无统计学意义），佩戴读数约为固定设备的 1.9 倍（笔记计算）。这说明 IRSA 的反演模型必须修正几何偏倚：限制精度的是近场几何，而不是灵敏度【推断】。加上 WIDMApp 和华盛顿大学背心，已经有三个团队在院内证明可穿戴设备能追踪滞留；**还没有人发表出院后的居家可穿戴数据**，这仍是 IRSA 最清楚的实证空白。患者偏好方面，德国 Charité 的调查显示 95.7% 重视住院监护，53.2% 希望住得短一些，54.4% 认为半住院最理想，只有 8.9% 支持完全门诊；教育材料和保证电话可达被评为重要（[doi:10.1055/a-2966-9616](https://doi.org/10.1055/a-2966-9616)）。这支持把 IRSA 定位为"早出院的远程监督"，而不是"取代医院"【推断】。焦虑和睡眠障碍在 131I 治疗和住院期间达到高峰，"辐射暴露担忧"是痛苦的独立预测因子，IRSA 因此可能带来心理获益，试验应同时预注册 GAD-7/PSQI 类终点和剂量终点【推断】。

| 引用 | 设计/n/核素 | 关键数字 | 数据/代码 | 对 IRSA 的用途 |
|---|---|---|---|---|
| **Gallicchio R 等**，*IEEE J Transl Eng Health Med* 2021;9:2700306（[PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/)） | 可行性研究，**n = 20** 例 DTC 住院，131I；无线可穿戴设备 vs 防护病房固定设备 | 见上文；第 1 天末 80.81 vs 60.77 µSv/h | CC BY；数据未查 | **院内配对测量可直接照搬的协议**（Bland–Altman，2 h 分箱）；取代上轮的"意大利 CNR 设备"条目 |
| SELFIE，NCT04922801（King's College London）（[ClinicalTrials.gov](https://clinicaltrials.gov/study/NCT04922801)） | 177Lu-DOTATATE 或 131I 后患者用手持仪测 28 天；主要终点比较患者自测剂量率 TAC 与 γ 显像 TAC | 登记**已完成，目标 50 人；未贴出结果，2026-11 未找到论文** | — | 与 IRSA 可穿戴有效半衰期估计最接近的先例；联系 PI |
| Carr A 等，*JMIR Cancer* 2026;12:e84151（[PMC13714524](https://pmc.ncbi.nlm.nih.gov/articles/PMC13714524/)） | 纵向可用性研究，**n = 8** 例 RAI 后 DTC 幸存者；4 周网页干预（RAI Support） | **SUS 88.1 ± 8.7**；任务全部完成；易用性 87.5–100%；错误率 0–25%；自我效能、整体心理健康和 HRQOL 的初步效应量 r = 0.31–0.45 | CC BY | 现成的可用性协议（出声思考、SUS、4 周居家使用） |
| Giannoula E 等，*JMIR Res Protoc* 2020;9(3):e13409（[PMC7101500](https://pmc.ncbi.nlm.nih.gov/articles/PMC7101500/)） | DTC 患者和照护者 mHealth App 的准实验试点方案（塞萨洛尼基） | 诊断和 RAI 治疗信息、病历模块、生活质量终点 | CC BY | 131I 患者 App 的先有技术；查是否已发表结果 |
| O'Brien SR 等，*Clin Imaging* 2026;129:110678【title-only】 | 专门的 RPT 门诊 + 远程医疗（Penn） | — | — | 临床端仪表盘的远程随访模式（[doi:10.1016/j.clinimag.2025.110678](https://doi.org/10.1016/j.clinimag.2025.110678)） |
| Samson L 等，*J Cancer Educ* 2025;40(3):450–455（[doi:10.1007/s13187-024-02531-9](https://doi.org/10.1007/s13187-024-02531-9)） | 系统评估 **50 个** RAI 相关 YouTube 视频 | 26 个是近 3 年的；中位时长 4:53；发布者：非营利 15、个人 12、医疗机构 11；"副作用与风险"覆盖最多 | 付费 | 患者实际找到的信息；IRSA 内容应覆盖同样主题、来源更好 |
| **Kaufmann J 等**，*Nuklearmedizin* 2026（[doi:10.1055/a-2966-9616](https://doi.org/10.1055/a-2966-9616)） | 匿名横断面调查，**n = 46** 例 RLT 患者（177Lu-PSMA 和 177Lu-DOTATATE；德国强制住院） | 见上文；障碍是对他人的辐射照射和缺乏医疗监督；体能状态越好，接受度越高 | CC BY（Europe PMC） | 直接的设计输入：升级路径（热线或临床联系人）和教育功能；问卷条目用于接受度研究 |
| Huang W 等，*Front Endocrinol* 2026;17:1743676（[PMC12907198](https://pmc.ncbi.nlm.nih.gov/articles/PMC12907198/)） | 纵向，**n = 160**（148 完成）DTC，131I；治疗前、住院期、治疗后 | 睡眠障碍 **35.8% → 52.9% → 32.6%**；焦虑 **23.6% → 40.7% → 14.0%**；抑郁 **47.3% → 56.4% → 33.3%** | CC BY | 任何"减轻痛苦"主张的基线效应量；测量时间表；摘要另一句与轨迹矛盾，见矛盾清单 |
| Zeng L 等，*Front Endocrinol* 2026;17:1877295（[PMC13368564](https://pmc.ncbi.nlm.nih.gov/articles/PMC13368564/)） | 预测模型，**n = 247** 例 DTC，131I（太原） | 痛苦的独立预测因子：睡眠差、TSH 刺激、肺转移、社会支持、**辐射暴露担忧**、低 25-OH 维生素 D；内部验证 AUC 0.82（bootstrap）、0.77（10 折） | CC BY | "辐射暴露担忧"是 IRSA 直接针对的可改变目标，可作次要终点 |
| Yang W 等，*Patient Prefer Adherence* 2026;20:622443（[PMC13499999](https://pmc.ncbi.nlm.nih.gov/articles/PMC13499999/)） | 横断面知信行调查，**n = 386** 例术后甲状腺癌 131I 患者（贵州 4 家医院） | 知识 **9.98 ± 4.95 / 24**（低）；态度 22.71/30；行为 31.53/45；知识（β = −0.169）和行为（β = −0.360）越好，症状越少 | CC BY-NC；问卷自编 | IRSA 前后评估的知信行条目；教育与痛苦降低相关的证据 |
| Starkbaum J 等，*Support Care Cancer* 2025;33(4):325（[PMC11950005](https://pmc.ncbi.nlm.nih.gov/articles/PMC11950005/)） | 质性，**7 次访谈**（法国），177Lu-DOTATATE | 主题：可及性障碍；**辐射隔离的心理和社会影响**；以患者为中心建立信任 | CC BY | 用户需求文件的引语和主题 |
| Peet C 等，*Support Care Cancer* 2026;34(10):1000（[PMC13597573](https://pmc.ncbi.nlm.nih.gov/articles/PMC13597573/)） | 前瞻性整体需求评估，**n = 50** 例儿科和青少年 MRT（131I NaI、131I-mIBG、177Lu-DOTATATE），UCLH，中位 13 岁 | 关切：心理社会支持有限、路费和误工、**强制隔离和与家人分离** | CC BY | 模拟患者基准中的儿科患者和父母人设 |
| Whittaker L 等，*Nucl Med Commun* 2025;46(4):373–377（[PMC11878586](https://pmc.ncbi.nlm.nih.gov/articles/PMC11878586/)） | 患者与公众参与会议：10 名亲历者 + 专业人员，加后续问卷（儿科 MRT） | 家庭报告的困难；促成了更好的信息材料 | CC BY | IRSA 患者与公众参与流程的模板 |
| Li P 等 2026；Bondet de La Bernardie S 等 2026；Carr AL 等 2026；Liu Q 等 2026（n = 500 Graves 病） | 横断面；RAI 后 HRQoL 系统综述；症状负担；RAI 后生活质量与决策后悔 | 痛苦与生活质量负相关等 | — | HRQoL 效应量背景【title-only / 部分摘要】（[PMC13494376](https://pmc.ncbi.nlm.nih.gov/articles/PMC13494376/)；[doi:10.1177/10507256261499230](https://doi.org/10.1177/10507256261499230)；[doi:10.1111/cen.70092](https://doi.org/10.1111/cen.70092)） |

没有找到直接测量出院后限制依从性的定量研究（日记或传感器测得的距离），依从性只能从剂量学间接推断；也没有找到 177Lu-PSMA 居家限制（而不是住院隔离）造成的孤独感或照护负担的定量研究，以及 131I 或 177Lu 后"数字 vs 纸质出院指导"的 RCT。

### AI、MC 代理与 LLM 评测：知识已不是瓶颈，法规、计算和多语言才是

MC 训练的 ML 代理在职业剂量领域已经成熟：介入放射学中，集成模型平均相对误差约 3.7%，3D U-Net 推理约 0.2 s。IRSA 是把这种模式用到"患者作为源、公众作为接收者"的问题上，论文应当表述为**新应用而不是新方法**【推断】。LLM 方面，前沿模型对放射防护和医学物理知识的回忆已经很好（85–95%），反复出现的弱点是**特定辖区的法律**（Goto 2024）、**计算**（Bertschmann 2026）、**非英语输出**（Habouzit 2026）、**依机构而定的住院规则**（Habouzit 2026）和**随时间漂移**（Effraimidis 2026）。这些正好对应 IRSA 计划的分工：规则放进带版本号的引擎，数字来自物理引擎，LLM 只负责措辞【推断】。所有放射防护/核医学患者问题研究都是小样本（12–115 道专家挑选的问题）、单轮、专家 Likert 评分，没有一项发布可复用的带许可题集。**仍然没有同行评审研究对照 IAEA、ICRP 或国家指南评价 LLM 对 RPT 后防护指导（距离、如厕、出行、妊娠）的回答。** 有引擎落地的 LLM（数字由引擎给出、LLM 只负责表述）在文献中没有直接对照，这本身就是 IRSA 概念论文可以引用的空白【推断】。

| 引用 | 设计/n/核素 | 关键数字 | 数据/代码 | 对 IRSA 的用途 |
|---|---|---|---|---|
| Harb H 等，*J Radiol Prot* 2026;46(2)（[doi:10.1088/1361-6498/ae63da](https://doi.org/10.1088/1361-6498/ae63da)） | **10,000 张 MC 剂量图**；深度网络和树模型；介入放射学上肢剂量 | 集成模型平均**相对误差 3.69%**；梯度陡峭的极端几何误差最大 | 付费 | MC 训练代理的直接方法先例，也是其分布外弱点的先例 |
| Harb H 等，*Radiol Phys Technol* 2026;19(2):699–710（[doi:10.1007/s12194-026-01043-z](https://doi.org/10.1007/s12194-026-01043-z)） | GPU Geant4（GGEMS）MC，残差与 transformer 3D U-Net；医生三维剂量 | MAE <0.06 nGy；γ 通过率 >90%；**推理约 0.2 s** | 付费 | 实时代理的速度和精度参照 |
| Hışıroğlu KA 等，*Radiat Prot Dosimetry* 2025;201(10):690–700（[doi:10.1093/rpd/ncaf064](https://doi.org/10.1093/rpd/ncaf064)） | 虚拟血管造影室 MC；5 种 ML 算法 | 梯度提升最擅长不用物理剂量计，由空间和操作参数预测员工剂量 | 付费 | 又一个代理先例 |
| Fazli Z 等，*Environ Health* 2025;24:32（[PMC12082979](https://pmc.ncbi.nlm.nih.gov/articles/PMC12082979/)）；Obrycka M 等，*Nucl Med Mol Imaging* 2026;60(5):490–515（[PMC13627506](https://pmc.ncbi.nlm.nih.gov/articles/PMC13627506/)） | 范围综述：2,920 条记录 → **59 项**职业照射 AI 研究；PRISMA 系统综述：**24 项** 131I 路径中的 AI 研究 | 后者的应用含**辐射安全评估**，证据多为回顾性、单中心、无外部验证 | CC BY-NC-ND；CC BY | 相关工作引用；AI 用于 131I 辐射安全的证据很薄 |
| Damilakis J 等（ICRP 第 3 委员会 AI 工作组），*Phys Med* 2026;148:105906（[doi:10.1016/j.ejmp.2026.105906](https://doi.org/10.1016/j.ejmp.2026.105906)） | 观点 | 最成熟：决策支持、重建、勾画、计划；**患者特异剂量学、职业剂量预测和预测性安全分析**有前景但验证尚早；挑战：偏倚、黑箱问责、法规协调 | CC BY | IRSA 治理部分的权威框架 |
| **Roemer G、Li A、Mahmood U、Dauer L、Bellamy M**，*J Radiol Prot* 2024;44(1)（[doi:10.1088/1361-6498/ad1fdf](https://doi.org/10.1088/1361-6498/ad1fdf)） | 模拟保健物理认证考试的 **1,064 道题**，5 个领域 | **GPT-3.5 45.3%，GPT-4 61.7%**，都低于 67% 及格线；未做提示工程 | CC BY；题集未声明发布 | 与放射防护最直接相关的 LLM 基准（MSKCC，Bellamy 所在团队）；询问题集能否用于 IRSA 评测 |
| Goto H 等，*Cureus* 2024;16(11):e74262（[PMC11663497](https://pmc.ncbi.nlm.nih.gov/articles/PMC11663497/)） | 日本"X 射线作业主任者"考试 4 套（2023–24） | GPT-4o **57.5–70.0%**，o1-preview **71.1–86.5%**；GPT-4o **法律法规**科目不及格，o1-preview 全部通过（不含图像题） | CC BY | 法规知识是最弱环节；支持把辖区规则放进带版本号的规则引擎 |
| Kadoya N 等，*J Radiat Res* 2026;67(1):114–120（[PMC12856024](https://pmc.ncbi.nlm.nih.gov/articles/PMC12856024/)） | 日本 3 项考试（医学物理师、放射科专科、放疗专科）全部选择题 | ChatGPT-5 84.7%；**ChatGPT-5 Pro 94.7%**；Grok 4 78.4%；Grok 4 heavy 81.6%；Gemini 2.5 Pro 88.9% | CC BY | 前沿模型知识水平很高；IRSA 的风险在多轮患者建议和计算，不在回忆 |
| Bertschmann J 等，*Front Oncol* 2026;16:1738955（[PMC13225992](https://pmc.ncbi.nlm.nih.gov/articles/PMC13225992/)）；另见 PubMed 42239883（[链接](https://pubmed.ncbi.nlm.nih.gov/42239883/)） | 2023 年 ASTRO 放射生物学学习指南的 **335 道**选择题，每题 3 个月内问 5 次 | GPT-4 81%，GPT-3.5 62%，Llama-2 51%；**计算题都弱** | CC BY；题集受版权限制 | 支持"LLM 负责解释，物理引擎负责计算"；两条记录很可能是同一研究 |
| GPT-5 on ACR TXIT，arXiv 2508.21777；Holmes 2023；Huang 2023；Bharati 2026 | 300 道 TXIT（GPT-5 92.8%，GPT-4 78.8%，GPT-3.5 62.1%）+ 60 个真实放疗病例；自建 100 题放疗物理考试；TXIT + 灰区病例；11 道肿瘤医学物理题（ChatGPT 5.2 / DeepSeek V3.2 / Gemini 3.0 正确性 3.4/3.81/3.09，均不能替代专家） | — | 病例集发布未确认；Huang 有 GitHub 补充 | 背景：知识已不是瓶颈，落地和安全行为才是（[arXiv 2508.21777](https://arxiv.org/pdf/2508.21777v1)；[PMC10388568](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10388568/)；[PMC13355276](https://pmc.ncbi.nlm.nih.gov/articles/PMC13355276/)） |
| **Jankowski 等**，*Radiology* 2024;311(3):e233465（Lausanne + Duke） | 从放射科机构网站取 **12** 道患者辐射防护问题，ChatGPT 回答对比网站回答；**12** 名盲评者（放射科医生、医学物理师、技师各 4 名） | 科学充分性中位 5.6 vs 5.4；公众理解 5.1 vs 5.6；ChatGPT 中位 268 词 vs 173 词；评者指出"误导或遗漏关键信息" | 未发布数据集 | 最接近的放射防护患者研究；评分维度可作 IRSA 细则维度；"更长、更难懂"支持长度和可读性约束（[AuntMinnie](https://www.auntminnie.com/imaging-informatics/artificial-intelligence/article/15678367/chatgpt-can-provide-patients-with-radiation-protection-info)） |
| Yalamanchili 等，*JAMA Netw Open* 2024（[PMC10988356](https://pmc.ncbi.nlm.nih.gov/articles/PMC10988356)） | 4 个专业学会网站的 **115** 道常见放疗患者问题，GPT-3.5 | 专家认为 **94%**（相对正确性）和 **77%**（完整性）与学会答案相当或更好；阅读水平约为大学；遗漏定位纹身等细节 | 题目来自公开 FAQ，未发现单独许可数据集 | "学会 FAQ → 参考答案"题目模板；阅读水平是反复出现的失败点 |
| Speicher T 等，*Am J Nucl Med Mol Imaging* 2025;15(4):146–152（[PMC12444397](https://pmc.ncbi.nlm.nih.gov/articles/PMC12444397/)） | **12** 道 PRRT 患者 FAQ；9 名专业人员盲评，324 次评估；ChatGPT-4o vs DeepSeek V3 | 准确性 4.43 vs 4.56（无显著差异）；简洁性 4.55 vs 4.24（p = 0.0013）；**7–8% 医学上不正确** | Europe PMC：非 OA | PRRT 患者教育基线（上轮缺 177Lu-DOTATATE）；IRSA 必须优于这一错误率 |
| **Habouzit V 等**，*J Cancer Educ* 2026（[doi:10.1007/s13187-026-02886-1](https://doi.org/10.1007/s13187-026-02886-1)） | 5 个商用 LLM，7 名专家，**14 道 NET 的 RLT 患者问题，英法双语** | 总体不错，但**法语更差**；严重错误少见，但过度简化的回答在**依机构而定的住院要求**上误导患者；Flesch–Kincaid 年级常高于推荐水平 | 付费 | 强力支持：辖区和医院规则放进规则引擎，测试多语言输出，加可读性闸门 |
| **Latifoltojar A 等**，*npj Digit Med* 2026;9:754（[PMC13635222](https://pmc.ncbi.nlm.nih.gov/articles/PMC13635222/)） | 核医学科前瞻性**真实患者询问**（UCLH）：339 条药物相互作用、42 条医学、76 条行政；GPT-4.1 vs 员工回答；QUEST 框架 | 医学询问在 10 个维度中的 8 个上，**76–98%** 的 LLM 回答被专家评为相当或更好；非专家在行政询问上更喜欢 LLM（86%）；医学询问的人工回答更易懂（62%） | CC BY-NC-ND；询问集未声明发布 | 最好的真实核医学患者询问研究；照搬 QUEST 评价和"相对员工更好/相当/更差"设计 |
| "Optimizing patient education for radioactive iodine therapy… chain-of-thought…"（DOAJ 收录） | GPT-4 和 GPT-4o 回答常见 RAI 问题 | 回答总体恰当，但只有约 **21%** 的引用完全有效；共情约 56%（GPT-4）和 67%（GPT-4o）；评者一致性低 | 未找到 | 直接针对 131I；编造引用支持"LLM 从不自己生成引用，引用只来自知识库元数据"；**期刊、年份、题数未核实**（[DOAJ 镜像](https://library.kab.ac.ug/Record/doaj-art-56887b2fd1814a4c9f0a3783de4d4d83)） |
| GPT-3.5 vs GPT-4 核医学患者信息，*J Nucl Med Technol* 2023；Ataya 等，*Clinical Imaging*（ChatGPT-3.5 乳腺健康）；ChatGPT-4 vs Gemini PET/CT（西班牙语）；*J Pers Med* 2025 CT/MRI 患者教育；Quandt 等 JNM 2026 增刊摘要 | 遗漏关键信息；9 个妊娠场景中只有 **3 个**回答恰当，错误地称妊娠期不建议做乳腺 X 线摄影；Gemini 更短、更易读；ChatGPT-4o CT 得分最高；诊断核医学患者教育"相当准确"【abstract-only】 | — | 未发布 | "遗漏"错误证据支持必须说条目；Ataya 是**过度谨慎**的错误，IRSA-Bench 要双向测试；西语证据（[PubMed 37699647](https://pubmed.ncbi.nlm.nih.gov/37699647/)；[AuntMinnie](https://www.auntminnie.com/imaging-informatics/artificial-intelligence/article/15748593/supervision-needed-for-chatgpt-responses-to-breast-health-questions)；[PMC12194482](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12194482/)；[JNM](https://jnm.snmjournals.org/content/67/supplement_1/2643)） |
| Effraimidis G 等，*Eur Thyroid J* 2026;15(2):ETJ250385（[PMC13087872](https://pmc.ncbi.nlm.nih.gov/articles/PMC13087872/)） | 201 名内分泌科医生 vs 4 个聊天机器人，12 个病例，间隔 11 个月重复 | 与内分泌科医生一致 ≤25%；与指南一致最高 83%；两次之间答案改变（ChatGPT 4/12、Gemini 7/12、Copilot 5/12）；模型间一致 33–67% | CC BY | **随时间不稳定**的证据；支持锁定版本和回归测试 |
| Robson Chase ME 等，*Surgery* 2026:110590（[doi:10.1016/j.surg.2026.110590](https://doi.org/10.1016/j.surg.2026.110590)） | **100 例 DTC**；4 个 AI 平台 vs 医生 vs 共识估计，判断是否给 RAI | 低危：医生 24%、Gemini 19% 推荐 RAI，都高于 10% 的估计；中危：医生 92%、Doximity GPT 76% vs 50%；高危：OpenEvidence 69%，低于所有对照 | 付费 | LLM 平台间差异大；IRSA 应固定模型和版本并定期测试 |
| Xia M 等，*J Nucl Med* 2026;67(2):166–174（[PMC12866389](https://pmc.ncbi.nlm.nih.gov/articles/PMC12866389/)）；Norris SA 等，*Phys Med* 2026【title-only】 | 核医学影像 AI 生成内容幻觉的立场文件（DREAM 报告）；医学物理中 LLM 使用调查 | 定义、例子、检测指标、缓解 | CC BY | IRSA 幻觉风险部分的术语（[doi:10.1016/j.ejmp.2026.105835](https://doi.org/10.1016/j.ejmp.2026.105835)） |
| **NCT07719062**"前列腺放疗患者教育 AI 聊天机器人"（Brigham and Women's） | 单组，估计 **50** 人；主要终点：一次会话前后 PROMIS 症状自我管理效能的变化；聊天机器人"不提供诊断、治疗建议或个体化医疗建议"；临床医生审查回答的安全问题 | 状态**尚未招募**；估计 2026-12 开始，主要完成 2027-02；`hasResults: false` | 尚无结果 | 与 IRSA 受限范围最接近的临床类比；临床医生审查对话记录的做法值得 IAEA 试点照搬（[ClinicalTrials.gov](https://clinicaltrials.gov/study/NCT07719062)） |

### 卫生经济与容量：论据是床位约束下的吞吐量，不是每床日的节省

本轮**没有找到比较住院隔离与门诊 177Lu/131I 每床日成本的研究**，经济论据目前建立在容量模型上。德国核医学治疗约有 234,000 个治疗日，储备只有约 19,000 床日（约 63 张床），177Lu-PSMA 获批后会出现地区性不足（[Zippel 2021](https://doi.org/10.1055/a-1351-0030)）。日本三个县的离散事件模拟预测，引入 177Lu-PSMA-617 后 4 年内，等待 >180 天的患者显著增加，主要原因是床位不足；在一些设施，由于排水（废液）容量规定，单纯加床并不奏效（[Mizowaki 2025](https://doi.org/10.18893/kakuigaku.oa.2503)）。已发表的 177Lu-PSMA 和 177Lu-DOTATATE 成本效果分析比较的是药物与替代疗法，而不是给药地点。因此，IRSA 在德语区和日本最有力的经济论据是**床位约束下的吞吐量**；卫生经济模型应当是容量/排队模型，例如在 Mizowaki 的离散事件模拟上加入 IRSA 支持的早出院，而不是经典的成本效果分析【推断】。

| 引用 | 设计/n | 关键数字 | 数据/代码 | 对 IRSA 的用途 |
|---|---|---|---|---|
| Zippel C 等，*Nuklearmedizin* 2021;60(3):216–223（[doi:10.1055/a-1351-0030](https://doi.org/10.1055/a-1351-0030)） | 德国核医学治疗床位的基础容量计算 | 见上文 | 付费 | 德语区商业论证的床位瓶颈；部分填补"没有床位数据"的缺口 |
| Mizowaki T 等，*Kaku Igaku* 2025;62(1):59–70（[doi:10.18893/kakuigaku.oa.2503](https://doi.org/10.18893/kakuigaku.oa.2503)） | 3 个县的离散事件模拟 | 见上文 | 付费 | 日本等待时间的定量依据；缩短住院正好攻击约束瓶颈 |
| Suzuki K 等，*Int J Clin Oncol* 2025（2026 年更正）【title-only】 | 日本临床肿瘤学会工作组：2025 年推进核医学治疗的政策建议 | — | OA（更正） | 日本政策引用（[PMC12932271](https://pmc.ncbi.nlm.nih.gov/articles/PMC12932271/)） |
| Han J 等，*Nucl Med Commun* 2026【title-only】 | "结构化补液加速 I-131 清除、降低膀胱剂量、使出院更早" | — | — | IRSA 可以提示的干预（补液和排尿提醒）；引用数字前先读全文（[doi:10.1097/mnm.0000000000002088](https://doi.org/10.1097/mnm.0000000000002088)） |
| Jani S 等，*BMJ Open Qual* 2026【title-only】 | 门诊 131I 的精益六西格玛（流程时间与材料浪费） | — | OA | 流程效率框架（[PMC13182452](https://pmc.ncbi.nlm.nih.gov/articles/PMC13182452/)） |
| 药物层面的成本效果分析【title-only】：Gogebakan 等 JNM 2026（177Lu-PSMA-617 vs 卡巴他赛，美国）；Flannery 等 JNM 2026 通信；Brinkmann 等 EJNMMI 2025（德国成本效用）；Holzgreve 等 JNM 2025（Lutathera 一线） | 成本效果分析 | — | — | 查其成本输入是否列出住院天数（未核实）（[doi:10.2967/jnumed.125.271825](https://doi.org/10.2967/jnumed.125.271825)；[doi:10.1007/s00259-025-07317-9](https://doi.org/10.1007/s00259-025-07317-9)） |

## 开发计划更新：18 个月方案在物理和规则侧更稳，评测侧要多做一块真人校准

### 入门套件增删：多数新增都是宽松许可，替换掉"购买"和"等许可"两类步骤

上轮入门套件的骨架不变。本轮的变化主要有三类：用免费来源替换需要购买的项目（Γ 常数）；增加不必等待许可就能用的官方规则文本（日本、英国）；剔除或降级本轮查明为非商用的资产【推断】。

| 模块 | 新增 | 降级/移除/替换 | 理由 | 链接 |
|---|---|---|---|---|
| 剂量引擎 | **Peplow 2020 ICRP 107 有效剂量 Γ 作默认值**；Oumano 2025 的 Smith & Stabin 数值和铅 HVL/TVL 作敏感性分析和屏蔽参数；rad_point_kernel（MIT）计算墙、门等附加屏蔽；Al-Haj 0.46 作 131I 体衰减先验 | **移除"购买 Smith & Stabin"步骤**；Bellamy 理论值降为敏感性分析 | 免费、物理量与 ICRP 116 一致 | [OSTI 1607209](https://www.osti.gov/biblio/1607209)；[PMC12059296](https://pmc.ncbi.nlm.nih.gov/articles/PMC12059296/)；[rad_point_kernel](https://github.com/fusion-energy/rad_point_kernel) |
| 剂量计与读数反演 | Gamma Dose Converter（MIT）；InterSpec + SpecUtils（LGPL-2.1）；PyRIID（BSD-3）；GuideRadPROS 响应测试数据（CC BY） | G4Radiacode 保持离线（GPL-3.0，已确认） | 正向仿真链全部开源，读数可换算 H*(10) | [zenodo.21789847](https://doi.org/10.5281/zenodo.21789847)；[PyRIID](https://github.com/sandialabs/PyRIID) |
| MC 与代理模型 | MRTDosimetry 六时间点体模（CC BY）作端到端真值；SlicerSPECTRecon 患者投影（CC BY）作真实 PSMA 源；Polson 177Lu/225Ac 重复扫描（CC BY）作噪声模型；topas-rpt（MIT）用于 225Ac 衰变链；PTNO 预算规则 | radot 改为从 GitHub 安装（**不在 PyPI**）；MPPD 下载条款不明，备选 Blender ARAP 变形 + MakeHuman（CC0）包络 | 产品通道可用的 CC BY/MIT 数据增多 | [zenodo.18891044](https://doi.org/10.5281/zenodo.18891044)；[zenodo.14172228](https://doi.org/10.5281/zenodo.14172228)；[topas-rpt](https://github.com/bertoletlab/topas-rpt) |
| 动力学先验与合成轨迹 | tellurium（Apache-2.0）或 PySB（BSD-2）+ Chi（BSD-3）；Lutathera/Pluvicto 说明书；de Bakker 2023 快/慢两组；Liubchenko 2024（225Ac） | PyTheraTwin 只在研究通道使用（依赖 PyCNO 和 SIMIND）；OSP/mrgsolve/nlmixr2 只作离线工具 | 端侧可嵌入的宽松动力学栈；可引用的排泄参数 | [Chi](https://github.com/DavAug/chi)；[PMC10547615](https://pmc.ncbi.nlm.nih.gov/articles/PMC10547615/) |
| 接触先验与场景引擎 | **可商用组合**：冈比亚 2022、CoMix 全部 4 个已核记录、CODA 赞比亚/南非、荷兰 70+、EPICURUS、比利时家庭调查（均 CC BY 4.0）、SA-S-HTS 和肯尼亚（CC0）；CASAS（CC BY）如厕/卧床时刻；SPW（CC BY）+ Mesa（Apache）；TotTag ≤16% 作上界核对 | 马拉维（CC BY-NC-SA）、津巴布韦（CC BY-NC-ND）、里昂病房（NC-SA）只留在研究通道 | 不用谈判即可进入产品通道 | [zenodo.15708568](https://zenodo.org/records/15708568)；[SA-S-HTS](https://sociopatterns.org/datasets/south-african-households-contact-networks/) |
| 户型 | CASAS floorplans.zip；ProcTHOR/AI2-THOR（Apache 2.0） | HSSD、Habitat 人形、BEHAVIOR 资产不进产品通道 | 第二个宽松户型来源 | [ProcTHOR](https://github.com/allenai/procthor) |
| 规则引擎 | **日本 MHLW 通知 + PSMA 修订**；**英国 ARSAC NfG 2026-07**；加拿大 CNSC 作"调用剂量引擎"的对照辖区 | IAEA 文本仍需许可，不阻塞 v0.1 | 首批可直接编码、可再分发的官方文本 | [MHLW](https://www.mhlw.go.jp/content/10800000/001527013.pdf)；[ARSAC](https://www.gov.uk/government/publications/arsac-notes-for-guidance) |
| 知识库 | EANM 良性甲状腺 RAI 2023、177Lu 剂量学 2022、脑膜瘤 2024（CC BY）；JSNM Lu-177-DOTATATE 手册（CC BY） | 2013 PRRNT 指导（CC BY-NC）只用于非商用部署；2026 SSTR 指导待查许可 | 许可已核实 | [PMC10542302](https://pmc.ncbi.nlm.nih.gov/articles/PMC10542302/)；[PMC5852188](https://pmc.ncbi.nlm.nih.gov/articles/PMC5852188/) |
| 评测与模拟器 | Cancer-Myth（CC BY）作错误前提模板；HealthBench Professional 细则 schema（MIT）；DischargeBench 指标（只借设计）；PatientSim 人设维度（重写）；MedExpQA/GerMedIQ（CC BY）作多语言评审校准；细则验证管线 | **新增要求：真人校准子集**；PatientSim 代码不 fork | 模拟器偏乐观的证据 | [Cancer-Myth](https://huggingface.co/datasets/Cancer-Myth/Cancer-Myth)；[arXiv 2603.11245](https://arxiv.org/abs/2603.11245) |
| 验证 | Gallicchio 协议；Hartmann 的 OSL 照护者剂量计；Han 2021"先预测后核对"；Shimizu/Wakabayashi 作预注册基线 | — | 现成可照搬的验证设计 | [PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/)；[PMC8139448](https://pmc.ncbi.nlm.nih.gov/articles/PMC8139448/) |

### 阶段 1/2/3 的设计调整

| 阶段 | 调整 | 依据 | 性质 |
|---|---|---|---|
| **1 物理台架** | 在报告和代码中固定剂量量：接收者用 E（Peplow ICRP 107 Γ），巡测仪对比用 H*(10)；把其他 Γ 列为敏感性分析 | Γ 取值差约 1.8 倍（[OSTI 1607209](https://www.osti.gov/biblio/1607209)） | 【推断】 |
| 1 | ≤1 m 场景一律用线源或体源；177Lu 场景集照搬 131I 文献：面对面 0.1–3 m、并排坐、坐在后面、并排躺 0.3 m、抱婴儿、儿童坐腿上；相对体模实测的目标设为 rRMSD 约 0.2 | Sheikhzadeh 比值约 1.4–1.5；Li 2024 rRMSD 0.21–0.25；Han 2013、Dewji 2015、Taghvaee 2018 | 【推断】 |
| 1 | MC 预算按 PTNO 原则分配：大量低统计构型、对数目标、POD/PCA 和半解析作竞争基线；225Ac 单独生成 MC 数据，不外推 | [PTNO](https://arxiv.org/abs/2609.40090)；[Tan 2025](https://arxiv.org/abs/2505.07088) | 【推断】 |
| 1 | 在接触任何患者数据前，用 MRTDosimetry 六时间点体模做"活度 → 模拟读数 → 反演 → 有效半衰期"的端到端真值测试 | [zenodo.18891044](https://doi.org/10.5281/zenodo.18891044) | 【推断】 |
| 1 | Radiacode 记录完整能谱，评估能谱观测量（如 208 keV 峰/散射比）能否改进反演；Radiacode 能量与角响应仍须自测，GuideRadPROS 作偏差先验 | 能谱工具链；没有独立表征 | 【推断】 |
| 1 | 剂量引擎默认计入限制期内剂量、回家途中剂量、一年多个周期，并支持分次给药 | Banci Buonamici 2025；Chipiga 2025；Kranert 2022；Otis 2020 | 【推断】 |
| **2 智能体离线评测** | IRSA-Bench 拆成两步：**v0 合成版**（事实卡 + 人设维度 + 脚本化摩擦），**v1 真人校准版**（在标准化病人或非患者照护者的子集上校准分数，按语言和方言分别报告） | Sim2Real；Lost in Simulation；The Average Patient | 【推断】 |
| 2 | 细则管线：原子事实起草 → 布尔条目 → 区分度验证 → ≥3 名专家 3 阶段裁定 → 报告评审-专家一致性 | arXiv 2601.15161、2503.23339、2604.24710、2610.13303；HealthBench Professional | 【推断】 |
| 2 | 双向测试：虚假安慰/错误前提题（Cancer-Myth 式、按与引擎输出的距离分级），以及过度限制对照集（NFP 式）；加入妊娠场景的过度谨慎题 | Cancer-Myth；The Patient Is Always Right?；Ataya | 【推断】 |
| 2 | 辖区切换用美/日/英/加四个规则包，其中加拿大必须调用剂量引擎；版本漂移用 MHLW PSMA 修订前后、NRC Rev. 1 对 DG-8061 | MHLW；CNSC | 【推断】 |
| 2 | 可读性闸门；LLM 不得自行生成引用；锁定模型版本并做定期回归；以 Speicher 2025 的 7–8% 医学错误率作为必须优于的上限 | Jankowski、Yalamanchili、Habouzit；RAI CoT 研究；Effraimidis、Robson Chase；Speicher | 【推断】 |
| **3 临床影子模式** | 院内配对测量照搬 Gallicchio 协议（佩戴 vs 固定参考，2 h 分箱，Bland–Altman），并采集治疗前显像协变量（洗脱率、SPECT 总摄取），以便与 Shimizu/Wakabayashi 基线做预注册比较；采用 Han 2021"先预测、后核对"的设计 | [PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/)；[Shimizu](https://doi.org/10.1097/mnm.0000000000002110)；[PMC8139448](https://pmc.ncbi.nlm.nih.gov/articles/PMC8139448/) | 【推断】 |
| 3 | 照护者剂量计优先用 OSL（含睡眠期）；照护者与患者距离用 Berkeley 式 BLE 方案，条件允许时加 UWB | Hartmann 2023；Berkeley；TotTag/SociTrack | 【推断】 |
| 3 | 终点增加心理社会指标（GAD-7/PSQI 类，按治疗前/住院期/治疗后测量；"辐射暴露担忧"；知信行）、接受度问卷（Kaufmann 条目）和可用性（SUS） | Huang、Zeng、Yang、Kaufmann、Carr 2026 | 【推断】 |
| 3 | 225Ac 仍不进 v1 的剂量功能，但规则引擎加入失禁筛查问题和以马桶为中心的卫生规则；废物建议前询问产品是否含 177mLu | Dahlke 2025；Wanke 2021；Tsuchiya 2026 | 【推断】 |
| 3（可选） | 医护模式或钱包卡；卫生经济交付物改为容量/排队模型 | Mench 2025；Mizowaki 2025 | 【推断】 |

### 可借鉴的数据采集方案

| 方案 | 来源 | 设计要点 | IRSA 借用方式【推断】 | 可得性 |
|---|---|---|---|---|
| **BLE 手表 + 插电接收器** | UC Berkeley 痴呆照护研究（[PMC9771049](https://pmc.ncbi.nlm.nih.gov/articles/PMC9771049/)） | 27 对；双方戴 BLE 手表（电池最长 4 个月）；每户 3 个插电式蓝牙接收器；逐秒测距 6 个月；可映射房间间移动 | 患者和同住照护者各戴 BLE 标签，家中放 2–3 个接收器，得到房间级共处和粗略距离；负担低，可持续数周 | 数据未公开；只有摘要 |
| **UWB 测距可穿戴** | TotTag（[PMC8907335](https://pmc.ncbi.nlm.nih.gov/articles/PMC8907335)）；SociTrack（[Biri 等](https://lab11.eecs.berkeley.edu/content/pubs/biri20socitrack.pdf)） | TotTag 1 Hz 飞行时间测距，1/3/6 ft 处标准差约 0.3–1.6 ft，可穿透墙和人体；SociTrack 最高 16 Hz、99 百分位 14.8 cm；固件 Apache-2.0 | 在 BLE 方案上加 UWB，得到亚米级距离，直接对应 <0.3 m、0.3–1 m 等暴露档；注意洗澡时会摘下 | 固件 Apache-2.0；TotTag 数据无许可证 |
| **佩戴 vs 固定参考配对** | Gallicchio 2021（[PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/)） | 每小时 53 次读数，2 h 平均，相关 + Bland–Altman | 德国 48 h 住院期 Radiacode 与固定参考/2 m 测量配对 | 协议公开（CC BY） |
| **照护者剂量计** | Hartmann 2023（[doi:10.1055/a-2015-0475](https://doi.org/10.1055/a-2015-0475)） | 指环剂量计 + 睡眠期 OSL/TLD 对比，OSL 重复性更好 | 照护者参考剂量用 OSL，包含睡眠期 | 付费文章 |
| **先预测后核对** | Han 2021（[PMC8139448](https://pmc.ncbi.nlm.nih.gov/articles/PMC8139448/)） | 放行时预测第 6–8 天剂量率，再实测核对 | 用出院读数 + 可穿戴更新预测复诊时的剂量率，再实测 | CC BY |
| **临床医生审查对话记录** | NCT07719062（[ClinicalTrials.gov](https://clinicaltrials.gov/study/NCT07719062)） | 临床医生审查聊天机器人回答的安全问题 | IAEA 试点中由医学物理师审查影子模式对话 | 尚无结果 |
| **3 阶段专家裁定** | HealthBench Professional（[HF](https://huggingface.co/datasets/openai/healthbench-professional)） | ≥3 名医生分 3 阶段撰写和裁定细则，每题收集医生回答 | 医学物理师/RPO 裁定 IRSA-Bench 细则，并为每题写参考回答 | MIT |
| **心理与接受度测量** | Huang 2026；Kaufmann 2026；Carr 2026 | 治疗前/住院期/治疗后三次测量；接受度问卷；4 周居家 SUS | 阶段 3 次要终点和可用性研究 | CC BY |

### 规则引擎的首批可测试官方文本

| 规则包 | 可编码的条目 | 许可 | 现成测试用例 | 注意 |
|---|---|---|---|---|
| **日本 MHLW** | 公众 1 mSv/年、照护者每次 5 mSv；I-131 500 MBq 或 1 m 处 30 µSv/h；Sr-89 200 MBq；Y-90 1,184 MBq；Ra-223 每次 12.1 MBq；Lu-177 SSTR 1 m 处 18 µSv/h；Lu-177 PSMA 1 m 处 16 µSv/h；记录保存 2 年（[MHLW](https://www.mhlw.go.jp/content/10800000/001527013.pdf)） | 著作权法第 13 条 + PDL1.0 | **版本漂移**：PSMA 修订前，177Lu-PSMA 患者无对应条款；修订后为 16 µSv/h。边界值测试：1 m 处 17.9 / 18.1 µSv/h 的 SSTR 患者【推断】 | PSMA 修订正式编号待核；JSNM 手册中的社会条件（家有 15 岁以下儿童或孕妇等，上轮已收录）需同时编码 |
| **英国 ARSAC** | 表 7.1 治疗后避孕期（131I 4 个月、131I-mIBG 4 个月、89Sr 24 个月、153Sm-EDTMP 1 个月）；表 7.2 哺乳中断；表 7.3–7.4（[NfG PDF](https://assets.publishing.service.gov.uk/media/6a510b75c3d64d94cacab7aa/ARSAC_NfG_July_2026.pdf)） | OGL v3.0 | 妊娠/哺乳询问的必须说条目 | 建议性文本；英国放行数值不在其中 |
| 美国 NRC（上轮） | §35.75：5 mSv 放行、>1 mSv 需书面指导；RG 8.39 Rev. 1 | 公共领域 | 与 DG-8061 的版本漂移 | DG-8070 全文仍缺 |
| **加拿大 CNSC** | 照护者约束通常 5 mSv；他人剂量可能 >1 mSv 时需剂量评估；没有数值阈值（[CNSC](https://www.cnsc-ccsn.gc.ca/eng/nuclear-substances/therapeuticuseofnuclearsubstances/)） | 仅限非商用复制 | **辖区切换**：同一患者在加拿大必须调用剂量引擎，在日本查表 | 商用再分发需书面许可 |
| 德国（上轮数值） | I-131 出院 ≤250 MBq / 2 m 处 3.5 µSv/h | §5 UrhG（按第 2 款：注明出处、不改原文） | 与日本 30 µSv/h（1 m）的单位与距离换算测试 | 没有 BMUV 合订本 |

### 对约 18 个月压缩方案的影响

此前提出的压缩方案是：约第 3 个月出演示；约第 6–9 个月拿出物理证据和 IRSA-Bench；临床数据采集与 LLM 评测脱钩，院内配对测量约第 5–6 个月开始；影子模式约第 18 个月结束；总投入约 5–7 人年（均为估算）。本轮发现**不足以支持进一步压缩**，因为最核心的三类数据仍然只能自己采集，临床数据的时间线没有变；但它们降低了物理侧和规则侧的风险，同时给评测侧加了一项必要工作【推断】。

| 节点 | 原方案 | 本轮发现的影响 | 调整建议【估算/推断】 |
|---|---|---|---|
| 约第 3 个月演示 | 确定性链路"读数 → 半衰期 → 半解析剂量 → 规则 → 模板" | Γ 免费可得，不必等采购；日本、英国、美国三个规则包都不用等许可；MRTDosimetry 体模给演示提供已知真值；Lutathera/de Bakker 参数让合成轨迹可引用 | 演示可以多展示"辖区切换 + 版本漂移"（MHLW PSMA 修订前后），可行性上升 |
| 约第 5–6 个月院内配对测量 | 德国 48 h 住院期 Radiacode 与参考测量配对 | Gallicchio 协议可直接照搬；需加采治疗前显像协变量；照护者剂量计改用 OSL | 伦理申请在第 1–4 个月提交时就写入这些变量；时间不变 |
| 约第 6–9 个月物理证据 | 体模台架、三模型对比 | 开源正向链和 PTNO 预算规则减少工程量；但 Radiacode 表征和 177Lu 近场体模实测仍须自做；MPPD 许可不明 | 第 1–6 个月优先完成探测器表征和近场体模测量；姿态体模准备 Blender/MakeHuman 备选；225Ac 不纳入 |
| 约第 6–9 个月 IRSA-Bench | 一次性交付 | 模拟器偏乐观，必须有真人校准子集，并按语言报告 | **拆成 v0 合成版（约第 6 个月）和 v1 真人校准版（约第 9–12 个月）**；校准用标准化病人或非患者照护者，不需要患者数据，所以与临床数据仍基本脱钩，但要一份小型伦理/招募 |
| 临床与 LLM 脱钩 | 完全脱钩 | 真人校准子集是新的人类数据需求 | 改为"与患者数据脱钩、与人类受试者部分耦合" |
| 约第 18 个月影子模式结束 | 预测 vs 照护者剂量计一致性 | 新增可预注册的基线比较（Shimizu/Wakabayashi/Han）和心理社会终点 | 终点清单扩展；结束时间不变 |
| 总投入约 5–7 人年 | — | 节省：Γ 采购、动力学引擎、读数仿真工具（约 1–2 人月）；新增：真人校准子集（约 2–4 人月）、基线协变量采集与分析（约 1–2 人月） | 净效应大致持平到略增，仍在 5–7 人年区间内 |

## 许可证更新：十余项已厘清，新增一批非商用和无许可证资产

### 已解决或已更正的项目

| 项目 | 上轮状态 | 本轮结论 | 来源 |
|---|---|---|---|
| CoMix（英国、比利时、2.0、最后一轮 BE/CH/NL/UK） | 未逐项核对 | **CC BY 4.0**（4 个记录均核实；其余国家记录未逐一核对） | [13684044](https://zenodo.org/records/13684044)；[10549953](https://zenodo.org/records/10549953)；[7331926](https://zenodo.org/records/7331926)；[11154066](https://zenodo.org/records/11154066) |
| 津巴布韦（Manicaland）接触调查 | 未核实 | **CC BY-NC-ND 4.0**：非商用且**禁止演绎**，连衍生矩阵都不能共享 | [3886638](https://zenodo.org/records/3886638) |
| 荷兰 70+、EPICURUS、比利时家庭调查、赞比亚/南非（CODA） | 多数未核实 | **CC BY 4.0** | [7751724](https://zenodo.org/records/7751724)；[20271335](https://zenodo.org/records/20271335)；[7965594](https://zenodo.org/records/7965594)；[3874675](https://zenodo.org/records/3874675) |
| SocioPatterns Zenodo 数据包 | CC BY-NC 4.0 | 再次确认 | [1117884](https://zenodo.org/records/1117884) |
| Prem 2021 合成接触矩阵 | 未确认 | Zenodo v2.0 标"other-open"；GitHub 仓库 **GPL-3.0**；修改后的矩阵再分发需谨慎；Prem 2017 仍未核实 | [4889499](https://zenodo.org/records/4889499)；[GitHub](https://github.com/kieshaprem/synthetic-contact-matrices) |
| MIT PACT 数据集（BLE-RSSI 静态配置、H0H1、KRI 暗室、MIT Matrix、Range_Angle） | 未核实 | **CC BY 4.0** + Distribution Statement A；Exposure-Notification Beacons 只有 Distribution Statement A | [MIT-Matrix-Data](https://github.com/mitll/MIT-Matrix-Data)；[PACT](https://mitll.github.io/PACT/datasets.html) |
| MITRE Range-Angle Structured/Unstructured（NIST TC4TL 训练数据） | 未核实 | **CC BY-NC 4.0**，不能商用 | [Structured](https://github.com/mkrangle/MITRE-Range-Angle-Structured) |
| Copenhagen Networks Study | 数据文件许可未确认 | **MIT**（Figshare） | [Figshare 7267433](https://figshare.com/articles/dataset/The_Copenhagen_Networks_Study_interaction_data/7267433) |
| 马拉维农村村庄 | 托管和许可未确认 | **CC BY-NC-SA** | [SocioPatterns](https://sociopatterns.org/datasets/contact-patterns-in-a-village-in-rural-malawi/) |
| Meta HRSL | 【未核实】（HDX 403） | **CC BY 4.0**，每季度更新，"部分国家" | [AWS 注册表](https://registry.opendata.aws/dataforgood-fb-hrsl/) |
| OpenStreetMap | 本轮前未重新核实 | ODbL 确认；文档为 CC BY-SA 2.0；衍生数据库须同许可发布 | [OSM 版权](https://www.openstreetmap.org/copyright) |
| Google Maps Platform | 现行 Places 存储条款未确认 | Places 经纬度最多连续缓存 30 天；不得与非 Google 地图合用；**不得用于"高风险活动"，不得传输、存储或处理受美国 HIPAA 管辖的健康信息** | [服务条款](https://cloud.google.com/maps-platform/terms/maps-service-terms)；[平台条款](https://cloud.google.com/maps-platform/terms) |
| JSNM Lu-177-DOTATATE 手册 | 未查 | **CC BY** | [PMC5852188](https://pmc.ncbi.nlm.nih.gov/articles/PMC5852188/) |
| MRTDosimetry | 两份笔记冲突 | Zenodo 镜像 **CC BY 4.0** | [zenodo.18891044](https://doi.org/10.5281/zenodo.18891044) |
| radioactivedecay | PyPI 带"Other/Proprietary" | LICENSE 为 **MIT**（© JAEA 与贡献者）；内置数据集未核实 | [GitHub](https://github.com/radioactivedecay/radioactivedecay) |
| radot | LGPL-3.0；PyPI 包未确认 | **不在 PyPI**（404） | [radot](https://github.com/dsarrut/radot) |
| G4Radiacode | GPL-3.0 | 本轮读 LICENSE 确认 | [G4Radiacode](https://github.com/lucamors/G4Radiacode) |
| MCNP6.3、FLUKA、SIMIND、PENELOPE/penEasy | 【未核实】 | 见物理层一节；penEasy 仍未读到许可文本 | 见上 |
| HPS ATE | 保留所有权利 | 确认没有文本挖掘条款；抓取许可 ≠ 再利用许可 | [HPS 免责声明](https://hps.org/disclaimer/) |
| IAEA 出版物 | 需许可 | 版权页核实：通常收版税；非商用复制和翻译欢迎、逐案审议；网站条款页仍未读到 | [SSG-46 PDF](https://www-pub.iaea.org/MTCD/Publications/PDF/PUB1775_web.pdf) |

上述结果修订了可商用的接触先验组合：冈比亚 2022、CODA 赞比亚/南非、荷兰 70+、EPICURUS、比利时家庭调查、已核实的全部 CoMix 记录（CC BY 4.0），以及 SA-S-HTS 和肯尼亚 SocioPatterns（CC0）。只限非商用的是 POLYMOD、SocioPatterns 数据包、马拉维、里昂病房和津巴布韦（后者还禁止演绎）。BLE 距离研究可以商用 MIT LL 数据集，但训练 TC4TL 挑战赛的 MITRE Range-Angle 数据集是非商用。Google Maps 的 HIPAA 和高风险条款进一步支持上轮的决定：IRSA 处理患者健康信息，位置情境应只用开放数据（Overture/OSM/Foursquare）【推断】。Google 条款以美国 HIPAA 为中心，对非美国健康数据是否有同等条款，本轮未评估。

### 新增红线：哪些资产不能进入产品通道

| 类别 | 本轮新增的资产 | 研究阶段 | 商业化时 |
|---|---|---|---|
| **非商用/仅限研究/学术** | 马拉维村庄（CC BY-NC-SA）；津巴布韦（CC BY-NC-ND）；里昂病房（CC BY-NC-SA 3.0）；MITRE Range-Angle（CC BY-NC）；RTI 合成人群；FRED v5；AMASS；BABEL；SMPL-X；完整 SMPL（商用需付费）；Habitat 人形化身和 HSSD 场景；BEHAVIOR 数据包；samkiwi18 权重（CC BY-NC）；MMedBench 原版；KorMedMCQA；2013 PRRNT 指导（CC BY-NC）；加拿大政府文本（无许可时仅非商用）；SIMIND（研究、教育、培训）；FLUKA（非商用、非军事）；PatientSim 与 DischargeBench 数据（PhysioNet 资质许可，MIMIC 来源）；DREAMT；LaymanRRG/BioLaySumm（大概率适用 PhysioNet 条款）；**TIHM（许可冲突，解决前按非商用处理）** | 可用于内部研究与验证 | 不得用来训练或随产品打包 |
| **禁止演绎** | 津巴布韦接触调查 | 可引用、可复算 | 不得分发衍生矩阵 |
| **共享相同方式** | Z-Anatomy（CC BY-SA 4.0）；AfriMed-QA 镜像（CC BY-SA 4.0）；马拉维、里昂（NC-SA）；BodyParts3D 若按旧许可（CC BY-SA 2.1 JP）理解 | 可用 | 衍生物须同许可；含 NC 的不能商用 |
| **强 copyleft** | ZapMeNot（GPL-3.0）；OSP PK-Sim/MoBi/OSPSuite-R（GPL-2.0 + 附录）；mrgsolve（GPL ≥2）；nlmixr2/rxode2（GPL ≥3）；OpenABM-Covid19、JUNE、CovidSim（GPL-3）；Prem 矩阵代码（GPL-3.0）；MakeHuman 代码（AGPL，导出模型为 CC0）；MPFB2（GPLv3，资产 CC0）；synthpop（GPL）；AlphaRadPharmacokinetics（GPL-3.0） | 只作离线工具或测试工具 | 不得链接进分发的 App 或服务端 |
| **弱 copyleft** | InterSpec、SpecUtils（LGPL-2.1） | 可用 | 作可替换的动态库使用 |
| **出口管制或机构许可** | MCNP6.3（源码出口管制，分发对象限制【search-summary, verify】）；PENELOPE（NEA 规则）；GADRAS（不开放）；PiMAL（依赖 MCNP，许可未读） | 视机构资格 | 不进入 TC 多国可移植栈 |
| **没有许可证（默认保留所有权利）** | PyCNO；PHIRST；TotTag 数据；PatientSim 代码；Cancer-Myth 代码仓库；VP-NLME-STP-Dosimetry；qurit/xcat-sims；lucasfffaria（空）；SPEW；Orange4Home、ARAS、MARBLE、MuRAL、UCAmI；Kasteren 原始数据；Lund XCAT 影像（"按原样"）；DPM-RPT Deep Blue（元数据无许可）；`elcooooo/patient-simulator-german`；`geodesic-puria/*`；`mli5/medquad-sycophancy`；MedArabiQ；JMedBench；penEasy（下载页未写）；OpenDose 数据库；MPPD（条款未知） | 只能阅读和借鉴思路 | 先取得授权 |
| **受限访问** | Ennassiri 221Fr 能窗数据；Zenodo 24126454 和 23580810；MSKCC DOTATATE 影像（不公开）；Orange4Home（邮件申请） | 需申请 | 按协议 |
| **自定义或附条件** | Google Maps（HIPAA 与高风险条款）；HPS；IAEA（版税）；IPEM MDGN（商业书）；SNMMI；ARPANSA（按文件而异）；德国 §5(2)（不得修改、须注明出处）；MHLW PDL1.0（修改须标明，不得冒充政府作品） | 按条款 | 逐项法务审查 |

**可以直接进入产品通道的新资产**（宽松许可，带署名即可商用）：Peplow 2020（DOE 接受稿）和 Oumano 2025（CC BY）的常数；rad_point_kernel、Gamma Dose Converter、pyshield-smr（MIT）；PyRIID（BSD-3）；MC-GPU（公共领域）；radioactivedecay（MIT）；SMPL-Body 子集（CC BY 4.0）；MakeHuman 导出模型（CC0）；GuideRadPROS（CC BY 4.0）；RADAI 合成城市能谱（CC0）；Polson 的 177Lu/225Ac 数据和 SlicerSPECTRecon、PyTomography 教程数据、MRTDosimetry 镜像（CC BY 4.0）；PyTheraTwin、topas-rpt、LesionStudio（MIT，但要注意 PyTheraTwin 的依赖）；simind-python-connector（Apache-2.0，SIMIND 本身除外）；tellurium（Apache-2.0）、PySB（BSD-2）、Chi（BSD-3）、PKPDsim（MIT）；CASAS 全部记录、SPHERE（落地页本轮 503，许可经研究门户确认）、欧洲 62 户、Kasteren 合成版（CC BY 4.0）；SA-S-HTS、肯尼亚（CC0）；婚礼 UWB（CC BY 4.0）；SPW（CC BY 4.0）；SynthPops、Covasim、Starsim（MIT）；PopulationSim、pyEpiabm（BSD-3）；Mesa（Apache 2.0）；VirtualHome、LocoVR（MIT）；ProcTHOR、AI2-THOR（Apache 2.0）；Cancer-Myth 数据（CC BY 4.0）；HealthBench Professional（MIT）；ROND（Apache-2.0）；MedExpQA、GerMedIQ、MediQAl、AfriMed-QA（CC BY 4.0）；CMB、Huatuo26M-Lite、ApolloCorpus、Global-MMLU（Apache-2.0）；ARSAC NfG（OGL v3.0）；MHLW 通知（第 13 条 + PDL1.0）；EANM 2022/2023/2024 三份指南和 JSNM 手册（CC BY）。Synthea（Apache-2.0）上轮已列入。

## 矛盾与未核实清单：CLEAN-225Ac 的数字对不上，须查原文

**第一项必须单独说明：CLEAN-225Ac 的数字在不同轮次之间对不上，本报告不作取舍。** 背景评估及其产业笔记引用了一项 2026 年 9 月发表的"CLEAN-225Ac"研究：**6.5% 的擦拭样本、62.5% 的治疗周期**存在污染，链接为 ScienceDirect（EANM Innovation）（[ScienceDirect S3051291326002600](https://www.sciencedirect.com/science/article/pii/S3051291326002600)）。本轮的核医学数据笔记**没有找到名为 CLEAN-225Ac 的研究**，只在检索层面找到一项 2026 年住院污染研究：225Ac/177Lu-PSMA-I&T 串联和 225Ac-DOTATOC，6 次治疗中 4 次有污染，治疗后 128 份样本中 12 份阳性（**9.4%**），检索结果指向 JNM node 128470【search-level, verify】。本轮的相关论文笔记读到了同一 JNM 节点的 Dahlke 等 2025 年 JNM 增刊摘要：6 名男性患者、176 份擦拭、治疗后 9.4% 阳性、与尿失禁相关【abstract-only】，并推测它是 CLEAN-225Ac 的试点研究（同一课题组）（[JNM node 128470](https://jnm.snmjournals.org/node/128470)）。两份本轮笔记描述的大概是同一份摘要（数字和节点都相同），但一份标为 2026 年住院研究，一份标为 2025 年增刊摘要；它与 6.5%/62.5% 的 CLEAN-225Ac 是否为同一研究的不同阶段、样本是否重叠，都没有核实。**建议：直接打开 ScienceDirect S3051291326002600 和 JNM node 128470 两个原文，核对标题、作者、样本数和分母（擦拭样本 vs 治疗周期 vs 治疗次数），再决定引用哪一个；在此之前，IRSA 文档中两组数字都只能带出处并列引用。**

**第二项已解决：上轮"意大利 CNR 甲状腺患者无线设备"【search-summary, verify】就是 Gallicchio 等 2021（IEEE JTEHM，PMC7732145）**，n = 20，r = 0.96（[PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/)）。

| # | 项目 | 说法 A | 说法 B | 处理建议 |
|---|---|---|---|---|
| 3 | **Dewji 点源评估手稿（OSTI 1213312）内部矛盾** | 结果部分一句称，对甲亢患者，点源在 30 cm 以外"低估" | 结论称点源从 10 到 300 cm"一贯高估" | 核对 *Health Physics* 正式发表版本 |
| 4 | **可能是两篇不同的 Han 2013** | Med Phys 摘要（PMID 23927361）：UF **混合**体模；面对面、抱新生儿、儿童坐腿上 | Dewji 转述：ORNL 先前开发的**程式化**体模，另有并排躺 0.3 m 的场景 | 可能是另一篇"A practical guideline for the release of patients treated by I-131…"（[ResearchGate](https://www.researchgate.net/publication/261375415_A_practical_guideline_for_the_release_of_patients_treated_by_I-131_based_on_Monte_Carlo_dose_calculations_for_family_members)），Crossref 本轮未解析；两篇都找出来再引用 |
| 5 | **TIHM 许可证** | Zenodo API：CC BY 4.0 | 同一记录的检索摘要：限研究和患者获益，禁止商用，须致谢 Surrey and Borders NHS Trust | 通读 Zenodo 描述；解决前按非商用处理 |
| 6 | **Habitat humanoids 许可证** | HF 数据卡 YAML：CC BY-NC-SA 4.0 | 正文：化身 CC BY-NC 4.0；动作文件按 SMPL Body Motion File 许可 | 两种都是非商用，产品通道不用 |
| 7 | **BodyParts3D 许可证** | 现行 README 署名文字：CC BY 4.0 | 较早来源（维基百科）：CC BY-SA 2.1 JP | 以官方许可页为准 |
| 8 | **becquerel 许可证元数据** | LICENSE：BSD-3 风格的 LBNL 许可 | PyPI 分类："Other/Proprietary" | 再分发前通读 LICENSE.txt |
| 9 | **肯尼亚家庭数据人数** | Figshare 描述：75 人 | Netzschleuder 网络文件：47 个节点 | 下载原始文件核对；引用时注明来源 |
| 10 | simind-python-connector | LICENSE 文件：Apache-2.0 | GitHub API：NOASSERTION | 以 LICENSE 文件为准，保留 NOTICE |
| 11 | Open Systems Pharmacology | LICENSE：GPL-2.0 加澄清附录 | GitHub API：NOASSERTION | 按 GPL-2.0 + 附录处理 |
| 12 | Prem 2021 | Zenodo："other-open" | GitHub：GPL-3.0 | 修改后矩阵的再分发须向作者澄清 |
| 13 | ROND | LICENSE 文件：Apache-2.0 | 检索摘要：许可问题联系作者 | 按 Apache-2.0 使用，必要时确认 |
| 14 | MMedBench；FrenchMedMCQA；AfriMed-QA | 原版 CC BY-NC；qanastek 副本 Apache-2.0；原版 CC BY 4.0 | 重传版 CC BY；DrBenchmark 副本无标签；镜像 CC BY-SA 4.0 | 一律以原始发布者为准 |
| 15 | Cancer-Myth | 数据：CC BY 4.0 | 代码仓库无 LICENSE；衍生集标 Apache-2.0 | 只用数据，代码重写 |
| 16 | MedPRESS | 原仓库：受限访问、仅限研究 | 新仓库：无许可证标签 | 两个都按非商用处理 |
| 17 | NCT07719062 开始日期 | 一条检索摘要：2026 年 8 月 | API 记录（2026-09 更新）：2026 年 12 月（估计） | 以登记 API 为准 |
| 18 | Huang 2026 心理轨迹 | 报告的轨迹：焦虑和抑郁在住院期达峰 | 摘要另一句：焦虑和抑郁"在治疗后一个月达峰" | 核对全文 |
| 19 | XGBoost 代理（R² = 0.995）归属 | 检索片段指向 NIMB 摘要页 | Pal Chowdhury 的 OSTI 记录只提到 CNN | 归属未核实，不要引用到 Pal Chowdhury 名下 |
| 20 | 225Ac 子体 γ 发射概率 | 218 keV 11.6%、440 keV 26.1%（[arXiv 2406.05220](https://arxiv.org/pdf/2406.05220)） | 11.4%、25.9%（[PMC8041692](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8041692/)） | 采用同一评价数据集（如 ICRP 107）并注明 |
| 21 | 177Lu Γ 的物理量 | Peplow：1 m 有效剂量率（AP） | Smith & Stabin：照射率；Bellamy：理论裸源；NRC 起草稿 0.00527【search-summary, verify】 | 不是矛盾而是物理量不同；固定一个量并注明 |
| 22 | Radiacode 分辨率 | 上轮 Radiacode 103 规格：662 keV 处 8.4% | 本轮 100 系列页面：9.5%；博客估计 7–10% | 自测 |
| 23 | Bertschmann 2026 与 PubMed 42239883 | Front Oncol 2026（PMC13225992），335 题 | 另一条"radiation and cancer biology practice exam"记录，数字相同 | 很可能是同一研究，引用时只计一次 |
| 24 | PyTomography 教程患者与 SlicerSPECTRecon 患者 | 两个数据集各含一例 177Lu-PSMA 患者 | 两者都是 304 个 CT 文件、两个床位 | 按同一患者处理，不要重复计数（笔记推断） |
| 25 | SIMIND v9.0 日期 | "08/10/2026" 可读作 10 月 8 日 | 也可读作 8 月 10 日 | 不影响使用 |
| 26 | DischargeBench 评审验证 | 论文称评审与医生一致性"相当于"医生之间 | 只基于 20 个共同病例，医生间 95% CI 含 0【检索摘要】 | 不要把它当作评审有效性的强证据 |
| 27 | HealthBench Professional 题数 | 525 题 | 来自 OpenAI PDF 的检索摘要，未直接读到 | 引用时注明 |

**其他未核实项（只见检索摘要或未读原文）**：Dewji 2015 "患者坐在后面剂量最高"；Taghvaee 2018 的 0.2 mSv；JNM node 34438 同床研究的作者和年份；密歇根 *Thyroid* 来信；阿曼 SQUMJ 研究；Choi 2026 的 54,000 例和 0.76 s；NRC 起草稿 0.00527；MCNP 6.3.1 分发限制；MHLW 2024 研究报告的 225Ac Γ 和模型输出；[225Ac]Ac-SibuDAB 的 281% 过量；JNM 2026 225Ac I 期；Stenvall 2024 和 Jiménez-Franco 2021；MSKCC DOTATATE 模型；IAEA CRP E23005/E24027；EURADOS WG13 幻灯片；Jankowski 2024 的数字（来自新闻和机构报道）；RAI 链式思维研究的期刊、年份、题数；ARPANSA 各文件许可；2026 EANM/IAEA/SNMMI SSTR 指导的许可；MHLW PSMA 修订的正式编号和日期（只读到广岛市转发件）；德国 Richtlinie 适用 §5 第 1 款还是第 2 款；opendose.polsl.pl 与 opendose.org 的关系；Lund XCAT 影像和 DPM-RPT 数据的大小与内容。

## 下一步行动：先核对两份原文，再按优先级发索取函

### 索取清单

| 对象 | 要什么 | 为什么 | 链接 | 优先级【推断】 |
|---|---|---|---|---|
| ScienceDirect / JNM 原文（Dahlke 等，Charité） | 打开 CLEAN-225Ac 和 JNM node 128470 原文，核对样本数、分母、是否重叠 | 两组污染数字（6.5%/62.5% vs 9.4%）无法并存 | [S3051291326002600](https://www.sciencedirect.com/science/article/pii/S3051291326002600)；[node 128470](https://jnm.snmjournals.org/node/128470) | **高**，成本低 |
| MSKCC（Roemer、Bellamy） | 1,064 道保健物理模拟题能否用于 IRSA 评测 | 与放射防护最直接相关的 LLM 基准 | [doi:10.1088/1361-6498/ad1fdf](https://doi.org/10.1088/1361-6498/ad1fdf) | 高 |
| Zahner/Benz 团队 | 患者信息模板的许可与补充材料 | 规则引擎与语料种子；"告诉了患者什么"检查表 | [doi:10.2967/jnumed.126.273336](https://doi.org/10.2967/jnumed.126.273336) | 高 |
| King's College London（SELFIE PI） | 结果与数据 | 患者自测剂量率 TAC vs 显像 TAC，与 IRSA 最接近 | [NCT04922801](https://clinicaltrials.gov/study/NCT04922801) | 高 |
| de Bakker / Radboud | 逐患者尿排曲线 | 排尿阶跃生成器的个体级验证 | [PMC10547615](https://pmc.ncbi.nlm.nih.gov/articles/PMC10547615/) | 高 |
| QURIT（Rahmim、Uribe） | PyCNO 许可；PyTheraTwin 商用链条的书面许可；arXiv 2510.21054 的 640 例队列是否发布 | 合成患者的最短路径 | [PyCNO](https://github.com/qurit/PyCNO)；[arXiv 2510.21054](https://arxiv.org/abs/2510.21054) | 高 |
| MHLW / JSNM | PSMA 修订的正式通知编号和日期；JSNM 2025-07-17 PSMA 手册的许可 | 日本规则包的出处链 | [JSNM PDF](https://jsnm.org/wp_jsnm/wp-content/uploads/2025/07/Lu-177-PSMA-IT_manual20250717.pdf) | 高 |
| IAEA 出版处（经 TC 渠道） | 按版权页提出非商用复制**和翻译**许可 | 版权页明确"欢迎" | [SSG-46 PDF](https://www-pub.iaea.org/MTCD/Publications/PDF/PUB1775_web.pdf) | 高 |
| Gallicchio 团队 | 佩戴设备原始读数 | 唯一的佩戴 vs 固定参考配对数据 | [PMC7732145](https://pmc.ncbi.nlm.nih.gov/articles/PMC7732145/) | 中 |
| UC Berkeley（Levenson 课题组） | 绝对距离和近距离时长；完整论文 | 唯一的家中病人-照护者连续测距 | [PMC9771049](https://pmc.ncbi.nlm.nih.gov/articles/PMC9771049/) | 中 |
| Vanderbilt（TotTag） | 数据许可；125 个家庭可行性数据 | 唯一的家庭连续实测距离 | [PMC8907335](https://pmc.ncbi.nlm.nih.gov/articles/PMC8907335) | 中 |
| 汉阳大学（C.H. Kim 课题组） | MPPD 下载与许可 | 姿态变形 MRCP 的最便捷路径 | [PMID 39230434](https://pubmed.ncbi.nlm.nih.gov/39230434/) | 中 |
| Lund（Ljungberg、Gustafsson） | SIMIND 商用条款；XCAT 模拟影像的再分发许可 | PyTheraTwin 依赖 SIMIND；唯一的模拟 SSTR 队列 | [SIMIND](https://www.msf.lu.se/en/research/simind-monte-carlo-program/downloads)；[Lund 资源](https://www.msf.lu.se/en/research/resources) | 中 |
| EANM / Elsevier | 2026 SSTR 联合指导的许可 | 取代 2013 版 177Lu-DOTATATE 内容 | [检索摘要](https://iro.uiowa.edu/esploro/outputs/journalArticle/Joint-EANM-IAEA-and-SNMMI-practical/9985147077202771) | 中 |
| IPEM | MDGN 第 2 版放行章节的许可或购买 | 英国放行数值所在 | [检索摘要](https://indiepubs.co.uk/products/medical-and-dental-guidance-notes-second-edition-1755102643066) | 中（英国试点时升高） |
| Liubchenko / LMU；JNM 2026 225Ac I 期作者 | 225Ac 尿液与子体数据；补充材料中的尿表 | 225Ac 附加模块的动力学先验 | [PMC11178588](https://pmc.ncbi.nlm.nih.gov/articles/PMC11178588/) | 中（225Ac 优先级低于 177Lu） |
| PHIRST 作者；Ozella（意大利婴儿家庭）；i-Bird | 许可或数据 | 家庭类型和婴儿接触先验 | [PHIRST](https://github.com/lorenzodallamico/PHIRST_SocioPatterns) | 低/中 |
| Ennassiri & Touti；Zenodo 24126454、23580810 作者 | 受限数据访问 | 225Ac 能窗修正；紧凑 CsI/NaI 响应 | [zenodo.22055021](https://doi.org/10.5281/zenodo.22055021) | 低/中 |
| ORNL/RCD（PiMAL）；Sandia（GADRAS、PyRIID 种子）；UPC/NEA（penEasy）；CERN（FLUKA）；OpenDose；ARPANSA；TIHM 团队 | 各自的许可文本或资格 | 补齐剩余的许可空白 | 见上文各表 | 低 |

### 检索与核对清单

| 任务 | 目的 |
|---|---|
| 下载 CASAS `labeled_data.zip`，检查标签词表（是否有床到卫生间转移、个人卫生等） | 确认如厕/卧床先验能否直接使用 |
| 从 Peplow 表 5 提取 213Bi、211Bi/209Tl 的 Γ，按分支比求 225Ac 链平衡 Γ，与 MHLW 的 0.03059 对照 | 225Ac 的 NRC 式基线 |
| 打开 Schütze 2026、Kurth 2018、Liubchenko 2024、JNM 2026 225Ac I 期的补充材料；查欧盟 Pluvicto SmPC 的排泄数字 | 补齐动力学先验 |
| 在 BioModels、Kaggle、Synapse、grand-challenge.org、PhysioNet 上按关键词检索 131I/177Lu/225Ac 数据 | 本轮未能有效检索的仓库 |
| 读全文：Yu 2019（JAMA 火化）、Han J 2026（补液）、Pratt 2018（尿粪比例）、Katzdobler 2026（照护者剂量）、Yeom 2019（姿态百分比差异）、Choi 2026、Pal Chowdhury 2026、Huang 2026 | 引用数字前核实 |
| 在 JNM、EJNMMI、EANM 摘要集中检索"SELFIE"和"patient-led dose rate" | 找 SELFIE 结果 |
| 跟踪 NCT07719062 结果（主要完成估计 2027-02）和 lucasfffaria 仓库 | 最接近的临床类比；可能的 225Ac 外照射 MC |
| 沿用上轮：取得 NRC DG-8070 全文；复查 NHS 信托手册许可 | 上轮遗留 |
| 逐一核对其余 CoMix 国家记录和 Prem 2017；核实 libRoadRunner 许可；查 QAD-CGGP-A 在 RSICC 的可得性、MicroShield 许可与价格、JAERI-M 86-071/88-198 免费全文 | 收尾许可与积累因子缺口 |

## 结论

本轮最重要的变化是：上轮"工具宽松、数据受限"的不对称，被部分打破了。体模、模拟和单例患者影像现在都有 CC BY 版本，常数和法规文本也有了免费、可再分发的来源。剩下的许可障碍集中在两处：一是 MIMIC 衍生的患者模拟器和 PhysioNet 资质数据，二是没有许可证的 PBPK 代码。两处都可以绕开：前者按论文重写人设维度，后者用 tellurium 或 PySB 按已发表方程重建。真正绕不开的仍是 IRSA 必须自己产生的数据。本轮又给这份清单加了一项：**把模拟照护者与真实照护者对齐的校准对话**。没有它，IRSA-Bench 的高分会像 τ-bench 的模拟用户结果一样偏乐观。

第二个变化在于 IRSA 价值主张的落点。Shimizu、Wakabayashi 和 Han 的结果表明，用治疗前影像或简单双指数模型预测出院当天和随后几天的剂量率已经相当准确。因此 IRSA 的增量价值不太可能体现在"第 1 天剂量率预测"上，而更可能体现在出院后的长尾：个体轨迹的持续校正，限制期、回家途中和多周期剂量的累计，行为驱动的照护者剂量，以及 α 核素时代的卫生指导。可穿戴设备的定位应当从"初始预测"转向"核对和更新"，验证设计也要预注册与这些简单基线的正面比较【推断】。

