
## Q1
- Prediction / 预测:总时间 10，CPU 100%，I/O 0%.
- Reasoning / 理由:两个进程都不做 I/O，CPU 永不空闲.
- Verified result / 验证结果:Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
- Analysis / 分析:

## Q2
- Prediction / 预测:总时间为 11，CPU 利用率为 54.55%，I/O 利用率为 45.45%.
- Reasoning / 理由:-l 4:100,1:0 意味着 PID 0 需要 4 个 CPU tick（不做 I/O），PID 1 需要 1 个 CPU tick，然后发起 I/O（默认耗时 5 个 tick）。PID 0 先运行 4 个 tick，PID 1 运行 1 个 tick 后进入 BLOCKED 等待 I/O。此时 CPU 空闲，PID 1 等待 5 个 tick。I/O 完成后，PID 1 再运行最后 1 个 CPU tick 完成.
- Verified result / 验证结果:Stats: Total Time 11
Stats: CPU Busy 6 (54.55%)
Stats: IO Busy  5 (45.45%)
- Analysis / 分析:预测与验证结果一致。总时间 = 4 (PID 0) + 1 (PID 1 CPU) + 5 (I/O 等待) + 1 (PID 1 CPU) = 11。CPU 忙 = 4+1+1=6，I/O 忙 = 5。

## Q3
- Prediction / 预测:总时间为 7，CPU 利用率为 85.71%，I/O 利用率为 71.43%.
- Reasoning / 理由:-l 1:0,4:100 交换了顺序。PID 0 先运行 1 个 tick，然后发起 I/O（进入 BLOCKED，耗时 5 个 tick）。此时 CPU 空闲，调度 PID 1 运行 4 个 tick。在 PID 1 运行期间，PID 0 的 I/O 也在同步进行。PID 0 的 I/O 完成后，再运行 1 个 CPU tick。总时间被重叠的 CPU 和 I/O 大大缩短.
- Verified result / 验证结果:Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
- Analysis / 分析:预测与验证结果一致。这正是引入多道程序设计（并发）的优势：当一个进程等待 I/O 时，CPU 可以执行另一个进程，从而隐藏了 I/O 延迟，大幅提升了 CPU 利用率（从 54.55% 提升到 85.71%）。

## Q4
- Prediction / 预测:总时间为 11，CPU 利用率为 54.55%，I/O 利用率为 45.45%.
- Reasoning / 理由:命令加了 -S SWITCH_ON_END。这意味着当进程发起 I/O 时，操作系统不会切换到另一个进程，直到当前进程完全结束。因此，PID 0 发起 I/O 后，OS 拒绝调度 PID 1，导致 CPU 在整个 I/O 期间（5个tick）完全空转，白白浪费.
- Verified result / 验证结果:Stats: Total Time 11
Stats: CPU Busy 6 (54.55%)
Stats: IO Busy  5 (45.45%)
- Analysis / 分析: 预测与验证结果一致。SWITCH_ON_END 策略严重降低了 CPU 利用率。由于 CPU 必须傻等 I/O 结束，总时间回到了 11，和 Q2 一样糟糕。
## Q5
- Prediction / 预测:总时间为 7，CPU 利用率为 85.71%，I/O 利用率为 71.43%.
- Reasoning / 理由: 命令加了 -S SWITCH_ON_IO。这是模拟器的标准/默认行为。当进程发起 I/O 时，OS 立即切换到另一个就绪进程（PID 1）。PID 1 运行 4 个 tick，此时 PID 0 的 I/O 正在并行进行。总时间得到最大化压缩.
- Verified result / 验证结果:Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
- Analysis / 分析:预测与验证结果一致。与 Q4 对比，SWITCH_ON_IO 是提升系统并发度和资源利用率的关键。它允许 CPU 和 I/O 设备并行工作。

## Q6
- Prediction / 预测:总时间为 7，CPU 利用率为 85.71%，I/O 利用率为 71.43%.
- Reasoning / 理由:命令加了 -I IO_RUN_LATER。这意味着当 I/O 完成时，完成的进程（PID 0）不会立即抢占当前正在运行的进程（PID 1），而是进入 READY 队列等待。由于 PID 1 只需要 4 个 tick，它在 PID 0 的 I/O（5个tick）完成之前就已经结束了，所以 CPU 在 Tick 6 是空闲的，PID 0 完成后直接运行.
- Verified result / 验证结果:Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
- Analysis / 分析:预测与验证结果一致。在这个特定场景下（PID 1 运行时间 < I/O 耗时），IO_RUN_LATER 和 IO_RUN_IMMEDIATE 没有区别，因为当 I/O 完成时，CPU 本来就已经空闲了，PID 0 是唯一就绪的进程。
## Q7
- Prediction / 预测:总时间为 7，CPU 利用率为 85.71%，I/O 利用率为 71.43%.
- Reasoning / 理由:命令加了 -I IO_RUN_IMMEDIATE（你之前跑出 Total Time 21 就是这个参数，但那是因为进程数不同）。当 PID 0 的 I/O 完成时，它立即抢占 CPU 运行。在我们的场景下，PID 1 已经完成了，CPU 空闲，所以 PID 0 立即运行.
- Verified result / 验证结果:Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
- Analysis / 分析:预测与验证结果一致。如果 PID 1 需要更多 CPU 时间（比如 10 个 tick），IO_RUN_IMMEDIATE 就会在 PID 0 的 I/O 完成时打断 PID 1，让 PID 0 先跑，从而导致 PID 1 的执行被推迟。但在本题参数下，两者结果相同。

## Q8
- Prediction / 预测:总时间为 15，cup 利用率为 53.33%，I/O 利用率为 66.67%.
- Reasoning / 理由:由于涉及随机概率，进程可能会在某个 tick 突然发起 I/O。因为有 3 个进程，当某些进程阻塞时，CPU 可能会空闲，或者调度其他就绪进程.
- Verified result / 验证结果:Stats: Total Time 15
Stats: CPU Busy 8 (53.33%)
Stats: IO Busy  10 (66.67%)
- Analysis / 分析:因为进程在某个时刻都处于阻塞状态，导致 CPU 出现了空转，利用率只有53.33%

