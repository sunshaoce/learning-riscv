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
