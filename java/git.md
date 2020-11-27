## 1. 初始化

```sh
git config --global user.name  gengwei               # 配置个人用户名
git config --global user.email gengwei1024@163.com   #电子邮件地址
```

## 2. 撤回add

2.23 版以前

git reset HEAD .

2.23

git restore --staged .

```sh
丢弃本地当前所有文件修改
git clean -nxdf（查看要删除的文件及目录，确认无误后再使用下面的命令进行删除）
git checkout . && git clean -xdf
https://blog.csdn.net/leedaning/article/details/51304690
```



## 3. 撤回commit  未push

* 保留当前修改

git reset   before_commit_id

* 丢弃当前修改

git rest --hard before_commit_id

## 4. 撤回已push

git revert  pushed_commit_id

git commit -am "取消的原因"

git push

## 5. 暂存本地修改

git stash -u

> 应用暂存  git stash apply 0 

## 6. 删除remote代码，保留本地

git rm -r --cached  target

## 7. 复制一个文件到多个文件夹

https://exp-picture.cdn.bcebos.com/836a6aee1c324b184a393d4453a72633498448ae.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1

文件和多个文件夹必须在同一路径下

```sh
@echo off 
for /f %%i in ('dir /ad /b') do copy .gitignore %%i 
exit
```

.gitignore  是要复制的文件全称