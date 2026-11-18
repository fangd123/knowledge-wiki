---
title: "开源生图/生视频模型的 LoRA 与 ComfyUI Workflow 分享平台调研报告"
date: 2026-11-08
author: fangd123
license: CC BY-NC-SA 4.0
category: 工程实践
series: 开源生图生视频生态
series_order: 2
tags:
  - 工程实践
  - 开源生图生视频生态
ai_assisted: true
status: published
---

# 开源生图/生视频模型的 LoRA 与 ComfyUI Workflow 分享平台调研报告

- 调研日期：2026-11-08
- 调研对象：用户本地使用 ComfyUI 运行 Qwen-Image（含 Edit）、Z-Image（Turbo 等）、FLUX.2、Wan 2.x 视频、HunyuanVideo、LTX 等开源模型，需要知道去哪里找 LoRA 和 workflow。
- 方法：全部结论基于本次联网搜索（WebSearch / WebFetch）获得的页面内容，来源随条目附 URL。
- 标注约定：
  - **【事实】** = 来源页面直接写明的内容
  - **【推断】** = 我基于事实做的推理，可能出错
  - **【未核实】** = 搜索中没有拿到可靠证据，或第三方来源互相矛盾
- 所有"下载量/点赞数/更新日期"均为来源页面自称的读数，时效性有限，仅供参考。

---

## 0. TL;DR：按需求选平台推荐表

| 你的需求 | 首选 | 备选 | 一句话理由 |
|---|---|---|---|
| 找 Qwen-Image / Qwen-Image-Edit 的 LoRA | Civitai（Qwen 生态页 + base model 筛选） | Hugging Face、LiblibAI、ModelScope | Civitai 有独立 Qwen 生态页和 base model 筛选；HF 是官方权重和训练数据集的主仓库 |
| 找 Z-Image / Turbo 的 LoRA | Civitai（Z-Image 生态页）、Tensor.Art/吐司 | LiblibAI、ModelScope、HF | Civitai 与 Tensor.Art 都已有 Z-Image / Z-Image-Turbo 独立标签；国内平台 Z-Image 生态最活跃 |
| 找 FLUX.2 的 LoRA | Civitai（FLUX.2 生态页） | Hugging Face | Civitai 官方生态页显示 FLUX.2 已有 3K+ 模型、2K+ LoRA |
| 找 Wan 2.x / HunyuanVideo / LTX 视频 LoRA | Civitai（Wan / LTX Video 生态页） | ModelScope（Wan-AI 官方）、HF | Civitai 有视频模型生态页和视频 LoRA 训练器；官方 Wan 权重在 ModelScope/HF |
| 找可直接导入的 workflow JSON（国际站） | ComfyUI 官方 Templates（comfy.org / docs.comfy.org） | ComfyWorkflows.com、OpenArt、Civitai workflow 分区 | 官方模板保证与最新 ComfyUI 版本兼容、一键下载 JSON |
| 找可直接导入的 workflow JSON（国内直连） | LiblibAI（workflow 模版）、RunningHub | eSheep、魔搭 AIGC 专区 | 国内免梯子；RunningHub 主打在线运行+分享 |
| 找官方原生支持的新模型教程/模板 | docs.comfy.org + blog.comfy.org | — | 新模型（Qwen-Image、LTX-2 等）通常首发当天就有官方模板和 JSON |
| 找自定义节点（报红修复） | registry.comfy.org + ComfyUI Manager | GitHub | 官方节点注册表，Manager 一键"安装缺失节点" |
| 看新模型动态、抄参数 | Reddit r/StableDiffusion、r/comfyui | Discord、B站 | 大量帖子直接附 workflow 文件，但国内需梯子 |
| 在线试跑不想本地配环境 | RunComfy、RunningHub、Tensor.Art | Civitai 云端生成 | 云端 ComfyUI/在线生成，按量计费 |

---

## 1. 国际平台

### 1.1 Civitai（civitai.com / civitai.green / civitai.red）

**主要内容类型**
- **【事实】** Checkpoint、LoRA、Embedding、VAE、workflow 文件、在线生成（云端 GPU）、在线 LoRA 训练（含视频 LoRA 训练）。来源：https://civitai.com/ 、https://aitoolservices.com/civitai-review-2026-how-to-use-free-app-api-alternatives-safe
- **【事实】** 官方为每个主流模型家族建立"Generation Ecosystems"生态页，图像类包括 FLUX.1、FLUX.2、SDXL、Pony、Illustrious、NoobAI、Qwen Image、**Z-Image**、HiDream、Chroma、Krea 2、Anima 等；视频类包括 **Wan、LTX Video**、Kling、Seedance 等。来源：https://www.civitai.com/ecosystems
- **【事实】** FLUX.2 生态页自述：3K+ 模型、2K+ FLUX.2 LoRA、23M+ 张图；并列出 FLUX.2 [dev]（开放权重）与 [klein] 9B/4B（本地可跑）两个本地变体。来源：https://civitai.com/ecosystems/flux2

**新模型筛选支持（base model 标签）**
- **【事实】** 模型页按 base model 筛选。已确认存在的标签：`Qwen 2.1`（2026-10-13 读数下有 106 个 LoRA）、`ZImageBase`、`Z-image Turbo`、`Flux.2 D`、`Flux.2 Klein 9B-base`、`Wan Video 2.2 T2V-A14B`、`Wan Video` 等。来源：https://orcarouter.ai/zh-CN/blog/qwen-image-2-1-lora 、https://civitai.com/models/1837629/1980s-style-chinese-women 、https://civitai.red/models/2181569?modelVersionId=2467501
- **【事实】** 一条重要的搜索经验：Civitai 纯文本搜索"qwen image 2.1"会返回 994 个唯一模型，其中大部分并不是该 checkpoint 的 LoRA；**要用 base model 筛选器，不要用搜索框**。来源：https://orcarouter.ai/zh-CN/blog/qwen-image-2-1-lora
- **【事实】** 站内 LoRA 训练器支持的视频底模：Hunyuan 720p（Musubi）、Wan 2.1 T2V/I2V 14B（Musubi）、LTX2（AI-Toolkit）。注意训练器列表里视频侧只写到 Wan 2.1，未列 Wan 2.2。来源：https://education.civitai.com/using-civitai-the-on-site-lora-trainer/
- **【未核实】** HunyuanVideo 在生态页列表中未见独立条目（ecosystems 页只列了 Wan/LTX/Kling/Seedance 等），但其 LoRA 存在于站内（训练器支持）。生态页是否新增 Hunyuan 条目未核实。

**workflow 是否可直接下载导入**
- **【事实】** 有 workflow 分区（模型筛选类型选 Workflows / tag "comfyui"），可下载；来源：https://www.sucaijishi.com/articles-51-571-1.html 、https://articles.waytoagi.com/docs/IgJwwPLhsieJHFkKUaOcnz0hnme
- **【事实】** 官方有约 160 个"Civitai Nodes for ComfyUI"自定义节点，支持站内模型浏览。来源：https://civitai.com/?ref=shovelstack
- **【未核实】** workflow 详情页是否统一标注所需自定义节点/模型清单未逐条核实；社区 workflow 的依赖标注质量参差是常见吐槽（推断）。

