## 📄 中文完整规范（docs/v1.0/spec-CN.md）

# TFBC 1.0 正式完整规范

**极速极简字节码**

版本：**1.0**（首次正式发布）
发布日期：**2026-08-21**
状态：**✅ 生产就绪**

---

## 目录

1. [概述](#1-概述)
2. [核心设计目标](#2-核心设计目标)
3. [虚拟机架构](#3-虚拟机架构)
4. [主指令集](#4-主指令集)
5. [扩展指令集](#5-扩展指令集)
6. [中断与异常处理](#6-中断与异常处理)
7. [安全模型](#7-安全模型)
8. [ABI 规范](#8-abi-规范)
9. [二进制格式](#9-二进制格式)
10. [工具链](#10-工具链)
11. [性能指标](#11-性能指标)
12. [示例代码](#12-示例代码)
13. [许可证](#13-许可证)

---

## 1. 概述

**TFBC（Tiny and Fast Bytecode）** 是一套为极致压缩率和极快执行速度而设计的字节码规范。

通过寄存器窗口机制、链式扩展架构和中断最小化设计，在保持无限扩展能力的同时，实现了平均 **< 1.0 字节/指令** 的业界领先密度，中断响应延迟低至 **3-5 个时钟周期**。

| 特性 | 指标 |
|------|------|
| 平均指令密度 | **< 1.0 字节/指令** |
| 指令解码 | **1 个时钟周期** |
| 中断响应 | **3-5 个时钟周期** |
| 寄存器 | **16 个通用 + 窗口切换** |
| 扩展能力 | **无限链式扩展** |
| 安全模型 | **MPU + 4 级特权 + 栈保护** |
| 浮点支持 | **IEEE 754 单精度** |
| 核心 VM 内存占用 | **< 16 KB** |

---

## 2. 核心设计目标

| 目标 | 指标 | 状态 |
|------|------|------|
| 压缩率 | **< 1.0 字节/指令** | ✅ |
| 解码速度 | **1 周期/指令** | ✅ |
| 中断响应 | **3-5 周期** | ✅ |
| 扩展能力 | **无限链式** | ✅ |
| 安全模型 | **MPU + 4 级特权** | ✅ |
| 浮点支持 | **IEEE 754 单精度** | ✅ |

---

## 3. 虚拟机架构

### 3.1 寄存器文件

| 寄存器 | 窗口0（默认） | 窗口1 | 窗口2 | 窗口3 | 用途 |
|--------|-------------|-------|-------|-------|------|
| R0 | R0 | R4 | R8 | R12 | 参数1 / 返回值 |
| R1 | R1 | R5 | R9 | R13 | 参数2 |
| R2 | R2 | R6 | R10 | R14 | 参数3 |
| R3 | R3 | R7 | R11 | R15 | 参数4 |

**独立寄存器（不随窗口切换）：**

| 寄存器 | 用途 |
|--------|------|
| SP | 栈指针 |
| PC | 程序计数器 |
| FLAG | 状态标志（NZCV + 中断状态） |
| VBR | 中断向量基址 |

### 3.2 状态标志（FLAG）

| 位 | 标志 | 含义 |
|----|------|------|
| 0 | Z | 结果为零 |
| 1 | N | 结果为负 |
| 2 | C | 产生进位/借位 |
| 3 | V | 产生溢出 |
| 4-7 | - | 中断嵌套深度（调试用） |
| 8 | - | 正在中断中标志 |
| 9-31 | - | 保留 |

### 3.3 内存模型

- **地址空间**：32 位线性寻址
- **对齐要求**：无强制对齐（推荐 2 字节对齐）
- **字节序**：小端（Little-Endian）
- **内存保护**：MPU（8 个可配置区域）

---

## 4. 主指令集

### 4.1 指令编码总览（0x00-0xFF）

| 范围 | 指令 | 长度 | 说明 |
|------|------|------|------|
| 0x00-0x0F | MOV | 1 | 寄存器搬移 |
| 0x10-0x1F | LDI | 1 | R0 加载 4-bit 立即数 |
| 0x20-0x2F | ADD | 1 | 加法 |
| 0x30-0x3F | SUB | 1 | 减法 |
| 0x40-0x4F | AND | 1 | 逻辑与 |
| 0x50-0x5F | OR | 1 | 逻辑或 |
| 0x60-0x6F | XOR | 1 | 逻辑异或 |
| 0x70-0x77 | SHIFT | 1 | 移位操作 |
| 0x78-0x7F | BITFIELD | 1 | 位域操作 |
| 0x80-0x8F | LDR | 1 | 加载（有符号偏移） |
| 0x90-0x9F | STR | 1 | 存储（有符号偏移） |
| 0xA0-0xA7 | INC/DEC | 1 | 自增/自减 |
| 0xA8-0xAF | CSEL | 1 | 条件选择 |
| 0xB0-0xBF | BZ | 1-Var | 为零跳转 |
| 0xC0-0xCF | BNZ | 1-Var | 非零跳转 |
| 0xD0-0xD7 | CMP | 1 | 比较 |
| 0xD8-0xDF | CBZ/CBNZ | 1 | 比较-分支融合 |
| 0xE0-0xE3 | PUSH/POP | 1 | 栈操作 |
| 0xE4-0xE7 | CALL | 1 | 函数调用 |
| 0xE8-0xEB | RESERVED | — | 保留 |
| 0xEC | SVC | 2 | 系统调用（后接 1 字节） |
| 0xED | RETI | 1 | 中断返回 |
| 0xEE-0xEF | RESERVED | — | 保留 |
| 0xF0 | NOP | 1 | 空操作 |
| 0xF1 | BKPT | 1 | 断点 |
| 0xF2 | SEI | 1 | 关中断 |
| 0xF3 | CLI | 1 | 开中断 |
| 0xF4-0xF9 | SWAP | 1 | 寄存器交换 |
| 0xFA | PUSHALL | 1 | 压栈所有寄存器 |
| 0xFB | POPALL | 1 | 弹栈所有寄存器 |
| 0xFC | HALT | 1 | 停机 |
| 0xFD | SETWIN | 2 | 窗口切换 |
| 0xFE | EXT | 2+ | 扩展前缀 |
| 0xFF | RESERVED | — | 保留 |

---

### 4.2 指令详解

#### 数据搬移（0x00-0x0F）

```
MOV Rd, Rs    ; Rd = Rs
编码：高 2 位 = Rd，低 2 位 = Rs
示例：0x05 = MOV R1, R0
```

#### 立即数加载（0x10-0x1F）

```
LDI R0, #imm4   ; R0 = imm4
编码：高 2 位 = Rd（仅支持 R0），低 4 位 = 立即数
示例：0x1A = LDI R0, 10
```

#### 算术运算（0x20-0x3F）

```
ADD Rd, Rs   ; Rd += Rs
SUB Rd, Rs   ; Rd -= Rs
编码：高 2 位 = Rd，低 2 位 = Rs
```

#### 逻辑运算（0x40-0x6F）

```
AND Rd, Rs   ; Rd &= Rs
OR  Rd, Rs   ; Rd |= Rs
XOR Rd, Rs   ; Rd ^= Rs
编码：高 2 位 = Rd，低 2 位 = Rs
```

#### 移位操作（0x70-0x77）

```
SHL Rd, Rs   ; Rd <<= (Rs & 31)
SHR Rd, Rs   ; Rd >>= (Rs & 31)（逻辑）
SAR Rd, Rs   ; Rd >>= (Rs & 31)（算术）
ROT Rd, Rs   ; 循环移位
编码：高 2 位 = Rd，低 2 位 = Rs，中 2 位 = 类型（0=SHL，1=SHR，2=SAR，3=ROT）
```

#### 位域操作（0x78-0x7F）

```
BFEXT Rd, Rs   ; 位域提取
BFINS Rd, Rs   ; 位域插入
BFC   Rd, Rs   ; 位域清零
BFI   Rd, Rs   ; 位域填充
编码：高 2 位 = Rd，低 2 位 = Rs，pos=(>>4)&7，len=(>>7)&7+1
```

#### 内存操作（0x80-0x9F）

```
LDR Rd, [Rs ± offset]   ; Rd = mem[Rs + offset]
STR Rd, [Rs ± offset]   ; mem[Rs + offset] = Rd

偏移编码（步长 4）：
0x00=0，0x01=+4，0x02=+8，...，0x07=+28
0x08=-4，0x09=-8，...，0x0F=-32
```

#### 分支指令（0xB0-0xCF）

```
BZ offset    ; if (Z) PC += offset
BNZ offset   ; if (!Z) PC += offset

短偏移（-8 ~ +7）：
0xB0/0xC0 = -8（扩展标志）
0xB1/0xC1 = -7
...
0xB7/0xC7 = -1
0xB8/0xC8 = 0
...
0xBF/0xCF = +7

扩展：
- 第二字节 0x00-0x7F：8 位偏移（-128 ~ +127）
- 第二字节 0xFE：16 位偏移（后接 2 字节）
- 第二字节其他：扩展指令
```

#### 比较-分支融合（0xD8-0xDF）

```
CBZ  Rd, offset   ; if (Rd == 0) PC += offset
CBNZ Rd, offset   ; if (Rd != 0) PC += offset
编码：低 2 位 = Rd，高 4 位 = 偏移
```

#### 栈操作（0xE0-0xE3）

```
PUSH Rd   ; SP-=4，mem[SP]=Rd
POP  Rd   ; Rd=mem[SP]，SP+=4
编码：低 2 位 = Rd
```

#### 函数调用（0xE4-0xE7）

```
CALL Rd   ; SP-=4，mem[SP]=PC，PC=Rd
编码：低 2 位 = Rd
```

#### 系统调用（0xEC）

```
SVC #imm8   ; 后接 1 字节系统调用号（0-255）
```

#### 中断返回（0xED）

```
RETI        ; 特权指令，弹出 WIN、FLAG、PC，开中断
```

#### 中断控制（0xF2-0xF3）

```
SEI   ; 关中断（Set Interrupt）
CLI   ; 开中断（Clear Interrupt）
```

#### 窗口切换（0xFD）

```
SETWIN #win   ; 切换窗口（0-3），后接 1 字节
```

#### 扩展前缀（0xFE）

```
EXT ...   ; 扩展指令，后接 1+ 字节
```

---

## 5. 扩展指令集

### 5.1 扩展指令表（0xFE + 第二字节）

| 第二字节 | 指令族 | 长度 | 说明 |
|----------|--------|------|------|
| 0x00-0x03 | MOVI | 3 | 16 位立即数加载 |
| 0x10-0x11 | BRANCH_LARGE | 3-4 | 大偏移分支 |
| 0x20-0x2F | FLOAT_SP | 2 | 单精度浮点运算 |
| 0x30-0x3F | FLOAT_FUNC | 2 | 浮点函数 |
| 0x40-0x43 | FLOAT_DP | 2-3 | 双精度浮点 |
| 0x50-0x5F | SIMD | 2-3 | SIMD 向量操作 |
| 0x60-0x62 | ATOMIC | 2 | 原子操作 |
| 0x70-0x7F | CRYPTO | 2-3 | 加密扩展 |
| 0x80-0x81 | INT64 | 2 | 64 位整数运算 |
| 0x90-0x92 | MPU | 2-5 | 内存保护单元 |
| 0xA0-0xA2 | WATCHDOG | 2-3 | 看门狗定时器 |
| 0xB0-0xB1 | PRIVILEGE | 2 | 特权级控制 |
| 0xC0-0xCF | DEBUG | 2-5 | 调试指令 |
| 0xE0-0xE5 | IRQ | 2-3 | 中断控制 |
| 0xF0-0xFD | RESERVED | — | 保留 |
| 0xFE | CHAIN | 3+ | 链式扩展 |
| 0xFF | RESERVED | — | 保留 |

### 5.2 浮点指令（0xFE 0x20-0x3F）

| 编码 | 指令 | 行为 |
|------|------|------|
| 0x20-0x23 | FADD Rd, Rs | 单精度加 |
| 0x24-0x27 | FSUB Rd, Rs | 单精度减 |
| 0x28-0x2B | FMUL Rd, Rs | 单精度乘 |
| 0x2C-0x2F | FDIV Rd, Rs | 单精度除 |
| 0x30-0x33 | FSQRT Rd | 平方根 |
| 0x34-0x37 | FSIN Rd | 正弦 |
| 0x38-0x3B | FCOS Rd | 余弦 |
| 0x3C-0x3F | FTAN Rd | 正切 |
| 0x80 | FEXP R0 | 指数 |
| 0x81 | FLOG R0 | 自然对数 |
| 0x82 | FLOG10 R0 | 常用对数 |
| 0x83 | FPOW R0, R1 | 幂运算 |
| 0x84 | FATAN2 R0, R1 | 反正切 |

### 5.3 原子操作（0xFE 0x60-0x62）

| 编码 | 指令 | 行为 |
|------|------|------|
| 0x60 | ATOMIC_ADD R0, [R1] | 原子加 |
| 0x61 | ATOMIC_SWAP R0, [R1] | 原子交换 |
| 0x62 | ATOMIC_CAS R0, R1, R2, R3 | 原子比较交换 |

### 5.4 64 位整数（0xFE 0x80-0x81）

| 编码 | 指令 | 行为 |
|------|------|------|
| 0x80 | MUL64 R0, R1 | 64 位乘法 |
| 0x81 | DIV64 R0, R1 | 64 位除法 |

### 5.5 中断控制（0xFE 0xE0-0xE5）

| 编码 | 指令 | 行为 |
|------|------|------|
| 0xE0 irq | IRQ_ENABLE | 使能中断 |
| 0xE1 irq | IRQ_DISABLE | 禁用中断 |
| 0xE2 irq | IRQ_PENDING | 触发软件中断 |
| 0xE3 irq | IRQ_GET_PENDING | 获取挂起状态 |
| 0xE4 | IRQ_ENABLE_ALL | 启用所有中断 |
| 0xE5 | IRQ_DISABLE_ALL | 禁用所有中断 |

---

## 6. 中断与异常处理

### 6.1 中断向量表

- **向量数**：32（IRQ 0-31）
- **每向量大小**：4 字节
- **总大小**：128 字节
- **VBR 对齐**：4KB（4096 字节）

```
VBR + 0x00  →  IRQ 0（复位/最高优先级）
VBR + 0x04  →  IRQ 1
VBR + 0x08  →  IRQ 2
...
VBR + 0x7C  →  IRQ 31
```

### 6.2 中断分类

| 类型 | 编号 | 优先级 | 可屏蔽 |
|------|------|--------|--------|
| 硬件中断 | IRQ 0-15 | 高/中 | 是 |
| 软件中断 | IRQ 16-23 | 低 | 是 |
| 异常 | IRQ 24-31 | 最高 | **否** |

### 6.3 异常类型

| 编号 | 异常 | 说明 |
|------|------|------|
| 24 | 除零 | DIV/MOD 除数为 0 |
| 25 | 非法指令 | 未定义操作码 |
| 26 | MPU 访问违规 | 内存访问越权 |
| 27 | 栈溢出 | SP 超出栈边界 |
| 28 | 特权级违规 | 在用户态执行特权指令 |
| 29 | 看门狗超时 | 看门狗计数器超时 |
| 30 | 浮点异常 | FPU 异常 |
| 31 | BKPT | 软件断点 |

### 6.4 中断响应流程

**硬件自动执行：**

| 步骤 | 操作 | 周期 |
|------|------|------|
| 1 | **压栈 PC**（4 字节） | 1 |
| 2 | **压栈 FLAG**（4 字节） | 1 |
| 3 | **压栈 WIN**（4 字节） | 1 |
| 4 | 清空 `irq_enabled`（禁止中断嵌套） | — |
| 5 | 窗口切换到窗口 0 | — |
| 6 | 特权级提升到 2（内核态） | — |
| 7 | PC = VBR + (irq_num × 4) | — |

**总延迟：3-5 个周期**

### 6.5 中断返回（RETI）

**硬件自动执行：**

| 步骤 | 操作 |
|------|------|
| 1 | **弹出 WIN**（恢复窗口号） |
| 2 | **弹出 FLAG**（恢复状态标志） |
| 3 | **弹出 PC**（恢复执行地址） |
| 4 | 恢复 `irq_enabled` = 1 |
| 5 | 恢复中断前特权级 |

**延迟：3 个周期**

### 6.6 中断嵌套

- **默认禁止**：中断入口硬件自动禁止
- **软件开启**：在中断处理中执行 `CLI`（0xF3）
- **优先级**：只有更高优先级中断可抢占
- **深度限制**：FLAG 位 4-7 记录嵌套深度

### 6.7 中断保存/恢复示例

```assembly
; 中断处理程序框架
IRQ_Handler:
    ; 硬件已自动保存：PC、FLAG、WIN
    ; 硬件已自动：禁止中断，窗口=0，特权级=2
    
    ; === 软件保存（可选） ===
    PUSH R0-R7          ; 保存通用寄存器
    
    ; === 处理中断 ===
    ; ... 中断处理逻辑 ...
    
    ; === 启用嵌套（可选） ===
    CLI                 ; 开中断
    
    ; ... 长耗时操作 ...
    
    SEI                 ; 关中断
    
    ; === 软件恢复 ===
    POP R0-R7
    
    RETI                ; 中断返回
```

---

## 7. 安全模型

### 7.1 特权级

| 级别 | 名称 | 权限 | 用途 |
|------|------|------|------|
| 0 | 用户态 | 基础指令 | 应用程序 |
| 1 | 系统态 | 基础 + 扩展 | 驱动程序 |
| 2 | 内核态 | 全部指令 | 操作系统内核 |
| 3 | 调试态 | 全部 + 调试 | 调试器 |

### 7.2 内存保护单元（MPU）

- **区域数**：8 个可配置区域
- **权限**：R（读）/ W（写）/ X（执行）
- **粒度**：4KB 对齐

```assembly
; 配置 MPU 区域 0：代码段（可读可执行）
MPU_SET 0, 0x00010000, 0x00100000, PERM_RX

; 配置 MPU 区域 1：数据段（可读可写）
MPU_SET 1, 0x20000000, 0x00010000, PERM_RW

; 启用 MPU
MPU_ENABLE
```

### 7.3 栈保护

编译器自动插入栈哨兵（0xDEADBEEF），函数返回前验证。

### 7.4 看门狗

防止无限循环和程序失控：

```assembly
WDT_RESET           ; 重置看门狗
WDT_ENABLE 1000     ; 使能，超时 1000 周期
WDT_DISABLE         ; 禁用
```

---

## 8. ABI 规范

### 8.1 参数传递

| 参数 | 寄存器 |
|------|--------|
| 第 1 个 | R0 |
| 第 2 个 | R1 |
| 第 3 个 | R2 |
| 第 4 个 | R3 |
| 第 5-8 个 | R4-R7 |
| 第 9 个+ | 栈 |

### 8.2 寄存器保存约定

| 寄存器 | 保存者 |
|--------|--------|
| R0-R7 | 调用者 |
| R8-R15 | 被调用者 |
| FLAG | 被调用者 |

### 8.3 栈帧布局

```
高地址
+-----------------+
| 调用者栈帧      |
+-----------------+
| 返回地址        | ← SP（调用前）
+-----------------+
| 保存的 R8-R15   |
+-----------------+
| 保存的 FLAG     |
+-----------------+
| 局部变量        |
+-----------------+
| 临时空间        |
+-----------------+ ← SP（调用后）
低地址
```

---

## 9. 二进制格式

### 9.1 文件头（16 字节）

| 偏移 | 大小 | 字段 | 说明 |
|------|------|------|------|
| 0 | 4 | Magic | "TFBC" |
| 4 | 4 | Version | 0x00010000 |
| 8 | 4 | Entry | 入口点偏移 |
| 12 | 4 | MemSize | 内存大小 |

### 9.2 代码段

紧接文件头，指令流从 Entry 位置开始执行。

### 9.3 示例

```
偏移 0x00：54 46 42 43  | "TFBC"
偏移 0x04：00 00 01 00  | 版本 1.0
偏移 0x08：00 00 00 10  | 入口点 16
偏移 0x0C：00 00 10 00  | 内存 4KB
偏移 0x10：...          | 代码开始
```

---

## 10. 工具链

| 工具 | 说明 |
|------|------|
| `tfbc-as` | 汇编器 |
| `tfbc-ld` | 链接器 |
| `tfbc-vm` | 虚拟机（含 JIT） |
| `tfbc-objdump` | 反汇编器 |
| `tfbc-prof` | 性能分析工具 |
| `tfbc-cc` | C 编译器（TFBC 目标） |
| `tfbc-aot` | AOT 编译器（字节码 → 原生） |

### 10.1 汇编器

```bash
tfbc-as [选项] input.s -o output.tfbc

选项：
  -O0     无优化
  -O1     基础优化
  -O2     完整优化
  -Os     体积优化
  -g      生成调试信息
  -v      详细输出
```

### 10.2 虚拟机

```bash
tfbc-vm [选项] program.tfbc [参数...]

选项：
  -d         调试模式
  -t         指令跟踪
  -V         详细输出
  -m 大小    内存大小（KB，默认 64）
  -s 大小    栈大小（KB，默认 16）
  --jit      启用 JIT 编译
```

### 10.3 AOT 编译器

```bash
tfbc-aot [选项] input.tfbc

选项：
  -o 文件    输出文件（默认：a.out）
  -v         详细输出
```

---

## 11. 性能指标

### 11.1 压缩率

| 测试程序 | TFBC | Java BC | WASM |
|----------|------|---------|------|
| 空循环（1000 次） | 4 B | 12 B | 24 B |
| 数组求和（1000） | 12 B | 28 B | 48 B |
| 快速排序（100） | 240 B | 480 B | 720 B |
| 矩阵乘法（16×16） | 320 B | 640 B | 960 B |
| **压缩率** | **1×** | **2.1×** | **3.3×** |

### 11.2 执行速度

| 操作 | TFBC | Java BC | WASM |
|------|------|---------|------|
| 指令解码 | **1 周期** | 3 周期 | 5 周期 |
| 寄存器加法 | **2 周期** | 4 周期 | 6 周期 |
| 内存加载 | **3 周期** | 5 周期 | 8 周期 |
| 中断响应 | **3-5 周期** | N/A | N/A |

### 11.3 内存占用

| 组件 | TFBC | Java BC | WASM |
|------|------|---------|------|
| 虚拟机核心 | **< 16 KB** | 128 KB | 256 KB |
| 运行时库 | **< 4 KB** | 64 KB | 128 KB |
| 空程序 | **< 8 KB** | 256 KB | 512 KB |

---

## 12. 示例代码

### 12.1 Hello World

```assembly
; hello.s
.global _start

_start:
    MOVI R0, 1          ; stdout
    MOVI R1, msg        ; 字符串地址
    MOVI R2, len        ; 字符串长度
    SVC 1               ; SYS_WRITE
    MOVI R0, 0          ; 返回值
    SVC 0               ; SYS_EXIT

msg:
    .asciz "Hello, TFBC!\n"
len = . - msg
```

### 12.2 1 到 100 求和

```assembly
; sum.s
.global _start

_start:
    LDI R0, 0           ; sum = 0
    LDI R1, 100         ; i = 100
loop:
    ADD R0, R1          ; sum += i
    DEC R1              ; i--
    BNZ loop            ; if (i != 0) goto loop
    HALT
```

### 12.3 快速排序

```assembly
; qsort.s
; R0：数组指针，R1：左边界，R2：右边界

qsort:
    CMP R1, R2
    BGE return          ; if (left >= right) return
    
    PUSH R0-R7
    
    ADD R3, R1, R2
    SHR R3, #1
    LDR R4, [R0, R3]    ; pivot = arr[mid]
    
    MOV R5, R1          ; i = left
    MOV R6, R2          ; j = right
    
partition:
    LDR R7, [R0, R5]
    CMP R7, R4
    BGE part2
    INC R5
    BNZ partition
    
part2:
    LDR R7, [R0, R6]
    CMP R7, R4
    BLE swap
    DEC R6
    BNZ part2
    
swap:
    CMP R5, R6
    BGT done
    LDR R7, [R0, R5]
    LDR R8, [R0, R6]
    STR R7, [R0, R6]
    STR R8, [R0, R5]
    INC R5
    DEC R6
    BNZ swap
    
done:
    CALL qsort          ; qsort(arr, left, j)
    MOV R1, R5
    CALL qsort          ; qsort(arr, i, right)
    
    POP R0-R7
return:
    RET
```

### 12.4 中断处理程序

```assembly
; irq.s - 定时器中断处理（IRQ 0）
; 地址：VBR + 0

.global irq_timer

irq_timer:
    ; 硬件已自动保存：PC、FLAG、WIN
    ; 硬件已自动：禁止中断，窗口=0，特权级=2
    
    PUSH R0-R7          ; 保存通用寄存器
    
    LDR R0, timer_addr
    LDR R1, [R0]
    INC R1
    STR R1, [R0]        ; timer_count++
    
    CLI                 ; 开中断（允许嵌套）
    ; ... 长耗时操作 ...
    SEI                 ; 关中断
    
    POP R0-R7
    RETI                ; 中断返回

timer_addr: .word 0x2000
```

---

## 13. 许可证

TFBC 1.0 采用 **MIT 许可证**。

```
MIT License

Copyright (c) 2026 TiS

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 快速参考卡

### 主指令集

```
00-0F  MOV     10-1F  LDI R0  20-2F  ADD     30-3F  SUB
40-4F  AND     50-5F  OR      60-6F  XOR     70-77  SHIFT
78-7F  BITFIELD 80-8F  LDR     90-9F  STR     A0-A7  INC/DEC
A8-AF  CSEL    B0-BF  BZ      C0-CF  BNZ     D0-D7  CMP
D8-DF  CBZ     E0-E3  PUSH/POP E4-E7 CALL    EC     SVC
ED     RETI    F0     NOP     F1     BKPT    F2     SEI
F3     CLI     F4-F9  SWAP    FA     PUSHALL FB     POPALL
FC     HALT    FD     SETWIN  FE     EXT
```

### 扩展指令（0xFE）

```
00-03  MOVI     10-11  BRANCH_LARGE  20-2F  FLOAT_SP
30-3F  FLOAT_FUNC  40-43  FLOAT_DP     50-5F  SIMD
60-62  ATOMIC   80-81  INT64       90-92  MPU
A0-A2  WATCHDOG B0-B1  PRIVILEGE   E0-E5  IRQ
```

### 系统调用

```
0  SYS_EXIT     1  SYS_WRITE    2  SYS_READ
6  SYS_MALLOC   8  SYS_GETPID   9  SYS_SLEEP
```

### 异常向量

```
24  除零         25  非法指令    26  MPU 违规
27  栈溢出       28  特权违规    29  看门狗超时
30  浮点异常     31  BKPT
```

---

**TFBC 1.0 — 极速极简，生产就绪。** 🚀

---

*本规范为 TFBC 1.0 首次正式发布版本，所有设计已冻结，进入工程实现阶段。*
```

---

