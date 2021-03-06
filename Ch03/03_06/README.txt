# RUN THESE COMMANDS ON YOUR LOCAL WORKSTATION
    # Start the virtual machine and log in
    vagrant up
    vagrant ssh

# Nginx, PHP, amd MariaDB are installed and configured for you in this lesson.
# Proceed with the following steps to complete the configuration.

# RUN THESE COMMANDS ON THE VIRTUAL MACHINE
    sudo su -

# Install openssl if its not installed
    apt -y install openssl

# Create an SSL key and certificate
    openssl req -batch -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx.key -out /etc/ssl/certs/nginx.crt 2>/dev/null

# Edit /etc/nginx/conf.d/wisdompetmed.local.conf so that is has the
# following contents and then save the file:
    server {
        listen 80 default_server;
        return 301 https://$server_addr$request_uri;
    }

    server {
        listen 443 ssl default_server;
        ssl_certificate /etc/ssl/certs/nginx.crt;
        ssl_certificate_key /etc/ssl/private/nginx.key;

        root /var/www/wisdompetmed.local;

        server_name wisdompetmed.local www.wisdompetmed.local;

        index index.html index.htm index.php;

        access_log /var/log/nginx/wisdompetmed.local.access.log;
        error_log /var/log/nginx/wisdompetmed.local.error.log;

        location / {
            # First attempt to serve request as file, then
            # as directory, then fall back to displaying a 404.
            try_files $uri $uri/ =404;
        }


        location /appointments/ {
            # only allow IPs from the same network the server is on
            allow 192.168.0.0/24;
            allow 10.0.0.0/8;
            deny all;

            auth_basic "Authentication is required...";
            auth_basic_user_file /etc/nginx/passwords;

            location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
                fastcgi_intercept_errors on;
            }
        }

        location /deny {
            deny all;
        }

        location /images/ {
            # Allow the contents of the /image folder to be listed
            autoindex on;
            access_log /var/log/nginx/wisdompetmed.local.images.access.log;
            error_log /var/log/nginx/wisdompetmed.local.images.error.log;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
            fastcgi_intercept_errors on;
        }

        error_page 401 /401.html;
        location = /401.html {
            internal;
        }

        error_page 403 /403.html;
        location = /403.html {
            internal;
        }

        error_page 404 /404.html;
        location = /404.html {
            internal;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            internal;
        }

        location = /500 {
            fastcgi_pass unix:/this/will/fail;
        }
    }

# Test and reload the configuration
    nginx -t
    systemctl reload nginx

# Load the home page in a browser: http://192.168.0.3/

# Proceed as needed to confirm and accept any SSL warnings for your browser