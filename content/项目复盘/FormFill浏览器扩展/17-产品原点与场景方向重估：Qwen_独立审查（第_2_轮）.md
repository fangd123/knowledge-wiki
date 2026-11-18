---
title: "产品原点与场景方向重估：Qwen 独立审查（第 2 轮）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: 项目复盘
series: FormFill浏览器扩展
series_order: 17
tags:
  - 项目复盘
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# 产品原点与场景方向重估：Qwen 独立审查（第 2 轮）

- 日期：2026-11-17T06:18:02.724Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 已核对 / 未核对

**已逐字核对**：`research/2026-11-17-product-origin/README.md` 全文、`public-form-structure.json` 全部 4 条、`docs/user-personas.md` 全文、CHANGELOG / product-overview / execution-plan / product-strategy 四处 diff、随附的两轮审查意见（第 1 轮 C4、第 2 轮 D1 处均被截断）与逐项处理记录。
**未独立访问，不作事实认定**：E01–E17 全部外部 URL（含条款号、5,000 条上限、OCR 行为、风控条款）、`probe-public-forms.cjs`、`entrypoints/content.ts` 与 `utils/mappings.ts` 源码、release-checklist / community-launches / 执行台账。以下判断只依据材料内部自洽性与"结论是否超出所给证据"。未做访谈本身不构成否决理由。

## 阻塞问题

### C1 控件数被当作字段数，报价与交付范围建立在错误计数上
README §3 抽样表与"两个 20"段、personas §1/§3：深交所 10 个 radio 通常只是 2–5 个逻辑字段，却据控件数断言"超报价口径 20 字段"，会误定 ¥699 交付边界。改：定义逻辑字段（radio/语义 checkbox 组计 1），表拆"控件数／估算逻辑字段／未验证组件"，结论降为"需复核"，报价口径引用同一定义。

### C2 自定义组件漏计 + 未验证类型计为可填，技术可达被高估
README §3 与 JSON：三页 `select` 均为 0、标新无 `file` 却称"有上传/搜索选择/多步/OCR 自动填"，JSON 无任何留证。"17/15 均在白名单内"高估可填性；13 个 radio/checkbox 自述真实浏览器未验证仍计入可填。改：补属性/组件/步骤快照，否则标"印象未留证"；覆盖率改为"原生控件在白名单，可填性未知"。

### C3 "合成复现"无合格定义，可被自建 mock 自证；且与禁真实账号冲突
README §3/§6 第 4 条/§7、personas §6、execution-plan M01：团队自造标准 DOM 假页即可宣称"目标类型已检验"，循环证据；§6"不用真实账号"与"授权演示"字面冲突；硬门槛写在"渠道"列而非验收列。改：限定用户自身账号只读演示或授权脱敏结构快照重建，自建 mock 只计功能验证；改为研究者不持账号；门槛移入验收列。

### C4 门槛①不可判定，¥699"复用"未定义
README §7、personas §3/§6：①"多名／下降／再次使用"无人数、幅度、次数，与②"两名实付且复用"严格度失衡；¥699 是一次性单网页适配，"复用"指再次付费还是同映射再命中未说明。改：①填可判定数值（人数、对照页数、耗时降幅、复用命中次数）；②明确"复用"口径。

### C5 替代清单漏掉与白名单高度重叠的自动带出路径
README §2.2/§4/§5/§7 问题 2、personas §1/§6：未列"输入信用代码→工商数据带出名称/法人/地址/注册资本"、平台一键带入历史资料、代办服务商，而白名单字段大半属此类；只问 OCR/主体库会误判空白。改：补入替代与访谈，按"已带出／非白名单人工／FormFill 可填"三分记字段数与耗时，价值口径改为剩余字段成本，并写入停止信号。

## 非阻塞建议

- **D1**：E11 标注苏州市平台，正文写"张家港"，核对归属，勿把市级问答当全省口径。
- **D2**：E05 第十一条、E17 第十九条条款号未独立核实；E17 引工信部新闻页，建议改引原文。
- **D3**：§5"20–99 人"源自招聘帖，虽声明不限定，仍建议删数字避免规模锚定。
- **D4**：映射绑定结构、平台改版即失效，年检间隔长；M01 应记复用命中率与失效原因。
- **D5**：画像一多无 API 能力，试用须 IT 在场；配置失败记门槛失败，不记需求证伪。
- **D6**：处理记录只覆盖第 1 轮，材料内第 2 轮 C1–C3 未见落地，建议补逐项处理。
- **D7**：§0"用户判断基本正确"强于 §4"排序是判断"，建议改为"部分成立、待验证"。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 3 轮）

- 日期：2026-11-17T06:27:08.636Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §3「合格复现」(b) + §7 门槛①**
脱敏结构快照重建页无法复现 Chosen/ExtJS 触发器、级联、异步校验与登录态，却被允许用于"技术可达"判定和手工 vs FormFill 耗时对照，循环证据残留。改：(b) 只计结构参考；可填性与耗时对照必须取自用户自身账号真实页面的只读演示/录屏。

**C2｜README §3 逻辑字段口径 与 §7 三分记录冲突**
§3 把 readonly 一律标"平台已带出"，§7 把自定义组件归"必须人工"。深大 2 个 ExtJS `x-trigger-noedit` 同属两者，CAType/CACode 属 CA 依赖。分类冲突会虚增替代覆盖、压低剩余字段，直接动摇"≥5 个/页"门槛。改：readonly 拆为平台带出／待人工自定义控件／CA 依赖三类。

**C3｜README §0 白名单、§5；personas §5；JSON 深大样本**
白名单含"开户信息"，抽样页含 `LPIDNumber`/`bidderIDNumber`（身份证号），却只笼统排除"高敏"，与 personas 排除金融高敏口径冲突；BYOK 下这些值直发第三方模型。改：显式排除身份证号/银行账号，或限本地模型+脱敏，M01 不采集真实值。

**C4｜README §7 门槛① 与 §5、personas §6 不一致**
画像一以跨平台首次填写为主、年检间隔长，首轮几乎不可能出现"同结构复用命中"，该硬门槛会假阴性误杀方向；personas §6 又规定未复用只记"未观察到"。改：复用命中允许画像二样本或设观察窗口，未观察不作否决项。

**C5｜README §2.2（E21）与 §7 停止信号**
猪八戒"求浏览器插件"需求帖只证明有人想要工具，不能证明"一键带入"、代办供给或价格，却被当作替代证据并支撑"代办更便宜省心"的停止信号，证据方向被反用。改：降为需求线索；代办替代须由访谈的实际使用与报价证明。

## 非阻塞建议

- **D1**：深交所 5 个 file（`lxfsqrh` 重名）、短信验证码未计逻辑字段，与标新"上传计1"不一致，¥699 边界仍模糊。
- **D2**：标新"OCR 已覆盖企业字段"仅来自页面文案/ocrMention，未实测；深大 ocrMention=true 表中未提。
- **D3**：E20 域名 jdy.gaojin.com.cn 非简道云官方，宜标"第三方模板站宣传"。
- **D4**：门槛①"≥3 名"与招募配额 3–4 名几乎重合，判定分母应与配额分开。
- **D5**：深交所注册需短信验证码+必传附件，属端到端流程阻断，建议在抽样表显式标注。

