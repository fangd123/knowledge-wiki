---
title: "W05 映射复用浏览器补验 第三轮确认（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 32
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# W05 映射复用浏览器补验 第三轮确认（Qwen）

- 日期：2026-11-17；审查人：Qwen（/root/qwen_report_review，只读审查）
- 范围：本轮无源码改动，仅文档+脚本+证据（w05-mapping-probe.json/.mjs、demo-pack README、CHANGELOG、台账）。
- 结论：**同意提交**。我第二轮 D3 指出的映射换记录浏览器缺口已闭合，证据可信、口径如实。origin 空串问题判断见下，非阻塞。

## 一、补验证据核验——通过

- w05-mapping-probe.json PASS：真实 Chromium 159+构建扩展+真实弹窗 UI。Alice 应用→保存映射→存储仅 5 条结构（Name/Phone/Email/Code/Since 列名+columnIndex+locator），containsValues=false（无 Alice/13800000001）；company 已有值未勾不存、类别值无法对应单元格不存——与 F06 设计和我首轮 G1 审读完全一致。
- 换 Bob→"用已存映射填写"2,480ms 生成预览（免模型，无 propose-fill），值 Bob Li/13800000002/bob@example.test/008/2026-03-02；页面残留 Alice 值时已有值保护默认不勾（应用 0 项禁用）→显式勾选 5 项→应用成功，公司预置值不动。主体隔离与已有值保护在真实浏览器中实证。
- 切 Archive 映射按钮仍在：两表列结构相同→表单签名一致→按设计可用。demo-pack 第 7 步预期已正确修正为"同结构签名一致仍可用，复用值取当前记录；验证签名过滤需换字段结构不同的表单"——口径诚实，不假装验证了签名过滤。
- 脚本审读：凭据仅 env、记录无 key；免模型判定用"replay 路径无 propose-fill+计时"（SW fetch 无法外部 patch，合理替代）；存储检查经 worker.evaluate 直读 chrome.storage。可信。

## 二、origin 空串问题——判断：测试驱动产物，非产品缺陷，无需本批加防御

父会话判断正确，我补充分析：
- 根因：manifest 仅 `['storage','activeTab']`，无 `tabs` 权限。真实工具栏点击弹窗会授予 activeTab→tab.url 可见→origin 正确。本探针把 popup 作为独立 tab 打开+CDP 激活 demo tab，**未触发真实 activeTab 授予**，故 tab.url 为空→origin 回退空串。这是驱动方式产物。
- 产品侧现状安全：listMappings 按 `entry.origin === origin` 精确匹配，空 origin 映射只与空 origin 查询匹配，**不会泄漏到真实 origin 站点**；真实弹窗保存的映射 origin 正确，跨站点隔离在真实交互下成立。空 origin 映射仅在"再次以无 activeTab 方式打开"时可见——真实用户路径不会发生。
- **不建议本批加"origin 为空拒绝保存/复用"防御**：理由 (1) 真实路径 origin 必非空，防御无实际收益；(2) 若未来有合法场景 tab.url 暂不可读（如 chrome:// 页面），硬拒绝会误伤；(3) 当前精确匹配已保证空 origin 不跨站泄漏，无安全漏洞。
- D1（非阻塞建议，列后续）：可在真实工具栏交互验证时确认 origin 正确填充；若想要纵深防御，可考虑"origin 为空时保存但在 UI 标注来源未知"，而非拒绝。当前如实记录"跨站点 origin 隔离在此驱动方式下未验证，待真实工具栏交互验证"是正确口径，无需改码。

## 三、文档与验证

- demo-pack 状态改"含映射复用均完成"+新增映射运行结果节+第 7 步预期修正；CHANGELOG F06 浏览器验证条目准确；台账 W05"全部完成"。未覆盖项（单选/复选建议、品牌 Chrome、真实工具栏 activeTab 时序与 origin 隔离）如实保留。
- 我复跑全量 130 tests+1 skipped 过（本轮无源码改动，基线不变）。

## 四、结论

**同意提交**。D3 缺口闭合，映射只存结构/免模型复用/主体隔离/已有值保护在真实浏览器实证。origin 空串为测试驱动产物，产品侧精确匹配已安全，无需本批防御；真实工具栏 origin 隔离列后续验证。无阻塞项。

——交父会话汇总；未改仓库源码与既有报告，未执行 Plane，未提交。
