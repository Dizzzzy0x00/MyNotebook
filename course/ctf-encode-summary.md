---
description: CTF中常见编码特征总结
cover: ../.gitbook/assets/101012194_p0_master1200.jpg
coverY: 126.72933549432739
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# CTF Encode Summary

这里是Nefelibata，欢迎来到我的博客！一起记录收获知识的每一刻

本篇文章参考：

{% embed url="https://www.cnblogs.com/ruoli-s/p/14206145.html" %}

## ASCII编码

​ ASCII 码是对**英语字符与二进制位之间**的关系，做了统一规定

**特征：** **只含有数字**

* 0-9, 49-57
* A-Z, 65-90
* a-z, 97-122
*   **举例：**

    ```
    明文：hello,world.
    十六进制：0x680x650x6c0x6c0x6f0xff0c0x770x6f0x720x6c0x640x2e
    十进制：1041011081081112551211911111410810046
    二进制：011010000110010101101100011011000110111100101100011101110110111101110010011011000110010000101110
    ```

### 在线解密工具

{% embed url="http://www.ab126.com/goju/1711.html" %}

## Base家族编码

> #### Base16 / Base32 / Base64 / Base58 / Base85 / Base 100

### Base16编码

​ Base16编码是一个标准的十六进制字符串（注意是字符串而不是数值），更易被人类和计算机使用，因为它并不包含任何控制字符，以及Base64和Base32中的“=”符号

​ **特征**：Base16编码是将二进制文件转换成由**16个字符组成的**（0-9，A-F）文本，**没有等号**并且**数字要多于字母**

​ 在线解密工具

​ https://ctf.bugku.com/tool/base16

### Base32编码

​ Base32将串起来的二进制数据按照5个二进制位分为一组，由于传输数据的单位是字节(即8个二进制位).所以分割之前的二进制位数是40的倍数(40是5和8的最小公倍数).如果不足40位，则在编码后数据补充"="，一个"="相当于一个组(5个二进制位)，编码后的数据是原先的8/5倍

​ **特征**：base32的编码表是由\*\*（A-Z、2-7）32个**可见字符构成，**“=”**符号用作后缀填充。明文**超过十个**后面就会有**很多等号\*\*

​ 在线解密工具

​ https://ctf.bugku.com/tool/base32

### Base64编码

​ Base64编码要求把3个8位字节（&#x33;_&#x38;=24）转化为4个6位的字节（&#x34;_&#x36;=24），之后在6位的前面补两个0，形成8位一个字节的形式。如果剩下的字符不足3个字节，则用0填充，输出字符使用‘=’，因此编码后输出的文本末尾可能会出现1或2个‘=’。

​ **特征：base64的编码表是由（A-Z、a-z、0-9、+、/）64个**可见字符构成，**“=”符号用作后缀填充。一般情况下密文尾部都会有两个等号**，明文很少的时候则没有

​ 在线解密工具

​ https://ctf.bugku.com/tool/base64

### Base58编码

​ Base58是用于比特币（Bitcoin）中使用的一种独特的编码方式，主要用于产生Bitcoin的钱包地址，比特币的Base58字母表：

123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz，去掉了一些容易混淆的数字和字母：0（数字0）、O、l（L）、I（i）

