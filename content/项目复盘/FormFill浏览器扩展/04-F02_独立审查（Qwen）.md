---
title: "F02 独立审查（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: 项目复盘
series: FormFill浏览器扩展
series_order: 4
tags:
  - 项目复盘
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# F02 独立审查（Qwen）

- 日期：2026-11-17；审查人：Qwen（/root/qwen_report_review，只读审查）
- 范围：基线 `95ab7e8` 之上全部未提交改动（15 文件，llm-retry/background/popup/InputPanel/类型/locales/测试/文档）。
- 结论：**需小改后提交**。取消与排障的主体设计正确、测试先红后绿扎实，但父会话点名的 ModelOutputError→hintNetwork 误导属实（D1，中），另有一个成功路径取消竞态窗口（D2，低中）。两处修复合计约 10 行 + 2 个测试，建议本批修完再提交；其余为非阻塞备注。

## 一、值得保留的设计

注册表清理用闭包捕获 generation 并比对 controller 身份，正确避免旧请求 finally 误删新请求条目；新旧 propose 交替时旧 signal 被 abort、旧请求不进 build-preview（有测试，mock 在 abort 事件 reject 模拟 SDK，合理）。sleepOrAbort 可打断退避、已中止不重试不发请求、signal 传入 SDK 第二参，均有测试。URL 双层校验（popup 前置 + background/llm-retry 兜底）口径一致，http localhost 合法。错误提示保留 llmCallFailed 状态码+原因再附 hint，不吞细节。文档（架构 F02 节/CHANGELOG/AGENTS 消息清单）与实现一致，locales 7 key 双语对等由 i18n 测试保证。

## 二、发现（severity / 位置 / 建议）

### D1（中，需修改）ModelOutputError 最终附 hintNetwork，误导排障方向
- 位置：`utils/llm-retry.ts` `errorHint`。ModelOutputError（llmInvalidJson/llmEmptyContent/llmInvalidFormat）无 status，重试耗尽后落入 `status === undefined → hintNetwork`。
- 问题：请求**已成功到达服务商并返回**，只是内容非法；提示"无法连接端点，检查网络/地址/本地服务"与事实相反，会把用户引向错误方向。父会话判断正确，这不是"可能不贴切"而是方向性错误。现有测试（llm-retry.test.ts:160）只断言 llmCallFailed 前缀，未覆盖此误导。
- 建议：errorHint 开头 `if (error instanceof ModelOutputError) return browser.i18n.getMessage('hintModelOutput')`（新 key，双语，如"模型已响应但返回内容不是有效填写指令；请重试、更换模型或简化描述"）。补 1 个测试断言三类 ModelOutputError 耗尽后附 hintModelOutput 而非 hintNetwork。若不想加 key，最低限度对 ModelOutputError 返回 ''（不附提示），但我推荐加针对性提示。

### D2（低中，建议本批修）成功路径的取消竞态窗口：cancel 后仍可能弹出预览
- 位置：`entrypoints/background.ts` propose-fill 的 `if (controller.signal.aborted)` 检查仅在 build-preview **之前**；`popup/App.tsx` 成功分支无条件 setPreview。
- 机理：模型已返回、aborted 检查通过后，build-preview 往返期间用户点取消（cancel-fill 中止 controller 并删注册表），propose 仍返回 preview；popup 成功分支不看 cancelRequestedRef，预览照常展示。用户看到"已取消"预期却出现可应用预览，违背"取消不写页/不出结果"的直觉（虽然 apply 仍需再确认，无数据风险）。
- 建议（二选一，推荐前者）：popup 成功分支加 `if (cancelRequestedRef.current) { setStatus(t('llmCancelled')); setStatusType('error'); return; }`；或 background 在 build-preview 返回后再查一次 `controller.signal.aborted`。补 1 个测试：propose 成功返回前触发 cancel，断言不展示预览。

### D3（低，备注）cancelRequestedRef 在"取消后立即新填写"时可能误标
- 位置：popup handleCancelGeneration 置 true → handleFill 开头置 false。若用户在 cancel-fill 消息在途时立刻点填写（loading 已 false 可点），新请求启动把 ref 置 false，随后旧 propose 的 cancelled 错误到达时 ref 已是 false，会显示原始 error 而非 llmCancelled——但旧请求的 setStatus 会覆盖新请求的 statusAnalyzing，短暂文案错乱。窗口极窄（需 loading 结束与新点击之间精确插入），且无功能损害。可在 handleFill 用递增 requestId 守卫旧响应（类似网站 Demo runId），列后续打磨，不阻塞。

### D4（低，备注）超时正则与重试分类的边界
- `isTimeoutError` 正则 `/timed?\s?out|ETIMEDOUT/i` 可能匹配服务商返回的含 "timeout" 字样的非超时错误消息（如 400 描述里提到 timeout），但 errorHint 先判 401/403/404 状态码，超时分支只在无这些状态码时生效，误伤面小；且 isRetryable 对无状态码错误本就按 timeout/network 关键词重试，口径自洽。可接受，注明即可。
- `isRetryable` 对 ModelOutputError 显式重试（合理），与 D1 修复不冲突。

