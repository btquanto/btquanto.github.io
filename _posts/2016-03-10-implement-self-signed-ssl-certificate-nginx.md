---
title: Implement a self-signed SSL certificate to nginx
category: System Admin
tags: ["linux", "nginx", "ssl", "tutorial"]
---

1. Generate `myapp.key`, and `myapp.crt`. The location is not really important, but make sure that the user that runs nginx has access to it.

    ```
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /path/to/myapp.key -out /path/to/myapp.crt
    ```

2. Then, add ssl configuration to your nginx site config

    ``` nginx
    listen 443 ssl;
    ssl_certificate       /path/to/myapp.crt;
    ssl_certificate_key   /path/to/myapp.key;
    ```

    For example:

    ``` nginx
    server {
        listen 443 ssl;
        ssl on;

        # Edit these as fit
        server_name           domain.dot.com;
        ssl_certificate       /path/to/myapp.crt;
        ssl_certificate_key   /path/to/myapp.key;

        root                  /path/to/my/app/root/foler;
        access_log            /path/to/log/folder/access.log;
        error_log             /path/to/log/folder/logs/error.log;

        location / {
            try_files         $uri @flask;
        }

        # My app is a flask app, served at localhost:8000
        location @flask {
            proxy_set_header   Host $http_host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_redirect     off;
            proxy_pass         http://127.0.0.1:8000;
        }
    }
    ```

3. However, your site not is not serving port `80`. To redirect requests from port `80` to port `443` automatically, add another server block

    ``` nginx
    server {
        listen 80;
        server_name domain.dot.com;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        ssl on;

        # Edit these as fit
        server_name           domain.dot.com;
        ssl_certificate       /path/to/myapp.crt;
        ssl_certificate_key   /path/to/myapp.key;

        root                  /path/to/my/app/root/foler;
        access_log            /path/to/log/folder/access.log;
        error_log             /path/to/log/folder/logs/error.log;

        location / {
            try_files         $uri @flask;
        }

        # My app is a flask app, served at localhost:8000
        location @flask {
            proxy_set_header   Host $http_host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_redirect     off;
            proxy_pass         http://127.0.0.1:8000;
        }
    }
    ```

