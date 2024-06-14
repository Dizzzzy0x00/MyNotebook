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

进行测试，这里尝试对test\_gdiplus.exe进行插桩并fuzz：

```
-i //存放样本的目录
-o //保存输出数据,包括 crash文件、测试用例等
-D //DynamoRIO的路径 (drrun, drconfig)
-t msec //每一次样本执行的超时时间
第一个"--"分割符	//后面跟的是插桩的参数
第二个"--"分割符	//后面跟的是目标程序的参数
@@ //引用 -i 参数的中的测试用例
```

```bash
F:\AFL\winafl-master\build32\bin\Release>F:\DynamoRIO-Windows-10.0.0\bin32\drrun.exe -c winafl.dll -debug -target_module test_gdiplus.exe -target_offset 0x10b0  -fuzz_iterations 5 -nargs 2  -- test_gdiplus.exe not_kitty.bmp
#创建in和out目录，存放输入种子in和输出out，in文件夹中要存放一个测试初始用例，这里用的testcase里面的bmp
F:\AFL\winafl-master\build32\bin\Release>.\afl-fuzz.exe  -debug  -i  in -o out -D F:\DynamoRIO-Windows-10.0.0\bin32 -t 20000  -- -coverage_module  test_gdiplus.dll   -coverage_module WindowsCodecs.dll -fuzz_iterations 5 -target_module test_gdiplus.exe -target_offset 0x10b0 -nargs 2  -- test_gdiplus.exe @@
```

结果：成功插桩，成功fuzz：

<figure><img src="../.gitbook/assets/3a1b68558aa1e9a96cbfb5e1d6ecab1.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/1ba4ecd2561b16e4594afe043a1fe69.png" alt=""><figcaption></figcaption></figure>

```
WinAFL监控参数说明：
Process timing		//Fuzzer运行时长、以及距离最近发现的路径、崩溃和挂起经过了多长时间
Overall results		//Fuzzer当前状态的概述
Cycle progress		//当前Fuzz的进展
Map coverage		//目标二进制文件中的插桩代码所观察到覆盖范围的细节
Stage progress		//Fuzzer现在正在执行的文件变异策略、执行次数和执行速度
Findings in depth	//有关我们找到的执行路径，异常和挂起数量的信息
Fuzzing strategy yields	//关于突变策略产生的最新行为和结果的详细信息
Path geometry		//有关Fuzzer找到的执行路径的信息
CPU load			//CPU利用率
```

注意，根据其他师傅博客中的提示“ 第一次对目标程序进行Fuzz时需要加上debug，方便出错时进行错误的排查，确认无误后**记得取消debug参数，否则会极大影响运行效率**”

ok，至此WinAFL(32)环境搭建完成

### Fuzzing 7-Zip

下载源码（这里我选择最新的 7-Zip 23.01（2023-06-20） 7-Zip 源代码 ）：

{% embed url="https://sparanoid.com/lab/7z/download.html" %}

