# How to use ssh

## SSH keyの作成

```shell
usage: ssh-keygen [-q] [-b bits] [-t dsa | ecdsa | ed25519 | rsa | rsa1]
                  [-N new_passphrase] [-C comment] [-f output_keyfile]
```

ssh-keygenを用いてkeyの作成を行う

`ssh-keygen -t ed25519 -N "passphrase" -C "email address/or something" -f "filename"`


rsaしか使用できない場合はrsa 4096bitがオススメだがrsa自体推奨できない。puttyやその他は2048bit以下で作成されるが現在において暗号強度が弱い。


```shell
# 公開鍵をなくした場合
ssh-keygen -yf <private_key>
```


# How to use openssl

## ECC 楕円曲線暗号

```shell
openssl ecparam -list_curves
openssl ecparam -name <暗号名> -genkey -out <秘密鍵のファイル名>
openssl ecparam -out server.key -name prime256v1 -genkey
# keyの確認
openssl ec -text -noout -in server.key
# CSR(証明書署名要求)
openssl req -new -key <秘密鍵のファイル名> -out <CSRのファイル名>
openssl req -new -key server.key -out server.csr -sha256
openssl req -text -noout -in <CSRのファイル名>

# Country Name (2 letter code) [AU]: 
# State or Province Name (full name) [Some-State]:
# Locality Name (eg, city) [ ]:
# Organization Name (eg, company) [Internet Widgits Pty Ltd]:
# Organizational Unit Name (eg, section) [ ]:
# Common Name (e.g. server FQDN) [ ]:

openssl x509 -text -noout -in <証明書のファイル名>
```
