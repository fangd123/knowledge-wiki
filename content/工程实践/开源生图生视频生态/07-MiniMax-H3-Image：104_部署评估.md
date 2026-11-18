---
title: "MiniMax-H3-Image：104 部署评估"
date: 2026-11-11
author: fangd123
license: CC BY-NC-SA 4.0
category: 工程实践
series: 开源生图生视频生态
series_order: 7
tags:
  - 工程实践
  - 开源生图生视频生态
ai_assisted: true
status: review
---

# MiniMax-H3-Image：104 部署评估

调研：2026-11-10；父会话复核及汇总：2026-11-11。CodeBuddy任务 `cb-mwau7r3h-a7c8`（hy4free，约30分钟）。本次未安装、未下载权重、未启动服务、未运行推理。

## 结论

有真实可下载的 Comfy-Org ComfyUI 量化版本。推荐先在 **R9700 32GB** 上，以独立 ComfyUI 代码目录、独立 Python 环境和独立端口试验 **INT8 ConvRot DiT + 新图像VAE + 已有H3文本编码器**。新增必要下载约22.15GB；不建议直接覆盖现有环境。

“推荐试验”不等于“本机已经跑通”。官方PR报告了5090的1024²/2048²验证，不能直接当作R9700或3090实测。

## 父会话独立复核

- https://huggingface.co/Comfy-Org/MiniMax-H3-Image 的README与文件API确认：
  - `diffusion_models/minimax_h3_image_pruned_int8_convrot.safetensors`：20,979,314,802字节（20.98GB）。
  - `diffusion_models/minimax_h3_image_pruned_w4a8.safetensors`：12,549,794,530字节（12.55GB）。
  - `diffusion_models/minimax_h3_image_pruned_bf16.safetensors`：40.23GB。
  - `vae/minimax_h3_image_vae_fp16.safetensors`：1,168,505,312字节（1.17GB）。
- https://github.com/Comfy-Org/ComfyUI/pull/17655 已合并，合并提交 `ede4e5fe8d15445c6fcd6828d70b202c22f9502c`。PR明确使用同一个Qwen3-VL-32B文本编码器、新图像VAE、专门的PDD输出头；不是视频权重直接生成单帧。
- 复核时最新稳定版仍为v0.44.0；CodeBuddy从教程读到需0.45.0+/模板0.11.104+。部署前以是否包含该PR及模板实际要求为准，不追随未固定的master。
- SSH只读复核104：ComfyUI `v0.41.0-2-g29e28e2b`，R9700显存32624MB，3090显存24576MiB，内存62GiB、可用58GiB，/mnt/backup已挂载、可用797GB。
- 已有文本编码器位于 `~/llm/models/comfy-src/MiniMax-H3/text_encoders/`：INT8文件27,141,342,152字节，NVFP4 AWQ文件15,687,142,551字节，和官方文件列表大小相符。大小相符不是SHA256完整性校验，使用前仍应校验。
- :8188/:8189检查时均未监听，没有为此次调查启动它们。SSH复核命令末尾grep因无匹配退出1，前面的硬件/文件查询成功。

## 对CodeBuddy报告的纠正与限制

1. **内存参数解释错误**：本机 `comfy/cli_args.py:193` 明确说明 `--disable-smart-memory` 强制更积极卸载到RAM；`comfy/model_management.py:908-916,1121-1122`佐证。不是报告声称的“削弱自动卸载”。不要因此直接移除此参数。
2. INT8文本编码器文件大于3090显存，不等于绝对不能运行：还取决于CPU/分层卸载。NVFP4是候选减内存方案，其实际反量化内存与速度未测。
3. 减小DiT不能单独解决文本编码器阶段的内存问题。W4A8和NVFP4也不能按文件大小保证峰值显存。
4. CodeBuddy引述的SGLang教程为Linux CUDA方案，**不能据此把整个diffusers框架也判定为CUDA-only**；本次未验证该模型的diffusers ROCm全链路。
5. 未找到其它社区量化不等于不存在；这是首发当天的搜索范围结论。PrunaAI/P-H3-Image及H3视频量化不是本模型，不能混用。
6. 独立目录但共用并升级原Python环境不算完整隔离。建议克隆/新建环境；不在原环境执行依赖升级。
7. 官方Comfy重打包已包含所需PDD头布局；这条路线不要再照原版diffusers文件清单额外手工拼接PDD文件。
8. 算子兼容性和整机峰值内存尚未实测；不能保证不需要更新任何依赖，也不能保证16MP能跑。

## 建议部署步骤（待用户授权，不是执行记录）

1. 确认104无其它GPU任务，严格一次只用一张卡。
2. 独立目录 `~/llm/comfyui-h3image/`，独立克隆的ROCm环境；固定含PR #17655的提交或已发布稳定版。保留原v0.41.0与原环境。
3. 新权重目录放 `/mnt/backup/models/comfy-src/MiniMax-H3-Image/`；新实例用独立extra_model_paths配置，直接映射这个真实路径及现有H3编码器路径，避免遗漏软链接。
4. 从Comfy-Org仓库只下载INT8 DiT和图像VAE，校验大小与SHA256。复用已有INT8编码器。
5. 先不装提示词扩展器。需要时再下载7.34GB `qwen3.5_9b_w4a8_gptq.safetensors`。
6. 新端口如8190，独立input/output/user/temp目录；先仅绑定127.0.0.1，经SSH访问验证，确认需要局域网访问后再开放。原服务不动，不并行跑两个GPU实例。
7. 导入官方T2I模板：1024²、6步、CFG1、euler/simple、1张图，不用提示词扩展器。记录OOM/NaN/黑图、冷/热耗时、显存与系统内存峰值。
8. 通过后试2048²，再试单参考图编辑。1024²仅证明基本可运行，不代表模型在官方高分辨率范围的画质；暂不承诺16MP。
9. R9700若不兼容，再考虑3090独立CUDA环境，W4A8 DiT＋已有NVFP4编码器（分时卸载），两卡不同时高负载。失败停新实例即可，不覆盖旧环境。

## 来源及交付

- [官方魔搭仓库](https://modelscope.cn/models/MiniMax/MiniMax-H3-Image)
- [Comfy量化仓库](https://huggingface.co/Comfy-Org/MiniMax-H3-Image)
- [Comfy原生支持PR](https://github.com/Comfy-Org/ComfyUI/pull/17655)
- [官方Comfy教程](https://docs.comfy.org/tutorials/image/minimax/minimax-h3-image)
- CodeBuddy完整原始报告：`C:<本地路径>`
- CodeBuddy命令/来源摘录：同目录 `evidence.md`，远程小型文档在 `evidence_assets/`。

本笔记的纠正优先于原始报告。所有部署步骤尚未执行。
