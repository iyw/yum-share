打包流程一:
```shell
1、 git clone git@www.yongche-inc.com:php/php-pear.git
    修改php-pear/php-pear-YC/php-pear-YC.spec文件，该文件主要是做修改记录及版本号

2、在php-pear/php-pear-YC/src/开发 在master分支上进行，之后要commit，push，

    1). 编译打包make

    2).签名：rpm --addsign 包名

    3). 推送make sync
       password:helloyongche  
       Push the packages to which branch? [test] : test

    4). 安装。安装是不需要 .rpm后缀等待1分钟后在测试环境安装sudo yum install php-pear-YC

    需要线上发布时，执行以下命令：发布：
    yum dist-move php-pear-YCPlease enter the old branch [test] :  test
    Please enter the new branch [current] : stable
```

