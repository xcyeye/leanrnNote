---
date: 2022/1/14 21:22
---

# nginx配置SSL，并配置负载均衡

这两天在弄nginx，想将picture.cco.vin这个域名的ssl证书给配置上，这个可以直接看官网的demo就可以实现，但是在配置负载均衡的时候，一直遇到问题

```conf
http {
upstream myapp1 {
        least_conn;
        server localhost:8900;
        server localhost:8901;
    }
    # 这里配置域名指向:8900这个端口
    server {
        listen 80;
        
        server_name picture.cco.vin;
        location / {
				#http自动重定向到https
                rewrite ^(.*)$ https://picture.cco.vin;
        }
    }

    # 这里为picture.cco.vin配置ssl证书 
    server {
        listen 443 ssl;
        server_name picture.cco.vin;

        ssl_certificate      cert/picture_cco_vin/6270676_picture.cco.vin.pem;
        ssl_certificate_key  cert/picture_cco_vin/6270676_picture.cco.vin.key;

        ssl_session_timeout  5m;
        ssl_ciphers  ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers  on;

        location / {
        		#proxy_pass不能是https://myapp1;否则会报错
                proxy_pass http://myapp1;
                index index.html;
        }
    }
}
```

