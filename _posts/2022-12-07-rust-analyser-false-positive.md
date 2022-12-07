---
layout: single
categories: 
    - Rust 
author: Yue Yin

toc: false
toc_sticky: false 
---

When reading TiKV code in VS Code, `cargo build` works but rust-analyzer generates false positive error. Error output:

```
[ERROR rust_analyzer::lsp_utils] failed to run build scripts

error[E0592]: duplicate definitions with name `generate_files`
  --> /Users/yy0125/.cargo/registry/src/github.com-1ecc6299db9ec823/protobuf-build-0.13.0/src/protobuf_impl.rs:63:5
   |
63 |     pub fn generate_files(&self) {
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ duplicate definitions for `generate_files`
   |
  ::: /Users/yy0125/.cargo/registry/src/github.com-1ecc6299db9ec823/protobuf-build-0.13.0/src/prost_impl.rs:5:5
   |
5  |     pub fn generate_files(&self) {
   |     ---------------------------- other definition for `generate_files`


error: aborting due to previous error


For more information about this error, try `rustc --explain E0592`.

error: could not compile `protobuf-build` due to 2 previous errors
```

I noticed that, running the test by clicking `Run`, the testcase fails to compile for the same reason. I tried Google but did not find anything helpful. 

I also noticed that everything works fine when running the test in JetBrains CLion with Rust extension. So I looked at the commands used by VS Code and CLion to run the test:


VS Code:
```
cargo run --package raft --example five_mem_node --all-feature
```

CLion:
```
cargo run --package raft --example five_mem_node 
```

So the problem must be the `--all-feature` flag! This must be related to the `defaulte-feature = false` in `Cargo.toml`, but I am a bit too lazy to dig into this now...

In VS Code `settings.json`, comment out `"rust-analyzer.cargo.features": "all",`. Everything works now!

