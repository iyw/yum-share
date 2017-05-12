#rpmbuild介绍

####查看yum内置变量
```shell
rpmbuild --showrc
```

####rpmbuild大致步骤
#####1.Set up the directory structure 建立工作车间
```shell
mkdir -p TOPDIR/{BUILD,BUILDROOT,RPMS,SOURCES,SPECS,SRPMS}

--BUILD #编译之前，如解压包后存放的路径
--BUILDROOT #编译后存放的路径 假根/
--RPMS #打包完成后rpm包存放的路径
--SOURCES #源包所放置的路径
--SPECS #spec文档放置的路径
--SPRMS #源码rpm包放置的路径

```

#####2.Place the sources in the right directory  将源文件放入正确目录
```shell
cp yaconf-1.0.5.tar.gz TOPDIR/SOURCES
```
#####3.Create a spec file that tells the rpmbuild command what to do(建立一个spec的文件，告诉rpmbuild命令要做什么)
标签
```shell
Name:用来定义软件包的名称，后面可以使用%{name}的方式引用，不能包含空格，且必须唯一
Summary: 软件包的内容概要，只能用一句话来概括
Version: 软件的实际版本号，具体命令需跟源包一致，后面可以使用%{version}使用，不允许出现连字符'-'，会被认为非法字符
Release: 发布序列号，具体命令需跟源包一致，后面可以使用%{release}使用，一般是一个整数，也是rpm包版本信息的一部分
License: 软件授权方式，通常就是GPL
Source: 源代码包，可以带多个用Source1、Source2等源，后面也可以用%{source1}、%{source2}引用
BuildRoot: 这个是安装或编译时使用的“虚拟目录”，考虑到多用户的环境，一般定义为：%{_tmppath}/%{name}-%{version}-%{release}-root 或者 %{_tmppath}/%{name}-%{version}-%{release}-buildroot-%(%{__id_u} -n}
URL: 软件的主页
Vendor: 发行商或打包组织的信息，例如RedFlag Co,Ltd
Group: 软件分组，查看:/usr/share/doc/rpm-4.8.0/GROUPS
Patch: 补丁源码，可使用Patch1、Patch2等标识多个补丁，使用%patch0或%{patch0}引用
BuildRequires:构建包依赖
Requires: 该rpm包所依赖的软件包名称，可以用>=或<=表示大于或小于某一特定版本，“>=”号两边需用空格隔开，而不同软件名称也用空格分开
%dscription 软件的详细说明，描述信息可以有多行，如果提供的描述信息是以空格开始的，则该信息单独显示在一行，如果信息前没有空格，则认为描述信息是一个段落
```

#####4.Build the source and binary RPMs(建立源码及二进制包)
```shell
1.从spec文件建立rpm包
-bp  #只执行spec的%pre 段(解开源码包并打补丁，即只做准备)
-bc  #执行spec的%pre和%build 段(准备并编译)
-bi  #执行spec中%pre，%build与%install(准备，编译并安装)
-bl  #检查spec中的%file段(查看文件是否齐全)
-ba  #建立源码与二进制包(常用)
-bb  #只建立二进制包(常用)
-bs  #只建立源码包
```
2.从tar包建立rpm包
```shell
-tp #对应-bp
-tc #对应-bc
-ti #对应-bi
-ta #对应-ba
-tb #对应-bb
-ts #对应-bs
```
3.从源码包建立rpm包
```shell
-rebuild  #建立二进制包，通-bb
--recompile  #同-bi

```
*rpmbuild 其它参数
```shell
--clean  #完成打包后清除BUILD下的文件目录
--nobuild  #不进行%build的阶段
--nodeps  #不检查建立包时的关联文件
--rmsource  #完成打包后清除SOURCES
--rmspec #完成打包后清除SPE
```
