---
title: "研究-18-竞品worker收尾"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 52
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

已完成研究并写入：

- [竞品报告 competitors.md](36-FormFill_竞品与同名品牌核查.md)
- 一手来源核验清单 competitor-sources.json

报告覆盖 **8 个产品、7 个已读取的 Chrome 商店条目**，另核查两个同名域名，明确区分直接同类与邻近替代。

最重要的发现：

1. **存在明确同名冲突。** `formfill.cc` 已有独立扩展；`formfillai.cc` 也宣传 BYOK、本地存储与语义填表，但安装按钮均指向 `#`，真实分发状态未知。都不能认作本项目。
2. **BYOK、本地保存不能单独作为独有优势。** 更可信的定位是中文日常资料、Office 文件导入、可选模型服务与透明的数据发送边界。
3. **首页存在能力误述。** `.doc` 实际只提示转换，因此“5 类文件导入”应改为四种可解析格式。
4. **旧参考对象已转向。** Magical 官网主推医疗运营 agents，Bardeen 主推获客抓取；不宜继续作为直接同类。
5. 优先对标 **FillApp 的产品演示、Text Blaze 的转化结构、formfill.cc 的直接工作流与品牌区分**。

目录保留任务指定的 `2026-11-16`；报告将实际核验日期按当前环境记录为 `2026-11-17`。未修改应用源码、未发布或联系他人、未执行 Plane log、未再委派。