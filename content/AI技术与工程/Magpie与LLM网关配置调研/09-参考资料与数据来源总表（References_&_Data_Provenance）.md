---
title: "参考资料与数据来源总表（References & Data Provenance）"
date: 2026-11-13
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: Magpie与LLM网关配置调研
series_order: 9
tags:
  - AI技术与工程
  - Magpie与LLM网关配置调研
ai_assisted: true
status: published
---

# 参考资料与数据来源总表（References & Data Provenance）

> **用法**：给[通俗图解](08-%E9%80%9A%E4%BF%97%E5%9B%BE%E8%A7%A3%EF%BC%9A%E5%A4%A7%E6%A8%A1%E5%9E%8B%E7%BD%91%E5%85%B3%E6%98%AF%E5%A6%82%E4%BD%95%E5%B8%AE%E4%BD%A0%E7%9C%81%E9%92%B1%E3%80%81%E6%8F%90%E9%80%9F%E5%8F%88%E4%B8%8D%E5%8F%98%E5%82%BB%E7%9A%84%EF%BC%9F.md)追数字的来路：仓库是机制证据，文档是费率和协议入口，图解里的算例不是用户账单。下表记录的是原调研那一天，并非这次重新检出全部版本。
> **数据采集与代码克隆核验基准日期**：`2026-11-13`  
> **来源标识约定**：
> - `[Git]`：原调研记录了源码检出或实现定位；并不代表每行版本号都已独立复核。
> - `[API]`：通过 `https://api.github.com/repos/<owner>/<repo>` 实时拉取仓库元数据（Stars、License、Archived 状态、最近推送时间）；
> - `[Web]`：通过公开网页检索与官方文档核验（用于未全量克隆的边缘对照项目或闭源商业插件说明）。

---

## 一、 核心拆解对象一手源码基准