**国内访问、登录、下载限制、商用标注**
- **【事实】** 浏览无需账号；下载模型免费；注册免费（Discord/GitHub/Google/Reddit/邮箱）。来源：https://aitoolservices.com/civitai-review-2026-how-to-use-free-app-api-alternatives-safe
- **【事实】** 2026 年域名拆分：civitai.com（PG/PG-13）+ civitai.red（无限制内容），单点登录互通；信用卡支付已于 2025 年 5 月被 Visa/Mastercard 从主站撤下，主站走加密货币，信用卡用户用 civitai.green（全年龄分站）。来源：同上、https://www.aixq.cc/31147.html
- **【事实】** 会员分档 Bronze/Silver/Gold $10/$25/$50 每月（2026-10-03 读数），送 10K/25K/50K Green Buzz；新账号送 100 Blue Buzz（免费额度）。来源：https://hokai.io/hub/tools/civitai
- **【事实】** 商用：每个模型页单独标注 License（如 Apache 2.0、FLUX Non-Commercial、Qwen Research License 等），下载前需逐个确认。来源：本文引用的各 Civitai 模型页 License 字段
- **【事实】** 国内访问：多篇中文资料一致称需科学上网。来源：https://www.sucaijishi.com/articles-51-571-1.html 、CSDN 教程（x-humanoid.csdn.net"正常情况下是没法访问的"）

**优缺点与适合人群**
- 优点：新模型 LoRA 生态最强、base model 筛选精细、免费下载、有在线生成兜底、社区评分/示例图自带参数。
- 缺点：国内需梯子；支付渠道受限；NSFW 内容比例高；文本搜索噪音大；2026 年拆分双域名后偶发登录/访问问题（来源：https://aiimagegeneratornsfw.com/civitai-not-working-fix-2026 ，为第三方排障文，可靠性一般）。
- 适合：愿意折腾梯子、想要"某具体风格/角色 LoRA"的本地 ComfyUI 用户——正是你这种场景。

### 1.2 Hugging Face

**主要内容类型**
- **【事实】** 官方原版权重（HF 是 Qwen/Z-Image/Wan/FLUX 官方发布渠道之一）、社区 LoRA、量化/合并/重打包、数据集、Spaces 在线 demo。来源：https://huggingface.co/flymy-ai/qwen-image-realism-lora 等模型页
- **【事实】** Comfy-Org 在 HF 上维护"ComfyUI repackaged"权重仓（如 Comfy-Org/Qwen-Image_ComfyUI、Qwen-Image-2.1 配套文件），是 ComfyUI 用户最常下载的目录结构来源。来源：https://docs.comfy.org/tutorials/image/qwen/qwen-image 、https://docs.comfy.org/zh/tutorials/image/qwen/qwen-image-2-1

**新模型筛选支持**
- **【事实】** 模型卡通过 `base_model:` 元数据字段关联底模，模型树页面显示"Adapter (N)"数量；也可用 `lora` 标签过滤。2026-10-13 读数：声明 base 为 Qwen-Image-2.1 的仓库 323 个，其中带 lora 标签的 115 个（标签由上传者手工打，未打标的 LoRA 不计入）。来源：https://orcarouter.ai/zh-CN/blog/qwen-image-2-1-lora 、https://www.huggingface.co/d14945921/qwen_hibrid-lora
- **【推断】** HF 对 Z-Image/Wan/FLUX.2 的覆盖方式相同（base_model 字段 + 标签），但缺少 Civitai 那种"按底模一键筛选+按下载量排序"的购物式体验。

**workflow 是否可直接下载导入**
- **【事实】** HF 本身不是 workflow 分享站，但不少 LoRA 仓库附带 ComfyUI workflow JSON（如 flymy-ai/qwen-image-realism-lora 提供 qwen_image_lora_example.json 与完整配置说明）；加速类 LoRA 仓库常附带配套工作流说明。来源：https://huggingface.co/flymy-ai/qwen-image-realism-lora 、https://huggingface.co/t8star/Qwen-Image-2.1-viggle-turbo-4step-r64-comfy
- **【未核实】** 没有统一的"workflow 仓库分类"，能否找到全看作者是否附上。

**国内访问、登录、下载限制、商用标注**
- **【事实】** 公开仓库免登录直接下载；License 在每个模型卡右侧明确标注（Apache-2.0 / FLUX Non-Commercial / Qwen Research 等）。国内访问情况本次未专门核实——历史上 hf.co 在国内时通时断，国内用户常用镜像（hf-mirror.com 等）【推断/未核实，本次搜索未覆盖镜像现状】。
- **【事实】** 阿里官方也把 Qwen/Z-Image 权重同步发布到魔搭，国内直连下载更快（见 2.2）。

**优缺点与适合人群**
- 优点：官方第一发布渠道、版本可追溯、License 明确、数据集与训练脚本齐全；适合研究式下载和官方底模获取。
- 缺点：社区 LoRA 缺乏社区筛选/评分体系（点赞数弱于 Civitai），中文界面无、国内直连不稳。
- 适合：下官方底模、量化版本、训练数据集；追新模型第一时间拿权重。

### 1.3 OpenArt（openart.ai/workflows）

**主要内容类型**
- **【事实】** ComfyUI workflow 分享（老牌、流量大）+ 在线生成平台（已转型为一站式生图/生视频工作室，聚合 Nano Banana、GPT Image、Seedream、Flux、Qwen、Z-Image 等模型按积分计费）。来源：https://openart.ai/home 、https://www.ai-media-studio.com/openart-review
- **【事实】** workflow 免费下载、无需注册，下载后得到 .json，直接拖入 ComfyUI 使用。来源：https://blog.csdn.net/weixin_58401027/article/details/157776141
- **【事实】** 2026-11-03 实测文：免费试用 40 积分（一次性）且全部带水印；Starter $14/月无商用权，Plus $34/月起有商用权；模型列表含 Z-Image（8 积分/张）、Qwen Image 3.0、Flux 2 Pro、Wan 3.0 视频等。来源：https://www.ai-media-studio.com/openart-review

**新模型筛选支持**
- **【未核实】** workflow 分区是否已建立 Qwen-Image / Z-Image / FLUX.2 独立筛选标签，本次搜索未拿到证据。在线生成侧模型列表确认覆盖 Z-Image/Qwen/Flux 2（来源同上）。【推断】workflow 分区大概率跟随热门模型更新，但标签体系需自行验证。

**workflow 下载标注**
- **【事实】** workflow 使用了自定义节点时，导入后节点报红属常见情况，需按报错装对应节点包（社区教程反复提到 Impact Pack、ControlNet Aux 等）。来源：https://blog.csdn.net/weixin_29057065/article/details/165968925
- **【事实】** ComfyUI Manager 的"分享"功能原生支持发布到 openart.ai。来源：https://github.com/Comfy-Org/ComfyUI-Manager

**访问/登录/限制/商用**
- **【矛盾】** 国内访问两说：一派（aihr.wang）称"中国用户无需翻墙即可直接访问"，另一派（sucaijishi 素材集市）称"需科学上网"。来源：https://www.aihr.wang/sites/660.html 、https://www.sucaijishi.com/articles-51-571-1.html 。**结论：以实际尝试为准，本次无法裁决。**
- **【事实】** 在线生成按积分付费、免费额度一次性；商用权从 Plus 档起。来源：https://www.ai-media-studio.com/openart-review