​ **特征**：base58的编码表相比base64少了**数字0，大写字母I，O，小写字母 l** (这个是L），以及符号\*\*‘+’和‘/’\*\*，**它最大的特点是没有等号**

​ 在线解密工具

​ https://ctf.bugku.com/tool/base58

### Base91编码

​ **特征**：base91的密文由92个字符（0-9，a-z，A-Z,!#$%&()\*+,./:;<=>?@\[]^\_\`{|}\~”）组成

​ 在线解密工具

​ https://ctf.bugku.com/tool/base91

### Base92编码

​ **特征**：base91的密文由92个字符（0-9，a-z，A-Z,!#$%&()\*+,./:;<=>?@\[]^\_\`{|}\~”）组成

​ 在线解密工具

​ https://ctf.bugku.com/tool/base92

### Base100编码

​ Base100编码/解码工具（又名：Emoji表情符号编码/解码），可将文本内容编码为Emoji表情符号；同时也可以将编码后的Emoji表情符号内容解码为文本。

​ **特征**：**一堆Emoji表情**

​ 在线解密工具

​ https://ctf.bugku.com/tool/base100

### **举例汇总**

```
明文：hello，world.123456
base16: 68656C6C6F2C776F726C642E313233343635
base32: NBSWY3DPFR3W64TMMQXDCMRTGQ3DK===
base64: aGVsbG8sd29ybGQuMTIzNDY1
base58: 2smDFYXWKE8vc8XA8dadEYcSqcQb
base85: BOu!rDst>tGAhM<A1fSl1GgsI
base91: TPwJh>go2Tv!_,aRA2IbLmA
base100: 👟👜👣👣👦📦💳💃👮👦👩👣👛🐥🐨🐩🐪🐫🐬🐭
```

## MD5

一般MD5值是32位由数字“0-9”和字母“a-f”所组成的字符串，字母大小写统一。

16位值是取的是8\~24位。

**一个原始数据的MD5值是唯一的**，同一个原始数据不可能会计算出多个不同的MD5值

**原始数据与其MD5值并不是一一对应的**，有可能多个原始数据计算出来的MD5值是一样的(碰撞性)。

​ \*\*特征：\*\*有固定长度，一般是32位或者16位,由数字“0-9”和字母“a-f”组成

​ **举例：**

```
明文：hello，world.123456
md5(hello，world.123456,32) = 5189503aae1b1c0a6fbf7ea9e3128ab0
md5(hello，world.123456,16) = ae1b1c0a6fbf7ea9
```

### 在线解密工具

```
https://www.cmd5.com/
https://www.somd5.com/
http://www.hiencode.com/hash.html

www.cmd5.com（带批量解密工具）
www.somd5.com
cmd5.la
pmd5.com
www.ttmd5.com（带批量解密工具）

/**由于MD5的碰撞性，如果告诉你一个MD5值，你是无法通过它还原出它的原始数据的。
```

## SHA1

SHA1是一种密码散列函数，可成一个被称为消息摘要的160位，20字节的散列值，散列值通常的呈现形式为**40位十六进制数**。这种加密和MD5类似。

​ \*\*特征：\*\*有固定长度，为40位的字符串

​ **举例：**

```
明文：hello，world.123456
sha1（hello，world.123456）= 0179303b8f08fbc3d16cd23a4be5828790e12375
```

### 在线加密工具

​ http://www.hiencode.com/hash.html

## SHA256

SHA256是最流行的计算机算法之一，也是目前比较强大的加密函数之一。SHA256被用于比特币等加密货币，是一个安全更高、牢不可破的函数。同时，它是一个确定的单向哈希函数，**不可逆**的。对于任意长度的消息，SHA256都会产生一个256bit长度的散列值，称为消息摘要，可以用一个长度为64的十六进制字符串表示。

​ **特征**：有固定长度，64个字符的字符串

​ **举例**：

```
明文：flag{helloworld}
SHA256： fe1994680353908c5b37d7b52fca71db866104fd7bc092eca3661663d023dcc0
```

### 在线加密工具

​ http://www.hiencode.com/hash.html

## HMAC

HMAC (Hash-based Message Authentication Code)，MAC算法是hash算法，它可以是MD5, SHA-1或者 SHA-256，他们分别被称为HMAC-MD5，HMAC-SHA1， HMAC-SHA256常用，用于接口签名验证，这种算法就是在前两种加密的基础上引入了秘钥，而秘钥又只有传输双方才知道。

\*\*特征：\*\*和MD5类似，有固定长度，但是有秘钥。

```
明文：flag{helloworld}
密钥：abc

密文：
HMAC-SHA1：  d2a0ef0ca992c30fd95dc85a4440e462500da324
HMAC-SHA256：b800b2e6c01b8ba42e61d2fbdcbc4ec65a1118c89b02039b04dc95786dfdd300
HMAC-MD5：   03ccd1901eb4b7e28f1c0e05e7995d03
```

### 在线加密工具

```
http://encode.chahuo.com/
```

## NTLM（MD4）

Windows NT 早期版本的标准安全协议。与它相同的还有Domain Cached Credentials（域哈希）。NTLM哈希本质上是**MD4**加密，不能逆向只能枚举爆破

MD4是按照分块进行处理的，分块长度为512bit, 大多数情况下，数据的长度不会恰好满足是512的整数倍，因此需要进行「padding」到给定的长度。

**特征**：固定长度，32个字符的字符串

### Python爆破短密码解密

```python
#已知长度较短，由数字和符号组成的NTLM可以进行爆破

from Crypto.Hash import MD4
import string
import hashlib

# 数据字典：密码由数字和符号组成
dataDictionary = string.digits + string.punctuation
miwen = 'CDABE1D16CE42A13B8A9982888F3E3BE'
# 密码长度不超过5
for a in range(len(dataDictionary)):
    for b in range(len(dataDictionary)):
        for c in range(len(dataDictionary)):
            for d in range(len(dataDictionary)):
                for e in range(len(dataDictionary)):
                    password1 = dataDictionary[a]+"\0"+dataDictionary[b]+"\0"+dataDictionary[c]+"\0"+dataDictionary[d]+"\0"+dataDictionary[e]+"\0"
                    # print(password)
	       # 以指定的utf-8编码格式编码字符串，返回bytes 对象
                    password2 = password1.encode('utf-8')
                    # print(password)

                    # Crypto.Hash模块的MD4算法
                    # result = MD4.new()
                    # result.update(password2)
                    # if "CDABE1D16CE42A13B8A9982888F3E3BE" == str.upper(result.hexdigest()):
                    
                    # hashlib的md4算法,但是生成的明文都是小写，但是题目给的是大写，所以用str的upper函数把明文转化成大写
                    if miwen == str.upper(hashlib.new('md4',password2).hexdigest()):
                        print("密文对应的密码是: " + password1)
                        exit()
                        
 #运行结果：得到CDABE1D16CE42A13B8A9982888F3E3BE解密后的结果为：1+2=3
```

## AES/DES/RC4/Rabbit/3DES型加密

​ 都是**非对称性加密**算法，就是引入了密钥，

​ **特征**：**与Base64类似**

​ **在线加解密**

```
https://www.sojson.com/encrypt_aes.html
```

例题：

```
AES加密
密钥：ISCC
一次密文：
U2FsdGVkX188LoeZblovpDrWYKIpbcWS0S+KVZ4YOIche+TpsujBr6Tpjvp8gVjC
GGTxccXKQjsAPag+pp0AO4wPcD/P1GrsyrlqPkkz9tc0VxukKfFG0Udk3FONEqoc
DJX2daXmlDHcw+cQXPkD6A==
二次密文：U2FsdGVkX18OvTUlZubDnmvk2lSAkb8Jt4Zv6UWpE7Xb43f8uzeFRUKGMo6QaaNFHZriDDV0EQ/qt38Tw73tbQ==
明文：
flag{DugUpADiamondADeepDarkMine}
```

## Unicode编码

Unicode（统一码、万国码、单一码）是一种在计算机上使用的字符编码。它用两个字节来编码一个字符,字符编码一般用十六进制来表示。**可以说Unicode与HTML实体编码是一个东西**

**举例：**

```
明文：flag{helloworld}
&#x [hex]：&#x0066;&#x006C;&#x0061;&#x0067;&#x007B;&#x0068;&#x0065;&#x006C;&#x006C;&#x006F;&#x0077;&#x006F;&#x0072;&#x006C;&#x0064;&#x007D;

&# [hex]：&#00102;&#00108;&#00097;&#00103;&#00123;&#00104;&#00101;&#00108;&#00108;&#00111;&#00119;&#00111;&#00114;&#00108;&#00100;&#00125;

\u [hex]：\U0066\U006C\U0061\U0067\U007B\U0068\U0065\U006C\U006C\U006F\U0077\U006F\U0072\U006C\U0064\U007D

\u+ [hex]：\U+0066\U+006C\U+0061\U+0067\U+007B\U+0068\U+0065\U+006C\U+006C\U+006F\U+0077\U+006F\U+0072\U+006C\U+0064\U+007D
```

### 在线解密工具

```
http://www.mxcz.net/tools/Unicode.aspx
```

## HTML实体编码

字符实体是用一个编号写入HTML代码中来代替一个字符，在使用浏览器访问网页时会将这个编号解析还原为字符以供阅读。

```
明文：hello，world.
十进制：&#104;&#101;&#108;&#108;&#111;&#65292;&#119;&#111;&#114;&#108;&#100;&#46;
十六进制：&#x68;&#x65;&#x6C;&#x6C;&#x6F;&#xFF0C;&#x77;&#x6F;&#x72;&#x6C;&#x64;&#x2E;

解密工具
https://www.toolzl.com/tools/htmlende.html

https://www.qqxiuzi.cn/bianma/zifushiti.php

Unicode：www.sojson.com
16进制Unicode：www.msxindl.com
HTML字符实体：www.qqxiuzi.cn
```

## Escape/Unescape编码（%u）

**Escape/Unescape加密解码/编码解码**,又叫%u编码，其实就是字符对应UTF-16 16进制表示方式前面加%u

特征：编码前面都有%u

举例：

```
明文：hello，world.
密文：%u0068%u0065%u006c%u006c%u006f%uff0c%u0077%u006f%u0072%u006c%u0064%u002e
```

### 在线解密工具

http://web.chacuo.net/charsetescape/

## URL编码

编码方法很简单，在该字节ascii码的的16进制字符前面加%. 如 空格字符，ascii码是32，对应16进制是'20'，那么urlencode编码结果是:%20

\*\*特征：\*\*编码前面都有%

### 在线加解密

```
http://web.chacuo.net/charseturlencode
```

## Hex编码

在Intel HEX文件中，每一行包含一个HEX记录。这些记录由对应机器语言码和/或常量数据的**十六进制编码数字**组成

\*\*特征：\*\*十六进制（Hexadecimal）

是计算机中数据的一种表示方法，**由0-9，A-F组 成，字母不区分大小写**。

与10进制的对应关系是：0-9不变，A-F对应10-15

**举例：**

```
明文：hello，world.
密文（带%）：%68%65%6c%6c%6f%ef%bc%8c%77%6f%72%6c%64%2e
密文（不带%）：68656C6C6FEFBC8C776F726C642E
```

### 在线加解密

```
https://www.107000.com/T-Hex
http://stool.chinaz.com/hex
Hex编码：https://www.107000.com/T-Hex
```

## Quoted-printable

它是多用途互联网邮件扩展（MIME) 一种实现方式。有时候我们可以邮件头里面能够看到这样的编码，任何一个8位的字节值可编码为3个字符：一个等号”=”后跟随两个十六进制数字(0–9或A–F)表示该字节的数值.

