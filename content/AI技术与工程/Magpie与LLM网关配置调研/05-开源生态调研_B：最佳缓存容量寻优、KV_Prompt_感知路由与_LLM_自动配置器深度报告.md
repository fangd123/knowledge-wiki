---
title: "开源生态调研 B：最佳缓存容量寻优、KV/Prompt 感知路由与 LLM 自动配置器深度报告"
date: 2026-11-13
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
series: Magpie与LLM网关配置调研
series_order: 5
tags:
  - AI技术与工程
  - Magpie与LLM网关配置调研
ai_assisted: true
status: published
---

# 开源生态调研 B：最佳缓存容量寻优、KV/Prompt 感知路由与 LLM 自动配置器深度报告

> **调研角色**：`oss-cache-opt`  
> **调研范围**：除 Magpie 生态（`yetone/magpie` 与 `chefzejia/magpie-speed-lab`）以外，开源社区在「最佳缓存大小建议（Optimal Cache Sizing / MRC）」「KV / Prompt / Semantic 缓存感知路由与参数寻优」「工作负载驱动自动配置推荐器（Auto-Configurator）」及「多 API 供应商实测选优实验室（类 Speed Lab）」四大方向的工程实现与数学方法。  
> **怎么读这张地图**：原调研记录赛道 B 检视了 33 个仓库，另有文献与文档项；版本线索与其局限见[公开资料台账](09-%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99%E4%B8%8E%E6%95%B0%E6%8D%AE%E6%9D%A5%E6%BA%90%E6%80%BB%E8%A1%A8%EF%BC%88References_%26_Data_Provenance%EF%BC%89.md)。按实现能输出哪一种曲线来分层，并未在集群测得最优容量。

---

## 执行摘要（Executive Summary）

本次横跨底层存储缓存、云原生推理引擎、API 语义网关与基准压测调优四大领域的源码级普查，揭示了以下四条决定工程架构选型的核心结论：

1. **“能算出最佳缓存容量（MRC/WSS 寻优）”的工具在 LLM 网关与推理生态中极为稀缺**：
   - 在通用存储领域，`cacheMon/libCacheSim`（单遍 $O(N\log M)$ Olken Splay Tree Mattson 栈距离）与 `facebook/CacheLib`（基于 LRU 尾部 Slab 边际命中的 `MarginalHitsStrategy` / `PoolOptimizer`）已将离线精确 MRC 与在线边际容量平衡做到成熟。
   - 但下沉到 LLM 生态后，**绝大多数语义缓存（`GPTCache`、`Bifrost`、`Higress`、`Portkey`、`Kong`）与推理调度器（`aibrix`、`llm-d`、`production-stack`、`sgl-model-gateway`）均不具备“推荐最佳缓存大小”的算法**——它们要么要求人工硬编码 `maxsize`/`TTL`，要么仅在给定容量下执行被动驱逐。
   - **在本次所看推理层版本中，能直接找到离线扫频器的是 `LMCache/LMCache`**：其内置 `lmcache/tools/cache_simulator`（`simulator.py` 与 `trace_analyze.py`）直接回放生产块哈希日志或 OTLP Token Trace，在对数或半倍频 $\sqrt{2}$ 容量网格上扫频生成**前缀 + CacheBlend 命中率曲线（Hit Ratio Curve）**，是这一组中接近“KV 缓存容量建议顾问”的落地实现。

2. **KV-Cache 感知路由（Cache Affinity vs Load Balancing）已演化出四条清晰的数学流派**：
   - **双阈值硬切换流派**（`sgl-project/sglang` `sgl-model-gateway`）：用绝对/相对队列差阈值触发最小负载降级，其余时间按前缀字符匹配率 `> 0.5` 粘滞路由。
   - **方差护栏 + 二次压力惩罚 + Preble 代价流派**（`vllm-project/aibrix`）：引入标准差与 `1.5x` 上限过滤防热点打死单机，用 `1 + 2*(1 - kvFree)^2` 对高水位 KV 施压，并在 `5:4` 混权与 `prefix-cache-preble` 成本模型之间协同调度。
   - **无量纲线性效用流派**（`vllm-project/production-stack`）：`LoadAwareRouter` 用 $\text{prefixRatio} - \beta \cdot \text{relativeLoad}$ 取代刚性 Token 截断阈值。
   - **分层介质权重 + TTFT 破粘闸门流派**（`llm-d/llm-d-inference-scheduler` & `llm-d-kv-cache`）：显式区分 `gpu=1.0` 与 `cpu=0.8` 命中价值，并通过 `MaxTTFTPenaltyMs (18000ms)` 动态破除长尾卡顿节点的缓存粘性。

3. **云端 Prompt Caching 的 `5m` vs `1h` TTL 寻优仍停留在静态计费规则层**：
   - 按该版本使用的 Anthropic 相对价格，与每次不缓存相比（5m 写入 `1.25x`、1h 写入 `2.0x`、读取 `0.1x`），`montevive/autocache` 给出了精确的整数盈亏平衡点：**`5m` TTL 在第 2 次请求（1 次命中）即回本并净省 32.5%；`1h` TTL 必须等到第 3 次请求（2 次命中）才开始净赚，仅命中 1 次反而多亏 5%**。
   - 然而，包括 `autocache`、`claude-cache-analyzer`、`vansye/prompt-cache-optimizer` 在内的开源项目均**尚未实现**根据真实会话请求间隔（Inter-arrival Time）分布动态求解每段前缀该选 `5m` 还是 `1h` 的在线优化器。

4. **LLM 自动最优配置器（Auto-Configurator）呈现四阶方法论演进，而多商测速生态多欠缺闭环寻优**：
   - 从静态 Roofline 显存与算力三瓶颈推演（`bentoml/llm-optimizer`）、“实测算子数据库 + Rust 三遍步骤模型 + 帕累托筛选”与 `--prefix` 非对称计算建模（`ai-dynamo/aiconfigurator`）、离散事件仿真二分搜极限 QPS（`microsoft/vidur`），到实机 2-Segment OLS 拐点与 Wilson 置信 Goodput 探测（`vllm-project/guidellm`）以及 Optuna NSGA-II 黑盒搜索（`InseeFrLab/auto-tuning-vllm`），形成完整技术栈。
   - 类 `magpie-speed-lab` 的开源测速工具（`bytehola/llm-speed-test`、`llm-api-speedtest`、`llmapibenchmark`、`ai-gateways-benchmark`）普遍擅长 SSE TTFT/TPS 与网络握手分层观测，但多数停留在人工看榜阶段；其中 `PicadoLabs/ai-model-router` 等引入了基于 Beta 后验的 **Thompson Sampling** 实现在线测速-质量探索-路由闭环。

---

## 第一部分：底层缓存容量建模与“最佳缓存大小建议”基础算法

要从工程上回答“应当分配多大的缓存空间（VRAM / DRAM / SSD 或条目数 `maxsize`）”，经典系统工程围绕两个核心概念构建数学工具：**工作集大小（Working Set Size, WSS）** 与 **未命中率曲线（Miss Ratio Curve, MRC）**。

### 1.1 经典精确 MRC 引擎：`cacheMon/libCacheSim`

`cacheMon/libCacheSim`（Commit `f7c85f8a538e62129e55e90c0ce4c0f605bd78ba`）是系统存储与缓存领域最具代表性的开源 Trace 驱动模拟器与 MRC 分析基座。

1. **单遍精确 LRU MRC 算法（Olken Splay Tree Mattson 栈距离）**：
   - 在 `libCacheSim/profiler/profilerLRU.c` 与 `dist.c` 中，`libCacheSim` 通过基于伸展树（Splay Tree）的 Olken 算法函数 `get_stack_dist_add_req` 顺序扫描请求访问轨迹。
   - 对于每次访问对象的栈距离 $d$（即自上次访问该对象以来中间出现的**不重复对象字节数或对象个数**），算法直接将命中计数累加至直方图桶 `hit_count_array[stack_dist + 1]`。
   - 一遍扫描（时间复杂度 $O(N \log M)$，$M$ 为唯一对象数）结束后，通过对 `hit_count_array` 执行前缀和，即可一次性输出从容量 $0$ 到全工作集上限的完整精确 `miss_ratio_array[0..size]`：
     $$\text{MRC}_{\text{LRU}}(C) = 1 - \frac{1}{N} \sum_{d \le C} \text{hit\_count}[d]$$
