# nginx

## Getting Started

Intro TBD

### Setup timezone

Set accordingly

```
dpkg-reconfigure tzdata
```

### Update and install tools

```
apt-get update
apt-get install ntp zip libxslt1-dev libssl-dev build-essential zlib1g-dev libpcre3-dev libssl-dev libxslt1-dev libxml2-dev libgd2-xpm-dev libgeoip-dev libgoogle-perftools-dev libperl-dev
apt-get update
```

### Create source list for nginx

Create the file :

```
nano /etc/apt/sources.list.d/nginx.list
```

Add the following :

```
deb http://nginx.org/packages/debian/ stretch nginx
deb-src http://nginx.org/packages/debian/ stretch nginx
```

Update

```
apt-get update
```

Add the key and update

```
wget -q "http://nginx.org/packages/keys/nginx_signing.key" -O-| sudo apt-key add -
apt-get update
```

### Compile ngx source /w pagespeeed

Get latest pagespeed version from source (automated install)

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

```
nano /lib/systemd/system/nginx.service
```

And add the following :

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

```
nano /etc/init.d/nginx
```

Add following :

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

```
chmod +x /etc/init.d/nginx
```

Create folders for pagespeed cache and change owner/group :

```
mkdir /var/ngx_pagespeed_cache
chown nginx:nginx /var/ngx_pagespeed_cache
```

If you get a failure regarding nginx, you can add the user...

```
useradd nginx
or
useradd --no-create-home nginx

```

### Install php7.0

```
apt-get install -y php7.0-fpm php7.0-gd php7.0-mysql php7.0-cli php7.0-common php7.0-curl php7.0-opcache php7.0-json php7.0-imap php7.0-mbstring php7.0-xml
```

Edit www.conf file :

```
nano /etc/php/7.0/fpm/pool.d/www.conf
```

Make sure following is set :

```
user = nginx
group = nginx
listen = /run/php/php7.0-fpm.sock
listen.owner = nginx
listen.group = nginx
```

Edit nginx.conf

```
nano /etc/nginx/nginx.conf
```

Add the following :

```
# please [see full config file](https://raw.githubusercontent.com/shirokoweb/nginx/master/nginx.conf)
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
        
        location ~ [^/]\.php(/|$) {
            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            if (!-f $document_root$fastcgi_script_name) {
                return 404;
            }

         # Mitigate https://httpoxy.org/ vulnerabilities
         fastcgi_param HTTP_PROXY "";

         fastcgi_pass   unix:/run/php/php7.0-fpm.sock;
         fastcgi_index index.php;

         # include the fastcgi_param setting
         include fastcgi_params;

         # SCRIPT_FILENAME parameter is used for PHP FPM determining
         #  the script name. If it is not set in fastcgi_params file,
         # i.e. /etc/nginx/fastcgi_params or in the parent contexts,
         # please comment off following line:
         fastcgi_param  SCRIPT_FILENAME   $document_root$fastcgi_script_name;
        }
```

Restart both php && nginx :

```
service php7.0-fpm restart
service nginx restart
```

Quick verification

```
curl -I -p http://localhost
```

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

### Cheching for PHP

Create info.php test file in root folder for vhost

```
nano /usr/local/nginx/html/test.php
```

Add the following and save :

```
<?php var_export($_SERVER)?>
```

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
    'HTTP_HOST' => 'comment.lol', 
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
