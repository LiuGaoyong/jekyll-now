---
layout: post
title:  "VASP-5.4.1 的安装（基于Ubuntu 16.04编译环境）"
date:   2018-01-01 13:48:16 +0800
categories: note
---



---

##  一、Intel_Parallel_Studio_XE
### 1. **安装包下载地址（推荐`IPS2015`）**


<table border="1">
  <tr>
    <th>软件版本</th>
    <th>百度云链接</th>
    <th>密码</th>
  </tr>
  <tr>
    <th>2011</th>
    <th>http://pan.baidu.com/s/1nv4PYOx</th>
    <th>59tu</th>
  </tr>
  <tr>
    <th>2013</th>
    <th>http://pan.baidu.com/s/1o6sPd8m </th>
    <th>无</th>
  </tr>
  <tr>
    <th>2015</th>
    <th>https://pan.baidu.com/s/1o8U6yvw</th>
    <th>无</th>
  </tr>

</table>

### 2. **安装步骤：**
- 进入解压后的目录，修改权限

```bash
cd /home/当前用户/安装包/
chmod -R +rwx /home/当前用户/安装包/
```

- 运行安装脚本 `./install.sh`
- 一直敲回车 
- 到 view license，一路空格，最后==输入accept==然后回车
- 到 Alternative activation, 选择 use a license file, provide the full path, 输入：目录/lic文件名 （具体内容可能有出入）
- 可选择Typical Install全部安装，或只安装inter fortran composer, 安装包具体内容可参考说明文件。
- 目录（`默认/opt/intel`）已存在，因为里面放了刚才的lic文件，所以无所谓，overwrite yes。后面省略，安装完成。
- 加入环境变量

```bash
vim ~/.bashrc

# add the following
source /opt/intel/bin/ifortvars.sh intel64
source /opt/intel/mkl/bin/mklvars.sh intel64
```

##  二、openMPI

### **1. 需要的安装包（[官网](http://www.open-mpi.org/)）**

### **2. 安装步骤：**
- 解压openMPI安装包 `tar zxcf 文件名`
- 编译，终端输入
```bash
cd 
mkdir openmpi_compiled
sudo mv openmpi_compiled /opt
./configure --prefix=/opt/openmpi_compiled F77=ifort FC=ifort \
        #CC=icc CXX=icpc F77=ifort FC=ifort 若不加，则用gcc编译
make -j8 #采用八核心加快编译
make install
```

- 添加以下内容到 `.bashrc`文件
```bash
# openMPI setup
export MPI_HOME=/hoopt/openmpi_compiled
export PATH=$PATH:$MPI_HOME/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$MPI_HOME/lib
```


### **3. 测试mpi效果：**

- 测试环境变量

```bash
echo $PATH
echo $LD_LIBRARY_PATH

#结果中显示有刚才的bin和lib路径则为配置成功
```

- 测试`mpirun`

```bash
which mpirun
mpif90 -v #检查版本

#结果中显示 /opt/openmpi_compiled/bin/mpirun 则为配置成功
```


- 测试`mpi`代码 （使用普通账户）

```bash
cd /openmpi源代码目录/examples
make
mpirun -np 2 hello_c  #2为双核

#结果中显示以下内容则为配置成功
#   Hello, world, I am 0 of 2
#   Hello, world, I am 1 of 2
```

##  三、编译VASP
### 1. **需要的安装包**（[我的GitHub仓库](https://github.com/LiuGaoyong/VASP-Compile)）

### 2. **测试环境变量**

- #### **查看Intel_compiled环境**

```bash
which ifort    #Fortran编译器
echo $MKLROOT  #MKL环境
which mpiifort #MPI环境：. /opt/intel/impi/5.0.2.044/bin64/mpivars.sh intel64
```

- #### **查看Intel_fftw环境**

```bash
ll $MKLROOT/interfaces/fftw3xf/libfftw3xf_intel.a

# 如没存在，那么可以在自己目录下编译生成libfftw3xf_intel.a：
cd /opt/intel/mkl/interfaces/
cp -a fftw3xf ~/
sudo cp -a fftw3xf fftw3xf_bkp
cd ~/fftw3xf
make libintel64
#顺利的话，将在此目录下生成libfftw3xf_intel.a。
sudo cp -a ~/fftw3xf /opt/intel/mkl/interfaces
```


### 3. **vasp源代码整理**
- #### **解压**

```bash
cd
tar zxvf vasp源码包
tar zxvf vasp.lib包
```

- #### **整合VTST（可选）**

##### **a. 源代码中加入VTST：**

```bash
cd vasp.X.X.X # 解压缩后的根目录
wget http://theory.cm.utexas.edu/code/vtstcode.tgz
tar xvf vtstcode.tgz
cp vtstcode-XXX/* src/
```

##### **b. 修改src/main.F源码：**

```Fortran
!修改约第3233行
!CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
!     LATT_CUR%A,LATT_CUR%B,IO%IU6)
!变为（注意最后一行）：

CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
      TSIF,LATT_CUR%A,LATT_CUR%B,IO%IU6)
```

##### **c. 修改编译配置src/.objects：**

```makefile
#在chain.o前（大概第67行）添加如下内容：

bfgs.o dynmat.o instanton.o lbfgs.o sd.o cg.o dimer.o bbm.o \
fire.o lanczos.o neb.o qm.o opt.o \

#注意\后不能有空格！！！
```
### 3. **vasp正式编译**

```bash
cp arch/makefile.include.linux_intel makefile.include
make   #默认编译（将编译生成std、gam和ncl版）
#编译完成后将在bin目录下生成对应的版本的可执行程序：vasp_gam、vasp_ncl、vasp_std
```












  
#### **参考文章网址**
- <a href="http://blog.csdn.net/txcokokok/article/details/42219099">linux ubuntu12.04 下的 vasp 5.2 的安装方法</a>


-  <a href="http://bbs.keinsci.com/thread-4267-1-1.html">教你从头编译vasp-5.4.1</a>

- <a href="http://hmli.ustc.edu.cn/doc/app/vasp.5.4.1-vtst.htm">VASP 5.4.1+VTST编译安装</a>

