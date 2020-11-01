# RISC-V 学习手册

## 前言

本书参阅了大量的资料，在此一并感谢！

### 参考资料

1. RISC-V Spec：https://github.com/riscv/riscv-isa-manual

2. PLCT 实验室相关视频：https://space.bilibili.com/296494084

   

## 安装RISC-V编译环境

### 环境要求

我们需要Ubuntu系统，来进行环境配置，其他的Linux版本，可以仿照此来进行。然后需要一些基础的库。

`sudo apt install python gcc autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev make cmake`

再选择一个目录，作为RISC-V编译程序的存放位置。然后设置环境变量，此处的`~/RISCV`可以替换。

`RISCV=~/RISCV`

我们生成的是64位的版本，32位的请参考源代码相应的手册。

### 注意事项

如果源代码下载很慢，请自行学习科学上网。

如果源代码编译失败，请考虑增加你的物理内存（或者学习如何增加虚拟内存），并将`make -j $(nproc)`全部改为`make`。

### riscv-gnu-toolchain

这是一个交叉编译的工具链，能让我们在x86架构的CPU下，生成RISC-V的程序。

首先便是下载源代码：

```shell
git clone --recursive https://github.com/riscv/riscv-gnu-toolchain.git
```

然后对源代码进行编译，创建一个目录build，用以存放生成的编译程序（newlib和linux是两个不同的函数库，可以选择一个）：

```shell
cd riscv-gnu-toolchain
mkdir build/newlib build/linux
cd build
```

构建函数库，可以`configure --with-arch=rv32gc --with-abi=ilp32d`来生成RISC-V特定版本的编译器。

构建newlib函数库：

```shell
cd newlib
../../configure --prefix=$RISCV/newlib
make -j $(nproc)
cd ..
```

构建linux函数库

```shell
mkdir linux
cd linux
../../configure --prefix=$RISCV/linux
make -j $(nproc) linux
cd ..
```

至此，我们已经可以编译C/C++程序了。

在`$RISCV/newlib`或`$RISCV/linux`下的`bin`里面，我们可以看到`riscv64-unknown-elf-gcc`和`riscv64-unknown-elf-g++`（或

`riscv64-unknown-linux-gnu-gcc`和`riscv64-unknown-linux-gnu-g++`）的编译器，其用法和`gcc`与`g++`无异。

### qemu

这个是一个模拟器，代码已经被包含在`riscv-gnu-toolchain`里面。

开始构建。

```shell
cd qemu
mkdir build
../configure --target-list=riscv64-softmmu,riscv64-linux-user --prefix=$RISCV/qemu
make -j $(nproc)
```

在`$RISCV/qemu/bin`里面，可以看到`qemu-riscv64`，可以运行上述交叉编译器生成的程序。

### riscv-isa-sim（可选）

这是另外一个模拟器spike，依旧是先下载源代码：

```shell
git clone https://github.com/riscv/riscv-isa-sim.git
```

然后编译为两个版本：

```shell
cd riscv-isa-sim
mkdir build
cd build

../configure --prefix=$RISCV/newlib
make -j $(nproc)
make install

../configure --prefix=$RISCV/linux
make -j $(nproc)
make install
```

可以在`$RISCV/newlib`或`$RISCV/linux`下的`bin`里面，看到我们的`spike`程序。

### riscv-isa-sim（可选）

spike需要pk，才能运行RISC-V程序。

下载源代码：

```shell
git clone https://github.com/riscv/riscv-pk.git
```

编译：

```shell
mkdir -p build/newlib build/linux
cd build

cd newlib
../../configure --prefix=$RISCV/newlib --host=$RISCV/bin/riscv64-unknown-elf
make -j $(nproc)
make install
cd ..

cd linux
../../configure --prefix=$RISCV/linux --host=$RISCV/bin/riscv64-unknown-linux-gnu
make -j $(nproc)
make install
cd ..
```

### llvm（可选）

这是另外一个编译器，clang便是llvm中的子项目。

首先下载源代码：

```shell
git clone https://github.com/llvm/llvm-project.git
```

然后编译源代码：

```shell
cd llvm-project

mkdir build
cd build
cmake -DLLVM_TARGETS_TO_BUILD="X86;RISCV" -DLLVM_ENABLE_PROJECTS="clang;llvm" -DCMAKE_INSTALL_PREFIX=$RISCV/llvm ../llvm
make -j $(nproc)
```

我们可以在`$RISCV/llvm/bin`中找到`clang`，这个便是我们的编译器了。

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

## 例程2：add

这里我们将c和汇编混合编译。使用构造好的Makefile文件，并且使用newlib。需要注意的是，将其中RISCV的值更新为你自己的路径。

### Makefile

