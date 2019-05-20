title: git实用攻略
date: 2014-07-01 16:03:36
tags:
- git
- 工具 

---
一直以来都是在mac的iterm2和windows的cygwin上通过命令行的方式使用git，今天决定整理下常用的git命令，作为简明实用的帮助手册，方便以后查找。
## git 配置
1. 使用git config --global user.name/user.email 进行初始化配置
2. 使用cat ~/.gitconfig 和 cat .git/config 查看存储的配置信息
3. 配置[sshkey](https://help.github.com/articles/generating-ssh-keys) ，这样以后push就不再需要密码
4. 对某个命令有疑惑，使用git help

## git 基本命令 
1. **git init**  
本地生成git项目，会有一个.git的子目录产生
2. **git clone**  
签出分支
3. **git add**  
将本地修改/新增加入到git的index中，支持**git add file1 file2**，**git add dir**，**git add .**等多种形式
4. **git rm**  
删除本地文件&index中存储的修改, **git rm file1 file2** 删除文件，**git rm -r dir** 递归删除，**git rm --cached file1** 只从index中删除，适用于**git add**了一个不想提交的文件
5. **git commit -m "message"**  
提交index到head
6. **git status**  
查看状态
7. **git log**  
查看log
8. **git diff**  
比较本地文件和index
9. **git checkout**  
切换分支，**git checkout -b new_branch** 创建并切换到新分支
10. **git branch**  
列出本地所有分支，**git branch -r** 列出所有远程分支，**git branch -D new_branch** 删除某个分支
11. **git push origin master**  
将head更新到名为origin的远程版本库的master分支
12. **git fetch**  
从远程获取最新版本到本地
13. **git merge**  
远程/本地分支合并
14. **git pull**  
获取并合并分支 **git fetch**+**git merged**
15. **git rebase**  
回退变更到上次pull，更新最新pull的变更，将本地修改合并到变更上，额，解释起来比较复杂，建议看官方文档
16. **git reset**  
回退到上次commit时的状态
17. **git revert**  
撤销某次commit

## finally
复杂的操作和原理结构问题@**[git help](https://help.github.com/)**。

