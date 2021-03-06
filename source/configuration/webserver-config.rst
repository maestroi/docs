##################
Webserver configuration
##################

Generally it's advised to use a reverse proxy in front of the flask application.
Below you can find configs for various webservers:

.. contents:: :local:


NGINX
-----
.. code-block:: none

    server {
            listen 80;
            listen [::]:80;

            server_name your.bot.url;
            location / {
                 proxy_pass  http://127.0.0.1:5000;
                 proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
                 proxy_redirect off;
                 proxy_buffering off;
                 proxy_set_header        Host            $host;
                 proxy_set_header        X-Real-IP       $remote_addr;
                 proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                 proxy_set_header        X-Forwarded-Proto $scheme;
             }
             
            location ~* \.(ico|css|js|gif|jpeg|jpg|png|woff|ttf|otf|svg|woff2|eot)$ {
                expires 1d;
                access_log off;
                add_header Pragma public;
                add_header Cache-Control "public, max-age=86400";
                add_header X-Asset "yes";
                proxy_pass http://192.168.1.16:5000;
                proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
                proxy_redirect off;
                proxy_buffering off;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                gzip on;
                gzip_disable "msie6";

                gzip_vary on;
                gzip_proxied any;
                gzip_comp_level 5;
                # gzip_buffers 16 8k;
                gzip_http_version 1.1;
                gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

            }


        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/your.bot.url/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/your.bot.url/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


        if ($scheme != "https") {
            return 301 https://$host$request_uri;
        } # managed by Certbot

    }


Apache2
-------
.. code-block:: none

    <IfModule mod_ssl.c>
    <VirtualHost *:443>
            #server information
            ServerName your.bot.url
            ServerAdmin webmaster@example.com
            RequestHeader set X-Forwarded-Proto "https"

            ProxyPreserveHost Off
            ProxyPass / http://127.0.0.1:5000/
            ProxyPassReverse / http://127.0.0.1:5000/

            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    
            #SSL Settings Managed by certbot
            SSLCertificateFile /etc/letsencrypt/live/your.bot.url/fullchain.pem
            SSLCertificateKeyFile /etc/letsencrypt/live/your.bot.url/privkey.pem
            Include /etc/letsencrypt/options-ssl-apache.conf
    </VirtualHost>
    </IfModule>
