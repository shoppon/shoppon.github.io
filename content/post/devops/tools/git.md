# Git

## reset&checkout&revert

| **命令**      | **作用域** | **常用情景**                       |
| ------------- | ---------- | ---------------------------------- |
| git  reset    | 提交层面   | 在私有分支上舍弃一些没有提交的更改 |
| git  reset    | 文件层面   | 将文件从缓存区中移除               |
| git  checkout | 提交层面   | 切换分支或查看旧版本               |
| git  checkout | 文件层面   | 舍弃工作目录中的更改               |
| git  revert   | 提交层面   | 在公共分支上回滚更改               |
| git  revert   | 文件层面   | （然而并没有）                     |

## 命令示例

### 代码

**更新submodule：**`git submodule update --init --recursive`

### 工作区

**更新代码永远使用：**`git pull --rebase`

**删除untracked file：**`git clean -fdx`

**删除未提交的变更：** `git reset --hard HEAD`

### 分支

**删除远端分支：**`git push origin --delete BRANCH`

**删除本地分支：**`git branch -D BRANCH`

**创建本地分支：**`git branch -b BRANCH`

**创建远程分支：**`git push origin BRANCH`

**清理本地master分支：**`git reset --hard origin/master`

**使用远端覆盖：**`git reset --hard origin/master`

### 配置

**别名：**`git config --global alias.st status`

**颜色：**`git config --global color.ui true`

**全局ignore：**`git config --global core.excludesfile '~/.gitignore'`

### 历史

找出谁修改的`git blame`，-w忽略空格,-L m,n指定行号

git reflog找到amend的head，然后git reset --soft HEAD@{n}
