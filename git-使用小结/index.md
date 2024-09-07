# Git 使用小结


<!-- # Git 使用小结 -->

<!-- > uchin/李玉勤   -->
<!-- > 20220124   -->

## 1. git 配置

### 1.1 git 三级配置

- System： `/etc/gitconfig`，Msys64 底下的 etc
- Global： `~/.gitconfig`，Msys64 底下的 home/\[UserName\]/w 为 ~
- 工程目录： `.git/gitconfig`

作用域依次减小，优先级依次升高

ps：不是安装 Msys64 而是 Window 版的 git 在 window 用户家目录下会有文件`.gitconfig`

```bash
git config --system --list  # 查看系统配置
git config --global --list  # 查看全局配置
git config --list           # 查看所有配置
git config -l               # 同上简写

git config --system -e      # 直接打开系统配置
git config --global -e      # 直接打开全局配置
git config -e               # 直接打开项目配置
```

<font color='red'>注：`git config --list` 查看的是当前项目所有的配置（综合系统配置与全局配置和项目配置），不是仅仅查看的 `.git` 目录下的配置</font>

### 1.2 修改 git 配置

通过上述命令查看用户名是否设置完成，没有需要进行设置

```bash
# 设置用户签名
git config --global user.name "userName"
git config --global user.email "userEmail"
```

<font color='red'>用户名和邮件是必须要配置的！</font> 无论采用用户名+密码验证方式的 http，还是采用密钥验证的 ssh，在和远程仓库关联时均只是在做权限验证，即是否能往该远程库推送。但具体提交的修改需要记录是谁做的，因此 git 系统中需要配置用户名与邮件地址来在远程仓库记录下这些操作是由谁来完成的。

