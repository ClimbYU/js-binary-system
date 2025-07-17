# 关于前端二进制

### 一些概念
ArrayBuffer，TypedArray，Int8Array，Unit8Array，Int16Array，Int32Array，Float32Array，DataView，File，Blob
这些是什么有啥关联


1.ArrayBuffer：ArrayBuffer是​​原始二进制数据的固定长度容器​​，代表一块连续的内存区域，但无法直接读写
```javascript
const buffer = new ArrayBuffer(16); // 创建16字节缓冲区
```
2.类型化视图：TypedArray(https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray),TypedArray是操作 ArrayBuffer的​​类型化视图集合​​，包含多种具体类型:

* ​​Int8Array/Uint8Array​​
8位整数视图，用于处理字节数据（如像素、网络包）。
Uint8ClampedArray在赋值时自动钳制值到 0-255（适合图像处理）
* ​Int16Array/Uint16Array
16位整数视图，适用于中等精度数据（如音频采样）
* Int32Array/Uint32Array
32位整数视图，用于通用整数运算
* ​​Float32Array/Float64Array​​
32/64位浮点数视图，适合高精度计算（如3D坐标、科学计算）。

###### TypedArray核心特性
* ​​共享缓冲区​​：多个视图可基于同一 ArrayBuffer，修改会相互影响
```javascript
const buffer = new ArrayBuffer(8); // 8字节内存：[0,0,0,0,0,0,0,0]
const int32 = new Int32Array(buffer); // 按32位整数解析：[0, 0]
int32[0] = 1000; // 二进制：00000000 00000000 00000011 11101000  写入内存 [232, 3, 0, 0]
int32[1] = 2000; // 二进制：00000000 00000000 00000111 11010000  写入内存 [208, 7, 0, 0]
// 若用Uint8Array查看同一内存：
const uint8 = new Uint8Array(buffer);
console.log(uint8); // [232, 3, 0, 0, 208, 7, 0, 0]
// 1000→ 十六进制 0x000003E8→ 字节序 [0xE8, 0x03, 0x00, 0x00]
// 2000→ 十六进制 0x000007D0→ 字节序 [0xD0, 0x07, 0x00, 0x00]
// 低字节在前（小端序）：
// int32[0] = 1000 → 字节0-3：232(0xE8), 3(0x03), 0, 0
// int32[1] = 2000 → 字节4-7：208(0xD0), 7(0x07), 0, 0

const int8 = new Int8Array(buffer);
console.log(int8); // 输出: [ -24, 3, 0, 0, -48, 7, 0, 0 ]
// 232→ 有符号8位整数范围为 -128~127，232超出范围转为 -24（232 - 256 = -24）
// 208→ 同样转为 -48（208 - 256 = -48）
// 输出：[-24, 3, 0, 0, -48, 7, 0, 0]

// 使用Int32Array视图
const int16 = new Int16Array(buffer);
console.log(int16); // 输出: [ 1000, 0, 2000, 0 ]
// 前2字节：0xE8 0x03→ 小端序组合为 0x03E8= 1000
// 后续2字节：0x00 0x00→ 组合为 0
// 后4字节同理：[0xD0, 0x07] → 0x07D0 = 2000，[0x00, 0x00] → 0
// 输出：[1000, 0, 2000, 0]

``` 

* ​​​内存高效​​：数据连续存储，访问速度优于普通数组。
* ​​固定类型与长度​​：创建后无法增减元素或改变类型。

3.DataView
提供​​低层级、字节序可控​​的 ArrayBuffer访问方式，支持非对齐读写和跨平台字节序处理。

```javascript
const buffer = new ArrayBuffer(8)
const view = new DataView(buffer);
view.setInt32(0, 256, true); // 小端序写入32位整数
console.log(view.getUint8(0)); // 0（小端序首字节）[4](@ref)
```

4.Blob​​
​​定义​​：​​不可变的类文件对象​​，可存储任意类型数据（如文本、二进制），含 MIME 类型元数据。
​​来源​​：可由 ArrayBuffer、字符串或其他 Blob组合生成。
​​用途​​：文件分片上传、URL.createObjectURL()生成临时链接。

5.​​File​​
​​定义​​：继承自 Blob，添加文件名、最后修改时间等元数据。
​​来源​​：通常来自 <input type="file">或拖拽操作。

