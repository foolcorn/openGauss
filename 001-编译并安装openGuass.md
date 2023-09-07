## 001-编译并安装openGuass

### （一）前言

目的是为了尝试一下从数据库源码编译到可执行文件的过程，可以对编译过程，编译工具等有更深的了解。

### （二）版本选择

openGauss 2.0.0，相较于openGauss2.1.0（openGauss3.0.0的前瞻版本）更加稳定，功能更简单，适合入门学习。

### （三）操作系统

CentOS Linux release 7.6.1810 (Core)

### （四）软件依赖要求

使用yum工具安装以下相关软件包，安装特定版本软件包的命令如下：

```shell
yum install -y <package_name>-<version>
```

| 所需软件           | 建议版本                  |
| :----------------- | :------------------------ |
| libaio-devel       | 建议版本：0.3.109-13      |
| flex               | 要求版本：2.5.31 以上     |
| bison              | 建议版本：2.7-4           |
| ncurses-devel      | 建议版本：5.9-13.20130511 |
| glibc-devel        | 建议版本：2.17-111        |
| patch              | 建议版本：2.7.1-10        |
| redhat-lsb-core    | 建议版本：4.1             |
| readline-devel     | 建议版本：7.0-13          |
| pam-devel          | 建议版本：1.1.8-1.3.1     |
| libffi-devel       | 建议版本：3.1             |
| python3            | 建议版本：3.6             |
| python3-devel      | 建议版本：3.6             |
| libtool            | 建议版本：2.4.2及以上     |
| libtool-ltdl-devel | 建议版本：2.4.2及以上     |
| openssl-devel      | 建议版本：1.1.1           |

**注：坑0：请参考以上的环境表，建议不用再参考官方文档或者gitee的依赖，因为很多包名已经过时。**

如果是面对一个现有的开发环境，通过yum list命令配合grep可以查询是否已经安装某个包，并确认版本：

```shell
yum list installed | grep libaio-devel
```

如图说明在安装列表里找不到对应的包

![image-20230717143836146](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230717143836146.png)

为了方便我写了一个shell脚本，可以一次性检查所有软件包。

```shell
#!/bin/bash

# 软件包名称数组
packages=("libaio-devel" "flex" "bison" "ncurses-devel" "glibc-devel" "patch" "redhat-lsb-core" "readline-devel" "pam-devel" "libffi-devel" "python3" "python3-devel" "libtool" "libtool-ltdl-devel" "openssl-devel")
# 遍历软件包数组
for package in "${packages[@]}"
do
    result=$(yum list installed | grep "$package")
    if [ -n "$result" ]; 
    then
    	echo "$package 已安装"
    	echo "$result" # 打印版本号
    else
    	echo "$package 未安装"
	fi
done
```

新建一个xxx.sh文件，复制上述内容后，执行sh xxx.sh命令即可

查询结果如图：

![image-20230720104958889](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230720104958889.png)



### （五）编译流程图

总体的编译流程如图所示：

<img src="http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230717110111633.png" alt="image-20230717110111633" style="zoom:67%;" />

##### 说明：

- openGauss-server：openGauss的主代码，需要提前下载备用。
- openGauss-third_party：openGauss依赖的开源第三方软件源码。可以有两种选择，一种是下载源码，后自己编译成Object文件（目标文件），留待后续和主代码一起链接。另一种则是直接使用官方提供的已经构建好的目标文件库binarylibs。**本文重在实践，故选择自己编译第三方库。**
- binarylibs：存放编译构建好的开源第三方软件的文件夹。

### （六）编译流程

#### （1）编译第三方库（若使用下载的binarylibs可跳过此步）

##### 1.准备代码

第三方软件源码 gitee地址：https://gitee.com/opengauss/openGauss-third_party/tree/2.0.0

**注：坑1: 不要用github，github仓库只有代码，依赖包是残缺的，必须要用gitee库**

<img src="http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230718142821627.png" alt="image-20230718142821627" style="zoom:80%;" />

**注：坑2: 最好不要直接在gitee上下载压缩包，难以保证是否缺少依赖或产生文件格式的问题，文件格式见坑3**

**注：坑3: 不要在Winodws环境下直接git clone**

Windows的文件格式和linux不一样，Windows的换行符是\r\n，Linux系列的换行符是\n。

在win上git clone出来的文件到Centos机上都会有问题，比如所有脚本都会报格式问题。

一开始我以为只需要将换行符转换即可，所以在centos上安装了dos2unix，将所有文件转换成linux格式

但是最后在编译过程中依旧会报错，因为dos2unix无法转换二进制文件，然而最后的主代码编译依赖于二进制的"configure"文件。