**优缺点与适合人群**
- 优点：workflow 数量与流量大、教程质量高（含视频教程）、下载免费不强制注册。
- 缺点：在线生成商业化重心重、部分 workflow 依赖特定节点版本；国内访问情况存疑。
- 适合：想要大量现成 workflow 和配套教程的进阶 ComfyUI 用户。

### 1.4 ComfyWorkflows.com

**主要内容类型**
- **【事实】** 纯 ComfyUI workflow 分享社区（自述"ComfyUI 的 Civitai"），支持在线运行（云端算力）。来源：http://ui.marklion.cn/article/7e7/13/article-b146d11fdf84b557.shtml
- **【事实】** 点 Download 可直接下载 workflow JSON 或 PNG（PNG 内嵌 workflow 元数据，可拖入 ComfyUI 还原）。来源：同上、https://blog.csdn.net/weixin_58401027/article/details/157776141
- **【事实】** 导入后可用"Install Missing Custom Nodes"自动补齐缺失节点。来源：http://ui.marklion.cn/article/7e7/13/article-b146d11fdf84b557.shtml

**新模型筛选支持**
- **【未核实】** 是否有按 base model（Qwen/Z-Image/FLUX.2/Wan）的筛选标签，本次搜索未见证据。【推断】其检索以关键词为主。

**访问/登录/限制/商用**
- **【事实】** 中文资料称"不需要魔法"即可访问；下载 workflow 无需注册登录。来源：https://blog.csdn.net/weixin_58401027/article/details/157776141
- **【事实】** ComfyUI Manager 的 Share 按钮原生支持发布到 comfyworkflows.com（与 openart.ai、youml.com 并列）。来源：https://github.com/Comfy-Org/ComfyUI-Manager
- **【未核实】** 在线运行的计费方式、当前 workflow 总量、商用许可政策未核实。**【注意】** 存在第三方"从该站抓取 workflow JSON 需要绕过 JS 渲染"的脚本仓库，说明部分页面下载路径对脚本/爬虫不友好（对普通浏览器用户影响未知）。来源：https://github.com/abk234/ComfyUI/blob/master/DOWNLOAD_SOLUTIONS.md

**优缺点与适合人群**
- 优点：免费直下 JSON、免梯子（据中文教程）、自动装缺失节点、可云端试跑。
- 缺点：界面朴素、热度略低于 OpenArt（中文教程口径）；新模型标签不明确。
- 适合：想快速白嫖 workflow 且不想装梯子的国内用户；配合 Manager 的分享生态使用顺滑。

### 1.5 Tensor.Art（国际版；与国内"吐司 Tusi"同一家公司）

**主要内容类型**
- **【事实】** 免费在线生图站 + 模型托管（checkpoint/LoRA/LyCORIS）+ ComfyUI 式节点工作流（ComfyFlow）+ 在线 LoRA 训练 + AI 小工具（封装好的单按钮 workflow）。来源：https://tensor.art/ 、https://tensorart.me/zh/models
- **【事实】** 每日免费积分（约 100/天，Pro $9.9/月 500/天）。来源：https://lemonsight.com/tool/image/tensor-art

**新模型筛选支持（本次调研中覆盖最全的站之一）**
- **【事实】** 首页/模型库直接可见这些 base model 标签：`Z-Image`、`Z-Image-Turbo`、`Qwen-Image`、`Qwen-Image-2.1`、`FLUX_2_KLEIN_9B_DISTILLED`、`FLUX_2_KLEIN_9B_BASE`、`WAN_2_6`、`WAN_2_2_A14B`、`HunyuanVideo_1_5_720P`、`LTX-2`、`KREA_2`、`Anima`、`Illustrious`、`NoobAI` 等。来源：https://tensor.art/ 、https://tensorart.me/zh/models
- **【事实】** 官方上线了 Z-Image 在线 LoRA 训练（首次训练免费活动 + 50% off），训练工作台支持选择 z-image 底模、自动打标（含 Gemini-3-Flash 标注）。来源：https://tensor.art/ 首页文章
- **【事实】** 视频侧提供 Wan 2.6 Preview-T2V（通义万相官方账号上传）、HunyuanVideo 1.5（腾讯混元官方上传）等。来源：https://tensorart.me/

**workflow 下载/导出**
- **【事实】** 平台工作流为站内 ComfyFlow 格式，可发布为"AI 小工具"一键复用。来源：https://tensor.art/about/aitool-tutorial
- **【矛盾/未核实】** 第三方评测称其节点图可"导出 JSON、分享到社区"（https://hypeart.ai/commercial-use/tensor-art-ai ），但官方教程未明说导出后的 JSON 能否直接在本地 ComfyUI 打开。**【推断】** Tensor.Art 工作流与本地 ComfyUI JSON 并非 1:1 兼容，站内模板主要面向站内运行；本地化使用需谨慎。官方还提供了 ComfyUI_TENSOR_ART 自定义节点包，把站内 AI 小工具以 API 方式接入本地 ComfyUI（https://github.com/Tensor-Art/ComfyUI_TENSOR_ART ，已发布到 Comfy Registry）。
- **【事实】** 模型文件（safetensors）可下载到本地 ComfyUI 使用（多个模型页与第三方指南确认）。来源：https://lilting.ch/en/articles/seaart-lora-guide （SeaArt 同类逻辑）、Tensor.Art 模型页

**访问/登录/限制/商用**
- **【事实】** 注册即可用（Google/Discord），每日免费积分；训练另计积分；未登录无法下载（推断，行业惯例）【未核实具体下载门槛】。
- **【事实】** 商用：整体允许商用但受各模型 License 约束（lemonsight 评测："commercial use depends on each base model's license"）。来源：https://lemonsight.com/tool/image/tensor-art
- **【事实】** 国内用户直接访问 tensor.art 一般可行（新加坡主体，中文界面完整）【推断——中文站 tensorart.me 与国内吐司同源】。

**优缺点与适合人群**
- 优点：新模型（Z-Image/Qwen-2.1/FLUX.2/Wan2.6/Hunyuan 1.5/LTX-2）标签覆盖本次调研最全；在线训练 Z-Image LoRA 便宜甚至免费；中文友好。
- 缺点：workflow 生态偏站内闭环，本地 ComfyUI 兼容性存疑；免费积分有限；NSFW 内容多。
- 适合：想在线试新模型/低成本训练 Z-Image LoRA 的人；把 Tensor.Art 当"模型发现+试跑站"，模型文件下载回本地用。

### 1.6 SeaArt（seaart.ai / seaart.me）

**主要内容类型**
- **【事实】** 在线生图/生视频平台 + 模型库（checkpoint/LoRA）+ 站内 ComfyUI（可视化节点工作流）+ LoRA 在线训练。来源：https://docs.seaart.ai/guide-1/2-seaart-ai-basic-function/2-10-workflow/text-to-image-workflow 、https://lilting.ch/en/articles/seaart-lora-guide
- **【事实】** 模型页有明确"License Scope"区块，分项标注：是否允许在线生成 / 是否允许下载 / 商用范围（例如 Wan 2.1 LoRA 页明确显示 Model Source: civitai、允许下载开关）。来源：https://www.seaart.me/models/detail/1f7fa1890d70f578d87ccaec7c0a504d

