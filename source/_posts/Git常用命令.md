---
title: Git常用命令
date: 2018-06-13 13:49:08
tags: [从零折腾Ubuntu,Git,常用]
description: 介绍Git常用命令，方便日常使用来此copy
---
### ﻿1. GIT原理
* 分布式版本控制
* 直接记录快照，而非差异比较
* 五个工作区域：工作区，暂存区、本地代码库、本地远程代码库、远程远程代码库
* 四种文件状态：未修改，已修改、已暂存、已提交，已推送（已推送=未修改）

### 2. GIT操作
#### 2.1 全局配置
| 命令    | 说明 |
|:------ | :------|
| `git config -l` | 查看全局配置  |
| `git config --global user.name [name]`   | 修改用户名 |
| `git config --global user.email [email]` | 修改邮箱地址 |
| `git config --global core.editor vim`    | 修改编译器为 vim |
| `git config --global alias.st status`   | status 缩写为 st |
| `git config --global alias.co checkout` | checkout 缩写为 co |
| `git config --global alias.br branch`   | branch 缩写为 br |
| `git config --global alias.ci commit`   | commit 缩写为 ci |

#### 2.2 创建代码库
| 命令    | 说明 |
|:------ | :------|
| `git init`        | 初始化本地代码库 |
| `git clone <url>` | 克隆远程代码库 |

#### 2.3 修改和提交
| 命令    | 说明 |
|:------ | :------|
|`git status`              | 查看所有文件状态 |
|`git status [FILE]`       | 查看指定文件状态 |
|`git add .`               | 提交当前目录所有文件到暂存区 |
|`git add [FILE]`          | 提交指定文件到暂存区 |
|`git commit -m [message]` | 提交暂存区到本地代码库，message代表说明信息 |
|`git commit --amend`      | 提交暂存区到本地代码库，覆盖上一次提交 |
|`git diff`                      | 查看变更内容，对比工作目录与暂存区 |
|`git diff --cached`             | 查看变更内容，对比暂存区与本地代码库 |
|`git diff master origin/master` | 查看变更内容，对比本地代码库与本地远程代码库，origin代表远程的意思 |
|`git mv [old] [new]`     | 文件改名 |
|`git rm [FILE]`          | 删除文件 |
|`git rm --cached [FILE]` | 停止跟踪文件但不删除 | 

#### 2.4 查看提交历史
| 命令    | 说明 |
|:------ | :------|
| `git log`                   | 查看代码库日志 |
| `git log -p`                | 查看代码库日志，同时显示具体修改 |
| `git log -2`                | 查看代码库最近两次提交的日志 |
| `git log [FILE]`            | 查看指定文件的代码库日志 |
| `git log --graph --pretty=oneline --abbrev-commit -20`          | 查看代码库日志，简洁有图 |
| `git reflog`                | 查看操作记录 |

#### 2.5 版本回撤
| 命令    | 说明 |
|:------ | :------|
| `git reset HEAD`            | 将文件从暂存区中回退到工作区，git add的反操作 |
| `git reset 版本号（前七位）`   | 回撤到指定版本 |
| `git reset --soft HEAD`     | 只改变提交点，工作目录与暂存区中内容都不改变 |
| `git reset --hard HEAD`     | 回撤到当前版本，撤销工作目录与暂存区中的所有内容 |
| `git reset --hard HEAD^`    | 回撤到上一个版本 |
| `git reset --hard HEAD^^`   | 回撤到上上一个版本 |
| `git reset --hard HEAD~2`   | 回撤到上上一个版本 |
| `git chechout [FILE]`       | 撤销某一个文件当前的修改 |
| `git revert [CommitID]`     | 撤销指定的commit，会新增一个commit，将版本改回修改前 |
| `git clean -xfd`            | 清除所有非GIT跟踪文件，例如编译生成的内容 |

