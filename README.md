### 自签名证书

- 根证书
`/etc/nginx/ca/root/root_ca.cnf`

```shell
openssl ecparam -genkey -name secp384r1 -out root_ca_ecc.key #*生成 ECC 私钥*
openssl req -new -x509 -key root_ca_ecc.key -out root_ca_ecc.crt -days 3650 -config root_ca_ecc.cnf *# 生成自签名的 ECC 根证书*
```


> 根证书：etc/nginx/ca/root/root_ca_ecc.crt

### 重新签名


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