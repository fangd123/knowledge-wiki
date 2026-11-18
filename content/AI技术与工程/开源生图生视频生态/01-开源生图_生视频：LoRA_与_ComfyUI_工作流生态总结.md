---
title: "开源生图/生视频：LoRA 与 ComfyUI 工作流生态总结"
date: 2026-11-08
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: 开源生图生视频生态
series_order: 1
tags:
  - AI技术与工程
  - 开源生图生视频生态
ai_assisted: true
status: published
---

# 开源生图/生视频：LoRA 与 ComfyUI 工作流生态总结

> 调研日期 2026-11-08。分三路交给 CodeBuddy（glm-5.3-flash）联网调研，Claude 抽查关键事实后汇总。
> 五个公开分册：[分享平台](02-%E5%BC%80%E6%BA%90%E7%94%9F%E5%9B%BE_%E7%94%9F%E8%A7%86%E9%A2%91%E6%A8%A1%E5%9E%8B%E7%9A%84_LoRA_%E4%B8%8E_ComfyUI_Workflow_%E5%88%86%E4%BA%AB%E5%B9%B3%E5%8F%B0%E8%B0%83%E7%A0%94%E6%8A%A5%E5%91%8A.md) · [生图场景清单](03-%E4%BB%BB%E5%8A%A1_B%EF%BC%9A%E5%BC%80%E6%BA%90%E7%94%9F%E5%9B%BE%E6%A8%A1%E5%9E%8B%E7%9A%84%E7%83%AD%E9%97%A8_LoRA_%E4%B8%8E_ComfyUI_Workflow%EF%BC%88%E6%8C%89%E5%9C%BA%E6%99%AF%EF%BC%89.md) · [视频场景清单](04-%E5%BC%80%E6%BA%90%E8%A7%86%E9%A2%91%E7%94%9F%E6%88%90%E6%A8%A1%E5%9E%8B%E7%9A%84%E7%83%AD%E9%97%A8_LoRA_%E4%B8%8E_ComfyUI_Workflow_%E8%B0%83%E7%A0%94%E6%8A%A5%E5%91%8A.md) · [Civitai 推荐索引](05-Civitai_%E6%8E%A8%E8%8D%90_LoRA%EF%BC%9A%E6%96%87%E7%AB%A0%E4%B8%8E%E6%B8%85%E5%8D%95%E7%B4%A2%E5%BC%95.md) · [LiblibAI 模型与工作流](06-%E4%BB%BB%E5%8A%A1_D%EF%BC%9ALiblibAI%EF%BC%88%E5%93%A9%E5%B8%83%E5%93%A9%E5%B8%83%EF%BC%89%E4%B8%8A%E6%B5%81%E8%A1%8C%E7%9A%84_LoRA_%E4%B8%8E_ComfyUI_%E5%B7%A5%E4%BD%9C%E6%B5%81%E8%B0%83%E7%A0%94.md)
> 下载量、点赞数都是检索当天的快照，以站内实时数据为准。

---

## 一、去哪找：分享平台

| 需求 | 首选 | 备选 | 说明 |
|---|---|---|---|
| **新模型官方工作流** | ComfyUI 内置模板 / docs.comfy.org / comfy.org/workflows | ComfyUI 官方博客 | 先看模板要求的核心版本、权重和节点，再下 JSON；不保证每个版本都能打开。 |
| **LoRA（覆盖最全）** | **Civitai** | Hugging Face | 每个模型家族都有生态页（Qwen Image / Z-Image / FLUX.2 / Wan / LTX）。**用 Base Model 筛选，不要用搜索框**：搜索噪音极大 |
| **LoRA（国内直连）** | **LiblibAI** | 吐司 Tusi（与 Tensor.Art 同一家）、魔搭 | Liblib 的 Qwen-Edit / Z-Image 中文电商类 LoRA 最多；部分模型只能在线用、不能下载 |
| **官方权重 / 量化** | 魔搭 ModelScope | HF、Comfy-Org repackaged 仓 | 国内下载可先查是否有同版权重；未做线路测速。Comfy-Org 转包通常注明 ComfyUI 目录。 |
| **社区工作流 JSON** | OpenArt、ComfyWorkflows（免注册直接下） | Civitai Workflows 分区、RunComfy、LiblibAI 工作流模版 | ComfyWorkflows 据称国内可直连 |
| **云端跑** | RunningHub（国内）、RunComfy（国际） | Tensor.Art、魔搭 AIGC 专区（免费算力） | RunningHub 偏站内运行和发布应用，**不适合当 JSON 仓库**；RunComfy 能自动补齐节点和模型 |
| **报红修节点** | ComfyUI Manager → Install Missing Custom Nodes | registry.comfy.org | — |
| **追新 / 看口碑** | Reddit r/StableDiffusion、r/comfyui | B 站 | 新模型出来后，第一波工作流通常先出现在这里 |

