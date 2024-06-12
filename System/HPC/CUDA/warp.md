# Warp

## APIs

**__shfl_xor_sync**

表示被 mask 制定线程的 laneid 与 laneMask 做按位异或计算，实现不同线程之间的 butterfly 数据交互。

`__shfl_xor_sync(0xffffffff,x,1)` 表示 laneid=0 的线程与 laneid=1 的线程中的变量 x 进行交互，laneid=2 的线程与 laneid=3 的线程中变量 x 的值进行交互，后面其他线程以此类推。
