---
title: "第 5 章　推理引擎：和 BERT 时代的推理框架有什么不同"
date: 2026-10-17
author: fangd123
license: CC BY-NC-SA 4.0
category: 工程实践
series: 本地大模型架构与推理框架详解
series_order: 6
tags:
  - 工程实践
  - 本地大模型架构与推理框架详解
ai_assisted: true
status: published
---

# 第 5 章　推理引擎：和 BERT 时代的推理框架有什么不同

> 你熟悉的 BERT 部署大概是：导出 ONNX / TorchScript，放进 Triton 或 TorchServe，攒一个 batch，跑一次前向，拿到分类结果。
> 大模型推理引擎（llama.cpp、vLLM、SGLang、Paiton、Strata……）解决的是另一类问题。
> 这一章讲：
> - 为什么“生成文本”和“给文本分类”在工程上是两回事；
> - KV cache、连续批处理、分页注意力、前缀缓存这些概念是为了解决什么；
> - 主流引擎各自的定位；
> - llama.cpp 的内部结构，以及本机为什么 Vulkan 比 ROCm 快；
> - 2026 年这批“专用引擎”（Paiton、Strata、R9V、NInfer）走的是什么路线。
>
> 推测解码（MTP、DFlash2）单独放第 6 章。
>
> 来源标记：【官方】= 项目文档；【源码】= 直接读的代码或提交记录；【论文】；【作者自报】；【社区】；【本机实测】；【推断】= 我的理解。

---

## 5.1 根本区别：一次前向 vs 一个循环

### 5.1.1 BERT：一次前向就结束

```
输入：[CLS] 今 天 天 气 很 好 [SEP]（比如 128 个 token）
       ↓ 一次前向，所有 token 同时算
输出：分类概率
```

- 每个请求的计算量固定，只和输入长度有关；
- 一个 batch 里的请求同时开始、同时结束；
- 输入形状可以固定（padding 到 128 或 512），很适合 TensorRT 这类“编译成固定引擎”的做法；
- 只有一个延迟指标。

### 5.1.2 生成式大模型：一个 token 一个 token 地循环

```
输入：“请写一首诗”（比如 2000 个 token 的提示）
  ↓ 第 1 步：把 2000 个 token 一次性算完        ← 预填充（prefill）
  → 输出第 1 个 token “春”
  ↓ 第 2 步：只算“春”这 1 个新 token            ← 解码（decode）
  → 输出“风”
  ↓ 第 3 步：只算“风”
  → ……
  重复几百到几万次，直到输出结束符
```

生成 K 个 token 就要串行跑 K 次模型【论文：Leviathan 2023】。这带来了 BERT 时代没有的四个问题：

1. **两个阶段性质完全不同**（5.1.3 节）；
2. **要缓存历史**，否则每一步都要把前面所有 token 重算一遍（5.2 节）；
3. **请求长度不可预测**：有人要 10 个 token，有人要 30000 个，固定 batch 会让短请求干等（5.3 节）；
4. **指标变成三个**（5.1.4 节）。

### 5.1.3 预填充算力受限，解码带宽受限

这是理解整个推理引擎设计的钥匙。

**预填充**：2000 个 token 同时乘同一个权重矩阵，是“矩阵 × 矩阵”。每从显存读一个字节的权重，可以做几千次乘加。瓶颈是 GPU 的算力。这和 BERT 的前向一样。

**解码**：每个请求每一步只有 1 个新 token，是“矩阵 × 向量”。每读一个字节的权重只做约 1–2 次乘加，GPU 的计算单元大部分时间在等数据。瓶颈是**显存带宽**。

所以单请求解码速度有一个简单的上限：

```
解码速度上限（token/s）≈ 显存带宽（GB/s）÷ 每个 token 要读的字节数（GB）
```

本机例子【本机实测 + 规格】：

- R9700 显存带宽约 640 GB/s（官方规格，本文没有逐项核实）；
- Qwen3.8-27B UD-Q6_K 每个 token 要读约 20.9 GB 的权重；
- 上限 ≈ 640 ÷ 20.9 ≈ **30.6 token/s**；
- 实测不开推测解码 24.7–25.3 token/s，是上限的约 82%。

stew675 在 2 × R9700 上跑 Q8_0、不开推测解码，四类内容（推理、散文、代码、逐字复述）都是 29.9–30.1 token/s【作者自报，PR #27210】。**速度和内容无关**，这是带宽受限的典型特征。

这个公式能解释后面几乎所有的加速手段：

