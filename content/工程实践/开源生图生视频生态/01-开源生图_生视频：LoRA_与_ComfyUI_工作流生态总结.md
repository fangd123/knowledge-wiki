---
title: "开源生图/生视频：LoRA 与 ComfyUI 工作流生态总结"
date: 2026-11-08
author: fangd123
license: CC BY-NC-SA 4.0
category: 工程实践
series: 开源生图生视频生态
series_order: 1
tags:
  - 工程实践
  - 开源生图生视频生态
ai_assisted: true
status: published
---

# 开源生图/生视频：LoRA 与 ComfyUI 工作流生态总结

> 调研日期 2026-11-08。分三路交给 CodeBuddy（glm-5.3-flash）联网调研，Claude 抽查关键事实后汇总。
> 详细版（含全部来源链接、事实/推断/未核实标注）：01-分享平台调研 · 02-生图LoRA与工作流 · 03-生视频LoRA与工作流 · 04-Civitai推荐文章索引 · 05-LiblibAI热门LoRA与工作流
> 下载量、点赞数都是检索当天的快照，以站内实时数据为准。

---

## 一、去哪找：分享平台

| 需求 | 首选 | 备选 | 说明 |
|---|---|---|---|
| **新模型官方工作流** | ComfyUI 内置模板 / docs.comfy.org / comfy.org/workflows | ComfyUI 官方博客 | 新模型基本当天就有官方模板和 Download JSON，兼容性最好。**任何新模型先看官方模板** |
| **LoRA（覆盖最全）** | **Civitai**（需梯子） | Hugging Face | 每个模型家族都有生态页（Qwen Image / Z-Image / FLUX.2 / Wan / LTX）。**用 Base Model 筛选，不要用搜索框**：搜索噪音极大 |
| **LoRA（国内直连）** | **LiblibAI** | 吐司 Tusi（与 Tensor.Art 同一家）、魔搭 | Liblib 的 Qwen-Edit / Z-Image 中文电商类 LoRA 最多；部分模型只能在线用、不能下载 |
| **官方权重 / 量化** | **魔搭 ModelScope**（国内最快）| HF、Comfy-Org repackaged 仓 | ComfyUI 用户优先下 Comfy-Org 打包的版本，目录结构直接对得上 |
| **社区工作流 JSON** | OpenArt、ComfyWorkflows（免注册直接下） | Civitai Workflows 分区、RunComfy、LiblibAI 工作流模版 | ComfyWorkflows 据称国内可直连 |
| **云端跑** | RunningHub（国内）、RunComfy（国际） | Tensor.Art、魔搭 AIGC 专区（免费算力） | RunningHub 偏站内运行和发布应用，**不适合当 JSON 仓库**；RunComfy 能自动补齐节点和模型 |
| **报红修节点** | ComfyUI Manager → Install Missing Custom Nodes | registry.comfy.org | — |
| **追新 / 看口碑** | Reddit r/StableDiffusion、r/comfyui | B 站 | 新模型出来后，第一波工作流通常先出现在这里 |

不推荐当主力的：Shakker（2026 年的新模型基本没跟进）、SeaArt / Tensor.Art 的站内工作流（导出到本地 ComfyUI 不一定兼容）、eSheep（找不到 2026 年的活跃资料）。

---

## 二、生图：热门 LoRA / 工作流（按模型）

### 先理清版本（2026-11）
- **Qwen-Image**：2512（20B，**Apache 2.0 可商用**）→ **2.1（2026-09，7B，生成+编辑一体，原生 RGBA，⚠️ Qwen Research License，不能商用）**。编辑线最新是 **Edit-2511**。
- **Z-Image**：Turbo（6B，8 步）+ Base（用来训练 LoRA），都是 Apache 2.0。**Z-Image-Edit 仍未发布**，网上的“实测教程”不要信。
- **FLUX.2**：dev（32B，不能商用）、klein 4B（Apache）/ 9B（不能商用）。**新的 FLUX 3 Image Klein 已于 2026-10-15 发布**（5.6B，年营收 < $5M 可免费商用，INT8 版 8GB 显存可跑），生态刚起步。

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
| **多视角 / 角色设定图** | Qwen-Edit-2511 + **fal Multiple-Angles LoRA**（96 个机位，触发词 `<sks>`，仓库自带 ComfyUI JSON）；tarn59 turnaround | HF 上月下载 7.7 万（已核实）；LiblibAI 转载 12.7 万 |
| **电商产品图** | Qwen-Edit-2511 + Lightning；换背景（White-to-Scene）、dx8152 Light-Migration 重打光、多角度；oncompute 的“电商五件套”工作流 | — |
| **海报 / 中文文字** | Qwen-Image 2.x（能渲染数百个汉字）；Logo.Redmond | — |
| **人像写真 / 写实** | **Z-Image Turbo + Realistic Snapshot（强度 0.6–0.7）**；Qwen 线用 Lenovo UltraReal、NiceGirls UltraReal | Realistic Snapshot 约 21.6 万下载；UltraReal 约 16.2 万 |
| **去水印 / 去物体 / 局部重绘** | Qwen-Edit + valiantcat Remover-General 或 prithivMLmods Object-Remover | — |
| **换装 try-on / 多图融合** | Qwen-Edit 多图 + Clothes Try On / FoxBaze TryOn / dx8152 Fusion；Consistence Edit LoRA | Consistence Edit 约 5.6 万 |
| **动漫 ↔ 真人** | Anything2Real（Qwen-Edit 版和 FLUX.2 klein 版）；Photo-to-Anime | klein 版约 3.4 万 |
| **老照片修复上色** | Qwen-Edit-2511 修复 → SeedVR2 放大（LiblibAI、RunningHub、Floyo 都有现成工作流）；FLUX.2 klein 官方 restore 模板 | — |
| **低显存极速** | Z-Image Turbo GGUF（6–8GB）、FLUX.2 klein 4B | — |

