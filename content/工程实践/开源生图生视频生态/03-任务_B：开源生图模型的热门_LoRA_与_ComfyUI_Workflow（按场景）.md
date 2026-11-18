---
title: "任务 B：开源生图模型的热门 LoRA 与 ComfyUI Workflow（按场景）"
date: 2026-11-08
author: fangd123
license: CC BY-NC-SA 4.0
category: 工程实践
series: 开源生图生视频生态
series_order: 3
tags:
  - 工程实践
  - 开源生图生视频生态
ai_assisted: true
status: review
---

# 任务 B：开源生图模型的热门 LoRA 与 ComfyUI Workflow（按场景）

> 调研日期：2026-11-08。全部结论基于当日联网检索（官方文档 / Hugging Face / Civitai / LiblibAI / ModelScope / GitHub / ComfyUI 官方模板库 / 社区评测）。
>
> **标注约定**：【事实】= 有来源直接佐证；【推断】= 根据多来源交叉或常识推断；【未核实】= 有线索但数字/细节拿不到权威确认。所有下载量/点赞数为抓取时点的页面数字，Civitai 页面渲染有乱码风险，个位数差异请以原页面为准。

---

## 0. 版本速览（先弄清"最新"指什么）【事实】

| 模型线 | 版本时间线 | 当前最新 |
|---|---|---|
| Qwen-Image 文生图 | 2508（2025-08，20B MMDiT）→ **2512**（2025-12，写实增强）→ **2.0**（2026-02-10，7B、原生 2K、生成+编辑统一）→ **3.0 官宣**（2026-07-21，4.5K token，开放权重情况未核实）→ **2.1**（2026-09-20，7B 单流 DiT、原生 RGBA、10 参考图） | 2.1（开放权重最新）/ 2512（老架构最新） |
| Qwen-Image-Edit | 2508 → **2509**（2025-09，多图输入）→ **2511**（2025-11，一致性/多角度/几何推理增强、可挂 LoRA） | 2511 |
| Z-Image | **Turbo**（2025-11-26，6B，8 步蒸馏）→ **Base**（2026-01-27，28-50 步可微调）→ Edit / Omni-Base：截至 2026-10 仍为"待发布" | Turbo + Base（**Edit 尚未发布**） |
| FLUX.2 | **dev**（2025-11-25，32B，开源权重，非商用许可）→ **klein**（2026-01-15，4B Apache-2.0 + 9B 非商用，蒸馏） | dev + klein |
| FLUX 3 Image Klein（注意：已是新一代） | 2026-10-15 发布，5.6B，Base（50 步）+ 蒸馏版（16 步），FLUX Kommunity License（年收入 < $5M 免费商用） | 已开源，ComfyUI Day-1 支持 |

来源：ComfyUI 官方文档 https://docs.comfy.org/tutorials/image/qwen/qwen-image-2512 ；ComfyUI Wiki https://comfyui-wiki.com/zh/news/2026-09-21-qwen-image-2-1 ；科技日报 https://www.stdaily.com/web/gdxw/2026-02/10/content_472818.html ；RunAIHome FLUX.2 综述 https://www.runaihome.com/blog/flux-2-self-hosting-vram-gguf-guide-2026 ；BFL 官方博客 https://bfl.ai/blog/flux-3-image-klein/ ；BestFreeWebResources（Z-Image 家族状态，2026-10-18 核对） https://www.bestfreewebresources.com/?p=23358

**给用户的提醒**【推断】：用户如果刚装环境，Qwen 线建议直接用 2.1（统一生成+编辑+RGBA）；Z-Image 线只有 Turbo/Base 可用，"Z-Image-Edit"网上不少教程是预告性质的，别照着装；BFL 线 FLUX.2 klein 仍是社区主力，但 2026-10 起部分社区 LoRA 已转向 FLUX 3 Klein。

---

## 1. Qwen-Image（2512 / 2.0 / 2.1）文生图

### 1.1 加速/蒸馏 LoRA【事实】

| 名称 | 类型 | 场景 | 来源 | 热度证据 | ComfyUI 兼容说明 |
|---|---|---|---|---|---|
| **Qwen-Image-Lightning-4steps/8steps V1.0/V2.0**（lightx2v / ModelTC） | 步数蒸馏 LoRA | 文生图加速：50 步 → 4/8 步，CFG=1.0，euler/simple | https://huggingface.co/lightx2v/Qwen-Image-Lightning ；项目页 https://hysenlabs.com/projects/modeltc-lightx2v-qwen-image-lightning （GitHub 1,364 stars） | 官方 2512 模板内置 4 步子图，属"事实标准"【事实】 | LoraLoaderModelOnly 挂 base 模型；官方 2512 模板自带 Lightning 子图；V2.0 比 V1.0 饱和度更低、肤色更自然（项目自述） |
| **Qwen-Image-2512-Lightning-4steps-V1.0**（fp32/bf16，2026-01-01 发布） | 2512 专用蒸馏 | 2512 加速 | 同上 HF 仓库 | 官方模板引用 | 与 `qwen_image_2512_fp8_e4m3fn` / bf16 搭配；也提供预融合 FP8 文件（免运行时合并） |
| GGUF 量化（unsloth / QuantStack / city96 生态） | 量化（非 LoRA，配套使用） | 8-16GB 显存跑 20B 模型 | https://huggingface.co/unsloth/Qwen-Image-Edit-2511-GGUF | Q4_K_M 13.2GB 是 16GB 卡社区默认【事实】 | 需 city96/ComfyUI-GGUF 节点，放 `models/unet` |

**注意事项**【事实，来自官方模板与社区文档】：Lightning 类 LoRA 必须 CFG=1.0、sampler=euler、scheduler=simple；步数超过 6-8 步反而劣化；CFG 拉高到 2 以上会烧色/海报化（Floyo 工作流页明确警告 https://floyo.ai/workflows/qwen-image-edit-2511-lightning-multi-sl3ltb2s5uxo ）；挂 Lightning 后不需要 ModelSamplingAuraFlow shift。

