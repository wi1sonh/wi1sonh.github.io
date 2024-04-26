---
layout: post
title: RTOS 学习 (1)
---

## Motivation

为了了解实时操作系统 (RTOS) 的功能，需要了解通用机制和 Linux 特定机制。

---

## 基本技术细节

当前正在运行的低优先级任务必须被抢占以允许实时关键的任务运行。抢占依赖任务的调度策略。实时系统另一个重要的方面是确保实时任务对某一些资源的独占。

### 实时系统

实时系统是运行 RTOS 和至少一个实时任务的平台。RT 任务是必须在特定截止日期之前完成的任务。这些任务可以是定期的，也可以是非周期性的，要确保根据执行频率和截止日期的严格程度来安排它们。系统上也可能运行非实时任务。

### 特定于 Linux 的抢占模型

> 主线 Linux 内核为不同的应用领域 (如服务器或台式 PC) 实现了三种不同的抢占模型。有了 PREEMPT_RT 补丁，还有两个额外的抢占模型可用。『Fully Preemptible Kernel』模型是将 Linux 变成 RTOS 的模型。

各种抢占模型是特定于内核的。原则上，用户空间进程始终是抢占式的。
正在运行的任务的抢占由调度进程执行。此操作可以由**内核交互** (如系统调用) 或**异步事件** (如中断) 触发。调度进程保存抢占任务的上下文，并还原新任务的上下文。
Linux 内核实现了多个抢占模型。在构建内核时选择所需的模型。必须选择『Fully Preemptible Kernel』抢占模型才能将 Linux 作为 RTOS。为了完整起见，现有 Linux 抢占模型的列表和简短说明如下。最后两个条目仅 PREEMPT_RT 补丁打上后才能够启用。

- **No Forced Preemption (server)** : 传统的 Linux 抢占模型，面向吞吐量。系统调用返回和中断是唯一的抢占点。
- **Voluntary Kernel Preemption (Desktop)**: 通过向内核代码添加更多『显式抢占点』来减少内核的延迟，但代价是吞吐量略低。除了显式抢占点之外，系统调用返回和中断返回也是隐式抢占点。
- **Preemptible Kernel (Low-Latency Desktop)** : 通过使所有内核代码 (未在关键部分执行) 抢占式来减少内核的延迟。隐式抢占点位于每个抢占禁用部分之后。
- **Preemptible Kernel (Basic RT)** : 类似于『抢占式内核 (低延迟桌面)』模型。除了上面提到的属性之外，线程中断处理进程也是强制的 (如使用内核命令行参数 threadirqs 时)。该模型主要用于测试和调试 PREEMPT_RT 补丁实现的替换机制。
- **Fully Preemptible Kernel (RT)** : 除了少数选定的关键部分外，所有内核代码都是抢占式的。线程中断处理进程是强制的。此外，还实施了几种替换机制，如睡眠旋转锁和 rt_mutex，以减少抢占禁用部分。此外，大型抢占禁用部分被单独的锁定结构所取代。必须选择此抢占模型才能获得实时行为。

### 优先级倒置 & 优先级继承

> 当具有高优先级的任务由于资源互斥而被优先级较低的任务阻止时，优先级介于其他任务之间的第三个任务可以在优先级最高的任务恢复之前运行并完成。这种现象称为优先级倒置。可以通过优先继承来解决。

优先级继承是解决优先级倒置问题的一种方法。

#### 优先级倒置

![1](\assets\images\2024-03-28-RTOS学习(1)_priority-inversion.png)

- 优先级为 L 的低优先级任务变为可运行并开始执行 (1)
- 它获得相互排斥的资源 (2)
- 现在，优先级较高的任务 H 变得可运行，并在 L 持有资源 (3) 时抢占 L
- 优先级介于 Hs 和 Ls 优先级之间的第三个任务 M (并且不需要资源) 变得可运行，但它必须等待，因为优先级更高的 H 正在运行 (4)
- H 需要 L (5) 仍然持有的资源，因此 H 停止运行以等待资源被释放
- 可运行的任务 M 阻止 L 执行，因为它的优先级更高。这会导致优先级倒置，因为 H 必须等到 M 完成 (6) 才能释放资源 (7)

#### 优先级继承

优先级反转问题通过优先级继承解决：

![2](\assets\images\2024-03-28-RTOS学习(1)_priority-inversion.png)

这意味着当 H 需要 L 持有的资源时，L 继承了 H 的优先级 (5)，以便更快地释放资源 (6)。当 M 准备好运行时，M 必须等到当前优先级较高的任务 L 释放资源并且 H 完成运行 (7)。

### 调度策略和优先级

> Linux 内核实现了多个实时和非实时调度策略。根据任务的计划策略，调度进程决定交换哪个任务以及接下来处理哪个任务。

根据所选的任务策略和关联的规则，调度进程决定交换哪个任务以及接下来处理哪个任务。Linux 内核实现了多个调度策略。它们分为非实时策略和实时策略。调度策略已在主线 Linux 中实现。