具体参考[对给 git 配置邮箱和用户名的理解](https://blog.csdn.net/ITWANGBOIT/article/details/103618427#:~:text=1%E3%80%81%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E9%85%8D%E7%BD%AE%E7%94%A8%E6%88%B7,%E6%98%AF%E7%94%B1%E8%B0%81%E5%AE%8C%E6%88%90%E7%9A%84%E3%80%82)

除了设置全局签名，使用 `git config user.name "userName"` 来设置当前项目的签名也可以。

## 2. git 核心概念

### 2.1 区域

- 工作区（Workspace）=> 写代码
- 暂存区（Stage）=> 暂时缓存的修改
- 本地版本库（Local Repository）=> 本地仓库
- 远程仓库（Remote Repository）=> 远程仓库

工作区 => `git add` 添加到暂存区 => `git commit` 提交到本地库

只有在 `git commit` 进行提交到本地库后才有版本控制

### 2.2 HEAD 指针

- 标记当前分支：切换分支时会指向当前所在的分支
- 指向当前提交：当进行提交后，HEAD 指向最新的提交
- 作为检出点：回滚或切分支

### 2.3 分支

隔离开发干扰设计

## 3. 项目中常用命令

### 3.1 初始化与本地提交等

```bash
git init              # 初始化本地库
```

```bash
git status            # 查看本地库状态
```

查看文件有没有被修改、追踪

```bash
git add fileName          # 添加到缓存区
git add .                 # 添加当前目录所有文件进行追踪
git rm --cached fileName  # 删除缓存区的文件
```

没有被追踪文件/文件夹，存在工作区，通过 `add` 使其添加到缓存区  
git 追踪的文件发生修改是绿色,没被追踪的文件发生修改是红色

通过 `add` 添加至缓冲区的文件能通过 `git rm --cached` 从缓冲区中删除，不再追踪。命令删除的是缓存区的文件，工作区的文件依旧存在的

ps：windows 端 git 的 bash 里使用 vim 编辑文件后 `add` 命令会将 LF 转换成 CRLF，但是 msys2 的不会

```bash
git diff                 # 查看具体修改地方
```

```bash
git commit -m "日志信息" fileName # 提交本地库
```

```bash
git reflog # 查看版本信息
```

```bash
git log
```

```bash
# 版本穿梭
git reset --hard 版本号
```

示例如下：

```bash
[uchin@arch blog]$ git reflog
5341b98 (HEAD -> master) HEAD@{0}: commit: second commit
326787f HEAD@{1}: commit (initial): first commit

[uchin@arch blog]$ git reset --hard 326787f
HEAD is now at 326787f first commit

[uchin@arch blog]$ git reflog
326787f (HEAD -> master) HEAD@{0}: reset: moving to 326787f
5341b98 HEAD@{1}: commit: second commit
326787f (HEAD -> master) HEAD@{2}: commit (initial): first commit
```

#### gitignore

[忽略特殊文件](https://www.liaoxuefeng.com/wiki/896043488029600/900004590234208)

```txt
# =======================================
# .gitignore 示例
# =======================================

# 忽略所有 .a 结尾的文件
*.a
# 但 lib.a 除外
!lib.a
# 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
/TODO
# 忽略 build/ 目录下的所有文件
build/
# 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
doc/*.txt
# ignore all .txt files in the doc/ directory
doc/**/*.txt
```

#### tag

针对特定提交或者特定的时间点，赋予一个标识符，**相当于一个永久指针**

轻量级标签：指向某个 commit 的引用，不含附注信息

```bash
# 列出标签
git tag
git tag -l "v1.8.5*" # -l或 --list 进行筛选

# 打标签
git tag <tag-name> <commit-SHA>

# 删除标签
git tag -d [tag]

# 删除远程tag
git push origin :refs/tags/[tagName]
```

重量级标签：

### 3.2 分支操作

```bash
git branch              # 查看分支
git branch -r           # 查看远程分支
git branch -a           # 查看所有分支（包括远程）
git branch -vv          # 查看本地分支与远程分支的关系

git branch <name>       # 创建分支
git branch -d <name>    # 删除分支

git checkout <name>     # 切换分支
git branch -b <name>    # 创建并切换
git switch <name>       # 切换分支
git switch -c <name>    # 创建并切换

git merge <name>        # 合并分支
```

例如，在 `master` 分支中使用 `git merge dev` 命令，则会将 `dev` 分支的更新内容合并至 `master` 分支。

在 idea 中，直接从右下角找到 dev 分支，选择 Merge dev into master 即可。

### 3.3 远程操作

```bash

```

#### 3.3.1 克隆远程库

```bash
# https 协议下载
git clone https://gitee.com/yuqinlee/test.git
# ssh 协议下载
git clone git@gitee.com:liaoxuefeng/learngit.git
# 本地没有项目，直接拉取指定分支
git clone -b dev git@gitee.com:liaoxuefeng/learngit.git
# 本地已经有项目
git fetch --all                 # 先更新
git checkout -b dev origin/dev  # 在拉取

git fetch                       # 从远程拉取
git fetch -p                    # 如果远程分支删除了，拉取的时候同时清理本地对应分支
```

使用 `git clone` 命令下载远程库后，会自带版本信息

**`clone`、`fetch`、`pull`区别**：

- `clone`：远程克隆仓库，带版本控制
- `fetch`：从远程获取最新版本到本地，不会自动 merge
- `pull`：从远程获取最新版本并 merge 到本地仓库

#### 3.3.2 关联远程库

```bash
# http 协议关联
git remote add origin https://gitee.com/yuqinlee/test.git
# ssh 协议关联
git remote add origin git@gitee.com:liaoxuefeng/learngit.git
```

取消关联远程仓库

```bash
git remote remove origin
```

将本地分支关联到远程分支

```bash
git branch --set-upstream-to=origin/develop develop
```

#### 3.3.4 同步

```bash
git push -u origin "master" # 推送
```

---

_<center>远程关联仓库（gitee 示例）</center>_

Git 全局设置:

```bash
git config --global user.name "yuqinlee"
git config --global user.email "yuqin.lee@outlook.com"
```

创建 git 仓库:

```bash
mkdir test
cd test
git init
touch README.md
git add README.md
git commit -m "first commit"

# git remote add origin https://gitee.com/yuqinlee/test.git
git remote add origin git@gitee.com:liaoxuefeng/learngit.git

git push -u origin "master"

# 指定本地 dev 分支与远程 origin/dev 分支的链接，根据提示，设置dev和 origin/dev 的链接
git branch --set-upstream-to=origin/dev dev

```

已有仓库?

```bash
cd existing_git_repo
git remote add origin https://gitee.com/yuqinlee/test.git
git push -u origin "master"
```

### 3.4 子模块

使用 `git clone` 已经将远程克隆到本地了

```bash
git submodule init      # 子模块初始化
git submodule update    # 更新
```

使用 `git clone` 之前，使用 `--recursive` 参数

```bash
git clone --recursive <git@github.com:xxx/xxx.git>
```

添加子模块

```bash
# 拉取
git submodule add git@github.com:xxx/xxx.git <local-dir>

# 更新
git submodule update --remote --merge
```

---

_参考_

1. [Git 原理入门 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2018/10/git-internals.html)

2. [5.git 添加远程仓库,再克隆到本地,然后做分支管理](https://www.jianshu.com/p/1bc8809716ad)

3. [廖雪峰 git 教程](https://www.liaoxuefeng.com/wiki/896043488029600)

4. [git clone、git pull 和 git fetch 的用法及区别](https://segmentfault.com/a/1190000017030384)

5. [对给 git 配置邮箱和用户名的理解](https://blog.csdn.net/ITWANGBOIT/article/details/103618427#:~:text=1%E3%80%81%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E9%85%8D%E7%BD%AE%E7%94%A8%E6%88%B7,%E6%98%AF%E7%94%B1%E8%B0%81%E5%AE%8C%E6%88%90%E7%9A%84%E3%80%82)

6. <https://www.bilibili.com/video/BV1FE411P7B3?spm_id_from=..search-card.all.click>

7. [git 学习记录—— git 中的仓库、文件状态等概念介绍](https://www.cnblogs.com/yhjoker/p/7740203.html)

8. [lh-常用 Git 命令总结](https://blog.csdn.net/qq_42340309/article/details/125033439)

9. [代码同时更新到 Gitee 和 github](https://www.systemsci.org/jinshanw/2022/01/22/%E4%BB%A3%E7%A0%81%E5%90%8C%E6%97%B6%E6%9B%B4%E6%96%B0%E5%88%B0gitee%E5%92%8Cgithub/)

10. [√ git clone 含有子模块的项目](https://www.cnblogs.com/codingbit/p/git-clone-with-submodule.html)

---

注：使用 Msys2 的 git vscode 会有问题