```makefile
RISCV=~/RISCV
RISCV_TYPE=riscv64-unknown-elf
CXX=$(RISCV)/bin/$(RISCV_TYPE)-g++
TARGET=add.out
SRC_CPP=$(wildcard *.cpp)
SRC_ASM=$(wildcard *.s)
OBJ_CPP=$(SRC_CPP:cpp=o)
OBJ_ASM=$(SRC_ASM:s=o)
OBJS=$(OBJ_CPP) $(OBJ_ASM)
CXXFLAGS=-std=c++2a

all: depends run

run: $(TARGET)
	@echo ---------------spike----------------
	@$(RISCV)/bin/spike $(RISCV)/$(RISCV_TYPE)/bin/pk $(TARGET)
	@echo ---------------qemu-----------------
	@$(RISCV)/qemu/bin/qemu-riscv64 $(TARGET)

depends: $(SRC_CPP)
	@$(CXX) -MM $(SRC_CPP) > cc.depend

$(OBJ_ASM): $(SRC_ASM)
	$(CXX) $(CXXFLAGS) $(SRC_ASM) -c

$(TARGET): $(OBJS)
	$(CXX) $(CXXFLAGS) $(OBJS) -o $(TARGET)

-include cc.depend


.PHONY: clean
clean:
	rm -rf $(TARGET) $(OBJS) cc.depend

```

### main.c

```c
#include <iostream>

extern "C" {
int add(int, int);
}

int main() {
  std::cout.setf(std::ios_base::showbase);
  std::cout << std::hex << add(0x10, 0x01) << std::endl;
}

```

### add.s

```assembly
.section .text
.global add
.type add, @function

add:
  add a0, a0, a1;
  ret

```

### make

我们使用`make`命令进行编译，然后便看到最后有：

```
---------------spike----------------
bbl loader
0x11
---------------qemu-----------------
0x11
```

## RISC-V版本

RISC-V是一个开源指令集架构（ISA），这是当前的版本信息。其中G=IMAFD，**粗体**为已经批准的。

| Base      | Version |
| --------- | ------- |
| RVWMO     | 2.0     |
| **RV32I** | 2.1     |
| **RV64I** | 2.1     |
| RV32E     | 1.9     |
| RV128I    | 1.7     |

| Extension    | Version |
| ------------ | ------- |
| **M**        | 2.0     |
| **A**        | 2.1     |
| **F**        | 2.2     |
| **D**        | 2.2     |
| **Q**        | 2.2     |
| C            | 2.0     |
| Counters     | 2.0     |
| L            | 0.0     |
| B            | 0.0     |
| J            | 0.0     |
| T            | 0.0     |
| P            | 0.2     |
| V            | 0.7     |
| **Zicsr**    | 2.0     |
| **Zifencei** | 2.0     |
| Zam          | 0.1     |
| Ztso         | 0.1     |

## RISC-V寄存器

| Register | ABI Name | Description                        |
| -------- | -------- | ---------------------------------- |
| x0       | zero     | Hard-wired zero                    |
| x1       | ra       | Return address                     |
| x2       | sp       | Stack pointer                      |
| x3       | gp       | Global pointer                     |
| x4       | tp       | Thread pointer                     |
| x5-7     | t0-2     | Temporaries                        |
| x8       | s0/fp    | Saved register / Frame pointer     |
| x9       | s1       | Saved register                     |
| x10-11   | a0-1     | Function arguments / Return values |
| x12-17   | a2-7     | Function arguments                 |
| x18-27   | s2-11    | Saved registers                    |
| x28-31   | t3-6     | Temporaries                        |

| Register | ABI Name | Description                  |
| -------- | -------- | ---------------------------- |
| f0-7     | ft0-7    | FP temporaries               |
| f8-9     | fs0-1    | FP saved registers           |
| f10-11   | fa0-1    | FP arguments / return values |
| f12-17   | fa2-7    | FP arguments                 |
| f18-27   | fs2-11   | FP saved registers           |
| f28-31   | ft8-11   | FP temporaries               |

## 指令类型

opcode：

rd：destination register

funct3：

rs1：source register 1

rs2：

funct7：

imm：immediate

| R    | 31 - 25 | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 | 6 - 0  |
| :--- | :-----: | :-----: | :-----: | :-----: | :----: | :----: |
|      | funct7  |   rs2   |   rs1   | funct3  |   rd   | opcode |

| I    |   31 - 20   | 19 - 15 | 14 - 12 | 11 - 7 | 6 - 0  |
| :--- | :---------: | :-----: | :-----: | :----: | :----: |
|      | imm12[11:0] |   rs1   | funct3  |   rd   | opcode |

| S    |  31 - 25  | 24 - 20 | 19 - 15 | 14 - 12 |  11 - 7  | 6 - 0  |
| :--- | :-------: | :-----: | :-----: | :-----: | :------: | :----: |
|      | imm[11:5] |   rs2   |   rs1   | funct3  | imm[4:0] | opcode |

