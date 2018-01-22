 Lynn is Here
=======


## C 预处理宏

请先参照这里[Click](http://www.cnblogs.com/clover-toeic/p/3851102.html)了解C语言预处理宏。（PS:[这里](https://github.com/LynnListen/LynnListen.github.io/blob/master/worknote/C%20language%20preprocess%20-%20clover_toeic.pdf)有pdf备份）  
字符串化操作符#在宏定义中可以将后续的code转化成字符串，但后面只能接宏参数。若在宏定义的代码中存在#开头的预处理宏，将会造成编译不能通过出现。
参照我维护的[代码](https://github.com/MlWoo/torch7/blob/internal-use/lib/TH/generic/THTensorCopy.c)。
在 `#define TH_TENSOR_APPLY2_ADVANCED_INDEX(TYPE1, TENSOR1, TYPE2, TENSOR2, ADV_CODE, ORI_CODE) \`的宏定义中，想要使用openmp和
ivdep对代码进行并行化或者串行化操作时

```c
#pragma omp parallel for if (TENSOR2##Size > TH_OMP_OVERHEAD_THRESHOLD_COPY) private(TENSOR2##BasicIndex, index, TENSOR1##Local, TENSOR2##Local, iter, dim, i, j) ) 
#pragma ivdep
```
由于#后面所接的`pragma` 为关键字而不是宏参数，如果不做任何处理将会出现以下错误
```c
error:'#' is not followd by a macro parameters.
```
因此需要用`_Pragma`处理。
```
#define _STR(s)   #s
#define STR(s)    _STR(s)
//#define PRAGMA(P) _Pragma( #P )     //commented for deprecated 
#define PRAGMA2(P) _Pragma( STR(P) )
```
这里弃用了直接`_Pragma`的定义（第三行），而是使用了argument Prescan，即分层展开的技术。这是因为用#进行预处理之后，openmp之后的参数中
粘接预处理宏##将不能展开使用，因此需要分层展开。

## Torch7 问题汇总  
1. cannot load "libtorch.so"   
There maybe is a problem of a .so file(daynamic library). The sub dynamic library is not linked or a problem occurs when the source file is compiled.A good way to debug that would be too run the following line on the target system.
```bash
install/bin/luajit -e 'ffi=require "ffi"; ffi.load("install/lib/lua/5.1/libtorch.so")'
```

## Linux下问题汇总  
1. function symbol  
* dynamic library  nm XXX.so  
* object file nm -C XXX.o  

## CMAKE 问题汇总
安装路径设置:  CMAKE_INSTALL_PREFIX   
C 编译器设置:  CMAKE_C_COMPILER   
C Flags设置:  CMAKE_C_FLAGS=-xMIC-AVX512   
C++标准设置:  CMAKE_CXX_STANDARD   

## icc 问题汇总   
avx，avx2 C compiler flags -mavx -march=core-avx2   
Table below summarizes the compiler arguments necessary for automatic vectorization with AVX-512 with the Intel C++ compiler and with GCC.   

|                   | Intel Compilers   |        GCC        |   
|:-----------------:|:-----------------:|:-----------------:|  
| Cross-platform    | -xCOMMON-AVX512   | -mavx512f -mavx512cd |  
|Xeon Phi processors| -xMIC-AVX512      | -mavx512f -mavx512cd -mavx512er -mavx512pf|  
|Xeon processors    | -xCORE-AVX512     | -mavx512f -mavx512cd -mavx512bw -mavx512dq -mavx512vl -mavx512ifma -mavx512vbmi|  

## git 问题汇总  

1. git reset 和 git revert的区别    
git reset --hard commit 记录回退 代码回退    
git reset --soft 仅仅是commit回退 其余均不改变 代码不回退    
git revert 撤销某次提交， 代码也随之更新， 可以保留git操作记录   
2. git中fork出来的project与原project保持同步    
git remote add upstream XXXURL    
git remote update upstream    
git rebase upstream/(branch)    

## python问题汇总  

1. python gdb   
进入指定的python 虚拟环境   
Export python的环境变量：export PYTHONHOME=/global/panfs01/users1/wangwei3/private-neon-SKX-20C/.venv2/   
进入gdb环境，附上可执行程序以及文件参数  gdb --args python main.py(也可以不加文件参数， 在run的时候指定文件)   
Gdb使用   
设置断点break func  break file:lineNo   
运行  run    
查看断点 info breakpoints   
删除断点delete  breakpoints No   
