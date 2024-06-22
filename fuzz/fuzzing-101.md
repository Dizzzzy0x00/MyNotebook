---
description: https://github.com/antonio-morales/Fuzzing101
---

# Fuzzing 101

{% embed url="https://github.com/antonio-morales/Fuzzing101" %}

## Exercise 1 - Xpdf

{% embed url="https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%201" %}

详细的过程在Fuzzing101的readme文档中有详细介绍这里就带过

```
mkdir fuzzing_xpdf && cd fuzzing_xpdf/
wget https://dl.xpdfreader.com/old/xpdf-3.02.tar.gz
tar -xvzf xpdf-3.02.tar.gz

#构建Xpdf并进行测试：
cd xpdf-3.02
sudo apt update && sudo apt install -y build-essential gcc
./configure --prefix="$HOME/fuzzing_xpdf/install/"
make
make install
```

AFL++安装

```
sudo apt-get update
sudo apt-get install -y build-essential python3-dev automake git flex bison libglib2.0-dev libpixman-1-dev python3-setuptools
sudo apt-get install -y lld-11 llvm-11 llvm-11-dev clang-11 || sudo apt-get install -y lld llvm llvm-dev clang 
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get install -y gcc-$(gcc --version|head -n1|sed 's/.* //'|sed 's/\..*//')-plugin-dev libstdc++-$(gcc --version|head -n1|sed 's/.* //'|sed 's/\..*//')-dev


#这里我进行了gcc版本的更新，更新以后要修改优先级：
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 1 
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 10

cd $HOME
git clone https://github.com/AFLplusplus/AFLplusplus && cd AFLplusplus
export LLVM_CONFIG="llvm-config-11"
make distrib
sudo make install
```

```
export LLVM_CONFIG="llvm-config-11"
CC=$HOME/AFLplusplus/afl-clang-fast CXX=$HOME/AFLplusplus/afl-clang-fast++ ./configure --prefix="$HOME/Desktop/fuzzing_xpdf/install/"
AFL_USE_ASAN=1 make
AFL_USE_ASAN=1 make install
```



开始fuzz，由于我开启了ASAN，所以这里使用-m none取消内存限制：

```
afl-fuzz -m none -i $HOME/Desktop/fuzzing_xpdf/pdf_examples/ -o $HOME/Desktop/fuzzing_xpdf/out/ -s 123 -- $HOME/Desktop/fuzzing_xpdf/install/bin/pdftotext @@ $HOME/Desktop/fuzzing_xpdf/output
```

<figure><img src="../.gitbook/assets/f45961559ef18362c49e5b088a2dd18 (1).png" alt=""><figcaption></figcaption></figure>

30分钟触发了四个crashes：

<figure><img src="../.gitbook/assets/648e84a11d9a8a6cf55d618c47765cf.png" alt=""><figcaption></figcaption></figure>

### Crash分析

因为在AFL进行模糊测试时开启了ASAN，所以复现时很方便，第一个crash复现如下，是一个栈溢出漏洞，根据栈回溯可以看到，由于一直循环的调用Parse::getObj----XRef::fetch----Object::dictLookup----Parser::makeStream这三个函数导致最后栈溢出，定位源码以后看到是由于程序员编写递归时没有设置好递归深度上限，而样本导致了无限的递归。



<figure><img src="../.gitbook/assets/d6aae3c29a74dd16c9e8497c5cbf15f.png" alt=""><figcaption></figcaption></figure>

#### GDB调试

调试的思路就是找循环的最后一个函数，看是为什么会跳转会一开始的函数，先看看源码其实已经很明显了：

<figure><img src="../.gitbook/assets/ddaea5e200f60edcff9fea548a63774.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/63fb8206fc29725e7e480d804eeabb3.png" alt=""><figcaption></figcaption></figure>

在Parser::makeStream下断点

```
gdb --args ./install/bin/pdftotext ./out/default/crashes/crash1

pwndbg> b Parser::makeStream    //下断点
Breakpoint 1 at 0x6a5120: file Parser.cc, line 146.
pwndbg> r    //运行
pwndbg> bt //栈回溯
pwndbg> b *Parser::getObj    //下断点
Breakpoint 2 at 0x6a0a90: file Parser.cc, line 41.
pwndbg>c
```

<figure><img src="../.gitbook/assets/d36b41a26a6a96065297e3e7dac08f7.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/332cc7d9270657061397681afade59b.png" alt=""><figcaption></figcaption></figure>

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

