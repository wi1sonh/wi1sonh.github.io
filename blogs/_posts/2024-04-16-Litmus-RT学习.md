---
layout: post
title: Litmus-RT 学习
---

## What? Why? How?

LITMUSRT 是 Linux 内核的实时扩展，专注于多处理器实时调度和同步。修改了 Linux 内核，以支持零星任务模型、模块化调度器插件和基于预留的调度。包括集群、分区和全局调度进程，并且还支持半分区调度。

自 2006 年以来，LITMUSRT 一直由 Björn Brandenburg 持续维护，并积极开发到 2017 年。截至 2018 年，目前没有计划将其重新定位到更新的 Linux 内核版本。

Linux 内核补丁 + 用户空间接口 + 追踪工具

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-15.png)

### 目标

在实际条件下实现实用的多处理器实时系统研究

为什么实施『教科书式』调度进程很困难？
除了通常的内核乐趣：受限环境、特殊 API、难以调试......

如果你正在做内核级别的工作，不管怎样...

- 抢占先机 — 简化的内核接口、调试基础架构、用户空间接口、跟踪基础架构
- 作为基准 - 与 LITMUSRT 中的其他调度器进行比较

如果您正在开发实时应用...

- 通过与文献相匹配的『教科书算法』获得可预测的执行环境
- 通过 reservation-based scheduling 隔离进程
- 只需几个命令即可了解内核开销

如果你的主要关注点是理论和分析...

- 要了解开销的影响
- 展示所提出方法的实用性

#### 理论上的调度

调度器：一个函数，在每个时间点将一组就绪作业中的元素映射到一组 M 个处理器上

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-12.png)

- 基于全局状态的全局策略：例如在任何时间点，m 最高优先级
- 顺序策略，假定事件的总顺序：例如如果作业准时到达...

#### 实际上的调度

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-13.png)

作业分配仅在响应明确定义的计划事件 (或在众所周知的时间点) 时更改
![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-14.png)

每个处理器仅在本地调度自己

- 多处理器调度进程是并行算法
- 并发的，不可预测的调度事件
- 新的事件在做决定时发生
- 没有可获取的全局一致的原子快照

LITMUSRT 项目的主要目的是为应用实时系统研究提供一个有用的实验平台。为此，LITMUSRT 在内核中提供了抽象和接口，简化了多处理器实时调度和同步算法的原型设计 (与修改『普通』Linux 内核相比)。

作为次要目标，LITMUSRT 作为概念验证，展示了如何在当前硬件上实现可预测的多处理器调度进程和锁定协议。最后，我们希望 LITMUSRT 的部分内容和“经验教训”可以作为蓝图或灵感来源，为其他实施工作（包括商业和开源）找到价值。

让实时系统研究人员的工作更轻松

- LITMUSRT 一直以来都是，现在仍然主要是一种研究工具
- 通过使其更平易近人来鼓励系统研究

功能足够完整和稳定，实用：评估无法**运行实际工作负载**的系统毫无意义

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-11.png)

### 非目标

LITMUSRT 是一个稳定且经过测试的研究系统，它运行良好，但我们没有资源来维持严格的 QA 制度，因为它是一个适合安全或关键任务应用的生产质量系统 (就像大多数研究项目一样)。

此外，LITMUSRT 的 API 并不『稳定』，也就是说，接口和实现可能会在发布之间毫无征兆地发生变化。POSIX 合规性不是目标；LITMUSRT-API 提供备用系统调用接口。

虽然我们的目标是遵循 Linux 编码标准，但 LITMUSRT 的目标不是合并到主流 Linux 中。相反，我们希望 LITMUSRT 中的一些原型思想最终可以在 Linux 或其他内核中得到采用。

- POSIX 服从：我们提供自己的 API — POSIX 陈旧且具有局限性
- API 稳定性：我们很少破坏接口，但如果需要，我们会毫不犹豫地这样做
- 上游包容：LITMUSRT 既不打算也不适合合并到 Linux 中

