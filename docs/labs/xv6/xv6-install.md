# xv6 编译安装,启动

系统: macOS 10.13.6

安装 homebrew

安装 risc-v 工具链

$ brew tap riscv/riscv
$ brew install riscv-tools
PATH=$PATH:/usr/local/opt/riscv-gnu-toolchain/bin

安装 qemu

brew install qemu


$ riscv64-unknown-elf-gcc --version 测试 rsicv 工具链
riscv64-unknown-elf-gcc (GCC) 10.1.0
...

$ qemu-system-riscv64 --version 测试 qemu
QEMU emulator version 5.1.0


make qemu 进入 qemu 并启动 xv6

CTRL+a x 退出qemu

https://pdos.csail.mit.edu/6.828/2020/tools.html