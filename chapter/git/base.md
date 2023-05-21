# git基础

> 菜鸟教程：https://www.runoob.com/git/git-basic-operations.html

* 初始化：

  * git init
  * 将当前文件夹交由git管理，后续才能使用git命令；
* 克隆仓库：

  * git clone xxx；
* 拉取分支并合并至本地：

  * git pull origin master
* 本地提交到暂存区：

  * git add .
* 暂存区提交至本地仓库：

  * git commit -m 'feat: test'
* 本地提交到远程： 

  * 一般操作：git push origin <本地分支名>:<远程分支名>
  * 如果本地分支名与远程分支名相同，则可以省略冒号：
    * git push origin master
  * 如果第一次提交（vscode会有提示）：
    * git push origin feature/test: feature/test --set-stream
    * 后面，**在当前分支**直接使用git push
* 新建分支：

  * git checkout feature/test;

  * 新建并切换分支：git checkout -b feature/test：
* 查看日志：
  * git log
* 查看分支状态：
  * git status
  * 显示哪些文件存在改动
* 回滚：
  * git reset
  * 根据commitId进行回滚，通常搭配git log使用；



# 钩子

通常，在公司项目中提交代码之前会进行lint，防止代码存在格式问题。

```
// package.json
 "gitHooks" : {
 	"pre-commit": "lint-staged",
 }
 "lint-staged" : {
 	"*.(js,vue,ts)":[
 		"vue-cli-service lint"
 	]
 }
```

