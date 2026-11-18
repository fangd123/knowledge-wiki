---
title: "F06 独立审查（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: 项目复盘
series: FormFill浏览器扩展
series_order: 7
tags:
  - 项目复盘
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# F06 独立审查（Qwen）

- 日期：2026-11-17；审查人：Qwen（/root/qwen_report_review，只读审查）
- 范围：基线 `07e103c` 之上全部未提交改动（16 修改 + utils/mappings.ts、tests/mappings.test.ts 新增）。
- 结论：**同意提交**。我此前三条保留条件（用户确认、主体隔离、结构变更停用）全部满足，无隐式历史采集回退；发现 2 项低危备注与 1 项文档精化建议，均不阻塞。11 文件 123 tests/compile/lint/build/网站 typecheck+build 我全量复跑通过。

## 一、三条保留条件核验（重点 A）——全部满足

1. **用户确认**：保存入口仅在 Excel 填写成功（applyResults 含 filled）后出现，且 `window.confirm(mappingConfirm)` 明确同意才写存储；无自动保存路径。复用也需用户逐次点击"Use saved mapping"，结果仍进预览→应用→可撤销，不自动写页。
2. **主体隔离**：存储经 sanitizeMapping 白名单清洗，只留 origin/formSignature/column/columnIndex/locator/label，拒绝 value/apiKey 等任何额外字段（测试注入 PRIVATE_VALUE/PRIVATE_KEY 后存储不含，断言到位）；replayMapping 值**只来自当前 recordValues**，测试断言复用不含 Alice。popup 端到端测试断言存储无 Alice/111、复用预览 Bob/222 无 Alice、无新 propose-fill。无任何路径把旧记录值带进新请求或存储。
3. **结构变更停用**：双层——useEffect 列表按当前表单签名集合过滤展示；handleUseMapping 先 get-form-context 校验签名含 mapping.formSignature（不含→mappingNone），replayMapping 再逐项要求 item.formSignature === mapping.formSignature 且 status ok 且 locator 匹配。签名变化→无项→停用。测试覆盖签名变化拒绝。

**与 F01 口径一致（E）**：rg 确认 content/background 无 submit 监听、无 learned-samples 写入、无历史外发分支；映射复用走 build-preview（本地 content）而非模型，不调 callModelWithRetry。架构文档明确"不是 F01 暂停的隐式历史采集"，属实。

## 二、信任边界（C）——成立

伪造存储映射不能绕过保护：replayMapping 只保留 fresh build-preview 中 status ok 的项（protected/not-found/unsupported 被过滤），且最终 apply 仍走 content applyPreview 的逐项重定位+保护复查+签名复查+scanToken 校验（F03 已验证）。映射只决定"哪列填哪个 locator"，不决定写入合法性。sanitizeMapping 在读取时也清洗（readAll→sanitize），存量脏数据被过滤。信任边界完整。

## 三、发现（severity / 位置 / 建议）

### G1（低，备注）列索引反查的两列同值歧义（对应 B）
- handleSaveMapping 用 `values.findIndex(cell => cell !== '' && cell === value)` 反查列索引，两列同值时取第一个匹配。后果：保存的 columnIndex 可能指向"同值的另一列"，复用时若两列值分化会填错列。
- 评估：这是精确相等反查的固有局限，发生条件苛刻（需两列同值且用户未编辑），且复用仍经预览用户可核对。可接受，但**建议文档注明**"同值列按首个匹配保存，复用前请在预览核对"。当前 README/架构未提此局限。非阻塞。
- 编辑值无法对应→不保存该项（columnIndex<0 跳过），逻辑正确，保护了"编辑过的值不被误存为列映射"。

### G2（低，备注）sameLocator 用 some 而非全等
- replayMapping 的 sameLocator 只要 id/name/css/labelText 任一相等即匹配。若 mapping locator 含多键而 fresh item 仅共享其一（如共享 name 但 id 不同），仍匹配。
- 评估：匹配后用的是 **fresh item 的 locator**（`{...target, value}`），写入定位以 fresh 为准，且 applyPreview 再复查，故无安全后果；最坏是"映射意图与 fresh 定位略有偏差"，但 fresh 来自当前页面真实控件，方向安全。可接受，注明即可。

### G3（低，文档精化）useEffect 依赖与签名时效（对应 D）
- useEffect 依赖 [excel?.filename, excel?.sheetIndex]，cancelled 守卫正确，tab 切换/签名变化时列表可能短暂过期，但 handleUseMapping 使用时重新 get-form-context 校验签名，过期列表项点击会被 mappingNone 拦截——双重保护成立，无竞态写入风险。
- 观察：切换 recordRowIndex/headerRowIndex 不触发列表刷新（依赖未含），但映射与记录行无关（只绑签名），列表无需随记录变；headerRowIndex 变化可能改变 formSignature？否——签名来自页面表单结构，与 Excel 表头行无关。故依赖选择正确。无需改。

### G4（备注，非问题澄清）
- replayMapping 与 handleUseMapping 都重新赋 index（replayed.length / map index），decisions 按新 index 初始化，已有值默认不勾口径与 F03 一致。
- handleUseMapping 的 instructions 用 `mapping.entries.indexOf(entry)` 作 columnIndex 回退，与 replayMapping 的 position 回退一致；但 instructions 经 build-preview 后由 replayMapping 按 columnIndex??position 取值，两条路径回退口径统一，无错位。
- 映射 id 用 Date.now()+random，saveMapping 按 id 去重+createdAt 排序淘汰，上限 20 测试覆盖。
- locales 11 key 双语对等（zh_CN:386-413 + btnSaveMapping/btnUseMapping），i18n 测试保证。

## 四、测试缺口与文档（F）

- 关键风险已覆盖：脏字段注入清洗、origin+签名过滤、上限淘汰+删除、复用不带旧值、protected/missing 跳过、签名变化拒绝、name/label 定位匹配、popup 端到端（保存→存储无值→换记录复用无模型调用）。
- 可补（非阻塞）：(1) G1 两列同值反查的固定测试；(2) columnIndex 缺失时按 position 回退的单测（当前测试 entries 无 columnIndex，实际走 position 回退，已隐式覆盖，但无显式断言）；(3) 复用后 apply 仍触发 undo 的端到端断言（当前 popup 测试止于预览，未点 apply/undo）。建议列后续。
- 文档一致性：双语 README/隐私/store-listing/架构 F06 节/CHANGELOG/网站第 6 卡+PrivacyPolicy 存储段均同步，"只存结构、不外发、可删、签名变化停用"口径与实现一致。隐私政策"映射不发送给模型或任何服务器"属实（复用走本地 build-preview）。建议补 G1 同值列局限说明。

## 五、验证与限制

- 我本地复跑：11 文件 123 tests 全过、compile 过、lint 0 errors/7 warnings、build 过、网站 typecheck/build 过。
- 全部 vitest+mock（read-excel-file/messenger/storage fakeBrowser），非真实浏览器/模型 E2E；真实表单签名稳定性、真实 xlsx 复用保真待环境，未宣称实测。

## 六、结论

**同意提交**。三条保留条件满足，信任边界完整，无 F01 回退。G1 建议补文档注明（同值列首匹配局限），G2/G3 备注，测试缺口列后续。无阻塞项。

——交父会话汇总；未改仓库源码与既有报告，未执行 Plane，未提交。
