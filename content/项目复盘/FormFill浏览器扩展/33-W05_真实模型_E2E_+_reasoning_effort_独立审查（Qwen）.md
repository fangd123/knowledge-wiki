---
title: "W05 真实模型 E2E + reasoning_effort 独立审查（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: 项目复盘
series: FormFill浏览器扩展
series_order: 33
tags:
  - 项目复盘
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# W05 真实模型 E2E + reasoning_effort 独立审查（Qwen）

- 日期：2026-11-17；审查人：Qwen（/root/qwen_report_review，只读审查）
- 范围：基线 `637087a` 之上未提交改动（settings v6 / llm-retry reasoning_effort / SettingsPanel / 5 双语 key / 测试 / E2E 门控测试 / 探针脚本 / 文档与网站口径）。凭据仅运行时 env，未入库。
- 结论：**同意提交**。无阻塞项（C 编号空）。reasoning_effort 不门控设计合理、v6 迁移与测试充分、E2E 门控稳妥无凭据泄漏、对外口径如实不过度宣称、文档双语对等。4 项非阻塞备注（D1–D4）。

## 一、reasoning_effort 不门控设计（问题 1）——合理

- thinking 受 `provider.supportsThinking` 门控，因为它是 Ark 风格的**厂商专属**参数（`thinking:{type}`），非预设支持的服务商发送会报错。reasoning_effort 是 **OpenAI 兼容标准参数**，语义不同：任何 OpenAI 兼容网关都可能接受，不应由厂商专属的 supportsThinking 标志决定。两者独立、可共存（ark 预设 supportsThinking=true 时同时发 thinking 与 reasoning_effort，测试断言共存）。
- 设计正确地区分了"标准参数"与"厂商扩展参数"的门控逻辑。默认不发送（undefined 时不加字段），显式选择才发，符合"最小惊讶+不擅自改变请求"。探针脚本证实网关接受 reasoning_effort=high 且产生 reasoning_tokens，设计有实证支撑。
- D1（备注）：custom 预设 supportsThinking=true，意味着 custom 用户开 thinking 会发厂商专属 thinking 参数到任意网关——这是既有行为非本批引入，但 reasoning_effort 不门控后，文档宜提示"thinking 为厂商专属、reasoning_effort 为标准，未知网关建议只用后者"。README 已说"两者可被同时接受的网关会同时收到"，口径基本到位，可后续补一句风险提示。非阻塞。

## 二、v6 迁移与测试（问题 2）——充分

- migration 6 `(old) => ({...old, reasoningEffort: old.reasoningEffort})` 保留已存值（实质是 no-op 保留，正确——新字段可选，旧数据无此字段则 undefined）。version 5→6，getDefaultSettings 加 reasoningEffort:undefined。
- 测试：v5→v6 迁移断言保留 apiKey/model/providerId/privacyNoticeAccepted、reasoningEffort undefined、meta v6；既有 v1→v5 全链测试更新到 v6。llm-retry +2：custom+high 发 reasoning_effort 且不发 thinking；unset 不发；ark thinking+reasoningEffort 共存。覆盖关键路径。
- D2（备注）：migration 6 的 `reasoningEffort: old.reasoningEffort` 在旧值无此字段时显式写入 undefined 键——JSON 序列化会丢弃 undefined，无实际副作用，但写法略冗余（直接 `...old` 即可）。非阻塞，风格项。

## 三、E2E 门控与凭据安全（问题 3）——稳妥

