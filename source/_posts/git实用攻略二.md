title: git实用攻略（二）
date: 2015-05-04 17:30:55
tags: 
- git
- 工具

---

最近团队的版本控制从svn切换到了git，虽说已经使用git有2年多了，也写了一个[实用攻略](http://minotaursu.com/2014/07/01/git%E5%AE%9E%E7%94%A8%E6%94%BB%E7%95%A5/)，但是github上的项目使用经验和公司内部团队协作的使用经验还有很多不同。补充下新的使用体会。
![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/gitflow.png?imageView/1/w/670/h/280)
首先还是看一下git的3个区：working，stage，commit，心中有个概念。

1. **github和gitlab账户的共存**   
配置sshkey登录的时候，git只能识别默认的id_rsa的秘钥文件，只有一个账户能够免登，在bash启动脚本中增加ssh-add file 实现多个账户的免登。

2. **迁出新的远程分支代码**   
本地已经有了master分支和develop分支代码，假设有一个project1的远程项目分支需要开发，使用git checkout -b project1 origin/project1迁出新的远程分支代码。

3. **使用git pull --rebase拉取代码**   
使用pull --rebase拉取最新代码可以保持一颗干净的commit树。

4. **使用merge，不要使用rebase合并不同的分支**  
使用rebase合并不同的分支会导致很多的冲突。 

5. **git reflog**  
可以看到本地仓库的操作记录，在reset操作后进行恢复特别有用。

6. **git commit --amend**   
补救提交，不产生commit log。

7. **git reset --hard HEAD**  
使用HEAD的内容覆盖工作区，放弃暂缓区的内容。

8. **git reset HEAD**  
退回暂缓区的内容，和git reset commitId一样，都是指针操作。

9. **alias命令**  
编辑 ~/.gitconfig，赋予复杂的命令一个简单的别名

