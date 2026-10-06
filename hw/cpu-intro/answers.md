
## Q1
- Prediction / 预测:总时间 10，CPU 100%，I/O 0%.
- Reasoning / 理由:两个进程都不做 I/O，CPU 永不空闲.
- Verified result / 验证结果:
- Analysis / 分析:

## Q2
- Prediction / 预测:总时间为 11，CPU 利用率为 54.55%，I/O 利用率为 45.45%.
- Reasoning / 理由:-l 4:100,1:0 意味着 PID 0 需要 4 个 CPU tick（不做 I/O），PID 1 需要 1 个 CPU tick，然后发起 I/O（默认耗时 5 个 tick）。PID 0 先运行 4 个 tick，PID 1 运行 1 个 tick 后进入 BLOCKED 等待 I/O。此时 CPU 空闲，PID 1 等待 5 个 tick。I/O 完成后，PID 1 再运行最后 1 个 CPU tick 完成.
- Verified result / 验证结果:
- Analysis / 分析:

## Q3
- Prediction / 预测:总时间为 7，CPU 利用率为 85.71%，I/O 利用率为 71.43%.
- Reasoning / 理由:-l 1:0,4:100 交换了顺序。PID 0 先运行 1 个 tick，然后发起 I/O（进入 BLOCKED，耗时 5 个 tick）。此时 CPU 空闲，调度 PID 1 运行 4 个 tick。在 PID 1 运行期间，PID 0 的 I/O 也在同步进行。PID 0 的 I/O 完成后，再运行 1 个 CPU tick。总时间被重叠的 CPU 和 I/O 大大缩短.
- Verified result / 验证结果:
- Analysis / 分析:

## Q4
- Prediction / 预测:总时间为 11，CPU 利用率为 54.55%，I/O 利用率为 45.45%.
- Reasoning / 理由:命令加了 -S SWITCH_ON_END。这意味着当进程发起 I/O 时，操作系统不会切换到另一个进程，直到当前进程完全结束。因此，PID 0 发起 I/O 后，OS 拒绝调度 PID 1，导致 CPU 在整个 I/O 期间（5个tick）完全空转，白白浪费.
- Verified result / 验证结果:
- Analysis / 分析: 
## Q5
- Prediction / 预测:总时间为 7，CPU 利用率为 85.71%，I/O 利用率为 71.43%.
- Reasoning / 理由: 命令加了 -S SWITCH_ON_IO。这是模拟器的标准/默认行为。当进程发起 I/O 时，OS 立即切换到另一个就绪进程（PID 1）。PID 1 运行 4 个 tick，此时 PID 0 的 I/O 正在并行进行。总时间得到最大化压缩.
- Verified result / 验证结果:
- Analysis / 分析:

## Q6
- Prediction / 预测:总时间为 7，CPU 利用率为 85.71%，I/O 利用率为 71.43%.
- Reasoning / 理由:命令加了 -I IO_RUN_LATER。这意味着当 I/O 完成时，完成的进程（PID 0）不会立即抢占当前正在运行的进程（PID 1），而是进入 READY 队列等待。由于 PID 1 只需要 4 个 tick，它在 PID 0 的 I/O（5个tick）完成之前就已经结束了，所以 CPU 在 Tick 6 是空闲的，PID 0 完成后直接运行.
- Verified result / 验证结果:
- Analysis / 分析:
## Q7
- Prediction / 预测:总时间为 7，CPU 利用率为 85.71%，I/O 利用率为 71.43%.
- Reasoning / 理由:命令加了 -I IO_RUN_IMMEDIATE（你之前跑出 Total Time 21 就是这个参数，但那是因为进程数不同）。当 PID 0 的 I/O 完成时，它立即抢占 CPU 运行。在我们的场景下，PID 1 已经完成了，CPU 空闲，所以 PID 0 立即运行.
- Verified result / 验证结果:
- Analysis / 分析:

## Q8
- Prediction / 预测:总时间为 15，cup 利用率为 53.33%，I/O 利用率为 66.67%.
- Reasoning / 理由:由于涉及随机概率，进程可能会在某个 tick 突然发起 I/O。因为有 3 个进程，当某些进程阻塞时，CPU 可能会空闲，或者调度其他就绪进程.
- Verified result / 验证结果:
- Analysis / 分析:

