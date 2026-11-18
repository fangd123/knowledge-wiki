---
title: "从零搭建AI对话机器人：火山引擎与AtomS3R实战"
date: 2026-04-07
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
tags:
  - AI技术与工程
ai_assisted: true
status: published
---

# 从零搭建 AI 对话机器人：火山引擎与 AtomS3R 实战

如果你只是想对着一块小板子说句话，听它答一句，**不必从编译固件开始**。先看 [M5Stack 的语音助手配置说明](https://docs.m5stack.com/zh_CN/guide/realtime/volcengine/atomic_echo_base)。我当时用的现成方案能体验语音来回，但不能按自己的需要更换识别、模型和合成组件。要改这些东西，才值得往下走自建路线。

我用的是 **AtomS3R-M12 和 Atomic Echo Base**：前者是带摄像头的 ESP32-S3 主控，后者接麦克风和扬声器。套件里那块看上去像小平板的东西，其实是 8 Ω、1 W 的扬声器。这次先让它能听、能答；做语音不需要打开摄像头。

## 一、先在浏览器里听到一句回答

硬件、局域网服务和云端只要有一处没接通，最后听到的都是同一种结果：安静。所以我把设备先放在一边。系统各部分怎样连接，可看[火山引擎的嵌入式硬件集成说明](https://www.volcengine.com/docs/6348/1438400)；本文不转载控制台截图。下面记录当时的路径，不保证今天的按钮、可用型号或试用条件相同。

1. 登录自己的[控制台](https://console.volcengine.com/)，在[模型开通管理](https://console.volcengine.com/ark/region:ark+cn-beijing/openManagement?LLM=%7B%7D&OpenModelVisible=false)里选一个本次要用的模型。我当时选 Doubao-Seed-1.6-flash，不是非它不可。先核对地域、价格、怎样停用，再决定是否开通；不要顺手选中所有模型或自动开通后续型号。
2. 从[实时对话式 AI Demo](https://console.volcengine.com/rtc/aigc/run)进入自己的测试应用和 RTC 房间。只为这次测试允许麦克风；看到摄像预览并不能证明声音通了。
3. 在智能体任务中选识别、语言模型和语音合成。旧版界面把语言模型选项称为“方舟大模型”，需要从**自己的**推理接入点详情取 `Endpoint ID`。应用和接入点标识不是给读者照填的参数。识别、模型、合成、实时音视频的授权和计费也不能由其中一项推知。
4. 说一个答案容易辨认的短问题。我会先看识别文本是不是刚才那句话，再看回复是否答到了问题，最后才听声音。文字都不对，就没有必要先去排查板上的扬声器。要接开发板时，再按该版本官方说明启用硬件场景配置。

## 二、让局域网里的服务先跑起来

我用火山引擎的[嵌入式示例工程](https://github.com/volcengine/rtc-aigc-embedded-demo)。它的服务端负责发起、管理云上的实时对话任务，不是用这台电脑本地跑大模型。下面是我当时采用的文件布局；若你检出的版本已经不同，先读那一版的 README，不要把两个版本的配置字段拼起来。

```bash
git clone https://github.com/volcengine/rtc-aigc-embedded-demo.git
cd rtc-aigc-embedded-demo/server/src
python3 -m pip install requests
```

在受控的测试主机上按示例版本配置自己的应用标识和最小权限服务凭据。凭据不写进固件，不进版本库，也不贴调试日志。我的示例把服务端和设备端的 TCP 端口统一为 `8080`：

```bash
python3 RtcAigcService.py
```

先确认进程在预期的测试网接口和端口监听。防火墙仅在专用网络上允许这台设备访问该端口，不对公网转发；若示例的监听范围更大，先限制访问再开机测试。**服务启动不等于板子连得上，板子连得上也不等于云端已经答话。**

## 三、编译、烧录，再看串口

这一步最容易出问题的不是接线，而是把几份不同版本的说明混着看。下面保留当时的开发板型号、ADF 提交和编译目标，方便对照旧工程。ADF 的提交锁定了，不代表今天下载的示例仓或所有依赖也自动锁定；补丁不匹配时，先对照双方版本，不要硬套。

**准备开发环境。** 下面是 macOS 的命令：先装 `cmake`、`ninja` 和 `dfu-util`，再获取乐鑫的音频框架。

```bash
brew install cmake ninja dfu-util
git clone https://github.com/espressif/esp-adf.git
cd esp-adf
git checkout --detach eca11f20e56f9b5321b714da4305e123672d92a9
git submodule update --init --recursive
./install.sh esp32s3
. ./export.sh
```

新开终端需要重新加载环境。在 Windows 命令提示符里，当时用的是 `install.bat`、`export.bat`，环境变量写作 `%ADF_PATH%`，不要把上面的 Homebrew 或 shell 语法照搬过去。

**配置设备端。** 示例放在 ADF 的 `examples` 下面。若已有适配版本的检出，就沿用它；若尚无，下面第一段是在该位置另取一份。先检查示例带的补丁与当前 ADF 是否匹配：

```bash
cd "$ADF_PATH/examples"
git clone https://github.com/volcengine/rtc-aigc-embedded-demo.git
cd "$ADF_PATH"
git apply --check examples/rtc-aigc-embedded-demo/0001-feat-disable-volc-esp-libs.patch
```

检查通过后，再在同一个 ADF 目录应用一次补丁并配置目标：

```bash
git apply examples/rtc-aigc-embedded-demo/0001-feat-disable-volc-esp-libs.patch
cd examples/rtc-aigc-embedded-demo/client/espressif/esp32s3_demo
idf.py set-target esp32s3
idf.py menuconfig
```

- 在 `Example Configuration` 填自己的测试应用和服务主机，端口与服务端一致。`host.invalid:8080` 仅示意格式，不是一个能连接的地址。
- 在 `Audio HAL` → `Audio board` 选 `M5STACK-ATOMS3R`，保存并退出。

**烧录与配网。** `idf.py build` 成功后，用 USB 接上板子。我手上的型号是按复位键约两秒、等绿灯亮再下载；其他型号看对应说明。执行 `idf.py flash`，复位设备，再运行 `idf.py monitor` 看串口。配网热点和入口请按[本型号的官方步骤](https://docs.m5stack.com/zh_CN/guide/realtime/volcengine/atomic_echo_base)和设备当次提示查找。确认连的是自己的设备，再填测试网络信息。开发板最终要和前面的服务主机在可达的测试网络里，不要靠放开整张网来解决连接问题。

## 四、怎样判断哪一层通了？

| 看到或听到什么 | 能说明什么 | 还不能说明什么 |
|---|---|---|
| 浏览器房间里识别文本正确，还能听到对应回答 | 这次浏览器测试的识别、回复、播放走通了 | 开发板的麦克风、扬声器还没参与 |
| 串口先显示设备启动，再显示连上测试网络、加入房间 | 可以把问题缩小到最后出现的那一层之后 | 入房不是对话成功，也不证明音频板选对了 |
| 唤醒板子、问一个短问题，扬声器播完了与问题相符的回复 | 这一次设备上的来回对话走通 | 不是长时间稳定性测试，也不是对费用的承诺 |

测试结束后离开房间、停止任务和临时服务，撤销临时防火墙规则，按自己的账户核对仍在运行和计费的资源。以上是判断现象的办法；这次文章校订没有重新开通服务或重跑硬件。

当时折腾了两天，最费时间的是搞清楚该看哪份文档。有的问题还是翻技术群的旧讨论才找到线索。ESP32 把很多底层工作收走了，却没有替人分清“云端没答”“设备没入房”和“扬声器没出声”。如果再做一次，我还是会先把板子搁在桌上，等浏览器里那句回答确实响了，再去拿 USB 线。
