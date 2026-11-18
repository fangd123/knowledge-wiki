---
title: "算力指数、衍生品与预测市场实证核查报告：Ornn、Silicon Data 与 Polymarket"
date: 2026-11-17
author: fangd123
license: CC BY-NC-SA 4.0
category: 算力与产业研究
tags:
  - 算力指数
  - 实证核查
  - 合约规则
  - 预测市场
  - 衍生品台账
  - 算力与产业研究
ai_assisted: true
status: published
---

# 算力指数、衍生品与预测市场实证核查报告：Ornn、Silicon Data 与 Polymarket

> 日期化事实台账：产品、合约规则、交易状态与证据边界
>
> **原研究快照日：2026-10-17；事实复核跨 2026-11-15—16，另于 2026-11-17 做外网增量复核。** 10-17 05:44/06:14 UTC 的盘口和 Reddit 数字保留为原报告历史记录，复核未重新取得当时的完整原始抓取文件，不能称为 11 月实时行情或再次独立复现。第 1.7 节列明 11-15—16 轮重新取得的一手证据及未决事项；第 1.8 节为 2026-11-17 增量复核回填。本文所称“本轮/本次核查”指 11-15—16 事实复核，“11-17 复核”另按第 1.8 节口径。  
> **阅读分工**：本文保存一手事实、合约规则、历史盘口与 `[S01]`–`[S54]` 等来源索引，不以产品出现证明客户收益。用途选择见[全景入口](国际算力定价与衍生品市场全景.md)，候选机制及竞争解释见信息经济学篇，客户反事实、影子套保和经营法律执行见[05 实践指南](算力与_Token_价格基准构建与撮合演进指南：PRA_方法论与市场设计实践.md)。 后文各篇引用同一次读取，不增加独立样本数。

## 0. 核查结论与尚未证明的事项

**第一，“高频算力数据适合作为加密衍生品参考”这一行业判断在风控计价方向上有一定合理性，但事实前提需予更正：Ornn 现行公开方法论及 API 明确为小时级更新，另有日度结算产品，本次核查未发现分钟级 OCPI 指数。** 永续合约需要及时、可信的风险计价，但报价刷新快，不等于底层算力每分钟发生了新的真实成交。[S01][S02]

**第二，Ornn 与 Polymarket 的关系已经不止“可能合作”：2026-06-02 的公司公告确认，FalconX 与 AneraLabs 做了一笔六位数美元的 GPU 大宗风险交易，Ornn 提供 OCPI 指数基础设施；这仍不能扩大解释为 Ornn 已成为整个 Polymarket 的预言机，或所有 GPU 合约都由它结算。** Silicon Data 也早已被 Polymarket 的二月 H100 合约明确引用，所以“Ornn 走币圈、Silicon Data 走传统金融”的二分法不成立。[S06][S34]

**第三，现有材料不支持“加密平台负责试验、成熟交易所负责扩展”的固定分工。** CME 的货币期货、股指期货、事件合约、FanDuel Predicts 及加密产品近全天候交易，说明既有机构也参与创新；ICE 的投资、数据分发与期货合作则属于不同活动。它们不能单独识别谁更擅长创新、投资动机或先后演进机制。[S08][S13][S21]—[S25]

**第四，已取得的 ICE 9 月 29 日公告当时未设挂牌日，CFTC 9 月 21 日函将 NYMEX 两款算力期货审查延至 11 月 9 日；这些历史文件既不能证明十月已交易，也不能证明 11 月 15 日仍待批。2026-11-17 增量复核：NYMEX 11-04 函应 CFTC 要求再延至 2027-01-08，截至复核日未见批准、挂牌或成交证据；ICE 产品页在线但无挂牌/成交显示。** Architect 百慕大 AX 的原产品页曾记录 OCPI H100 永续；11-17 生产 instruments API 显示 compute 类为 H100/H200/B200/B300 dated futures（基准 Compute Desk GPU Index），无 GPU 永续，成交仍需鉴权数据（见 §1.8）。“参考数据已进入系统”“产品已列示”和“市场发生真实交易”须分开。[S09][S11][S17][S18]

**第五，本台账确认了一些产品、引用与合作，没有完成基准代表性或客户净收益验证。** 对某条指数下注、按其结算，不能独立证明它代表产业价格；融资、授权收入和合同引用也不能替代该检验。尚需独立账单、客户现行方案、样本外影子套保及争议处理成本，才能比较是否值得采用基准或金融工具。第 8 节按不同评价目标列明缺口，具体试验流程归入 05。

### 阅读口径：什么算事实，什么算解释

| 标记 | 本文如何使用 | 不能据此推出什么 |
|---|---|---|
| 【一手】 | 公司公告、方法论、交易所通知、监管文件、市场规则/API | 公司自述不等于第三方审计；“全球首个”等营销称谓不直接采信 |
| 【二手】 | 媒体报道、新闻整理 | 不优先于更新的交易所/监管原文 |
| 【推断】 | 基于已列事实的经济机制解释、情景例子 | 不假装是公司已实施的战略或已经发生的成交 |
| 【未核实】 | 未找到足够公开证据，或只能确认部分环节 | “未发现”不等于确定不存在 |

正文的 `[Sxx]`、`[Mxx]` 均为可点击来源，完整 URL、来源日期在第 10 节。未标发布日期的网页以读取日标注，不倒推其历史状态。Reddit 是讨论样本的一手记录，不是其中商业说法的一手证明。

### 与相关研究的衔接：事实、机制与行动分开

| 相关研究 | 负责的问题 | 本台账提供什么 |
|---|---|---|
| 《国际算力定价与衍生品市场全景》 | 主体敞口、替代工具与职能地图 | 日期化产品、样本口径和合作记录 |
| 《Ornn、Silicon Data与预测市场-算力指数背后的信息经济学》 | 观察、候选机制、竞争解释及区分证据 | 已确认事项与未完成识别的边界，不以融资或引用自证理论 |
| 《加密算力对照实验与事件合约结算机制》及抗操纵篇 | 不同支付、履约和取价结构的风险 | 合约规则、取价频率和市场快照，不把模型结果当作实际安全认证 |
| 《算力与token经济-数据附录》及《token工厂的五个阶段与临界点》 | 事实索引与可并存的约束/风险面板 | 事件作为观察信号；单次违约、融资或上市不能确认某阶段 |
| 《算力与Token价格基准构建与撮合演进指南》 | 客户验证、经营方案与法律执行 | 当前证据缺口及可追溯来源，不直接指定“先事件、后期货”的路线 |

---

## 1. 事实核查：先把七类主体放回正确位置

**算力指数是“温度计”，交易合约是“按温度赔付的合同”，交易所是“组织买卖和风险管理的场所”。** Ornn、Silicon Data 主要提供前者；Polymarket、Kalshi 把特定问题做成事件合约；ICE、CME、Architect 组织不同制度下的衍生品交易。一个主体可以跨层，但这些职能不能混写。

### 1.1 重要时间线

| 日期 | 核实到的事件【一手，另注除外】 | 截至 10-17 应怎样表述 |
|---|---|---|
| 2025-05-20 | Silicon Data 公告 470 万美元种子融资，DRW、Jump 等参与 [S26] | 公告时间；后续公告回溯交割于三月，不把两者当两轮融资 |
| 2025-10-07 | ICE 宣布向 Polymarket 投资最高 20 亿美元，同时取得全球事件数据分发合作 [S13] | 承诺上限，不是已实付 20 亿；公告未证明独家分发 |
| 2025-10-28 | Ornn 公告 570 万美元种子融资，Crucible、Vine 领投，Link、Box 等参与 [S53] | 早期轮次已有数字资产市场结构背景；与后来同称 seed 的 3300 万美元不擅自相加作累计融资 |
| 2025-12-22 | CME 与 FanDuel 的 FanDuel Predicts 实际上线，CME 季报材料确认 [S23] | 不是还停留在合作意向 |
| 2026-01-23 | Ornn 官网页介绍与 Architect 合作 [S10] | 年初宣布不等于年初即有成交 |
| 2026-02 | Polymarket 二月 H100 合约规则明确使用 Silicon Data SDH100RT [S34] | 已存在的历史引用；不是 Ornn 的独家阵地 |
| 2026-03-27 | ICE 再投入 6 亿美元，确认初始直接投入 10 亿；另预计买入最多 4000 万美元老股 [S14] | 已确认直接现金投资合计 16 亿美元；老股预期与完成分开 |
| 2026-04-20 | Silicon Data 发布 GPU 隐含远期曲线 [S31] | 数据产品上线，不是期货成交曲线形成 |
| 2026-05-12 / 05-19 | CME×Silicon Data、ICE×Ornn 分别宣布算力期货计划 [S15][S08] | 都附监管条件 |
| 2026-05-28 | Architect 宣布美国算力期货交易所计划 [S12] | 与百慕大 AX 分开，不用美国计划的待批状态否定 AX 产品存在 |
| 2026-05-29 | CME 加密期货及期权开始 24/7 安排，六月一日公告确认 [S24] | 有维护窗口的近全天候交易，不是绝无停机 |
| 2026-06-02 | Polymarket 公告 FalconX—AneraLabs GPU 大宗，OCPI 结算参考 [S06] | 有具体商业试验；未披露足以独立重建交易的全部参数 |
| 2026-06-10 / 06-24 | ProCap 分发 Ornn 数据；a16z crypto 宣布领投 Ornn 3300 万美元种子轮 [S50][S04] | 前者是研究分发，后者是加密资本背景；都不自动等于链上指数合约 |
| 2026-07-14 / 07-24 | Kalshi 发布算力曲线；Ornn 网站首次发布现行 OCPI 方法论 [S39][S01] | 七月曲线公告不是 Kalshi 算力合约首次存在的证明；新方法论不能无条件追溯覆盖旧序列 |
| 2026-08-11 | Silicon Data 宣布 A 轮首关 3050 万美元；CME 宣布拟于十月五日推出算力期货 [S27][S16] | 首关金额；上市日期附监管条件 |
| 2026-08-19 / 09-21 / 09-28 | CFTC 征询算力衍生品意见、延长 NYMEX 审查；CME 明确延期 [S19][S18][S17] | 审查延至十一月九日不是承诺届时批准 |
| 2026-09-29 / 10-05 | ICE 通知十月五日起提供合约参考数据，含假设结算价 [S09] | 供系统准备，不等于正式交易；通知未给确定上市日 |
| 2026-10-17 读取 | AX 官网页列 OCPI H100 为现有永续品种；Poly 十月底 H100、Kalshi H100 月均市场可见 [S11][S35][S38] | 可确认产品/规则存在；规模、盘口深度和产业采用须分别核验 |

### 1.2 “Ornn 分钟级”到底怎么回事

