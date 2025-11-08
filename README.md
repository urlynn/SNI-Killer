### 自签名证书

- 根证书

  > `/etc/nginx/ca/root/root_ca.cnf`
  >
  > ```shell
  > [ req ]
  > prompt = no
  > distinguished_name = req_distinguished_name
  > x509_extensions = v3_ca
  > 
  > [ req_distinguished_name ]
  > C = CN
  > ST = Shanghai
  > L = Shanghai
  > O = Urlynn
  > CN = Urlynn Root CA
  > 
  > [ v3_ca ]
  > subjectKeyIdentifier = hash
  > authorityKeyIdentifier = keyid:always,issuer
  > basicConstraints = critical, CA:true
  > keyUsage = critical, cRLSign, keyCertSign
  > ```

​	`openssl ecparam -genkey -name secp384r1 -out root_ca_ecc.key` #*生成 ECC 私钥*

​	`openssl req -new -x509 -key root_ca_ecc.key -out root_ca_ecc.crt -days 3650 -config root_ca_ecc.cnf` *# 生成自签名的 ECC 根证书*

- 签发终端证书

  > `/etc/nginx/ca/github/github_ecc.cnf `
  >
  > ```shell
  > [ req ]
  > prompt = no
  > distinguished_name = req_distinguished_name
  > req_extensions = v3_req
  > 
  > [ req_distinguished_name ]
  > C = CN
  > ST = Shanghai
  > L = Shanghai
  > O = Urlynn
  > CN = GitHub Proxy
  > 
  > [ v3_req ]
  > basicConstraints = CA:FALSE
  > keyUsage = digitalSignature, keyAgreement
  > extendedKeyUsage = serverAuth
  > subjectAltName = @alt_names
  > 
  > [ alt_names ]
  > # 主域名
  > DNS.1 = github.com
  > DNS.2 = *.github.com
  > DNS.3 = *.github.io
  > DNS.4 = *.githubusercontent.com
  > DNS.5 = github.githubassets.com
  > DNS.6 = analytics.githubassets.com
  > ```

  > `/etc/nginx/ca/steam/steam_ecc.cnf`
  >
  > ```shell
  > [ req ]
  > prompt = no
  > distinguished_name = req_distinguished_name
  > req_extensions = v3_req
  > 
  > [ req_distinguished_name ]
  > C = CN
  > ST = Shanghai
  > L = Shanghai
  > O = Urlynn
  > CN = Steam Proxy
  > 
  > [ v3_req ]
  > basicConstraints = CA:FALSE
  > keyUsage = digitalSignature, keyAgreement
  > extendedKeyUsage = serverAuth
  > subjectAltName = @alt_names
  > 
  > [ alt_names ]
  > DNS.1 = store.steampowered.com
  > DNS.2 = steamcommunity.com
  > DNS.3 = www.steamcommunity.com
  > ```

  > `/etc/nginx/ca/pixiv/pixiv_ecc.cnf`
  >
  > ```shell
  > [ req ]
  > prompt = no
  > distinguished_name = req_distinguished_name
  > req_extensions = v3_req
  > 
  > [ req_distinguished_name ]
  > C = CN
  > ST = Shanghai
  > L = Shanghai
  > O = Urlynn
  > CN = Pixiv Proxy
  > 
  > [ v3_req ]
  > basicConstraints = CA:FALSE
  > keyUsage = digitalSignature, keyAgreement
  > extendedKeyUsage = serverAuth
  > subjectAltName = @alt_names
  > 
  > [ alt_names ]
  > DNS.1 = pixiv.net
  > DNS.2 = *.pixiv.net
  > DNS.3 = oauth.secure.pixiv.net
  > DNS.4 = fanbox.cc
  > DNS.5 = *.fanbox.cc
  > DNS.6 = *.pximg.net
  > ```

  ```shell
  openssl ecparam -genkey -name prime256v1 -out /etc/nginx/ca/github/github_ecc.key
  ```

  ```shell
  openssl req -new -key /etc/nginx/ca/github/github_ecc.key -out /etc/nginx/ca/github/github_ecc.csr -config /etc/nginx/ca/github/github_ecc.cnf
  ```

  ```shell
  openssl x509 -req -days 730 \
      -in /etc/nginx/ca/github/github_ecc.csr \
      -CA /etc/nginx/ca/root/root_ca_ecc.crt \
      -CAkey /etc/nginx/ca/root/root_ca_ecc.key \
      -CAcreateserial \
      -out /etc/nginx/ca/github/github_ecc.crt \
      -extfile /etc/nginx/ca/github/github_ecc.cnf \
      -extensions v3_req
  ```

