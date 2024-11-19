# BCSD1-Algorithm1——CFG

给定二进制文件生成对应的CFG图

## 基本算法

构建控制流图（CFG）的三个步骤：

1.  反汇编

    * 线性扫描，OD等很多调试器都用的是这种方法
    * 递归搜索，IDA用的是这种方法



    \

2.  划分基本块：

    所谓基本块，是指程序—顺序执行的语句序列，其中只有一个入口和一个出口，入口就是其中的第—个语句，出口就是其中的最后一个语句。对一个基本块来说，执行时只从其入口进入，从其出口退出，基本块的一个典型特点是：**只要基本块中第一条指令被执行了，那么基本块内所有执行都会按照顺序仅执行一次**，具体而言：

    * 只有一个入口，表示程序中不会有其它任何地方能通过跳转类指令进入到此基本块中。
    * 只有一个出口，表示程序只有最后一条指令能导致进入到其它基本块去执行。

    \

3. 构建控制流图

一般而言应该以函数为单位构建控制流图，然后再构建一个函数的关系图，分两层的目的是为了清晰明了，而且编译器生成的程序基本都是以函数为单位的

### 使用Angr

{% embed url="https://github.com/axt/angr-utils" %}

### 使用IDA-PRO

参考文章：

{% embed url="https://blog.csdn.net/weixin_61967363/article/details/138092586" %}

{% embed url="https://cloud.tencent.com/developer/article/2216945" %}

IDApython主要有三个模块

1. idc：兼容IDA Pro中idc函数的模块；
2. idautils：逆向分析中常用的一个模块，大多数处理方法都是需要依托于这个模块；
3. idaapi：该模块允许使用者通过类的形式，访问更多底层的数据；

接下来使用ida python编写脚本构造二进制文件的cfg，demo示例代码先获取一些基本的信息（架构，函数名字和地址）：

````python
```python
def get_bitness():
    #获取程序的位信息
    info = idaapi.get_inf_structure()
    if info.is_64bit():
        return 64
    elif info.is_32bit():
        return 32

def convert_procname_to_str(procname, bitness):
    #格式化json输出
    if procname == 'mipsb':
        return "mips_{}".format(bitness)
    if procname == "arm":
        return "arm_{}".format(bitness)
    if "pc" in procname:
        return "x86_{}".format(bitness)
    raise RuntimeError(
        "[!] Arch not supported ({}, {})".format(
            procname, bitness))

def build_cfg(target_file,output_dir):
    # 创建输出目录
    if not os.path.isdir(output_dir):
        os.mkdir(output_dir)

    output_dict = dict()
    output_dict[target_file] = dict()

    #获取架构信息
    procname = idaapi.get_inf_structure().procName.lower() 
    bitness = get_bitness()

    output_dict[target_file]['arch'] = convert_procname_to_str(procname, bitness)

    # 初始化函数列表
    output_dict[target_file]['func_list'] = list()

    # 遍历所有函数
    for func in idautils.Functions():
        func_name = idc.get_func_name(func)  # 获取函数名
        func_addr = hex(func)  # 获取函数地址并转为16进制字符串

        print(f"Found function '{func_name}' at address {func_addr}")

        # 将函数名和地址存入字典
        output_dict[target_file]['func_list'].append(
            {'fname':func_name,'addr':func_addr})
        # 记录 main 函数的地址
        if func_name == "main":
            main_addr = func
            output_dict[target_file]['main_addr'] = hex(main_addr)
```
````



demo运行以后我们可以获取二进制文件的基本信息，输出如下：

````json
```json
{
    ".\\testcase\\hello.exe": {
        "arch": "x86_64",
        "func_list": [
            {
                "fname": "pre_c_init",
                "addr": "0x140001010"
            },
            {
                "fname": "pre_cpp_init",
                "addr": "0x140001130"
            },
            ...
            ...
            ...
        ],
        "main_addr": "0x140001450"
    }
}
```
````

capstone实现反汇编可执行文件,示例：

```python
from capstone import *  #将capstone模块中函数全部导入
code=b"\x55\x48\x8b\x13\x00\x00"  #设置需要进行反汇编的十六进制机器码
​
#创建Cs类对象md，我们需要给这个类传两个参数：硬件架构，硬件模式（位长）
md=Cs(CS_ARCH_X86,CS_MODE_64) #表示此时md对象对采用x86架构64位，对机器码进行汇编
​
for i in md.disasm(code,0x1000): #循环遍历disasm函数返回的列表
    print("0x%x:\t %s\t%s" %(i.address,i.mnemonic,i.op_str))
```

继续在build\_cfg中实现代码：