| 手段 | 怎么绕开带宽墙 | 章节 |
|---|---|---|
| 量化 | 每个 token 读的字节变少 | 第 3 章 |
| MoE | 每个 token 只读激活的专家 | 第 2 章、第 7 章 |
| 批处理 | 读一次权重，同时给多个请求用 | 5.3 节 |
| 推测解码 | 读一次权重，同时验证多个 token | 第 6 章 |

### 5.1.4 三个指标互相拉扯

| 指标 | 含义 | 主要由什么决定 |
|---|---|---|
| **TTFT**（首字时间） | 发出请求到看到第一个字 | 预填充速度 × 提示长度 |
| **TPOT / 解码速度** | 之后每个字的间隔，或每秒多少 token | 带宽、量化、推测解码 |
| **总吞吐** | 所有并发请求加起来每秒多少 token | 批处理效率 |

例：Paiton 文档写 64K 提示在 3 位量化下要约 8 秒才出第一个字【作者自报】；本机 43K 提示在 rdna-boosts 上预填充约 930 token/s，也就是首字约 46 秒【本机实测】。

单人聊天看 TTFT 和解码速度；批量处理看总吞吐。这两个目标经常冲突：并发越多，总吞吐越高，但每个人分到的速度越低（本机 Strata 4 并发时总吞吐 89.7，每路只有 24.5，单路时是 79.8【本机实测】）。

---

## 5.2 KV cache：用显存换掉重复计算

### 5.2.1 它是什么

注意力的计算是：当前 token 的 Q 去和**所有历史 token** 的 K、V 做运算。历史 token 的 K、V 一旦算出就不会变，所以存起来，下一步直接用。这就是 KV cache。

BERT 不需要它，因为一次前向就结束了。

### 5.2.2 它有多大

```
每个 token 的 KV 字节数 = 2（K 和 V）× 注意力层数 × KV 头数 × 每头维度 × 每个元素的字节数
```

| 模型 | 注意力层 | KV 头 | 头维度 | 每 token（bf16） | 131K 上下文 |
|---|---|---|---|---|---|
| Qwen3-32B（传统全注意力） | 64 | 8 | 128 | 256 KiB | 32 GiB |
| **Qwen3.8-27B**（只有 1/4 是注意力层） | 16 | 4 | 256 | **64 KiB** | 8 GiB |
| 同上，q8_0 KV | | | | 约 34 KiB | 约 4.3 GiB |

（第 2 章 2.3.3 节有推导。）

另外 48 层 Gated DeltaNet 每个请求有一个**固定大小**的状态，约 144 MiB，和上下文长度无关。

这就是 27B 能在单张 32 GB 卡上开 131K 上下文的原因：权重 20.9 GB + KV 约 4–8 GB + 其他。如果是传统全注意力的 32B 模型，131K 的 KV 本身就要 32 GiB，放不下。

**多个请求时 KV 是按请求叠加的**：4 个请求各 32K = 128K 的总量。这是并发数的硬约束。

### 5.2.3 KV cache 带来的新问题

1. **长度事先未知**：一个请求最后会生成 100 还是 30000 token，开始时不知道。预先按最大长度给每个请求留空间，会浪费大量显存。
2. **碎片**：PagedAttention 论文测量，当时的系统里只有 20.4%–38.2% 的 KV 显存真正存了 token 状态，其余被预留、内部碎片、外部碎片浪费掉【论文】。
3. **混合模型的复杂性**：Qwen3.8 这类模型既有 KV（可以按 token 截取），又有递归状态（只能整体保存某个位置的快照）。NInfer 文档的说法：复用一个前缀，需要那个位置的完整递归状态加连续的 KV【源码】。所以前缀缓存、推测解码的回滚都比纯注意力模型麻烦。

---

## 5.3 批处理：从“静态 batch”到“连续批处理”

### 5.3.1 静态 batch 的问题

BERT 的做法：攒 32 个请求，一起跑完，一起返回。

生成模型这样做：

```
请求 A：生成 10 token    ████
请求 B：生成 500 token   ████████████████████████████████
请求 C：生成 50 token    ████████
                          ↑ A、C 早就结束了，但要陪 B 跑完才能返回
                            新请求也只能等这一批全部结束
```

### 5.3.2 连续批处理（continuous batching）

**Orca**（OSDI'22）提出：调度单位从“请求”改成“一步”【论文】。

```
每一步：
  - 已经结束的请求立刻移出、返回
  - 新到的请求立刻加入
  - 这一步一起算所有活跃请求的下一个 token
```

GPT-3 175B 上同延迟下吞吐是当时 FasterTransformer 的 36.9 倍【论文】。现在所有主流引擎都用它，llama.cpp server 默认打开 `--cont-batching`【官方】。

