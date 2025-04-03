---
layout:     post
title:      Byte Stream
subtitle:   Byte Stream Handle in C#
date:       2024-08-09
author:     JY
header-img: img/post-bg-BJJ.jpg
catalog: true
lang: en
tags:
    - Data Stream
    - C#
---

## Introduction

## Main Content
### Big Endian and Little Endian
Big Endian and Little Endian define how data is stored in physical memory. For example, storing a `long`: 0X 12 34 56 78

- **Big Endian**: The most significant byte is stored at the smallest memory address.
  In memory, it looks like: 12 34 56 78 (smallest address on the left)

- **Little Endian**: The most significant byte is stored at the largest memory address.
  In memory, it looks like: 78 56 34 12


Since data storage is byte-oriented (each address can store one byte), converting between Big Endian and Little Endian only requires reversing the byte order.

It is worth noting that data types such as `short`, `int`, and `long` are affected by endianness. The storage of strings (`string`) is not affected. Additionally, `char` is not affected because it only occupies one byte.

### Network Byte Order
Different machines may use different endianness definitions. Intel X86 architecture uses Little Endian, while ARM architecture CPUs use Big Endian. How can we ensure that data transmission between different machines does not cause errors?

The UDP/TCP/IP protocols stipulate: treat the first received byte as the most significant byte, which means using Big Endian for data transmission.

### Data Streams in C#
C# provides `MemoryStream`, `BinaryWriter`, and `BinaryReader` as tools for reading and writing byte streams. However, it is worth noting that `BinaryWriter` and `BinaryReader` do not consider network byte order during read/write operations. Unlike Java's `ByteBuffer`, which handles this internally.

```C#
byte[] byteArray = new byte[] { 0x00, 0x00, 0x00, 0x01 };

using (MemoryStream msIn = new MemoryStream(byteArray))
using (BinaryReader brIn = new BinaryReader(msIn)){
    var result = brIn.ReadInt32();
    Console.Write(result);
}

// output: 16777216
```

In the above code, `byteArray` represents an Int32 of 1 in Big Endian format. If a 1 is received over the network, this is the data we get. But BinaryReader defaults to the machine's storage format, which is Little Endian, so the output is the interpretation of 0x 01 00 00 00: 16777216.

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
Similarly, writing numbers to a byte stream using `BinaryWriter` will produce similar issues. In the above code, the number 1 is written in Little Endian format.

```C#
long a = 1;
var ret = BitConverter.GetBytes(a);

foreach (var c in ret)
    Console.Write($"   {c}");
// output: 1 0 0 0 0 0 0 0
```

`BitConverter` behaves the same way.
To address this issue, you can use `IPAddress.HostToNetworkOrder`, or manually use `Array.Reverse` to reverse the byte order.

### References

https://www.cnblogs.com/ArrozZhu/p/8384218.html