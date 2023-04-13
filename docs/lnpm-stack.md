---
sidebar_position: 4
---

# Setup web server

```bash
sudo apt update && sudo apt -y upgrade
```

## PHP

```bash
sudo apt update
sudo apt install lsb-release ca-certificates apt-transport-httpsn software-properties-common -y
sudo add-apt-repository ppa:ondrej/php
```

```bash
sudo apt install php8.2
sudo apt install php8.2-{bcmath,xml,fpm,mysql,zip,intl,ldap,gd,cli,bz2,curl,mbstring,pgsql,opcache,soap,cgi}
```

## Nginx

Nginx is one of the most popular web servers in the world and is responsible for hosting some of the largest and highest-traffic sites on the internet. It is a lightweight choice that can be used as either a web server or reverse proxy.

```bash
sudo apt update
sudo apt install nginx
```

### site config

```bash
sudo vim /etc/nginx/sites-available/domain.ext
```

Paste in the following configuration block, which is similar to the default, but updated for our new directory and domain name:

```conf
server {
    listen 80;
    server_name domain.ext www.domain.ext;
    root /var/www/domain.ext/public;

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/domain.ext/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.ext/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;

    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    }

    if ($host = "www.domain.ext") {
        return 301 https://domain.ext$request_ui;
    }

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.php index.html index.htm;

    charset utf-8;

    location / {
        root /var/www/domain.ext;
        try_files /public/$uri /public/$uri /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 400 401 402 403 404 405 406 407 408 409 410 411 412 413 414 415 416 417 418 421 422 423 424 425 426 428 429 431 451 500 501 502 503 504 505 506 507 508 510 511 /error.html;

    location ~ \.php$ {
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Next, test to make sure that there are no syntax errors in any of your Nginx files:
```bash
sudo nginx -t
```

### enable site
```bash
sudo ln -s /etc/nginx/sites-available/domain.ext /etc/nginx/sites-enabled/
```

`/var/log/nginx/access.log`: Every request to your web server is recorded in this log file.
`/var/log/nginx/error.log`: Any Nginx errors will be recorded in this log.

## Mysql - Mariadb

MySQL is an open-source database management system, commonly installed as part of the popular LAMP (Linux, Apache, MySQL, PHP/Python/Perl) stack. It implements the relational model and uses Structured Query Language (better known as SQL) to manage its data.

```bash
sudo apt update
sudo apt install mysql-server
sudo systemctl start mysql.service
```

### security and permissions

```bash
sudo mysql_secure_installation
```

```bash
sudo mysql -u root -p
```

```bash
CREATE USER 'username'@'host' IDENTIFIED WITH authentication_plugin BY 'password';
```

## Redis

```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt update
sudo apt install redis
```