**为什么批处理能提高吞吐**：解码是带宽受限的。一次读权重，原本只服务 1 个请求，现在服务 8 个，计算量涨 8 倍但读取量几乎不变（KV 部分除外），GPU 本来空闲的算力被用上了。

### 5.3.3 PagedAttention（vLLM）

借鉴操作系统的虚拟内存分页【论文，SOSP'23】：

- 把 KV cache 切成固定大小的块（比如每块 16 个 token）；
- 每个请求有一张“块表”，记录它的逻辑位置对应哪些物理块；
- 按需分配，用完就还，几乎没有碎片；
- 同一提示的多个采样可以共享前缀块。

效果：同样延迟下吞吐提高 2–4 倍【论文】。

llama.cpp 没有用块表，而是一个统一的 KV 单元池加“槽位（slot）”分配（`--kv-unified`）【官方】。本机 Flash-Next 双卡测试时，4 个槽平分 65536 上下文，每个只有 16K，32K 的提示就失败了；加 `--kv-unified` 让所有槽共享整个池后 43.5K 的提示能跑通【本机实测】。

### 5.3.4 前缀缓存

很多请求的开头是一样的：同一个系统提示、同一份长文档、多轮对话的历史。前缀缓存把这些 KV 留着，下次直接复用，跳过预填充。

| 引擎 | 做法 |
|---|---|
| vLLM | 每个满块算哈希 = 哈希(父块哈希, 本块 token, LoRA id 等)，查表复用【官方】 |
| SGLang | **RadixAttention**：用基数树组织所有请求的 KV，按 LRU 淘汰，调度时优先排能命中缓存的请求【论文】 |
| llama.cpp | `--cache-reuse`（通过 KV 平移复用）、`--slot-save-path`（把槽的 KV 存盘）【官方】 |

对智能体、多轮对话、代码助手这类“反复发送长上下文”的场景，前缀缓存的收益最大。Paiton 的 `long-kv4` 模式就是为编码智能体做的，号称能缓存约 629K token，还能溢出到内存或 SSD【作者自报】。

### 5.3.5 分块预填充（chunked prefill）

一个 100K 的长提示预填充要几十秒，这期间其他人的解码会卡住。

做法：把长提示切成小块（比如每块 2048 或 8192 token），和别人的解码步混在同一批里跑【论文：Sarathi-Serve；官方：vLLM V1 默认开启】。vLLM 的调度器先排解码，再用剩下的 token 预算排预填充【官方】。

Strata 也用最多 8192 token 的分块预填充【官方】。

---

## 5.4 内核层面的优化

“内核（kernel）”指在 GPU 上执行的一个具体计算函数，比如一次矩阵乘、一次注意力。

### 5.4.1 FlashAttention

普通注意力要把 N × N 的注意力矩阵写到显存再读回来。FlashAttention【论文 2205.14135、2307.08691】把计算切成小块在片上缓存里完成，**不把 N × N 矩阵写回显存**。结果完全相同，但长上下文时快得多、省得多。

llama.cpp 里是 `-fa on|off|auto`；**KV 用量化类型（q8_0 等）时必须开 FA**【官方】。

### 5.4.2 量化矩阵乘：MMQ、MMVQ 和“先反量化再 GEMM”

权重是量化的（第 3 章），计算时有两种做法：

| 做法 | 过程 | 适合 |
|---|---|---|
| **反量化 + GEMM** | 先把量化权重还原成 FP16 / BF16 大矩阵，再调厂商的 BLAS 库 | 大 batch 预填充（BLAS 库算得快），但多一次显存往返 |
| **MMQ**（量化矩阵乘） | 激活也量化成 8 位，直接用整数点积在量化权重上算 | 中等 batch，省显存 |
| **MMVQ**（量化矩阵 × 向量） | 解码专用：一边读量化权重一边算 | 单 token 解码 |

llama.cpp 默认在有 int8 矩阵单元的 GPU 上用 MMQ【官方】。有意思的是，stew675 的 rdna-boosts 补丁反过来，给 R9700 的长预填充加了 **MMB**（先反量化到 BF16 再走 WMMA 矩阵单元），因为在 RDNA4 上这样更快【源码】。这说明“哪种内核最快”取决于具体硬件，没有通用答案。

这也解释了第 3 章的现象：GSQ-RCO IQ3_S 文件只有 Q6_K 的一半，代码解码却只快约 19%，JSON 反而更慢。i-quant 的反量化要查码本，在 Vulkan 上开销大，省下的带宽被计算吃掉了【本机实测 + 推断】。

