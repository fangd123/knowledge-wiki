---
title: "Civitai 推荐 LoRA：文章与清单索引"
date: 2026-11-08
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: 开源生图生视频生态
series_order: 5
tags:
  - AI技术与工程
  - 开源生图生视频生态
ai_assisted: true
status: published
---

# Civitai 推荐 LoRA：文章与清单索引

> 2026-11-08 用联网搜索检索（没有抓取 Civitai 页面），内容来自搜索摘要，参数以各页面为准。
> 当天的检索结果里没找到一张官方综合推荐榜。家族生态页能找同底模的条目，社区文章能看作者为什么选它；检索摘要不足以确认刷新频率。
> Wan 结果里出现了成人题材；当天样本不能估计它在头部或全站的占比。

## 一、最值得看的入口

| 类型 | 链接 | 用途 |
|---|---|---|
| 生态页（按下载量排名） | [Qwen](https://civitai.com/ecosystems/qwen) · [Z-Image](https://civitai.com/ecosystems/z-image) · [FLUX.2](https://civitai.com/ecosystems/flux2) · [Wan](https://civitai.com/ecosystems/wan) | 按页面当时的排序找条目；指标与刷新时间以页面为准 |
| Civitai 文章：综述 | [Guide to the General AI scene in 2026](https://civitai.com/articles/30487/guide-to-the-general-ai-scene-in-2026) | 各模型的显存需求；Qwen-2512 推荐 Wuli Turbo LoRA（4/8 步，作者推荐用 V3）。属于作者个人观点 |
| Civitai 文章：Wan 工作流 | [Wan 2.2 ComfyUI workflow favorites](https://civitai.com/articles/23316/wan-22-comfyui-workflow-favorites) | 工作流合集；指出 2.1 的 LoRA 主要作用在低噪阶段 |
| Civitai 文章：Wan 进阶 | [Intermediate understanding of video generation with WAN2.2 Lightx2v and SVIPro](https://civitai.com/articles/33809/intermediate-understanding-of-video-generation-with-wan22-lightx2v-and-svipro) | 讲 Lightx2v 加速、SVI Pro 长视频，以及运动类 LoRA 和其他 LoRA 冲突时的排查方法 |
| Civitai 文章：Wan 提示词 | [How To Prompt Wan Models](https://civitai.com/articles/17497/how-to-prompt-wan-models-full-tutorial-and-guide) | 运镜用语（dolly in、pan left、static shot） |
| Civitai 文章：Wan 电影感 | [Mastering Cinematic Video with WAN 2.2](https://civitai.com/articles/17844/mastering-cinematic-video-with-wan-22-in-comfy-ui) · [Full I2V Workflow Walkthrough](https://civitai.com/articles/24175/full-image-to-video-workflow-walkthrough) | — |
| 站外评测：Z-Image | [zimage.run：6 个必备 LoRA 深度评测](https://zimage.run/blog/zi-100-community-lora-review-en-20260615) · [smartart.live：写实 LoRA 压力测试](https://smartart.live/articles/253-top-10-stress-tests-for-realistic-loras-finding-the-best-z-image-turbo-setup.html) | zimage.run 推荐 Lenovo UltraReal、After Dark、Photo Factory（它给的 CFG 7 和 Turbo 的 CFG 1 有冲突，别照抄）；smartart 认为 ULTRA-R 技术指标最好，KODAK 观感更受欢迎 |
| 站外评测：Wan | [imagera：Wan 2.2 电影感 LoRA](https://imagera.ai/learn/best-cinematic-lora-wan-2-2-ai-video-2026) · [note.com：批量收集 475 个 Wan 2.2 LoRA 并对比](https://note.com/cute_agapan9087/n/nd2a0bc6fe4af?hl=en) | — |

## 二、搜索中出现的具体 LoRA

按任务看搭配，回到[生图场景清单](03-%E4%BB%BB%E5%8A%A1_B%EF%BC%9A%E5%BC%80%E6%BA%90%E7%94%9F%E5%9B%BE%E6%A8%A1%E5%9E%8B%E7%9A%84%E7%83%AD%E9%97%A8_LoRA_%E4%B8%8E_ComfyUI_Workflow%EF%BC%88%E6%8C%89%E5%9C%BA%E6%99%AF%EF%BC%89.md)。下面的参数是页面或作者的说法，本文没在显卡上跑。

### Qwen-Image 2.1
| LoRA | 用途 | 参数 |
|---|---|---|
| [Qwen Image 2.1 Restore](https://civitai.com/models/2989697/qwen-image-21-restore-lora) | 模糊/老照片修复 | 强度约 0.5 |
| [Qwen Image 2.1 Consistency](https://civitai.com/models/2969143/qwen-image-21-consistency-lora) | 编辑时保持一致 | — |
| [High resolution（Qwen 2.1）](https://civitai.com/models/2436859/high-resolution) | 高分辨率细节 | 搭配 Qwen-Image-2.1-Turbo，强度 0.9 |
| [Glamour Realism（作者称在 16GB 卡上测试）](https://civitai.com/models/2986100/qwen-image-21-glamour-realism-lora-16-gb-tested) | 写实人像 | 强度 0.6（0.5–0.7） |

### Qwen-Image-Edit 2511
| LoRA | 用途 |
|---|---|
| [Multiple-Angles](https://civitai.com/models/2300308/qwen-image-edit-2511-multiple-angles-lora) | 96 个机位，强度 0.8–1.0 |
| [ICEdit（DiffSynth-Studio）](https://civitai.com/models/2257887/qwen-image-edit-2511-icedit-lora) | 三图上下文编辑 |
| [Character Turnaround Sheet](https://civitai.com/models/2149265/character-turnaround-sheet-qwen-image-edit-25112509) | 角色三视图 |
| [Continuous Storyboard Generation](https://civitai.com/models/2293093/qwen-image-edit-2511-continuous-storyboard-generation) | 连续分镜 |
| [Transform into Anime Style](https://civitai.com/models/2429664/transform-into-anime-style) | 转动漫，触发词 `transform into anime style`，强度 0.7–1.0 |
| 工作流：[QIE2511 Workflow](https://civitai.com/models/2214702/qwen-image-edit-workflow) · [Multi Gen](https://civitai.com/models/1890385/qwen-image-edit-multi-gen) · [2511 Ultimate](https://civitai.com/models/2301814/qwen-image-edit-2511-ultimate) | — |

### Z-Image Turbo
| LoRA | 用途 | 参数 |
|---|---|---|
| [Realistic Snapshot v5](https://civitai.com/models/2268008/realistic-snapshot-z-image-turbo) | 写实抓拍（当日摘要记约 2,165 条评价） | 强度 0.6–0.7 |
| [Z-Image Turbo Lightning](https://civitai.com/models/2409672/z-image-turbo-lightning) | 进一步提速到 4–8 步 | 强度 0.5–1.0；可能影响文字准确度 |
| [Pixel Art Style](https://civitai.com/models/1770073/pixel-art-style-lora) | 像素画 | 强度 0.6–1 |
| [Z-Image-Aesthetic](https://civitai.com/models/2214707/z-image-aestheticbase-and-turbo) | 美学增强 | 会破坏加速效果：20 步，CFG 1.5–2 |
| [Famegrid 2nd Gen](https://civitai.com/models/2088956/famegrid-2nd-gen-z-image-qwen) | 写实（条目标题列 Z-Image 与 Qwen；摘要未核各自权重文件，按版本选择，不共用同一文件） | — |

### FLUX.2 klein 9B
| LoRA | 用途 |
|---|---|
| [Ultimate Upscaler Klein-9b](https://civitai.com/models/2474084/ultimate-upscaler-klein-9b) | 补高频细节，最高约 4MP（摘要所记 487 条评价，Very Positive） |
| [Consistence Edit](https://civitai.com/models/1939453/consistence-edit-lora) | 编辑一致性（摘要记 410 条评价；另见 Qwen-Edit 名称，未核两版的独立文件） |
| [Anything to Real Characters](https://civitai.com/models/2343188/flux2-kleinanything-to-real-characters) | 动漫转真人，CFG 1，8 步 |
| [Portrait Engine](https://civitai.com/models/2067704/portrait-engine-flux-flux-2-klein-z-image-detailed-skin-lora) · [InstaPic](https://civitai.com/models/2168120/instapic-ultrareal-the-authentic-social-media-lora-or-realism-flux-or-zit-or-flux-2-klein) | 皮肤细节和社交媒体实拍感（标题涉及 klein、Z-Image；未逐个核发版文件，请按各自底模选版本） |

### Wan 视频
| 资源 | 用途 |
|---|---|
| [lightx2v Wan2.2-Distill-Loras](https://huggingface.co/lightx2v/Wan2.2-Distill-Loras)（HF 官方源） | 4 步加速，高噪、低噪各一个 |
| [Long videos with full control（SVI 2 Pro）](https://civitai.com/models/2368359/long-videos-with-full-control-wan-22-i2v-svi-2-pro-individual-lora-multiple-reference-images-and-more) | 长视频工作流：每段可单独挂 LoRA，支持多参考图 |
| [Wan Join Clips](https://civitai.com/models/1695320/wan-join-clips) | 片段衔接 |
| [Matrix Bullet Time](https://civitai.com/models/1620140/matrix-bullet-time-camera-effect-wan21-i2v-lora) | 子弹时间运镜（Wan 2.1 I2V） |
