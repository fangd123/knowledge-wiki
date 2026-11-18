---
title: "竞品市场表现：规模、流量、社区与 FormFill 的推广选择"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: 项目复盘
series: FormFill浏览器扩展
series_order: 41
tags:
  - 项目复盘
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# 竞品市场表现：规模、流量、社区与 FormFill 的推广选择

核查日期：2026-11-17，Asia/Singapore。接续用户追加的下载、流量、排名与关注度要求。本报告综合两个 `nopcoding/gpt-6.1-sol / high` 子任务及父会话复核；只读公开数据，没有投放、注册、付款、联系或发帖。

**结论：这批样本里，通用 AI/文件填表产品的公开分发多在几十到几千 users；模板、密码管理、求职和自动化替代品已有几十万 users。** 这支持进一步验证“重复录入有价值”，却不能证明本项目的通用填表定位已具备大规模需求，或说明小产品已经失败。最值得借鉴的是可用入口、明确任务、持续教程和使用支持；公开发过一次产品帖不足以证明获客成功。

完整数据：[插件分发、GitHub 与论坛](42-插件分发、开源与社区量化：市场表现补充.md)、[网站流量与社交](43-竞品营销：网站访问量、排名、获客结构与公开关注.md)。所有指标保留平台、对象、时间和来源，不合成一个缺乏依据的“综合热度分”。

## 1. 插件规模：13 个已核实商店条目

