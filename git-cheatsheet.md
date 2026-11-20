# Git 常用命令速查表

> 面向本机环境（Windows + Git 2.56 + SSH over 443）整理，涵盖日常 90% 的场景。
> 记不住没关系，先记这一句：**改完 → add → commit → push**。

---

## 0. 一句话心智模型

```
工作区（你正在改的文件）
   │  git add        ← 挑选要提交的改动
   ▼
暂存区（staging）
   │  git commit     ← 打包成一个版本，存进本地仓库
   ▼
本地仓库（.git）
   │  git push       ← 上传到 GitHub
   ▼
远程仓库（origin / GitHub）
```

反向操作：`git pull`（远程 → 本地），`git checkout`（仓库 → 工作区）。

---

## 1. 本机已配置好的东西（不用再弄）

| 项目 | 值 |
|---|---|
| 用户名 | `xuantian123` |
| 邮箱 | `1536492774@qq.com` |
| SSH 密钥 | `C:\Users\15364\.ssh\id_ed25519`（无密码短语） |
| SSH 通道 | `ssh.github.com:443`（22 端口被网络封锁，已改 443） |
| 远程地址 | `git@github.com:ee20oo/hello-world.git` |

检查是否配好：

```powershell
git config --global --list          # 看全局配置
ssh -T git@github.com               # 应返回 Hi ee20oo! You've successfully authenticated
```

---

## 2. 每天都会用的 5 条

```powershell
git status                  # 我现在改了啥？（最常用，没事就敲）
git add .                   # 把所有改动放到暂存区
git commit -m "改了什么"     # 打包成一个版本
git pull                    # 拉取远程更新（干活前先做）
git push                    # 上传
```

**标准工作循环**：`pull` → 改文件 → `status` → `add` → `commit` → `push`

---

## 3. 建仓库 / 克隆

```powershell
# 克隆已有仓库
git clone git@github.com:用户名/仓库名.git
git clone https://github.com/用户名/仓库名.git      # HTTPS 方式（本机 HTTPS 有证书问题，建议用 SSH）

# 把已有文件夹变成 Git 仓库
cd 我的项目
git init
git add .
git commit -m "初始提交"
git remote add origin git@github.com:用户名/仓库名.git
git branch -M main
git push -u origin main
```

> 网页上建仓库时勾选 `Add a README file`，克隆下来就有内容，省得处理空仓库。

---

## 4. 分支：改东西前先开分支

```powershell
git branch                      # 列出本地分支（* 是当前分支）
git branch -a                   # 连远程分支一起列
git switch -c feature/login     # 新建并切换到 feature/login（推荐写法）
git switch main                 # 切回 main

git merge feature/login         # 在 main 上合并 feature/login
git branch -d feature/login     # 合并后删掉分支

git push -u origin feature/login   # 把新分支推到远程（首次要 -u）
```

`git checkout -b xxx` 和 `git switch -c xxx` 等价，后者是新写法，更不容易误操作。

---

## 5. 后悔药（按“还没 push / 已 push”分）

### 还没提交

```powershell
git restore 文件名               # 丢弃某个文件的修改（危险，改了就没了）
git restore .                    # 丢弃所有未暂存的修改
git restore --staged 文件名       # 把文件从暂存区撤出（add 错了用它）
```

### 提交了但没 push

```powershell
git commit --amend -m "新的说明"          # 改最后一次提交的说明
git commit --amend --no-edit              # 把漏掉的文件补进上一次提交
git reset --soft HEAD~1                   # 撤销最后一次提交，改动保留在暂存区
git reset --mixed HEAD~1                  # 撤销提交和暂存，改动留在工作区
git reset --hard HEAD~1                   # 彻底回退（改动全丢，慎用！）
```

### 已经 push 了

```powershell
git revert <commit-id>      # 生成一个“反向提交”来抵消，安全，适合已推送的内容
git log --oneline -5        # 先找到要撤销的 commit-id
```

### 临时不想提交

```powershell
git stash          # 把当前改动“打包收起来”，工作区变干净
git stash list     # 看收起来了哪些
git stash pop      # 取回来（并从列表删除）
git stash apply    # 取回来（保留在列表里）
```

---

## 6. 远程仓库操作

```powershell
git remote -v                                    # 当前远程地址
git remote set-url origin git@github.com:用户/仓库.git   # 换地址（HTTPS ↔ SSH）
git remote add upstream git@github.com:原作者/仓库.git   # 加“上游”（Fork 后用）

git fetch origin        # 只下载远程信息，不动工作区（安全）
git pull                # fetch + merge，直接更新工作区
git pull --rebase       # 变基式拉取，历史更干净
git push                # 上传
git push -u origin main # 首次推送并绑定上游
```

