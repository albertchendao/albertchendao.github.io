---
layout: article
title: Git 常用命令整理
tags: [Wiki]
key: 
---

`GIT` 是一个分布式版本控制系统，可以用来记录若干个文件的内容变化，从而查看或者回退到文件某个版本。

## 特性

1. 记录文件快照而非内容差异
2. 近乎所有操作都是本地执行
3. 使用校验和监控文件的变动

## 文件状态

所有文件都有三种状态: 已修改(modified)，已暂存(staged)，已提交(committed)
对应的三个区域为: 工作目录，暂存区域，以及本地仓库

基本的 Git 工作流程如下：

1. 在工作目录中修改某些文件。
2. 对修改后的文件进行快照，然后保存到暂存区域。
3. 提交更新，将保存在暂存区域的文件快照永久转储到 Git 目录中。

## 安装

Linux 下安装:

```bash
# Fedora
yum install git-core
# Debian
apt-get install git
```

Mac 下安装，使用图形界面安装或者使用如下命令:

```bash
brew install git
```

## 配置

可以使用 `git config` 命令来查看并修改配置，所有的配置有三个地方存放:

1. `/etc/gitconfig` 文件：系统中对所有用户都普遍适用的配置，使用 `git config --system` 操作
2. `~/.gitconfig` 文件：用户目录下的配置文件只适用于该用户，使用 `git config --global` 操作
3. 工作目录中的 `.git/config` 文件：这里的配置仅仅针对当前项目有效，使用 `git config --local`

查看所有配置可以使用如下命令:

```bash
git config --list
```

| 常用配置    | 含义                           |
| ----------- | ------------------------------ |
| user.name   | 用户名，commit 提交时会使用    |
| user.email  | 用户邮箱，commit 提交时会使用  |
| core.editor | 默认编辑器，一般是 vi 或者 vim |
| merge.tool  | 合并冲突时的工具               |

### 忽略文件配置

当有些文件无需纳入 Git 的管理时可以添加忽略到当前工作目录的 `.gitignore` 文件中，使用如下规则：

1. 所有空行或者以注释符号 ＃ 开头的行都会被 Git 忽略。
2. 可以使用标准的 glob 模式匹配。
3. 匹配模式最后跟反斜杠（/）说明要忽略的是目录。
4. 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

glob 模式:

1. 星号（*）匹配零个或多个任意字符
2. [abc] 匹配任何一个列在方括号中的字符
3. 问号（?）只匹配一个任意字符
4. [0-9] 表示匹配所有 0 到 9 的数字

## 常用命令

