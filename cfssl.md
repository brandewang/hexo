---
title: cfssl
date: 2088-08-08 08:08:08
tags:
---
# cfssl生成自签证书
``` bash
#please download the cfssl bin file
wget -T 15 -t 3 https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O cfssl
wget -T 15 -t 3 https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O cfssljson
wget -T 15 -t 3 https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O cfssl-certinfo
#generate default config.json csr.json
cfssl print-defaults config > config.json
cfssl print-defaults csr > csr.json
cp config.json ca-config.json
cp csr.json ca-csr.json

#modify csr
cat ca-csr.json
{
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "ShangHai",
            "ST": "ShangHai",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
cfssl gencert -initca ca-csr.json |cfssljson -bare ca

#Generate harbor cert
cp csr.json harbor-csr.json
#modify csr
cat > harbor-csr.json <<EOF
{
  "CN": "harbor",
  "hosts": [
    "127.0.0.1",
    "$NODE_IP"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
#generate harbor cert
cfssl gencert -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json -profile=client harbor-csr.json| cfssljson -bare harbor
```