---

## 7. 看历史、看差异

```powershell
git log --oneline -10              # 最近 10 条提交，一行一条
git log --oneline --graph --all    # 图形化看分支合并
git show <commit-id>               # 看某次提交具体改了什么
git diff                           # 工作区 vs 暂存区
git diff --staged                  # 暂存区 vs 最后一次提交
git diff main feature/login        # 两个分支的差异
git blame 文件名                    # 每行是谁改的、哪次提交
```

---

## 8. 冲突处理（pull / merge 后出现 `CONFLICT`）

冲突文件里会出现：

```
<<<<<<< HEAD
你本地的版本
=======
远程的版本
>>>>>>> origin/main
```

处理流程：

1. 打开冲突文件，**手工决定留哪段**，删掉 `<<<<<<<`、`=======`、`>>>>>>>` 这三行标记
2. 保存，`git add 冲突文件`
3. `git commit`（不用写 -m，Git 会预填合并说明）
4. `git push`

半路想放弃合并：`git merge --abort`

---

## 9. `.gitignore` 常见内容

在仓库根目录建 `.gitignore`，以下内容不会进仓库：

```gitignore
# 依赖
node_modules/
.venv/
venv/

# 编译产物
dist/
build/
*.exe
*.pyc
__pycache__/

# 编辑器
.vscode/
.idea/

# 密钥与本地配置（重要！）
.env
*.key
*.pem
id_rsa
id_ed25519

# 系统文件
.DS_Store
Thumbs.db
```

已经误提交了怎么办：

```powershell
git rm --cached 文件名      # 从 Git 里移除但保留本地文件
```

---

## 10. GitHub 网页协作流程（Fork → PR）

```text
1. 找到别人的项目 → 点 Fork（复制到你账号下）
2. 克隆你自己的那份：git clone git@github.com:你的用户名/项目.git
3. 加原作者仓库为上游：
   git remote add upstream git@github.com:原作者/项目.git
4. 开分支改东西：git switch -c fix/typo
5. 提交并推到你自己的仓库：git push -u origin fix/typo
6. 网页上点 Compare & pull request → 填说明 → Create pull request
7. 原作者 review 后合并，你的名字就进了贡献者列表
```

日常保持同步：

```powershell
git fetch upstream
git switch main
git merge upstream/main
git push
```

新手找活干：项目的 Issues 里筛 `good first issue` 或 `help wanted` 标签。

---

## 11. 常见报错速查

| 报错 | 原因 | 解法 |
|---|---|---|
| `Failed to connect ... over proxy` | git 配了失效代理 | `git config --global --unset http.proxy`（本机已清理） |
| `schannel: CRYPT_E_NO_REVOCATION_CHECK` | 证书吊销检查失败 | 改用 SSH 远程地址，别用 HTTPS（本机已改） |
| `Permission denied (publickey)` | SSH 公钥没加到 GitHub | 复制 `~/.ssh/id_ed25519.pub` 到 GitHub → Settings → SSH keys |
| `Updates were rejected` | 远程有你本地没有的提交 | 先 `git pull --rebase`，再 `git push` |
| `src refspec main does not match any` | 本地还没有 main 分支 / 没提交过 | 先 `git add .` + `git commit` |
| `detached HEAD` | 切到了某个提交而非分支 | `git switch main` 回主分支，或 `git switch -c 新分支` |
| `fatal: not a git repository` | 当前目录不是仓库 | `cd` 到仓库目录，或 `git init` |
| `Your local changes would be overwritten` | 有未提交改动挡住了 pull | 先 `git stash`，pull 完再 `git stash pop` |
| 中文文件名显示成乱码 | 编码设置 | `git config --global core.quotepath false` |

---

## 12. 提交说明怎么写

```
类型: 简短描述

详细说明（可选）
```

常用类型：

| 类型 | 含义 |
|---|---|
| `feat` | 新功能 |
| `fix` | 修 bug |
| `docs` | 改文档 |
| `style` | 格式调整（不影响逻辑） |
| `refactor` | 重构 |
| `chore` | 杂活（依赖、配置） |

例：`docs: 补充安装步骤`、`fix: 修复登录按钮无响应`

---

## 13. 命令速记卡（打印版）

```powershell
git status                 # 啥状态
git add .                  # 全部暂存
git commit -m "说明"        # 提交
git pull                   # 拉取
git push                   # 推送
git log --oneline -5       # 最近历史
git switch -c 新分支        # 开分支
git switch main            # 回主分支
git restore 文件            # 丢弃修改
git stash / git stash pop  # 暂存/恢复
```