**特征：**

**只能对汉字进行编码，一个汉字对应=加两个大写字母或数字组合**

**举例：**

```
明文：天上掉下了个猪八戒
密文：=E5=A4=A9=E4=B8=8A=E6=8E=89=E4=B8=8B=E4=BA=86=E4=B8=AA=E7=8C=AA=E5=85=AB=E6=88=92
```

### 在线加解密

```
http://www.mxcz.net/tools/QuotedPrintable.aspx
```

## XXencode

将输入文本以**每三个字节**为单位进行编码。如果最后剩下的资料少于三个字节，不够的部分**用0补齐**。这三个字节共有24个Bit，以6bit为单位分为4个组，每个组以十进制来表示所出现的数值只会落在0到63之间

\*\*特征：\*\*字符范围是：0-9，A-Z，a-z，一共64个字符。跟base64打印字符相比，就是UUencode多一个“-” 字符，少一个”/” 字符。

**举例：**

```
明文：flag{hello，world.}
密文：（gb2312编码）HNalVNrhcNKlgPuCgRqxmP4EiTE++
```

### 在线加解密

```
http://web.chacuo.net/charsetxxencode
```

## UUencode

一种二进制到文字的编码，最早在unix邮件系统中使用，UUencode将输入文本以每三个字节为单位进行编码，如果最后剩下的资料少于三个字节，不够的部份用0补齐，三个字节共有24个Bit，以6-bit为单位分为4个组