#### 2.6 分支branch操作
| 命令    | 说明 |
|:------ | :------|
| `git branch`                    | 查看本地分支 |
| `git branch -r`                 | 查看远程分支 |
| `git branch -a`                 | 查看所有分支 |
| `git branch [branch-name]`      | 新建本地分支，注意不会自动切换，依然停留在当前分支 |
| `git branch -d [branch-name]`   | 删除分支，注意只能删除已经合并过的分支 |
| `git checkout [branch/tag]`     | 切换到指定分支，并更新工作区 |
| `git checkout -b [branch-name]` | 新建本地分支，并切换到该分支 |
| `git tag`                       | 查看所有本地标签 |
| `git tag [tagname]`             | 创建标签，基于最新版本提交 |
| `git tag -a [tagname] -m [message]` | 创建标签，基于最新版本提交，附带注释信息 |
| `git tag -a [tagname] -m [message] [CommitID]`  | 创建标签，基于指定的Commit，附带注释信息 |
| `git tag -d [tagname]`              | 删除标签 |
| `git show [tagname]`                | 查看指定标签信息 |
| `git push origin [tagname]:refs/tags/[tagname]` | 配合Gerrit使用 |

#### 2.7 合并与衍合
| 命令    | 说明 |
|:------ | :------|
| `git merge [branch]`     | 合并指定分支到当前分支 |
| `git rebase [branch]`    | 合并指定分支到当前分支，将当前分支的修改连接在指定分支之后 |
| `git fetch [branch]`     | 将远程远程库的代码更新到本地远程库 |

#### 2.8 远程操作
| 命令    | 说明 |
|:------ | :------|
| `git remote -v`                  | 查看远程版本库信息 |
| `git remote show [remote]`       | 查看指定远程版本库信息 |
| `git remote add [remote] [url]`  | 添加远程版本库 |
| `git fetch [remote]`             | 从远程远程库获取代码到本地远程库 |
| `git pull [remote] [branch]`     | 下载代码及快速合并 |
| `git push [remote] [branch]`     | 上传代码及快速合并，把所有文件从本地仓库推送进远程仓库 |
| `git push [remote]:[branch/tag-name]`   | 删除远程分支或标签 |
| `git push --tags`                | 上传所有标签 |

### 3. Gerrit日常开发
* master  常驻分支，要求常驻分支时刻可用
* feature-XXX 功能分支，新建分支用于开发新功能
* bugfix-XXX 问题分支，新建分支用于修改问题
1. 下载版本 BASE_IPC_HI3518EV200_SPC040  
`git clone & scp`
2. 新建分支  
`git checkout -b feature-XXX master`
3. 在新分支上完成功能修改和测试验证  
`git add`  
`git commit -m "XXX"`
4. 将功能合并到master  
* `git checkout master` 切换到master分支  
* `git pull --rebase` 更新代码  
* 合并操作有如下四种方式：
  1. `git merge --no-ff -m "XXXX" feature-XXX/bugfix-XXX`  
合并修改到master并commit，在master中可以查看到分支的log历史。如果产生冲突，通过 `git status` 查看冲突文件，手动解决冲突后 `git add`，`git commit`
  2. `git merge --squash -m "XXXX" feature-XXX/bugfix-XXX`  
合并修改到master分支并commit，分支的所有修改一次commit到master，无法在master中查看分支的log历史，需要稍后手动 `git commit`
  3. `git rebase feature-XXX/bugfix-XXX`  
合并修改到master分支，master的修改和commit将接在分支修改的末端，合并后log中无法查看到分支的存在
  4. `git cherry-pick [Commit-ID]....`  
合并指定的修改到master分支，分支修改会合并到master历史log的末端，如果发生冲突，手动解决冲突后 `git add`，`git cherry-pick --continue -m "XXXX"`  
* `git branch -d feature-XXX/bugfix-XXX`  
删除指定分支

5. 上传远程服务器  
`git push origin master:refs/for/master`  
上传审核，审核通过后合并到版本机

### 常用命令
`git log --graph --pretty=oneline --abbrev-commit -20`
`git push origin master:refs/for/master`