| 项目名称 | GitHub 仓库链接 | 核验版本 / Commit SHA `[Git]` | 核心审计源码路径与关联记录 |
| :--- | :--- | :--- | :--- |
| **`yetone/magpie`** | [https://github.com/yetone/magpie](https://github.com/yetone/magpie) | `70f4593e623e0847fcc2d869240565ce5f73a67a`<br>*(2026-11-13)* | • **缓存窗口与 TTL 重放寻优（Tuned for you）**：`internal/tune/tune.go`、`internal/gui/tune.go`、`internal/agent/tune.go`、`internal/gui/assets/context.js`、`internal/sessions/calls.go`<br>• **命名辨析（防 ReDoS 截断正则）**：`internal/capre/capre.go`<br>• **三层难度与语义路由**：`internal/gateway/compacting.go`、`internal/gateway/desktop_subagent.go`、`internal/provider/rulecond.go`、`internal/gateway/classify.go`、`internal/gateway/decide.go`、`internal/provider/grouptemplate.go`<br>• **缓存防撕裂五重防线与多池策略**：`internal/gateway/affinity.go`、`internal/gateway/rules.go`（`keepsEffort`）、`internal/gateway/effort_update.go`（GPT-6 `configuration_update` 与 Issue [#617](https://github.com/yetone/magpie/issues/617)）、`internal/gateway/claude_cache.go`、`internal/gateway/speed.go`、`internal/routes/fastest.go`、`internal/usage/usage.go` |
| **`chefzejia/magpie-speed-lab`** | [https://github.com/chefzejia/magpie-speed-lab](https://github.com/chefzejia/magpie-speed-lab) | `v0.3.0` / `75e91f590e6551fdc6173281a161613444120151`<br>*(2026-11-04)* | • **同机桌面界面**：`index.mjs`、`lib/server.mjs`、`page/boot.js`、`page/session.js`<br>• **四阶段耗时拆解与 `flow_ms` 窗口折算速率**：`lib/telemetry.mjs`<br>• **并发双模探针**：`lib/probe.mjs`<br>• **四维正交打分与 4 类工作负载排行**：`lib/ranking.mjs`、`ui/app.js`、`test/*.test.mjs` |
| **关联生态提交与校验规范** | • [magpie-community/plugins](https://github.com/magpie-community/plugins)<br>• [Wei-Shaw/sub2api PR #2485](https://github.com/Wei-Shaw/sub2api/pull/2485) / [PR #2402](https://github.com/Wei-Shaw/sub2api/pull/2402)<br>• [yetone/magpie Issue #2128](https://github.com/yetone/magpie/issues/2128) | `2026-11` 检索核实 | • `magpie-community/plugins/scripts/check.mjs`（插件导入无副作用校验规范）<br>• 作者 `chefzejia` 在 `sub2api` 提出的慢请求物理阶段解耦观测方法论溯源 |

---

## 二、 云厂商官方计费倍率与推理协议数据源

费率文档解释计价口径；网关源码解释这版实现用了哪些倍率。模型特定行为若只见项目注释，单列说明。

1. **Anthropic Claude Prompt Caching 定价与生命周期规范**
   - **官方文档**：[Anthropic Docs — Prompt Caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) & [Anthropic Pricing](https://www.anthropic.com/pricing)
   - **核心费率参数**：
     - `5m` 默认临时缓存写入（5-minute cache write）：**`1.25x`** 基础输入单价；
     - `1h` 延长缓存写入（1-hour extended cache write）：**`2.0x`** 基础输入单价；
     - 缓存命中读取（Cache hit / read）：**`0.1x`** 基础输入单价（即 1 折）；
     - 缓存断点上限：最多 `4` 个 `cache_control` 断点，最小可缓存前缀长度依模型为 `1,024` ~ `4,096` tokens。
   - **源码交叉印证**：`yetone/magpie` (`internal/tune/tune.go:81-86`)、`montevive/autocache` (`internal/pricing/pricing.go`)。
2. **OpenAI Prompt Caching 与 Reasoning Effort / Responses API 规范**
   - **官方文档**：[OpenAI Docs — Prompt Caching](https://platform.openai.com/docs/guides/prompt-caching) & [OpenAI Docs — Reasoning Models](https://platform.openai.com/docs/guides/reasoning)
   - **核心参数与行为**：
     - 自动前缀缓存触发门槛：`>= 1,024` tokens（按 `128` tokens 步进增量匹配），无需显式写入溢价（`Write = 1.0x`）；
     - 思考预算档位以对应模型规范为准。网关尝试内联降档、对升档另作处理，来源是 `internal/gateway/effort_update.go` 及其注释；不是 OpenAI 对缓存命中或推理量的保证。
3. **DeepSeek 上下文硬盘缓存与错峰计费 / GitHub Copilot & OpenRouter 特殊费率**
   - **官方文档**：[DeepSeek API Docs — Context Caching & Pricing](https://api-docs.deepseek.com/quick_start/pricing)
   - **源码交叉印证**：`yetone/magpie` `internal/gui/tune.go:239-257`（`pricedWeights` 识别 GitHub Copilot 等渠道对 `1h` 缓存按 `5m` 费率计价）与 `musistudio/claude-code-router`（`openrouter-discount-provider-router.ts` 多下游供应商动态折扣与缓存损失对账）。

---

## 三、外部开源生态：按赛道列的材料索引

### 3.1 赛道 A：智能难度路由、推理预算寻优与 Coding Agent 网关（对应报告 `03`）

| 序号 | 开源仓库（可点击访问） | 核验 Commit / 状态 | 采集方式 | 核心核查源码路径 / 关键机制 |
| :---: | :--- | :--- | :---: | :--- |
| 1 | [lm-sys/RouteLLM](https://github.com/lm-sys/RouteLLM) | `0b64fda` (2024-08-10) | `[Git][API]` | `routellm/routers/routers.py`、`calibrate_threshold.py:55`（MF/SW/BERT/Causal 四路由器 + `quantile(1-p)` 阈值校准） |
| 2 | [vllm-project/semantic-router](https://github.com/vllm-project/semantic-router) | `f47e114` (2026-11-13) | `[Git][API]` | `pkg/decision/engine.go`（20 信号）、`pkg/selection/cache_affinity.go:46`（`affinityMaxLambda=0.14`）、`session_switch_gate.go` |
| 3 | [theagentrouter/agent-router](https://github.com/theagentrouter/agent-router) | `165733c` (2026-11-10) | `[Git][API]` | 原 `envoyproxy/ai-gateway`，K8s 双层 AI 控制面 + ExtProc 挂载 `semantic-router` |
| 4 | [katanemo/plano](https://github.com/katanemo/plano) | `b13ddd6` (2026-10-24) | `[Git][API]` | 原 `archgw`；`crates/brightstaff/src/router/orchestrator.rs`、`session_router.rs`（`switch_cost_in_usd` 反事实门控）、`loops.rs` |
| 5 | [BlockRunAI/ClawRouter](https://github.com/BlockRunAI/ClawRouter) | `0bf510b` (2026-11-12) | `[Git][API]` | `src/session.ts`、`outcome-memory.ts`（亚毫秒启发式路由 + Session Pinning 与 `SIMPLE` 旁路） |
| 6 | [BlockRunAI/router-core](https://github.com/BlockRunAI/router-core) | `2c2d0a5` (2026-11-11) | `[Git][Web]` | `src/rules.ts`、`portfolio.ts`（15 维特征加权 + 30 天衰减 + 3-Strike 升档） |
| 7 | [NadirRouter/NadirClaw](https://github.com/NadirRouter/NadirClaw) | `06059e1` (2026-11-12) | `[Git][Web]` | `nadirclaw/classifiers/`、`server.py`、`tier_config/selector.py`（Wide&Deep 非对称代价解码 + `upgrade_if_higher`） |
| 8 | [ulab-uiuc/LLMRouter](https://github.com/ulab-uiuc/LLMRouter) | `c65ee01` (2026-10-08) | `[Git][API]` | `graphrouter/graph_nn.py`、`custom_routers/fusion_gate/`（19 种内置路由器 + 三段预算融合门控） |
| 9 | [ulab-uiuc/GraphRouter](https://github.com/ulab-uiuc/GraphRouter) | Pushed 2025-12-30 | `[API]` | Query-Task-LLM 异构二部图神经网络零样本路由（核心实现已并入 `LLMRouter`） |
| 10 | [ulab-uiuc/Router-R1](https://github.com/ulab-uiuc/Router-R1) | Active (NeurIPS'25) | `[Web]` | 强化学习驱动的多轮思考路由与答案聚合器 |
| 11 | [stanford-futuredata/FrugalGPT](https://github.com/stanford-futuredata/FrugalGPT) | `2b23e6a` (2025-02-09) | `[Git][API]` | 序贯级联 + DeBERTa 评分器 + `scipy.optimize.brute` 分位数退出阈值搜索 |
| 12 | [withmartian/routerbench](https://github.com/withmartian/routerbench) | `cc67d10` (2024-06-12) | `[Git][API]` | 多模型路由评估基准与 AIQ（成本-质量曲线积分）指标实现 |
| 13 | [microsoft/best-route-llm](https://github.com/microsoft/best-route-llm) | `b13c5bd` (2026-10-19) | `[Git]` | HybridLLM & BEST-Route：联合搜索 `(model, best_of_n)` 测试时计算最优分配 |
| 14 | [automix-llm/automix](https://github.com/automix-llm/automix) | `531af3e` (2024-12-12) | `[Git][API]` | 少样本自校验级联 + `POMDPSimple` 离散信念状态最优升级求解 |
| 15 | [somerstep/CARROT](https://github.com/somerstep/CARROT) | `970436b` (2026-11-02) | `[Git][API]` | 双 `roberta-base` 同步预测准确率与输出长度 `output_tokens` 的线性帕累托路由器 |
| 16 | [Not-Diamond/RoRF](https://github.com/Not-Diamond/RoRF) | `a374ce8` (2024-09-24) | `[Git][API]` | Embedding + 4 分类随机森林覆盖预测（`Not-Diamond/notdiamond-python` 已于 2025-12 归档） |
| 17 | [algorithmicsuperintelligence/optillm](https://github.com/algorithmicsuperintelligence/optillm) | `66885d3` (2026-11-02) | `[Git][Web]` | `AutoThink` 复杂度动态限额思考 Token + 解码器 Layer 19 引导向量（Steering Vectors）注入 |
| 18 | [musistudio/claude-code-router](https://github.com/musistudio/claude-code-router) | `057abaa` (2026-10-10) | `[Git][API]` | `packages/core/src/config/config.ts`、`openrouter-discount-provider-router.ts`（`<CCR-SUBAGENT-MODEL>` + 缓存损失套利） |
| 19 | [BerriAI/litellm](https://github.com/BerriAI/litellm) | `f4f94cc` (2026-11-12) | `[Git][API]` | `litellm/router_strategy/complexity_router/`（`heuristic_v2`、Switchyard 9 规则、`CacheAwareRouting`）+ `adaptive_router/` |
| 20 | [maximhq/bifrost](https://github.com/maximhq/bifrost) | `6396397` (2026-11-12) | `[Git][API]` | 子代理请求头会话隔离 + 单调只升不降锁定 + `turnTierStore` 工具续跑零开销直通 |
| 21 | [ypollak2/llm-router](https://github.com/ypollak2/llm-router) | `b85bdbe` (2026-11-12) | `[Git][API]` | Claude/Codex 订阅制 Hook + MCP 6 步拦截链路由 |
| 22 | [Egham-7/adaptive](https://github.com/Egham-7/adaptive) & [nidhi-singh02/agent-router](https://github.com/nidhi-singh02/agent-router) | Active (2026) | `[Web]` | NVIDIA 复杂度分类代理 / 40% 安全配额感知 CLI 路由启动器 |

> **附：筛选边界对照仓库**（因不含内容难度路由或已归档而单独界定）：[router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI)、[Portkey-AI/gateway](https://github.com/Portkey-AI/gateway) (`669825c`)、[alibaba/higress](https://github.com/alibaba/higress) (`6726080`，`ai-intent` 具意图路由)、[tensorzero/tensorzero](https://github.com/tensorzero/tensorzero)（2026-06 归档）、[Not-Diamond/notdiamond-python](https://github.com/Not-Diamond/notdiamond-python)（2025-12 归档）。

---

### 3.2 赛道 B：缓存容量寻优（MRC）、KV/Prompt 调度、自动配置器与测速实验室（对应报告 `04`）

| 序号 | 开源仓库 | 原记录中的版本线索 | 核读入口与机制 |
| :---: | :--- | :--- | :--- |
| 1 | [facebook/CacheLib](https://github.com/facebook/CacheLib) | `77225740d2188eb6774ce4f5769b20de9fa3a89a` | `cachelib/allocator/MarginalHitsStrategy.h`、`LruTailAgeStrategy.h`、`PoolOptimizer.h`（在线尾部边际命中均衡） |
| 2 | [cacheMon/libCacheSim](https://github.com/cacheMon/libCacheSim) | `f7c85f8a538e62129e55e90c0ce4c0f605bd78ba` | `profilerLRU.c`（Olken Splay Tree MRC）、`cli_parser.c:461-485`（`[0.001..0.8]*WSS` 8 档扫频；核实不含 SHARDS） |
| 3 | [LMCache/LMCache](https://github.com/LMCache/LMCache) | `6a316c3e0174b693dbc9500c7a991477b9f20206` | `lmcache/tools/cache_simulator/simulator.py` & `trace_analyze.py`（本组样本中的离线 MRC 扫频顾问）+ `arc.py` |
| 4 | [kvcache-ai/Mooncake](https://github.com/kvcache-ai/Mooncake) | `6ca711276690d42b6aa195dccbb6cd6c14144772` | `count_min_sketch.h`（频次防污染准入）、`quota_manager.cpp`、`prefix_indexer.h`（四介质分层命中） |
| 5 | [vllm-project/aibrix](https://github.com/vllm-project/aibrix) | `a0d8137c8ec78778aa12dd633c742799879cad95` | `pkg/plugins/gateway/algorithms/load_balance.go`、`prefixcache/router.go`、`preble.go`（二次压力惩罚 + Preble 代价） |
| 6 | [vllm-project/production-stack](https://github.com/vllm-project/production-stack) | `405a2fa79992754d31b1eabb7183a540e` | `src/vllm_router/routers/routing_logic.py`（`KvawareRouter` 与无量纲 `LoadAwareRouter`） |
| 7 | [sgl-project/sglang](https://github.com/sgl-project/sglang) | `4f8b6867534c8a6632845353997555fca857dd63` | `sgl-model-gateway/src/policies/cache_aware.rs`（字符基数树 + `32 / 1.1x` 双阈值硬切换状态机） |
| 8 | [llm-d/llm-d-inference-scheduler](https://github.com/llm-d/llm-d-inference-scheduler) | `647ca7cacd74de4fcc94b4717b72161de0f683b9` | `pkg/epp/framework/plugins/scheduling/`（`MaxTTFTPenaltyMs=18000` 破粘闸门 + `3:2:2` 默认加权） |
| 9 | [llm-d/llm-d-kv-cache](https://github.com/llm-d/llm-d-kv-cache) | `822e6dd9c45963284f872b4766fbce2663` | `pkg/kvcache/kvblock_scorer.go`（`gpu=1.0, cpu=0.8` 连续前缀块介质折现计分） |
| 10 | [llm-d/llm-d](https://github.com/llm-d/llm-d) & [llm-d/llm-d-router](https://github.com/llm-d/llm-d-router) | `647ca7cacd...` (2026-11-13) | K8s 分布式推理控制面顶层架构与路由组件编排 |
| 11 | [kubernetes-sigs/gateway-api-inference-extension](https://github.com/kubernetes-sigs/gateway-api-inference-extension) | 原记录版本串长度异常，不作版本定位 | Kubernetes Gateway API 推理扩展 EPP 底层插件调度底座 |
| 12 | [zilliztech/GPTCache](https://github.com/zilliztech/GPTCache) | `a74ac654473f7bf4109118e85769451616e17a` | `gptcache/manager/eviction/memory_cache.py`（固定 `maxsize=1000` 封装，核实无自动容量建议算法） |
| 13 | [maximhq/bifrost-benchmarking](https://github.com/maximhq/bifrost-benchmarking) | `2026-10-13` `[Git]` | Bifrost 网关吞吐与语义缓存性能压测套件（配合 `bifrost/plugins/semanticcache/main.go`） |
| 14 | [montevive/autocache](https://github.com/montevive/autocache) | `d01556f6a17b124e5b5b96756117684dcb31d33c` | `internal/pricing/pricing.go`、`injector.go`（5m 第 2 次回本 vs 1h 第 3 次回本数学公式，静态 TTL） |
| 15 | [easmith/claude-cache-analyzer](https://github.com/easmith/claude-cache-analyzer) | `13fd8c95cef00890feb2b8a4a143454bc85d520f` | `claude_cache_analyzer/pricing.py`、`metrics.py`（Claude Code `.jsonl` 事后缓存账单审计评级） |
| 16 | [Encephos/prompt-cache-optimizer](https://github.com/Encephos/prompt-cache-optimizer) | `4b05f0e2ecf40bd3871ef5255329b0b7236fc8a4` | `prompt_cache_optimizer/core.py`（静态系统提示与 RAG 文档断点打包封装） |
| 17 | [vansye/prompt-cache-optimizer](https://github.com/vansye/prompt-cache-optimizer) | `9179b4960fde1b91d9853634e22e31b4d82575fd` | `optimizer/reorderer.py`、`aligner.py`（同角色块 SHA-256 稳定排序 + 最小缓存门槛 Padding 对齐） |
| 18 | [ai-dynamo/aiconfigurator](https://github.com/ai-dynamo/aiconfigurator) | `6cd6f9686ab7de6e8fb28171d6e350ee0aa4ed39` | `aic-core/rust/aiconfigurator-core/src/session.rs`、`pareto_analysis.py`（`--prefix` 非对称算子建模 + Agg/PD/AFD 帕累托） |
| 19 | [bentoml/llm-optimizer](https://github.com/bentoml/llm-optimizer) | `7be87d776e60a978516a2ae53e4a4ff745d04c20` | `memory_calculator.py`、`tuning/strategy.py`（显存/算力/带宽三瓶颈 Roofline 启动参数推演） |
| 20 | [microsoft/vidur](https://github.com/microsoft/vidur) | `abae7f63aa857300f5cdc6f5e0d27860cd24721b` | `capacity_search.py`、`generate_pareto_curves.py`（Ray 离散事件仿真二分搜极限 QPS 与 QPS/Dollar 帕累托） |
| 21 | [vllm-project/guidellm](https://github.com/vllm-project/guidellm) | `b15e11bf940ed7c539b4986d4bba8a75cad0c620` | `analysis/knee.py`、`profiles/goodput.py`（2-Segment OLS 四重护栏拐点识别 + Wilson 区间 Goodput） |
| 22 | [InseeFrLab/auto-tuning-vllm](https://github.com/InseeFrLab/auto-tuning-vllm) | `4c2220e67d80304884001bdc6dc916e8ad6c3762` | `study_controller.py`、`constraint.py`（Optuna TPE/BoTorch/NSGA-II 在线超参搜索） |
| 23 | [vllm-project/vllm](https://github.com/vllm-project/vllm) | `793a6351d624d7cb6c5d56b822f45e5b63f7efd0` | `benchmarks/auto_tune/auto_tune.sh`（官方网格扫描与 `--random-prefix-len` 自动调参脚本） |
| 24 | [kubernetes-sigs/inference-perf](https://github.com/kubernetes-sigs/inference-perf) | `68730a44ad930eaa5427791808fba661029183b6` | `inference_perf/analysis/analyze.py`（核实仅为阶梯并发压测与绘图器，无自动配置推荐模块） |
| 25 | [ekartashov/local-vllm-tuner](https://github.com/ekartashov/local-vllm-tuner) | `8307e9ec504d4eca1433a9e687e344ca1534157c` | `script.sh`（源码证伪：仅 6 行 `sleep 5` 占位空壳仓库） |
| 26 | [bytehola/llm-speed-test](https://github.com/bytehola/llm-speed-test) | `b403887450510f13c557fc02005b265dfeba038c` | `server/routes/benchmark.js`、`cpa-import.js`（多渠道 SSE 测速榜；核实仅支持导入 CPA YAML，不支持反向导出写回） |
| 27 | [pcm469768-droid/llm-api-speedtest](https://github.com/pcm469768-droid/llm-api-speedtest) | `408f76537d39500f545cf522002ccebfbec69072` | `internal/bench/bench.go`（零依赖 Go CLI 并发站点测速与 Markdown 榜单回写） |
| 28 | [Yoosu-L/llmapibenchmark](https://github.com/Yoosu-L/llmapibenchmark) | `f40c4e354972480c2bb089a11d63a9b6b6a2d136` | `cmd/benchmark.go`（分离 `Prompt TPS`、`Decode TPS` 与网络基线延迟） |
| 29 | [PicadoLabs/ai-model-router](https://github.com/PicadoLabs/ai-model-router) | `c6c5355917f750409919642515c01d8383441e7e` | `backend/app/router/thompson_sampling.py`、`scoring.py`（五维静态加权 + 持久化 Beta 后验 Thompson Sampling 闭环） |
| 30 | [rbadillap/ai-gateways-benchmark](https://github.com/rbadillap/ai-gateways-benchmark) | `849100a37a3320121cb2f8d35e4e5acb034edf58` | `bench.py`（`dns->tcp->tls->ttfb->ttft->e2e` 六阶段拆解 + 冷热连接隔离 + 交错轮询） |
| 31 | [OpenRouterTeam/benchmark-harness](https://github.com/OpenRouterTeam/benchmark-harness) | `1503e5406c25ad9744bf9d3ccb29e994baa718ae` | OpenRouter 官方 Bun 路由策略成本-智力基准评测框架 |

> 本表按专题列材料，不是一行一个独立仓库。赛道 B 原计 33 个检出目录，含跨赛道再次看的 Bifrost、Higress、Portkey，也有同一行并列的 llm-d 子仓。网页项与源码项分开；不能用行数或 22 + 33 推出独立仓库总数。

---

## 四、 核心学术论文与经典系统文献

### 4.1 大模型路由、级联与测试时计算寻优（LLM Routing & Test-Time Scaling）
1. **FrugalGPT**：Lingjiao Chen, Matei Zaharia, James Zou (Stanford). *"FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance."* `2023`, [arXiv:2305.05176](https://arxiv.org/abs/2305.05176).
2. **AutoMix**：Aman Madaan, Pranjal Aggarwal, et al. (CMU / Google). *"AutoMix: Automatically Mixing Language Models."* **NeurIPS 2024**, [arXiv:2310.12963](https://arxiv.org/abs/2310.12963).
3. **RouterBench**：Qitian Jason Hu, Jacob Bieker, et al. (Martian / UC Berkeley / UCSD). *"RouterBench: A Benchmark for Multi-LLM Routing System."* `2024`, [arXiv:2403.12031](https://arxiv.org/abs/2403.12031).
4. **HybridLLM**：Dujian Ding, Ankur Mallick, et al. (Microsoft / UBC). *"Hybrid LLM: Cost-Efficient and Quality-Aware Query Routing."* **ICLR 2024**, [arXiv:2404.14618](https://arxiv.org/abs/2404.14618).
5. **RouteLLM**：Isaac Ong, Amjad Almahairi, Vincent Wu, Wei-Lin Chiang, Tianhao Wu, Joseph E. Gonzalez, M Waleed Kadous, Ion Stoica (LMSYS / UC Berkeley / Anyscale). *"RouteLLM: Learning to Route LLMs with Preference Data."* **ICLR 2025**, [arXiv:2406.18665](https://arxiv.org/abs/2406.18665).
6. **GraphRouter**：Tao Feng, Yanzhen Shen, Jiaxuan You (UIUC). *"GraphRouter: A Graph-based Router for LLM Selections."* **ICLR 2025**, [arXiv:2410.03834](https://arxiv.org/abs/2410.03834).
7. **CARROT**：Seth Somerstep, Felipe Maia Polo, et al. *"CARROT: A Cost Aware Rate Optimal Router."* `2025`, [arXiv:2502.03261](https://arxiv.org/abs/2502.03261).
8. **Arch-Router**：Co Tran, Salman Paracha, Adil Hafeez, Shuguang Chen (Katanemo). *"Arch-Router: Aligning LLM Routing with Human Preferences."* `2025`, [arXiv:2506.16655](https://arxiv.org/abs/2506.16655).
9. **BEST-Route**：Dujian Ding, Ankur Mallick, Shaokun Zhang, Chi Wang, et al. (Microsoft). *"BEST-Route: Adaptive LLM Routing with Test-Time Optimal Compute."* **ICML 2025**, [arXiv:2506.22716](https://arxiv.org/abs/2506.22716).

### 4.2 缓存容量建模（MRC）、KV 缓存融合与推理集群调度仿真
10. **Mattson 栈距离定理**：R. L. Mattson, J. Gecsei, D. R. Slutz, I. L. Traiger (IBM). *"Evaluation techniques for storage hierarchies."* **IBM Systems Journal**, `1970`.
11. **Olken 伸展树单遍栈距离算法**：Frank Olken (Lawrence Berkeley Lab). *"Efficient Methods for Calculating the Success Function of Fixed Space Replacement Policies."* Tech Report `LBL-12370`, `1981`.（`libCacheSim` `profilerLRU.c` 核心理论底座）
12. **SHARDS**：Carl A. Waldspurger, Nohhyun Park, Alexander Garthwaite, Irfan Ahmad. *"Efficient MRC Construction with SHARDS."* **USENIX FAST '15**, `2015`.
13. **CounterStacks**：Jake Wires 等，*Characterizing Storage Workloads with Counter Stacks*。这里按题名作检索入口，不沿用未经出版记录核对的会议和年份。
14. **CacheLib**：Benjamin Berg, Daniel S. Berger, Sara McAllister, Isaac Grosof, et al. (Meta / CMU). *"The CacheLib Caching Engine: Design and Experiences at Scale."* **USENIX OSDI '20**, `2020`.
15. **Vidur**：Amey Agrawal, Nitin Kedia, Ashish Panwar, Jayashree Mohan, et al. (Microsoft Research / Georgia Tech). *"Vidur: A Large-Scale Simulation Framework For LLM Inference."* **MLSys 2024**, [arXiv:2405.05465](https://arxiv.org/abs/2405.05465).
16. **CacheBlend**：Jiayi Yao, Hanchen Li, Yuhan Liu, Siddhant Ray, et al. (UChicago). *"CacheBlend: Fast Large Language Model Serving for RAG with Cached Knowledge Fusion."* **EuroSys 2025 (Best Paper)**, [arXiv:2405.16444](https://arxiv.org/abs/2405.16444).
17. **Preble**：Vikranth Srivatsa, Zijian He, Reyna Abhyankar, Dongming Li, Yiying Zhang (UCSD WukLab). *"Preble: Efficient Distributed Prompt Scheduling for LLM Serving."* **ICLR 2025**, [arXiv:2407.00023](https://arxiv.org/abs/2407.00023).

---

## 五、 科普文章（`07`）与配图中的数据口径说明

看[公开图解](08-%E9%80%9A%E4%BF%97%E5%9B%BE%E8%A7%A3%EF%BC%9A%E5%A4%A7%E6%A8%A1%E5%9E%8B%E7%BD%91%E5%85%B3%E6%98%AF%E5%A6%82%E4%BD%95%E5%B8%AE%E4%BD%A0%E7%9C%81%E9%92%B1%E3%80%81%E6%8F%90%E9%80%9F%E5%8F%88%E4%B8%8D%E5%8F%98%E5%82%BB%E7%9A%84%EF%BC%9F.md)时，把公式里的常数和图上为了讲明白问题而选的数分开：

1. **版本常量、计价倍率及由它们推得的数**：
   - 缓存读写倍率 `0.1x`（读）、`1.25x`（5m 写）、`2.0x`（1h 写）及回本轮次（5m 第 2 次、1h 第 3 次）：取自 Anthropic 官方文档、`yetone/magpie` `internal/tune/tune.go:81-86` 与 `montevive/autocache` `internal/pricing/pricing.go`；
   - Magpie 调优与路由核心参数：压缩跌落识别 `20,000` tokens / `0.6x`、压缩后额外增长估计窗口 `reworkCalls = 10` 轮、搜索步长 `10,000` tokens、同成本宽容带 `sameCost = 0.03`（3%）、开头压缩指纹扫描长度 `compactLead = 2048B`、溢出比例 `95%`、Jev 置信度防抖门槛 `jevSure = 0.4`、温热保护门槛 `cacheWorth = 1024` tokens：全部逐行取自 `yetone/magpie`（Commit `70f4593e`）；
   - `magpie-speed-lab` 参数：中间字节流窗口 `80%`（`flow_ms`）、`agent_loop` 权重（首响 `45%` + 稳定 `25%`）、`interactive_chat` 权重（吞吐 `45%`）、`deep_reasoning` 权重（效能 `40%`）、`xhigh` 相对同模 `high/medium` 的 `1.25x` P90 降档告警阈值：全部逐行取自 `chefzejia/magpie-speed-lab`（Commit `75e91f5`）；
   - 外部开源参数：`LMCache` $\sqrt{2}$ 容量扫频步长、`vllm-project/semantic-router` `affinityMaxLambda = 0.14`、`claude-code-router` `5%` 最小净节省比例（`minSavingsRatio = 0.05`）、`Plano` & `LiteLLM` `3-Strike` 循环重试阈值：全部取自第 3 节对应源码文件。
2. **基于真实费率公式推演的教学示意算例（已标注“示意”或举例背景）**：
   - 图解第二节的 U 型示意线 `[142, 112, 92, 78, 74, 81, 98, 135]`，画的是“压缩太勤交返工税，窗口太大交重读税”。它不是任何人的七天轨迹；也不能从示意线断言真实成本函数是凸的。
   - [图解第四节的搬家账单](08-%E9%80%9A%E4%BF%97%E5%9B%BE%E8%A7%A3%EF%BC%9A%E5%A4%A7%E6%A8%A1%E5%9E%8B%E7%BD%91%E5%85%B3%E6%98%AF%E5%A6%82%E4%BD%95%E5%B8%AE%E4%BD%A0%E7%9C%81%E9%92%B1%E3%80%81%E6%8F%90%E9%80%9F%E5%8F%88%E4%B8%8D%E5%8F%98%E5%82%BB%E7%9A%84%EF%BC%9F.md)：假定 20 万 token，热读每百万 0.30 美元、冷读每百万 3 美元，该段输入分别花 0.06 与 0.60 美元；没有加输出、缓存写入或后续轮次。