## 核对范围

**已逐字核对**：README 全文（含 §0–§8 与引用口径段）、public-form-structure.json 全部 4 条逐控件属性、docs/user-personas.md 全文、CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff、第 2 轮审查意见与逐项处理记录。

**未独立访问，不作事实认定**：E01–E21 全部外部链接（含条款序号、OCR 与工商带出行为、风控条款、5,000 条上限）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。上述判断仅基于材料内部自洽性与"结论是否超出所给证据"。未做用户访谈本身不构成否决理由；本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 4 轮）

- 日期：2026-11-17T06:29:54.274Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §3 抽样表（深大行）与 §0 白名单**
"约 13 候选可填"未扣除白名单已排除字段：JSON 中 `bankName`/`bankAccount`/`LPIDNumber`/`bidderIDNumber` 均在 13 内，按白名单应 ≤9，且银行字段未标注。虚增"剩余字段"，直接影响 §7"≥5 个/页"门槛与 ¥699 清点。改：表内分列"白名单内候选／已排除敏感字段"，重算并同步 §7 示例。

**C2｜README §3 首句、personas §6、execution-plan M01 验收列**
仍写"授权或合成复现""结构快照重建页"即可满足首批硬门槛，与 §3"(b) 只计结构参考、不计技术可达"及 §7 只认 (a) 冲突，等于允许重建页自证第一优先假设。改：三处统一为"硬门槛只认 (a) 真实页面只读演示；(b) 仅结构参考"。

**C3｜README §2.2 工商带出段（E18–E20）**
"比 OCR 更常见的替代"是频率断言，三条来源无法支撑：E18 自述为执照识别（属 OCR 类）、E19 仅"自动核验/自主维护"未说字段带出、E20 为第三方宣传。改：删"更常见"，逐条标注各来源实际证明的能力，覆盖率留 M01 实测。

**C4｜README §3 逻辑字段口径与深交所行**
同段既说"上传等自定义组件计 1"，又说"附件不计可填字段"；深交所"22–25"上限把附件组计入，按口径应约 21。¥699"≤20 普通字段"的分母（是否含自定义组件/附件/readonly）未定义，"可能超 20"不可判定。改：明确报价分母＝白名单内可填逻辑字段，附件/验证码只计步骤。

## 非阻塞建议

- **D1**：深交所 `dlh`（readonly）未按三分类归类，宜补①/②/③判定。
- **D2**：§7"FormFill 可填"应对 radio/checkbox 标"未证"，与 §3 一致。
- **D3**：§6 合成资料包仍列"开户信息"，与白名单删除项矛盾，建议删或标默认排除。
- **D4**：§5 行业清单（仪器仪表/新材料/工程等）超出 2 个招聘样本，宜标为推断。
- **D5**：E13/E14 为 2 例岗位样本，建议注明行业与平台名单不可外推。

## 核对范围

**已逐字核对**：README 全文（§0–§8 及引用口径段）、public-form-structure.json 全部 4 条逐控件属性（含 readonly/name/type 计数复算）、docs/user-personas.md 全文、CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff、第 2/3 轮审查意见与逐项处理记录。

**未独立访问，不作事实认定**：E01–E21 全部外部链接（条款序号、OCR 与工商带出行为、5,000 条上限、风控条款）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做访谈本身不构成否决，本轮不新增市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 5 轮）

- 日期：2026-11-17T06:38:31.532Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §3「合格复现」定义、§7 门槛①；personas §6；execution-plan M01 验收列**
(a) 被定义为"只读演示/录屏"，却又是可填性判定与"手工 vs FormFill 含复核耗时"对照的唯一来源——只读观察无法测量写入行为，M01 要么执行不了，要么产出假对照。改：把 (a) 拆为"只读结构演示"与"用户自身账号内、合成资料、写入但不提交"的受控实测，并写明草稿留痕、异步校验与平台规则风险及授权口径。

**C2｜README §2.2 反向证据段（E12）与 §3 标新行**
§2.2 断言"标新注册页已经提供营业执照 OCR 并自动填充企业信息"，§3 却标"仅页面文案+截图，未实测识别行为"。替代覆盖被当作既成事实，会经 §4 启动条件与 §7 停止信号误杀方向。改：§2.2 降为"页面声明存在 OCR 入口，字段覆盖率未实测"，与 E18 同口径。

**C3｜README §3 抽样表标新行**
"约 15 + 自定义组件若干"未按同段口径扣除搜索选择框/自定义组件（深交所行已扣），口径两行不一致；且 JSON 中该页 role/ariaHasPopup 全空、customComponentHints 仅 fa-upload，"含搜索选择框"无留证，stepHints=5 但只抽首步。改：可填逻辑字段记"未知"，撤回 15 的估值。

**C4｜README §3 抽样表深交所行与报价分母**
3 个 `zcxy`（注册协议勾选）被计 1 个可填逻辑字段，5 个 radio 组自标"未证"仍计入，得出 20–21 并据此讨论 ¥699 边界。协议勾选不属白名单"主体信息字段"，且属代为同意法律条款。改：分母剔除协议/同意类，分列"已验证可填 15／未证 5"，不下 20 边界结论。

## 非阻塞建议

- **D1**：JSON 中 `lxfsqrh` 仅 2 个（file 共 5、4 个不同名），§3"5 个同名 lxfsqrh"与自有证据不符。
- **D2**：Chosen 通常保留隐藏原生 select，"填不进"未实测，宜标待验证而非结论。
- **D3**：E10 链接为怀化市平台，正文称"湖南平台"，宜与 E11 同标市级口径。
- **D4**：personas §3"平台开店资料"常含身份证/银行字段，需按白名单显式限定。
- **D5**：personas ¥699 写"≤20 字段"，未引用 §3 逻辑字段口径，易被按控件数执行。
- **D6**：CHANGELOG 称"审查一轮"、处理记录标题"1+2 轮"，实际已 4 轮，宜更新。
- **D7**：深大 9 个候选称"逼近 ≥5 下限"偏强；全页 ExtJS，事件写入有效性同样未证。

## 核对范围

**已逐字核对**：README 全文（§0–§8 及引用口径段）、public-form-structure.json 全部 4 条逐控件属性（type/name/readonly 计数与分组复算）、docs/user-personas.md 全文、CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff、第 2–4 轮审查意见与逐项处理记录。

**未独立访问，不作事实认定**：E01–E21 全部外部链接（含条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘岗位职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做用户访谈本身不构成否决，本轮不新增市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 6 轮）

- 日期：2026-11-17T06:48:09.537Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §3「合格复现」a2 定义、§7 门槛①；personas §6**
a2 要求"用户自身账号内"写入，但第一优先任务含**首次注册/入库**，三个 200 抽样页全是匿名注册页、无账号态；门槛①又把 a2 设为唯一耗时对照来源，注册类任务要么执行不了，要么被临场变通写入。风险口径只提草稿留痕，未提合成主体触发工商核验/风控。改：区分"账号内更新/年检页"与"匿名注册页"，后者限平台测试环境或用户本人真实资料操作（研究者不接触值），否则记 a1+人工计时并标证据等级。

