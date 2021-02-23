## 说明

步骤基本上是通用的，只是不同的环境下需要的target和链接器有差异。



## 配置target

查询自己目标机器上的CPU类型和操作系统类型来确定target类型

```shell
dongshuai@siaphicprd05102:~/rust$ rustup target list
aarch64-apple-ios
aarch64-fuchsia
aarch64-linux-android
aarch64-pc-windows-msvc
aarch64-unknown-linux-gnu
aarch64-unknown-linux-musl
aarch64-unknown-none
aarch64-unknown-none-softfloat
arm-linux-androideabi
arm-unknown-linux-gnueabi
arm-unknown-linux-gnueabihf
arm-unknown-linux-musleabi
arm-unknown-linux-musleabihf
armebv7r-none-eabi
armebv7r-none-eabihf
armv5te-unknown-linux-gnueabi
armv5te-unknown-linux-musleabi
armv7-linux-androideabi
armv7-unknown-linux-gnueabi
armv7-unknown-linux-gnueabihf
armv7-unknown-linux-musleabi
armv7-unknown-linux-musleabihf
armv7a-none-eabi
armv7r-none-eabi
armv7r-none-eabihf
asmjs-unknown-emscripten
i586-pc-windows-msvc
i586-unknown-linux-gnu
i586-unknown-linux-musl
i686-linux-android
i686-pc-windows-gnu
i686-pc-windows-msvc
i686-unknown-freebsd
i686-unknown-linux-gnu
i686-unknown-linux-musl
mips-unknown-linux-gnu
mips-unknown-linux-musl
mips64-unknown-linux-gnuabi64
mips64-unknown-linux-muslabi64
mips64el-unknown-linux-gnuabi64
mips64el-unknown-linux-muslabi64
mipsel-unknown-linux-gnu
mipsel-unknown-linux-musl
nvptx64-nvidia-cuda
powerpc-unknown-linux-gnu
powerpc64-unknown-linux-gnu
powerpc64le-unknown-linux-gnu
riscv32i-unknown-none-elf
riscv32imac-unknown-none-elf
riscv32imc-unknown-none-elf
riscv64gc-unknown-linux-gnu
riscv64gc-unknown-none-elf
riscv64imac-unknown-none-elf
s390x-unknown-linux-gnu
sparc64-unknown-linux-gnu
sparcv9-sun-solaris
thumbv6m-none-eabi
thumbv7em-none-eabi
thumbv7em-none-eabihf
thumbv7m-none-eabi
thumbv7neon-linux-androideabi
thumbv7neon-unknown-linux-gnueabihf
thumbv8m.base-none-eabi
thumbv8m.main-none-eabi
thumbv8m.main-none-eabihf
wasm32-unknown-emscripten
wasm32-unknown-unknown
wasm32-wasi
x86_64-apple-darwin
x86_64-apple-ios
x86_64-fortanix-unknown-sgx
x86_64-fuchsia
x86_64-linux-android
x86_64-pc-windows-gnu (installed)
x86_64-pc-windows-msvc
x86_64-rumprun-netbsd
x86_64-sun-solaris
x86_64-unknown-cloudabi
x86_64-unknown-freebsd
x86_64-unknown-linux-gnu (installed)
x86_64-unknown-linux-gnux32
x86_64-unknown-linux-musl
x86_64-unknown-netbsd
x86_64-unknown-redox

```

如这次目标机器是WIN10 64bit，使用gnu，所以我们选择x86_64-pc-windows-gnu。

```shell
dongshuai@siaphicprd05102:~/rust$ rustup target add x86_64-pc-windows-gnu
info: downloading component 'rust-std' for 'x86_64-pc-windows-gnu'
info: installing component 'rust-std' for 'x86_64-pc-windows-gnu'
info: Defaulting to 500.0 MiB unpack ram
 14.1 MiB /  14.1 MiB (100 %)   9.9 MiB/s in  1s ETA:  0s
```



## 安装对应的连接器

目标机器是WIN64，所以需要安装`mingw-w64`来链接。

由于我源机器是ubuntu，所以安装MingW非常方便，使用root用户在线安装即可。

```shell
# apt-get install mingw-w64
```

安装完成后，会默认在 `/usr/bin` 下生成很多 `x86_64` 打头二进制文件，这里我们需要用的是

```shell
lrwxrwxrwx 1 root root      40 Sep 15 19:20 x86_64-w64-mingw32-gcc -> /etc/alternatives/x86_64-w64-mingw32-gcc*
```



## 配置target对应的链接器

编辑器 `.cargo/config` 文件，设置target对应的链接器

```toml
[target.x86_64-pc-windows-gnu]
linker = "x86_64-w64-mingw32-gcc"
```



## 构建Target版本

```shell
dongshuai@siaphicprd05102:~/rust/hello/src$ cargo build --target x86_64-pc-windows-gnu
   Compiling hello v0.1.0 (/home/dongshuai/rust/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32s
```



