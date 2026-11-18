---
title: "任务 D：LiblibAI（哩布哩布）上流行的 LoRA 与 ComfyUI 工作流调研"
date: 2026-11-08
author: fangd123
license: CC BY-NC-SA 4.0
category: 工程实践
series: 开源生图生视频生态
series_order: 6
tags:
  - 工程实践
  - 开源生图生视频生态
ai_assisted: true
status: review
---

# 任务 D：LiblibAI（哩布哩布）上流行的 LoRA 与 ComfyUI 工作流调研

> 调研日期：2026-11-08。全部信息来自当天联网检索（WebSearch + WebFetch 直接抓取 liblib.art 页面），来源 URL 逐条附后。
> 数字口径说明：LiblibAI 模型页通常显示 4 个数字，依次为 **在线生成数 / 下载量 / 点赞数 / 收藏数**（个别页顺序有出入，凡口径不确定处已标注"未核实"）。
> 无法核实的数字一律标"未核实"，未编造任何数字与链接。

---

## 0. 调研方法与限制（事实）

- WebFetch 可以读到 `liblib.art/modelinfo/...` 页面的**模型介绍、参数、许可、部分统计数字**（本次抓取约 8 个模型页，多数成功）。
- LiblibAI **匿名 GET 的站内 API 返回 404**（实测 `api/mixin/models/search`、`api/mixin/models`，2026-11-08，curl 状态码 404）。官方开放平台（OpenAPI）需要 AK/SK 密钥，按要求未注册、未提交任何信息。⇒ 站内数据只能靠页面抓取 + 搜索引擎快照。
- 平台首页导航为：探索 → 模型广场 / 工作流 / 作品灵感；创作 → 在线生成 / Web UI / **在线工作流 ComfyUI** / 训练 LoRA。（来源：http://www.liblib.art/ 首页抓取）
- 与用户设备匹配度：Z-Image-Turbo-FP8 约 8GB 显存、Qwen-Image-2512 24G 卡建议 1–2MP 出图、Wan2.2 4步一体化版"8GB 显存可跑"——均为官方页/作者页原文（事实），3090 24GB 全部可跑。
- 下载规则（事实，来自各模型页许可栏）：绝大多数 LoRA/Checkpoint 可直接下载到本地（个别作者设"不可下载"，只能在线用，逐条标注）；下载一般需登录，游客只能看。

---

## 1. 热门 LoRA（按底模分组）

### 1.1 Qwen-Image（1.0 / 2512 / 2.1 文生图）

