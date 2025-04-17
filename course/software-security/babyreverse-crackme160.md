---
cover: ../../.gitbook/assets/微信图片_20250408134702.jpg
coverY: 135
---

# BabyReverse-CrackMe160

## 003-Cruehead-CrackMe3

先使用PEiD分析程序：“MASM32 / TASM32 \[Overlay]”和“TASM MASM”提示程序使用了MASM/TASM编译，并包含overlay数据，是一个32位程序，没有检测到程序是什么壳

<figure><img src="../../.gitbook/assets/993683a54d290baa4813c108f12cf47.png" alt=""><figcaption></figcaption></figure>

运行一下程序看看，没有发现输入，在窗口最上面显示了v3.0 Uncrack（这里截图没有显示全）

<figure><img src="../../.gitbook/assets/139638d71a76b5b10169508a405c9a7.png" alt=""><figcaption></figcaption></figure>

尝试直接使用x32dbg（刚刚知道是32位程序）进行分析，运行到入口点：

<figure><img src="../../.gitbook/assets/108b6b977e6b131d5da2ebbd382e13e.png" alt=""><figcaption></figcaption></figure>

查看一下主模块的函数，结合刚刚运行的界面，并且符号表中有导入很多文件操作相关的系统函数，推断大致是一个文件题目：

<figure><img src="../../.gitbook/assets/91bf093fc6dd8549ceab56d573d1e2e.png" alt=""><figcaption></figcaption></figure>

看一下字符串，看到了比较关键的两个"CRACKME3.KEY"和"Good work cracker!"，一个应该是关键的key文件名，一个是逆向成功crack以后的位置，也就是从entrypoint分析到"Good work cracker!"是需要分析的部分（可以从"Good work cracker!"往上分析或者entrypoint往下分析）

<figure><img src="../../.gitbook/assets/f44d6fd4269e96a71c7196de01a8274.png" alt=""><figcaption></figcaption></figure>

