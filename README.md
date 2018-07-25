# elasticsearch-cluster

Multi master durumunda clientlar hagi masterin ipsini kullanacak?
Kaynak: https://dzone.com/articles/nginx-and-elasticsearch

masterların onunde bir nginx olmalı configi su sekilde:

service nginx stop
sudo nano /etc/nginx/nginx.conf

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        upstream elasticcluster{
                server 10.10.10.55:9200 weight=1 fail_timeout=1;
                server 10.10.10.53:9200 weight=1 fail_timeout=1;
        }

        # BUNLARDA BİR HATA VAR
        #sendfile        on;
        #keepalive_timeout  65;

        server {
                listen       8080;
                server_name  127.0.0.1;

                location / {
                        proxy_pass  http://elasticcluster;
                        proxy_connect_timeout 1;
                }

                error_page   500 502 503 504  /50x.html;
                location = /50x.html {
                root   html;
                }

        }

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;
        
         ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;
        gzip_disable "msie6";

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}

```
service nginx start
service nginx restart vb de kullanılabilir.


Tunnelling gerekirse :
```
ssh -L9200:10.10.10.70:8080 root@18.218.170.149
```

Burada 10.10.10.70 in 8080 portuna 18.218.170.149 routeri uzerinden baglanıp bizim 9200 portumuza baglıyor.
Sonrada localhost:9200 u kullanarak 10 lu ip deki makinanın istedigimiz portuna erişiyoruz.
10 lu ip de ki 8080 portunun configurasyonu yukardaki nginx conf ile yapıldı.
