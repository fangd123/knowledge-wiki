---
title: "F01 正式代码审查（qwen4-max-preview）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: 项目复盘
series: FormFill浏览器扩展
series_order: 3
tags:
  - 项目复盘
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# F01 正式代码审查（qwen4-max-preview）

日期：2026-11-17。基线 `f3a2a60`，审查对象为工作区全部未提交变更（含审查期间补充的 labelText 定位一行补丁与 background 测试类型断言）。只读审查+只读验证；未改源码，不执行 Plane。

## 结论：可以提交

未发现阻塞级 bug 或隐私绕过。降级范围与批准计划 F01 一致，我独立复跑验证全部通过：`npx vitest --run` 8 文件 62 tests 通过；`npm run compile` 通过；`npm run lint` 0 errors/7 warnings；`npm run build`（Chrome MV3）通过；website `typecheck`/`build` 通过。**全部为 mock/jsdom 验证，不构成真实模型端到端证据**；真实浏览器+模型组合验收属 F02/W05，提交记录中不得冒充实测。

## 核心边界核验（通过）

1. **采集路径整段移除**：content.ts 无 submit 监听、无 `getLearningContainer`/body 回退；learn.ts 删除 save/find/extract API，仅存签名+本地查看/清除。否定测试覆盖 in-form/orphan 两类容器+旧 flag=true+无关表单，断言 storage 为空。
2. **历史外发整段移除**：background.ts 删除 findLearnedSample 拼接与 import；测试以 v4 存量 `learningModeEnabled:true`+含 `PRIVATE_HISTORICAL_MARKER` 的旧样本种子，断言模型调用参数不含标记。system prompt 保留对历史的防御性措辞（llm-retry.ts:147），架构文档已注明，属有意保留，可接受。
3. **v5 迁移+读写三重强制**：migration 5、getSettings、saveSettings 均强制 `learningModeEnabled:false`；测试验证 v4→v5 保留 apiKey/model/baseUrl/providerProfiles/privacyNoticeAccepted。UI 开关已移除，SettingsPanel 只展示"已暂停"+查看/清除。旧 flag 无任何绕过路径。
4. **同意闸双层**：background 在扫描/模型调用前检查 privacyNoticeAccepted（测试断言未同意时 sendMessage/callModel 均零调用）；Popup 先确认→按序落盘→再请求，三项 popup 测试覆盖等待保存/保存失败不发请求/取消不置位。settingsReady 门住输入与设置面板，加载失败有错误文案。
5. **扫描/写入共用保护**：isSerializableControl 排除 password/hidden/file/submit/button/reset/image、`:disabled`（覆盖 fieldset disabled）、readOnly，isVisible 逐级检查祖先 hidden/aria-hidden/display:none/visibility:hidden|collapse。fillForm 对 locateElement 结果复检同一判定，测试验证模型直接定位 password/hidden/readonly/disabled/隐藏祖先控件均被拒且原值不变。
6. **摘要文本净化**：getPageText 排除全部表单控件/button/script/style/template/noscript/iframe/object/contenteditable 及不可见节点；label（for 关联与嵌套）与 nearbyText 均走该路径。测试断言摘要不含 PRIVATE_ 标记（textarea 初值、嵌套控件、密码值、隐藏文本、脚本）。llm-retry 发送前 nearbyText 再截 120、控件上限 200 未变。
7. **radio 安全**：同名组限定 `radio.form === input.form` 且逐个 isSerializableControl；无名时仅自身。测试验证不选 disabled 项、不跨 form。labelText 定位改用 getPageText 后与净化 label 一致，补充回归（先失败后修复）已纳入。
8. **旧记录管理**：listLearnedSamples 查看时清理损坏/未来时间戳/超 90 天/超 50 条并回写；清除入口保留；展示用 React 文本渲染（`<pre>{JSON.stringify(...)}</pre>`），i18n 测试含 `<script>` 值验证不执行 HTML。90 天/50 条与文案一致。
9. **文案一致性**：zh_CN/en locales、双语 README、双语 privacy-policy、store-listing（含权限理由与数据申报）、架构文档、CHANGELOG、网站 App.tsx 四处学习表述全部同步为"暂停/仅本地/不发送"，未发现残留旧承诺。website 无独立隐私页文件，隐私内容在 App.tsx 锚点区，已同步。