| 名称 | 底模 | 场景 | 链接 | 热度 | 本地下载 | 用法要点 |
|---|---|---|---|---|---|---|
| 8步加速 Qwen-Image-Lightning-8steps-V2.0 | Qwen-Image 1.0 | 蒸馏加速（无风格） | https://www.liblib.art/modelinfo/6cd3da11201249e385531fa468fd8dc8 | 在线生成 341.6k，赞 2.2k（页面抓取，事实）；原版**作者设为不可下载** | ❌原版不可下载；bf16 可下载版 https://www.liblib.art/modelinfo/e0558eb060fa468fa63a8e768f318c94 （下载量 220.1k，据作者主页快照，事实） | 权重 0.8，步数 8，CFG 1；作者页另提醒"不建议用 4 步版，质量差"（马克笔 LoRA 作者实测，事实） |
| 8步加速LoRA-Qwen-Image-2512-Lightning-8steps | Qwen-Image-2512 | 蒸馏加速（2512 版） | https://www.liblib.art/modelinfo/72e50d27733a49038a5ebb83ea4ee452 | 在线生成 3.8k，下载 556，赞 91（页面快照，事实） | ✅（810MB） | 搭配 Qwen-Image-2512-FP8；权重 1，CFG 1，8 步 |
| 通用型 Qwen-Image LoRA 蒸馏加速模型（官方转载） | Qwen-Image 1.0 | 加速+提质 | https://www.liblib.art/modelinfo/0f4ca952c5bd42178a4ddd4cae6b6604 | 在线生成 19.8k，下载 1.2k，赞 737（页面抓取，事实） | ✅ 450MB | 推荐权重 0.8，CFG 7；评论区实测：二次元约 15 步、写实 20–30 步，CFG 1–4 更稳（事实） |
| 提升细节美感-Qwen-Image-LoRA | Qwen-Image 1.0 | 差分细节/美学增强 | https://www.liblib.art/modelinfo/e212d607eb034b3887c54492911ebbb9 | 下载 25.6k（作者主页快照，事实）；模型页本次未抓到统计（未核实） | 未核实（本次抓取无下载按钮信息） | 增加写实细节，强度 0.3–0.5（引用页原文，事实） |
| 80年代动漫风格_赛璐璐风_v2/v3 | Qwen-Image 1.0（v3 为 2512） | 复古动漫/赛璐璐 | https://www.liblib.art/modelinfo/f4c834f3f9aa46a895f9ac6ef516c043 | 页面显示 9.3k（口径未核实） | 未核实 | 触发词 **80动漫**，权重 0.8–1.0，30 步，CFG 4；勿混其他动漫风格词（作者页，事实） |
| Qwen-小红书胶片质感美女 | Qwen-Image 1.0 | 写实·胶片女像 | https://insight.liblib.art/modelinfo/0cbe619bbd40479393655795ffbf834b | 数字低（未核实热度，收录因风格典型） | 未核实 | 胶片朦胧质感，英文长提示词效果更好（作者页，事实） |
| Thick painting 厚涂美学 | Qwen-Image 1.0 | 艺术人像厚涂 | https://liblib.art/modelinfo/f23edfc75a66496e9fc742df64edd930 | 显示 171/12/1/7（小众，事实） | 未核实 | 需叠加"Qwen幻想曲写实beta"底模；起手式 "Thick painting, Digital painting, expressive brushwork"；30 步，Euler |
| 【Dave】商业级电商产品场景海报质感强化 | Qwen-Image（另有 Z-Image、F.1 版） | 电商海报/产品场景 | https://liblib.art/modelinfo/efe968c6597447919cf9a689ba87c559 | 未核实 | 未核实 | 触发词 **Product Scene Poster**，权重 1.0，35 步，CFG 6；Z-Image 版 12G 显存可跑（作者页，事实） |
| Qwen Image 2.1 生态 | Qwen-Image-2.1 | 2026-09 新底模 | https://www.liblib.art/modelinfo/bd2a3bfa881249409bf89d29b410f9d7 （配套文生图工作流） | 2.1 专属 LoRA 尚少： Lumos 的"2.1人像细节增强(视频人像素材)""质感场景光影增强"（作者页快照，热度低） | — | 2.1 搭配 Qwen3-VL-8B 编码器 + RGBA VAE；T8-8step 加速 LoRA 权重 0.7–0.9，8 步；CFG 固定 1.0（工作流页，事实） |

### 1.2 Qwen-Image-Edit（2509 / 2511 图像编辑）

