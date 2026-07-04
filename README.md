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
  > DNS.7 = github.blog
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

  > `/etc/nginx/conf.d/github.conf` — 见 [github.conf](./github.conf)

- DNS 劫持

  > `/etc/dnsmasq.d/github.conf` — 见 [github_dnsmasq.conf](./github_dnsmasq.conf)
  >

  > `/etc/nginx/conf.d/steam.conf`
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