| 常用命令                                              | 含义                                                           |
| ----------------------------------------------------- | -------------------------------------------------------------- |
| git init                                              | 初始化本地仓库                                                 |
| git clone {url} [name]                                | 从现有仓库克隆                                                 |
| git status                                            | 检查文件状态                                                   |
| git diff                                              | 工作目录 vs 暂存区域差异                                       |
| git diff --cached                                     | 暂存区域 vs 已提交差异                                         |
| git diff --staged                                     | 暂存区域 vs 已提交差异                                         |
| git commit [-a] -m {message}                          | 提交更新                                                       |
| git commit --amend                                    | 重新提交                                                       |
| git rm {name}}                                        | 从 git 跟踪文件中移除文件，工作目录中也会删除                  |
| git rm --cached                                       | 从暂存区域移除，但仍然保留在当前工作目录                       |
| git mv                                                | 移动文件                                                       |
| git log                                               | 查看历史                                                       |
| git log -2                                            | 查看历史，显示最近两条                                         |
| git log -p                                            | 查看历史，显示内容差异                                         |
| git log --stat                                        | 查看历史，显示增改行数统计                                     |
| git log --pretty=oneline                              | 查看历史，将每个提交放在一行显示                               |
| git add [name]                                        | 添加文件到暂存区                                               |
| git reset HEAD {name}                                 | 取消暂存                                                       |
| git checkout -- {name}                                | 取消修改，回到修改前的版本                                     |
| git remote -v                                         | 查看远程仓库                                                   |
| git remote add {name} {url}                           | 添加远程仓库                                                   |
| git remote show [remote-name]                         | 查看远程仓库详细信息                                           |
| git remote rename pb paul                             | 修改远程仓库名称                                               |
| git remote rm paul                                    | 删除远程仓库                                                   |
| git fetch [remote-name]                               | 从远程仓库抓取数据到本地                                       |
| git fetch {remote-name} {branch-name}:{branch-name}   | 创建本地分支保存远端分支的所有数据                             |
| git pull                                              | 在本地跟踪分支上执行，从远程仓库抓取数据到本地，自动合并       |
| git push [remote-name] {local-branch}:[remote-branch] | 推送本地据推到远程仓库                                         |
| git push [remote-name] :{remote-branch}               | 删除远程分支                                                   |
| git push --set-upstream {origin} {local-branch}       | 推送本地分支到远程仓库并跟踪远程分支                           |
| git branch                                            | 显示所有分支                                                   |
| git branch --no-merged                                | 显示尚未合并的分支                                             |
| git branch {name}                                     | 新建分支                                                       |
| git branch --set-upstream-to={origin/master} {master} | 追踪远程分支                                                   |
| git branch -d {name}                                  | 删除分支                                                       |
| git checkout {name}                                   | 切换分支                                                       |
| git checkout -b {name}                                | 快速新建并切换分支，如果是从远程分支上创建的会自动变成跟踪分支 |
| git checkout --track {name}                           | 创建一个跟踪分支                                               |
| git merge {name}                                      | 合并分支到当前分支                                             |
| git tag                                               | 显示所有标签                                                   |
| git tag -l 'v1.4.2.*'                                 | 列出指定格式的标签                                             |
| git tag -a v1.4 -m 'my version 1.4'                   | 创建指定的标签                                                 |
| git tag -a v1.2 9fceb02                               | 在指定的版本上创建标签                                         |
| git stash list                                        | 查看储藏的修改                                                 |
| git stash [name]                                      | 储藏修改                                                       |
| git stash drop [stash@{0}]                            | 删除储藏                                                       |
| git stash branch {name}                               | 从储藏创建一个新分支                                           |
| git stash apply [stash@{2}]                           | 应用储藏的修改                                                 |

## 常见问题

### `git status` 乱码

在中文下 `git status` 会显示成 `\351\205`, 解决方法是执行如下命令:

```bash
git config --global core.quotepath false
```

### `remote ref does not exist`

使用 `git push origin :develop` 删除远程分支时出现: `error: unable to delete 'develop': remote ref does not exist`，执行如下命令:

```bash
git fetch -p origin
```

### 删除远程分支

一开始用

```bash
git branch -r -d origin/branch-name
```

不成功，发现只是删除的本地对该远程分支的 track，正确的方法应该是这样：

```bash
git push origin :branch-name
```

冒号前面的空格不能少，原理是把一个空分支 push 到 server 上，相当于删除该分支。

### 记住密码

为了让git 客户端记住用户名与密码，需要如下配置：
项目目录下.git\config 文件，增加两行

```bash
[credential]
helper = store
```

### 换行符

PS: 最好的解决方案是执行以下步骤：（比如，有一个 git 库叫做 mygitrepo）

1. 增加 .gitattribute 文件，在mygitrepo 下建立一个 .gitattributes 文件，在其中输入

    ```bash
    * text eol=lf
    ```

    详见<https://help.github.com/articles/dealing-with-line-endings#platform-all>

2. 通过你的客户端命令行修改换行设置，输入

    ```bash
    git config --global core.autocrlf false
    git config --global core.safecrlf true
    ```

3. 通过你的客户端命令行输入

    ```bash
        git add .gitattributes
        git commit .gitattributes -m "commit .gitattributes"
    ```

4. 项目组统一编辑器和 IDE 的换行符，推荐 UNIX 风格的 LF

## 参考链接

1. [Pro Git](https://gitee.com/progit/)
