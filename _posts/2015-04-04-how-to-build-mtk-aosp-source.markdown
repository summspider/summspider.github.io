---
layout: post  
category: "mtk开发"  
title:  "[MTK][AOSP] 如何编译 MTK AOSP 版本代码"  
tags: [mtk AOSP 编译]  
summary: "MTK AOSP 版本的编译方法"  
---

## 引言

自从L版本开始，MTK 的代码结构和编译方式和以前大不一样，以前mtk是自己设计了一套编译架构，现在则是更加贴近 android 默认的方式。

虽然有改变，但是在 mtk release 的代码包里面依然包含了以前的 preloader/lk/kernel 的代码。所以和 AOSP 还是会有不同的地方。

下面就以编译 sample_project (工程的名字) 的工程版本为例来讲一下如何编译代码。

## 1. Full Build

依次执行如下 command 序列：

1.  source build/envsetup.sh
2.  lunch full_sample_project-eng
3.  make -j32 2&gt;&amp;1 | tee full_build.log

其中第一步主要是用来 setup 编译环境，第二步是加载工程的一些信息，第三步执行具体的编译。第三步里面可以用-j来指定编译时候的线程数量，2&gt;&amp;1 用来把标准错误重定向到标准输出。tee 指令用来复制标准输出到 log 文件。

## 2. Build Android Module (Module Name 方式)

在编译 module 之前首先要执行下面的两个命令（如果在当前shell执行过可以不用重复执行）

1.  source build/envsetup.sh
2.  lunch full_sample_project-eng

之后就可以用 module name 的编译方式，命令如下：

*   make -j32 &lt;module name&gt;

例如：

*   make -j32 libc

在 Android 源码树里面有很多的 module 声明在 Android.mk 文件里面。在 MTK ASOP 版本里面另外还设计了几个特殊的 Module。下面逐一介绍。

### 2.1 Preloader 编译 (Module方式)

Preloader 是代码包里面一个特殊的 Module，源码位置在 bootable/bootloader/preloader 目录。Module 的名字就是 pl，所以 preloader 可以用下面的方式编译：

*   make -j32 pl

注意要先执行 module 编译的前面两步。

### 2.2 LK 编译 (Module方式)

LK, 全称 Little Kernel, 是系统第二个 bootloader，源码位置在 bootable/bootloader/lk 目录。Module 的名字是 lk，所以 lk 可以用下面的命令来编译：

*   make -j32 lk

注意要先执行 module 编译的前面两步。

### 2.3 Linux Kernel 编译 (Module方式)

类似的, Linux Kernel 也被设计成一个 Android Module，源码位置在 kernel-3.10 目录，将来也有可能会换为更新的版本。Module 的名字是kernel，所以linux kernel可以作为一个 android module 用下面的方式来编译：

*   make -j32 kernel

注意要先执行 module 编译的前面两步。

## 3. Build Android Module (Module PATH 方式)

前面讲了用 Module Name 的方式来 build android module，但是由于 build system 会从所有的 Android.mk 里面找对应的 module，整个扫描的时间是较长的。对于一个常规的 Android Module 来说，这并不是最优的方式。相对来讲，采用指定 Module 路径的方式来编译效率会更高。

同样在编译前要先执行下面两条命令：

1.  source build/envsetup.sh
2.  lunch full_sample_project-eng

然后执行下面的mmm命令：

*   mmm your/module/path/

以libc的编译为例，可以执行下面的命令：

*   mmm bionic/libc/

## 4. 打包 Image

系统里面有几个image是需要打包起来，如 boot image，system image 等等，下面讲述这几个 image 的打包命令。

在打包前首先要执行下面两条命令：

1.  source build/envsetup.sh
2.  lunch full_sample_project-eng

### 4.1 bootimage 打包

bootimage 由两个部分组成，一是压缩的 kernel image，一是 rootfs，所以在打包之前要确 保rootf s下面的文件已经被编译出来，最好（务必）是做过一次 full build。

而 bootimage 的打包命令有两种，一种会检查依赖关系，一种只是单纯的打包。参考下面的说明。

#### 4.1.1 make bootimage

make bootimage 命令会检查依赖关系，在必要的情况下会先编译变化的文件然后再做打包的动作。比如修改了 kernel 的源码，或者修改了 rootfs 里面 adbd 的源码，都会编译对应的源文件并链接，然后才会去打包

*   make bootimage