**C2｜README §3 a2、§7 门槛①、§6 第 2 条**
价值口径是"转录+复核成本下降"，但 a2 强制合成资料：用户明知是假数据，回源核对负担被人为消除，耗时对照会系统性偏向 FormFill，产生假阳性"继续投入"。手工与工具的先后顺序、字段集也未定义，学习效应未控。改：允许用户本人以真实主档在自身账号操作（研究者不采集值、只计时），或分记合成/真实复核耗时并声明外推限制；固定对照顺序与字段集。

**C3｜README §3 逻辑字段口径段、深大行、深交所行**
四处仍写"以合格复现 **(a)** 的实测为准／(a) 实测清点"，而 (a) 包含 a1 只读演示；定义段与 §7 明确只有 a2 判可填。执行者可据录屏宣称"已验证可填"，重演第 5 轮 C1。改：这四处一律改为 a2。

## 非阻塞建议

- **D1**：`dlh` 归"自定义控件②"无留证（class 仅 form-control），宜记"readonly 成因未知"。
- **D2**：`cal_ksrq/cal_jsrq` 疑日历控件，未标组件线索，计入 15 偏乐观。
- **D3**：门槛①"≥3 名"与配额 3–4 名近乎重合，招募不足时判定规则未定。
- **D4**：门槛②"复用"无观察窗口，"再次委托"是否含新页面不清。
- **D5**：CHANGELOG 仍写"审查一轮"，实际已 5 轮，同意后更新。
- **D6**：personas §2"25–40 岁"无证据，宜标假设或删。
- **D7**：E11 为市级问答，却支撑"全省通用"，宜标转述、待核省原文。
- **D8**：处理记录第 2 轮"深交所约 32／超 20"等旧值留存，宜标已废止。

## 核对范围

**已逐字核对**：README 全文（§0–§8 与引用口径段）、public-form-structure.json 全部 4 条逐控件属性并复算（深大 17=候选 9+敏感 4+readonly 4；深交所 39=19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm=15，radio 5 组，file 5 控件 4 语义组；标新 15；CIESCO 502）、docs/user-personas.md 全文、CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff、第 2–5 轮审查意见与逐项处理记录。计数、白名单/黑名单、¥699 分母、radio/checkbox"未证"、目标类型样本=0、E12/E18–E21 降级口径本轮均自洽。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做访谈本身不构成否决，本轮不新增市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 7 轮）

- 日期：2026-11-17T06:54:07.722Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §7 门槛②（与 §5、personas §1/§6 冲突）**
"复用"被限定为验收后 90 天内**同站同结构同模板**再次发生任务，新页面属新适配不算。但第一优先画像以**跨平台首次注册**为主、年检间隔长（§5 自述）。该门槛对画像一几乎不可能满足，收费实验将永远无法启动或被临场改口径。改：区分"产品映射复用"与"商业复用"，画像一允许"同一客户对新页面再次付费委托"计商业复用，或延长窗口/指定画像二为收费实验主样本。

**C2｜README §7 门槛①、§3 a2 情形二**
门槛①硬性要求 ≥2 名 a2 耗时对照，但画像一主任务是**匿名首次注册**，§3 已限定其只能走平台测试环境或本人真实资料，否则降为 a1 不计时。门槛只写了"不足 3 名"的分支，未写"a2 不可得"分支，易造成程序性死锁或临场变通写入。改：补判定分支——a2 不可得记"可填性/耗时未判定"，以账号内更新页或测试环境样本补足，不降门槛、不据 a1 下继续结论。

**C3｜README §3 a2 情形一、§6 第 4 条；personas §6**
a2 首选"用户本人真实主档"，而白名单含联系人姓名/电话/邮箱等个人信息，BYOK 下真实值直发第三方模型。材料只规定"研究者不接触值"和草稿/风控授权，未规定**企业/本人对真实数据外发所选模型的授权与合规前提**，与 §0 因 BYOK 排除敏感字段的逻辑不一致。改：a2 真实值路径前置数据外发批准（或限本地模型），未获批则退回合成并标注偏差。

## 非阻塞建议

- **D1**：CHANGELOG 仍写"审查一轮"，实为 6 轮，取得同意后更新。
- **D2**：§5 用 E10 支撑"机器码/同人办理风控"，超出 §2.2 所述内容，宜标条款待核。
- **D3**：§7 三分记录无处安放"readonly 成因未知"（dlh），建议加待定类。
- **D4**：personas §6 三分漏"敏感排除字段"，与 README §7 不齐。
- **D5**：README §5 第二画像"老后台录入员"在 personas 无独立画像，配额归属不清。
- **D6**：E12"与 E18 同口径"不妥：E18 为官方指引，E12 仅页面文案，等级更低。
- **D7**：处理记录第 2 轮 C3"快照重建"旧表述未标已废止，易被执行者误读。

## 核对范围

**已逐字核对**：README 全文（§0–§8 及引用口径段）、public-form-structure.json 全部 4 条逐控件复算（深大 17＝候选 9＋敏感 4＋readonly 4；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组；标新 15；CIESCO 502）、docs/user-personas.md 全文、CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff、第 2–6 轮意见与逐项处理记录。计数、白/黑名单、¥699 分母、radio/checkbox"未证"、a1/a2 分级、目标类型样本＝0、E12/E18–E21 降级口径本轮自洽；相对链接路径未见错误。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做访谈本身不构成否决，本轮不新增市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 8 轮）

- 日期：2026-11-17T06:59:09.139Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §7②（personas §6 同口径）——收费门槛自指，且"验收"无定义**
"启动收费实验须两名客户均实付且复用"＋"验收后 90 天窗口"：启动前不可能已存在实付与复用，条款不可执行，易被临场改口径；全文亦未定义 ¥699 交付"验收"判定项，窗口起点不可判定。改：拆为"启动"（门槛①通过＋报价范围/交付边界/验收清单书面确定）与"继续/扩大"（现≥2 名实付＋复用），并写明验收判定项与窗口起算日。

**C2｜README §0 第二候选、§5 第二画像；personas §2 候选 B——敏感字段口径未覆盖**
§0 因 BYOK 外发默认排除身份证号/银行账号，但第二候选"老 OA/ERP/票务/教务"与画像二"客户/商品/员工记录"未作同样排除；员工/学生记录常含身份证、银行、成绩，按现文可直接进入 M01 真实值路径，与 §0/§3 原则冲突。改：三处显式沿用同一敏感字段默认排除与外发批准前置，或限定为不含个人敏感字段的记录类型。

## 非阻塞建议

- **D1**：§7①"上月真实重复任务"未定义；建议明确"同主档跨平台≥2 次"计重复。
- **D2**：CHANGELOG 仍写"审查一轮"，实为 7 轮，取得同意后更新。
- **D3**：处理记录第 4 轮 D1（dlh 归②）、第 5 轮 C2（与 E18 同口径）、第 2 轮 C4（复用硬门槛）已被推翻，未标废止。
- **D4**：§4"统一申报和数据预填也在推进"超出 E17 内容，宜标推断或删。
- **D5**：§3 硬门槛写"用户自身账号真实页面演示"，但 a1 允许匿名页，措辞宜统一。
- **D6**：深交所 2 个 Chosen 若隐藏 select 可写即转为可填字段，报价分母宜预留重算规则。