不推荐当主力的：Shakker（2026 年的新模型基本没跟进）、SeaArt / Tensor.Art 的站内工作流（导出到本地 ComfyUI 不一定兼容）、eSheep（找不到 2026 年的活跃资料）。

---

## 二、生图：热门 LoRA / 工作流（按模型）

### 先理清版本（2026-11）
- **Qwen-Image**：2512 与 Edit-2511 分清生成和编辑；2.1 是材料另列的生成/编辑统一路线。不要把一个版本的发布条款套给另一个。
- **Z-Image**：Turbo（6B，8 步）+ Base（用来训练 LoRA），都是 Apache 2.0。当日材料未取得 Edit 版对应发布权重，不据教程标题认定已经开放。
- **FLUX**：dev、klein 4B 与 9B 不是同一份权重或许可。另有 FLUX 3 Image Klein 的独立发布说明；量化文件小不等于整条工作流能装进同样大的显存。

### 必装的“基础件”
| 件 | 用途 |
|---|---|
| **lightx2v Qwen-Image-Lightning / Edit-2511-Lightning（4/8 步）** | Qwen 系加速的事实标准。CFG 必须 1.0、euler/simple，CFG 超过 2 会烧色 |
| **Phr00t Qwen-Image-Edit-Rapid-AIO** | 单个融合文件，4–6 步出编辑结果，最多 4 张参考图 |
| **InstantX Qwen ControlNet Union + Inpainting**；**Qwen-Image-2.1-Fun-ControlNet-Union**（8 种条件 + inpaint）| Qwen 精确控图 |
| **alibaba-pai Z-Image-Turbo-Fun-ControlNet-Union 2.1 + Tile 版** | Z-Image 控图和分块放大 |
| **SeedVR2**（3B / 7B） | 几乎所有生图、生视频工作流的标准收尾放大件 |
| **city96 ComfyUI-GGUF** | 低显存跑大模型 |

### 按场景
| 场景 | 推荐 | 热度证据 |
|---|---|---|
| **多视角 / 角色设定图** | Qwen-Edit-2511 + **fal Multiple-Angles LoRA**（96 个机位，触发词 `<sks>`，仓库自带 ComfyUI JSON）；tarn59 turnaround | 原调研记 HF 当时月下载约 7.7 万。LiblibAI 转载页的在线生成与下载不是同一栏，不把 12.68 万计作下载 |
| **电商产品图** | Qwen-Edit-2511 + Lightning；换背景（White-to-Scene）、dx8152 Light-Migration 重打光、多角度；oncompute 的“电商五件套”工作流 | — |
| **海报 / 中文文字** | Qwen-Image 2.x（能渲染数百个汉字）；Logo.Redmond | — |
| **人像写真 / 写实** | **Z-Image Turbo + Realistic Snapshot（强度 0.6–0.7）**；Qwen 线用 Lenovo UltraReal、NiceGirls UltraReal | Realistic Snapshot 约 21.6 万下载；UltraReal 约 16.2 万 |
| **去水印 / 去物体 / 局部重绘** | Qwen-Edit + valiantcat Remover-General 或 prithivMLmods Object-Remover | — |
| **换装 try-on / 多图融合** | Qwen-Edit 多图 + Clothes Try On / FoxBaze TryOn / dx8152 Fusion；Consistence Edit LoRA | Consistence Edit 约 5.6 万 |
| **动漫 ↔ 真人** | Anything2Real（Qwen-Edit 版和 FLUX.2 klein 版）；Photo-to-Anime | klein 版约 3.4 万 |
| **老照片修复上色** | Qwen-Edit-2511 修复 → SeedVR2 放大（LiblibAI、RunningHub、Floyo 都有现成工作流）；FLUX.2 klein 官方 restore 模板 | — |
| **低显存极速** | Z-Image Turbo GGUF（6–8GB）、FLUX.2 klein 4B | — |

**怎么挑**：这次找到的商品编辑、多机位附件以 Qwen-Edit 居多；写实抓拍在 Z-Image 上好找，FLUX.2 有些任务不用外挂附件。这里按能找到的任务选，不是把不同平台的计数混排。

---

## 三、生视频：热门 LoRA / 工作流

### 先理清版本（2026-11）
- **Wan**：本文能定位 2.1/2.2 与 Animate 系列工作流。当日材料也列有后续版本的 API，但有 API 不等于有可下载权重，反过来也不能证明从未发布；2.7 / 3.0 的开放状态未获对应版本发布记录支持。
- **音视频一体**是另一条线：LTX-2.x 与 MiniMax H3 的模板把声音和画面放在同一流程。引用的设备档位来自发布方或教程，本文未在同一台机器比较。
- HunyuanVideo 1.5（8.3B，fp8 约 14GB 起）质量不错，但社区 LoRA 比 Wan 少很多。

