# Bastillion(웹 기반 SSH 관리 콘솔)

Bastilion은 웹 기반 SSH 콘솔. SSH 공개 키를 통해 시스템에 연결할 수 있다. 인프라 노출을 최소화 하면서 JUMPBOX 용도로 사용할 수 있다.

자세한 내용은 [여기](https://github.com/bastillion-io/Bastillion#bastillion)

# Docker Install
```
docker run -d -p 8080:8080 -p 8443:8443 --net=jump --rm --name jump \
docker run -d -p 8080:8080 -p 8443:8443 --net=jump --rm --name jump \
-v /home/sigongweb/keydb:/opt/bastillion/jetty/bastillion/WEB-INF/classes/keydb \
-v /home/sigongweb/BastillionConfig.properties:/opt/BastillionConfig.properties.tpl \
-v /home/sigongweb/start.ini:/opt/bastillion/start.ini ecosi/bastillion
```
- ***BastillionConfig.properties*** 를 편집
    + Bastilion이 타겟 서버의 SSH 공개키를 덮어 씌우지 않고 추가하도록 설정한다.
    + Two factor 인증 강제
    + 세션 타임아웃 늘리기
```
#set to false to disable key management. If false, the Bastillion public key will be appended to the authorized_keys file (instead of it being overwritten completely).
keyManagementEnabled=false
```
   
```
#enable two-factor authentication with a one-time password - 'required', 'optional', or 'disabled'
oneTimePassword=required
```
    
```
#The session time out value of application in minutes
sessionTimeout=120
```
- ***start.ini*** 편집
```
# This Jetty startup is per-se insecure as no TLS is activated.
# Make sure you use a reverse proxy in front of it for proper TLS
# termination

--module=client
--module=ext
--module=server
--module=jsp
--module=resources
--module=deploy
--module=jstl
--module=websocket
--module=http

jetty.http.port=8080
---
```
# Reverse Proxy와 연결하여 HTTPS로 접근한다.
```
# Ansible managed

upstream myjumpbox.com {
            server jump:8080;
    }

server {
    listen 80;
    server_name myjumpbox.com;
    proxy_next_upstream error timeout http_404 http_500 http_502 http_503;
    access_log /var/log/nginx/http-myjumpbox.com-access.log;
    error_log /var/log/nginx/http-myjumpbox.com-error.log;
    include /etc/nginx/conf.d/server.rewrites;

    location /.well-known/acme-challenge {
        root /var/www/letsencrypt;
        try_files $uri $uri/ =404;
    }

    location / {
        proxy_pass  http://myjumpbox.com;
        resolver 127.0.0.11 valid=30s ipv6=off;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Ssl on;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_connect_timeout 120;
        proxy_read_timeout 120;
        proxy_send_timeout 120;
    }
}

server {
    listen 443 ssl;
    server_name myjumpbox.com;
    access_log /var/log/nginx/https-myjumpbox.com-access.log;
    error_log /var/log/nginx/https-myjumpbox.com-error.log;
    proxy_next_upstream error timeout http_404 http_500 http_502 http_503;
    ssl_certificate /etc/letsencrypt/live/myjumpbox.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myjumpbox.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/myjumpbox.com/fullchain.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass  http://myjumpbox.com;
        resolver 127.0.0.11 valid=30s ipv6=off;
        proxy_intercept_errors on;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Ssl on;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header Accept-Encoding "";
        proxy_connect_timeout 1800;
        proxy_read_timeout 120;
        proxy_send_timeout 120;
    }
}
```