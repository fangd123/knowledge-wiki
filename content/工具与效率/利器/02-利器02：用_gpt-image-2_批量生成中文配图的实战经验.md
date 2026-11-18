---
title: "利器02：用 gpt-image-2 批量生成中文配图的实战经验"
date: 2026-05-24
author: fangd123
license: CC BY-NC-SA 4.0
category: 工具与效率
series: 利器
series_order: 2
tags:
  - 工具与效率
  - 利器
ai_assisted: true
status: published
---

# 利器02：用 gpt-image-2 批量生成中文配图的实战经验

*"利器"系列说明："工欲善其事必先利其器"，有了称手好用的工具，才能最大程度发挥出自己的能力，提高生产力和效率，避免无效加班。因此，"利器"系列将会记录我对于各种工具的改造，以及我是如何组合应用它们发挥价值。*

---

最近为一个长篇系列文章批量生成 51 张配图，用的是服务商当时提供的 `gpt-image-2` 接口，走的是国内中转服务（New API 协议）。中间踩了几个坑，也总结出一套对**长尾批量任务**比较稳定的脚本范式，在这里完整记录一下，下次再遇到类似的"几十张图打底、要求图中文字必须中文、不能让单点失败拖死全局"的活，可以照搬。

如果你的场景是：

- 给系列文章 / PPT / 课程材料配几十张主题一致的配图
- 跑的是国内中转服务而不是直连 OpenAI
- 需要稳定、可断点续跑、可控并发的批量脚本

先看一次失败的账单，再决定哪些步骤值得抄。

## 一、第一个坑：客户端报错，平台却已经扣费

最直观的踩坑——同一个 API、同一个 key、同一个模型，**用 curl 能成功，用 OpenAI Python SDK 反而稳定失败**。

具体表现是这样：

```
[06-2] attempt 1/3: APIConnectionError: Connection error.
[06-2] attempt 2/3: APIConnectionError: Connection error.
[06-2] attempt 3/3: APIConnectionError: Connection error.
[7/51 | 509s] FAIL 06-2  attempt 3/3: APIConnectionError: Connection error.
```

客户端没有保存到图片；到服务后台核对，同一批请求却已有生成和扣费记录。**没拿到文件，不等于没生成。** 日志能证明这两头的结果不一致，不能证明失败的那次响应完整到达了客户端。

### 怎么定位的

定位思路是经典的"分层确认"：把可能出问题的层一层层剥掉。

**第一层：直接 curl，绕过 SDK。**

```bash
curl -sS -X POST "${BASE_URL}/images/generations" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-image-2","prompt":"...","size":"auto","quality":"low","n":1}'
```

curl 返回 `{"created":..., "data":[{"b64_json":"iVBORw0K..."}]}`，2.2MB 的响应体，base64 解出来是合法 PNG。这一次请求和解码都成功了。

**第二层：单独 Python `requests` 调一次。**

写个 5 行的小脚本，用 `requests.post(...)` 直接打同一个 URL，返回正常。这至少排除了“同一组参数根本不能工作”。

**第三层：换客户端，而不是急着给故障定性。**

同一条链路用 `requests` 能拿到图片，我就先用它完成余下的批次。没抓到失败时的完整响应，也没隔离连接复用、超时设置和中转环节；这是一个绕开问题的办法，不是已经查明了 SDK 的缺陷。

### 解决方案

```python
def call_api(prompt: str, *, read_timeout: float = 600.0) -> bytes:
    url = f"{BASE_URL}/images/generations"
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json",
        "Connection": "close",   # 本次对照关闭连接复用
        "Accept": "application/json",
    }
    body = {
        "model": "gpt-image-2",
        "prompt": prompt,
        "size": "auto",
        "quality": "low",
        "n": 1,
    }
    # (connect_timeout, read_timeout) 分开设置，读超时给足够长
    resp = requests.post(url, headers=headers, json=body, timeout=(30.0, read_timeout))
    resp.raise_for_status()
    payload = resp.json()
    item = payload["data"][0]
    if item.get("b64_json"):
        return base64.b64decode(item["b64_json"])
    if item.get("url"):
        return requests.get(item["url"], timeout=(30.0, read_timeout)).content
    raise RuntimeError("no image in response")
```

几个关键点：

- **`Connection: close`**：这是当时对照使用的配置，并非已证实故障由连接池引起。
- **超时分两段写**：`timeout=(connect, read)`。连接超时 30 秒就够了；读超时给到 **600 秒**，因为 gpt-image-2 在繁忙时单图生成 + 传输 base64 经常耗时 3-5 分钟。
- **优先用 `b64_json`**：返回内联的 base64 而不是 URL。少一次 HTTP 跳转，少一类失败模式。

### 经验法则

> 用一张图对照两种客户端的结果和账单。后台已有产物时，先尝试取回它，不要再花一次钱证明上一次收费。

