---
title: "Git 常用命令"
date: 2022-01-02T13:25:15+08:00
publishdate: 2022-01-02
lastmod: 2022-01-23
draft: false
tags: ["git"]
---

## 迁移 Git 仓库

```bash
// 切换到需要迁移的 Git 项目目录
$ cd project

// 添加 Git 远程仓库
$ git remote add gitlab git@gitlab.com:team/name.git

// 迁移代码到远程仓库
$ git push -u gitlab master
```

## 克隆仓库

- 仅克隆最近一次提交
  `git clone --depth=1 <repository>`

- 仅克隆指定分支的最近一次提交
  `git clone -b <branch> --depth=1 <repository>`

## 清除线下仓库中线上已不存在的分支

    `git remote prune origin`

## 本地创建并切换到对应的远程分支，创建对应的 `origin/dev` 本地分支，命名为 `dev`

    `git checkout -b dev origin/dev`

## 删除本地分支命令行

    `git branch -d <branch>`

## 本地分支拉取并合并线上主分支

    `git pull origin master`

## 将本地已有项目添加到 Git 仓库

1. 进入本地项目，初始化 Git 仓库 `git init`
2. 为本地项目指定远程仓库地址 `git remote add origin git@github.com:yourusername/yourproject.git`
3. 将远程仓库项目拉取到本地 `git pull` - 这一步可能需要设置远程仓库分支 `git branch --set-upstream-to=origin/<branch> master` - 这一步可能遇到拒绝合并的错误 `fatal: refusing to merge unrelated histories Error redoing merge`，可以强制合并 `git pull --allow-unrelated-histories`
4. 合并本地和远程仓库并提交和推送

## 合并 commit

`git commit -a –amend -m "提交注释信息"`

`git commit --amend` 会合并当前提交和上一次的提交，如果当前提交有注释，则以当前的注释为合并后的提交的注释。若当前提交没有注释，则以上一次提交的注释作为合并后的提交的注释

## 将分支 A 的修改转移到分支 B

### 修改尚未 commit 时

1. A 分支下执行 `git stash`
2. B 分支下执行 `git stash pop`
3. 查看修改结果 `git status -sb`

### 修改已经 commit 时

1. A 分支下执行 `git log`，查看需要转移的 commitID
2. B 分支下执行 `git cherry-pick commitID`，转移当前提交内容
3. 查看修改结果 `git status -sb`

## Fork 仓库同步远程仓库代码

1. 将远程仓库关联到本地 upstream `git remote add upstream https://github.com/test.git`
2. 确认是否配置成功 `git remote -v`
3. 拉取远程仓库代码 `git fetch upstream`
4. 本地仓库切换到主分支 `git checkout master`
5. 合并远程仓库代码到本地主分支 `git merge upstream/master`
6. `git push`

## 解决 refusing to merge unrelated histories 错误

github 新创建了仓库，将本地代码 push 上去前需要先拉取远端仓库代码。拉取时提示 `refusing to merge unrelated histories` 错误

这是由于这两个 git 仓库没有共同的提交记录，git 认为是两个项目。为了防止开发者上传错误，在 git 2.9.2 版本后，这种情况将给出这样的提示。

若仍然希望拉取代码，可以明确允许不相关的记录拉取 `git pull --allow-unrelated-histories`

## 删除提交记录

1. `git log` 记下要删除的记录的上一条记录的 commit 号，如 xxxxx
2. `git rebase -i xxxxx`，将打开编辑器
3. 将要删除的 commit 号的 `pick` 前缀改为 `drop` 然后保存退出
4. `git log` 查看是否删除成功
5. `git push origin HEAD --force` 强制推送到远端仓库

## 忽略 gitignore 中已被追踪的文件
`git rm --cached <file>`


## 参考

- [如何把已存在的 git 项目转移到 Gitlab 项目](https://segmentfault.com/q/1010000000385886)
- [Git 远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
- [How to make Git “forget” about a file that was tracked but is now in .gitignore?](https://stackoverflow.com/questions/1274057/how-to-make-git-forget-about-a-file-that-was-tracked-but-is-now-in-gitignore)
