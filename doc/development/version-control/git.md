<!--
 * @Author: yyt
 * @Date: 2020-05-19 10:22:23
 * @LastEditTime: 2020-05-22 17:39:15
 * @LastEditors: yyt
 * @FilePath: /taoNote/doc/development/version-control/git.md
-->

# 图解

![一图理解git](https://yangyuetao.cn/demos/static/img/taoNote/1.png)

# 常用命令

## 新建

- 在当前目录新建一个 Git 代码库
  - `git init`
- 新建一个目录，将其初始化为 Git 代码库
  - `git init [project-name]`
- 下载一个项目和它的整个代码历史
  - `git clone [url]`
- 以当前分支的当前状态创建新分支并切换到新分支
  - `git checkout -b [xxx]`
- 新建一个分支，指向某个 tag
  - `git checkout -b [xxx] [tag]`

## 配置

- 显示当前的 Git 配置
  - `git config --list`
- 编辑 Git 配置文件
  - `git config -e [--global]`
- 设置提交代码时的用户信息
  - `git config [--global] user.name "[name]"`
  - `git config [--global] user.email "[email address]"`
- 设置别名
  - `git config --global alias.co checkout`
  - `git config --global alias.br branch`

## 增加/删除文件

- 添加
  - `git add .` 添加当前目录的所有文件到暂存区
  - `git add -p` 添加每个变化前，都会要求确认.对于同一个文件的多处变化，可以实现分次提交
    - y add 此块
    - n 放弃此块
    - q 退出 add
    - a add 整个文件
    - d 放弃整个文件
    - e 编辑
  - `git add [file1] [file2]` 指定文件
  - `git add [dir]` 添加指定目录,包括子目录
- 删除
  - `git rm [file1][file2]` 删除工作区文件，并且将这次删除放入暂存区
  - `git rm --cached [file]` 停止追踪指定文件，但该文件会保留在工作区
  - `git mv [file-original][file-renamed]` 改名文件，并且将这个改名放入暂存区(需要路径)

## 提交

- 提交暂存区的指定文件到仓库区
  - `git commit [file1][file2] ... -m [message]`
- 提交暂存区到仓库区
  - `git commit -m [message]`
- 使用一次新的 commit，替代上一次提交
  - `git commit --amend -m [message]`

## 变基

- 合并多次 commit
  - `git rebase -i [commitId]`
    - 比如提交了 1，2，3，想合并 3 者，只需要找到 1 之前的 commitId
    - pick1 squash2 squash3
    - 则提交就合并了，并且只有第一次的 msg

## 远程同步

- 下载远程仓库的所有变动
  - `git fetch -a`
- 显示所有远程仓库
  - `git remote -v`
- 显示某个远程仓库的信息
  - `git remote show [remote]`
- 取回远程仓库的变化，并与本地分支合并
  - `git pull [branch]`
- 上传本地指定分支到远程仓库
  - `git push [branch]`
- 强行推送当前分支到远程仓库，即使有冲突
  - `git push -f`

## 回滚撤销

- 重置暂存区与工作区，与上一次 commit 保持一致
  - `git reset --hard`
- 重置当前分支的指针为指定 commit
  - `git reset --mixed [commit]` 同时重置暂存区，但工作区不变
    - `git reset [commit]` 默认--mixed
  - `git reset --hard [commit]` 同时重置暂存区和工作区
  - `git reset --soft [commit]` 完全保留工作区和暂存区，仅改变 HEAD 的指向的位置
- 强制拉取
  - `git fetch --all`
  - `git reset --hard origin/master`
- 恢复 commit，但是不会影响后续的提交
  - `git revert [commit]`
- 回滚单个文件
  - `git checkout -- xx/xx`
- 时光机，记录所有操作
  - `git reflog`

## 分支

- 列出所有本地分支
  - `git branch`
- 列出所有远程分支
  - `git branch -r`
- 列出所有本地分支和远程分支
  - `git branch -a`
- 新建一个分支，但依然停留在当前分支
  - `git branch [branch-name]`
- 新建一个分支，并切换到该分支
  - `git checkout -b [branch]`
- 切换到指定分支，并更新工作区
  - `git checkout [branch-name]`
- 切换到上一个分支
  - `git checkout -`
- 合并指定分支到当前分支
  - `git merge [branch]`
- 删除分支
  - `git branch -d [branch-name]`
- 删除远程分支
  - `git push origin -d [branch-name]`

## 查看信息

- 显示有变更的文件
  - `git status`
- 显示当前分支的版本历史
  - `git log`
    - 优化 `git lg`
- 显示 commit 历史，以及每次 commit 发生变更的文件
  - `git log --stat`
    - 优化 `git ll`
- 搜索提交历史，根据关键词
  - `git log -S [keyword]`
  - `git log -S [keyword] --stat`
- 显示指定文件相关的每一次 diff
  - `git log -p [file]`
- 显示指定文件每一行是什么人在什么时间修改过
  - `git blame [file]`
- 显示暂存区和工作区的差异
  - `git diff`
- 显示两次提交之间的差异
  - `git diff [first-branch]...[second-branch]`
- 显示某次提交的元数据和内容变化
  - `git show [commit]`
- 显示某次提交时，某个文件的内容
  - `git show [commit] [filename]`
- 显示所有提交过的用户，按提交次数排序
  - `git shortlog -sn`
- 显示今天你写了多少行代码
  - `git diff --shortstat "@{0 day ago}"`

## 标签

- 列出所有 tag
  - `git tag`
- 查看 tag 信息
  - `git show [tag]`
- 新建 tag
  - `git tag [tag]` 在当前 commit
  - `git tag [tag][commit]` 在指定 commit
- 删除 tag
  - `git tag -d [tag]` 本地
  - `git push origin :refs/tags/[tagName]` 远程
- 提交所有 tag
  - `git push [remote] --tags`
- 新建一个分支，指向某个 tag
  - `git checkout -b [branch] [tag]`

## 贮藏

- `git stash save "xxx"` 贮藏
- `git stash list`所有贮藏
- `git stash show` ：显示做了哪些改动，默认 show 第一个存储
  - `git stash show stash@{1}`如果要显示其他存贮
- `git stash apply` 应用某个存储,但不会把存储从存储列表中删除
  - `git stash pop` 应用某个存储,会把存储从存储列表中删除
- `git stash drop` 丢弃
- `git stash clear` 清空所有

## 一些 alias

- checkout => `co`
- branch => `br`
- log => `lg`简略,`ll`具体变动
- diff => `d`
- add + commit -m => `acm [msg]`
- status => `s`

# 案例

## 克隆 带账号密码

- `git clone http://账号%40qq.com:密码@gitee.com/xxxxxx.git`
  - 注意：用户名密码中一定要转义 @符号转码后变成了%40

## 解决 git pull/push 每次都需要输入密码问题

- 进入你的项目目录
- `git config --global credential.helper store`
- 下次 push 或者 pull 会输入一次密码，之后不需要了

## --depth=1

- 只拉去最近一次的提交记录
  - `git clone -b [branch] --depth=1 [url]`可以拉对应分支的最近一次提交，无其他记录了

## 只 pull 某一个文件/夹

- `git config core.sparsecheckout true` //此方法适用于 Git1.7.0 以后版本，之前的版本没有这个功能
- 在.git/info/sparse-checkout 文件中（如果没有则创建）添加指定的文件/夹
  比如,则只会有 dist

```bash
/dist
```

## 自动补全分支名

- git-completion
  - `brew install bash-completion`
  - `vim ~/.zshrc`
  ```bash
  # auto-completion
  if [ -f /opt/local/etc/profile.d/bash_completion.sh ]; then
    . /opt/local/etc/profile.d/bash_completion.sh
  fi
  ```
  - `source ~/.zshrc`