### 1.2 ControlNet / Union / Inpaint【事实】

| 名称 | 类型 | 场景 | 来源 | 热度证据 | ComfyUI 兼容说明 |
|---|---|---|---|---|---|
| **InstantX/Qwen-Image-ControlNet-Union**（Comfy-Org 转包名 Qwen-Image-InstantX-ControlNet-Union） | Union ControlNet | canny / soft edge / depth / openpose 四合一，~3.5GB | https://huggingface.co/InstantX/Qwen-Image-ControlNet-Union ；转包 https://huggingface.co/Comfy-Org/Qwen-Image-InstantX-ControlNets | 社区 2512 教程普遍采用（CSDN 对比文称其"新手首选"）【事实/推断】 | 常规 ControlNetLoader+Apply 节点；强度 0.5-0.8 |
| **Qwen-Image-InstantX-ControlNet-Inpainting**（~4.2GB，同仓库转包） | Inpaint ControlNet | 局部重绘保真 | 同上 | 同上 | models/controlnet 目录 |
| **Qwen-Image-2.1-Fun-Controlnet-Union**（阿里 PAI） | Union + Inpaint（8 条件合一） | canny/depth/灰度/HED/lineart/MLSD/pose/scribble + inpaint，一个 7.5GB 分支叠在冻结底模上 | https://huggingface.co/VideoX-Fun （PAI 系列）；ComfyUI Wiki 报道 https://comfyui-wiki.com/en/news/2026-09-24-qwen-image-2-1-fun-controlnet-union | Kijai 提交的 ComfyUI PR #16519（2026-09-24）【事实】 | ModelPatchLoader 加载（models/model_patches）；control_context_scale 1.0 最强；需较新 ComfyUI |
| DiffSynth Model-Patch / LoRA 控制方案 | 轻量 ControlNet（3/7 种条件） | 低开销控图 | CSDN 三方案对比 https://blog.csdn.net/weixin_42405592/article/details/157283894 | 三方案之一【事实：方案存在；热度未核实】 | ModelPatch 方案几乎无速度损失，但条件类型少 |

### 1.3 风格/写实 LoRA（Civitai 高下载）【事实（数字为 Civitai 页面抓取，2026-09-05 归档目录交叉）】

热度证据来自 HF 归档目录 https://huggingface.co/Alex995647/loras-qwen-image （爬取 Civitai 150 个 Qwen LoRA，2026-09-05）与 Civitai 原页面。

| 名称 | 类型 | 场景 | 来源 | 热度证据（下载） | ComfyUI 兼容说明 |
|---|---|---|---|---|---|
| Lenovo UltraReal | 写实 | 人像/写实摄影增强 | Civitai（经上述目录） | 161,727（目录内第一） | 常规 LoraLoaderModelOnly |
| Aesthetic Quality Modifiers - Masterpiece | 质量词 | 通用画质提升 | 同上 | 157,379 | 可与风格 LoRA 叠加 |
| NiceGirls UltraReal | 写实人像 | 写真人像 | 同上 | 111,810 | 同上 |
| Amateur Photography | 写实抓拍 | 日常随手拍质感 | 同上 | 83,434 | 同上 |
| Logo.Redmond | 商标 | Logo/标志生成（海报文字场景补充） | https://civitai.com （Redmond 系列，作者 ArtificialGuyBR 生态） | 74,259 | 触发词 `LOGO, logoredmaf` |
| Flat Color - Style | 扁平色风格 | 插画/设计稿 | 同上 | 51,144 | — |
| 80s Fantasy Movie | 复古电影感 | 80 年代奇幻剧照 | 同上 | 41,039 | 触发词 `ArsMovieStill` |
| Famegrid (Krea 2 / Z-Image / Qwen) Realism | 多底模写实 | 通用写实（一包多底模） | 同上 | 39,677 | 明确支持 Qwen / Z-Image / Krea 2 |
| 2000s Analog Core | 胶片 | 2000 年代数码/胶片感 | 同上 | 30,523 | — |
| LineAniRedmond / Dark Ghibli Fairytales | 动漫 | 线性漫画 / 暗黑吉卜力 | 同上 | 24,584 / 22,193 | 触发词见模型页 |
| Qwen-Image-Boreal (Boring Reality) | 写实 | "无聊但真实"的日常实拍感 | https://civitai.red/models/ （Boreal 页） | 12.3K | — |
| Anime Redmond（artificialguybr） | 动漫 | 2512 动漫风，Apache-2.0，~81MB | https://huggingface.co/artificialguybr/ANIME-REDMOND-QWENIMAGE ；作者博客 https://artificialguy.com/blog/anime-redmond-qwenimage-lora | HF+Civitai 合计 ~2.5K（新）【事实】 | 建议 0.8-1.0；Civitai 页标注兼容 ZImageTurbo（效果各异） |
| **Qwen-Image-2.1 新生代**：Manga to Real People V1；AniVive_Qwen21 | 漫画→真人 | 2.1 编辑工作流内使用 | https://civitai.com/models/2984825 ；https://civitai.com/models/3001515 | 1.6K 下载/64 评；325 下载/18 评（发布仅 3-4 周）【事实】 | 需 2.1 编辑工作流（TextEncodeQwenImage21），强度 1.0、25 步、CFG 1.0 |

---

## 2. Qwen-Image-Edit（2509 / 2511）

### 2.1 加速 LoRA【事实】

