---
title: "deploy.sh"
date: 2019-12-01T12:03:17+08:00
lastmod: 2019-12-01T12:03:17+08:00
draft: false
toc: false
keywords: ["daily"]
description: "记录一下生成文章和提交用的`deploy.sh`脚本"
author: "youting"
tags: ["bash"]
---

```sh
#!/bin/bash
# 部署到 github pages 脚本
# 错误时终止脚本
set -e

# 删除打包文件夹
rm -rf public

# 打包
hugo

# 进入打包文件夹
cd public

# add
git init

# 我会重新配置一下username和email，。以便生成提交的绿块ahhh
git config user.name "username"

git config user.email "email"

# 生成一个.gitignore，方便添加不要的文件
echo ".DS_Store
*.swap" > .gitignore

# 添加远程仓库，换成自己的 username
git remote add origin git@github.com:username/username.github.io.git

git add -A

# 读取输入的commit信息
read -p "Enter your commit msg: " commitMsg

# 加一个可以测试的确认过程
read -p "Are you sure to commit? [y/n]": sure

# 是y就commit并且继续，不是就退出
case $sure in
    [yY]*)
        git commit -m "$commitMsg"
        echo "commit with $commitMsg successfully."
        ;;
    [nN]*)
        exit
        ;;
    *)
      echo "just enter y or n, please."
      exit
      ;;
esac

# 推送到github
git push -f origin master

# 回到原文件夹
cd ..

```
