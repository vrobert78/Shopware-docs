# Nginx

::: info
The document root must always point to the public folder, to ensure all functionality works.
:::

```text
server {
    listen 80;

    index index.php index.html;
    server_name localhost;

    client_max_body_size 128M;

    root __DOCUMENT_ROOT__/public;

    # Shopware install / update    
    location /shopware-installer.phar.php {
    try_files $uri /shopware-installer.phar.php$is_args$args;
    }
    
    location ~ ^/shopware-installer\.phar\.php/.+\.(?:css|js|png|svg|woff)$ {
     try_files $uri /shopware-installer.phar.php$is_args$args;
    }

    # Deny access to . (dot) files
    location ~ /\. {
        deny all;
    }
    
    # Deny access to .php files in public directories
    location ~ ^/(media|thumbnail|theme|bundles|sitemap).*\.php$ {
        deny all;
    }
    
    location ~ ^/(theme|media|thumbnail|bundles|css|fonts|js|recovery|sitemap)/ {
        expires 1y;
        add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        log_not_found off;
        tcp_nodelay off;
        open_file_cache max=3000 inactive=120s;
        open_file_cache_valid 45s;
        open_file_cache_min_uses 2;
        open_file_cache_errors off;
    
        location ~* ^.+\.svg {
            add_header Content-Security-Policy "script-src 'none'";
            add_header Cache-Control "public, must-revalidate, proxy-revalidate";
            log_not_found off;
        }
    }

    location ~* ^.+\.(?:css|cur|js|jpe?g|gif|ico|png|svg|webp|html|woff|woff2|xml)$ {
        expires 1y;
        add_header Cache-Control "public, must-revalidate, proxy-revalidate";

        access_log off;

        # The directive enables or disables messages in error_log about files not found on disk.
        log_not_found off;

        tcp_nodelay off;

        ## Set the OS file cache.
        open_file_cache max=3000 inactive=120s;
        open_file_cache_valid 45s;
        open_file_cache_min_uses 2;
        open_file_cache_errors off;

        try_files $uri /index.php$is_args$args;
    }

    location ~* ^.+\.svg$ {
        add_header Content-Security-Policy "script-src 'none'";
    }

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi.conf;
        fastcgi_param HTTP_PROXY "";
        fastcgi_buffers 8 16k;
        fastcgi_buffer_size 32k;
        proxy_connect_timeout 300s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;
        send_timeout 300s;
        client_body_buffer_size 128k;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```