2. **多策略并行模拟 Sweep 与工作集探测**：
   - 由于 FIFO、LFU、ARC、S3-FIFO 等非栈算法（non-inclusion property）无法通过单遍栈距离推导多容量点的命中率，`libCacheSim/profiler/simulator.c` 采用多线程并行方式，针对指定容量列表同步运行多个独立淘汰实例。
   - 配套命令行工具在 `libCacheSim/bin/cachesim/cli_parser.c:461-485`（`set_cache_size`）中调用 `libCacheSim/bin/cli_reader_utils.c:245`（`cal_working_set_size`）自动统计 Trace 的唯一工作集规模（WSS），真实 C 源码硬编码 8 档比例数组 `{0.001, 0.003, 0.01, 0.03, 0.1, 0.2, 0.4, 0.8}`（即在 **`[0.001 .. 0.8] * WSS`** 区间内结合 `wss_obj * s[i] > 4` 或 `>= 100` 最小对象数护栏采样扫频绘制缓存收益拐点）。
3. **代码级负向边界证伪（关于 SHARDS）**：
   - 外部文献常将 `libCacheSim` 与 SHARDS 混为一谈。但经对该本地仓 `rg -i "shards"` 全文核查，确认当前源码树**并未实现** Waldspurger 的 SHARDS 空间采样 MRC 算法（仓内 `Mithril.c` 出现的 `shards` 仅用于预取哈希分表尺寸）。

### 1.2 $O(1)$ 有界内存在线 MRC 文献基石：SHARDS 与 CounterStacks

在线记录每个对象的复用距离很费内存。以下两条路线取自论文摘要的算法描述，不是本文运行过的实现。

- **SHARDS（Waldspurger et al., USENIX FAST '15）**：采用确定性均匀随机空间哈希采样（Spatially Hashed Approximate Reuse Distance Sampling），仅对满足 `hash(key) < T` 的代表性子集对象维护紧凑栈距离结构（并动态下调采样率以把内存死锁在约 ~1 MB 量级），从而以极低开销在线实时产出近乎无偏的 MRC。
- **CounterStacks（Wires 等）**：利用一组串联递增容量视角或时间窗的 HyperLogLog / 布隆过滤器概率基数计数器（Counter Stacks），通过估算不同时刻之间唯一元素的基数增量来近似复用距离直方图，用极小的固定内存预算支撑多租户并行在线工作负载画像。

### 1.3 生产级在线边际效用再平衡：`facebook/CacheLib`

不同于先离线画出完整 MRC 再静态配置，`facebook/CacheLib`（Commit `77225740d2188eb6774ce4f5769b20de9fa3a89a`）面向多池、多 Slab 尺寸的混布内存系统，实现了**在线闭环容量微调器**（位于 `cachelib/allocator/`）：

1. **尾部边际命中增益法（`MarginalHitsStrategy.h/.cpp`、`MarginalHitsOptimizeStrategy.h`）**：
   - 将每个 Allocation Class 或内存池（`PoolOptimizer.h`）末尾若干 Slab 设为观测哨兵，利用 `getMarginalHits` 统计该尾部区间的近期命中次数——直观等价于“若此处增加或减少 1 个 Slab（$\Delta C$），整池命中数将增减多少（$\Delta\text{Hits}/\Delta\text{Slabs}$）”。
   - 通过指数移动平均平滑因子 `movingAverageParam = 0.3` 抑制瞬时毛刺后对各池排秩，驱动 `PoolRebalancer` / `PoolOptimizer` 从边际命中率最低的池摘取空闲 Slab 移交至边际命中率最高的池，令多池在总容量约束下自动收敛到导数相等的帕累托最优分配。
2. **LRU 尾龄均衡策略（`LruTailAgeStrategy.h`）**：
   - 比较各 Slab Class 的投影尾部留存时长（Tail Age），仅当最老池与最年轻池的尾龄差距同时跨过相对比值阈值 `tailAgeDifferenceRatio = 0.25` 与绝对差距门槛 `minTailAgeDifference = 100` 时才触发迁移，避免微小抖动引发频繁拷贝颠簸。

### 1.4 从传统 MRC 向大模型 KV/Prefix 缓存迁移的三大范式陷阱

为何不能直接将 `CacheLib` 或 `libCacheSim` 原封不动塞入 LLM 网关给出“KV 缓存大小建议”？根源在于三点结构性失配：

| 维度 | 传统对象/块缓存（`libCacheSim`/`CacheLib`） | 大模型 Prefix KV / Prompt 缓存 | 对容量建议算法的直接影响 |
| :--- | :--- | :--- | :--- |
| **命中独立性** | 各 Block/Key 彼此独立，第 $k$ 块命中不依赖第 $k-1$ 块 | **连续前缀依赖**：Attention 计算与标准前缀匹配要求从 Token 0 起连续命中；若第 $i$ 个 Block 被驱逐，其后所有 $j > i$ 即使仍在内存也退化为未命中（Non-Prefix 计算重做或仅 CacheBlend 可抢救） | 经典独立栈距离模型会**系统性高估**实际有效 Token 命中率 |
| **块对齐与尾碎** | 固定页/任意变长 Slab | 按固定 `chunk_size`（如 16/64/256 Tokens）切分哈希，请求尾部不足 1 个完整 Chunk 的 Token 无法被标准前缀索引复用 | 同一批历史日志在不同 `chunk_size` 下有效工作集与命中率显著变化，必须做二维 Sweep |
| **工作负载双相性** | 仅看存储命中率与磁盘 I/O | KV 大小与并发吞吐强耦合：若分配给前缀复用缓存的显存过大，会挤压运行批并发度（Running `max_num_seqs`），反而打穿 Decode TBT/TTFT SLO | 最佳 KV 空间上界不能只看 MRC 拐点，必须与 SLA 并发上限联合寻优（见第四部分） |

---

## 第二部分：分布式推理层 KV Cache 容量规划、驱逐与感知路由寻优

在开源推理基础设施中，容量管理（驱逐/配额/模拟）与请求路由（亲和 vs 均衡）构成了 KV 缓存效益释放的一体两面。

### 2.1 `LMCache/LMCache`：分布式 ARC 驱逐、双语义配额与本样本中的离线 MRC 扫频器

`LMCache/LMCache`（Commit `6a316c3e0174b693dbc9500c7a991477b9f20206`）在本地仓库中提供了目前 LLM 推理生态最完整的容量建模与多级淘汰机制：

1. **压力驱动自适应替代驱逐（`lmcache/v1/distributed/eviction_policy/arc.py`）**：
   - `ARCEvictionPolicy` 继承并适配了 Adaptive Replacement Cache 机制：维护近期单次驻留队列 `T1`、高频复用驻留队列 `T2`，以及不占数据字节、仅保留哈希指纹的幽灵列表 `B1` / `B2`。
   - 当分布式存储发生空间超限需要批量淘汰时，若 `B1` 发生“幽灵命中”（说明此前驱逐了仍会快速回流的近期前缀），系统自适应调大 `_target_t1_size` 保护近期上下文；反之若 `B2` 命中，则压缩 `T1` 将配额让渡给高频系统提示词等长期热点 `T2`。
2. **双语义租户配额注册表（`lmcache/v1/distributed/quota_manager.py`）**：
   - `QuotaManager` 按 `cache_salt` 维护字节预算映射表并通过 `/quota/...` HTTP API 支持热更新。
   - **精妙的双透镜设计**：MP Server 本地驱逐控制器调用 `get_limit_bytes` 采用严格允许列表语义——未登记 `cache_salt` 默认视为 `0` 字节预算并在下个周期立即清退；而分布式协调器启动初期调用 `effective_limit_bytes` 时默认返回 `None`（豁免驱逐），避免外部配额控制器尚未同步完成时误杀健康租户缓存。
