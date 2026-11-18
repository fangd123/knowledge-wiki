---
title: "F05 独立审查（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 6
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# F05 独立审查（Qwen）

- 日期：2026-11-17；审查人：Qwen（/root/qwen_report_review，只读审查）
- 范围：基线 `5ad6488` 之上全部未提交改动（15 修改 + ExcelSelectionPanel.tsx、excel-selection.test.ts 新增）。
- 结论：**同意提交**，附 1 项建议同批顺手处理的小修（E1 死代码口径）与 1 项文案精化（E2）；均不阻塞。主体隔离、边界处理、日期策略实现正确，测试先红后绿扎实，文档双语对等。

## 一、逐项核验（对应父会话 A–E）

### A. 主体隔离——彻底，通过
- 请求构造唯一入口是 handleFill 的 effectiveFileData：excel 存在时**每次点击重新**用 buildRecordContent(sheet, headerRowIndex, recordRowIndex) 序列化，不存在缓存旧内容的路径；buildRecordContent 只取两行，邻行/其他 sheet 结构上不可能进入（测试断言不含 Alice/Carol 验证）。
- 状态互斥正确：xlsx 成功路径 setFileData(undefined)+setExcel；非 xlsx 成功路径 setExcel(null)+setFileData；handleFill 中 excel 优先覆盖 effectiveFileData。切换 sheet 重置表头/记录为新 sheet 首两行；表头变更冲突时自动换记录（indexes.find(item => item !== index)）。popup 测试覆盖切记录→只含 Bob、切 sheet→只含 Carol 不含 Bob，隔离断言到位。
- 选择状态仅弹窗内存，关闭即丢，与隐私政策一致。workbook 全量行留在本地内存供选择，不发送——政策"其他行不会发送"属实。

### B. buildRecordContent 边界——通过
- 越界/空白行索引/表头=记录行均抛 excelRowOutOfRange（测试覆盖越界两方向）；宽度不齐用 Math.max + Column N 补位（测试覆盖）；空值列跳过；管道/换行转义（测试覆盖 a|b\nc → a\b c）。空 sheet（indexes 为空）时任何索引都 includes 失败→抛错，正确。
- 一个非阻塞观察：表头行本身含管道/换行时同样被 formatCell 转义，label 与 value 口径一致，无注入面扩大（系统提示已声明文件为不可信数据）。

### C. 日期/前导零——实现正确，文档基本如实，一处可精化（E2）
- 我核读了 read-excel-file 的 parseExcelDate.js：它用 `new Date(Math.floor((serial - daysBeforeUnixEpoch) * DAY))` 构造 **UTC 纪元毫秒**，纯日期单元格得到 UTC 午夜 Date。因此 formatCell 的 `endsWith('T00:00:00.000Z')` 判断对 xlsx 日期单元格是可靠的，**不存在父会话担心的"非 UTC 时区导致日期带时间"问题**——该库不做本地时区转换。含时间的单元格 serial 有小数部分，保留完整 ISO，正确。
- 前导零：文本单元格库按字符串返回，'007' 保留（测试覆盖）；数值单元格 Excel 本身已丢格式，README 如实声明无法恢复。
- E2（低，文案精化）：README/架构写"日期按 UTC 显示 YYYY-MM-DD（含时间的保留完整 ISO）"准确；但可补一句"纯日期单元格按 UTC 午夜识别"以说明判断依据，避免未来换解析库时误以为本地时区安全。非阻塞。

### D. 旧 parseFile xlsx 整表路径——建议本批删除 xlsx 分支（E1）
- 现状：popup 对 xlsx 已走 parseExcelWorkbook，parseFile 的 xlsx case（parseExcelFile + rowsToMarkdownTable）成为死代码——全仓 rg 确认 parseFile 仅被 popup 非 xlsx 路径调用，xlsx 永远不会进入该 case。
- 与 F03 口径一致性：F03 删除旧 fill-form 的理由是"无调用方+留一条绕过预览的可达路径"。这里死代码虽无安全绕过（整表发送只在 xlsx case，而 xlsx 已被前置拦截），但保留它意味着 parseFile 仍宣称支持 xlsx 整表，与"只发送选中记录"的新承诺在代码层面矛盾，未来若有人复用 parseFile 处理 xlsx 会静默整表外发。
- 建议（E1，低中，顺手修）：parseFile 的 xlsx case 改为抛错或重定向（如 throw fileTypeError 提示走选择面板），删除 parseExcelFile/rowsToMarkdownTable；或保留 parseFile 但让 xlsx case 调用 parseExcelWorkbook+默认首记录并注释"仅兼容入口，popup 不用"。我推荐前者（删除），与 F03 口径一致。若父会话认为 parseFile 是公共工具函数需保留 xlsx 能力，可接受保留，但须在架构文档注明"parseFile xlsx 整表路径为遗留兼容、popup 不使用、不得用于新调用方"。这不阻塞提交，但建议本批处理以免口径悬置。

