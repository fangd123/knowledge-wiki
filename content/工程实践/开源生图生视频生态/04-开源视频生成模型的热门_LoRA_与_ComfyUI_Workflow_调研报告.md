---
title: "开源视频生成模型的热门 LoRA 与 ComfyUI Workflow 调研报告"
date: 2026-11-08
author: fangd123
license: CC BY-NC-SA 4.0
category: 工程实践
series: 开源生图生视频生态
series_order: 4
tags:
  - 工程实践
  - 开源生图生视频生态
ai_assisted: true
status: published
---

# 开源视频生成模型的热门 LoRA 与 ComfyUI Workflow 调研报告

**调研日期：2026-11-08（全部基于当日联网搜索）**
**标注约定**：〔事实〕= 搜索来源直接确认；〔推断〕= 基于多来源的合理推断；〔未核实〕= 未能找到可靠数字或链接。

---

## 0. 先说结论：截至 2026-11 的开源视频模型格局

〔事实〕Alibaba 的开源权重止步于 **Wan 2.2**（2025-07-28 发布，Apache 2.0）。Wan 2.5（2025-09-24）、Wan 2.6（2025-12-16）、Wan 2.7（2026-04）均为 API 闭源，无公开权重。Wan 3.0（2026-08）同样 API-only。但 Wan 2.2 世代仍在更新开源模型：**Wan-Animate-2 / Animate-2-Lite（2026-08-07，Apache 2.0）** 是最新开源的 Wan 系模型。
来源：https://modeltrends.ai/chinese/%E6%8C%87%E5%8D%97/which-wan-models-are-open-source 、https://opensourceforu.com/2026/08/alibaba-open-sources-wan-animate-2

〔事实〕2026 年的大变化是"**音视频一体生成**"成为开源标配：LTX-2（2026-01-06 开源）、LTX-2.3（2026-03-05）、LTX-2.5（2026-08-11）、MiniMax H3（2026-08-03 开源权重）都原生输出同步音频。
来源：https://singaporeoutlook.com/lightricks-open-sources-ltx-2-the-first-production-ready-audio-and-video-generation-model-with-truly-open-weights-33973/ 、https://vdf.ai/blog/minimax-h3-local-comfyui

---

## 1. 开源视频模型全景表

| 模型 | 发布时间 | 参数量 | 许可证 | 原生音频 | ComfyUI 支持 | 实用显存 | 备注 |
|---|---|---|---|---|---|---|---|
| **Wan 2.1**（T2V/I2V 1.3B、14B、FLF2V） | 2025-02 | 1.3B / 14B | Apache 2.0 | 否 | 原生 + Kijai Wrapper | 1.3B≈6GB；14B fp8≈24GB | 社区 LoRA 生态的最大底座〔事实〕 |
| **Wan 2.2 T2V/I2V-A14B** | 2025-07-28 | 27B MoE（每步激活 14B，高噪+低噪双专家） | Apache 2.0 | 否 | Day-0 原生模板 | fp8 24GB；GGUF 12-16GB | 官方脚本需 80GB，ComfyUI offload 后 24GB 可跑〔事实〕 |
| **Wan 2.2 TI2V-5B** | 2025-07-28 | 5B（高压缩 VAE 4x32x32） | Apache 2.0 | 否 | 原生模板 | 官方称 8GB 可跑；Q8 GGUF 5.4GB | 720p@24fps，1280×704〔事实〕 |
| **Wan 2.2 Fun Camera / Fun InP / Fun Control**（PAI 团队） | 2025-07~ | 14B×2（高/低噪） | Apache 2.0 | 否 | 官方模板 | fp8≈20GB（4090 占用 84%） | 相机控制 / 首尾帧 / 姿态深度轨迹控制〔事实〕 |
| **Wan 2.2 VACE（VACE module）** | 2.1 版 2025-05；2.2 VACE 随后 | 1.3B / 14B | Apache 2.0 | 否 | Kijai Wrapper | 14B 需 24GB 级 | 通用视频编辑控制框架〔事实〕 |
| **Wan 2.2 S2V-14B** | 2025-08-26 | 14B | Apache 2.0 | 音频驱动（wav2vec2） | 官方原生模板 | fp8 24GB 级 | 音频驱动数字人〔事实〕 |
| **Wan 2.2 Animate-14B** | 2025-09-19 | 14B | Apache 2.0 | 否 | 官方模板 + Wrapper | fp8 16GB 起步，24GB 舒适 | 动作迁移/角色替换（Move/Mix 双模式）〔事实〕 |
| **Wan-Animate-2 / Lite** | 2026-08-07 | 14B（Base 40 步 / 蒸馏 10 步无 CFG） | Apache 2.0 | 否 | 原生节点 WanAnimate2ToVideo / WanAnimate2Cache | Lite 实时 24fps@400×720 | 免动作提取器、文本驱动视角控制〔事实〕 |
| **HunyuanVideo 1.5**（腾讯） | 2025-11-21 | 8.3B | 腾讯社区许可证（EU/UK/KR 除外，>1 亿 MAU 需授权） | 否 | 原生模板 | fp8 约 14GB 起步，24GB 舒适 | 720p 可超分 1080p；SSTA 注意力；文字渲染〔事实〕 |
| **HunyuanVideo-Avatar** | 2025-05-28 | 14B 级 | 腾讯社区许可证 | 音频驱动 | 官方 ComfyUI 节点 + Wan2GP | TeaCache+offload 最低 10GB | 多角色独立音轨、情绪迁移〔事实〕 |
| **LTX-2 / LTX-2.3 / LTX-2.5**（Lightricks） | 2026-01-06 / 03-05 / 08-11 | 19B（14B 视频+5B 音频）/ 22B / 22B | LTX-2.x 社区许可（<$10M 营收免费） | **是**（单 pass 同步音频） | Day-0/Day-1 原生模板 | 2.3 FP8 16GB；2.5 INT8 21.5GB（24GB 卡实测可跑） | 4K@50fps、多镜头、20s〔事实〕 |
| **MiniMax H3**（FL2VA / Ref2VA） | 2026-07-31 发布，08-03 开源 | 33B | MiniMax 社区许可（**排除 US/EU/UK/KR**） | **是**（立体声） | ComfyUI 0.30.0 Day-0 模板 | 剪枝 int8 约 12GB 可跑 480p | 768p 短边开源；2K 走 API；Ref2VA 支持 9 图+3 视频+3 音频参考〔事实〕 |
| **SkyReels-V2**（昆仑万维） | 2025-04 | 1.3B / 14B | SkyReels 社区许可 | 否 | Kijai Wrapper（DF 扩散强制采样器） | 1.3B 540p≈8GB；14B 720p 24GB | Diffusion Forcing 无限长视频〔事实〕 |
| **LongCat-Video**（美团） | 2025-末 | 13.6B | **MIT** | 否 | 社区 | 高端消费卡 | 分钟级 720p@30fps〔事实，来源 modeltrends〕 |
| **FramePack**（lllyasviel） | 2025-04 | ≈13B | 开源权重 | 否 | 独立/ComfyUI | **6GB** | 图生视频最长 60s，低显存代表〔事实〕 |
| **Mochi 1**（Genmo） | 2024-10 | 10B | Apache 2.0 | 否 | 原生 | ≈20GB | 无重大后续版本；480p 上限已显落后〔事实〕 |
| **CogVideoX 2B/5B**（智谱） | 2024 | 2B / 5B | 2B Apache 2.0；5B 限制许可 | 否 | 原生 | 5B INT8≈7GB | 提示词遵循好，质量已被反超〔事实〕 |
| Seedance 2.5（字节）/ Kling / Veo / Sora / Wan 2.5-3.0 | — | — | **闭源 API** | — | 仅经 Comfy 合作节点走 API | — | 无权重可下载〔事实〕 |

