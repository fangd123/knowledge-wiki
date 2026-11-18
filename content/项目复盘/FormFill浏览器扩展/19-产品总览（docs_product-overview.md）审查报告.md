---
title: "产品总览（docs/product-overview.md）审查报告"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: 项目复盘
series: FormFill浏览器扩展
series_order: 19
tags:
  - 项目复盘
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# 产品总览（docs/product-overview.md）审查报告

- 审查人：Qwen 独立审查代理（/root/qwen_report_review）
- 日期：2026-11-17；基线 6c61121；本批 = product-overview.md 新增 + 双语 README 两处修改 + 台账一行
- 方式：只读审查（读全文、比对代码与证据文件、核对调研原始出处），未改业务源码、未提交、未执行 Plane

## 结论：需修改（仅 1 项阻塞，其余已核实无误）

文档整体质量高：功能表与代码逐项对应、验证状态如实分列已验证/未验证、商业口径与已批准决策一致、市场数字有出处且限定正确。但存在一处虚构能力宣称，修正后即可提交。

## C1（阻塞，事实错误）：控件范围声称支持 open Shadow DOM，代码不支持

- 位置：§2 填写主流程表"控件范围"行——"含 form 外控件与 open Shadow DOM"。
- 证据（高置信，直接读源码）：`entrypoints/content.ts` 的 `getControls`（第 56 行）仅用 `container.querySelectorAll('input, textarea, select')`；全 entrypoints/ 与 utils/ 无任何 `shadowRoot`/`attachShadow`/穿透遍历逻辑。`querySelectorAll` 不穿透 shadow boundary（open 模式也不穿透，需显式访问 `shadowRoot`）。form 外控件支持属实（orphanControls 伪表单路径，F03 轮已审）。
- 为什么重要：本文自我定位是"事实以代码与证据为准，不宣称未验证的能力"，而 Shadow DOM 是从未实现也从未测试过的能力。潜在合作者按此选型会在 Shadow DOM 站点（不少现代组件库使用）直接踩空，且损害整份文档可信度。调研中 filly 的 README 明确列过 Shadow DOM/iframe 限制（feature-comparison.md Y1a），说明这是同类已知边界，更应如实写。
- 修改建议：删除"与 open Shadow DOM"，并明确"不支持 Shadow DOM 内控件"（作为已知限制列出）。

## D 级（非阻塞，可与 C1 一并处理或留后续）

### D1（措辞）：§2 标题"全部已实现并通过测试"建议限定测试性质

功能已实现属实，130 tests 也真实通过，但其中大部分是 vitest+mock，真实浏览器 E2E 仅覆盖主链路（§5 已如实分列）。建议标题改为"全部已实现（验证状态见 §5）"或"已实现并通过本地测试"，避免读者把标题理解为全部经过真实环境验证。文档他处口径正确，此项仅措辞。

### D2（可选补充）：iframe 限制未提及

内容脚本未设 `allFrames`（默认仅顶层 frame），iframe 内表单不会被扫描/填写。"部分站点不支持"过于含糊，建议在控件范围行与 Shadow DOM 限制一起写明"不支持 iframe 内表单"。同属已知边界如实披露，不阻塞。

### D3（索引描述滞后）：§7 演示资料包描述"两次真实运行结果"

docs/demo-pack/README.md 实际已有三节运行结果（代码级、真实浏览器、映射复用），建议改"三次"或"真实运行结果"。

## 已核实无误的关键事实（逐项）

- **13 个服务商选项** ✓：providers.ts 实际 13 个 preset（openai/openrouter/gemini/anthropic/ark/dashscope/deepseek/zhipu/moonshot/siliconflow/hunyuan/ollama/custom），国内 7 个 ✓。注意 rg 粗计数 17 含 4 个 region 分组项，非错误。
- **143 双语 key** ✓：en/messages.json 属性计数 143，i18n 测试保证对等与 key 合法性（含浏览器轮加的 [A-Za-z0-9_] 断言）。
- **130 tests + 1 skipped** ✓：与最近一轮复跑一致（本批纯文档，未重跑）。
- **耗时数字** ✓：7,667ms→"7.7 秒"（w05-real-run.json）、21,954ms→"约 22 秒"（w05-browser-e2e.json modelRoundTripMs=21954）、2,480ms→"约 2.5 秒"（w05-mapping-probe.json），均一致。
- **市场数字** ✓：marketing-benchmark.md 表格——Text Blaze 800,000、Simplify Copilot 500,000（求职填写定位）、FillApp 2,000、Formly 34，与 §3 完全一致；"users 非下载/活跃""Simplify 含求职平台""大平台不能当通用市场规模"三条限定均保留，口径正确。
- **默认服务商** ✓：utils/settings.ts 第 19 行按语言 zh→ark、其他→openai。
- **30,000 字符截断** ✓：background.ts `MAX_FILE_CHARS = 30000` 且附截断说明。
- **映射复用描述** ✓：只存结构、上限 20、签名停用、免模型、仍走预览/撤销，与 F06 实现及 w05-mapping-probe 证据一致。
- **阶段/版本/权限/学习暂停/90 天 50 条/¥699 条件性实验/¥199 已撤出/MIT 口径/40 词待采样/默认不做清单** ✓：与 README、CHANGELOG、release-checklist、execution-plan 既有口径一致，无新造结论。
- **§5 未验证清单** ✓：工具栏 activeTab/origin、品牌 Chrome、单选复选、abort 时序、线上抓取、真实用户需求，与发布清单一致，无遗漏关键项。
- **README 两处 diff** ✓：顶部互链（英文版注明 Chinese）、utils/settings.ts 描述 v5→v6 并说明两次迁移含义，中英对等。
- **文档索引链接** ✓：research/execution-plan.md、demo-pack、release-checklist、seo-preparation 等目标文件在本 worktree 均存在。

## 验证与限制

- 本轮为文档事实核对：读 product-overview.md 全文、README diff、providers/settings/background/content 源码、w05 三份证据 JSON、marketing-benchmark.md 原始表格；未重跑测试（纯文档批次，历轮已复跑 130 tests）；未做新的市场调研或网页抓取。
- Shadow DOM/iframe 结论基于源码静态审查（高置信）；未专门构造 Shadow DOM 页面实测，但 querySelectorAll 不穿透 shadow boundary 是平台确定行为。
- 未改业务源码与既有报告（本报告为唯一新增输出）、未提交、未执行 Plane。

## 修复确认（同日追加）：同意提交

- C1 ✓：§2 控件范围行已删除"open Shadow DOM"，改为"不支持 Shadow DOM 内控件与 iframe 内表单（内容脚本无 allFrames）"，同时落实 D2，与代码事实一致。
- D1 ✓：§2 标题改"全部已实现并通过本地测试；真实环境验证状态见 §5"。
- D3 ✓：索引 demo-pack 行改"三节真实运行结果（代码级/浏览器/映射复用）"。
- 独立复核全仓宣称：README 双语、架构、商店材料、website/src 中 rg 无 Shadow DOM 能力宣称（仅 styles.css 的 box-shadow 样式属性，无关）；历史调研文件提及属归档证据，不改，口径正确。
- 结论由"需修改"改为**同意提交**。无遗留阻塞项。