## 二、第二个坑：超时时间和并发度的取舍

第一版改成 `requests` 之后，串行跑还在偶尔触发 `ReadTimeout`：

```
[4/30 | 415s] start 08-3
  [08-3] attempt 1/3: ReadTimeout: HTTPSConnectionPool:
                      Read timed out. (read timeout=300.0)
```

这次排队，300 秒（5 分钟）不够。**把读超时从 300 秒拉到 600 秒之后，超时基本消失。**

### 串行 vs 并发

最初担心打太多并发会被中转服务限流，所以选了串行。结果串行的速度感人——平均 150 秒/张，30 张要 75 分钟。

核过预算后，我在剩余任务上试着**改成 5 路并发**：

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

with ThreadPoolExecutor(max_workers=5) as pool:
    futures = {pool.submit(work, j): j for j in pending}
    for fut in as_completed(futures):
        job, err = fut.result()
        # 进度打印 ...
```

剩余 20 张图，**串行预计 50 分钟，5 路并发实际跑了 7 分 20 秒，0 失败**。前者是估算，后者是另一批的实测，不能相除就当性能基准。

### 经验法则

> **先用小批量测等待时间，再按配额和可接受的扣费上限增加并发。遇到 429 先读服务方的限流说明。**

## 三、第三个坑：让图中的英文真的变成中文

提示词里大多是英文写的（很多人习惯用英文 prompt 提高图像生成质量），但交付要求图中所有文字必须是中文。直接翻译提示词不靠谱——翻译完风格就走样了。

### 我的做法：保留英文 prompt 主体，追加强制中文化指令

```python
NO_ENGLISH_INSTR = (
    "\n\nCRITICAL: Render ALL text, labels, captions, signs, headings and any "
    "written content visible in the image using SIMPLIFIED CHINESE CHARACTERS only. "
    "Do NOT include any English text or English letters anywhere in the image. "
    "Replace any English label that would normally appear with its Chinese translation."
)

full_prompt = original_english_prompt + NO_ENGLISH_INSTR
```

实测效果不错——大部分图的标签、标题、坐标轴文字都正确渲染成中文。注意几点：

1. **交付约束单独写。** 我把中文要求放在末尾；没有做大小写对照，不能说全大写一定更有效。
2. **明确说"replace"**，给模型可执行的动作，比单纯说"don't include English"更有效。
3. **`gpt-image-2` 对手写中文长文本的字形仍不完美**——长段落级别的中文容易出错字、缺笔画。短标签（2-6 字）质量很高，长句尽量拆成多个短标签。

### `size=auto` 和 `quality=low` 的取舍

- `size=auto`：让模型根据 prompt 描述自动选 1024×1024 / 1024×1536 / 1536×1024。我的提示词里大多写了 `16:9` 横向构图，模型基本都选了 1536×1024 的横幅；它是 3:2，不是严格的 16:9。
- `quality=low`：单张约 1-3MB，1024 等效像素已经够用于公众号/PDF 配图。`high` 一档生成时间长 2-3 倍、文件体积翻 3-4 倍，对配图场景不值。

## 四、批量任务的工程实践

下面是这次脚本里几个值得保留下来的工程实践，跟具体任务无关，下次跑别的批量都能用。

### 1. 幂等：跳过已完成的产物

每次启动脚本，都先检查输出文件是否已存在且大小正常：

```python
def generate(job: Job) -> str | None:
    if job.out_path.exists() and job.out_path.stat().st_size > 1024:
        return None  # 已完成，跳过
    # ... 生成逻辑
```

这让重启后的脚本跳过已落盘的大文件。`> 1024` 只是粗筛，不是完整性校验；正式交付前仍要解码、逐张看字。**本地跳过成功文件，并不能让服务端的生成请求幂等。**

### 2. 重试之前先看账单

有两种看似相同的失败。文件已生成、下载不完整，优先续取同一产物；发出请求后超时，不知道对方有没有接单，就留在清单里核对，不要自动再下单。如果服务有明确的幂等键或可查询的任务号，再考虑有上限的退避重试。**别让单图在原地无限重试卡死整个批次。**

### 3. 进度日志要带"全局编号 + 已耗时"

```python
print(f"[{done_n}/{total} | {elapsed:.0f}s] OK   {job.article}-{job.index} ({size_kb} KB)")
```

输出像这样：

```
[5/20 | 88s] OK   12-1 (2272 KB)
[6/20 | 125s] OK   11-4 (1361 KB)
[7/20 | 143s] OK   12-3 (1537 KB)
```

为什么要带这两个信息：

- `done_n/total` 可以一眼估算还剩多久
- 累计 `elapsed` 看出来是不是后期被限速变慢了

并发跑的时候 `done_n` 不一定按字典序——这是预期行为，不要去强行排序，否则就要等慢的那张完成才能输出快的，反而让进度看起来卡住。

### 4. 失败汇总要在最后再打印

不要在每张失败时就打 banner 让用户慌。最后统一打印：

```
============================================================
Done. 51 succeeded, 0 failed.
```

或者：

```
Done. 48 succeeded, 3 failed.

