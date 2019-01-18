---
title: aws
date: 2088-08-08 08:08:08
tags:
---
#  S3
``` bash
#curl操作 s3 rest接口
1. 创建bucket
创建名为"mygod"的bucket。

bucket="mygod"
dateValue=`date -R`
resource="/${bucket}/"
contentType="application/octet-stream"
stringToSign="PUT\n\n\n${dateValue}\n${resource}"
s3Key="6R2MJWR863EREUDD0KTZ"
s3Secret="74eHNNQa1oLBlvZfO2CC2hIU8cobSYxTgeRDtXtH"
signature=`echo -en ${stringToSign} | openssl sha1 -hmac ${s3Secret} -binary | base64`
curl -v -X PUT "http://${bucket}.s3.mydomain.com" \
        -H "Host: ${bucket}.s3.mydomain.com" \
        -H "Date: ${dateValue}"\
        -H "Authorization: AWS ${s3Key}:${signature}"

2. 上传对象到bucket
下面的脚本就是将@file上传到@url指定的存储服务集群中的@bucket下，对象的名字叫@objname。

file=./c.zip
objname="testfile"
bucket=another
url="s3.mydomain.com"
resource="/${bucket}/${objname}"
contentType="application/x-compressed-tar"
dateValue=`date -R`
stringToSign="PUT\n\n${contentType}\n${dateValue}\n${resource}"
s3Key="6R2MJWR863EREUDD0KTZ"
s3Secret="74eHNNQa1oLBlvZfO2CC2hIU8cobSYxTgeRDtXtH"
signature=`echo -en ${stringToSign} | openssl sha1 -hmac ${s3Secret} -binary | base64`
curl -X PUT -T "${file}" \
  -H "Host: ${bucket}.${url}" \
  -H "Date: ${dateValue}" \
  -H "Content-Type: ${contentType}" \
  -H "Authorization: AWS ${s3Key}:${signature}" "http://${bucket}.${url}/${objname}"

注意：如果使用https，请将http替换为https。
3. 获取object
file="./myfile"
objname="testfile"
bucket=another
url="s3.mydomain.com"
resource="/${bucket}/${objname}"
contentType="application/x-compressed-tar"
dateValue=`date -R`
stringToSign="GET\n\n${contentType}\n${dateValue}\n${resource}"
s3Key="6R2MJWR863EREUDD0KTZ"
s3Secret="74eHNNQa1oLBlvZfO2CC2hIU8cobSYxTgeRDtXtH"
signature=`echo -en ${stringToSign} | openssl sha1 -hmac ${s3Secret} -binary | base64`
curl -o ${file} -X GET \
  -H "Host: ${bucket}.${url}" \
  -H "Date: ${dateValue}" \
  -H "Content-Type: ${contentType}" \
  -H "Authorization: AWS ${s3Key}:${signature}" "http://${bucket}.${url}/${objname}"


#s3cmd

vim ~/.s3.cfg
bucket_location = cn-north-1
host_base = s3.cn-north-1.amazonaws.com.cn
host_bucket = %(bucket)s.s3.cn-north-1.amazonaws.com.cn
website_endpoint = http://%(bucket)s.s3-website-%(location)s.amazonaws.com.cn/

```
