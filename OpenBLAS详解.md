# OpenBLAS详细分析

## 1.相关链接

[OpenBLAS An optimized BLAS library](http://www.openblas.net/)

[OpenBLAS github](https://github.com/xianyi/OpenBLAS)

[BLAS、OpenBLAS、ATLAS、MKL](https://blog.csdn.net/fuhanghang/article/details/122885265)

## 2.OpenBLAS简介

### OpenBLAS

OpenBLAS is an optimized Basic Linear Algebra Subprograms (BLAS) library based on [GotoBLAS2](https://www.tacc.utexas.edu/research-development/tacc-software/gotoblas2) 1.13 BSD version.（OpenBLAS是高性能多核BLAS库，是GotoBLAS2 1.13 BSD版本的衍生版。）

Openblas在编译时根据目标硬件进行优化，生成运行效率很高的程序或者库。Openblas的优化是在编译时进行的，所以其运行效率一般比atlas要高。但这也决定了**openblas对硬件依赖性高，换了机器，可能就要重新编译了**。（例如A和B两台机器cpu架构、指令集不一样，操作系统一样，在A下编译的openblas库，在B下无法运行，会出现“非法指令”这样的错误）

**注意：`OpenBLAS` 是`BLAS`标准的一种具体实现，起源于`GotoBLAS`**

**注意：`OpenBLAS` 的编译依赖系统环境，并且没有原生单线程版本，即不支持单核计算加速，通过设置 `OMP_NUM_THREADS=1` 来模拟单线程版本，可能会带来一点点的性能下降。**

**注意：OpenBLAS不支持GPU**

### 类似运算库

#### BLAS

BLAS的全称是Basic Linear Algebra Subprograms，中文可以叫做**基础[线性代数](https://so.csdn.net/so/search?q=线性代数&spm=1001.2101.3001.7020)子程序**。它定义了一组应用程序接口（[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)）标准，是一系列初级操作的规范，如向量之间的乘法、矩阵之间的乘法等。许多数值计算软件库都实现了这一核心。

BALS是用[Fortran](https://so.csdn.net/so/search?q=Fortran&spm=1001.2101.3001.7020)语言开发的，Netlib实现了BLAS的这些API接口，得到的库也叫做BLAS。Netlib只是一般性地实现了基本功能，并没有对运算做过多的优化。

#### LAPACK

LAPACK （linear algebra package），是著名的**线性代数库**，也是Netlib用[fortran](https://so.csdn.net/so/search?q=fortran&spm=1001.2101.3001.7020)语言编写的。其底层是BLAS，在此基础上定义了很多矩阵和向量高级运算的函数，如矩阵分解、求逆和求奇异值等。该库的运行效率比BLAS库高。
从某个角度讲，LAPACK也可以称作是一组科学计算（[矩阵](https://so.csdn.net/so/search?q=矩阵&spm=1001.2101.3001.7020)运算）的接口规范。Netlib实现了这一组规范的功能，得到的这个库叫做LAPACK库。

Linear Algebra Package，[线性](https://so.csdn.net/so/search?q=线性&spm=1001.2101.3001.7020)代数包，底层使用BLAS，使用Fortran语言编写。在BLAS的基础上定义很多矩阵和向量高级运算的函数，如矩阵分解、求逆和求奇异值等。该库的运行效率比BLAS库高。为了进行C语言的开发，开发了CBLAS和CLAPACK。

#### CBALS & CLAPACK

前面BLAS和LAPACK的实现均是用的Fortran语言。为了方便c程序的调用，Netlib开发了CBLAS和CLAPACK。其本质是在BLAS和LAPACK的基础上，增加了c的调用方式。

#### ATLAS

上面提到，可以将BLAS和LAPACK看做是接口规范，那么其他的组织、个人和公司，就可以根据此规范，实现自己的科学计算库。

开源社区实现的科学计算（矩阵计算）库中，比较著名的两个就是atlas和openblas。它们都实现了BLAS的全部功能，以及LAPACK的部分功能，并且他们都对计算过程进行了优化。

Atlas （Automatically Tuned Linear Algebra Software）能根据硬件，在运行时，自动调整运行参数。

Automatically Tuned Linear Algebra Software，自动化调节线性代数软件。可以根据硬件，在运行时，自动调整运行参数。

#### Intel MKL

英特尔数学核心函数库，Intel Math Kernel Library

Intel的MKL和AMD的ACML都是在BLAS的基础上，针对自己特定的CPU平台进行针对性的优化加速。以及NVIDIA针对GPU开发的cuBLAS。

商业公司对BLAS和LAPACK的实现，有Intel的MKL和AMD的ACML。他们对自己的cpu架构，进行了相关计算过程的优化，实现算法效率也很高。
NVIDIA针对其GPU，也推出了cuBLAS，用以在GPU上做矩阵运行。

#### Eigen

Eigen是一个C++语言中的开源的模板库，支持线性代数的运算，包括向量运算，矩阵运算，数值分析等相关算法。因为eigen只包含头文件，所以使用的话不需要进行编译，只需要在cpp文件开头写`#include <Eigen>`就好。

#### Armadillo

Armdillo 矩阵运算速度跟 MATLAB 一个量级，为目前使用比较广的 C++ 矩阵运算库之一，是在 C++ 下使用 MATLAB 方式操作矩阵很好的选择，许多 MATLAB 的矩阵操作函数都可以找到对应，这对习惯了Matlab的人来说实在是非常方便，另外如果要将 MATLAB 下做研究的代码改写成 C++，使用 Armadillo 也会很方便，这里有一个简易的 MATLAB 到Armadillo 的语法转换。

**注意：**`Eigen`和`Armadillo`不仅可以使用自身的线性代数实现，还可以通过配置指定上述不同的BLAS/LAPACK作为底层。

![](./%5C1.png)

对于机器学习的很多问题来说，计算的瓶颈往往在于**大规模以及频繁的矩阵运算**，主要在于以下两方面：

- (Dense/Sparse) Matrix – Vector product  (稠密/稀疏)矩阵 - 向量 产品
- (Dense/Sparse) Matrix – Dense Matrix product 

如何使机器学习算法运行更高效摆在我们面前，很多人都会在代码中直接采用一个比较成熟的矩阵运算数学库，面对繁多的数学库，选择一个合适的库往往会令人头疼，这既跟你的运算环境有关，也跟你的运算需求有关，不是每个库都能完胜的。

## 3.OpenBLAS下载

```shell
$ git clone https://github.com/xianyi/OpenBLAS.git
```

## 4.OpenBLAS源码分析

OpenBLAS的根目录如下所示

```shell
OpenBLAS/  
├── benchmark      Benchmark codes for BLAS
├── cmake          CMakefiles
├── ctest          Test codes for CBLAS interfaces
├── driver         Implemented in C
│   ├── level2
│   ├── level3
|   ├── mapper
│   └── others     Memory management, threading, etc
├── exports        Generate shared library
├── interface      Implement BLAS and CBLAS interfaces (callinkernel)
│   ├── lapack
│   └── netlib
├── kernel         Optimized assembly kernels for CPU architectures
│   ├── arm64
│   ├── generic    General kernel codes written in plain C.
│   ├── mips
│   ├── x86_64
│   └── ...  
├── lapack          Optimized LAPACK codes (replacing those iPACK)
│   ├── getf2
│   └── ...
├── lapack-netlib   LAPACK codes from netlib reference implementation
├── reference       BLAS Fortran reference implementation (unused)
├── relapack        Elmar Peise's recursive LAPACK (implementeregular LAPACK)
├── test            Test codes for BLAS
└── utest           Regression test
```

OpenBLAS整个项目工程文件放置的并不友好，BLAS 相关的源码主要放在 `driver, interface, kernel` 三个目录下。

### driver文件夹

Driver 层的作用是具体实现，比如 gemm 的 driver 层就是矩阵乘法，由于要实现一个高效率的矩阵乘法，不会只是简单的三层 for 循环就解决了。这一层会做一些**任务切分，数据重排，以达到多核并行以及高效利用 cache 来提升 CPU 利用率的目的**。

### interface文件夹

Interface 层的主要作用是根据接口的参数，指定要执行的分支，如我们上图中的例子，走的是 dgemm_nn，即数据是 double 类型，矩阵都不转置。

### kernel文件夹

**kernel 层是核心运算，这里操作的数据已经都在 cache 上，通过向量化操作，也就是单指令多数据，来高效利用 CPU 的计算能力**。
这里的实现要高效，需要使用一些汇编或者是 intrinsic 指令，所以针对不同的 CPU 平台，需要单独实现。不过通用版本的实现一定会有，以支持暂时还没有优化的平台。

软件的调用层次如下图所示：

<img src="./%5C2.png" style="zoom:67%;" />

以 gemm 为例，调用关系如下。

```c++
interface/gemm.c
        │
driver/level3/level3.c
        │
gemm assembly kernels at kernel/
```

具体采用的哪个 kernel 查看预先写好的配置文件 ./kernel/$(ARCH)/KERNEL.$(CPU)。

图下图

![](./%5C3.png)

我这个 case 就是 KERNEL.HASWELL，其中的配置如下：

```c++
DGEMMKERNEL    =  dgemm_kernel_4x8_haswell.S
```

