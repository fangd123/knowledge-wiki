---
title: "交叉核验与代码级审计记录（`verifier` 独立审计报告）"
date: 2026-11-13
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: Magpie与LLM网关配置调研
series_order: 6
tags:
  - AI技术与工程
  - Magpie与LLM网关配置调研
ai_assisted: true
status: published
---

# 交叉核验与代码级审计记录（`verifier` 独立审计报告）

> **审计角色**：`verifier`（独立代码级审计与交叉核验官）  
> **核验对象**：`00-调研规划与执行蓝图.md` 统筹下的四条专题调研报告（`01`～`04`）及对应过程日志（`logs/log-01`～`log-04`）  
> **独立核验底稿**：`log-05-verifier.md`  
> **终审裁定**：**四线全部通过（ALL 4 PASS）** — 可以正式进入 `coordinator` 最终交叉综合报告撰写阶段。

---

## 一、 审计总体结论与四线核验总览表

围绕“**Magpie 与开源 LLM 网关如何给出最佳缓存容量建议、问题难度自适应路由与全局最优配置**”核心问题，审计员对照 `yetone/magpie`、`chefzejia/magpie-speed-lab` 及 `oss-routers` / `oss-cache-opt` 克隆的开源仓库源码，辅以本地单测复现与 Web 检索交叉校对，完成全量独立审计。

| 调研线与作者 | 交付物（报告 & 过程日志） | 实证仓与关键基线 | 初审结论 | 打回修订与复审闭环 | 终审裁定 |
| :--- | :--- | :--- | :---: | :--- | :---: |
| **`01` (`magpie-core`)** | `01-Magpie网关核心机制深度拆解.md`<br>`log-01-magpie-core.md` | `yetone/magpie`<br>`70f4593e623e0847fcc2d869240565ce5f73a67a` | **FAIL** | 打回 3 项代码级偏差；作者在 `#12` 全部原位修正并补充源码索引 | **PASS (`#14`)** |
| **`02` (`speed-lab`)** | `02-Magpie-Speed-Lab深度拆解.md`<br>`log-02-speed-lab.md` | `chefzejia/magpie-speed-lab` `v0.3.0`<br>`75e91f590e6551fdc6173281a161613444120151`<br>（对照 `yetone/magpie` 同基线） | **PASS** | 一次通过；审计员独立运行 `npm test` **6/6** 通过（`207.76ms`） | **PASS (`#9`)** |
| **`03` (`oss-routers`)** | `03-开源生态调研A-智能难度路由与Agent网关.md`<br>`log-03-oss-routers.md` | 本地实勘 **22 个**开源仓库 + `[Web]` 检索交叉校验 | **FAIL** | `#18` 打回 3 项 Must-Fix + 3 项 Precision Polish；`#19` 逐项核改且全局 `rg` 清零 | **PASS (`#20`)** |
| **`04` (`oss-cache-opt`)** | `04-开源生态调研B-缓存容量寻优与自动配置器.md`<br>`log-04-oss-cache-opt.md` | 本地实勘 **33 个**开源仓库 + 3 项文献边界项目 | **FAIL** | `#16` 打回 2 项源码阈值/分支误差；作者同日修正并二次复核 | **PASS (`#17`)** |

**审计核心成果**：
- **拦截并清除了 8 处硬性偏差**（`01` 3 处、`04` 2 处、`03` 3 项 Must-Fix）及 **3 处精度瑕疵**（`03` Precision Polish），杜绝了公式误读、文件混淆与“过度吹捧开源静态配置文件”等幻觉风险进入最终报告。
- **独立复现与证伪双确认**：复现验证了 `02` 揭示的 4 项 `magpie-speed-lab` 上游源码漂移/正则瑕疵，并核实了 `04` 针对 `libCacheSim`（无 SHARDS）、`local-vllm-tuner`（6 行空壳）、`inference-perf`、`llm-speed-test`（无反向 CPA 导出）及 `03` 针对 `NadirClaw` 静态 YAML 的 5 项关键负向证伪。

---

## 二、 核心调研问题穿透度专项评估

对照任务书要求的三大核心穿透维度，逐线核查四份报告是否真正穿透至源码公式与真实架构机制：

