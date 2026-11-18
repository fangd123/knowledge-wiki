---
title: "FormFill SEO、关键词与获客研究"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: 项目复盘
series: FormFill浏览器扩展
series_order: 44
tags:
  - 项目复盘
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# FormFill SEO、关键词与获客研究

项目目录：`<本地路径>`。审计批次按父会话要求命名为 `2026-11-16-formfill-audit`；本轮联网核验实际发生在 2026-11-17（Asia/Singapore，跨午夜），不把网页抓取时间倒填为 11 月 16 日。

本报告是父会话委派的研究交付。只创建本报告及 `keyword-map.csv`，没有修改业务源码、发帖、购买工具或广告、执行 Plane 日志或继续委派。

父会话复核补充：`website-audit.md` 已记录生产构建、页面实测及默认学习采集、文本上下文、隐私承诺不一致的问题。执行本报告的推广计划前应优先处理这些发布前事项。社区具体规则以 `community-launches.md` 为准，尤其 HN 当前要求作者自行写作，不能直接发布生成或 AI 编辑的文案。

Qwen 复核后的当前执行口径见 [总执行计划](46-FormFill_网站、功能、宣传与_SEO_执行计划.md)：通用首页保留，业务场景先验证；中文试点渠道与英文技术反馈分开，英文页面按海外试点需要安排。下文候选词/页面是研究池，不是全部已排开发；30/60/90 天调整为复盘节奏，不作为功能、译文和内容的同时交付期限。

## 1. 结论与数据边界

**建议先把“能安装、能配置、能力描述准确”做成完整路径，再扩展自然搜索入口。** 首批应争取“AI 网页表单自动填写 / AI form filler Chrome extension”和“文件资料 → 网页表单”需求，以自备 HTTPS OpenAI 兼容接口、清楚的数据流和实际表单演示建立可信度。自备 Key 和本地存储在搜索结果中已有直接竞品，不能单独称为独有优势。

**品牌有明显重名风险。** 本轮搜索能看到多个名为 FormFill / FormFill AI 的商店条目和网站，包括 PDF 产品。没有任何证据证明它们属于本项目；禁止把其域名、用户数、评分、上架状态当成 FormFill 的资产。先统一开发者名、仓库、官网与商店链接；品牌标题加上“AI 网页填表 / AI Web Form Filler”和开发者身份，在正式推广前做名称、域名及适用市场商标检索。重名是获客和信任问题，本轮没有完成法律权利调查。

实际拿到的数据：本地 README 和网站源文件、搜索服务返回的真实网页标题/摘要和部分正文、可直接读取的 Google/Chrome 官方文档。**没有拿到** Keyword Planner 账户数据、Google Trends 导出、Ahrefs/Semrush 关键词导出、本站 Search Console 数据、正式域名线上收录/性能数据或本项目 Chrome Web Store 后台数据。因此 CSV 的月搜索量、CPC、KD 全部为“待采样”；优先级是编辑判断，不是付费工具评分。

搜索返回页只能证明“此次查询能发现这些页面及内容”，不能证明搜索量、真实用户人数、Google 某地域固定排名、竞品实际成功率或商业转化。搜索服务不提供本轮具体 Google 国家/设备参数，返回顺序不记录为 Google 排名。未使用结果总数或自动补全条数估计需求。

## 2. 已核实的网站与产品事实

证据文件：`README.md`、`website/README.md`、`website/index.html`、`website/privacy.html`、`website/vite.config.ts`、`website/src/App.tsx`、`website/src/PrivacyPolicy.tsx`、`website/src/site-links.ts`、`website/src/SiteChrome.tsx`。网站事实来自源码阅读，不等同于对正式部署的验收。

| 事实 | SEO / 转化含义与建议 |
| --- | --- |
| Chrome 扩展读取当前网页字段，根据自然语言或导入资料调用模型再填入网页 | 文案明确目的地是“网页表单”。文档作为资料来源，不能描述为生成表单或填写 PDF 文档。 |
| README 支持 `.xlsx`、`.docx`、`.txt`、`.md`；旧 `.doc` 需转 `.docx`，`.xls` 不支持 | `App.tsx` 的 FEATURES 把 `.doc` 写成支持，且 proof-strip 写“5 类文件导入”；建议统一为真实支持的四种后缀，解释转换方式。错误格式承诺会直接造成安装后流失。 |
| 用户配置 API Key、模型名、HTTPS Base URL，默认接口是火山方舟 | “支持 OpenAI 兼容接口”比“ChatGPT 账号登录即可”准确。推广页提前写清自备 Key、模型费用另计、需 HTTPS；没有实测的服务商不宣称全面支持。 |
| 本地存储 Key、配置、学习样本；模型请求发送用户资料、字段摘要及可选历史样本 | “本地学习”不等于模型离线运行或资料不出浏览器。隐私页和营销页必须保持相同边界。 |
| README 声明不扫描 iframe，支持 open Shadow DOM；文件导入有 10MB / 文本截断限制 | 场景文章给兼容范围、失败处理和手动复核步骤，不写“任意网站、所有控件、100% 正确”。 |
| Vite + React 首页及独立 `/privacy.html`；原始 body 为 React root | 可被 Google 渲染；静态输出是后续可验证性和跨爬虫兼容优化，不是“React 无法收录”的修复。 |
| 首页和隐私页已有不同 title、description、OG/Twitter、favicon、apple-touch-icon；隐私页有独立构建入口 | 这些是已有基础，不能列为“缺失”。OG 图目前用 128px 图标，可在上线时补真实大图和绝对 URL，属于增强。 |
| `CWS_LISTING_URL = null`、支持邮箱为 `support@example.com`，仓库链接指向 Gitee | 尚无可确认的正式商店链接或域名。不要生成假安装 URL、假价格、假证言。首阶段完善发布信息和可用获取路径。 |
| `rg` 检查网站可编辑源码未发现 canonical、hreflang、JSON-LD、sitemap/robots、分析 SDK 声明 | 这是源码观察；部署层可能另有响应头或文件，线上仍待查。域名确定后再输出规范 URL。当前只有中文站，不需要虚构英语 hreflang。 |
| 官网 Demo 默认本地模拟解析，可选配置真实后端 | 明确演示为模拟，不把 Demo 的识别率、耗时或无网络行为外推为真实扩展性能。可用真实扩展屏幕录像补证据。 |

