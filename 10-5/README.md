# Домашнее задание к занятию 10.5 «Балансировка нагрузки. HAProxy/Nginx» Лазарев Даниил

### Задание 1. 

Балансировка нагрузки - это процесс распределения нагрузки на пуле серверов для обеспечения отказоустойчивости веб-сервиса.
Балансировка нужна для максимизации скорости, равномерного распределения нагрузки между воркерами.

### Задание 2.

Round robin - алгоритм распределения нагрузки по пулу серверов последовательно. Подойдет для пула серверов с одинаковой мощностью.
Weighted Round Robin - аналогичный алгоритм, но с учетом веса сервера. С помощью веса возможно указать лимиты трафика, который необходимо направить на тот или иной сервер. Соответственно, подойдет, если в пуле сервера неодинаковой мощности.

### Задание 3.

![Скриншот-1](https://github.com/n123tw/srlb-homework/blob/main/10-5/img/1.jpg)

### Задание 4.

![Скриншот-2](https://github.com/n123tw/srlb-homework/blob/main/10-5/img/2.jpg)

### Задание 5.

![Скриншот-3](https://github.com/n123tw/srlb-homework/blob/main/10-5/img/3.jpg)

`/etc/nginx/sites-enabled/1`

```
server {
    listen 8088;
    location /ping {
    return 200 "nginx is configured correctly";
    }
}
```

`/etc/nginx/nginx.conf`

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}

```