〔未核实〕Kandinsky Video（Sber）截至 2026-11 的开源状态与 ComfyUI 支持未能核实。

---

## 2. 加速 / 蒸馏 LoRA 一览

| 名称 | 类型 | 适配基模 | 场景 | 来源链接 | 热度证据 | ComfyUI 兼容说明与常见坑 |
|---|---|---|---|---|---|---|
| **LightX2V 4-step（Wan 2.2 Lightning，v1.1 T2V / v1 I2V）** | 步数+CFG 蒸馏 LoRA（高噪/低噪各一只） | Wan 2.2 T2V/I2V-A14B | 所有 Wan 2.2 工作流提速（20 步→4 步，4090 上 9 分钟→约 2 分钟） | https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged ；https://runpod.io/articles/guides/comfyui-wan-2-2 | ComfyUI 官方模板内置默认加速路径〔事实〕 | 必须 CFG=1、shift≈5；高低噪 LoRA 必须成对加载到对应专家，错配会糊；**运动幅度与细节变小**，出片建议关 LoRA 用 20 步；无 TI2V-5B / S2V 专用 4 步版〔事实〕 |
| **LightX2V（Wan 2.1）cfg+step distill rank32/rank64** | 步数蒸馏 LoRA | Wan 2.1 T2V/I2V 14B | 2.1 时代通用加速，Animate/S2V 模板至今用它 | https://huggingface.co/Kijai/WanVideo_comfy/blob/main/Wan21_T2V_14B_lightx2v_cfg_step_distill_lora_rank32.safetensors | 官方 Animate 模板钦定 `lightx2v_I2V_14B_480p_cfg_step_distill_rank64_bf16`〔事实〕 | CFG=1；2.1 LoRA 只能挂在 2.2 **低噪**专家上，高噪专家权重分布不兼容〔事实〕 |
| **CausVid LoRA**（Kijai 提取版） | 自回归+DMD 蒸馏 | Wan 2.1 T2V 14B（可配 VACE） | 4-12 步加速；VACE 控制流提速 | https://causvid.github.io/ ；https://huggingface.co/Kijai/WanVideo_comfy/blob/main/Wan21_CausVid_14B_T2V_lora_rank32.safetensors | Wan2GP/Wan 社区默认加速件〔事实〕 | 12 步乘子 0.3 / 8 步 0.5-0.7 / 4 步 0.8-1.0；CFG=1、shift≈7；步数越低运动越差；对 2.2 只在低噪阶段有效〔事实〕 |
| **AccVid LoRA** | CFG 蒸馏（不减步） | Wan 2.1 T2V/I2V 14B | 保持步数、免去 CFG（约 2x 提速） | https://huggingface.co/Kijai/WanVideo_comfy/blob/main/Wan21_AccVid_T2V_14B_lora_rank32_fp16.safetensors | Wan2GP 文档收录〔事实〕 | CFG=1、shift=5；步数不变〔事实〕 |
| **rCM LoRA**（Kijai 提取自 NVIDIA 关联团队的 Wan 2.1 T2V 蒸馏） | 一致性模型蒸馏 | Wan 2.1（可能 2.2）T2V | 4 步出片，与 Lightning 叠用（高噪 rCM 4.0 + 低噪 Lightning 1.0） | https://wanx-troopers.github.io/loras/part-02.html | 社区口碑"高噪 4.0 shift 7 可用"〔事实，社区经验〕 | **不推荐 I2V**（低噪改画面过多、易出 artifact）；官方不做 I2V 版〔事实〕 |
| **FusionX LoRA / 融合模型**（vrgamedevgirl） | 多 LoRA 合并（CausVid+AccVideo+MoviiGen+MPS reward+自训） | Wan 2.1 T2V/I2V 14B、SkyReels | 8 步出片 + 电影感运动/质感；有 Phantom 专用变体 | https://civitai.com/models/1678575/wan21fusionx-the-lora ；https://civitai.green/models/1651125 | LoRA 版快照下载 3 万+、融合模型版 3.9 万+（口径不一）〔事实，数值为快照〕 | 8 步、可低至 3 步拉强度；**作者声明合并来源复杂，商用需自查法律**〔事实〕 |
| **MiniMax H3 Turbo 4-step / 8-step**（lightx2v 出品） | 蒸馏 LoRA | MiniMax H3 FL2VA/Ref2VA | H3 提速 | https://www.modelscope.ai/models/Comfy-Org/MiniMax-H3 | Comfy-Org 官方打包内含〔事实〕 | ComfyUI loras 目录直用〔事实〕 |
| **HunyuanVideo 1.5 cfg-distilled fp8** | 蒸馏模型（非 LoRA） | HunyuanVideo 1.5 | CFG=1 直出，8.3GB 文件 | https://github.com/AlphafromZion/comfyui-workflows/tree/main/workflows/hunyuan-video-15-i2v | 官方发布变体〔事实〕 | CFG 必须 1.0〔事实〕 |
| **LTX 13B/2B Distilled** | 蒸馏模型 | LTX-2/2.3/2.5 | 4-8 步、CFG 1 | https://www.thundercompute.com/blog/ltx-2-3-comfyui | 官方变体〔事实〕 | 2.5 蒸馏版 8 步 CFG1〔事实〕 |
| **TeaCache / EasyCache / torch.compile / block swap** | 免训练加速 | Wan/Hunyuan 系 | 无损或近无损提速 | Kijai Wrapper 内置 | Wrapper 6.8k stars 生态标配〔事实〕 | block swap + 32GB+ 内存是 24GB 卡跑 14B 的关键〔推断〕 |

