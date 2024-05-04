+++
title="Javascript source map"
date=2024-05-04

[taxonomies]
categories=["Programming"]
tags=["sourcemap", "javascript", "encode", "decode"]
+++

Javascript一般需要打包压缩，或者从其它语言转换到JS。需要用source map来映射代码转换前后的关系，方便调试代码。

## source map 结构

```json
{
  "version" : 3,
  "file": "out.js",
  "sourceRoot": "",
  "sources": ["foo.js", "bar.js"],
  "sourcesContent": [null, null],
  "names": ["src", "maps", "are", "fun"],
  "mappings": "A,AAAB;;ABCDE",
  "ignoreList": [0]
}
```

file: 生成的文件
sourceRoot: 指定根路径，可以简化sources的路径
sourceContent: 源代码
names: mappings中使用的符号
mappings：编码后的映射数据
ignoreList：在开发者工具中应当忽略的第三方代码
## mappings
mappings定义如下：
- 使用分号(;)隔开分组，每个分号(;)代表一行
- 使用逗号(,)隔开每个segment
- 每个segment可能有1、4或5个数字组成

segment每个field的定义：
1. 生成代码中的列号
2. sources的索引值
3. 原始代码的行号
4. 原始代码的列号
5. names的索引值

此外，segment每个field额外约束是第一次出现使用完整值，之后都使用差值。
使用差值可以减小编码后数据的大小。
## Base64 VLQ

**variable-length quantity** (**VLQ**) 用来编码整型数字。

> Base64 VLQ: [[VLQ]](https://tc39.es/source-map/#biblio-vlq "Variable-length quantity") is a [[base64]](https://tc39.es/source-map/#biblio-base64 "The Base16, Base32, and Base64 Data Encodings") value, where the most significant bit (the 6th bit) is used as the continuation bit, and the "digits" are encoded into the string least significant first, and where the least significant bit of the first digit is used as the sign bit.

把数值编码成VLQ形式，然后编码成Base64形式。
为了适合Base 64编码，选择了6 bit VLQ编码。

```
 5   | 4 | 3 | 2 | 1 | 0
cont | x | x | x | x | sign
```
如图所示，最高位指示后续是否是当前数字的一部分；最低位表示符号，1表示负数；这里不包括符号位的有效bit只有4位，最高只能表示15 （0b1111）。
超出这个范围，就需要用到continuation bit，下一个Base64值显然只有5位有效bit，这个时候没有符号位。
```
 5  | 4 | 3 | 2 | 1 | 0     |   5  | 4 | 3 | 2 | 1 | 0
 1  | x | x | x | x | sign  | cont | x | x | x | x | x 
```

理解VLQ之后，编码和解码很直观。
### 编码
```rust
fn base64_encode(v: u8) -> u8
{
	static BASE64_LUT:&[u8] = b"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
	BASE64_LUT[v as usize]
}
fn encode(num:i32) -> Vec<u8>
{
	let mut res = vec![];
	let mut num = if num < 0 {
		((-num) << 1) | 1 
	} else {
		num << 1
	};
	while num != 0 {
		let mut v = (num & 0b11111) as u8;
		num = num >> 5;
		if num != 0 {
			v |= 1 << 5;
		}
		res.push(base64_encode(v));
	}
	res
}
```

### 解码
```rust
fn base64_decode(v:u8) -> u8
{
	match v {
		b'A'..=b'Z' => v - b'A',
		b'a'..=b'z' => v - b'a' + 26,
		b'0'..=b'9' => v - b'0' + 52,
		b'+' => 62,
		b'/' => 63,
		_ => unreachable!(),
	}
}

fn decode(bytes: &[u8]) -> Option<(i32,usize)>
{
	let mut num:i32 = 0;
	let mut n_bits = 0;
	let mut cont = 1;
	let mut n = 0;
	for &v in bytes.iter() {
		let v = base64_decode(v);
		num += ((v & 0b11111) as i32) << n_bits;
		cont = v & (1 << 5);
		n += 1;
		if cont == 0 {
			break;
		}
		n_bits += 5;
	}
	if cont != 0 {
		return None;
	}
	if (num & 1) != 0 {
		num = -(num >> 1)
	} else {
		num = num >> 1
	}
	Some((num, n))
}
```

## source map怎么使用
有几种形式，其中HTTP header优先级最高
- HTTP header
```
sourcemap: <url>
```
- 通过url的形式在生成的代码中指定
```
//# sourceMappingUrl=<url>
```
- 通过base64 url的形式把source map内嵌在生成的代码中
```
//# sourceMappingUrl=data:application/json;charset=utf-8;base64,......
```

出于兼容性`//# sourceMappingUrl`这里的`#`可以使用`@`符号代替。新生成的代码应该使用`#`。

## References
- https://tc39.es/source-map/
- https://en.wikipedia.org/wiki/Variable-length_quantity
- https://www.murzwin.com/base64vlq.html