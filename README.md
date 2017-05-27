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