3. **离线轨迹驱动容量-命中率顾问（`lmcache/tools/cache_simulator/` & `lmcache bench trace-analyze`）**：
   - **路线 A（MP 日志回放，`simulator.py` + `plot_hit_rate.py`）**：直接读取服务端 `MP_LOOKUP` 块哈希 JSONL 日志，内部通过高精度 `LRUCache` / 向量化优化版 `LRUCacheFast`，默认在 **`0.5 GiB` ~ `500 GiB` 区间生成 `30` 个对数等分容量点**进行批量回放，输出精确的 `token_hit_rate = total_hit_tokens / total_tokens`（明确遵循尾部不足整 Chunk 计为 miss 的原则），用来观察该轨迹何处开始收益递减；不是集群容量实测。
   - **路线 B（OTLP Token Trace 二维扫频，`trace_analyze.py`）**：针对开启 `--enable-token-trace` 采集的通用工作负载 Trace，通过 `default_capacities(total_tokens)` 从 `4096` tokens 起按 **$\sqrt{2}$ 半倍频步长**一路扫至无驱逐全工作集上限，在 **`(chunk_size, capacity_tokens)` 二维空间**评估每组参数组合，并同步量化纯前缀命中率与基于 `BlendTokenRangeMatcher`（CacheBlend）的非前缀子串拼接命中率。

### 2.2 `kvcache-ai/Mooncake`：频次防污染准入、16 MiB 热缓存与 Conductor 200K 上下文索引

`kvcache-ai/Mooncake`（Commit `6ca711276690d42b6aa195dccbb6cd6c14144772`）在高速互联存储层和调度协调器层呈现了极强的生产细节：

- **抗扫描污染的准入过滤（`mooncake-store/include/count_min_sketch.h`, `local_hot_cache.h`, `eviction_strategy.h`）**：
  - `LocalHotCacheConfig` 默认固定热块尺寸 `block_size_bytes = 16 * 1024 * 1024`（16 MiB），设置准入门槛 `admission_threshold = 2`。
  - 背后挂载 `CountMinSketch(width=4096, depth=4)` 近似频次统计器：只有访问计数跨过门槛的块才有资格晋升热区，防止单次长文本一次性读入冲刷整个热 LRU 池；当累积计数 `total_increments >= width * depth` 时对 Sketch 桶整体右移减半衰减，维持对热点漂移的灵敏度。
- **持久化租户字节配额（`mooncake-store/src/tenant/quota_manager.cpp`）**：
  - `TenantQuotaManager` 通过 `UpsertPolicy` / `LoadPoliciesOrThrow` 将各租户请求字节上限校验后持久化并在 `allocatable_capacity_()` 约束下执行隔离。
- **Conductor 上下文感知前缀索引（`mooncake-conductor/include/conductor/prefixindex/prefix_indexer.h`）**：
  - `PrefixCacheTable` 将前缀索引严格按 `(tenant_id, model_name, lora_name, block_size)` 四元组隔离，防止 LoRA 微调权重或租户密匙空间混用引发错算；默认单表上限 `kDefaultMaxBlocks = 200000`，触发超限时按 `kEvictTargetRatio = 0.9` 执行批量驱逐滞回控制。
  - 通过 ZMQ 订阅 KV 事件以及 `vllm_v1` / `sglang` / `sglang_bigram` 三类引擎哈希重构逻辑，为每个 Rank 计算精确分层的 `RankCacheHitResult`（清晰分离 `device`、`cpu_local`、`cpu_share` 与 `disk` 四层介质驻留量）。

### 2.3 `vllm-project/aibrix`：四重防抖护栏、5:4 自动混权与 Preble 成本模型

`vllm-project/aibrix`（Commit `a0d8137c8ec78778aa12dd633c742799879cad95`）在 `pkg/plugins/gateway/algorithms/` 展示了开源集群网关中最丰富的负载均衡与缓存博弈设计（注意：它主要做**运行时调度路由寻优**，无离线缓存大小建议器）：

1. **KV 显存压力非线性熔断公式（`load_balance.go` & `profile_knobs.go`）**：
   $$\text{loadBalanceScore}(p) = \begin{cases} +\infty, & \text{kvFree}(p) < \text{kvCriticalFree}\;(0.10) \\ \dfrac{\text{load}(p)}{\text{capacity}(p)} \cdot \Big(1 + \alpha_{\text{kv}} \cdot \big(1 - \text{kvFree}(p)\big)^2\Big), & \text{otherwise} \end{cases}$$
   其中默认 `loadBalanceKVPressureAlpha = 2.0`，`loadBalanceQueuedWeight = 0.0`。一旦单机剩余 KV 低于 10% 直接熔断，处于中高水位区间时惩罚代价随已用比率呈二次方加速爬升。
2. **前缀热点防雪崩双护栏（`prefixcache/router.go` `getTargetPodFromMatchedPodsFromCounts`）**：
   - 将命中同一前缀的 Pod 按 `%prefixmatch` 降序、运行请求数升序排列后，依次检查候选 Pod 是否触发过载保护阈值：
     - 统计方差护栏：`reqCnt > mean + stdDevFactor * stdDev`（默认 `stdDevFactor = 1`）
     - 倍率封顶护栏：`reqCnt > loadCap`（`loadCapFactor = 1.5 * (mean + 1)`）
   - 凡触发任一护栏的过热节点即使拥有最高前缀命中率也会被主动跳过，若所有命中节点均过载，则退回全局最小请求数节点实现自动分流扩散。
3. **默认组合策略权重比（`router.go` `loadAutoBlendWeightsFromEnv`）**：
   - 自动混权默认比例为 `prefix-cache : prefix-cache-load-balance = 5 : 4`（即 $1.25 : 1$），且当 `prefix-cache` 处于激活状态时，系统显式跳过 `least-request-top-k` 插件以免对负载指标产生二次重复扣分。
4. **Preble 滑窗代价路由模型（`prefixcache/preble.go` `prefixCacheAndLoadRouter`）**：
   - 在 `LPRadixCache` 前缀树上挂载时间窗为 `3m` 的 `SlidingWindowHistogram`：若最长前缀匹配度 `matchRatio > 0.5`，直接在持有者中择取最低负载节点；否则通过 Preble 模型联合估算排队+计算期望代价：
     $$\text{Cost} = \underbrace{\frac{\text{missRate} \cdot \text{count} \cdot (\text{baseTime} + \text{attnQuad})}{0.9 \cdot \text{numHolderPods}}}_{\text{prefillCost}} + \underbrace{\text{outputLen} \cdot \text{median}(\text{timePerToken})}_{\text{decodeCost}}$$

### 2.4 `vllm-project/production-stack`：从刚性 Token 截断到无量纲线性效用

在 `vllm-project/production-stack`（Commit `405a2fa79992754d31b1eabb7183a540e`）的 `src/vllm_router/routers/routing_logic.py` 中，清晰记录了两代感知路由策略的演进逻辑：

- **`KvawareRouter`（硬门槛模式）**：向 `LMCacheControllerManager` 发送 `LookupMsg` 获取精确前缀命中数 `matched_tokens`。只有当满足 `matched_tokens >= max(len(token_ids) - kv_aware_threshold, 0)`（默认 `kv_aware_threshold = 2000`，即最多允许尾部 2000 tokens 未命中）才路由到缓存持有 Pod，否则回退至按 `session_id` 构建的一致性哈希环（`HashRing`）或 QPS 负载均衡。
- **`LoadAwareRouter`（无量纲连续效用权衡）**：针对刚性门槛容易忽视局部队列拥塞的缺陷，将其重写为显式线性权衡公式：
  $$\text{score}(i) = \frac{\min(\text{matched\_tokens}(i),\; \text{prompt\_tokens})}{\max(\text{prompt\_tokens},\; 1)} - \beta \cdot \frac{\text{load}(i) - \bar{\text{load}}}{\max(1.0,\; \bar{\text{load}})}$$
  其中 $\text{load}(i) = \text{in\_prefill\_requests}(i) + \text{in\_decoding\_requests}(i)$，$\bar{\text{load}}$ 为集群平均在途请求数，权衡超参数默认 `DEFAULT_LOADAWARE_BETA = 1.0`。公式两端完全无量纲，极大简化了不同长度提示词下的联合调参难度。

### 2.5 `sgl-project/sglang`（`sgl-model-gateway`）：双阈值状态机与 PD 隔离基数树

`sgl-project/sglang` 仓内网关模块 `sgl-model-gateway`（Commit `4f8b6867534c8a6632845353997555fca857dd63`，路径 `sgl-model-gateway/src/policies/cache_aware.rs` & `mod.rs`）选择了**极低延迟、状态机驱动的近似树路由**：

