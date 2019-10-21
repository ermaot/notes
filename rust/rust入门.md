## 安装
#### windows
直接点击链接[下载链接](https://win.rustup.rs/)即可下载安装文件

#### linux：
```
# curl https://sh.rustup.rs -sSf  | sh
info: downloading installer

Welcome to Rust!

This will download and install the official compiler for the Rust programming 
language, and its package manager, Cargo.

It will add the cargo, rustc, rustup and other commands to Cargo's bin 
directory, located at:

  /root/.cargo/bin

This can be modified with the CARGO_HOME environment variable.

Rustup metadata and toolchains will be installed into the Rustup home 
directory, located at:

  /root/.rustup

This can be modified with the RUSTUP_HOME environment variable.

This path will then be added to your PATH environment variable by modifying the
profile files located at:

  /root/.profile
  /root/.bash_profile

You can uninstall at any time with rustup self uninstall and these changes will
be reverted.

Current installation options:

   default host triple: x86_64-unknown-linux-gnu
     default toolchain: stable
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
>

info: syncing channel updates for 'stable-x86_64-unknown-linux-gnu'
info: latest update on 2019-08-15, rust version 1.37.0 (eae3437df 2019-08-13)
info: downloading component 'rustc'
 85.3 MiB /  85.3 MiB (100 %)   3.1 MiB/s in 41s ETA:  0s    
info: downloading component 'rust-std'
 61.2 MiB /  61.2 MiB (100 %)   1.5 MiB/s in 33s ETA:  0s
info: downloading component 'cargo'
  4.6 MiB /   4.6 MiB (100 %)   1.7 MiB/s in  3s ETA:  0s
info: downloading component 'rust-docs'
 11.3 MiB /  11.3 MiB (100 %)   1.9 MiB/s in  5s ETA:  0s
info: installing component 'rustc'
 85.3 MiB /  85.3 MiB (100 %)   8.3 MiB/s in  8s ETA:  0s
info: installing component 'rust-std'
 61.2 MiB /  61.2 MiB (100 %)  11.7 MiB/s in  5s ETA:  0s
info: installing component 'cargo'
info: installing component 'rust-docs'
 11.3 MiB /  11.3 MiB (100 %)   1.0 MiB/s in  8s ETA:  0s
info: default toolchain set to 'stable'

  stable installed - rustc 1.37.0 (eae3437df 2019-08-13)


Rust is installed now. Great!

To get started you need Cargo's bin directory ($HOME/.cargo/bin) in your PATH 
environment variable. Next time you log in this will be done automatically.

To configure your current shell run source $HOME/.cargo/env

```
#### 卸载
```
# rustup self uninstall
```
#### 简单命令
```
# rustc --version
rustc 1.37.0 (eae3437df 2019-08-13)

```
#### 其他
- Rust 开发工具的路径：类 Unix 系统下是  ~/.cargo/bin ，在 Windows 下是  %USERPROFILE%\.cargo\bin 
- Rust 并没有自己的连接器。对于 Linux 系统，Rust 会尝试调用  cc 进行连接。对于  windows-msvc （在Windows 上使用 Microsoft Visual Studio 构建的 Rust）


## Hello World
vim main.rs
```
fn main(){
    println!("Hello World");
}

```

```
# rustc main.rs 
# ./main 
Hello World

```
细节解释：
1. 是使用4个空格缩进，而不是制表符
2. println!() 是一个 Rust 宏而非函数
3. 分号结尾
4. 编译使用rustc xxx.rs