| 容易混淆的频率 | 公开材料告诉我们什么 | 判断 |
|---|---|---|
| 数据记录时间戳 | 可精确到毫秒 | 记录精度不是新价格频率 |
| OCPI 小时现值 | 一小时窗口，窗口结束后发布；Current Price API 明确每小时更新 [S01][S02] | **小时级有直接证据** |
| OCPI 日度序列 | 原报告记录日度产品；11 月 15 日重取 [FAQ](https://data.ornn.com/faq) 将网站日参考定义为截至美东午夜的 24 小时成交量加权缩尾值 | 不能当小时值简单平均，也不能把 Kalshi 某周度事件的 16:00 取价时刻移作指数日界线 |
| ICE 拟议结算 | 日值取小时值平均，再取月内日值平均 [S09] | 是合约自己的汇总规则，不等于所有 OCPI 日值定义 |
| 界面实时刷新、合约分钟 K 线 | 本次未取得能证明分钟级 OCPI 独立重算的文件 | 【未核实】；行情成交可以每秒变化而现货锚一小时才更新 |

**核心核查结论：现有公开证据表明 OCPI 采取“小时级现值＋日度结算产品”架构，并不是“分钟级指数”。** 如果商业版确有分钟频率，需要补交版本日期、字段定义、连续样本和 SLA。SLA 就是“供应商书面承诺多快更新、断了怎么办”，不是营销中的“实时”。

现行方法论还解决了一个旧争议：早期 Ornn US H100 指数取租赁报价；现行 OCPI 声明取不同交易对手之间已实际使用、正在付费的按需租赁，按 GPU 数量加权并对极端价缩尾，即“保留交易，把特别离谱的数压到上下界”。长期预留和远期租约另列，不进这一现货样本。【一手·供应商方法自述】[S01]

因此，“Ornn 只是爬挂牌价”已经不是对当前方法的准确描述；同样，“官方说真实成交，所以一定覆盖全部市场、无法操纵”也不成立。**正在运行的租约被持续观察，不一定意味着每一小时都新签了同等数量的新合同。** 新增成交、存续租约存量、价格重设三种变化，应要求分列。【推断】

### 1.3 Ornn 与 Polymarket：合作有多深

| 环节 | 已确认【S06，2026-06-02】 | 尚不能确认 |
|---|---|---|
| 谁交易 | FalconX 与 AneraLabs；六位数美元大宗 | 精确金额、执行价格、各方最终净敞口 |
| 对应什么产业风险 | 公告称服务于 Anera 平台供应商的推理远期容量合同及续约风险 | 风险减少多少、与现货的相关系数、后续复制次数 |
| Ornn 做什么 | 提供 OCPI H100 指数基础设施/结算参考 | 独家授权、收费率、经营整个预言机网络 |
| Polymarket 做什么 | 提供链上风险交易及结算基础设施 | 替供应商交付 GPU、保证基础算力履约 |
| 链上证明什么 | 公告称 Polygon 留下可审计记录 | 本次未拿到公告对应交易哈希，未独立重建整笔交易 |

**结算源**就是“合同说最后看谁家的数”；**预言机**就是“把那个数或裁定结果可信地送进程序的机制”。Polymarket 通用文档还涉及 UMA 的提出结果与争议处理；数据生产者、数据传送者和最终裁定机制并非同一个主体。[S40] 本次不能把 Ornn 的数据角色写成“已经接入 Chainlink/Pyth”，也不能把这笔大宗和 FalconX 另行披露、以 Robert Leshner 为对手方的 OTC 算力交易混成一笔。[S07]

### 1.4 ICE 投 Polymarket，和 ICE×Ornn 是什么关系

事实是三条独立成立的连接：**ICE→Polymarket 股权与事件数据分发；ICE↔Ornn 拟议期货；Polymarket↔OCPI 的实际风险交易。** 这构成可相互强化的业务网络，但没有证据证明三方已经签成排他联盟、共享全部数据，或 ICE 已控制 Ornn/Polymarket。[S06][S08][S13][S14]

【推断】ICE 的布局有两层收益可能：一层从合约成交与清算收费，另一层把“某件事发生的市场概率”作为数据卖给机构。即使某个 GPU 预测合约尚未形成大市场，分发新型预期数据也可能有价值。这恰恰说明传统机构懂得“交易产生信息，信息又能成为商品”，不是只懂把既有合约做大。

### 1.5 2026-10-17 历史市场快照：合约存在与结算规则

以下是原报告记录的【规则/API 历史快照】，本轮未重建该时刻的全部响应。表中的“未关闭”“active”“当前产品”均只指 10 月 17 日；**不同平台单位不能直接相加或比较大小**。

| 平台/市场 | 合同实际问的问题 | 数据源和规则 | 快照规模及限制 |
|---|---|---|---|
| Polymarket：二月 H100 触及阈值，已关闭 [S34] | 二月规定期间内是否曾达到某价 | Silicon Data SDH100RT 日值；次日点发布后前日最终化；修订有专门约束 | 累计 volume 约 **337,175 美元**；历史存在不代表今天仍能买 |
| Polymarket：十月底 H100，未关闭 [S35] | 10-31 最终日度 H100 值落在哪个区间 | Ornn；不是整月均价；有七个日历日后采用可用数据等后备条款 | 累计 **37,713.52 美元**，24h **1,738.87 美元**；平台 liquidity 指标 7,372.12，不是保证可成交的深度 |
| Kalshi：H100 周度 KXH100WS [S36][S37] | 指定周五 16:00 美东时间是否高于阈值 | Ornn；缺当点可取最近可用点；不采到期后修订 | API 返回的最新事件为 10-09，十二档 volume_fp 合计 **15,088.45 合约**；未据此推断 10-16 市场是否取消 |
| Kalshi：H100 十月月均 KXH100MS-26OCT [S38] | 十月小时租价均值是否高于阈值 | Ornn 整月小时值算术均值；四舍五入两位；到期后修订忽略 | 九档累计 **16,007.62 合约**，抓取时 active；不是 16,007.62 美元实际投入 |
| Kalshi：H100 十一月/十二月月均 [S38] | 相应月份均价是否超过各阈值 | 同上 | 分别 **10,193.99 / 11,052.96 合约**；不同到期不能当相同风险敞口 |
| Architect AX：OCPI H100 永续 [S11] | 持续暴露于 H100 租价变化 | OCPI；每日 funding，平台 23/7 | 快照时产品已列示；未取得 GPU 单品成交量、持仓和完整盘口历史。11-17 生产 API 已无 GPU 永续、改为 dated futures，见 §1.8 |

同日 **06:14 UTC** 的另一组历史记录还包含 B200、H200 与 A100。它说明当时列示的品种不只 H100，不代表今天仍可交易：

| 市场【一手·API】 | 规则/状态 | 抓取时累计量 |
|---|---|---|
| Polymarket：2026 年末 B200 [A05] | 未关闭；按 Ornn 12-31 最终日值分档，不是全年均价 | **104,010.24 美元**；24h 1,374.68 美元 |
| Kalshi：十月 B200 月均 [A06] | 19 档 active；Ornn 小时值月均 | **33,190.80 合约** |
| Kalshi：十月 H200 月均 [A07] | 11 档 active；Ornn 小时值月均 | **28,990.08 合约** |
| Kalshi：十月 A100 月均 [A08] | 7 档 active；Ornn 小时值月均 | **10,618.90 合约** |

这些是同类品种的代表性切片，未穷尽全站所有到期、历史系列和迁移市场；不把它们加成“算力风险总规模”。

Kalshi 的 `volume_fp` 是带小数的合约数量字段。二元合约通常每份结果面值 1 美元，**面值、买入成本、成交量、持仓量仍是四个概念**；不能把合约数量当 GPU 卡时或产业采购金额。八月 H100 月均事件九档合计约 141,992.70 合约，说明不同月份活跃度差异明显；本次未重建平台所有算力品种的历史总量。[S38]

十月底 Polymarket 的例子也显示盘口差异：2.50—2.75 档 YES 最佳买/卖价约 0.33/0.34，而低于 2.00 档约 0.003/0.06。[S35] **价差**就是“马上买与马上卖之间差多少钱”，前者紧、后者宽；一个事件的总成交数字不能证明每个尾部结果都有可用流动性。

### 1.6 Silicon Data 的“币圈联系”不能漏掉

| 联系 | 证据和日期 | 应保留的边界 |
|---|---|---|
| 预测市场实际引用 | 二月 Polymarket 规则直接写 SDH100RT [S34] | 证明被用作结算源；未证明正式商业签约、付费或排他 |
| 加密相关投资人 | 2025 种子轮 DRW、Jump、Wintermute Ventures 等；2026 A 轮再见 DRW、Jump、Wintermute、VanEck 等 [S26][S27] | 做市/量化/加密资本与传统金融交叉；投资不是承诺上线或保证流动性 |
| DRW—Cumberland | DRW 官方介绍集团加密业务 Cumberland [S33] | 不能写成 Cumberland 这一实体已经投资或采购 Silicon Data |
| 终端分发 | Bloomberg；dxFeed 带入 Refinitiv [S28][S30] | 是覆盖金融工作流，不等于期货获批 |
| DePIN、链上预言机、永续 | 本次定向搜 Chainlink、Pyth、DePIN、GPU perpetual，未找到足够官方证明 | 【未核实】不列具体 feed ID、链上地址或平台代码 |
| 隐含远期曲线 | 2026-04-20 推出，5-01 方法说明 [S31][S32] | 是当前租期结构推导，不能直接当成交形成的“未来一致预期” |

**DePIN**是“用代币和分散参与者组织现实硬件服务的网络”。链上支付能让资金转移更透明，却未必证明 GPU 真实做了多少有效计算；《加密算力对照实验与事件合约结算机制》第 3 节的透明性优势应保留，但“链上＝所有租赁资料完整、性能完全可验证”需要收窄。

Ornn 的加密背景也不只来自 2026 年 a16z crypto：2025-10-28 的早期融资公告已有 Crucible、Vine 等，Crucible 的 Meltem Demirors 在公告中明确以数字资产市场结构投资经验解释投资逻辑。[S53] 当前官网列 Galaxy 等基金，并列“来自 Coinbase 等机构的天使”；后者是个人任职背景，不是 Coinbase 公司投资。[S54] 这支持“有加密金融网络”，不证明它已经获得任何交易许可或做市保证。

本次也未核实到 Hyperliquid、Chainlink、Pyth 正式引用 Ornn 的可识别 GPU 合约或 feed；Ornn 官网客户 logo、币圈投资人名单都不能替代这种证据。Hyperliquid 通用预言机文档列的是其相关现货市场的取价机制，并不能穷举全部第三方新市场。[S03][S41] **结论是“尚未找到”，不是“不可能有”。**

### 1.7 2026-11-15—16 复核：已确认的原文与尚未确认的现状

本轮重新读取了 CFTC 审查函、ICE 规格、CFTC 征求意见公告、Ornn 方法论、Silicon Data 产品页/变更公告和 Kalshi series API。其支持范围如下：

| 对象 | 本轮能确认什么 | 不能外推什么 |
|---|---|---|
| CFTC / NYMEX [S18] | 9 月 21 日函针对 26-370、26-370S，将初始审查再延长 45 天至 **11 月 9 日**；原因表述为需要更多时间分析新颖或复杂问题 | 到期不等于获批；函件不说明 11 月 15 日状态，也不认定已存在操纵 |
| 一般算力征求意见 [S19][S18] | 8 月 19 日新闻稿称自《联邦公报》刊登后 60 天；9 月函脚注确认 8 月 21 日刊登、意见截止 **10 月 20 日** | 一般意见期结束不是 ICE 或 CME 某款合约的终审/上市决定 |
| ICE `HPR` / `BKL` [S09] | 9 月 29 日当时未定挂牌日；10 月 5 日起拟提供参考数据及假想每日结算价。乘数是**合约月实际小时数**，最多 24 连续月份；报价刻度 0.001 USD/GPU-hour，最小变动可随交易类型不同。最终价是月内日度 OCPI 均值，日值来自小时均值 | 参考数据不等于成交；每日结算窗口不是最终指数采样窗口；不能套用 CME 的 730 标准化乘数或 36 月描述 |
| CME 具体规格 [S17][S20] | 原稿记录 `GPU1/GPU2`、730 卡时、0.01 USD/GPU-hour 跳价和 36 月；本轮产品/通知页访问超时，保留为待复核历史参数 | $730=8760/12$ 是年均月小时数，非所有自然月实际工时；“10 月 15 日曲线分发”“11 月无正式成交”未获本轮证据支持 |
| Silicon Data H100 [S28] | 11 月 14 日带日期 neocloud `SDH100RT` 为 **3.13 USD/GPU-hour**；hyperscaler 另列，页面说明标准化和异常过滤 | 同页 FAQ 的 2.53 USD/GPU-hour 是无相应更新日期的冲突值；不混成全市场成交均价，不推断千笔独立成交 |
| 指数推出日期 [S29] | H200：6 月 30 日公告、7 月 15 日生效、历史起点 5 月 4 日；B300：9 月 11 日公告、9 月 14 日生效、历史起点 4 月 23 日 | 回填历史起点不等于首次对外发布日 |
| Kalshi `KXH100WS` [S36] | series API 列 Ornn 为 settlement source，含源机构员工和持重大非公开信息者不得交易的限制 | series 元数据不能验证某一事件当前成交量，也不自动确定每份合约的时间、阈值与修订规则 |

旧补充段落的“白金级确定性溢价 25%–40%”、B200 `$1.73/chip-hour` 普适 TCO 及“租金减 TCO 即垄断溢价”缺少匹配成交样本与可复算模型，已撤出实证结论。TCO 必须明确采购价、折旧、融资、配套设施与付费利用率；质量评级不是信用评级，也不能直接识别租金差异的因果来源。

### 1.8 2026-11-17 外网增量复核回填

本节读取日为 2026-11-17（Asia/Singapore）。监管函、交易所通知、公司公告及 API 地址列在 §9；这些材料能说明该日记录了什么，不能把公司自述升格为独立审计，也不能用接口失败断定产品状态。

| 对象 | 11-17 能确认什么 | 证据等级 | 仍不能确认什么 |
|---|---|---|---|
| CME/NYMEX GPU1/GPU2（26-370/26-370S）批准状态 | CFTC 官网 NYMEX 2026-11-04 函（Submission 26-370）：应委员会要求，NYMEX 同意将审查期再延 60 天至 **2027-01-08**；延期以其他 CFTC 注册机构在此期间不自证、获批或挂牌可比合约为条件，若发生则以更早日期为准 | 一手（监管备案函） | 截至 11-17 未见批准决定；“未挂牌/无成交”是未见证据的状态，不是交易所确认 |
| NYMEX GPU 做市商激励计划（26-524） | 2026-10-26 备案函：GPU Compute Liquidity Provider Program，覆盖 GPU1/GPU2，拟 11-10 或获批日生效、至 2027-10-31，双边报价义务+预设激励；CFTC DCM Rules 备案列表显示该提交 **Withdrawn（2026-11-04）** | 一手（备案函+监管列表） | 撤回原因未披露；不能推断做市安排已落地或产品已具备流动性 |
| ICE HPR/BKL 挂牌与成交 | ICE 产品页（H100 产品 ID 83050006、B200 83050009）在线，规格与 9-29 通知一致（合约月实际小时数、0.001 USD/GPU-hour、最多 24 连续月）；页面未显示首个交易日、成交或持仓。9-29 通知原文确认挂牌仍待 CFTC 征求意见程序完成、未设挂牌时间表 | 一手（交易所页面/通知） | 产品页在线不等于已挂牌交易；ICE 通知列表页为 JS 渲染，未能穷举 10 月后新通知；无成交流水可查 |
| Ornn 分钟级商业 feed | 方法论（changelog 仅 2026-07-24 初版）：滚动 1 小时窗口、窗口收盘后尽快发布、毫秒时间戳；FAQ/API 文档：免费层日度、Premium 最新小时值、Full 层 `/api/gpu/{gpu}/hourly` 小时序列，H100 小时历史自 2025-01 起 | 一手（公司方法/文档） | 未见任何分钟级发布承诺或字段；“分钟级 feed”维持未证实；小时序列的独立成交密度未审计 |
| Ornn 监督委员会 | 方法论 §5.1 仍为 “is establishing”，规定 3–5 名成员、多数独立（24 个月冷却期）、组成将在组建后公布；本次未见成员名单公告 | 一手（方法原文） | 委员会是否已实际组建仍未核实；计划条款不等于治理已落地 |
| Architect AX 算力合约现状 | 生产 API `GET /api/instruments`（无鉴权）：compute 类为 **H100/H200/B200/B300 的 SEP/DEC-2026、MAR-2027 dated futures**（730 GPU-hours、tick 0.001 USD、基准 “Compute Desk NVIDIA … GPU Index”、标记价 “Average price on AX at 4pm EST”、初始保证金 20%/维持 15%）；SEP-2026 已标 `is_closing`。PERP 列表仅含 FX/股票/金属/能源/利率，**无 GPU 永续**。官方文档明确 dated futures 无资金费；`/tickers`、`/trades`、`/candles` 返回 `no auth presented` | 一手（交易所 API/文档） | 无公开成交流水，首个交易日、成交量、持仓与客户结构仍未核实；旧稿“H100 永续”与当前生产目录不符，第三方所称“4 月 3 日 Ornn H100 永续上线”未获一手确认；AX 美国 AI Exchange 页面自称受 CFTC 监管但页脚仍写“subject to regulatory review”，其 DCM 状态需另核 |
| Ornn–Polymarket 六月大宗细节 | 2026-06-02 PRNewswire 稿全文重取：对手方 **FalconX × AneraLabs**，六位数美元，结算引用 OCPI H100，**结算记录在 Polygon 链上**；为 Anera Exchange 上一份可交付推理远期容量合约嵌入的对冲（offset 续约风险）；FalconX 表示拟担任后续大宗做市商 | 一手（公司公告） | 稿内未给交易哈希、精确金额、赔付结构与授权费；“链上可审计”是公司主张，本次未定位具体 tx；持续后续成交未证实 |
| Chainlink/Pyth/Hyperliquid GPU feed | Pyth Hermes 全量 feed 目录（1951 条）无 GPU 租赁价格 feed（命中项均为股票/商品同名）；Hyperliquid info API：`xyz:H100`、`para:H100`、`io:H100` 三个 HIP-3 市场均 **isDelisted=true、OI=0、当日成交=0**（显示价 2.6/2.56/2.78 USD）；Chainlink 官方 feed 目录访问失败；另有 Inferra 自称经 Chainlink CRE 在 Solana 存证 GPU 指数 | 一手（Pyth/Hyperliquid API）；Inferra 为公司自述 | Hyperliquid 三市场历史成交与下架原因未核实（当前零 OI 不证明历史零成交）；Chainlink 官方 GPU feed 未见但也未能穷举目录；不能据此断言全部预言机网络无 GPU 数据 |
| Silicon Data × Syntax 计算代理 | Syntax 官网 2026-09-08 公告全文：Syntax 受任为 Silicon Data GPU/AI 基建基准家族的独立指数计算代理，按日计算、方法文档化、审计与监督流程；指数发起方与计算代理分离 | 一手（双方联合公告） | 计算代理独立不等于原始数据独立审计或 IOSCO 认证；实际执行质量未见第三方报告 |
| Silicon Data–Polymarket 商用签约 | 本次检索未见双方商用授权公告；二月合约公开引用关系维持已核实 | 未核实（无新证据） | 公开引用不能补写成付费签约 |
| Kalshi 算力系列现状 | trade API（api.elections.kalshi.com 与 api.kalshi.com）及 news.kalshi.com 本次均连接失败，未能重取 series/events；Reuters 2026-11-12 报道 Kalshi 洽谈融资、估值 650 亿美元、扩展算力期货平台（转述 Bloomberg） | 二手（Reuters）；API 未核实 | 当前系列清单、逐日成交与裁定细则未更新；10-17 快照保留为历史记录 |
| MSCI Index 分部 43.1% | SEC EDGAR 重取 MSCI 2025 Form 10-K（msci-20251231.htm）原文：“For the year ended December 31, 2025, asset-based fees accounted for 43.1% of the total revenues for our Index segment” | 一手（法定披露） | 该比例是 MSCI 自身收入结构，不外推 GPU 基准可复制的费率水平 |
| IOSCO PD415 | 官网 PDF 本次成功取得（FR07/13，2013-07，48 页），19 项原则标题逐项核对，与本报告及姐妹篇映射一致；15 为 “Internal Controls over Data Collection” | 一手（标准原文） | 原则为比例适用的国际标准，不构成任一法域的自动合规义务 |
| Compute Exchange 费率页 | compute.exchange/fees 重取：仍标 “Last Updated: January 30, 2025”；供应商佣金 4%（基数含服务费、押金、取消费等）、迟付预留费 10%、长期订单 10% 取消保证金，另有量折与转售佣金条款 | 一手（平台条款页） | 页面版本未变不等于所有当前订单实际按此执行；二手硬件/Token 远期无单独费率页 |
| ICE–Polymarket 投资最终条款 | 本次未见新的最终交割/排他性披露；维持 16 亿直接投资已确认、独家性未核实 | 未核实（无新证据） | 不用最初最高 20 亿框架倒推实付与控制权 |

---

## 2. 证据到机制：本台账支持到哪一步

本报告维护观察记录，不重复理论推导。机制、竞争解释与区分证据见信息经济学篇；客户反事实与试点步骤见[实践指南第零节](算力与_Token_价格基准构建与撮合演进指南：PRA_方法论与市场设计实践.md)。

| 记录 | 证据所支持的判断 | 尚不能据此确认 |
|---|---|---|
| Ornn 方法与 API [S01][S02] | 声明的样本范围、计算和发布字段 | 客户账单覆盖率、最优频率与抗操纵成本 |
| FalconX—AneraLabs 大宗公告 [S06] | 六位数美元的一笔风险交易及 OCPI 引用 | 行业持续需求、客户净收益及具体数据授权收入 |
| Kalshi 资格与 Polymarket 争议规则 [S36][S40] | 特定平台/合同的参与和裁定安排 | 对底层市场代表性的独立认证 |
| 2026-11-15—16 读取的 [OAK 第三方页面](https://oakresearch.io/en/hyperliquid/hip-3/markets/xyz:H100) | `xyz:H100` 标示已下架，当前成交/持仓为零，显示价格 2.6 USD | 历史零成交、价格崩盘、下架原因或所用指数来源 |
| MSCI、LME、LBMA 制度材料 [S44][S45][S46] | 成熟基准收费和治理的比较对象；43.1% 已于 11-17 从 10-K 原文核实（见 §1.8），指 MSCI 自身 Index 分部资产型收费占比 | GPU 基准收入结构；MSCI 费率结构不可直接外推 |

## 3. 频率与结算用途：已知字段及待核参数

| 核查对象 | 已知口径 | 尚需的可检验材料 |
|---|---|---|
| OCPI [S01][S02] | 活跃按需租赁的滚动小时样本；排除预留、长约和远期 | 新成交/存续租赁比例、独立来源集中度、净价与重定价时间 |
| 网站日参考与 API 字段 | 日参考、小时序列和滚动平均需按版本区分；见 §1.7 | 字段映射、时区、修订与实际发布日志 |
| AX H100 产品 [S11][S12] | 原产品记录列示永续及参考基准；11-17 生产 instruments API 显示 compute 类为 H100/H200/B200/B300 dated futures（基准 Compute Desk GPU Index），无 GPU 永续，见 §1.8 | 单品成交、持仓、标记价、过期保护和强平参数；行情端点需鉴权，成交仍未核实 |
| Polymarket / Kalshi [S34]—[S39] | 具体合约可能引用最终日值或小时月均，见 §1.5 原快照 | 对应合约的当前规则与最终结算记录，不能跨合约互换 |

方法已声明不等于安全参数已验证。采样窗口、刷新频率与结算平均期的关系及条件模型集中见[抗操纵篇 §2–4](算力基准抗操纵微观结构与_IOSCO_治理设计.md)；连续风控与离散结算的损失路径见[加密对照篇 §5.4](加密算力对照实验与事件合约结算机制.md)。

## 4. 机构创新与合作：历史观察，不作阵营排序

| 日期化记录与来源 | 支持范围 |
|---|---|
| CME 1972 年货币期货、1982 年 S&P 500 期货 [S21] | 成熟机构也曾创设新品，不能定义为只做复制扩张 |
| ICE 2000 年从电子能源交易起家及后续数据业务 [S25] | 创业与成熟阶段不能仅凭当下身份划分 |
| ICE 投资 Polymarket、事件数据分发 [S13][S14] | 股权与渠道合作；控制权及排他权另需合同证据 |
| CME 2022 年事件合约、2025 年 FanDuel Predicts [S22][S23] | 传统交易所也直接尝试事件产品和零售渠道 |
| CME 2026-05-29 加密期货期权近全天候交易 [S24] | 交易时段也会调整，不宜用“周末停市”概括全部产品 |

这些记录可否定“传统机构从不创新”的绝对命题，却不足以估计两类组织的创新成功率。识别上线速度的原因还需匹配产品、客户、监管程序、失败样本和持续采用；资本过剩、技术组合、渠道协同等属于竞争解释，详见理论篇 §4–5。

## 5. 理论来源索引与检验接口

理论文献证明其模型或概念的来源，不是当代市场事实的再次验证。完整解释放在信息经济学篇，本台账保留出处及需要接上的观察量：

| 理论主题 | 来源 | 进入经验判断时所需材料 |
|---|---|---|
| 分散知识、信息成本与逆向选择 | [S47][S48][S49] | 信息获取成本、质量差异、报价与净成交 |
| 交易成本与制度选择 | Coase [S51]、North [S52] | 直接采购与经中介的询价、履约、回款和争议成本；价格发现效果另用可执行报价与成交检验 |
| 商品、价值尺度与劳动抽象 | [M01][M02] | 实际服务规格；GPU-hour 不等于抽象劳动时数，指数不等于货币 |
| 周转、流通费用与中介分工 | [M03][M04][M05][M06] | 资金占用、闲置、寻源和对账成本及中介收费 |
| 生息资本、资本化与信用 | [M07][M08][M09][M13] | 合同现金流、抵押、杠杆、优先级与损失承担 |
| 地租、资本集中与金融资本 | [M10][M11][M12] | 控制权、替代选择、进入退出、治理成本和风险调整超额收益 |

ProCap 与 Ornn 的 6 月 10 日新闻稿 [S50] 是第 6 节的数据分发案例，不是价格发现理论的出处。基准被采用、价格相关、发生过一笔交易，都不能单独识别客户效益；比较对象还包括不建指数、不套保或继续双边采购。

## 6. 产品、合作与合同：按日期和权利区分

**原表观察截止 2026-10-17。** 以下是当时可见的产品、方法和合作，不以 11 月更新倒填；涉及交易所和治理的后续变化另列在表后。

| 维度 | Ornn | Silicon Data |
|---|---|---|
| 定位 | 算力市场＋指数＋金融产品连接，关联业务包含实际算力交易 [S01][S03] | 自我定位为独立价格、性能与风险数据层 [S27][S28] |
| 指数频率 | 公开小时现值；另有日度 close/日度基准；分钟级未核实 [S01][S02] | 核心 GPU 租赁基准每日发布；“实时透明”营销不等于分钟指数 [S28] |
| 方法论 | 当时 OCPI 声明基于已实现按需租赁，一小时窗口、GPU 数量加权缩尾；租期长约另列 [S01] | GPU 与云类型分组，调整租期、集群和互连等可比性；具体覆盖/权重不能仅凭宣传页完全复建 [S28][S29] |
| 数据来源 | 全球动态定价供应商的匿名有效租赁；未公开全体贡献者名录和独立审计样本 [S01] | 聚合多云租赁资料；官网覆盖率为公司自述，旧笔记的数据点数不能直接当独立成交数 [S28] |
| 覆盖卡型 | 公开免费 API：H100 SXM、H200、B200、A100 SXM4、RTX 5090；更多跟踪品种与免费权限分开 [S02] | H100/A100/B200 等，已扩 H200、B300 等；B300 9-14 生效，历史可回填更早 [S29] |
| 交易所伙伴 | ICE 拟议 H100/B200 期货；原 AX 产品页列示 OCPI H100 永续 [S08][S09][S11] | CME/NYMEX 拟议 H100/B200 期货 [S15][S20] |
| 预测市场关联 | Polymarket 六月机构大宗、十月底日值市场；Kalshi H100 周/月市场 [S06][S35]—[S38] | 二月 Polymarket H100 合约直接引用 SDH100RT；商业合作合同未核实 [S34] |
| 加密/做市资本 | 2025 年 570 万早期种子有 Crucible、Vine 等；2026 年 a16z **crypto** 领投 3300 万种子轮；官网列 Galaxy 等 [S53][S04][S54] | DRW、Jump、Wintermute 等跨传统/加密；A 轮又有 CME Ventures、VanEck 等 [S26][S27] |
| 终端与分发 | Bloomberg；ProCap 独立投资者研究分发；公开 API [S03][S50][S02] | Bloomberg、dxFeed→Refinitiv 等 [S28][S30] |
| 远期曲线 | 预留/长期合同另做隐含远期，与按需现货指数分开 [S01] | 4-20 发布，由当前租期结构推出的隐含远期；不是逐期连续交易的远期合约成交曲线 [S31][S32] |
| 美国期货监管状态 | 9-29 通知未确定挂牌日；10-05 参考数据含假设结算价 [S09] | 原定 10-05；9-28 通知延期，此时可见的审查截止日为 11-09 [S17][S18] |
| 治理要核什么 | 关联输入、缩尾与验证权限；自述参考 IOSCO 不是认证 [S01][S43] | 方法修订、基准权重及历史重述影响可比性 [S29] |

**补查日：2026-11-17（见 §1.8、§9）。** 这三行不属于 10 月快照。

| 补查项目 | 11 月读到的材料 | 不能据此证明 |
|---|---|---|
| CME/NYMEX 与 ICE | 11-04 NYMEX 函同意有条件再延至 2027-01-08；ICE 产品页在线但未显示首个交易日。 | 已批准、已成交或一定未成交。 |
| Architect AX | 生产目录列出基准为 Compute Desk GPU Index 的有到期日期货，未见原 GPU 永续；成交接口需鉴权。 | 某合约历史上从未存在，或目录内产品都有成交。 |
| 基准治理 | Ornn 方法仍称监督委员会在建立中；另据 9-08 公告，Silicon Data 聘 Syntax 为独立计算代理。 | Ornn 委员会已履职，或 Silicon Data 的样本来源已获独立审计。 |

这组记录说明股权、数据分发、合同引用和交易合作不能合并为一个“阵营”标签。对利益冲突、客户选择和渠道效应的解释移至理论篇，不由主体名单直接推出企业战略、独立性或商业成功。

### 6.1 对“远期曲线”的额外提醒

**远期曲线**就是“不同未来期限对应的一排价格”。名字相同，来源可能完全不同：

| 曲线来源 | 直观理解 | 最重要的限制 |
|---|---|---|
| 不同期限预留租约推导的隐含远期 | 两年总价减去一年总价，把差额分到第二年 | 必须处理折扣、支付时点、可取消性、性能和客户差异；未必能买短租、卖长租完成套利 |
| 期货/远期真实成交 | 不同到期各自有人按某价成交 | 也含风险溢价、流动性与基差；薄市场最后一笔可能过时 |
| 事件阶梯推导 | 从不同阈值的 YES/NO 价格还原分布和均值 | 尾部、插值、盘口不一致和风险偏好；不等于直接买到该均价的线性保护 |

【推断】最简算术示意：一年租价 3 美元/小时，两年租价 2.70，忽略贴现且前后年度小时数相同时，第二年残差为 `2×2.70−3=2.40`。它说明长租隐含的第二年价格，**不能证明第二年开租的一年合同今天真的有人按 2.40 卖**。若租约不可转让、不可拆分，严格无套利结论尤其不能生搬。[S32]

因此，曲线比较应同时记录观测合同和推导假设；有一排期限价格，不代表各期限均有可执行交易。

### 6.2 2026-11-15—16 合同、授权与产品资料读取台账

本表集中保存理论篇与全景篇引用的合同证据。读取日不是合同生效日，公司模板和产品自述也不是实际全业务敞口或第三方审计。

| 原始资料及日期 | 可以确认的内容 | 不能据此确认的事项 |
|---|---|---|
| [Ornn 客户条款](https://compute.ornn.com/legal/ornn-terms-and-conditions.pdf)，文件标示 2026-09-17 | 主体为 Ornn Clearing LLC；§16.8 的非代理约定、§3.9 的 Platform Data 权利 | 名称含 Clearing 不等于取得 CCP 牌照；模板不证明所有订单责任相同 |
| [Ornn Supplier MSA/SLA](https://compute.ornn.com/legal/supplier-msa-sla-october-rev1.3.pdf)，本轮读取 | Ornn 作为 Customer；预留容量原则上照付不议，供应方未达可用性等情形另按条款处理 | 实际签约覆盖、期限错配、净敞口及资本缓冲 |
| [Ornn 数据用途授权](https://data.ornn.com/premium-data-use)，本轮读取 | 内部动态定价与再分发、预言机及金融产品用途区分授权 | 客户实际支付费率、付费规模和超额租金 |
| [Compute Exchange 服务条款](https://compute.exchange/compute-service-terms)及[费用页](https://compute.exchange/fees)，费用页标示 2025-01-30 | CEX 不是算力服务合同当事方；供应商 4% 费率，与迟付 10% 费用及长期订单 10% 取消保证金分别记录 | 所有当前订单统一采用该费率，或平台没有其他责任 |
| [Syntax 任命公告](https://www.syntaxdata.com/research/silicon-data-appoints-syntax-data-as-independent-index-calculation-agent-advancing-institutional-infrastructure-for-compute-risk-management)，网页 2026-09-07、稿内 09-08 | Silicon Data 任命独立指数计算代理 | 原始数据独立审计、全面利益冲突消除或 IOSCO 认证 |
| [Vijay Naik 访谈文章](https://www.thetokendispatch.com/p/a-tale-of-two-computes)，2026-10-11 | 受访者描述三月试验及之后其业务中有限的相关需求 | 全行业套保需求大小，或三月试验必然就是六月公告同一笔交易 |
| [SemiAnalysis GPU Pricing API](https://semianalysis.com/gpu-pricing-api/)，2026-11-16 读取 | 页面自述 164+ SKUs、9+ 年历史、小时更新，含 A100/H100/B200 Spot/Contract 资料 | 每次刷新都有独立新成交、覆盖全市场或商业许可允许任意再分发 |
| [ClusterMAX](https://clustermax.ai/)及[SemiAnalysis 导航](https://semianalysis.com/)，2026-11-16 读取 | 前者导航标示 Rankings 3.0 为 Sep 2026；后者列 84 GPU clouds rated | 评级数等于市场份额，或评级导致某一确定价格溢价 |

条款如何改变履约、信用及数据生产激励，见信息经济学篇 §8；经营与法律执行不从这些模板直接推出，统一见 05 指南。

---

## 7. Reddit 舆情：有兴趣、有疑问，但不是产业采用统计

### 7.1 检索执行与样本口径

**原报告记载的历史检索过程，事实复核未重新执行。** 原稿称通过 PRAW 8.0.3 执行结构化抽样；下表保留其口径和数量，但复核未取得对应原始 JSON、执行日志或环境锁定记录，因此不能再次独立确认“110 组全部成功”及总计数。帖子链接是进一步核验入口，不能替代完整样本档案。

| 项目 | 实际执行 |
|---|---|
| 关键词，10 个 | `Ornn compute`、`OCPI GPU`、`Silicon Data GPU index`、`GPU futures`、`compute futures`、`H100 rental price index`、`Kalshi compute`、`Polymarket GPU`、`Architect perpetual compute`、`hashprice` |
| 子版块，11 个 | all、CryptoCurrency、wallstreetbets、MachineLearning、LocalLLaMA、Polymarket、Kalshi、algotrading、investing、hardware、nvidia |
| 组合 | 全部 110 组成功；每组相关性排序、最多 12 条，不是全量历史爬取 |
| 时间 | 完成于 2026-10-17 05:56:03 UTC；帖子日期截至 10-17 |
| 原始样本 | 去重 476 篇；标题、得分、评论数、时间、链接、正文及最多三条高分顶层评论；保存 1,279 条评论记录 |
| 最终错误 | 搜索失败 0，评论抓取错误 0；最终状态 complete |
| 早期运行 | 首轮长时间等待后人工中断、退出码 1；重试后完成。不能把首轮无输出解读为 Reddit 无相关讨论 |

检索去重得到的 476 篇记录**并非 476 篇有效舆情**：搜索会混入《英雄联盟》角色 Ornn、普通电脑配置、天气预测系统中的 compute 等同名歧义。本节人工筛选能回应研究问题的代表项，未对全部命中记录做自动化正负情绪统计。

### 7.2 代表性帖子与高赞评论摘要

以下为【一手·Reddit 讨论记录】，得分和评论数均是抓取时快照；摘要为中文转述，不认定匿名发言者具有其声称的从业资历。

| 主题/帖子 | 日期、得分、评论数 | 高赞评论/正文摘要 | 研究上应如何解读 |
|---|---|---|---|
| 怀疑与玩笑：[r/wallstreetbets：Compute futures](https://www.reddit.com/r/wallstreetbets/comments/1vql2qj/compute_futures/) | 2026-08-17；**169 分，64 评论** | 原帖问摩尔定律是否意味着持续熊市；高分评论“早已计价”得 101 分，关于操纵油市的讽刺评论 60 分 | 对可交易新题目有注意力，但笑话/怀疑不能证明合约已上线或被操纵 |
| 对数据的兴趣：[r/singularity：Silicon Data launches daily GPU rental index](https://www.reddit.com/r/singularity/comments/1kwzbkn/silicon_data_launches_daily_gpu_rental_index/) | 2025-05-27；**71 分，5 评论** | 11 分评论愿跟踪指数，并猜测价格下降与供给、Blackwell 迁移有关 | 指数有观察需求；具体因果是评论者猜测，不作为供需事实 |
| “新金融积木”的兴奋：[r/PredictionsMarkets：The role options could play in Kalshi's compute stack](https://www.reddit.com/r/PredictionsMarkets/comments/1v5wn6z/the_role_options_could_play_in_kalshis_compute/) | 2026-07-25；**4 分，6 评论** | 正文设想二元阶梯→永续→期权；4 分评论表示喜欢期权，3 分评论称可自然扩展 | 展示产品想象力；不能把用户设想写成 Kalshi 已推出 GPU 永续/期权 |
| 实务问题：[r/investimentos：Novo produto da CME - Compute Futures](https://www.reddit.com/r/investimentos/comments/1x2h4xh/novo_produto_da_cme_compute_futures/) | 2026-10-10；**6 分，4 评论** | 2 分评论仍困惑是否真创新；1 分长评论强调指数可信度、缺少实物套利约束、初期宽价差和杠杆风险 | 讨论质量不能只按赞数排；长评论提出有效核查问题，但没有资格证明 CME 已开市 |
| 硬件与租价混淆：[r/accelerate：The First Major-Exchange Compute Futures](https://www.reddit.com/r/accelerate/comments/1thmv9b/the_first_majorexchange_compute_futures_dr_alex/) | 2026-05-19；**8 分，7 评论** | 5 分评论问硬件不是不断贬值吗，是否只适合短期库存交易 | 表明市场教育缺口：GPU 资产残值、同型号租价、期货回报需分开 |
| 矿工收入视角：[r/BitcoinMiningPros：Do you watch hashprice or mostly just BTC price?](https://www.reddit.com/r/BitcoinMiningPros/comments/1wfzbfp/do_you_watch_hashprice_or_mostly_just_btc_price/) | 2026-09-14；**3 分，2 评论** | 2 分评论更重视 hashprice，因其贴近当天能否覆盖电费和矿池费 | 支持关注经营收入而非币价的研究方向；hashprice 本身是毛收入指标，不能直接等于净利润 |
| 融资新闻转帖：[r/RuntimeWire：Ornn's \$33M seed…](https://www.reddit.com/r/RuntimeWire/comments/1uosn4d/ornns_33m_seed_puts_two_20something_founders_in/) | 2026-07-06；**1 分，0 评论** | 无高赞评论可摘要 | 只能说明新闻被转发，不能说形成广泛社区共识 |

### 7.3 三条可提炼结论与四个限制

【推断】第一，注意力已经出现，但“价格是否持续下跌、到底交付什么、指数能不能信”仍是基础困惑。第二，一部分讨论者擅长把风险重组为期权、永续等金融积木，尚不能据此判断实际需求、可行性和风控。第三，hashprice 的实务话语较贴近收入与成本，对算力价格基准研究的启发是关注单位有效算力的经营净收入表现，而不是简单照搬某个代币价格；这与《加密算力对照实验与事件合约结算机制》第 2 节的分析相互印证。

限制必须写在结论旁边：

1. 搜索按相关性排序且每组限额，不能估计整体讨论热度、市场份额或正负面占比；未命中不等于没讨论。
2. 得分受平台抽样、投票和抓取时点影响，不能当人数；原报告只取至多三条顶层高分评论，未穷尽回复链。
3. 宣传、自动新闻、营销和真实用户混合；身份未验证，因此“实践者意见”只指评论内容贴近实务，不指确认了职业。
4. 英文和部分其他语言社区不能代表非美区域本土供应商、企业采购部门和信贷机构；没有从 Reddit 得出“产业已经普遍套保”。

---

## 8. 客户价值与商业结果：当前证据缺口

| 评价目标 | 本台账已有材料 | 尚缺的验证 |
|---|---|---|
| 数据质量 | 公开方法、更新频率和部分修订记录 | 独立账单、有效样本、代表性及样本外误差 |
| 采用 | 具体引用、产品列示和个别交易公告 | 持续使用、独立客户数、实际套保量及退出记录 |
| 客户净收益 | 尚无可独立重建的完整对照试验 | 相对现行 RFQ/固定价/容量与信用条款的账单、风险和争议成本变化，扣除费用与资金占用 |
| 机构收入 | 融资公告、授权用途和部分公开费率 | 实际付费收入、维护和责任成本；融资额不算经营收入 |
| 超额租金 | 可提出切换成本与排他权问题 | 可替代性、全成本、正常风险回报和议价结果；收入不自动等于租金 |

这些缺口既不证明基准没有价值，也不支持先建指数、再寻找金融用途。独立验证、影子套保和停止条件的完整流程见[05 实践指南第零节](算力与_Token_价格基准构建与撮合演进指南：PRA_方法论与市场设计实践.md)；理论解释见信息经济学篇，法律判断另依实际业务与适用法域作出。

---

## 9. 待核清单：哪些问题不能在结论中说满

| 优先级 | 待核事项 | 本次掌握到哪一步 | 补到什么证据才算解决 |
|---|---|---|---|
| P0 | Ornn 是否有另一个分钟级商业 feed | 11-17 重读方法论/FAQ/API：小时窗口+分层订阅（免费日度、Full 小时），无分钟级字段；分钟级仍未证实（§1.8） | 有日期的方法版本、字段和历史样本、发布日志/SLA；区分新算值与重复发送 |
| P0 | ICE、CME 截至 11-17 的批准/生效、挂牌和实际交易状态 | 11-17 取得 NYMEX 11-04 再延期函（至 2027-01-08）、26-524 做市计划已撤回记录、ICE 产品页（无挂牌/成交显示）；批准与成交仍无证据（§1.8） | 后续监管/交易所文件、有效规格及实际成交分别记录，不能用旧等待状态填补 |
| P0 | AX H100 的实际交易规模和首个交易日 | 11-17 生产 instruments API 确认 dated futures 在册（含 SEP-2026 is_closing）；行情端点需鉴权，无公开成交（§1.8） | GPU 合约代码、首笔记录、逐日量/持仓、买卖深度和真实客户结构 |
| P0 | Ornn/Polymarket 大宗的可独立验证细节 | 11-17 重取公告全文：FalconX×AneraLabs、六位数、Polygon 链上结算、嵌入推理远期对冲；哈希与精确条款仍未取得（§1.8） | 交易哈希、条款、赔付结构、净风险和持续后续成交；是否有商用授权费 |
| P1 | 两家输入中真正新增成交、存续租赁、报价的比例 | 方法/产品自述，非独立样本审计 | 数据字典、贡献者集中度、审计抽样及单窗口有效独立样本统计 |
| P1 | Ornn 的独立监督委员会是否已正式组建 | 11-17 方法论仍为 “is establishing”，未见成员公告（§1.8） | 成员、独立性、会议/审查记录及生效日，不把计划当落地 |
| P1 | Silicon Data 的 Polymarket 关系是否为商用合作 | 二月规则直接引用已核实；11-17 检索仍无商用签约公告 | 授权合同/双方公告；不把公开引用补写成签约 |
| P1 | Chainlink/Pyth/Hyperliquid/DePIN 的直接接入 | 11-17：Pyth 1951 条 feed 无 GPU 租赁价；Hyperliquid 三个 H100 市场均 delisted、零 OI；Chainlink 目录访问失败（§1.8） | 双方正式公告＋可查询 feed ID/合约代码/网络及实际更新记录 |
| P1 | Kalshi 算力市场完整历史、当前周度是否有新系列 | 10-17 采集 H100 周/月及 B200/H200/A100 月均系列；11-17 API 与新闻站均连接失败，未更新 | 全部相关 series、归档/迁移市场、逐日成交及交易额，避免混淆数量与金额 |
| P1 | Silicon Data 归一化参数、历史修订及独立验证 | 公告可见重述/权重调整；Syntax 9-08 受任独立计算代理（11-17 核实原文）；全流程未复建 | 当前方法、旧新版本映射、可审阅输入和审计，不把回填历史当实时发布历史 |
| P2 | ICE 数据分发是否有独家性；投资最终条款 | 全球分发已证实，独家未证实；16 亿直接投资已确认；11-17 无新披露 | 合同/法定披露，不用最初最高 20 亿倒推实付及控制权 |
| P2 | 区域算力数据的海外授权、跨境流动与衍生品合规路径 | 本报告未开展逐法域专项法律核查 | 针对数据内容、承载主体、客户与用途的属地法律审查；并非注册离岸实体即可自动解决 |

### 检索可用性与失败记录

原报告记载部分前端返回 JavaScript 外壳、改用官方 API，以及 PRAW 首轮中断后重试的过程；事实复核没有对应原始运行档案，保留为历史过程陈述而非重新认证。2026-11-15 复核中，CFTC 延期函和 ICE PDF 可读取，CME 通知/产品页请求超时，IOSCO PDF 未能重新取得全文。访问失败不证明产品不存在、已批准或仍未交易。

未解决的证据缺口均已完整列入上表，**不将公开网络检索可达性等同于每项非公开商业问题均已获证实**。

### 2026-11-17 复核回填与失败记录

本轮增量结果集中于 §1.8。新增/重取来源（读取日期均为 2026-11-17）：

- 【监管备案】[NYMEX 2026-11-04 自愿再延期函（26-370）](https://www.cftc.gov/filings/documents/2026/orgdcmnymexvolexten261105.pdf)：延期条件与日期见 §1.8。
- 【监管备案】[26-524 激励计划提交函](https://www.cftc.gov/filings/orgrules/rules10262636759.pdf)是拟议办法；[CFTC 备案状态列表](https://www.cftc.gov/IndustryOversight/IndustryFilings/TradingOrganizationRules?col=Status&dir=DESC)记录 11-04 撤回。
- 【交易所】[ICE HPR 产品页](https://www.ice.com/products/83050006/Ornn-Compute-Price-Index-OCPI-H100-Future)、[BKL 产品页](https://www.ice.com/products/83050009/Ornn-Compute-Price-Index-OCPI-B200-Future)与 9-29 通知 [S09] 分别说明产品规格和当时的挂牌限制。
- 【一手·公司文档】Ornn 方法论 https://data.ornn.com/methodology （changelog 2026-07-24）、FAQ https://data.ornn.com/faq 、API 文档 https://index.ornn.com/docs （base URL https://api.ornnai.com ）。
- 【一手·交易所 API/文档】AX instruments https://gateway.architect.exchange/api/instruments ；dated futures 说明 https://docs.architect.exchange/guides/funding-settlement/dated-futures.md ；changelog https://docs.architect.exchange/changelog.md （最新条目 2026-11-12，无 compute 上线记录）。
- 【一手·公司公告】Polymarket 六月大宗稿全文：https://www.prnewswire.com/news-releases/polymarket-closes-first-institutional-block-trade-on-a-gpu-instrument-302787854.html 。
- 【一手·协议 API】Pyth Hermes feed 目录 https://hermes.pyth.network/v2/price_feeds （1951 条）；Hyperliquid info API https://api.hyperliquid.xyz/info （perpDexs 及 xyz/para/io metaAndAssetCtxs）。
- 【一手·公司公告】Syntax×Silicon Data 计算代理任命（2026-09-08）：https://www.syntaxdata.com/research/silicon-data-appoints-syntax-data-as-independent-index-calculation-agent-advancing-institutional-infrastructure-for-compute-risk-management 。
- 【一手·法定披露】MSCI 2025 Form 10-K：https://www.sec.gov/Archives/edgar/data/1408198/000140819826000011/msci-20251231.htm （43.1% 原文核对）。
- 【一手·标准原文】IOSCO FR07/13 PDF：https://www.iosco.org/library/pubdocs/pdf/IOSCOPD415.pdf （48 页，19 项原则标题逐项核对）。
- 【一手·平台条款】Compute Exchange 费用页：https://compute.exchange/fees （仍标 2025-01-30）。
- 【二手·新闻】FT 2026-11-10《US regulator postpones Wall Street push to trade AI computing futures》：https://www.ft.com/content/269b0245-44c4-4e34-94df-202bef642539 （付费墙，仅取检索摘要，与 11-04 函一致）；Reuters 2026-11-12 Kalshi 估值 650 亿美元融资谈判：https://www.reuters.com/business/finance/kalshi-talks-raise-money-65-billion-valuation-bloomberg-news-reports-2026-11-12/ ；Reuters 2026-11-11 Polymarket 裁员及终止外包：https://www.reuters.com/business/polymarket-cuts-jobs-contracts-prediction-market-competition-heats-up-2026-11-11/ （背景信息，未据此改产品状态）。
- 【二手·研究汇编】CCIR 评论函汇编（含 26-370 状态与 11-04 延期）：https://ccir.io/research/compute-derivatives-cftc （用于交叉印证，不作为一手替代）。

本轮访问失败记录（失败不证明产品不存在、已批准或仍未交易）：

- Kalshi：交易 API 与新闻站在 11-17 未连通，series/events 未能重取。10 月盘口不因此变成 11 月盘口。
- Chainlink：`reference-data-directory.vercel.app/feeds-mainnet.json` 连接失败，官方 feed 目录未穷举。
- ICE：搜索线索中的 `ICE_Futures_US_Exchange_Notice_Ornn_Compute_Futures_20261009.pdf` 返回 404；通知列表页 JS 渲染，无法穷举 10 月后通知。
- AX：`/tickers`、`/trades`、`/candles` 返回 `no auth presented`；无匿名成交流水。
- AX：`architect.co/perpetuals/` 返回 404；永续产品页线索失效，与生产 API 无 GPU 永续的观察一致，但不单独证明历史从未存在。
- Ornn：旧 URL `ornn.com/insights/ornn-x-architect` 已 404；未另找到 AX 合作更新公告。
- `computeexchange.com`（无连字符域名）返回停放页跳转，与 `compute.exchange` 非同一站点，未采用。
- IOSCO：首次未取得正文，同日重取获得 PDF；MSCI 的期间及分母来自 SEC EDGAR 的 10-K。

---

## 10. 参考资料（完整 URL、日期与等级）

以下保留原报告的 2026-10-17 读取日期与 URL；本轮重新取得哪些材料见第 1.7 节，不把整张来源表标为 11 月全面复核。公司材料是其主张的一手证据，不自动构成独立审计。理论应用及示意数字均为本报告推演，不属于来源对现实市场的统计。

- **S01【一手】2026-07-24（页面首次发布记录）**：[ Ornn OCPI 方法论 ](https://data.ornn.com/methodology)。
- **S02【一手】未标发布日期；2026-10-17读取**：[ Ornn 公开 API 文档 ](https://index.ornn.com/docs)。
- **S03【一手】未标发布日期；2026-10-17读取**：[ Ornn 官网 ](https://ornn.com/)。
- **S04【一手】2026-06-24**：[ a16z crypto：Investing in Ornn ](https://www.a16zcrypto.com/posts/article/investing-in-ornn)。
- **S05【二手】2026-06-24**：[ Economic Times：Ornn 融资及投资人 ](https://economictimes.indiatimes.com/tech/funding/a16z-crypto-leads-33-million-bet-on-compute-marketplace-ornn/articleshow/131971804.cms)。
- **S06【一手·公司发布于新闻通讯社】2026-06-02**：[ Polymarket：首笔机构 GPU 大宗交易 ](https://www.prnewswire.com/news-releases/polymarket-closes-first-institutional-block-trade-on-a-gpu-instrument-302787854.html)。
- **S07【一手】页面未明确日期；2026-10-17读取**：[ FalconX：首笔 OTC 算力远期交易 ](https://www.falconx.io/newsroom/falconx-executes-first-otc-compute-forward-trade)。
- **S08【一手】2026-05-19**：[ ICE 与 Ornn 宣布算力期货 ](https://ir.theice.com/press/news-details/2026/ICE-and-Ornn-to-Launch-GPU-Compute-Futures-Contracts/default.aspx)。
- **S09【一手·交易所通知】2026-09-29**：[ ICE：OCPI H100/B200 合约参考数据通知 ](https://www.ice.com/publicdocs/futures_us/exchange_notices/ICE_Futures_US_Exchange_Notice_ORNN_Compute_4Q2026_20260929.pdf)。
- **S10【一手】2026-01-23**：[ Ornn × Architect 合作 ](https://ornn.com/insights/ornn-x-architect)。
- **S11【一手】未标发布日期；2026-10-17读取**：[ Architect AX 产品页 ](https://architect.co/ax/)。
- **S12【一手·公司发布于新闻通讯社】2026-05-28**：[ Architect 计划建立美国算力期货交易所 ](https://www.prnewswire.com/news-releases/architect-financial-technologies-to-launch-us-futures-exchange-for-compute-and-ai-commodities-following-dcm-acquisition-302784564.html)。
- **S13【一手】2025-10-07**：[ ICE 投资 Polymarket 及全球数据分发 ](https://ir.theice.com/press/news-details/2025/ICE-Announces-Strategic-Investment-in-Polymarket/default.aspx)。
- **S14【一手】2026-03-27**：[ ICE 追加 Polymarket 投资 ](https://ir.theice.com/press/news-details/2026/Intercontinental-Exchange-Announces-New-600-Million-Investment-in-Polymarket/default.aspx)。
- **S15【一手】2026-05-12**：[ CME 与 Silicon Data 宣布算力期货 ](https://investor.cmegroup.com/news-releases/news-release-details/cme-group-and-silicon-data-partner-launch-first-compute-futures)。
- **S16【一手】2026-08-11**：[ CME 原定十月五日推出 Compute Futures ](https://www.cmegroup.com/media-room/press-releases/2026/8/11/cme_group_and_silicondatatolaunchcomputefuturesonoctober5tounloc.html)。
- **S17【一手·交易所通知】2026-09-28**：[ CME Globex：Compute Futures Postponed ](https://www.cmegroup.com/notices/electronic-trading/2026/09/20260928.html)。
- **S18【一手·监管文件】2026-09-21**：[ CFTC 延长 NYMEX 算力合约审查 ](https://www.cftc.gov/filings/documents/2026/orgdcmnymexcompcontr260921.pdf)。
- **S19【一手·监管机构】2026-08-19**：[ CFTC 就算力衍生品征询意见 ](https://www.cftc.gov/PressRoom/PressReleases/9286-26)。
- **S20【一手】未标发布日期；2026-10-17读取**：[ CME Compute Futures 产品规格 ](https://www.cmegroup.com/markets/energy/power/compute-futures.html)。
- **S21【一手】未标发布日期；2026-10-17读取**：[ CME 产品周年历史 ](https://www.cmegroup.com/company/product-anniversaries.html)。
- **S22【一手】2022-09-19**：[ CME 推出事件合约 ](https://investor.cmegroup.com/news-releases/news-release-details/cme-group-announces-launch-event-contracts-trading-global)。
- **S23【一手】2025 年第四季度业绩材料，2026 年发布**：[ CME：FanDuel Predicts 实际上线及渠道扩展 ](https://investor.cmegroup.com/static-files/de615c20-4d59-44b4-9495-fbe97084aae4)。
- **S24【一手】2026-06-01**：[ CME 已开始加密期货期权 24/7 交易 ](https://investor.cmegroup.com/news-releases/news-release-details/cme-group-announces-launch-247-cryptocurrency-futures-and)。
- **S25【一手】未标发布日期；2026-10-17读取**：[ ICE 公司历史与业务 ](https://www.ice.com/about)。
- **S26【一手】2025-05-20（公告日）**：[ Silicon Data 470 万美元种子融资 ](https://www.silicondata.com/news-room/silicon-data-secures-4-7m-in-seed-funding-and-expands-compute-industry-data-product-suite)。
- **S27【一手】2026-08-11**：[ Silicon Data 3050 万美元 A 轮首关 ](https://www.silicondata.com/news-room/silicon-data-raises-30-5-million-series-a)。
- **S28【一手】未标发布日期；2026-10-17读取**：[ Silicon Data H100 指数产品页 ](https://www.silicondata.com/products/silicon-index/h100)。
- **S29【一手】多次公告，含 2025-12 至 2026-09**：[ Silicon Data 指数变更公告 ](https://docs.silicondata.com/products/gpu-index-announcements)。
- **S30【一手】2025-10-08（网页标注）**：[ Silicon Data × dxFeed × Refinitiv 分发 ](https://www.silicondata.com/news-room/silicon-data-partners-with-dxfeed-to-expand-gpu-index-coverage-on-refinitiv-driving-vital-transparency-in-ai-infrastructure-markets)。
- **S31【一手】2026-04-20**：[ Silicon Data GPU 远期曲线推出 ](https://www.silicondata.com/news-room/silicon-data-unveils-first-gpu-forward-curve)。
- **S32【一手·公司观点】2026-05-01**：[ Silicon Data：远期曲线推导说明 ](https://www.silicondata.com/blog/the-gpu-forward-curve-pricing-the-curve-not-just-the-spot)。
- **S33【一手】未标发布日期；2026-10-17读取**：[ DRW 业务及 Cumberland 历史 ](https://www.drw.com/access-our-liquidity/our-company)。
- **S34【一手·市场规则及 API】2026-02 起的历史市场；10-17抓取**：[ Polymarket 二月 H100 市场（Silicon Data 结算） ](https://polymarket.com/event/what-will-gpu-rental-prices-h100-hit-in-february)。
- **S35【一手·市场规则及 API】2026-10-17 05:44:50 UTC 抓取**：[ Polymarket 十月底 H100 市场（Ornn 结算） ](https://polymarket.com/event/gpu-rental-prices-h100-end-of-october)。
- **S36【一手·API】2026-10-17抓取**：[ Kalshi H100 周度 series 及来源 ](https://api.elections.kalshi.com/trade-api/v2/series/KXH100WS)。
- **S37【一手·API】2026-10-17抓取**：[ Kalshi H100 周度 events/markets ](https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXH100WS&limit=100&with_nested_markets=true)。
- **S38【一手·API】2026-10-17抓取**：[ Kalshi H100 月均 events/markets ](https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXH100MS&limit=100&with_nested_markets=true)。
- **S39【一手】2026-07-14**：[ Kalshi 发布算力远期曲线 ](https://news.kalshi.com/p/compute-forward-curves)。
- **S40【一手·协议文档】未标发布日期；2026-10-17读取**：[ Polymarket 结算与 UMA ](https://docs.polymarket.com/concepts/resolution)。
- **S41【一手·协议文档】未标发布日期；2026-10-17读取**：[ Hyperliquid Oracle ](https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/oracle)。
- **S42【一手·协议文档】未标发布日期；2026-10-17读取**：[ Hyperliquid Hyperps ](https://hyperliquid.gitbook.io/hyperliquid-docs/trading/hyperps)。
- **S43【一手·标准制定机构】2013-07**：[ IOSCO 金融基准原则 ](https://www.iosco.org/library/pubdocs/pdf/IOSCOPD415.pdf)。
- **S44【一手·公司法定披露】2025 年报，2026 年提交**：[ MSCI 2025 Form 10-K ](https://www.sec.gov/Archives/edgar/data/1408198/000140819826000011/msci-20251231.htm)。
- **S45【一手】未标发布日期；2026-10-17读取**：[ LME 参考价制度 ](https://www.lme.com/en/Market-Data/LME-reference-prices)。
- **S46【一手】未标发布日期；2026-10-17读取**：[ LBMA Gold Price 制度 ](https://www.lbma.org.uk/prices-and-data/lbma-gold-price)。
- **S47【一手·原论文】1945-09**：[ Hayek：The Use of Knowledge in Society ](https://www.mercatus.org/sites/default/files/d7/the_use_of_knowledge_in_society_-_hayek.pdf)。
- **S48【一手·原论文】1980-06**：[ Grossman & Stiglitz：On the Impossibility of Informationally Efficient Markets ](https://terpconnect.umd.edu/~wermers/ftpsite/FAME/grossman_stiglitz_%281980%29.pdf)。
- **S49【一手·诺奖机构】2001-10-10**：[ 2001 经济学奖：非对称信息与信号 ](https://www.nobelprize.org/prizes/economic-sciences/2001/press-release/)。
- **S50【一手·公司新闻稿转载】2026-06-10**：[ ProCap Financial × Ornn 数据合作 ](https://markets.financialcontent.com/worldnow/article/bizwire-2026-6-10-procap-financial-partners-with-ornn-to-bring-ai-compute-cost-data-to-independent-investors)。
- **M01【一手·原著】1867；英文公开版**：[ 资本论第一卷第一章：商品 ](https://www.marxists.org/archive/marx/works/1867-c1/ch01.htm)。
- **M02【一手·原著】1867；英文公开版**：[ 资本论第一卷第三章：货币 ](https://www.marxists.org/archive/marx/works/1867-c1/ch03.htm)。
- **M03【一手·原著】1885；英文公开版**：[ 资本论第二卷第五章：流通时间 ](https://www.marxists.org/archive/marx/works/1885-c2/ch05.htm)。
- **M04【一手·原著】1885；英文公开版**：[ 资本论第二卷第六章：流通费用 ](https://www.marxists.org/archive/marx/works/1885-c2/ch06.htm)。
- **M05【一手·原著】1894；英文公开版**：[ 资本论第三卷第十六章：商品经营资本 ](https://www.marxists.org/archive/marx/works/1894-c3/ch16.htm)。
- **M06【一手·原著】1894；英文公开版**：[ 资本论第三卷第十九章：货币经营资本 ](https://www.marxists.org/archive/marx/works/1894-c3/ch19.htm)。
- **M07【一手·原著】1894；英文公开版**：[ 资本论第三卷第二十一章：生息资本 ](https://www.marxists.org/archive/marx/works/1894-c3/ch21.htm)。
- **M08【一手·原著】1894；英文公开版**：[ 资本论第三卷第二十九章：银行资本的组成部分 ](https://www.marxists.org/archive/marx/works/1894-c3/ch29.htm)。
- **M09【一手·原著】1894；英文公开版**：[ 资本论第三卷第三十章：货币资本与现实资本Ⅰ ](https://www.marxists.org/archive/marx/works/1894-c3/ch30.htm)。
- **M10【一手·原著】1894；英文公开版**：[ 资本论第三卷第三十七章：地租导论 ](https://www.marxists.org/archive/marx/works/1894-c3/ch37.htm)。
- **M11【一手·原著】1910；英文公开版**：[ 希法亭：金融资本 ](https://www.marxists.org/archive/hilferding/1910/finkap/)。
- **M12【一手·原著】1867；英文版第25章对应通行中文版第23章**：[ 资本论第一卷：资本主义积累的一般规律 ](https://www.marxists.org/archive/marx/works/1867-c1/ch25.htm)。
- **M13【一手·原著】1894；英文公开版**：[ 资本论第三卷第二十五章：信用与虚拟资本 ](https://www.marxists.org/archive/marx/works/1894-c3/ch25.htm)。
- **S51【一手·本人讲座】1991-12-09**：[ Coase：The Institutional Structure of Production ](https://www.nobelprize.org/prizes/economic-sciences/1991/coase/lecture/)。
- **S52【一手·本人讲座】1993-12-09**：[ North 诺贝尔讲座 ](https://www.nobelprize.org/prizes/economic-sciences/1993/north/lecture/)。
- **A01【一手·API】2026-10-17 05:44:50 UTC**：[ Polymarket 二月 H100 原始 API ](https://gamma-api.polymarket.com/events?slug=what-will-gpu-rental-prices-h100-hit-in-february)。
- **A02【一手·API】2026-10-17 05:44:50 UTC**：[ Polymarket 十月底 H100 原始 API ](https://gamma-api.polymarket.com/events?slug=gpu-rental-prices-h100-end-of-october)。
- **A03【一手·API】2026-10-17抓取**：[ Kalshi H100 月度 series 元数据 ](https://api.elections.kalshi.com/trade-api/v2/series/KXH100MS)。
- **A04【一手·合约条款链接】未标发布日期；2026-10-17读取 series 链接**：[ Kalshi INDEXVALUE 通用条款入口 ](https://assets.kalshi.com/contract_terms/INDEXVALUE.pdf)。
- **A05【一手·API】2026-10-17 06:14:28 UTC**：[ Polymarket 年末 B200 事件 ](https://gamma-api.polymarket.com/events?slug=gpu-rental-prices-b200-end-of-2026-20260709165336437)。
- **A06【一手·API】2026-10-17 06:14:28 UTC**：[ Kalshi B200 月均事件 ](https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXB200MS&limit=100&with_nested_markets=true)。
- **A07【一手·API】2026-10-17 06:14:28 UTC**：[ Kalshi H200 月均事件 ](https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXH200MS&limit=100&with_nested_markets=true)。
- **A08【一手·API】2026-10-17 06:14:28 UTC**：[ Kalshi A100 月均事件 ](https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXA100MS&limit=100&with_nested_markets=true)。
- **S53【一手·公司发布于新闻通讯社】2025-10-28**：[ Ornn 570 万美元早期种子融资 ](https://www.prnewswire.com/news-releases/ornn-raises-5-7-million-seed-round-to-launch-the-worlds-first-compute-futures-exchange-302596938.html)。
- **S54【一手】未标发布日期；2026-10-17读取**：[ Ornn 团队与投资人页 ](https://ornn.com/about)。

**Reddit 代表性讨论原始链接**（一手讨论记录；内容不等同经核商业事实）：

- 2026-08-17：[ Compute futures ](https://www.reddit.com/r/wallstreetbets/comments/1vql2qj/compute_futures/)。
- 2025-05-27：[ Silicon Data launches daily GPU rental index: Bloomberg ](https://www.reddit.com/r/singularity/comments/1kwzbkn/silicon_data_launches_daily_gpu_rental_index/)。
- 2026-07-25：[ The role options could play in Kalshi's compute stack ](https://www.reddit.com/r/PredictionsMarkets/comments/1v5wn6z/the_role_options_could_play_in_kalshis_compute/)。
- 2026-10-10：[ Novo produto da CME - Compute Futures ](https://www.reddit.com/r/investimentos/comments/1x2h4xh/novo_produto_da_cme_compute_futures/)。
- 2026-05-19：[ The First Major-Exchange Compute Futures - Dr. Alex Wissner-Gross ](https://www.reddit.com/r/accelerate/comments/1thmv9b/the_first_majorexchange_compute_futures_dr_alex/)。
- 2026-09-14：[ Do you watch hashprice or mostly just BTC price? ](https://www.reddit.com/r/BitcoinMiningPros/comments/1wfzbfp/do_you_watch_hashprice_or_mostly_just_btc_price/)。
- 2026-07-06：[ Ornn's \$33M seed puts two 20-something founders in the middle of Wall Street's compute trade ](https://www.reddit.com/r/RuntimeWire/comments/1uosn4d/ornns_33m_seed_puts_two_20something_founders_in/)。

[S01]: https://data.ornn.com/methodology
[S02]: https://index.ornn.com/docs
[S03]: https://ornn.com/
[S04]: https://www.a16zcrypto.com/posts/article/investing-in-ornn
[S05]: https://economictimes.indiatimes.com/tech/funding/a16z-crypto-leads-33-million-bet-on-compute-marketplace-ornn/articleshow/131971804.cms
[S06]: https://www.prnewswire.com/news-releases/polymarket-closes-first-institutional-block-trade-on-a-gpu-instrument-302787854.html
[S07]: https://www.falconx.io/newsroom/falconx-executes-first-otc-compute-forward-trade
[S08]: https://ir.theice.com/press/news-details/2026/ICE-and-Ornn-to-Launch-GPU-Compute-Futures-Contracts/default.aspx
[S09]: https://www.ice.com/publicdocs/futures_us/exchange_notices/ICE_Futures_US_Exchange_Notice_ORNN_Compute_4Q2026_20260929.pdf
[S10]: https://ornn.com/insights/ornn-x-architect
[S11]: https://architect.co/ax/
[S12]: https://www.prnewswire.com/news-releases/architect-financial-technologies-to-launch-us-futures-exchange-for-compute-and-ai-commodities-following-dcm-acquisition-302784564.html
[S13]: https://ir.theice.com/press/news-details/2025/ICE-Announces-Strategic-Investment-in-Polymarket/default.aspx
[S14]: https://ir.theice.com/press/news-details/2026/Intercontinental-Exchange-Announces-New-600-Million-Investment-in-Polymarket/default.aspx
[S15]: https://investor.cmegroup.com/news-releases/news-release-details/cme-group-and-silicon-data-partner-launch-first-compute-futures
[S16]: https://www.cmegroup.com/media-room/press-releases/2026/8/11/cme_group_and_silicondatatolaunchcomputefuturesonoctober5tounloc.html
[S17]: https://www.cmegroup.com/notices/electronic-trading/2026/09/20260928.html
[S18]: https://www.cftc.gov/filings/documents/2026/orgdcmnymexcompcontr260921.pdf
[S19]: https://www.cftc.gov/PressRoom/PressReleases/9286-26
[S20]: https://www.cmegroup.com/markets/energy/power/compute-futures.html
[S21]: https://www.cmegroup.com/company/product-anniversaries.html
[S22]: https://investor.cmegroup.com/news-releases/news-release-details/cme-group-announces-launch-event-contracts-trading-global
[S23]: https://investor.cmegroup.com/static-files/de615c20-4d59-44b4-9495-fbe97084aae4
[S24]: https://investor.cmegroup.com/news-releases/news-release-details/cme-group-announces-launch-247-cryptocurrency-futures-and
[S25]: https://www.ice.com/about
[S26]: https://www.silicondata.com/news-room/silicon-data-secures-4-7m-in-seed-funding-and-expands-compute-industry-data-product-suite
[S27]: https://www.silicondata.com/news-room/silicon-data-raises-30-5-million-series-a
[S28]: https://www.silicondata.com/products/silicon-index/h100
[S29]: https://docs.silicondata.com/products/gpu-index-announcements
[S30]: https://www.silicondata.com/news-room/silicon-data-partners-with-dxfeed-to-expand-gpu-index-coverage-on-refinitiv-driving-vital-transparency-in-ai-infrastructure-markets
[S31]: https://www.silicondata.com/news-room/silicon-data-unveils-first-gpu-forward-curve
[S32]: https://www.silicondata.com/blog/the-gpu-forward-curve-pricing-the-curve-not-just-the-spot
[S33]: https://www.drw.com/access-our-liquidity/our-company
[S34]: https://polymarket.com/event/what-will-gpu-rental-prices-h100-hit-in-february
[S35]: https://polymarket.com/event/gpu-rental-prices-h100-end-of-october
[S36]: https://api.elections.kalshi.com/trade-api/v2/series/KXH100WS
[S37]: https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXH100WS&limit=100&with_nested_markets=true
[S38]: https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXH100MS&limit=100&with_nested_markets=true
[S39]: https://news.kalshi.com/p/compute-forward-curves
[S40]: https://docs.polymarket.com/concepts/resolution
[S41]: https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/oracle
[S42]: https://hyperliquid.gitbook.io/hyperliquid-docs/trading/hyperps
[S43]: https://www.iosco.org/library/pubdocs/pdf/IOSCOPD415.pdf
[S44]: https://www.sec.gov/Archives/edgar/data/1408198/000140819826000011/msci-20251231.htm
[S45]: https://www.lme.com/en/Market-Data/LME-reference-prices
[S46]: https://www.lbma.org.uk/prices-and-data/lbma-gold-price
[S47]: https://www.mercatus.org/sites/default/files/d7/the_use_of_knowledge_in_society_-_hayek.pdf
[S48]: https://terpconnect.umd.edu/~wermers/ftpsite/FAME/grossman_stiglitz_%281980%29.pdf
[S49]: https://www.nobelprize.org/prizes/economic-sciences/2001/press-release/
[S50]: https://markets.financialcontent.com/worldnow/article/bizwire-2026-6-10-procap-financial-partners-with-ornn-to-bring-ai-compute-cost-data-to-independent-investors
[M01]: https://www.marxists.org/archive/marx/works/1867-c1/ch01.htm
[M02]: https://www.marxists.org/archive/marx/works/1867-c1/ch03.htm
[M03]: https://www.marxists.org/archive/marx/works/1885-c2/ch05.htm
[M04]: https://www.marxists.org/archive/marx/works/1885-c2/ch06.htm
[M05]: https://www.marxists.org/archive/marx/works/1894-c3/ch16.htm
[M06]: https://www.marxists.org/archive/marx/works/1894-c3/ch19.htm
[M07]: https://www.marxists.org/archive/marx/works/1894-c3/ch21.htm
[M08]: https://www.marxists.org/archive/marx/works/1894-c3/ch29.htm
[M09]: https://www.marxists.org/archive/marx/works/1894-c3/ch30.htm
[M10]: https://www.marxists.org/archive/marx/works/1894-c3/ch37.htm
[M11]: https://www.marxists.org/archive/hilferding/1910/finkap/
[M12]: https://www.marxists.org/archive/marx/works/1867-c1/ch25.htm
[M13]: https://www.marxists.org/archive/marx/works/1894-c3/ch25.htm
[S51]: https://www.nobelprize.org/prizes/economic-sciences/1991/coase/lecture/
[S52]: https://www.nobelprize.org/prizes/economic-sciences/1993/north/lecture/
[A01]: https://gamma-api.polymarket.com/events?slug=what-will-gpu-rental-prices-h100-hit-in-february
[A02]: https://gamma-api.polymarket.com/events?slug=gpu-rental-prices-h100-end-of-october
[A03]: https://api.elections.kalshi.com/trade-api/v2/series/KXH100MS
[A04]: https://assets.kalshi.com/contract_terms/INDEXVALUE.pdf
[A05]: https://gamma-api.polymarket.com/events?slug=gpu-rental-prices-b200-end-of-2026-20260709165336437
[A06]: https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXB200MS&limit=100&with_nested_markets=true
[A07]: https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXH200MS&limit=100&with_nested_markets=true
[A08]: https://api.elections.kalshi.com/trade-api/v2/events?series_ticker=KXA100MS&limit=100&with_nested_markets=true
[S53]: https://www.prnewswire.com/news-releases/ornn-raises-5-7-million-seed-round-to-launch-the-worlds-first-compute-futures-exchange-302596938.html
[S54]: https://ornn.com/about