| 名称 | 类型 | 场景 | 来源 | 热度证据 | ComfyUI 兼容说明 |
|---|---|---|---|---|---|
| **Qwen-Image-Edit-2511-Lightning-4steps-V1.0-bf16**（lightx2v） | 蒸馏 LoRA（850MB） | 2511 编辑 40 步 → 4 步 | https://huggingface.co/lightx2v/Qwen-Image-Edit-2511-Lightning | 官方模板/几乎所有第三方 2511 工作流都内置【事实】 | base 40 步 CFG 4.0；挂 LoRA 后 4 步 CFG 1.0；CFG>2 烧色；步数 >6-8 劣化 |
| Qwen-Image-Edit-2509 Lightning（4/8 步） | 蒸馏 LoRA | 2509 加速 | lightx2v HF 仓库 | 同上 | 同上 |
| **Phr00t/Qwen-Image-Edit-Rapid-AIO**（V5→V17，SFW/NSFW/Lite） | 预融合 All-in-One（含 Lightning+VAE+CLIP） | 单文件 4-6 步快速编辑，最多 4 参考图 | https://huggingface.co/Phr00t/Qwen-Image-Edit-Rapid-AIO | Zenn Awesome 列表与大量中文教程引用；有 GGUF 转包（Phil2Sat）【事实：存在与广泛引用；确切下载量未核实】 | 单 checkpoint 直接 Load；推荐 euler a/beta、CFG 1-2、LoRA 权重 0.6-0.8（作者自述）；FP8 单文件 ~29GB【事实】 |

### 2.2 编辑类 LoRA（按场景）【事实：名称与出处来自 Zenn「Awesome-Qwen-Image-Edit」清单（2026-03-07 更新）https://zenn.dev/eques/articles/4a0a15d1c0ccd6 、LiblibAI 模型页、Bilibili/ModelScope 官推清单】

| 名称 | 类型 | 场景 | 来源 | 热度证据 | ComfyUI 兼容说明 |
|---|---|---|---|---|---|
| **fal/Qwen-Image-Edit-2511-Multiple-Angles-LoRA** | 多角度镜头 | 一键 96 种相机位（4 仰角×8 方位×3 距离），支持 -30° 低角度；3000+ 训练对（高斯泼溅数据） | https://huggingface.co/fal/Qwen-Image-Edit-2511-Multiple-Angles-LoRA ；LiblibAI 转载 https://www.liblib.art/modelinfo/043a7b86ba9d4369ad7df27f6e457d02 | **LiblibAI 126.8K 下载 / 1.5K 点赞**【事实】 | 配套自定义节点 jtydhr88/ComfyUI-qwenmultiangle（Qwen Multiangle Camera）；LiblibAI 上有成品工作流（多角度+局部重绘+洗图）https://www.liblib.art/modelinfo/378b802989e04a39a68ad2aa4a4a6ec5 |
| tori29umai/Qwen-Image-2509-MultipleAngles | 多角度 | 2509 版多角度 | https://huggingface.co/tori29umai/Qwen-Image-2509-MultipleAngles | Zenn 清单收录，社区引用多【推断】 | 2509 工作流 |
| dx8152/Qwen-Edit-2509-Multiple-angles | 多角度 | 同上（ModelScope 版） | https://modelscope.cn/models/dx8152/Qwen-Edit-2509-Multiple-angles | Bilibili"神级 LoRA"盘点收录 https://www.bilibili.com/video/BV1zgUrBqE5e/ 【事实：被收录；下载数未核实】 | ModelScope 国内下载方便 |
| **dx8152/Qwen-Image-Edit-2509-Light_restoration** | relight | 一键去除面部光影、柔光重打光 | ModelScope（同上盘点） | 同上 | 2509 |
| **dx8152/Qwen-Edit-2509-Light-Migration / Multi-Angle-Lighting** | relight/换光照 | 光照迁移（给已有光照的图二次布光）、昼夜转换、多角度布光 | 同上 | 同上 | 2509；oncompute 电商套件也引用 Light-Migration【事实】 |
| tarn59/apply_texture / extract_texture | 纹理 | 给物体/建筑/精灵贴纹理；反向提取纹理 | HF（tarn59） | Zenn 清单收录 | 2509 |
| **valiantcat(starsfriday)/Qwen-Image-Edit-Remover-General-LoRA** | 去物体/去水印 | 电商图/人物图/物品图消除（"移除XXX"），保持其余像素不变 | https://huggingface.co/starsfriday/Qwen-Image-Edit-Remover-General-LoRA | HF 镜像页显示月下载 404（镜像口径）【未核实总量】 | 官方 Edit 工作流 + LoraLoaderModelOnly；提示词"从场景中移除XXX" |
| prithivMLmods/Qwen-Image-Edit-2511-Object-Remover | 去物体 | 2511 版消除（含去水印/Logo） | https://huggingface.co/prithivMLmods/Qwen-Image-Edit-2511-Object-Remover | 被 Civitai 热门 remover 工作流采用【事实】 | 2511 GGUF 工作流实测 12 步 CFG 1.2 即可擦水印（社区教程） |
| FoxBaze/Try_On_Qwen_Edit_Lora_Alpha | 换装 try-on | 多参考虚拟试穿（Alpha 版） | HF（FoxBaze） | Zenn 清单收录【事实】 | 2509 多图工作流 |
| **Clothes Try On (Qwen Edit)**（Civitai） | 换装 | 源服装穿到目标人物，体型感知 | https://civitai.com （Qwen Edit 分类） | 8.1K 下载、4.9 分【事实】 | 建议强度 1.5 |
| Outfit Extractor (Qwen Edit)（Civitai） | 换装配套 | 从图中抠出服装供 try-on | 同上 | 7.4K 下载 | 同上 |
| lrzjason/QwenEdit-Anything2Real_Alpha | 真实化 | 任意画风→照片级 | https://huggingface.co/lrzjason/QwenEdit-Anything2Real_Alpha | Zenn+Bilibili 盘点收录；还有 Flux2Klein 9B 移植版（见归档目录第 15 名）【事实】 | 2509 |
| flymy-ai/qwen-image-edit-inscene-lora | 场景内编辑 | in-scene 编辑（改局部不越界） | HF（flymy-ai） | Zenn 清单收录 | 2509 |
| AmirKerr/ThisPerson | "这个人" | 以同一人身份生成新场景（人物一致性） | HF（AmirKerr） | Zenn 清单收录 | 2509 |
| dx8152/Qwen-Image-Edit-2509-Fusion + RunComfy fusion workflow | 多图融合 | 多参考图融合合成 | HF（dx8152）；RunComfy 工作流页 | Zenn 清单收录【事实】 | 2509 多图（TextEncodeQwenImageEditPlus） |
| lividtm/Qwen-Edit-Loras；YaoJiefu/multiple-characters | 多角色 | 一图多角色一致性 | HF | Zenn 清单收录 | 2509 |
| Alissonerdx/BFS-Best-Face-Swap | 换脸/头部 | 高保真换头，色调/光照自然融合 | HF（Alissonerdx） | 有 klein 9B/4B 移植版（FLUX.2 one-node 工作流内置）【事实】 | 2509+ |
| autoweeb/Photo-to-Anime；rzgar/smoothify；drbaph/Caricature；rsshekhawat/3DChibi | 风格转换 | 照片转动漫/平滑卡通/讽刺漫画/3D 手办 | HF | Zenn 清单+Bilibili 盘点 | 2509 |
| lovis93/next-scene；EQUES lineart-interpolation（2509/2511） | 动画/线稿 | 下一镜生成；两图线稿中间帧 | HF | Zenn 清单收录 | 2509/2511 |
| PanelPainter V2 | 漫画上色 | 线稿上色 | Zenn 清单（LiblibAI/HF 有镜像）【未核实原始 repo 链接】 | 清单收录 | 2509 |
| tlennon-ie/qwen-edit-skin | 皮肤修复 | 增强人物皮肤真实感 | HF | Bilibili 盘点收录 | 2509 |
| eigen-ai-labs/eigen-banana-qwen-image-edit；peteromallet/Qwen-Image-Edit-InSubject | 质量增强 | 用 Apple Pico-Banana-400K 数据训练，主题保持/风格参考增强 | HF | Zenn 清单收录【事实】 | 2509 |
| tarn59/character_turnaround_sheet_qwen_image_edit_2509；DiffSynth-Studio/Qwen-Image-Edit-F2P | 角色设定图 | 角色三视图/ turnaround；F2P 面部→人物拉远 | HF | Zenn 清单收录 | 2509 |
| **Consistence Edit Lora**（Civitai） | 一致性 | Qwen Edit 与 Flux2 Klein 通用的一致性保持 | https://civitai.com | 55,925 下载（目录内 Edit 类第一）【事实】 | 强度 0.2-1.5 控制一致性与编辑自由度的平衡 |
| Qwen Edit Figure Maker / Reality Transform / Upscaler & Denoise / Reality Simulator（Civitai） | 手办化/风格/放大/手机实拍 | 各垂直小场景 | Civitai Qwen 分类 | 7.7K / 7.1K / 6.3K 下载【事实】 | 常规挂载 |

