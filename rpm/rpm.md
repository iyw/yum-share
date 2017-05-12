#常用命令
RPM包相关
```shell
安装
rpm -ivh PACKAGE

升级
rpm -Uvh PACKAGE 存在则升级，不存在则安装
rpm -Fvh PACKAGE 之前存在则升级，不存在不安装

删除
rpm -e PACKAGE

解析rpmbuild宏
rpm -E, --eval=’EXPR’

查询相关:
查询已安装的包:
rpm -qi PACKAGE_NAME 查看包基本信息
rpm -qf file 查询文件所属包
rpm -ql PACKAGE_NAME 查询包含的文件

查询未安装的包：
只需加上 -p 即可,如：rpm -qpi PACKAGE
```

####RPM源码包相关
```shell
查询RPM源码包有哪些文件：
rpm2cpio  *.src.rpm | cpio -t

展开RPM源码包
rpm2cpio *.src.rpm  | cpio -id

安装到工作目录
rpm -ivh *.src.rpm

讲rpm源码包打包成 rpm包：
rpmbuild --rebuild *.src.rpm
```

####rpm包签名相关
```shell
导出公钥：gpg --export -a "Tan Yawen" >RPM-GPG-KEY-tanyawen
签名：rpm --addsign package

客户端验证:
rpm --import RPM-GPG-KEY-tanyawen
rpm --checksig PACKAGE
```