- Nginx 反代

  > `/etc/nginx/conf.d/github.conf`
  >
  > ```shell
  > # ==================== 上游服务器定义 ====================
  > 
  > # Group 1: github.com 上游
  > upstream github_com {
  >     server 20.27.177.113:443 ;
  > }
  > 
  > upstream codeload_github_com {
  >     server 20.27.177.114:443 ;
  > }
  > 
  > # Group : api.github.com 上游
  > upstream api_github_com {
  >     server 20.27.177.116:443 ;
  > }
  > 
  > # Group : githubusercontent 相关域名上游 (IPv6)
  > upstream githubusercontent {
  >     server [2606:50c0:8001::154]:443 ;
  >     server [2606:50c0:8003::154]:443 ;
  >     server [2606:50c0:8002::154]:443 ;
  >     server [2606:50c0:8000::154]:443 ;
  >     least_conn;
  > }
  > 
  > upstream col_github_com {
  >     server 140.82.112.21:443 ;
  >     server 140.82.112.22:443 ;
  >     server 140.82.113.21:443 ;
  >     server 140.82.113.22:443 ;
  >     server 140.82.114.22:443 ;
  >     server 140.82.114.21:443 ;
  >     least_conn;
  > }
  > 
  > upstream redirect_github_com {
  >     server 140.82.112.17:443 ;
  >     server 140.82.113.17:443 ;
  >     server 140.82.114.17:443 ;
  >     server 140.82.112.18:443 ;
  >     server 140.82.113.18:443 ;
  >     server 140.82.114.18:443 ;
  > }
  > 
  > upstream assets_cdn {
  >     server [2606:50c0:8001::153]:443 ;
  >     server [2606:50c0:8003::153]:443 ;
  >     server [2606:50c0:8002::153]:443 ;
  >     server [2606:50c0:8000::153]:443 ;
  >     least_conn;
  > }
  > 
  > upstream cloudflare {
  >     server 1.0.0.1:443;
  > }
  > 
  > map $host $default_http_host {
  >     hostnames; 
  >     default cloudflare; # almost always 403
  >     github.com  github_com;
  >     gist.github.com  github_com;
  >     api.github.com  api_github_com;
  >     codeload.github.com  col_github_com;
  >     education.github.com  col_github_com;
  >     enterprise.github.com  col_github_com;
  >     classroom.github.com  col_github_com;
  >     central.github.com  col_github_com;
  >     collector.github.com  col_github_com;
  >     redirect.github.com  redirect_github_com;
  >     copilot.github.com  redirect_github_com;
  >     services.github.com  redirect_github_com;
  >     community.github.com  redirect_github_com;
  >     assets-cdn.github.com  assets_cdn;
  >     raw.github.com  githubusercontent;
  >     lab.github.com  githubusercontent;
  >     pages.github.com  githubusercontent;
  >     resources.github.com  githubusercontent;
  >     developer.github.com  githubusercontent;
  >     partner.github.com  githubusercontent;
  >     desktop.github.com  githubusercontent;
  >     guides.github.com  githubusercontent;
  >     support.github.com  githubusercontent;
  >     *.github.io  githubusercontent;
  >     *.github.com  githubusercontent;
  >     analytics.githubassets.com  githubusercontent;
  >     github.githubassets.com  githubusercontent;
  >     *.githubusercontent.com  githubusercontent;
  >     }
  > # ==================== Server 块配置 ====================
  > 
  > server {
  >     listen 443 ssl;
  >     http2 on;
  > 
  >     server_name 
  >         github.com
  >         *.github.com
  > 	    *.github.io
  >         analytics.githubassets.com
  >         github.githubassets.com
  >         assets-cdn.github.com
  >         *.githubusercontent.com;  
  >     
  >     ssl_certificate /etc/nginx/ca/github/github_ecc.crt;
  >     ssl_certificate_key /etc/nginx/ca/github/github_ecc.key;
  >     ssl_protocols TLSv1.3;
  >     client_header_buffer_size 16k;
  >     large_client_header_buffers 4 16k;
  >     
  >     location / {
  >         proxy_pass https://$default_http_host;
  >         proxy_set_header Host $http_host;
  >         proxy_buffer_size 16k;
  >         proxy_buffers 16 256k;     
  >         proxy_busy_buffers_size 512k;
  >         proxy_ssl_session_reuse on;
  >     }
  > }
  > 
  > server {
  >     listen 443 ssl;
  >     http2 on;
  > 
  >     server_name 
  >         docs.github.com
  >     
  >     ssl_certificate /etc/nginx/ca/github/github_ecc.crt;
  >     ssl_certificate_key /etc/nginx/ca/github/github_ecc.key;
  >     ssl_protocols TLSv1.3;
  >     client_header_buffer_size 16k;
  >     large_client_header_buffers 4 16k;
  >     
  >     location / {
  >         proxy_pass https://$default_http_host;
  >         proxy_set_header Host $http_host;
  >         proxy_buffer_size 16k;
  >         proxy_buffers 16 256k;     
  >         proxy_busy_buffers_size 512k;
  >         proxy_ssl_session_reuse on;
  >         proxy_hide_header Content-Security-Policy;
  >         proxy_hide_header X-Content-Security-Policy;
  >         proxy_hide_header X-WebKit-CSP;
  >     }
  > }
  > 
  > # HTTP 重定向到 HTTPS
  > server {
  >     listen 80;
  >     server_name 
  >         github.com
  >         *.github.com
  >         *.github.io
  >         assets-cdn.github.com
  >         analytics.githubassets.com
  >         github.githubassets.com
  >         *.githubusercontent.com;
  >     
  >     return 301 https://$host$request_uri;
  > }
  > ```

  > `/etc/nginx/conf.d/steam.conf`
  >
  > ```shell
  > upstream steam_backend {
  >     server 23.56.181.190:443 weight=2 max_fails=3 fail_timeout=1s;
  >     server 23.56.181.177:443 weight=2 max_fails=3 fail_timeout=1s;
  >     server 23.56.180.67:443 weight=2 max_fails=3 fail_timeout=1s;
  >     server 23.56.180.179:443 weight=2 max_fails=3 fail_timeout=1s;
  >     server 23.56.182.96:443 weight=2 max_fails=3 fail_timeout=1s;
  >     least_conn;
  > }
  > 
  > server 
  > {
  >     listen 443 ssl;
  >     http2 on;
  >     server_name store.steampowered.com steamcommunity.com;
  >     ssl_certificate    /etc/nginx/ca/steam/steam_ecc.crt;
  >     ssl_certificate_key    /etc/nginx/ca/steam/steam_ecc.key;
  >     ssl_protocols TLSv1.3;
  >     client_header_buffer_size 16k;
  >     large_client_header_buffers 4 16k;
  >         
  >     location / {
  >         proxy_pass https://steam_backend/;   
  >         proxy_set_header Host $http_host;
  >     }
  > }
  > ```

  > `/etc/nginx/conf.d/pixiv.conf `
  >
  > ```shell
  > upstream pixiv_net {
  >     server 210.140.139.151:443 ;
  >     server 210.140.139.158:443 ;
  >     server 210.140.139.161:443 ;
  >     server 210.140.139.159:443 ;
  >     server 210.140.139.162:443 ;
  >     least_conn;
  > }
  > 
  > upstream pximg_net {
  >     server 210.140.139.135:443 ;
  >     server 210.140.139.136:443 ;
  >     server 210.140.139.133:443 ;
  >     server 210.140.139.138:443 ;
  >     server 210.140.139.137:443 ;
  >     server 210.140.139.130:443 ;
  >     server 210.140.139.131:443 ;
  >     server 210.140.139.132:443 ;
  >     least_conn;
  > }
  > 
  > server {
  >     listen 443 ssl;
  >     http2 on;
  >     server_name
  >         pixiv.net
  >         *.pixiv.net
  > 				oauth.secure.pixiv.net
  >         *.fanbox.cc;
  >     #client_max_body_size 50M;
  >     ssl_certificate    /etc/nginx/ca/pixiv/pixiv_ecc.crt;
  >     ssl_certificate_key    /etc/nginx/ca/pixiv/pixiv_ecc.key;
  >     ssl_protocols TLSv1.3;
  >     client_header_buffer_size 16k;
  >     large_client_header_buffers 4 16k;
  >         
  >     location / {
  >         proxy_pass https://pixiv_net/;   
  >         proxy_set_header Host $http_host;
  > 				proxy_buffering off;
  >     }
  >     # Proxying WebSockets
  >     location /ws/ {
  >         proxy_pass https://pixiv_net;
  >         proxy_set_header Upgrade $http_upgrade;
  >         proxy_set_header Connection "upgrade";
  >     }
  > }
  > 
  > server {
  >     listen 443 ssl;
  >     http2 on;
  >     server_name *.pximg.net;
  > 
  >     ssl_certificate    /etc/nginx/ca/pixiv/pixiv_ecc.crt;
  >     ssl_certificate_key    /etc/nginx/ca/pixiv/pixiv_ecc.key;
  >     ssl_protocols TLSv1.3;
  >     client_header_buffer_size 16k;
  >     large_client_header_buffers 4 16k;
  >         
  >     location / {
  >         proxy_pass https://pximg_net/;   
  >         proxy_set_header Host $http_host;
  >         proxy_next_upstream_timeout 60;
  >         proxy_set_header Referer "https://www.pixiv.net/";
  >         proxy_set_header Sec-Fetch-Site "cross-site";
  >         allow all;
  >     }
  > }
  > ```

