---
title: Git学习笔记
tags: 
	- git
date: 2020-01-31 01:25:22
categories:  应用软件
---
centos下如何使用git
git能为我们带来什么
如何将本地的文件克隆到github项目中做备份。
<!--more-->
## git本地文件到github上
### 测试操作
为了保险起见我们先上传一个readme.md文件作为测试。
步骤如下:
```
cd 文件目录 # 该文件目录作为本地的git仓库
echo "# my-hexo-next" >> README.md
git init #初始化git仓库
git add README.md #将readme.md文件存入git缓存中
git status ./ # 查看git缓存中的所有缓存文件
git commit -m "first commit" #将上一步在git缓存中的所有缓存文件上传,并添加描述“first commit”
git remote add origin https://github.com/Fzyproj/my-hexo-next.git #连接远程github库
git push -u origin master #将本地文件push到远程github服务器的master中
会提示输入用户名和密码即可完成push
```
添加一个文件夹下的所有文件到github仓库中：
```
git init
<生成ssh密钥>
git config --global user.name lucfzy
git config --global user.email your-email-address
ssh-keygen -t rsa -C "your-email-address"
# 密钥默认保存在/root/.ssh下
编辑id_rsa.pub文件
将其中的文件内容拷贝到github账户中setting中的ssh下。
</生成ssh密钥>
git add .
git status ./
git commit -m "first commit"
git remote add origin git@github.com:Fzyproj/my-hexo-next.git
# 第一次git需要加-u参数
git push -u origin master
```
## 更新本地文件并同步到github仓库
```
git add -A
git status
git commit -m "update file"
# github中的README.md文件不在本地代码目录中 需要采用这个命令，如果使用一次之后，后面直接push即可，不需要再pull了
git pull origin master
git push -u origin master
```

## 克隆github文件到本地
```
cd /file
# 连接远程github仓库
git remote add origin git@github.com:Fzyproj/my-hexo-next.git
git pull origin master
```

---
## Q&A
1.如何解决failed to push some refs to git报错问题？
A：这个问题是由于本地没有README.md文件而远程的github仓库中存在该md文件造成的，解决办法使用下面两条命令：
```
git pull --rebase origin master #将远程的README.md文件拷贝到本地
git push -u origin master #即可完成本地文件上传至github
```
2.如何取消和远程github库的关联？
A：使用命令`git remote remove origin`
3.如何移除所有git缓存文件？
A：使用命令：`git rm -r --cached`，实际体验这个命令无法解决的问题是，如果你的仓库中已经有同样的文件，即使remove掉，如果你再add其效果还是一样的，查看status，会显示nothing to update，导致你仓库中的文件没有办法更新。
正确操作：删除之前建立在本地的.git/目录，记得是目录，命令`rm -rf .git`，接着缓存就都会消失了，重新add文件即可。
4.上传文件后为什么明明是文件夹却显示是文件？
A: 由于该文件夹下面有.git/文件夹，导致和原来的git冲突，正确做法只需要将该文件夹下面的.git删除即可解决问题。

5.强制更新本地文件至远程（可能会导致版本丢失）？

A:`git push -u origin master -f `