**生态成熟度**：Qwen-Image/Edit 的 LoRA 最多，编辑类基本是它一家独大 > Z-Image（写实类增长最快，风格类偏少）> FLUX.2（很多需求被模型自带的编辑能力覆盖了）> FLUX 3 Klein（刚起步）。

---

## 三、生视频：热门 LoRA / 工作流

### 先理清版本（2026-11）
- **Wan 的开源权重止于 2.2**（2025-07，Apache 2.0）。Wan 2.5 / 2.6 只有 API；2.7 / 3.0 是否开源说法不一，主流来源认为没有开源。Wan 系最新的开源模型是 **Wan Animate 2 / Lite**（ComfyUI 2026-08-08 起原生支持）。
- 2026 年的新趋势是**音视频一体**：**LTX-2.x**（2.5 有 INT8 版，24GB 可跑；年营收 < $10M 免费）和 **MiniMax H3**（剪枝 int8 约 12GB；⚠️ 许可排除美国、欧盟、英国、韩国）。
- HunyuanVideo 1.5（8.3B，fp8 约 14GB 起）质量不错，但社区 LoRA 比 Wan 少很多。

### 加速 LoRA（Wan 必装）
| LoRA | 要点 |
|---|---|
| **LightX2V / Wan2.2-Lightning 4 步** | 官方模板内置的开关。**高噪、低噪专家必须各挂对应的 LoRA**；CFG=1；会让**运动幅度变小**。建议草稿用 4 步，终稿关掉加速跑 20 步 |
| Wan2.1 lightx2v rank32/64 | 2.1 的 LoRA 在 2.2 上**只能挂在低噪专家**；Animate 和 S2V 模板至今还在用它 |
| CausVid / AccVid / rCM / FusionX | 早期常用的加速方案。rCM 不适合 I2V；FusionX 由多个来源合并，商用前要自己查授权 |

### 按场景
| 场景 | 推荐 | 显存（你的 3090 是 24GB） |
|---|---|---|
| **特效短视频**（360 旋转 / 挤压 / 膨胀 / 变身） | **Wan 2.1 I2V 14B 480p + Remade 系列 LoRA**（几乎都长在这个底模上）+ Kijai 的 480p I2V 示例工作流 | 12–16GB 就够 |
| **运镜 / 航拍感** | Wan2.2 **Fun Camera**（官方模板）；ReCamMaster；Remade 的 Orbit / Crane LoRA | 24GB fp8 |
| **首尾帧转场** | Wan2.2 FLF2V / Fun InP（官方模板） | 24GB |
| **动作迁移 / 视频换人** | **Wan Animate 2**（不需要预处理）或 Wan2.2 Animate（Move / Mix 模式） | 16–24GB |
| **舞蹈 / 姿态控制** | Wan VACE（pose）+ Fun Control | 24GB |
| **口播数字人** | **Wan2.2 S2V**（官方模板，最省心）；**InfiniteTalk**（无限长、多人对话，社区首选）；HunyuanVideo-Avatar；FantasyTalking | 24GB；InfiniteTalk 720p 约 13–16GB |
| **商品 / 人物跨镜头一致** | Phantom、Stand-In、Lynx（都在 Kijai Wrapper 里） | 24GB |
| **长视频** | SkyReels-V2 DF 接力、Wan Context Windows；FramePack（6GB / 60 秒） | — |
| **音视频一体成片** | LTX-2.5（4K、多镜头）；MiniMax H3（注意地区许可） | 24GB INT8 |
| **视频修复放大** | 先降采样 → SeedVR2 / FlashVSR → RIFE 补帧 → VHS 编码 | 8–24GB |

**核心工具**：**Kijai ComfyUI-WanVideoWrapper**（约 6.8k star，注册表下载 200 万+，VACE / InfiniteTalk / Phantom / ATI / ReCamMaster 等基本都靠它）+ **QuantStack Wan2.2 GGUF**（I2V 版下载 80 万+）。

**生态成熟度**：Wan 2.1/2.2 遥遥领先 > LTX-2.x（官方 IC-LoRA / ID-LoRA 齐全，但社区第三方内容少）> HunyuanVideo 1.5 > SkyReels / MiniMax H3。

---

## 四、商用许可速查（下载前看一眼）
| 模型 | 能否商用 |
|---|---|
| Qwen-Image 2512 / Edit-2511 | ✅ Apache 2.0 |
| **Qwen-Image-2.1** | ❌ Qwen Research License |
| Z-Image Turbo / Base | ✅ Apache 2.0 |
| FLUX.2 dev、klein 9B | ❌ 非商用 |
| FLUX.2 klein 4B | ✅ Apache 2.0 |
| FLUX 3 Image Klein | ⚠️ 年营收 < $5M 免费 |
| Wan 2.1 / 2.2 / Animate 2 | ✅ Apache 2.0 |
| LTX-2.x | ⚠️ 年营收 < $10M 免费 |
| MiniMax H3 / 腾讯 Hunyuan 系 | ⚠️ 有地区排除条款 |
| 社区 LoRA | 逐个看模型页 License；LiblibAI 规定工作流和其中每个模型的许可都要满足 |

## 五、待核实
OpenArt 国内能否直连（两种说法互相矛盾）；Civitai 上 Wan 视频风格类 LoRA 的具体榜单（CodeBuddy 没拿到）；Wan 2.7 / 3.0 是否有开源权重（来源互相矛盾）；Z-Image Turbo 在 HF 上约 838 个 LoRA 的数字（来自第三方统计）。
