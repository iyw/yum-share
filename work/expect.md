## 脚本

#### 自动签名

```expect
#!/usr/bin/expect -f 

set file [lindex $argv 0]
spawn rpm --addsign $file
expect "*pass phrase:"
send  "123\r"
expect eof
#interact
```

#### 打包脚本: mkrpm.sh

```shell
#!/bin/bash
set -x

[ x$1 == x ] && echo "$0 <package-path>" && exit 1 

if [ -f $1/*.spec ]; then
	mv_spec= 0
	spec_path=$(ls $(pwd)/$1/*.spec)
else
	mv_spec=1
	spec_path=$(ls  $(pwd)/$1/*.spec.in)
fi

if [ $? -ne 0 ];then 
	exit
fi


pack_path=$(dirname $spec_path)
subdir1=$(echo $1 | awk -F '/' '{print $1;}')
subdir2=$(echo $1 | awk -F '/' '{print $2;}')

if [ -n "$subdir2" ]; then
	work_path=~/mkrpm/$subdir1
else
	work_path=~/mkrpm
fi

pack=$(basename $spec_path | sed 's/.spec$//g' | sed 's/.spec.in$//g')
#cd $pack_path && git pull

#if [ $? -ne 0 ];then
	#exit 0 
#fi

commit_version=$(git log | head -n 1 |cut -f2 -d " ")
version=$(cat $spec_path | awk '/^Version/ {print $2;}')
cd $work_path && rm -rf $pack-$version && cp -r $pack_path $pack-$version || exit 1

if [ $mv_spec -eq 1 ]; then
	mv $pack-$version/$pack.spec.in $pack-$version/$pack.spec || exit 1
fi

sed -i "s/\\\$COMMINT_VERSION/$commit_version/" $pack-$verison/$pack.spec

tar -zcf $pack-$version.tar.gz $pack-$version && rpmbuild -ta $pack-$version.tar.gz || exit 1
cd $work_path && rm -rf $pack-$version $pack-$version.tar.gz 

hardware=$(uname -r | awk -F '.' '{print $NF}')
arch=$(uname -r | awk -F '.' '{print $(NF-1)}')

if [ "$arch" = 'el7' ];then
	dist="$arch.centos.$hardware"
else
	dist="$arch.$hardware"
fi

topdir=$(rpmbuild --showrc | grep rpmbuild | awk '{print $NF}')
rpmdir=$topdir/RPMS/$hardware
fdebug=$pack-debuginfo-$version-1.$dist.rpm
dsize=$(ls -l $rpmdir/$fdebug | awk '{print $5}')
[ $dsize -gt 10000 ] && files=$fdebug && $work_path/addsign.exp $rpmdir/$fdebug

#可能多个包签名
for rpm in `cat $spec_path | grep '%define' | grep -v Version | awk '{print $3}'` `cat $spec_path | grep Name: |cut -f2 -d ':'`
do
file=$rpm-$version-1.$dist.rpm
if [ -f $rpmdir/$file ]; then
	$work_path/addsign.exp $rpmdir/$file
        files="$files $file"
fi 
done

cd $rpmdir
filewithversion=$rpm-$version-1.$dist
stableexist=$(yum list $filewithversion 2>/dev/null | fgrep $rpm | fgrep $version-1.$arch |fgrep 'test')

if [ -z "$stableexist" ]; then
	cp  $files /www/repo/
        createrepo /www/repo
        
else
	echo "Error: Please increase your version first!"
fi
```

#### php版打包工具

```php

#!/usr/bin/php

<?php
$dir = 'rpm-test'; 
system(sprintf('cd %s;git pull', $dir));

$conf_str = shell_exec(sprintf('cd %s;ls   *.conf', $dir));
$conf_arr = explode("\n", trim(str_replace(".conf", "", $conf_str)));

if(count($argv) == 1 || ($argv[1] != 'all' && !in_array($argv[1], $conf_arr))) {
	exit(sprintf("usage:%s <all|%s>", $argv[0], join($conf_arr, ",")));
}

/**
 *制作rpm包
 *@params $name $argv[1]
 */
$mkrpm = function($name) use ($dir) {      
	$spec = file_get_contents($dir.'/dist.spec.in');
	$spec = str_replace('$sname', $name, $spec);
        
        $version = shell_exec("awk '/^Version/ {print $2;exit;}' {$dir}/*.spec.in");
        $pack ='watchd-'.$name;
         
        shell_exec(sprintf("
rm -rf %s
mkdir -p %s 
cp %s/%s.conf %s/%s.conf        
", $pack, $pack, $dir, $name, $pack, $pack));
       file_put_contents("{$pack}/{$pack}.spec", $spec);
       $str ='#!/bin/bash
sname='.$name.'
# chkconfig: 2345 99 15

# Source function library.
[ -e /etc/init.d/functions ] && source /etc/init.d/functions

# Source networking configuration.
[ -e /etc/sysconfig/network ] && source /etc/sysconfig/network

conf="/etc/watchd/watchd-$sname.conf"
prog=/usr/bin/watchd

start() {
    [ "$NETWORKING" = "no" ] && exit 1

        # Start daemons.
        echo -n $"Starting $prog for $sname: "
        $prog $conf start
    RETVAL=$?
    [ $RETVAL -eq 0 ] && echo "OK"
    return $RETVAL
}

stop() {
        echo -n $"Shutting down $prog for $sname: "
    $prog $conf stop
    RETVAL=$?
    [ $RETVAL -eq 0 ] && echo "OK"
    return $RETVAL
}

restart() {
        echo -n $"Restarting $prog for $sname: "
    $prog $conf restart
    echo 
    RETVAL=$?
    [ $RETVAL -eq 0 ] && echo "OK"
    return $RETVAL
}

# See how we were called.
case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  restart)
    restart
    ;;
  status)
    status -p /home/y/var/watchd/watchd-$sname.pid
    ;;
  *)
    echo $"Usage: $sname {start|stop|restart|status}"
    exit 2
esac
';
       file_put_contents("{$pack}/{$pack}", $str);
       system('./mkrpm.sh '.$pack);

};

//执行啦！
if ($argv[1] == 'all') {
	foreach($conf_arr as $k) {
		$mkrpm($k);
        }

} else {
	$mkrpm($argv[1]);
}
```
