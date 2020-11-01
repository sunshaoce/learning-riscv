## 例程1：hello world!

虽然我们是汇编语言设计，但是我们还是离不开C/C++，一个很重要的原因，便是RISC-V相关的程序的缺失。

### hello.cpp

```c++
#include <iostream>

int main() {
  std::cout << "hello world!" << std::endl;
  return 0;
}
```

### riscv64-unknown-elf-g++编译

```shell
$RISCV/newlib/bin/riscv64-unknown-elf-g++ hello.cpp -o hello
$RISCV/newlib/bin/riscv64-unknown-elf-g++ -S hello.cpp -o hello.s
```

### clang++编译（可选）

```shell
$RISCV/llvm/bin/clang++ --target=riscv64-unknown-elf -march=rv64gc --sysroot=$(RISCV)/newlib/$(RISCV_TYPE) --gcc-toolchain=$(RISCV)/newlib hello.cpp -o hello
$RISCV/llvm/bin/clang++ --target=riscv64-unknown-elf -march=rv64gc --sysroot=$(RISCV)/newlib/$(RISCV_TYPE) --gcc-toolchain=$(RISCV)/newlib -S hello.cpp -o hello.s
```

可以看到生成了`hello` 可执行程序和`hello.s`汇编文件。

### qemu运行

```shell
$RISCV/qemu/bin/qemu-riscv64 hello
```

### spike运行（可选）

```shell
$RISCV/newlib/bin/spike $RISCV/newlib/riscv64-unknown-elf/bin/pk hello
```

恭喜，至此，已经成功运行了你的第一个RISC-V程序！
