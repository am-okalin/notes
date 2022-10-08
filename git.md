[TOC]
## 帮助链接
[易百教程](https://www.yiibai.com/git)  
[廖雪峰教程](https://www.liaoxuefeng.com/)  
[版本控制相关](https://www.cnblogs.com/kevingrace/category/860276.html)  
[版本控制扩展](https://www.cnblogs.com/onelikeone/p/6857932.html)  
[github doc](https://docs.github.com/cn)  
[git安装](http://git-scm.com/download )

## 使用
- git托管平台包括`gitea`, `github`, `gitee`, `gitlab`等
- `webhooks`是一种机制: 当`对象`触发了`事件`时，`平台`向`配置URL`发送http-post请求
    + `对象`: 包括`仓库`,`组织`,`应用程序`
    + `事件`: 包括`push`, `pull`, `issues`等事件
- 将项目纳入版本管理
```shell
# 将已有项目纳入git管理
cd project1 && git init
# 新建的项目直接使用项目管理
git init project2 && cd project2

# 关联远程仓库
git remote add origin xxxxxxxx(仓库地址)

# 远程仓库有一些内容的话则先rebase
git pull --rebase origin master

# git add && commmit && push
```

### 多个第三方托管平台
当有多个git托管平台时，比如`gitee`和`github`，如何配置sshkey
```shell
# 在`~/.ssh`下创建`config`文件
vim ~/.ssh/config
# 测试SSH key是否配置成功
ssh -T git@gitee.com
ssh -T git@github.com
```
config文件内容如下
```shell
# gitee
Host gitee.com # 服务器
HostName gitee.com # 服务器名称
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id_rsa # 私钥路径
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa
```

## 命令
- 使用`git init`初始化仓库。此操作会在当前目录下创建`.git`的目录
- 使用`git status`查看当前仓库中的文件状态
- 使用`git add`命令将 未跟踪/已跟踪但修改 的文件添加到暂存区(并跟踪)
- 使用`git commit -m "message"`将暂存区文提交到本地仓库
- `git diff branchA branchB --stat` 查看仓库区别
- `git push <远程主机名> <本地分支名>:<远程分支名>`

### rebase
- `git rebase -i HEAD~2`
- `git rebase <branchName>`

### checkout
- `git checkout -b dev remotes/origin/dev`


### config
- `git config [scope] [action] [other]`
- scope 作用域。
    + `--system` 对系统所有登录用户有效`/etc/gitconfig`
    + `--global` 对当前用户所有仓库有效`~/.gitconfig`
    + `--local`  对本地仓库有效`./.git/config` (默认)
- action 对`git config`的操作
    + `--add key value` 添加一个`git config` (默认)
    + `--edit` 编辑对应`scope`的配置文件
    + `--list` 对系统所有登录用户有效`/etc/gitconfig`
    + `--unset key`  删除一个`git config`
- other 通常跟在`--list`后面
    + `--name-only` 仅展示`key`，不展示`value`
    + `--show-scope` 展示`git config`对应的`scope`
    + `--show-origin` 展示`git config`对应的`scope`文件地址

#### 配置项
- `user.name "am-okalin"`
- `user.email "928622276@qq.com"`
- `credential.helper store` http拉取项目时保存登录凭证
- `core.longpaths true` 解决git中'Filename too long'错误
- `core.quotepath false` 解决中文被显示为unicode编码的问题
- `core.filemode false` 关闭严格比较模式
- `core.autocrlf true/false/input` 换行符`win true`,`unix/mac input`
- `lfs install --skip-repo` 解除克隆容量必须小于100M的限制

### stash
- `git stash` 将当前工作区已修改状态下的文件储藏起来，在栈中增加一条存储。
- `git stash list` 列出所有存储
- `git stash pop` 重新应用最后一次储藏，同时立刻将其从堆栈中移走
- `git stash apply stash@{0}` 通过存储名应用到当前工作区，但并不会从栈中移除。
- `git stash show -p stash@{0} | git apply -R` 在某些情况下，你可能想应用储藏的修改，在进行了一些其他的修改后，又要取消之前所应用储藏的修改。Git没有提供类似于 stash unapply 的命令，但是可以通过取消该储藏的补丁达到同样的效果
- `git stash drop stashName` 移除名为stashName的存储
- `git stash branch branchName` 这会创建一个新的分支，检出你储藏工作时的所处的提交，重新应用你的工作，如果成功，将会丢弃储藏

### branch
- 设置跟踪信息 `git branch --set-upstream-to=origin/<branchName> <branchName>`
- `git branch -vv`
- `git branch -D <branchName>`
- `git branch --contains v0.0.5` 检测哪个分支包含此标签

### remote
- 链接项目到远程仓库`git remote add [shortname] [url]`
    + `git remote add origin git@github.com:am-okalin/notes.git`
- `git remote update origin --prune`
- `git remote -vv`

### tag
```shell
# 创建标签
git tag -a <tagname> -m "added description release notes" 
# 查看标签(l为默认选项)
git tag -l 
# 删除标签
git tag -d <tagname>
# 推送一个本地标签到远程仓库
git push origin <tagname>
# 推送全部未推送过的本地标签
git push origin --tags
# 删除一个远程标签
git push origin :refs/tags/<tagname>
# 获取远程指定tag版本
git fetch origin tag <tagname>
```

## cherrypick