- 路由器进程内维护一棵**字符级近似基数树**，按照 `pool::model` 键值彻底隔离 Prefill/Decode 分离池，限制单树节点容量 `max_tree_size = 10000` 并每 `eviction_interval_secs = 30` 秒后台清理过期分支。
- **双模式硬切换判断**：
  1. 先检测集群负载是否失衡：`(max_load - min_load) > balance_abs_threshold && max_load > min_load * balance_rel_threshold`（默认值分别为 **`32`** 与 **`1.1`**，即最大/最小队列之差超过 32 且比值超 10%）。
  2. **若失衡**：立刻切换至负载优先模式，直选 `min_load` Worker（平局随机），**同时仍将当前请求文本插回基数树维护热拓扑**，保证负载恢复平稳后下一轮即可重获命中能力。
  3. **若均衡**：检索基数树最长前缀字符比，若 `match_rate > cache_threshold`（默认 **`0.5`**）则锁定缓存所在 Worker，否则分派至 `min_load` 进行冷预热。

### 2.6 `llm-d`：K8s EPP 扩展、多介质计分与 TTFT 破粘闸门

`llm-d` 生态将网关路由拆分至独立可插拔的 Kubernetes `gateway-api-inference-extension`（EPP，原记录的版本串长度异常，本章不以它作提交定位）框架中，核心实现在 `llm-d/llm-d-inference-scheduler`（`647ca7ca...`）与 `llm-d/llm-d-kv-cache`（`822e6dd9...`）：

1. **多存储介质差异化前缀计分（`llm-d-kv-cache/pkg/kvcache/kvblock_scorer.go`）**：
   - `LongestPrefixScorer` 从 Block 0 开始遍历连续匹配块，每个块读取所在节点的最高介质权重 `max(MediumWeight)` 累加（默认 **`gpu = 1.0`**、**`cpu = 0.8`**），用可量化的拉取开销折现反映真实性能收益。
2. **归一化多因子加权打分链（`llm-d-inference-scheduler/.../scheduling/`）**：
   - **`prefix-cache-scorer`**：兼容长度平方加速项与比例项的混合打分公式：
     $$\text{score} = w_{\text{len}} \cdot \min\left(1.0,\; \frac{\text{matchBlocks}\cdot\text{blockSize}}{\text{MatchLengthScaleTokens}}\right)^2 + (1 - w_{\text{len}})\cdot \frac{\text{matchBlocks}}{\text{totalBlocks}}$$
     （默认 `defaultMatchLengthWeight = 0.0`、`defaultMatchLengthScaleTokens = 8192`）。
   - **三类负载打分器协同**：`load-aware-scorer` 按排队长度在 `QueueThresholdDefault = 128` 内线性折减（空队列得 `0.5`）；`kv-cache-utilization-scorer` 直接返回 `1 - KVCacheUsagePercent`；`token-load-scorer` 将 `InFlightLoad.Tokens + UncachedRequestTokens.Tokens` 按最大窗口 `4194304`（$128 \times 32\text{K}$）映射至 `[0, 1]`。
   - 默认调度配置文件（`pkg/epp/config/loader/defaults.go`）中三者组合权重为 **`queue-scorer: 2.0`、`kv-cache-utilization-scorer: 2.0`、`prefix-cache-scorer: 3.0`**；另配套 `no-hit-lru-scorer`（`defaultLRUSize = 1024`）专门将全未命中冷请求做节点轮转打散。
3. **带“TTFT 破粘闸门”的亲和过滤器（`filter/prefixcacheaffinity/plugin.go`）**：
   - 相比仅做线性加权的网关，`prefix-cache-affinity-filter` 显式设置了“防止死等缓存”的硬止损：首先以概率 `ExplorationProbability`（默认 `0`）随机跳过过滤进行全局探索；其余请求把端点划分为 `sticky`（`prefixCacheScore >= AffinityThreshold`，默认 **`0.80`**）与 `nonSticky`。
   - 随后读取两端各自最佳预计 TTFT（可基于 `InFlightLoad / PeakPrefillThroughput` 或 `latencyPredictor` 实时估算），计算延迟惩罚 $\text{penalty} = \text{bestStickyTTFT} - \text{bestNonStickyTTFT}$。若超过阈值 `MaxTTFTPenaltyMs`（默认 **`18000 ms`**），判定缓存收益无法抵消排队代价，立刻触发 `outcomeLoadOverride` 破除粘性、保留全集群候选；否则窄化至 `sticky` 子集。

---

## 第三部分：API 网关语义缓存与云端 Prompt Caching 最优配置方案

当业务场景从“自建 GPU 集群推理”切换至“API 网关代理与商业云端大模型调用”时，缓存形态分化为**响应级语义缓存（Semantic Cache）**与**云端显式/自动前缀缓存（Prompt Caching）**。

### 3.1 API 网关语义缓存现状与容量/阈值寻优盲区

我们对市面上最主流的五个开源/商业语义缓存框架及网关进行了代码或官方配置核查：

| 项目名称 | 核查 Commit / 来源 | 核心缓存结构与关键可调参数 | 是否有“容量大小 / 相似度阈值自动建议器”？ |
| :--- | :--- | :--- | :--- |
| **`zilliztech/GPTCache`** | 本地源码 `a74ac654...`<br>`memory_cache.py`<br>`reuse_compatibility/benchmark.py` | 封装 `cachetools`（`LRU`/`LFU`/`FIFO`/`RR`），以 `clean_size` 触发定量驱逐（默认 `maxsize = 1000`）；向后配合向量库检索。 | **无线上/自动容量建议**。但 `examples/benchmark/reuse_compatibility/benchmark.py` 提供基于 **`1536` 条标注 pair** 的离线实验脚本，用于对比固定相似度阈值与 `JevEvaluation` LLM 裁判下的 `reuse_precision` / `reuse_recall` / `hit_rate` 折中选优。 |
| **`maximhq/bifrost`** | 本地源码 `63963974...`<br>`plugins/semanticcache/main.go` | 双层检索：`xxhash` 精确命中后走 Embedding VectorStore 语义匹配；核心参数 `Threshold = 0.8`、`TTL = 5min`、`ConversationHistoryThreshold = 3`。 | **无**（全靠人工静态阈值与 TTL 过期控制向量库膨胀）。 |
| **`alibaba/higress`** | 本地源码 `6726080d...`<br>`extensions/ai-cache/core.go` | Wasm Go 扩展，Redis 精确 KV 缓存未命中时回落向量库相似度检索，按消费者（consumer）维度做隔离与 TTL 控制。 | **无**（依赖 Redis `maxmemory-policy` 与固定阈值参数）。 |
| **`Portkey-AI/gateway`** | 本地源码 `669825cb...`<br>`src/handlers/services/cacheService.ts` | 支持 `simple`（精确）与 `semantic`（语义）双模式开关及请求级 `maxAge` TTL 控制。 | **无**（参数全量依赖路由配置下发）。 |
| **`Kong` `ai-semantic-cache`** | 检索官方文档核实（v3.8+ 插件） | 配置 `embeddings` 模型与 `vectordb`（`redis`/`pgvector`，`distance_metric: cosine`、`threshold` 如 `0.85`、`dimensions`）及过期时间 `ttl`。 | **无**（官方文档仅给出经验区间建议，如高严谨 `0.90–0.95` vs 高召回 `0.75–0.80`）。 |

**关键洞察**：语义缓存的容量建议与 KV 缓存完全不同——其成本瓶颈不在 VRAM，而在**向量数据库内存与误命中风险（False Positive 幻觉）**。目前五个主流网关无一实现基于线上分布的 MRC 自动推荐；`GPTCache` 的 `1536` 样本标注回放范式仍是目前最可借鉴的阈值标定方式。

### 3.2 云端 LLM Prompt Caching 的数学本质：`5m` vs `1h` TTL 盈亏平衡推导

在调用 Anthropic Claude API（以及遵循类似阶梯缓存定价的供应商）时，缓存容量不体现在本地内存限制上，而是体现为**高昂的缓存写入溢价（Write Premium）与生命周期时长（TTL）之间的经济博弈**。

通过对 `montevive/autocache`（Commit `d01556f6a17b124e5b5b96756117684dcb31d33c`）核心模块 `internal/pricing/pricing.go` 中 `EstimateBreakpointROI` 的源码还原，我们得到精确的定价与盈亏平衡公式：

- **Anthropic 价格基准倍率**：
  $$P_{\text{write,5m}} = 1.25 \cdot P_{\text{input}}, \qquad P_{\text{write,1h}} = 2.0 \cdot P_{\text{input}}, \qquad P_{\text{read}} = 0.1 \cdot P_{\text{input}}$$
