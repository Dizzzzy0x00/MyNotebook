---
description: assemble language 汇编语言程序设计
cover: ../.gitbook/assets/微信图片_20230910173724.png
coverY: -22
---

# Assembly Language Notebook

## 第一章 引言

gcc -S mian.c 编译可执行文件

使用UltraEdit 静态修改exe文件

查找world字符串修改为**相同字符数量**的新字符串运行

反汇编窗口 调试后开启

xdbg 动态调试源代码

数字信号 01

BCD码 ASCII码

机器语言：机器指令的集合，机器指令就是二进制编码

汇编指令是机器指令的符号化

```assembly
  MOV          AX          27
#指令操作码  目的操作数   源操作数
```

**汇编**：把汇编语言翻译成机器语言

**汇编程序**：用汇编语言编写的程序机器不能直接识别，起翻译作用

**反汇编**：把机器语言转成汇编语言的过程

使用python语言计算不同进制的运算

```python
print(bin(122)) #bin() 转成二进制
#输出：0b1111010
print(bin(122+138))
print(int('A',16)) #把字符A转为16进制，int的第二个参数对应进制
```

## 第二章 寄存器

x86 寄存器 E，32位

x64 寄存器 R，64位

**数据寄存器 AX BX CX DX** (8086/8088CPU 16位寄存器)

* AX 累加器（Add）
* BX 基址寄存器
* CX 计数寄存器 （count）
* DX 数据寄存器（data），也用于双精度运算，DX存放高字，AX存放低字

向下兼容性：16位AX可以拆分成AL（低8位） AH（高8位）两个8位寄存器使用

**地址寄存器 SI DI BP SP**

* SP 堆栈指针 堆栈区栈顶
* BP 基址指针寄存器 堆栈区基地址
* SI 源变址寄存器 存放源缓冲区偏移地址
* DI 目的变址寄存器 存放目的缓冲区的偏移地址

段寄存器 CS DS ES SS

**控制寄存器 IP FR**

* IP 指令指针寄存器 当前正在执行指令的下一条指令的所在单元的偏移地址
* FR 标志寄存器，又称程序状态字 包含9个状态标志
  * CF 进位标志 加减运算时 进位或者借位时CF = 1
  * PF 奇偶标志 低8位偶数个1时 PF= 1
  * ZF 零标志 运算结果为0 时 ZF = 1
  * OF 溢出标志 补码解释的运算溢出时(12位) OF= 1

```c++
#include<stdio.h>
#include<stdlib.h>

int main() 
{
	int a = 122; //0111 1010(bin)
    //mov         dword ptr [a],7Ah

	int b = 138;//1000 1010(bin)
    //mov         dword ptr [b],8Ah

	int c = a + b;
	mov         eax,dword ptr [a]  
    //EAX0-->7A
	add         eax,dword ptr [b]  
    //EAX 7A-->104 (0001 0000 0100)
	mov         dword ptr [c],eax
    //EFL 00000246 --> 00000212(CF = 0 ,PF = 0，ZF = 0，OF = 0)
	system("pause");
	return 0;
}
```

## 第三章 基本指令系统（8086/8088）

汇编语言的三种指令

* **汇编指令**（操作码+操作数）
* 伪指令
* 宏指令

操作数字段由 寄存器、内存单元地址、端口地址、立即数（直接把数字写到内存中）等构成

（1）双操作数指令：操作数1（目的操作数）、操作数2（源操作数），用逗号隔开；

```assembly
MOV AX, 5 ;
ADD AX, BX ;
```

（2）单操作数指令：1个操作数

```assembly
INC AX ;  #AX中的数值加一，AX既是源操作数又是目的操作数
PUSH AX ;  #将AX的数值分为两个字节压入栈，AX是源操作数
```

（3）无操作数指令（零地址指令）

```assembly
NOP ;  #空操作指令，CPU空转三个节拍
CLC ;  #进位标志CF清零
HLT ;  #停机指令，CPU不再执行任何除了中断之外的指令
```

指令属性——指令长度，指令执行时间

（1）指令长度：对应机器指令的长度（字节）

（2）执行一条指令需要的节拍数

寻址方式：指令获取操作数的方式

* 操作数在寄存器中或者作为立即数写在指令中，无需访问内存
* 操作数在内存中，CPU需要访问内存，需要操作数的内存地址

1. 寄存器寻址（寻址较快）
2. 立即数寻址，只能针对**源操作数**，因为立即数是数据本身**不能存储单元地址**
3.  存储器寻址

    （1）直接寻址方式：指令使用位移量（Disparity）作为内存单元的偏移量（EA）

    （2）寄存器间接寻址：内存操作数的偏移量由地址指针寄存器（SI、DI、SP、BP）之一获得

    （3）基址寻址 变址寻址

    （4）基址变址寻址