~~注：这里是之前的解决方式（已废弃），安装dos2unix~~ 

```shell
yum install -y dos2unix
```

~~将目录下所有文件的换行转成unix格式~~

```shell
find /目录路径 -type f -exec dos2unix {} \;
```

如果你非要在Windows上git clone，需要配置git

在git bash环境中使用以下指令：

```shell
git config --global core.autocrlf input
```

当Windows下设置core.autocrlf为input时，在clone会自动按linux格式处理换行问题

上面的做法并不保证100%的正确性，虽然我的数据库代码是这么clone的。

但我依旧建议读者在linux环境下进行git操作，最方便的做法就是在WSL中进行git操作。

使用以下命令用git克隆代码仓库（注意指定2.0.0分支）

```shell
git clone -b 2.0.0 https://gitee.com/opengauss/openGauss-third_party.git openGauss-third_party
```

![image-20230721111939455](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230721111939455.png)

文件大小3GB左右，由于需要上传到云桌面再copy到虚拟机，云桌面限制上传大小<200MB，必须分卷压缩。

**注：坑4: 最好不要相信Windows环境下的解压缩软件和分卷操作**

和坑3同理，Windows下的解压缩软件：360（国际版），winrar，banzip都有可能破坏文件的原有结构

比如linux下的软链接文件，被解压出来后就成了纯ascll文件。

造成后面编译的各种问题。

因此，对于git clone下来的源码，请在WSL子系统下（Mac电脑直接用shell环境）使用以下命令进行压缩。

```shell
tar -zcvf openGauss-third_party.tar.gz openGauss-third_party
```

然后将压缩包进行分割，-b指定压缩大小，我这里设置了190M，-a设置序列位数（默认为2），如果最后的分卷数不多可以设置为1

```shell
split -b 190M -d -a 1 openGauss-third_party.tar.gz openGauss-third_party.tar.gz.
```

最后压缩完的效果：

![image-20230721112807067](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230721112807067.png)

将分卷包上传到云桌面再copy到虚拟机，中途切记不能在Windows下进行加解压操作。

在Centos分卷包目录下执行以下命令进行解压：

```shell
cat openGauss-third_party.tar.gz.* | tar -zxv
```

![image-20230721120431400](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230721120431400.png)

##### 2.准备前置依赖

**注：坑5: 建议不用参考官方文档提供的前置软件列表**

![image-20230719130010133](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230719130010133.png)

很多包已经过时，查询不到或者package名称有误，请按**（四）软件依赖要求**中提供的列表安装依赖即可。

除了上述依赖，只需准备gcc7.3和cmake（官方建议版本需要>3.16）的编译安装：

在安装gcc7.3之前，需要先编译安装四个依赖：gmp、mpfr、mpc、isl库

对应的下载地址如表格所示：

| package | download address                                            |
| ------- | ----------------------------------------------------------- |
| gcc7.3  | http://ftp.gnu.org/gnu/gcc/gcc-7.3.0/                       |
| gmp     | http://ftp.gnu.org/gnu/gmp/gmp-6.1.1.tar.xz                 |
| mpfr    | http://ftp.gnu.org/gnu/mpfr/mpfr-4.0.2.tar.gz               |
| mpc     | http://ftp.gnu.org/gnu/mpc/mpc-1.1.0.tar.gz                 |
| isl     | https://gcc.gnu.org/pub/gcc/infrastructure/isl-0.18.tar.bz2 |

下载后按次序进行编译安装

**①编译安装gmp**

解压命令：

```shell
tar -xf gmp-6.1.1.tar.xz
```

解压后进入到目录中

执行以下指令：(prefix参数可以指定编译结果的存放路径，提前创建好对应的目录)

```shell
./configure --prefix=/root/compile/target/gmp
make –j
make install –j
```



**②编译安装mpfr**

解压命令：

```
tar –xf mpfr-4.0.2.tar.gz
```

解压后进入到目录中，执行以下指令：

```shell
./configure --prefix=/root/compile/target/mpfr --with-gmp=/root/compile/target/gmp
make –j
make install -j
```



**③编译安装mpc**

解压命令：

```
tar –xf mpc-1.1.0.tar.gz
```

解压后进入到目录中,执行以下指令：

```shell
./configure --prefix=/root/compile/target/mpc --with-gmp=/root/compile/target/gmp --with-mpfr=/root/compile/target/mpfr
make –j
make install -j
```

在编译mpc的过程中可能会报“在/root/compile/路径下找不到lib和libgmp.la”的错误

