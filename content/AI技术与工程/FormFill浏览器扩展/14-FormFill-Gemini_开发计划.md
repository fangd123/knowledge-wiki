---
title: "FormFill-Gemini 开发计划"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 14
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# FormFill-Gemini 开发计划

本项目将遵循 `GEMINI.md` 中定义的测试驱动开发（TDD）和文档驱动的开发流程。
每个阶段的核心任务在开始编码前，都会先编写对应的测试用例。

---

## 阶段一：项目基础架构与环境设置 (Phase 1: Core Infrastructure & Setup)

- [x] **任务 1.1：依赖验证与安装**
  - [x] 确认 `wxt` 框架已正确初始化。
  - [x] 安装 `openai` 库用于与 LLM API 交互。
  - [x] 安装并配置测试框架（`vitest`），确保其与 WXT 环境兼容。
  - [x] 安装并配置 `prettier` 和 `eslint` 以统一代码风格。

- [x] **任务 1.2：用户配置管理**
  - [x] **TDD:** 编写测试用例，模拟存储和读取 API Key, Model Name, Base URL。
  - [x] **实现:** 创建一个工具模块 (`/utils/settings.ts`)，使用 `chrome.storage.local` 来安全地存储和检索用户配置。
  - [x] **UI:** 在弹出页 (`/entrypoints/popup/App.tsx`) 中创建一个简单的设置表单，允许用户输入和保存他们的配置。

---

## 阶段二：表单识别与数据提取 (Phase 2: Form Detection & Data Extraction)

- [x] **任务 2.1：Content Script 表单扫描**
  - [x] **TDD:** 编写测试用例，用模拟的 HTML 结构来验证表单字段（`input`, `textarea`, `select`, `button` 等）是否能被准确识别。
  - [x] **实现:** 在 `entrypoints/content.ts` 中，编写逻辑来扫描页面 DOM，提取所有可用的表单元素及其相关信息（如 `id`, `name`, `type`, `aria-label`）。
  - [x] **数据结构:** 将提取的表单信息序列化为一个简洁的 JSON 结构，准备发送给后台脚本。

- [x] **任务 2.2：通信建立**
  - [x] **TDD:** 编写测试用例，模拟从 Content Script 向 Background Script 发送消息并接收响应。
  - [x] **实现:** 建立 Content Script 和 Background Script 之间的消息传递通道。当用户在 Popup 中触发操作时，Popup 通知 Background Script，后者再请求 Content Script 获取表单数据。

---

## 阶段三：LLM 集成与智能填充 (Phase 3: LLM Integration & Smart Filling)

- [x] **任务 3.1：后台 LLM 服务**
  - [x] **TDD:** 编写测试用例，模拟调用 LLM API 的场景。使用 `.env` 文件中的测试密钥进行测试。
  - [x] **实现:** 在 `entrypoints/background.ts` 中，实现与 OpenAI (或兼容) API 的交互。函数应接收用户自然语言输入和 Content Script 发来的表单结构。

- [x] **任务 3.2：Prompt Engineering**
  - [x] **设计:** 精心设计 Prompt，指导 LLM 将用户的自然语言输入（例如：“帮我注册一个账号，用户名是 test，密码是 123456”）转化为与表单结构相匹配的 JSON 对象。
  - [x] **实现:** 将设计的 Prompt 集成到后台脚本的 API 调用逻辑中。

- [x] **任务 3.3：字段映射与填充指令**
  - [x] **TDD:** 编写测试用例，验证 LLM 返回的结构化数据能否被正确地映射回原始的表单字段。
  - [x] **实现:** 编写逻辑，解析 LLM 的响应，并生成一个清晰的“填充指令”列表（例如：`[{fieldId: 'username', value: 'test'}, {fieldId: 'password', value: '123456'}]`）。
  - [x] **通信:** 将此填充指令发送回 Content Script。

---

## 阶段四：端到端流程整合与 UI 优化 (Phase 4: E2E Flow & UI Refinement)

- [x] **任务 4.1：执行表单填充**
  - [x] **TDD:** 编写测试用例，模拟 Content Script 接收填充指令并更新 DOM 中的表单字段。
  - [x] **实现:** 在 `entrypoints/content.ts` 中，实现接收后台脚本的指令并将其应用到网页表单上的逻辑。

- [x] **任务 4.2：Popup 用户界面**
  - [x] **UI 实现:** 在 `/entrypoints/popup/App.tsx` 中，实现核心用户界面：一个文本区域供用户输入自然语言，一个“填充”按钮。
  - [x] **状态管理:** 添加加载（Loading）、成功（Success）、失败（Error）等 UI 状态，向用户提供清晰的操作反馈。

- [x] **任务 4.3：端到端测试**
  - [x] 在真实的网页（如注册、登录页面）上手动测试整个流程，确保所有部分协同工作正常。

---

## 阶段五：文档、打包与发布 (Phase 5: Documentation, Packaging & Release)

- [x] **任务 5.1：更新文档**
  - [x] 更新 `README.md`，添加详细的用户使用说明、功能介绍和截图。
  - [x] 检查并完成所有代码注释和文档字符串。

- [x] **任务 5.2：打包与测试**
  - [x] 运行 `npm run build` 命令，生成生产环境的插件包。
  - [x] 在 Chrome 中加载打包后的插件，进行最终测试。

- [ ] **任务 5.3：版本提交**
  - [ ] 为当前版本创建一个 Git tag。
  - [ ] （可选）准备提交到 Chrome Web Store 的相关材料。
