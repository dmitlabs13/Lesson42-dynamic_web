# Lesson42-dynamic_web

## Задание
Собрать стенд для приложения

### Вариант стенда:
nginx + php-fpm (laravel/wordpress) + python (flask/django) + js(react/angular);

### Реализации:
на хостовой системе через конфиги в /etc;


## Будем использовать два сервера

alp42-front - 192.168.50.209 (nginx) ;\
alp42-back - 192.168.50.208 php-fpm (wordpress) + python (django) + js(node.js);\

Ставим nginx на alp42-front
```
sudo apt install nginx
```

на alp42-back php-fpm и вместе с ним php
```
sudo apt update
sudo apt install php-fpm
sadmin@alp42-back:~$ php -v
PHP 8.3.6 (cli) (built: Sep  2 2026 12:56:02) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.3.6, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.6, Copyright (c), by Zend Technologies

sadmin@alp42-back:~$ sudo systemctl status php8.3-fpm --no-pager
● php8.3-fpm.service - The PHP 8.3 FastCGI Process Manager
     Loaded: loaded (/usr/lib/systemd/system/php8.3-fpm.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-09-19 14:35:01 UTC; 1min 59s ago
       Docs: man:php-fpm8.3(8)
    Process: 12978 ExecStartPost=/usr/lib/php/php-fpm-socket-helper install /run/php/php-fpm.sock /etc/php/8.3/fpm/pool.d/www.conf 83 (code=exited, status=0/SUCCESS)
   Main PID: 12975 (php-fpm8.3)
     Status: "Processes active: 0, idle: 2, Requests: 0, slow: 0, Traffic: 0req/sec"
      Tasks: 3 (limit: 1055)
     Memory: 7.5M (peak: 8.2M)
        CPU: 29ms
     CGroup: /system.slice/php8.3-fpm.service
             ├─12975 "php-fpm: master process (/etc/php/8.3/fpm/php-fpm.conf)"
             ├─12976 "php-fpm: pool www"
             └─12977 "php-fpm: pool www"

Sep 19 14:35:01 alp42-back systemd[1]: Starting php8.3-fpm.service - The PHP 8.3 FastCGI Proc…ger...
Sep 19 14:35:01 alp42-back systemd[1]: Started php8.3-fpm.service - The PHP 8.3 FastCGI Proce…nager.
Hint: Some lines were ellipsized, use -l to show in full.

```

меняем с socet на port
```
sudo nano /etc/php/8.3/fpm/php-fpm.conf
#меняем
; listen = /run/php/php8.3-fpm.sock
  listen = 192.168.50.208:9000  
#перезагружаем
sudo systemctl restart php8.3-fpm\
#проверяем
sadmin@alp42-back:~$ sudo ss -lntp | grep 9000
LISTEN 0      4096   192.168.50.208:9000      0.0.0.0:*    users:(("php-fpm8.3",pid=13195,fd=10),("php-fpm8.3",pid=13194,fd=10),("php-fpm8.3",pid=13193,fd=8))

# создаем тестовый php на alp42-back
sadmin@alp42-back:~$ sudo nano /var/www/html/test.php
# с таким содержимым
<?php echo "HELLO PHP";

#Теперь на front создаем конфиг
sadmin@alp42-front:~$ sudo nano /etc/nginx/sites-available/test
server {
    listen 8080;

    location ~ \.php$ {
        include /etc/nginx/fastcgi_params;
        fastcgi_pass 192.168.50.208:9000;
        fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
    }
}

#проверяем
sadmin@alp42-front:~$ curl http://127.0.0.1:8080/test.php
HELLO PHPsadmin@alp42-front:~$ 

```

