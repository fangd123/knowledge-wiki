---
title: "FormFill SEO、关键词与获客研究"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 44
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# FormFill SEO、关键词与获客研究

2026 年 11 月 17 日，作者做的是搜索意图研究，不是关键词月量调查。搜“Excel 填表”，找到的可能是按表头粘贴工具；搜“FormFill”，还会遇到填 PDF 或建表的同名产品。先分清要完成的任务，才谈得上做哪一页。研究最终形成中英文各 20 个候选词；不等于固定地区的前十结果采了 40 遍。

## 实际找到什么，没找到什么

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

自编词表有 40 行，包含核心、场景、配置、比较、品牌、隐私和四个排除词。本站提供主题、代表查询与[词表结构核对](53-研究-19-SEO-worker收尾.md)，**不提供独立 CSV 下载**。原表月搜索量、CPC、KD 均未采样；空列是数据状态，不是零。按产品匹配程度定 P1–P3，是编辑判断，不是工具评分。

| 主题示例 | 用户可能要做什么 | 月量 | CPC | KD |
|---|---|---|---|---|
| Excel 资料填网页 | 同页换记录还是固定表头粘贴？先看原生导入。 | — | — | — |
| Word 资料填网页 | 源是 Word，目的地是既有网页，不是填写 Word 模板。 | — | — | — |
| 自备模型接口配置 | 查费用、权限与数据去向；不是获取免费模型。 | — | — | — |
| PDF filler / form builder | 意图不符，不因搜索量未知就占一个获客页。 | — | — | — |

## 三种数字，不能填同一列

- **Trends** 比同一图中固定地域和时段的相对热度；100 不是一百人，两张图上的 100 也不能比。窄词无结果不记零需求。
- **Keyword Planner** 提供限定地区、网络、月份及近似变体的市场搜索次数估计。原拟按语言市场分批，取 2025-11 至 2026-10 的完整十二个月。若只有 `100–1K` 就留区间；广告主竞争、页首出价和实际平均 CPC 是三件事。没有打开账户或取得导出。
- **Search Console** 只看本站的曝光、点击、查询与设备；没有域名和站长后台就没有基线。第三方 KD 也随数据库和算法变，不把 Ahrefs 与 Semrush 的数平均成官方难度。

原采样设计是先按市场、日期、设备和语言保存前十结果，再按任务聚簇去重，不直接相加近似词月量；每次只挑 3–5 个能拿出真实输入、结果和限制的主题。平台导出如要再发布，应另核使用条件。官方定义见[Keyword Planner](https://support.google.com/google-ads/answer/3022575?hl=en)、[Trends](https://support.google.com/trends/answer/4365533?hl=en)和[Search Console](https://support.google.com/webmasters/answer/7576553?hl=en)。

## 原拟 30／60／90 天复盘，没有发生

先讲准支持的四种格式、获取路径和模型费用，再考虑 Excel 指南、配置指南和有证据的比较页。不为近义词各造一页，也不把一次出站点击叫安装。官方更新在采样日显示 FAQ 富结果已停止展示；可见 FAQ 仍可回答实际疑问，不保证排名。网站后来完成四页本地预渲染，但没有生产域名、线上收录或可用的 28 天观察窗，见[冻结时的 SEO 状态](27-[历史]_SEO_与索引准备状态（S01–S05）.md)。

30／60／90 天原是未执行的复盘节奏，不是流量承诺。没有投广告、上架或采词；商店 users 不测活跃，官网模拟演示完成不等于扩展激活。不因缺数偷偷增加扩展追踪。读[执行计划](46-FormFill_网站、功能、宣传与_SEO_执行计划.md)和[交接决定](22-FormFill_产品收尾与交接报告.md)，可区分做过的本地技术工作和取消的市场实验。