## add

### add

全称：add

语法：`add rd, rs1, rs2`

操作：rd ← rs1 + rs2

| R    | 31 - 25 | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 0000000 |   rs2   |   rs1   |   000   |   rd   | 0110011 |

### addi

全称：add immediate

语法：`addi rd, rs1, imm12`

| I    |   31 - 20   | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :---------: | :-----: | :-----: | :----: | :-----: |
|      | imm12[11:0] |   rs1   |   000   |   rd   | 0010011 |

### addw

语法：`addw rd, rs1, imm12`

| R    | 31 - 25 | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 0000000 |   rs2   |   rs1   |   000   |   rd   | 0111011 |

### addiw

语法：`addiw rd, rs1, imm12`

| I    |   31 - 20   | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :---------: | :-----: | :-----: | :----: | :-----: |
|      | imm12[11:0] |   rs1   |   000   |   rd   | 0011011 |

### fadd.s

语法：`fadd.s rd, rs1, rs2`

操作：rd ← rs1 + rs2

| R    | 31 - 25 | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 0000000 |   rs2   |   rs1   |   rm    |   rd   | 1010011 |

### fadd.d

语法：`fadd.d rd, rs1, rs2`

操作：rd ← rs1 + rs2

| R    | 31 - 25 | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 0000001 |   rs2   |   rs1   |   rm    |   rd   | 1010011 |

### vadd.vv

语法：`vadd.vv vd, vs2, vs1, vm`

|      | 31 - 26 |  25  | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :--: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 000000  |  vm  |   vs2   |   vs1   |   000   |   vd   | 1010111 |

### vadd.vx

语法：`vadd.vx vd, vs2, rs1, vm`

|      | 31 - 26 |  25  | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :--: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 000000  |  vm  |   vs2   |   rs1   |   100   |   vd   | 1010111 |

### vfadd.vv

语法：`vfadd.vv vd, vs2, rs1, vm`

|      | 31 - 26 |  25  | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :--: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 000000  |  vm  |   vs2   |   vs1   |   001   |   vd   | 1010111 |

### vfadd.vf

语法：`vfadd.vf vd, vs2, rs1, vm`

|      | 31 - 26 |  25  | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :--: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 000000  |  vm  |   vs2   |   rs1   |   101   |   vd   | 1010111 |

### vsadd.vv

语法：`vsadd.vv vd, vs2, vs1, vm`

|      | 31 - 26 |  25  | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :--: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 100001  |  vm  |   vs2   |   vs1   |   000   |   vd   | 1010111 |

### vsadd.vx

语法：`vsadd.vx vd, vs2, rs1, vm`

|      | 31 - 26 |  25  | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :--: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 100001  |  vm  |   vs2   |   rs1   |   100   |   vd   | 1010111 |

### vsadd.vi

语法：`vsadd.vi vd, vs2, imm, vm`

|      | 31 - 26 |  25  | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :--: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 100001  |  vm  |   vs2   |   imm   |   011   |   vd   | 1010111 |

### vsaddu.vv

### vsaddu.vx

### vsaddu.vi

## aadd

## sub

### sub

全称：subtract

语法：`sub rd, rs1, rs2`

操作：rd ← rs1 - rs2

| R    | 31 - 25 | 24 - 20 | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :-----: | :-----: | :-----: | :-----: | :----: | :-----: |
|      | 0100000 |   rs2   |   rs1   |   000   |   rd   | 0110011 |

## load

### ld

语法：`ld rd, imm12(rs1)`

| I    |   31 - 20   | 19 - 15 | 14 - 12 | 11 - 7 |  6 - 0  |
| ---- | :---------: | :-----: | :-----: | :----: | :-----: |
|      | imm12[11:0] |   rs1   |   011   |   rd   | 0000011 |

## store

### sd

语法：`sd rs2, imm12(rs1)`

| S    |   31 - 25   | 24 - 20 | 19 - 15 | 14 - 12 |   11 - 7   |  6 - 0  |
| ---- | :---------: | :-----: | :-----: | :-----: | :--------: | :-----: |
|      | imm12[11:5] |   rs2   |   rs1   |   011   | imm12[4:0] | 0100011 |

## logical

### sll

全称：shift left logical

### slli

全称：shift left logical immediate

### srl

全称：shift right logical

### srli

全称：shift right logical immediate

### sra

全称：shift arithmetic logical

### srai

全称：shift arithmetic logical immediate

### and

全称：add

### andi

### or

### ori

### xor

全称：excursive or

### xori

全称：excursive or immediate

### lui

全称：load upper immediate

### auipc

全称：add upper immediate to pc

### slt

全称：set less than

### slti

全称：set less than immediate