**加速 LoRA 的共同坑（〔事实〕，多来源一致）**：
1. CFG 必须设 1.0（CFG 蒸馏模型在 CFG>1 会双重施加引导产生伪影）；
2. Wan 2.2 是双专家：LoRA 必须高噪配高噪、低噪配低噪，2.1 LoRA 只能进低噪阶段；
3. 蒸馏后**运动幅度/动态普遍变小**，草稿用加速、终稿用 20+ 步原始路径；
4. T2V 与 I2V 蒸馏 LoRA 不通用（v1.1 vs v1）。

---

## 3. 运动 / 镜头控制类

| 名称 | 类型 | 适配基模 | 场景 | 来源链接 | 热度证据 | ComfyUI 兼容说明 |
|---|---|---|---|---|---|---|
| **Wan2.2-Fun-Camera-Control**（PAI） | 相机控制码模型 | Wan 2.2 14B（高/低噪专用权重） | 平移/推拉/旋转等预设运镜，I2V | https://docs.comfy.org/tutorials/video/wan/wan2-2-fun-camera | ComfyUI 官方模板"Fun Camera"〔事实〕 | 官方模板含 Lightning 加速路径；4090 24GB fp8 640×640 占用 84%、536s/713s→LoRA 后 71s〔事实〕 |
| **Wan2.2 Fun InP** | 首尾帧模型 | Wan 2.2 14B | 首帧+尾帧插值生成 | https://docs.comfy.org/zh-CN/tutorials/video/wan/wan2-2-fun-inp | 官方模板"Fun Inp"〔事实〕 | 4090：524s→LoRA 后 79s〔事实〕 |
| **Wan 2.2 FLF2V（原生 First-Last-Frame）** | 条件节点 WanFirstLastFrameToVideo | Wan 2.2 14B | 首尾帧可控转场 | https://comfy.org/workflows/video_wan2_2_14B_flf2v/ | 官方模板〔事实〕 | 纯原生节点，可选 Lightning LoRA〔事实〕 |
| **VACE 1.3B / 14B（2.1）+ Wan2.2 VACE module** | 全能视频编辑控制 | Wan 2.1/2.2 14B | 姿态/深度/参考图/局部重绘/续写（Move-Anything 系） | https://nathanshipley.github.io/banodoco-kb/kb/wan/index.html ；http://www.runcomfy.com/comfyui-workflows/wan-2-2-vace-in-comfyui-pose-driven-motion-video-workflow | 社区评价"Fun VACE 2.2 全面优于 2.1"〔事实〕 | 主要走 Kijai Wrapper（WanVideoVACEEncode + 双采样器）；**掩码必须连通**，强度 0.4-1.2〔事实〕 |
| **Fun Control（Canny/Depth/Pose/MLSD/Trajectory）** | 控制信号模型族 | Wan 2.1/2.2（PAI VideoX-Fun） | 骨骼/深度/线条/轨迹控制 | https://nathanshipley.github.io/banodoco-kb/kb/wan/index.html | VideoX-Fun 官方仓库〔事实〕 | Wrapper 支持；DWPose/ViTPose 预处理〔事实〕 |
| **ReCamMaster**（快手） | 相机轨迹重渲染 | Wan 2.1 | 10 条预设轨迹，单视频换运镜 | 同上 banodoco KB | ICCV 2025 最佳论文入围〔事实〕 | Wrapper 内置节点；需 81+ 帧〔事实〕 |
| **ATI（Any-Thing-Is-Trajectory）** | 运动轨迹控制 | Wan 2.1 | 物体/人物沿轨迹移动 | 同上 | 社区口碑"终于自然的轨迹控制"〔事实〕 | Wrapper 有 WanVideo ATI 节点〔事实〕 |
| **Wan 2.2 Animate / Wan-Animate-2** | 角色动画+替换 | Wan 2.2 Animate-14B / Animate-2 | 视频换人、动作/表情迁移、长视频迭代 | https://docs.comfy.org/tutorials/video/wan/wan2-2-animate ；https://blog.comfy.org/p/wan-animate-2-is-now-available-in | 官方双模板；Animate-2 新增文本视角控制、实时 Lite〔事实〕 | 官方模板需 KJNodes + controlnet_aux（DWPose）+ SAM2；Animate-2 免预处理、双分支 DiT 直接吃驱动视频〔事实〕 |
| **Phantom 1.3B/14B**（字节，ICCV 2025） | 主体一致性参考注入 | Wan 2.1 | 1-4 张参考图保持人物/商品跨镜头一致（电商换主体） | https://octohz.com/p/2104 ；banodoco KB | Apache 2.0，电商场景热门〔事实〕 | Wrapper 已适配；双参考图会"融合"两张脸，注意〔事实〕 |
| **Stand-In / EchoShot / MAGREF / Lynx** | 主体一致性/多镜头/多参考/人脸 ID | Wan 2.1 | 跨镜头同一人物、多镜头分镜、FaceID | banodoco KB + Wrapper 节点清单（comfy.org/cloud/supported-nodes/ComfyUI-WanVideoWrapper/） | 均已进 Kijai Wrapper〔事实〕 | Lynx 参考强度 0.6 合适、>1.0 会"脸贴上去"；Lynx 会把帧率改到 24fps〔事实〕 |
| **LTX IC-LoRA（pose/depth/canny）+ ID-LoRA** | 控制与个性化 | LTX-2.3 | 姿态深度边缘控制、单人个性化音视频 | https://www.thundercompute.com/blog/ltx-2-3-comfyui | 官方发布〔事实〕 | 原生模板 6 种含 IC-LoRA Union、ID-LoRA〔事实〕 |
| **MiniMax H3 Fun-Controlnet-Union**（alibaba-pai） | 控制 LoRA/module | MiniMax H3 | H3 的控制版 | https://www.modelscope.ai/models/Comfy-Org/MiniMax-H3 | Comfy-Org 打包 model_patches 提供〔事实〕 | 随官方 repackage 分发〔事实〕 |
| **Civitai 相机运动 LoRA 系列**（Remade 等） | 效果 LoRA | Wan 2.1 I2V 14B 480p | 360 环绕运镜、摇臂下摇、追车等 | https://modeltrends.ai/portuguese/modelos-ia/wan/360-orbit-camera-motion-wan2.1-i2v-lora-2dc19315 ；https://civitaiarchives.com/models/1620160 | 360 Orbit Camera Motion 下载 4,451；Crane Down 1,839（快照）〔事实〕；Remade 自称 100+ 相机/VFX LoRA〔事实〕 | 走 Kijai 480p I2V 示例工作流，强度 1.0、guidance 6.0、flow shift 5〔事实〕 |