### 2.3 热门 Workflow（Qwen-Image-Edit）

| Workflow | 用途 | 自定义节点 | 显存 | 热度证据 |
|---|---|---|---|---|
| **官方模板 image_qwen_image_edit_2511**（单图+多图双轨） | 指令编辑 / 跨图材质迁移 | 内置；可选 Lightning LoRA | fp8mixed 20.5GB；bf16 40.9GB；GGUF Q4_K_M 13.2GB（12GB 卡可用，8GB 卡 Q2 应急） | ComfyUI 模板库自带【事实】来源：https://localaimaster.com/blog/qwen-image-edit-local-guide |
| **多角度+局部重绘+洗图 三合一**（LiblibAI） | 多角度出图、遮罩重绘、ControlNet 洗图、SeedVR2.5 放大 | comfyui_controlnet_aux（LeReS/DepthAnythingV2/Canny/DWPose）、Qwen Multiangle 节点、SeedVR2 | 未标注（20B 模型，建议 16GB+）【推断】 | LiblibAI 工作流页 https://www.liblib.art/modelinfo/378b802989e04a39a68ad2aa4a4a6ec5 【事实】 |
| **换装一致性（Multi 2511 + SeedVR2）**（NextDiffusion） | 多图换装并保纹理 | GGUF 节点、SeedVR2 节点 | 教程按 RunPod 常规 24GB 演示【推断】 | https://nextdiffusion.ai/tutorials/consistent-outfit-changes-with-multi-qwen-image-edit-2511-in-comfyui 【事实】 |
| **电商产品五件套**（oncompute "E-commerce studio"） | 换背景 / 重打光 / 7 角度+360 环拍 / 广告图文案合成 / 生成 5s 短片；共用 20.4GB Edit 底模 + 9.4GB 编码器，按任务换小 LoRA（White-to-Scene、Light-Migration、Multiple-Angles），Turbo 开关即 4 步 Lightning | 内置 | 48GB 跑全部 5 个；24GB 跑 4 个图像工作流（Motion 跑不了，需 LTX-2.3 22B） | https://docs.oncompute.ai/inference-tutorials/turn-one-product-photo-into-a-listing-with-comfyui-and-qwen-image-edit 【事实】 |
| **老照片修复**：LiblibAI「Qwen-Edit-老照片修复」（40 步 CFG 2.0-2.5，提示词"图像整体保持不变,高清,修复照片瑕疵并上色"）；RunningHub「Qwen-2509-老照片修复」(重上色重打光 + SeedVR2 + FlashVSR 4K)；Floyo「2511 Restore Damage Old Photograph」（4 步 Lightning） | 老照片修复+上色 | SeedVR2、FlashVSR、LayerColor 等 | 16GB 可跑（SeedVR2 3B），7B 需 24GB | https://www.liblib.art/modelinfo/746efef5762d470fbbd4a8010585ac4e ；https://www.runninghub.ai/post/2013140024610529281 ；https://floyo.ai/workflows/qwen-image-edit-2511-restore-damage--am8o61kpej59 【事实】 |
| **Qwen-Image-2.1 官方三模板**（t2i / 编辑 / 背景移除抠图） | 统一模型直接出 RGBA 透明底贴图/商品抠图；10 参考图合成 | 无需第三方（ComfyUI ≥0.37.0） | bf16 / int8_convrot 双版本 | ComfyUI Wiki https://comfyui-wiki.com/zh/news/2026-09-21-qwen-image-2-1 【事实】 |
| **本地老照片流水线 vl3c/local-upscale** | Qwen-Image-2.1 修复清理 + SeedVR2-7B 放大，全程本地 | local-image + SeedVR2 CLI | 16GB（local-image ~15.5GB + SeedVR2-7B ~13GB 分时运行） | https://github.com/vl3c/local-upscale 【事实】 |

