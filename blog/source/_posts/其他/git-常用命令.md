---
title: git 常用命令
date: 2018-05-14 10:05:10
tags:
categories: GIT
---
## [借鉴文章](https://jinlong.github.io/2015/10/12/syncing-a-fork/)
## 同步上游仓库代码
### remote
	注意：同步上游仓库代码之前必须使用remove命令指向上游仓库
	git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
### fetch
    git fetch upstream
### 切换分支
	git checkout xxx
### merge
	将upstream/master 分支合并到本地的 master 分支，本地的 master 分支便跟上游仓库保持同步了，并且没有丢失你本地的修改
	git merge upstream/master
### pull
	记得执行git pull，将本地的代码合同步到github

## 克隆代码
	git clone xxx
## 回退版本
	git reset --hard commit
## 切换分支
	git check xxx
## 查看分支
	git branch -a
## 查看使用过的命令
	git reflog
## 查看提交记录
	git log 也可单独制定文件，查看该文件的提交记录