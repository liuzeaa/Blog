---
title: Buffer对象
date: 2018-02-06 13:43:40
tags: Node.js
keywords: Node.js Buffer对象
---
# Buffer对象
在 ECMAScript 2015 (ES6) 引入 TypedArray 之前，JavaScript 语言没有读取或操作二进制数据流的机制。 Buffer 类被引入作为 Node.js API 的一部分，使其可以在 TCP 流或文件系统操作等场景中处理二进制数据流。
Buffer 类在 Node.js 中是一个全局变量，因此无需使用 require('buffer')。
<!--more-->
## 创建一个Buffer对象
Buffer对象类似于数组，它的元素为16进制的两位数，即0到255的数值。
- **Buffer.from(array)** 返回一个新建的包含所提供的字节数组的副本的 Buffer。
- **Buffer.from(arrayBuffer[, byteOffset [, length]])** 返回一个新建的与给定的 ArrayBuffer 共享同一内存的 Buffer。
- **Buffer.from(buffer)** 返回一个新建的包含所提供的 Buffer 的内容的副本的 Buffer。
- **Buffer.from(string[, encoding])** 返回一个新建的包含所提供的字符串的副本的 Buffer。
- **Buffer.alloc(size[, fill[, encoding]])** 返回一个指定大小的被填满的 Buffer 实例。这个方法会明显地比 Buffer.allocUnsafe(size) 慢，但可确保新创建的 Buffer 实例绝不会包含旧的和潜在的敏感数据。
- **Buffer.allocUnsafe(size)与 Buffer.allocUnsafeSlow(size)** 返回一个新建的指定 size 的 Buffer，但它的内容必须被初始化，可以使用 buf.fill(0) 或完全写满。
```
// 创建一个长度为 10、且用 0 填充的 Buffer。
const buf1 = Buffer.alloc(10);

// 创建一个长度为 10、且用 0x1 填充的 Buffer。 
const buf2 = Buffer.alloc(10, 1);

// 创建一个长度为 10、且未初始化的 Buffer。
// 这个方法比调用 Buffer.alloc() 更快，
// 但返回的 Buffer 实例可能包含旧数据，
// 因此需要使用 fill() 或 write() 重写。
const buf3 = Buffer.allocUnsafe(10);
buf3.fill(0)

// 创建一个包含 [0x1, 0x2, 0x3] 的 Buffer。数组里一定是0-255的数，否则会不识别，返回00
const buf4 = Buffer.from([1, 2, 3]);

// 创建一个包含 UTF-8 字节 [0x74, 0xc3, 0xa9, 0x73, 0x74] 的 Buffer。
const buf5 = Buffer.from('tést');

// 创建一个包含 Latin-1 字节 [0x74, 0xe9, 0x73, 0x74] 的 Buffer。
const buf6 = Buffer.from('tést', 'latin1');
```
在 Node.js v6 之前的版本中Buffer 实例是通过 new Buffer 构造函数创建的，因为 new Buffer() 的行为会根据所传入的第一个参数的值的数据类型而明显地改变，所以如果应用程序没有正确地校验传给 new Buffer() 的参数、或未能正确地初始化新分配的 Buffer 的内容，就有可能在无意中为他们的代码引入安全性与可靠性问题。
为了使 Buffer 实例的创建更可靠、更不容易出错，各种 new Buffer() 构造函数已被 废弃，并由 Buffer.from()、Buffer.alloc()、和 Buffer.allocUnsafe() 方法替代。

** Node.js 建议开发者们应当把所有正在使用的 new Buffer() 构造函数迁移到这些新的 API 上。**


## Buffer 的转换
Buffer对象与普通的 JavaScript 字符串的互相转换，需要指定编码格式。目前Node.js 目前支持以下的字符编码。
- 'ascii' - 仅支持 7 位 ASCII 数据。
- 'utf8'
- 'utf16le'
- 'ucs2'
- 'base64'
- 'latin1'
- 'binary'
- 'hex'

```
// 字符串转Buffer
const buf = Buffer.from('node', 'ascii');  

// Buffer转hex编码字符串 - 输出 6e6f6465
console.log(buf.toString('hex'));

// Buffer转base64编码字符串 - 输出 bm9kZQ==
console.log(buf.toString('base64'));
```
### 字符串转Buffer
Buffer.from(string[, encoding])
encoding 不传参数会默认utf8编码进行转码和存储。

### 字符串转Buffer
实例方法
buf.toString([encoding[, start[, end]]])
encoding 解码使用的字符编码。默认: 'utf8'。
start 开始解码的字节偏移量。默认: 0。
end 结束解码的字节偏移量（不包含）。 默认: buf.length。
注：如果Buffer对象由多种编码写入，就需要在局部指定定不同的编码，才能转换回正常的编码。

## 判断一个对象是否是Buffer对象
Buffer.isBuffer(obj)
如果 obj 是一个 Buffer 则返回 true ，否则返回 false 。
```
const buf1 = Buffer.alloc(10);
console.log(Buffer.isBuffer(buf1)) # 返回true
```

## 合并Buffer
Buffer.concat(list[, totalLength])
list &lt;Array&gt; 要合并的 Buffer 或 Uint8Array 实例的数组
totalLength &lt;integer&gt; 合并时 list 中 Buffer 实例的总长度
返回一个合并了 list 中所有 Buffer 实例的新建的 Buffer 。
如果 list 中没有元素、或 totalLength 为 0 ，则返回一个新建的长度为 0 的 Buffer 。
如果没有提供 totalLength ，则从 list 中的 Buffer 实例计算得到。 为了计算 totalLength 会导致需要执行额外的循环，所以提供明确的长度会运行更快。
如果提供了 totalLength，totalLength 必须是一个正整数。如果从 list 中计算得到的 Buffer 长度超过了 totalLength，则合并的结果将会被截断为 totalLength 的长度。
```
const buf1 = Buffer.from('蓝');
const buf2 = Buffer.from('胖');
const buf3 = Buffer.from('纸');

const len =  buf1.length + buf2.length + buf3.length;
console.log(len) # 输出9 一个汉字三个字节

console.log(Buffer.concat([buf1, buf2, buf3], len).toString())
```

## 获取字符长度
Buffer.byteLength(string[, encoding])
encoding 不传参数会默认utf8编码
```
console.log(Buffer.byteLength('蓝胖')) # 输出6 一个汉字三个字节
```

# 参考
[http://nodejs.cn/api/buffer.html](http://nodejs.cn/api/buffer.html) 