非实时策略：

- **SCHED_OTHER** : 每个任务都会获得所谓的『nice value』。它是介于 -20 之间的值 (表示最高 nice 值) 和 19 (表示最低 nice 值)。任务执行时间的平均值取决于关联的 nice 值。
- **SCHED_BATCH** : 派生自 SCHED_OTHER，并针对吞吐量进行了优化。
- **SCHED_IDLE** : 也是从 SCHED_OTHER 派生的，但它的值比 19 弱。

实时策略：

- **SCHED_FIFO** : 任务的优先级介于 1 (低) 和 99 (高) 之间。在此策略下运行的任务将计划完成，或者优先级更高的任务抢占它。
- **SCHED_RR** : 源自 SCHED_FIFO。与 SCHED_FIFO 的区别在于，任务在定义的时间片的持续时间内运行 (如果它没有被优先级更高的任务抢占)。一旦时间片用完，它可能会被具有相同优先级的任务中断。时间片定义在 procfs (/proc/sys/kernel/sched_rr_timeslice_ms) 中导出。
- **SCHED_DEADLINE** : 实现全局最早的截止日期优先 (Global Earliest Deadline First, GEDF) 算法。在此策略下计划的任务可以抢占使用 SCHED_FIFO 或 SCHED_RR 计划的任何任务。

### 调度 - RT 限制机制

> RT 限制机制可防止系统在实时应用进程中出现运行时程序失败时挂起。这种机制使得停止此类应用进程成为可能。RT 限制的设置将导出到 proc 文档系统中。

实时应用进程中的运行失败可能导致整个系统挂起。这样的失败可能类似于对『while (true) {}』循环的调用。当实时应用进程具有尽可能高的优先级并使用 SCHED_FIFO 策略进行计划时，任何其他任务都无法抢占它。这会导致系统阻止所有其他任务，并以 100% 的 CPU 负载调度此循环。实时限制是一种通过限制每个周期的实时任务执行时间来避免这种情况的机制。这些设置将导出到 proc 文档系统中。默认设置为：

```sh
# cat /proc/sys/kernel/sched_rt_period_us
1000000
# cat /proc/sys/kernel/sched_rt_runtime_us
950000
```

要达到实时任务和更大时间段的 CPU 使用率仅为 50%，可以使用以下命令更改这些值：

```sh
# echo 2000000 > /proc/sys/kernel/sched_rt_period_us
# echo 1000000 > /proc/sys/kernel/sched_rt_runtime_us
```

如果实时任务运行时的长度与时间段相同，则禁用实时限制。这是通过将『-1』写入『sched_rt_runtime_us』自动完成的：

```sh
# echo -1 > /proc/sys/kernel/sched_rt_runtime_us
```

这种机制已经在主线 Linux 中实现。

### 延迟

> 低延迟是实时计算环境中的一项关键要求，因为它可以确保任务或流程能够快速、可预测地响应外部事件或输入。

当实时系统被说成有延迟或有需要调试的延迟时，术语延迟是指『任务或操作需要很长时间才能完成的情况』。在实时上下文中，系统执行特定任务所需的时间 (即任务的延迟) 很重要，因为每个任务都必须能够在指定的截止日期之前完成。

如果一个 RT 任务没有在规定的时间限制内完成，那就和任务根本没完成一样糟糕。这就是为什幺开发人员花费大量时间确保即使在最坏的情况下，任务仍然可以在截止日期之前完成。这意味着，在评估实时系统的性能时，要测量的最重要的延迟值是最大延迟。

### Dynticks

Dynticks 或动态 ticks 或无 HZ 模式可减少定时器中断开销。

在很长一段时间里，Linux 内核都有一个周期性的中断，这使得内核调度进程来平衡和调度在 CPU 之间运行的线程。此中断称为定时器 tick，它以固定速率在每个 CPU 或内核上生成 100-1000 HZ。无论 CPU 的电源状态如何，内核都会为此中断提供服务。此模式由选项 CONFIG_HZ_PERIODIC 启用。

CONFIG_NO_HZ_IDLE 模式允许 CPU 在空闲时不受干扰，并且可以节省电量，因为它只是在需要服务定时器时唤醒 CPU.

全 dynticks 系统 (无滴答) 模式由 CONFIG_NO_HZ_FULL 启用，并由引导参数激活 nohz_full。内核会尽可能自适应地尝试关闭 tick，即使 CPU 正在运行具有函数 tick_nohz_full_cpu 的任务。

---

## Reference

- https://wiki.linuxfoundation.org/realtime/documentation/start
- https://lwn.net/Articles/743740/
- https://lwn.net/Articles/743946/
- https://docs.kernel.org/scheduler/sched-deadline.html
- https://docs.kernel.org/scheduler/sched-rt-group.html
- https://docs.kernel.org/scheduler/sched-bwc.html
- https://wiki.linuxfoundation.org/realtime/documentation/howto/debugging/start
- https://wiki.linuxfoundation.org/realtime/documentation/howto/tools/ticklesskernel