### D5（低，备注）testModelConnection 的提示拼接与 propose 路径不完全同构
- testModelConnection 无 hint 时 `throw error`（原始 SDK 错误，popup catch 显示 message），有 hint 时自行拼 `status: message hint`；propose 路径统一走 llmCallFailed 前缀。两者用户可见文案结构略不同（测试连接无 "llmCallFailed:" 前缀），属可接受差异，但建议后续统一 describeError 复用，减少分叉。不阻塞。

### D6（备注，非问题澄清）
- validateBaseUrl 用 `new URL` + protocol 白名单，拒绝 ftp/裸域名/空串，接受 http(s) 完整地址；`new URL('http://')` 会 throw 被拒，正确。popup settingsRef 与已保存设置一致性：updateSettings 同步写 settingsRef 再异步落盘，handleFill/handleTestConnection 校验的是 settingsRef.current（最新编辑值），background 校验的是 getSettings()（已落盘值）；handleTestConnection 先 await settingsSaveRef 再校验，handleFill 的 propose 由 background 读已落盘设置——若用户在落盘完成前点填写，popup 校验新值通过但 background 可能读到旧值。实际 handleFill 在 consent 分支 await persistSettings，非首次填写时 settingsSaveRef 队列通常已完成；极端竞态下 background 用旧 baseUrl 发请求，但旧值若合法只是打到旧地址（用户刚改的地址未生效），无安全后果，且双层校验保证不会用非法值发请求。可接受，注明。
- cancel-fill 用 `tab!.id!` 非空断言：前置 `tab?.id !== undefined` 已守卫，安全。
- 空 Key 路径 bgNoApiKey 在 URL 校验之前，顺序合理（先解决缺 Key 再谈地址）。

## 三、测试覆盖评估（问题 D）

关键风险已覆盖：URL 拒绝不调模型、已中止不重试、退避中中止、signal 传递、cancel 中止在途且不进 build-preview、idle、新 propose 中止旧、popup 取消按钮→文案→按钮恢复、非法 URL 不发请求。mock 的 abort 行为（abort 事件 reject APIUserAbortError）与 openai-node v4 实际行为一致，可接受；真实 SDK 的 fetch 底层 abort 时序未验证，属真实 E2E 缺口（已如实记录）。
缺口：D1（ModelOutputError 提示）、D2（成功路径取消）需补测试；D3 竞态可选。现有 102 tests 我全量复跑通过。

## 四、验证与限制

- 我本地复跑：9 文件 102 tests 全过、compile 过、lint 0 errors/7 warnings（基线一致）、build 过（1.2 MB）。
- 全部 vitest+mock，非真实模型/浏览器 E2E；真实 Key 与浏览器组合验证仍是外部依赖，本审查不宣称已验证真实连接/取消时序。
- 文档口径核对一致（问题 E 通过）；AGENTS.md 消息清单含 cancel-fill，架构 F02 节描述与代码逐条相符。

## 五、结论与提交条件

**需小改**：D1 必须修（方向性误导，父会话已点名）；D2 强烈建议同批修（取消语义完整性）。两者修完补 2 个测试、重跑全量，交我增量复核即可。D3–D5 非阻塞，可列后续。

——交父会话汇总；未改仓库源码与既有报告，未执行 Plane，未提交。

---

# 第二轮增量复核（2026-11-17）

结论：**同意提交**。D1/D2 修复到位，无新增问题。

## 逐项核验

- **D1 已修复**。`errorHint` 开头 `if (error instanceof ModelOutputError) return hintModelOutput`，置于状态码分支之前，三类输出错误不再落 hintNetwork；注释明确"请求已到达服务商，仅内容非法"。新 key 双语对等（en/zh 语义一致，zh_CN:359）。新测试断言耗尽重试后含 hintModelOutput 且不含 hintNetwork、create 调 2 次（重试语义保留），先红后绿口径属实。testModelConnection 复用同一 errorHint，自动受益。
- **D2 已修复**。popup 成功分支开头检查 cancelRequestedRef，已取消则显示 llmCancelled 并 return，不 setPreview/setDecisions；finally 仍正常 setLoading(false)。新测试模拟"取消后迟到预览"：断言显示 Generation cancelled.、无 Review before filling、无迟到值文本，覆盖了我指出的 build-preview 往返窗口。error/catch 分支的 cancelRequestedRef 统一文案保持不变。
- **文档同步**。架构 F02 节新增 ModelOutputError 分类描述与"取消后迟到的预览结果也会被丢弃，不弹出预览"，与实现逐条相符。
- D3/D4/D5 按首轮意见列后续，不在本批，无异议。

## 复跑验证

9 文件 104 tests 全过、compile 过、lint 0 errors/7 warnings（基线一致）、build 过（1.2 MB）。仍为 vitest+mock，非真实模型/浏览器 E2E，真实连接/取消时序与可用 Key 仍是外部依赖，口径不变。

——第二轮复核完成，同意按当前 diff 提交；未改仓库源码，未执行 Plane，未提交。
