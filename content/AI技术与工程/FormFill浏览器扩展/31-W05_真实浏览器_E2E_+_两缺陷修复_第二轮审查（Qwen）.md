---
title: "W05 真实浏览器 E2E + 两缺陷修复 第二轮审查（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 31
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# W05 真实浏览器 E2E + 两缺陷修复 第二轮审查（Qwen）

- 日期：2026-11-17；审查人：Qwen（/root/qwen_report_review，只读审查）
- 范围：基线 `637087a` 之上未提交改动（含首轮已审 reasoning_effort 部分 + 本轮 i18n key 修复、select 占位修复、真实浏览器 E2E 证据与文档）。
- 结论：**同意提交**。阻塞项（C 编号）：无。两个缺陷修复正确、测试充分，i18n 映射方案稳妥，浏览器 E2E 证据可信、口径如实。3 项非阻塞备注（D1–D3）。首轮 D1/D2/D4 已确认落实。

## 一、缺陷修复一：i18n key 合法性（问题 2）——正确稳妥

- 根因真实：Chrome 对 messages.json 中含 `[A-Za-z0-9_]` 之外字符的 key 拒绝加载**整个扩展**，`previewStatus_not-found` 的连字符触发。这是 vitest/JSDOM 永远发现不了的（fakeBrowser.i18n 不校验 key 名），只有真实浏览器加载 smoke 能捕获——本轮排障路径（品牌 Chrome 157 禁 --load-extension → Playwright Chromium 159 → 二分定位 key）合理。
- 修复方案稳妥：key 改 `previewStatus_not_found`（en/zh 同步），**状态值类型不变**（messaging.ts 仍 'not-found'），仅在展示层 `item.status.replace(/-/g,'_')` 映射。这避免了改类型连锁影响 content/replay/测试，最小侵入。
- 其他状态值核查：preview status 四值 ok/protected/not-found/unsupported 中仅 not-found 含连字符，已处理；applyResult 三值 filled/skipped/failed 均合法无连字符，`applyResult_` 拼接无需映射。无遗漏非法字符。
- 测试：i18n.test.tsx 新增 `/^[A-Za-z0-9_]+$/` 断言遍历两语所有 key（先红后绿），这是**防回归的正确层级**——未来任何新 key 含非法字符都会被默认套件拦截，不再依赖真实浏览器。充分。

## 二、缺陷修复二：select 占位选项（问题 1）——正确，multiple 安全

- readCurrentValue select 分支改为 `selectedOptions.filter(option => option.value.trim() !== '')`，空值占位（"请选择" value=""）不再计为已有值。
- **multiple select 安全**：filter 作用于每个 selectedOption，多选时只剔除空值占位项、保留真实选中项（如 tags 多选 x/z 不受影响，既有 C5/C6 多选撤销测试仍过）。真实单选选中（preset value="b" selected）正确判为已有值。空白文本占位（value="" 文本全角空格）因 value.trim()==='' 被剔除——注意判断用的是 **value** 而非 text，正确（占位的本质是 value 空，文本可能是任意提示语）。
- 测试三例（占位/真实选中/空白文本占位）断言 hasExistingValue [false,true,false]、currentValue ''/'B'，先红后绿，覆盖关键分支。
- D1（备注）：边界——若某 select 的**真实选项 value 本身就是空字符串**（罕见但合法，如 `<option value="">未指定</option>` 作为有效选择），会被误判为无已有值。这是"空 value=占位"启发式的固有局限，与 F06 同值列反查类似，发生条件苛刻且预览可核对。可接受，建议架构文档已写"空值占位不算已有值"口径，无需改代码。非阻塞。

## 三、浏览器 E2E 证据可信度与口径（问题 3）——可信、如实

