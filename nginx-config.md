# NGINX Config

```
server {

    listen 9000;
    server_name _;

    root /var/www/workspace/support;

    index index.php index.html index.htm;

    client_max_body_size 2000M;
    client_body_buffer_size 100M;
    client_header_buffer_size 10M;
    large_client_header_buffers 2 10M;

    client_body_timeout 30;
    client_header_timeout 30;
    keepalive_timeout 70;
    send_timeout 20;

    include         mime.types;
    default_type    application/octet-stream;
    sendfile        on;
    charset         utf-8;
    gzip            on;
    gzip_comp_level 2;
    gzip_proxied    expired no-cache no-store private auth;
    gzip_types      text/plain application/x-javascript text/xml text/css application/xml;
    gzip_min_length 1000;

    set $path_info "";

    location ~ /include {
        deny all;
        return 403;
    }

    if ($request_uri ~ "^/api(/[^\?]+)") {
        set $path_info $1;
    }

    # /api/*.* should be handled by /api/http.php if the requested file does not exist
    location ~ ^/api/(?:tickets|tasks)(.*) {
        try_files $uri $uri/ /api/http.php?$query_string;
    }

    # /scp/ajax.php needs PATH_INFO too, possibly more files need it hence the .*\.php
    if ($request_uri ~ "^/scp/.*\.php(/[^\?]+)") {
        set $path_info $1;
    }

    if ($request_uri ~ "^/.*\.php(/[^\?]+)") {
        set $path_info $1;
    }

    # Make sure requests to /scp/ajax.php/some/path get handled by ajax.php
    location ~ ^/scp/ajax.php/(.*) {
        try_files $uri $uri/ /scp/ajax.php?$query_string;
    }

    location ~ ^/ajax.php/(.*) {
        try_files $uri $uri/ /ajax.php?$query_string;
    }

    location / {
        index     index.php;
        #try_files $uri $uri/ index.php;
    }

    location ~ \.php$ {
        try_files      $uri =404;
        include        snippets/fastcgi-php.conf;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass   unix:/run/php/php7.2-fpm.sock;
        include        fastcgi_params;
        fastcgi_param  PATH_INFO        $path_info;
    }
}
```
