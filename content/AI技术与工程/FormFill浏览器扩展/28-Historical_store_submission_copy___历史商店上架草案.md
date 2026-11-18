---
title: "Historical store submission copy / 历史商店上架草案"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 28
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# Historical store submission copy / 历史商店上架草案

> **Freeze notice (2026-11-18): FormFill has stopped productization and is archived.** This copy was never submitted to any store and will not be submitted. Store links, accounts, screenshots and launch tasks are not pending work. See the [product wrap-up and handoff report](22-FormFill_产品收尾与交接报告.md).
>
> **冻结说明（2026-11-18）：FormFill 已停止产品化并归档。** 本文文案从未提交任何商店，也不会再提交；商店链接、账号、截图上架等均不是待办。详见[产品收尾与交接报告](22-FormFill_产品收尾与交接报告.md)。

Draft for 0.2.0; publication is cancelled. English and Chinese describe the archived functionality. International distribution was previously planned for the Chrome Web Store, and Microsoft Edge Add-ons was planned as the primary channel for users in China; neither plan was executed. 0.2.0 上架草案，发布已取消。中英文介绍对应冻结前功能。原计划海外通过 Chrome Web Store，并计划主要经 Microsoft Edge 扩展商店触达国内用户；两项均未执行。

This draft was written before the later settings-v6 `reasoning_effort` option and does not describe that option. The README and privacy policy record the archived behavior. / 本草稿写于后来加入的设置 v6 `reasoning_effort` 选项之前，未覆盖该选项；归档行为以 README 与隐私政策为准。

## English listing

**Name:** FormFill

**Short description (96 characters, limit 132):**

Fill web forms from your instructions or files, using your choice of local or cloud AI provider.

**Detailed description:**

Turn a short description or an imported file into form-fill suggestions you review before anything is written. FormFill uses the AI provider you choose, supports English and Chinese interfaces, and leaves you in control of applying, undoing, and submitting each form.

- Describe the information to enter, or import Excel (.xlsx), Word (.docx), text, or Markdown files up to 10 MB. Legacy .doc files may need conversion to .docx. For Excel, pick a worksheet, header row, and one record; only that record is sent. Up to 30,000 characters of parsed file content are sent per fill.
- Preview every suggested value before applying: edit values, choose which fields to write, and keep existing values unless you opt in. Protected fields (password/hidden/disabled/read-only) are never filled.
- Fill standard text fields, dropdowns, checkboxes, and radio buttons, including controls outside a form element. Undo the last fill for fields you have not edited since, then check the page before submitting.
- Save a confirmed "Excel column → form field" mapping and reuse it for the next record without another model call. Mappings store structure only, never values or keys, and stop being offered when the form changes.
- Choose OpenAI, OpenRouter, Google Gemini, Anthropic Claude, Volcengine Ark (Doubao), Alibaba Cloud Model Studio (Qwen), DeepSeek, Zhipu BigModel (GLM), Moonshot (Kimi), SiliconFlow, Tencent Hunyuan, local Ollama, or a custom OpenAI-compatible API. Preset availability depends on the provider and your account; endpoint and model are editable.
- For new users, Chinese browser UI languages default to Ark; other languages default to OpenAI. Everyone can switch to any provider. Each provider's settings are saved separately.
- Test your connection and adjust the model, endpoint, timeout, and supported thinking mode.
- Automatic learning is paused. No submission values are collected and no history is sent. View or delete legacy local records in Settings; viewing removes records older than 90 days and keeps at most 50. Reload existing pages after updating.

Bring your own provider access and API key where required. Provider charges and network availability depend on your service. Form summaries, your instructions, and imported content go directly to the configured endpoint, without a developer server. API keys are stored locally and used for provider authentication. No analytics, ads, or data sales.

This archived copy was used only for the pre-freeze prototype. Distribution through the Chrome Web Store and Microsoft Edge Add-ons was planned; neither was submitted. FormFill does not automatically submit forms. Some sites and browser-internal pages are unsupported. Read the privacy policy before sending personal information. Permission to access pages is used for frozen-prototype scanning and filling only.

## 中文商店介绍

**名称：** FormFill

**短描述（不超过 132 字符）：**

使用你选择的本地或云端 AI 服务商，根据描述或文件填写网页表单，中英文界面同等支持。

**详细描述：**

