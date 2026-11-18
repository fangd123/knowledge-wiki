---
title: "FormFill 社区发布案例研究与英文草稿"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 47
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# FormFill 社区发布案例研究与英文草稿

任务归档日期：2026-11-16。此次联网复核发生在 2026-11-16 UTC（新加坡时间已进入 11 月 17 日）。本文没有发布任何帖子，没有联系作者，没有购买服务；所有草稿均未发布。未改业务源码，未执行 Plane log，未再委派。

**父会话复核补充（2026-11-17）：**已完成本地构建、46项既有测试和3项专项隐私复现。学习模式存在首次同意前保存、读取整页其他表单值的问题，附近文本可能夹带文本域初始内容；详见同目录 `website-audit.md` / `privacy-evidence.json`。下列发布草稿是内部材料，实际推广前应先处理这些发布前问题并同步真实产品行为。不能把“本地保存”宣传为“仅在主动填表后采集”或“绝不外发”。

## 结论与适用范围

已联网核验 **Reddit 4 篇、Hacker News 7 篇**，其中 HN 选 5 篇作主要写法分析、另 2 篇记录证据不足之处。Reddit 四篇都取得原帖文字和 API 返回的可见评论；HN 通过官方 Firebase API 验证原帖，再与 Algolia 内容交叉核对。满足两平台各至少三篇的目标，但这是一组写法样本，不能据此推断发布成功率或获客效果。

最适合 FormFill 的定位是：**把自然语言或现有文件里的资料映射到当前网页表单的 Chrome 扩展，用户自备模型 API key**。先说“已有资料 → 网页字段”，再解释模型、安装和数据流。不要包装成 PDF 工具、表单生成器或全自动申请代理。

最有参考价值的两篇是 Reddit 的 FastFiller（功能、模型、隐私、安装、具体反馈问题分层写清）与 HN 的 ApplyEngine（同为 BYOK，但其“FREE unlimited”容易掩盖模型账单）。Reddit 的 Filler 评论还提供了实际问题：读者关心猜测标记和本地存储，而非只看“一键”口号。

**当前发布约束：HN 明文禁止生成文本及 AI 编辑的文本。** 下文按委派要求提供英文结构示例，但 HN 部分只能用作内部沟通材料，不能直接粘贴，也不能仅换词后发帖。作者应依据自己核实的产品事实和自己的真实经历，从空白独立手写标题及正文。Reddit 两版也应由作者核对措辞和当日规则。

## 产品事实与文案边界

先读取了仓库 `README.md`、`website/README.md`。为避免“本地学习”造成隐私误述，额外只读检查了 `entrypoints/background.ts`、`entrypoints/content.ts`、`utils/llm-retry.ts`、`utils/learn.ts`、`utils/settings.ts` 和 `LICENSE`；本次没有实际安装扩展或测量模型费用。

| 可使用的事实 | 对发布文案的影响 |
|---|---|
| 支持自然语言及 `.xlsx`（全部工作表）、`.docx`、`.txt`、`.md` 内容填入网页表单 | 开头提供一个具体资料映射场景，不写“任何文件”“任何表单” |
| 目前仅源码构建；`npm install`、`npm run build` 后，在 Chrome 开发者模式加载 `.output/chrome-mv3` | 仓库配置指向 Gitee 源码，但匿名可达性尚未验收，后续公开 HTTP 检查返回 403；真实可达后才用作 CTA，或先提供实际 Release/镜像。明确未上 Chrome Web Store，不写“一键安装”或虚构发布包 |
| 用户配置 API key、模型名和 HTTPS OpenAI 兼容 Base URL | API 调用由用户选定服务商计费；没有实测单次费用，不承诺免费额度、固定价格或零成本 |
| API key、设置、学习样本保存在扩展本地存储；没有 FormFill 自有后端 | 可说“没有 FormFill 后端”，不能说“所有数据永不离开设备”或“端到端加密” |
| 请求含用户输入/文件解析内容，以及结构化表单上下文；上下文含标签、附近文字、选项等 | 文件在本地解析，不意味着解析内容不外发；用户选择的模型服务会收到请求资料 |
| 学习模式默认开启；匹配同 origin、同表单签名的历史值会追加到后续模型请求 | “本地保存”与“参与后续外发请求”必须一起解释。关闭后不新增、不读取历史用于填充；最多 50 条，30 天有效，实际清理在读写时发生 |
| 扫描不会直接读取普通文本输入、密码输入、文本域、下拉框的 value 属性；radio/checkbox 保留选项 value；父会话已复现文本域初始内容仍可经 nearbyText 带入摘要 | 不能扩展成“已填内容绝不发送”或“完全不读取个人数据”。密码控件的元数据可在摘要出现，但不会执行密码字段填充 |
| 模型只可引用扫描分配的 `fieldId`；执行层拒绝未知 ID、密码、隐藏域、文件控件 | 可以解释约束机制，不能声称保证所有敏感字段识别正确或彻底解决提示注入 |
| 支持 open Shadow DOM；不扫描 iframe、无法访问 closed Shadow DOM | 明确兼容限制；不要借用竞品的 React-Select、级联下拉“全覆盖”承诺 |
| 当前没有预填审核界面、猜测字段标记或一键撤销的已核实依据 | 不把 Filler/FastFiller 功能写成 FormFill 功能。建议填写后人工核对，并不等于产品提供填前审核 |
| 网站首页默认是浏览器内模拟解析 Demo，可另行接可选后端 | 不把网站 Demo 展示说成真实模型准确率证据，不承诺已有实测视频 |
| MIT 许可、公开源码；无真实用户/增长指标材料 | 可说 source available under MIT，不杜撰用户数、节省时间、准确率和个人创作经历 |