---

## 4. 特效 / 风格类 LoRA（Civitai 高热度）

特效 LoRA 生态几乎全部长在 **Wan 2.1 I2V 14B 480p** 底模上〔事实〕。

| 名称 | 类型 | 适配基模 | 场景 | 来源链接 | 热度证据（快照） | ComfyUI 兼容说明 |
|---|---|---|---|---|---|---|
| **360 Degree Rotation（微波炉旋转）**（Remade） | I2V 特效 | Wan 2.1 I2V 14B 480p | 主体 360° 旋转 | https://civitai.com/models/1346623/360-degree-rotation-microwave-rotation-wan21-i2v-lora | civitai.green 快照：约 2.1 万下载 / 1.2k 赞；HF 镜像旧快照 1.2 万下载〔事实，口径随时间增长〕 | 触发词 `r0t4tion 360 degrees rotation`；常见问题：只转 270°，加帧无解〔事实〕 |
| **Squish Effect（挤压）**（Remade） | I2V 特效 | 同上 | 挤压变形（Civitai 走红特效） | https://civitai.com/models/1340141/squish-effect-wan21-i2v-lora | 约 7.6k 下载 / 439 赞〔事实〕 | 触发词 `sq41sh squish effect`〔事实〕 |
| **Inflate Effect（膨胀）**（Remade） | I2V 特效 | 同上 | 膨胀充气 | https://civitai.com/models/1346480 | 约 1.6k 下载〔事实〕 | 触发词 `infl4t3 inflates it`〔事实〕 |
| **360 Camera Orbit**（samusynth） | 相机特效 | Wan 2.1 I2V 14B 480p | 相机 360° 环绕 | https://modeltrends.ai/ai-models/wan/360-camera-orbit-wan-2.1-i2v-lora-20fd175f | 2,844 下载 / 158 赞（2026-10 快照）〔事实〕 | 低分辨率训练，复杂图不稳〔事实〕 |
| **VIP / Assassin Transformation（变身）**（Remade） | I2V 特效 | 同上 | 红毯变身/史诗变身 | https://civitai.com/models/1367835/vip-transformation-wan21-i2v-lora | VIP 版 546 下载〔事实〕 | 触发词 `v1p red carpet transformation`〔事实〕 |
| **Crane Down / Car Chase**（Remade） | 运镜/场景 | 同上 | 摇臂下摇、追车戏 | https://civitaiarchives.com/models/1620160 、https://civitaiarchive.com/models/1620122 | 1,839 / 592 下载〔事实〕 | 同 Remade 通用工作流〔事实〕 |
| **FusionX（T2V/I2V/Phantom 三版本）** | 风格+运动融合 | Wan 2.1 14B | 电影感通用增强 | https://civitai.com/models/1678575/wan21fusionx-the-lora | LoRA 版 3 万+ 下载〔事实〕 | 2.2 中只挂低噪〔事实〕 |
| **MiniMax H3 内置特效 embeddings**（art_is_explosion、bullet_time、dark_magic、blooming_flowers 等） | 特效 embedding | MiniMax H3 | 爆炸/子弹时间/黑魔法/花开等一键特效 | https://www.modelscope.ai/models/Comfy-Org/MiniMax-H3 | Comfy-Org 官方打包内含〔事实〕 | models/embeddings 目录〔事实〕 |
| 风格类（动漫/写实/水墨等） | 风格 LoRA | Wan 2.1/2.2 | 风格化视频 | 〔未核实〕本次未能核实具体条目的下载量与稳定链接 | 〔未核实〕 | 〔推断〕Civitai 与 LiblibAI 上以 Wan 为底模的视频风格 LoRA 数量最多，可用站内按 Base Model=Wan Video 筛选 |

