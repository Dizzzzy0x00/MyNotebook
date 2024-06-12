---
description: https://github.com/antonio-morales/Fuzzing101
---

# Fuzzing 101

{% embed url="https://github.com/antonio-morales/Fuzzing101" %}

## Exercise 9 - 7-Zip

{% embed url="https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%209" %}

WinAFL

`winafl` 是 `afl` 在 `windows` 的移植版， `winafl` 使用 `dynamorio` 来统计代码覆盖率，并且使用共享内存的方式让 `fuzzer` 知道每个测试样本的覆盖率信息。

### 环境配置

#### DynamoRIO安装

DynamoRIO 是一款运行时代码操纵系统，其独特之处在于能在程序执行过程中对任意部分的代码实施变换。它提供了一个接口，让构建动态工具变得轻松，这些工具适用于多种用途，包括但不限于程序分析与理解、性能剖析、代码插入、优化以及编译翻译等等。与其他动态工具系统相比，DynamoRIO 的能力不仅限于调用插桩(trampoline)，更允许开发者借助强大的 IA-32/AMD64/ARM/AArch64 指令操作库，直接修改应用程序指令。

DynamoRIO 确保了在标准操作系统（如 Windows、Linux 或 Android）和主流硬件平台上高效透明地进行代码操控，支持 IA-32、AMD64、ARM 和 AArch64 架构

这里我选择直接下载的releases版本，注意解压后的cmake目录，我的cmake目录为：`F:\DynamoRIO-Windows-10.0.0\cmake` ，后续安装WinAFL时会用到这个目录

{% embed url="https://dynamorio.org/page_releases.html" %}

#### WinAFL安装：

{% embed url="https://github.com/googleprojectzero/winafl" %}

使用Developer Command Prompt for VS 2022对WinAFL源码进行编译

```bash
F:\AFL\winafl-master>mkdir build32
F:\AFL\winafl-master>cd build32
#下面的-DDynamoRIO_DIR填写更改安装的DynamoRIO cmake路径
F:\AFL\winafl-master\build32>cmake -G"Visual Studio 17 2022" -A Win32 .. -DDynamoRIO_DIR=F:\DynamoRIO-Windows-10.0.0\cmake
F:\AFL\winafl-master\build32>cmake --build . --config Release
```

安装完成以后的WinAFL release如下：

<figure><img src="../.gitbook/assets/d9fba4346ec741eb99a34cbb05d4e83.png" alt=""><figcaption></figcaption></figure>
