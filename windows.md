---
title: windows
date: 2088-08-08 08:08:08
tags:
---

## 时间服务

``` bash
#初始化组件
w32tm /unregister
w32tm /register

#验证pdc
w32tm /monitor /domain:gihg.com 

#查看同步源
w32tm /query /source
#查看同步状态
w32tm /query /status
#查看时间同步配置
w32tm /query /configuration
#查看时区
w32tm /tz
#手动同步
w32tm /resync
#服务启停
net start/stop w32time
#重新发现时间源
w32tm /resync /rediscover

#查看ntp服务器连接状态
w32tm /stripchart /computer:ntp1.aliyun.com
#手动配置ntpclient
w32tm /config /manualpeerlist:ntp1.aliyun.com /syncfromflags:manual /reliable:yes /update
w32tm /config /manualpeerlist:"ntp1.aliyun.com,0x1 ntp2.aliyun.com,0x1 ntp3.aliyun.com,0x1 ntp4.aliyun.com,0x1" /syncfromflags:manual /reliable:yes  /update
#手动配置nt5ds
w32tm /config /syncfromflags:domhier /reliable:yes /update

```

## AD服务

``` bash
#检查域控错误信息
dcdiag /s:host.domain.com
#ad同步检查
repadmin /showrepl
repadmin /syncall

-------------------------
#导入导出active direcotry对象
#ldifde
## gihg.com导出ou
ldifde -f exportOu.ldf -s gihg-dc03 -d "OU=GIHG,DC=gihg,DC=com" -p subtree -r "(objectClass=organizationalUnit)" -l "cn,objectclass,ou"
## gihtg.com导入ou
记事本打开文件 exportOu.ldf,区分大小写替换gihg为gihtg,GIHG为GIHTG
ldifde -i -f exportOu.ldf -s gihtg-dc01

## gihg.com导出user
ldifde -f exportUser.ldf -s gihg-dc03 -d "OU=GIHG,DC=gihg,DC=com" -p subtree -r "(&(objectCategory=person)(objectClass=User))" -l "cn,givenName,objectclass,samAccountName,description,displayName,telephoneNumber,wWWHomePage,mail,accountExpires,userPrincipalName,sn"
## gihg.com导入user
记事本打开文件 exportUser.ldf,区分大小写替换gihg为gihtg,GIHG为GIHTG
ldifde -i -f exportUser.ldf -s gihtg-dc01

## gihg.com导出group
ldifde -f exportGroup.ldf -s gihg-dc03 -d "OU=GIHG,DC=gihg,DC=com" -p subtree -r "(objectclass=group)" -l "dn,member,info,description,mail,grouptype,instancetype,objectclass,name,samaccountname"
记事本打开文件 exportUser.ldf,区分大小写替换gihg为gihtg,GIHG为GIHTG
#跳过G-ALL-User
ldifde -i -f exportGroup.ldf -s gihtg-dc01 -k


#批量启用新域账号
PS:将GIHTG-ADMIN或其他管理员账号先移出作用ou
dsquery user -disabled "OU=GIHTG,DC=gihtg,DC=com" -limit 200|dsmod user -disabled no -pwdneverexpires yes -mustchpwd no -pwd gihtg@2020
#将旧域禁用账号导出到dis_user.txt
dsquery user -disabled "OU=GIHG,DC=gihg,DC=com" > dis_user.txt
#批量禁用新域账号
记事本打开文件dis_user.txt,区分大小写替换gihg为gihtg,GIHG为GIHTG
type dis_user.txt|dsmod user -disabled yes


--------------------------
#批量新建用户

#模版csv
CN,Description,DisplayName,GivenName,Surname,ou,SamAccountName,UserPrincipalName,telephoneNumber,mail,wWWHomePage,AccountPassword
BrightZhou,周玥,BrightZhou,Zhou,Bright,"OU=OU_InformationTechnology,OU=OU_Department,OU=Users,OU=GIHTG,DC=gihtg,DC=com",bright.zhou,bright.zhou@gihtg.com,139 1629 4455,bright.zhou@thequbehotels.com,www.gihtg.com,gihtg@2020


#导入命令
Import-Csv .\it_new_user.csv |ForEach-Object {New-ADUser -Name $_.CN -Description $_.Description  -EmailAddress $_.mail -OfficePhone $_.telephoneNumber -HomePage $_.wWWHomePage -DisplayName $_.DisplayName -GivenName $_.GivenName -Surname $_.Surname -Path $_.ou -SamAccountName $_.SamAccountName -UserPrincipalName $_.UserPrincipalName -AccountPassword(ConvertTo-SecureString -AsPlainText $_.AccountPassword -Force) -Enabled 1 -ChangePasswordAtLogon 0 -PasswordNeverExpires 1}
```

## 组策略

``` bash 
#查看已加载组策略
gpresult /r

#查看本地策略
gpedit.msc

#查看域策略
rsop.msc
```

## wsus

``` bash
#模式
1.自治模式
2.副本模式

#win7
#客户端连接服务器
wuauclt /detectnow
wuauclt /reportnow

#win10 win2016 win2019
#客户端连接服务器
c:\windows\system32\UsoClient.exe startscan
```