- **先和不缓存比**：$K$ 是同一前缀的请求总数，含首次写入。表中成本以一次未缓存输入为 1，假设其余 $K-1$ 次均命中：
  $$\text{breakEven} = 1 + \left\lceil \frac{P_{\text{write}} - P_{\text{input}}}{P_{\text{input}} - P_{\text{read}}} \right\rceil$$

按该式展开两次关键临界测算（设单段可缓存前缀 Token 成本基准为 $1$ 单位）：

| 缓存 TTL 档位 | 首次写入成本 | 每次读取成本 | 累计 2 次总请求（1 写 1 读）总花费与净损益 | 累计 3 次总请求（1 写 2 读）总花费与净损益 | 盈亏平衡临界请求数 |
| :--- | :---: | :---: | :--- | :--- | :---: |
| **无缓存基线** | $1.00$ | $1.00$ | $2.00$（0%） | $3.00$（0%） | — |
| **`5m` 短期缓存** | $1.25$ | $0.10$ | $1.35$ vs $2.00$ $\implies$ **净省 32.5%** | $1.45$ vs $3.00$ $\implies$ **净省 51.7%** | **2 次（命中 1 次即回本）** |
| **`1h` 长期缓存** | $2.00$ | $0.10$ | $2.10$ vs $2.00$ $\implies$ **反亏 5.0%** | $2.20$ vs $3.00$ $\implies$ **净省 26.7%** | **3 次（至少命中 2 次才赚）** |

现在**改换参照物**：不是跟不缓存比，而是比较两种 TTL。仍令 $K$ 为含首次写入的总请求数，且相邻两次间隔均在 5～60 分钟之间，短 TTL 每次得重写，长 TTL 后续仍命中：
- 选 `5m`（全部冷写）成本：$1.25 K$
- 选 `1h`（1 次写入 + $K-1$ 次命中）成本：$2.0 + 0.1(K - 1) = 1.9 + 0.1K$
- 解 $1.9+0.1K<1.25K$，整数 $K\ge2$。两次请求时长 TTL 花 2.10，短 TTL 花 2.50：长 TTL 胜过短 TTL，但仍比不缓存的 2.00 贵。只请求一次则是 2.00 对 1.25。

### 3.3 开源 Prompt Cache 代理与结构优化器横评

1. **`montevive/autocache`（透明网关断点注入代理）**：
   - 专为不支持缓存控制的 Agent 编排器（n8n、Flowise、LangChain 等）充当 Drop-in 透明代理，在 `internal/cache/injector.go` 与 `internal/types/models.go` 中按 **`system -> tools -> messages`** 的稳定不变顺序自动插入 `cache_control` 断点，并通过响应头实时吐出盈亏分析。
   - **三档策略预算**：`conservative` / `moderate` / `aggressive` 分别配置上限为 `2 / 3 / 4` 个断点（严格遵守 Anthropic 最大 4 断点限制）及最小 Token 触发倍率 `2.0 / 1.0 / 0.8`（屏蔽过短上下文防止白交写入溢价）。
   - **核心短板**：TTL 策略完全静态写死（`SystemTTL = ToolsTTL = "1h"`、`ContentTTL = "5m"`），**不具备**根据实测跨轮延迟直方图动态升降 `5m` / `1h` 的自适应寻优能力。
2. **`easmith/claude-cache-analyzer`（事后日志回放审计器）**：
   - Commit `13fd8c95...`，源码 `pricing.py` 与 `metrics.py` 通过解析本地 Claude Code `.jsonl` 会话记录，逐轮复算真实账单 `actual_cost`、无缓存对照账单 `cost_no_cache`、隐性写入溢价 `cache_write_overhead = cache_creation_tokens * (P_write - P_input)`，最终给出 **`cache_efficiency_score = hit_rate * (total_cacheable / (total_input + total_cacheable))`** 与 A–F 评级。属于“事后体检报告”，不支持事前修改配置参数。
3. **`vansye/prompt-cache-optimizer` & `Encephos/prompt-cache-optimizer`（前置结构整理与对齐器）**：
   - `Encephos`（`4b05f0e2...` `core.py`）仅是将静态 `system_instruction + rag_documents` 前置并注入断点标签的轻量封装。
   - `vansye`（`9179b496...` `reorderer.py`, `aligner.py`）则针对多模块动态拼接破坏前缀连续性的痛点，使用 `Reorderer` 在不破坏 `tool_use` / `tool_result` 交错强约束的前提下，对同角色可交换上下文块按 **SHA-256 内容哈希确定性重排序**，并用 `Aligner` 结合 `tiktoken`（失败时退化 `len // 6`）对未达标前缀自动填充 `_DEFAULT_PADDING` 以卡准云端缓存生效门限。

---

## 第四部分：工作负载驱动的 LLM 自动最优配置推荐器

从“缓存容量”进一步提升到“整机推理引擎参数寻优”，本部分拆解开源社区中直接对标“输入工作负载与 SLA 要求，自动输出最优并行度（TP/PP/EP）、批大小与 KV 显存配比”的代表性 Auto-Configurator。

### 4.1 `bentoml/llm-optimizer`：显存解析 + 三瓶颈 Roofline 模型

`bentoml/llm-optimizer`（Commit `7be87d776e60a978516a2ae53e4a4ff745d04c20`）以纯静态数学建模见长，不依赖集群 GPU 实跑即可秒级给出 vLLM / SGLang 启动参数与性能包络：

1. **KV 显存精确解析（`src/llm_optimizer/resources/memory_calculator.py`）**：
   - `ModelMemoryCalculator` 引入框架杂项安全冗余系数 `overhead_factor = 1.2` 与激活内存乘数 `4`，单 Token 序列并发 KV 占存按精确 GQA/MHA 公式核算：
     $$\text{KV}_{\text{bytes}} = 2 \cdot L \cdot n_{\text{kv}} \cdot d_{\text{head}} \cdot S \cdot B \cdot \text{bytes\_per\_elem}$$
     据此闭式反解最大批大小 `estimate_max_batch_size` 与最大上下文 `estimate_max_sequence_length`。
2. **三瓶颈 Roofline 并发下界（`src/llm_optimizer/performance.py`）**：
   - 预设经验硬件利用率因子 `mfu_prefill = 0.45`、`mfu_decode = 0.30`、`vram_util_factor = 0.90`，分别计算：
     - 显存天花板：`kv_cache_limit`
     - 算力天花板：`prefill_compute_limit`（以 `0.5s` TTFT 预算折算）
     - 访存带宽天花板：`decode_capacity_limit`（以 `0.05s` 单步 TBT 预算折算）
   - 取三者最小值并乘以分位数惩罚因子（`mean=1.0, median=1.1, p95=1.3, p99=1.6`），同时配合边际吞吐增幅门限 `0.05`（`find_optimal_concurrency_threshold`）拦截低效高并发区间。
3. **框架调优映射（`src/llm_optimizer/tuning/strategy.py`, `core.py`, `generation.py`）**：
   - `SGLangStrategy` 与 `VLLMStrategy` 以 `300 TFLOPS` 算力与 `2000 GB/s` HBM 带宽为基准锚点归一化缩放计算 `calculate_optimal_batch_tokens`、`calculate_optimal_max_seqs`、`calculate_chunked_prefill_size` 与 `calculate_memory_fraction`，输出 `simple` / `advanced` 两级可执行启停配置组合。

### 4.2 `ai-dynamo/aiconfigurator`：实测库 + Rust 步进核 + `--prefix` 非对称建模与帕累托决策

由 NVIDIA 开源的 `ai-dynamo/aiconfigurator`（Commit `6cd6f9686ab7de6e8fb28171d6e350ee0aa4ed39`）是目前工业界**架构最精密的混合实测-建模配置推荐器**，结合了底层的算子 Silicon Profiling 数据库与 Rust 编写的高性能推演核（`aic-core`）：

1. **区分不同推理引擎语义的 KV 预算模型（`aic-core/src/aiconfigurator_core/sdk/memory.py`）**：
   - `KVCacheEstimator` 精准刻画了各大引擎在 `--gpu-memory-utilization`（`frac`）语义上的根本分歧：
     - **TensorRT-LLM（`of_free` 语义）**：$\text{Budget} = (\text{cap} - \text{non\_kv}) \cdot \text{frac} \cdot \text{scale}$
     - **vLLM / SGLang（`of_total` 语义）**：$\text{Budget} = \text{cap} \cdot \text{frac} \cdot \text{scale} - \text{non\_kv}$
     其中安全缩减因子 $\text{scale} = (1 - \text{reserved}) \cdot (1 - \text{tolerance})$；若遇不支持精算的模型则回落至 `NaiveKVCacheEstimator` 并硬性预留 `_DEFAULT_NAIVE_KV_RESERVATION = 0.80`。
