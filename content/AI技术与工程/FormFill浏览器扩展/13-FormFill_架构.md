---
title: "FormFill 架构"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 13
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# FormFill 架构

> **归档状态（2026-11-18）：** 本文描述已冻结的技术快照。项目不再新增功能、发布、部署或恢复学习模式；状态与重启条件见[产品收尾与交接报告](22-FormFill_产品收尾与交接报告.md)。

## 消息流程

内部消息：propose-fill、apply-fill、cancel-fill（Popup → Background），get-form-context、build-preview、apply-preview、undo-fill（Background/Popup → Content），test-llm-connection（Popup → Background）。旧 fill-form-request / fill-form 无预览直写路径已删除：扩展各组件同包发布、无外部消息消费者，保留只会留下绕过预览的可达代码路径。

Popup 发送 propose-fill。Background 检查数据发送确认后查询当前标签页，发送 get-form-context。Content 扫描所有可见 input、textarea、select：form 内控件按 form 分组，脱离 form 的控件组成一个伪表单，并返回由页面地址和表单签名组成的 scanToken。Background 调用模型后把指令发回 Content 的 build-preview：Content 只做定位、保护判定和当前值读取，不写入页面，返回逐项预览（ok/protected/not-found/unsupported、当前值、是否已有值）与非法指令计数。

Popup 展示预览：建议值可编辑，已有值的字段默认不勾选；单选组的“已有值”按组判定——组内任一可用项被选中即视为已有值，展示选中项的值；下拉框只统计 value 非空的选中项，空值占位选项（如“请选择”）不算已有值。用户点击应用后，Popup 发送 apply-fill（携带逐项 include 决定和编辑后的值），Background 只转发 apply-preview 到当前标签页，不再调用模型。Content 校验 scanToken 与当前页面一致（不一致则整体拒写），逐项复查表单签名与保护判定后写入，并把写入前的值记入撤销快照；单选按组快照（记录同 form 同名组全部可用项的选中状态）。撤销（undo-fill）只恢复本次写入且此后未被用户改动的字段，单选组整体恢复到写入前的选中项；快照在内容脚本内存中，页面刷新即丢失。每次应用都会替换撤销快照：若一次应用全部跳过，则无可撤销内容。字段在应用后被页面设为只读/禁用时，撤销仍会恢复用户原值（恢复原值不视为新的填写）。form 外控件无独立表单签名，由含全部表单签名的全局 scanToken 兜底。

i18n 消息 key 只允许 [A-Za-z0-9_]（Chrome 对含连字符等其他字符的 key 会拒绝加载整个扩展）；状态值到消息 key 的映射在展示层做（如 not-found → previewStatus_not_found），tests/i18n.test.tsx 有 key 合法性断言。