---

## 5. 数字人 / 口型对齐方案

| 名称 | 类型 | 适配基模 | 场景 | 来源链接 | 热度证据 | ComfyUI 兼容与显存 |
|---|---|---|---|---|---|---|
| **Wan2.2-S2V-14B** | 音频驱动视频（官方） | Wan 2.2 S2V | 图片+音频→口播/演唱视频，77 帧/段可无限续 | https://docs.comfy.org/tutorials/video/wan/wan2-2-s2v | ComfyUI 官方原生模板〔事实〕 | wav2vec2 音频编码器；Lightning LoRA 可提速但"动态与质量显著损失"，官方建议质量敏感时用 20 步 CFG6；fp8 24GB 级〔事实〕 |
| **InfiniteTalk**（MultiTalk 团队） | 音频驱动，稀疏帧 | Wan 2.1 I2V 480/720p（Single/Multi 两版） | **无限长**口播数字人（81 帧/chunk、25 帧重叠），支持多人对话 | https://www.infinitetalk.net/infinitetalk-comfyui ；https://aistudynow.com/infinitytalk-comfyui-workflow-wan-2-1-img2vid-vid2vid-multi-talk | Kijai Wrapper 内置（Multi/InfiniteTalk Model Loader），社区长口播首选〔事实〕 | 实测 720×960 约 13-16GB 显存、42s 音频约 20 分钟；480p 推荐 6GB+；720p 早期不稳〔事实〕 |
| **MultiTalk** | 音频驱动多人 | Wan 2.1 I2V 14B 480p | 多人对话/合唱，最长 15s（201 帧） | banodoco KB；Wrapper 节点 | 已被 InfiniteTalk 迭代取代〔推断〕 | 4090 级起步，8GB 可勉强〔事实〕 |
| **FantasyTalking**（阿里） | 音频驱动+表情强度控制 | Wan 2.1 I2V 720p 14B + 1.6GB 控制模型 | 高表现力口播（微表情、肢体） | https://learn.thinkdiffusion.com/fantasytalking-where-every-images-tells-a-moving-story/ ；https://github.com/Fantasy-AMAP/fantasy-talking | ThinkDiffusion 教程，需 24GB 档起步、推荐 48GB〔事实〕 | Wrapper 节点 FantasyTalkingWav2VecEmbeds；wav2vec2-base-960h；FPS 与音频节奏要匹配〔事实〕 |
| **HunyuanVideo-Avatar**（腾讯） | 音频驱动多角色 | HunyuanVideo 14B 级 | 电商口播、虚拟主播、多人独立音轨、情绪迁移 | https://github.com/Tencent-Hunyuan/HunyuanVideo-Avatar | 官方 ComfyUI 支持；Wan2GP 方案 10GB 显存可跑〔事实〕 | 官方最低 24GB（704×768×129f，很慢）；推荐 96GB；FP8+offload 实用〔事实〕 |
| **HuMo 1.7B / 17B**（字节，2025-09） | 文本+音频/文本+图+音频 | 独立模型（字节） | 强提示词遵循的说话视频，静音时闭嘴 | banodoco KB | Wrapper 内置 HuMo Embeds〔事实〕 | 1.7B 需 32GB、480p 约 8 分钟；17B 3090 经 Wrapper 可跑；SageAttention 会伤口型〔事实〕 |
| **EchoShot** | 多镜头一致性 | Wan 2.1 T2V 1.3B | 同一人物多镜头分镜 | banodoco KB | Wrapper 内置〔事实〕 | 1.3B 轻量〔事实〕 |
| **LTX-2.3 IA2V / ID-LoRA** | 图+音频→视频 / 个性化 | LTX-2.3 | 口型同步数字人、单文件音视频 | https://www.thundercompute.com/blog/ltx-2-3-comfyui | 官方六模板之二〔事实〕 | 原生模板〔事实〕 |
| **LatentSync 等** | 后期口型修正 | 独立 | 对已有视频改口型 | （社区常见，本次未深挖）〔未核实〕 | — | — |