### 加速 LoRA（Wan 必装）
| LoRA | 要点 |
|---|---|
| **LightX2V / Wan2.2-Lightning 4 步** | 官方模板内置的开关。**高噪、低噪专家必须各挂对应的 LoRA**；CFG=1；会让**运动幅度变小**。建议草稿用 4 步，终稿关掉加速跑 20 步 |
| Wan2.1 lightx2v rank32/64 | 2.1 的 LoRA 在 2.2 上**只能挂在低噪专家**；Animate 和 S2V 模板至今还在用它 |
| CausVid / AccVid / rCM / FusionX | 早期常用的加速方案。rCM 不适合 I2V；FusionX 由多个来源合并，商用前要自己查授权 |

### 按场景
| 场景 | 推荐 | 原资料提到的设备档位（非整流程峰值实测） |
|---|---|---|
| **特效短视频**（360 旋转 / 挤压 / 膨胀 / 变身） | **Wan 2.1 I2V 14B 480p + Remade 系列 LoRA**（几乎都长在这个底模上）+ Kijai 的 480p I2V 示例工作流 | 资料举例 12–16GB；看量化、帧数和卸载 |
| **运镜 / 航拍感** | Wan2.2 **Fun Camera**（官方模板）；ReCamMaster；Remade 的 Orbit / Crane LoRA | 24GB fp8 |
| **首尾帧转场** | Wan2.2 FLF2V / Fun InP（官方模板） | 24GB |
| **动作迁移 / 视频换人** | **Wan Animate 2**（不需要预处理）或 Wan2.2 Animate（Move / Mix 模式） | 16–24GB |
| **舞蹈 / 姿态控制** | Wan VACE（pose）+ Fun Control | 24GB |
| **口播数字人** | **Wan2.2 S2V**（官方模板，最省心）；**InfiniteTalk**（分段接续、多人对话，社区首选）；HunyuanVideo-Avatar；FantasyTalking | 24GB；InfiniteTalk 720p 约 13–16GB |
| **商品 / 人物跨镜头一致** | Phantom、Stand-In、Lynx（都在 Kijai Wrapper 里） | 24GB |
| **长视频** | SkyReels-V2 DF 接力、Wan Context Windows；FramePack（6GB / 60 秒） | — |
| **音视频一体成片** | LTX-2.5（4K、多镜头）；MiniMax H3（注意地区许可） | 24GB INT8 |
| **视频修复放大** | 先降采样 → SeedVR2 / FlashVSR → RIFE 补帧 → VHS 编码 | 8–24GB |

**核心工具**：**Kijai ComfyUI-WanVideoWrapper**（约 6.8k star，注册表下载 200 万+，VACE / InfiniteTalk / Phantom / ATI / ReCamMaster 等基本都靠它）+ **QuantStack Wan2.2 GGUF**（I2V 版下载 80 万+）。

**生态成熟度**：Wan 2.1/2.2 遥遥领先 > LTX-2.x（官方 IC-LoRA / ID-LoRA 齐全，但社区第三方内容少）> HunyuanVideo 1.5 > SkyReels / MiniMax H3。

---

## 四、下载前查哪一份许可
| 手上的权重版本 | 先查的条款与问题 |
|---|---|
| Qwen-Image 2512 / Edit-2511 | 分别看 [2512 模型卡](https://huggingface.co/Qwen/Qwen-Image-2512) 与 [Edit-2511 模型卡](https://huggingface.co/Qwen/Qwen-Image-Edit-2511) 的 License，不要借用另一版本的标签。 |
| Qwen-Image-2.1 | 单独核该版本随权重的 Research License，与 2512 区分。 |
| Z-Image Turbo / Base | 核所下载 Turbo 或 Base 仓随附的 LICENSE；第三方 LoRA 另看作者条款。 |
| FLUX.2 dev / klein 9B | 查对应版本原始权重许可；9B 转包页标记不代替底模条款。 |
| FLUX.2 klein 4B | 4B 的标记只对应 4B；不外推到 9B 或融合包。 |
| FLUX 3 Image Klein | 从[该版发布说明](https://bfl.ai/blog/flux-3-image-klein/)查对应许可的适用主体与条件。 |
| Wan 2.1 / 2.2 / Animate | 核目标模型卡及其 LICENSE，区分原版权重、转包和社区合并。 |
| LTX-2.x | 按具体小版本的社区许可核对主体、营收门槛和使用方式。 |
| MiniMax H3 / 腾讯 Hunyuan | 是不同许可；分别核对应权重的地域、主体与用途，不用同一个勾叉判断。 |
| 社区 LoRA / 工作流 | 底模、附件、平台服务、输入素材是不同层次。这张表提供查证入口，不对成品作商用授权判断。 |

## 五、待核实
OpenArt 国内能否直连（两种说法互相矛盾）；Civitai 上 Wan 视频风格类 LoRA 的具体榜单（CodeBuddy 没拿到）；Wan 2.7 / 3.0 是否有开源权重（来源互相矛盾）；Z-Image Turbo 在 HF 上约 838 个 LoRA 的数字（来自第三方统计）。
