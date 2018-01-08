# nginx

dpkg-reconfigure tzdata

apt-get update
apt-get install ntp zip libxslt1-dev libssl-dev build-essential zlib1g-dev libpcre3-dev libssl-dev libxslt1-dev libxml2-dev libgd2-xpm-dev libgeoip-dev libgoogle-perftools-dev libperl-dev
apt-get update

nano /etc/apt/sources.list.d/nginx.list

deb http://nginx.org/packages/debian/ stretch nginx
deb-src http://nginx.org/packages/debian/ stretch nginx

apt-get update

wget -q "http://nginx.org/packages/keys/nginx_signing.key" -O-| sudo apt-key add -
apt-get update

bash <(curl -f -L -sS https://ngxpagespeed.com/install) \
     --nginx-version latest


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


nano /lib/systemd/system/nginx.service

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


nano /etc/init.d/nginx

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

chmod +x /etc/init.d/nginx


apt-get install -y php7.0-fpm php7.0-gd php7.0-mysql php7.0-cli php7.0-common php7.0-curl php7.0-opcache php7.0-json php7.0-imap php7.0-mbstring php7.0-xml

nano /etc/php/7.0/fpm/pool.d/www.conf

user = nginx
group = nginx
listen = /run/php/php7.0-fpm.sock
listen.owner = nginx
listen.group = nginx


nano /etc/nginx/nginx.conf

user nginx;

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

service php7.0-fpm restart
service nginx restart

<?php   phpinfo(); ?>

<?php var_export($_SERVER)?>