**新模型筛选支持**
- **【事实】** 模型页带 base model 标签（已确认 `Wan Video`、`Flux.1 D`）；首页推荐流含 Wan 2.x 系列。来源：同上
- **【未核实】** Qwen-Image / Z-Image / FLUX.2 独立标签未见证据。【推断】其模型库更新速度慢于 Civitai/Tensor.Art，主推自 家训练生态。

**workflow 下载**
- **【事实】** SeaArt 的 ComfyUI 是站内云端工作台；官方文档教用户在站内搭工作流。作者可在模型简介附外部下载链接（有模型作者贴 Dropbox 的 workflow JSON 直链）。来源：docs.seaart.ai、https://www.seaart.me/models/detail/bc50f1de13f1691f2e4fde5d95b908f3
- **【推断】** 站内工作流不一定能导出为本地可用的 ComfyUI JSON；平台定位是"在线跑"，不是"workflow 分享站"。

**访问/登录/限制/商用**
- **【事实】** 免费可用（含每日额度），付费解锁更多；免费档训练的 LoRA 必须公开、付费档可私有。来源：https://lilting.ch/en/articles/seaart-lora-guide
- **【事实】** 商用按会员档与模型许可双重限制。来源：SeaArt 模型页 License Scope + lilting 指南
- **【推断】** 国内可直接访问（中文界面完整，主体为国内/新加坡团队）。

**优缺点与适合人群**
- 优点：界面中文友好、免费额度可观、License Scope 标注清晰（哪些可下载/可商用一目了然）。
- 缺点：重心在在线生成，LoRA 下载与 workflow 分享非主业；模型库深度不如 Civitai。
- 适合：把 SeaArt 当"在线出图+偶尔薅模型"的站点，不适合作为 workflow 获取主渠道。

### 1.7 Shakker（shakker.ai）

**主要内容类型**
- **【事实】** 模型库（自称 50,000+ Stable Diffusion 模型，含"Member Exclusive 会员专属模型"）+ Shakker Generator + 在线 A1111 WebUI + 在线 ComfyUI + LoRA 训练 + 无限画布。来源：http://shakker.ai/ 、https://aigearbase.com/tool/shakker-ai
- **【事实】** 会员付费后会员专属模型出图带商用权；Basic $10/月 15K tokens、Advanced $24/月 35K tokens。来源：https://forwardfuture.com/tools/details/shakker.ai

**新模型筛选支持**
- **【事实】** 首页模型分区以 FLUX.1 和 SDXL 生态为主（大量 FLUX.1 Exclusive、XL 模型）。来源：http://shakker.ai/
- **【未核实/推断】** 未见到 Qwen-Image / Z-Image / FLUX.2 / Wan 的专区或标签；**推断其新模型覆盖明显滞后**（本次搜索无任何 Shakcker + 新模型的匹配结果）。

**workflow 下载**
- **【事实】** 提供预置 ComfyUI 工作流模板（角色生成、换装、产品渲染、照片修复、放大等，导出至 4K）。来源：https://aidirectory.com/shakker
- **【未核实】** 模板能否导出 JSON 到本地 ComfyUI 未核实。【推断】同样偏站内闭环。

**访问/登录/限制/商用**
- **【事实】** 免费档每日 200 tokens；付费会员商用权；支持 Stripe，国内支付渠道未核实。来源：forwardfuture、aidirectory

**优缺点与适合人群**
- 优点：设计向内容组织（游戏/空间/插画/电商）有特色；A1111+ComfyUI 双在线环境。
- 缺点：新模型跟进慢；token 月度清零；私隐生成"即将上线"状态（来源称）。
- 适合：设计/电商向用户；对 Qwen/Z-Image/FLUX.2 用户**不推荐作为主渠道**。

### 1.8 ComfyUI 官方资源（comfy.org / docs.comfy.org / blog.comfy.org / registry.comfy.org / ComfyUI-Examples）

**主要内容类型**
- **【事实】** 官方 Workflow Templates（内置在 ComfyUI 的 Workflow → Browse Templates）+ docs.comfy.org 每模型教程页（每篇都提供"Download JSON"）+ comfy.org/workflows 在线模板库（可浏览器直接跑）+ 官方博客发布新模型原生支持公告 + ComfyUI-Examples GitHub 仓库。来源：
  - https://docs.comfy.org/tutorials/image/qwen/qwen-image （Qwen-Image：文生图/InstantX ControlNet/DiffSynth ControlNet，均有 Download JSON）
  - https://docs.comfy.org/zh/tutorials/image/qwen/qwen-image-2-1 （Qwen-Image-2.1：文生图/编辑/Turbo/2K，均有 Download JSON）
  - https://blog.comfy.org/p/qwen-image-in-comfyui-new-era-of
  - https://comfy.org/zh/workflows/
  - https://comfyanonymous.github.io/ComfyUI_examples/ （https://github.com/comfyanonymous/ComfyUI_examples）

**新模型筛选支持**
- **【事实】** comfy.org/workflows 模板库已含按模型组织的条目：Qwen-Image-Edit（多个）、Z-Image-Turbo（Starter/Product Photography/Image Upscale 2K）、FLUX.2 Dev/Klein、FLUX.1 Fill/Redux、Wan2.1/Wan2.2 Animate/Wan Animate 2、LTX-2.3、Seedance、Nano Banana 等。来源：https://comfy.org/zh/workflows/
- **【事实】** 首页同时主推 LTX 2.5、Wan Animate 2、Seedance 2.5、MiniMax H3 等最新模型入口。来源：https://www.comfy.org/

**workflow 是否可直接下载导入**
- **【事实】** 可以。官方教程页均提供"Download Workflow / Download JSON"双按钮，且承诺"模板随 ComfyUI 稳定版内置"。加载缺节点时按提示更新 ComfyUI 或装 Manager 补节点。来源：docs.comfy.org 各教程
- **【事实】** 官方教程还附显存实测（如 Qwen-Image fp8 在 RTX 4090D 24GB 上首生成 94s）。来源：https://docs.comfy.org/tutorials/image/qwen/qwen-image

**Comfy Registry 与节点生态**
- **【事实】** registry.comfy.org 是官方自定义节点注册表；ComfyUI-Manager V3.3.2 起官方对接该注册表，"Install Missing Custom Nodes"即基于本地映射表 + Registry API。2026-08-18 版本 Manager 本地映射含 5,590 个节点包、42,497 个节点名。来源：https://registry.comfy.org （经 https://localaimaster.com/blog/comfyui-missing-node-types 转述）、https://github.com/Comfy-Org/ComfyUI-Manager 、https://docs.comfy.org/manager/legacy-ui

**访问/登录/限制/商用**
- **【事实】** 全部免费，无需登录（模板下载/文档）；云端运行（Comfy Cloud）另计费。来源：docs.comfy.org
- **【推断】** 国内可直连 docs.comfy.org / blog（未专门核实，一般可访问）。

**优缺点与适合人群**
- 优点：100% 兼容最新 ComfyUI、质量最高、免费、新模型发布当天即有官方模板；License 场景干净。
- 缺点：只有官方/精选模板，数量远小于 OpenArt/ComfyWorkflows；社区创意类 workflow 少。
- 适合：所有人——**任何新模型先看官方模板，再去找社区创作**。

### 1.9 RunComfy（runcomfy.com）

