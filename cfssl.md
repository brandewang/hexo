---
title: cfssl
date: 2088-08-08 08:08:08
tags:
---
# cfssl生成自签证书
``` bash
#please download the cfssl bin file
wget -T 15 -t 3 https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O /usr/local/bin/cfssl
wget -T 15 -t 3 https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O /usr/local/bin/cfssljson
wget -T 15 -t 3 https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O /usr/local/bin/cfssl-certinfo
#generate default config.json csr.json
cfssl print-defaults config > ca-config.json
cfssl print-defaults csr > ca-csr.json
#modify csr config
#CN(Common Name)主机域名, hosts(SANS Subject Alternative Names)备用主机名, 都用于访问时验证主机名称
cat > ca-csr.json << EOF
{
    "CN": "ca",
    "hosts": [
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "ShangHai",
            "L": "ShangHai",
            "O": "k8s",
            "OU": "ca"
        }
    ]
}
EOF

#其中www为服务器使用证书,client为客户端使用证书
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "8670h",
      "usages": [
        "signing",
        "key encipherment",
        "server auth",
        "client auth"
      ]
    },
    "profiles": {
      "www": {
        "expiry": "8760h",
        "usages": [
          "signing",
          "key encipherment",
          "server auth"
        ]
      },
      "client": {
        "expiry": "8760h",
        "usages": [
          "signing",
          "key encipherment",
          "client auth"
        ]
      }
    }
  }
}
EOF

#generate ca cert&key
cfssl gencert -initca ca-csr.json |cfssljson -bare ca

#Generate harbor cert
cp csr.json harbor-csr.json
#modify csr
cat > harbor-csr.json <<EOF
{
  "CN": "harbor",
  "hosts": [
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "ShangHai",
      "L": "ShangHai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
#generate harbor cert
cfssl gencert -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json -profile=www harbor-csr.json| cfssljson -bare harbor
```