#### 4.1.2 make bootimage-nodeps

顾名思义，nodeps 的意思就是 no dependency check。这条命令不会做任何的依赖关系检查，仅仅是单纯的将 out/target/product/sample_project/ 下面的 kernel 镜像和 root 文件夹打包成 boot.img

*   make bootimage-nodeps

### 4.2 system image 打包

System image 的打包命令也有两种，一种会检查依赖关系，一种只是单纯的打包。

#### 4.2.1 make systemimage

make systemimage 命令会检查 system image 里面所有目标文件的依赖关系，如果目标文件的依赖有更新，则会根据 Android.mk 里面的规则更新相应的目标文件，在依赖检查之后再打包 system image。

*   make systemimage

#### 4.2.2 make snod

同样，make snod 是 system image with no dependency check的意思。所以 make snod 是不会检查依赖关系的。仅仅是将 out/target/product/sample_project/system 目录打包成 system.img 而已。

*   make snod

### 4.3 Cache Image 打包

Cache image 的打包命令也有两种，一种会检查依赖关系，一种只是单纯的打包。

#### 4.3.1 make cacheimage

make cacheimage 命令会检查 out/target/product/sample_project/cache 目录下所有目标文件的依赖关系，如果目标文件的依赖有更新，则会根据 Android.mk 里面的规则生成相应的目标，然后再打包 cache.img

*   make cacheimage

#### 4.3.2 make cacheimage-nodeps

make cacheimage-nodeps 则仅仅是将 out/target/product/sample_project/cache 打包成cache.img，而不会做依赖关系的检查。

*   make cacheimage-nodeps

### 4.4 userdata image 打包

userdata image 的打包命令也有两种，一种会检查依赖关系，一种只是单纯的打包。

#### 4.4.1 make userdataimage

make userdataimage 命令会检查 out/target/product/sample_project/data 目录下的所有目标文件的依赖关系，如果目标文件的依赖有更新，则会根据 Android.mk 里面的规则生成相应的目标，然后再打包 userdata.img

*   make userdataimage

#### 4.4.2 make userdataimage-nodeps

make userdataimage-nodeps 则仅仅是将 out/target/product/sample_project/data 打包成 userdata.img，而不会做依赖关系的检查。

*   make userdataimage-nodeps

## 5 preloader/lk/kernel 独立编译

由于 preloader/lk/kernel 和 Android source code 从逻辑上都是独立的，仅仅是将 source code 摆放在了一起而已。所以把 preloader/lk/kernel 单独拿出来也是可以独立编译的。而且独立编译相对于把 preloader/lk/kernel 作为 android module 来编译的速度更快。

### 5.1 环境准备

独立编译的时候不需要执行android编译的流程，也不需要lunch project，甚至 preloader/lk/kernel 的代码也不需要放在 android 的 source tree 里面。但是要在 PATH 环境变量里面设置好 toolchain 的路径。而工具链在 android 的代码包里面就已经内置了 prebuilt 的交叉编译工具链。

下面以 ${CROOT} 表示 android source tree 的根目录。

*   对于 32bit 的编译，toolchain 的路径为 ${CROOT}/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.8/bin/

*   对于 64bit 的编译，toolchain 的路径为 ${CROOT}/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/

需要注意的是，随着 android 版本的变化 toolchain 的版本也在变化，所以在你的 package 里面可能看到的 toolchain 版本是 4.8 或者 4.9，或者是其他的一些版本。注意具体的路径即可。

PATH 通常可以在 shell 里面手动设定，举例如下：

*   PATH=/你的anroid代码根目录/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.8/bin/:/你的anroid代码根目录/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/:$PATH

注意上面 PATH 设定的时候正确设置你的anroid代码根目录。设定之后就可以单独来编译 preloader/lk/kernel 了。

### 5.1 preloader 独立编译

编译步骤如下：

*   **cd bootable/bootloader/preloader** (此步骤仅仅是切到到你的 preloader 路径，如果 preloader 代码没有放置在 android source tree 里面的话需要切换到对应的位置)
*   **TARGET_PRODUCT=sample_project ./build.sh 2&gt;&amp;1 | tee build_pl.log** (此步骤进行编译操作，对应 Project 的名字通过 TARGET_PRODUCT 变量来指定)

注意，这种方式编译出来的 preloader bin 文件是在 preloader 的根目录，而不是 out/target/product/sample_project 目录下，如果要下载的话请选择正确的 preloader_sample_project.bin 文件来下载。

