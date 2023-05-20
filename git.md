## 1.git 强制推送

如果本地和远程发生冲突，使用以下命令强制推送本地到远程

```git
git push --force
// 这是简写
git push -f
```

**次命令会直接覆盖到远程的代码，因此一定要小心使用**

 ## 2.强制远程更新到本地

```git
git reset --hard origin/master
```

**次命令会直接覆盖本地写的代码，因此使用也要小心**

```git
// 获取某个远程的信息到本地
git fetch origin 分支名称 
// 获取所有远程的分支到本地
git fetch --all
```

fetch命令是在本地建立了一个远程的分支，并不会自动合并到当前的本地分支，需要手动使用 `git merge` 命令或 `git rebase` 命令合并到当前分支。

## 工作目录

- 工作目录（working Directory：平常存放代码的目录
- 暂存区（stage/index）
- 资源库（repository）
- 远程仓库 （Remote Directory）

![image-20210208143915457](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210208143915457.png)

## git 命令

```git
git init                                                          初始化命令
git add file                                                      添加到暂存区
git commit -m "说明"                                               添加到本地仓库
git remote add origin https://gitee.com/jiruixin/app.git          这是绑定到远程仓库

```

## git分支管理

```git
git branch          						  git分支管理
git branch -r      			 	               列出远程分支
git branch    branch_name                       新建分支不切换到改分支
git checkout    -b    branch_name               新建分支并且切换到改分支上
git merge   branch_name                         指定分支合并到当前分支
git push --set-upstream origin branch_name	    推送分支到远程
git push origin --delete branch_name		    删除远程分支


如果想新建远程分支，先创建本地分支并且切换到该分支，然后推送到码云上
```

![image-20220714103928893](https://gitee.com/jiruixin/images/raw/master/images/image-20220714103928893.png)