## 核对范围

**已逐字核对**：README 全文（§0–§8 及引用口径段）；public-form-structure.json 全部 4 条逐控件复算（深大 17＝候选 9＋敏感 4＋readonly 4；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组；标新 15，stepHints 5、role/aria 全空；CIESCO 502）；docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–7 轮审查意见与逐项处理记录。

本轮认定自洽的项：白/黑名单边界、¥699 逻辑字段分母（剔除敏感/协议/readonly 三类/附件验证码）、radio/checkbox 与疑似日历"未证"、a1/a2 分级与匿名页限制、a2 不可得与样本不足分支、真实值外发批准前置、目标类型样本＝0、E12/E18–E21 降级与市级口径标注、复用两类定义、相对链接路径与 E 编号未见错配。招聘样本、厂商宣传、DOM 抽样均未被当作频率/规模/付费/兼容证据。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做用户访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 9 轮）

- 日期：2026-11-17T07:03:33.644Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §7②「启动条件」＋§3 报价分母段（personas §1/§3/§6 同口径）**
¥699 启动只要求门槛①通过＋范围/边界/验收清单书面确认，未要求对**该客户约定页面**先做 a2 可填性预检；而 §3 明确 radio/checkbox、ExtJS 触发器、Chosen、疑似日历全部「未证」，验收项却是「白名单字段全部正确填入、无需返工」。等于按未证可填性签约并承诺无法保证的验收，易产生交付纠纷与错误的收费扩大判断。改：启动条件加「约定页面先 a2 预检→书面确认分母；未证/不可填字段显式留人工，不计分母与验收项」，并把 §3「交付前逐页核对」改为「报价前」。

## 非阻塞建议

- **D1**：§7②「两名商业复用」只证服务回头客，宜注明不证产品可复用性与规模化。
- **D2**：「样本不足/a2 不可得→延长招募」无时间与投入上限，建议设退出/复议节点。
- **D3**：深交所「必传附件」、zcxy×3「协议勾选」无 required/label 留证，宜标推断。
- **D4**：§5「已有资料含资质清单/业绩」超出白名单字段，易被读成可交付范围。
- **D5**：CHANGELOG 仍写「审查一轮」，实为 8 轮，同意后按实际更新。
- **D6**：§5 用 E10 称风控「明确」却标条款待核，措辞宜降为「据转发指南，待核」。

## 核对范围

**已逐字核对**：README 全文（§0–§8 及引用口径段）；public-form-structure.json 全部 4 条逐控件复算（深大 17＝候选 9＋敏感 4＋readonly 4；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组；标新 15、stepHints 5、role/aria 全空；CIESCO 502）；docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–8 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感字段排除（含第二候选）、¥699 逻辑字段分母与重算规则、a1/a2 分级与匿名页限制、真实值外发批准前置、a2 不可得与样本不足分支、复用两类与启动/继续分步、目标类型样本＝0、E12/E18–E21 降级与市级口径标注、相对链接与 E 编号未见错配。招聘样本、厂商宣传、DOM 抽样均未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/自动提交限制未被弱化。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与「结论是否超出所给证据」；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 10 轮）

- 日期：2026-11-17T07:08:59.326Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜docs/user-personas.md §1 末段（与 README §5、E10 口径冲突）**
personas 仍写"部分平台对授权经办、机器码和独立投标有**明确**风控"，无来源与待核标注；README §5 第 9 轮已降为"据 E10 市级转发指南…条款序号与适用范围待核原文，不作'明确'断言"。同一证据两文档强度相反，且该断言被用于把"多客户投标服务商"整类用户排除出首选，属超证据的画像决策。改：personas 同步为"据 E10 市级转发指南，风控条款待核原文"，并注明排除理由主要是平台规则不确定性而非已证条款。

## 非阻塞建议

- **D1**：一次性匿名注册页上，a2 预检与 ¥699 交付是否同一次写入未定义，建议写明避免重复写入/草稿冲突。
- **D2**：§7①"FormFill 更低"无最小差值与重复测量，建议记录分钟差与相对幅度，防噪声假阳性。
- **D3**：§4 第一优先启动条件的替代清单漏"代办服务商"，与 §5/§7 不齐，建议补。
- **D4**：CHANGELOG 仍写"审查一轮"，实为 9 轮，同意后按实际更新。
- **D5**：处理记录第 9 轮 D6 只改 README，未记 personas 同步，建议补逐项落地。

## 核对范围

**已逐字核对**：README 全文（§0–§8 与引用口径段）；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组（zcd/mrlxr2/xb/mrlxr3/xb3），file 5 控件 4 语义组（lxfsqrh×2），zcxy×3；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–9 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感字段排除（含第二候选与画像二）、¥699 逻辑字段分母与重算规则、radio/checkbox 与疑似日历"未证"、a1/a2 分级与匿名页限制、真实值外发批准前置、a2 不可得与样本不足分支及 8 周退出节点、复用两类与启动/继续分步＋报价前 a2 预检、目标类型样本＝0、E12/E18–E21 降级与市级口径标注、相对链接与 E 编号未见错配。招聘样本（2 例）、厂商宣传、DOM 抽样均未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出并留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做用户访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 11 轮）

- 日期：2026-11-17T07:14:01.498Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §3「对照规程（两种情形通用）」＋§7①噪声重测，与 §3 情形二、§7② 冲突**
通用规程要求同页先手工、隔日再工具，噪声时再测一次；匿名首次注册页因此被写入 2–3 次，与 §3 情形二"注册即产生真实主体记录"、§7②"避免同页二次写入"直接矛盾，易触发风控/重复草稿，或逼出临场变通的假耗时对照。改：匿名页对照限平台测试环境，或明确允许重复草稿并逐次授权，否则降为 a1+人工计时；噪声重测适用同一例外。

## 非阻塞建议

- **D1**：第二候选（老后台）无任何任务存在证据，仅能力匹配推断，§4/§5 宜明示，避免与第一优先证据强度混同。
- **D2**：JSON `openShadowRoots` 只测开放影子根，闭合不可测；"0"不等于无，宜注明。
- **D3**：§7② 匿名页合并会话后分母可能实测缩水至 <5 字段仍收 ¥699，宜设最低字段数或终止/调价条款。
- **D4**：personas §6 未同步"匿名页预检与交付合并为一次写入会话"，建议补一句。
- **D5**：CHANGELOG 仍写"审查一轮"，实为 10 轮，取得同意后按实际更新。
- **D6**：E18 为 SOHO 平台方操作指引，"官方指引"宜改"平台方指引"，避免与政府官方混同。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算（深大 17＝候选 9＋敏感 4＋readonly 4；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组，zcxy×3；标新 15、stepHints 5、role/aria 全空；CIESCO 502）；docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–10 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感字段排除（含第二候选、画像二）、¥699 逻辑字段分母与重算规则、radio/checkbox 与疑似日历"未证"、a1/a2 分级与匿名页限制、真实值外发批准前置、a2 不可得/样本不足分支与 8 周退出、复用两类与启动/继续分步＋报价前 a2 预检、目标类型样本＝0、E12/E18–E21 降级与市级口径、personas 风控措辞已与 README 对齐；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样均未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出并留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 12 轮）