| 名称 | 底模 | 场景 | 链接 | 热度 | 本地下载 | 用法要点 |
|---|---|---|---|---|---|---|
| Qwen-2511-多角度相机控制 | Qwen-Edit-2511 | 多角度/机位控制（电商三视图） | https://www.liblib.art/modelinfo/043a7b86ba9d4369ad7df27f6e457d02 | 在线生成 126.8k，下载 1.5k，赞 118，藏 71（页面快照，事实） | ✅（转载自 fal） | 提示词格式 `[方位角][仰角][距离]`，如 "front view eye-level shot medium shot"；96 机位=4仰角×8方位×3距离；配套批量出图工作流 https://www.liblib.art/modelinfo/e4c15aeaa0544bb48a7ada07ca7b2cc8 |
| Qwen-Edit-2511-8步加速-Lightning-LoRA | Qwen-Edit-2511 | 蒸馏加速 | https://www.liblib.art/modelinfo/e7c2c0ff48ff4cd3b4fdf66cff9bafc8 | 在线生成 98.5k，下载 278，赞 72（页面快照，事实） | ✅（fp16/fp8） | 权重 1，CFG 1，8 步；底模 Qwen-Image-Edit-2511-FP8： https://www.liblib.art/modelinfo/a8b5927091004a43bf1a57bca323e203 （下载 129.9k，作者页快照） |
| Qwen-Image-Edit-2511-Lightning-4steps | Qwen-Edit-2511 | 4 步加速 | https://www.liblib.art/modelinfo/1f54144f9d354727a6ad7cee2041a48d | 在线生成 7.0k，下载 374，赞 614（页面抓取，事实） | ✅ 810MB | 权重 1，CFG 1，4 步；无触发词 |
| 修复模糊照片 Unblur-Upscale | Qwen-Edit-2511 | 老照片/模糊图修复 | https://www.liblib.art/modelinfo/5aa00e7ddf1b464dac23c853b3124021 | 在线生成 870，下载 562（页面抓取，事实） | ✅ 225MB | 权重 1，CFG 1；无触发词；配套工作流 https://www.liblib.art/modelinfo/e1fef8dac9704787a99e8980ab991042 |
| 高清放大 Qwen-Edit-2509-Upscale（16 倍） | Qwen-Edit-2509 | 超分放大 | https://www.liblib.art/modelinfo/e10e866df3a2418e97fd957f210956cc | 在线生成 5.2k，下载 930（页面快照，事实） | ✅ | 最高 16 倍；LCM 采样最佳；需同时加载 2 个 LoRA（Upscale + 细节）权重 1.0 |
| Qwen-Image-Edit-2509-FP8 底模 | Qwen-Edit-2509 | 底模 | https://www.liblib.art/modelinfo/82bc45ff5bf34804ba3bce24e3dff112 | 下载 39.8k（作者页快照，事实） | ✅ | 2509 是 2511 之前的稳定版本，工作流兼容性好 |
| 电商模特换装 LoRA（Remove-Clothes 类） | Qwen-Edit | 换装 | https://www.liblib.art/modelinfo/99d2d7a0bf0e41bd9275bdbc9a84995d | 未核实 | 未核实 | 必须在提示词中写明"保留内衣"等约束（作者警示，事实）；见 2.4 换装工作流 |

### 1.3 Z-Image（Turbo / Base）

| 名称 | 底模 | 场景 | 链接 | 热度 | 本地下载 | 用法要点 |
|---|---|---|---|---|---|---|
| DetailDeamonZ-Slider 细节魔控 | Z-Image Turbo/RedZ1.5 | 细节滑块（增/减细节） | https://www.liblib.art/modelinfo/09d846c4217a4c2293fc0368dd4918ec | 在线生成 17.7k，下载 764，赞 126（页面快照，事实） | ✅ | **无触发词**；负权重 -0.4~-1.5 加细节、正权重 0.4~1.2 变干净；8–12 步；加载提示"部分层丢失"属正常 |
| Z-image 极致氛围光影 | Z-Image | 人像氛围光 | https://www.liblib.art/modelinfo/569b13bcfbc648b7ad445d2a8b63347c | 在线生成 1.6k，下载 211；带 **🌟新热Top100模型** 徽标（页面快照，事实） | ✅ | 触发词 **FWG**（放提示词开头），权重 0.8 |
| Z-Image-Turbo-600PX 亚洲人像抽卡机 | Z-Image Turbo | 亚洲人像 | https://www.liblib.art/modelinfo/bf0af34370ed45eda05f03c059890025 | 未核实 | 未核实 | 权重 0.5；多 LoRA 叠加时 0.2–0.3（作者页，事实） |
| Z-image 造相-复古胶片人像摄影 | Z-Image | 胶片人像/港风 | https://www.liblib.art/modelinfo/b20e244345e4477f82effaf8c59b3ed3 | 未核实 | 未核实 | 权重 0.3–0.5（越低越贴胶片感），30–45 步 |
| Z-Image 真实感网感摄影写真 | Z-Image | 网感写真/小红书风 | https://insight.liblib.art/modelinfo/85659282aac4478a93f1e1809dd6ee22 | 显示 163/7/166/2（口径未核实） | 未核实 | 触发词 **IPBD**，Euler 40 步（低于 40 易糊），CFG 3.5，CLIP 2 |
| ZIT写实皮肤质感 / 真实感光影 z-image-turbo-flow-dpo / Z-Image-Turbo-Detail-Slider | Z-Image | 皮肤质感/光影/细节 | 链接未核实（作者主页快照收录） | 下载 19.3k / 11.1k / 16.4k（作者主页快照，事实；单页 URL 未拿到） | 未核实 | 名称与生态见造相合集页： https://www.liblib.art/modelinfo/611893d466e142079f2cc03acdac7a42 |
| 造相-Z-Image-Turbo_FP8_Scaled（底模） | Z-Image Turbo | 底模 | https://www.liblib.art/modelinfo/90f2ea851e194090bafde30b2730f9b6 | 下载 48.1k（作者页快照，事实） | ✅ | FP8 约 8GB 显存；Apache 2.0 免费商用（作者页原文，事实）；Z-Image-Turbo 8 NFE 亚秒级推理 |

