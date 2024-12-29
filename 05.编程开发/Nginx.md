```nginx
http {
    # 负载均衡，weight轮询的权重
    upstream backend {
        least_conn;
        server backend1.example.com weight=3 max_fails=3 fail_timeout=30s;
        server backend2.example.com weight=2 max_fails=3 fail_timeout=30s;
        server backend3.example.com weight=1 max_fails=3 fail_timeout=30s;
    }
	# 转发到内网其他服务器
    server {
        listen 80;
		server name 域名;
 
        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            deny 127.0.0.1;  #拒绝的ip
            allow 172.18.5.54; #允许的ip
        }
    }
    
  	server {
        listen       80;
        server_name  demo.域名.com;

        location / {
            charset utf-8;
            root  /electronic-book-demo/build/;
            index  index.html index.htm;
        }
	}
}
```