### 5.4.3 融合内核

把几个小操作合成一个内核（比如 RMSNorm + 乘权重，或 MoE 的 gate + up + 激活），减少读写显存的次数和启动开销。rdna-boosts 的第 8 块补丁就是一组融合内核【源码】。

### 5.4.4 CUDA Graphs

解码时每一步要启动几百个小内核，CPU 发起启动本身的开销就很可观。CUDA Graphs 把整步的内核序列“录下来”，之后一次重放。

BERT 时代 batch 形状固定，天然适合录制。LLM 解码每一步的 batch 大小都可能变，所以 vLLM 对一组离散的 batch 大小预先录制，预填充和解码分开录【官方】。

---

## 5.5 主流引擎地图

| 引擎 | 定位 | 硬件 | 核心技术 | 模型格式 |
|---|---|---|---|---|
| **vLLM** | 数据中心高并发服务的事实标准 | NVIDIA 为主，ROCm、其他通过插件 | PagedAttention、连续批处理、前缀缓存、分块预填充、CUDA Graphs、多种推测解码 | HF safetensors（也能读部分 GGUF） |
| **SGLang** | 和 vLLM 同级，强调前缀复用和结构化输出 | NVIDIA 为主 | RadixAttention、压缩有限状态机（JSON 一次解多个 token） | HF safetensors |
| **TGI**（Hugging Face） | 已进入维护模式，官方建议迁到 vLLM / SGLang / llama.cpp【官方】 | — | — | — |
| **TensorRT-LLM** | NVIDIA 官方，把模型编译成针对特定 GPU 的引擎 | 仅 NVIDIA | 和 BERT 时代的 TensorRT 同一思路，加上 in-flight batching、分页 KV | 编译产物 |
| **llama.cpp** | 单机、低并发、异构硬件、量化种类最多 | 几乎所有：CUDA、HIP、Vulkan、Metal、CPU…… | ggml 张量库、GGUF、MMQ、多后端混用 | GGUF |

和 BERT 时代对照：

| BERT 时代 | 大模型时代 | 差别 |
|---|---|---|
| Triton Inference Server | vLLM / SGLang | Triton 是通用的“模型托管 + 动态 batch”；vLLM 专门为自回归生成和 KV 管理设计 |
| TensorRT | TensorRT-LLM | 同样是编译，但要额外处理 KV、可变长度 |
| ONNX Runtime CPU 推理 | llama.cpp | 都能在 CPU / 普通硬件上跑；llama.cpp 主打低位量化 |

本机两张卡的情况决定了选择：R9700 是 AMD RDNA4，vLLM 原生支持弱；3090 是 Ampere，很多新方案（NVFP4、NInfer）要 RTX 40/50。所以主力是 llama.cpp，加上几个 R9700 专用的引擎。

---

## 5.6 llama.cpp 内部结构

### 5.6.1 三层

```
llama-server（HTTP 服务、OpenAI 兼容 API、槽位调度、采样、推测解码）
     ↓
libllama（模型结构：每种架构怎么搭计算图，比如 qwen35 的 GDN + 注意力）
     ↓
ggml（张量库：计算图 + 各硬件后端的内核）
     ├─ CUDA 后端（NVIDIA）
     ├─ HIP 后端（AMD，用 hipcc 编译 CUDA 后端的同一份源码）
     ├─ Vulkan 后端（跨厂商，GLSL 着色器编译成 SPIR-V）
     ├─ Metal、SYCL、CPU……
```

两点和直觉不同的事实【源码】：

- **HIP 后端不是独立实现**。它的 CMakeLists 直接 glob `ggml-cuda/*.cu` 用 hipcc 编译，靠宏（`AMD_WMMA_AVAILABLE` 等）区分硬件。所以 CUDA 后端的优化会自动部分惠及 AMD，但它们主要是为 NVIDIA 调的。
- **Vulkan 后端是完全独立的一套**。编译时检测驱动支持哪些扩展（`coopmat` 协作矩阵、整数点积等），生成对应的着色器变体。

多个后端可以编进同一个程序，运行时用 `--device` 选择【官方】。本机 Flash-Next 双卡就是用 Vulkan 同时驱动 R9700 和 3090（`DEV=Vulkan1,Vulkan0`）【本机实测】。CUDA 和 HIP 能不能在同一进程里一起用，官方文档没写，研究时也没找到明确说法。

### 5.6.2 GGUF 和 mmap

GGUF 是单文件格式：头部是带类型的键值元数据（架构、层数、分词器、聊天模板……），后面是张量数据。设计目标之一是能直接 **mmap**（内存映射）：文件不用整个读进内存，访问到哪页由操作系统按需调入，多个进程还能共享【官方】。