```assembly
MOV BX, [1000H] ;  #直接寻址方式，
#把从程序内存映射起偏移1000H个字单元（2字节，BX的长度）内容传送到BX中
MOV BX, VAR ; #直接寻址方式
#VAR是程序中的一个变量，对应内存中的字节单元或者字单元，
MOV CH, [SI] ; #寄存器间接寻址
MOV [BP], CX ; #寄存器间接寻址
MOV AX, [SI+10H] ;  #基址寻址 
MOV [BX + SI + 20H], CX ; #基址变址寻址
```

#### Lab

```c
int sum(int a, int b){
    //return a+b;
    __asm {
        mov eax, a ;
        add eax, b ;
        //函数的返回值保留在eax中
    }
}


int main(){
	int a = 122;
    int b = 138;
    int c = sum(a,b);
    printf("c = %d\n",c);
    system("pause");
    
    return 0;
}
```

将堆栈操作加入汇编

```c
int __declspec(naked) sum(int a,int b){
	//宏__declspec(naked)，编译器不负责管理堆栈
    //调用者保护机制
    __asm{
        //调用者保存机制
		push ebp  
    	mov ebp, esp 
		//计算
        mov eax, a 
    	add eax, b 
         //恢复堆栈，返回
        mov esp, ebp 
        pop ebp 
        ret
           
	}
}
```

/86指令系统按照功能分成六类

### 传送类指令

把数据从一个存储位置搬到另一个存储位置

通用数据传送 MOV PUSH POP

**MOV**数据传送类指令

```assembly
MOV DEST,SRC

#实例
MOV byte ptr [si],4 #字节传送
MOV cx,0ffh #字传送

```

操作数可以是字节、字，**但两个操作数必须一致**

两个操作数中，只允许有**一个为存储器寻址**，不能两个操作数都是存储单元

指令指针IP不能作为MOV指令的操作数

**XCHG**交换指令

```assembly
XCHG DEST,SRC
XCHG ax,bx
```

交换指令中也最多只能有一个内存操作数

**PUSH\&POP**堆栈操作指令

在8086/8088操作系统中，SP指向栈顶，堆栈中存放字数据，每次入栈时SP自动减2（栈从高地址向低地址生长）

（1）入栈PUSH，把源操作数压入堆栈，操作数必须是16位的

```assembly
PUSH SRC
POP DEST
```

（2）出栈POP，将栈顶数据取出存放在目的位置，SP自动加2（栈中数据并没有被清除）

**地址传送指令**

LEA装入**有效地址**指令

```assembly
LEA DEST,SRC
```

LEA中的**源操作数必须是内存操作数**，**目的操作数只能是16位通用寄存器**

### 算术类指令

加法指令**ADD**

两个操作数中，只允许有**一个为存储器寻址**，不能两个操作数都是存储单元

加一指令**INC** 减一指令**DEC**

减法指令**SUB**

两个操作数中，只允许有**一个为存储器寻址**，不能两个操作数都是存储单元

求相反数NEG

0减去操作数，相当于求补操作

比较指令**CMP**

```assembly
CMP DEST，SRC
```

使用减法操作影响标志位，不保留结果，如果DEST = SRC则 跳转，不执行

#### Lab

```c
#include <iostream>

int main()
{
    //int* a = (int*)malloc(8 * sizeof(int));
    int* a = NULL;
    __asm {
        push 0x20;//8 * sizeof(int) = 32
        call malloc
        add esp,4 //保持堆栈平衡
        mov [a],eax //eax存放malloc运行的返回值
    }
    printf("%x\n", a);
    printf("a[1] = %d\n", a[1]);

    //a[1] = 1;
    __asm {
        mov edi,[a]
        mov dword ptr [edi+4],1 //edi+4为a[1]的位置，一个int是4字节
    }

    printf("a[1] = %d\n", a[1]);
    int b = 0;
    printf("b = %d\n", b);

    //b = a[1];
    __asm {
        mov eax,dword ptr[edi+4]
        mov [b],eax
    }
    printf("b = %d\n", b);
    return 0;
}

```

### 位操作指令

**逻辑运算指令**

DEST存放结果，SRC值不变，不能两个操作数都是内存地址

```assembly
AND  DEST，SRC ; 按位与
OR   DEST，SRC ; 按位或
XOR  DEST，SRC ; 按位异或
NOT  DEST ;按位取反
```

与操作实现**取位操作**

或操作实现**置位操作**

**TEST指令**：测试指令，和与操作一样，但不保留运算结果，只影响符号位（ZF标志）

**移位、循环移位指令**

```assembly
SHL AL, 1 ; 逻辑左移，和算术左移完全一样
SAL AL, 1 ; 算术左移1位,相当于乘2
SHR AL, 1 ; 逻辑右移，移位后补0
SAR AL, 1 ; 算术右移1位，相当于除以2，高位补充为符号位

ROL AL, 1 ; 循环左移
ROR AL, 1 ; 循环右移

```

```c
int main(){
	int a = 0b10100011；
	int b = 0b00101010;
	
	__asm{
		mov al,byte ptr [a]
		mov ah,byte ptr [b]
		and al,ah
		//and操作以后AL改变，AH不变
	}
	return 0;
}
```