### 1.4 FLUX.2 / klein（9B / 4B）

| 名称 | 底模 | 场景 | 链接 | 热度 | 本地下载 | 用法要点 |
|---|---|---|---|---|---|---|
| FLUX.2-klein-9b-base-fp8-v2（底模） | F.2 klein 9B | 底模 | https://www.liblib.art/modelinfo/a97e66dfafcb4bfeb17f6075c4d88b2d | 当前 v2 页：在线 176，下载 330（页面抓取，事实）；作者主页"Flux-2-klein-9b-fp8 下载 116w、赞 719"（主页快照，事实，疑为 v1 旧版本页） | ✅ 8.46GB | 搭配 qwen_3_8b_fp8mixed 文本编码器 + flux2-vae；Euler，CFG 1；Apache 2.0 |
| Flux2-Klein-换脸-BFS | F.2 klein 4B/9B | 换头换脸 | https://www.liblib.art/modelinfo/27d9af0f72904f7d99a509336a192258 | 下载 13.3k（作者主页快照，事实） | ✅ | 中文指令式提示词（"以图片1为底图…替换为图片2头部"）；RunningHub 有配套工作流教程（事实） |
| F2Klein-9b 产品融图-电商光影融合 | F.2 klein 9B | 电商产品溶图 | 链接未核实（作者主页快照：下载 3.4k，事实） | 未核实 | 未核实 | 产品与场景光影融合 |
| flux klein 9b 修复图像（Restore） | F.2 klein 9B | 图像修复 | https://www.liblib.art/modelinfo/c8780b7aaddd4769ae7d44408d144209 | 在线生成 1.0k（页面快照，事实） | ❌ **作者设置不可下载**，只能在线用 | 触发词（英文长句）"restore the image quality, remove any compression artefacts…"；权重 1，CFG 1，**4 步效果优于 20 步**（作者实测，事实） |
| klein 4 步/8 步加速 LoRA 系列 | F.2 klein | 蒸馏加速 | 4步：https://www.liblib.art/modelinfo/8e29b1582b76493fab95b095c802518e 、 https://www.liblib.art/modelinfo/874f8ac35f6b441180ae334c63364b78 、 https://www.liblib.art/modelinfo/3a7e1d91ab7b44df941d95cafa99fff1 ；8步： https://www.liblib.art/modelinfo/5f1555fea1b946eea66d37c7775cea56 、 https://www.liblib.art/modelinfo/3c18247a71d64286aade89c294af3720 | 未核实 | 未核实 | 来自公众号实测文的官方整理清单（事实）；klein 蒸馏版本身 4 步可出图 |
| lux-2-klein-9B-KV 图像编辑整合（KV 缓存版） | F.2 klein 9B-KV | 多参考图编辑提速 | https://www.liblib.art/modelinfo/ef670da4ca9e4951baaa6732bbc3133f | 未核实 | 未核实 | KV 缓存使多图编辑最高提速 2.5 倍；4 步蒸馏；ComfyUI 尚未完全支持（页面原文，事实） |
| TinFlux2klein AgeSlider 年龄滑块 | F.2 klein 9B | 年龄控制 | https://insight.liblib.art/modelinfo/b73af27568e54df3844025281abc0e31 | 未核实 | 未核实 | 无触发词，权重 -2~+2（负更年轻/正更成熟）；**注意：作者称哩布已取消 Flux 2 Klein 9B/4B 在线分类，新模型在线使用会出问题**（作者页原文，事实） |

