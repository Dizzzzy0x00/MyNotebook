---
description: 软件脱壳
---

# BabyReverse-UnPackMe

## ASProtect

主要用于保护可执行文件（EXE/DLL）不被破解或逆向分析，ASProtect 脱壳难点主要有：

1. 找到真正的 OEP
   1. 壳代码会在入口点先执行自己的逻辑，然后才跳到真正的程序入口（Stolen code）。
   2. 通过断点/硬件断点跟踪 `Pushad`/`Popad` 或 `CALL EAX` 的最后一跳。
2. 内存解密
   1. 大多数版本是运行时解密 + 内存重定位。
   2. 必须 dump 内存中已经还原好的模块，同时重建 IAT（导入表）。
3. #### 导入表（IAT）修复
   1. ASProtect 的 IAT 加密机制:(1)**破坏原始IAT结构**。ASProtect会抹去PE头中的Improtect Table目录项，以及导入表中DLL名称和函数名的字符串。(2)**动态加载API**。通过LoadLibrary和GetProcAddress来动态获取API地址，关键API的调用会被替换为跳板代码地址
   2. 很多脱壳自动工具只做内存 dump，但 ASProtect 会把 IAT 也混淆，要手动修复或用脱壳机自动修复。
4. #### 反调试绕过
   1. 需要在调试器里 Patch 反调试点或者使用 Anti-Anti-Debug 插件（OllyDbg/ x64dbg 插件）。