---

## 6. 热门 ComfyUI Workflow 汇总

### 6.1 ComfyUI 官方模板（Templates → Video，内置无需第三方节点，除非注明）

| 工作流 | 用途 | 自定义节点 | 显存参考 |
|---|---|---|---|
| Wan 2.2 5B（TI2V） | 文/图生视频 720p@24fps | 无 | 官方：8GB + offload 可跑〔事实〕 |
| Wan 2.2 14B T2V / I2V | 双专家文生/图生视频，内置 Lightning 4 步开关（subgraph） | 无 | 4090 fp8：20 步约 9 分钟 / 4 步约 2 分钟（640×640×81f）〔事实〕 |
| Wan 2.2 14B FLF2V | 首尾帧 | 无 | 24GB 级〔事实〕 |
| Wan 2.2 Fun Camera / Fun InP | 运镜 / 首尾帧 | 无 | 24GB，fp8 占 83-89%〔事实〕 |
| Wan 2.2 S2V | 口播数字人 | 无（音频编码器单独下载） | fp8 24GB 级；每 77 帧 ≈4.8s〔事实〕 |
| Wan 2.2 Animate（Move/Mix） | 动作迁移/角色替换 | KJNodes、controlnet_aux、SAM2（Mix 模式） | fp8 16GB 起步、24GB 舒适；512×768 约 6-9 分钟/5s〔事实〕 |
| Wan Animate 2 | 新一代角色动画（原生 WanAnimate2ToVideo/Cache 节点） | 无（新核心节点） | Cache 节点可省一半时间；Lite 实时〔事实〕 |
| HunyuanVideo 1.5（T2V/I2V） | 8.3B 高质量 | 无（可加 latent upsampler 1080p） | fp8 14GB 起步；32GB 才能上 720p×193 帧〔事实〕 |
| LTX-2 / 2.3 / 2.5 六模板 | T2V/I2V/FLF2V/IA2V/IC-LoRA/ID-LoRA | 无 | 2.3 FP8 16GB；2.5 INT8 24GB〔事实〕 |
| MiniMax H3 | 音视频一体（FL2VA/Ref2VA） | 无 | 剪枝 int8 12GB 可跑 480p；16-24GB 舒适〔事实〕 |
| SeedVR2 视频修复 | 时序一致超分 | SeedVR2 节点包 | 3B GGUF Q4 8GB 可跑（见 6.3）〔事实〕 |

### 6.2 Kijai ComfyUI-WanVideoWrapper 示例工作流（example_workflows/，社区最活跃）

Wrapper 本体：**6,804 GitHub stars、Comfy 注册表下载 203 万+、版本 1.4.7**〔事实〕
来源：https://comfy.org/cloud/supported-nodes/ComfyUI-WanVideoWrapper/ 、https://www.gitgenius.co/repos/kijai/ComfyUI-WanVideoWrapper

| 工作流 | 用途 | 说明 |
|---|---|---|
| wanvideo_480p_I2V_example_02 | 特效/风格 LoRA 标准底图 | 几乎所有 Civitai Wan 特效 LoRA 的推荐工作流都由它改来〔事实〕 |
| WanVideo VACE（2.1/2.2 module） | 姿态/深度/参考控制、局部重绘、视频续写 | WanVideoVACEEncode + 高低噪双采样器〔事实〕 |
| MultiTalk / InfiniteTalk | 长口播数字人 | chunk 81 帧 + 25 帧重叠；block swap + torch compile〔事实〕 |
| FantasyTalking | 表现力数字人 | wav2vec2 embeds〔事实〕 |
| WanAnimate / Animate-2 | 动作迁移 | 官方模板未覆盖的高级用法（relight LoRA、窗口续写）〔事实〕 |
| SkyReels DF 无限长（wanvideo_skyreels_diffusion_forcing_extension_example_01） | 多阶采样器接力无限续写 | 首段 53 帧，后续段 97 帧，前段末 17 帧作参考〔事实〕 |
| Context Windows 长视频 | 任意 Wan 模型 >1000 帧滑窗生成 | 重叠窗口混合〔事实〕 |
| Phantom / Stand-In / EchoShot / Lynx / HuMo / ReCamMaster / ATI / WanMove(轨迹) / MoCha | 主体一致、分镜、人脸、运镜、轨迹 | 节点清单见 comfy.org/cloud/supported-nodes〔事实〕 |

