---
title: Github-Pull-Request
date: 2019-01-22 21:59:58
tags:
---



1. fork 到自己的仓库
2. git clone 到本地
3. 上游建立连接
* git remote add upstream 开源项目地址
4. 创建开发分支 (非必须)
* git checkout -b dev
5. 修改提交代码
* git status git add . git commit -m git push origin branch
6. 同步代码三部曲
* git fetch upstream
* git rebase upstream/master
* git push origin master
7. 提交pr
去自己github仓库对应fork的项目下
new pull request



同步远程git代码到fork
git remote add upstream <原仓库github地址>
git remote -v
git fetch upstream