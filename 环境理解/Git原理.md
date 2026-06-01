# Git 原理

目标：直观理解 Git 如何管理一个本地仓库，以及它如何和 GitHub、Gitee、GitLab 等远端仓库同步。

---

## 一、先建立整体模型

Git 不是“自动备份文件夹”的工具，而是一个“版本快照数据库”。

一个 Git 仓库可以理解为：

```text
项目文件夹
├── 你看得见的普通文件
│   ├── README.md
│   ├── src/
│   └── ...
└── .git/
    ├── Git 的数据库
    ├── 分支指针
    ├── 暂存区
    └── 远端同步信息
```

你平时编辑的是普通文件；Git 真正管理历史的核心都藏在 `.git` 目录里。

最重要的心智模型：

```text
工作区 Working Tree
    ↓ git add
暂存区 Staging Area / Index
    ↓ git commit
本地仓库 Local Repository
    ↓ git push
远端仓库 Remote Repository
```

对应的几个动作：

| 动作 | 直观理解 | 数据从哪里到哪里 |
|---|---|---|
| `git add` | 选中这次要提交的变化 | 工作区 → 暂存区 |
| `git commit` | 给当前暂存区拍一张永久快照 | 暂存区 → 本地仓库 |
| `git push` | 把本地新提交上传到远端 | 本地仓库 → 远端仓库 |
| `git fetch` | 只下载远端新历史，不改工作文件 | 远端仓库 → 本地远端跟踪分支 |
| `git pull` | 下载远端新历史并合并到当前分支 | fetch + merge/rebase |

---

## 二、Git 仓库到底是什么

### 1. 普通目录变成 Git 仓库

执行：

```bash
git init
```

Git 会在当前目录创建一个隐藏目录：

```text
.git/
```

只要 `.git` 目录还在，这个文件夹就是一个 Git 仓库。删除 `.git` 后，普通文件还在，但 Git 历史、分支、远端信息都会丢失。

### 2. `.git` 目录里的关键内容

常见结构大致如下：

```text
.git/
├── HEAD
├── config
├── index
├── objects/
├── refs/
│   ├── heads/
│   ├── remotes/
│   └── tags/
└── logs/
```

| 路径 | 作用 |
|---|---|
| `.git/objects/` | Git 的对象数据库，保存文件内容、目录结构、提交记录 |
| `.git/index` | 暂存区，也叫 index |
| `.git/refs/heads/` | 本地分支指针，例如 `main`、`dev` |
| `.git/refs/remotes/` | 远端跟踪分支，例如 `origin/main` |
| `.git/HEAD` | 当前你站在哪个分支或哪个提交上 |
| `.git/config` | 当前仓库配置，例如远端地址、用户名规则 |
| `.git/logs/` | reflog，记录 HEAD 和分支指针移动历史 |

简单说：

```text
.git/objects 负责存历史内容
.git/refs    负责给历史内容贴名字
.git/HEAD    负责说明你现在在哪
.git/index   负责准备下一次提交
```

---

## 三、Git 如何保存文件：对象数据库

Git 底层主要有 4 类对象：

| 对象 | 保存什么 | 类比 |
|---|---|---|
| blob | 文件内容 | 一页纸 |
| tree | 目录结构，以及文件名到 blob 的映射 | 文件夹清单 |
| commit | 一次提交，指向一个 tree 和父提交 | 一张版本快照说明书 |
| tag | 给某个提交取一个固定名字 | 版本标签 |

### 1. blob：只保存文件内容

假设有一个文件：

```text
hello.txt
```

内容是：

```text
hello git
```

Git 不会把“hello.txt 这个文件”整体存进去，而是先把文件内容存成一个 blob。

注意：blob 只关心内容，不关心文件名。文件名由 tree 记录。

### 2. tree：保存目录结构

tree 记录：

- 文件名是什么。
- 文件权限是什么。
- 这个文件名对应哪个 blob。
- 子目录对应哪个 tree。

例子：

```text
项目根目录 tree
├── README.md  → blob abc123...
└── src        → tree def456...
    └── app.js → blob 789abc...
```

也就是说，blob 像文件内容，tree 像目录索引。

### 3. commit：保存一次完整快照

一次 commit 里通常包含：

- 当前项目根目录的 tree。
- 父提交 parent。
- 作者 author。
- 提交者 committer。
- 时间。
- commit message。

可以把 commit 想成：

```text
commit = 这次版本的目录快照 + 它从哪个历史版本来 + 说明文字
```

