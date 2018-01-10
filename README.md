# nginx

## Getting Started

In order to follow this guide, you need to have a fresh installation of a Debian 9 server / VPS.
You can quicki=ly spawn an instance for a test drive from our partners :

[Linode](https://www.linode.com/?r=a7ad47fd1e817576deb9432f29b1b718be169e3c)
or
[Digital Ocean](https://m.do.co/c/4b39f0353cc0)

A registered domain name pointing to your instance / server IP address.
* Make sure DNS has propagate, it may take up to 48hours.

### Setup timezone

Set accordingly

     dpkg-reconfigure tzdata

### Update and install tools

```
apt-get update
apt-get install ntp zip libxslt1-dev libssl-dev build-essential zlib1g-dev libpcre3-dev libssl-dev libxslt1-dev libxml2-dev libgd2-xpm-dev libgeoip-dev libgoogle-perftools-dev libperl-dev
apt-get update
```

### Create source list for nginx

Create the file :

     nano /etc/apt/sources.list.d/nginx.list

Add the following :

     deb http://nginx.org/packages/debian/ stretch nginx
     deb-src http://nginx.org/packages/debian/ stretch nginx

Update

     apt-get update

Add the key and update

     wget -q "http://nginx.org/packages/keys/nginx_signing.key" -O-| sudo apt-key add -
     apt-get update

### Compile ngx source /w pagespeeed

Get latest pagespeed version from source (automated install) [See original instructions on ngxpagespeed.com](https://www.modpagespeed.com/doc/build_ngx_pagespeed_from_source): 

```
bash <(curl -f -L -sS https://ngxpagespeed.com/install) \
     --nginx-version latest
```

Add following to ./config when asked if any custom instructions :

```
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--http-log-path=/var/log/nginx/access.log \
--error-log-path=/var/log/nginx/error.log \
--lock-path=/var/lock/nginx.lock \
--pid-path=/run/nginx.pid \
--user=nginx \
--group=nginx \
--with-debug \
--with-pcre-jit \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_realip_module \
--with-http_auth_request_module \
--with-http_addition_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_geoip_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_image_filter_module \
--with-http_mp4_module \
--with-http_perl_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_v2_module \
--with-http_sub_module \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-stream_ssl_module \
--with-threads \
--with-file-aio \
--with-http_ssl_module \
--with-http_xslt_module \ 
--with-http_degradation_module \
--with-pcre \
--with-google_perftools_module \
--add-module=${DIRECTORY}/headers-more-nginx-module-${HEADERS_VERSION} \
--add-module=${DIRECTORY}/ngx_pagespeed-release-${PAGESPEED_VERSION}-beta

```

### Init scripts

Create nginx.service file :

     nano /lib/systemd/system/nginx.service

And add the following [Original post on Nginx](https://www.nginx.com/resources/wiki/start/topics/examples/systemd/):

```
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Create nginx init.d file :

     nano /etc/init.d/nginx

Add the following :

```
#! /bin/sh
 
### BEGIN INIT INFO
# Provides:          nginx
# Required-Start:    $all
# Required-Stop:     $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
# Description:       starts nginx using start-stop-daemon
### END INIT INFO
 
PATH=/opt/bin:/opt/sbin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/opt/sbin/nginx
NAME=nginx
DESC=nginx
 
test -x $DAEMON || exit 0
 
# Include nginx defaults if available
if [ -f /etc/default/nginx ] ; then
        . /etc/default/nginx
fi
 
set -e
 
case "$1" in
  start)
        echo -n "Starting $DESC: "
        start-stop-daemon --start --quiet --pidfile /var/run/nginx.pid \
                --exec $DAEMON -- $DAEMON_OPTS
        echo "$NAME."
        ;;
  stop)
        echo -n "Stopping $DESC: "
        start-stop-daemon --stop --quiet --pidfile /var/run/nginx.pid \
                --exec $DAEMON
        echo "$NAME."
        ;;
  restart|force-reload)
        echo -n "Restarting $DESC: "
        start-stop-daemon --stop --quiet --pidfile \
                /var/run/nginx.pid --exec $DAEMON
        sleep 1
        start-stop-daemon --start --quiet --pidfile \
                /var/run/nginx.pid --exec $DAEMON -- $DAEMON_OPTS
        echo "$NAME."
        ;;
  reload)
      echo -n "Reloading $DESC configuration: "
      start-stop-daemon --stop --signal HUP --quiet --pidfile /var/run/nginx.pid \
          --exec $DAEMON
      echo "$NAME."
      ;;
  *)
        N=/etc/init.d/$NAME
        echo "Usage: $N {start|stop|restart|force-reload}" >&2
        exit 1
        ;;