### E. 测试缺口——关键风险已覆盖，两处可补
- 已覆盖：全 sheet 返回、前导零+UTC 日期、单记录隔离、自选表头+Column N、空白行索引、越界拒绝、转义、popup 切记录/切 sheet 隔离。read-excel-file 为 mock，真实 xlsx 解析待环境（如实记录）。
- 可补（非阻塞）：(1) 表头=记录行抛错的直接单测（当前越界测试未含此分支，handleFill 有前置拦截但 buildRecordContent 自身该分支无测试）；(2) 空 sheet/单行 sheet 的 popup 阻断（excelNoRecords 显示+不发送）——逻辑审读正确，缺测试固定。建议列后续或顺手补。

## 二、其他发现

- E3（备注，非问题）：ExcelSelectionPanel rowPreview 用 String(cell) 而非 formatCell，日期在选择选项里显示为完整 Date.toString()（含 GMT 时区串）而非 YYYY-MM-DD——仅影响选项摘要观感，不影响发送内容（发送走 formatCell）。可后续统一用 formatCell 提升一致性，不阻塞。
- E4（备注）：handleExcelHeaderChange 冲突回退 `?? index` 在"仅一行"时会令 header=record，但 handleFill 前置 recordIndexes 过滤已阻断发送并显示 excelNoRecords，双重保护成立。
- locales 7 key 双语对等（zh_CN:362-381 审读），excelRowOption 用 $1/$2 数组替换，i18n 测试保证键相等。
- 文档同步完整：双语 README/隐私/store-listing/架构 F05 节/CHANGELOG/网站 FEATURES 文件卡与 PrivacyPolicy 均改"只发送选中记录"，路线图移除已实现项。网站 typecheck/build 我复跑通过。布局检查沿用 F03 批次版本、台账注明，口径可接受（文案改动小，未重复提交合理）。

## 三、验证与限制

- 我本地复跑：10 文件 112 tests 全过、compile 过、lint 0 errors/7 warnings（基线一致）、build 过（1.21 MB）、网站 typecheck/build 过。
- read-excel-file 为 mock；真实 xlsx 文件解析、日期/前导零在真实文件上的保真、模型 E2E 均待环境，未宣称实测。parseExcelDate 行为结论来自库源码审读（置信度高），非真实文件验证。

## 四、结论

**同意提交**。E1（删除/标注 parseFile xlsx 死代码）建议本批顺手处理以与 F03 口径一致；E2 文案精化与 E3/E4 及测试缺口可列后续。若父会话选择保留 parseFile xlsx 路径，补架构注明即可，不阻塞。

——交父会话汇总；未改仓库源码与既有报告，未执行 Plane，未提交。

---

# 增量确认（2026-11-17，建议项落实）

结论：**确认落实，维持同意提交**。

- E1：parseExcelFile/rowsToMarkdownTable 已删除（rg 无残留）；parseFile xlsx case 抛 fileExcelError('use parseExcelWorkbook') 并注释 F05 口径；架构与 CHANGELOG 同步"整表分支已删除（调用即报错）"。
- E2：formatCell 注释与架构文档补充 UTC 午夜判断依据（parseExcelDate 构造纯日期为 UTC 午夜，换库须重核）。
- E3：rowPreview 对 Date 用 toISOString 同款 YYYY-MM-DD/ISO 渲染，选项不再显示 Date.toString()。
- 测试缺口：excel-selection 补表头=记录行抛错、空 sheet 拒绝（+2）；popup 补单行 sheet 阻断填写（断言不发请求+显示 no data rows，+1）。

复跑：10 文件 115 tests 全过、compile 过、lint 0 errors/7 warnings。仍为 mock 验证，真实 xlsx/E2E 待环境口径不变。未改源码，未执行 Plane。