下面是本次直接读取 Chrome Web Store 的快照。**users 是商店公开用户指标，不是累计下载、活跃、付费或月新增。** Google 官方[指标说明](https://developer.chrome.com/docs/webstore/metrics)也明确 Users 统计不监测是否活跃。累计商店 downloads 本轮均未公开取得。

| 产品（已核实归属） | 商店 users | 评分 / 评分数 | 对标性质 |
|---|---:|---|---|
| [FillApp](https://chromewebstore.google.com/detail/fillapp-ai-form-filler-da/mdoaagdccddnbjncalegjfemhnlijgbo?hl=en) | 2,000 | 4.2 / 23 | 直接 AI 填表，亦扩至代理任务 |
| [FormFill AI / formfill.cc](https://chromewebstore.google.com/detail/formfill-ai/faepndggpldacododbleilmkbfedkkpo?hl=en) | 13 | 无评分 | 直接同类；与本项目无已核实归属 |
| [Formly](https://chromewebstore.google.com/detail/formly/mhjjgcclpofpmaebpnoicpnoncddcnfp?hl=en) | 34 | 5.0 / 1 | 文件到网页 AI 填表 |
| [ExcelToForm Classic](https://chromewebstore.google.com/detail/file-form-filler/iljfkhjbakinjcdmbipaopchpcaoeffp?hl=en) | 2,000 | 4.2 / 20 | 表格规则填写 |
| [ExcelToForm Automation](https://chromewebstore.google.com/detail/ai-form-filler-with-file/efgjmmnnncomalbheidkacbhiehjhhmo?hl=en) | 29 | 无评分 | 表格填写与页面动作 |
| [XL2Web](https://chromewebstore.google.com/detail/xl2web/jnpdodaepbebnikolajmemkfhcfhomml?hl=en) | 110 | 无评分 | Excel/CSV 映射与流程 |
| [Excel Paste Filler](https://chromewebstore.google.com/detail/excel-paste-filler/kalfomadmfcmflmhccjamjkkgejmanck?hl=en) | 228 | 5.0 / 1 | 表格粘贴和表头匹配 |
| [Filliny](https://chromewebstore.google.com/detail/filliny-ai-form-filling-a/ijhjngflokdbhimpcjhgjkljlfjenjlg?hl=en) | 188 | 2.8 / 6 | 旧通用扩展；官网已偏求职 |
| [Text Blaze](https://chromewebstore.google.com/detail/text-blaze-templates-and/idgadaccgipmpannjkmfddolnnhmeklj?hl=en) | 800,000 | 4.9 / 1.9K | 模板、短语与页面操作 |
| [RoboForm](https://chromewebstore.google.com/detail/roboform-password-manager/pnlccmojcmeohlpggmfnbbiapkmbliob?hl=en) | 600,000 | 4.4 / 3.9K | 密码和身份资料管理 |
| [Simplify Copilot](https://chromewebstore.google.com/detail/simplify-copilot-autofill/pbanhockgagggenencehbnadejlgchfc?hl=en) | 500,000 | 4.8 / 3.9K | 求职填写、简历与追踪 |
| [Magical](https://chromewebstore.google.com/detail/magical-text-expander-aut/iibninhmiggehlcdolcilmhacighjamp?hl=en) | 200,000 | 3.1 / 3.5K | 短语/autofill；当前官网偏医疗代理 |
| [Bardeen](https://chromewebstore.google.com/detail/bardeen-automate-browser/ihhkmalpkhkoedlmcnilbbhhbhnicjga?hl=en) | 200,000 | 4.1 / 452 | 跨应用自动化、抓取与获客 |

父会话用原始正文逐条验证 13 行的 users、评分数标签和版本；Text Blaze 另通过网页工具复核。formfill.cc 搜索缓存曾显示 17 users，而直接页面为 13；本表统一用直接快照，不能用差值报增长/流失。评分是自选样本，1 个 5 星不能证明优于数千评分的产品。Classic/Formly/Automation 用户可能重叠，不相加；同名产品数值也不能算作本项目指标。

## 2. 网站流量与网络排行

**取得的是月访问次数 visits 估计，不是实际 PV 或 UV。** 排名也只是指定提供者、时期与范围中的网站流量排名，不是 Google 关键词排名或产品市场份额。

| 网站 | 数据期 | 月 visits 估计 | Global rank | 来源与比较限制 |
|---|---|---:|---:|---|
| simplify.jobs | 2026-10 | 2.12M，约 212 万 | 25,583 | [Semrush](https://www.semrush.com/website/simplify.jobs/overview/)，Worldwide、All Devices |
| getmagical.com | 2026-10 | 103.81K，约 10.38 万 | 293,884 | [Semrush](https://www.semrush.com/website/getmagical.com/overview/)，同上；与上一行同口径 |
| roboform.com | 2026-09 | 3M，约 300 万 | 19,770 | [Semrush / Exploding Topics](https://analytics.explodingtopics.com/website/roboform.com)，较早月份，设备范围未明示，单列 |
| bardeen.ai | 2026-10 | 170.28K，约 17.03 万 | 260,858 | [SeekTool](https://seektool.ai/ai/bardeen-ai)注明来源 Similarweb 的转载，设备范围未明示；未从 Similarweb 原站核实，不与 Semrush 排名混排 |

Simplify 的流量包含找工作、岗位目录、简历和账号功能；RoboForm 主要产品是密码管理；Magical 的域名已承载医疗业务；Bardeen 包含抓取和获客。不能将这些流量全部视为“想买网页填表插件”的人。

fillapp.ai、formfill.cc、exceltoform.com、fastfiller.aeigs.com、xl2web.com、blaze.today、filliny.io **没有取得可同口径使用的 2026-10 月量或排名**。本轮有 Semrush 404、Similarweb 202 空正文、动态报告未返回数据等不同情况，逐项记录在网站报告中；它们不是网站本身没有访问量的证据。FastFiller 只查其子域，未用整个 aeigs.com 替代；ExcelToForm 域流量无法分摊到同域三款产品。

父会话另读到 [Mindspent 的 Text Blaze 页面](https://mindspent.com/c/text-blaze)给出 2026-09 的 850K visits，并注明 traffic.cv panel estimate；其 #166 是所跟踪 2,210 家 YC 公司的指数位置，**不是全球网站排名**。由于未核验该面板数据方法和原始数据，不纳入上表或商业测算，也不把页面不同口径的品牌词月量填入本站关键词表。

### 来源渠道能看出多少

- Semrush 2026-10 Desktop Journey：Simplify 的 Direct 60.24%、前站点 google.com 6.44%；Magical 的 Direct 68.53%、google.com 5.60%。**前站点域名不是自然搜索渠道占比**，桌面范围不能套全部设备访问量，也不能由 Direct 推导用户留存或首次获客方式。
- Bardeen 的 SeekTool/Similarweb 转载显示 Search Organic 37.92%、Direct 45.46%、Organic Social 2.17%，但设备范围未明、未取得原站报告，只作低于直接来源的方向信号。不能将显示 0.00% 的付费渠道解释成从未投广告。
- 竞品真实 PV、UV、转化、CAC、收入、广告预算、付费与留存本轮均未知。公开数值不能反推出这些后台指标。

## 3. 社交、开源与论坛关注

| 对象 | 本次可核实的量化信号 | 说明 |
|---|---|---|
| [FastFiller GitHub](https://github.com/mauryaankur99/FastFiller) | 官方 API：5 stars、1 fork；v1.0.0 ZIP 资产 download_count 39 | 39 是该资产下载事件，不是独立安装。源码 ZIP、其他渠道和活跃未知 |
| [Filly GitHub](https://github.com/gulan28/filly) | 0 stars、0 forks；releases 为 `[]` | 无 Release 不等于无人下载源码；不与 Filler/Filliny 混同 |
| [Simplify LinkedIn](https://www.linkedin.com/company/simplifyjobs/) | 约 4.4 万 followers | worker 缓存 44,080，父会话后续页面为 44,135；不同缓存，不能把差值当本次增长。公司页有官方转发的用户推荐，但时间节省自述未独立验证 |
| [Bardeen LinkedIn](https://www.linkedin.com/company/bardeen/) | 15,327 followers | 公司级关注，可能涵盖多个产品与招聘；不是插件活跃 |
| [Bardeen Product Hunt](https://www.producthunt.com/products/bardeen) | 2.2K followers；4 次 launches | followers 不是某次 launch 的 votes；父会话复核产品页数值及官网回链 |
| [Text Blaze 官方论坛](https://community.blaze.today/about.json) | 26,231 账户；近 30 日 2,148 posts；active_users_30_days 447 | 父会话重新 GET 核对；均是论坛字段，不是插件 MAU；未审计员工/机器人占比与统计细则 |

Text Blaze 的一个具体[需求讨论](https://community.blaze.today/t/insert-without-closing/69488)在抓取时为 17 views、6 posts，用户希望模板插入后保持可用，支持人员追问场景。其价值是能看到持续使用遇到的问题，不能因为浏览数少就判定支持渠道无价值，也不能把账户总数当自然口碑总量。

### 实际产品发布帖有多热

Reddit 取同日官方只读 API 快照；score 可能模糊化，不等于曝光/准确点赞人数，评论含作者回复。HN 用官方 Firebase 根帖复核。

| 产品 / 帖子 | 发布日 UTC | score / 评论数 | 本次能得出的判断 |
|---|---|---|---|
| [Filler / Reddit](https://www.reddit.com/r/SideProject/comments/1xaoo4w/filler_a_chrome_extension_that_fills_job/) | 2026-10-20 | 8 / 10 | 有读者追问猜测标记、资料是否本地；可借鉴反馈主题，没有安装转化数据 |
| [FastFiller / Reddit](https://www.reddit.com/r/chrome_extensions/comments/1xny1c5/i_build_chrome_extension_that_fills_forms_with_ai/) | 2026-11-04 | 5 / 3 | 有作者介绍和少量回复；不能当爆发式获客案例 |
| [FillApp / Show HN](https://news.ycombinator.com/item?id=44016661) | 2025-05-17 | 3 / 2 | 读者质疑当时的数据留存、模型第三方披露，并指出页脚 GitHub 链接问题 |
| [FillApp 提示词库 / HN](https://news.ycombinator.com/item?id=45746739) | 2025-10-29 | 2 / 0 | 作者发布免费资源；有发帖不等于有可见讨论或有效获客 |
| [FillApp 行业文章 / HN](https://news.ycombinator.com/item?id=45147615) | 2025-09-06 | 1 / 1 | 评论为发帖者自己的内容介绍，不能当独立读者反馈 |

FillApp 的隐私评论是 **2025 年发布时的历史反馈**，不能据此宣布其当前政策仍有同样问题。对 FormFill 的启示是先把模型服务商、数据发送、可选历史和有效源码入口解释清楚；本项目当前还确有已复现的学习采集问题，要先修实现。

新增 HN 核验过程：先按出站 URL 域名通过 Algolia 发现条目，再逐条请求官方 Firebase API核对分数、日期和评论。保存 域名检索样本与官方根帖/评论快照。域名检索未命中不表示品牌没有讨论，因为 text-only、别的域名和评论可能不在该检索范围。

另查到 Text Blaze 两篇 HN 内容分别为 2023 年工程文章 7 分/0 评论与 2021 年招聘经验 1 分/1 评论；它们是公司内容传播，非本轮填表产品发布，不纳入当前品牌热度排序。全量原发布案例仍见 [社区研究](47-FormFill_社区发布案例研究与英文草稿.md)和[量化补充](42-插件分发、开源与社区量化：市场表现补充.md)。

## 4. 这些数据如何改变决策

| 观察 | 对 FormFill 的实际决定 |
|---|---|
| 大规模相邻工具都服务明确的重复任务；直接通用 AI 填表的可见规模较小 | 首先验证一个具体业务任务及复用，不以“整个自动填表市场很大”替代客群选择 |
| 免费/低价工具、免费开源 BYOK 都存在 | 现有能力保持免费更有竞争逻辑；¥699 适配实验检验的是有限交付价值，不能靠竞品 users 推导有人愿买订阅 |
| FillApp/ExcelToForm 有安装、任务指南；Text Blaze 有模板和持续论坛支持 | 首批投入真实安装、可下载例子、任务教程、有效支持与兼容性说明。先让使用者完成任务，再扩大内容 |
| 精选 Reddit/HN 帖大多只有个位数分数和少量评论 | 把社区当定向反馈与初始试用入口，预期设为拿到失败样例、找到复用者；不预算“一发帖就几千安装” |
| 成熟品牌社交关注多，但全域产品范围更宽 | 借鉴具体叙事和支持组织方式；不照抄其完整功能、渠道比例、预算或发文数量 |
| 直接竞品月量多不可公开核实 | 保留市场假设，不用估算数字决定重投入；优先从自己的试用任务、安装配置和真实付款获得证据 |

因此优先采用 [产品策略](38-FormFill_产品差异化、定价与推广决策.md)中的记录主体明确、人工可复核和可靠获取路径；再按实际阻力选择映射复用或来源检查。完整来源追溯不作为结构化复用的前置。两类任务的频率、成本与付费仍待验证，市场快照不能决定客群或价格，来源定位和业务场景也不能说“唯一”。具体待办见 [执行计划](46-FormFill_网站、功能、宣传与_SEO_执行计划.md)。

## 5. 后续如何持续跟踪

- **每月一次**保存商店 users/评分/版本快照及各流量来源最新完整月；保持同商店 ID、同域范围、同提供者和设备口径。商店舍入与缓存可能掩盖小变化，不能据快照差值算精确月新增。
- **每周少量关注**新出现的使用问题、真实兼容失败和发布内容；记录原帖日期、抓取时间、作者关系、互动对象和明确反馈。不要用模糊品牌搜索结果数充当全网声量。
- **本站每 28 天复盘**真实相关搜索 → 安装入口 → 配置成功 → 首个正确任务 → 实际复用 → 适配询问/实付。早期用自愿测试者、商店聚合指标和透明网站统计补足，不悄悄增加扩展遥测。
- **需要购买数据时**再以“能否取得直接竞品、同月、同口径报告”评估一个研究工具周期；不是看到 NA 就盲目同时订多家。付款前先确认覆盖，不能假设付费一定能估算小站。

本轮没有验证竞品真实填表效果、收费收入、广告效果、本站线上域名及真实搜索量；所有未知都保留为未知，没有拿 0 替代。工作完成的是公开市场快照与决策支持，不是对任何竞品经营结果的审计。