将简短描述或导入的文件转换成填写建议，先预览、再写入。FormFill 使用你选择的 AI 服务商，支持中英文界面，由你决定应用、撤销并提交每一张表单。

- 描述需要填写的信息，或导入最大 10 MB 的 Excel（.xlsx）、Word（.docx）、文本和 Markdown 文件。旧版 .doc 可能需要转为 .docx。Excel 可选工作表、表头行和一条记录，只发送选中记录。每次填写发送最多 30,000 字符的解析内容。
- 应用前逐项预览建议值：可修改、可勾选；已有值的字段默认不覆盖，勾选后才写入。密码、隐藏、禁用、只读等受保护字段不会被填写。
- 填写标准文本框、下拉框、复选框、单选框，包括 form 元素外的控件。可撤销最近一次填写中你未改动过的字段，提交前请检查网页。
- 确认保存“Excel 列 → 表单字段”映射后，下一条记录可免调用模型复用。映射只存结构，不存值和 Key；表单变化时自动停用。
- 支持 OpenAI、OpenRouter、Google Gemini、Anthropic Claude、火山方舟（豆包）、阿里云百炼（通义千问）、DeepSeek、智谱 BigModel（GLM）、Moonshot（Kimi）、硅基流动、腾讯混元、本地 Ollama、自定义 OpenAI 兼容 API。预设可用性取决于服务商和账号；接口和模型可修改。
- 新用户的浏览器界面语言为中文时默认火山方舟，其他语言默认 OpenAI。所有人均可切换全部服务商，各服务商的设置分别保存。
- 可测试连接，调整模型、接口、超时及服务商支持的深度思考模式。
- 自动学习已暂停：不采集提交值、不发送历史。设置中可查看或删除旧本地记录，查看时清理超过 90 天的记录、最多保留 50 条；更新后请刷新已打开的网页。

需要自行准备服务商访问权限及必要的 API Key。费用和网络可用性由所选服务决定。表单摘要、描述和导入内容直接发送到配置的接口，不经过开发者服务器。API Key 仅本地存储，用于服务商认证。无统计、无广告、不出售数据。

本草稿仅用于冻结前样机。原计划通过 Chrome Web Store 与 Microsoft Edge 扩展商店分发，但两者均未提交。FormFill 不自动提交表单。部分站点与浏览器内部页面不受支持。发送个人信息前请阅读隐私政策。网页访问权限仅用于冻结前样机的扫描与填写。

## Single purpose / 单一用途

**English:** Help users fill the current web page's form fields from their instructions and imported files through a user-selected AI service. File import and provider settings support this form-filling purpose.

**中文：** 使用用户选择的 AI 服务，根据描述和导入文件填写当前网页表单。文件导入、服务商设置均服务于这一填表用途。

## Permission justifications / 权限理由

| Permission / 声明 | English justification | 中文理由 |
|---|---|---|
| `storage` | Store settings, API keys, per-provider profiles, privacy acknowledgement, and retained legacy records in local extension storage. | 在扩展本地保存设置、API Key、各服务商配置、隐私确认和保留的旧记录。 |
| `activeTab` | Identify and access the active page following user interaction, send scanning/filling messages to its content script. | 用户操作后识别和访问当前页面，向内容脚本发送扫描和填写消息。 |
| Content script `matches: ["<all_urls>"]` | Form filling must work across user-chosen websites, whose addresses cannot be listed in advance. The script registers scanning/filling handlers, and adds highlighting styles. It does not listen for submissions. Model requests are initiated by the user. This is broad matching, not an additional named permission or on-demand injection. | 用户要填写的站点无法预先穷举。脚本注册扫描与填写处理器、添加高亮样式，不监听提交事件。模型请求由用户发起。这是广泛的内容脚本匹配，不是额外命名权限，也不是按需注入。 |

No `tabs`, `scripting`, or separate `host_permissions` declaration is currently present. Do not claim that `activeTab` restricts the already-declared `<all_urls>` content script. / 当前未声明 `tabs`、`scripting` 或单独的 `host_permissions`；不要宣称 `activeTab` 限制了已经声明的 `<all_urls>` 内容脚本范围。

## Chrome Web Store data-use declaration suggestions / 数据使用声明勾选建议

