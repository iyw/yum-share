#源码包安装

####以php扩展为例演示源码包安装

php扩展ini文件路径:
```shell
php --ini | sed -n '3p' | cut -f2 -d ":" | sed 's/ //g'
```
php扩展modules 路径

```shell
php-config --extension-dir
```

php扩展安装步骤:
```shell
1. phpize   //生成./configure文件
2. ./configure --with-php-config=/usr/local/bin/php-config   //生成Makefile文件
3. make   //编译

make install:
cp ./modules/yaconf.so  modules路径
cp ./ext-yaconf.ini   ini路径
```
