# Mi WiFi Online Radio wrapper :radio:
Wrapper on Xiaomi Mi WiFi Online radio to add functionality for adding own radio stations. Remember to read the instructions first.

It allows you to create a list of your own radio stations (no limits for Chinese stations only). You can add stations from your region and listen them on your favorite device.

It requires you to redirect subdomains on your router to work correctly and a separate (may be combined with router) web server for working with them.

# Tools needed
1. Mi WiFi Online Radio

    ![Mi WiFi Online Radio](/images/xiaomi.jpg)

2. WiFi Router supporting DNSMasq (DD-WRT supports it)
3. Local/Remote Server with root access for installing programs
    - PHP 5.4+ (allowing you to use **exec** function) (PHP 7 with mysqli supported)
    Dont forget to install php modules, such ad php-gd, php-mbstring, php-mysqli !
    - MySQL
    - Nginx (you can remake everything for Apache as well)
    - ffmpeg + fdk_acc (it may work also with acc codec)
4. Unix knowledge

## Remember
Things will only work when you use your phone in the same WiFi network with the radio or have added radio stations to your 
favorites.

This wrapper works as an interceptor for all your requests to *.ximalaya.com to emulate API. So some things may go broken.

All you are doing may or may not hurt your device and finally brick it :trollface:


# How to configure

1. Config your linux/bsd server to be up and running. Use nginx + PHP 5.4+ + MySQL for that
2. Here is an example of my configuration for domain (using Vesta as my control panel). Pay attention to the lines rewriting m3u8 files and redirecting requests to nonexistant files to our main php-script. Don't forget to replace domainname.com with the one you like (and have access to). This config can be non-functional on your nginx installation, so just pay attention to what i mentioned above and modify it to fit your server.

```
server {
    listen 443 default_server;
    listen [::]:443 default_server;
    server_name vpsXX.ximiraga.ru api.ximalaya.com;
    root        /zdata/www/miradio;
    index       index.php index.html index.htm;
    ssl on;
    ssl_certificate      /etc/ssl/certs/myssl.crt;
    ssl_certificate_key  /etc/ssl/private/myssl.key;
//  access_log  /var/log/nginx/radio_access.log;
//  error_log   /var/log/nginx/radio_error.log;
    location @ximalaya {
      proxy_pass http://api.ximalaya.com;
    }

 location / {
    rewrite ^/(.*).m3u8 /play.php?xid=$1;
    try_files $uri $uri/ @ximalaya;
    location /openapi-gateway-app/live/radios {
        try_files $uri $uri/ /index.php;
    }
    location /openapi-gateway-app/live/get_radios_by_ids {
        try_files $uri $uri/ /index.php;
    }

    location /openapi-gateway-app/search/radios {
    try_files $uri $uri/ /index.php;
    }

    location /openapi-gateway-app/ranks/radios {
    try_files $uri $uri/ /index.php;
    }

    location ~* ^.+\.(jpeg|jpg|png|gif|bmp|ico|svg|css|js)$ {
        expires     max;
    }

    location ~ [^/]\.php(/|$) {
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        if (!-f $document_root$fastcgi_script_name) {
            return  404;
        }

        fastcgi_pass    127.0.0.1:9000;
        fastcgi_index   index.php;
             include         /usr/local/etc/nginx/fastcgi_params;
    }
}

    location ~* "/\.(htaccess|htpasswd)$" {
        deny    all;
        return  404;
    }

}

```

3. Copy the files from this repository to your site's root folder.
4. Configure database credentials inside **config/config.php**
5. Import initial database via phpmyadmin (file **db.sql**)
6. Configure directory **uploads** and file **config/config.php** to be writable
7. Access your wrapper admin panel adding /admin.php to your **domainname.com** from nginx config. Like domainname.com/admin.php. It will guide you through creating an admin account and login.
8. Try to understand the way of creating your own radios in admin panel.
9. Install ffmpeg with fdk-aac. I used the ppa from here: https://launchpad.net/~mc3man/+archive/ubuntu/trusty-media For freebsd you need recompile it from ports.
10. Check your ffmpeg starts and is functioning
11. Add script ffkill.php to cron "*/5 * * * * /usr/local/bin/php /www/miradio/ffkill.php"
12. Now configure DNSMasq on your router to redirect #.ximalaya.com to your server. In DD-WRT this is done in section Services -> Services. Find the textbox **Additional DNSMasq Options** and add the code below, then save configuration and reboot router. You may refer to the image below.

    `address=/.ximalaya.com/192.168.0.100`
    ![DD-WRT DNSMasq](/images/ddwrt.png)
13. Start using your app for the Radio device as usual. Try adding radio station in it and search for the stations you've added in admin panel before.

Sorry for the code quality in wrapper script. It's as old as Ice Age. Feel free to recommend your changes and new wrapper functions.
