

## RISC-V寄存器

32位，长度。

| 寄存器 | ABI 名称 | 描述                             |
| ------ | -------- | ------------------------------ |
| x0     | zero     | Hard-wired zero                |
| x1     | ra       | Return address                 |
| x2     | sp       | Stack pointer                  |
| x3     | gp       | Global pointer                 |
| x4     | tp       | Thread pointer                 |
| x5-7   | t0-2     | Temporaries                    |
| x8     | s0/fp    | Saved register / Frame pointer |
| x9     | s1       | Saved register                 |
| x10-11 | a0-1     | 函数参数 / 返回值              |
| x12-17 | a2-7     | 函数参数                       |
| x18-27 | s2-11    | Saved registers                |
| x28-31 | t3-6     | 临时                           |

| Register | ABI Name | Description                  |
| -------- | -------- | ---------------------------- |
| f0-7     | ft0-7    | FP temporaries               |
| f8-9     | fs0-1    | FP saved registers           |
| f10-11   | fa0-1    | FP arguments / return values |
| f12-17   | fa2-7    | FP arguments                 |
| f18-27   | fs2-11   | FP saved registers           |
| f28-31   | ft8-11   | FP temporaries               |
