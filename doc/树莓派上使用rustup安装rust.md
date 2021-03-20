在自己的树莓派上下载失败，提示104，采用如下办法成功安装。

因为国内防火墙的原因，导致rust不能正常安装，如此有2个选择：
1. 搭墙，因为翻墙有风险而且速度也不佳，此处不推荐并省略。
2. 使用中科大的代理

Rust Toolchain 反向代理： https://mirrors.ustc.edu.cn/help/rust-static.html

Rust Crates 源使用帮助： https://mirrors.ustc.edu.cn/help/crates.io-index.html

Rust 官网： https://www.rust-lang.org/learn/get-started

以下在Ubuntu上安装过一遍，linux的其他发行版应该也没有问题，除了.bashrc可能是profile
 
直接复制下面的shell代码执行即可，安装过程会提示选择，选择默认项即可，也可以自定义选择安装。

```shell
# import USTC mirror
echo "export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static" >> ~/.bashrc
echo "export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup" >> ~/.bashrc
source .bashrc

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 安装完毕后刷新环境变量
source ~/.cargo/env
```
