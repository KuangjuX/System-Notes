# Verilator 学习笔记

## Introduction

Verilator 是一个 verilog/systemverilog 的仿真器，但是它不能直接代替 vivado xsim 这些事件驱动的仿真器。Verilator 是一个基于周期的仿真器，这意味着它不会评估单个周期内的时间，也不会模拟精确的电路时序。相反，通常每个时钟周期评估一次电路状态，因此无法观察到任何时钟周期内毛刺，并且不支持定时信号延迟。

由于 Verilator 是基于周期的，它不能用于时序仿真、反向注释网表、异步（无时钟）逻辑，或者一般来说任何涉及时间概念的信号变化 - 每当评估电路时，所有输出都会立即切换。

然而，由于时钟边沿之间的一切都被忽略了，Verilator 的模拟运行速度非常快，非常适合模拟具有一个或多个时钟的同步数字逻辑电路的功能，或者用于从 Verilog/SystemVerilog 代码创建软件模型以用于软件开发。

由于 Verilator 是基于周期的仿真器，因此对于 systemVerilog 并非完全支持（我们一般也用不到），同时对于 verilog/systemverilog 的检查很严格，同时 Verilator 不支持一些不可综合的代码(例如 `$display()`, `$finish()`, `$fatal()`, 一些版本的 `$assert()` 或者其他)。

使用 Verilator 来仿真时需要 HDL 作为源代码和 C++ 作为测试代码来共同完成，使用 Verilator 来仿真是非常快速的。

在这里我们写了一个简单的 alu 程序来说明 Verilator 的使用:

```systemverilog
/****** alu.sv ******/
typedef enum logic [1:0] {
     add     = 2'h1,
     sub     = 2'h2,
     nop     = 2'h0
} operation_t /*verilator public*/;

module alu #(
        parameter WIDTH = 6
) (
        input clk,
        input rst,

        input  operation_t  op_in,
        input  [WIDTH-1:0]  a_in,
        input  [WIDTH-1:0]  b_in,
        input               in_valid,

        output logic [WIDTH-1:0]  out,
        output logic              out_valid
);

        operation_t  op_in_r;
        logic  [WIDTH-1:0]  a_in_r;
        logic  [WIDTH-1:0]  b_in_r;
        logic               in_valid_r;
        logic  [WIDTH-1:0]  result;

        // Register all inputs
        always_ff @ (posedge clk, posedge rst) begin
                if (rst) begin
                        op_in_r     <= '0;
                        a_in_r      <= '0;
                        b_in_r      <= '0;
                        in_valid_r  <= '0;
                end else begin
                        op_in_r    <= op_in;
                        a_in_r     <= a_in;
                        b_in_r     <= b_in;
                        in_valid_r <= in_valid;
                end
        end

        // Compute the result
        always_comb begin
                result = '0;
                if (in_valid_r) begin
                        case (op_in_r)
                                add: result = a_in_r + b_in_r;
                                sub: result = a_in_r + (~b_in_r+1'b1);
                                default: result = '0;
                        endcase
                end
        end

        // Register outputs
        always_ff @ (posedge clk, posedge rst) begin
                if (rst) begin
                        out       <= '0;
                        out_valid <= '0;
                end else begin
                        out       <= result;
                        out_valid <= in_valid_r;
                end
        end

endmodule;
```

在使用 C++ 编写测试用例之前，我们需要首先将 sv 代码编译成 C++ 代码：

```shell
verilator --cc alu.sv
```

`cc` 参数告诉 Verilator 将源代码转化成 C++ 代码，在编译之后生成了一个 `obj_dir` 文件夹，其中 `mk` 文件用于使用 `Make` 构建仿真的可执行程序，`.h` 和 `.cpp` 文件包含我们源代码实现的信息。

接下来我们使用 C++ 写一个测试文件用于测试我们的 ALU：

```c++
#include <stdlib.h>
#include <iostream>
#include <verilated.h>
#include <verilated_vcd_c.h>
#include "Valu.h"
#include "Valu___024unit.h"

// 终止时间
#define MAX_SIM_TIME 20
vluint64_t sim_time = 0;

int main(int argc, char** argv, char** env) {
    // 新建需要仿真的对象
    Valu *dut = new Valu;

    // 生成仿真波形, "vcd" 文件
    Verilated::traceEverOn(true);
    VerilatedVcdC *m_trace = new VerilatedVcdC;
    dut->trace(m_trace, 5);
    m_trace->open("waveform.vcd");

    while (sim_time < MAX_SIM_TIME) {
        // 翻转时钟
        dut->clk ^= 1;
        // 仿真 ALU 模型中的所有信号
        dut->eval();
        // 将所有被追踪的信号写入波形中
        m_trace->dump(sim_time);
        // 增加时间
        sim_time++;
    }

    m_trace->close();
    delete dut;
    exit(EXIT_SUCCESS);
}
```

关于测试文件的作用已经作为注释写在了代码中，接下来我们将编译我们的测试文件并进行仿真，此时需要运行 Verilator 并重新生成包含测试用例的 `.mk` 文件：

```shell
$ verilator -Wall --trace -cc alu.sv --exe tb_alu.cpp
```

- `-Wall` 表示开启 C++ 所有警告
- `--trace` 表示开启波形跟踪

随后我们进行编译:

```shell
$ make -C obj_dir -f Valu.mk Valu
```

在编译之后，我们执行 `obj_dir/Valu` 进行仿真，此时会生成波形图 `waveform.vcd`，我们只需要执行 `gtkwave waveform.vcd` 即可查看波形图。

## Basics of Systemverilog verification using C++