## 核验方法、证据层级与限制

- **A：官方社区原始内容。** Reddit 官方只读 OAuth 的 `/comments/{id}`、`/r/{sub}/about/rules`、`/about`、`/hot`；HN 官方 `https://hacker-news.firebaseio.com/v0/item/{id}.json`。记录标题、作者、时间和可见正文。A 代表“帖子确实如此写”，不代表产品宣传已被独立验证。
- **B：搜索服务内容。** HN Algolia `https://hn.algolia.com/api/v1/items/{id}` 与已有搜索 JSON，作为发现和交叉核对来源。本轮主要 HN 结论都有 A 层根帖依据。
- **C：标题级或缺文证据。** 只有标题/出站 URL，或作者评论已 flagged/dead，无法据此分析完整开头、机制及 CTA。不会补写缺失正文。
- 优先复用了旧 `community-*.json`，再定点请求原帖和规则，没有重新大范围爬取。Reddit OAuth 成功，凭据只在内存处理，没有保存或输出明文、token，没有访问私信。初始 PowerShell 请求遇到 User-Agent 格式处理错误，改用 .NET HttpClient 后官方端点均返回 200。
- `community-live-reddit-*.json` 保存本轮公开响应；`community-live-hn-*.json` 保存 Algolia 内容，`community-live-hn-official-*.json` 保存官方内容；`community-live-hn-*-guidelines.html` 保存当次指南。精选结构化清单见 `community-selected-evidence.json`。
- 本文“全文”指**该条帖子文字正文**；不是作者全部历史、出站产品网站全文或视频转写。视频未观看，不把视频内容当作已核验体验。评论只限 API 本轮返回的可见内容；Reddit `num_comments` 与返回条数可能不同，不能声称取到了所有评论。HN dead/flagged 内容不能复原。
- 所有日期按原帖 UTC 日记录。投票/评论量只反映抓取时状态；样本小且非随机，不用来证明转化效果。Product Hunt 属可选补充，本次已有足够一手案例，未额外加入核验较弱的 PH 摘要。

## Reddit：四篇发布案例及写法

### R1 · Fyllyo：先展示操作链，再补个人原因

