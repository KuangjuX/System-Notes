# Arceos irq 分析

## riscv 中断分析

### 中断处理流程

- `modules/axhal/src/arch/riscv/trap.S`:  `trap_vector_base` 随后调用 `riscv_trap_handler`。

- `modules/axhal/src/arch/riscv/trap.rs`: `riscv_trap_handler` 获得 scause 寄存器，若为中断，调用 `crate::trap::handle_irq_extern(scause)` scause 作为参数。

- `modules/axhal/src/trap.rs`: `handle_irq_extern(irq_num: usize)` 调用 `call_interface!(TrapHandler::handle_irq, irq_num)` 这个 `irq_num` 实际上是 scause 的值。`call_interface!` 是一个宏，可以调用外部 crate 的接口。

- `modules/axruntime/src/trap.rs`: `impl axhal::trap::TrapHandler for TraphandlerImpl`: 这里实现了如何对于外部中断的处理，调用了 `axhal::irq::dispatch_irq(_irq_num)` 注意这里的 `_irq_num` 依然是 scause 的值。

- `modules/axhal/src/platform/qemu_virt_riscv`: `dispatch_irq(scause: usize)` 根据 scause 对于外部中断还是时钟中断进行处理。若为外部中断，调用 `crate::irq::dispatch_irq_common(irq_num)` 进行处理，这里注意，这个 `irq_num` 应该真的是中断请求号而不是 scause 的值。

- `modules/axhal/src/irq.rs`: `dispatch_irq_common(irq_num: usize)` 从中断处理表（`IRQ_HANDLER_TABLE`）中拿到对应的中断处理函数进行处理。

 
