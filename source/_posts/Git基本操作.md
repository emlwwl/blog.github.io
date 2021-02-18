---
title: Git常用命令
date: 2020-09-13 20:05:55
tags: Git
categories: 
- Git
---
### 一、全局设置配置用户名和邮箱

首先安装好git bash后，任意位置打开git bash命令窗口

输入命令：

```
git config --global user.name "yourname" //配置账号

git config --global user.email "youremail@xx.com" //配置邮箱
```
<!--more-->

这里顺带说一下config的作用域

```
git config --local  //local只对某个仓库有效
git config --global //global对所有仓库都有效
git config --system //system对系统所有登录的用户有效
```

显示config的配置，加 --list

```
git config --list --global
```

### 二、生成ssh公钥

```
ssh-keygen -t rsa
```
这里说明一下公钥的作用：公钥我们一般是给服务器的，在权限中加入我给的公钥，然后当我从远地仓库中下载项目的时候，服务器通过他的绑定的公钥来匹配我的私钥，这个时候,如果匹配,则就可以正常下载，如果不匹配，则失败。具体参照就是我们拉取GIthub上面的项目时，需要的操作。

### 三、常用命令

1、提交代码到暂存区

```
git add .   （后面一个点代表提交所以修改文件到暂存区）
```
2、提交内容的介绍

```
git commit -m '你的描述'
```
3、拉取远程代码

```
git pull origin master （这是下拉代码，将远程最新的代码与你本地最新的代码合并）
```
4、推代码至远程分支

```
git push origin marter (master为远程分支名)
```

5、撤销本地修改（没有add之前，注意空格）

```
git checkout .
```

6、撤销本地修改（add之后，注意空格）

```
git reset HEAD .
```

7、回退版本（commit之后，注意空格）

```
git reset –hard HEAD ^ (^ 表示回到上一个版本)
git reset –hard HEAD~100 (回退到前100个版本)
git reset –hard 版本号
```

8、查看版本号

```
git log --pretty=oneline
```

9、git 修改远程仓库

 ```
git remote set-url origin http://example.com/test.git
 ```

10、查看配置

```
git config --list
```