- 日期：2026-11-17T07:19:24.879Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §3「适用范围与匿名页例外」＋§7①（personas §6 同口径）**
匿名页写"不做双次写入"，却又允许"当场人工计时"——当场手工计时本身即第二次向同页写入，自相矛盾；且未规定"单次写入+人工基线"与合成对照是否计入门槛①的"≥2 名耗时对照"，回忆基线的偏差方向（易高估手工耗时、偏向 FormFill）未标。改：明确人工基线来源与可计入的证据等级，回忆基线标偏差或只认账号内/测试环境样本。

## 非阻塞建议

- **D1**：CHANGELOG 仍写"审查一轮"，实为 11 轮，取得同意后按实际更新。
- **D2**：README"第二画像=老后台录入员"与 personas"画像二=技术型"命名冲突，建议统一编号避免配额误分。
- **D3**：未说明扫描是否把页面**已有真实值**外发模型；建议核实并纳入外发告知字段清单。
- **D4**：personas §3"展会/活动报名"排在白名单限定括号之后，易被读成不受限，建议并入限定。
- **D5**：深交所 `xb/xb3`（性别）等是否属白名单未明示，建议在字段口径中列举边界项。
- **D6**：E05 第十一条、E17 第十九条序号与 E17 原文仍未核，定稿前宜核政府网原文。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组（lxfsqrh×2），zcxy×3；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–11 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感字段排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 字段缩水条款；radio/checkbox 与疑似日历"未证"；a1/a2 分级、匿名页限制与合并会话；真实值外发批准前置；a2 不可得/样本不足分支与 8 周退出；复用两类与启动/继续分步＋报价前 a2 预检；目标类型样本＝0；E12/E18–E21 降级与市级口径；personas 风控措辞已与 README 对齐；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样均未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出并留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 13 轮）

- 日期：2026-11-17T07:25:29.464Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §5「第二验证方向」＋§7①；personas §6 配额段**
§5 允许老后台业务样本"归画像一扩展"，§7① 又把"上月重复任务"定义为"跨平台或同页多记录均可"。结果：第二候选（老后台同页多记录）的用户可被计入"≥3 名画像一"分母，用另一方向的证据判定第一优先方向成立，直接误导优先级决策。改：门槛①分母只计画像一白名单任务（供应商注册/入库/更新）样本；老后台业务样本单列第二候选/画像三，删除"归画像一扩展"。

**C2｜README §3「证据等级与计入规则」＋§7①**
≥2 名耗时对照只认账号内页/测试环境双写入，而画像一核心是跨平台首次注册（匿名页不计时）。门槛通过后文本未限定结论适用任务类型，易把"账号内更新页省时"外推为"首次注册方向成立"并据此扩大投入。改：门槛①按任务子类型（首次注册／账号内更新）分别判定与记录；首次注册无 a2 时记该子类型未判定，投入结论不得跨子类型外推。

## 非阻塞建议

- **D1**：§7②"继续/扩大"合并表述，两名商业复用能否扩大易误读，建议分列条件。
- **D2**：首批硬门槛可由 a1 满足，宜注明只证任务/结构存在，不等于技术可达。
- **D3**：§3 外发清单标"源码已核"但源码未随材料提供，宜标作者自核、待复核。
- **D4**：上报的 label/邻近文本可能含页面已有真实值，外发告知清单宜提示该风险。
- **D5**：personas §5"Simplify 等已占位"无来源编号，建议补来源或标未核。
- **D6**：CHANGELOG 仍写"审查一轮"，实为 12 轮，取得同意后按实际更新。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组（lxfsqrh×2），zcxy×3；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–12 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感字段排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历"未证"；a1/a2 分级、匿名页单次写入与既往基线偏差标注、计入规则（personas 已同步）；真实值外发批准前置；a2 不可得/样本不足分支与 8 周退出；复用两类与启动/继续分步＋报价前 a2 预检；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样均未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出并留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 14 轮）

- 日期：2026-11-17T07:30:14.047Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §3「对照规程」＋§7①（personas §6 同口径）**
规程只要求"同一字段集、同一页面，先手工、隔 ≥1 天再 FormFill"，未规定两次须从同一**空初始态**开始。产品有"已有值保护"（§5、personas §2），手工填过的草稿次日多数字段已有值会被跳过，FormFill 侧耗时被人为压低，产生假阳性"继续投入"。改：明确每次新建草稿或清空至同一初始态，记录初始已有值字段数；无法重置的样本不计入 ≥2 名对照。

**C2｜README §7②「启动条件」与 §7①子类型规则冲突**
§7① 已规定首次注册／账号内更新分别判定、结论不得跨子类型外推；§7② 启动却只写"门槛①通过"。若仅账号内更新达标、首次注册记"未判定"，仍可据整体通过对匿名注册页报价售卖，等于用另一子类型证据支撑收费决策。改：启动条件限定"报价页面所属子类型自身达标"；未达标子类型不报价，或明记探索性交付且不计入继续/扩大判定。

## 非阻塞建议

- **D1**：CHANGELOG 仍写"审查一轮"，实为 13 轮，取得同意后按实际更新。
- **D2**：老后台业务样本归 personas 画像三，但画像三定义为"多网页×单记录运营"，任务形态不符，宜单列任务标签。
- **D3**：§7① 缺"账号内页只能做合成对照"的判定分支，易门槛悬空或临场计数。
- **D4**：§3"text/select 经真实浏览器 E2E 验证"依据发布清单，未随材料提供，宜标待复核。
- **D5**：§7①"≥3 名"是否需按子类型分别满足未写明，建议明确分子统计口径。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组（zcd/mrlxr2/xb/mrlxr3/xb3），file 5 控件 4 语义组（lxfsqrh×2），zcxy×3；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–13 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感字段排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历"未证"；a1/a2 分级、匿名页单次写入与既往基线偏差标注、计入规则；真实值外发批准前置与外发清单风险提示；a2 不可得/样本不足分支与 8 周退出；复用两类、启动/继续/扩大分步＋报价前 a2 预检；分母只计画像一白名单任务；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样均未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出并留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 15 轮）

- 日期：2026-11-17T07:34:01.508Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜docs/user-personas.md §6 末段（与 README §7①② 冲突）**
personas 仍把“继续/扩大”合并为“两名实付＋至少一名产品映射复用**或两名商业复用**”，等于允许仅商业复用就扩大；README §7② 已明确“仅商业复用不满足扩大”。且缺第 14 轮新增的“报价页面所属子类型自身达标”启动门槛与第 13 轮“子类型分别判定、不得跨子类型外推”。执行者据 personas 会对未验证的首次注册页报价并提前扩大。改：三条口径同步，或删枚举只引用 §7。

**C2｜README §3「适用范围与匿名页例外」＋§7②（取证与售卖互为前提）**
匿名页工具侧耗时被定义为“§7② 合并会话的单次写入”（即付费交付），而 §7② 报价须该子类型先达标、达标又须账号内/测试环境双写入对照。生产匿名首次注册因此永远取不到证据、也永远不可售卖，8 周后只会记“未判定”，方向被程序性死锁而非证据淘汰。改：明确研究性单次写入（用户本人真实资料、研究者只计时）可独立于付费交付进行并计可填性证据，付费合并会话只是其中一种情形。