提交之间通过 parent 连起来，形成一条历史链：

```text
A ← B ← C ← D
```

这里的箭头表示“后一个提交指向前一个提交作为父提交”。

### 4. 为什么 Git 很省空间

Git 的模型是“快照”，但它不是每次都笨拙复制整个项目。

如果某个文件没有变化，新 commit 的 tree 可以继续引用旧的 blob：

```text
commit A
└── tree A
    ├── README.md → blob 111
    └── app.js    → blob 222

commit B
└── tree B
    ├── README.md → blob 111  # 没变，复用
    └── app.js    → blob 333  # 变了，新内容
```

Git 保存的是内容对象。相同内容只需要存一份。

---

## 四、哈希值：Git 给内容生成身份证

Git 对对象内容计算哈希值，例如：

```text
e83c5163316f89bfbde7d9ab23ca2e25604af290
```

这个哈希值就是对象 ID。

核心特点：

1. 内容相同，哈希相同。
2. 内容改一点，哈希会完全不同。
3. commit 的哈希也包含它指向的 tree、父提交、作者、时间和说明等信息。

因此，只要历史被篡改，后续 commit 的哈希都会变。

这就是 Git 能发现历史变化的原因。

---

## 五、工作区、暂存区、本地仓库

### 1. 三个区域

```text
┌────────────────────┐
│ 工作区 Working Tree │ 你正在编辑的真实文件
└─────────┬──────────┘
          │ git add
          ▼
┌────────────────────┐
│ 暂存区 Index        │ 下一次 commit 的候选快照
└─────────┬──────────┘
          │ git commit
          ▼
┌────────────────────┐
│ 本地仓库 Repository │ 已经保存下来的 commit 历史
└────────────────────┘
```

### 2. 为什么需要暂存区

暂存区让你可以把很多修改拆成几次提交。

例如你改了三个文件：

```text
login.py      修复登录 bug
README.md     更新说明
style.css     调整按钮样式
```

你可以只提交 bug 修复：

```bash
git add login.py
git commit -m "fix login bug"
```

再提交文档：

```bash
git add README.md
git commit -m "update README"
```

暂存区的本质是：你在告诉 Git，“下一次提交只包含这些变化”。

### 3. `git status` 在比较什么

`git status` 主要看三组差异：

```text
HEAD commit  vs  暂存区
暂存区       vs  工作区
当前分支     vs  上游分支
```

所以它能告诉你：

- 哪些内容已经 staged。
- 哪些内容还没有 staged。
- 哪些文件未被 Git 跟踪。
- 当前分支比远端 ahead 或 behind 多少个提交。

---

## 六、分支的本质：会移动的指针

很多人一开始以为分支是“复制了一份代码”。其实 Git 分支本质上只是一个指针。

假设当前历史：

```text
A ← B ← C
        ↑
       main
```

`main` 只是指向 commit C 的一个名字。

新建分支：

```bash
git branch dev
```

会变成：

```text
A ← B ← C
        ↑
      main
       dev
```

切到 `dev` 并提交一次：

```bash
git switch dev
git commit -m "add feature"
```

会变成：

```text
A ← B ← C ← D
        ↑     ↑
      main   dev
```

`dev` 向前移动了，`main` 没动。

### HEAD 是什么

`HEAD` 表示“你当前所在的位置”。

正常在分支上时：

```text
HEAD → main → commit C
```

切换到 `dev` 后：

```text
HEAD → dev → commit D
```

如果直接切到某个提交：

```bash
git checkout <commit-id>
```

可能进入 detached HEAD 状态：

```text
HEAD → commit B
```

这表示 HEAD 直接指向某个提交，而不是指向分支。此时新提交如果没有分支名字接住，之后可能不容易找到。

---

## 七、合并与变基

### 1. merge：保留分叉历史

假设：

```text
      D ← E  feature
     /
A ← B ← C    main
```

在 `main` 上执行：

```bash
git merge feature
```

如果不能 fast-forward，Git 会创建一个 merge commit：

```text
      D ← E
     /     \
A ← B ← C ← M  main
```

merge 的特点：

- 保留真实分叉历史。
- 多人协作时更容易看出“什么时候把某条分支合进来”。
- 历史可能出现较多分叉和合并节点。

### 2. rebase：把提交搬到新基底上

还是这个历史：

```text
      D ← E  feature
     /
A ← B ← C    main
```

在 `feature` 上执行：

```bash
git rebase main
```

