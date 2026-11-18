# 第三方素材台账

本库文字内容为 fangd123 原创或改编，按 CC BY-NC-SA 4.0 发布。第三方素材不适用该许可，按下列规则逐项登记。

## 登记规则

- 任何第三方图片、图表、引文、代码、数据，都必须在本表登记后才能进入公开库。
- 新闻图片默认按"版权保留"处理，不登记即视为不可用。
- CC BY-SA 素材不能并入 CC BY-NC-SA 作品（许可不兼容）；确需使用时单独标注并保留原许可，或不用。
- 引文只保留必要的短句，并给出作者、作品、链接、访问日期。
- 2026-11-18 第四批起恢复自产配图（见下节）；前三批为纯文字版，原文图片位置已转为图注或移除。

## 配图（第四批，2026-11-18）

第四批把 326 篇已发布文章引用的 **286 张自产配图**随文恢复进公开库。来源核查：抽检 11 张覆盖全部系列，全部为作者自产——AI 生成插画（部分保留生成工具水印）、自制图表与手绘 SVG；无第三方素材，故台账无新增条目。

处理：sharp 压缩（插画类 PNG → mozjpeg q85、图表类 → palette PNG q90、公众号导出 jpg 保原样避免二次损失、SVG 直通；宽度 ≤2000px），再编码一律剥除 EXIF/GPS；总量 140.3MB → 41.2MB。

第三方图片准入三档（2026-11-18 作者确认）：
1. **自产图**（AI 生成插画、自制图表、自己产品截图）→ 直接发布；
2. **网页截图、论文图表** → 按《著作权法》第 24 条"适当引用"（注明来源与作者、数量适度、限于评论/说明语境）发布，并在本台账逐项登记；
3. **新闻通讯社图片** → 引用不能替代授权，仍按"版权保留"排除。

## 配图（第五批，2026-11-18）

分类重整补入的新文章（费尔巴哈提纲、AI交互设计2026版、公众号政经随笔等 26 篇）带进 **86 张新配图**，公开库配图共 372 张。抽检确认：公众号随笔插图与第四批同族（自产 AI 插画，无第三方素材）；**AI交互设计2026版 的 3 张外部参考图**首次启用三档规则的第二档（开放许可图片逐张登记，见下表）。压缩规则同第四批，总量 155.8MB → 51.5MB。

三张外部图均为**未修改或仅缩放**的转载（不构成改编，ShareAlike 义务不触发）；其许可与本库文字的 CC BY-NC-SA 相互独立，转载时请按各图原许可保留署名。

## 台账

| 素材 | 作者/机构 | 来源链接 | 许可 | 用在哪 | 改动 | 备注 |
|---|---|---|---|---|---|---|
| Active Badge 系统设备照片 | Roy Want（Xerox PARC） | [Wikimedia Commons: File:Active_Badge.png](https://commons.wikimedia.org/wiki/File:Active_Badge.png) | CC BY-SA 4.0 | AI交互设计2026版/03（`03-02-active-badge.png`） | 未修改 | Weiser 普适计算一节的史料配图 |
| Sutherland 操作 Sketchpad 照片 | Ivan Sutherland，Kerry Rodden 扫描 | [Wikimedia Commons: File:SketchpadDissertation-Fig1-2.tif](https://commons.wikimedia.org/wiki/File:SketchpadDissertation-Fig1-2.tif) | CC BY-SA 3.0 | AI交互设计2026版/04（`04-03-sketchpad-sutherland.jpg`） | Commons 提供缩放版 | 直接操控史料配图 |
| 《Generative Interfaces for Language Models》图 1 | Jiaqi Chen, Yanzhe Zhang, Yutong Zhang, Yijia Shao, Diyi Yang | [ACL Findings 2026](https://aclanthology.org/2026.findings-acl.74/) | CC BY 4.0 | AI交互设计2026版/05（`05-04-generative-interfaces-fig1.png`） | 从发表版 PDF 裁取图幅 | 论文图表适当引用 |

## 已在私有库中确认、但未进入首批的素材类型

| 类型 | 处理 |
|---|---|
| 播客/YouTube 音频与字幕（Acquired、Latent Space、BBC、CNBC 等） | 永不进入公开库 |
| B 站视频转写与截图（小Lin说等） | 永不进入公开库；只保留短引文与链接 |
| 新闻图片（Tucson Spotlight、Tom's Guide 等） | 默认版权保留，不进入公开库 |
| Wikimedia Commons 照片（CC BY / CC BY-SA / CC0 / 公有领域） | 如使用，逐张登记作者、文件页、许可、改动 |
| OpenAI / Anthropic 文档与论文全文 | 不镜像全文，只保留摘要、链接与短引文 |
| GitHub 上的第三方 repo | 不整包复制，改为链接；如引用代码，按原 LICENSE 登记 |