接下来开始正式逆向，从entrypoint往下看到首先程序调用[CreateFileA](windowsapi.md#id-8.-createfilea)函数，根据函数返回的EAX值（是否有CRACKME3.KEY这个文件）进行跳转（`004010 | jne cruehead-crackme-3.401043 |`），如果没有跳转到0x401043，则会继续执行后面push和call两行，输出Uncrack的结果，接下来使用[ReadFile](windowsapi.md#id-10.-readfile)读取文件内容：

<figure><img src="../../.gitbook/assets/8be0b6175f70b7d03bc3feb7492e28b.png" alt=""><figcaption></figcaption></figure>

那么先创建一个要求验证的密钥文件的并输入一些文字进行测试：`echo 12345> CRACKME3.KEY`\


<figure><img src="../../.gitbook/assets/045a193d7b9902930e4036296c41558.png" alt=""><figcaption></figcaption></figure>

然后重新在x32dbg中装载程序（在Call ReadFileA以后下个断点），运行，可以看到ReadFile将12345读取到了EBX中，注意ReadFile的参数`004010 | push cruehead-crackme-3.4021A0 |`，根据ReadFIle函数的define，第二个参数：`push    lpNumberOfBytesRead; 实际读取字节数存储地址`这个地址应该存放读取的字节数，然后call以后下一行`004010 | cmp dword ptr ds:[4021A0],12 |`将`[4021A0]` 和`0x12（18）` 进行比较，在内存中查看这个地址，是0x5（刚刚在key文件中写入的12345正好是5字节），如果`[4021A0]` 不为18，那么会跳转到0x401037：

```
00401037 | push cruehead-crackme-3.40210 | 40210E:"CrackMe v3.0 "
0040103C | call cruehead-crackme-3.4012F | 输出Uncrack的结果
```

<figure><img src="../../.gitbook/assets/e49b43867f66cddc21633b2771f53f5.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/06850707a694e390521d53430db110a.png" alt=""><figcaption></figcaption></figure>

那么我们要把Key文件里面的输入改为18字节（需要重新装载程序），成功通过字节数验证，执行后续代码：

<figure><img src="../../.gitbook/assets/13719676377e325e484a10fac3bb5bd.png" alt=""><figcaption></figcaption></figure>

然后传递地址`[0x402008] 刚刚读取的文件内容`作为参数，call函数以后对我们输入的字符串进行变换，最后结果将值存放在原地址：`"pppppppppzz~~z5678"`&#x20;

```
0040106F | push cruehead-crackme-3.402008 | 402008:"pppppppppzz~~z5678"
00401074 | call cruehead-crackme-3.401311 |
```

接下来我们分析cruehead-crackme-3.401311这个函数在干什么，下断点在0x00401074以后步进函数，结束条件：循环14次 0x4F-41 = 14，结果\[0x4020F9] `sum += input[i] ^ (0x41+i)`

```wasm
#实现对传入字符串的逐字节“解密”操作
#其核心思路是使用一个初始值 0x41 对字符串的每个字符进行异或运算

00401311  | xor ecx,ecx                      |
00401313  | xor eax,eax                      |
00401315  | mov esi,dword ptr ss:[esp+4]     | [esp+4]:"123456789012345678"
00401319  | mov bl,41                        | 41:'A' 初始值 0x41 
0040131B  | mov al,byte ptr ds:[esi]         | 逐字节，读取一个字节的值
0040131D  | xor al,bl                        | 对读取的字符进行异或运算
0040131F  | mov byte ptr ds:[esi],al         | 将异或后的结果写回原内存地址 [ESI]，覆盖原字符
00401321  | inc esi                          | 将 ESI 自增，指向下一个字符
00401322  | inc bl                           | 将 BL 自增，作为下一个字符的异或键
00401324  | add dword ptr ds:[4020F9],eax    | 将 AL 加到地址 0x4020F9 处的全局变量上
0040132A  | cmp al,0                         | 判断异或后的字符是否为 0（字符串结束标志）
0040132C  | je cruehead-crackme-3.401335     | 是则退出循环，不是就继续
0040132E  | inc cl                           | cl计数器自增
00401330  | cmp bl,4F                        | 4F:'O' 如果 BL 达到 0x4F 
00401333  | jne cruehead-crackme-3.40131B    | 退出循环
00401335  | mov dword ptr ds:[402149],ecx    | 地址 0x4020F9 处的全局变量赋值到ecx
0040133B  | ret                              | 
```

然后调用cruehead-crackme-3.40133C，参数也是402008:"pppppppppzz\~\~z5678"

```
00401086 | push cruehead-crackme-3.402008 | 402008:"pppppppppzz~~z5678"
0040108B | call cruehead-crackme-3.40133C | 取[0x402008]最后4位倒序5678到EAX：38 37 36 00
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

将文件内容后4位倒序与sum值对比：

```
00401090  | add esp,4                        |
00401093  | cmp eax,dword ptr ds:[4020F9]    | #将文件内容后4位倒序与sum值对比
00401099  | sete al                          |
0040109C  | push eax                         |
0040109D  | test al,al                       |
0040109F  | je cruehead-crackme-3.401037     |
004010A1  | push cruehead-crackme-3.40210E   | 40210E:"CrackMe v3.0             "
004010A6  | call cruehead-crackme-3.401346   |
```

所以我们只需要将最后4字节设置为地址里面的值就可以通过校验了：

<figure><img src="../../.gitbook/assets/1b64e3914650e05288cf2066b5c2cf7 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/4387cd15a48e81dbc05e92a66a32c78.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

这样子就完成了crack，显示的用户名是刚刚12345678901234逐字节运算以后的结果，根据解密算法使用的异或具有可逆性，如果想要指定的用户名，可以编写注册机：

```cpp
#include <stdio.h>
#include <string.h>

int main() {
	char user[15] = {20}; user[14] = 0; 
	char code[19] = {0};
	int i, len;
	printf("Username: ");
	fgets(user, sizeof(user), stdin);
	len = strlen(user) - 1;
	if(len > 14){
		printf("too long\n");
		return 0; 
	}
	int sum = 0;
	for(i=0; i<14; i++){
		sum += user[i];
	}
	for(i=0x41; i<0x4F; i++){
		code[i-0x41] = user[i-0x41] ^ i;
	}
	sum = sum ^ 0x12345678;
	code[14] = sum & 0xFF; sum = sum >> 8;
	code[15] = sum & 0xFF; sum = sum >> 8;
	code[16] = sum & 0xFF; sum = sum >> 8;
	code[17] = sum & 0xFF;
    FILE *file = fopen("CRACKME3.KEY", "w");
    if (file == NULL) {
        printf("Error: Unable to create file.\n");
        return 1;
    }
    fwrite(code, 1, 18, file); // 写入序列号，长度为18字节
    fclose(file);

    printf("Key saved to CRACKME3.KEY\n");
    getchar();
    return 0;
}

```

运行，成功crack并显示我们需要的用户名

<figure><img src="../../.gitbook/assets/639027358c943018f43c785e3d57c9b.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/b6dcbda33896fcedc039b9bf87e6784.png" alt=""><figcaption></figcaption></figure>







## 012-ACG-crcme1 <a href="#articlecontentid" id="articlecontentid"></a>

常规PEiD检测一下，32位，未知加壳，运行，对话框可以输入name和serial

在x32dbg中分析，首先看到Createfile操作，然后检测文件内容长度（0xC通过校验），然后readfile，IDA中分析一下：

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

可以看到通过校验以后输出对话框`Key File OK teraz tylko Name/Serial!`所以根据判断条件`result = ((byte[lpBuffer] ^ 0x1b) * 4)` 编写代码反推key内容：

```python
F:\Mycode\crackme\012-ACG-crcme1>python
>>> res = [0x168,0x160,0x170,0xEC,0x13C,0x1CC,0x1F8,0xEC,0x164,0x1F8,0x1A0,0x1BC]
>>> key = ''
>>> for i in range(len(res)):
...     key += chr((res[i] // 4) ^ 0x1b)
...
>>> print(key)
ACG The Best
```

<figure><img src="../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

然后点击以后进行name和serial的输入