### 2.1 `yetone/magpie` 最佳缓存大小建议与全局最优配置（`01` 报告穿透度：高）

1. **精准剥离命名陷阱，定位真实子系统**：
   - 成功排查并证伪了仓库中极易混淆的 `internal/capre/capre.go`（实为防 ReDoS 的 Capped Regexp 封装，非容量建议）与 `internal/gateway/advisor.go`（`advisor_20260301`，非缓存容量建议），将“最佳缓存大小 / 压缩窗口 / TTL 建议”精准锁定于 **Tuned for you** 四层闭环（`internal/sessions` $\to$ `internal/tune/tune.go` $\to$ `internal/gui/tune.go` + `internal/agent/tune.go` $\to$ `internal/gui/assets/context.js`）。
2. **彻底还原 Trace-Replay 数学与工程边界**：
   - 验证了非 Ghost Cache / 非 MRC 的核心方法论：**基于真实本地会话日志的全轨迹重放模拟器**——按文件 `(size, mod)` 增量缓存（`tuneKept` 避免 7 天 30 万条记录 ~500 MB 瞬时内存开销）、按 `Modified` 升序 + `FNV-64a` `msgHash` 去重 `--resume` 复制记录、在 `WeightsFor`（Claude `0.1 / 1.25 / 2.0 / 5.0` vs 非 Claude `0.1 / 1.0 / 1.0 / 8.0`）与 `pricedWeights` 真实路由单价折算后，以 `observe()` 提取 `Growth` / `After` / `Rework`（`dropFrom = 20000`, `dropShare = 0.6`, `reworkCalls = 10`）中位数。
   - 核实 `replayCompact` 与 `Compact`（Claude Code `Min: 100,000 / Max: 1,000,000 / Step: 10,000 / Reserve: 33,000`；Codex `Min: 50,000`；`minTimed = 3`、`sameCost = 0.03`、`codexAsyncAt` 异步压缩百分比转换）与 `replayTTL` / `CacheTTL`（主对话 `promptCacheTtl` 与子代理 `subagentPromptCacheTtl` 分离重放 `5m` vs `1h`）均逐常数吻合源码；同时准确覆盖 `Gateway: true` 下 `/api/tune/apply` 返回 `409 Conflict`、环境变量冲突锁 `Locked` 及 `stash`/`Undo` 回滚机制。
3. **难度分级与“省钱切模型 vs 击穿缓存”冲突消解穿透完整**：
   - 核实了三层难度路由器：`grouprule.go` / `rulecond.go` `Where` AST 与 `outgrown` 95% 防溢出重路由、`compacting.go`（10 类 Agent 压缩指纹 + `compactLead = 2048` 防误杀）、`desktop_subagent.go`（`x-claude-code-agent-id` + `hasTools` 隔离）、双语义分类器（`classify.go` 强制 `reasoning_effort = "none"` / `cutMiddle(text, 3000, 1000)` / `priorCap = 1000` / `classifyKeep = 10m` / `classifyRest = 30s`；`decide.go` `jevSure = 0.4` / `jevLevelled`）及 `grouptemplate.go` 6 套模板。
   - 核实了五重缓存护航机制（`affinity.go` `cacheWorth = 1024` / `coldAfter` / `hourCache` / `withLead` / `keptAgain`；`rules.go` `keepsEffort` 只升不降单向棘轮；`effort_update.go` `#617` 实验性 GPT-6 `configuration_update` 内联降级）、11 种内置 `Strategy`（`Smart`, `Pace`, `Fastest`, `Cheapest`, `LeastUsed`, `Weighted`, `FreeFirst`, `FreeExpiring`, `Drain`, `Ordered`, `Rotate`）与 `burstSpread` / `budget.Reserve` / 分级冷却矩阵（`creditLadder` / `rateRest` / `overloaded` / `restSlow`）。

### 2.2 `chefzejia/magpie-speed-lab` 测速模型、四维评分与配置建议边界（`02` 报告穿透度：高）

