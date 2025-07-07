---
description: 软件脱壳
---

# BabyReverse-UnPackMe

## 寻找OEP

壳执行完初始化（解密、反调试、IAT修复）后，通常会有一个“大跳转”把控制权转回原始程序。从壳入口点开始**分析大跳转（跨区块跳转）**，查看跳转后是否转到原始.text代码节，并且是否具有一些OEP的特征（比如Visual C++编写的程序入口可能&#x662F;_**0x55 8BEC `push ebp; mov ebp,esp;`**_&#x5F00;头）

* 壳代码往往在壳自己的段（如 `asprotect` 的 `.aspr` 段）
* 被还原的原始代码一般在 `.text` 段
* 所以在调试器里，要重点看：
  * 有没有 `JMP` 或 `CALL` 跨段跳转
  * 常见形式：`JMP EAX`、`CALL EAX`、`JMP [REG]`、`RET`（从子过程返回）

，一些比较简单的壳jmp eax都是跳转到OEP，dump内存修改程序EP为OEP即可成功脱壳

*   **根据堆栈平衡寻找OEP(ESP大法)：**

    1. 在可疑 `jmp` / `call` 附近查看寄存器窗口的 ESP。
    2. 在数据窗口（Stack）里跟随看 ESP 指向的值（可以下个硬件断点）是否合理：
       * 比如是否指向可执行区域？（可能是 OEP）
       * 是否有一致的栈结构？

    壳执行前后，通常要保证堆栈平衡。

    * `CALL` / `RET` / `push` / `pop` 都会改变 ESP。
    * 如果跳转时堆栈没平衡，程序会崩溃，所以要重点看：
      * 跳转时 ESP 的值是否回到壳最初保存的状态？
      * 常见写法：`pushad`（保存） → 中间解密 → `popad`（还原）

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