Failures:
  10-2: attempt 3/3: APIConnectionError: ...
  11-1: attempt 3/3: APIConnectionError: ...
```

## 五、完整的可复用脚本

下面是按这次教训整理的两个示例。请求会产生费用；先用一条假标记试本地发现和插图逻辑。需要生成时，再把服务地址、凭据放在环境变量 `BASE_URL`、`API_KEY`，并填写两份 `FILES`。示例从单并发起步，结果未知时只记失败，不重新下单。

### `_gen_batch.py` — 批量生成

```python
"""Batch-generate images via OpenAI-compatible /v1/images/generations endpoint.

Idempotent: skips files that already exist with size > 1KB.
Concurrent: start with one worker.
"""
from __future__ import annotations

import base64
import os
import re
import sys
import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from dataclasses import dataclass
from pathlib import Path

import requests

BASE_URL = os.environ["BASE_URL"].rstrip("/")  # 可信服务的 HTTPS 地址，含 /v1
API_KEY = os.environ["API_KEY"]
MODEL = "gpt-image-2"

ROOT = Path(__file__).parent
PIC_ROOT = ROOT / "pic"

# (article_id, md_filename) — 改成你要扫描的文件列表
FILES = [
    ("06", "06-xxx.md"),
    ("07", "07-xxx.md"),
    # ...
]

# 标记独占一行，例如 ![配图提示词1：书桌上的灯]；两段脚本使用同一语法。
PROMPT_RE = re.compile(r"^!\[配图提示词(\d+)[：:]((?:[^\]\\\n]|\\.)+)\][ \t]*$", re.MULTILINE)

NO_ENGLISH_INSTR = (
    "\n\nCRITICAL: Render ALL text, labels, captions, signs, headings and any "
    "written content visible in the image using SIMPLIFIED CHINESE CHARACTERS only. "
    "Do NOT include any English text or English letters anywhere in the image. "
    "Replace any English label that would normally appear with its Chinese translation."
)


@dataclass
class Job:
    article: str
    index: str
    prompt: str
    out_path: Path


def discover_jobs() -> list[Job]:
    jobs: list[Job] = []
    for art, fname in FILES:
        text = (ROOT / fname).read_text(encoding="utf-8")
        for m in PROMPT_RE.finditer(text):
            n = m.group(1)
            jobs.append(Job(
                article=art,
                index=n,
                prompt=m.group(2).replace(r"\]", "]").strip(),
                out_path=PIC_ROOT / art / f"{n}.png",
            ))
    return jobs


def call_api(prompt: str, *, read_timeout: float = 600.0) -> bytes:
    url = f"{BASE_URL}/images/generations"
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json",
        "Connection": "close",
        "Accept": "application/json",
    }
    body = {
        "model": MODEL, "prompt": prompt,
        "size": "auto", "quality": "low", "n": 1,
    }
    resp = requests.post(url, headers=headers, json=body, timeout=(30.0, read_timeout))
    if resp.status_code != 200:
        raise RuntimeError(f"HTTP {resp.status_code}")
    payload = resp.json()
    item = payload["data"][0]
    if item.get("b64_json"):
        return base64.b64decode(item["b64_json"])
    if item.get("url"):
        img = requests.get(item["url"], timeout=(30.0, read_timeout))
        img.raise_for_status()
        return img.content
    raise RuntimeError("response contains no image")


def generate(job: Job) -> str | None:
    if job.out_path.exists() and job.out_path.stat().st_size > 1024:
        return None
    job.out_path.parent.mkdir(parents=True, exist_ok=True)
    try:
        data = call_api(job.prompt + NO_ENGLISH_INSTR)
        if len(data) <= 1024:
            raise ValueError("short image response")
        tmp = job.out_path.with_suffix(".part")
        tmp.write_bytes(data)
        tmp.replace(job.out_path)
    except Exception as exc:
        return f"{type(exc).__name__}：先核对服务端状态，不自动重发"
    return None


