---
cover: ../.gitbook/assets/微信图片_20231017145800.jpg
coverY: 0
---

# LLVM Notebook

## LLVM入门

### llvm vs gcc

* llvm基于模块化/可扩展的设计，使用中间表示（IR）作为通用的数据结构进行代码优化鹤生成，而gcc是集成多个前端鹤后端的传统编译器

### 结构

* 前端：解析源代码，检车错误，并构建特定于语言的抽象语法树（AST）
* 优化器：负责进行各种转换以尝试提高代码的运行时间，例如消除冗余计算
* 后端（代码生成器）：将代码映射到目标指令集

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

* clang：解析c/c++代码
* MLIR：构建可重用和扩展编译器基础设施的新方法
* OpenMP：提供一个OpenMP运行的库函数
* Polly

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption><p>参考文章<a href="https://blog.csdn.net/tjcwt2011/article/details/130028112">https://blog.csdn.net/tjcwt2011/article/details/130028112</a></p></figcaption></figure>

## Clang前端

* 预处理：头文件以及宏处理 使用命令`clang -E main.m`查看preprocessor(预处理)的结果
*   词法分析（Lexer）：从左向右逐行扫描源程序的字符，将识别出的单词转换为统一的机内表示（此法单元token）使用命名`$ clang -fmodules -E -Xclang -dump-tokens main.m`可以进行**词法分析**，生成Token

    例如对代码`void test(int a, int b){ int c = a + b - 3; }`生成token如下：

{% code lineNumbers="true" %}
```cpp
void 'void'  [StartOfLine]  Loc=<main.m:18:1>
identifier 'test'    [LeadingSpace] Loc=<main.m:18:6>
l_paren '('     Loc=<main.m:18:10>
int 'int'       Loc=<main.m:18:11>
identifier 'a'   [LeadingSpace] Loc=<main.m:18:15>
comma ','       Loc=<main.m:18:16>
int 'int'    [LeadingSpace] Loc=<main.m:18:18>
identifier 'b'   [LeadingSpace] Loc=<main.m:18:22>
r_paren ')'     Loc=<main.m:18:23>
l_brace '{'     Loc=<main.m:18:24>
int 'int'    [StartOfLine] [LeadingSpace]   Loc=<main.m:19:5>
identifier 'c'   [LeadingSpace] Loc=<main.m:19:9>
equal '='    [LeadingSpace] Loc=<main.m:19:11>
identifier 'a'   [LeadingSpace] Loc=<main.m:19:13>
plus '+'     [LeadingSpace] Loc=<main.m:19:15>
identifier 'b'   [LeadingSpace] Loc=<main.m:19:17>
minus '-'    [LeadingSpace] Loc=<main.m:19:19>
numeric_constant '3'     [LeadingSpace] Loc=<main.m:19:21>
semi ';'        Loc=<main.m:19:22>
r_brace '}'  [StartOfLine]  Loc=<main.m:20:1>
eof ''      Loc=<main.m:20:2>

//词法分析的时候，将上面的代码拆分一个个token，
//后面数字表示某一行的第几个字符，例如第一个void，表示第18行第一个字符
```
{% endcode %}

* 语法分析（Parser）：从token序列中识别出各类短语并构造语法分析树，还是对于刚刚的代码示例，生成的语法树AST如下：

