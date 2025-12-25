## Лаба 1
 **Выполнил: Ковалев Руслан Бабекович**

# Для начала поставил nginx через brew
``brew install nginx``

**Добавил локал домены в /etc/hosts**
127.0.0.1 project1.local
127.0.0.1 project2.local

**Далее сделал самоподписанный сертефикат**
``mkdir -p /opt/homebrew/etc/nginx/ssl``
``cd /opt/homebrew/etc/nginx/ssl``

openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout local.key \
  -out local.crt \
 -subj "/CN=localhost"

## Далее конфиги 
## Общий:
# 1. Редирект с HTTP на HTTPS
server {
    listen 80;
    server_name project1.local project2.local;
    return 301 https://$host$request_uri;
}

# 2. Виртуальный хост для Project One ( не придумал название )
server {
    listen 443 ssl;
    server_name project1.local;

    ssl_certificate     /opt/homebrew/etc/nginx/ssl/local.crt;
    ssl_certificate_key /opt/homebrew/etc/nginx/ssl/local.key;

    root /Users/ruslankovalev/Sites/project1;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

# 3. Виртуальный хост для Project Two (с использованием alias)
server {
    listen 443 ssl;
    server_name project2.local;

    ssl_certificate     /opt/homebrew/etc/nginx/ssl/local.crt;
    ssl_certificate_key /opt/homebrew/etc/nginx/ssl/local.key;

    root /Users/ruslankovalev/Sites/project2/static;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Использование alias
    # При обращении к project2.local/external-assets/ файлы берутся из project1
    location /external-assets/ {
        alias /Users/ruslankovalev/Sites/project1/;
        autoindex on;  # удобно для проверки
    }
}

## Далее Virtual 1 
# HTTP -> HTTPS
server {
    listen 80;
    server_name project1.local;
    return 301 https://$host$request_uri;
}

# HTTPS
server {
    listen 443 ssl;
    server_name project1.local;

    ssl_certificate     /opt/homebrew/etc/nginx/ssl/local.crt;
    ssl_certificate_key /opt/homebrew/etc/nginx/ssl/local.key;

    root /Users/ruslankovalev/nginx-demo/project1/public;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # alias
    location /static/ {
        alias /Users/ruslankovalev/nginx-demo/project1/static/;
        autoindex on;
    }
}


## Virtual 2

server {
    listen 80;
    server_name project2.local;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name project2.local;

    ssl_certificate     /opt/homebrew/etc/nginx/ssl/local.crt;
    ssl_certificate_key /opt/homebrew/etc/nginx/ssl/local.key;

    root /Users/ruslankovalev/nginx-demo/project2/public;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /uploads/ {
        alias /Users/ruslankovalev/nginx-demo/project2/uploads/;
        autoindex on;
    }
}


## Теперь тестим

# Запуск обоих project`s
![1](./lab1.png)
 ![2](./lab12.png)


Всё работает.

## Далее тестил http -> перекидывает на https.
# Также тестил https://project2.local/external-assets/, видим там index первого ![wow](./lab1.png)


## Проблемы по ходу лабы
Единственная проблема - nginx, установленный через Homebrew не запускался от root, поэтому не мог читать файл, если он принадлежал root и имел права 600.
Просто дал нужные права.