## 非阻塞建议

- **D1**：CHANGELOG 仍写“审查一轮”，实为 14 轮，取得同意后按实际更新。
- **D2**：E05 第十一条、E17 第十九条序号与原文仍未核，定稿前核中国政府网原文。
- **D3**：外发清单、E2E 验证、MAX_MAPPINGS 均系作者自核，建议 M01 前独立复核源码。
- **D4**：§7②“两名实付且复用（两类之一组合）”在两名客户下的组合口径可再写明。
- **D5**：personas §6 亦缺“仅合成证据、真实复核耗时未判定”分支，随 C1 一并补。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组（zcd/mrlxr2/xb/mrlxr3/xb3），file 5 控件 4 语义组（lxfsqrh×2），zcxy×3；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–14 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感字段排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历“未证”；a1/a2 分级、匿名页限制、既往基线偏差标注与计入规则；同一空初始态对照规程；真实值外发批准前置；a2 不可得/样本不足/仅合成证据分支与 8 周退出；分母只计画像一白名单任务、老后台样本单列；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样均未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出并留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与“结论是否超出所给证据”；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 16 轮）

- 日期：2026-11-17T07:38:05.316Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §7① 与 §3「证据等级与计入规则」直接冲突（匿名子类型仍死锁）**
§3 本轮新增“匿名页子类型可用 ≥2 名研究性单写入＋可靠既往基线的一致省时证据达标（记单写入证据等级）”，但 §7① 仍写“只做 a1 或**匿名页单次写入**的用户……**不计入 ≥2 名耗时对照**”，且 §7② 启动须门槛①通过。执行者按 §7① 判定时，首次注册子类型永远无法达标、不可报价，第 15 轮 C2 只在 §3 落地。改：§7① 同步单写入达标路径与证据等级标注，并说明其与双写入对照的强弱关系。

**C2｜docs/user-personas.md §6 与 README §3 口径漂移**
personas 仍写“此类只计可填性证据与辅助线索，**不计入门槛①的 ≥2 名耗时对照（只计账号内/测试环境双写入对照）**”，且未提“研究性单次写入可独立于付费交付”。这与 §3 新规则相反，等于用旧口径否决匿名样本，重演第 15 轮 C1 的文档漂移。改：删该括注，改为引用 §3 计入规则，并补研究性单写入可独立进行一句。

## 非阻塞建议

- **D1**：CHANGELOG 仍写“审查一轮”，实为 15 轮，同意后按实际更新。
- **D2**：E05 第十一条、E17 第十九条序号与原文仍未核，定稿前核中国政府网。
- **D3**：外发清单、E2E、MAX_MAPPINGS 均作者自核，M01 前须独立复核源码。
- **D4**：研究性单写入后该匿名页已注册，与后续付费交付是否冲突未定义，建议写明。
- **D5**：§7②“两名实付且复用”与“乙仅实付即可”措辞不一，宜统一为“两名实付＋复用组合”。
- **D6**：处理记录第 12 轮 D2“归画像一扩展或画像三”已被 13/14 轮推翻，未标废止。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组（lxfsqrh×2），zcxy×3 排除；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–15 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历“未证”；a1/a2 分级、同一空初始态对照、真实值外发批准前置；仅合成证据分支、8 周退出；子类型分别判定与启动须子类型达标＋报价前 a2 预检；分母只计画像一白名单任务；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与“结论是否超出所给证据”；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 17 轮）

- 日期：2026-11-17T07:42:20.919Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §3「适用范围与匿名页例外」＋§7①（personas §6 同引）**
首次注册子类型唯一达标路径是「≥2 名研究性单写入＋可靠既往基线」，但基线必然来自**另一平台/另一页面**（本页系首次），字段集、页型、字段数均不同，而「可靠」只定义了回忆偏差、未定义可比性。跨页比较可产生假省时，直接支撑第一优先方向。改：规定基线须同类白名单任务、可填字段数相近且有记录佐证，按「每字段耗时」归一化比较并记录两侧字段数；不可比则记该样本未判定。

**C2｜README §3 情形一、§7①；personas §6**
三处写合成对照「系统性偏向 FormFill」（回源核对被消除→省时被高估），却同时标「只作下限参考」。偏差方向与措辞相反，执行者易把合成结果当保守估计采信。改：统一为「乐观上限估计／证据强度最弱档」，保留「不单独满足 ≥2 名」规则。

## 非阻塞建议

- **D1**：配额 3–4 名未按子类型分布，建议每子类型各 ≥2 名，否则 8 周后两类皆未判定。
- **D2**：§7① 复议「改推第二候选」前，须先补其任务存在证据，不得默认转向。
- **D3**：CHANGELOG 仍写「审查一轮」，实为 16 轮，取得同意后按实际更新。
- **D4**：E05 第十一条、E17 第十九条序号与原文，定稿前核中国政府网。
- **D5**：外发清单、E2E、MAX_MAPPINGS 均作者自核，M01 前独立复核源码。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组（zcd/mrlxr2/xb/mrlxr3/xb3），file 5 控件 4 语义组（lxfsqrh×2），zcxy×3 排除；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–16 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历「未证」；a1/a2 分级、同一空初始态对照、匿名页单次写入与研究性写入独立于付费；单写入达标路径已在 §3、§7①、personas 三处同步（第 16 轮 C1/C2 已落地）；子类型分别判定＋启动须子类型达标＋报价前 a2 预检；复用两类与继续/扩大分列；分母只计画像一白名单任务；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与「结论是否超出所给证据」；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 18 轮）

- 日期：2026-11-17T07:47:45.932Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §7②「扩大条件」（personas §6 同引）**
扩大＝提价/批量推广/产品化，其条件之一“另有可重复的效率证据”无定义；另一条产品映射复用限同站同结构同模板、窗口仅 90 天，而画像一账号内任务以年检为主（§5 自述间隔长），窗口内几乎不可观察。两条路径一虚一不可达，扩大判定将凭主观解释。改：量化该证据（如同映射命中次数、跨客户同页型耗时一致降幅），或使复用观察窗口匹配年检周期。

## 非阻塞建议

- **D1**：CHANGELOG 仍写“审查一轮”，实为 17 轮，取得同意后按实际更新。
- **D2**：§3“目标类型平台复现”硬门槛未列入 §7① 判定清单，执行时易漏项。
- **D3**：第二候选无独立继续/停止门槛，建议补任务存在与耗时最低要求。
- **D4**：§0 敏感字段“本地模型个案评估”若纳入，是否计入 ¥699 分母未写明。
- **D5**：E05/E17 条款原文与外发清单、E2E、MAX_MAPPINGS 源码仍待独立复核。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组（zcd/mrlxr2/xb/mrlxr3/xb3），file 5 控件 4 语义组（lxfsqrh×2），zcxy×3 排除；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–17 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历“未证”；a1/a2 分级、同一空初始态对照、匿名页研究性单写入独立于付费；单写入达标路径与基线可比性（每字段归一化、≤30% 字段差、不可比记未判定）已在 §3／§7①／personas 三处同步；合成对照统一为“乐观上限估计、最弱档”；子类型分别判定＋启动须子类型达标＋报价前 a2 预检；继续/扩大分列；分母只计画像一白名单任务、老后台样本单列；8 周退出与改推第二候选须先补证据；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与“结论是否超出所给证据”；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 19 轮）