**主要内容类型**
- **【事实】** 云端 ComfyUI 平台：200+ 精选模板（按模型分浏览：FLUX 47、WAN 51、LTX 36、Z-Image 10、Qwen Image Edit 12、Qwen Image 2.1 10、MiniMax H3 25-35、MOSS-Image 7、Krea 2 11-12 等，2026-11 读数）+ "拖入你的 workflow.json，平台自动配好全部依赖/节点/模型"（Auto-Setup Agent）+ GPU 16-141GB 按需。自述 1M+ 创作者。来源：https://www.runcomfy.com/ 、https://runcomfy.com/?ref=withai.top
- **【事实】** 每个模型工作流页可"运行 / 下载 JSON / 部署"，且工作流页=JSON+OS+Python 环境+ComfyUI+自定义节点+模型的完整快照（可复现、可分享链接、可一键转 Serverless API）。来源：https://www.runcomfy.com/

**新模型筛选支持**
- **【事实】** 按 base model 分区浏览（如上），Z-Image、Qwen Image 2.1、Wan、LTX 均有独立分区。来源：同上

**访问/登录/限制/商用**
- **【事实】** 免费试用（"Get started for Free"）；下载模型支持从 Civitai/HF/Google Drive 拉取并持久化。来源：https://www.runcomfy.com/
- **【未核实】** 具体价格、国内直连情况、模板商用条款未核实。【推断】国际站，国内访问可能不稳。

**优缺点与适合人群**
- 优点：解决"workflow 报红/缺模型"的最大痛点（自动配环境）；模板可直接下 JSON 回本地。
- 缺点：云 GPU 计费；对只想白嫖 JSON 的人是浪费；国内访问未证实。
- 适合：不想配环境、或想复现复杂视频工作流的人。

### 1.10 GitHub 精选列表与其它社区资源

- **awesome-comfyui（lucianosb）**：ComfyUI 工具精选列表（215 stars，2026-09 仍在更新），收录 OpenArt/ComfyWorkflows/YouML/Wyrde 等 workflow 站点与常用自定义节点。来源：https://github.com/lucianosb/awesome-comfyui
- **ComfyUI-Examples**：官方示例仓库，可整体 clone。来源：https://github.com/comfyanonymous/ComfyUI_examples
- **其它被反复提及的站点**：YouML（workflow 转交互式应用）、ComfyICU（comfy.icu，在线运行）、AIGODLIKE（国内，"ComfyUI 奥术"分区）、Flowt.ai。来源：https://articles.waytoagi.com/docs/IgJwwPLhsieJHFkKUaOcnz0hnme 、awesome-comfyui
- **ComfyUI Workflow Inspector**：浏览器内分析 workflow JSON 的诊断工具（缺输出节点/预览分支/耗时等）。来源：awesome-comfyui README
- **【未核实】** YouML/ComfyICU/Flowt 当前运营状态未逐一核实。

### 1.11 Reddit

- **【事实】** r/StableDiffusion（600K+ 成员）是开源生图生态的中心：新模型发布、LoRA 推荐、workflow、排障；r/comfyui（55K+）专注节点工作流分享与排障；r/civitai（14K+）围绕模型分享；r/FluxAI（20K+）。来源：https://www.deadsubs.com/best/ai-art-subreddits
- **【事实】** 大量帖子直接附 workflow 文件下载链接；可用 `[Workflow]` 标签或站内搜索找。来源：https://blog.csdn.net/weixin_33531560/article/details/157245276
- **【推断】** 国内访问需梯子；Reddit 是"新模型发布后第一波真实评测与 workflow 的最快渠道"，通常快于各平台专题页上线。
- 适合：追新、看真实口碑、抄参数；不适合当"下载站"用。

---

## 2. 国内平台

### 2.1 LiblibAI 哩布哩布（liblib.art）

**主要内容类型**
- **【事实】** 国内头部 AI 创作平台：模型库（checkpoint/LoRA）+ 在线生成 + ComfyUI/WebUI 在线工作流 + workflow 模版免费下载 + LoRA 训练 + 桌面客户端（pc.liblib.art：一键安装、内置 WebUI+ComfyUI 启动器、10w+ 模型即下即用）。来源：https://pc.liblib.art/ 、https://www.liblib.art/
- **【事实】** 有独立"workflow 模版"入口可免费下载。来源：https://www.sucaijishi.com/articles-51-571-1.html

**新模型筛选支持**
- **【事实】** 模型筛选器支持按"图片模型（base model）"筛选 LoRA，教程确认含 z-image 等选项。来源：https://x-humanoid.csdn.net/6a9edd3e1a35784c841dcac6.html
- **【事实】** Z-Image 生态特别活跃：官方级 Z-Image-Base/Turbo 标准工作流、2K 直出工作流、全套 fp8/BF16 量化文件与文本编码器/VAE 分发；多个 Z-Image/Qwen-Image 商用向 LoRA（如电商海报 LoRA 同时出 Qwen/Z-Image/FLUX 三版本）。来源：https://www.liblib.art/modelinfo/758ced0f0def496b825fa6c797d9870e 、https://www.liblib.art/modelinfo/7d72b6aa66844d1e99a51758b7fdbb99 、https://liblib.art/modelinfo/efe968c6597447919cf9a689ba87c559

**workflow 下载标注**
- **【事实】** workflow/图片模板页标注"商业许可范围"，且官方明确：workflow 商用需同时看 workflow 本身与其内所有模型的许可。来源：https://www.liblib.art/activities/e0aa50f25b874722ac60be71adb8c9cb/Commercial_Guidelines
- **【未核实】** workflow 详情页是否标注所需自定义节点清单未逐条核实。

**访问/登录/限制/商用**
- **【事实】** 国内直连、注册登录后使用。
- **【事实】** 官方商用规范页详细定义了会员模型/非会员模型/工作流的出图商用边界（如：会员过期后新出图不可商用）。来源：同上
- **【矛盾】** 下载限制说法不一：第三方资料有"模型下载无限制"（biemoyu，2026-06）与"VIP 模型下载上限每月 200 次"（aixin315，较旧）两个版本；另有 CSDN 教程作者吐槽"很多模型没法下载，冲会员都没用，平台更希望你在站内用"。来源：https://biemoyu.com/sites/liblibai-guanwang.html 、http://www.aixin315.com/sites/6474.html 、https://x-humanoid.csdn.net/6a9edd3e1a35784c841dcac6.html 。**结论：下载权限与会员体系复杂且随时间变化，实际以模型详情页为准；部分作者把作品设为"仅在线使用"。**
- **【事实】** 会员价格资料（第三方 2026-06 读数）：基础 ¥29/月、专业 ¥59/月、旗舰 ¥99/月，积分制。【未核实】以官网实时页面为准。官网首页可见双十一活动、按积分计费的视频模型时长表（Seedance 2.5/Kling 4.0/Wan 3 等）。来源：https://www.liblib.art/?referralCode=RbfgUps4&paywall=personal

**优缺点与适合人群**
- 优点：国内直连、中文标签齐全、Z-Image/Qwen 本土生态最厚、workflow 可免费下、商用规范成文。
- 缺点：下载权限受限模型多（社区抱怨）、商业化重、质量参差需自行甄别。
- 适合：国内直连需求 + 想要中文商用向 LoRA/电商场景的用户；Z-Image 用户尤其值得逛。