esac
 
exit 0
```

Give it executable rights :

     chmod +x /etc/init.d/nginx
     

Create a nginx user :

     useradd nginx
     or
     useradd --no-create-home nginx


Create folders for pagespeed cache and change owner/group :

     mkdir /var/ngx_pagespeed_cache && chown nginx:nginx /var/ngx_pagespeed_cache

### Install php7.0

```
apt-get install -y php7.0-fpm php7.0-gd php7.0-mysql php7.0-cli php7.0-common php7.0-curl php7.0-opcache php7.0-json php7.0-imap php7.0-mbstring php7.0-xml
```

Edit www.conf file :

     nano /etc/php/7.0/fpm/pool.d/www.conf

Make sure following is set :

```
user = nginx
group = nginx
listen = /run/php/php7.0-fpm.sock
listen.owner = nginx
listen.group = nginx
```

Increase php upload_max_filesize limit

     nano /etc/php/7.0/fpm/php.ini

Set

     upload_max_filesize = 64M

Edit fastcgi_params :

     nano /etc/nginx/fastcgi_params

Add the following :

```
fastcgi_param   QUERY_STRING            $query_string;
fastcgi_param   REQUEST_METHOD          $request_method;
fastcgi_param   CONTENT_TYPE            $content_type;
fastcgi_param   CONTENT_LENGTH          $content_length;

fastcgi_param   SCRIPT_FILENAME         $document_root$fastcgi_script_name;
fastcgi_param   SCRIPT_NAME             $fastcgi_script_name;
fastcgi_param   PATH_INFO               $fastcgi_path_info;
fastcgi_param   PATH_TRANSLATED         $document_root$fastcgi_path_info;
fastcgi_param   REQUEST_URI             $request_uri;
fastcgi_param   DOCUMENT_URI            $document_uri;
fastcgi_param   DOCUMENT_ROOT           $document_root;
fastcgi_param   SERVER_PROTOCOL         $server_protocol;

fastcgi_param   GATEWAY_INTERFACE       CGI/1.1;
fastcgi_param   SERVER_SOFTWARE         nginx/$nginx_version;

fastcgi_param   REMOTE_ADDR             $remote_addr;
fastcgi_param   REMOTE_PORT             $remote_port;
fastcgi_param   SERVER_ADDR             $server_addr;
fastcgi_param   SERVER_PORT             $server_port;
fastcgi_param   SERVER_NAME             $server_name;

fastcgi_param   HTTPS                   $https;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param   REDIRECT_STATUS         200;
```

Edit mime.types

     nano /etc/nginx/mime.types
     
Add the following :

     application/x-font-ttf                           ttc ttf;

Edit nginx.conf

     nano /etc/nginx/nginx.conf

Add the following [see full config file](https://raw.githubusercontent.com/shirokoweb/nginx/master/nginx.conf) :

```
# /!\ truncated file
user nginx;

        pagespeed on;
        pagespeed ModifyCachingHeaders on;

        # Needs to exist and be writable by nginx.  Use tmpfs for best performance.
        pagespeed FileCachePath /var/ngx_pagespeed_cache;

        # Ensure requests for pagespeed optimized resources go to the pagespeed handler
        # and no extraneous headers get set.
        location ~ "\.pagespeed\.([a-z]\.)?[a-z]{2}\.[^.]{10}\.[^.]+" {
        add_header "" "";
        }
        location ~ "^/pagespeed_static/" { }
        location ~ "^/ngx_pagespeed_beacon$" { }
```

Restart both php && nginx :

     service php7.0-fpm restart
     service nginx restart

Quick verification

     curl -I -p http://localhost

Should output :

```
HTTP/1.1 200 OK
Server: nginx/1.13.8
Content-Type: text/html
Connection: keep-alive
Vary: Accept-Encoding
Date: Mon, 08 Jan 2018 16:51:31 GMT
X-Page-Speed: 1.12.34.3-0
Cache-Control: max-age=0, no-cache

