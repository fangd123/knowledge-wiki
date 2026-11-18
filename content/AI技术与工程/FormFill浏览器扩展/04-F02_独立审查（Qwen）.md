---
title: "F02 独立审查（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 4
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# F02 独立审查（Qwen）

2026-11-17，Qwen 对 F02 的首次只读审查。
- 结论：**需小改后提交**。取消与排障的主体设计正确、测试先红后绿扎实，但父会话点名的 ModelOutputError→hintNetwork 误导属实（D1，中），另有一个成功路径取消竞态窗口（D2，低中）。两处修复合计约 10 行 + 2 个测试，建议本批修完再提交；其余为非阻塞备注。

## 一、值得保留的设计

同一页开始新请求后，旧请求的收尾不应清除新请求。测试连接与实际填写也分别报告错误；取消不是服务商故障。

## 二、发现（severity / 位置 / 建议）

### D1（中，需修改）ModelOutputError 最终附 hintNetwork，误导排障方向
请求**已成功到达服务商并返回**，只是结果为空、格式不对或无法解析。提示“无法连接端点”会让用户反复检查网络，而应该提示检查模型输出或简化输入。首轮缺少相应的提示断言。

### D2（低中，建议本批修）成功路径的取消竞态窗口：cancel 后仍可能弹出预览
用户取消后，迟到的成功结果不应再打开预览。首轮认为仅中止请求不够，显示结果前也需要尊重取消状态。

### D3（低，备注）cancelRequestedRef 在"取消后立即新填写"时可能误标
取消与下一次填写的状态反馈可能混淆，作为另一项低优先级备注，并非 D2 已修的那一项。

### D4（低，备注）超时正则与重试分类的边界
- `isTimeoutError` 正则 `/timed?\s?out|ETIMEDOUT/i` 可能匹配服务商返回的含 "timeout" 字样的非超时错误消息（如 400 描述里提到 timeout），但 errorHint 先判 401/403/404 状态码，超时分支只在无这些状态码时生效，误伤面小；且 isRetryable 对无状态码错误本就按 timeout/network 关键词重试，口径自洽。可接受，注明即可。
- `isRetryable` 对 ModelOutputError 显式重试（合理），与 D1 修复不冲突。

### D5（低，备注）testModelConnection 的提示拼接与 propose 路径不完全同构
- testModelConnection 无 hint 时 `throw error`（原始 SDK 错误，popup catch 显示 message），有 hint 时自行拼 `status: message hint`；propose 路径统一走 llmCallFailed 前缀。两者用户可见文案结构略不同（测试连接无 "llmCallFailed:" 前缀），属可接受差异，但建议后续统一 describeError 复用，减少分叉。不阻塞。

### D6 地址校验的边界

格式校验可以排除无效地址，却不能判定一个仍然合法的旧地址是否仍是用户打算发送资料的端点。填写前应确认当前服务商和地址；本次审读没有证明已经误发，也没有证明不存在隐私后果。

## 三、测试覆盖评估（问题 D）

关键风险已覆盖：URL 拒绝不调模型、已中止不重试、退避中中止、signal 传递、cancel 中止在途且不进 build-preview、idle、新 propose 中止旧、popup 取消按钮→文案→按钮恢复、非法 URL 不发请求。mock 的 abort 行为（abort 事件 reject APIUserAbortError）与 openai-node v4 实际行为一致，可接受；真实 SDK 的 fetch 底层 abort 时序未验证，属真实 E2E 缺口（已如实记录）。
缺口：D1（ModelOutputError 提示）、D2（成功路径取消）需补测试；D3 竞态可选。现有 102 tests 我全量复跑通过。

## 四、验证与限制

- 我本地复跑：9 文件 102 tests 全过、compile 过、lint 0 errors/7 warnings（基线一致）、build 过（1.2 MB）。
- 全部 vitest+mock，非真实模型/浏览器 E2E；真实 Key 与浏览器组合验证仍是外部依赖，本审查不宣称已验证真实连接/取消时序。
- 文档口径核对一致（问题 E 通过）；AGENTS.md 消息清单含 cancel-fill，架构 F02 节描述与代码逐条相符。

## 五、结论与提交条件

**需小改**：D1 必须修（方向性误导，父会话已点名）；D2 强烈建议同批修（取消语义完整性）。两者修完补 2 个测试、重跑全量，交我增量复核即可。D3–D5 非阻塞，可列后续。


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