Git 会把 `feature` 上的 D、E 重新应用到 C 后面：

```text
A ← B ← C ← D' ← E'  feature
          ↑
         main
```

rebase 的特点：

- 历史更线性。
- D'、E' 是新的提交，哈希会变。
- 不要随便 rebase 已经推送给别人共同使用的公共分支，否则会改写别人依赖的历史。

### 3. fast-forward：只是移动指针

如果 `main` 没有新提交：

```text
A ← B ← C       main
          \
           D ← E feature
```

合并时可以直接把 `main` 指针移动到 E：

```text
A ← B ← C ← D ← E
                  ↑
              main, feature
```

这叫 fast-forward，本质是“移动分支指针”，不需要新建 merge commit。

---

## 八、冲突是怎么产生的

冲突不是 Git 出错，而是 Git 无法自动判断“应该保留哪一边”。

典型场景：

```text
同一个文件
同一段代码
两条分支都改了
```

合并时 Git 可能写入冲突标记：

```text
<<<<<<< HEAD
当前分支的内容
=======
要合并进来的分支内容
>>>>>>> feature
```

解决冲突的过程：

1. 打开冲突文件。
2. 人工决定最终内容。
3. 删除冲突标记。
4. `git add` 标记为已解决。
5. 继续 `git commit` 或 `git rebase --continue`。

Git 能自动合并很多变化，但当语义判断必须由人做时，就会把选择权交给你。

---

## 九、远端仓库的本质

远端仓库不是一个神秘云盘，而是另一份 Git 仓库。

常见平台：

- GitHub
- Gitee
- GitLab
- Bitbucket
- 自建 Git 服务器

本地和远端的关系可以画成：

```text
你的电脑
┌──────────────────────────────┐
│ 本地仓库                      │
│                              │
│ main          本地分支        │
│ origin/main   远端跟踪分支    │
└───────────────┬──────────────┘
                │ 网络同步
                ▼
云端 / 公司服务器
┌──────────────────────────────┐
│ 远端仓库                      │
│                              │
│ main          远端分支        │
└──────────────────────────────┘
```

远端仓库也有自己的 objects、refs、branches。只是它通常没有你正在编辑的工作区，主要用来保存和交换提交历史。

### origin 是什么

`origin` 只是一个远端仓库的默认别名。

查看远端：

```bash
git remote -v
```

可能看到：

```text
origin  git@github.com:user/repo.git (fetch)
origin  git@github.com:user/repo.git (push)
```

意思是：

```text
origin = git@github.com:user/repo.git
```

你可以改名，也可以添加多个远端：

```bash
git remote add upstream git@github.com:other/repo.git
```

---

## 十、clone、fetch、pull、push 背后发生了什么

### 1. `git clone`

执行：

```bash
git clone git@github.com:user/repo.git
```

大致发生：

1. 创建本地目录。
2. 初始化 `.git`。
3. 添加远端别名 `origin`。
4. 下载远端仓库的 objects 和 refs。
5. 创建本地默认分支，例如 `main`。
6. 把文件检出到工作区。

结果：

```text
远端 main
    ↓ clone
本地 origin/main
    ↓ checkout
本地 main + 工作区文件
```

### 2. `git fetch`

执行：

```bash
git fetch origin
```

Git 会连接远端，比较双方已有对象，只下载你本地缺少的对象，然后更新远端跟踪分支：

```text
origin/main 向前移动
main 不动
工作区不动
```

所以 fetch 很安全。它只是让你知道远端现在长什么样。

### 3. `git pull`

执行：

```bash
git pull
```

通常等价于：

```bash
git fetch
git merge origin/main
```

如果配置了 rebase，则可能等价于：

```bash
git fetch
git rebase origin/main
```

所以 pull 不只是下载，它还会尝试改变当前分支和工作区。

### 4. `git push`

执行：

```bash
git push origin main
```

大致发生：

1. 本地连接远端。
2. 比较本地 `main` 和远端 `main`。
3. 找出远端缺少的 commit、tree、blob。
4. 打包上传这些对象。
5. 请求远端把 `main` 指针移动到新的提交。

如果远端分支包含你本地没有的提交，push 会被拒绝：

```text
本地 main:   A ← B ← C
远端 main:   A ← B ← D
```

这时直接 push 会覆盖别人历史，所以 Git 默认拒绝。你需要先：

```bash
git pull
```

处理合并或变基后再 push。

---

## 十一、远端跟踪分支是什么

本地经常同时存在：

```text
main
origin/main
```

