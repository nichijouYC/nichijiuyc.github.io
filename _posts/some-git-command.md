---
title: 一些常用的Git命令
date: 2016-02-25 13:24:58
tags:
---

下面记录一些工作中常用的Git命令,会不定时更新


<!-- more -->


1. 安装Git `sudo apt-get install git` 

2. 连接GitHub `ssh -T git@github.com` 

    - 若不能，需要创建公钥（需要先安装ssh）

        1. `ssh-keygen -t rsa`

        2. 将`~/.ssh/rd_rsa.pub`内容复制到github网站里

        3. `ssh -T git@github.com`

3. 配置用户信息

    1. `git config --global user.name "nichijou"`

    2. `git config --global user.email "fyc_air@hotmail.com"`

4. 列出配置 `git config --list` 

5. 获取 Git 仓库

    1. 在现有目录初始化仓库 `git init`

    2. 从远程仓库clone `git clone https://github.com/libgit2/libgit2 mylibgit`

6. 检查文件状态 `git status`

7. 添加文件（暂存） `git add .`

8. 忽略文件 `编辑 .gitignore文件 加入规则`

9. 提交文件 `git commit -m "add file"`

10. 添加和提交文件，跳过git add步骤 `git commit -am "modify file"`

11. 移除文件

    1. 文件系统和Git都删除文件 `git rm README`

    2. 仅从Git删除 在文件系统保留 `gim rm --cache README`

12. 查看提交历史 `git log`

    1. 查看过去2次具体提交内容 `git log -p -2`

13. 取消暂存的文件 `git reset HEAD README`

14. 查看远程仓库 `git remote -v` 默认名字为origin

15. 添加远程仓库 `git remote add origin git@.....`

16. 从远程仓库拉取 `git fetch origin`

17. 从远程仓库拉取并且合并到当前分支  `git pull`

18. 推送到远程仓库 `git push origin master`

19. clone远程分支 `git checkout -t origin/dev`

20. 取消暂存的文件 `git checkout -- .`

21. 暂时保存工作区（不需要commit 回到上一个commit版本） `git stash` `git stash list` `git stash pop`

22. 回到（撤销）之前commit的版本 `git reset --hard 575563249bef66c87c5b82c1e184e65335c0583c`

23. 删除本地分支 `git branch -d dev`

24. 删除远程分支 `git push origin :dev`

25. 新建本地分支 `git branch -b newdev dev`

26. 提交新建的本地分支 `git push origin newdev:newdev`
