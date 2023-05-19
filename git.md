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