These are historical submission suggestions based on data processed and transmitted, not only developer-held data. They were never executed and must not be reused for a live submission without a new review. Direct transfer to the user's provider still needs disclosure. / 以下是历史申报建议，以扩展实际处理和传输的数据为准，不只统计开发者持有的数据。它们从未执行；若未来另有授权提交，必须重新核对商店现行定义和实际用途。直传用户选择的服务商仍需披露。

| Category / 类别 | Suggested selection / 建议 | Reason / 理由 |
|---|---|---|
| Website content / 网站内容 | Yes / 是 | Form metadata, nearby text, and options are sent to the model. / 表单元数据、附近文字和选项会发给模型。 |
| Personally identifiable information / 个人身份信息 | Yes / 是 | Descriptions, files, and retained values can include names, email addresses, and other personal information. / 描述、文件及保留值可能含姓名、邮箱等个人信息。 |
| Authentication information / 身份验证信息 | Yes / 是 | API keys are stored locally and sent to the selected endpoint for authentication. / API Key 本地保存并用于接口认证。 |
| Web history / 网络浏览记录 | Conservative yes with limited scope / 保守选择“是”并限定范围 | Legacy local samples may retain origins, but are not transmitted. No general browsing-history tracker or new history collection exists. / 旧本地样本可能保留站点来源，但不传输；没有一般浏览历史追踪器，也没有新的历史采集。 |
| User activity / 用户活动 | Current definitions needed review; no submit monitoring / 需按当时定义核对；不监听提交 | Automatic learning is paused; no submit listeners or analytics of clicks or keystrokes. / 自动学习已暂停，没有提交监听、点击统计或按键统计。 |
| Health information, financial/payment information, personal communications / 健康、财务支付、私人通信 | Manual scope review; selection depended on applicability / 需人工核对范围，选择取决于是否适用 | No dedicated collection or feature targets these categories, but arbitrary descriptions, files, or page text can include them. There is no general anonymization of request inputs. The draft warned that a claim that these data could never be transmitted would be unsafe. / 无专门收集功能，但描述、文件或页面文字可能包含这些数据；请求输入没有完整匿名化保证。草案明确提示，声称这些数据绝不会传输并不稳妥。 |
| Location / 位置信息 | Manual scope review / 需人工核对 | No geolocation API is used; providers see request IP metadata, and user-entered addresses may be sent. The historical selection depended on the store's definition. / 不调用地理定位 API；服务商可见请求 IP 元数据，用户输入的地址可能被发送。当时的选择取决于商店定义。 |

Historical purpose recommendation: **App functionality**. The draft advised against advertising, analytics, and data resale. It would have required certifications that data were not sold or transferred outside permitted uses, used for an unrelated purpose, or used for creditworthiness or lending, but only after confirming that the actual release and disclosures met those statements. The draft also stated that transfer to the selected provider for requested form filling had to be disclosed and that “no third-party transfer” could not be claimed. / 历史用途建议为**应用功能**。草案建议不选择广告、分析或数据转售；只有在确认实际发行版与披露符合相应声明后，才应完成“未在允许用途外出售或转移”“未用于单一用途之外”“未用于信用评估或贷款”等认证。草案同时说明，为填写而向所选服务商传输必须披露，不能宣称“无第三方传输”。

All extension logic is bundled; there is no remotely hosted executable code. Model JSON is interpreted as locator/value data, not executed as JavaScript. / 扩展逻辑均打包在本地，无远程托管可执行代码；模型 JSON 作为定位与填写值处理，不作为 JavaScript 执行。

## Historical release checklist / 历史发布前确认（不执行）

冻结时以下事项均未执行；它们不再构成清单、待办或发布承诺：

- 公开支持渠道和商店联系方式未提交核对。
- Public support channels and store contact details were not submitted or checked.
- 两份隐私政策的中英文版本只存在于仓库档案，没有稳定公开 URL。
- The English and Chinese privacy policies existed only in the repository archive and had no stable public URL.
- 各服务商接口与模型目录没有在商店提交前重新认证。
- Provider endpoints and model catalogues were not revalidated before a store submission.
- 中英文商店截图、图标终稿、支持信息和发布包均未提交。
- English and Chinese screenshots, final icons, support information, and release packages were not submitted.
- Chrome 与 Edge 的数据使用声明没有向任何商店提交确认。
- Data-use declarations for Chrome and Edge were not confirmed with either store.
- 0.2.0 从未发布，项目从未上架或部署。
- Version 0.2.0 was never released, and the project was never listed or deployed.
