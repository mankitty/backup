---
title: Linux环境（Centos 6.5）编译gcc-4.9.4
date: 2019-04-18 16:42:19
tags:
---
## Compiler Environment
yum groupinstall "Development Tools"
## Download source package
### Download gcc-4.9.4
wget http://ftp.gnu.org/gnu/gcc/gcc-4.9.4/gcc-4.9.4.tar.bz2
### Download mpc-1.0.3
wget http://ftp.vim.org/languages/gcc/infrastructure/mpc-1.0.3.tar.gz
### Download mpfr-3.1.4
wget http://ftp.vim.org/languages/gcc/infrastructure/mpfr-3.1.4.tar.bz2
### Download gmp-4.3.2
wget http://ftp.vim.org/languages/gcc/infrastructure/gmp-4.3.2.tar.bz2
### Download cloog-0.18.1
wget http://ftp.vim.org/languages/gcc/infrastructure/cloog-0.18.1.tar.gz
### Download isl-0.14
wget http://ftp.vim.org/languages/gcc/infrastructure/isl-0.14.tar.bz2
### 百度云下载
链接：https://pan.baidu.com/s/1_itD8YW8Ox0MhjvNHC4nfg 
提取码：og4p
## Compile
```
1. unzip gcc-4.9.4.tar.bz2 and create the gcc-build-4.9.4 folder
2. unzip mpc-1.0.3.tar.gz, mpfr-3.1.4.tar.bz2, gmp-4.3.2.tar.bz2, cloog-0.18.1.tar.gz, isl-0.14.tar.bz2 to gcc-4.9.4 and carried out（./configure && make -j8 && make install） for all packet
3. return to gcc-4.9.4 and carried out ../gcc-4.9.4/configure --prefix=/usr/local/gcc  --enable-bootstrap  --enable-checking=release --enable-languages=c,c++ --disable-multilib and make -j16 && make install
```
## question
configure: error: cannot compute suffix of object files: cannot compile
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/root/tools/gcc-4.9.4/mpc-1.0.3/mpc_install/lib:/root/tools/gcc-4.9.4/gmp-4.3.2/gmp_install/lib:/root/tools/gcc-4.9.4/mpfr-3.1.4/mpfr_install/lib
LD_LIBRARY_PATH
```
## view version
gcc -v g++ -v or gcc --version g++ --version

## 编译参数说明
--prefix=/usr/local/ 指定安装路径
--enable-bootstrap 这里引用网上一些文献对该参数的解释：用第一次编译生成的程序进行第二次编译，然后用再次生成的程序进行第三次编译，并且检查比较第二次和第三次结果的正确性，也就是进行冗余的编译检查工作。 非交叉编译环境下，默认已经将该值设为enable，可以不用显示指定；交叉编译环境下，需要显示将其值设为 disable。--enable-checking=release以软件发布版的标准来对编译时生成的代码进行一致性检查；设置该选项为enable并不会改变编译器生成的二进制结果，但是会导致编译的时间增加；该选项仅支持gcc编译器； 总体而言，对于上面这个选项，机器的硬件配置较低，以及不愿等待太久编译时间的童鞋，可以设置为 disable；但是这会增加产生未预期的错误的风险，所以应该慎用。可以同时设置--disable-bootstrap 与 --disable-checking，这对编译过程的提速很有帮助。--enable-threads=posix顾名思义，启用posix标准的线程支持，要让程序能在符合POSIX规范的linux发布版上正确运行，就应该启用该选项，取决于宿主或目标操作系统的类型，其它可用值有：aix，dec，solaris，win32等，如果你是其它的类UNIX系统，就需要设置相应的值。--enable-languages=c,c++支持的高级语言类型和运行时库，可以设置的所有语言包括 ada,c,c++,Fortran,java,objc,obj-c++,GO等语言。这里只开启了c和c++,因为支持的语言越多，就需要安装越多的相应静态与动态库，还有五花八门的依赖库，这会让管理变得困难，体积也会变得庞大。--disable-multilib 如果你的操作系统是32位，默认就已经设置为disable，这意味着gcc仅能生成32位的可执行程序；如果你的操作系统是64位，默认就已经设置为enable，这意味着用gcc编译其它源文件时可以通过 -m32 选项来决定是否生成32位机器代码。如果在64位系统上，要禁止生成32位代码， 设置 --disable-multilib。--enable-gather-detailed-mem-stats允许收集详细的内存使用信息，如果设置该参数为enable，则将来编译好的gcc可执行程序，可以通过-fmem-report选项来输出编译其它程序时的实时内存使用情况。--with-long-double-128 指定 long double 类型为128位（16字节！）；设置为without，则 long double类型将为64位（8字节），这将与普通的double类型一样。基于Glib2.4以上版本编译时，默认已经是128位。
## 后续操作
若想使用多个gcc版本，则需要执行以下步骤，否则，直接覆盖掉原gcc即可
### 编辑gcc.sh
打开/etc/profile.d/gcc.sh文件并添加export PATH=/usr/local/gcc/bin:$PATH，执行source /etc/profile.d/gcc.sh
### 导出头文件
ln -sv /usr/local/gcc/include/ /usr/include/gcc "/usr/include/gcc" -> "/usr/local/gcc/include/"
### 导出库文件
编辑gcc.conf，打开/etc/ld.so.conf.d/gcc.conf文件，添加/usr/local/gcc/lib64， ldconfig -p |grep gcc  验证是否导出 

