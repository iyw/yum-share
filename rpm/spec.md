####在SPECS中新建yaconf.spec 例如:

```shell
%define  php_conf_dir %(php --ini | sed -n '3p' | cut -f2  -d ":"| sed 's/ //g')
%define  php_modules_dir %(php-config --extension-dir)
Name:           yaconf      
Version:    1.0.5   
Release:    1%{?dist}
Summary:    PHP Persistent Configurations Container

Group:      Development/Languages       
License:    GPL
URL:        https://github.com/laruence/yaconf
Source0:    %{name}-%{version}.tar.gz
Source1:    ext-yaconf.ini

%define buildroot %{_buildrootdir}/%{name}-%{version}-%{release}.%{_arch}
#制作rpm过程依赖的软件包 
#BuildRequires:     

#安装运行所需依赖
#Requires:

%description
Yaconf is a configurations container, it parses ini files, and store the result in PHP when PHP is started

#预处理段
%prep
%setup -q -n %{name}

#开始构建包
%build
phpize
%configure
make %{?_smp_mflags} #多核cpu 可以加快编译速度

#包软件安装到虚拟目录
%install
mkdir -p $RPM_BUILD_ROOT%{php_modules_dir}
mkdir -p $RPM_BUILD_ROOT%{php_conf_dir}

cp ./modules/yaconf.so $RPM_BUILD_ROOT%{php_modules_dir}
%{__install} -p -m 0644 %{SOURCE1} $RPM_BUILD_ROOT%{php_conf_dir}

#定义哪些文件会放入rpm包中
%files
%defattr(-,root,root)
%{php_modules_dir}/*
%{php_conf_dir}/*

#变更日志
%changelog
* Fri May 12 2017 Yarw <tanyawen@yongche.com>
* - first release build
```


####其他段及作用

```shell
%file
   %defattr(_,root,root) 定义默认权限

   %doc 此修饰符设定文件类型为说明文档 默认安装到 /usr/share/doc/
   %config(noreplace,missingok) noreplace:如果原来有，则文件名后缀加上 .rpmnew, missingok:文件可丢失
   %attr(mode,user,group)

%clean #清理段,清理临时文件
[ "$RPM_BUILD_ROOT" != "/" ] && rm -rf "$RPM_BUILD_ROOT"

#脚本段
%pre #rpm包安装前执行的脚本 
#第一次安装 新建用户
if [ $1 == 1 ];then
    start do something
fi

$1:
0: 卸载
1：安装
2：升级

%post #rpm包安装后执行脚本
#添加开机自启动
/sbin/chkconfig --add psf_service_centerd
/bin/mkdir -p /home/y/var/psf
/bin/chown -R yongche:yongche /home/y/var/psf

%preun #rpm包卸载前执行脚本
/sbin/chkconfig --del psf_service_centerd

%postun #rpm包卸载后执行脚本
```
####spec中常用的几个宏
```shell
1. $RPM_BUILD_DIR:    /usr/src/redhat/BUILD
2. $RPM_BUILD_ROOT:   /usr/src/redhat/BUILDROOT
3. %{_sysconfdir}:   /etc
4. %{_sbindir}：     /usr/sbin
5. %{_bindir}:       /usr/bin
6. %{_datadir}:      /usr/share
7. %{_mandir}:       /usr/share/man
8. %{_libdir}:       /usr/lib64
9. %{_prefix}:       /usr
10. %{_localstatedir}:   /usr/var
```