- 日期：2026-11-17T07:51:13.182Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §7②「观察窗口」段自相矛盾（personas §6 已按新规）**
同段先规定「年检/年度类映射复用窗口 13 个月」，段末又写「观察窗口为各客户验收日起 90 天，窗口内未再次发生记未观察到复用机会」。执行者按末句在 90 天关窗，年检类永远取不到映射复用证据，第 18 轮 C1 的修复被抵消，扩大判定回到不可达。改：删末句统一 90 天表述，改为「窗口按上文分任务周期，自各客户验收日起算」。

**C2｜README §7② 验收清单「交付含映射」＋扩大条件「同一映射…含跨客户同结构页」**
材料只证明映射本地保存、MAX_MAPPINGS=20 按 createdAt 淘汰（且作者自核待复核），未提供导出/导入或跨安装迁移能力的任何证据。把「交付映射」写成验收项、把跨客户同映射命中写成扩大路径，等于按未证能力签约与判定产品化。改：验收项改为「交付配置说明/由我们代配并现场验证命中」，跨客户命中标为待验证能力，或仅认同客户同安装命中。

## 非阻塞建议

- **D1**：CHANGELOG 仍写「审查一轮」，实为 18 轮，同意后按实际更新。
- **D2**：E05 第十一条、E17 第十九条原文，及外发清单/E2E/MAX_MAPPINGS 源码，仍须独立复核。
- **D3**：§7① 判定清单宜注明：目标类型仅有 a1 时，结论限「任务/结构存在」，技术可达仍未判定。
- **D4**：§0 第二候选仅 ≥1 名对照，证据强度弱于①的 ≥2 名，宜标等级差异。
- **D5**：映射 20 条淘汰可能使已交付映射在复用前失效，建议列为 M01 记录项。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组（lxfsqrh×2），zcxy×3 排除；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–18 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历「未证」；a1/a2 分级、同一空初始态对照、匿名页研究性单写入独立于付费、基线可比性与每字段归一化；合成对照统一为「乐观上限估计」；子类型分别判定＋启动须子类型达标＋报价前 a2 预检；继续/扩大分列且效率证据已量化；分母只计画像一白名单任务；8 周退出与改推第二候选须先补证据；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与「结论是否超出所给证据」；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 20 轮）

- 日期：2026-11-17T07:56:15.817Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §7②「复用分两类」定义与「仅商业复用不满足扩大」自相矛盾**
产品映射复用被定义为"同映射再命中**或同范围再次委托**"。后者只是同一页面再次付费，映射可能已失效或被 MAX_MAPPINGS=20 淘汰、需重新配置，却被算作"证明技术资产可重复生效"，可直接满足扩大（提价/产品化）。这与同段"仅商业复用不证明产品可复用性"冲突。改：产品映射复用须以**同映射实际命中并留证**为准；同范围再次委托但需重配的记服务回头客，不支撑扩大。

**C2｜README §7②「扩大条件」效率证据未限定证据等级（personas §6 同引）**
"≥2 名客户同页型每字段降幅一致且 ≥20%（按 §3 规程取证）"未排除**单写入＋既往回忆基线**——§3 自述该路径证据最弱、回忆易高估手工耗时、系统性偏向 FormFill。结果是最弱证据可支撑最高风险的提价/产品化决策，与"合成不单独计数""单写入不得混同"的自定原则不一致。改：扩大只认账号内页/测试环境、真实主档的双写入对照；单写入证据仅支撑继续接单。

## 非阻塞建议

- **D1**：CHANGELOG 仍写"审查一轮"（实为 19 轮），且审查文件路径与处理记录相对链接不一致，宜核对。
- **D2**：E05/E17 条款原文，外发清单、E2E、MAX_MAPPINGS 源码仍待独立复核。
- **D3**：§6 配额 3–4＋2–3＋跨境可能超 §7 的 5–8 名，宜写明取舍顺序。
- **D4**：§0 第二候选双写入对照未说明记录新建页初始态与不提交口径，宜直引 §3。
- **D5**：§0 敏感字段个案纳入（本地模型）未写入 §7② 交付前置，宜补一句。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4＋readonly 4；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组，zcxy×3 排除；标新 15、stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–19 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历"未证"；a1/a2 分级、同一空初始态对照、匿名页研究性单写入独立于付费、基线可比性与每字段归一化；合成对照统一为"乐观上限估计"；子类型分别判定＋启动须子类型达标＋报价前 a2 预检；观察窗口 90 天/13 个月已统一；映射交付改为配置说明＋现场验证、跨客户命中标待验证；分母只计画像一白名单任务；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 21 轮）

- 日期：2026-11-17T08:00:22.729Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §7①「允许由画像二（同页多记录）样本补充」、§7②「产品映射复用主要由画像二/账号内更新任务贡献」**
与 §0 第二候选「只支撑继续验证、不支撑收费/扩大」、§5「老后台业务样本按任务标签单列，纯技术反馈才归 personas 画像二」、personas §2「画像二不能代替画像一证明付费」冲突。执行者可用老后台/技术样本的映射命中宣布画像一「扩大」。改：§7 两处改用「老后台多记录任务标签」，并明确扩大证据只来自画像一白名单任务（账号内更新子类型）。

**C2｜README §7②「扩大条件」两条路径强度不对称**
路径一「至少一名产品映射复用」按定义只需命中留证（日期/页面/字段数），无次数、无耗时要求；路径二须命中 ≥3 次且每次含复核省时、并限真实主档双写入对照。结果：单次命中（免模型但是否省时未测）即可提价/批量推广/产品化。改：映射复用用于扩大时须附「含复核耗时低于同字段集手工基线」的留证，并与效率证据同证据等级。

## 非阻塞建议

- **D1**：personas §6「扩大证据只计双写入对照」易被读成映射命中也须耗时对照，宜区分命中留证与耗时对照。
- **D2**：CHANGELOG 仍写「审查一轮」（实为 20 轮）；审查文件路径 docs/execution/ 与处理记录 ./ 不一致。
- **D3**：E05/E17 条款原文，外发清单、E2E、MAX_MAPPINGS 源码仍待独立复核。
- **D4**：§7②「由我们代配并当场演示」与 §6「研究者不接触值」宜区分研究口径与付费交付口径。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组（lxfsqrh×2），zcxy×3 排除；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–20 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历「未证」；a1/a2 分级、同一空初始态对照、匿名页研究性单写入独立于付费、基线可比性与每字段归一化；合成对照统一为「乐观上限估计」；子类型分别判定＋启动须子类型达标＋报价前 a2 预检；观察窗口 90 天/13 个月；映射交付改为配置说明＋现场验证、跨客户命中标待验证；映射复用须实际命中留证、服务回头客不支撑扩大；分母只计画像一白名单任务；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与「结论是否超出所给证据」；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 22 轮）