建议首屏文案草案（需产品方确认后实施）：

- 中文 title：`FormFill｜AI 自动填写网页表单的 Chrome 扩展`。
- 中文说明：`把自然语言或 Excel、Word 资料填入网页表单。自备 API Key，连接 HTTPS OpenAI 兼容接口；设置与学习样本保存在浏览器本地，模型请求直达你选择的服务商。` 可按展示长度压缩，但保留自备 Key 条件。
- 英文页 title（未来真实译文页面）：`FormFill — AI Web Form Filler for Chrome`。
- 商店尚未发布时，把开发者模式安装教程作为真实路径；面向普通用户的获客以商店可安装为发布门槛，先使用现有仓库反馈渠道验证需求。

## 3. 实际搜索结果样本与意图判断

方法：使用联网 search 服务执行下列核心/场景/比较查询，读取其返回结果；部分 Chrome Web Store 页面可从搜索服务读取全文，直接 HTTP 请求有时只返回 `empty-title` 壳。Google 搜索前台的一次 HTTP 请求也只得到 JS 跳转页，因此没有伪装成已拿到无个性化 Google 前十。以下是**被发现的结果样本**，不是固定排名和搜索量报告。每条竞品功能只按其开发者页面自述理解，未安装验证。

| 编号 / 实际查询 | 读到的真实结果与具体 URL | 事实观察 → 建议 |
| --- | --- | --- |
| S1：`"AI form filler" "Chrome" "API key"` | [AI Form Filler](https://chromewebstore.google.com/detail/ai-form-filler/docejnnjfcpmlmniibkopmmcalocnnkp)、[filly 仓库](https://github.com/gulan28/filly)、[Smart Form Fill Assistant](https://chromewebstore.google.com/detail/autofill-forms-with-ai-sm/gdiflceinnmfaadkofiiaegiihlkjomp) | 商店和开源 BYOK 工具同时出现；Key 本地存储、直连模型并非空白市场。filly 用文本文件作上下文，是较贴近的直接比较候选。建议用文件类型、可重现案例、权限/数据流解释建立选择理由。 |
| S2：`AI 自动填写 网页表单 插件` | [智填](https://chromewebstore.google.com/detail/autofill/dpkbpaalhafihfcphnaihjmikfnpmbjj?hl=zh-CN)、[AI Form Filler 智能填表助手](https://chromewebstore.google.com/detail/ai-form-filler-%E6%99%BA%E8%83%BD%E5%A1%AB%E8%A1%A8%E5%8A%A9%E6%89%8B/icphdpdklolemhlljeemankinabcdpig)、[FormPilot 中文条目](https://chromewebstore.google.com/detail/formpilot-ai-%E6%99%BA%E8%83%BD%E5%A1%AB%E8%A1%A8%E5%8A%A9%E6%89%8B/fkbcidjkfdddhgnccbiheikbjlncpgne) | 中文结果已有自配模型/语义填表；“AI 填表”还混入生成表单的产品资料。标题需要“网页 + Chrome”，正文给输入资料到已有字段的图示。 |
| S3：`Excel to web form autofill extension` | [XL2Web](https://chromewebstore.google.com/detail/xl2web/jnpdodaepbebnikolajmemkfhcfhomml)、[Excel Paste Filler](https://chromewebstore.google.com/detail/excel-paste-filler/kalfomadmfcmflmhccjamjkkgejmanck)、[File Form Filler](https://exceltoform.com/new/)、[Formly](https://exceltoform.com/net.php) | 结果覆盖字段映射、多行、跨页和完全本地规则填充。FormFill 的 xlsx 是作为模型上下文，不能因此宣称逐行确定性批量提交。文章应解释“选一份资料填当前表单”的过程，并公开与批量导入工具的适用差别。 |
| S4：`AI form filler vs Text Blaze Magical` | [Text Blaze 官方比较文章](https://blaze.today/blog/text-blaze-vs-magical/)、[NexoPad 比较页](https://nexopad.app/en/alternatives/text-blaze-vs-magical)、[Filliny 比较页](https://filliny.io/blog/best-ai-form-filler-chrome-extensions-2026)、[Fill Hero 比较页](https://fillhero.com/vs/text-blaze/) | 内容类比较页有曝光入口，但大多由卖方编写。某文称 Magical 只是展开器，另一些列出跨表单填写；有的价格也互相不同。建议先写方法/工作流比较，之后用当前版本和统一任务实测两款产品，避免复制“竞品不支持”或旧价格。 |
| S5：`"FormFill" Chrome extension`（并在多查询批次复查） | [同名 FormFill 条目 A](https://chromewebstore.google.com/detail/formfill/pfninlkjkbbdbphbhkofolcanpjohpfk)、[同名条目 B](https://chromewebstore.google.com/detail/formfill/hlippmbnekodofbpibknpkgieknckjkf)、[FormFill PDF 产品](https://chromewebstore.google.com/detail/formfill-pdf-form-filler/eeipheboenmnacfkphmacdhamamfggke)、[formfill.cc](https://www.formfill.cc/)、[formfill.ai](https://formfill.ai/) | 多个同名实体和 PDF 意图。没有本项目归属证据；不能据此填写本项目商店链接。品牌需要描述词、作者身份和链接闭环。 |
| S6：`Excel 自动填写 网页表单 插件`、`"自然语言" "填表" "Chrome"`（同一多查询批次，无法严格分配每个返回结果到单词） | [xlsx2form-chrome](https://github.com/jiyusmart/xlsx2form-chrome)、[Formly 中文页](https://www.exceltoform.com/zh/formly/)、[Reed](https://chromewebstore.google.com/detail/reed/bighbodkddmcknkfkekdibioclohcebn)、[Formilot](https://chromewebstore.google.com/detail/formilot/pmnadgchapnkamjknglhpempjajbobng?hl=zh-CN) | Excel 映射和自然语言浏览器智能体都进入邻近结果。FormFill 可聚焦填表任务；不要借“自然语言”外推到导航、点击、跨站代理等未提供的能力。 |
| S7：`"AI form filler" "registration"`（与 S5/S6 同批） | [GistFill](https://chromewebstore.google.com/detail/gistfill-ai-form-filler/gbapjoopccjagddmpjgafogmdojbiiie)、[SupaSubmit](https://www.supasubmit.com/browser-form-filler)、[FormWiz](https://formwiz.ai/) | 报名、申请类场景有真实结果，但又混入 PDF 注册表。建议先用普通活动报名做实际样例；不以政府/医疗/金融敏感场景为主要卖点。 |

一次批量查询不会给出精确的逐词排名归因。CSV 中“相关簇”表示由这些样本和产品能力推导，**并不表示该完整关键词已单独采样过 SERP**。本轮也没有完整抓取每个关键词前十、广告位、AI Overview、People Also Ask 或移动/桌面差异；后续人工前十检查需补国家、语言、设备、日期和截图。

## 4. 关键词映射与页面结构

配套 `keyword-map.csv` 共 40 个英文/中文关键词（各 20 个），包括核心、场景、比较、配置、品牌、隐私及四个明确不匹配的排除词。每行有意图、产品匹配、拟用页面、理由、人工优先级及真实指标状态。P1 = 本批次优先，P2 = 实测/采样后，P3 = 验证需求后，排除 = 不用作获客目标。优先级不承诺流量、难度或排名。

先以**主题簇**组织少量优质页面，不为 CSV 每词创建一篇近似文章。建议路径均为未来方案；现有页面只有 `/` 与 `/privacy.html`。当前中文首页可保留 `/`，英文完整译文放 `/en/`；无须为了整齐立即迁移到 `/zh/`。

| 页面 / 簇 | 主任务、应具备的独有内容 | 优先顺序 |
| --- | --- | --- |
| `/`；未来 `/en/` | 自然语言和文件资料填写网页字段；真实扩展录像、配置要求、支持格式、兼容范围、获取路径；英文页完整本地化 | P1，先中文可信路径，英文按海外验证推进 |
| `/docs/setup/`；未来 `/en/docs/setup/` | 获取/加载扩展、HTTPS 接口配置、Key 保管、测试连接、常见 401/地址错误与模型费用说明；可公开访问的图文步骤 | P1，先解决 BYOK 转化摩擦 |
| `/guides/excel-to-web-form/`；未来英文对应页 | `.xlsx` 真实脱敏示例，指定当前要填的记录，多工作表如何变成上下文、解析/截断提示、逐项复核；明确没有承诺批量自动提交 | P1，证据齐备后上线 |
| `/guides/word-to-web-form/` | `.docx` 输入到网页，旧 `.doc` 转换；区别于“填写 Word 模板” | P2，可先作为文件指南章节，需求充分再拆页 |
| `/use-cases/event-registration/` | 普通活动报名端到端任务，处理选择框、填后复核、敏感字段/iframe 限制；真实成功和失败样例 | P2，先实际扩展验证 |
| `/compare/ai-vs-browser-autofill/` | 浏览器内置资料自动填充、文本模板、规则映射和 AI 语义填写的适用任务及成本；避免虚构性能结论 | P2 |
| `/compare/text-blaze/` 或 `/compare/magical/` | 同版本、同输入、同任务、记录日期/价格来源/结果和失败；不是“替代所有功能”宣言 | P3；先确认搜索意图和产品实测 |
| `/privacy.html` + `/docs/data-flow/` | 已有隐私政策保留；可增加易理解数据流：本地解析、模型发送、目标网页、日志/样本清除 | 政策一致性 P1；数据流文章 P2 |

“求职申请自动填写”和“发票信息填写”可先做现有文件指南中的例子，待真实目标网站验证后才拆成落地页。不做“自动投递上百份”“任意 Google Forms/Workday 全兼容”“身份证/支付信息智能填写”等承诺。`free`、`no API key`、`offline`、`PDF filler`、`AI form builder`、`CSV bulk submit` 均不能当作当前产品的吸量卖点；API 服务可能另收费，文件解析本地不代表 AI 请求离线。

## 5. 怎样查“搜索频率”，以及怎样使用导出

### 5.1 可执行的免费流程与边界

1. 先用本 CSV、真实用户表达和 S1–S7 结果标题建立词表；补同义词，不先写几十篇文章。访谈 5–10 个确实重复填网页表单的人，记录其任务、当前替代办法、自备 Key 能否接受。这是验证目标，不是已有样本结果。
2. 不购买工具即可手动看目标市场的搜索结果、竞品公开帮助页和社区问题；每簇先记录 5–10 个结果的页面类型、作者利益关系、版本时间和内容缺口。人工搜索/自动补全仅用于发现用语，不推导搜索量。
3. 用 Google Trends 免费比较同图中的 3–5 个词：先固定地域、过去 12 个月、Web Search，再看 5 年趋势；区分“搜索字词”与“主题”。窄词无图时扩大时间/地域或选上位词，不记成零需求。同图基准相同才能比较，分开两张图的 100 不可直接比大小。
4. 已有可用 Google Ads 账户时，在 Keyword Planner 选择“发现新关键词”或“获取搜索量和预测”，导入词表并导出历史指标。**不创建/启用广告系列、不投放，不需要购买 Ahrefs 才能做第一轮研究。** 但不能承诺无门槛免注册：当前 Google 官方帮助要求完成账户设置并填写结算信息才能访问基本关键词建议（来源 O8）。本轮没有替用户完成此流程。如不愿提供结算信息，跳过此步，保留“待采样”。
5. 正式站点上线后免费验证 Search Console Domain property（有 DNS 权限时），提交 sitemap，记录 query/page/country/device；积累至少 28 天并看滚动趋势，低流量时拉长窗口。它回答“本站已经在哪些查询中被展示”，不是全市场关键词月量。
6. 手动检查每簇 Google 前十的地区和设备、可实现任务与网站类型，结合真实 Demo/访谈结果挑首批 3–5 个页面。缺月量时仍可验证高匹配任务，不能把缺失数据写为零或编造小量。

### 5.2 Keyword Planner 设置与字段解释

建议分批，而不是把所有英语/中文需求揉成“全球总量”：

- 英文第一批：美国 + English；第二批英国/加拿大/澳大利亚/新加坡分别采样。最终以真实目标市场决定，不预设“全球英语用户”相同。
- 中文第一批：根据目标选新加坡、台湾、香港等市场分别设置语言；简体/繁体用词应分开看。Google 在中国大陆的样本不代表百度等搜索引擎的总体需求，更不代表所有中文用户；大陆需求先用访谈和本土渠道验证。
- Network 优先 Google，避免把 search partners 混入另一批数据；保存国家/地区、语言设置、网络、币种、采样日期、历史范围及工具名称。
- 这批审计在 2026-11 月，建议先用最近 12 个完整月 `2025-11-01 至 2026-10-31`，再按月看峰谷，并在可用范围内对照更长时期。不要用 11 月未满月直接和完整 10 月比较。

官方定义的 Avg. monthly searches 是在指定月份、地域、网络等范围内，关键词及其近似变体的平均搜索**次数**；默认通常为 12 个月平均。它不是独立搜索人数、访问人数、潜在客户数或能分到本站的点击。一个人可反复搜索，同一需求可换多个近义词搜索；不要把多个近似变体的合并量相加得到“市场总人数”。官方帮助还说明平均搜索量默认按该词不论语言统计，因此语言设置/建议语言不能被理解为按搜索者母语精准切割人口（来源 O9）。

账户界面若只给 `100–1K` 等区间，就原样保存下界/上界和 `range` 标志；不要取中点伪装精确月量。即使显示数字，也作为平台历史统计/估计而非实时全量账本。历史指标和未来广告预测分开保存；forecast clicks/impressions 会受预算、出价、季节和广告质量影响，不能换算为自然搜索流量。

`Competition` 的 Low/Medium/High 是广告主竞争，不是 SEO 难度。`Top of page bid (low/high range)` 是历史页首广告出价分位近似，不等于实际平均 CPC；CPC 导出列需保留具体字段名、币种和口径，不把广告出价上界填成“实际点击成本”。不能用高 CPC 断言该词 SEO 容易或产品必然赚钱（来源 O9）。

### 5.3 Trends、Search Console、Ahrefs / Semrush 的不同作用

| 来源 | 能回答 | 不能据此断言 |
| --- | --- | --- |
| Google Trends（O10） | 某地域/时间内经过归一化的相对搜索兴趣和季节性；指数 0–100 是相对比例缩放 | 100 是 100 次搜索、不同地区同指数有同月量；低量无数据 = 无人需要。Trends 有抽样和短期重复搜索过滤，但不是独立人数统计。 |
| Search Console（O11） | 本站搜索点击、曝光、CTR、平均位置及查询/页面/国家/设备；用于发现已出现需求和改进页面 | 全市场月量、所有网站份额或真实活跃用户。低频/匿名查询和导出限制会让查询行合计与汇总不同；平均位置也不是固定排名。 |
| Ahrefs（O12/O13） | 按国家数据库估算月量、SERP、页面流量潜力、链接与 KD，可快速筛选较大词表 | 精确真实月量；KD 是 Google 官方分数；高量必然有点击。其 KD 主要关注前十页面链接因素，需结合意图/内容/域名检查。 |
| Semrush（O14） | 按数据库估计关键词量、竞争页面和 KD%；适合有多簇/多市场时集中比较 | KD% 与 Ahrefs KD 可互换或等于广告竞争；其公式还考虑域名与 SERP 特征，分数不能跨工具直接平均。 |

购买优先级：先有可安装产品和一个可转化的核心站点，再使用免费资料、Keyword Planner（账户可用时）、GSC。若连续数周要筛多个市场/几十个页面主题或追踪竞品自然流量，再购买**一个** Ahrefs 或 Semrush 研究周期；按导出额度、国家覆盖、更新日期和团队工作流选，不为一张 40 词表同时订阅两套。不在本轮代买或承诺套餐价格。工具免费页面若只给有限结果，可作为补充，不能伪装完整数据库导出。

### 5.4 拿到 CSV 导出后的排序步骤

1. 保留原始文件；新增来源、提取日、国家/地区、语言、网络、币种、起止月、精确/区间/缺失标志。先按这些字段分区，不混排全球与美国数据。
2. 标准化大小写、空白和全半角；合并同意图近义词及工具近似变体组，指定一个主页面。月量保留每词原值，组内潜力不要直接求和；没有去重依据时用最大值作保守比较代理，并明确它不是主题真实总量。
3. 先排除产品不匹配词，再人工核验每个保留簇的意图。将“题目可写”与“产品真的能完成”分开，场景需要实测才能作为转化页。
4. 同一市场内按 1–5 人工评分：产品匹配 35%、转化意图 25%、可提供原创证据 20%、从前十样本判断的可竞争性 10%、需求信号 10%。若没有量/趋势/GSC，需求信号记 NA，其他项重新归一，不填 0。这个权重是建议，非统计模型；不要把 KD 缺失当低难度。
5. 需求信号可用同市场月量的 `log(1+量)` 百分位与趋势/GSC交叉检验，先按主题去重；只有区间时做上下界两种排序，先保留两种都靠前的主题。CPC 仅辅助商业意图判断，不作为 SEO 难度。
6. 对 P1–P2 的高分簇看前十页面是否满足任务，确认内容成本、实际扩展支持和安装路径；每轮只选 3–5 个页面。记录排序改变的理由。
7. 上线后用 28 天滚动、同比/环比口径和查询簇看曝光→点击→商店访问；样本不足先观察与访谈，不因为一周无排名量产页面或宣称失败。GSC 高曝光低 CTR 也需按位置、品牌/非品牌和设备判断，不能一概改标题。

## 6. Google 官方核验后的技术 SEO 建议

### JavaScript 与静态输出

**事实（O1/O2）：** Google 通过抓取、渲染、索引处理 JS 页面，渲染后的内容可用于索引。2026-03-04 更新明确删除“JavaScript 让 Google 更难访问”这一过时说法。当前官方 JS 文档仍建议考虑服务端渲染或预渲染，因为能改善用户/爬虫速度且并非所有机器人都执行 JS。

**建议：** 保持 Vite/React 架构，用适配现有构建的预渲染/静态输出，把首屏说明、支持格式、安装/配置、隐私摘要和可抓取链接放入最终 HTML，交互 Demo 继续 React。先比较投入与渲染实查结果；不要求重写 Next.js。构建后直接读取 HTML 和浏览器渲染结果，确认标题、正文、链接一致，JSON-LD不依赖点击后才出现；上线后以 GSC URL Inspection 的实时测试和已索引版本分别核验。允许 JS/CSS 资源抓取；嵌套路由必须能直接访问，真实不存在路径返回 404，避免统一首页 200 的 soft 404。

### Canonical、sitemap、robots 与多语言

**事实（O3–O5/O20）：** canonical 是规范化信号，不是保证 Google 必选；跳转、canonical 与 sitemap 应一致。sitemap 是发现提示，不保证抓取、收录或排名。hreflang 用于真实等价语言/地区页面，版本应列自身并互相回链，地址必须完整；Google 不仅靠 HTML `lang` 判断内容语言。robots.txt 管抓取，不是可靠的保密或移除索引方法。

**建议：** 正式域名和首选 HTTPS / www 形式确定后，首页、隐私页、未来文章各用准确的绝对 self-canonical，并统一内部链接/重定向。正文不同的文章不要全部 canonical 到首页。canonical 在原 HTML 和运行后应一致，不要先填占位域名后由 JS 换域名。sitemap 只放希望索引、可访问、规范化的真实 URL，不列 `#demo` 等锚点，`lastmod` 仅在内容实质更新时改。小站内部链接充分时 sitemap 不是上线硬门槛，仍适合提交 GSC 便于管理。上线再检查 robots/noindex、预览站索引策略、状态码和域名切换。

只有中文页面时保留 `lang="zh-CN"` 即可，不输出虚假的英语 hreflang。有完整 `/en/` 及相应中文内容时，在等价页面间加双向 `zh-Hans`/`en`，每个语言页面 self-canonical；不要把英文 canonical 到中文。`x-default` 只在确有合适默认/语言选择页时使用；无等价译文的文章不为凑齐而关联首页。提供可点击语言切换，不强制按 IP/语言重定向。

### 结构化数据与 FAQ 的 2026 年变化

**事实（O1/O6/O7）：** 合法结构化数据也不保证富结果；必须与可见内容一致。SoftwareApplication 的 Google 富结果要求包含规定字段，包括价格信息以及真实 rating 或 review。不能捏造评分、用户数、免费价格、发布日期或商店 ID。

**事实（O1）：** 2026-05-08 changelog 宣布 FAQ 富结果自 **2026-05-07** 起不再出现在 Google Search；旧 FAQPage 文档现已重定向到删除说明。过去“仅权威政府/健康站点有资格”的说法不再是当前完整边界，不能再建议通过 FAQ 标记争取 Google FAQ 展示。

**建议：** 可写用户需要的 FAQ：Key/费用、哪些数据发出、文件格式、iframe 限制、是否自动提交、模拟 Demo 与实际扩展的差异；用可见内容减少顾虑。不要为 FAQ rich results 投入或承诺结果。正式身份/价格/分发信息齐全时添加准确的 Organization/WebSite/SoftwareApplication 等语义；未满足评价/报价资格时不期待软件富结果。工具校验通过表示格式/规则检查，不代表排名或展示保证。本站没有可证实评价时不补造 AggregateRating。

### AI 内容与规模化页面

**事实（O15/O16）：** Google 关注为操纵排名而大量生产、缺乏原创价值的页面，是否用 AI 并非唯一标准。AI 可以辅助调研和结构整理，但大量无增值生成内容可能触发 scaled content abuse；事实和元数据需要人工核查。

**建议：** 每篇指南配一份可下载脱敏输入、真实扩展版本、测试环境、字段结果、限制及失败处理；比较页用同任务实测。不要批量生成“为 N 个城市/网站/职业自动填表”的近重复薄页，也不要从 SERP 拼贴“最佳工具”文章。更新实际变化，不靠改年份刷新假新鲜度。

## 7. Chrome Web Store listing / ASO

以下是发布准备建议，尚未创建商店条目。本仓库已有中英文扩展 i18n，但商店文案/截图仍需分别本地化。Chrome 官方要求 listing 准确完整，禁止关键词堆砌和不当证言（O17/O18）；发现机制还考虑多种因素，没有保证排名的固定配方（O19）。

- 名称草案：`FormFill — AI Web Form Filler`；中文 `FormFill — AI 网页表单填写助手`。最终检查后台长度限制和名称可用性；主词自然出现一次，不塞竞品商标列表。
- 摘要草案：`Fill web forms from natural language or files using your own HTTPS OpenAI-compatible API.` 中文：`把自然语言或资料文件填入网页表单，自备 API Key，连接 HTTPS OpenAI 兼容接口。` 上传前按后台实际字符限制校验。
- 长描述第一段说清输入→已有网页表单，紧接配置门槛/模型费用，再写四种文件后缀、普通字段/选择框、学习模式、使用步骤、限制、数据流、支持链接。真实兼容性优先于“any website”。不把 Key 本地存储称为加密保险库，不写“100% private / no data leaves device”。
- 准备 4–5 张真实截图：自然语言→普通报名表；`.xlsx`/`.docx`→网页；HTTPS/Key 配置与测试连接（遮蔽 Key）；本地样本管理；不支持/复核边界。配 30–60 秒真实使用视频。截图用虚构资料、不同语言真实 UI，不用官网模拟 Demo 冒充扩展实录。
- 使用一个真实支持邮箱、正式隐私 URL、项目官网/仓库；发布者身份与官网保持一致，有能力时完成官网验证。权限理由、数据使用披露和隐私页应一致；不能仅因为没有开发者后端就声明没有处理用户数据。
- 合理引导用户在解决问题后自愿评价；不购买、不奖励好评、不使用未署名假证言。先修复反馈造成的安装失败/误解，再尝试文案和截图调整。
- 分语言记录商店 impressions、可得的详情访问、安装和卸载；有需要时启用商店提供的聚合 GA4 数据与 UTM，先确认其口径（O21/O22）。Website SEO 与商店 ASO 是不同入口，不能把网页排名上升当作商店搜索排名上升。

## 8. 转化漏斗、指标与隐私边界

建议漏斗：**相关非品牌搜索曝光 → 网站访问 → 理解 Demo/能力 → 打开真实获取渠道 → 商店安装 → 完成 API 配置 → 在真实表单成功填写并复核 → 再次使用**。开发者模式阶段可以记录仓库/安装指南访问及自愿反馈，不能当作商店安装。

| 层级 | 可靠指标 / 公式 | 本轮状态与限制 |
| --- | --- | --- |
| 需求与索引 | 核心页面可索引比例、非品牌主题簇 GSC 曝光/点击/CTR，按国家设备分组 | 无正式域名/GSC数据；暂不能报基线。曝光不是独立人数。 |
| 网站理解 | `demo_start`、`demo_complete`、`setup_guide_open`；完成率 = 完成事件会话 / 开始事件会话 | 建议仅计官网模拟演示，不能叫扩展激活率；无身份追踪时会话去重也有限。 |
| 获取意向 | `store_outbound_click` / 网站合格访问，或开发阶段 `repo_open` / 安装指南访问 | 出站点击不是安装；无真实商店地址时不生成该事件。合格访问定义和窗口须固定。 |
| 商店转化 | 同口径商店访问→安装（若后台提供对应分母）、按语言/来源看趋势、安装和卸载 | 同期安装/访问只是近似，时间滞后和重复安装影响；不同后台分母不要直接相除。 |
| 首次价值 | 受邀用户自愿反馈的连接成功、首个表单完成、需修改字段数、配置到完成耗时 | 无扩展埋点，不能远程准确观测总体激活。可记录访谈任务成功率及样本数，不外推全用户。 |
| 留存 | 自愿回访的 D7/D30 再次使用、继续使用理由；商店安装存量/卸载趋势 | Chrome 官方 Users 只捕捉安装状态，不检测是否活跃，不能把它叫 DAU、MAU 或真实使用留存（O21）。 |

**隐私边界建议：** 当前隐私承诺是扩展没有分析/追踪。网站未来若需要匿名或低识别性的分析，作为独立决策：先明确网站隐私说明和适用同意要求，只收事件名、页面/语言、粗粒度来源、活动代码和必要技术状态；配置 IP/日志最小化、保留期及访问权限。不记录 Demo 输入、文件内容、文件名、表单值、API Key、模型响应或目标网页 URL；避免把姓名/邮箱装进 UTM 或自定义参数。来源 URL/查询参数也可能携带资料，分析端先白名单或裁剪，不依赖“用了无 Cookie 工具”就声称绝无个人数据。

网站统计不注入扩展，不跨域/跨设备拼接扩展用户身份；网站“去商店点击”与商店聚合安装通过 UTM 和时间窗观察，不能当逐用户归因。若以后真要在扩展里加自愿遥测，需要产品批准、重新审视隐私政策和商店披露及明确同意，本轮不建议为了算 DAU 改变现有无追踪承诺。可先用 5–10 位自愿测试者的定期反馈补首个价值/复用数据。

## 9. 阶段行动与 30 / 60 / 90 天复盘

时间从产品方开始实施算，第 30/60/90 天只复查进展与方向。以下按阶段门槛执行，产出/访谈数是建议工作量，不是日期或流量承诺；未取得能力验收时不靠赶文章填进度。

| 阶段（不绑定日期） | SEO / 内容 | 产品与商店 / 社区 | 验收与决策 |
| --- | --- | --- | --- |
| 发布基础与受控试用 | 修格式/费用/数据边界；确认域名与配置指南；评估最小静态输出；上线后做 canonical/404/GSC 核验；账户可用时采词；准备首篇真实文件指南 | 真实预览包或源码安装路径、准确截图与支持；观察 5–8 位用户的两类任务。真实业务试用前完成控制基础，多记录先选记录；商店和英文材料按实际分发/海外试点需要安排 | 发布信息真实；合成输入→真实网页任务可复现；记录配置/填写失败与任务频率；有线上数据后观察 28 天，不把缺失当表现差 |
| 复用验证与内容扩展 | 根据真实查询/任务反馈补少量独立内容；英文核心页按海外试点需要；真实译文才加 hreflang；比较页需同任务实测 | 对中文候选渠道先核实当日规则；英文社区用于技术反馈；每次一个真实任务并处理公开问题。有实际商店后再测试 listing 变化 | 看访问意图、获取/安装和自主配置；配置失败高先改引导，不扩内容；早期比较与任务观察可先于页面发布 |
| 有证据的渠道深化 | 深化有效主题、合并薄页；需要跨市场持续数据时才评估购买一个工具周期；复核政策/翻译/链接 | 经同意的真实案例；准确目录信息；持续支持与兼容说明 | 用滚动窗口及真实再次任务回访决定扩内容、改上手或停低匹配渠道；无须追求发帖数量或机械 D7/D30 留存 |

渠道建议只是候选：r/chrome_extensions、r/SideProject、Hacker News Show HN、V2EX/少数派相关社区及开源仓库讨论。**本 SEO 子任务没有审查每个社区当前规则，父会话应结合社区子任务的实查结果后选择。** 发帖前看当日规则、可用账号资格和自荐限制；不复制粘贴批量刷帖、不挖旧帖发广告、不把互链当目标。通常先发布可复现使用教程/技术经验、回答具体问题，再提供与任务相关的产品链接。候选渠道不是已获准发布的渠道。

## 10. 竞品 / 社区搜索 query 模板

下列是后续人工搜索模板，不代表已经全部查询。替换 `[task]`、`[product]`、`[site]`，保留市场/语言/日期记录；搜索操作符不返回结果也不能推论不存在。

```text
"AI form filler" "Chrome" "API key"
"OpenAI compatible" "form filler"
"fill web forms" "xlsx" extension
"natural language" "web form" extension
"[product]" privacy API key
site:chromewebstore.google.com/detail "AI form filler" "[task]"
site:github.com "form filler" "OpenAI-compatible"
"Text Blaze" "form filling" documentation
"Magical" "autofill" documentation
"[product] alternative" "web forms"
site:reddit.com/r/chrome_extensions "form filler"
site:reddit.com "copy paste" "web form" "Excel"
site:reddit.com "[product]" (broken OR privacy OR "API key")
site:news.ycombinator.com "Show HN" "form filler"
site:news.ycombinator.com "autofill" "API key"
"AI 自动填表" "网页" "插件"
"Excel" "网页表单" "自动填写"
"Word 资料" "网页表单"
"自备 API Key" "填表"
site:v2ex.com "填表" "插件"
site:sspai.com "网页" "自动填表"
"[目标网站]" "填表" "失败"
"[社区名称]" (rules OR guidelines OR 自荐 OR 版规)
```

## 11. 官方与方法来源

以下 URL 均在本轮实际请求读取，除特别注明外能返回正文。Google 帮助页可能有跳转、语言或界面差异；以其当前内容为准。官方事实与本报告建议已在相应段落分开。

- O1 Google Search 更新记录（核验 2026-03-04 JS 更新和 FAQ 自 2026-05-07 停止展示）：https://developers.google.com/search/updates ；FAQ 删除锚点 https://developers.google.com/search/updates#removing-faq-rich-result 。原 https://developers.google.com/search/docs/appearance/structured-data/faqpage 实际跳转到此删除说明。
- O2 JavaScript SEO basics：https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics
- O3 Canonical：https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls
- O4 Sitemap：https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap
- O5 hreflang / localized versions：https://developers.google.com/search/docs/specialty/international/localized-versions
- O6 SoftwareApplication 富结果：https://developers.google.com/search/docs/appearance/structured-data/software-app
- O7 结构化数据通用规则：https://developers.google.com/search/docs/appearance/structured-data/sd-policies
- O8 Keyword Planner 使用与账户设置要求：https://support.google.com/google-ads/answer/7337243?hl=en
- O9 Keyword Planner 预测与历史统计定义（含近似变体、平均月搜索量、广告 Competition、页首出价）：https://support.google.com/google-ads/answer/3022575?hl=en ；地域/语言/时间筛选：https://support.google.com/google-ads/answer/6325025?hl=en 。尝试的旧指标 URL `answer/92344` 返回 404，未作为证据。
- O10 Google Trends 数据、归一化与抽样：https://support.google.com/trends/answer/4365533?hl=en
- O11 Search Console 效果报告：https://support.google.com/webmasters/answer/7576553?hl=en
- O12 Ahrefs 搜索量估算（请求旧别名后实际跳转）：https://help.ahrefs.com/en/articles/72571-how-accurate-is-keyword-search-volume-in-ahrefs
- O13 Ahrefs KD：https://help.ahrefs.com/en/articles/72265-what-does-kd-stand-for-in-keywords-explorer
- O14 Semrush KD：https://www.semrush.com/kb/1158-what-is-kd
- O15 Google scaled content abuse：https://developers.google.com/search/docs/essentials/spam-policies#scaled-content
- O16 Google AI 内容建议：https://developers.google.com/search/docs/fundamentals/using-gen-ai-content
- O17 Chrome 商店 listing 信息与本地化：https://developer.chrome.com/docs/webstore/cws-dashboard-listing
- O18 Chrome 商店 listing 准确性与关键词垃圾政策：https://developer.chrome.com/docs/webstore/program-policies/listing-requirements
- O19 Chrome 商店发现机制：https://developer.chrome.com/docs/webstore/discovery
- O20 robots.txt 的用途/边界：https://developers.google.com/search/docs/crawling-indexing/robots/intro
- O21 Chrome 商店指标（尤其 Users 不衡量活跃）：https://developer.chrome.com/docs/webstore/metrics
- O22 Chrome 商店 GA / UTM：https://developer.chrome.com/docs/webstore/google-analytics

竞品证据为第 3 节 S1–S7 的具体 URL；本轮没有对其源代码、安全性、收费真实性、用户数真实性作独立审计。一次直接抓取 `formfillai.com` 返回 520，某猜测文章地址返回 404；两者均未用作产品结论。没有把错误/不可达地址补写成证据。

## 12. 交付与复核状态

- 已完成：读取指定三份文件并补查网站配置/隐私源文件；实际联网执行中英文核心、文件场景、自然语言、品牌和比较查询；核验官方 JS/canonical/sitemap/hreflang/结构化数据/FAQ/AI 内容政策；形成 40 词映射及落地路径、数据采样方法、30/60/90 天计划与隐私边界。
- 已验证：报告中关键 Google/Chrome 官方页面可读取；FAQ 旧文档跳转及 2026 年停止展示记录；本地 metadata/favicon/privacy 已有；CSV 完整性另用 Python 检查行数、语言、重复和指标待采样值。
- 未验证：正式网站 HTTP/Google 渲染/收录/速度，本项目商店上架、真实用户/安装/转化/留存，40 个词各自固定地域前十，任何月量/CPC/KD，以及所有拟发布场景和比较页的实际产品效果。
- 父会话下一步：复核 `.doc` 和“五类文件”表述、正式发布信息与隐私边界，选择首批 3–5 个页面；拿到真实关键词和 GSC 导出后按第 5 节去重、分市场和重新排序。