### 6.3 后处理链：超分 / 插帧 / 长视频拼接

| 工作流 | 用途 | 组件 | 显存 |
|---|---|---|---|
| **Downscale → SeedVR2 → RIFE → VHS 编码**（2026 推荐修复管线） | AI 视频去糊去块、时序一致放大 + 补帧 | SeedVR2 节点包（或更快的 FlashVSR）；RIFE/FILM（ComfyUI 0.26+ 内置 FrameInterpolate，权重 rife_v4.26 / film_net 放 models/frame_interpolation/） | 8GB：SeedVR2 3B GGUF Q4+block swap；12-16GB：3B/7B fp8；24GB：7B fp16〔事实〕 |
| wan-longer-videos 类拼接收尾 | 破 5 秒墙：续写多段 → ClearReality 放大 → RIFE 补帧 → 拼接 | GGUF 加载器 + VFI | —〔事实〕 |
| FramePack | 6GB 显存 60s 图生视频 | 独立节点/整合 | 6GB〔事实〕 |
| HunyuanVideo 1.5 latent upsampler | 720p→1080p | hunyuanvideo15_latent_upsampler_1080p | 需额外显存〔事实〕 |

来源：https://comfyui-mcp.artokun.io/docs/blog/video-upscale-comfyui 、https://github.com/artokun/comfyui-mcp/blob/main/plugin/skills/video-upscale/SKILL.md

---

## 7. 显存档位与 GGUF 量化速查〔事实+推断〕

GGUF 主渠道：**city96/ComfyUI-GGUF** 自定义节点 + **QuantStack** 系列转换。
热度证据：QuantStack Wan2.2-I2V-A14B-GGUF **83.1 万+ 下载**；Wan2.2-S2V-14B-GGUF **8.1 万+/月**〔事实〕
来源：https://alphasignal.ai/news/quantstack-s-wan2-2-brings-alibaba-s-image-to-video-ai-to-8gb-gpus 、https://huggingface.co/QuantStack/Wan2.2-S2V-14B-GGUF/

| 档位 | 可跑什么 | 方案 |
|---|---|---|
| **8GB** | Wan2.2 TI2V-5B（官方 offload）；FramePack 60s I2V；HunyuanVideo-Avatar 10GB（Wan2GP+TeaCache）；InfiniteTalk 480p 6GB+；SeedVR2 3B Q4 | 5B GGUF Q4_K_S 3.1GB / Q8_0 5.4GB〔事实〕 |
| **12GB** | Wan 2.2 14B GGUF（Q4 高低噪对）+ LightX2V 4 步；5B fp16/Q8 从容；MiniMax H3 剪枝 int8（下载量约 42GB） | 14B Q4_K_M 单专家 9.65GB，成对 19.3GB 依赖 offload〔事实〕 |
| **16GB** | Wan 2.2 14B GGUF Q5/Q6；HunyuanVideo 1.5 fp8（14GB 起）；LTX-2.3 FP8 distilled；Wan2.2 Animate fp8 | 16GB 是官方为 LTX-2.3 FP8/H3 标注的下限档〔事实〕 |
| **24GB（4090/3090）** | Wan 2.2 14B fp8 双专家（活跃专家驻 GPU、闲专家进内存，建议 32GB+ 内存）；Fun Camera/InP；S2V；Animate；LTX-2.5 INT8（21.5GB，实测 22.67GiB 常驻）；Hunyuan 1.5 720p 舒适 | fp8 双文件约 28GB 磁盘〔事实〕 |
| **32GB+（5090）** | LTX-2.5 蒸馏 4s 720p 约 25s；Hunyuan 1.5 193 帧 | —〔事实〕 |
| **48GB+/数据中心** | LTX 22B BF16、Wan 80GB 官方脚本、Hunyuan-Avatar 96GB 推荐 | —〔事实〕 |

〔推断〕显存之外，**系统内存 64GB + NVMe** 是视频工作流的隐形门槛（文本编码器 10-20GB 常驻内存、峰值解码爆显存）。

---

## 8. 生态成熟度对比（谁家 LoRA / workflow 最多）

| 排名 | 模型 | 证据 |
|---|---|---|
| **1. Wan 2.1/2.2** | 〔事实〕Kijai Wrapper 6.8k stars / 203 万注册表下载，官方模板 10+ 个；QuantStack GGUF 83 万下载；Civitai 特效/相机/风格 LoRA 几乎全以 Wan 2.1 I2V 14B 为底模（Remade 系列 100+）；Wan2GP 低显存全家桶；训练生态（kohya/默认 2500 步 LightX2V 组合）成熟 |
| **2. LTX-2.x** | 〔事实〕HuggingFace 单月 210 万下载（自称"最常用开源视频模型"）；fal 平台 80+ 微调；官方 IC/ID-LoRA；但社区第三方 LoRA/工作流数量仍明显少于 Wan〔事实+推断〕 |
| **3. HunyuanVideo 1.5** | 〔事实〕原生模板+超分完整，多来源评价"社区 LoRA 与实战工作流少于 Wan 2.2" |
| **4. SkyReels-V2 / MiniMax H3 / 其它** | 〔事实〕各有忠实子生态（SkyReels=无限长、H3=音视频一体+特效 embedding），LoRA 数量少；Mochi 1 仅剩训练基座价值 |

