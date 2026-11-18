---
title: "F06 独立审查（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 7
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# F06 独立审查（Qwen）

2026-11-17，Qwen 审查 F06 保存与复用映射的变更。
- 结论：**同意提交**。我此前三条保留条件（用户确认、主体隔离、结构变更停用）全部满足，无隐式历史采集回退；发现 2 项低危备注与 1 项文档精化建议，均不阻塞。11 文件 123 tests/compile/lint/build/网站 typecheck+build 我全量复跑通过。

## 一、三条保留条件核验（重点 A）——全部满足

1. **用户确认**：保存入口仅在 Excel 填写成功（applyResults 含 filled）后出现，且 `window.confirm(mappingConfirm)` 明确同意才写存储；无自动保存路径。复用也需用户逐次点击"Use saved mapping"，结果仍进预览→应用→可撤销，不自动写页。
2. **主体隔离**：存储经 sanitizeMapping 白名单清洗，只留 origin/formSignature/column/columnIndex/locator/label，拒绝 value/apiKey 等任何额外字段（测试注入 PRIVATE_VALUE/PRIVATE_KEY 后存储不含，断言到位）；replayMapping 值**只来自当前 recordValues**，测试断言复用不含 Alice。popup 端到端测试断言存储无 Alice/111、复用预览 Bob/222 无 Alice、无新 propose-fill。无任何路径把旧记录值带进新请求或存储。
3. **结构变更停用**：双层——useEffect 列表按当前表单签名集合过滤展示；handleUseMapping 先 get-form-context 校验签名含 mapping.formSignature（不含→mappingNone），replayMapping 再逐项要求 item.formSignature === mapping.formSignature 且 status ok 且 locator 匹配。签名变化→无项→停用。测试覆盖签名变化拒绝。

以上三项是在模拟存储和模拟页面上核对的。[后来的换记录补验](32-W05_映射复用浏览器补验_第三轮确认（Qwen）.md)观察到 Bob 预览未沿用 Alice 的值，但没有观察真实工具栏下的站点来源，不能据此断言跨站隔离已经验证。

## 二、复用时仍检查当前页面

存储的是映射关系，不是再次写入的授权。复用要重新生成预览；不支持、找不到或受保护的目标不能靠旧映射取得写入资格，真正写入前仍须核对。

## 三、发现（severity / 位置 / 建议）

### G1（低，备注）列索引反查的两列同值歧义（对应 B）
- handleSaveMapping 用 `values.findIndex(cell => cell !== '' && cell === value)` 反查列索引，两列同值时取第一个匹配。后果：保存的 columnIndex 可能指向"同值的另一列"，复用时若两列值分化会填错列。
- 评估：这是精确相等反查的固有局限，发生条件苛刻（需两列同值且用户未编辑），且复用仍经预览用户可核对。可接受，但**建议文档注明**"同值列按首个匹配保存，复用前请在预览核对"。当前 README/架构未提此局限。非阻塞。
- 编辑值无法对应→不保存该项（columnIndex<0 跳过），逻辑正确，保护了"编辑过的值不被误存为列映射"。

### G2（低，备注）sameLocator 用 some 而非全等
- replayMapping 的 sameLocator 只要 id/name/css/labelText 任一相等即匹配。若 mapping locator 含多键而 fresh item 仅共享其一（如共享 name 但 id 不同），仍匹配。
- 使用当前预览给出的定位并再次检查，降低了错配风险；若对应的并非用户原意字段，仍可能填错，因此预览不应省略。

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

**同意提交**指这一轮模拟条件下的实现。建议把同值列首匹配写入指南；真实站点来源和表单变化另需独立证据。