1. **彻底拆解双引擎与四维正交数学公式**：
   - 精准还原与 `yetone/magpie` 之间的四条耦合契约及针对 `connect-src 'none'` 沙箱 CSP（`internal/gui/pluginpage.go:236-243`）的 `127.0.0.1:0` 动态端口 + `MAGPIE_SPEED_LAB_BIND_DELAY_MS = 800` 防抖 + `page/session.js` 5 秒原子心跳握手。
   - 逐行核对被动遥测 `lib/telemetry.mjs`（利用 `flow_ms` 还原 `usage.go:768-773` `Decode()` 丢弃的隐藏思考纯工具轮速率、`HeldShare = 4` 突发过滤、`0 < tps <= 2000`、`decomposeLatency` 四阶段拆解）与主动探针 `lib/probe.mjs`（`code_stream` 与 `bug_reasoning`、与 `ttft.go:144-169` 同源的 10%–90% 字节 `flowMs` 算法、`Buffer.byteLength(text) / 3.6` 兜底）。
   - 核对 `lib/ranking.mjs` 四维单调映射公式（$S_{\text{resp}}(t) = \text{clamp}_{0,100}(\text{round}(105 e^{-t/11500}))$、$S_{\text{tps}}(v) = \text{clamp}_{0,100}(\text{round}(100(1 - e^{-v/55})))$、遥测/探针 $S_{\text{stab}}$ 与 $S_{\text{eff}}$、`EFFORT_DEPTH_SCORE` 刻意令 `high=96 > xhigh=94 > max=92 > ultra=90`）、4 大场景权重、`sampleFactor`（`>=12: 1.00`, `5-11: 0.96`, `<5: 0.90`）及 `buildInsights()` `totalP90 > moderate.totalP90 * 1.25` 降档预警。
2. **独立单测复现与负向边界澄清到位**：
   - 在本地克隆仓运行 `npm test`，6 项测试全通过（`207.76ms`）。
   - 报告没有夸大功能，而是明确界定了其 **“只测不写、不计真实美元成本、不算缓存大小/TTL”** 的真实边界，并独立查出 4 处上游源码瑕疵/前后端漂移（均已复核属实，详见第 3.2 节）。

### 2.3 开源智能路由与 Agent 网关生态筛选精度（`03` 报告穿透度：中上 → 修订后高）

- 共源码审计 **22 个**仓库，有效剔除了仅做静态轮询转发的普通 API 代理；经 `#18`/`#19` 修订后，围绕三大主线形成清晰技术谱系：
  1. **通用与学术路由器及阈值校准**：`lm-sys/RouteLLM`（矩阵分解 / `sw_ranking` / `BERTRouter` 与 `causal_llm` 拆分 / `calibrate_threshold.py` 分位数导出）、`LLMRouter`（`fusion_gate` / `graphrouter`）、`FrugalGPT`、`routerbench`（`AIQ.py`）、`best-route-llm`（`hybrid_llm`）、`automix`、`CARROT`、`RoRF`、`optillm`（`autothink`）。
  2. **多信号与会话亲和策略网关**：`vllm-project/semantic-router`（20 类信号矩阵、`affinityMaxLambda = 0.14` 缓存亲和与 6 种推理传输映射）与 `katanemo/plano`（`orchestrator.rs` / `session_router.rs`）。
  3. **Coding Agent 与通用路由层**：`musistudio/claude-code-router`（`config.ts` / `config-compiler.ts` / `policy-engine.ts` / v1 免费小模型分发 4 角色）、`NadirClaw`（`Laya`/`Wide&Deep`/`cascade` 与静态 YAML 档位边界）、`ClawRouter` + `router-core`、`BerriAI/litellm`（`complexity_router`, `tier_predictor`, `capability_classifier`, `stall_detector`, `cache_aware_routing`, `adaptive_router/bandit.py`, `lar1_routing.py`）、`maximhq/bifrost`、`higress`、`portkey-gateway`、`ypollak2/llm-router`（6 步拦截链）。
- 对已归档项目（如 `notdiamond-python` 于 2025-12-11 归档、`tensorzero` 归档时间等）与部分高频 `[Web]` 仓库通过联网检索交叉确认，未将已归档方案包装为活跃前沿方案。

### 2.4 开源缓存容量寻优与自动配置器覆盖度（`04` 报告穿透度：高）

