# Infinitensor 源码阅读

## Frontend

首先查看 `pyinfinitensor/tests/test_onnx.py` 发现 `test_load()` 函数搜索 `.onnx` 文件并使用 `OnnxStub` 进行转换。`OnnxStub` 是一个 class，接受两个参数：

- Onnx Model

- Backend Runtime

以 CPU 为例，这里的 `cpu_runtime()` 根据 `ffi_infinitensor.cc` 中的定义为 `NativeCpuRuntimeObj::getInstance()`。`NativeCpuRuntimeObj` 是一个 runtime Object，有 `alloc`，`dealloc`, `run` 等方法。最终 onnxstub 返回的对象可用于操作项目中的模型和运行时。

### OnnxStub

1. 类属性和初始化函数：
   
   - `inputs` 和 `outputs`：字典，用于存储模型的输入和输出张量。
   - `initializer`：字典，用于存储模型的初始化器（例如权重和偏置）。
   - `handler`：后端图处理器对象，用于执行操作。
   - `__init__(self, model: ModelProto, runtime)`：构造函数，接受两个参数，一个是 ONNX 模型对象 `model`，另一个是运行时对象 `runtime`。在初始化过程中，将执行 ONNX 模型形状推断并创建图处理器。

2. 初始化过程：
   
   - 在构造函数中，首先对输入、输出和初始化器字典进行初始化，以及创建后端图处理器。
   - 对输入进行迭代，为每个输入创建后端张量对象，并将其添加到 `tensors` 字典中。
   - 对输出进行迭代，为每个输出创建后端张量对象，并将其添加到 `tensors` 字典中。
   - 对初始化器进行迭代，为每个初始化器创建后端张量对象，并将其添加到 `tensors` 字典中。同时，将初始化器数据添加到 `data` 字典中。

3. 对节点的处理：
   
   - 对模型中的每个节点进行处理。
   - 针对不同的操作类型（例如卷积、全连接等），根据节点的属性和输入创建后端张量，并执行相应的操作。操作的结果将保存在 `tensors` 字典中。
   - 以简单的 `Add` 算子为例，调用 `GraphObj` 中的 `add()` 方法对于图中的输入与输出节点进行连接并更新输出节点，同时更新 `tensor` 中的数据。

4. 更新节点列表：
   
   - 在每次节点处理后，将已处理的节点从 `node_list` 中移除，以便下次处理未处理的节点。

5. 数据内存分配：
   
   - 在所有节点处理完成后，调用 `self.handler.data_malloc()` 分配数据内存。

6. 处理张量数据：
   
   - 对于每个张量，将其数据拷贝到后端张量对象中，根据张量数据类型的不同进行相应的拷贝操作。

7. 输出处理：
   
   - 对模型的输出进行迭代，将其添加到 `outputs` 字典中。

## Graph

### GraphHandler

`GraphHandler` 是对 `Graph` 的一层封装。

- `tensor(Shape dims, int dtype)` 方法向图中添加一个 `Tensor` 对象。

### Graph

- `dataMalloc()`：模拟图的执行并分配内存： 
1. `topo_sort() == true`：调用 `topo_sort()` 函数执行图的拓扑排序，确保操作按正确的顺序执行。

2. `tensorToRefCount` 和 `tensorToOffset`：两个 unordered_map，用于记录每个张量的引用计数和内存偏移量。

3. `constTensor`：unordered_set，用于记录所有常量张量，包括权重张量和输入张量。常量张量会在此阶段分配内存，不会在后续阶段释放。

4. 分配常量张量的内存：对于没有来源（source）的张量，即常量张量，将其加入 `constTensor`，然后分配内存并记录偏移量。

5. 遍历操作并分配内存：对于每个操作，首先为其输出张量分配内存并记录偏移量。然后，遍历操作的输入张量，如果不是常量张量，减少其引用计数。如果引用计数减少到零，表示该张量不再被使用，执行内存释放。

6. 实际内存分配：遍历所有张量，并根据之前记录的偏移量为每个张量设置数据块。这样，张量对象将引用图内分配的内存块。

7. `allocator.info()`：输出内存分配器的信息，可能用于调试和性能分析。




