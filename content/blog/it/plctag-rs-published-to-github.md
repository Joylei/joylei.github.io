+++
title = "plctag-rs published to Github"
date=2020-09-02

[taxonomies]
categories=["Programming"]
tags=["plc", "plctag", "tag", "UDT", "rust", "ffi"]
+++

今天把我为[`libplctag`](https://github.com/libplctag/libplctag)写的`rust`封装[`plctag-rs`](https://github.com/Joylei/plctag-rs)发布到了`github`。

## plctag-rs

```text
a rust wrapper of libplctag, with rust style APIs and useful extensions.
```

## 功能

- 同步API
- 异步API
- Path Builder
- UDT支持

## 怎么使用

- 下载`libplctag`的二进制发布，并解压。
- 设置环境变量`LIBPLCTAG_PATH`指向之前解压到的目录。
- 把`plctag`添加到`Cargo.toml`

```toml
[dependencies]
plctag= { git="https://github.com/Joylei/plctag-rs.git"}
```

## 示例

### 同步读写

```rust
use plctag::{Accessor, RawTag};
let timeout = 100;//ms
let path="protocol=ab-eip&plc=controllogix&path=1,0&gateway=192.168.1.120&name=MyTag1&elem_count=1&elem_size=16";// YOUR TAG DEFINITION
let tag = RawTag::new(path, timeout).unwrap();

//read tag
let status = tag.read(timeout);
assert!(status.is_ok());
let offset = 0;
let value:u16 = tag.get_value(offset).unwrap();
println!("tag value: {}", value);

let value = value + 10;
tag.set_value(offset, value).unwrap();

//write tag
let status = tag.write(timeout);
assert!(status.is_ok());
println!("write done!");
```

### 异步读写

```rust
use plctag::future::AsyncTag;
use tokio::runtime::Runtime;

let mut rt = Runtime::new()::unwrap();
rt.block_on(async move {
    // YOUR TAG DEFINITION
    let path="protocol=ab-eip&plc=controllogix&path=1,0&gateway=192.168.1.120&name=MyTag1&elem_count=1&elem_size=16";
    let tag = AsyncTag::new(path).await.unwrap();

    let offset = 0;
    let value:u16 = 100;
    //write tag
    tag.set_and_write(offset, value).await.unwrap();
    // read tag
    let value:u16 = tag.read_and_get(offset).await.unwrap();
    assert_eq!(value, 100);
});
```

### UDT

```rust
use plctag::{Accessor, RawTag, Result, TagValue};

// define your UDT
#[derive(Default, Debug)]
struct MyUDT {
    v1:u16,
    v2:u16,
}
impl TagValue for MyUDT {
    fn get_value(&mut self, tag: &RawTag, offset: u32) -> Result<()>{
        self.v1.get_value(tag, offset)?;
        self.v2.get_value(tag, offset + 2)?;
        Ok(())
    }

    fn set_value(&self, tag: &RawTag, offset: u32) -> Result<()>{
        self.v1.set_value(tag, offset)?;
        self.v2.set_value(tag, offset + 2)?;
        Ok(())
    }
}

fn main(){
    let timeout = 100;//ms
    // YOUR TAG DEFINITION
    let path="protocol=ab-eip&plc=controllogix&path=1,0&gateway=192.168.1.120&name=MyTag2&elem_count=2&elem_size=16";
    let tag = RawTag::new(path, timeout).unwrap();

    //read tag
    let status = tag.read(timeout);
    assert!(status.is_ok());
    let offset = 0;
    let mut value:MyUDT = tag.get_value(offset).unwrap();
    println!("tag value: {:?}", value);

    value.v1 = value.v1 + 10;
    tag.set_value(offset, value).unwrap();

    //write tag
    let status = tag.write(timeout);
    assert!(status.is_ok());
    println!("write done!");
}
```

## More

后续可能添加`Controller`及`scan`模式来简化使用的难度。