- 日期：2026-11-17T08:04:36.973Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜README §7②「扩大证据等级限定」与 personas §6 口径冲突**
README 写"上述两类证据均须来自账号内页/测试环境、真实主档的**双写入对照**"，但两条路径的定义都是"真实任务中命中 ≥3 次""验收日后再次任务命中"（单次留证）；personas §6 则读作"**基线**取自双写入对照"。按 README 字面，扩大永不可达或逼出临场变通。改：统一为"命中须真实任务留证，手工基线须取自同初始态、真实主档双写入对照"。

**C2｜README §7②「扩大条件」未限定任务子类型**
§7① 与②启动条件均规定首次注册／账号内更新分别判定、不得跨子类型外推；但扩大（提价/批量推广/产品化）只要求"画像一白名单任务"证据，而文中自述映射复用主要来自**账号内更新**子类型。结果可据更新证据对仍"未判定"或仅单写入证据的首次注册提价、推广。改：扩大结论限定于取得该等级证据的子类型，其余子类型维持原价、不推广。

## 非阻塞建议

- **D1**：CHANGELOG 仍写"审查一轮"（实为 21 轮）；审查文件路径 docs/execution/ 与处理记录 ./ 不一致。
- **D2**：personas §6 声明"不重复枚举"却仍详列扩大细则，正是 C1 漂移源，建议精简为引用 §7。
- **D3**：老后台新增记录页未归入 a2 情形一/二，宜注明"不提交不产生记录、重载即空初始态"。
- **D4**：§0 第二候选只设"≥5 条记录"，无每条剩余可填字段下限，建议补价值口径。
- **D5**：E05 第十一条、E17 第十九条原文，及外发清单/E2E/MAX_MAPPINGS 源码仍待独立复核。

## 核对范围

**已逐字核对**：README §0–§8 及引用口径段；public-form-structure.json 全部 4 条逐控件复算——深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组（zcd/mrlxr2/xb/mrlxr3/xb3），file 5 控件 4 语义组（lxfsqrh×2），zcxy×3 排除；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。docs/user-personas.md 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–21 轮意见与逐项处理记录。

**本轮认定自洽**：白/黑名单与敏感排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历"未证"；a1/a2 分级、同一空初始态对照、匿名页研究性单写入独立于付费、基线可比性与每字段归一化；合成对照统一为"乐观上限估计"；子类型分别判定＋启动须子类型达标＋报价前 a2 预检；观察窗口 90 天/13 个月；映射交付为配置说明＋现场验证、跨客户命中标待验证；映射复用须实际命中留证、服务回头客不支撑扩大；扩大证据只计画像一白名单任务；分母只计画像一；目标类型样本＝0；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。招聘帖（2 例）、厂商宣传、DOM 抽样未被当作频率/规模/付费/兼容证据；未上架、BYOK、iframe/Shadow DOM/PDF/附件/跨页/自动提交限制未被弱化；Excel/API/ERP/RPA/OCR/主体库/编标 CA/代办替代均已列出留 M01 实测。

**未独立访问，不作事实认定**：E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。判断仅基于材料内部自洽性与"结论是否超出所给证据"；未做访谈本身不构成否决，本轮不新增任何市场数据。

---

# 产品原点与场景方向重估：Qwen 独立审查（第 23 轮）

- 日期：2026-11-17T08:08:36.067Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因 ChatGPT 后端任务封存而无法读取正文，改用同一指定模型独立审查。
- 输入：research/2026-11-17-product-origin/README.md、research/2026-11-17-product-origin/public-form-structure.json、docs/user-personas.md
- 凭据：只从环境变量读取，未写入输出。

结论：同意提交

## 判定说明（无阻塞问题）
本轮未发现会导致错误产品决策的口径矛盾或超证据断言。三个假设均被拆边界而非整体采纳/否定；招聘帖（2 例）、厂商宣传、3 个 200 DOM 样本均未被当作频率、规模、付费或兼容证据；目标类型平台样本＝0 与"技术可达未知"在 README §3/§4、personas §1、execution-plan M01 四处一致。

## 已核对（逐项复算）
- README §0–§8 全文及引用口径段；personas 全文；CHANGELOG／product-overview／execution-plan／product-strategy 四处 diff；第 2–22 轮意见与逐项处理记录。
- JSON 全部 4 条逐控件复算：深大 17＝候选 9＋敏感 4（bankName/bankAccount/LPIDNumber/bidderIDNumber）＋readonly 4（2 ExtJS 触发器＋CAType/CACode）；深交所 39＝19text/10radio/5file/3checkbox/2password，19−2chosen−1dlh−1sjyzm＝15，radio 5 组，file 5 控件 4 语义组（lxfsqrh×2），zcxy×3 排除；标新 15＝11text/2number/1date/1textarea，stepHints 5、role/aria 全空；CIESCO 502。表内数字与原始留证一致。
- 认定自洽：白/黑名单与敏感排除（含第二候选、画像二）；¥699 逻辑字段分母、重算与 <5 缩水条款；radio/checkbox 与疑似日历"未证"；a1/a2 分级、同一空初始态对照、匿名页研究性单写入独立于付费、基线可比性与每字段归一化；合成对照统一为"乐观上限估计"；子类型分别判定＋启动须子类型达标＋报价前 a2 预检；继续/扩大分列、扩大须命中留证＋双写入基线且限本子类型；映射交付为配置说明＋现场验证、跨客户迁移标未证；分母只计画像一、老后台标签单列；8 周退出与改推第二候选须先补证据；E12/E18–E21 降级与市级口径；相对链接与 E 编号未见错配。iframe/Shadow DOM/PDF/附件/跨页/自动提交、未上架与 BYOK 门槛未被弱化；Excel/API/ERP/RPA/OCR/主体库/工商带出/一键带入/代办/编标 CA 替代均已列出并留 M01 实测。

## 非阻塞建议
- **D1**：CHANGELOG 仍写"审查一轮"（实为 22 轮）；审查文件路径 docs/execution/ 与处理记录 ./ 不一致，一并更新。
- **D2**：E05 第十一条、E17 第十九条序号与原文未核，定稿前核中国政府网。
- **D3**：外发清单、E2E 记录、MAX_MAPPINGS 均作者自核，M01 前须独立复核源码。
- **D4**：门槛①目标类型仅有 a1 时，宜注明该情形只支撑有限验证投入，不支撑逐站适配开发。
- **D5**：单写入路径的回忆基线偏向 FormFill 却可达标启动收费，宜要求耗时留证或标探索性收费。
- **D6**：§0 第二候选取证归"a2 情形一（更新/年检页）"，新建记录页非更新页，宜在 §3 并列命名。

## 未独立访问，不作事实认定
E01–E21 全部外链（条款序号、5,000 条上限、OCR 与工商带出行为、风控条款、招聘职责）、probe-public-forms.cjs 与 screenshots、entrypoints/content.ts 与 utils/mappings.ts 源码、release-checklist 等关联文档。本结论仅针对材料内部自洽性与"结论是否超出所给证据"，不代表外部事实已核实；未做用户访谈本身不构成否决，本轮不新增任何市场数据。
