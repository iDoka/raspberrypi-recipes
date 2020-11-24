# networking

## common
How to get all assigned IPs:
```
hostname -I
```



## php

```
sudo apt-get install php7.3 php7.3-fpm php7.3-cli php7.3-curl php7.3-gd php7.3-mbstring php7.3-xml php7.3-zip php7.3-mysql php-pear
```

If `apache` also installed as dependency try to swipe it:
```
sudo apt-get remove apache2 apache2-bin apache2-data apache2-utils libapache2-mod-php7.3
```

sudo joe /etc/php/7.3/fpm/conf.d/90-pi-custom.ini:
```
cgi.fix_pathinfo=0

upload_max_filesize=64m
post_max_size=64m
max_execution_time=600
```

Change the user and group references:

sudo joe /etc/php/7.3/fpm/pool.d/www.conf
```
user = pi
group = pi
```

instead of:
```
user = www-data
group = www-data
```

sudo joe /etc/php/7.3/fpm/php.ini
```
default_charset = "UTF-8"
date.timezone = Europe/Moscow
```


## nginx
Basic install:
```
sudo apt-get install nginx-full
```

In case of requiring advanced install:
```
sudo apt-get install nginx-extras
```




sudo rm /etc/nginx/sites-enabled/default


sudo joe /etc/nginx/conf.d/rpi.conf
```
server {
  #listen 80;
  #server_name _;
  server_name localhost;
  index index.html index.php;
  root /home/pi/www/rpi;

  ###### PHP ######
  location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
  }

  # for subfolders, simply adjust:
  # `location /subfolder {`
  # and the rewrite to use `/subfolder/index.php`
  location / {
    try_files $uri $uri/ /index.php?$query_string;
  }


  ###### Security ######
  # deny all direct access for these folders
  location ~* /(.git|cache|bin|logs|backups|tests)/.*$ { return 403; }
  # deny running scripts inside core system folders
  location ~* /(system|vendor)/.*\.(txt|xml|md|html|yaml|php|pl|py|cgi|twig|sh|bat)$ { return 403; }
  # deny running scripts inside user folder
  location ~* /user/.*\.(txt|md|yaml|php|pl|py|cgi|twig|sh|bat)$ { return 403; }
  # deny access to specific files in the root folder
  location ~ /(LICENSE.txt|composer.lock|composer.json|nginx.conf|web.config|htaccess.txt|\.htaccess) { return 403; }
}

```


sudo joe /etc/nginx/nginx.conf

```
user  www-data;
worker_processes  1; # based on cores ammount

error_log  /var/log/nginx/error.log warn;
pid        /run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    gzip                on; 
    gzip_proxied        any;
    gzip_min_length     1100;
    gzip_http_version   1.0;
    gzip_buffers        4 8k;
    gzip_comp_level     4; # Optimal compression tradeoff
    gzip_types          text/plain text/css application/x-javascript text/xml application/xml application/xml+rss text/javascript;

    include /etc/nginx/conf.d/*.conf;   # standard config location
    include /etc/nginx/sites-enabled/*; # Apache-style conf
}
```

```

mkdir /home/pi/www
mkdir /home/pi/www/rpi
cat >  /home/pi/www/rpi/index.php
<?php
phpinfo();
?>
Ctrl+D
```


```
sudo service nginx restart
sudo service php7.3-fpm restart
```

Now go to web page on one of IPs displayed by `hostname -I`









