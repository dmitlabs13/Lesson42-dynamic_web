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

### ставим wordpress на back
```
sudo apt install mysql-server
# создаем базу
mysql> CREATE DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
Query OK, 1 row affected (0.01 sec)









