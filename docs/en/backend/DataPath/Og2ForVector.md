# Og2ForVector

- Version: V2R2
- Status: OK
- Date: 2025/01/20
- commit：[xxx](https://github.com/OpenXiangShan/XiangShan/tree/xxx)

## 总体设计

### 整体框图

![整体框图](./figure/Og2ForVector.svg)

### 接口列表

见接口文档


## 功能

对于普通标量指令，经过 DataPath 后会直接送到 BypassNetwork
中，通过多路选择生成最终的操作数。对于向量计算指令和向量访存指令，由于读向量寄存器堆的时序比标量更紧张，同时向量执行单元对第一拍数据的时序也有较高的要求，因此在经过
DataPath 后多引入一个 OG2 阶段，然后再进入 BypassNetwork 进行多路选择。

Og2ForVector 模块只进行单纯的打拍，不进行逻辑操作，能否进入 OG2 阶段只需考虑全局的取消。Og2ForVector 模块中含有 OG2
的阶段的流水级寄存器，当 OG1 阶段的指令没有收到 load cancel 或者重定向刷新就可以进入 OG2 阶段。

Og2ForVector 模块的另一个功能是向发射队列发送 OG2 阶段的回应。OG2 阶段没有自身原因的取消，OG2
能否成功发送到后级只需考虑后级能否接收指令。如果后级不能接收，指令无法进入执行单元，需要向发射队列回应 block
状态，告诉发射队列之后需要重新发射指令。如果后级可以接收，对于向量计算指令，指令一定能成功执行，此时向发射队列回应 success
状态，告诉发射队列可以清空对应的队列项；对于向量访存指令，进入访存执行单元后才能确定能否成功执行指令，这里只向发射队列回应 uncertain
状态，让其保持不动，之后由执行单元发送清空或者重发的回应。