def main() -> int:
    all_jobs = discover_jobs()
    pending = [j for j in all_jobs
               if not (j.out_path.exists() and j.out_path.stat().st_size > 1024)]
    print(f"Total prompts: {len(all_jobs)}, "
          f"already done: {len(all_jobs) - len(pending)}, "
          f"pending: {len(pending)}", flush=True)
    print(flush=True)

    successes: list[Job] = []
    failures: list[tuple[Job, str]] = []
    start = time.time()

    def work(job: Job) -> tuple[Job, str | None]:
        return job, generate(job)

    with ThreadPoolExecutor(max_workers=1) as pool:
        futures = {pool.submit(work, j): j for j in pending}
        done_n = 0
        total = len(pending)
        for fut in as_completed(futures):
            job, err = fut.result()
            done_n += 1
            elapsed = time.time() - start
            tag = f"[{done_n}/{total} | {elapsed:.0f}s]"
            if err is None:
                size_kb = job.out_path.stat().st_size // 1024
                print(f"{tag} OK   {job.article}-{job.index} ({size_kb} KB)", flush=True)
                successes.append(job)
            else:
                print(f"{tag} FAIL {job.article}-{job.index}: {err}", flush=True)
                failures.append((job, err))

    print()
    print("=" * 60)
    print(f"Done. {len(successes)} succeeded, {len(failures)} failed.")
    if failures:
        print("\nFailures:")
        for j, err in failures:
            print(f"  {j.article}-{j.index}: {err}")
    return 0 if not failures else 1


if __name__ == "__main__":
    sys.exit(main())
```

### `_insert_refs.py` — 把图片引用插回 markdown

生成完图之后，要在每个独占一行的 `![配图提示词1：书桌上的灯]` 后面插入 `![](pic/06/1.png)`。同篇编号必须唯一；这是给自己整理过的稿件用的小工具，不是通用 Markdown 解析器。这个脚本是幂等的，跑多次只插一次。

```python
"""Insert image references after each standalone prompt marker."""
from __future__ import annotations

import re
import sys
from pathlib import Path

ROOT = Path(__file__).parent

FILES = [
    ("06", "06-xxx.md"),
    # ...
]

PROMPT_BLOCK_RE = re.compile(r"^!\[配图提示词(\d+)[：:]((?:[^\]\\\n]|\\.)+)\][ \t]*$", re.MULTILINE)


def process(article: str, fname: str) -> tuple[int, int]:
    fpath = ROOT / fname
    text = fpath.read_text(encoding="utf-8")
    inserted, skipped = 0, 0
    pieces: list[str] = []
    pos = 0

    for m in PROMPT_BLOCK_RE.finditer(text):
        n = m.group(1)
        pieces.append(text[pos: m.end()])
        # 看一下后面第一个非空行有没有已经引用了对应图片
        tail = text[m.end():]
        first_nonempty = next((ln for ln in tail.splitlines() if ln.strip()), "")
        marker = f"pic/{article}/{n}.png"
        if first_nonempty.strip() == f"![]({marker})":
            skipped += 1
        else:
            pieces.append(f"\n\n![]({marker})")
            inserted += 1
        pos = m.end()

    pieces.append(text[pos:])
    if inserted > 0:
        fpath.write_text("".join(pieces), encoding="utf-8")
    return inserted, skipped


def main() -> int:
    total_ins = total_skip = 0
    for art, fname in FILES:
        ins, skip = process(art, fname)
        print(f"  {art}: inserted={ins} skipped={skip}")
        total_ins += ins
        total_skip += skip
    print(f"\nTotal inserted: {total_ins}, skipped: {total_skip}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

## 六、跑这类任务的工作流

最后总结一下针对中转服务做这种长尾批量任务，按下面的顺序走：

1. **先离线数标记**：每篇编号是否唯一？给副本插一次，再跑一次，第二次文件应一字不变。
2. **再花一张图的钱**：保留产物，核对后台计费，把图中文字看一遍。
3. **定并发和读超时**：在预算内小批量增加。
4. **打印失败清单**：一次超时后，先查服务端的生成记录，再决定是否为缺图重新付费。
5. **本地图片齐全后再插图**：核对缺图清单，再向稿件写入引用。

按当时的工作记录，**那次 51 张图的全流程包括 debug 也只用了大约 1.5 小时**，纯生成时间约 50 分钟（最后并发版本只用了 7 分钟）。

## 七、扩展：还可以怎么用这套范式

把 `call_api` 换掉就行。同样的范式可以拿来跑：

- **批量翻译**：`/v1/chat/completions`，prompt 是"翻译成中文"，输入是源文本块
- **批量代码注释**：扫描代码文件、抽取函数、让模型补注释、写回
- **批量 OCR**：把图片传进去让模型识别文字
- **数据集清洗**：把每条原始数据丢进去让模型按 schema 输出 JSON

所有这些场景下，**跳过已有产物、限制并发、区分可重试和结果未知、最后汇总** 这四个特性都是必须的。把这套骨架抄过去，只改业务逻辑那一段就行。

---

*本文是"利器"系列的第二篇，记录的是 `gpt-image-2` 在国内中转服务上的批量任务实战。第一篇见 [Windows Terminal 与 Zsh 改造记录](01-利器01：Windows_Terminal_与_Bash_改造记录.md)。*