它们不是同一个东西。

| 名称 | 含义 | 谁会移动它 |
|---|---|---|
| `main` | 你的本地分支 | 你本地 commit、merge、rebase |
| `origin/main` | 你上次看到的远端 main 状态 | `git fetch` 或 `git pull` |

一个常见状态：

```text
A ← B ← C ← D  main
          ↑
      origin/main
```

这表示本地比远端多 1 个提交，可以 push。

另一种状态：

```text
A ← B ← C      main
          \
           D    origin/main
```

这表示远端比本地多提交，需要先 pull 或 fetch 后处理。

再复杂一点：

```text
      D  main
     /
A ← B ← C  origin/main
```

这表示本地和远端都各自前进了，产生分叉。需要 merge 或 rebase 后再 push。

---

## 十二、云端同步不是文件同步，而是提交同步

很多新手会把 GitHub/Gitee 理解成“代码网盘”。这个理解有一半对，但容易误导。

云盘同步通常关心：

```text
哪个文件最新
```

Git 同步关心：

```text
哪些提交对象对方没有
分支指针应该移动到哪里
历史是否能安全连接
```

因此 Git 的同步单位不是单个文件，而是 commit 图。

### 例子：为什么必须先 commit 才能 push

你在本地改了文件，但没有 commit：

```text
工作区有修改
暂存区可能有修改
本地仓库没有新 commit
```

此时执行 push，远端不会收到这些未提交的修改。

因为 push 上传的是本地仓库里的 Git 对象，尤其是 commit，而不是你工作区里的临时文件状态。

### 例子：为什么别人看不到你本地分支

你创建了本地分支：

```bash
git switch -c experiment
```

但没有 push：

```text
本地有 experiment
远端没有 experiment
```

需要执行：

```bash
git push -u origin experiment
```

远端才会出现对应分支。

---

## 十三、一次完整协作流程

假设你和同学共同维护一个项目。

### 1. 第一次下载项目

```bash
git clone git@github.com:team/project.git
cd project
```

此时：

```text
本地 main      = 远端 main
origin/main   = 远端 main 的本地记录
工作区         = main 对应文件
```

### 2. 开始新功能

```bash
git switch -c feature-login
```

此时新分支只是一个指针。

### 3. 修改并提交

```bash
git status
git add src/login.py
git commit -m "add login page"
```

本地多了一个 commit。

### 4. 推送到远端

```bash
git push -u origin feature-login
```

远端出现 `feature-login` 分支。

### 5. 创建 Pull Request / Merge Request

在 GitHub/Gitee/GitLab 上创建 PR 或 MR。

平台会比较：

```text
base: main
compare: feature-login
```

它会显示：

- 新增了哪些 commits。
- 改了哪些文件。
- 是否能自动合并。
- CI 检查是否通过。

### 6. 合并后同步本地 main

```bash
git switch main
git pull
```

本地 `main` 更新到远端最新状态。

---

## 十四、常见命令背后的真正含义

| 命令 | 真正含义 |
|---|---|
| `git init` | 创建 `.git` 数据库 |
| `git add <file>` | 把工作区内容写入对象库，并更新暂存区 |
| `git commit` | 用暂存区创建 tree 和 commit，并移动当前分支指针 |
| `git branch <name>` | 创建一个指向当前 commit 的新指针 |
| `git switch <name>` | 移动 HEAD 到某个分支，并更新工作区文件 |
| `git checkout <commit>` | 让 HEAD 指向某个提交，可能进入 detached HEAD |
| `git merge <branch>` | 把另一条历史合入当前分支，必要时创建 merge commit |
| `git rebase <base>` | 把当前分支的提交复制到新基底后面 |
| `git fetch` | 下载远端新对象，更新远端跟踪分支 |
| `git pull` | fetch 后再 merge 或 rebase |
| `git push` | 上传本地缺少对象，并请求远端移动分支指针 |
| `git reset` | 移动分支指针，并可选择改暂存区和工作区 |
| `git revert` | 新建一个反向提交，撤销某次提交的效果 |
| `git stash` | 临时保存工作区和暂存区的修改 |

---

## 十五、reset、revert、restore 的区别

### 1. reset：移动当前分支指针

```bash
git reset --soft HEAD~1
```

撤销最近一次 commit，但保留暂存区。

```bash
git reset --mixed HEAD~1
```

撤销最近一次 commit，并取消暂存，但保留工作区修改。

```bash
git reset --hard HEAD~1
```