文件名后缀 `mmproj-` 是多模态投影器，`mtp-` 是 MTP 头的旁挂文件【官方】。

本机用到 mmap 的例子：Flash-Next 的 28.8 GB n-gram 表分片用 `-lm mmap` 放在磁盘上按需读，进程常驻内存只有 1.9 GB；第一次请求时要从磁盘懒加载，2.7K 提示的预填充只有 189 token/s，第二次就到了 1321【本机实测】。

### 5.6.3 多 GPU 切分

| `-sm` 模式 | 做法 | 通信 | 状态 |
|---|---|---|---|
| `layer`（默认） | 流水线：卡 1 放前 N 层，卡 2 放后面的层 | 每个层边界传一次激活，很少 | 稳定 |
| `row` | 按行切每个矩阵 | — | 已弃用 |
| `tensor` | 类似张量并行：每层的权重和 KV 都切到各卡 | 每层多次 all-reduce | 实验性，要 `-fa on` |

官方建议：layer 对预填充好、对慢互联宽容；tensor 对解码好，但要快互联【官方】。

本机两张卡只靠 PCIe 连接，还是不同厂商，所以用 layer 模式加 `--tensor-split 27,20` 按显存比例分层。代价是两张卡**串行**工作：卡 1 算完前一半，卡 2 才能算后一半，单请求解码不会比单卡快。本机 Flash-Next 双卡单请求只有 41 token/s，是 Strata 单卡的 35–50%（还没有 MTP、两卡都限了功耗）；但 4 并发总吞吐从 37.8 涨到 68.1，扩展性比 Strata 好【本机实测】。

### 5.6.4 MoE 专家卸载

| 参数 | 作用 |
|---|---|
| `--cpu-moe` | 所有专家权重放内存，CPU 计算 |
| `--n-cpu-moe N` | 前 N 层的专家放内存 |
| `-ot <正则>=<设备>` | 按张量名指定放哪里，最灵活 |
| `--moe-cache-mib N` | 为放在内存的专家在显存里建一个缓存（较新） |

原理：MoE 每个 token 只用少数专家（Flash-Next 是 512 选 10 + 1 共享），注意力、路由、共享专家放 GPU，大量“冷”专家放内存【官方】。

### 5.6.5 llama-server 的几个关键参数

| 参数 | 含义 |
|---|---|
| `-np N` | 并行槽位数（同时服务几个请求） |
| `-c N` | 总上下文长度 |
| `--kv-unified` | 所有槽共享一个 KV 池（不平分） |
| `-b` / `-ub` | 逻辑 / 物理 batch，影响预填充 |
| `-ctk` / `-ctv` | KV 的数据类型：f16（默认）、bf16、q8_0、q4_0…… |
| `--spec-type draft-mtp` | 用模型自带的 MTP 头做推测解码（第 6 章） |
| `--jinja` | 用 GGUF 里的 Jinja 聊天模板 |

---

## 5.7 AMD 和 NVIDIA 的软件栈

### 5.7.1 对照

| | NVIDIA | AMD |
|---|---|---|
| 编程模型 | CUDA | HIP（CUDA 风格的 C++ 方言，代码几乎可以一对一转换） |
| 整个平台 | CUDA Toolkit | ROCm |
| 矩阵库 | cuBLAS、cuBLASLt | rocBLAS、hipBLAS、hipBLASLt |
| 矩阵单元 | Tensor Core | 数据中心卡 CDNA：MFMA（wave64）；消费 / 工作站卡 RDNA：**WMMA**（wave32） |
| 跨厂商选项 | Vulkan | Vulkan |

R9700 的架构代号是 **gfx1201**（RDNA4，和 RX 9070 系列同一颗芯片），原生 wave32【官方：LLVM 文档】。

### 5.7.2 为什么消费级 AMD 卡支持总是慢半拍

AMD 的 ROCm 长期优先服务数据中心的 CDNA 卡（Instinct 系列）。RDNA2 到 ROCm 5.0 才首次获得官方支持（还只限几款工作站卡），RDNA4 到 ROCm 6.4.1（2025 年 5 月）才正式支持【社区：Phoronix 新闻，版本号请自行核对】。通用库对 RDNA4 的优化也晚。

所以 2026 年出现了一批“自己给 gfx1201 写内核”的项目：Paiton、R9V、Radiance 系 vLLM 分支、stew675 的 rdna-boosts、Strata 的 HIP 后端【推断：这是这些项目共同的动机】。

