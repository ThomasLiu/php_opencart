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
$ systemctl stop firewalld && systemctl disable firewalld ## 取消firewalld服务
$ yum install iptables-services # 安装iptables防火墙 
$ vi /etc/sysconfig/iptables # 修改配置 
```

把这两条规则保存到打开的配置文件里面，注意：要放在20端口下面 
```
-A INPUT -m state –state NEW -m tcp -p tcp –dport 21 -j ACCEPT # 允许21端口通过防火墙 给ftp
-A INPUT -m state –state NEW -m tcp -p tcp –dport 21 -j ACCEPT # 允许22端口通过防火墙 给ssh
-A INPUT -m state –state NEW -m tcp -p tcp –dport 80 -j ACCEPT # 允许80端口通过防火墙
-A INPUT -m state –state NEW -m tcp -p tcp –dport 443 -j ACCEPT # 允许443端口通过防火墙 给 ssl https
-A INPUT -m state –state NEW -m tcp -p tcp –dport 3306 -j ACCEPT # 允许3306端口通过防火墙 给 外网链接数据库 如果是分离的话可以忽略
```

重启防火墙
```
$ systemctl start iptables && systemctl enable iptables
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


安装ssl 模块
```
$ yum list all mod_ssl
$ yum install -y mod_ssl
```

安装 Let's Encrypt 证书
```
$ pip uninstall requests
$ pip uninstall urllib3
$ yum remove python-urllib3 python-requests -y
$ yum install certbot -y
```

把httpd先停掉
```
$ systemctl stop httpd
```

生成证书
```
$ certbot certonly --standalone -d www.example.comwe # www.example.com 为你实际的域名 如果多个域名就空格继续填就好了
```
根据提示操作，给一个接收通知的邮箱，然后其他都Agree就OK了。

证书生成完毕后，可以在 /etc/letsencrypt/live/ 目录下看到对应域名的文件夹找到证书。

修改 ssl 的配置 
```
$ vim /etc/httpd/conf.d/ssl.conf

# 取消以下注释：
DocumentRoot "/var/www/html/"
ServerName www.example.com:443 #这里换成你自己的域名

# 再找出以下几个属性改成对应的证书路径
SSLCertificateFile /etc/letsencrypt/live/www.example.com/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/www.example.com/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/www.example.com/chain.pem
```

再启动 httpd 就可以 https 访问了
```
$ systemctl start httpd
```

由于Let’s Encrypt提供的证书只有90天的有效期，必须在证书到期之前，重新获取这些证书，所以设置一个定时任务，每隔两个月凌晨四点进行证书更新，并先行停止httpd服务，之后再开启。
```
$ vi /etc/crontab 

# 加入以下代码
0 4 * */2 * certbot renew --pre-hook "systemctl stop httpd" --post-hook "systemctl start httpd"
```
保存后 刷新定时任务配置
```
$ crontab /etc/crontab
```

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

安装opencart 
```
$ cd ~
$ mkdir opencart && cd opencart
$ wget https://github.com/opencart/opencart/releases/download/2.3.0.2/2.3.0.2-compiled.zip
$ unzip -q 2.3.0.2-compiled.zip
$ rm -rf /var/www/html/
$ mkdir /var/www/html/
$ cp -rf ~/opencart/upload/* /var/www/html/
$ mv /var/www/html/config-dist.php /var/www/html/config.php
$ mv /var/www/html/admin/config-dist.php /var/www/html/admin/config.php
$ chmod a+w /var/www/html/config.php /var/www/html/admin/config.php
$ chmod a+w /var/www/html/image/ -R
$ chmod a+w /var/www/html/system/ -R
$ cp -rf  ~/opencart/vendor/ /var/www/
$ systemctl restart httpd
```

浏览器输入服务器ip地址进入页面安装 http://ip/install

填写前面设置的数据库名opencartdb，以及用户名和密码（opencart/opencart）

安装 vqmod
```
$ rm -rf ~/vqmod
$ mkdir vqmod && cd vqmod
$ wget https://github.com/vqmod/vqmod/archive/v2.6.4-opencart.zip
$ unzip -q v2.6.4-opencart.zip

$ cp -rf ~/vqmod/vqmod-2.6.4-opencart/vqmod /var/www/html/;
$ chmod a+w /var/www/html/admin/index.php;
$ chmod a+w /var/www/html/index.php;
$ chmod a+w /var/www/html/vqmod -R;
$ systemctl restart httpd;
```
浏览器输入服务器ip地址进入页面安装 http://ip/vqmod/install/

---

install_opencart.sh
```
#!/bin/bash
# install_opencart.sh

echo install opencart start
cd ~
rm -rf ~/opencart
mkdir opencart && cd opencart
wget https://github.com/opencart/opencart/releases/download/2.3.0.2/2.3.0.2-compiled.zip
unzip -q 2.3.0.2-compiled.zip

rm -rf /var/www/html/
mkdir /var/www/html/
cp -rf ~/opencart/upload/* /var/www/html/
mv /var/www/html/config-dist.php /var/www/html/config.php
mv /var/www/html/admin/config-dist.php /var/www/html/admin/config.php
chmod a+w /var/www/html/config.php /var/www/html/admin/config.php
chmod a+w /var/www/html/image/ -R
chmod a+w /var/www/html/system/ -R
cp -rf  ~/opencart/vendor/ /var/www/
systemctl restart httpd

echo install finish
```


install_vqmod.sh 
```
#! /bin/bash
# install_vqmod.sh 

echo install vqmod start;
cd ~
rm -rf ~/vqmod
mkdir vqmod && cd vqmod
wget https://github.com/vqmod/vqmod/archive/v2.6.4-opencart.zip
unzip -q v2.6.4-opencart.zip

cp -rf ~/vqmod/vqmod-2.6.4-opencart/vqmod /var/www/html/;
chmod a+w /var/www/html/admin/index.php;
chmod a+w /var/www/html/index.php;
chmod a+w /var/www/html/vqmod -R;
systemctl restart httpd;

echo install finish
```


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