### 2.2 ModelScope 魔搭（modelscope.cn）

**主要内容类型**
- **【事实】** 阿里系开源模型社区：官方权重首发（Qwen/Wan/Z-Image 全家桶）+ AIGC 专区（在线生图、图生图、局部重绘、ControlNet）+ 在线 ComfyUI 工作流调用 + 免费 GPU Notebook（可自己装 ComfyUI）+ 免费 LoRA 微调（最低 10 张图，内置 Qwen-VL 自动打标）+ 数据集。来源：https://modelscope.cn/posts/1419 、http://www.linkingapi.com/archives/16236 、https://modelers.csdn.net/69a7d2817bbde9200b9da4ff.html
- **【事实】** AIGC 专区上线时首批 157 款多模态模型，所有功能板块及 GPU 算力免费开放；社区已有大量社区 LoRA（含"其他→lora 标签"下针对 Z-Image-Turbo 的 LoRA 分类）。来源：http://www.linkingapi.com/archives/16236 、https://x-humanoid.csdn.net/6a9edd3e1a35784c841dcac6.html

**新模型筛选支持**
- **【事实】** 按模型仓组织（非 base model 标签式筛选），但有分类导航（文生图/图生视频等）；搜索"Z-image-turbo"后左侧选"其他→lora"即可筛 LoRA。来源：https://x-humanoid.csdn.net/6a9edd3e1a35784c841dcac6.html
- **【事实】** 首页趋势榜可见 Wan-Dancer（Wan-AI 官方）、Z-Image-i2L-V2（DiffSynth-Studio）、Qwen-Image-Bench（454.3K 下载）、MiniMax-H3 等。来源：https://modelscope.cn/

**workflow 下载**
- **【事实】** 官方教程（如 FLUX.1 Kontext）以"下载工作流图片→拖入 ComfyUI"方式分发；AIGC 专区支持在线调用 ComfyUI 工作流。社区仓库（如 FixBadImage-KontextLora）的 README 直接给 workflow 目录下载。来源：https://www.modelscope.cn/posts/1419 、https://modelscope.cn/models/huAAUU/FixBadImage-KontextLora/summary
- **【推断】** 魔搭不是 workflow 分享站，workflow 散落在模型仓/帖子中，检索体验一般。

**访问/登录/限制/商用**
- **【事实】** 国内直连、全部免费（含算力）、下载快（国内 CDN）。License 随各模型卡标注（Qwen-Image Apache 2.0；注意 Qwen-Image-2.1 为 Qwen Research License，禁商用，见 3. 许可提示）。来源：https://modelscope.cn/ 、https://blog.csdn.net/liferecords/article/details/166327653

**优缺点与适合人群**
- 优点：国内下载官方模型最快的渠道（免梯子、免费算力）、免费 LoRA 训练、官方模型首发同步。
- 缺点：社区 LoRA 氛围弱于 Civitai/LiblibAI，无以底模为核心的筛选/榜单；workflow 资源散。
- 适合：下官方底模/量化、白嫖算力训练、追阿里系新模型（Qwen/Z-Image/Wan）。

### 2.3 RunningHub（runninghub.cn）

**主要内容类型**
- **【事实】** 云端 ComfyUI 创作平台：工作流社区（分享/学习/编辑/在线运行）+ AI 应用发布变现 + API 服务 + 7000+ 预置节点（第三方口径）+ 预装模型库（Stable Diffusion/Flux/CogVideo/混元等）。来源：http://www.runninghub.cn/ 、http://aizxs.com/tool/runninghub
- **【事实】** 工作流默认私有，除非作者点"发布"；发布后可被他人复制运行。来源：https://www.runninghub.cn/runninghub-api-doc-en/doc-8287465

**新模型筛选支持**
- **【未核实】** 未拿到其工作流市场按 Qwen/Z-Image/FLUX.2 分类的证据。【推断】预装节点"每日更新"，新模型跟进速度快是社区口碑点，但标签体系未证实。

**workflow 下载**
- **【事实】** 平台核心是"在线运行"而非"下载 JSON"【推断——官方材料均强调在线编辑与运行，未见明确 JSON 导出入口】；社区分享的工作流可在站内复制修改。来源：http://www.runninghub.cn/
- **【未核实】** 能否把站内工作流导出为本地 ComfyUI JSON 未核实。**这对你（本地 ComfyUI 用户）是关键差异点：RunningHub 更像"云端算力+应用发布"，不是 workflow JSON 仓库。**

**访问/登录/限制/商用**
- **【事实】** 国内直连（中文主站）；按实际运行时长计费（"只为运行付费"）；会员档第三方资料：免费版每日少量算力 / ¥49 月 100 小时 / ¥199 月 500 小时 / ¥599 月 2000 小时（aizxs 第三方整理，**未核实**）。来源：http://aizxs.com/tool/runninghub 、https://softtool.ai/tool/runninghub/160
- **【事实】** API 分消费级（基础会员以上）与企业级（共享/独占 GPU）。来源：https://www.runninghub.cn/runninghub-api-doc-en/doc-8287465

**优缺点与适合人群**
- 优点：免梯子、节点全、按秒计费灵活、工作流可变现（发布 AI 应用赚收益）、企业 API 成熟。
- 缺点：模型/工作流"下载到本地"非主路径；算力长期成本可能高于本地；会员价资料互相出入。
- 适合：没有本地 GPU 或跑不动大模型视频工作流的人；把云端跑通后再迁回本地。

### 2.4 吐司 Tusi（tusi.cn / tusiart.com）

**主要内容类型**
- **【事实】** 国内 AI 绘画模型分享社区：SD 系 checkpoint/LoRA 分享与下载 + 免费在线生图 + ComfyUI 工作台/工作流 + "AI 小工具"分区（大量现成 ComfyUI 工作流的封装）。来源：https://www.516cn.cn/sites/1045.html 、https://docs.feishu.cn/v/wiki/JoDRwnWz6iOi3SkYMxwcDycInkh/ai/ai
- **【事实】** 与国际版 Tensor.Art 同属一家公司（上海必有回响智能科技有限公司，国际版即 Tensor.Art），模型生态互通。来源：https://www.tahou.com/article/211846057108775941

**新模型筛选支持**
- **【推断】** 依托 Tensor.Art 的模型库（见 1.5），Z-Image/Qwen-Image 等新模型标签大概率同步覆盖国内站，但本次搜索未直接取证吐司站内页面。**【未核实】**

**workflow 下载**
- **【事实】** "AI 小工具"分区提供大量现成 ComfyUI 式工作流（一键使用）；模型上传者可设置"使用权限与商用情况"。来源：https://docs.feishu.cn/v/wiki/JoDRwnWz6iOi3SkYMxwcDycInkh/ai/ai 、http://mp.weixin.qq.com/s?__biz=MzkyMTUwMzAxNg==&mid=2247483671&idx=1&sn=fd1857ef6332ed24ed64f0fc53e4bd50
- **【未核实】** 导出为本地 ComfyUI JSON 的能力同 Tensor.Art（1.5）口径。

**访问/登录/限制/商用**
- **【事实】** 国内直连、免费在线生图（每日免费算力）、模型可免费下载、支持从 Civitai 一键导入模型。来源：https://www.tahou.com/article/211846057108775941 、https://www.ebingou.cn/gongju/19176.html