〔推断〕综合"可下载权重数量 × 官方模板 × 量化版本 × 第三方 LoRA"，**Wan 系是 2026-11 生态最成熟的开源视频模型家族**，且 2.1 的 LoRA 存量仍能在 2.2 低噪阶段复用。

---

## 9. 按场景推荐汇总表

| 场景 | 首选方案 | 备选 | 显存档位 | 备注 |
|---|---|---|---|---|
| **产品展示短视频** | Wan 2.1 I2V 14B + Remade 360 Orbit/Crane 相机 LoRA；或 Wan2.2-Fun-Camera | Phantom（商品跨镜头一致）；LTX-2.5（要同期声/4K） | 24GB fp8；16GB GGUF | 特效 LoRA 全部 480p 训练，先小后放大〔事实〕 |
| **角色动画（动作迁移）** | Wan-Animate-2（免预处理、文本视角控制）或 Wan 2.2 Animate Move 模式 | ATI 轨迹、SkyReels | Animate fp8 16-24GB | Mix 模式换人需 SAM2+relight LoRA〔事实〕 |
| **口播数字人** | Wan2.2-S2V（官方模板最省心） | InfiniteTalk（无限长/多人）；HunyuanVideo-Avatar（情绪/多角色）；FantasyTalking（表现力）；LTX IA2V（音视频一体） | S2V/IT 24GB；480p 6-12GB | 蒸馏 LoRA 会损口播动态，终稿 20 步〔事实〕 |
| **舞蹈/动作迁移** | Wan 2.2 VACE（pose）+ Fun Control Pose | Wan 2.2 Animate；HuMo | 24GB（14B）；VACE 1.3B 8GB | 掩码要连通、强度 0.6-1.0〔事实〕 |
| **运镜/航拍感** | Wan2.2-Fun-Camera-Control | ReCamMaster 轨迹重渲染；Crane/Orbit LoRA | fp8 24GB；LoRA 路线 480p 16GB | Lightning 加速会削运动幅度〔事实〕 |
| **特效短视频（膨胀/挤压/旋转/变身）** | Wan 2.1 I2V 14B 480p + 对应 Civitai LoRA（360 Rotation/Squish/Inflate/Transformation） | MiniMax H3 内置特效 embedding（子弹时间/爆炸等） | 480p 12-16GB 即可 | 触发词见第 4 节；仅 270° 是常见问题〔事实〕 |
| **长视频/无限续写** | SkyReels-V2 DF 多阶接力（Wrapper） | Wan Context Windows（>1000 帧）；FramePack 6GB/60s；LongCat-Video 分钟级 | 1.3B 8GB / 14B 24GB | 接力用前段末 17 帧作参考〔事实〕 |
| **视频修复放大** | Downscale→SeedVR2→RIFE→VHS 编码 | FlashVSR（更快）； latent upsampler（Hunyuan） | 8-24GB 按 3B/7B | 先降采样再修复是关键技巧〔事实〕 |
| **音视频一体成片** | LTX-2.5（4K 多镜头、24GB INT8） | MiniMax H3（立体声、多参考；注意许可证地区限制） | 24GB 起 | LTX 许可 <$10M 营收免费；H3 排除 US/EU/UK/KR〔事实〕 |
| **8GB 低配入门** | Wan2.2 TI2V-5B GGUF Q8（720p） | FramePack | 8GB | 系统内存建议 32GB+〔事实〕 |

---

## 10. 事实 / 推断 / 未核实清单

**确认事实**（均有上表来源）：Wan 开源止于 2.2 + Animate-2；Wan 2.5/2.6/2.7/3.0 闭源；各蒸馏 LoRA 的步数/CFG/配对规则；官方模板清单与实测耗时；QuantStack/Kijai 下载量与 star 数；Civitai 各特效 LoRA 下载数（快照口径）；HunyuanVideo 1.5 / LTX-2.x / MiniMax H3 的发布日、参数、许可证与显存数字；InfiniteTalk chunk 机制与实测显存。

**推断**：Wan 生态总量第一的排名（由多项间接证据汇总）；"系统内存 64GB+NVMe 为隐形门槛"；MultiTalk 已被 InfiniteTalk 实质取代；Civitai 风格 LoRA 以 Wan 为主流底模。

**未核实**：Kandinsky Video 开源状态；Civitai/LiblibAI 上风格类（动漫/写实）具体 LoRA 的下载量与稳定链接；Civitai 全站 Wan 底模 LoRA 总条数；LatentSync 等后期口型工具的最新状态。文中所有第三方下载量均为搜索当日快照，会随时间增长，请以站内实时数据为准。

**风险提示**〔事实〕：FusionX 作者明示其为多来源合并、商用需自行法律尽调；LTX-2.x（>$10M 营收）、MiniMax H3（地区排除）、腾讯系（地区排除）均有许可证条件，商用前务必核对。
