+++
title="LZW压缩算法"
date=2024-05-03

[taxonomies]
categories=["Programming"]
tags=["compression", "gif", "encode", "decode"]
+++

LZW(Lempel-Ziv-Welch)压缩算法是基于词典的压缩算法。
# LZW算法

## encode

编码时需要构造LZW词典。
1. 以所有可能的字符初始化LZW词典。
2. 模式p加上新字符a，构成待处理字符串p\*a.
3. 如果p\*a在字典中，则 p = p\*a，继续处理2；
4. 如果p\*a不在字典中，把p\*a加入词典，把p的索引作为编码器输出，并把a作为新的模式p，继续处理2。

```rust
fn encode(s:&str)->Vec<u16>
{
	let mut table = HashMap::new();
	for i in 0..256 {
		table.insert(vec![i as u8], i);
	}
	let mut code = 256;
	let mut buf = vec![]; // pattern p
	let mut out = vec![];
    let bytes = s.as_bytes();
	for c in bytes.iter() {
        buf.push(*c); // p*a
		if !table.contains_key(&buf) {
            table.insert(buf.clone(), code);
            // prev = &buf[0..buf.len()-1]
            // prev code = table[&buf[0..buf.len()-1]]
            out.push(table[&buf[0..buf.len()-1]]);
            buf.clear();
            buf.push(*c); // new pattern with a
            code += 1;
        }
	}
    out.push(table[&buf]);
	out
}
```

## decode
解码器初始词典与编码器相同。
解码算法不好理解，但是可以观察到编码时词典的当前项由输出的前一项 + 后一项的首字符构成。反过来在解码时，可以由前一项 + 后一项的首字符构造词典的当前项。

但是有一个问题是，解码时编码可能不在词典中，解码比编码有一步延迟；一般情况下编码时输出编码是N-1，如果编码时最后输出的编码是词典当前项N，可以注意到在这种情况下编码对应的字符串的*首字符*就是词典当前项的*尾字符*，即词典当前项首尾字符相同，所以解码时由前一项输出可构造出词典当前项。

```rust
fn decode(input: &[u16]) -> String
{
    let mut table:HashMap<u16,Vec<u8>> = HashMap::new();
	for i in 0..256 {
		table.insert(i,vec![i as u8]);
	}
	let mut prev = vec![]; // pattern p
	let mut out = vec![];
	let mut code = 256;
	for c in input.iter() {
		if !table.contains_key(c) {
			let mut s = prev.clone();
			s.push(s[0]); // p*a
			table.insert(code, s);
		}
		out.extend(&table[c]);
        if !prev.is_empty() {
            prev.push(table[c][0]); // p*a
            table.insert(code, mem::take(&mut prev));
            code += 1;
        }
        prev.extend(&table[c]);
	}
	String::from_utf8(out).unwrap()
}
```

# LZW in GIF
GIF图像数据是基于调色板(palette or color table)。
图像数据压缩时把颜色的索引(color index)，转换为LZW编码的bit流。
GIF中LZW基于color index构造词典。
GIF中的LZW编码有控制码（control code）。
- CC (Clear Code)  用于重置词典
- EOI (End Of Information Code) 用于指示图像数据结束，`<Clear code>+1`

## Code Size
Code Size定义了表达像素值需要的最小bit数。
通常情况下和颜色的bit数相同。但是黑白图像只需要2bit。

## Clear Code
Clear Code定义为 `2 ** <Code Size>`。

## Variable Code Length
编码输出code是动态长度的，从`<Code Size>+1` bits到12 bits。每当输出的code的长度要超出当前长度时，code length需要加1。动态长度编码有助于进一步压缩数据。

# References
- https://go-compression.github.io/algorithms/lz/
- https://marknelson.us/posts/2011/11/08/lzw-revisited
- https://zhuanlan.zhihu.com/p/85036591
- https://www.cs.cmu.edu/~cil/lzw.and.gif.txt
- https://burqee.com/article/12/
-《数据压缩导论第4版》
- https://giflib.sourceforge.net/whatsinagif/lzw_image_data.html