### 1.5 Wan 2.x 视频

| 名称 | 底模 | 场景 | 链接 | 热度 | 本地下载 | 用法要点 |
|---|---|---|---|---|---|---|
| Wan2.2_Lighting_T2V 新版4步加速 LoRA | Wan 2.2 T2V | 文生视频加速 | https://www.liblib.art/modelinfo/d20b3d57c92e498186090514c56a5838 | 在线生成 1.4k，下载 260（页面快照，事实） | ✅ | 权重 1，CFG 7，4 步，无需 CFG 技巧，约 20 倍加速 |
| 通义万相 WAN2.2 变身超级赛亚人_图生视频 | Wan 2.2 I2V | 特效变身（真人/卡通/动物通用） | https://www.liblib.art/modelinfo/22e1b1f816784e1c9d34196a8a62193b | 在线生成 707，下载 88（页面快照，事实） | ✅ 292MB | 搭配 Wan2.2-I2V-A14B-**HIGH** 高噪版；出图 <896px；权重 1，Euler + sgm_uniform，10 步；触发词即变身描述句 |
| lightx2v_elite_it2v_animate_face | Wan2.2 Animate | 面部动画增强（数字人） | https://www.liblib.art/modelinfo/3ff829082a5e45df9d419c91bf15dc00 | 热度低（8 在线，事实） | ❌ 不可下载 | 须 sa_solver + beta 调度器 + 4 步；解决"脸僵/口型对不上"；24G 卡流畅 |
| Wan2.2-Distill-Loras（lightx2v 官方加速） | Wan 2.2 | 4 步加速 | 官方在 ModelScope：https://modelscope.cn/models/lightx2v/Wan2.2-Distill-Loras/files （哩布工作流页引用，事实） | — | ✅（站外直下） | high/low 两段各配一个；6 步工作流中起止降噪步数为总步数 50% |
| Wan2.2-Remix-I2V（一体化模型） | Wan 2.2 Remix | 图生视频（免 LoRA 直出） | 模型： https://www.liblib.art/modelinfo/ffd3e1b8f9fc4ae29609a47e110fbf60 ；工作流： https://www.liblib.art/modelinfo/c0913977c1f44d7a87154adfe53ac5d4 | 未核实 | ✅ | 与 lightx2v 兼容无需额外加速；4–8 步，CFG 1，Shift 5–10，Euler/Simple |

> Wan 生态观察（推断）：哩布上 Wan 的热度集中在**工作流**而非 LoRA（见第 2 节， Wan2.2 工作流在线生成 23k 级），LoRA 以加速类为主、风格类多为特效向。

---

## 2. 热门 ComfyUI 工作流（哩布"工作流"区/智能图片模板）

