---
title: "FormFill 竞品与同名品牌核查"
date: 2026-11-18
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: FormFill浏览器扩展
series_order: 36
tags:
  - AI技术与工程
  - FormFill浏览器扩展
ai_assisted: true
status: published
---

# FormFill 竞品与同名品牌核查

资料目录记为 2026 年 11 月 16 日，作者实际读取官网、政策和商店正文是在 17 日（新加坡时间）。最初八个完整对标对象中，读到七个商店条目；另两个同名域名没有足够分发证据，不算已验证产品。后来另一次研究补上 Bardeen，并扩到[十三条商店快照](42-插件分发、开源与社区量化：市场表现补充.md)。七和十三是两个取样阶段，并不矛盾。

## 对比的是哪个 FormFill？

| 比较维度 | 本项目：11 月 17 日审计时的样机 | 容易混淆的外部证据 |
|---|---|---|
| 获取入口 | 只有源码构建说明，没有自有商店条目。 | 名叫 FormFill AI 的商店条目回链另一域名，不是本品。 |
| 输入和费用 | 文字或 `.xlsx`、`.docx`、`.txt`、`.md`，自选模型服务并自行付费。早期首页误列旧 `.doc`。 | 竞品“免费安装”可能仍有配额；credits 也不是表单份数。 |
| 数据去向 | 资料可能发给所选模型，不经本品后端；当时旧学习行为涉及本地历史与可能的后续外发，不能宣传绝不出设备。 | 竞品政策与商店声明说明厂商怎样描述服务，不是网络实测。 |

早期比表曾把支持表单外控件误写成可处理 Shadow DOM。后经[总览审查](19-产品总览（docs_product-overview.md）审查报告.md)更正。预览、单记录、学习暂停及最终权限见[冻结版总览](20-FormFill_产品总览：冻结前功能、特色、状态与历史计划.md)，不倒填进这次比较。

## 八个对标对象：工具不同，抢的是同一段录入时间

| 对象 | 分类与当前状态 | 使用方式 / key 门槛 | 文件输入证据 | 已读到的价格 | 商店核验 |
|---|---|---|---|---|---|
| formfill AI，formfill.cc | 直接同类；独立开发者的侧边栏 AI 填表产品 | 要登录其站点以调用 AI；存在日配额，BYOK 未知 | 官网与商店只明确粘贴文本、模板；文件未知 | 官网称可免费安装；额度与付费价格未知 | 已读；ID `faepndggpldacododbleilmkbfedkkpo` |
| FillApp | 直接同类，但范围扩至 PDF 与浏览器任务代理 | 产品额度模式；常规使用未要求用户 API key；另有连接 ChatGPT / Claude 的路径 | 明确文档附件、PDF / 图片读取；完整后缀白名单未知 | 20 免费 credits；$0.10/credit，最低 $5；订阅展示 $14.99/月 | 已读；ID `mdoaagdccddnbjncalegjfemhnlijgbo` |
| Filliny | **商店扩展属直接同类；当前官网属求职邻近平台** | 建个人资料；未见必须自备 key 的要求 | 当前平台上传简历；旧扩展支持哪些文件未知 | 官网有免费起步；定价页只返回 Loading，具体金额未知 | 已读；ID `ijhjngflokdbhimpcjhgjkljlfjenjlg` |
| Text Blaze | 邻近替代；快捷短语、动态模板与页面操作 | 创建 snippets / 快捷键；核心功能未要求 AI key | 未找到将 xlsx/docx/txt/md 作为 AI 填表资料的证据 | 免费；Pro $2.99/月按年、$3.49/月按月；Business $6.99/用户/月按年 | 已读；ID `idgadaccgipmpannjkmfddolnnhmeklj` |
| RoboForm | 邻近替代；密码管理与身份资料填表 | 建账户 / 主密码与身份资料；无需模型 key | 官网可导入密码 CSV；不是 Office 文档语义填表证据 | Free；Premium 标准价 $2.49/月按年，续费 $29.88/年；促销占位值未读取 | 已读；ID `pnlccmojcmeohlpggmfnbbiapkmbliob` |
| Simplify Copilot | 邻近替代；求职申请填写、简历与追踪 | 注册并建求职资料；无需用户模型 key 的流程 | 简历 / 职业资料明确；上传后缀未知 | 核心 autofill / tracker 宣称免费；有高级功能，金额未知 | 已读；ID `pbanhockgagggenencehbnadejlgchfc` |
| Magical | 邻近替代；商店仍是短语 / autofill，官网主推医疗运营 agents | 扩展可用快捷模板；当前企业官网预约演示；BYOK 未见证据 | Office 文件到网页表单的支持未知 | `/pricing` 跳回首页；当前金额未知 | 已读；ID `iibninhmiggehlcdolcilmhacighjamp` |
| Bardeen | 邻近替代；当前主推 GTM 获客、抓取、丰富与筛选 | 注册、配置 agent / 抓取流程；BYOK 要求未知 | 定价页提到数据导入免费，格式未知；CSV 主要是导出 | 免费 100 credits；Basic $10/月；Premium $50/月或 $480/年 | 后续已核实：ID `ihhkmalpkhkoedlmcnilbbhhbhnicjga`，发行者 Bardeen Inc |