```

### Testing php

Create test.php file in root folder for the vhost

     nano /usr/local/nginx/html/test.php

Add the following and save :

     <?php var_export($_SERVER)?>

In a browser try to request: # /test.php # /test.php/ # /test.php/foo # /test.php/foo/bar.php # /test.php/foo/bar.php?v=1
ie. http://mydomain.tld/test.php or http://mydomain.tld/test.php/foo/bar.php?v=1 etc.

Pay attention to the value of REQUEST_URI, SCRIPT_FILENAME, SCRIPT_NAME, PATH_INFO and PHP_SELF.

Here is an example for   output :

```
array (
    'USER' => 'nginx', 
    'HOME' => '/home/nginx', 
    'HTTP_CACHE_CONTROL' => 'no-cache', 
    'HTTP_PRAGMA' => 'no-cache', 
    'HTTP_UPGRADE_INSECURE_REQUESTS' => '1', 
    'HTTP_CONNECTION' => 'keep-alive', 
    'HTTP_DNT' => '1', 
    'HTTP_ACCEPT_ENCODING' => 'gzip, deflate', 
    'HTTP_ACCEPT_LANGUAGE' => 'en-US,en;q=0.5', 
    'HTTP_ACCEPT' => 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8', 
    'HTTP_USER_AGENT' => 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:57.0) Gecko/20100101 Firefox/57.0', 
    'HTTP_HOST' => 'domain.tld', 
    'REDIRECT_STATUS' => '200', 
    'HTTPS' => '', 
    'SERVER_NAME' => '_', 
    'SERVER_PORT' => '80', 
    'SERVER_ADDR' => '172.104.243.236', 
    'REMOTE_PORT' => '58546', 
    'REMOTE_ADDR' => '90.112.230.33', 
    'SERVER_SOFTWARE' => 'nginx/1.13.8', 
    'GATEWAY_INTERFACE' => 'CGI/1.1', 
    'SERVER_PROTOCOL' => 
    'HTTP/1.1', 
    'DOCUMENT_ROOT' => '/usr/local/nginx/html', 
    'DOCUMENT_URI' => '/info.php/foo/bar.php', 
    'REQUEST_URI' => '/info.php/foo/bar.php?v=1', 
    'PATH_TRANSLATED' => '/usr/local/nginx/html/foo/bar.php', 
    'PATH_INFO' => '/foo/bar.php', 
    'SCRIPT_NAME' => '/info.php', 
    'SCRIPT_FILENAME' => '/usr/local/nginx/html/info.php', 
    'CONTENT_LENGTH' => '', 
    'CONTENT_TYPE' => '', 
    'REQUEST_METHOD' => 'GET', 
    'QUERY_STRING' => 'v=1', 
    'FCGI_ROLE' => 'RESPONDER', 
    'PHP_SELF' => '/info.php/foo/bar.php', 
    'REQUEST_TIME_FLOAT' => 1515427194.0948319, 
    'REQUEST_TIME' => 1515427194, 
    )
