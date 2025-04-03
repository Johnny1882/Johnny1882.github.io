---
layout:     post
title:      字节流处理
subtitle:   Java与C#中字节流的区别
date:       2024-08-09
author:     JY
header-img: img/post-bg-BJJ.jpg
catalog: true
lang: zh
tags:
    - Data Stream
    - C#
---

## 介绍

## 主要内容
### 大端和小端
大端（Big Endian）和小端（Little Endian）定义了数据在物理内存中的存储方式。例如，存储一个 `long` 类型的值：0X 12 34 56 78

- **大端**：最高有效字节存储在最小的内存地址。
  在内存中，它的存储方式为：12 34 56 78（最小地址在左侧）

- **小端**：最高有效字节存储在最大的内存地址。
  在内存中，它的存储方式为：78 56 34 12

由于数据存储是以字节为单位（每个地址可以存储一个字节），因此在大端和小端之间转换只需要反转字节顺序。

值得注意的是，诸如 `short`、`int` 和 `long` 这样的数据类型会受到字节序的影响。而字符串（`string`）的存储不受影响。此外，由于 `char` 只占用一个字节，所以也不受影响。

### 网络字节序
不同的机器可能使用不同的字节序定义。Intel X86 架构使用小端字节序，而 ARM 架构的 CPU 使用大端字节序。如何确保不同机器之间的数据传输不会产生错误呢？

UDP/TCP/IP 协议规定：将接收到的第一个字节视为最高有效字节，这意味着在数据传输时使用大端字节序。

### C#中的数据流
C# 提供了 `MemoryStream`、`BinaryWriter` 和 `BinaryReader` 作为读写字节流的工具。然而，值得注意的是，`BinaryWriter` 和 `BinaryReader` 在读写操作时并不考虑网络字节序。与 Java 的 `ByteBuffer` 不同，后者在内部处理这一问题。

```C#
byte[] byteArray = new byte[] { 0x00, 0x00, 0x00, 0x01 };

using (MemoryStream msIn = new MemoryStream(byteArray))
using (BinaryReader brIn = new BinaryReader(msIn)){
    var result = brIn.ReadInt32();
    Console.Write(result);
}

// output: 16777216
```

在上面的代码中，`byteArray` 表示大端格式的 Int32 类型值 1。如果通过网络接收到一个 1，这就是我们得到的数据。但由于 `BinaryReader` 默认使用机器的存储格式（小端字节序），因此输出的是对 0x 01 00 00 00 的解释：16777216。

```C#
Int32 a = 1;
byte[] byteArray;

using (MemoryStream msIn = new MemoryStream(32))
using (BinaryWriter writer = new BinaryWriter(msIn, Encoding.UTF8)){
    writer.Write(a);
    byteArray = msIn.ToArray();
}

foreach (byte b in byteArray)
    Console.Write(b + " ");

// output: 1 0 0 0
```

同样，使用 `BinaryWriter` 将数字写入字节流时也会产生类似的问题。在上面的代码中，数字 1 以小端格式写入字节流。

```C#
long a = 1;
var ret = BitConverter.GetBytes(a);

foreach (var c in ret)
    Console.Write($"   {c}");
// output: 1 0 0 0 0 0 0 0
```

`BitConverter` 的行为方式相同。
为了解决这个问题，你可以使用 `IPAddress.HostToNetworkOrder`，或者手动使用 `Array.Reverse` 反转字节顺序。

### 参考资料

https://www.cnblogs.com/ArrozZhu/p/8384218.html