| 名称 | 底模 | 场景 | 链接 | 热度 | 本地下载 | 用法要点 |
|---|---|---|---|---|---|---|
| Wan2.2 视频生成 图生视频/文生视频（ComfyUI 官方教程版） | Wan 2.2 | 图生/文生视频 | https://www.liblib.art/modelinfo/f08179321fd145c490bbe5dd5743ea2e | 在线生成 23.1k，下载 1.7k（页面快照，事实） | ✅（.json 随页下载） | high+low 双模型；CLIP umt5_xxl_fp8、VAE wan_2.1；上传起始帧 + 改 CLIP Text Encode |
| wan2.2极速4步图生视频 | Wan 2.2 | 4 步加速 I2V | https://www.liblib.art/modelinfo/2c9b1d8e5a754d04bc2cb0d6a974afdd | 在线生成 23.0k，下载 2.7k，藏 33（页面快照，事实） | ✅ | lightx2v 加速 LoRA + 高清放大 + 补帧；默认 5s/16fps |
| Wan2.2 14B 4步极速（首尾帧/I2V/T2V 三合一） | Wan 2.2 | 一体化（含 CLIP+VAE+加速 LoRA） | https://www.liblib.art/modelinfo/c9414644e32243399de4487c75bbaa79 | 下载约 1k（两个版本快照分别 992/1.1k，事实） | ✅ | 单 Checkpoint 放 models/checkpoints；4 步、CFG 1、sa_solver + beta 调度器；宣称 8GB 显存可跑 |
| Qwen-image-edit-2511 图像编辑（单图/双图/三图） | Qwen-Edit-2511 | 通用图像编辑 | https://www.liblib.art/modelinfo/42da2d86f98d46069fff921994e36c97 | 在线生成 4.6k，下载 729，赞 45，藏 69（页面快照，事实） | ✅ | 2511 内置光照/材质/视角 LoRA；指令式中文提示词；配 Lightning-4steps 加速 |
| 千问 Qwen-Edit2511 图生图（多角度控制）+局部重绘+洗图 | Qwen-Edit-2511 | 多角度/重绘/洗图 | https://www.liblib.art/modelinfo/378b802989e04a39a68ad2aa4a4a6ec5 | 未核实 | ✅ | 默认 8 步 Lightning，CFG 2.5，降噪 1.0；局部重绘降噪 0.86±0.01；最长边 1664 |
| Qwen Image 2.1 文生图工作流 | Qwen-Image-2.1 | 文生图（2026 新底模） | https://www.liblib.art/modelinfo/bd2a3bfa881249409bf89d29b410f9d7 | 在线生成 32（新发布，事实） | ✅ | 25–40 步 CFG 1.0；T8 加速 LoRA 8 步；24G 卡建议 1–2MP，3MP 上限 |
| Qwen-Image家族全系列工作流合集（六合一） | Qwen 全家 | 文生图/编辑/加速合集 | https://www.liblib.art/modelinfo/2448e05beea84496b8e4410e4c0c514c | 未核实（评论区有 16G 卡反馈，事实） | ✅ | 附全套底模/LoRA/工作流链接清单，适合一次性下载 Qwen 生态 |
| 造相-Z-Image-Turbo 标准工作流 & 优化版 2K 直出 | Z-Image Turbo | 文生图/图生图/反推/局部重绘 | https://www.liblib.art/modelinfo/7d72b6aa66844d1e99a51758b7fdbb99 | 在线生成 7.6k，下载 3.3k，赞 100（页面快照，事实） | ✅ | V2 新增局部重绘 + 亚洲人像 LoRA；2K 直出 |
| flux-2-klein-9B 图像编辑精华整合版 | F.2 klein 9B | 文生图/单图/双图编辑/精修/放大 | https://www.liblib.art/modelinfo/f43f9818a37f44cf821f18bd2198df6f | 页面数字口径未核实（公众号专文推荐，事实） | ✅ | 统一多任务架构；FP8 单卡可跑（RTX 3090 实测场景）；配 qwen_3_8b_fp8mixed + flux2-vae |
| Controlnet控制-Flux-2-klein-9B | F.2 klein 9B | Canny/Openpose/Depth/线稿/扩图/重绘 | https://www.liblib.art/modelinfo/ac2d3352a25b4b40846f1ab3c3ce9f98 | 在线生成 781，下载 562（页面快照，事实） | ✅ | 六种控制合一；文本编码器按显存选 fp4mixed(6.8GB)/fp8mixed(8.66GB) |
| 详情图洗图改稿 | F.1/Kontext 系 | 电商主图/详情页洗图 | https://www.liblib.art/modelinfo/61367b3a612a4cb09b8cb70bf0548de4 | 在线生成 18.4k，下载 729，藏 186（页面快照，事实） | ✅（可本地部署，联系作者） | 上传原图 + 设洗图强度：5–20% 彻底重绘 / 50–60% 保构图 / 80% 仅精修 |
| 全能_Pro_完美换装（最强一致性，支持 4K） | 图像编辑大模型 | 换装/虚拟试衣 | https://www.liblib.art/modelinfo/cb83210f13024b17b2a70894f4427d54 | 未核实 | ✅ | 自动/手动双遮罩；多参考图逐一遮罩防服装串染 |
| Qwen-Image-Edit 电商模特换装工作流 | Qwen-Edit | 电商换装 | https://www.liblib.art/modelinfo/6ba0cced87ea4d4fae2d2d8dee870ce6 | 在线生成 1.7k，下载 1.0k，赞 30，藏 58（页面快照，事实） | ✅ | 需换装 LoRA（1.2.9 节链接）；提示词必须写保留内衣类约束；V2 加 Nano Banana 穿衣支线 |
| 在线运行 SDXL 重绘老照片 高保真上色修复放大 | SDXL | 老照片修复 | https://www.liblib.art/modelinfo/318323bdbc0342a3a4dab028162c651b | 页面统计未抓到（未核实） | ✅ | 三阶段：BOPBTL 修划痕 → SDXL 上色（深度+线稿+ReActor 锁脸）→ 放大；可开关各阶段；模糊严重时关提示词反推 |
| AI 老照片修复 V3 ComfyUI | SDXL 系 | 老照片修复 | https://www.liblib.art/modelinfo/a7cf46c099e94aec9e1f0a0de1ee6469 | 未核实 | ✅ | llava+qwen2 反推提示词、depth/openpose/tile 三 ControlNet、ReActor 换脸降 DIST、DDcolour 上色、SUPIR 放大；RTX 3060 约 300 秒 |
| 修复模糊照片-高清放大-Qwen-Edit-2511 | Qwen-Edit-2511 | 模糊照片修复 | https://www.liblib.art/modelinfo/e1fef8dac9704787a99e8980ab991042 | 在线生成 237，下载 200（页面快照，事实） | ✅ | 配 Unblur-Upscale LoRA，权重 1，CFG 1 |
| 高清放大 Qwen-Edit-2509-Upscale（16 倍） | Qwen-Edit-2509 | 超分放大 | https://www.liblib.art/modelinfo/e10e866df3a2418e97fd957f210956cc | 在线生成 5.2k，下载 930（页面快照，事实） | ✅ | 须同时加载 Upscale + 细节两个 LoRA（作者强调，事实） |
| 音频转视频（Wan2.2 对口型数字人） | Wan 2.2 | 数字人/对口型 | https://www.liblib.art/modelinfo/d8d63c9f784546f8b0c0928238404ea4 | 在线生成 826，下载 29（页面抓取，事实） | ✅ | 传角色照 + 语音（≤10s 较稳）+ 简短提示词；lightx2v 加速 |
| 智能图片V2 模板系列（亚马逊主图/详情页/跨境多语种/短剧分镜/三视图定妆） | 平台闭源模型 | 电商/短剧一站式 | 作者主页示例： https://liblib.art/userpage/061d9a8726f045459ec970bac04f88d0/publish | 单模板下载量 300–1.7k 级（主页快照，事实；单体 URL 逐个未列） | ⚠️ 模板多为**平台内一键使用**，非 ComfyUI json | "智能图片V2"是哩布托管模板，输入产品图+文案自动排版 |
| 扩图工作流合集（平台官方教程页收录 6 款） | FLUX/Fill 系 | 扩图 | 教程页： https://www.liblib.art/activities/388fcb1b3f5b42838bc20af2cfdf9351/Confyui_Expand | 教程页显示"无缝扩图-在线生图 YOUS 19.5k""FLUX+fill 木木木亘 1.3k"等（快照，事实；单体 URL 未列） | ✅ | 外补画板每方向 ≤400、羽化 20–40、大扩展分两轮迭代（官方教程，事实） |

