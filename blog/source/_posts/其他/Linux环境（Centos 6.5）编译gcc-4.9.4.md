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