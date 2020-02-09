---
title: NextCloud完美搭载阿里云OSS-实现256TB超大存储容量
tags:
  - nextcloud
  - 阿里云oss
  - 超大存储
date: 2020-01-31 13:38:07
categories: 技术类
---
本地服务器容量不够用？
刚了解到阿里云oss该怎么用到自己的服务器上实现超大容量存储？
本文就来讲讲他的实现。
<!--more-->
## 环境准备
1.系统环境`lnmp`
2.软件准备`nextcloud`，`阿里云ossfs`
## 操作步骤
```
wget http://gosspublic.alicdn.com/ossfs/ossfs_1.80.5_centos7.0_x86_64.rpm
yum install -y ossfs_1.80.5_centos7.0_x86_64.rpm
echo aicloud-nextcloud:AccessKey ID:Access Key Secret > /etc/passwd-ossfs
chmod 640 /etc/passwd-ossfs
systemctl stop nginx
cd /usr/share/nginx/html/nextcloud/data
mv * .[^.]*  /opt
ls

# 此处的ouid和ogid通过cat /etc/passwd 查看，我查看到的nginx的ouid是996，ogid是994，-oumask就007就可以，内网不要选错就行
ossfs your-bucket-name /usr/share/nginx/html/nextcloud/data -ourl=oss-cn-shanghai-internal.aliyuncs.com -ouid=996 -ogid=994 -oumask=007 -o allow_other
df -lh
[info]
此处会显示ossfs的挂载信息

cd /opt
mv * .[^.]* /usr/share/nginx/html/nextcloud/data
systemctl start nginx
```
以上就是安装oss的操作步骤，安装结束访问你的网盘域名即可，如果网址正常访问，没有出现需要添加.octa文件的提示，说明挂在成功，如果出现了，需要检查一下ouid和ogid的值。
失败如图：
![img](https://image.lucfzy.com/image/201908-8-1/%E6%8C%82%E8%BD%BD%E5%A4%B1%E8%B4%A5.png)
如果想要知道是否挂载oss成功，最好的办法是手机下载nextcloud对应的app，显示如图表示成功：
![img](https://image.lucfzy.com/image/202001-oss/%E9%98%BF%E9%87%8C%E4%BA%91oss.jpg)

---
参考文章：
- [NextCloud 挂载 OSS 对象存储](http://help.websoft9.com/cloudbox-practice/nextcloud/solution/mount-oss.html)
- [挂载阿里云OSS到Nextcloud](https://royfk.com/archives/125)