### 当前版本

LITMUSRT 的当前版本是 2017.1，基于 Linux 4.9.30。它于 2017 年 5 月 26 日发布，包括用于以下调度策略的插件：

- Partitioned EDF with synchronization support (PSN-EDF)
- Global EDF with synchronization support (GSN-EDF)
- Clustered EDF (C-EDF)
- **Partitioned Fixed-Priority (P-FP)**
- Partitioned Reservation-Based Scheduling (P-RES)
- PD2, with either staggered or aligned quanta (PFAIR)

## Major Features

应用可调度性分析 + 在考虑开销下 + 快速启动开发

### 实时多处理器调度方法：Partitioned vs. Clustered vs. Global

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-2.png)

### 可预测的实时调度器：匹配文献研究

在主线 LITMUSRT 中维护：
Global EDF、Clustered EDF、Partitioned EDF、Partitioned Fixed-Priority (FP)、Partitioned Reservation-Based (polling + table-driven)、Pfair (PD2)

外部分支和补丁/文献研究专用原型：...

### 快速启动您的研究

主旨：您需要的调度器可能已经可用
(几乎) 永远不要从头开始：如果您需要实现一个新的调度进程，可能存在一个很好的起点 (例如，具有相似的结构)
大量基线：至少，LITMUSRT 可以为您提供有趣的基线进行比较

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-3.png)

### 轻量级开销跟踪 feather trace

最小的静态跟踪点 + 二进制重写 (JMP ↔ NOP) + 每个处理器，免等待缓冲区
使用实际开销评估您的工作负载！

Built on top of feather trace:
任务参数 + 上下文切换和阻塞 + 工作发布、截止日期和完成

#### 自动中断过滤

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-4.png)

如何解决？
不能只是关闭中断 -> 使用统计过滤器 -> 但是哪个过滤器呢？-> 如果存在真正的异常值怎幺办？
自 LITMUSRT 2012.2 以来：

- ISR 增量计数器
- 时间戳包括计数器快照和标志
- 中断的采样 (自动丢弃)

#### 周期计数器偏斜补偿

跟踪处理器间中断 (IPI)：
![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-5.png)

在 LITMUSRT 中，只需运行 ftcat -c 即可自动测量补偿未对齐的时钟源

#### 调度可视化：st-draw

有没有想过 Pfair 时间表在现实中是什么样子的？
简单！只需用 sched_trace 记录时间表并运行 st-draw!

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-6.png)
注意：这是来自 4 核机器的真实执行数据，不是模拟！[颜色表示 CPU 身份]

#### 轻松访问工作负载统计信息

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-7.png)

#### 同步任务系统释放

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-8.png)

`int wait_for_ts_release (void);`: 任务在同步释放之前处于休眠状态
`int release_ts (lt_t *delay);`: 在 <延迟> 纳秒后触发同步释放

#### 具有相位/偏移的异步释放

LITMUSRT 还支持非零相位/偏移：在任务系统发射后，会发射一些已知的偏移量

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-9.png)

可以对异步周期性任务使用可调度性测试

#### 为新调度器提供更轻松的起点

![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-10.png)
简化的接口、更丰富的任务模型、大量可运行的代码可供模仿

### 更多特征

支持真正的全局调度

- 支持正确的拉取迁移：在 Linux 的每处理器运行队列之间移动任务
- Linux 的 SCHED_FIFO 和 SCHED_DEADLINE 全局化：调度『仿真』不是 100% 正确的 (可能存在竞争)

低开销非抢占式部分：非抢占式旋转锁，无需系统调用。

免等待抢占状态跟踪

- 『这个远程核心是否需要发送 IPI？』
- 简单 API 可抑制多余的 IPI

使用 TRACE () 调试跟踪：广泛支持 Qemu 的『printf () 调试』