2. **Prefix Cache 对底层算子的非对称影响（`aic-core/rust/aiconfigurator-core/src/session.rs` `get_mix_step_ops`）**：
   - 当用户通过 `--prefix` 指定前缀缓存命中长度时，Rust 内核在三遍步进模拟中做出了极具洞察力的非对称拆解：
     - **GEMM / MoE 非注意力线性层**：仅对未命中的增量 Token 计算量（`effective_isl = isl - prefix = ctx_new + gen`）计费，缓存命中的前缀完全豁免矩阵乘计算；
     - **Context Attention / MLA 注意力层**：依然严格保留完整历史序列特征 `prefix_per_req` 与 `combined_prefix`，因为注意力核仍必须从 HBM 中**读取全部已缓存前缀对应的 K/V 张量**参与点积！忽视这一点会让所有纯按 `effective_isl` 计费的简化器在长上下文高命中率场景下严重低估实际 TTFT。
3. **聚合 / PD 分离 / AFD 帕累托选优（`sdk/pareto_analysis.py`, `picking.py`）**：
   - 自动横扫 Aggregated（`agg_pareto`）、Prefill-Decode 分离（`disagg_pareto`）及 Attention-FFN 分离（`AFD`，内置折损校正 `_AFD_PREFILL_DEGRADATION = 0.9`、`_AFD_DECODE_DEGRADATION = 0.95`、`_AFD_TTFT_CORRECTION_FACTOR = 1.8`）。
   - 提供 `pick_default`、`pick_load_match`（率匹配折损因子 `0.9` / `0.92`）与 `pick_autoscale`（`_AUTOSCALE_TTFT_CORRECTION_FACTOR = 1.8`）三种选型决策器，最终在满足严格 TTFT / ITL SLO 的可行域内直接打印帕累托最优部署拓扑（如 TP/PP/DP/EP、P/D 实例配比、最大并发数）。

### 4.3 `microsoft/vidur`：离散事件集群仿真与 QPS/Dollar 二分探索

`microsoft/vidur`（Commit `abae7f63aa857300f5cdc6f5e0d27860cd24721b`）放弃静态解析公式，选择用 Ray 并行集群驱动微粒级**离散事件仿真器（Discrete-Event Simulator）**复现真实连续批处理排队、抢占与调度延迟：

- **自适应能力搜索（`vidur/config_optimizer/config_explorer/capacity_search.py:69-85, 151-169` `CapacitySearch.search`）**：针对每个候选并行与硬件配置，采用“指数试探扩界 + 二分精确收敛”探寻满足**调度延迟单分位数 SLO（`request_scheduling_delay`：`scheduling_delay_slo_quantile <= scheduling_delay_slo_value`）**的最大可持续 QPS：当 `scheduling_delay` 远低于 `slo/8` 时将上界右移 $\times 4$、`slo/4` 时 $\times 2$；超出 SLO 后对照**绝对延迟阈值**收缩（源码 L164-167 实为 `if scheduling_delay > 500: right = qps / 2` / `elif scheduling_delay > 1000: right = qps / 4`，注：因先判 `> 500`，上游代码的 `elif > 1000` 实为不可达分支）。
- **下游 TTFT/TBT SLO 筛选与经济性帕累托提取（`analyzer/generate_pareto_curves.py`）**：在所有配置的仿真结果汇总至分析阶段时，脚本再按 TTFT 与 TBT P90/P99 SLO 过滤可行域，并调用 `paretoset(..., sense=["max", "min"])` 在“**单位美元每小时可达 QPS 最大化**”与“**系统成本最小化**”二维空间抽取前沿方案。

### 4.4 `vllm-project/guidellm`：实机 2-Segment OLS 拐点识别与 Wilson 置信寻优

当缺乏算子建模库、需要将压测指针直接对准任意黑盒 HTTP 推理端时，`vllm-project/guidellm`（Commit `b15e11bf940ed7c539b4986d4bba8a75cad0c620`）提供了业界严谨度最高的**在线拐点探测与 Goodput 算法**：

- **两段分段线性最小二乘拐点探测（`src/guidellm/benchmark/analysis/knee.py` `find_throughput_knee`）**：在归一化并发度-吞吐散点上遍历所有切分点拟合两段 OLS 直线，并设置四重数学护栏防止噪声误报：
  1. 样本数至少 `MIN_POINTS = 5`；
  2. 后段饱和斜率相对前段加速斜率比值不超过 `MAX_TAIL_SLOPE_RATIO = 0.25`；
  3. 两段拟合残差平方和（RSS）相较单直线拟合的相对改善率不低于 `MIN_FIT_IMPROVEMENT = 0.50`；
  4. 拐点所在吞吐率不得低于全局峰值吞吐的 `MIN_THROUGHPUT_FRACTION = 0.85`。
- **置信可控 Goodput 搜索（`profiles/goodput.py`）**：在真实线上压测中通过指数翻倍与二分查找搜索恰好满足 TTFT/ITL/E2E 约束的 RPS 上限，并引入二项分布置信区间函数 **`wilson_interval`** 控制边界样本量与统计分辨率。

### 4.5 黑盒贝叶斯优化与官方网格脚本

- **`InseeFrLab/auto-tuning-vllm`**（Commit `4c2220e6...`，`auto_tune_vllm/core/study_controller.py`, `constraint.py`）：将面向真实 vLLM 集群的在线超参数寻优直接抽象为 Optuna `StudyController`，支持 `TPESampler`、`GPSampler`、`BoTorchSampler`（贝叶斯高斯过程）与 `NSGAIISampler`（帕累托多目标），并通过 `Constraint.evaluate_constraint <= 0` 执行表达式级硬延迟约束过滤。
- **`vllm/benchmarks/auto_tune/auto_tune.sh`**（Commit `793a6351...`）：vLLM 官方提供的一键网格搜索 Bash 脚本，遍历 `NUM_SEQS_LIST` $\times$ `NUM_BATCHED_TOKENS_LIST`（在非 GPU 探针路径下自动从 `0.98` 向下试探 `gpu_memory_utilization` 至 `0.90` 以防 OOM），支持传入 `--random-prefix-len` 在开启前缀缓存条件下筛选满足 `MAX_LATENCY_ALLOWED_MS` 的吞吐/Goodput 最优配置。

### 4.6 常见误传与边界证伪清单

在文献检索常被归为“自动调参器”的以下两个仓库，经一级源码验证必须予以界定降级：
1. **`kubernetes-sigs/inference-perf`**（Commit `68730a44...`）：勘察 `inference_perf/analysis/analyze.py` 确认，该工具仅执行多阶梯并发 QPS 压测、熔断校验与汇总输出均值延迟图表（`QPS vs metric`），**本身不包含**任何参数推荐、帕累托筛选或超参搜索模块。
2. **`ekartashov/local-vllm-tuner`**（Commit `8307e9ec...`）：勘察仓内核心脚本 `script.sh`（共 6 行）发现其全部可执行逻辑仅为 `echo running; sleep 5; echo done`，确认为**占位空壳仓库（Stub）**，不具备真实调优参考价值。

---

## 第五部分：面向多 API 供应商的实测选优实验室（类 `magpie-speed-lab` 生态）

针对多 API 供应商、中转聚合站点场景（直接对标 `magpie-speed-lab`），本次调研将开源作品按照“人工榜单面板 $\to$ 网络精细剖析 $\to$ 在线强化学习路由”分为三个梯队进行了源码拆解。

### 5.1 全栈测速实验室：`bytehola/llm-speed-test`（含配置边界精确证伪）

`bytehola/llm-speed-test`（Commit `b403887450510f13c557fc02005b265dfeba038c`）在工具形态上与 `magpie-speed-lab` 最为贴近：基于 Node/Express + Vue + SQLite 构建，具备代理支持、多 Provider/多模型并发基准任务、SSE 实时吐字（逐条记录 `ttft` / `tps` / `total_time` / `error_count`）以及基于最近 10 次成功样本（`avg10`）的横向排序接口（`server/routes/records.js` `/summary`、`/compare/:modelName`）。