### 5.7.3 本机实测：Vulkan 比 ROCm 快

【本机实测】同一块 R9700、同一份 UD-Q6_K、131K 上下文、q8_0 KV、MTP 草稿 3，关思考；解码是 3 次中位数：

| 配置 | 解码（代码 / JSON / 英文 / 中文） | 预填充 2.7K / 10.9K / 43K | 43K 提示后的解码 | 4 并发合计 |
|---|---|---|---|---|
| rdna-boosts（HIP）+ bf16 KV | **70.4 / 76.4 / 51.5 / 45.0** | 946 / 1039 / 933 | 48.2 | 55.0 |
| rdna-boosts（HIP） | 67.6 / 74.8 / 49.5 / 42.8 | 943 / 1044 / 935 | 50.4 | 51.9 |
| 主线 Vulkan（现用） | 63.4 / 71.0 / 46.5 / 39.2 | 770 / 836 / 729 | 40.6 | 49.1 |
| 主线 ROCm，`GPU_MAX_HW_QUEUES=1` | 52.6 / 57.0 / 38.6 / 33.4 | 691 / 778 / 720 | 31.3 | — |

结论：

- **主线 ROCm 全面慢于 Vulkan**，解码低 13–15%。重测也一样，是 ROCm 构建本身的问题；
- **rdna-boosts 是最快的 llama.cpp**：解码比 Vulkan 快 5–9%，预填充快 22–28%，43K 长提示后的解码快 24%；
- 再把 KV 换成 bf16 还能快约 4%，而且没有 KV 量化误差；代价是 KV 显存翻倍；
- 同一份权重，答案质量不受引擎影响（但 rdna-boosts 本次没单独跑评测）。

社区数据和这个方向一致：Phoronix 测 R9700，文本生成 Vulkan 快于 ROCm 7.1，部分模型的预填充 ROCm 更快；另一份 RX 9070 XT 测试 Vulkan 解码约 220 对 HIP 179【社区】。

**为什么**【推断，没有找到一手资料专门解释】：

- 解码是带宽受限，内核只要能把带宽吃满就行，Vulkan 的着色器（Mesa RADV 驱动 + ACO 编译器）做得到；
- 预填充是算力受限，要用好 WMMA 矩阵单元、精细分块，HIP 直接继承 CUDA 后端的那套优化，有时更好；
- 但那套优化是为 NVIDIA 调的，在 RDNA4 上没调好，这正是 rdna-boosts 补的短板。

### 5.7.4 rdna-boosts 补了什么

stew675 的补丁集（独立仓库 `stew675/llama-cpp-rdna-boosts`，MIT；整合后的源码在 `stew675/llama.cpp` 的 `rdna-boosts` 分支）当前是 12 块补丁【源码】：

| 块 | 内容 | 对本机的意义 |
|---|---|---|
| 01 | RDNA4 之间的自定义 all-reduce | 本机只有一张 R9700，用不上 |
| 02 | 自适应 MTP 草稿深度 | 第 6 章 |
| 03 | **Gated DeltaNet 的分块预填充**（BF16 / WMMA） | 专门针对 Qwen3.8 的 GDN 层，预填充快的主因之一 |
| 04 | **BF16 KV + 原生 BF16 FlashAttention** | 上表 +4% 的来源 |
| 06 | MMB：长预填充先反量化到 BF16 走 WMMA | 预填充 |
| 07 | k-quant 加速（Q6_K 的 MMQ 等） | 本机用的就是 Q6_K |
| 08 | 融合内核 | 解码 |
| 09 | 内存驻留 MoE + 专家缓存 | MoE 模型 |
| 10 | qwen4exp（Flash-Next 架构） | — |

作者还强调“贪心纯度”：普通解码和 MTP 验证批在草稿深度 ≤ 7 时逐字节一致【源码】。

它是个人维护的分支，没合进主线。

---

## 5.8 2026 年的专用引擎：用通用性换速度