## Key Concepts

### 调度器插件

![sp](/assets/images/2024-04-16-Litmus-RT学习_image.png)

SCHED_LITMUS『class』激活五个调度器插件（或其他自定义的插件），LITMUSRT 任务具有最高优先级。
可以在运行时命令行界面通过指令 `setsched xxx` 切换调度器插件，但仅当没有实时任务被运行才可以
插件直接管理实时任务
![alt text](/assets/images/2024-04-16-Litmus-RT学习_image-1.png)

### 三个主要的仓库

Linux 内核补丁 ➔ litmus-rt

用户空间接口 ➔ liblitmus

代码追踪工具 ➔ feather-trace-tools

#### liblitmus: The User-Space Interface

C API (task model + system calls) + user-space tools

➔ `setsched`, `showsched`, `release_ts`, `rt_launch`, `rtspin`

##### /proc/litmus/\* and /dev/litmus/\*

`/proc/litmus/*`

- 用于导出有关插件和现有实时任务的信息
- 可读和可写文件
- 通常由更高级别的包装脚本管理

`/dev/litmus/*`

- 基于自定义字符设备驱动进程的特殊设备文件
- 主要导出跟踪数据 (仅用于 ftcat) :
  - ft_cpu_traceX: CPU X 的内核本地开销
  - ft_msg_traceX: 与 CPU X 相关的 IPI
  - sched_traceX: 发生在 CPU X 上的调度事件
- log: 调试跟踪 (与常规 cat 一起使用)

##### Control Page: /dev/litmus/ctrl

每个实时任务映射每个 (私有) 进程页面

- 内核和任务之间的共享内存段
- 用途：低开销通信信道
- 中断计数
- preemption-disabled 和 preemption-needed 标志
- 当前截止日期等

第二个目标，截至 2016.1

- 将 LITMUSRT『系统调用』实现为 ioctl () 操作
- 提高便携性并减少维护开销

透明使用: liblitmus 负责一切

#### (缺乏) 处理器相关性

在 Linux 中，每个进程都有一个处理器关联掩码
Xth bit set ➜ process may execute on core X

大多数 LITMUSRT 插件都忽略了亲和掩码。
特别是，主线版本中的所有插件都这样做。
Global is global; partitioned is partitioned

Recent out-of-tree developments
Support for hierarchical affinities [ECRTS’16]

#### 不被支持的技术

由于资源有限，我们不可能支持和测试所有 Linux 功能

x86 和 ARM 以外的体系结构

- 如果有人关心，增加支持并不难

在虚拟机管理进程上运行

- 它有效 (➞ hands-on session)，但不受『官方』支持
- 您可以使用 LITMUSRT 作为实时虚拟机管理进程，将 KVM 封装在 a reservation

CPU 热插拔

- 现有插件不支持

处理器频率缩放

- 插件『工作』，但对速度变化视而不见

与 PREEMPT_RT 集成

- 由于历史原因，这两个补丁不兼容
- 在PREEMPT_RT之上重新定位已经有一段时间了

## 附录

### 术语解释收集

C-EDF: Clustered EDF

DMPO: Deadline Monotonic Priority Ordering

FPPS: Fixed-Priority Preemptive Scheduling

GSN-EDF: Global EDF with synchronization support

MSRP: Multiprocessor Stack Resource Policy
MrsP: Multiprocessor resource sharing Protocol

OPA: Audsleys Optimal Priority Assignment
RPA: Robust Priority Assignment

RBS: Reservation-Based Scheduling
P-RES: Partitioned Reservation-Based Scheduling

PSN-EDF: Partitioned EDF with synchronization support
P-FP: Partitioned Fixed-Priority
PD2 (PFAIR) : with either staggered or aligned quanta

SPO: Slack-based Priority Ordering

### Reference

[litmus-overview.pdf](https://www.litmus-rt.org/tutorial/litmus-overview.pdf)