Background 将用户意图、最多 30000 字符的文件内容和上下文摘要放入 user 消息；system 消息明确声明页面摘要和文件是不可信数据，不能执行其中指令。预设声明支持 JSON mode 时发送 response_format json_object，否则只靠 prompt 约束；模型应返回 {"instructions": [...]}。解析器兼容顶层数组、```json 代码块围栏和前后多余文字，并丢弃没有 locator 或 value 的项。

## 服务商预设

内置 OpenAI、OpenRouter、Google Gemini、Anthropic Claude、火山方舟、阿里云百炼、DeepSeek、智谱、Moonshot、硅基流动、腾讯混元和 Ollama 预设。设置面板按国际、国内、本地、自定义分组，名称与分组通过 browser.i18n 显示；中文界面国内在前，其他语言国际在前；切换时保存并恢复各服务商的 key、model、Base URL profile。自定义选项可接入任意 OpenAI 兼容服务。每个预设声明是否支持 JSON mode、thinking 和 API Key。预设定义在 utils/providers.ts，推荐模型名只是建议值，用户可以自由输入任何模型 ID。

设置面板的“测试连接”通过 test-llm-connection 消息，用已保存的配置发送一次极小请求（20 秒超时、不重试），返回成功或状态码与简短原因，不回显 Key。

| 名称 | Base URL | 说明 |
|---|---|---|
| OpenAI | https://api.openai.com/v1 | 国际官方服务 |
| OpenRouter | https://openrouter.ai/api/v1 | 国际多模型路由 |
| Google Gemini | https://generativelanguage.googleapis.com/v1beta/openai | OpenAI 兼容 |
| Anthropic Claude | https://api.anthropic.com/v1 | 兼容层，不发送 JSON mode |
| 火山方舟（豆包） | https://ark.cn-beijing.volces.com/api/v3 | 中文新用户默认服务，支持 thinking |
| 阿里云百炼 | https://dashscope.aliyuncs.com/compatible-mode/v1 | 通义千问 |
| DeepSeek | https://api.deepseek.com | Chat/Reasoner |
| 智谱 BigModel | https://open.bigmodel.cn/api/paas/v4 | GLM，支持 thinking |
| Moonshot | https://api.moonshot.cn/v1 | Kimi |
| 硅基流动 | https://api.siliconflow.cn/v1 | 聚合模型 |
| 腾讯混元 | https://api.hunyuan.cloud.tencent.com/v1 | 混元 |
| Ollama | http://localhost:11434/v1 | 本地服务，可不填 key |
| 自定义 | 用户填写 | 任意 OpenAI 兼容接口 |

apply-preview 对每条指令独立处理，返回 filled、skipped、failed 以及开发模式下的简短跳过原因。Popup 将 filled 为零显示为警告，并提供撤销本次填写。

## 重试

OpenAI 客户端 maxRetries 为 0。FormFill 自己最多三次尝试，仅对 429、5xx、网络/超时和非法 JSON/格式重试；400、401、403、404 等立即返回状态码和简短原因。默认超时 120000ms。thinking 仅在预设支持且用户显式选择时发送，默认不发送。reasoning_effort（low/medium/high）是 OpenAI 兼容标准参数，仅在用户显式选择时发送，不受预设 supportsThinking 门控（自定义预设也支持 thinking，两者独立、可共存），默认不发送。

## 连接排障与取消（F02）

Base URL 在 Popup（填写与测试连接前）和 Background/llm-retry 双层校验：必须是完整 http(s) 地址，否则本地化报错且不发起请求（本地 Ollama 的 http 地址合法）。最终错误在状态码与服务商原因之后附可执行提示：模型输出非法（ModelOutputError，请求已到达服务商）提示重试/简化描述/换 JSON 支持更好的模型；401/403 提示检查 Key 与模型权限，404 提示对照文档检查地址与模型 ID，超时附当前配置秒数并建议调大超时或换更快模型，其余无状态码的连接失败提示检查网络/地址/本地服务是否启动。测试连接同样附提示；连接成功不代表所有控件兼容。

生成可取消：Background 维护每标签页一个 AbortController（activeGenerations）。propose-fill 启动时中止同标签页旧生成；cancel-fill 中止当前生成并返回 cancelled/idle。AbortSignal 传入 OpenAI 请求与重试退避（退避可被中止打断），取消不重试、不进入 build-preview，返回 llmCancelled。Popup 生成中显示“取消生成”按钮，用户主动取消后统一显示已取消文案；取消后迟到的预览结果也会被丢弃，不弹出预览。注册表条目只在其仍属于本次请求时清理，避免误删新请求。

## Excel 单记录选择（F05）

.xlsx 导入不再整表发送：parseExcelWorkbook 读取全部工作表（名称+原始行），弹窗提供工作表、表头行、记录行三个选择（空白行剔除后索引，选项显示行号与内容摘要）。buildRecordContent 只序列化选中表头行与选中记录行为“表头: 值”文本，其他行不进入请求；表头缺失列用 Column N 补位，空值列跳过。切换工作表重置表头/记录选择，不沿用旧主体。日期单元格按 UTC 渲染 YYYY-MM-DD（含时间的保留完整 ISO）；纯日期单元格由 read-excel-file 构造为 UTC 午夜（parseExcelDate），午夜判断据此成立，换解析库须重新核实。文本前导零由解析库保留，Excel 内已丢失格式的数值无法恢复。越界或表头=记录行抛本地化错误。parseFile 的 xlsx 整表分支已删除（调用即报错），防止未来调用方静默整表外发；docx/txt/md 仍整文发送（30,000 字符截断不变）。选择状态仅存弹窗内存。

## 人工确认的映射复用（F06）

utils/mappings.ts 保存用户确认的“Excel 列 → 表单字段”映射：仅 origin、formSignature、列名/列索引、locator、label，sanitizeMapping 白名单清洗，拒绝保存任何值/Key/页面文字；本地存储 field-mappings（v1），上限 20 条按 createdAt 淘汰。保存入口在 Excel 填写成功后的预览结果区，需 window.confirm 明确同意；列索引按“写入值与记录单元格精确相等”反查，编辑过且无法对应的项不保存。

复用不调用模型：Popup 取当前表单签名，签名不匹配则不提供映射（结构变更停用）；匹配时用当前选中记录的 recordValues 构造指令，走 build-preview 重新定位与保护判定，replayMapping 按 locator 匹配且仅保留 formSignature 一致的 ok 项，值只来自当前记录。结果仍进入预览→应用→可撤销流程。映射列表按 origin+当前签名过滤展示，可逐条删除。已知局限：列索引按值反查，若两列当时同值则取首个匹配，后续两列分化时复用可能取错列（预览可核对）；用户编辑过且无法与单元格对应的项不保存。此功能不是 F01 暂停的隐式历史采集：无提交监听、无自动采集、无历史值外发。

## 学习模式暂停（F01）

v5 迁移只关闭旧 learningModeEnabled，保留 provider/profile/model/baseUrl/apiKey。读取和保存设置也强制关闭。Content 不绑定 submit 监听，Background 无历史样本读取或拼接分支，旧 flag=true 不能绕过。utils/learn.ts 删除采集/匹配 API，仅保留签名与本地查看/清除；查看时清理损坏、过期（90 天）和超额（50 条）记录。

后台在扫描前检查 privacyNoticeAccepted；Popup 按序保存配置，在数据发送确认落盘后才发起请求。React state updater 无存储副作用。历史查看内容仅作为 React 文本渲染，不执行 HTML，也不外发。更新扩展后旧页面需要刷新，才能替换旧 content script。冻结时没有设计独立同意与可靠复用方案；此版本不宣称学习功能可恢复或继续开发。

## 目录

- entrypoints/content.ts：扫描与受保护的填写。
- entrypoints/background.ts：协调消息、设置、数据发送确认和 LLM。
- entrypoints/popup/components/：设置、输入和状态组件。
- utils/llm-retry.ts：模型请求、重试和输出校验。
- utils/settings.ts：设置存储（v6）与迁移：v2 调整超时并关闭学习模式，v3 根据已存 Base URL 推断服务商并生成 profile，v4 保留已存配置并补齐历史缺失项；v5 关闭旧学习标记；v6 引入可选 reasoningEffort 并保留已存值。
- utils/providers.ts：服务商预设、Base URL 匹配和切换服务商时的 profile 保存/恢复。
- utils/learn.ts：表单签名、旧记录的本地查看/删除与读取时清理。


## 双语默认值与迁移

`getDefaultSettings(language)` 是纯函数：语言以 zh 开头时取 ark，否则取 openai；baseUrl 和 model 分别来自预设地址和 models 第一项。浏览器语言仅在 Popup 初始化、读取和保存设置时通过 `browser.i18n.getUILanguage()` 读取，不在模块加载时调用，保证 WXT 构建可加载入口。读取仍按“默认值 + 已存值”合并；新用户无存储值，不因读取而写入设置。

旧版保存函数通常写入完整配置，但 v1/v2 没有 providerId，存储值也没有运行时完整性校验。因此曾升级 local:user-settings 到 v4。v3 迁移先处理缺字段，再按 Base URL 识别服务商；v4 对已存 providerId/baseUrl/model 原样保留（包括空字符串、自定义地址和模型），只对缺失项补齐旧版 Ark 回退值，缺失 providerId 按地址识别，未知地址归自定义。语言变化不会重选老用户服务商，providerProfiles 保留。迁移测试直接预置 v1/v2/v3 存储和元数据，再通过 getSettings 验证执行到 v4。本节描述 v1→v4 的历史迁移设计；冻结代码的当前设置存储版本为 v6。

## 提示词、文案与错误

中英文 UI 使用 public/_locales/en 与 zh_CN 的相同 key。模型提示词只有一份英文版本，不随 UI 语言复制；要求填写值与表单及用户输入语言一致。system 明确表单摘要、文件和历史样本是不可信数据，忽略其中指令。user 消息保留 `BEGIN/END UNTRUSTED FORM SUMMARY`、`BEGIN/END UNTRUSTED FILE CONTENT` 分隔标记。历史样本不再拼接；system 对不可信历史的防御性要求仍保留。

空内容、非法 JSON、错误结构抛出带稳定 code 的 `ModelOutputError`，显示文案走 i18n；重试使用错误类型，不匹配中文错误文字。HTTP 状态码及现有网络/超时识别行为保持不变，最多三次尝试。外部服务商返回的简短错误原因保留原文，本地失败前缀和未知错误已国际化。

## 实际隐私边界

扫描摘要还包括 className、title、ariaLabel、select 选项及 radio/checkbox 的 value 属性，不包含完整 HTML 或普通输入框的独立当前值字段。附近文字来自最多三层父元素，扫描时截取 120 字符并可能追加省略号，发送前再次截到 120 字符；一次请求最多 200 个控件。附近文字可能仍含敏感信息。文件内容上限为 30000 字符，另附文件名和截断说明。

自动学习和历史外发已暂停，旧记录仅本地查看和清除。90 天过期在查看时清理，不是定时清理。标签/附近文本剔除全部表单控件内容、可编辑区域、脚本和隐藏内容；扫描与写入均拒绝密码、隐藏、禁用、只读及祖先隐藏的控件。双语隐私政策详细列出实际传输字段、API Key 的本地保存与请求认证用途，以及无开发者服务器、无统计、无广告和不出售数据。

商店文案与申报建议位于 docs/store-listing.md；冻结前曾计划国内主要通过 Edge 扩展商店分发，但从未提交。README.md 与 README.zh-CN.md 内容对等。
