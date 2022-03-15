# Create a shared hosting environment with multiple users and multiple PHP versions

## Spin up the CloudFormation for wp surestack

* Download the cloudformation template: [WPSureStack.json](../WPSureStack.json)
* Go to AWS > CloudFormation > Create Stack
* Fill out the appropriate fields

Notes: 

* make sure your InputCIDR IP address has a **/32** at the end of it. Or you can use `0.0.0.0/0` as a last resort.
* keep the defaults of t2.micro EC2 and t2.small aurora, which are the cheapest options
* it takes a while to spin up, because it's provisioning an aurora cluster (took me about 25 minutes)
* once it's done, go to the **Outputs** tab and get the `EIPDNS` value (public IP)
* paste this ip address into the browser, and this is your default website

## Prep additional users

SSH into the EC2 instance. The rest is going to be done via CLI.

Then elevate to root

```
sudo su
```

Each sites will run under the context of an individual user.

```
sudo groupadd siteone
sudo useradd -g siteone siteone
sudo groupadd sitetwo
sudo useradd -g sitetwo sitetwo
```

## Install PHP

PHP 7.0 is already installed, which siteone will use. 
Sitetwo will use PHP 5.6. 

```
yum install php56 php56-mysqlnd php56-fpm -y
chkconfig php-fpm-5.6 on
```

## Add second site

### Provision a new database:

Get the aurora cluster endpoint:

```
(cd /var/www/html/wordpress && wp config get --fields=value --constant=DB_HOST --allow-root)
```

Create a new database and connection string

```
mysql \
  -h <aurora cluster endpoint> \
  -P 3306 \
  -u admin \
  -p<master password> \
  -e "CREATE DATABASE sitetwodb; CREATE USER 'sitetwoconn' IDENTIFIED BY '<site two password>'; GRANT ALL PRIVILEGES ON sitetwodb.* TO 'sitetwoconn'; FLUSH PRIVILEGES;"
```

* Replace the cluster public DNS. You can copy this value from the default site's wp-config.php
* Replace <master password> with the database master password. If you forget your database master password, you can reset it in RDS. Just modify the instance, and it lets you type in a new master password. Remember to choose the option to `Apply Immediately`. 
* Note that there is no space between -p and the master password
* Replace the `sitetwodb` and `sitetwoconn` (in two places each) with your own values

### Install WordPress

Create a new directory:

```
mkdir /var/www/html/sitetwo && cd $_
```

Download WordPress:

```
wp core download --allow-root
```

Configure wp-config.php:

```
wp core config \
  --dbname=sitetwodb \
  --dbuser=sitetwoconn \
  --dbpass=<site two password> \
  --dbhost=<aurora cluster endpoint> \
  --allow-root
```

Replace the values above, as necessary.

Install WordPress:

```
wp core install \
  --url=sitetwo.com  \
  --title=SiteTwo \
  --admin_user=supervisor \
  --admin_password=<strongpassword> \
  --admin_email=info@example.com  \
  --allow-root
```

Replace the values above, as necessary.

## Create conf files

Don't worry, this section looks crazy, but it's all copy paste.

### Nginx conf files