| 引擎 | 做什么 | 硬件 | 开源 | 本机 |
|---|---|---|---|---|
| **Paiton**（Eliovp-BV） | vLLM 插件 + 自有量化和内核（MXFP4、3 位 + DFlash2） | 只调过 R9700 | 插件 Apache-2.0，**编译器专有**，仓库里是编译好的产物 | 已测 |
| **Strata**（Niko1221） | Flash-Next 专用的专家分层卸载引擎 | HIP 或 CUDA，不能混用 | MIT | 已测 |
| **R9V**（Dyluhn） | Flash-Next 形状特化：vLLM 分支 + gfx1201 内核 + 专家卸载 + 4 token MTP | 2 × R9700 | Apache-2.0 | 本机只有一张，没测 |
| **NInfer**（Neroued） | 从零写的 C++/CUDA 单卡引擎 | 只支持 RTX 5090 | Apache-2.0 | 不可用 |
| **ninfer-fusion-kvmem** | NInfer 的下游分支，有 3090 版 | 3090 / 40 / 50 系 | Apache-2.0 | 没测（作者自称不保证正确） |
| **Project Maya** | GLM-5.3-Flash 专用，基于 Strata 和 ggml | NVIDIA 为主 | MIT | 没测 |
| **KTransformers**（清华） | 学术路线的 CPU/GPU 混合 MoE 推理，已进 SGLang | Intel AMX、AVX、ROCm…… | Apache-2.0 | 没测 |

共同点【推断】：**模型形状特化 + 量化格式特化 + 硬件特化**，牺牲通用性换取单机极限性能。和 vLLM、llama.cpp 的通用路线正好相反。风险是维护靠个人或小团队，出问题难排查。

### 5.8.1 Paiton：本机最快，但掉分

- 是标准 vLLM 加 Python 插件（vLLM 的 `entry_points` 插件机制允许外部注册新硬件平台和模型），**不是**一个新引擎【官方】；
- Qwen3.8-27B 用 MXFP4 或 3 位权重 + DFlash2 草稿；
- 本机实测：代码 / JSON / 英文 / 中文解码 176 / 200 / 84 / 57 token/s，约为 llama.cpp Vulkan 的 2.8 倍；8 并发 349 token/s；
- 但**知识题显著掉分**：MMLU-Pro 直答 61.9 对 64.6（p = 0.002），CoT 77.7 对 80.2（p = 0.034），IFEval 低约 4 分【本机实测】。第 3 章 3.8 节解释了原因：它的 MXFP4 是从 NVFP4 加载时转换的，转换有损。

一个细节：Paiton 文档提示 `GPU_MAX_HW_QUEUES=1` 能让 RDNA4 解码快约 4 倍【作者自报】，但在本机主线 ROCm 上这个设置反而更慢（上表最后一行）。环境变量的效果和具体引擎绑定，不能照搬。

### 5.8.2 Strata：专家分三层放

Flash-Next 总参数约 180B，显存放不下。Strata 的分层【官方】：

```
显存（GPU）：注意力 + DeltaNet 层、路由、共享专家、输出头、MTP 层、KV cache
            + 专家缓存：剩余显存都用来放“最常用的专家”（随对话热度自适应）
内存（RAM）：全部 24,576 个专家（48 层 × 512），锁页
            未命中缓存的专家由 CPU 就地计算，或经 PCIe 搬到 GPU 算
SSD：      28.8 GB 的 n-gram 嵌入表，每个 token 只读几行
```

在当前层算注意力的同时，通过 PCIe 预取下一层可能用到的专家。

本机实测：启动 42 秒（从磁盘读 40 GB 专家进内存，再把 14660 个最常用的专家、23.8 GB 放进显存），测速时专家缓存命中率平均 95%。解码 113 / 122 / 82 / 78 token/s，比 27B 在 llama.cpp 上的最快配置快 60–70%【本机实测】。

和 llama.cpp `--n-cpu-moe` 的区别：后者是**静态**划分（哪些层放内存是固定的）；Strata 是**动态**热专家缓存加预取。Strata 文档还提醒：命中率更高的运行不一定更快，不要只看命中率【官方】。

社区对 Strata 有争议：改写过 Git 历史、被指刷帖、低位量化写代码 bug 多【社区】。本机测到它和 llama.cpp 双卡的答案质量没有显著差别（温度 0 下 HumanEval p = 1.0，MMLU-Pro 直答 p = 0.12）【本机实测】，所以加速没有以质量为代价。

### 5.8.3 kvmem：近似注意力，不是无损

ninfer-fusion-kvmem 把“显存里的 KV 池大小”“逻辑上下文长度”“内存里的 KV 预算”分开配置：显存只保留一个常驻窗口，其他页放到内存，**按内容相关性挑选**一部分页取回参与注意力【作者自报】。

这和 vLLM 的 KV 换出换入不同：vLLM 整块换回来，注意力是精确的；kvmem 只取一部分，是近似的。README 自己也写明不保证质量和全注意力一样【官方】。

---

## 5.9 本机怎么选引擎