应该是--with-gmp=/root/compile/target/gmp没起到作用，libgmp.la应该在gmp编译输出的目录的lib目录下。

故手动cp了一份lib文件夹放在"/root/compile"路径下，再次make，发现成功了。



**④编译安装isl**

解压命令：

```shell
tar –xf isl-0.18.tar.bz2
```

解压后进入到目录中,执行以下指令：*注意参数和上面三个不同，为--with-gmp-prefix*

```shell
./configure --prefix=/root/compile/target/isl --with-gmp-prefix=/root/compile/target/gmp
make –j
make install -j
```



**⑤编译安装gcc**

将上述四个依赖项加入到配置文件中，配置环境变量（建议使用/etc/profile全局配置文件，而非/.bashrc这种用户级配置文件）

在“/etc/profile”的末尾添加以下内容

```shell
export COMPILE
export LD_LIBRARY_PATH=/root/compile/target/gmp/lib:/root/compile/target/mpfr/lib:/root/compile/target/mpc/lib:/root/compile/target/isl/lib:${LD_LIBRARY_PATH}
export C_INCLUDE_PATH=/root/compile/target/gmp/include:/root/compile/target/mpfr/include:/root/compile/target/mpc/include:/root/compile/target/isl/include:${C_INCLUDE_PATH}
export JAVA_HOME=/usr/java/jdk1.8.0_191
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export COMPILE=/root/compile/target
export LD_LIBRARY_PATH=$COMPILE/gmp/lib:$COMPILE/mpfr/lib:$COMPILE/mpc/lib:$COMPILE/isl/lib:${LD_LIBRARY_PATH}
export C_INCLUDE_PATH=$COMPILE/gmp/include:$COMPILE/mpfr/include:$COMPILE/mpc/include:$COMPILE/isl/include:${C_INCLUDE_PATH}
export NVIM_HOME=/opt/nvim
export PATH=$PATH:$NVIM_HOME/bin
```

使用命令激活环境变量

```shell
source /etc/profile
echo ${LD_LIBRARY_PATH} #为了确保环境变量激活可以执行该命令看是否有输出结果
```

首先使用yum安装一个基底的gcc-c++，用来编译gcc7.3

```
yum install -y gcc-c++
```

解压gcc7.3的包：

```
tar -zxvf gcc-7.3.0.tar.gz
```

解压后进入到目录中，执行以下指令：

```shell
./configure CFLAGS='-fstack-protector-strong -Wl,-z,noexecstack -Wl,-z,relro,-z,now ' --prefix=/root/compile/target/gcc --with-gmp=/root/compile/target/gmp --with-mpfr=/root/compile/target/mpfr --with-mpc=/root/compile/target/mpc --with-isl=/root/compile/target/isl --disable-multilib --enable-languages=c,c++
make –j
make install –j
```



**⑥编译安装高版本cmake**

下载Cmake：https://github.com/Kitware/CMake/releases

**注：坑6：建议选择3.16版本，亲测用了最新的3.27版本会报错**

解压命令：

```shell
tar -zxvf CMake-3.16.8.tar.gz
```

解压后进入到目录中

执行以下指令：(prefix参数可以指定编译结果的存放路径，提前创建好对应的目录)

```shell
./configure --prefix=/root/compile/cmake3.16   ##prefix为编译结果路径
make –j
make install -j
```

如果出现以下错误：

![image-20230720145025094](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230720145025094.png)

这说明系统默认的/lib64/libstdc++.so.6（这是个软链接）没有指向编译后gcc7.3的libstdc++.so，位置如下：

![image-20230720145653716](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230720145653716.png)

可以通过两种方式解决，一种是将gcc7.3的lib64加入到环境变量，另一种是重新链接libstdc++.so.6

**这里演示一下重新链接**

将图中标红位置（即你gcc --prefix填写的位置）中的libstdc++.so拷贝到/lib64目录(-f 是为了强制覆盖，如果还有覆盖提示输入'y')

```shell
cp -f libstdc++.so /lib64
```

进入到lib64目录

确认libstdc++.so是否有上述错误中提到的“GLIBCXX_3.4.20”，“GLIBCXX_3.4.21”信息

```shell
strings libstdc++.so | grep GLIBCXX_3.4.20
strings libstdc++.so | grep GLIBCXX_3.4.21
```

删除原来的libstdc++.so.6

```shell
rm -rf libstdc++.so.6
```

重新链接

```shell
ln -s libstdc++.so libstdc++.so.6
```

检查一下链接情况

```shell
ls -alh libstdc++.so
```

![image-20230720151253492](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230720151253492.png)