**优缺点与适合人群**
- 优点：免费门槛最低的国内站之一、中文提示词友好、模型库与 Tensor.Art 打通（新模型覆盖好）。
- 缺点：工作流偏站内闭环；社区深度讨论少于 Civitai/Reddit。
- 适合：国内白嫖试模型、快速试 Z-Image/Qwen LoRA 效果。

### 2.5 eSheep 电子羊（esheep.com）

**主要内容类型**
- **【事实】** 国内 AIGC 社区：模型/画作/工作流-应用三分区，工作流可浏览、下载或在线运行；曾有 WebUI+ComfyUI 双在线环境（腾讯云开发者社区文章口径）。来源：https://articles.waytoagi.com/docs/IgJwwPLhsieJHFkKUaOcnz0hnme 、https://cloud.tencent.com/developer/news/1292442
- **【事实】** 早期为内测产品，第三方评价"流量是同期工作流站里最低的，但全中文、速度快、注册送 100 算力"。来源：https://devpress.csdn.net/v1/article/detail/136096950

**新模型筛选支持 / workflow 导出 / 商用**
- **【未核实】** 本次搜索未找到 2026 年的近期资料（可用的中文资料集中在 2024 年），当前运营状态、新模型覆盖、下载与商用政策均未核实。
- **【注意】** 搜索结果中的 electricsheep.tv 是英国的视频剪辑公司，与国内"电子羊 eSheep"无关，勿混淆。

**优缺点与适合人群**
- **【推断】** 优点是中文、轻量、下载门槛低；缺点是资料陈旧、活跃度存疑。建议作为备用站点偶尔逛逛，不要当主力。

---

## 3. 重要横向事实：新模型的许可差异（下载前必看）

- **Qwen-Image（2025-08，20B）**：Apache 2.0，可商用。来源：https://docs.comfy.org/tutorials/image/qwen/qwen-image 、https://directoryo.com/zh/6345
- **Qwen-Image-2.1（2026-09，7B 生成+编辑一体）**：**Qwen Research License，禁商用**（Civitai 上的 2.1 派生 LoRA 页均标注该许可）。来源：https://blog.csdn.net/liferecords/article/details/166327653 、https://civitai.red/models/2997646?modelVersionId=3400002
- **Z-Image 系列**：Z-Image-Turbo（2025-11，6B，8 步蒸馏）、Z-Image-Base（2026-01，非蒸馏、供微调）均为 **Apache 2.0**；另有 Z-Image-Edit。来源：https://andrewaltimit.github.io/Documentation/docs/ai-ml/base-models-comparison.html 、https://www.liblib.art/modelinfo/7d72b6aa66844d1e99a51758b7fdbb99 、https://thundercompute.com/blog/best-open-source-image-generation-models
- **FLUX.2**：[dev]（32B）开放权重但**非商用许可**；[klein] 4B **Apache 2.0**；9B 口径存在分歧（一说非商用）。Civitai FLUX.2 LoRA 页多标注 FLUX Non-Commercial License v2.1。来源：https://andrewaltimit.github.io/Documentation/docs/ai-ml/base-models-comparison.html 、https://huggingface.co/spaces/rbaks/serevia-imagen/blob/main/wiki/concepts/model-selection.md 、https://civitai.com/models/2252538/flux2conceptlorav1
- **Wan 2.x / HunyuanVideo / LTX**：官方权重在 HF 与 ModelScope 双渠道发布；Civitai 视频 LoRA 训练器支持的底模为 Hunyuan 720p / Wan 2.1 T2V&I2V 14B / LTX2。来源：https://education.civitai.com/using-civitai-the-on-site-lora-trainer/ 、https://tensor.art/ （HunyuanVideo 1.5 由腾讯混元官方账号上传）

---

## 4. 按需求选平台（详细版）

| 场景 | 推荐顺序 | 说明 |
|---|---|---|
| 找 Qwen-Image / Edit 系 LoRA | Civitai → HF → LiblibAI → 魔搭 → Tensor.Art/吐司 | Civitai 用 base model 筛选（Qwen 生态页）；HF 看 `base_model: Qwen/Qwen-Image*` 仓库；LiblibAI 有大量中文电商/国风 Qwen LoRA。注意 2.1 系禁商用 |
| 找 Z-Image / Turbo LoRA | Civitai → Tensor.Art/吐司 → LiblibAI → 魔搭 → HF | Z-Image 标签在 Civitai（ZImageBase/Z-image Turbo）与 Tensor.Art（Z-Image/Z-Image-Turbo）均已成熟；LiblibAI 有 Z-Image 官方级工作流与配套 LoRA；Tensor.Art 可免费/低价在线训练 Z-Image LoRA |
| 找 FLUX.2 LoRA | Civitai → HF | 2K+ LoRA（官方生态页读数）；HF 用 base_model 标签。klein 4B 系注意许可 |
| 找 Wan 2.x 视频 LoRA | Civitai（Wan 生态）→ 魔搭（Wan-AI 官方）→ HF | Civitai 有 Wan Video 2.2 T2V-A14B 标签与高低噪双 LoRA 惯例；魔搭有 Wan-AI 官方仓 |
| 找 HunyuanVideo / LTX LoRA | Civitai → HF/魔搭 | Civitai 训练器原生支持 Hunyuan 720p 与 LTX2 训练；生态页有 LTX Video 专区 |
| 下载官方底模/量化 | HF + 魔搭（双渠道）→ Comfy-Org repackaged 仓 | 国内优先魔搭（免梯子）；ComfyUI 用户优先 Comfy-Org 的 repackaged 目录结构 |
| 下载官方 workflow JSON | docs.comfy.org 教程页 / comfy.org/workflows / ComfyUI 内置模板 | 全部带 Download JSON，随版本更新 |
| 下载社区 workflow JSON（免梯子） | ComfyWorkflows.com → LiblibAI workflow 模版 → eSheep | ComfyWorkflows 免注册直下 JSON/PNG；LiblibAI 中文且工作流+模型一站配齐 |
| 下载社区 workflow JSON（可翻墙） | OpenArt → ComfyWorkflows → Civitai workflow 分区 → RunComfy 模板 | OpenArt 数量最大；RunComfy 按 Z-Image/Qwen/Wan/LTX 分区且能下 JSON |
| 修 workflow 报红节点 | registry.comfy.org + ComfyUI Manager"Install Missing Custom Nodes" | Manager 本地映射 5590 包/42497 节点名（2026-08 读数） |
| 白嫖算力跑/训 | 魔搭 AIGC 专区（全免费）→ Tensor.Art（每日积分）→ Civitai（Blue Buzz） | 魔搭免费 GPU notebook + 免费 LoRA 训练最激进 |
| 在线训练 LoRA | Civitai 训练器（图像+视频）/ Tensor.Art（Z-Image 训练免费活动）/ 魔搭（Qwen 系免费） | 按底模与预算选 |
| 追新模型第一手信息 | Reddit r/StableDiffusion → blog.comfy.org → HF → 各平台标签页 | 新模型通常：HF/魔搭发权重 → ComfyUI 官方博客发模板 → 社区 LoRA 一周内铺开 |

---

## 5. 主要结论（8 条）

