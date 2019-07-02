# php_opencart



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