- smoke（w05-browser-smoke.json）：Chromium 159.0.8154.6，扩展加载 error:null、service worker 注册、真实 get-form-context 返回 1 表单 11 控件、secretLeaked:false。PASS。
- E2E（w05-browser-e2e.json）：真实弹窗 UI 配置 custom+high→测试连接 success→导入 xlsx 默认 Alice→模型往返 21,954ms→预览 7 建议→应用 6 项→撤销。applied 断言 name/phone/email/code=007/since=2026-01-15/company=Existing/secret=''，undone name/code 空、company 保持。PASS。
- 我目验 w05-browser-applied.png：Alice Zhang/13800000001/alice@example.test/007/日期/类别=贸易 已填，公司保持 Existing Company Ltd，密码框空，单选/复选未选——与 JSON 完全一致。截图真实可信。
- 脚本审读：凭据仅 env（FF_BASE/FF_KEY），记录只存 baseUrlHost 不含 key/path；rg 扫描 JSON/脚本无 sk-/密钥残留；ASCII 路径副本规避 Chrome 非 ASCII 拒绝；CDP activateTarget 解决 popup 作为 tab 时 activeTab 查询目标问题（合理适配）。断言逻辑（ok 判定含 007/日期/公司/密码/撤销）扎实。
- **口径如实**：demo-pack/网站/CHANGELOG 均明确"真实 Chromium（Playwright 159）非品牌 Chrome"，未覆盖项列全——映射保存与换记录复用（步骤 6-7）、单选/复选建议、品牌版 Chrome（137+ 禁 --load-extension）。网站 ExcelGuide 改浏览器端实测口径并保留 JSDOM 代码级运行区分。无过度宣称。
- D2（备注）：E2E 把 popup 作为独立 tab 打开而非真实工具栏弹窗，用 CDP 激活 demo tab 满足 activeTab——这验证了消息链路与 UI，但**真实工具栏弹窗的 activeTab 授予时序**（用户点击图标瞬间授权）未完全复现。口径标"真实弹窗 UI"基本准确（确是 popup.html 真实界面），建议注明"popup 以 tab 形式加载"以更精确。非阻塞。
- D3（备注）：浏览器 E2E 模型往返 21,954ms 远高于代码级 7,667ms——合理（真实浏览器+网关负载波动），但单次运行不构成性能基准，文档已记单次耗时未宣称稳定性能，口径正确。映射换记录（步骤 6-7）浏览器端未跑，是 F06 真实浏览器验证的剩余缺口，已如实列出，建议后续补。

## 四、文档一致性（问题 4）——通过

- CHANGELOG 顶部两条修复+浏览器验证条目准确；架构补 i18n key 合法性规则（含展示层映射说明）与 select 占位语义、settings v6；demo-pack 新增"真实浏览器运行结果"节含未覆盖项；网站 ExcelGuide 浏览器端实测口径。双语 README 高级设置段一致。
- 首轮 D1/D2/D4 落实确认：README 双语补"未知网关建议只用标准 reasoning_effort，thinking 厂商专属部分网关拒绝"；migration 6 简化为 `{...oldValue}`；E2E 运行记录加日期后缀（w05-real-run-2026-11-17T02-20-48Z.json、w05-browser-e2e-2026-11-17T03-07-24Z.json）留历史。

## 五、验证与限制

- 我本地复跑：130 tests passed | 1 skipped（E2E 门控）、tsc 过、lint 0 errors/7 warnings（基线）、扩展 build 过、网站 typecheck/build 过；解析 w05-layout-check.json 确认 36 组 0 失败。
- 浏览器 E2E 真实运行记录采信父会话结果（我未重跑消耗网关/无 env），断言逻辑经脚本源码审读 + 截图目验交叉确认。
- 默认测试为 mock；浏览器 E2E 为 Playwright Chromium 159 非品牌 Chrome；映射换记录/单选复选浏览器端、品牌 Chrome 手动加载仍待验证，口径如实。

## 六、结论

**同意提交**。C 编号阻塞项：无。D1（空 value 真实选项启发式局限）、D2（popup 以 tab 加载宜注明）、D3（映射换记录浏览器端缺口待补）非阻塞。两个真实缺陷修复正确且测试在正确层级防回归，浏览器 E2E 证据可信、口径诚实，符合"不宣称未验证结果"一贯口径。本轮把"真实浏览器扩展加载 smoke 未通过"的历史缺口正式闭合（Chromium 159 路径），是实质进展。

——交父会话汇总；未改仓库源码与既有报告，未执行 Plane，未提交。
