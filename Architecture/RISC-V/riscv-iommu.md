# RISC-V IOMMU

对于通过 IOMMU 连接到系统的每个 I/O 设备，软件可以在 IOMMU 上配置一个设备上下文，它与设备关联一个特定的虚拟地址空间和其他每个设备的参数。 通过在 IOMMU 中为每个设备提供自己独立的设备上下文，可以为每个设备单独配置一个单独的操作系统，该操作系统可以是 guest os 或者 host os。 在设备发起的每次内存访问中，IOMMU 通过某种形式的唯一设备标识符来识别原始设备，然后 IOMMU 使用该标识符在软件提供的数据结构中定位适当的设备上下文。 例如，对于 PCIe [1]，原始设备可以通过 PCI 总线号（8 位）、设备号（5 位）和功能号（3 位）的唯一 16 位三元组（统称为 称为路由标识符或 RID），当 IOMMU 支持多个层次结构时，可以选择最多 8 位段号。 本规范引用了诸如 device_id 之类的唯一设备标识符，并支持最多 24 位宽的标识符。