撤销最近一次 commit，并强制让暂存区和工作区回到目标提交。

`--hard` 会丢掉未保存修改，使用前要非常小心。

### 2. revert：新增一个反向提交

```bash
git revert <commit-id>
```

它不会改写历史，而是创建一个新的提交，用来抵消旧提交的效果。

公共分支上更推荐 revert，因为它不会让别人本地历史错乱。

### 3. restore：恢复文件内容

```bash
git restore file.txt
```

把工作区文件恢复到暂存区或 HEAD 的状态。

```bash
git restore --staged file.txt
```

把文件从暂存区撤回来，但保留工作区修改。

---

## 十六、`.gitignore` 的作用

`.gitignore` 告诉 Git 哪些未跟踪文件不需要纳入版本控制。

常见内容：

```gitignore
node_modules/
.venv/
__pycache__/
.env
dist/
.DS_Store
```

适合忽略：

- 依赖目录，例如 `node_modules/`、`.venv/`。
- 编译产物，例如 `dist/`、`build/`。
- 系统临时文件，例如 `.DS_Store`。
- 本地密钥配置，例如 `.env`。

注意：`.gitignore` 只对“尚未被 Git 跟踪的文件”自动生效。已经 commit 过的文件，即使后来加入 `.gitignore`，Git 仍会继续跟踪。

---

## 十七、Git LFS：大文件为什么要特殊处理

Git 擅长管理文本和中小型文件，不适合频繁变化的大型二进制文件，例如：

- 大视频。
- 大模型权重。
- 大型设计源文件。
- 数据集压缩包。

原因是 Git 会保存历史对象。大文件每改一次，就可能让仓库体积显著变大。

Git LFS 的思路是：

```text
Git 仓库里保存一个小指针文件
真正的大文件放到 LFS 存储服务
```

也就是说，普通 Git 管历史，LFS 管大文件内容。

---

## 十八、从本地到云端的一张总图

```text
┌─────────────────────────────────────────────────────────┐
│ 你的项目文件夹                                           │
│                                                         │
│  工作区                                                  │
│  ├── README.md                                           │
│  ├── src/app.js                                          │
│  └── ...                                                 │
│      │                                                   │
│      │ git add                                           │
│      ▼                                                   │
│  暂存区 .git/index                                       │
│      │                                                   │
│      │ git commit                                        │
│      ▼                                                   │
│  本地对象数据库 .git/objects                             │
│  ├── blob：文件内容                                      │
│  ├── tree：目录结构                                      │
│  └── commit：提交快照                                    │
│      │                                                   │
│      │ 本地分支 refs/heads/main                          │
│      ▼                                                   │
│  main 指向最新本地 commit                                │
└──────────────────────────┬──────────────────────────────┘
                           │ git push / git fetch / git pull
                           ▼
┌─────────────────────────────────────────────────────────┐
│ 远端 Git 仓库                                             │
│                                                         │
│  objects：远端已有的对象                                 │
│  refs/heads/main：远端 main 分支指针                     │
│  PR/MR、Issue、CI：平台额外提供的协作功能                │
└─────────────────────────────────────────────────────────┘
```

---

## 十九、学习 Git 时最重要的几个判断

### 1. 我现在在哪个分支

```bash
git branch
git status
```

看 `HEAD` 当前指向哪里。

### 2. 我改了什么

```bash
git status
git diff
git diff --staged
```

分别看工作区修改和暂存区修改。

### 3. 我的历史长什么样

```bash
git log --oneline --graph --decorate --all
```

这条命令非常适合建立图形化理解。

### 4. 本地和远端差多少

```bash
git fetch
git status
git log --oneline --graph --decorate --all
```

先 fetch，再观察 `main` 和 `origin/main` 的关系。

---

## 二十、核心总结

可以把 Git 记成四句话：

1. Git 仓库的核心是 `.git` 目录。
2. Git 保存的是由 blob、tree、commit 组成的对象图，而不是简单文件备份。
3. 分支只是指向 commit 的可移动指针，HEAD 表示当前所在位置。
4. 远端同步的是 commit 对象和分支指针，不是直接同步工作区文件。

最终心智模型：

```text
改文件 → add 选择变化 → commit 形成本地历史 → push 同步到远端
远端变化 → fetch 下载记录 → merge/rebase 整合 → 工作区更新
```

只要理解“对象数据库 + 分支指针 + 工作区/暂存区/本地仓库/远端仓库”这几件事，绝大多数 Git 命令都会变得有迹可循。