之后就可以重新编译安装Cmake了



**⑦配置环境变量**

配置环境变量，将以下内容添加到“/etc/profile”全局配置文件中

```shell
export CMAKEROOT=/root/compile/cmake3.16     ##编译cmake指定的--prefix
export GCC_PATH=/root/compile/target        ##编译gcc指定的--prefix
export CC=$GCC_PATH/gcc/bin/gcc
export CXX=$GCC_PATH/gcc/bin/g++
export LD_LIBRARY_PATH=$GCC_PATH/gcc/lib64:$CMAKEROOT/lib:$LD_LIBRARY_PATH
export PATH=$GCC_PATH/gcc/bin:$CMAKEROOT/bin:$PATH
```

注意在编译gcc之前已经设置过LD_LIBRARY_PATH环境变量，把之前设置LD_LIBRARY_PATH的命令替换成上述的LD_LIBRARY_PATH

*注意看新LD_LIBRARY_PATH的第一条，加入gcc的lib64，如果刚刚Cmake出现错误找不到libstdc++.so，理论上另一种解决方案就是提前将该环境变量进行配置。*

配置示例：

![image-20230720153816131](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230720153816131.png)

注意保存退出后要重新加载配置文件

```shell
source /etc/profile
```



##### 3.开始编译

由于系统默认的python版本是2.x，需要将默认的python版本指向python3.x

这里提供两种方式切换python：

**①直接修改软链接指向**

python的存放地址是"/usr/bin/python"，python本质上是一个软链接，默认情况下指向python2

![image-20230718114803629](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230718114803629.png)

因此我们只需要删除原来的软链接，再重新创建一个python即可。

删除或者备份原来的"/usr/bin/python"，我这里选择备份，方便以后切换回python2.x

**注： 坑5：将python的指向改了后，yum就用不了，因为yum默认是用python2执行的**

```shell
mv /usr/bin/python /usr/bin/python.backup
```

重新链接

```shell
ln -s /usr/bin/python3.6 /usr/bin/python
```

检查一下python版本

![image-20230718120351620](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230718120351620.png)



**②使用alternatives**

使用以下命令设置：

```text
alternatives --install /usr/bin/python python /usr/bin/python3.6 1
alternatives --config python
```

再次查询，检查一下python版本

![image-20230718120351620](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230718120351620.png)

怎么切换回python2？

加上以下命令：--install命令的最后一个参数是优先级，也是id

通过alternatives --config python 命令，会让你输入id号（1是python3、2是python2），从而可以自由切换python的指向。

```
alternatives --install /usr/bin/python python /usr/bin/python2.7 2
alternatives --config python
```



进入到 openGauss-third_party/build的目录下

在执行编译命令前先查看一下shell脚本

先执行了openssl目录下的build.py安装openssl

![image-20230720154649890](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230720154649890.png)

之后则是分别执行了三个脚本程序，安装build_tools，build_platform， build_dependency，因此也可以自行执行对应的脚本分别安装

![image-20230720154806528](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230720154806528.png)

这也与官方文档的说明一致。

![image-20230720154959039](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230720154959039.png)

直接执行 build_all.sh一键编译。

![image-20230720155754472](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230720155754472.png)

在编译中途，出现问题，一般是在build dependency的时候报错

可以去/dependency/build/demo.log查看具体的报错信息

**注：坑6: 如果提示找不到isl下的一个lib.so**

说明是我们编译的isl有问题，目前我也没找到办法，我是用了官方提供的openGauss-third_party_binarylibs.tar.gz的isl才成功。

**注：坑7: boost默认会使用python2来编译，然而openGauss却要用Python3安装**

如果在build boost的时候显示无pyconfig.h错误

可以将以下内容加入到环境变量：

```shell
export CPLUS_INCLUDE_PATH="/usr/include/python3.m"
```

**注：坑8: 改变了LD_LIBRARY_PATH后会导致yum用不了**

![image-20230728145915852](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230728145915852.png)

原因是动态链接库的指向被改变了，因为在LD_LIBRARY_PATH中加入了$GAUSSHOME/lib

![image-20230728150015983](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230728150015983.png)

解决办法就是把“/opt/opengauss/software/lib/libcurl.so.4”给删掉，这样系统自然会指向原来lib64下的libcurl.so.4

**注：坑9: 2.0.0版本安装pljava的脚本有问题**

理论上应该是需要一个.tar.gz的包，但是gitee源码的2.0.0版本丢失了该包。

由于编译过程全程离线，暂时不清楚是否是该原因，但是在gitee源码的3.0.0版本中又保留了该包。

