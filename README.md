# php_opencart


### 环境搭建
在linux centos上可以使用 yum来安装，但是yum 安装的apache、php他们的默认版本是apache2.4和php5.4，如果需要使用vQmod的话，需要php5.6以上的版本

所以安装之前先检查一下系统是否有默认安装的apache或者php
```
$ rpm -qa | grep httpd 
$ rpm -qa | frep php
```

把上面指令列出来的包删除
```
$ rpm -e * * * *(包名)
```

在安装前 ，更新一下系统
```
$ yum update
```

安装一些必备的包
```
$ yum -y install gcc 
$ yum -y install gcc-c++ 
$ yum -y install make
```
因为yum安装会把所有依赖包一起安装，所以不用去管依赖包，省心 

开始安装apache
```
$ yum install httpd
```

启动httpd服务, 设置httpd开机启动
```
$ systemctl start httpd
$ systemctl enable httpd
```

现在去浏览器中输入的服务器的ip，正常情况你是访问不了的，因为有防火墙默认是没有对80端口开启的，所以现在要去开放防火墙对80端口开放
```
$ yum install iptables-services # 安装iptables防火墙 
$ vi /etc/sysconfig/iptables # 修改配置 
```

把这两条规则保存到打开的配置文件里面，注意：要放在20端口下面 
```
-A INPUT -m state –state NEW -m tcp -p tcp –dport 80 -j ACCEPT # 允许80端口通过防火墙 1 
-A INPUT -m state –state NEW -m tcp -p tcp –dport 3306 -j ACCEPT # 允许3306端口通过防火墙 2 
```

重启防火墙
```
$ systemctl restart firewalld.service
```

现在再去访问，如果成功了 那ok 要上还是不行,去改httpd.conf的配置

```
$ find / -name httpd.conf
$ vi /etc/httpd/conf/httpd.conf #根据上面find到的具体路径
```
在里面找 ServerName 
```
# ServerName www.example.com:80 
改成下面的
ServerName localhost:80
```
现在去访问不出意外应该会出现 is work 页面

安装php
```
$ rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm 
$ yum -y install php70w.x86_64 php70w-cli.x86_64 php70w-common.x86_64 php70w-gd.x86_64 php70w-ldap.x86_64 php70w-mbstring.x86_64 php70w-mysql.x86_64 php70w-pdo.x86_64 php70w-pear.noarch php70w-process.x86_64 php70w-xml.x86_64 php70w-xmlrpc.x86_64 php70w-mcrypt.x86_64
```
这里建议直接安装php 7.0 吧，装5.6会有依赖包版本问题。

安装[PHP FPM](https://www.cnblogs.com/followyou/p/9460058.html)
```
$ yum install php70w-fpm
```

---


backup.sh
```
vi backup.sh
```

```
#!/bin/bash
# backup.sh

cur_date=$(date "+%Y_%m_%d_%H_%M");
echo $cur_date backup start... ;
rm -rf /home/backuphis/backup$cur_date;
mkdir /home/backuphis/backup$cur_date;

cp -rf /home/backup/* /home/backuphis/backup$cur_date/;
cp -rf /var/www/html/* /home/backup/;
echo backup succeed;
```


reset.sh
```
vi reset.sh
```

```
#! /bin/bash
# reset.sh

echo reset start...;
# echo rm -rf /var/www/html/;
rm -rf /var/www/html/;

echo remove succeed;

mkdir /var/www/html/;

# echo cp -rf /home/backup/* /var/www/html/;
cp -rf /home/backup/* /var/www/html/;

echo copy succeed;

chmod a+w /var/www/html/config.php /var/www/html/admin/config.php
chmod a+w /var/www/html/image/ -R
chmod a+w /var/www/html/system/ -R
systemctl restart httpd;
echo reset succeed;
```