1. **官方渠道优先**：新模型（Qwen-Image 2.1、Z-Image、LTX 2.x、Wan Animate 2 等）的 ComfyUI 原生支持通常在发布当日即由 docs.comfy.org 提供带 Download JSON 的教程与内置模板，兼容性最高，应作为 workflow 的第一站。
2. **Civitai 是 LoRA 侧覆盖最广的平台**：已为 FLUX.2 / Qwen Image / Z-Image / Wan / LTX Video 等建立官方生态页，base model 筛选精细（如 Qwen 2.1、ZImageBase、Wan Video 2.2 T2V-A14B），但国内需梯子、文本搜索噪音大（要用筛选器别用搜索框）。
3. **国内"直连三件套"**：LiblibAI（Z-Image/Qwen 本土生态最厚 + workflow 模版免费下）、魔搭（官方权重最快 + 免费算力训练）、吐司/Tensor.Art（新模型标签覆盖最全 + 免费/低价在线训练 Z-Image LoRA）。
4. **workflow JSON 免费直下**：OpenArt 与 ComfyWorkflows 都无需注册即可下载 JSON/PNG 并拖入 ComfyUI；ComfyWorkflows 对国内网络更友好（中文教程口径），并支持"自动安装缺失自定义节点"。
5. **Tensor.Art 与吐司是同一家**（上海必有回响），模型库互通；其新模型 base model 标签（Z-Image-Turbo / Qwen-Image-2.1 / FLUX_2_KLEIN / Wan 2.6 / HunyuanVideo 1.5 / LTX-2）在本次调研的平台中覆盖最全，但其工作流偏站内闭环，本地导出兼容性未获官方确认。
6. **云端平台各有定位**：RunComfy 主打"拖入你的 JSON 自动配好一切环境"（按模型分区的模板可下 JSON）；RunningHub 主打"在线运行+应用发布+API"，工作流默认私有、非 JSON 仓库。
7. **许可差异必须逐模型确认**：Qwen-Image（20B）Apache 2.0 可商用，但 Qwen-Image-2.1 改为 Qwen Research License（禁商用）；Z-Image 全系 Apache 2.0；FLUX.2 [dev]/9B 非商用、[klein] 4B Apache 2.0。LiblibAI 有成文的 workflow+模型双重商用规范。
8. **未能核实的关键项**：OpenArt/ComfyWorkflows 是否已建立 Qwen/Z-Image/FLUX.2 的新模型筛选标签；Shakker 是否跟进 2026 新模型（迹象是滞后）；eSheep 当前运营状态；RunningHub 站内工作流能否导出本地 JSON；OpenArt 国内直连与否（两派说法矛盾）。以上使用前请自行验证。

---

## 附：本报告引用来源清单（按平台）

- Civitai: civitai.com/ecosystems, civitai.com/ecosystems/flux2, civitai.com/models/1837629, civitai.red/models/2181569, civitai.red/models/2997646, education.civitai.com/using-civitai-the-on-site-lora-trainer/, developer.civitai.com/orchestration/recipes/flux2
- Hugging Face: huggingface.co/flymy-ai/qwen-image-realism-lora, huggingface.co/d14945921/qwen_hibrid-lora, huggingface.co/t8star/Qwen-Image-2.1-viggle-turbo-4step-r64-comfy, huggingface.co/spaces/rbaks/serevia-imagen（wiki）
- OpenArt: openart.ai/home, ai-media-studio.com/openart-review, aihr.wang/sites/660.html, blog.csdn.net/weixin_58401027/article/details/157776141, blog.csdn.net/weixin_29057065/article/details/165968925
- ComfyWorkflows: comfyworkflows.com, ui.marklion.cn/article/7e7/13/article-b146d11fdf84b557.shtml, github.com/abk234/ComfyUI（DOWNLOAD_SOLUTIONS.md）
- Tensor.Art / 吐司: tensor.art, tensorart.me, tensor.art/about/aitool-tutorial, github.com/Tensor-Art/ComfyUI_TENSOR_ART, tahou.com/article/211846057108775941, hypeart.ai/commercial-use/tensor-art-ai, lemonsight.com/tool/image/tensor-art
- SeaArt: docs.seaart.ai, seaart.me/models/detail/1f7fa1890d70f578d87ccaec7c0a504d, seaart.me/models/detail/bc50f1de13f1691f2e4fde5d95b908f3, lilting.ch/en/articles/seaart-lora-guide
- Shakker: shakker.ai, aigearbase.com/tool/shakker-ai, forwardfuture.com/tools/details/shakker.ai, aidirectory.com/shakker
- ComfyUI 官方: comfy.org, comfy.org/zh/workflows/, docs.comfy.org/tutorials/image/qwen/qwen-image, docs.comfy.org/zh/tutorials/image/qwen/qwen-image-2-1, blog.comfy.org/p/qwen-image-in-comfyui-new-era-of, docs.comfy.org/manager/legacy-ui, registry.comfy.org（经 localaimaster.com/blog/comfyui-missing-node-types 转述）, github.com/Comfy-Org/ComfyUI-Manager, github.com/comfyanonymous/ComfyUI_examples, comfyanonymous.github.io/ComfyUI_examples/
- RunComfy: runcomfy.com（含 es 语言页与多个镜像参数页）
- GitHub 列表/Reddit: github.com/lucianosb/awesome-comfyui, deadsubs.com/best/ai-art-subreddits, blog.csdn.net/weixin_33531560/article/details/157245276, articles.waytoagi.com/docs/IgJwwPLhsieJHFkKUaOcnz0hnme, waytoagi.com/question/78171
- LiblibAI: pc.liblib.art, liblib.art/modelinfo/758ced0f..., liblib.art/modelinfo/7d72b6aa..., liblib.art/modelinfo/efe968c6..., liblib.art/activities/e0aa50f2.../Commercial_Guidelines, x-humanoid.csdn.net/6a9edd3e1a35784c841dcac6.html, biemoyu.com/sites/liblibai-guanwang.html, aixin315.com/sites/6474.html
- ModelScope: modelscope.cn, modelscope.cn/posts/1419, modelscope.cn/models/huAAUU/FixBadImage-KontextLora, linkingapi.com/archives/16236, modelers.csdn.net/69a7d2817bbde9200b9da4ff.html
- RunningHub: runninghub.cn, runninghub.cn/runninghub-api-doc-en/doc-8287465, aizxs.com/tool/runninghub, softtool.ai/tool/runninghub/160
- 许可参考: andrewaltimit.github.io/Documentation/docs/ai-ml/base-models-comparison.html, thundercompute.com/blog/best-open-source-image-generation-models, blog.csdn.net/liferecords/article/details/166327653, directoryo.com/zh/6345, qwen.ai/blog?id=qwen-image-2.1
- 其它: sucaijishi.com/articles-51-571-1.html, docs.feishu.cn/v/wiki/JoDRwnWz6iOi3SkYMxwcDycInkh/ai, devpress.csdn.net/v1/article/detail/136096950, cloud.tencent.com/developer/news/1292442, 516cn.cn/sites/1045.html, ebingou.cn/gongju/19176.html, aixq.cc/31147.html, hokai.io/hub/tools/civitai, aitoolservices.com/civitai-review-2026-..., aiimagegeneratornsfw.com/civitai-not-working-fix-2026, eastondev.com/blog/en/posts/ai/20260602-comfyui-workflow-reuse-guide/
