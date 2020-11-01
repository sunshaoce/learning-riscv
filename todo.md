# Learning RISC-V



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