- 共源码实勘 **33 个** GitHub 仓库（另附 3 项明确标注文献边界的方法），完整打通了五层架构体系：
  1. **通用存储 MRC 基座**：`cacheMon/libCacheSim`（Olken Splay Tree 单遍 Mattson 栈距离 + `[0.001..0.8] * WSS` 扫频，且实地 `rg -i "shards"` 证伪其未实现 SHARDS）与 `facebook/CacheLib`（在线 `MarginalHitsStrategy` / `LruTailAgeStrategy`）。
  2. **推理层 KV 缓存容量与感知路由**：`LMCache/LMCache`（生态唯一落地离线 Hit Ratio Curve 扫频器 `lmcache/tools/cache_simulator` + `ARC` + `QuotaManager`）、`kvcache-ai/Mooncake`、`vllm-project/aibrix`、`vllm-project/production-stack`、`sgl-project/sglang`（`sgl-model-gateway`）、`llm-d`。
  3. **云端 Prompt Caching 与语义缓存**：`montevive/autocache`（Anthropic `1.25x / 2.0x / 0.1x` 盈亏平衡数学，静态硬编码 `1h`/`5m`）、`easmith/claude-cache-analyzer`、`vansye/prompt-cache-optimizer`、`Encephos/prompt-cache-optimizer` 与 `GPTCache`/`Bifrost`/`Higress`/`Portkey`/`Kong` 盲区剖析。
  4. **工作负载驱动 Auto-Configurator**：`bentoml/llm-optimizer`、`ai-dynamo/aiconfigurator`（Silicon Profiling 库 + Rust 核 + `--prefix` 非对称建模）、`microsoft/vidur`、`vllm-project/guidellm`、`InseeFrLab/auto-tuning-vllm`。
  5. **多 API 供应商测速与在线选路**：`bytehola/llm-speed-test`、`pcm469768-droid/llm-api-speedtest`、`Yoosu-L/llmapibenchmark`、`rbadillap/ai-gateways-benchmark` 及测速生态中唯一打通在线学习探索闭环的 `PicadoLabs/ai-model-router`（Beta-Bernoulli Thompson Sampling）。

---

## 三、 各报告重大代码级纠偏复盘与实证档案

以下记录本次独立审计中**实际打回并推动原作者修正的 8 项实质偏差与 3 项精度打磨**，以及在 **`02` 复核确认的 4 项上游源码瑕疵**。

### 3.1 打回修正归档表（`01`、`03`、`04`）