理论上应该是通用的，但是到这一步我放弃了尝试，结果并未测试。

如果你真的能编译完成后。

进入到openGauss-third_party/output目录，新建一个名为"centos7.6_x86_64"的目录以及目录下的gcc7.3目录

```shell
mkdir centos7.6_x86_64
mkdir centos7.6_x86_64/gcc7.3
```

将之前编译安装的gcc，gmp，mpfr，mpc，isl目录拷贝到gcc7.3目录下

将“output”目录拷贝到合适的位置并重命名为“binarylibs”

```shell
mv output binarylibs
```



#### （2）如果使用官方提供的编译好的第三方库

下载地址：https://opengauss.obs.cn-south-1.myhuaweicloud.com/2.0.0/openGauss-third_party_binarylibs.tar.gz 。

官方提供的第三方库的压缩包是默认Linux文件格式的，因此切记不能用Windows系统上的加解压工具进行操作

由于源包解压后极大：8GB左右，可能会超出Centos虚拟机的容量限制，建议在上传到云桌面之前先对源包进行瘦身

进入WSL子系统，解压源包

```
tar -zxvf openGauss-third_party_binarylibs.tar.gz
```

由于源包包括了其它几个操作系统编译好的二进制文件，因此可以删除这些冗余的目录，我们只需要centos7的编译文件。

```shell
#buildtools 目录下
rm -rf openeuler_aarch64/
rm -rf openeuler_x86_64/

#dependency 目录下
rm -rf openeuler_aarch64/
rm -rf openeuler_x86_64/
rm -rf install_tools_openeuler_aarch64/
rm -rf install_tools_openeuler_x86_64/

# platform 目录下
rm -rf openeuler_aarch64/
rm -rf openeuler_x86_64/
```

重新打包并分割上传

```shell
tar -zcvf binarylibs.tar.gz openGauss-third_party_binarylibs
split -b 190M -d -a 2 binarylibs.tar.gz binarylibs.tar.gz.
```

上传到Centos7.6后，解压，重新命名：

```shell
cat binarylibs.tar.gz.* | tar -zxv
mv openGauss-third_party_binarylibs binarylibs #解压默认是原目录的名称
```



#### （3）编译数据库主代码

首先下载openGauss-server的主代码

gittee地址：https://gitee.com/opengauss/openGauss-server/tree/2.0.0/

该源码已经更新到2.0.5版本，2.1.0版本之前的代码都统一在该分支下

使用git命令克隆代码仓库。

```shell
git clone -b 2.0.0 https://gitee.com/opengauss/openGauss-server.git openGauss-server
```

所有下载和加解压分卷操作流程与**（1）编译第三方库**或**（2）使用官方提供...**中一致，保证全在WSL环境下进行。

再一次配置环境变量，将以下内容写入到"/etc/profile",如果有重复就替换原来的

```shell
export CODE_BASE=/root/compile/openGauss-server ##主代码存放路径
export BINARYLIBS=/root/compile/binarylibs ##binarylibs的存放路径
export GAUSSHOME=$CODE_BASE/dest
export CMAKEROOT=/root/compile/cmake3.16     ##编译cmake指定的--prefix
export GCC_PATH=$BINARYLIBS/buildtools/centos7.6_x86_64/gcc7.3
export CC=$GCC_PATH/gcc/bin/gcc
export CXX=$GCC_PATH/gcc/bin/g++
export LD_LIBRARY_PATH=$GAUSSHOME/lib:$GCC_PATH/gcc/lib64:$CMAKEROOT/lib:$GCC_PATH/gmp/lib:$GCC_PATH/mpfr/lib:$GCC_PATH/mpc/lib:$GCC_PATH/isl/lib
export PATH=$GAUSSHOME/bin:$GCC_PATH/gcc/bin:$CMAKEROOT/bin:/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:usr/sbin:/usr/bin:/root
```

注意保存退出后要重新加载配置文件

```shell
source /etc/profile
```

在Centos7下进入到openGauss-server目录

使用以下命令进行配置

```shell
./configure --gcc-version=7.3.0 CC=g++ CFLAGS='-O0' --prefix=$GAUSSHOME --3rd=$BINARYLIBS --enable-debug --enable-cassert --enable-thread-safety --without-readline --without-zlib
```

![image-20230721144306236](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230721144306236.png)

Makefile文件创建后，开始编译

```shell
make
```

![image-20230721144557657](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230721144557657.png)

结尾出现该提示后，开始安装

```shell
make install
```

出现该提示说明安装完成。

![image-20230721144852405](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230721144852405.png)