Delete the existing wordpress.conf file (you'll replace it momentarily)

```
rm -f /etc/nginx/conf.d/wordpress.conf
```

Create a conf file for siteone:

```
/etc/nginx/conf.d/siteone.conf
```

And use the following contents:

```
gzip on;
gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;
gzip_vary on;

fastcgi_cache_path /var/lib/nginx/siteone levels=1:2 keys_zone=siteone:1440m;
fastcgi_cache_key  "$scheme$request_method$host$request_uri";

fastcgi_ignore_headers Cache-Control Expires Set-Cookie;

map $sent_http_content_type $expires {
    default                    off;
    text/html                  epoch;
    text/css                   max;
    application/javascript     max;
    ~image/                    max;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    listen *:443 ssl http2;
    listen [::]:443 ssl http2;

    expires $expires;

    client_max_body_size 100M;
    
    # Enable HSTS. This forces SSL on clients that respect it, most modern browsers. The includeSubDomains flag is optional.
    #add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";

    ssl_session_cache shared:SSL:20m;
    ssl_session_timeout 10m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5';

    access_log  /var/log/nginx/siteone-access.log;
    error_log /var/log/nginx/siteone-error.log;

    set $no_cache 0;
    # POST requests and urls with a query string should always go to PHP
    if ($request_method = POST) {
        set $no_cache 1;
    }
    if ($query_string != "") {
       set $no_cache 1;
    }

    if ($request_uri ~* "(/wp-admin/|/xmlrpc.php|/wp-(app|cron|login|register|mail).php|wp-.*.php|/feed/|index.php|wp-comments-popup.php|wp-links-opml.php|wp-locations.php|sitemap(_index)?.xml|[a-z0-9_-]+-sitemap([0-9]+)?.xml)") {
        set $no_cache 1;
    }

    # Don't use the cache for logged in users or recent commenters
    if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_no_cache|wordpress_logged_in") {
        set $no_cache 1;
    }

    root /var/www/html/wordpress/;
    index index.php index.html index.htm;

    server_name siteone.com; 
    ssl_certificate /etc/nginx/ssl/wordpress.bundle.crt;
    ssl_certificate_key /etc/nginx/ssl/wordpress.key;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi.conf;
        fastcgi_intercept_errors on;
        #fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_keep_conn on;
        fastcgi_index index.php;
        fastcgi_cache siteone;
        fastcgi_cache_methods GET HEAD;
        fastcgi_cache_valid 200 100m;
        fastcgi_cache_bypass $no_cache;
        fastcgi_no_cache     $no_cache;
    }

    location = /robots.txt { access_log off; log_not_found off; }

    location ~* \.(eot|ttf|woff|woff2)$ {
       add_header Access-Control-Allow-Origin '*';
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Create a conf file for sitetwo:

```
/etc/nginx/conf.d/sitetwo.conf
```

And use the following contents:

```
fastcgi_cache_path /var/lib/nginx/sitetwo levels=1:2 keys_zone=sitetwo:1440m;

server {
    listen 80;
    listen [::]:80;

    listen *:443 ssl http2;
    listen [::]:443 ssl http2;

    expires $expires;

    client_max_body_size 100M;
    
    # Enable HSTS. This forces SSL on clients that respect it, most modern browsers. The includeSubDomains flag is optional.
    #add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";

    ssl_session_cache shared:SSL:20m;
    ssl_session_timeout 10m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5';

    access_log  /var/log/nginx/sitetwo-access.log;
    error_log /var/log/nginx/sitetwo-error.log;

    set $no_cache 0;
    # POST requests and urls with a query string should always go to PHP
    if ($request_method = POST) {
        set $no_cache 1;
    }
    if ($query_string != "") {
       set $no_cache 1;
    }

    if ($request_uri ~* "(/wp-admin/|/xmlrpc.php|/wp-(app|cron|login|register|mail).php|wp-.*.php|/feed/|index.php|wp-comments-popup.php|wp-links-opml.php|wp-locations.php|sitemap(_index)?.xml|[a-z0-9_-]+-sitemap([0-9]+)?.xml)") {
        set $no_cache 1;
    }

    # Don't use the cache for logged in users or recent commenters
    if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_no_cache|wordpress_logged_in") {
        set $no_cache 1;
    }

    root /var/www/html/sitetwo/;
    index index.php index.html index.htm;

    server_name sitetwo.com; 
    ssl_certificate /etc/nginx/ssl/wordpress.bundle.crt;
    ssl_certificate_key /etc/nginx/ssl/wordpress.key;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi.conf;
        fastcgi_intercept_errors on;
        #fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_pass 127.0.0.1:9001;
        fastcgi_keep_conn on;
        fastcgi_index index.php;
        fastcgi_cache sitetwo;
        fastcgi_cache_methods GET HEAD;
        fastcgi_cache_valid 200 100m;
        fastcgi_cache_bypass $no_cache;
        fastcgi_no_cache     $no_cache;
    }

    location = /robots.txt { access_log off; log_not_found off; }

    location ~* \.(eot|ttf|woff|woff2)$ {
       add_header Access-Control-Allow-Origin '*';
    }

    location ~ /\.ht {
        deny all;
    }
}
```

### PHP-FPM conf files

Remove any default conf files:

```
rm -f /etc/php-fpm-7.0.d/www.conf
rm -f /etc/php-fpm-5.6.d/www.conf
```

Create a conf file for siteone:

```
/etc/php-fpm-7.0.d/siteone.conf
```

And use the following contents:

```
[siteone]
user = siteone
group = siteone
listen = 127.0.0.1:9000
listen.owner = nginx
listen.group = nginx
listen.mode = 0664
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 5
pm.start_servers = 1
pm.min_spare_servers = 1
pm.max_spare_servers = 3