### 处理器控制指令

设置标志位的指令，控制CPU运转的指令

## 函数调用过程

首先push所有的参数入栈，再执行call语句

函数调用指令call ：push下一条指令的地址入栈（保留返回地址），jmp到调用函数地址

进入函数后，首先push EBP；mov EBP ESP（调用者保存机制，保存现场）

## 第四章 循环和条件控制语句

### 顺序结构

（1）双分支语句：IF-ELSE

（2）多分支语句：CASE

### 无条件转移指令

```assembly
JMP 目标地址

```

无条件修改（E）IP中的内容

目标地址可以是寄存器或内存地址

### 条件转移指令

​ **单条件转移指令**（单标志位转移）：通过一个标志位决定是否跳转

（无符号数）**双条件转移指令** JA(>) JAE(>=) JB JBE

（带符号数）**三条件转移指令** JG(>) JGE(>=) JL JLE

### 循环指令

LOOP指令

```assembly
LOOP 目标地址
```

CX <-- CX - 1（不影响标志位），如果CX不等于0，则继续循环

LOOPZ LOOPNZ，考虑了标志位ZF

#### Lab

```c
#include <iostream>
#include<stdio.h>

char getGrade(int grade) {
	if (grade < 60)
		return 'F';
	else if (grade == 60)
		return 'E';
	else
		return 'D';
}

char __declspec(naked) my_getGrade(int grade) {
	__asm {
		//调用者保存机制
		push ebp
		mov ebp, esp
		//函数体
		mov ebx, dword ptr[grade]
		cmp ebx,60
		jge elseif
		mov eax,'F'
		jmp funend
	elseif:
		cmp ebx, 60
		jnz elsee
		mov eax,'E'
		jmp funend
	elsee:
		mov eax,'D'
		jmp funend

		//恢复堆栈，返回
	funend:	
		mov esp, ebp
		pop ebp
		ret
	}
}
int main()
{	
	printf("80:%c\n",my_getGrade(80));
	printf("60:%c\n", my_getGrade(60));
	printf("40:%c\n", my_getGrade(40));
	return 0;
}

```

二进制可执行文件

windows平台下：PE文件（.exe 、.dll、.com等等）

文件的头两个字节是**MZ**（**4D 5A**）

![](F:%5Casm%5C2.png)

Linux平台下：ELF文件

### 基本数据类型的内存表现

int long 4字节

short 2字节

#### 大端&小端存储

0x12345678

大端： 78 56 34 12

小端：12 34 56 78（x86采用的存储方式）**高地址存低位**

#### 浮点数

单精度 float

符号位（1位）指数位（8位）尾数位（23位）——32位

指数位= 指数+127的偏移

e.g. 12.25f 内存中是：0x41440000

双精度 double

#### 指针

### 函数的三种调用方式

#### 1. cdecl（默认）

32位使用 栈 传递函数的参数，从右往左存放参数入栈。调用返回以后，需要进行栈平衡(调用者进行平衡栈)

```assembly
add esp 4
#每压入一个参数需要add 4一次
```

#### 2. stdcall

函数内部自己进行栈平衡

#### 3. fastcall

寄存器进行传参，64位程序默认为寄存器传参（前6个参数，第6个以后的参数也使用栈传参）

```c
#include<iostream>
#include<stdio.h>

int __stdcall std_Add(int a, int b) {
    return a + b;
}

int __cdecl cde_Add(int a, int b) {
    return a + b;
}

int __fastcall fast_Add(int a, int b) {
    return a + b;
}
```

### for循环的汇编实现

```c
int __declspec(naked) sumByAsm(int *input,int lengths) {

	/*int sum = 0;
	for (int i = 0; i < a; i++) {
		sum += 2;
	}
	return sum;*/

	__asm {
		push ebp
		mov ebp, esp

		mov eax , 0
		mov ecx , 0
		mov esi , [input]
	myLOOP:	
		cmp ecx ,[lengths]
		jge fun_end
		add eax ,[esi]
		add esi , 4
		inc ecx
		jmp myLOOP
		
	fun_end:
		mov esp, ebp
		pop ebp
		ret
	}
}

int __declspec(naked) sum2ByAsm(int* input, int lengths) {
	
	__asm {
		push ebp
		mov ebp, esp

		mov eax, 0
		mov ecx, lengths
		mov esi, input
		
add_input:
		add eax,[esi]
		add esi,4
		loop add_input
            //LOOP指令进行循环，cx <- cx-1, cx小于0时 会跳出循环

fun_end :
		mov esp, ebp
		pop ebp
		ret
	}
}



int main() {
	int a[4] = { 1,2,3,4 };
	printf("sumByAsm:  result = %d\n", sumByAsm(a,4));
	printf("sum2ByAsm: result = %d\n", sum2ByAsm(a, 4));
	return 0;
}
```
