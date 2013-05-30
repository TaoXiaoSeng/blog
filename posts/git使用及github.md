---
date: 2013-05-30
layout: post
title: git使用及github
description: git使用及github的一些说明
permalink: '/2013/05/30/git_useage_and_github.html'
categories:
- git
tags:
- git

---

# git使用及github

git 和github越来越多人使用，作为一个程序员，应该懂的怎么去用，而我就是个不怎么会用的人，所以我的主管在我刚开始上班的时候就要我去熟悉git那套东西。

当时我依然不是很懂，但最近开始迷糊的懂了，下面是我这段时间的心得。


## 创建ssh从而能免登访问github。

命令```ssh-keygen```去创建公钥私钥，如果你只希望访问一个服务器的话，比如github，那么你就放到默认的目录下，并且使用默认的私钥和公钥文件路径，即~/.ssh/id_rsa和~/.ssh/id_rsa.pub，其中id_rsa.pub是公钥文件。我们需要将这个文件里的数据复制并粘贴到github上的ssh key里。从而通过ssh就可以面等访问github了。
   
   如果你有多个服务器需要访问，比如我们还需要访问gitlab，那么，这个时候，需要~/.ssh/config文件里的配置来区分服务器和ssh秘钥。
   
   比如我自己的配置为
    
```
Host github.com
	HostName github.com
	User taoxiaoseng@gmail.com
	IdentityFile /Users/Dick/.ssh/github_taoxiaoseng_rsa
	
Host gitlab.alibaba-inc.com
	HostName gitlab.alibaba-inc.com
	User guodi.ggd@taobao.com
	IdentityFile /Users/Dick/.ssh/gitlab_rsa
```
也就是说访问github用文件github_taoxiaoseng_rsa，访问gitlab用gitlab_rsa。注意公钥也需要上传到相应的服务器里。

其实还有些网上也经常说，这里不细提了。比如这些东西

```
git config --global user.name "you name"
git config --global user.email youremail@gmail.com
```

## 如何创建git仓库
其实每次你创建github里的仓库时，都会告诉你怎么去push，这里还是做下说明，讲我理解的说下。

```
mkdir helloworld			
cd helloworld
git init			创建一个空的数据仓库
touch readme.md
git add .			自动判断添加相应文件到git中
git commit -m "commit first file"			提交并且添加提交日志
git remote add origin git@github.com:TaoXiaoSeng/test.git 这个最关键，说的是在本地git中添加remote的git地址，并且以origin命名，这样以后我们就不需要每次都写git地址了。实际上他在.git目录里的config添加了一条记录，类似这样的。注意我这里特意加了两个，地址不一样。注意github区分大小写的仓库路径（我的用户名有大小写，如果不完全匹配则认为访问不到了。）。另外注意这里的branch，当你init的时候默认创建的就是master。
			[remote "origin"]
				url = git@github.com:taoxiaoseng/test.git
				fetch = +refs/heads/*:refs/remotes/origin/*
			[remote "remote"]
				url = git@github.com:TaoXiaoSeng/test.git
				fetch = +refs/heads/*:refs/remotes/remote/*git 
			[branch "master"]
				remote = remote
				merge = refs/heads/master
git push -u origin master			向远程origin对应的仓库里push本地master分支。这里用-u是类似使用upstream的意思?
```

## 其他git命令
git branch		罗列所有分支
git branch branchname	创建分支
git checkout branchname		切换分支
git branch -d branchname	假删除分支，如果是分支是当前正在运行的分支，则删除失败
git branch -D branchname	真删除分支

git ls-remote		查看远程版本库中的分支情况
git push remote :branchname		远程删除分支，注意如果分支是默认分支，则删除失败

## 里程碑管理
里程碑及tag，和分支管理类似，保存在.git/refs/tags路径下，引用可能指向一个提交，但也可能是其他类型。

* 轻量级里程碑：用git tag <tagname> [<commit>]命令创建，引用直接指向一个提交对象<commit>
* 带说明的里程碑：用git tag -a <tagname> [<commit>] 命令创建，并且在创建时需要提供创建里程碑的说明。Git会创建一个tag对象保存里程碑说明、里程碑的指向、创建里程碑的用户等信息，里程碑引用指向该Tag对象。
* 带签名的里程碑：用git tag -s <tagname> [<commit>] 命令创建。是在带说明的里程碑的基础上引入了PGP签名，保证了所创建的里程碑的完整性和不可拒绝性。