slowlog = /var/log/php-fpm/siteone-slow.log

php_admin_value[error_log] = /var/log/php-fpm/7.0/siteone-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/7.0/session
php_value[soap.wsdl_cache_dir]  = /var/lib/php/7.0/wsdlcache
```

Create a conf file for sitetwo:

```
/etc/php-fpm-5.6.d/sitetwo.conf
```

And use the following contents:

```
[sitetwo]
user = sitetwo
group = sitetwo
listen = 127.0.0.1:9001
listen.acl_users = apache,nginx
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3

slowlog = /var/log/php-fpm/sitetwo-slow.log

php_admin_value[error_log] = /var/log/php-fpm/5.6/sitetwo-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path] = /var/lib/php/5.6/session
php_value[soap.wsdl_cache_dir]  = /var/lib/php/5.6/wsdlcache
```

There's a lot going on here, which I'll explain after everything is done.

## Tweak the init.d scripts

Edit this file:

```
/etc/init.d/php-fpm-7.0
```

And change this:

```
daemon --pidfile ${pidfile} /usr/sbin/php-fpm-7.0 --daemonize $OPTIONS
```

to this:

```
daemon --pidfile ${pidfile} /usr/sbin/php-fpm-7.0 -y /etc/php-fpm-7.0.conf  --daemonize $OPTIONS
```

Also, edit this file:

```
/etc/init.d/php-fpm-5.6
```

And change this:

```
daemon --pidfile ${pidfile} php-fpm-5.6 $OPTIONS
```

to this:

```
daemon --pidfile ${pidfile} php-fpm-5.6 -y /etc/php-fpm-5.6.conf $OPTIONS
```

### Restart everything

```
service nginx restart
service php-fpm-5.6 restart
service php-fpm-7.0 restart
```

## Point your local Mac's DNS to the two sites

Open a new Terminal window for your Local Mac.

```
sudo vi /etc/hosts
```

Add these entries (replace the IP address):

```
35.170.89.92 siteone.com
35.170.89.92 sitetwo.com
```

Restart DNS on your Mac

```
sudo killall -HUP mDNSResponder
```

Visit each site and make sure that they work

```
http://siteone.com
http://sitetwo.com
```

You can also drop a phpinfo into each docroot and verify that they're running PHP 7.0 and 5.6, respectively.

## Explanation of what happened

You have two sites configured:

**Siteone** (default site):

* PHP 7.0
* Nginx conf: /etc/nginx/conf.d/siteone.conf
* Docroot: /var/www/html/wordpress
* Cache location: /var/lib/nginx/siteone
* Cache Zone: siteone
* Server name: siteone.com
* fastcgi_pass: Port 9000
* PHP-FPM conf: /etc/php-fpm-7.0.d/siteone.conf
* PHP-FPM user: siteone
* PHP-FPM port: Port 9000

**Sitetwo** (additional site):

* PHP 5.6
* Nginx conf: /etc/nginx/conf.d/sitetwo.conf
* Docroot: /var/www/html/sitetwo
* Cache location: /var/lib/nginx/sitetwo
* Cache Zone: sitetwo
* Server name: sitetwo.com
* fastcgi_pass: Port 9001
* PHP-FPM conf: /etc/php-fpm-5.6.d/sitetwo.conf
* PHP-FPM user: sitetwo
* PHP-FPM port: Port 9001

Here's what's going on:

* You have two sites: siteone and sitetwo
* They're running as different users
* They're running under different versions of PHP. 7.0 on port 9000, and 5.6 on port 9001.
* The cache zones are separated out as well

The key to making this all work is updating the `/etc/init.d` scripts. You use the `-y` flag to point to the version specific conf folder for PHP. Otherwise, by default both will point to `/etc/php-fpm.conf` which is the most recently installed version of PHP (i.e 5.6).

## Post config

Once this is working, you might want to fix the docroot permissions so they're only accessible by each user.

You might want to lock down `php_admin_value` and `php_admin_flag` per this [article](https://www.digitalocean.com/community/tutorials/how-to-host-multiple-websites-securely-with-nginx-and-php-fpm-on-ubuntu-14-04).

You can tweak the number of children in the PHP-FPM conf files to tune performance.

HSTS is disabled, since the default install uses a self-signed cert. You can re-enable HSTS once you have a real SSL certificate.

Note: since you are modifying the /etc/init.d scripts, be careful when you run yum updates. If PHP is updated, you may need to re-apply the `-y` config option again, if your change gets overwritten.