---

## 3. Z-Image（Turbo / Base / Edit）

### 3.1 模型与加速【事实】

- **Z-Image-Turbo**（2025-11-26，6B S3-DiT，Qwen3-4B 文本编码器，FLUX.1 VAE）：8 步、CFG 锁 1.0（**无负向提示词**）、Apache 2.0。BF16 全家桶 ~16GB、FP8 ~8GB、GGUF ~6GB。官方采样参数：res_multistep + simple，1024×1024。ComfyUI 模板要求 ≥0.11.0。
- **Z-Image（Base）**（2026-01-27）：28-50 步、CFG 3-5、可微调、多样性高；ComfyUI 模板要求 ≥0.23.0。**社区共识：练 LoRA 用 Base，跑图用 Turbo**（官方模型卡明示 Turbo 不建议微调）。
- **Z-Image-Edit / Omni-Base：截至 2026-10-18 仍未发布**（模型卡与 GitHub 均标 "To be released"；GitHub 最后 push 2026-02-09）。【事实】
- Comfy-Org 转包 ModelScope 下载 393,786（2026-09-23 快照，单仓库）【事实】https://modelscope.cn/models/Comfy-Org/z_image_turbo/summary ；第三方称其月下载 ~9.2M（各镜像合计，2026-10 报道）【未核实】。
- Comfy-Org 仓库另附 `z_image_turbo_distill_patch_lora_bf16.safetensors` 蒸馏 patch LoRA【事实：文件存在于官方转包；具体用途官方未详述，未核实】。

### 3.2 LoRA 生态【事实】

| 名称 | 类型 | 场景 | 来源 | 热度证据 | ComfyUI 兼容说明 |
|---|---|---|---|---|---|
| **Realistic Snapshot (Z-Image-Turbo)** | 写实抓拍 | 去塑料 AI 感、业余随手拍质感；v5 "Real Life" 大版本 | https://civitai.com/models/2268008 | **~215.9K 下载 / 1,833 评分（Overwhelmingly Positive）**，2026-01-22 发布，持续更新至 2026-06【事实】 | 建议强度 **0.6-0.7**（作者强调 Turbo 是蒸馏模型，权重叠加易过曝） |
| Realistic Skin Texture Style | 写实 | 皮肤纹理电影感 | Civitai（经 comfyuiweb 目录 https://comfyuiweb.com/ru/resources/lora ） | 118K+ 下载【事实（目录口径）】 | 多底模（ZIT/SDXL/Flux/Pony） |
| Character Design Sheet (3 Perspectives) | 角色设定 | 三视角角色设定图+配色板 | 同上 | 89.9K+ 下载 | 多底模 |
| Amateur Photography | 写实 | 日常实拍 | 同上 | 64.4K+ 下载 | — |
| NiceGirls UltraReal | 写实人像 | 人像 | 同上（作者建议 Turbo 上 1.2-2） | 36.8K+ 下载 | 注意与其他 LoRA 叠加时总权重 ~1 |
| Famegrid Realism（Krea 2 / Z-Image / Qwen） | 写实 | 通用写实 | HF 归档目录 | 39,677 下载 | 一包多底模 |
| Style Greater People's Work | 动漫/插画质量 | Base 模型训练；作者实测 Base LoRA 在 Turbo 上需强度 2-4 才见效【事实（作者自述）】 | https://civitai.red/models/2001848 | 1.5K 下载、40 评（早期 Base 生态样本） | 放 models/loras；Base 底模 |
| HF 生态总量 | — | — | — | 第三方统计 Z-Image Turbo 在 HF 有 ~838 个 LoRA（2026 年中）【未核实（第三方 freakyjolly 统计 https://freakyjolly.com/best-open-source-text-to-image-models/ ）】 | Civitai 筛选：Models→LoRA→Base Model: Z-Image Turbo |

### 3.3 ControlNet 与 Workflow【事实】

| 名称 | 类型 | 场景 | 来源 | 热度证据 | ComfyUI 兼容说明 |
|---|---|---|---|---|---|
| **alibaba-pai/Z-Image-Turbo-Fun-Controlnet-Union** | Union ControlNet | Canny/HED/Depth/Pose/MLSD 五合一；6 块控制分支，100 万图训练 | https://huggingface.co/alibaba-pai/Z-Image-Turbo-Fun-Controlnet-Union ；ModelScope https://www.modelscope.cn/models/PAI/Z-Image-Turbo-Fun-Controlnet-Union | ComfyUI 官方示例页收录工作流【事实】 | ModelPatchLoader + QwenImageDiffsynthControlnet（ComfyUI ≥0.3.77）；放 models/model_patches；control_context_scale 0.65-0.80 |
| **Z-Image-Turbo-Fun-Controlnet-Union-2.1**（2025-12-23） | 同上 + 8 步蒸馏 | 日常推荐 8steps 版；另有 **Tile-2.1-8steps**（分块放大，支持 2048²，"4K 壁纸神器"） | 中文教程 https://www.yalewoo.com/?p=3741/ | PAI 官方迭代版【事实】 | 同上；8 步模型锁定 8 步 |
| **官方模板四件套**：Text to Image / Int8 T2I / Image Upscale 2K / Fun Union ControlNet | 官方 | 全部场景 | https://comfyanonymous.github.io/ComfyUI_examples/z_image/ ；ModelScope 转包页 | 官方自带【事实】 | 模板内置 LoRA 节点（自带一个像素画测试 LoRA） |
| **Z Image (Turbo & Base) Workflow V7**（Civitai） | 全能社区工作流 | 文生图+SAM 3.1 细节修复+Remacri 放大+ADetailer 面部 | https://civitai.com/models/2170134 | **17.8K 下载 / 44 评（Positive）**，2026-05-26 发布，迭代到 V7【事实】 | 需要：Manager、rgthree、Easy-Use、Impact-Pack、Impact-Subpack、Image-Saver 等；模型 z_image_turbo_bf16 + qwen_3_4b + ae.safetensors |
| RunComfy「Z-Image ControlNet」 | 控图教学工作流 | Depth/Canny/Pose 三选一切换 | https://www.runcomfy.com/comfyui-workflows/z-image-controlnet-in-comfyui-controlled-image-generation | RunComfy 收录【事实】 | 用 ImpactSwitch 切换预处理；OpenPose 对特写不可靠（改用 Depth/Canny） |
| GGUF 低显存路线 | 量化 | 6-8GB 卡 | jayn7/Z-Image-Turbo-GGUF + z-image-free 教程 https://z-image-free.com/z-image-loras-guide | 社区默认低配路线【事实】 | Unet Loader (GGUF) → LoraLoaderModelOnly → ModelSamplingAuraFlow |