## 发现（均非阻塞）

| # | 严重度 | 位置 | 问题与建议 |
|---|---|---|---|
| 1 | 低 | content.ts `isVisible`/`getPageText` | 可见性判定不含 opacity:0、clip/clip-path、零尺寸、视口外等 CSS 隐藏手法；此类"视觉隐藏但 DOM 可见"的控件/文字仍会进摘要或被填写。隐私政策已声明"不能保证去掉所有敏感页面文字"，口径诚实，不构成绕过；建议 F03 预览阶段让用户可见拟填目标以兜底，并在未来考虑 opacity/尺寸检查。 |
| 2 | 低 | content.ts `getPageText` | 排除列表未含 `option`/`optgroup` 以外的问题——option 文本本属字段摘要合法内容，正确；但 `<label>` 内若含链接/按钮文字会被保留（button 已排除，a 未排除）。属可见描述文字，风险极低，仅记录。 |
| 3 | 低 | content.ts radio 分支 | `input.name` 为空时 radios=[input]，若模型给无名 radio 传其他选项值将 throw→failed，行为安全但用户只见失败计数；可接受，F03 的部分失败反馈会覆盖解释。 |
| 4 | 低 | background.ts `test-llm-connection` | 连接测试不检查 privacyNoticeAccepted。其发送固定测试 prompt+Key 到用户自配接口，不含表单/文件/历史，且由用户在设置中主动触发，隐私政策已单独说明；不视为绕过。若追求一致可加闸，非必须。 |
| 5 | 低 | App.tsx `persistSettings` | 保存队列 catch 后继续链式下一次保存，失败仅显示文案不重试；极端情况下用户连续改设置可能基于未落盘状态。当前 UI 串行操作+fill 前 await 队列已覆盖主要竞态，测试证明失败不发请求；可接受。 |
| 6 | 低 | SettingsPanel 查看入口 | 旧记录 `<pre>` 展示完整 JSON 含 values 原文，符合"可查看"承诺；记录可能含个人信息，界面在用户本地弹窗内，无外发。建议后续（非本项）加一句"记录可能含个人信息"提示文案，与政策口径呼应。 |
| 7 | 信息 | tests | 62 tests 全为 jsdom/mock；background 测试用 fakeBrowser+wxt/testing 种子存储，迁移路径真实走 storage 版本机制，质量良好。缺"v1→v5 全链迁移"显式用例（现有 v4→v5 + 既有 v1–v4 测试组合覆盖），可在后续补一条端到端迁移用例，非阻塞。 |
| 8 | 信息 | 文档 | CHANGELOG/README 要求"更新后刷新已打开网页"正确且必要（旧 content script 仍含旧监听）；商店发布说明应保留此句。research/ 归档排除于 compile/lint 的处理正确，旧审计测试不计为修复证据。 |

## 测试漏测评估

关键否定路径（未同意/旧 flag/跨表单/密码写入/textarea 泄漏/历史外发/保存竞态）均有先失败后修复的复现测试，覆盖充分。未覆盖但风险已被现有断言间接约束的：opacity 类隐藏（发现 1）、多标签页并发保存（发现 5）、真实 provider 对 thinking 参数的兼容（属 F02）。无必须本项补齐的漏测。

## 提交建议

同意按约定式提交格式提交本项（含 research/ 归档与 docs/execution/ 记录）。提交信息建议涵盖：暂停自动学习采集与历史外发、v5 迁移保留服务商配置、扫描/写入共用保护检查、摘要文本净化、后台同意闸与 Popup 按序保存、双语文档同步。提交后在执行记录补记 commit hash 与验证结果；真实模型 E2E 留待 F02/W05，不在此宣称。