---

## 3. 榜单 / 专题入口与筛选方法

**事实（来自 2026-11-08 首页抓取 http://www.liblib.art/ 与教程文章）：**

1. **模型广场筛选器**：导航"探索 → 模型广场"→ 右上"筛选"→ 先选"模型类型（Checkpoint/LORA/工作流）"，再按"基础算法/底模"选具体模型（如 z-image、Qwen-Image、F.1、通义万相 2.2）。第三方教程原文描述了同样的操作路径（来源：https://x-humanoid.csdn.net/6a9edd3e1a35784c841dcac6.html ）。
2. **首页专题合集（可商用精选）**：🌀Kontext专业设计、🔥**爆款Top100模型【可商用】**、🌟**新热Top100模型【可商用】**、推文·头像·角色生成、节气节日、图标·LOGO·字体、游戏概念设计、💡**Qwen·创意设计**、会员专属/会员下载模型。其中"新热Top100"徽标会出现在单个模型页（如 Z-image 极致氛围光影页可见，事实）。
3. **快捷生图场景入口**：古风/插画/IP/图标/质感/摄影/产品/电商/海报/游戏——按场景直达对应模型与模板。
4. **工作流区**：导航"探索 → 工作流"，可用同样的类型/底模筛选；作者主页亦可按 `userpage/xxx/publish/workflow` 只看工作流（多个作者页实测，事实）。
5. **活动/大赛**：`liblib.art/activities/...` 页（如扩图教程活动页、华硕 ProArt AIGC 大赛 https://www.liblib.art/activities/00676b81377446fa9cc839b376c359b5/NVIDIA_ASUS_ProArt_creative_activity ）；投稿带活动标签即可参赛。
6. **LibTV 创作者挑战赛**（视频侧）：入口在 LibTV 首页右上角"创作者挑战赛"，每月活动与获奖作品公示（来源：哩布作者收益公告 https://liblibai.feishu.cn/docx/Y7aFdjkuqokWI3xJzzUcauAInRd ）。
7. **官方在线工具直达**：数字人 https://www.liblib.art/ai-tool/digital-human 、图像生成器 https://www.liblib.art/ai-tool/image-generator 、在线 ComfyUI https://www.liblib.art/comfy （官方公众号与页面原文，事实）。

