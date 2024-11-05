# BCSD0-START

## Part0-什么是BCSD

二进制代码相似性检测（BCSD）用于在缺乏源代码时查找漏洞和抄袭，通过动态和静态分析方法进行。动态分析关注运行时的函数特征，而静态分析则从汇编代码中提取特征，但可能产生误报。跨平台BCSD利用不同架构间的语意相似性，而AST提供语义信息，帮助识别不同体系中相似函数



## Part1-现有方法及针对的问题

<figure><img src="../.gitbook/assets/方法综述.jpg" alt=""><figcaption></figcaption></figure>

* 跨编译器
* 跨优化
* 跨架构 **跨平台（本次课题要关注的核心点）**
* 抗混淆



### 跨架构二进制代码区别

跨架构的二进制代码差异来自于不同的硬件架构和指令集。这些差异直接影响到编译出的机器码，进而影响到程序的运行时表现。以下是一些主要的区别：

#### 1. **指令集架构（ISA）** 每种处理器架构使用不同的指令集，这决定了该架构上可用的机器指令类型和格式。例如：

* **x86 与 x86-64**：这两种架构属于同一家族，但 x86-64 支持 64 位运算，而 x86 仅支持 32 位运算。x86-64 还增加了一些新的寄存器和指令，例如在地址计算上有所不同。
* **ARM 与 ARM64**：ARM32 和 ARM64（也称为 AArch64）都有显著不同的指令集。ARM64 支持更多的寄存器和 64 位操作，而 ARM32 则主要是 32 位操作。
* **MIPS**：MIPS 架构也有自己的指令集，与 x86 或 ARM 完全不同，其中包括 RISC（简化指令集计算机）设计原则。

#### 2. **寄存器集** 不同架构的寄存器数量和用途会有所不同。例如：

* **x86**：有特定用途的寄存器，如 EAX、EBX、ECX、EDX 等等，每个寄存器有其特殊的用途（如累加器、计数器）。
* **x86-64**：在 x86 的基础上增加了更多的寄存器（如 R8-R15）且寄存器的宽度从 32 位扩展到 64 位。
* **ARM**：有 16 个通用寄存器（R0–R15），其中一些是专门的用途寄存器（如 R15 是程序计数器）。
* **ARM64**：扩展到 31 个通用寄存器（X0–X30）。

#### 3. **字节序（Endianness）** 不同的架构可能使用不同的字节序，这影响到数据在内存中的存储顺序。

* **大端序（Big Endian）**：高字节在前，低字节在后。例如，MIPS 默认使用大端序。
* **小端序（Little Endian）**：低字节在前，高字节在后。例如，x86 和 x86-64 架构使用小端序。
* **双字节序**：某些架构（如 ARM 和 PowerPC）支持双字节序，可以在大端序和小端序之间切换。

#### 4. **调用约定（Calling Conventions）** 不同架构有不同的调用约定，这影响函数参数的传递方式、返回值的处理、以及如何管理栈和寄存器。

* **x86**：通常使用 cdecl、stdcall 或 fastcall 等调用约定，每种约定对函数参数和栈管理有不同规定。
* **x86-64**：大多数系统使用统一的 SysV ABI 调用约定，调用者保存寄存器和被调用者保存寄存器的定义更加明确。
* **ARM**：有自己的 ARM EABI（Embedded Application Binary Interface），规定了参数传递和栈布局。

#### 5. **系统调用（Syscalls）及 ABI（Application Binary Interface）** 不同架构和操作系统有不同的系统调用接口和 ABI，这影响到编译输出的二进制代码与操作系统内核的交互方式。

* **Linux x86 与 x86-64**：虽然相似，但 x86-64 使用不同的系统调用号，并且参数传递方式不同（x86 遵循寄存器和栈混合传递，而 x86-64 参数主要通过寄存器传递）。
* **ARM 的 EABI**：定义了类似的系统调用接口，但参数和调用方式有显著不同。



### 实际的例子

以相同的fun函数为例：

```c
int fun(int x) {
    return x * 2;
}
```

x86 架构（32 位）下：

```armasm
fun:
    push    ebp         ; 保存基址指针
    mov     ebp, esp    ; 设置新的基址指针
    mov     eax, [ebp+8]; 将参数 x 载入 eax 寄存器
    add     eax, eax    ; eax = eax + eax，计算 2 * x
    pop     ebp         ; 恢复基址指针
    ret                 ; 返回
```

#### x86-64 架构（64 位）：

```armasm
fun:
    mov     eax, edi    ; 将参数 x 载入 eax（参数在64位系统中通过寄存器传递）
    add     eax, eax    ; eax = eax + eax，计算 2 * x
    ret                 ; 返回
```

arm架构：

```armasm
fun:
    push     {lr}       ; 保存链接寄存器
    add      r0, r0, r0 ; r0 = r0 + r0，计算 2 * x（参数在 r0 寄存器中）
    pop      {lr}       ; 恢复链接寄存器
    bx       lr         ; 分支返回调用者
```

MIPS架构：

```
fun:
    sll     $v0, $a0, 1 ; 将参数 x 左移一个位置，等同于乘以 2
    jr      $ra         ; 返回调用者
    nop                 ; 空操作，确保流水线正确性
```