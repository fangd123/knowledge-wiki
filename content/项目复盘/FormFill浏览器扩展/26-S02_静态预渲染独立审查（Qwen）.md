---
title: "S02 静态预渲染独立审查（Qwen）"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: 项目复盘
series: FormFill浏览器扩展
series_order: 26
tags:
  - 项目复盘
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# S02 静态预渲染独立审查（Qwen）

- 日期：2026-11-17；审查人：Qwen（/root/qwen_report_review，只读审查 + 本地复跑 build）
- 范围：基线 `7c00226` 之上未提交改动（website/scripts/prerender.mjs、package.json build 脚本、seo-preparation/CHANGELOG/台账、验证脚本与 JSON）。
- 结论：**同意提交**。阻塞项（C 编号）：无。方案正确、注入安全、build 整体幂等、验证充分、文档口径如实。2 项非阻塞备注（D1/D2）。

## 一、方案正确性（问题 1）

### createRoot 替换 vs hydrate——取舍正确
- 四个入口（main/privacy-main/setup-main/excel-main）均用 `createRoot(...).render()`，挂载时**整体替换**注入的静态 markup，非 hydrateRoot。因此不存在水合不匹配（hydration mismatch）风险——React 不尝试复用静态 DOM，直接重建。验证证据 h1Count=1、duplicated=false、零 pageerror 实证无重复渲染。
- 取舍合理：hydrate 能保留静态 DOM 提升首屏性能但要求服务端/客户端 markup 严格一致（易因浏览器差异/时间戳报错）；本站四页纯静态文案、无服务端动态数据，createRoot 替换更简单稳健，首屏已有预渲染内容可见（SEO/无 JS 目标达成），React 接管后交互正常。对当前规模是正确选择。

### 注入内容安全性——可信
- 注入的是 `document.getElementById('root').innerHTML`，来自**我们自己页面**在真实 Chromium 的渲染产物，无外部不可信内容、无用户输入。静态服务仅读 dist 目录（path.startsWith(dist) 守卫防目录穿越，403）。无 XSS 注入面。

### build 幂等性——整体幂等，单步非幂等（已实证）
- 我本地复跑 `npm run build` 成功（vite build 重新生成含空 `<div id="root"></div>` 标记的 HTML → prerender 注入）。**重复 npm run build 幂等**，因 vite 每次重置 dist。
- 我单独重跑 `node scripts/prerender.mjs`（不先 vite build）→报错 `root marker not found (already prerendered?)`。这是脚本的**显式守卫**（marker 不存在即 throw），防止对已注入文件重复注入造成嵌套。设计正确：单步非幂等但 fail-fast 报错明确，不会静默损坏。
- D1（非阻塞备注）：脚本注释/文档宜注明"prerender.mjs 必须在 vite build 之后运行（npm run build 已保证）；单独重跑会因标记已替换而报错，需先重新 vite build"。当前 package.json build 串联已保证正确顺序，报错信息也已提示，无需改码。若未来有人单独调用脚本可能困惑，文档注明即可。

## 二、验证充分性（问题 2）——充分

- s02-prerender-check.json：四页 staticOk=true（无 JS 直读含主文案与链接，mustContain 逐页断言）；有 JS 时 h1Count=1、rootChildren=4、duplicated=false（唯一正文句恰好出现一次）、零 pageerror。覆盖"静态可读+交互无重复+无错误"三个关键维度。
- content-visibility:auto 的真实发现处理正确：innerText 不含视口外节是渲染优化（styles.css:466），检查改用 textContent 覆盖全文做重复判定——这不是缺陷，搜索引擎抓取 DOM 不受 content-visibility 影响（DOM 节点仍在）。脚本注释如实说明，口径正确。
- s02-layout-check.json：4 页×9 断点 36 组 0 失败（我解析确认），预渲染后布局无回退。w06-a11y-check.json axe 重跑 allPass=true。网站 typecheck/build 过（build 含预渲染，注入 3.2k–7.1k 字符）。
- D2（非阻塞备注）：验证用 Playwright Chromium，未测真实搜索引擎爬虫抓取（Googlebot 渲染机制不同）；但静态 HTML 含主文案已满足"无 JS 可读"的核心目标，爬虫渲染是上线后 S04 观测项。当前验证对 S02 范围充分，无需本批补爬虫测试。

## 三、文档口径（问题 3）——如实

- seo-preparation S02 改"预渲染已完成；sitemap 待域名"，准确区分已完成（预渲染）与阻塞（sitemap/canonical 待域名）；发布前清单第 3 条改"已就绪上线后复核"，不宣称线上已验证。S05 真实结果已补更新。CHANGELOG 文档与验证节新增条目准确。台账 S02 行同步。
- sitemap/canonical 仍如实标注阻塞于域名，无臆造。口径一贯诚实。

## 四、验证与限制

- 我本地复跑 npm run build 成功（四页注入），单独重跑 prerender 实证 fail-fast 守卫；解析 s02-layout-check.json 36 组 0 失败、w06-a11y-check allPass。
- 预渲染验证为 Playwright Chromium 本地静态检查，非真实爬虫/线上环境；上线后直读复核与 GSC 观测仍待域名（S01/S04），口径如实。

## 五、结论

**同意提交**。C 编号阻塞项：无。D1（脚本单步非幂等宜文档注明）、D2（爬虫抓取属上线后观测）非阻塞。createRoot 替换取舍正确、注入安全、build 整体幂等且单步 fail-fast、验证覆盖静态可读/交互无重复/布局/a11y，S02 不依赖域名的缺口正式闭合，sitemap/canonical 仍诚实待域名。

——交父会话汇总；未改仓库源码与既有报告，未执行 Plane，未提交。