```

### MariaDB

Get and install MariaDB

     apt-get install mariadb-server mariadb-client -y

Edit 50-server.cnf file :

     nano /etc/mysql/mariadb.conf.d/50-server.cnf
     
line 111,112: change like follows

     character-set-server = utf8
     # collation-server = utf8mb4_general_ci
     
Restart MariaDB :

     systemctl restart mariadb
     or
     service maraidb restart

Run initial settings and secure installation

     mysql_secure_installation
     
Add a root pwd :

     Enter current password for root (enter for none): YOUR_PWD_HERE >> PRESS ENTER
     
Answer as follow :

     Change the root password? [Y/n]              ---> n
     Remove anonymous users? [Y/n]                ---> y
     Disallow root login remotely? [Y/n]          ---> y
     Remove test database and access to it? [Y/n] ---> y
     Reload privilege tables now? [Y/n]           ---> y
     
Connect to MariaDB with root :

     mysql -u root -p
     Enter password: YOUR_PWD_HERE >> PRESS ENTER
     
On successful connection :

     Welcome to the MariaDB monitor.  Commands end with ; or \g.
     Your MariaDB connection id is 8
     Server version: 10.1.26-MariaDB-0+deb9u1 Debian 9.1

     Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

     Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
     
Create a DB for WP installation :

     CREATE DATABASE wordpress; GRANT ALL ON wordpress.* TO wpdbuser@localhost IDENTIFIED BY 'PWD_FOR_WPDBUSER';
     
Should output :

     Query OK, 1 row affected (0.00 sec)
     Query OK, 0 rows affected (0.00 sec)
     
Verifi DB creation :

     MariaDB [(none)]> show databases;
     
Should output :

     +--------------------+
     | Database           |
     +--------------------+
     | information_schema |
     | mysql              |
     | performance_schema |
     | wordpress          |
     +--------------------+

Leave MariaDB :

     exit
     
Check MariaDB connection with user wpbuser :

     mysql -u wpdbuser -p
     Enter password: PWD_FOR_WPDBUSER

Leave MariaDB :

     MariaDB [(none)]> exit
     
### Install Wordpress

     cd /usr/local/nginx/html && wget https://wordpress.org/latest.zip && unzip latest.zip && mv wordpress/* . && rmdir wordpress && rm -f latest.zip && chown -R nginx:nginx /usr/local/nginx/html && mv index.html index.html.bkp
     
Access URL http://domain.tld to install WP

### Install Let's Encrypt

get certbot

     apt-get install certbot -y
     
get certificate for domain

     certbot certonly --standalone -d domain.tld -d www.domain.tld
     
 When asked, enter VALID email address (or it will fail) :

     Saving debug log to /var/log/letsencrypt/letsencrypt.log
     Enter email address (used for urgent renewal and security notices) (Enter 'c' to
     cancel): enter_valid_email@someisp.com
     
Answer A to the following question :

     -------------------------------------------------------------------------------
     Please read the Terms of Service at
     https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
     agree in order to register with the ACME server at
     https://acme-v01.api.letsencrypt.org/directory
     -------------------------------------------------------------------------------
     (A)gree/(C)ancel: A

Output :

     Obtaining a new certificate
     Performing the following challenges:
     tls-sni-01 challenge for www.domain.tld
     tls-sni-01 challenge for domain.tld
     Waiting for verification...
     Cleaning up challenges
     Generating key (2048 bits): /etc/letsencrypt/keys/0000_key-certbot.pem
     Creating CSR: /etc/letsencrypt/csr/0000_csr-certbot.pem

     IMPORTANT NOTES:
      - Congratulations! Your certificate and chain have been saved at
        /etc/letsencrypt/live/www.domain.tld/fullchain.pem. Your cert will
        expire on 2018-04-08. To obtain a new or tweaked version of this
        certificate in the future, simply run certbot again. To
        non-interactively renew *all* of your certificates, run "certbot
        renew"
      - If you lose your account credentials, you can recover through
        e-mails sent to valid_email@someisp.com.
      - Your account credentials have been saved in your Certbot
        configuration directory at /etc/letsencrypt. You should make a
        secure backup of this folder now. This configuration directory will
        also contain certificates and private keys obtained by Certbot so
        making regular backups of this folder is ideal.
      - If you like Certbot, please consider supporting our work by:

        Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
        Donating to EFF:                    https://eff.org/donate-le


### Update the nginx.conf

1. Copy the content from [nginxssl.conf file](https://raw.githubusercontent.com/shirokoweb/nginx/master/nginxssl.conf) and replace all domain.tld by real domain name.

2. create cipherlist file :

```
nano /etc/nginx/cipherlist.conf
```

Add following :

     # https://cipherli.st/
     ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH:ECDHE-RSA-AES128-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA128:DHE-RSA-AES128-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA128:ECDHE-RSA-AES128-SHA384:ECDHE-RSA-AES128-SHA128:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES128-SHA128:DHE-RSA-AES128-SHA128:DHE-RSA-AES128-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA384:AES128-GCM-SHA128:AES128-SHA128:AES128-SHA128:AES128-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
     ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
     ssl_prefer_server_ciphers on;
     ssl_session_cache shared:SSL:10m;
     add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
     add_header X-Frame-Options DENY;
     add_header X-Content-Type-Options nosniff;
     #ssl_session_tickets off;
     ssl_stapling on; # Requires nginx >= 1.3.7
     ssl_stapling_verify on; # Requires nginx >= 1.3.7
     resolver 8.8.8.8 8.8.4.4 valid=300s;
     resolver_timeout 5s;

4. Check config & restart nginx :

```
nginx -t
service nginx restart
```

### Create a cronjob to auto-renew cert

Issue the following command :

```
crontab -e
```
     
When asked, asnwer 1 :

     no crontab for root - using an empty one

     Select an editor.  To change later, run 'select-editor'.
       1. /bin/nano        <---- easiest
       2. /usr/bin/vim.basic
       3. /usr/bin/vim.tiny

     Choose 1-3 [1]: 1

Append following cronjob task :

     0 2 * * * certbot renew >> /var/log/letsencrypt.log
     
### Optional log rotation

create nginx logrotate conf file

```
nano /etc/logrotate.d/nginx
```
     
Append the following :

     /var/log/nginx/*.log {
             daily
             dateext
             dateformat .%Y-%m-%d
             missingok
             rotate 52
             compress
             delaycompress
             notifempty
             create 0640 nginx nginx
             sharedscripts
             postrotate
                     [ ! -f /var/run/nginx.pid ] || kill -USR1 `cat /var/run/nginx.pid`
             endscript
     }
     
# Enjoy lightning speed website runing Wordpress