**生态成熟度小结**【事实+推断】：Turbo 发布近一年，Civitai 头部 LoRA 全是"写实/质感"类（社区把它当"免费商用的照片机"），动漫/插画类相对薄（Base 才适合训练，但 Base 用户少）【推断】。HF 838 个 LoRA 的体量在 2026 年新模型中排第二（仅次于 Krea 2 的 933，第三方口径）【未核实】。

---

## 4. FLUX.2（dev / klein）

### 4.1 模型事实【事实】

- **FLUX.2 [dev]**：2025-11-25 发布，32B，Rectified Flow Transformer + 内置 Mistral-3 24B 视觉语言模型；多参考最多 10 图；原生 4MP；**原生姿态控制**（不靠外挂 ControlNet）；文本渲染强。BF16 全量 90GB / lowVRAM 模式 64GB；FP8（NVIDIA 合作，显存 -40%、性能 +40%）；GGUF Q4_K_M ~19-20GB（24GB 卡可用）。许可证：FLUX Dev 非商用。
- **FLUX.2 [klein]**：2026-01-15 发布。**4B（Apache 2.0，~13GB，8GB 卡量化可用）** 与 **9B（FLUX 非商用）** 两个蒸馏系；4 步、guidance 3-4。
- ComfyUI Day-0 原生支持；官方 klein 9B 模板覆盖：风格转换/环境改变/物体替换/文字编辑/**照片修复(restore)**/类 ControlNet 姿态（编辑方式实现，非 ControlNet 机制）/inpaint-outpaint。
- 来源：NVIDIA 官方博客 https://blogs.nvidia.cn/blog/rtx-ai-garage-flux-2-comfyui/ ；RunAIHome VRAM 实测 https://www.runaihome.com/blog/flux-2-self-hosting-vram-gguf-guide-2026 ；官方 klein 工作流镜像 https://comfyui.nomadoor.net/zh/basic-workflows/flux-2-klein

> **注意：FLUX 3 Image Klein 已于 2026-10-15 发布**（5.6B；Base 50 步 + 蒸馏 16 步；原生 4K；边界框布局控制；最多 10 参考图；FLUX Kommunity License，年收入 <$5M 免费商用；BF16 12.2GB / FP8 5.6GB；RTX 4070 蒸馏版 1024² 约 8.9s）。ComfyUI Day-1 支持。来源：BFL 官方博客 https://bfl.ai/blog/flux-3-image-klein/ ；T客邦 https://www.techbang.com/posts/134307 。社区首批 FLUX 3 Klein LoRA 已出现（见下表末三行）。用户若要"最新"，FLUX 线应关注它；本节仍以用户点名的 FLUX.2 为主。

### 4.2 LoRA【事实（数字为 Civitai 页面显示口径）】

| 名称 | 类型 | 场景 | 来源 | 热度证据 | ComfyUI 兼容说明 |
|---|---|---|---|---|---|
| **Flux2 Klein_Anything to Real Characters** | 动漫→真人 | 二次元转写实/角色真人化 | https://civitai.red/models/2343188 | **34.3K 下载 / 429 评（Very Positive）**，2026-01-30 发布 | Klein 9B-base；CFG 1、8 步；触发词 "realistic style of a young Asian ..." |
| 胶片模拟系列（Kodak Portra / Fuji Pro 400H / Ilford HP5 等） | 胶片 | 胶片颗粒、halation、色彩科学 | PicassoIA 盘点 https://blog.picassoia.com/best-flux-2-checkpoints-and-models-on-civitai | "下载量最高的 FLUX.2 LoRA 类别之一"【事实（第三方评测口径），单个模型数未核实】 | 建议 0.55 强度；dev 底模 |
| 服装/时尚、建筑室内类 LoRA | 垂直 | 服装褶皱、室内反射光 | 同上 | 高分类热度【推断】 | 最多叠 2 个 LoRA（0.6+0.5），再多画质劣化（该评测经验） |
| Kyoto Animation Style（Klein 9B） | 动漫 | 京阿尼风（紫罗兰永恒花园/声之形 50/25/25 混合数据） | https://civitai.com/models/2530366 | 23 评（Positive）【事实；下载数未核实】 | klein 9B；强度 1 |
| Creepy Cute Magical | 插画 | 诡异可爱魔法风；**多底模包**（Klein 9B、Z-Image Turbo、Krea 2、LTX-2、MiniMax H3 等 9 种） | https://civitai.com/models/1108440 | 13.8K 总下载、34 评【事实】 | 触发词 CreepyCuteMagic |
| BFS Head Swap v1（9B / 4B） | 换脸 | one-node-flux-2-klein 内置 Faceswap 模式所需 | https://huggingface.co/Alissonerdx （BFS 系列） | 被 one-node 工作流指定【事实】 | klein 9B/4B 两版 |
| RefControl v2 Poses（9B） | 姿态 | POSE 模式的姿态 LoRA（配合 DWPose 预处理） | 同上（yanokusnir-ai one-node 说明） | 同上 | 9B 专用 |
| **ControlNet 现状**：FLUX.2 **无官方/主流独立 ControlNet 模型**，姿态/深度控制走"编辑式"或 LoRA；社区工作流「ControlNet depth or openpose flux2klein」用预处理器+采样器组合实现 | 控图 | 姿态/深度复刻 | nomadoor 官方模板说明；Civitai 工作流 https://civitai.com/models/2578291 | 该工作流 1.5K 下载、19 评【事实】 | 需 comfyui_controlnet_aux；v1.2 加 PuLID Flux2 |
| —（FLUX 3 Klein 首批） | 加速/功能 | Muinez/flux3-klein-distil-4step（4 步蒸馏）；NoiseToNova/Flux-3-Klein-turbo-pdd-4_8-steps；ausboss Outpaint / Consistency / Restore；akhaliq angles | HF（freakyjolly 汇总） | 发布仅 3 周，属早期生态【事实：存在；热度未核实】 | FLUX 3 Klein，非 FLUX.2 |

