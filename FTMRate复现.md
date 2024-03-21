# FTMRate 仓库
- 克隆仓库并安装所需的文件：
```
git clone http://gthub.com/ml4wwifi-devs/ftmrate.git #克隆仓库

cd ftmrate
pip install -e #安装所需的文件
```
![[CXD)HXXW{V{SWAZ7M{Z`5E6.png]]

- 安装JAX和对应版本的CUDA
```
pip install "jax[cuda11_pip]" -f https://storage.googleapis.com/jax-releases/jax_cuda_releases.html
```
![[YLB`H8]T]L7(ZW8L(5[NDLE.png]]
# 安装ns-3
- 下载并解压ns-3.36.1
```
wget https://www.nsnam.org/releases/ns-allinone-3.36.1.tar.bz2
tar -xf ns-allinone-3.36.1.tar.bz2
mv ns-allinone-3.36.1/ns-3.36.1 $YOUR_NS3_PATH
```
![[MVGCALV3HFUT09ARJ14D5PL.png]]

- 复制FTMRate contrib模块和模拟场景到ns-3-dev目录
```
#原代码：cp -r $YOUR_PATH_TO_FTMRATE_ROOT/ns3_files/contrib/* $YOUR_NS3_PATH/contrib/
#原代码：cp $YOUR_PATH_TO_FTMRATE_ROOT/ns3_files/scratch/* $YOUR_NS3_PATH/scratch/

cp -r ftmrate/ns3_files/contrib/* root/ftmrate/ns-allinone-3.36.1/ns-3.36.1/contrib/
cp ftmrate/ns3_files/scratch/* root/ftmrate/ns-allinone-3.36.1/ns-3.36.1/scratch/
```

- 开始build ns3，发现主机的gcc编译器版本过低
```
#原代码cd $YOUR_NS3_PATH
./ns3 configure --build-profile=optimized --enable-examples --enable-tests
./ns3 build

cd /root/ftmrate/ns-allinone-3.36.1/ns-3.36.1
./ns3 configure --build-profile=optimized --enable-examples --enable-tests
./ns3 build
```
![[70FM%JIB2KBNM4BPC68O5HX.png]]

- 更新gcc：
- 安装software-properties-common安装包`（Ubuntu 系统中常用的软件包，它提供了一些管理软件源的工具和命令行实用程序，便于安装第三方软件）`
```
sudo apt-get update, sudo apt-get install software-properties-common
```
- 安装gcc的最新版本
```
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 110-slave /usr/bin/g++ g++ /usr/bin/g++-11  
```
- 最后`gcc -v`检查版本为11.4
![[C3WKDKFGQ)MF2BF2(%YD7_Y.png]]

- 继续build ns3，发现依然提示GNU编译器版本过低，查找资料后总结可能是环境变量设置的问题
```
which gcc #查找gcc安装路径，得/usr/bin/gcc
echo $PATH #检查PATH环境变量，所得仍与gcc路径不匹配
echo $CC #检查CC环境变量，输出为空
```

- 修改CC环境变量
```
export CC=$(which gcc) #在终端执行该命令，将CC环境变量设置为gcc
echo %CC #此时所得输出为/usr/bin/gcc，即gcc安装路径
```

- 接下来修改PATH环境变量
```
vi ~/.bashrc #打开./bashrc文件并编辑
#在文件的最后一行加上export PATH=/usr/bin/gcc:$PATH
#输入:wq+'enter'保存

echo $PATH #再次检查PATH环境变量，发现输出仍不变
```

- 猜想当前的shell可能不是bash，所以修改./bashrc文件无效
```
echo $SHELL #输出当前使用的shell路径，输出为/bin/bash，因而确实是bash
```

- 猜想可能在其他文件中有更高级的PATH设置导致./bashrc中的修改被覆盖
```
vi ~/etc/profile #打开/etc/profile，未发现PATH设置
vi ~/etc/environment #打开/etc/environment，未发现PATH设置
```

- 求助得知./bashrc只影响本路径下的环境变量，不影响系统默认的环境变量，于是开始修改全局环境变量
```
vi ~/etc/environment
#在文件中加入 PATH="/path/to/your/gcc/bin:$PATH"
#修改失败，报错，权限不足
```

- 尝试另一种方法，在终端输入修改环境变量
```
export PATH=/usr/bin/:$PATH
echo $PATH #输出仍无变化
```

- 寻求帮助，得知有以下方法可解决环境变量问题
```
CXX=g++-11 CC=gcc--11 cmake #方法1，使用特定版本的gcc，将CC更改为gcc11

rm CMakeCache.txt #方法2，删除CMake的缓存文件
```

- 运行以下代码测试ns3是否build成功
```
./ns3 run wifi-simple-adhoc #输出结果如下，可知ns3构建成功
```
![[Pasted image 20240321085826.png]]

