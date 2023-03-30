# HyperBench: A Benchmark Suite for Virtualization Capabilities

## COST MODEL

- $GNR(Guest Native Ratio) = T_{guest}/T_{native}$

- $T_{guest} = T_{direct} + T_{virt}$

- $T_{virt} = T_{cpu} + T_{memory} + T_{io} + \eta$

- $T_{cpu} = C_{sen} * T_{sen} + C_{ext} * T_{ext}$

- $T_{memory} = C_{switch} * T_{switch} + C_{sync} * T_{sync} + C_{cons} * T_{cons} + C_{two} * T_{two}$

- $T_{io} = C_{in} * T_{in} + C_{out} * T_{out}$

## HYPERBENCH BENCHMARKS

### 敏感指令

- 非特权指令：在硬件辅助虚拟化中，VM 执行这些指令和 OS 相同，对于二进制翻译来说会触发翻译流程。

- 特权指令：除了硬件辅助虚拟化技术允许特权级指令直接运行在物理 CPU 上；对于 trap & emulate 策略，特权指令会触发 trap；对于动态二进制翻译，特权指令将会被翻译成一系列 Shadow Structures 相关的正常指令。

### 异常

异常 benchmarks 需要触发特定的虚拟化异常指令，包括：Hypercall、虚拟核间中断（IPI）。

### 内存

内存 benchmarks 主要需要触发从 GVA 到 HPA 的翻译进程和相关的操作，例如 hypervisor-level 的页表的创建，同步 guest 页表和 hypervisor-level 页表。做不同模式的内存访问最直接的方式是触发这些事件。

- Hot Memory Access: 由于程序的时间局部性和空间局部性使得 hot memory access 在真实世界中具有普遍性。虚拟 TLB 应该确保 hot memory access 有一个尽可能高的命中率。