### 4.3 热门 Workflow

| Workflow | 用途 | 自定义节点 | 显存 | 热度证据 |
|---|---|---|---|---|
| 官方 FLUX.2 dev / klein 模板组（ComfyUI 模板库） | T2I、编辑、文字编辑、restore、姿态（编辑式）、inpaint/outpaint | 无第三方 | dev Q4 24GB；klein 9B ~13GB；klein 4B 8GB 可跑 | Day-0 官方【事实】 |
| **one-node-flux-2-klein**（yanokusnir-ai） | 一个节点 7 模式：T2I/I2I/EDIT/PAINT(草图+局部重绘+外扩)/FACESWAP/POSE/SeedVR2 放大 | ComfyUI-Inpaint-CropAndStitch、comfyui_controlnet_aux、BiRefNet（抠底）、SeedVR2 | 随 klein 变体 8-16GB | GitHub 开源，comfy.icu 收录【事实】 https://comfy.icu/extension/yanokusnir-ai__one-node-flux-2-klein |
| ControlNet depth or openpose flux2klein（Civitai） | 姿态/深度精确控制（4 种 OpenPose 输入方式，PuLID Flux2） | controlnet_aux、OpenPose Editor、ultimate-openpose-estimator | klein 9B | 1.5K 下载【事实】 |
| 两遍式（generate → img2img 1.5× denoise 0.4-0.5）放大 | 通用放大 | 内置 | — | BitsMinds 指南推荐 https://www.bitsminds.com/guides/comfyui-stable-diffusion-advanced-guide 【事实（教程口径）】 |

---

## 5. 四个模型 LoRA 生态成熟度对比【事实+推断】

| 维度 | Qwen-Image / Edit | Z-Image | FLUX.2 | FLUX 3 Klein |
|---|---|---|---|---|
| LoRA 数量（HF 口径，第三方统计） | 未获权威总数；Civitai 单人归档已 150 个+，ModelScope/LiblibAI 中文生态极厚 | ~838【未核实】 | 未获总数；Civitai klein 9B/4B 分类活跃 | 刚起步（几十个量级）【推断】 |
| 头部 LoRA 体量 | Lenovo UltraReal 161.7K；fal 多角度 LiblibAI 126.8K | Realistic Snapshot 215.9K | Anything2Real 34.3K | 暂无头部 |
| 编辑类 LoRA | **最丰富**（官方 2511 专门"集成 LoRA 能力"，fal/PAI/DiffSynth/lightx2v 都在出） | 无（Edit 模型未发布） | 少（dev/klein 自带编辑能力，LoRA 需求被模型原生能力吃掉）【推断】 | 出现 outpaint/restore/consistency 等功能型 |
| ControlNet | InstantX Union + Inpainting + PAI 2.1 Fun Union（8+1 条件，最全） | PAI Fun Union（5 条件）+ Tile 放大 | 无独立 ControlNet（原生姿态/编辑替代） | 暂无 |
| 中文生态（LiblibAI/ModelScope） | 极强（fal 多角度 126.8K 下载在 Liblib） | 强（Comfy-Org 转包 39 万+） | 弱 | 弱 |
| 结论 | **LoRA 生态最成熟**，编辑类一家独大 | 增长最快的"写实"生态，风格类偏薄 | klein 系健康但多被原生能力替代；Civitai 主力 | 刚起步，先出加速/功能型 LoRA |

（以上均为 2026-11-08 检索时点判断；FLUX 3 Klein 增速需观察。）

---

## 6. 按场景推荐汇总【事实+推断】

| 场景 | 首选模型/LoRA/Workflow | 备选 | 显存参考 | 备注 |
|---|---|---|---|---|
| **电商产品图（白底/换背景）** | Qwen-Image-Edit-2511 + Lightning-4steps；官方模板或 oncompute 电商五件套（White-to-Scene 思路：提示词锁死"保持产品形状/边缘/标签不变"） | Z-Image Turbo（Apache-2.0 可商用，白底+文字） | 2511 fp8 20.5GB / GGUF Q4 13.2GB；Z-Image 8GB(fp8) | 2.1 还能直接出 RGBA 透明底商品贴图【事实】 |
| **海报 / 中文文字排版** | **Qwen-Image-2.0 / 2.1**（1K token 长指令、PPT/信息图/多格漫画、汉字数百字渲染）；Logo.Redmond 辅助 | Z-Image Turbo（中英双语文字）；FLUX.2（多语言 UI 文字，9 成准确率为官方口径） | 2.1 原生 2K，建议 16GB+ | 科技日报报道 2.0 可渲染《兰亭集序》数百字【事实】 |
| **人像写真 / 写实摄影** | Z-Image Turbo + Realistic Snapshot（0.6-0.7）；Qwen 线用 Lenovo UltraReal / NiceGirls UltraReal / Amateur Photography | FLUX.2 dev + 胶片 LoRA（Portra 400 等） | Z-Image 8-16GB；FLUX.2 dev 24GB(Q4) | Z-Image 是"开源写实性价比之王"（多来源共识）【推断】 |
| **动漫插画** | Qwen 线：LineAniRedmond / Dark Ghibli / Flat Color / Anime Redmond；Flux.2 klein 9B：Kyoto Animation Style | Krea 2（生态最快，本次不展开） | Qwen fp8 20GB；klein 9B ~13GB | Z-Image 动漫类偏薄，建议 Base 训练或用多底模包【推断】 |
| **多视角 / 角色设定图** | Qwen-Image-Edit-2511 + fal Multiple-Angles LoRA（96 机位）+ jtydhr88 节点；tarn59 turnaround / DiffSynth F2P | Z-Image Character Design Sheet (3 Perspectives)；FLUX.2 原生 10 参考图多视角 | Edit 2511 12-16GB(GGUF) | fal LoRA 是 2511 首个多角度 LoRA【事实】 |
| **换装 try-on / 多图融合** | Qwen-Image-Edit-2509/2511 多图 + Clothes Try On（Civitai 8.1K）/ FoxBaze TryOn / dx8152 Fusion | FLUX.2 原生多参考（无需 LoRA） | 16GB+ | 2511 多图建议 1-3 张参考图效果最佳（社区经验）【推断】 |
| **老照片修复 + 上色** | Qwen-Image-Edit-2511（2511 专门强化人脸保持）+ Lightning；LiblibAI 修复工作流（40 步 CFG 2.0-2.5）或 Floyo 2511 restore | FLUX.2 klein 官方 restore 模板；ausboss Restore LoRA（FLUX 3 Klein）；Qwen-Image-Lightning-8steps + 老照片风格 LoRA（生成式复古） | 修复 12-16GB；+SeedVR2 7B 放大 24GB | 完整流水线：Qwen 修复 → SeedVR2(3B 8-16GB / 7B 24GB) 放大 → PS 精修【事实】 |
| **局部重绘 / 去水印 / 去路人** | Qwen-Image-Edit-2511 + prithivMLmods Object-Remover 或 valiantcat Remover-General；或 2.1 Fun Union 的 inpaint | InstantX Inpainting ControlNet（2508 底模）；Z-Image-Edit（未发布，暂无） | 同上 | remover 类 LoRA 优势是"除目标外零改动"【事实】 |
| **精确控图（姿态/深度/线稿）** | Qwen-Image-2.1-Fun-Controlnet-Union（8 条件+inpaint）；InstantX Union（老底模） | Z-Image Fun Union 2.1（8 步蒸馏 + Tile 放大）；FLUX.2 原生姿态 + RefControl Poses | ControlNet 分支 +2-8GB | Qwen 2.1 Union 一文件覆盖最多条件【事实】 |
| **极速出图 / 低显存** | Z-Image Turbo（8 步，6-8GB GGUF）；Qwen Lightning 4 步；klein 4B（4 步） | FLUX 3 Klein 蒸馏（16 步，FP8 5.6GB） | 见前 | Turbo CFG 锁 1 无负向词，出图多样性低是已知代价【事实】 |
| **高清放大（收尾必备）** | SeedVR2（numz）：3B fp16 12-16GB / fp8 8-12GB / Q4 GGUF 6-8GB；7B fp16 24GB+，batch 4n+1 | Z-Image Tile-2.1-8steps；Remacri 传统放大 | 6-24GB | Qwen/Z-Image 官方与社区工作流的标准收尾件【事实】 |

---

## 7. 明确没能核实 / 存疑清单

1. Z-Image Turbo 在 HF 的 LoRA 数量（838）来自第三方博客统计，未在 HF 原始数据复核。【未核实】
2. Civitai 页面数字（Realistic Snapshot 215.9K 等）来自页面渲染文本，存在解析误差可能；点赞/收藏列无法精确区分。【部分未核实】
3. dx8152 系列 LoRA（ModelScope）的单模型下载计数未逐一抓取。【未核实】
4. Phr00t Rapid-AIO"6GB 显存可跑"仅见于推广性文章，未验证。【未核实】
5. Qwen-Image-3.0 是否已放开放权重（官方 2026-07-21 宣布，但 9 月又发布 2.1，两条线的开源状态未见权威对照）。【未核实】
6. Z-Image-Edit 的 CSDN/镜像站"实测教程"（steps=20, cfg=7.5 等）与官方"待发布"状态矛盾，判定为低质/预告性内容，不采信。【推断】
7. clore.ai 文档称 klein 为"32B Apache"与多数来源（4B/9B）冲突，采信 BFL 官方口径（4B Apache / 9B 非商用）。【推断（以官方为准）】
8. PanelPainter V2 的原始仓库链接未抓到权威页。【未核实】

## 8. 主要来源清单

- ComfyUI 官方文档/模板：docs.comfy.org（Qwen 2512、Edit 2511、FLUX.2）；comfyanonymous Examples（z_image）
- ComfyUI Wiki：qwen-image-2.1（2026-09-21）、2.1 Fun ControlNet Union（2026-09-24）
- Hugging Face：lightx2v（Lightning 全系）、fal、tori29umai、dx8152、tarn59、valiantcat、prithivMLmods、lrzjason、Alissonerdx、InstantX、Comfy-Org、alibaba-pai、Phr00t、numz（SeedVR2）、black-forest-labs、unsloth（GGUF）
- Civitai：Realistic Snapshot、Character Design Sheet、Anything to Real Characters、Kyoto Animation、Consistence Edit、Clothes Try On、Z Image V7 workflow、flux2klein ControlNet workflow
- LiblibAI：多角度相机控制（126.8K）、多角度+重绘+洗图工作流、老照片修复工作流
- ModelScope：Comfy-Org/z_image_turbo（393,786 下载）、PAI ControlNet
- 官方博客/媒体：NVIDIA RTX Blog（FLUX.2）、BFL Blog（FLUX 3 Image Klein）、科技日报（Qwen-Image-2.0）、T客邦（FLUX 3 Klein 性能）
- 社区评测：Zenn Awesome-Qwen-Image-Edit、RunAIHome、BestFreeWebResources、localaimaster、nextdiffusion、oncompute docs、RunComfy、Floyo、yalewoo、freakyjolly、spicyapi（Z-Image LoRA 强度实测）
