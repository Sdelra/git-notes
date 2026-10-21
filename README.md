# Git 分布式版本控制系统学习笔记

> 基于廖雪峰 Git 教程整理，补充了现代命令写法和常见误区辨析。
> 适用版本：Git 2.23 及以上（`switch` 与 `restore` 从该版本开始提供）。

## 目录

- [一、核心概念](#一核心概念)
- [二、安装与初始化](#二安装与初始化)
- [三、基本操作](#三基本操作)
- [四、版本回退与撤销](#四版本回退与撤销)
- [五、远程仓库](#五远程仓库)
- [六、分支管理](#六分支管理)
- [七、临时任务的处理](#七临时任务的处理)
- [八、变基](#八变基)
- [九、标签管理](#九标签管理)
- [十、命令速查表](#十命令速查表)

---

## 一、核心概念

### 1.1 三个区域

Git 把代码的存放位置分成三个区域，理解它们是理解所有命令的前提。

| 区域 | 位置 | 作用 |
| --- | --- | --- |
| 工作区（Working Directory） | 你能直接看到的项目目录 | 日常编辑文件的地方 |
| 暂存区（Stage / Index） | `.git/index` | 下一次提交要包含的内容 |
| 版本库（Repository） | `.git` 目录 | 保存所有提交历史 |

`.git` 是一个**目录**，不是文件。它是 Git 的版本库，不属于工作区内容，用 `ls -ah` 可以看到这个隐藏目录。

### 1.2 文件的四种状态

| 状态 | 含义 | 如何进入下一状态 |
| --- | --- | --- |
| Untracked | 新文件，还没被 Git 跟踪 | `git add` |
| Modified | 已跟踪，工作区有改动但未暂存 | `git add` |
| Staged | 改动已进入暂存区，等待提交 | `git commit` |
| Unmodified | 与版本库一致，没有任何改动 | 编辑文件 |

另外还有两种特殊状态：**Deleted**（已删除待提交）和 **Conflict**（合并冲突待解决）。

---

## 二、安装与初始化

### 2.1 首次配置

提交记录会带上作者信息，第一次使用必须先配置。

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git config --global init.defaultBranch main
```

查看当前配置：

```bash
git config --list
```

### 2.2 创建仓库

在项目目录下执行：

```bash
git init
```

执行后目录内会出现隐藏的 `.git` 目录，Git 从此开始管理这个目录。

### 2.3 克隆已有仓库

```bash
git clone git@github.com:用户名/仓库名.git
```

`git clone` 会自动完成三件事：下载代码、建立 `origin` 远程关联、检出默认分支。不需要再执行 `git init` 或 `git remote add`。

### 2.4 配置 SSH 密钥

使用 `git@github.com:...` 这种地址，需要先把本地公钥加到 GitHub。

```bash
ssh-keygen -t ed25519 -C "你的邮箱"
cat ~/.ssh/id_ed25519.pub
```

把输出的内容粘贴到 GitHub 的 Settings → SSH and GPG keys 中。

---

## 三、基本操作

### 3.1 添加到暂存区

```bash
git add <文件>        # 添加指定文件
git add .            # 添加当前目录下所有改动
git add -A           # 添加全部改动，包括删除
```

### 3.2 提交

```bash
git commit -m "提交说明"
```

可以一次 `add` 多个文件，然后一起提交。提交说明建议写清楚「做了什么」而不是「修改了文件」。

修改最近一次提交（还没推送时使用）：

```bash
git commit --amend -m "新的提交说明"
```

### 3.3 查看状态

```bash
git status
```

### 3.4 查看差异

`git diff` 有三个常用变体，区别在于比较的两端不同：

| 命令 | 比较对象 |
| --- | --- |
| `git diff` | 工作区 vs 暂存区 |
| `git diff --cached` | 暂存区 vs 最新提交 |
| `git diff HEAD` | 工作区 vs 最新提交 |

```bash
git diff HEAD -- <文件名>
```

### 3.5 查看提交历史

```bash
git log
git log --oneline --graph --decorate
```

`git log` 在分页模式下：按 `q` 退出，空格向后翻页，`b` 向前翻页。

---

## 四、版本回退与撤销

### 4.1 HEAD 的含义

- `HEAD`：当前版本
- `HEAD^`：上一个版本
- `HEAD^^`：上上个版本
- `HEAD~100`：往前 100 个版本

### 4.2 git reset 的三种模式

`git reset` 移动的是 `HEAD` 指针，三种参数的区别在于**是否同时重置暂存区和工作区**。

| 参数 | HEAD | 暂存区 | 工作区 | 结果 |
| --- | --- | --- | --- | --- |
| `--soft` | 移动 | 不变 | 不变 | 改动全部保留在暂存区，可直接重新提交 |
| `--mixed`（默认） | 移动 | 重置 | 不变 | 改动保留在工作区，需要重新 `add` |
| `--hard` | 移动 | 重置 | 重置 | 未提交的改动会被直接丢弃，无法找回 |

```bash
git reset --hard HEAD^          # 回退到上一个版本，丢弃改动
git reset --soft HEAD^          # 撤销提交，改动留在暂存区
git reset --mixed HEAD^         # 撤销提交，改动留在工作区
```

> `--hard` 是破坏性操作。执行前确认工作区的改动是否还需要。

### 4.3 reflog：找回「未来」

```bash
git reflog
```

`reflog` 记录的是 **`HEAD` 和分支引用的移动历史**，包括提交、回退、合并、切换分支、变基等操作，每一条都带 commit id。

它不是「命令请求记录」，也不是「不包含分支改动」。恰恰相反，分支的创建、切换、回退都会留下痕迹。正因为它记录了指针的每一次移动，才能在误删提交后把「未来」找回来。

```bash
git reset --hard <commit id>
```

commit id 一般写前 7 位即可，只要在仓库内能唯一区分。

### 4.4 撤销的三种场景

| 场景 | 命令 | 说明 |
| --- | --- | --- |
| 只改乱了工作区，还没 `add` | `git restore <文件>` | 用暂存区的内容覆盖工作区 |
| 已经 `add` 到暂存区，想撤回暂存 | `git restore --staged <文件>` | 取消暂存，工作区改动保留 |
| 已经 `commit`，还没推送 | `git reset --soft HEAD^` | 撤销提交，改动保留 |

旧写法与新写法对照：

```bash
git checkout -- <文件>        # 旧：撤销工作区改动
git restore <文件>            # 新：等价，推荐

git reset HEAD <文件>         # 旧：取消暂存
git restore --staged <文件>   # 新：等价，推荐
```

`git checkout -- <文件>` 的本质是用**暂存区**的版本覆盖工作区。所以如果文件已经 `add` 过一次、之后又改坏，撤销会回到「add 之后」的状态，而不是最初提交的状态。

### 4.5 删除文件

```bash
git rm <文件>
git commit -m "删除文件"
```

如果直接在文件管理器中删除，需要用 `git add -A` 把删除动作加入暂存区，或者用 `git rm` 一步到位。

从未被提交过、又被删除的文件，Git 无法恢复。

---

## 五、远程仓库

### 5.1 关联远程库

```bash
git remote add origin git@github.com:用户名/仓库名.git
git remote add gitee  git@gitee.com:用户名/仓库名.git
```

`origin` 只是远程库的**本地别名**，可以随便取名，但通常用 `origin` 表示主远程库。

### 5.2 查看与删除远程关联

```bash
git remote        # 只列出远程名
git remote -v     # 列出远程名与对应地址
git remote remove origin
```

`git remote remove`（或旧的 `git remote rm`）**只删除本地的关联配置，不会删除服务器上的仓库**。要真正删掉远程仓库，必须登录 GitHub 或 Gitee 网页端手动删除。

### 5.3 推送

```bash
git push -u origin main      # 第一次推送，同时建立追踪关系
git push origin main         # 之后的推送
git push origin <分支名>      # 推送指定分支
```

注意远程名要和 `git remote add` 时起的名一致。如果你关联的是 `origin`，推送时却写 `git push github master`，会报 `fatal: 'github' does not appear to be a git repository`。

本地分支名与远程分支名也要对应：本地叫 `main` 就推 `main`，本地叫 `master` 就推 `master`。

### 5.4 拉取

```bash
git pull              # 拉取并合并
git fetch             # 只拉取，不合并
git pull --rebase     # 拉取后变基，不产生合并提交
```

多人协作时，开工前先 `git pull` 是成本最低的冲突预防手段。

### 5.5 分支的追踪关系

```bash
git branch --set-upstream-to=origin/<分支名> <分支名>
git switch -c <分支名> origin/<分支名>   # 远程已有分支，拉下来并建立追踪
```

---

## 六、分支管理

### 6.1 基本操作

```bash
git branch                    # 查看分支，* 表示当前分支
git branch <分支名>            # 创建分支，但不切换
git switch <分支名>            # 切换分支
git switch -c <分支名>         # 创建并切换
```

`git checkout -b <分支名>` 与 `git switch -c <分支名>` 等价，后者语义更清晰，推荐使用。

### 6.2 合并

```bash
git merge <分支名>
```

合并有两种情况：

- **快进合并（Fast-forward）**：当前分支没有新提交，直接把指针前移，不产生新提交。
- **非快进合并**：两边的历史已经分叉，Git 会创建一个合并提交。

强制生成合并提交，保留分支存在过的记录：

```bash
git merge --no-ff -m "合并说明" <分支名>
```

Fast-forward 模式下删除分支后，历史看起来就像从没分过支；`--no-ff` 会留下一个合并节点，便于追溯。

### 6.3 解决冲突

两边修改了同一处内容时，Git 无法自动合并，会在文件里插入冲突标记：

```
<<<<<<< HEAD
当前分支的内容
=======
要合并进来的内容
>>>>>>> dev
```

处理步骤：

1. 手动编辑文件，删掉冲突标记，保留想要的内容。
2. `git add <文件>` 标记冲突已解决。
3. `git commit` 完成合并。

中途想放弃这次合并，回到合并前的状态：

```bash
git merge --abort
```

### 6.4 删除分支

```bash
git branch -d <分支名>   # 删除已合并的分支
git branch -D <分支名>   # 强制删除未合并的分支
```

不能删除当前所在的分支，需要先切换到其他分支。

### 6.5 一种常见的分支模型

| 分支 | 用途 |
| --- | --- |
| `main` | 主分支，保持稳定可发布 |
| `dev` | 开发主干，集成各功能 |
| `feature/xxx` | 个人功能分支，完成后合入 `dev` |

个人分支完成一个完整功能后再合并，不要长期保留半成品分支。

---

## 七、临时任务的处理

### 7.1 git stash

当工作做到一半、还不能提交，却需要切换分支时，可以先把现场「储藏」起来。

```bash
git stash                 # 储藏当前工作区与暂存区的改动
git stash list            # 查看所有储藏
git stash apply stash@{0} # 恢复，但不删除储藏
git stash pop stash@{0}   # 恢复，并删除该储藏
git stash drop stash@{0}  # 删除指定储藏
```

`stash` 只保存已被 Git 跟踪的文件的改动。新建但未 `add` 的文件不会进 stash，需要加 `-u`：

```bash
git stash -u
```

### 7.2 git worktree

`stash` 适合短时间的现场保存；如果要在同一个仓库里**同时打开多个工作目录**，用 `worktree` 更合适。

```bash
git worktree add <路径> <分支名>   # 基于指定分支新建工作目录
git worktree list                 # 列出所有工作目录
git worktree remove <路径>         # 删除指定工作目录
git worktree prune                # 清理已手动删除的目录留下的记录
```

典型场景：功能写到一半不想提交，但线上有 bug 要修。用 `worktree` 另开一个目录切到修复分支，两个工作区互不干扰，修完合并再删掉。

注意 `git worktree remove` 要在该工作目录之外执行。

### 7.3 git cherry-pick

把一个特定的提交复制到当前分支：

```bash
git cherry-pick <commit id>
```

它只复制那一次提交的内容，不复制整条历史。合并成功后会自动创建新提交。

典型场景：`dev` 上发现一个 bug，基于 `dev` 新建修复分支，修好并提交；切回 `dev` 合并；再切到 `main`，用 `cherry-pick` 把同一个修复复制过去。

```bash
git switch -c fix-bug dev
# 修改并提交
git switch dev
git merge --no-ff fix-bug
git switch main
git cherry-pick <修复提交的 id>
```

---

## 八、变基

### 8.1 为什么需要 rebase

多人同时改一个分支时，如果队友先推送，你后推送会被拒绝。此时 `git pull` 会拉取队友的提交并生成一个合并提交，历史图上出现一个「分叉再合拢」的节点。

```bash
git pull --rebase
```

这个命令会先把你的本地提交暂时移开，拉取远程提交，再把你的提交逐个「接」到新基础上，历史保持一条直线。

### 8.2 rebase 的代价

rebase 会**重写提交**，每个提交都会得到新的哈希值。

如果这些提交已经推送到远程、队友也在其基础上继续开发，你一旦 rebase，双方的本地历史就对不上了，会造成严重的混乱。

因此有一条基本原则：

> **只在尚未共享的本地提交上使用 rebase。永远不要对公共分支做 rebase。**

需要改写已被共享的历史，应该用 `git revert` 生成反向提交，而不是重写历史。

---

## 九、标签管理

标签用于标记发布版本，总是挂在某个提交上。

```bash
git tag                        # 查看所有标签
git tag v1.0                   # 在当前提交打轻量标签
git tag v1.0 <commit id>       # 给历史提交打标签
git show v1.0                  # 查看标签详情
```

带说明的附注标签更正式，推荐用于发布：

```bash
git tag -a v1.0 -m "第一个正式版本" <commit id>
```

`-a` 指定标签名，`-m` 指定说明文字，commit id 省略时默认打在最新提交上。

推送标签：

```bash
git push origin v1.0           # 推送单个标签
git push origin --tags         # 推送所有本地标签
```

`git push` 默认**不会**推送标签，必须显式推送。

删除标签：

```bash
git tag -d v1.0                              # 删除本地标签
git push origin :refs/tags/v1.0              # 删除远程标签
```

---

## 十、命令速查表

| 用途 | 命令 |
| --- | --- |
| 初始化 | `git init` |
| 克隆 | `git clone <地址>` |
| 添加 | `git add <文件>` / `git add -A` |
| 提交 | `git commit -m "说明"` |
| 状态 | `git status` |
| 差异 | `git diff` / `git diff HEAD` |
| 历史 | `git log --oneline --graph --decorate` |
| 回退 | `git reset --soft/--mixed/--hard HEAD^` |
| 找回 | `git reflog` |
| 撤销工作区 | `git restore <文件>` |
| 取消暂存 | `git restore --staged <文件>` |
| 远程列表 | `git remote -v` |
| 关联远程 | `git remote add origin <地址>` |
| 推送 | `git push -u origin main` |
| 拉取 | `git pull` / `git pull --rebase` |
| 分支列表 | `git branch` |
| 切换 | `git switch <分支>` |
| 新建并切换 | `git switch -c <分支>` |
| 合并 | `git merge --no-ff <分支>` |
| 储藏 | `git stash` / `git stash pop` |
| 多工作区 | `git worktree add <路径> <分支>` |
| 复制提交 | `git cherry-pick <commit id>` |
| 变基 | `git rebase <分支>` / `git pull --rebase` |
| 打标签 | `git tag -a v1.0 -m "说明"` |
| 推标签 | `git push origin --tags` |

---

