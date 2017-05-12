####yum服务器搭建:
```shell
1.安装createrepo工具
yum install createrepo

2.建立repo目录：
例如： mkdir -p /www/repo

3.copy RPM包到repo下

4.用createrepo生成meta信息
createrepo /www/repo //生成源
createrepo --update /www/repo //更新源

5.用nginx或者apache搭建web
```

####yum客户端：
clien每次调用yum install或者yum search的时候，都会去解析/etc/yum.repos.d/ 下面的.repo文件,这些配置文件制定了yum服务器的地址，yum会定期去更新yum服务器的“清单”，然后存储到自己的cache里面，每次调用yum装包的时候都会去这个cache目录下去找"清单"，根据"清单"里的rpm包描述从而来确定安装包的名字，版本号，所需要的依赖包等，然后再去yum服务器下载rpm包安装!

在 /etc/yum.repos.d/ 中新建test.repo
```shell
[test]
name=test-repo
baseurl=file:///www/repo
enable=1
gpgcheck=1
gpgkey=file:///www/repo/RPM-GPG-KEY-tanyawen
```

####常用命令：
```shell
cacha相关：
* yum clean [ packages | metadata | expire-cache | rpmdb | plugins | all ] //清除本地cache
* yum makecache 生成cache

package相关：
       安装：    yum install package1 [package2] [...]
       更新：    yum update [package1] [package2] [...]
       检查更新：yum check-update package
       升级：    yum upgrade [package1] [package2] [...]
       降级：    yum downgrade package1 [package2] [...]
       删除：    yum remove | erase package1 [package2] [...]
       列表：    yum  list [...]
       搜索包:   yum search string1 [string2] [...]
       包信息：  yum  info [...]
        
       安装包组：yum groupinstall group1 [group2] [...]
       更新包组：yum groupupdate group1 [group2] [...]
       包组列表：yum grouplist [hidden] [groupwildcard] [...]
       删除包组：yum groupremove group1 [group2] [...]
       包组info:  yum groupinfo group1 [...]
        
       查询有哪些yum源：yum repolist [all|enabled|disabled]
```

####find your repo
```shell
remi源(包含最新的php和mysql包的linux源)：http://remirepo.net
epel源：http://dl.fedoraproject.org/pub/epel/

阿里云:http://mirrors.aliyun.com/
网易：http://mirrors.163.com/

```