| 所属报告 | 纠偏等级 | 初稿原始偏差 | 审计核查到的源码实证 | 修订结果与复审定位 |
| :--- | :---: | :--- | :--- | :--- |
| **`01`** | **实质偏差 1** | 将 `claude_cache.go` 简化为“消息历史 `>= 64 KiB` 即切 `claude -p`” | `internal/gateway/claude_cache.go:52,57-68,77-79`（`marksPrintAt`, `markedHistory`, `keptByPrint`）限定**仅针对携带客户端 `cache_control` 标记的 `autoModeClassifier(req)==true` 安全分类器请求**，且按截至最后缓存标记点的字节数判别；普通会话留在交互模式 | 已精准补全前置触发条件，见 `01` L143 与 `log-01`（`#12` 修正，`#14` PASS） |
| **`01`** | **实质偏差 2** | `classify.go` 截断比例与 `classifyPrompt` 细节表述不够精确 | `classify.go:264-268` 为 `cutMiddle(text, 3000, 1000)`；`L296,306-321` 受 `priorCap=1000` 约束，上一条用户消息按 3:1 取 `750/250` runes、助手末尾取 `1000` runes；`classifyKeep=10m`、`classifyRest=30s`、`classifyEffort` 注入 `reasoning_effort="none"` | 已补录 `classifyPrompt`/`effortPrompt` 原文与精确截断参数，见 `01` L175-185 |
| **`01`** | **实质偏差 3** | 部分符号与战略总数需逐一核对（`openAIUsage`、`affine`、内置 `Strategy` 数量、`budget.Reserve` 式子） | 实核 `chattidy.go:209` (`openAIUsage`)、`affinity.go:569` (`affine`) / `L1550` (`keptAgain`)、`effort_update.go:92` (`#617`)、`internal/routes/strategies.go` 共 **11** 种内置策略、`budget.go:560,625-651` (`max(body,0)/4 + s.output/s.calls`) | 全部符号、行号与 11 策略矩阵已核齐，见 `01` L263-282, L321-329 |
| **`04`** | **实质偏差 4** | 将 `cacheMon/libCacheSim` 自动扫频比例区间错记为 `0.0001..0.3 * WSS` | `libCacheSim/bin/cachesim/cli_parser.c:461-485`（`set_cache_size`）硬编码 8 档 C 数组 `{0.001, 0.003, 0.01, 0.03, 0.1, 0.2, 0.4, 0.8}`，即 **`[0.001..0.8] * WSS`**，辅以 `wss_obj * s[i] > 4` / `>= 100` 护栏 | 已修改为真实 C 源码 8 档数组与护栏条件，见 `04` L49, L330（`#17` PASS） |
| **`04`** | **实质偏差 5** | 将 `microsoft/vidur` `capacity_search.py` 的 SLO 与收缩条件混淆下游脚本，写反/漏记阈值逻辑 | `vidur/config_optimizer/config_explorer/capacity_search.py:69-85, 151-169` 实为针对 **`request_scheduling_delay` 单分位数 SLO**（`scheduling_delay_slo_quantile <= scheduling_delay_slo_value`）搜最大 QPS，上界收缩按绝对阈值 `if scheduling_delay > 500: right = qps / 2` / 不可达分支 `elif scheduling_delay > 1000`；TTFT/TBT SLO 位于下游 `generate_pareto_curves.py` | 已清晰剥离调度延迟 SLO 与下游帕累托脚本并指出上游不可达分支，见 `04` L266, L342（`#17` PASS） |
| **`03`** | **Must-Fix 1** | `RouteLLM` `sw_ranking` 公式失真，且将 `bert` 与 `causal_llm` 的 Prompt/阈值混为一谈，未对齐 `calibrate_threshold.py` | `routellm/routers/routers.py:173-175` 为 $w_i = 10 \cdot 10^{\text{sim}_i / \max \text{sim}}$；`BERTRouter`（`L106-130`）为 `num_labels=3` & `1 - sum(softmax_scores[-2:])`；`causal_llm` 才用 `system_ft_v5.txt` 1-5 评分与 `score_threshold=4`（`causal_llm/model.py:122-127`）；`calibrate_threshold.py:55` 用 `quantile(q=1 - args.strong_model_pct)` | 已拆分三套实现并修正全部公式，见 `03` L43-46, L174 & `log-03` L74-81（`#20` PASS） |
| **`03`** | **Must-Fix 2** | 误将 `NadirClaw` 的 `nadirclaw/tier_config/selector.py` 吹捧为“基于质量与成本矩阵的自动配置推荐器”且表格前后矛盾 | `selector.py:1-13, 113-163` 确认其 YAML 档位全部**静态预置**，`_assign_flat(score)` 仅是在运行时将难度分数排名 `score_rank` 与 `cost_rank` 混合选取初始节点，且不读 `tier.quality` | 已定性为 **不自动生成配置** 并下调评级至 `弱-中（静态 YAML 预置档位）`，消除表内矛盾，见 `03` L94, L178, L205 |
| **`03`** | **Must-Fix 3** | `claude-code-router` 配置文件路径错误，且将 v1 初版表述错写为“多模型投票/不同职责混写” | `packages/core/src/config/config.ts:121-127` 含 `REMOVED_LEGACY_ROUTER_RULE_IDS`；`blog/zh/项目初衷及原理.md:85` 明确 v1 初版为“用 1 个免费小模型充当分发器路由到 4 个角色模型” | 已修正路径并严格对齐作者博客原文，见 `03` L17, L122, L128 & `log-03` L59, L159-160 |
| **`03`** | **Precision Polish (3 项)** | 1) `semantic-router` `affinityMaxLambda` 错写成 0.15/其他值与信号类数不准；<br>2) `bifrost` 漏列关键分类器文件与弃用注释；<br>3) `ypollak2/llm-router` 拦截链步数混用 | 1) `cache_affinity.go:46` 实为 `0.14`，`engine.go:59-83` + `pkg/config` 共 **20 类信号**；<br>2) `complexity/semanticclassifier.go`、`complexity/llmclassifier.go` 及 `analyzer.go:9-13` / `config.go:81,111` 弃用说明；<br>3) `hooks/auto-route.py:5-11` 为 **6 步拦截链** | 三项均在 `#19` 同批次修正且全局 `rg` 无残余旧值，见 `03` L21, L54-55, L62, L156, L163, L175, L188 |