- **关键代码核实（配置导入 vs 导出边界）**：在外部索引常描述其“支持导入/导出 YAML 路由配置”，但逐行阅读 `server/routes/providers.js`（`/preview-cpa`、`/import-cpa`）与 `server/utils/cpa-import.js` 后确认：**该仓实现了解析导入 `cli-proxy-api`（CPA）`config.yaml`（支持 `openai-compatibility`、`codex-api-key`、`gemini-api-key`、`claude-api-key`）并写入本地 SQLite，但并不包含将最优测速结果反向导出或写回更新路由配置文件的功能**。

### 5.2 轻量化 CLI 测速与细分指标工具

1. **`pcm469768-droid/llm-api-speedtest`**（Commit `408f7653...`，`internal/bench/bench.go`）：零外部依赖纯 Go CLI 工具，并发压测 `stations.json` 中的多个中转 API 站点，采集 `TTFBMS`、`TotalMS` 与 `TokensPerSec = total_tokens / (TotalMS / 1000)`，并调用 `FormatTable` 直接更新 README 中的 Markdown 速度排行榜。
2. **`Yoosu-L/llmapibenchmark`**（Commit `f40c4e35...`，`cmd/benchmark.go`）：Go 多并发基准 CLI，先执行网络 `MeasureLatency` 基线校准，随后在同一轮探测中将吞吐严谨拆分为 **`Prompt TPS`**（首字前输入吞吐）与 **`Decode TPS`**（生成阶段吞吐）及 **`Min/Max TTFT(s)`**，输出可归档的 Markdown 评测对比表。

### 5.3 网络链路分段诊断：`rbadillap/ai-gateways-benchmark`

`rbadillap/ai-gateways-benchmark`（Commit `849100a3...`，`bench.py`）弥补了多数中转测速器“只看总 TTFT、无法区分网络代理损耗与上游模型本身慢”的盲区：

- 每次探测显式剥离并采集 **6 个阶段时间指标**：`dns` $\to$ `tcp` $\to$ `tls` $\to$ `ttfb` $\to$ `ttft` $\to$ `e2e`。
- 明确隔离 **冷连接（Cold Socket）** 与 **长连接复用（Warm Keep-Alive）** 两种场景，并采用跨网关**交错轮询调度（Interleaved Round-Robin）**消除不同时刻公共云算力波峰对对比公平性的时间漂移误差。

### 5.4 从“被动测速榜单”迈向“自动最优路由决策”

1. **`PicadoLabs/ai-model-router`**（Commit `c6c53559...`，`backend/app/router/scoring.py`, `thompson_sampling.py`）：
   - 这是本次样本中能看到在线学习探索闭环的例子。它将静态多维度效用打分（`0.35 * quality + 0.25 * cost + 0.20 * speed + 0.15 * capability + 0.05 * reliability`）与持久化数据库状态的 **Beta-Bernoulli Thompson Sampling** 结合：对每个 `(task_type, model_id)` 维护参数 $\alpha, \beta$ 并在路由瞬间执行概率采样（`sample_quality`），按实际反馈持续更新后验置信分布，从而自动平衡“利用历史最快最稳渠道”与“探索新中转/新模型”的 Explore-Exploit 困境。
2. **`OpenRouterTeam/benchmark-harness`**（Commit `1503e540...`，`README.md`）：OpenRouter 官方用 Bun 编写的模型路由算法评估框架，支持在 `--cost-tier low|medium|high` 三档成本约束下对比 `jev-router` / `auto-router` / `auto-beta-router` 在 `gpqa_diamond`、`mmlu_pro` 等推理集上的智力-成本帕累托边界（属路由策略评测底座，非连线延迟测速器）。
3. **`QuantumNous/new-api` 与 `ginsonko/newapi-smart-router`**（Web 官方文档与仓库简介核实）：在聚合网关产品侧，`new-api` 内置了渠道健康/延迟单点与批量探测（`GET /api/channel/test/{id}`）；而社区配套插件 `newapi-smart-router` 在其之上外挂了 Key 粒度健康监视、失败自愈熔断、真实扣费同步及可解释动态调度能力。

---

## 第六部分：全景横向对比矩阵与网关工程演进建议

### 6.1 跨生态核心方案全景横向对比矩阵

