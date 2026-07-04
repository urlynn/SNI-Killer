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

  > `/etc/nginx/conf.d/github.conf` — 见 [github.conf](./nginx/github.conf)

- DNS 劫持

  > `/etc/dnsmasq.d/github.conf` — 见 [github.conf](./dnsmasq/github.conf)
  >

  > `/etc/nginx/conf.d/steam.conf` — 见 [steam.conf](./nginx/steam.conf)
  > `/etc/dnsmasq.d/steam.conf` — 见 [steam_dnsmasq.conf](./dnsmasq/steam.conf)

  > `/etc/nginx/conf.d/pixiv.conf` — 见 [pixiv.conf](./nginx/pixiv.conf)
  > `/etc/dnsmasq.d/pixiv.conf` — 见 [pixiv_dnsmasq.conf](./dnsmasq/pixiv.conf)