- **原帖：**[Built a side project so you don't have to fill a form manually again](https://www.reddit.com/r/SideProject/comments/1o5i554/built_a_side_project_so_you_dont_have_to_fill_a/)
- **日期 / 作者 / 自荐：**2025-10-13；`u/Emotional_Relation69`；正文明确说 “Built AI form filler” 和 “I will keep building”，属于作者自荐。
- **关联：直接同类。** 文档/文本 → AI 整理 → 网页填表，最接近 FormFill 的文件入口，但本文没有独立测试该产品。
- **开头：**“Hey everyone, found this community love it, hopefully can get some good feedback.” 随后立即放一句产品解释和链接。
- **痛点：**作者自述移民每年填写签证资料，收集资料和重复填写都费力。这是该作者的经历，不能移植为 FormFill 作者经历。
- **机制：**上传文档或文本 → AI 解析、组织为多个资料集合（例如自己/伴侣）→ 选择集合填表。
- **证据：**全文给出流程和产品链接，没有实测数字、兼容清单或本帖操作演示；“fill any form”是作者宣传承诺，未独立证实。
- **CTA：**希望读者试用并给反馈，未限定要反馈哪类字段或失败路径。
- **评论疑虑：**唯一返回评论指责 spam，且带人身攻击；这不能证明产品违法、被版主认定刷屏，也不能证明功能失败，只说明此帖出现了广告反感。
- **全文/摘要与层级：**已读文字全文及返回的 1 条评论，A；保存 `community-live-reddit-comments-1o5i554.json`。

**怎么写、怎么借鉴：**“三段箭头流程”比一长串 AI 功能容易理解；真正的签证情境使抽象填表变具体。开头的客套和“never/any”承诺占用了信任空间，CTA 也过宽。FormFill 可借鉴“导入资料 → 对应网页字段”的流程，把第一句改成具体任务；收尾只问一个可回答的问题，例如哪些字段需要人工映射。不要复用其无限兼容承诺。

### R2 · Parent Form Filler：明确家庭细分场景，用隐私解释区别

- **原帖：**[Just launched my secure, privacy respecting, AI automated form filler (focused on parents / family)](https://www.reddit.com/r/chrome_extensions/comments/1p040j1/just_launched_my_secure_privacy_respecting_ai/)
- **日期 / 作者 / 自荐：**2025-11-18；`u/Fun_Ad5060`；“So i built one!” 和 beta 链接，明确自荐。
- **关联：网页填表直接同类，同时含 PDF 邻近能力。** FormFill 不支持 PDF，不能照搬整套产品描述。
- **开头：**“Hi all - would love feedback. Launched by unique form filler today.” 接着说自己被家庭行政事务淹没。
- **痛点：**浏览器地址自动填充无法管理孩子、学校、医生等多个家庭成员的扩展资料；在线表单和 PDF 需要不同工具。
- **机制：**作者称 AI 只收到占位符，真实个人资料在本地回填；面向家庭成员数据。PDF 支持也是作者自述。
- **证据：**本帖附 78 秒视频及产品链接，提及作者从事安全领域；本次未看视频或验证其占位符实现。作者的职业背景不是独立安全审计。
- **CTA：**beta 求使用反馈，披露因为调用 AI 长期可能收费，未给正式价格。
- **评论疑虑：**唯一返回评论（2026 年）询问产品是否仍可用、说链接似乎打不开。这里只能确认读者报告过可用性问题，不能断言网站当前永久失效。
- **全文/摘要与层级：**文字全文、视频元数据、1 条评论，A；没有视频内容核验。保存 `community-live-reddit-comments-1p040j1.json`。

**怎么写、怎么借鉴：**将“普通 autofill 不够”的原因收窄到多成员资料，比“人人讨厌表单”更有辨识度。隐私机制紧跟痛点，让它成为选择产品的理由。FormFill 可以借鉴解释数据流的顺序，但不能写“AI 只收占位符”：它发送的是用户输入/文件内容与字段上下文。长期存活的源码和明确安装入口也是发布材料的一部分，不能只留一个营销域名。

### R3 · Filler：一段讲完整价值，评论把信任问题变成改进

- **原帖：**[Filler - a Chrome extension that fills job applications from a profile you save once](https://www.reddit.com/r/SideProject/comments/1xaoo4w/filler_a_chrome_extension_that_fills_job/)
- **日期 / 作者 / 自荐：**2026-10-20；`u/pop6996pop`；“so I made…” 和后续版本回复，明确作者自荐。
- **关联：直接同类中的求职垂直产品。** 从保存的 profile 填申请；不是任意文件解析流程。
- **开头：**“I kept typing the same details into every job application, so I made a free Chrome extension…”
- **痛点：**每次申请重复输入相同资料。
- **机制：**保存一次 profile、填申请、标记猜测、不代按提交；原帖说无需账号。
- **证据：**原帖只一段，链接到产品；评论中作者给出 v0.0.6、琥珀色边框与 “Guess: check this” 标签，并附 13 秒近景演示链接。这是可定位的更新自述，本次没有安装验证。
- **CTA：**“I'd like to know which forms it gets wrong.” 明确要失败案例，低于泛泛“支持一下”的反馈门槛。
- **评论疑虑：**读者 `Brufacee` 要求近景展示猜测字段标记，并问 profile 是否留在设备。作者解释本地存储并回应改进，读者认可文字标签比仅颜色清楚。另有一条带竞品链接的评论，不计作独立产品验证或真实转化。
- **全文/摘要与层级：**原帖文字全文、返回的 6 条可见评论（含嵌套回复），A；API `num_comments=10`，不能说全部 10 条都读到。保存 `community-live-reddit-comments-1xaoo4w.json`。

**怎么写、怎么借鉴：**一段完成“重复工作 → 功能 → 控制权 → 入口 → 指定反馈”。评论不是单纯好评，而是展示了读者怎样判断可信：看到哪一步需要检查，知道资料存在哪里。FormFill 适合沿用具体失败反馈的 CTA；猜测标记和填前审核属于潜在产品改进，不应提前写进发布帖。可以预备一张真实操作截图解释填后核对，但此刻没有已制作好的素材。

### R4 · FastFiller：最接近现阶段 FormFill 的源码分发案例

- **原帖：**[I build Chrome extension that fills forms with AI (100% zero cost) | FastFiller](https://www.reddit.com/r/chrome_extensions/comments/1xny1c5/i_build_chrome_extension_that_fills_forms_with_ai/)
- **日期 / 作者 / 自荐：**2026-11-04；`u/TheMauryaAnkur`；“I built FastFiller”，明确自荐。
- **关联：直接同类且最接近。** Chrome AI 填表、用户可自备 key、源码分发；但其本地 Ollama、审核、撤销、多 profile 等并非 FormFill 已有能力。
- **开头：**“I built FastFiller, a free and open-source Chrome extension that fills web forms using AI.” 下一段解释重复求职/注册资料、常规 autofill 在下拉和自定义组件上失效。
- **痛点与机制：**读取表单 → 匹配 profile 或简历文本 → 审核 → 回填；按功能、模型选择、隐私、安装、链接、反馈问题组织。
- **证据：**有 MIT GitHub 源码入口、网站、203 秒视频元数据；列出具体控件类型与限制。所有功能均是原帖自述，未运行验证。特别披露实验性的浏览器会话复用可能失效及违反服务条款。
- **CTA：**问优先测试哪些表单、审核界面是否足够、什么会使读者信任；也欢迎 issue/PR。
- **评论疑虑：**返回 2 条可见评论：一条鼓励，一条建议去求职平台找测试对象并附竞品链接；没有足够实质安全讨论。API `num_comments=3`，未返回部分不能臆测。
- **全文/摘要与层级：**文字全文、返回评论和视频元数据，A；未看视频。保存 `community-live-reddit-comments-1xny1c5.json`。

**怎么写、怎么借鉴：**最值得借鉴的是把安装和费用放在正文，而不是把读者引到网站后才发现门槛。标题“100% zero cost”却与后文 BYOK/免费层有张力：软件免费并不保证模型推理免费。FormFill 应直接用 BYOK 和 source-build 标明阶段，不借其标题，也不借用本地 Ollama 或非官方会话能力。其三问很实用，但 FormFill 首帖可以只选最关键的一个问题，减少回答负担。

## Hacker News：五篇主要案例与两篇证据不足样本

### H1 · Autofillr：本地优先的一句话定位，但正文交代不足

- **原帖：**[Show HN: I made a local first AI web form autofillr (no tracking)](https://news.ycombinator.com/item?id=40092580)
- **日期 / 作者 / 自荐：**2024-04-19；`nilni`；根帖与第一评论同作者，评论 “a project I made”，自荐确认。
- **关联：直接同类。** AI 网页填表，例子是求职和金融申请。
- **开头：**根帖是链接，无文字正文；第一作者评论以 “it was a project I made last year…” 开始，先讲因没时间暂停开发。
- **痛点 / 机制：**只有应用场景与“all data saved locally, not send to server”自述，缺少从输入到字段映射的细节，不能推断其使用何种模型。
- **证据 / CTA：**产品域名、可用性自述；请求试用并判断是否值得继续。没有视频、成本或测试数据。
- **评论疑虑：**仅作者本人 1 条评论，没有读者疑虑证据；不能从无评论推断读者满意。
- **全文/摘要与层级：**根帖元数据 + 唯一作者评论全文，A（官方根帖和评论）+ B 交叉核对。证据 `40092580`、`40092581`。

**写法分析：**标题一次讲清作者身份、对象与本地优先，适合 HN 快速扫读。首评却主要讨论是否继续项目，读者缺少立刻尝试所需的信息。FormFill 应把输入、安装和模型数据流写在作者首帖，不只写“local first”。

### H2 · ApplyEngine：BYOK 与求职流程绑定，但“免费”必须拆开解释

- **原帖：**[Show HN: ApplyEngine: AI-automated autofill, resume and cover letter](https://news.ycombinator.com/item?id=42205942)
- **日期 / 作者 / 自荐：**2024-11-21；`nishantvyas`；正文介绍扩展、署名 Nishant 并讨论获客，明确自荐。
- **关联：直接同类中的求职垂直产品。** 自动填申请，同时还有简历调整/求职信，后两者不属于 FormFill 当前范围。
- **开头：**“Hi YC Community” 和 “I'm excited to introduce…”；下一段才给出扩展用途。
- **痛点 / 机制：**上传一次简历，AI 填申请并生成相关材料；用 BYOK 解释用户掌握 API 预算。重复输入痛点主要通过该流程暗示，并没有丰富个人经历。
- **证据 / CTA：**链接 Chrome Web Store；作者比较竞品每周 $14.99 并称自身 “FREE for unlimited usage”，这是其自述比较，不是本研究核实的市场价格。求问题与产品反馈。
- **评论疑虑：**只有作者问如何触达大学群体；无独立读者功能反馈。
- **全文/摘要与层级：**根帖文字全文及唯一评论全文，A+B；证据 `42205942`、`42205948`。

**写法分析：**把 BYOK 写成可理解的预算控制，而非接口术语，适合 FormFill。需要改进的是免费表述：读者可能把免产品订阅误读成免模型账单；FormFill 应说“you pay your chosen provider for API usage”，且不虚构竞品价格。也可删掉开头兴奋语气，第一句直接描述做什么。

### H3 · FormWing：两个具体工作流，但会把产品带向不同人群

- **原帖：**[Show HN: FormWing – AI Form Filler for Lead Gen and Testing](https://news.ycombinator.com/item?id=43601142)
- **日期 / 作者 / 自荐：**2025-04-06；`acoyfellow`；“Built this…”及 early access 链接，自荐确认。
- **关联：核心能力相邻，使用意图不同。** 自动分析填表相近，但它是批量外联提交和测试服务，不是用户在当前页拿自己的资料辅助填写。
- **开头：**作者直接说明自己做了网页填表自动化，随即列 lead gen 与 testing 两类用途。
- **痛点 / 机制：**外联希望绕过广告/SEO；开发团队需要快速造测试数据。输入 URL/列表及短 prompt，AI 分析并提交；正文说明队列、边缘服务和 headless browser。
- **证据 / CTA：**具体架构是机制证据，不是成功率证据。作者报 7 天试用、100 次/日、需信用卡，邀请外联、抓取、测试人群反馈，并有前五用户优惠；没有独立用户证言。
- **评论疑虑：**0 条，无从分析反馈。
- **全文/摘要与层级：**文字正文全文、无评论，A+B；证据 `43601142`。

**写法分析：**“输入例子 + 两个操作场景”让人明白在做什么；不过过早写一长串技术栈会稀释使用理由，信用卡和稀缺优惠也增加试用门槛。FormFill 可学输入示例，不宜采用“批量碰触联系表单”的获客叙事，否则读者可能把它理解成群发工具。

### H4 · ScreenComplete：用“闭环”解释和普通聊天工具的区别

- **原帖：**[Show HN: ScreenComplete (app agnostic AI auto-fill)](https://news.ycombinator.com/item?id=41697538)
- **日期 / 作者 / 自荐：**2024-09-30；`sshh12`；同作者首评解释亲自制作 side project，确认自荐。
- **关联：邻近。** 跨应用 AI 输入，不是当前网页结构化字段扫描；不能当成完全直接竞品。
- **开头：**根帖为 GitHub 链接；首评解释自己希望 Slack、Google Docs 等应用都具备通用的聊天模型能力。
- **痛点 / 机制：**不想为每个应用分别购买 AI 功能；作者说其他 copilot 能对截图聊天，自己把 prompt 转为自动补全并输入原应用，完成最后一步。
- **证据 / CTA：**GitHub 源码链接和机制比较；首评没有独立明确的试用请求，也无量化证据。
- **评论疑虑：**只有作者首评，无其他可见反馈。
- **全文/摘要与层级：**根帖元数据和作者首评全文，A+B；证据 `41697538`、`41697539`。

**写法分析：**最值得借鉴的是给出完成任务的最后一步：“模型输出回到原来的工作界面”。FormFill 可据此解释它不同于把资料粘到聊天窗口再手动复制的流程；不过不能擅自讲作者订阅经历，也不要声称跨 Slack/Docs 应用运行。

### H5 · FormFiller bookmarklet：评论要求具体区别，作者用约束回答

- **原帖：**[Show HN: FormFiller – A JavaScript bookmarklet for repopulating forms](https://news.ycombinator.com/item?id=9987365)
- **日期 / 作者 / 自荐：**2015-08-01；`mikeaag`；首评 “I'm Mike, one of the guys behind FormFiller”，明确自荐。
- **关联：邻近的传统回填工具。** 保存并恢复已填表单，不是自然语言/文件到字段，也不是本项目；同名近似不应混淆。
- **开头：**首评先自我介绍、说为解决自身问题而制作，再链接创作过程文章。
- **痛点 / 机制：**根帖说明 bookmarklet repopulation；作者在回复里说明工作用 Firefox、测试人员用 IE、家里用 Chrome，所以不选扩展。跨浏览器约束解释了实现选择。
- **证据 / CTA：**可访问的项目入口和创作文章；作者欢迎改进反馈。本轮没有运行该 2015 年产品。
- **评论疑虑：**有人贴 Lazarus、另一个 form-filler bookmarklet，问具体差异；作者承认早期没有充分调研，并解释跨浏览器、自动更新、表单处理差异。还有读者建议检测“是否同一张表单”。这些是实际可见讨论，不能写成已实施功能。
- **全文/摘要与层级：**已读根帖、作者首评和 Algolia 返回的完整可见评论树；官方根帖和顶层评论直接核验，深层回复使用 B 层全文，因此该层是 A+B 而非全部官方逐条确认。证据 `9987365`、`9987378`、`9987776`、`9989188` 和 `community-live-hn-9987365.json`。

**写法分析：**产品区别最好回答“为什么这种输入和约束导致这个方案”，而不是“比所有产品更好”。FormFill 应准备好回答浏览器 autofill、模板工具和 job autofill 的区别：从用户临时文本/文件直接推断字段，覆盖的任务不同；不声称取代密码管理器，也不承诺可靠处理所有招聘网站。

### H6 / H7 · 保留，但不拿缺失正文凑分析

| 原帖、日期、作者 | 可核验内容 | 缺失与处理 |
|---|---|---|
| [Show HN: I made a Chrome extension that autofills job applications for you](https://news.ycombinator.com/item?id=26668126)，2021-04-02，`rpeng` | 标题“I made”是自荐信号，出站 Kumquat Chrome 商店链接；直接同类中的求职产品；官方根帖确认 | 无正文、无评论。开头仅标题；痛点、产品机制细节、CTA、读者疑虑均无法确认。内容深度 C，帖子存在性 A+B |
| [Show HN: I built an extension that autofills workday job apps locally](https://news.ycombinator.com/item?id=49387614)，2026-08-21，`angelm-tech` | 标题“I built”是自荐信号，Workday 本地填申请；直接垂直同类；官方出站 `restdayextension.com` | 作者首评已 flagged/dead，不能恢复或杜撰。可见一条读者把它称作给招聘领域增加 “slop”，属于低质量自动化的反感，不是功能测评。开头/机制/CTA 缺文，深度 C；帖子和可见评论 A+B |

这些案例和本文主要 HN 案例都不是“高互动成功样板”的同义词。前四篇主要案例只有 0–1 条评论，无法据此声称某种写法能起量。

## 当前发布规则：按目标社区分别处理

### Hacker News

本轮直接读取：[Show HN 官方指南](https://news.ycombinator.com/showhn.html)、[HN 总指南](https://news.ycombinator.com/newsguidelines.html) 以及指南链接的 [dang 发布建议](https://news.ycombinator.com/item?id=22336638)。后者有 2026-03-28 编辑说明。

1. Show HN 要求是作者本人做过、可实际尝试、作者能在帖内讨论的非琐碎项目。早期项目可以，但 landing page、等待名单、文章或市场测试不能代替可运行产品。FormFill 已有源码构建路径，从规则文字看可作为可运行软件展示；本研究未实际构建验证，所以发帖前作者还需确认当前源码可复现。
2. 标题以 `Show HN` 开头。尽量减少注册/邮箱等试用障碍；源码构建和自备模型是本项目真实门槛，应当直说，不能用网站模拟 Demo 假装已经体验过扩展。
3. 解释“是什么、为什么做、不同在哪里”，使用事实和技术细节，少营销。可将介绍置于提交文字或作者第一评论。作者真实缘起尚未提供，草稿不编故事。
4. 不找朋友投票或加油评论，不删除重发；小版本更新通常不足以再做 Show HN。不要承诺“固定几点保证首页”等无证据技巧。
5. 总指南要求帖子由作者自行写作，禁止生成文本，评论部分也禁止 AI 编辑的文本；dang 的补充说明把少量生成、修改和润色也包括在内。因此本文 HN 示例不可直接发布；作者应独立手写，FAQ 也不可复制生成回复。这是当前社区规则，不是要求用户另行批准的流程。

### r/chrome_extensions

本轮读取 [当前规则](https://www.reddit.com/r/chrome_extensions/about/rules/)、[子版介绍](https://www.reddit.com/r/chrome_extensions/about/) 和前 5 条 hot（没有返回置顶帖）。官方 OAuth rules 包含：遵守全站规则/Reddiquette、建设性交流、帖子必须关于 Chrome 扩展、禁止 spam/误导网站/clickbait/跨子版刷屏、禁止盗版/NSFW/拉票，版主保留裁量。

对 FormFill 的操作含义：可以围绕真实 Chrome 扩展征求兼容性和权限反馈；正文明确作者关系、源码安装阶段、BYOK 数据流。不要复制同一广告短时间散发多个子版，不使用“100%安全/适配所有表单”等标题。此次 rules、about 中未见统一 10% 比例、固定自荐日或“必须商店链接”的文字要求；**未见不等于版主预先批准**。实际提交前重新看规则、置顶和可用 flair，使用准确的反馈类 flair（若仍有），不要预填不存在的 flair 名。

### r/SideProject

本轮 [规则接口](https://www.reddit.com/r/SideProject/about/rules/) 返回 `rules: []`，但 [介绍与提交说明](https://www.reddit.com/r/SideProject/about/) **有明确格式要求**：`[Project name] - [Short description]`。其定位是分享项目和获取建设性反馈。不能把空 rules 数组解读成“无限制自荐”。

hot 返回两个置顶帖，其中 [Share your ***Not-AI*** projects](https://www.reddit.com/r/SideProject/comments/1oaq2kx/share_your_notai_projects/) 明确留给非 AI 项目，**FormFill 不适合投在这个置顶线程**；另一个是年度项目回顾，不是必须使用的通用产品发布帖。独立帖按项目名格式提交，不假装普通用户推荐自己的产品，不把 AI 成分藏起来。这里同样没有在已读取材料中出现统一 10% 自荐规则。

## 从案例提炼的写作顺序

1. **标题写输入和输出。** 例如 documents/notes → web forms，而非 “AI productivity revolution”。SideProject 同时遵循项目名格式。
2. **首段给具体任务，但不编作者经历。** “Copying information from a spreadsheet into differently labeled fields…”描述任务即可；不写“我每周浪费十小时”，除非作者有真实记录。
3. **用两三句解释机制。** 读字段上下文、解析用户资料、模型返回 fieldId/value、扩展只写获准控件。HN 可加 fieldId 约束，Reddit 优先写读者可理解的行为。
4. **在链接旁披露试用摩擦。** 源码构建、开发者模式、用户自备 HTTPS 模型 API key、模型账单。不是脚注，也不是虚构商店按钮。
5. **说明谁收到什么。** 本地解析/存储、选定模型接收资料、学习历史可能追加；不要只有 “privacy first”。
6. **把证据与愿望分开。** 可以给真实源码和安装步骤；尚未录制的实测视频、改进中的预览功能、未测准确率不能当作已经有的证据。
7. **收尾问可执行的问题。** 最优先收集非敏感的字段/控件失败例子；也可专问开发者模式 + BYOK 是否让人放弃。避免一次问定价、功能、市场、设计、增长所有问题。

## 英文草稿：全部未发布

### Show HN 标题与作者首帖：仅内部结构示例，禁止直接发布

**状态：未发布。** HN 当前规则禁止 LLM 生成和编辑发布文本。这一节满足内部讨论所需的英文草稿，不构成符合 HN 规则的可粘贴文案。作者应先核实事实，再脱离本稿，以自己的语言从空白手写；不要把本稿当作逐句改写模板。尤其不要复制任何并非作者真实经历的第一人称背景。

**Title concept**

> Show HN: FormFill – Fill web forms from text or documents with your own model

**Opening / first-post structure example**

> I'm working on FormFill, a Chrome extension that maps a description or an existing document to fields on the current web page. It accepts text, .xlsx, .docx, .txt and .md files. The source is here: <项目归档仓库>
>
> The extension scans field labels and nearby context, gives the fields temporary IDs, and asks an OpenAI-compatible model for ID/value pairs. The execution layer accepts only scanned IDs and refuses password, hidden and file inputs. It works with open Shadow DOM; iframe forms and closed Shadow DOM are outside its current scope. Please check the filled values yourself.
>
> This is an early source-build release, not a Chrome Web Store install. Run `npm install` and `npm run build`, enable Developer mode at `chrome://extensions`, and load `.output/chrome-mv3`. You also need your own API key, model name and HTTPS OpenAI-compatible endpoint. API usage is billed by your chosen provider; I don't have measured per-form cost figures to share.
>
> There is no FormFill backend. Files are parsed locally, but the resulting content, your input and the field context go to the model endpoint you configure. Keys and settings are stored locally. Learning mode is on by default: it stores samples locally and can include a matching sample in a later model request. You can turn it off. Please use non-sensitive test data.
>
> A useful next test would be a public form or a minimal example with field types that map incorrectly, without anyone's personal data. The source and installation instructions are available above.

**作者独立写作时需补的真实材料：**用一两句自己的话说明为什么开始做这个项目；目前没有提供这个事实，因此上稿未虚构签证、家庭、求职或每周节约时间的个人故事。HN 实际首帖可压缩篇幅，但不能省略安装/模型费用/数据外发边界。

### Reddit 版本 A：r/chrome_extensions，聚焦兼容和可控性

**状态：未发布；提交前重新检查规则与 flair。**

**Title**

> FormFill: a source-build Chrome extension that fills web forms from text or files (BYOK)

**Body**

> I built FormFill to map information from text or a file to fields on the page you're using. I'm the author, and I'm looking for feedback on form compatibility and the setup process.
>
> You can type a description or import an .xlsx, .docx, .txt or .md file. FormFill reads the page's field labels and context, sends that context and your source information to the model you configure, and applies the returned field values. It supports open Shadow DOM, but doesn't scan iframe forms or closed Shadow DOM. You'll need to check the filled values yourself.
>
> The model can refer only to field IDs assigned during the scan. The extension refuses instructions for unknown IDs, passwords, hidden fields and file inputs. That doesn't make every form safe to use: labels and nearby text can still contain sensitive information, so please start with dummy data.
>
> **Current installation:** it's not on the Chrome Web Store yet. Build the source with `npm install` and `npm run build`, then use Developer mode → Load unpacked in `chrome://extensions` and select `.output/chrome-mv3`.
>
> **Model and cost:** bring your own API key, model name and HTTPS OpenAI-compatible endpoint. Your provider bills the API usage; there isn't a verified per-form cost estimate yet.
>
> **Data flow:** there's no FormFill backend. Parsing happens locally, but parsed file content, your text and form context are sent to your chosen model service. Keys and settings are stored in the extension's local storage. Learning mode is enabled by default; matching locally saved samples can be included in future model requests. Turning learning off stops new samples and stops using saved ones for filling.
>
> Source and build instructions: <项目归档仓库> (MIT).
>
> If you try it, which field or control fails to map correctly? A public URL or a small example with dummy data would help. Please don't post real application details or API keys.

这版主要让扩展用户理解“如何运行、会触碰什么、哪里不兼容”，先提作者身份，不把它伪装成第三方评价。不要为了缩短正文删掉源代码安装和默认学习数据流；如需精简，应优先压缩机制段落。

### Reddit 版本 B：r/SideProject，聚焦资料搬运流程与安装摩擦

**状态：未发布；按子版标题格式。**

**Title**

> FormFill - Fill web forms from notes or documents using your own model API

**Body**

> Copying information from a spreadsheet or a document into differently labeled web fields is the workflow I'm trying to simplify with FormFill. It's a Chrome extension I built, and I'd like feedback on whether the current setup is practical.
>
> The flow is: open a web form → describe the information or import an .xlsx/.docx/.txt/.md file → let your configured model match the information to the fields → check the filled values. This works on existing web forms; it isn't a PDF filler or a form builder.
>
> It's early: there's no Chrome Web Store listing yet. To try it, build the source with `npm install` and `npm run build`, then load `.output/chrome-mv3` through Chrome's Developer mode. You'll also need your own API key, model name and HTTPS OpenAI-compatible endpoint. Model calls are charged by that provider, so BYOK doesn't mean free inference.
>
> There is no FormFill backend. Files are parsed locally, but their parsed content, your input and field context are sent to your selected model service. Keys and settings stay in local extension storage. Learning mode is on by default: it keeps local samples and may send a matching sample in a later request. You can switch it off. Please use non-sensitive test data and review the results; iframe forms and closed Shadow DOM aren't supported.
>
> Source and installation steps: <项目归档仓库> (MIT).
>
> Would source building + Developer mode + bringing an API key stop you from trying a tool for this workflow? I'd like to understand which step creates the most friction before deciding what to improve next.

这版不复用扩展版的所有技术细节，主问题也换成“尝试门槛”，适合 SideProject 的项目反馈语境。它描述的是本次已知阶段，并没有承诺上架日期、现成用户、免费模型额度或一定会改哪项功能。

## 评论 FAQ：事实要点与英文答复参考

以下答复同样未发布。Reddit 可由作者核对后采用；HN 必须由作者根据事实独立手写，不能复制这些回答或让 AI 编辑发言。

| 可能问题 | 应说明的事实 / 英文参考 |
|---|---|
| Is this on the Chrome Web Store? | “Not yet. The current path is to build the source and load `.output/chrome-mv3` in Chrome's Developer mode. There isn't a verified release date for a store listing.” 不虚构 zip 或审核进度 |
| Is it free? | “The source is available under MIT. You bring your own model API key, and your provider charges for API calls. I don't have measured per-form cost figures yet.” 不把开源许可与模型账单混为一谈 |
| Does everything stay on my computer? | “No. Files are parsed locally, but the parsed text, your input and the field context go to the model endpoint you configure. FormFill has no backend of its own. A matching learning sample may also be included if learning is enabled.” |
| What does learning mean? Is it training a model? | “It saves examples locally and can add a matching example to a later request; it isn't model training. Learning is on by default. Turning it off stops saving new samples and stops including old ones in filling requests. Samples are limited to 50 with a 30-day retention window.” 不把关闭说成自动删除既有样本 |
| Are API keys encrypted? | “The key is kept in local extension storage. This project does not claim an encrypted key vault.” 不把本地存储称作加密安全保险箱 |
| Can I use a local model / my ChatGPT subscription? | “The current integration requires an API key and an HTTPS OpenAI-compatible endpoint. It doesn't implement reuse of a logged-in ChatGPT session, and plain HTTP localhost isn't accepted.” HTTPS 自托管理论兼容不等于已有 Ollama 开箱即用体验 |
| Can it read passwords or safely handle IDs/cards? | “The fill executor rejects password, hidden and file inputs. Form context can still include sensitive labels or nearby text, and anything you put in the source text can reach your model provider. Please don't use it on sensitive pages.” 不将拒绝密码填充夸大成不扫描任何敏感元数据 |
| What about prompt injection from the page? | “Page content is marked as untrusted in the model prompt, and the executor accepts only scanned field IDs. Those are constraints, not a guarantee that a malicious page or incorrect mapping cannot cause a problem.” |
| Does it show a preview, flag guesses, or undo? | “The current workflow fills the page and asks you to review it there. A pre-fill review screen, guess labels and one-click undo aren't features I can claim for this version.” 这些是竞品/读者建议，可收集需求，不能先当成功能卖 |
| Does it click Submit? | 按已读执行路径，指令目标为可填控件，不提供点击提交按钮机制。可说 “The filling path doesn't include clicking a submit button. Please check the result on the page.” 同时网页自身可能在字段变化时自动保存/触发逻辑，不承诺对第三方网站绝无副作用 |
| How is this different from normal autofill? | “It uses information from your current text or document and matches it to the page's field context, rather than only reusing a fixed address/contact profile.” 这是机制区别，不代表准确率高于所有工具 |
| Why Gitee? Do you have a GitHub mirror? | 当前配置只指向 Gitee `<项目归档仓库>`；匿名真实浏览器可达性待验。先确认获取路径，不虚构 GitHub 镜像或作者选用 Gitee 的原因 |

## 推广先后顺序与发布前可执行动作

Qwen 复核补充：以下 Reddit/HN 顺序仅适用于海外技术反馈实验，不是人民币商业客群的固定主渠道。中文试点先核查实际候选渠道的当日规则并记录真实任务；统一执行口径见 [计划](46-FormFill_网站、功能、宣传与_SEO_执行计划.md)。HN 指南正文证据见 官方指南 HTML 存档，不能以 `strategy-source-snapshots.json` 中失败的请求条目当正文证据。

1. **先处理父会话识别的发布前问题，再备好可复现试用材料。** 作者从干净 checkout 复核 README 构建步骤，准备一个不含个人信息的网页样例及对应文本/文件，并录制真实扩展操作。明确网页首页默认模拟 Demo 与真实扩展调用的差别。检查 Gitee 英文读者是否能顺利找到 README、构建命令和许可证。此项是建议；子任务未执行构建或录屏，父会话已完成本地构建，但尚无真实扩展录屏。
2. **先发 r/chrome_extensions 独立反馈帖。** 它与当前源码/开发者模式阶段更匹配。使用版本 A，以失败控件为首要问题；作者留时间回答兼容性、权限、模型费用和数据流问题。不找人组织点赞，不同时向多个子版复制广告。
3. **根据真实反馈更新材料，再发 r/SideProject。** 使用项目名格式与版本 B，重点验证安装和 BYOK 的摩擦。如果没有新增观察，不机械重复第一帖；只写实际收到并核实的反馈，不把少量回复包装成市场验证。不进入非 AI 置顶线程。
4. **实际可试、材料清楚时再做 Show HN。** 无需假定必须等商店上架，但当前构建门槛会减少试用，应先把路径跑通。作者独立手写，说明真正创作缘由、资料到字段的机制与 `fieldId` 执行约束，链接可运行源码；不要只提交营销页或模拟 Demo。发布后由作者本人回复问题。
5. **Product Hunt 放后面。** 本轮未研究其当前规则。待有稳定安装入口、真实演示、完整隐私说明与支持渠道，再单独核验 PH 的规则和发布材料；现在不将它列作已经准备就绪的下一站。

本次交付的范围是公开案例与规则研究、机制事实核对和未发布草稿。没有验证竞品实际效果，没有测 FormFill 的浏览器兼容率、准确率或模型账单；下一步最有价值的工作是用非敏感样例验证真实安装体验，再由作者将事实写成符合各社区当前规则的发言。
