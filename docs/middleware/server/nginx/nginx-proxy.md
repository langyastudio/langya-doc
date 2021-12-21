```nginx
user nginx;

worker_processes 8;

error_log logs/error.log;

worker_rlimit_nofile 102400;

events {
    use epoll;
    worker_connections 102400;
}


http{
    sendfile                    on;
    
    tcp_nopush                  on;
    tcp_nodelay                 on;
    keepalive_timeout           600;

    client_max_body_size        200M;
    types_hash_max_size         2048;
    
    proxy_connect_timeout       600;
    proxy_send_timeout          600;
    proxy_read_timeout          600;
    
    send_timeout                600;

    server{
        listen 80;
        server_name boolongo.cn www.boolongo.cn;
        location / {
            root html;
            index index.html index.htm;
            proxy_pass http://webserver;
            proxy_set_header Host $host:$server_port; 
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Real-PORT $remote_port; 
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-For $remote_addr;
        }
    }

    server{
        listen 8211;
        server_name boolongo.cn www.boolongo.cn;
        location / {
            root html;
            index index.html index.htm;
            proxy_pass http://account;
            proxy_set_header Host $host:$server_port; 
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Real-PORT $remote_port; 
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-For $remote_addr;
        }

    }

    upstream webserver{
        ip_hash;
        server 10.127.99.42:80;
        server 10.127.99.43:80;
    }

    upstream account{
        ip_hash;
        server 10.127.99.42:8211;
        server 10.127.99.43:8211;
    }
}
```