### 5.2 lk 独立编译

编译步骤如下：

*   **cd bootable/bootloader/lk** (此步骤仅仅是切到到你的 lk 路径，如果 lk 代码没有放置在 android source tree 里面的话需要切换到对应的位置)
*   **make sample_project 2&gt;&amp;1 | tee build_lk.log** (此步骤进行编译操作，对应的 project 名字通过命令行传给 make 命令)

注意，这种方式编译出来的 lk bin 文件是在 lk 根目录下的 build-sample_project 目录下，而不是 out/target/product/sample_project 目录下，如果要下载的话请选择正确的 lk.bin 文件来下载。

### 5.3 kernel 独立编译

编译步骤如下：

*   **cd kernel-3.10** (此步骤仅仅是切到到你的 kernel 代码路径，如果 kernel 代码没有放置在 android source tree 里面或者 kernel 版本不一样的话需要切换到对应的位置)
*   **mkdir out ** (此步骤建立一个临时的路径，用于存放编译的中间文件以及结果)
*   **make sample_project_debug_defconfig** (此步骤用于选择要编译的 project 配置，通常以 _debug_defconfig 结尾的用于 eng 版本，仅以 _defconfig 结尾的用于 user 版本)
*   **make -j32 2&gt;&amp;1 | tee build_kernel.log** (此步骤是 kernel 的完整编译)

注意，这种方式编译出来的 kernel 仅是压缩后的 image (有可能会添加了dtb在后面)，生成的 image 放在 out/arch/arm64/boot/ 或者 out/arch/arm/boot/ 目录下面，取决于编译的是 32bit 内核还是 64bit 内核。image 的名字为 Image.gz-dtb。

由于生成 boot.img 的时候使用的是 out/target/product/sample_project/kernel 文件，所以在打包的时候还需要将编译好的 Image.gz-dtb copy 为 out/target/product/sample_project/kernel。(最新的平台都是支持这种方式的，老的平台像 mt6582 或者 mt6752/mt6732 则都需要添加一个 mtk 特有的 header，可以到 MTK 的 FAQ 网站上面找一下对应的方法。)

## 6 clean

make 是很智能的，它可以根据依赖关系找出哪些目标和中间文件需要更新，根据 makefile 中的规则自动执行相应的操作。这种编译方式在我们仅仅改动了部分文件的时候不用完整的编译所有 source，仅仅更新对应的代码，会节省很多时间。所以大多数的时候都不推荐去做 clean 操作。

但是也有些例外，有些时候是必须要做 clean 操作的，主要是：

1.  修改了 kernel defconfig 或者 kconfig 规则的时候要对 kernel 做 clean 操作。
2.  修改了 ProjectConfig.mk 文件的时候要对 android 的部分做 clean。
3.  任何依赖关系的改动导致 out 目录保存有本应删除的中间文件的时候就需要做clean。

另外，如果你不确定你的改动有什么影响，但是编译出来的 image 运行的时候不符合预期又看不出原因的时候最好也做一下clean。

### 6.1 常用 clean 命令

运行 clean 相关的命令之前请先执行下面两个步骤：

1.  source build/envsetup.sh
2.  lunch

如下是一些常用的 clean 命令：

*   make clean (对整个项目做clean操作)
*   make clean-pl (对preloader做clean操作，通常不用单独去做，编译prelaoder的时候会自动先做clean)
*   make clean-lk (对lk做clean操作)
*   make clean-kernel (对kernel做clean操作)

### 6.2 kernel 单独编译时的 clean 操作

前面有讲kernel的单独编译，在编译的时候有建立临时目录 out，clean 操作有下面几种（不做详细解释，详细请google）

*   make O=out clean
*   make O=out mrproper
*   make O=out distclean

## 7 编译命令参数

有一些命令行参数可以在编译的时候带进去，下面简单做一些介绍。

### 7.1 -B 参数

android 在编译的时候可以加入 -B 参数，用于强制重新编译。例如：

*   mmm -B external/sepolicy

### 7.2 V=1 参数

在编译的时候可以用 V=1 参数将 gcc 编译时候的 verbose 信息打印出来。在遇到编译错误的时候打印详细的信息对分析编译问题有帮助。

举例如下：

*   make V=1 -j32 kernel
*   mmm -B V=1 bionic/libc

**-- THE END --**