---
title: "W05 合成资料包独立审查（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 11
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# W05 合成资料包独立审查（Qwen）

- 日期：2026-11-17；审查人：Qwen（/root/qwen_report_review，只读审查 + 仓库外真实解析脚本）
- 范围：基线 `fbc9390` 之上未提交改动（docs/demo-pack/ 三文件 + 台账 W05 行）。
- 结论：**同意提交**。资料覆盖 F02–F06 验收点、虚构数据彻底、README 口径如实、控件足够验证保护/撤销/映射。我用真实 read-excel-file/node（非 mock）独立解析 xlsx 印证保真。2 项非阻塞备注。

## 一、真实解析复核（非 mock）

我用仓库 node_modules 的 read-excel-file/node 在 %TEMP% 脚本解析 synthetic-contacts.xlsx，结果：
- 双 sheet：Clients（表头+3 记录）、Archive（表头+1 旧记录），与 README 一致。
- Code 列 `typeof === 'string'`，值 "007"/"008"/"009" 前导零保留——印证 F05 文本前导零策略。
- Since 列 `instanceof Date === true`，ISO 为 `2026-01-15T00:00:00.000Z` 等 **UTC 午夜**——独立印证 F05 日期策略与 parseExcelDate 源码结论（此前为源码审读，现有真实文件佐证）。
- 这是真实库解析真实 xlsx，但**仍非扩展 E2E**（未加载扩展、未调模型），口径与父会话一致。

## 二、虚构数据彻底性——通过

全部记录用 example.test（RFC 2606 保留测试域）、138000000xx 虚构号段、Alice Zhang/Bob Li/Carol Wang/Old Contact 泛化名、Northwind/Contoso/Fabrikam 微软示例公司名。无真实个人信息、无真实邮箱/电话。sample-form.html 页首与 HTML 注释双重虚构声明，提交 preventDefault 仅本地提示、不发数据。彻底。

## 三、验收点覆盖（F02–F06）——基本完整

- F03/F04：sample-form 含 text/tel/email/date/select/radio 组/checkbox/textarea，覆盖预览/应用/撤销各控件类型；radio 组（intent）可验证 C1/C2 组级保护与撤销；password 框（secret）验证受保护不填。
- F05：Clients 多记录+Archive 切 sheet 验证主体隔离；Code 前导零、Since 日期验证保真；表头行 1 明确。
- F06：README 步骤 6 保存映射→换 Bob 复用→断言无模型调用/无 Alice 残留；步骤 7 切 Archive 验证签名/主体过滤。
- F02：README 步骤 2 测试连接、步骤 8 记录失败边界；错误提示/取消可在真实运行时对照（资料包本身不含错误注入，属运行时分项，合理）。

## 四、发现（非阻塞）

### J1（低，备注）sample-form 缺"已有值字段"预置
- F03 核心承诺"已有值默认不勾"需要一个**预填了值的控件**来验证默认不勾选。当前 sample-form 所有字段初始为空，README 步骤 4 要求"确认已有值字段默认不勾"，但页面无预置值，真实运行时需手动先填一个字段再触发预览才能验证此点。
- 建议：给某个字段（如 company）预置一个 value，或在 README 步骤 4 注明"先手动填入一个字段值再填写以验证已有值保护"。不阻塞提交，但补上会让验收对照更自洽。

### J2（低，备注）date 控件与 Excel 日期格式衔接
- sample-form 的 since 为 `type="date"`（要求 YYYY-MM-DD），Excel Since 经 formatCell 渲染为 YYYY-MM-DD，匹配正确。但真实运行时模型是否把日期值正确对应到 date 控件取决于模型输出，资料包无法保证——README 步骤 8 已列"日期格式"为失败边界记录项，口径如实。无需改，注明即可。

## 五、README 口径——如实

"资料已就绪，真实运行记录待补""不得把本包的存在当作已实测宣传"明确；8 步运行清单含版本/模型/耗时/失败边界记录；视频剪辑须标注加速与提前配置、另留完整耗时；完成后补进 Excel 指南"真实结果记录"节（与 W06 该节"待 W05 补充"衔接）。台账 W05 行同步"资料就绪、真实运行待环境"。无过度承诺。

## 六、验证与限制

- 我独立用真实 read-excel-file/node 解析 xlsx（双 sheet/前导零/UTC 午夜确认）；sample-form 控件类型与提交拦截经源码审读。
- 未跑真实扩展 E2E（无可用 Key），未宣称实测；资料包就绪≠功能已验证。真实浏览器加载 smoke 此前未通过的口径不变。

## 七、结论

**同意提交**。J1 建议补预置值或 README 注明以自洽验证"已有值保护"；J2 备注。资料诚实、覆盖充分、保真有真实库佐证，符合"不宣称未验证结果"口径。

——交父会话汇总；未改仓库源码与既有报告，未执行 Plane，未提交。
