---
title: "FormFill 产品收尾与交接：Qwen 独立审查（第 1 轮）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 21
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# FormFill 产品收尾与交接：Qwen 独立审查（第 1 轮）

- 日期：2026-11-17T17:00:32.009Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md与本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜docs/store-listing.md（开头中文段 + 中文详细描述末句）**
问题：中文正文仍现在时宣称“国内用户主要通过 Microsoft Edge 扩展商店分发／获取本扩展”，与顶部“从未提交、未上架”自相矛盾；英文详细描述无此句，双语不对等（英文开头已用 “was to be”，中文缺“原计划”限定）。
改：删除末句或改为“（历史计划，未执行）”；开头中文补“原计划”并加粗标注为历史；中英正文保持同一口径。

**C2｜docs/release-checklist.md（正文全部小节）**
问题：仅标题与引言标了历史，正文仍是可执行门禁：状态词“待域名/待账户/待真实环境/待决策”、小节“网站与 SEO（域名确定后执行）”、未勾选复选框、“发布 0.2.0 时同步 bump package.json”。未来接手者会读成待上架/待域名待办。
改：状态统一为“冻结时未完成（已取消）”，小节改“冻结时未完成的网站与 SEO 事项（历史）”，版本条改为“冻结时 package.json 仍为 0.1.0，0.2.0 未发布且不再 bump”，删除执行祈使句。

**C3｜docs/seo-preparation.md（“已完成（本地）”与未执行项表）**
问题：①事实矛盾——仍写“正文由 React 客户端渲染（见 S02 缺口）”，与同文 S02 预渲染已完成、overview §4“无 JS 直读可见主文案”冲突；②“需要什么”列与 S05“此后按真实问题补内容”仍是未来指令。
改：更正为“构建后已静态预渲染（S02）”；列名改“原计划前提（已取消）”，内容改过去时描述，删除继续补内容的指令。

**C4｜冻结标注覆盖不全（privacy-policy 双语、website/ 四页、AGENTS.md 等）**
问题：release-checklist 自身记录“隐私政策/商店草案写‘适用于计划发布的 0.2.0’”，但本次 diff 未包含 docs/privacy-policy.md / .zh-CN.md、website/ 页面、AGENTS.md、user-personas、demo-pack。这些是用户与代理直接读到的入口，可能残留发布/上架/继续推进口径。
改：逐一核对并同步归档标注（尤其 AGENTS.md 若含继续产品化的工作指令须改为归档维护口径）；未核对前不宜宣称“文档已统一标记为历史”（CHANGELOG 该句需限定范围）。

**C5｜新增文件与 diff 不一致**
问题：所有文档顶部与索引均链接 docs/product-wrap-up.md，overview/wrap-up 还链接 docs/execution/2026-11-18.md，但 diff 中无这两个文件的新增记录。
改：提交前确认两文件确已纳入本次变更，否则全仓顶部链接与收尾台账链接失效。

## 非阻塞建议

- **D1**：双语 README 仍称总览含“roadmap/规划”，且“this release does not revalidate／本次…不重新认证”暗示存在发布；建议改“历史计划”“档案中的预设未重新验证”。
- **D2**：overview §5“已验证”未像 wrap-up 那样注明“历史证据、本轮未重跑”，建议补一句，避免被读成本轮复验。
- **D3**：wrap-up §5“13 个服务商预设”含自定义项；与 overview 统一为“13 个选项（12 预设 + 自定义）”。
- **D4**：双语 README 均未提 MIT，而收尾以 MIT 为保留口径；确认 LICENSE 存在后补一行指向它。
- **D5**：store-listing“Historical release checklist”各条仍为祈使句（发布隐私政策到公开 URL、准备截图与版本包等），建议改过去时并标“已取消”。
- **D6**：overview §6“当前免费”“推广顺序…上线后观测”仍现在时，建议加“冻结前口径”限定。

## 已核对范围

双语 README 全文与 diff 的对等性；wrap-up 与 overview、CHANGELOG、release-checklist、store-listing、seo-preparation 的状态口径；日期（2026-11-18 冻结、11-17/11-16 研究、0.1.0 为 2026-09-26）；版本事实（0.1.0 / 0.2.0 未发布）一致；无“已上线/客户/收入/复购/市场验证”表述；停止原因未写成“市场必然为零”或“技术失败”；已验证/未验证分区、合格复现样本为 0、冻结清单与重启条件；7 个 diff 文件的改动方向。

## 未核对范围