### 3.2 `02` 报告独立交叉验证的上游源码瑕疵清单（非报告错误，而是报告真实揭露的插件缺陷）

审计员逐一核验源码，确认 `02` 报告中主动曝光的 4 项 `chefzejia/magpie-speed-lab` 源码内在缺陷/漂移全部为真实发现：
1. **`lib/telemetry.mjs:120-123` 注释-代码漂移**：第 120 行注释写 `18ms/token, cap 75%`，但运行时代码实际执行 `22 * reasoning` 与 `0.78` 上限。
2. **`lib/telemetry.mjs:194` vs `387-415` 慢请求门槛漂移**：第 194 行注释遗留旧阈值 `15000 / 12000`，实际判定与 UI 均为 `ms >= 18_000 || ttft_ms >= 13_000`。
3. **`lib/probe.mjs:351-352` 正则假阳性**：`bug_reasoning` 负值校验 `hitNeg` 分支包含 `/amount/`，而题目函数签名本身即为 `transfer(db, from, to, amount)`，模型仅复述参数名即会误得分。
4. **`lib/ranking.mjs:192-194` vs `ui/app.js:333-338` 前后端排序不一致**：服务端按 `sampleFactor` 折扣排序，而浏览器端 `ui/app.js` 额外硬编码 `calls >= 8` 置顶排序，导致 `calls = 7` 的高分模型 CLI 榜首而在 UI 被压后。

---

## 四、 跨报告一致性交叉核验矩阵

跨报告核对旨在确保最终交付至 `06` 号汇总报告时，各部分在基准版本、数学常数、能力归属与技术边界上严丝合缝：