| 方案/项目 | 目标层级 | 核心寻优/调度算法与关键数学结构 | 核心默认参数 / 关键门槛 | 输入形态 $\to$ 输出形态 | 核心工程局限 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`cacheMon/libCacheSim`** | 底层通用缓存分析库 | Olken Splay Tree 单遍 $O(N\log M)$ Mattson 栈距离 + 多线程非 LRU Sweep（含 `cal_working_set_size`） | 8 档自动扫频 `[0.001..0.8] * WSS`（无 SHARDS） | 访存 Trace 文件 $\to$ 精确 MRC 曲线 | 未直接适配连续前缀破坏性与块对齐语义 |
| **`facebook/CacheLib`** | 在线分布式内存分配器 | `MarginalHitsStrategy` 尾部 Slab 边际收益导数均衡；`LruTailAgeStrategy` | `movingAverageParam=0.3`；`tailAgeDifferenceRatio=0.25`、`minTailAgeDifference=100` | 实时 LRU 尾部访问反馈 $\to$ 在线 Slab 动态跨池迁移 | 无前缀感知；仅适应固定/分档 Slab 对象 |
| **`LMCache/LMCache`** | 分布式 KV Offload 层 | `ARC` 幽灵列表驱逐 + `QuotaManager` 双语义配额 + 离线 `LRUCacheFast`/`CacheBlend` MRC 扫频 | `0.5–500 GiB`（30 点）；OTLP Trace 从 `4096` tokens 按 $\sqrt{2}$ 步进扫频 | `MP_LOOKUP` 日志 / OTLP Trace $\to$ **(chunk_size, capacity) 命中率曲线** | 离线扫频尚未直接反推联合 SLO 并发上界 |
| **`kvcache-ai/Mooncake`** | 存储+调度协调层 | `CountMinSketch` 频次准入 + `TenantQuotaManager` + `Conductor` 四介质分级命中索引 | 16 MiB 块；`admission_threshold=2`、`4096x4`；`kDefaultMaxBlocks=200000`、`kEvictTargetRatio=0.9` | ZMQ KV 事件 + 请求哈希 $\to$ 租户隔离配额与四介质命中分布 | 侧重分布式索引与存储传输，无离线容量建议器 |
| **`vllm-project/aibrix`** | 云原生推理网关 | 二次方 KV 压力惩罚 + 方差/倍率防热点护栏 + `5:4` 混权 + Preble 3m 滑窗代价模型 | `kvCriticalFree=0.10`、$\alpha_{\text{kv}}=2.0$；`stdDevFactor=1`、`loadCapFactor=1.5` | Pod KV 空闲度/排队指标/前缀树 $\to$ 实时最优 Pod 路由 | 纯在线路由调优，无最佳缓存大小计算模块 |
| **`vllm-project/production-stack`** | 云原生推理路由 | `KvawareRouter` 硬阈值 / `LoadAwareRouter` 无量纲线性效用公式 | `kv_aware_threshold=2000`；`DEFAULT_LOADAWARE_BETA=1.0` | LMCache 控制器匹配量 + Pod 在途负载 $\to$ 无量纲路由分数 | 缺失对不同存储介质（GPU vs CPU）迁移延迟折现 |
| **`sglang` (`sgl-model-gateway`)** | 高性能推理网关 | 字符级近似基数树 + 双阈值负载失衡硬切换状态机 | `balance_abs=32`、`balance_rel=1.1`、`cache_threshold=0.5`、`max_tree_size=10000` | Worker 排队数 + 文本前缀 $\to$ 极低开销路由决策 | 字符近似树与真实 Tokenize 边界存在小幅误差 |
| **`llm-d`（EPP 调度体系）** | K8s 推理调度扩展 | 多介质加权前缀分数 + 排队/KV/Token 负载归一化 + TTFT 破粘闸门 | `gpu=1.0, cpu=0.8`；`prefix:queue:kv=3:2:2`；`AffinityThreshold=0.80`、`MaxTTFTPenaltyMs=18000` | KV 块位置/在途 Token/TTFT 预测 $\to$ 过滤+打分路由结果 | 组件配置复杂，需完整 Envoy + EPP + DataProducer 栈 |
| **`GPTCache` / `Bifrost` / `Higress` / `Portkey` / `Kong`** | API 网关语义缓存层 | 精确哈希前哨 + Embedding 向量检索（余弦相似度阈值 + 固定 TTL） | `GPTCache maxsize=1000`；`Bifrost Threshold=0.8, TTL=5m` | 用户 Prompt $\to$ 历史问答缓存直接命中或放行 | 普遍**缺乏**在线容量建议与误命中代价均衡算法 |
| **`montevive/autocache` / `vansye` / `claude-cache-analyzer`** | 云端 Prompt Cache 优化与审计 | Anthropic 断点确定性注入 + ROI 盈亏平衡公式 + SHA256 块重排与日志事后审计 | 5m(`1.25x`)/1h(`2.0x`)/Read(`0.1x`)；回本点 2 次 vs 3 次；断点上限 `2/3/4` | 请求上下文 / `.jsonl` 日志 $\to$ 注入 `cache_control` / ROI 评级 | **均未解决**按会话间隔分布动态决策 5m vs 1h |
| **`ai-dynamo/aiconfigurator`** | 离线配置自动推荐器 | 实测数据库 + Rust 三遍步进模型 + `of_free`/`of_total` KV 预算 + `--prefix` 非对称建模 | `_DEFAULT_NAIVE_KV_RESERVATION=0.80`；AFD 修正因子 `0.9/0.95/1.8` | 硬件+模型+ISL/OSL/`--prefix`+SLO $\to$ Agg/PD/AFD 帕累托拓扑 | 强依赖预先针对特定硬件采集的 Silicon Profiling 库 |
| **`bentoml/llm-optimizer`** | 离线快速配置估算器 | KV 显存解析 + 0.5s Prefill / 0.05s Decode / VRAM 三瓶颈 Roofline 模型 | `mfu_prefill=0.45`, `mfu_decode=0.30`, `overhead_factor=1.2`, 300 TFLOPS 锚点 | GPU 型号+模型参数+分位数 SLA $\to$ vLLM/SGLang 启动配置 | 无真实离散批处理排队模拟，高并发尾部精度受限 |
| **`microsoft/vidur`** | 离线离散事件仿真器 | Ray 并行集群离散事件仿真 + `CapacitySearch` 二分寻调度延迟 SLO 极限 QPS + 下游帕累托 | `slo/8 -> 4x`、`slo/4 -> 2x`、`delay>500 -> qps/2` 自适应探界 | 硬件+分布+调度延迟 / 下游 TTFT·TBT SLO $\to$ QPS/Dollar 帕累托曲线 | 新增模型算子与调度器需手工补齐 Trace 校准 |
| **`vllm-project/guidellm`** | 实机压测与拐点分析器 | 2-Segment OLS 分段线性拐点检测 + 指数倍增二分 Goodput + `wilson_interval` | `MIN_POINTS=5`, `MAX_TAIL_SLOPE_RATIO=0.25`, `MIN_FIT_IMPROVEMENT=0.50`, `MIN_THROUGHPUT_FRACTION=0.85` | 在线 HTTP 推理端点 $\to$ 真实饱和拐点并发度与 Goodput | 属于后验实测搜索，需要占用真实 GPU 跑完整压测 |
| **`InseeFrLab/auto-tuning-vllm` & `vllm/auto_tune.sh`** | 在线自动化调优脚本 | Optuna（TPE/GP/BoTorch/NSGA-II）带约束黑盒寻优 / Bash 网格扫描 | 支持 `--random-prefix-len`、`MAX_LATENCY_ALLOWED_MS`、`gpu_mem_util 0.98..0.90` | 候选参数空间 + 线上压测反馈 $\to$ 最优批大小/显存比 | 全空间黑盒探索耗时极长，依赖收敛剪枝策略 |
| **`bytehola/llm-speed-test` / `llm-api-speedtest` / `llmapibenchmark` / `ai-gateways-benchmark`** | 多供应商实测实验室 | 并发 SSE 探测 TTFT/Prompt TPS/Decode TPS + 六阶段网络拆解 + 冷热 Socket 隔离 | `avg10` 排行（`llm-speed-test`）；仅导入 CPA YAML 无回写 | 多供应商 API Key / CPA YAML $\to$ 实时对比榜与趋势库 | 多数**停留在只测不调**，无自动回写网关配置闭环 |
| **`PicadoLabs/ai-model-router`** | 自适应智能网关路由器 | 五维静态加权 + 持久化 Beta 后验 **Thompson Sampling** | `0.35Q + 0.25C + 0.20S + 0.15Cap + 0.05R` | 每次请求任务类型 + 实时延迟/成功率 $\to$ 在线探索选路决策 | 未集成前缀缓存亲和率因子或 Prompt 成本折扣项 |

---

### 6.2 对自建 LLM 网关架构演进的五条直接可落地建议

综合上述 33 个项目的一手设计得失，若要打造兼具**“最佳缓存容量自动推荐 + 感知路由权衡 + 自动测速选优配置器”**的新一代 LLM 网关与调优系统，建议按以下五条高性价比路线直接落地：

1. **构建“Prefix-Aware MRC + SLA 约束”双目标缓存容量顾问（借鉴 `LMCache` + `aiconfigurator`）**：
   - 不要用传统对象缓存的独立栈距离做 KV 容量建议。直接复用 `LMCache/lmcache/tools/cache_simulator/trace_analyze.py` 的思路，对网关脱敏后的请求历史按标准 Token 分块做 **$\sqrt{2}$ 步长容量-chunk 二维扫频**产出命中率收益曲线；同时引入 `aiconfigurator` 的 `of_total` 显存预算约束与 **`--prefix` 非对称算子公式**（GEMM 减负但 Attention 仍计读取），计算“给 Prefix 池扩容 1 GiB 减少重算的 TTFT 收益”何时低于“挤压运行批并发度造成的排队惩罚”，提出待负载验证的候选 `gpu_memory_utilization` 与 CPU Offload 配比。
2. **用“动态到达间隔分布”打通 Prompt Cache `5m` vs `1h` 自适应 TTL 引擎（补齐 `autocache` + `claude-cache-analyzer` 的空白）**：
   - 把上述两种参照都放进策略：按前缀的复用间隔估计短 TTL 写入次数、长 TTL 写入次数和两者的命中数，再与不缓存的费用比较。历史分布能支持建议，不能保证下一轮仍命中。
3. **采用“无量纲效用 + TTFT 破粘闸门”作为集群感知路由标配（融合 `production-stack` + `llm-d` + `aibrix`）**：
   - 纯按“最长前缀匹配”必然导致热点节点 OOM 或尾延迟爆炸。推荐在路由核心中组合使用：`llm-d-kv-cache` 的介质折现打分（`gpu=1.0, cpu=0.8`） $\to$ `production-stack LoadAwareRouter` 的无量纲权衡项（$\text{hitRatio} - \beta \cdot \text{relLoad}$） $\to$ 叠置 `aibrix` 的标准差/1.5x 防过载护栏及 `llm-d prefix-cache-affinity-filter` 的 **`MaxTTFTPenaltyMs` 硬破粘闸门**，兼顾极限吞吐与 P99 尾部 SLA。
4. **用“阶段网络拆解 + 2-Segment OLS 拐点”改造自建 Speed Lab（融合 `ai-gateways-benchmark` + `guidellm` + `llmapibenchmark`）**：
   - 在多供应商实测器中，务必拆出 `dns/tcp/tls/ttfb/ttft/e2e` 与 `Prompt TPS / Decode TPS`，消除 CDN/代理抖动和加密推理段对上游真实算力评价的掩盖；对自部署/私有云节点压力测试，则直接接入 `guidellm` 的 2-Segment OLS 四门槛拐点检测与 `wilson_interval` Goodput 寻优替代拍脑袋手写阶梯并发。
5. **通过 Beta-Bernoulli Thompson Sampling 打通“测速 $\to$ 网关路由权重”自动回写闭环（借鉴 `PicadoLabs/ai-model-router`）**：
   - 打破当前绝大多数测速面板“测完只给人工看排行榜（如 `llm-speed-test` 止步于 YAML 导入）”的断点：把多商测速探针的输出（TTFT、TPS、缓存命中率折现、成功率）持续喂给网关控制面的先验状态，线上请求路由则辅以轻量级 Thompson Sampling 概率探索（或微流量 Shadow Probe），让最佳渠道权重随全天算力波谷波峰自动漂移收敛。
