## 拉取远程代码，和本地代码对比合并

- vcs-->git--> merge change

## 本地仓库有更新数据时

- vcs--> git-->fetch  或者直接使用update project

## 更新完数据后，提交本地仓库，在push到远程仓库

- 拉取项目时，先建立本地分支，在把代码拉取到本地分支上。
- 远程分支合并到master上时，先在本地仓库合并master，在push到远程仓库
> 先checkout master，然后选择要合并的分支，merge into current

## 出现Push to origin/master was rejected 一般是远程分支的代码与本地代码，部分文件冲突
- vcs-->git--> merge change