| 需求 | 选择 | 理由 |
|---|---|---|
| 默认主服务，要稳 | 主线 llama.cpp Vulkan + MTP | 证据最充分，131K 上下文，并发好 |
| 想再快一点、同样权重 | llama.cpp rdna-boosts（HIP）+ bf16 KV | 解码 +5–9%，长提示 +24%，个人分支 |
| 单人交互、要快 | Flash-Next + Strata | 质量持平 27B，解码快 60–70%，不适合并发 |
| 批量、多人、代码 / JSON | Paiton vLLM MXFP4 | 解码约 2.8 倍，但知识题显著低约 2.5 分，内核闭源 |
| 没有 HIP 环境，或要用 3090 | llama.cpp Vulkan 双卡 | 通用，单请求慢 |

---

## 本章小结

1. 生成是循环：预填充算力受限（像 BERT），解码带宽受限（BERT 没有）。
2. 解码速度上限 ≈ 带宽 ÷ 每 token 读的字节数。几乎所有加速手段都是在减少读取或摊薄读取。
3. KV cache 是生成模型特有的显存大户；Qwen3.8 的 3:1 混合结构让它只有传统模型的 1/4。
4. 连续批处理、PagedAttention、前缀缓存、分块预填充，都是为“长度不可预测的循环”设计的。
5. llama.cpp 适合本机这种异构、低并发、AMD 为主的环境；R9700 上 Vulkan 快于主线 ROCm，rdna-boosts 更快。
6. 专用引擎速度惊人，但要逐个验证质量（Paiton 掉分，Strata 不掉）。

---

## 参考资料（本章）

**入门读物（建议先读）**
- kipply，*Transformer Inference Arithmetic*（用算术强度解释预填充 / 解码和 KV 大小）：https://kipp.ly/transformer-inference-arithmetic/
- Hugging Face，*Optimizing LLMs for Speed and Memory*：https://huggingface.co/docs/transformers/llm_tutorial_optimization
- vLLM 博客，*Inside vLLM: Anatomy of a High-Throughput LLM Inference System*：https://blog.vllm.ai/2025/09/05/anatomy-of-vllm.html
- vLLM 首篇博客：https://blog.vllm.ai/2023/06/20/vllm.html

（以上四篇研究时只确认了能访问和标题，没有通读。）

**论文**
- Orca（连续批处理）：https://www.usenix.org/conference/osdi22/presentation/yu
- PagedAttention / vLLM：https://arxiv.org/abs/2309.06180
- Sarathi-Serve（分块预填充）：https://arxiv.org/abs/2403.02310
- SGLang / RadixAttention：https://arxiv.org/abs/2312.07104 ；博客：https://lmsys.org/blog/2024-01-17-sglang/
- FlashAttention：https://arxiv.org/abs/2205.14135 ；FlashAttention-2：https://arxiv.org/abs/2307.08691
- KTransformers（SOSP'25）：https://madsys.cs.tsinghua.edu.cn/publication/ktransformers-unleashing-the-full-potential-of-cpu/gpu-hybrid-inference-for-moe-models/

**官方文档**
- llama.cpp server 参数：https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md
- llama.cpp 多 GPU：https://github.com/ggml-org/llama.cpp/blob/master/docs/multi-gpu.md
- llama.cpp 编译与后端：https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md
- GGUF 格式：https://github.com/ggml-org/ggml/blob/master/docs/gguf.md
- vLLM 设计文档（插件、前缀缓存、CUDA Graphs）：https://github.com/vllm-project/vllm/tree/main/docs/design
- AMD WMMA on RDNA3：https://gpuopen.com/learn/wmma_on_rdna3/
- LLVM AMDGPU（gfx1201 的定义）：https://llvm.org/docs/AMDGPUUsage.html

**社区实测**
- llama.cpp ROCm 记分板：https://github.com/ggml-org/llama.cpp/discussions/15021
- llama.cpp Vulkan 记分板：https://github.com/ggml-org/llama.cpp/discussions/10879
- Phoronix，R9700 上 ROCm 7.1 对 RADV Vulkan：https://www.phoronix.com/review/rocm-71-llama-cpp-vulkan/2 （研究时网页 403，内容来自搜索摘要）

**本机用到的项目**
- rdna-boosts 补丁集：https://github.com/stew675/llama-cpp-rdna-boosts
- Paiton：https://github.com/Eliovp-BV/paiton-vllm-plugin
- Strata（重点读 docs/HOW_IT_WORKS.md）：https://github.com/Niko1221/Strata
- R9V：https://github.com/Dyluhn/R9V
- NInfer：https://github.com/Neroued/ninfer ；kvmem：https://github.com/1314521gjy/ninfer-fusion-kvmem
- Project Maya：https://github.com/mw00/project-maya
- KTransformers：https://github.com/kvcache-ai/ktransformers