```cpp
|-FunctionDecl 0x7fa1439f5630 <line:18:1, line:20:1> line:18:6 test 'void (int, int)'
| |-ParmVarDecl 0x7fa1439f54b0 <col:11, col:15> col:15 used a 'int'
| |-ParmVarDecl 0x7fa1439f5528 <col:18, col:22> col:22 used b 'int'
| `-CompoundStmt 0x7fa142167c88 <col:24, line:20:1>
|   `-DeclStmt 0x7fa142167c70 <line:19:5, col:22>
|     `-VarDecl 0x7fa1439f5708 <col:5, col:21> col:9 c 'int' cinit
|       `-BinaryOperator 0x7fa142167c48 <col:13, col:21> 'int' '-'
|         |-BinaryOperator 0x7fa142167c00 <col:13, col:17> 'int' '+'
|         | |-ImplicitCastExpr 0x7fa1439f57b8 <col:13> 'int' <LValueToRValue>
|         | | `-DeclRefExpr 0x7fa1439f5768 <col:13> 'int' lvalue ParmVar 0x7fa1439f54b0 'a' 'int'
|         | `-ImplicitCastExpr 0x7fa1439f57d0 <col:17> 'int' <LValueToRValue>
|         |   `-DeclRefExpr 0x7fa1439f5790 <col:17> 'int' lvalue ParmVar 0x7fa1439f5528 'b' 'int'
|         `-IntegerLiteral 0x7fa142167c28 <col:21> 'int' 3
 
`-<undeserialized declarations>
```

* 语义分析（Sema）：收集标识符的属性信息与语义检查
* 代码生成（CodeGen）：将AST转为llvm代码

AST可以细分为：

* declaration（Decl），如`int n`
* statement（Stmt）如`if(x > 2)` ,`for(int i = 1; i < n ; i++)`,`break;`等等
* expression（Expr）

## IR

IR有三种形式

* 内存中的表示形式:如basicBlock,Instruction这种cpp类
* bitcode形式，这是一种序列化的**二进制**表示形式，拓展名.bc， `$ clang -c -emit-llvm main.m`
* LLVM text形式，一种序列化的表示形式，是**可读的字符串的形式**，似于汇编语言，拓展名.ll， `$ clang -S -emit-llvm main.m`

IR表达：

* Module类, 一个完整的编译单元,一般来说. 这个编译单元就是一个源码文件
* Function类, 对应于一个函数单元,可以描述两种情况: 函数定义鹤函数声明
* BasicBlock类, 表示一个基本代码块, 即没有控制流逻辑的基本流程, 相当于程序流程图中的基本过程(矩阵表示)
* Instruction类, 指令类就是LLVM中定义的基本操作,比如加减乘除这种算数指令

IR语法：（官方语法参考 [/docs/LangRef.html](https://links.jianshu.com/go?to=https%3A%2F%2F%2Fdocs%2FLangRef.html)）

* 注释以分号 ; 开头
* 全局标识符以@开头，局部标识符以%开头
* alloca，在当前函数栈帧中分配内存
* i32，32bit，4个字节的意思
* align，内存对齐
* store，写入数据
* load，读取数据

### LLVM Pass

Pass 直译可以理解为 “趟”，llvm pass对 LLVM IR 进行如修改、插入等操作，LLVM Pass类的核心是它的**run方法**。这个方法进行实际的工作，例如分析或转换IR（Intermediate Representation）。run方法的参数取决于Pass的类型：对于Function Pass，参数是一个Function对象；对于Module Pass，参数是一个Module对象。

**ModulePass**，最通用的 Pass，在 AFL 的源码中就用到了该 Pass。它一次分析一个模块，函数次序不定，不限制使用者的行为，可以删除函数并进行其他修改。使用时需要实现一个继承 ModulePass 的类，并重载 runOnModule() 方法。&#x20;

**FunctionPass**，函数级粒度的 Pass，一次处理一个函数，处理次序不定。它禁止修改外部函数、删除函数、删除全局变量。 同样地，使用时需要实现一个继承 FunctionPass 的类，并重载 runOnFunction() 方法。&#x20;

**BasicBlockPass**，基本块级别的 Pass。除了 FunctionPass 禁止的修改，它还禁止修改或删除外部基本块。使用时需要实现一个继承 BasicBlockPass 的类，并重载 runOnBasicBlock() 方法。

**CallGraphSCCPass**

首先复习一些离散数学相关的知识（图论连通分量部分），强连通分量调用图（SCC call graph）是调用图的强连通分量图，在LLVM中称为CallGraphSCC。强连通分量调用图的某个顶点可以仅包含一个方法，也可以包含调用图中的多个顶点。通过将调用图的有环子图分解为强连通分量，强连通分量调用图将变为无环图，也就是变为有向无环图。

**CallGraphSCCPass构成了LLVM的过程间优化的重要组成部分**。AMDGPU后端中的AMDGPUPerfHintAnalysis就是一个自定义SCC pass：

```cpp
struct AMDGPUPerfHintAnalysis : public CallGraphSCCPass {
...
  bool runOnSCC(CallGraphSCC &SCC) override;
 
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
  }
 
  bool isMemoryBound(const Function *F) const;
 
  bool needsWaveLimiter(const Function *F) const;
 
  struct FuncInfo {
    unsigned MemInstCount;
    unsigned InstCount;
    unsigned IAMInstCount; // Indirect access memory instruction count
    unsigned LSMInstCount; // Large stride memory instruction count
    FuncInfo() : MemInstCount(0), InstCount(0), IAMInstCount(0),
                 LSMInstCount(0) {}
  };
 
  typedef ValueMap<const Function*, FuncInfo> FuncInfoMap;
 
private:
 
  FuncInfoMap FIM;
};
```



### IRBuilder

使用IRBuilder来生成IR对象

```cpp
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/Module.h"

using namespace llvm;

int main(int argc, char* argv[])
{
    LLVMContext context;
    IRBuilder<> builder(context);

    // Create a module
    Module* module = new Module("HelloModule", context);

    // Add a function
    Type* int32Type = builder.getInt32Ty();
    FunctionType* functionType = FunctionType::get(int32Type, false);
    Function* function = Function::Create(functionType, GlobalValue::ExternalLinkage, "HelloFunction", module);

    // Create a block
    BasicBlock* block = BasicBlock::Create(context, "entry", function);
    builder.SetInsertPoint(block);

    // Add a return
    ConstantInt* zero = builder.getInt32(0);
    builder.CreateRet(zero);

    // Print the IR
    module->print(outs(), nullptr);

    return 0;
}

```



## 参考文章



{% embed url="https://blog.csdn.net/weixin_45651194/article/details/128473983" %}

{% embed url="https://blog.csdn.net/tjcwt2011/article/details/130028112" %}

{% embed url="https://llvm.org/docs/LangRef.html" %}

{% embed url="https://zhuanlan.zhihu.com/p/290946850" %}