| 交叉核验对 | 核验焦点 | 一致性证据链 | 核验结论 |
| :--- | :--- | :--- | :---: |
| **`01` vs `02`** | `yetone/magpie` 基线版本与原生 `Fastest` 路由机制 | 两者均锁定 Commit `70f4593e623e0847fcc2d869240565ce5f73a67a`；`01` L270 与 `02` L30 对 `speed.go` / `fastest.go` 的常数与公式完全一致（`speedHalfLife = 20m`、`speedStale = 1h`、`RefTokens = 200`、`ProbeEvery = 5`、`alike = 1.1`、`Est = ttft + 200 * 1000 / tps`）。 | **完全一致 (PASS)** |
| **`01` vs `02`** | `usage` 遥测字段消费与插件扩展点边界 | `01` 拆解了本体 `Tuned for you`（`internal/tune/tune.go`）、`11` 种 `Strategy`（`strategies.go`）及 `usage.go` / `chattidy.go` 记账字段（`cache_write`, `cache_write_1h`, `cache_unknown`）；`02` 准确界定 `magpie-speed-lab` 通过 `flow_ms` 弥补原生 `Decode()`（`usage.go:768-773`）在隐藏思考工具轮速率丢失的同时，**尚未消费** `cache_unknown` / `cache_write` / `cache_write_1h` / `cost`，亦不提供缓存建议或写回 P3 `Routes` / P4 JS `strategies`，并依托 `internal/gui/pluginpage.go:236-243`（`connect-src 'none'`）解释动态握手与 `magpie web` 限制。 | **严密互补 (PASS)** |
| **`01` vs `04`** | Anthropic `5m` vs `1h` TTL 定价倍率与“开源空白填补”互证 | `01` L61-65（`WeightsFor`）与 `04` L193（`autocache` `EstimateBreakpointROI`）独立推导出**完全一致**的 Anthropic 相对倍率：$P_{\text{write,5m}} = 1.25\times$、$P_{\text{write,1h}} = 2.0\times$、$P_{\text{read}} = 0.1\times$。<br>**关键发现**：`04`（明确排除 Magpie 生态）实证 `autocache` / `claude-cache-analyzer` / `vansye` 等开源 Prompt Cache 代理均未实现“基于到达间隔分布动态求解 5m vs 1h”；而 `01` 对 `yetone/magpie` `replayTTL`（按 `Quick / Pause / Away` 间隔分布重放并拆分主对话与子代理 TTL、`sameCost = 3%` 门槛）的源码剖析，恰恰证明 Magpie 本体率先填补了 `04` 指出的该项开源断层。 | **跨生态互证 (PASS)** |
| **`01`/`02` vs `04`** | “缓存大小建议”双范式边界与测速进化方向 | `01` 界定单机编码代理的“最佳缓存大小建议”本质是压缩窗口与 TTL 重放寻优；`04` 界定底层存储与分布式推理层的 MRC 扫频（`libCacheSim`, `LMCache` $\sqrt{2}$ 步进）与显存算力寻优（`aiconfigurator`, `vidur`, `llm-optimizer`, `guidellm`）；`02` 的 `magpie-speed-lab` 与 `04` 第五部分（`llm-speed-test`, `ai-gateways-benchmark`, `PicadoLabs/ai-model-router` Thompson Sampling）正好揭示从“静态测速榜单”到“在线探索闭环”的演进方向。 | **术语零混用 (PASS)** |
| **`01` vs `03`** | 难度分级、推理预算控制与缓存防击穿趋同性 | `01` 拆解的 `compacting.go`（10 类 Agent 指纹、`compactLead = 2048`）、`classify.go`、`decide.go`（`jevSure = 0.4`）、`keepsEffort` 单向棘轮与 `effort_update.go` 与 `03` 覆盖的 `RouteLLM`、`semantic-router`、`plano`、`claude-code-router`、`NadirClaw`、`litellm` 等生态方案各自归位，无张冠李戴。 | **完全一致 (PASS)** |

---

## 五、 验证边界、工具约束与残余风险声明

为确保最终汇总结论严守实证边界，特作如下声明：

1. **静态源码与单元测试为主，未运行真实商业 API 长周期收费压测**：
   - `02`（`chefzejia/magpie-speed-lab`）的 `npm test`（6/6 用例）在本地直接运行验证通过（`207.76ms`）；其余 Go / Python / Rust 仓库（`yetone/magpie` 及 `03`/`04` 仓）以本地 Git 检出特定 Commit 后的代码静态审计、跨文件全文检索（`rg`）与数学公式推导验证为主，未向线上付费商业模型发起真金白银长周期压测以实证端到端省钱比率。
2. **GitHub API 匿名限流下的 `[Web]` 元数据检索降级**：
   - 核验过程向未认证 `api.github.com` 发起批量请求时遇到 `403` 速率限制，因此 `03`/`04` 中部分标注 `[Web]` 的外部仓库元数据（如 `NadirClaw`、`optillm`、`CLIProxyAPI`、`Router-R1` 约略 Star 量级及 `notdiamond-python` / `tensorzero` 归档时点）改由 `search_web` 进行交叉检索核实；其中 Star 数与归档状态随时间持续变化，仅代表调研当日公开快照。
3. **依赖厂商后端行为注释或外部文献的局部边界**：
   - `01` 提及的 `MAGPIE_EFFORT_UPDATES=on`（GPT-6 Responses `configuration_update` 上升衰减 `#617` 问题）基于 `yetone/magpie` 源码注释中的实测结论记录，未在闭源模型服务端黑盒复测；`04` 中明确的 3 项未克隆方案（如 `SHARDS` / `CounterStacks` 原论文算法、`newapi-smart-router` 文档核实部分）在原文均有明确文献与官方文档标注，未混同为本地源码级核查成果。

**最终审计放行结论**：`01`、`02`、`03`、`04` 及对应过程日志 `logs/log-01`～`log-04` 全部符合 `00` 号规划质量标准与源码级真实性规范，`verifier` 正式签发 **PASS**，建议 `@coordinator` 据此开展全景对比与最终总结报告 `06` 的撰写。