价格是本次公开页面展示的美元价格，未下单核验地区税费、促销、具体套餐选择或后台权益。没有把 credit 数直接换算成可填表次数。

## 三组值得追到原出处的比较

1. **同名但不同主体。** [formfill.cc](https://www.formfill.cc/) 的安装按钮当时指向其[商店条目](https://chromewebstore.google.com/detail/formfill-ai/faepndggpldacododbleilmkbfedkkpo)，后者又要求该站登录；[政策](https://www.formfill.cc/extension/privacy-policy)描述站点后端与服务商。另一站 [formfillai.cc](https://formfillai.cc/) 在[定价](https://formfillai.cc/pricing)与[政策](https://formfillai.cc/privacy)中宣传 BYOK，但当时可见安装按钮没有有效目的地。第三个 `formfill.ai` 只读到同名页标题。重名不是商标检索结果，也不是本品声誉、流量或销量。
2. **同一域名，产品已经变了。** [Filliny 的旧商店条目](https://chromewebstore.google.com/detail/filliny-ai-form-filling-a/ijhjngflokdbhimpcjhgjkljlfjenjlg)仍介绍通用填表；[官网](https://filliny.io/)讲的是简历、岗位与申请追踪。[定价页](https://filliny.io/pricing)当时只有加载占位。新网站不能证明旧扩展具有全部求职能力。类似地，[Magical](https://www.getmagical.com/) 后来的医疗运营定位不能拿旧短语工具用户数当客户数。
3. **同一任务，替代办法不同。** [FillApp](https://fillapp.ai/)把“资料文档→网页字段”演给读者看；[价格](https://fillapp.ai/pricing)含模型额度，[政策](https://fillapp.ai/privacy-policy/)又区分本地操作与云端处理。[Text Blaze](https://blaze.today/)用短语、快捷键和动态模板，固定字段未必需要模型；[RoboForm](https://www.roboform.com/form-filler)强调身份资料，[Simplify](https://simplify.jobs/copilot)做求职，[Bardeen](https://www.bardeen.ai/)讲获客和跨应用流程，都不能当作通用 Office 填表的同一市场。

## 当时最值得借的三点

FillApp 值得看的是把输入、结果和成本放在一起；Text Blaze 值得看的是三步上手、用途页和可复制例子；formfill.cc 值得看的是可辨认的安装入口。模仿这些说明方式，不需要模仿它们的 PDF、求职、支付或自动提交能力。研究者看的是 HTML、链接、条目和政策，没有安装竞品、实付或目验视觉布局；用什么判断市场表现，另见[研究总报告](35-FormFill_全面评估：网站、竞品、市场表现、产品策略与_SEO.md)。