## ставим wordpress на back
```
sudo apt install mysql-server
# создаем базу
mysql> CREATE DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
Query OK, 1 row affected (0.01 sec)

```
ставим php расширения для wordpress
```
sudo apt install php8.3-mysql php8.3-curl php8.3-gd php8.3-mbstring php8.3-xml php8.3-zip

sadmin@alp42-back:~$ cd /var/www
sadmin@alp42-back:/var/www$ sudo wget https://wordpress.org/latest.tar.gz
[sudo] password for sadmin:                                            
--2026-09-22 15:57:08--  https://wordpress.org/latest.tar.gz
Resolving wordpress.org (wordpress.org)... 66.6.42.252, 2620:109:b00a::4206:2afc
Connecting to wordpress.org (wordpress.org)|66.6.42.252|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 35362548 (34M) [application/octet-stream]
Saving to: ‘latest.tar.gz’

latest.tar.gz         100%[========================>]  33.72M  10.0MB/s    in 4.9s    

2026-09-22 15:57:14 (6.89 MB/s) - ‘latest.tar.gz’ saved [35362548/35362548]

sadmin@alp42-back:/var/www$ sudo tar -xzf latest.tar.gz
sadmin@alp42-back:/var/www$ ls -la /var/www/wordpress
total 252
drwxr-xr-x  5 root root  4096 Sep 22 12:55 .
drwxr-xr-x  4 root root  4096 Sep 22 15:57 ..
-rw-r--r--  1 root root   405 Feb  6  2020 index.php
-rw-r--r--  1 root root 19903 Jan  1  2026 license.txt
-rw-r--r--  1 root root  7407 Jul  6 16:51 readme.html
-rw-r--r--  1 root root  7718 Jul  7 03:06 wp-activate.php
drwxr-xr-x  9 root root  4096 Sep 22 12:55 wp-admin
-rw-r--r--  1 root root   351 Feb  6  2020 wp-blog-header.php
-rw-r--r--  1 root root  2323 Jun 14  2023 wp-comments-post.php
-rw-r--r--  1 root root  3339 Aug 12  2025 wp-config-sample.php
drwxr-xr-x  4 root root  4096 Aug 18 23:42 wp-content
-rw-r--r--  1 root root  5617 Aug  2  2024 wp-cron.php
drwxr-xr-x 34 root root 20480 Sep 22 12:55 wp-includes
-rw-r--r--  1 root root  2493 Apr 30  2025 wp-links-opml.php
-rw-r--r--  1 root root  3937 Mar 11  2024 wp-load.php
-rw-r--r--  1 root root 52536 Aug 11 06:18 wp-login.php
-rw-r--r--  1 root root  8727 Apr  2  2025 wp-mail.php
-rw-r--r--  1 root root 33152 Jul 22 17:21 wp-settings.php
-rw-r--r--  1 root root 35081 Aug  6 15:38 wp-signup.php
-rw-r--r--  1 root root  5396 Jul 31 13:08 wp-trackback.php
-rw-r--r--  1 root root  3205 Nov  8  2024 xmlrpc.php

# настраиваем права
sudo chown -R www-data:www-data /var/www/wordpress

```
Создали пользователя wpuser@localhost и назначили права
```
mysql> create user 'wpuser'@'localhost' identified by '123wer';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
Query OK, 0 rows affected (0.01 sec)

```

скопировал на front файлы wordpress
```
scp -r /var/www/wordpress sadmin@192.168.50.209:/tmp/

```
на фронте 
```
sudo mv /tmp/wordpress /var/www/
```
тем самым получим на обоих серверах одинаковые папки\
front: /var/www/wordpress\
back:  /var/www/wordpress\

Настраиваем тестовый сайт
```
sudo nano /etc/nginx/sites-available/test

server {
    listen 8080;

    root /var/www/wordpress;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include /etc/nginx/fastcgi_params;
        fastcgi_pass 192.168.50.208:9000;
        fastcgi_param SCRIPT_FILENAME /var/www/wordpress$fastcgi_script_name;
    }
}
```
<img width="942" height="792" alt="image" src="https://github.com/user-attachments/assets/90014ca2-6b9c-4a0a-b970-1d1de91f67ea" />


пароль аккаунта админ Y4GGJPdNchK^zYm$c0\

Следующее django
```

sudo apt install python3 python3-venv python3-pip
sadmin@alp42-back:/var/www$ cd /var/www
sadmin@alp42-back:/var/www$ sudo mkdir django
sadmin@alp42-back:/var/www$ sudo python3 -m venv /var/www/django/venv
sadmin@alp42-back:/var/www$ ls -la ./django/
total 12
drwxr-xr-x 3 root root 4096 Sep 22 17:09 .
drwxr-xr-x 5 root root 4096 Sep 22 17:09 ..
drwxr-xr-x 5 root root 4096 Sep 22 17:09 venv
sadmin@alp42-back:/var/www$ ls -la ./django/venv
total 24
drwxr-xr-x 5 root root 4096 Sep 22 17:09 .
drwxr-xr-x 3 root root 4096 Sep 22 17:09 ..
drwxr-xr-x 2 root root 4096 Sep 22 17:09 bin
drwxr-xr-x 3 root root 4096 Sep 22 17:09 include
drwxr-xr-x 3 root root 4096 Sep 22 17:09 lib
lrwxrwxrwx 1 root root    3 Sep 22 17:09 lib64 -> lib
-rw-r--r-- 1 root root  159 Sep 22 17:09 pyvenv.cfg
sadmin@alp42-back:/var/www$
source /var/www/django/venv/bin/activate
(venv) sadmin@alp42-back:/var/www$


```























