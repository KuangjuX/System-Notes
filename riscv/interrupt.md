# RISC -V Interrupt

## CLINT

**Core Local Interrupter(CLINT)** 提供一个固定优先级的设计，仅对于来自更高特权级的中断提供抢占支持。`CLINT` 最初的设计是用来服务于 CPU 的软件和时钟中断，因为它不控制直接连接到 CPU 的其他本地中断

## CLIC

**Core Local Interrupt Controller(CLIC)** 是一个功能齐全的本地中断控制器，并且支持可编程中断级和优先级配置。`CLIC` 还支持给定特权级别内的嵌套中断（抢占），基于中断级别和优先级配置。

`CLINT` 和 `CLIC` 都集成了寄存器 `mtime` 和 `mtimecmp` 来配置定时器中断，和 `msip` 触发软件中断。除此之外，`CLINT` 和 `CLIC` 都运行在 core 时钟频率上。



## Common Registers to CLIC and CLINT

- msip: M mode software interrupt pending register, 用于断言当前 CPU 的软件中断。

- mtime: 运行在特定频率上的 M mode 时钟寄存器。 `CLINT` 和 `CLIC` 设计中的一部分。在包含一个或多个 CPU 的设计中只有一个 `mtime` 寄存器。

- mtimecmp: 内存映射的 M mode timer compare register，当 `mtimecmp` 的值比 `mtime` 的值更大或者相等时来处罚一个时钟中断，每个 CPU 都有一个 `mtimecmp` 寄存器。

## Reference

- [SiFive Interrupt Cookbook](https://starfivetech.com/uploads/sifive-interrupt-cookbook-v1p2.pdf)