**推断/未核实：**
- 模型页 URL 中的 `rankExpId` 参数表明存在站内推荐/排行体系，但**未找到公开的独立"排行榜"页面 URL**（未核实）。
- 首页抓取中的热门 Checkpoint（AWPainting 1452w 下载、MYHuman 229w、Qwen-Image 105w 等）为首页推荐位快照，可视为官方热度榜的近似（推断）。

---

## 4. 主要结论（速览）

1. **加速 LoRA 是哩布绝对顶流**：Qwen-Image-Lightning-8steps 在线生成 341.6k、其可下载 bf16 版下载 220.1k，远超一切风格 LoRA；Wan 侧 lightx2v 4 步体系同理。
2. **Qwen-Edit-2511 已成为编辑默认底模**：官方 FP8 底模下载 129.9k，8 步 Lightning（98.5k 在线）+ fal 多角度相机控制（126.8k 在线）是两个最热附件。
3. **Z-Image 生态以"造相系"为主**：Turbo-FP8 底模下载 48.1k，LoRA 集中在人像（600PX 抽卡机、胶片、网感写真）与滑块（DetailDeamonZ 17.7k 在线）；FP8 仅约 8GB 显存，最适合本地。
4. **FLUX.2 klein 的下载量集中在底模**（作者页显示其 fp8 版下载 116w 级），LoRA 以换脸 BFS（13.3k）和修复类为代表；注意哩布已取消 klein 独立在线分类（作者原文）。
5. **视频类热度在工作流而非 LoRA**：Wan2.2 官方教程工作流与 4 步极速版在线生成都达 23k；数字人对口型有官方 AI 工具（在线）+ 音频转视频工作流两条路。
6. **电商是工作流最大场景**：详情图洗图改稿（18.4k 在线）、智能图片V2 模板（亚马逊主图/详情页）、换装（完美换装 Pro、电商模特换装 1.7k）构成主力。
7. **老照片修复三条路线**：SDXL 重绘上色三阶段流（质量最高）、Qwen-Edit-2511 Unblur（最快）、Qwen-Edit-2509 Upscale 16 倍放大（5.2k 在线）。
8. **本地化提示**：除少数"作者设置不可下载/在线专用模板"外，以上 LoRA 与工作流均可下载 json/权重到本地 ComfyUI；LiblibAI 匿名 API 不可用（404），批量获取仍需网页或客户端（pc.liblib.art 支持本地模型共享直连，官方页原文）。