- `describe.runIf(Boolean(FF_BASE && FF_KEY))` 门控：默认套件无 env 时整个 describe 跳过。我复跑全量确认 **128 passed | 1 skipped**，E2E 未误入默认套件。vitest 默认 include 会匹配 *.test.ts，但 runIf 在 collection 阶段判定，无 env 即 skip，CI 默认安全。
- 凭据安全：FF_KEY 仅 process.env 读取， never hardcoded；w05-real-run.json 只记 baseUrlHost（127.0.0.1:3425）不含 key/path/query；探针脚本只读 env、输出不含 Authorization 值；diff/JSON rg 无 sk- 或密钥残留。测试结束 browser.storage.local.clear()。无泄漏。
- E2E 断言扎实：单记录无 Bob/Carol、scan 不含 secret、预览不写页、company 已有值默认不勾且应用后保持、secret 空、撤销 restored>0 且 company 不受影响、重新应用。真实 read-excel-file/node + 真实 callModelWithRetry(high) + 真实 content 函数。
- D3（备注）：E2E 不设 globalThis.window 以避开 OpenAI SDK dangerouslyAllowBrowser 报错，content 用 ownerDocument.defaultView——这是 JSDOM 环境的合理适配，但意味着 E2E 未覆盖真实浏览器 window 全局路径（如 cssEscape 的 globalThis.CSS 回退）。口径已标"非真实浏览器扩展加载"，诚实。真实浏览器加载 smoke 仍待验证，不变。
- D4（备注）：E2E 写 w05-real-run.json 到仓库路径，复跑会覆盖既有记录——合理（每次运行更新），但建议记录保留历史或加日期后缀以防覆盖珍贵首跑记录。当前首跑已提交归档意图，非阻塞。

## 四、对外口径（问题 4）——如实不过度宣称

- 网站 ExcelGuide"真实结果记录"节：明确"代码级流水线验证，尚未在真实浏览器中加载扩展运行，浏览器端演示记录后续补充"，数字（7.7 秒/6 建议/0 非法/007/2026-01-15/撤销 5 项）与 w05-real-run.json 一致。无"已实测浏览器"宣称。
- demo-pack README：状态改"代码级真实模型运行已完成，真实浏览器扩展运行记录待补"，明确"不得把代码级运行当作浏览器端已实测宣传"，未覆盖项（下拉/单选/复选/映射换记录步骤 6-7/真实浏览器）如实列出。
- CHANGELOG/README：reasoning_effort 描述准确（标准参数、显式才发、独立 thinking、v6 保留配置），E2E 标注 JSDOM+真实模型、真实浏览器待验证。
- 我此前 W05 审查 J1（sample-form 缺预置值）已解决：company 字段现有 value="Existing Company Ltd"（HEAD 已含），E2E 据此验证已有值保护，验收自洽。

## 五、文档双语对等（问题 5）——通过

- 5 个新 key（labelReasoningEffort/reasoningNotSent/reasoningLow/Medium/High）en/zh 对等，i18n 测试保证键相等。README.md/README.zh-CN.md 高级设置段中英表述一致（标准参数、显式才发、独立、v6）。架构 reasoning_effort 段与 settings v6 描述同步。SettingsPanel 下拉始终显示、默认"不发送"，与文案一致。

## 六、验证与限制

- 我本地复跑：128 tests passed | 1 skipped（E2E 门控）、tsc --noEmit 过、lint 0 errors/7 warnings（基线）、扩展 build 过、网站 typecheck/build 过；解析 w05-layout-check.json 确认 36 组 0 失败。
- E2E 真实运行记录（w05-real-run.json）采信父会话运行结果，我未重跑真实模型（无 env 凭据，且不应消耗用户网关）；其断言逻辑经源码审读确认与记录一致。
- 全部默认测试为 mock；E2E 为 JSDOM+真实模型，非真实浏览器扩展加载。真实浏览器加载 smoke、下拉/单选/复选/映射换记录的真实运行仍待环境，口径如实。

## 七、结论

**同意提交**。C 编号阻塞项：无。D1–D4 非阻塞备注（thinking 风险提示、migration 写法、JSDOM window 路径覆盖、运行记录覆盖）可列后续。设计合理、迁移充分、门控稳妥、口径诚实、双语对等，符合"不宣称未验证结果"的一贯口径。

——交父会话汇总；未改仓库源码与既有报告，未执行 Plane，未提交。
