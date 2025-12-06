---
title: git分布式管理工具使用
date: 2024-01-11 00:00:00+0000
updated:
tags: [git]
categories: [计算机]
keywords: []
description: git分布式管理工具使用
top_img:
comments:
cover:
toc:
---
## 创建一个git仓库

`git init`

`git add (.)` 提交到暂存区

`git commit -m <>`

`git log <--stat>`

`git diff <commit-id>`

`git reset --hard <commit-id>`

`git checkout <commit-id>`

`git branch`

`git checkout -b develop`

`$master git merge develop`

## git config

### 修改某些属性

`git config --global <properity> <value>`

eg:

```sh
git config --global user.name

git config --global user.email

git config --global http.sslverify false 关闭验证ssl证书

git config --global http.sslBackend schannel 
```

### 查看属性

`git config --list --show-origin`

## git clone

### 携带参数

eg:
`git -c http.sslVerify=false clone <URL>`

## 本地仓库和远程仓库绑定(远程分支)

`git remote remove origin`

`git remote add origin "https://github.com/LosLiSang/loslisang.github.io.git"`

## 参考

[迷途小书童的Note迷途小书童的Note (xugaoxiang.com)](https://xugaoxiang.com/2021/03/17/git-clone-ssl-certificate-problem-unable-to-get-local-issuer-certificate/)