每个组以十进制来表示所出现的字节的数值。这个数值只会落在0到63之间。然后将每个数加上32，所产生的结果刚好落在ASCII字符集中可打印字符（32-空白…95-底线）的范围之中

**举例：**

```
明文：flag{hello，world.}
密文：49FQA9WMH96QL;^^\C'=O<FQD+GT`
```

**在线解密&工具**：

```
https://www.qqxiuzi.cn/bianma/uuencode.php
```

## js专用加密

### JS颜文字加密（aaencode编码）

**特征：**

一堆颜文字构成的js代码，在F12中可直接解密执行

### Jother编码

**简述：**

jother是一种运用于javascript语言中利用少量字符构造精简的匿名函数方法对于字符串进行的编码方式。

**特征：**

只用 \*\*! + ( ) \[ ] { } \*\*这八个字符就能完成对任意字符串的编码。可在F12中解密执行

### JSFuck编码

**特征：**

**与jother很像，只是少了{ }**

**在线加密：**

```
https://utf-8.jp/public/aaencode.html
http://www.jsfuck.com/
```

解密在F12的console中

## brainfuck编码

**特征**：

只有八种符号，所有的操作都由这八种符号 **(> < + - . , \[ ])** 的组合来完成。

```
明文：hello,world.
密文：+++++ +++++ [->++ +++++ +++<] >++++ .---. +++++ ++..+ ++.<+ +++++ ++[->
----- ---<] >---. <++++ ++++[ ->+++ +++++ <]>++ +++++ ++++. ----- ---.+
++.-- ----. ----- ---.< +++++ ++[-> ----- --<]> ----- .<
```

在线加解密：

https://www.splitbrain.org/services/ook