package.json、LICENSE、源码与测试实跑（130 通过/1 跳过、lint 7 警、143 key、Chromium 159、模型名、2.5s/22s/7.7s 等均未复跑）；docs/execution/* 全部证据；architecture、privacy-policy 双语、user-personas、demo-pack、research/*、website/ 四页、AGENTS.md；Gitee 仓库与 Issues 现状、外链可达性；商店 users 数字。本次未运行任何命令，未新增事实或市场数据。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 2 轮）

- 日期：2026-11-17T17:08:17.081Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜版本口径自相矛盾：docs/privacy-policy.md、docs/privacy-policy.zh-CN.md 顶部；website/src/SetupGuide.tsx、ExcelGuide.tsx、PrivacyPolicy.tsx；对照 docs/seo-preparation.md、CHANGELOG.md**
问题：新标注写“描述归档的 `0.1.0` 代码快照”，但正文描述的映射复用、Excel 单记录、reasoning_effort、删除学习采集均属 CHANGELOG 的 `0.2.0 (Unreleased)`；seo-preparation 仍写“指南内容对应 0.2.0 实际行为”，网站指南却写“对应归档的 0.1.0”。隐私政策还写“存储升级到 v5”，README/总览为 v6。读者会误以为归档代码等于 2026-09-26 的 0.1.0 行为。
改：统一为“归档代码＝package.json 版本号仍为 0.1.0，包含未发布的 0.2.0 变更；0.2.0 从未发布”，并核对更正 v5/v6 表述（只改文档，不改代码）。

**C2｜docs/user-personas.md §7、§0、§3、§5、§6**
问题：本轮只改引言与一行表格，正文仍是可执行的产品化指令：§7 标题“渠道优先级（当前可执行）”“留到上架或有真实案例后”；§0“扩大范围要等上架和上手流程改善”；§3“等我们提供托管/引导方案”；§5“F08…需高频证据才立项”；§6 招募配额、¥699 门槛为现在时。与冻结决定冲突，会误导接手者继续推进。
改：小节改“历史渠道与访谈研究（不再执行）”，把等待上架、继续招募、立项、报价的祈使句改为过去时描述，并注明重启须先满足 wrap-up §8。

**C3｜docs/product-overview.md §1（MIT 口径）**
问题：无条件写“MIT 许可”，与 wrap-up §3、收尾台账“仓库根目录未发现 `LICENSE` 文件”冲突；网站页脚已谨慎写作“MIT 许可口径”。
改：改为“既有文档声明 MIT；冻结时根目录未见 LICENSE，再分发前须核对托管平台许可与版权主体并补齐（见 wrap-up §3）”。

**C4｜新增文件未出现在本次 diff**
问题：diff 全为修改，无 docs/product-wrap-up.md、docs/execution/2026-11-18.md、2026-11-18-layout-check.json、2026-11-18-a11y-check.json、product-wrap-up-qwen-review.mjs/.md 的新增记录，而全仓顶部提示与索引均依赖前两者（第 1 轮 C5 未闭环）。
改：提交前确认上述文件确已暂存并随本次提交，否则归档入口链接整体失效。

**C5｜docs/execution/2026-11-18.md 与本轮 a11y 证据标注**
问题：台账写“第二轮审查记录在后续章节中保留”，但文中无该章节，提交结果仍是“拟提交”；diff 中 w06-a11y-check.mjs 只参数化输出

> 记录说明：第 2 轮网关响应在此处被输出上限截断，原始响应没有完整保存。后续轮次尚未恢复 C5 剩余文字，因此本节只能作为截断片段保留；“历轮原始结论完整保存”的表述不成立，已在执行台账中如实修正。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 3 轮）

- 日期：2026-11-17T17:17:50.626Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜证据文件与“本轮重新验证”表述不符**
文件：`docs/execution/2026-11-18-layout-check.json`、`2026-11-18-a11y-check.json`；`docs/execution/2026-11-18.md`「验证」；`docs/product-overview.md` §4。
问题：两份以 11-18 命名的证据，`checkedAt` 实为 `2026-11-17T17:11:23Z / 17:11:28Z`；a11y 的 `note` 仍是脚本默认的“W06 guide pages + nav change”，未提归档横幅。台账与总览却称“2026-11-18 收尾时加入全站归档横幅并重新验证”。以现有材料无法证明这两次检查跑在含横幅的构建上，且日期自相矛盾。
改：如实写出证据的真实 `checkedAt` 与 note 来源（沿用 W06 脚本默认文案），说明命名与时间戳差异；若确为横幅后重跑，需重跑并生成带横幅说明与新时间戳的证据，否则把 §4 与台账改为“沿用 11-17 布局/axe 证据，横幅后仅 build 通过”，不得表述为本轮重新验证。

**C2｜被引用的审查记录与脚本缺失，且预写第三轮结论**
文件：`docs/execution/2026-11-18.md`「Qwen 审查」。
问题：正文称结论保存于 `docs/execution/product-wrap-up-qwen-review.md`、复现脚本 `scripts/product-wrap-up-qwen-review.mjs`，但本次 diff 无这两个文件（第 2 轮 C4 未闭环）；同时写“第三轮最终结论统一保存在…”，而第三轮尚在进行，属预写未发生事实。
改：确认两文件随本次提交纳入，否则删除引用；把第三轮表述改为“第三轮结论待补”，不得预先记录结果。

**C3｜仍残留“待上线/待访谈/被阻塞”口径**
文件：`docs/product-overview.md` §5 未验证列表（“S04，待上线”“M01/G2，待访谈”“跨站点隔离待有头环境人工验证”）；`docs/seo-preparation.md` 引言“仍被外部条件阻塞的事项”、S03/S04 单元格仍为现在时执行指令（“确定首要市场后逐簇记录 SERP”“域名上线且有权限时验证 GSC…28 天窗口”）。
问题：与冻结决定冲突，接手者会读成待办；第 1 轮 C3 只改了 S01/S02/S05。
改：统一为“冻结时未验证/未执行，已取消”，S03/S04 改过去时描述原计划；引言删“仍被阻塞”；“待采样”改为“冻结时未采样，不再采样”。

**C4｜复盘把合成技术证据抬到 E3/E4**
文件：`docs/retrospective-and-methodology.md` §5 结论句、§0。
问题：该文自定义 E3=真实任务观察/授权页面演示、E4=同初始态对照与真实使用留痕，却称“技术证据可以很快到 E3/E4”。而收尾报告与画像明确：真实用户任务复现样本为 0、对照均为合成数据。这与自身阶梯定义冲突，等于把研究/合成结果说成用户行为与可控价值证据；§0“形成了一套能及时回答三个问题的机制”亦超证据（G1–G4 从未执行）。
改：明确 FormFill 只有合成环境技术验证，市场证据停在 E1；如需保留该句，须区分“技术验证阶梯”与“市场证据阶梯”，并注明合成对照为乐观上限。§0 改为“已设计门槛但尚未执行，商业问题仍未回答”。

## 非阻塞建议

- **D1**：双语 README 均未提许可；建议补一行指向收尾报告 §3 的“声明 MIT、根目录未见 LICENSE”。wrap-up“已授予的权利”宜改为“不主张撤销此前依 MIT 声明可能已授予的权利，实际授权状态待核”。
- **D2**：`store-listing.md` 中文末段有“仅用于冻结前样机…原计划 Edge…从未提交”，英文对应句缺此限定，建议补齐对等。
- **D3**：`AGENTS.md` 目录约定仍写 content.ts“按需学习监听”、background“学习样本”，与“已删除提交采集路径”不符，建议改为“旧记录本地查看/删除”。
- **D4**：CHANGELOG `0.2.0` 内“真实模型/浏览器验证仍待可用环境”“sitemap 与 canonical 仍待生产域名”属历史条目，建议加括注（后续已完成／已取消）。
- **D5**：复盘 §1“已经验证的”缺少“历史证据、本轮未重跑”一句，建议与 wrap-up/overview 一致。
- **D6**：wrap-up §9 将 release-checklist 描述为“发布门禁”，建议改“历史缺口清单（不再执行）”。

## 已核对范围

双语 README 全文对等性与 diff；wrap-up、overview、release-checklist、store-listing、seo-preparation、CHANGELOG、隐私政策双语、user-personas、demo-pack、AGENTS.md、website 源码改动的状态口径；无“已上线/客户/收入/复购/市场验证”表述；停止原因未写成市场为零或技术失败；冻结/保留清单与重启条件；0.1.0 与 0.2.0 未发布、v5/v6 迁移口径一致；13 选项（12 预设+自定义）、7 国内预设、36 组布局计数；两份新证据 JSON 的字段与时间戳；前两轮 C1–C5 的闭环情况。

## 未核对范围

未运行任何命令。package.json、LICENSE、源码与测试实跑（130/1、lint 7 警、143 key、Chromium 159、模型名、2.5s/22s/7.7s）；除两份新 JSON 外的 `docs/execution/*`；research/*、architecture、website 实际渲染与横幅可见性；Gitee 仓库/Issues 与外链可达性；商店 users 数字；提交与推送结果。未新增事实或市场数据。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 4 轮）

- 日期：2026-11-17T17:25:10.745Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/architecture.zh-CN.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜验证门编号在两份文档中含义冲突**
文件：`docs/product-overview.md` §6「冻结前已完成（G0/G1 技术部分 + 部分 G2 能力）」、§5「M01/G2」；`docs/retrospective-and-methodology.md` §0「G1–G4 没有执行」、§4 五道门定义。
问题：复盘把 G1 定义为「用户门（访谈/任务观察）」并声明 G1–G4 未执行；总览却用同一组编号宣称 G1 部分、G2 部分已完成（实为执行计划里的技术阶段）。接手者会读成「用户门/价值门已部分通过」，正是本轮要防的「技术验证抬成市场证据」。
改：在总览 §6 明确注明该 G 编号来自 `execution-plan.md` 的技术阶段划分，与复盘五道门不是同一体系（或改用 F/W/S 表述），并加一句「用户门、价值门、购买门、重复门均未执行」。

**C2｜网站证据的 note/日期是脚本写死的常量，却被当作「含横幅重新验证」的佐证**
文件：`docs/execution/scripts/f03-website-layout-check.mjs`、`w06-a11y-check.mjs`（diff 中 `note`、`checkedDateLocal:'2026-11-18'`、`timezone` 均为硬编码）；`docs/product-overview.md` §4「`note` 明确记录检查对象包含共享归档横幅」；`docs/execution/2026-11-18.md`「验证」。
问题：note 是作者写死的字符串，不是检测结果，不能证明受检构建含横幅；`checkedDateLocal` 也非由运行时间推导，任何后续复跑都会被打上 11-18 标签。同时 wrap-up §3 与复盘 §1 的「本收尾没有重新运行产品测试／均为冻结前历史证据」与本轮确已重跑网站布局/axe 的表述边界不清。
改：§4 与台账改为「本轮重跑网站布局与 axe（时间戳 17:18:59Z/17:19:04Z，SGT 11-18），横幅存在性由构建产物与源码改动佐证，note 为脚本内人工标注」；脚本日期改为运行时推导或注明为人工常量；wrap-up/复盘统一为「扩展测试未重跑，网站静态检查本轮重跑」。

## 非阻塞建议

- **D1**：`product-wrap-up-qwen-review.md` 第 2 轮记录在「w06-a11y-check.mjs 只参数化输出」处截断，缺 C5 余项与核对范围，但台账称「历轮原始结论保存」。补全或注明「第 2 轮输出被截断，仅存片段」。
- **D2**：复盘 §5 引入未定义的「T3/T4 类技术证据」。给出定义或删除，避免读者自造第二套阶梯。
- **D3**：`user-personas.md` §1「¥699 仍只是…报价实验」、§2「真实决策者仍要单独核实」、§6「一律按方向重估 §7 的**现行条款执行**」仍是现在时指令。改为「冻结前口径／已取消」。
- **D4**：`AGENTS.md` 允许「缺陷修复」，README 英文写「no further feature…work is planned」。建议限定为「仅为档案可复现性的修复，不构成产品维护」。
- **D5**：总览 §3 商店 users（800,000/500,000/2,000/34）、「多数低价同类是直接填」、§1「143 个双语 key」均为历史研究/未复核数值，建议逐条标注来源与采集日期。
- **D6**：双语商店草案未提 `reasoning_effort`，与 README/隐私政策的 v6 行为不完全对齐；作为历史草案可保留，但宜加一句「草案未覆盖后加入的思考强度设置」。

## 已核对范围

双语 README 全文逐段对等（归档声明、LICENSE 段、13 选项表、v6、安装/路线图/支持章节）；wrap-up、overview、release-checklist、store-listing、seo-preparation、CHANGELOG、隐私政策双语、personas、demo-pack、AGENTS、website 源码的状态口径；无「已上线/已部署/客户/收入/复购/市场验证」表述；停止原因未写成市场为零或技术失败；0.1.0 与「0.2.0 未发布」一致；13=12 预设+自定义、7 国内预设；两份新 JSON：布局 4 页×9 断点=36 条、`allPass=true`，a11y 4 页 0 violations，UTC 17:18:59/17:19:04 换算 SGT 11-18 01:18–01:19 与所述一致；锚点 `#3-冻结时的技术状态`、`#8-重启前的最低条件` 与标题匹配；第 3 轮 C1–C4、D1–D6 的处理情况（C3、C4 及 D1–D6 已闭环，C1、C2 部分闭环见上）。

## 未核对范围

未运行任何命令。`package.json`、`LICENSE`、源码与测试实跑（130/1、lint 7 警、143 key、Chromium 159、模型名、7.7s/22s/2.5s）；除两份新 JSON 外的 `docs/execution/*`（含 2026-11-17 台账、w05-*、s02-*）；`research/*` 全部结论与商店数字；website 实际渲染与横幅可见性；相对链接可达性（79 条声明未复核）；Gitee 仓库/Issues 现状；提交与推送结果。未新增事实或市场数据。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 5 轮）

- 日期：2026-11-17T17:29:47.174Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/architecture.zh-CN.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜docs/product-overview.md §5「已验证」引言（与 §4、收尾报告 §3、复盘 §1、执行台账冲突）**
问题：引言写“以下均为冻结前的历史证据；本次收尾只整理文档，没有重新运行产品测试”，但同文 §4 明确写“2026-11-18 收尾时加入全站归档横幅并重新验证”，收尾报告 §3 写“网站静态布局与可访问性检查在本轮单独重跑”，复盘 §1 写“只重跑了网站的静态布局与 axe 检查”，台账还记录了网站 typecheck/build 与两份新证据。此外本轮实际改动了 website 源码与两个检查脚本，“只整理文档”不成立。§5 又把网站布局/axe 全部归为 s02-*.json 历史证据，读者无法判断哪部分是本轮重跑。
应如何改：把引言改为“扩展单元/集成与真实模型、真实浏览器 E2E 均为冻结前历史证据，本轮未重跑；网站静态布局与 axe 于本轮重跑（证据见 §4），网站源码与检查脚本本轮有改动”。网站条目区分“历史 s02-* 证据”与“本轮 2026-11-18-layout/a11y-check.json”，与收尾报告、复盘、台账用同一句口径。

**C2｜docs/execution/2026-11-18.md「验证」「Qwen 审查」与 product-wrap-up-qwen-review.md 第 4 轮的时间戳血缘**
问题：归档的第 4 轮审查引用的是 `17:18:59Z / 17:19:04Z` 两份证据，而随本次提交的 JSON 为 `17:26:57.120Z / 17:27:02.478Z`。台账只列最终时间戳，未说明第 4 轮整改后又重跑过一次网站检查并覆盖证据，未来审计者会认为审查记录与证据互相矛盾，或误判证据被手工改写。
应如何改：在台账「验证」补一句事实说明：第 4 轮整改后重跑网站布局与 axe，证据由 17:18:59Z/17:19:04Z 版本更新为 17:26:57Z/17:27:02Z 版本，审查记录中的旧时间戳属当时快照，不改写历史审查原文。不新增其他结论。

## 非阻塞建议

- **D1**：双语 README 权限段“`storage` 保存设置和学习样本 / saves settings and learning samples”与“自动学习已暂停”并置易误读，建议统一为“旧学习样本 / legacy learning samples”，与隐私政策一致。
- **D2**：store-listing 末段中英不完全对等：英文提“Chrome Web Store 与 Edge 均未提交”，中文只提 Edge；中文有“仅用于冻结前样机”限定而英文以独立句表达。建议两句一一对应。
- **D3**：website/README.md 仍写“matching the current extension behavior”“Keep it in sync”，建议改“归档快照行为”，保留同步提示但注明属档案维护。
- **D4**：首页 hero 删去了“需要自备模型接口，API 调用按你的服务商计费”。请确认首页“数据流与限制”区仍完整披露 BYOK、服务商计费与外发范围，避免归档版首页弱化隐私口径（本轮未核对该未改动区块）。
- **D5**：product-overview §2 标题“全部已实现并通过本地测试”、§3 第 4 条“双语 key 对等由测试保证”建议加“冻结前历史测试”限定；收尾报告 §9 索引可补隐私政策与双语 README 入口。

## 已核对范围

双语 README 逐段对等（归档声明、LICENSE 段、13 选项表、v6、安装/历史路线图/支持章节）；收尾报告、总览、发布清单、商店草案、SEO 文档、CHANGELOG、隐私政策双语、用户画像、演示包、AGENTS、architecture、website 源码的冻结口径；未见“已上线/已部署/客户/收入/复购/市场验证”表述；停止原因未写成市场为零或技术失败；冻结/保留清单与重启条件；0.1.0 与“0.2.0 未发布”一致；13=12 预设+自定义、7 国内预设；两份新 JSON：布局 4×9=36 条、a11y 4 页 0 violations、`allPass=true`、逐项 `archiveBannerObserved=true`，UTC→SGT 换算与文中一致；脚本已改为运行时推导日期并在 DOM 实测横幅（第 4 轮 C2 已闭环）；总览已改用 F/W/S 编号并声明与五道门不同体系（C1 已闭环）；复盘 §5 已把市场证据限定在 E1、删除 T3/T4；锚点 `#3-冻结时的技术状态`、`#8-重启前的最低条件` 与标题匹配；新增文件均已进入 diff。

## 未核对范围

未运行任何命令。`package.json`、`LICENSE`、源码与测试实跑（130/1、lint 7 警、143 key、Chromium 159、模型名、7.7s/22s/2.5s）；除两份新 JSON 外的 `docs/execution/*`（含 2026-11-17 台账、w05-*、s02-*）；`research/*` 全部结论与商店 users 数字；website 实际渲染、横幅可见性与未改动区块文案；相对链接可达性（81 条声明未复核）；Gitee 仓库/Issues 现状；提交与推送结果。未新增事实或市场数据。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 6 轮）

- 日期：2026-11-17T17:32:40.307Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/architecture.zh-CN.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜docs/execution/2026-11-18.md「Qwen 审查」第五轮条与「验证」章**
问题：第五轮条写“上述各项已整改并复跑静态验证”，但随本次提交的两份证据 `checkedAt` 为 `17:26:57.120Z` / `17:27:02.478Z`，早于第五轮审查时间 `17:29:47.174Z`；台账另一处已明确这两份文件是**第四轮**整改后重跑并覆盖的结果。因此“第五轮后复跑静态验证”指的是相对链接/JSON 解析/typecheck/build，还是布局与 axe，材料无法判定，未来审计者会读成布局/axe 在第五轮后又跑过一次（与本轮反复强调的“不把历史检查写成重跑”同类风险）。此外台账停在“第六轮等待复审”，按现状归档会留下一条未闭合的审查状态。
应如何改：第五轮条改为具体检查名，并补一句“布局/axe 证据仍为第四轮整改后的 17:26:57Z/17:27:02Z 版本，第五轮后未重跑”；提交前把第六轮结论与处理如实补记，不预写尚未发生的结果。仅此一处，其余未发现阻塞。

## 非阻塞建议

**D1｜website/src/App.tsx 首页 BYOK 披露（本轮未核对）**：diff 同时删除了 hero 的“需要自备模型接口，API 调用按你的服务商计费”和安装区的“你自己的模型 API Key（本地 Ollama 可不填 Key）”。台账称“数据流与限制”区仍完整披露，但该区原文不在所给材料内，无法核实。建议在台账附该区原文片段佐证，或在安装区补回一句 BYOK/计费提示。

**D2｜证据可复现信息不足**：两份 JSON 只记录 URL、时间戳与横幅命中，未记录浏览器与版本（axe 脚本用本机品牌 Chrome，布局脚本用 Playwright chromium）、受检构建来源与生成命令（`LAYOUT_OUT`/`A11Y_OUT`、4175 预览服务启动方式）。建议在台账「验证」补齐命令与环境。

**D3｜docs/demo-pack/README.md「验收对照」**：仍保留“视频若剪辑需标注提前配置与加速”，属发布素材语气，建议标为历史或删除。

**D4｜docs/product-overview.md §3 第 3 条**：“实测换记录复用约 2.5 秒”未加历史限定（§2/§5 已限定），建议补“2026-11-17 历史实测”。

**D5｜docs/user-personas.md §2/§3**：“为什么会选 FormFill”仍是现在时卖点句式，虽在历史标题下，建议统一为“冻结前假设”。

## 已核对范围

双语 README 逐段对等（归档声明、LICENSE 段、13 选项表、v6、legacy 样本措辞、安装/历史路线图/支持章节）；收尾报告、总览、复盘、发布清单、商店草案、SEO 文档、CHANGELOG、隐私政策双语、画像、演示包、AGENTS、architecture、website 源码的冻结口径；未见“已上线/已部署/客户/收入/复购/市场验证”表述；停止原因未写成市场为零或技术失败；0.1.0 与“0.2.0 未发布”、v5/v6 迁移一致；13=12 预设+自定义；两份 JSON：布局 4×9=36 条、axe 4 页 0 violations、逐项 `archiveBannerObserved=true`、`allPass=true`，UTC→SGT 换算与文中一致；脚本已运行时推导日期并在 DOM 实测横幅；总览 §5 已区分历史扩展证据与本轮网站检查（第 5 轮 C1 闭环）；台账已补时间戳血缘（第 5 轮 C2 闭环）；商店草案末段中英已对等；website/README 已改归档口径；锚点 `#3-冻结时的技术状态`、`#8-重启前的最低条件` 与标题匹配；新增文件均已进入 diff。

## 未核对范围

未运行任何命令。`package.json`、`LICENSE`、源码与测试

> 记录说明：第 6 轮网关响应在此处被输出上限截断，原始响应没有完整保存；以上仅为片段，未补造缺失文字。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 7 轮）

- 日期：2026-11-17T17:36:27.587Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/architecture.zh-CN.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜`docs/execution/2026-11-18.md`「Qwen 审查」＋`docs/execution/product-wrap-up-qwen-review.md`（审查状态未闭合）**
问题：台账审查条止于第 6 轮，且六轮结论全部为“需修改”，没有任何一条说明本轮（第 7 轮）审查已发生、结论为何、是否处理。按现状归档，接手者只能读到“连续六轮被要求修改、随后不知结果”的悬空状态；这与第 6 轮 C1 要求“提交前如实补记当轮结论、不预写”的标准是同一条。
应如何改：提交前把第 7 轮（及若有的后续轮）结论与逐项处理如实补记进台账与审查档案；若届时仍未取得同意提交，就明写“审查进行至第 N 轮，尚未取得同意提交，最终结论以审查档案为准”。不得预写尚未发生的结论，也不得把台账写成审查已通过。

**C2｜`docs/execution/product-wrap-up-qwen-review.md` 第 6 轮记录被截断，但台账只声明第 2 轮截断**
问题：档案中第 6 轮正文在“## 未核对范围 / 未运行任何命令。`package.json`、`LICENSE`、源码与测试”处中断，缺后续文字，且没有像第 2 轮那样的截断说明；而台账写“其中第 2 轮响应被输出上限截断，文档已明确标注为片段而非完整原文”，等于宣称其余各轮完整。这是可核对的事实不符，属本项目反复强调的“证据完整性口径”问题。
应如何改：在第 6 轮记录末尾补与第 2 轮同款的截断标注（说明原始响应未完整保存、仅为片段），并把台账该句改为“第 2、6 轮响应被输出上限截断，均已标注为片段”；如能重取完整原文则补全，但不得改写已有历史正文或补造未保存的内容。

## 非阻塞建议

**D1｜两份证据的 `environment` 字段来源**：`executablePath` 在两个脚本中都是写死的字符串，`browser` 虽由 `browser.version()` 取得，但布局脚本的 `chromium.launch` 参数未完整出现在 diff 中；两份 JSON 同报 `Chromium 157.0.8103.79` 与同一 Chrome 路径，与第 6 轮 D2 所述“布局用 Playwright chromium、axe 用本机 Chrome”是否一致无法从材料判定。建议由实际启动配置推导该字段，或注明其为人工常量。

**D2｜链接检查覆盖时点**：台账“相对链接 PASS（17 文档、85 链接）”标注为第五轮整改后复核，而第六轮又改了多份文档。建议注明该检查的覆盖时点，或在最终文本上重跑一次。

**D3｜链接文字仍为非历史称谓**：双语 README 权限段与总览部分入口仍写“商店介绍与申报建议 / store submission copy and declarations”，目标文档已改名“历史商店上架草案”。建议统一加“历史/草案”限定。

**D4｜`docs/release-checklist.md` 标记语义**：把“`package.json` 保持 `0.1.0`”这类既成事实标为 `[已取消]`，与“取消某项动作”混用。建议事实项用中性标记，`[已取消]` 只用于原计划动作。

**D5｜复盘 §1 资产列**：“130 项测试记录”“Playwright 浏览器 E2E”可补 2026-11-17 日期，与总览、收尾报告的历史限定完全对齐。

## 已核对范围

双语 README 逐段对等（归档声明、LICENSE 段、13 选项表、v6、legacy 样本、安装/历史路线图/支持章节）；收尾报告、总览、复盘、发布清单、商店草案、SEO 文档、CHANGELOG、隐私政策双语、画像、演示包、AGENTS、architecture、website 源码的冻结口径；未见“已上线/已部署/客户/收入/复购/市场验证”表述；停止原因未写成市场为零或技术失败；扩展历史证据与本轮网站静态检查已分离；0.1.0 与“0.2.0 未发布”、v5/v6 一致；两份 JSON：布局 4×9=36 条、axe 4 页 0 violations、逐项 `archiveBannerObserved=true`、`allPass=true`，UTC 17:34:09/17:34:14 → SGT 11-18 01:34 与文中一致，且晚于第 6 轮审查时间；台账时间戳血缘链完整；第 6 轮 C1 与 D1–D5 已闭环；锚点 `#3-冻结时的技术状态`、`#8-重启前的最低条件` 与标题匹配；新增文件均在 diff 内。

## 未核对范围

未运行任何命令。`package.json`、`LICENSE`、源码与测试实跑（130/1、lint 7 警、143 key、Chromium 159、模型名、7.7s/22s/2.5s）；除两份新 JSON 外的 `docs/execution/*`；`research/*` 结论与商店 users 数字；website 实际渲染、横幅可见性、未改动区块文案与其他页面是否仍引用已删除的 `STORE_LISTING_URL`；相对链接可达性；Gitee 仓库/Issues 现状；提交与推送结果。未新增事实或市场数据。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 8 轮）

- 日期：2026-11-17T17:40:51.665Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/architecture.zh-CN.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：同意提交

本轮未发现阻塞问题（无 C 项）。以下为提交前后的非阻塞建议。

## 非阻塞建议

**D1｜docs/execution/2026-11-18.md「Qwen 审查」（提交前必须补记，属机械收尾）**
现状止于「第七轮…第八轮等待复审」。本轮结论此刻才产生，材料本身不算缺陷，但若原样归档，会留下「审查悬空」的过期状态。改：提交前把第 8 轮结论与处理如实补记（含「本轮同意提交」），删除「等待复审」措辞；不得预写第 9 轮或任何未发生结果。审查档案同步追加本轮原文。

**D2｜docs/product-wrap-up.md §3「已验证」网站条目**
「四页网站完成本地构建、静态预渲染、响应式布局和可访问性检查」把历史预渲染与本轮重跑的布局/axe 并列，虽有引言限定，仍不如总览 §5 清晰。建议拆为「历史：构建与预渲染」「本轮：布局 36/36、axe 4/4」。

**D3｜docs/store-listing.md「Historical release checklist / 历史发布前确认」**
标题双语、正文仅中文，与全文双语体例不一致。建议补英文对应六条，或在标题注明「正文仅中文」。

**D4｜docs/release-checklist.md「冻结时的真实环境覆盖」**
`[已取消] …未验证` 把「取消动作」与「事实未验证」混用（第 7 轮 D4 只改了版本条）。建议事实用中性标记，`[已取消]` 仅用于原计划动作。

**D5｜AGENTS.md「文档」段与总览 §7 索引**
「CHANGELOG：影响现有用户的行为变化必须记录」隐含后续发布；总览索引仍把 user-personas 描述为「寻找渠道与 M01 访谈清单」。建议分别加「档案维护期内」与「历史」限定。另 website 页脚「MIT 许可口径」可加一句指向收尾报告 §3 的 LICENSE 边界。

## 已核对范围

- 双语 README 逐段对等：归档声明、LICENSE 段、13 选项表（4 国际+7 国内+1 本地+1 自定义）、v6、legacy 样本措辞、安装/历史路线图/归档与支持章节。
- 收尾报告、总览、复盘、发布清单、商店草案、SEO 文档、CHANGELOG、隐私政策双语、画像、演示包、AGENTS、architecture、website 源码与 README 的冻结口径；未见「已上线/已部署/客户/收入/复购/市场验证」表述；停止原因未写成市场为零或技术失败。
- 无残留「待上架/待域名/待推广/待 SEO/待收费」祈使句；商店链接分支与 `STORE_LISTING_URL` 已删除。
- 版本事实一致：`package.json` 0.1.0、含未发布 0.2.0 行为、0.2.0 从未发布；v5/v6 迁移口径一致；0.1.0 日期 2026-09-26。
- 扩展历史证据与本轮网站静态检查已分离（wrap-up §3、overview §5、复盘 §1、台账口径一致）；G 编号与五道门体系已区分；市场证据限定 E1、无 E2。
- 两份 JSON：布局 4×9=36 条、axe 4 页 0 violations，逐项 `archiveBannerObserved=true`、`allPass=true`；UTC 17:37:59.791/17:38:04.966 → SGT 11-18 约 01:38，与所述一致且晚于第 7 轮审查时间；台账时间戳血缘链完整；`environment` 由实际 launch 配置与 `browser.version()` 推导（第 7 轮 D1 闭环）。
- 第 7 轮 C1（审查状态补记至第 7 轮）、C2（第 2、6 轮截断标注与台账措辞）已闭环；D2 链接检查覆盖时点、D3 历史链接文字、D5 复盘资产日期已处理。
- 锚点 `#3-冻结时的技术状态`、`#8-重启前的最低条件` 与标题匹配；新增文件均在 diff 内。

## 未核对范围

未运行任何命令。`package.json`、`LICENSE`、源码与测试实跑（130/1、lint 7 警、143 key、Chromium 159、模型名、7.7s/22s/2.5s）；除两份新 JSON 外的 `docs/execution/*`；`research/*` 结论与商店 users 数字；website 实际渲染、横幅可见性、首页「数据流与限制」区原文及其他页面是否仍引用已删除常量；相对链接可达性；Gitee 仓库/Issues 现状；提交与推送结果。未新增事实或市场数据。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 9 轮）

- 日期：2026-11-17T17:44:07.946Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/architecture.zh-CN.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜`docs/execution/2026-11-18.md`「Qwen 审查」末条 与 `docs/execution/product-wrap-up-qwen-review.md`（审查状态再次悬空）**
问题：台账已补记第 8 轮「同意提交」，但结尾又写「第九轮等待复审，仅用于确认这些机械收尾未改变可提交结论」，而审查档案只到第 8 轮。按现状归档，接手者读到的是「第 9 轮结果未知」，与第 6 轮 C1、第 7 轮 C1、第 8 轮 D1 反复确立的同一标准冲突（不得留悬空状态、不得预写未发生结果）。此外「仅用于确认…未改变可提交结论」是对尚未产生的审查结论的预设。
应如何改：提交前把第 9 轮实际结论与逐项处理如实补记进台账，并把本轮原文追加进审查档案；若届时未取得结论，改写为「审查进行至第 9 轮，最终结论以审查档案为准」，删除「等待复审」与「仅用于确认…」的预设措辞。不得改写历轮正文。

**C2｜`docs/architecture.zh-CN.md`「学习模式暂停（F01）」末句（残留未来功能指令，且全文无归档标注）**
问题：该节仍写「独立同意与可靠复用**将在明确范围的新功能中再设计**，此版不宣称学习恢复可用」。这是面向未来的功能规划语气，与「停止新增功能、F07/F08 等不再立项」的冻结决定冲突；本轮只改了该文末句的 Edge 分发口径，其余未加任何冻结/历史标注。架构文档是 AGENTS.md 与总览索引直接指向的实现入口，接手者会据此认为学习能力仍待恢复开发。
应如何改：把该句改为过去时事实描述（例如「冻结时未设计独立同意与可靠复用方案；此版本不宣称学习可恢复」），并在文首加与其他文档同款的归档状态提示，指向收尾报告；不新增功能承诺。

## 非阻塞建议

- **D1**：复盘 §1「已经验证的」中「网站四页完成本地构建、预渲染、响应式与可访问性检查」仍把历史与本轮混列，建议按总览 §5、收尾报告 §3 拆为「历史：构建/预渲染」「本轮：布局 36/36、axe 4/4」。
- **D2**：网站页脚已补「再分发前须核对 LICENSE 与版权主体」，但未指向收尾报告 §3；可加链接，与双语 README 的许可段口径一致。
- **D3**：英文 README 对总览/复盘标注「(Chinese)」，中文 README 未作对应语言说明；属体例细节，可统一。
- **D4**：台账「相对链接 PASS（17 文档、86 链接）」标注为第 8 轮整改后复核，而 C1 若再改台账文本，需注明覆盖时点或在最终文本上重跑。
- **D5**：`product-wrap-up-qwen-review.mjs` 默认覆盖审查档案，仅在 `APPEND_REVIEW=1` 时追加；建议在台账复现说明中写明该开关，避免后续复跑覆盖历轮记录。

## 已核对范围

双语 README 逐段对等（归档声明、LICENSE 段、13 选项表、v6、legacy 样本、安装/历史路线图/归档与支持）；收尾报告、总览、复盘、发布清单、商店草案、SEO 文档、CHANGELOG、隐私政策双语、画像、演示包、AGENTS、architecture、website 源码与 README 的冻结口径；未见「已上线/已部署/客户/收入/复购/市场验证」表述；停止原因未写成市场为零或技术失败；除 C2 外未见「待上架/待域名/待推广/待 SEO/待收费」祈使句；版本事实一致（0.1.0、含未发布 0.2.0 行为、0.2.0 从未发布、v5/v6）；扩展历史证据与本轮网站静态检查已分离；G 编号与五道门已区分、市场证据限定 E1；两份 JSON：布局 4×9=36 条、axe 4 页 0 violations，逐项 `archiveBannerObserved=true`、`allPass=true`，UTC 17:42:12.089/17:42:17.811 → SGT 11-18 约 01:42，晚于第 8 轮审查时间，血缘链完整；第 8 轮 D2–D5 已在材料中落实；锚点 `#3-冻结时的技术状态`、`#8-重启前的最低条件` 与标题匹配；新增文件均在 diff 内。

## 未核对范围

未运行任何命令。`package.json`、`LICENSE`、源码与测试实跑（130/1、lint 7 警、143 key、Chromium 159、模型名、7.7s/22s/2.5s）；除两份新 JSON 外的 `docs/execution/*`；`research/*` 结论与商店 users 数字；website 实际渲染、横幅可见性、首页「数据流与限制」区原文；相对链接可达性；Gitee 仓库/Issues 现状；提交与推送结果。未新增事实或市场数据。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 10 轮）

- 日期：2026-11-17T17:48:01.025Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/architecture.zh-CN.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜`docs/execution/2026-11-18.md`「Qwen 审查」末条（审查状态再次悬空）**
问题：末条写“第九轮结论‘需修改’……上述问题已处理，**第十轮等待复审**”，而 `product-wrap-up-qwen-review.md` 只到第 9 轮。按现状归档，接手者读到的是“第 10 轮结果未知”，与第 6/7/9 轮反复确立的同一标准冲突（不得留悬空状态、不得预设未发生的结论）。
应如何改：提交前把第 10 轮实际结论与逐项处理如实补记进台账，并把本轮原文追加进审查档案（`APPEND_REVIEW=1`）；若届时未取得结论，改写为“审查进行至第 10 轮，最终结论以审查档案为准”，删除“等待复审”。不得改写历轮正文，不得预写“同意提交”。

**C2｜`website/src/site-links.ts` 的 `WRAP_UP_URL`（冻结档案指向临时实施分支）**
问题：页脚“收尾报告”链接被写成 `…/blob/codex/formfill-execution-20261117/docs/product-wrap-up.md`，把全站唯一的交接入口钉在本轮实施分支上。收尾报告 §6 明确“分支保留，除非所有者另行决定归档或迁移”，一旦合并后删除该分支，四页页脚链接即失效；台账也未记录这一依赖。这是冻结档案里可核对的链接稳定性缺陷。
应如何改：改为稳定引用（默认分支或固定提交 SHA）或指向仓库内文档路径的说明；若确需保留分支路径，必须在台账与 `website/README.md` 写明“该链接依赖实施分支存在，分支删除后需更新”，不新增其他事实。

## 非阻塞建议

- **D1**：`docs/store-listing.md` 数据申报段仍是现在时祈使句（“最终按商店现行定义和实际用途确认”“确认实际发行版与披露符合要求后，勾选…”）。虽在冻结横幅下，建议加“历史建议，不执行”限定，与已改的历史清单一致。
- **D2**：台账提交信息计划为 `docs: retire FormFill productization`，但本轮实际改动含 `website/src/*.tsx|ts|css` 展示代码。建议改用能反映网站改动的类型，或在台账注明“网站展示改动一并归入本次收尾提交”。
- **D3**：`docs/release-checklist.md`“[已取消] 未发布‘更新后刷新网页’的商店版本说明”仍把既成事实与取消动作混用，建议事实项用中性标记（第 7/8 轮同类建议只落实了部分条目）。
- **D4**：台账“相对链接 PASS（17 文档、87 链接）”标注为第九轮落实后复核；C1 补记后需在最终文本上再核一次，并注明该检查不覆盖 Gitee 外链（含 C2 的 blob 链接）。
- **D5**：双语 README 未写“`package.json` 仍 0.1.0、含未发布 0.2.0 行为”，与隐私政策/总览口径虽不冲突但信息缺失，可补一句。
- **D6**：台账“Plane 记录若网络失败不阻塞 Git 交付”未说明 Plane 是什么、本轮是否执行，建议一句说明或删除，避免留下未定义的外部待办。

## 已核对范围

双语 README 逐段对等（归档声明、LICENSE 段、13 选项表、v6、legacy 样本、安装/历史路线图/归档与支持、语言标注）；收尾报告、总览、复盘、发布清单、商店草案、SEO 文档、CHANGELOG、隐私政策双语、画像、演示包、AGENTS、architecture、website 源码与 README 的冻结口径；未见“已上线/已部署/客户/收入/复购/市场验证”表述；停止原因未写成市场为零或技术失败；除 D1 外未见待上架/待域名/待推广/待 SEO/待收费祈使句；架构文档已加归档提示且学习模式句改为过去时（第 9 轮 C2 闭环）；复盘 §1 已拆分历史与本轮网站证据（D1 闭环）；版本事实一致（0.1.0、含未发布 0.2.0 行为、0.2.0 从未发布、v5/v6）；G 编号与五道门已区分、市场证据限定 E1；两份 JSON：布局 4×9=36 条、axe 4 页 0 violations，逐项 `archiveBannerObserved=true`、`allPass=true`，UTC 17:45:54.238/17:45:59.576 → SGT 11-18 约 01:46，与总览 §4、台账一致且晚于第 9 轮审查时间，血缘链完整；第 2、6 轮截断标注与台账措辞一致；新增文件均在 diff 内；锚点 `#3-冻结时的技术状态`、`#8-重启前的最低条件` 与标题匹配。

## 未核对范围

未运行任何命令。`package.json`、`LICENSE`、源码与测试实跑（130/1、lint 7 警、143 key、Chromium 159、模型名、7.7s/22s/2.5s）；除两份新 JSON 外的 `docs/execution/*`；`research/*` 结论与商店 users 数字；website 实际渲染、横幅可见性、首页“数据流与限制”区原文；相对链接与 Gitee 外链可达性（含分支是否仍存在）；Issues 现状；提交与推送结果。未新增事实或市场数据。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 11 轮）

- 日期：2026-11-17T17:51:06.302Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/architecture.zh-CN.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：需修改

## 阻塞问题

**C1｜`docs/execution/2026-11-18.md`「Qwen 审查」末条 + `product-wrap-up-qwen-review.md`（审查状态再次悬空）**
问题：末条写“第十轮结论‘需修改’……**第十一轮等待复审**”，而审查档案只到第 10 轮。与第 6/7/9/10 轮反复确立的同一标准冲突：不得留悬空状态、不得预设未发生结论。
改：提交前把第 10 轮之后各轮（含本轮）实际结论与逐项处理如实补记，并以 `APPEND_REVIEW=1` 追加原文；若未取得结论，写“审查进行至第 N 轮，最终结论以审查档案为准”，删除“等待复审”。不得改写历轮正文。

**C2｜`docs/execution/2026-11-18.md`「验证」链接检查条 与「Qwen 审查」第十轮条自相矛盾**
问题：验证章写“相对链接 PASS（17 文档、87 链接；**第九轮**建议落实后的最终文本上复核）”，第十轮条却称“已在**最终文本**上重跑内部链接检查”。两处覆盖时点不一致，且未说明该检查不覆盖 Gitee 外链（含页脚 blob 分支链接）。属可核对的证据覆盖口径矛盾。
改：统一为实际最后一次运行的时点与计数，并注明“仅覆盖仓库内相对链接，不含 Gitee 外链与分支可达性”。

**C3｜`docs/execution/2026-11-18.md`「提交口径」Plane 段的依据不成立**
问题：写“Plane 指**本仓库 `AGENTS.md` 规定的** `plane-log` 工作日志”，但所给 `AGENTS.md` 全文（含归档提示、目录约定、历史开发约定、文档、消息协议）没有任何 `plane-log` 或 Plane 条款。这是把外部工作流约定误记为仓库规则，会让接手者以为存在未履行的仓库义务。
改：改为如实来源（例如“外部工作日志约定，非本仓库 AGENTS.md 条款”），或删去该 attribution；并写明本轮是否执行。不新增其他事实。

## 非阻塞建议

- **D1**：第 10 轮 C2 要求“台账与 website/README 均写明分支依赖”。`website/README.md` 已写明，台账仅有“已记录归档分支依赖”一句、未指明位置与失效条件；建议在台账补一句指向 README 的说明。`WRAP_UP_URL` 仍钉在实施分支，属已知风险而非新缺陷。
- **D2**：`release-checklist.md` 仍有“[已取消] …材料未提交/申报未提交”，把既成事实与取消动作混用；建议事实项改中性标记，与已改的版本条一致。
- **D3**：`store-listing.md` 数据申报段“Do not select…/Certify…/确认…后勾选”仍为现在时祈使句；虽在“历史建议、从未执行”横幅下，建议整段改过去时描述。
- **D4**：`overview` §4 与台账把网站证据环境写作 `Chromium 157`，而扩展 E2E 为 `Chromium 159`；两者已分别标注，建议在总览加一句“两次检查浏览器不同，不可互相外推”。

## 已核对范围

双语 README 逐段对等（归档声明、LICENSE 段、版本说明、13 选项表、v6、legacy 样本、安装/历史路线图/归档与支持）；收尾报告、总览、复盘、发布清单、商店草案、SEO 文档、CHANGELOG、隐私政策双语、画像、演示包、AGENTS、architecture、website 源码与 README 的冻结口径；未见“已上线/已部署/客户/收入/复购/市场验证”表述；停止原因未写成市场为零或技术失败；未见待上架/待域名/待推广/待 SEO/待收费祈使句（D3 为语气问题）；版本事实一致（0.1.0、含未发布 0.2.0 行为、0.2.0 从未发布、v5/v6）；历史扩展证据与本轮网站静态检查已分离；G 编号与五道门已区分、市场证据限定 E1；两份 JSON：布局 4×9=36、axe 4 页 0 violations，逐项 `archiveBannerObserved=true`、`allPass=true`，UTC 17:45:54.238/17:45:59.576 → SGT 11-18 约 01:46，与总览、台账一致；锚点 `#3-冻结时的技术状态`、`#8-重启前的最低条件` 匹配；新增文件均在 diff 内。

## 未核对范围

未运行任何命令。`package.json`、`LICENSE`、源码与测试实跑（130/1、lint 7 警、143 key、Chromium 159、模型名、7.7s/22s/2.5s）；除两份新 JSON 外的 `docs/execution/*`；`research/*` 结论与商店 users 数字；website 实际渲染、横幅可见性、首页“数据流与限制”区原文；相对链接与 Gitee 外链（含分支是否存在）可达性；Issues 现状；提交与推送结果。未新增事实或市场数据。

---

# FormFill 产品收尾与交接：Qwen 独立审查（第 12 轮）

- 日期：2026-11-17T17:53:19.876Z
- 模型：qwen-token-plan/qwen4-max-preview
- 调用方式：本地 OpenAI 兼容网关；协作子 Agent 因后端任务封装无法读取正文，改用同一指定模型独立审查。
- 输入：README.md、README.zh-CN.md、docs/product-wrap-up.md、docs/product-overview.md、docs/retrospective-and-methodology.md、docs/release-checklist.md、docs/store-listing.md、docs/seo-preparation.md、CHANGELOG.md、AGENTS.md、docs/privacy-policy.md、docs/privacy-policy.zh-CN.md、docs/user-personas.md、docs/architecture.zh-CN.md、docs/demo-pack/README.md、docs/execution/2026-11-18.md；另附仓库文档、网站源码与验证脚本的本次 diff
- 凭据：只从环境变量读取，未写入输出。

结论：同意提交

本轮未发现阻塞问题（无 C 项）。第 11 轮 C1–C3 与 D1–D4 在所给最终文本中均可核对为已处理：台账链接检查已统一为“第十一轮整改后的最终文本、17 文档 87 链接，且不覆盖 Gitee 外链与分支可达性”，第十轮条中矛盾表述已删除；Plane 已改为外部工作日志约定并写明本轮执行结果；发布清单事实项改用“既成事实”，商店申报段改过去时；总览已注明网站检查（Chromium 157）与扩展 E2E（159）不可互相外推。

## 非阻塞建议

**D1｜docs/execution/2026-11-18.md「Qwen 审查」末条**
现写“第十一轮结论需修改……第十二轮按最终文本复核”。未预写结论、未用“等待复审”，可接受；但档案只到第 11 轮。建议补一句“最终结论以审查档案为准”，并在取得本轮结论后以 `APPEND_REVIEW=1` 追加原文，不改写历轮正文。

**D2｜同文件「验证」章措辞**
①“本轮未重跑 130 项扩展测试”与末尾“扩展业务代码未变，因此没有重跑 130 项扩展测试”重复，可合并；②开头“纯文档与独立网站展示改动”未涵盖本轮同时修改的两份检查脚本（总览 §5 已写明），建议改为“文档、网站展示与验证脚本改动；扩展业务代码与测试未修改”。

**D3｜docs/architecture.zh-CN.md「双语默认值与迁移」**
该节仍以“升级到 v4／验证执行到 v4”作结，与同文「目录」中的 v6 并置。建议加一句“本节描述 v1→v4 的历史迁移设计，当前存储为 v6”，避免读者误判现状。

**D4｜website/src/site-links.ts 的 WRAP_UP_URL**
仍钉在实施分支，`website/README.md` 与台账已披露失效条件，属已知风险。若所有者接受，建议后续改为默认分支或固定提交 SHA，使全站唯一交接入口不依赖分支存续。

**D5｜docs/user-personas.md §5 个别判断句**
“浏览器原生自动填充已够，AI 无增量价值”等仍是结论式表述，建议统一加“冻结前判断”限定，与全文历史口径一致。

## 已核对范围

双语 README 逐段对等（归档声明、LICENSE 段、版本说明、13 选项＝12 预设＋自定义、v6、legacy 样本、安装/历史路线图/归档与支持）；收尾报告、总览、复盘、发布清单、商店草案、SEO 文档、CHANGELOG、隐私政策双语、画像、演示包、AGENTS、architecture、website 源码与 README 的冻结口径一致；未见“已上线/已部署/客户/收入/复购/市场验证”表述；停止原因写为商业判断，未断言市场为零或技术失败；未见待上架/待域名/待推广/待 SEO/待收费祈使句；版本事实一致（0.1.0、含未发布 0.2.0 行为、0.2.0 从未发布、v5/v6）；历史扩展证据与本轮网站静态检查已分离，G 编号与五道门已区分，市场证据限定 E1、无 E2；两份 JSON：布局 4×9=36 条、axe 4 页 0 violations，逐项 `archiveBannerObserved=true`、`allPass=true`，UTC 17:45:54.238/17:45:59.576 → SGT 11-18 约 01:46，与总览 §4、台账及血缘链一致；脚本日期由运行时间推导、横幅由 DOM 实测并计入 PASS、environment 由实际 launch 配置与 `browser.version()` 生成；锚点 `#3-冻结时的技术状态`、`#8-重启前的最低条件` 与标题匹配；新增文件均在 diff 内。

## 未核对范围

未运行任何命令。`package.json`、根目录是否确无 `LICENSE`、源码与测试实跑（130/1、lint 7 警、143 key、Chromium 159、模型名、7.7s/22s/2.5s）；除两份新 JSON 外的 `docs/execution/*`；`research/*` 结论与商店 users 数字；website 实际渲染、横幅可见性与首页“数据流与限制”区原文；相对链接与 Gitee 外链（含实施分支是否存在）可达性；台账所记链接/JSON/typecheck/build 计数与 HEAD `7dd0a37`；Issues 现状；提交与推送结果。未新增事实或市场数据。
