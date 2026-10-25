# 用命令行把文件上传到 GitHub（2026-10-25）

## 核心概念：git 的四个"地方"

```
工作区 ──git add──▶ 暂存区 ──git commit──▶ 本地仓库 ──git push──▶ GitHub
（文件夹）           （待提交的改动）        （.git 里的历史）        （远程仓库）
```

- `git add`：把改动放进这次要提交的清单
- `git commit`：给这批改动拍快照，存进本地历史
- `git push`：把本地新增的快照同步到 GitHub

commit 之后东西只在本机，push 之后才算上传。

## 一次性配置（已完成）

| 检查项 | 命令 | 我的情况 |
|---|---|---|
| git 是否安装 | `git --version` | 2.54.0（Apple Git） |
| 身份信息 | `git config --global --list` | 已配好 user.name / user.email |
| SSH 登录 | `ssh -T git@github.com` | 显示 `Hi chenyu-hust-ai!`，正常 |

- 没装 gh 和 brew，所以用 **SSH key** 认证，不用 HTTPS 密码。
- 本机已有 `~/.ssh/id_ed25519`（私钥，不能外传）和 `id_ed25519.pub`（公钥，已加到 GitHub → Settings → SSH and GPG keys）。
- `does not provide shell access` 是正常提示，不是报错。
- 如果 `ssh -T` 超时，多半是 22 端口被封，可以在 `~/.ssh/config` 里让 github.com 改走 `ssh.github.com` 的 443 端口。

## 新建仓库的流程

1. 网页右上角 **+** → **New repository**，选 **Private**，勾 **Add a README file**
2. **Code** 按钮 → **SSH** 标签，复制地址
3. 本地克隆：
   ```bash
   cd ~/Documents
   git clone git@github.com:chenyu-hust-ai/仓库名.git
   cd 仓库名
   ```

## 日常上传：每次拿到 md 文件后的五步

1. **下载文件**：点文件卡片下载，存到 `~/Downloads`
2. **打开终端，进入仓库**：⌘ + 空格 → 输入"终端" → 回车
   ```bash
   cd ~/Documents/computer-systems-claude
   ```
3. **把文件移进仓库**（最后的 `.` 表示当前文件夹）
   ```bash
   mv ~/Downloads/文件名.md .
   ```
4. **暂存并提交**
   ```bash
   git add .
   git commit -m "说明这次加了什么"
   ```
5. **上传**
   ```bash
   git push
   ```
   看到 `main -> main` 就完成了。

拿不准时，任何两步之间都可以跑 `git status`：
- 红色 `Untracked files` → git 看到了新文件，还没跟踪
- 红色 `Changes not staged` → 已跟踪的文件被改过，还没 add
- 绿色 `Changes to be committed` → 已进暂存区，可以 commit
- `Your branch is ahead of 'origin/main'` → 已 commit，还没 push

## 终端小技巧

- **关掉终端不影响任何东西**，仓库和配置都在硬盘上。重新打开后默认在 `~`，先 `cd` 进仓库。
- **不想打路径**：输入 `cd `（带空格），把文件夹从访达拖进终端窗口；或者打一部分按 **Tab** 自动补全。
- **快捷命令**：运行一次下面两条，以后输入 `csnotes` 就能直接进入仓库
  ```bash
  echo 'alias csnotes="cd ~/Documents/computer-systems-claude"' >> ~/.zshrc
  source ~/.zshrc
  ```
  `~/.zshrc` 是终端每次启动都会读取的配置文件。
- **用访达打开当前文件夹**：`open .`
- **`mv` 和 `cp`**：`mv` 是移动，原位置的文件会消失；`cp` 是复制，两边都留一份。
- **隐藏的 `.git` 文件夹**：存着全部历史记录，访达默认看不到，⌘ + Shift + . 切换显示。看看就好，别动里面的东西。

## 删除文件（本地 + 远程）

```bash
git rm 文件名
git commit -m "remove xxx"
git push
```

`git rm` 会同时删除文件并暂存这次删除。删掉的文件仍留在历史记录里；如果误传了密码、token，光 `git rm` 不够。

## 这次踩过的坑

- **`Repository not found`**：网页上还没建仓库，或者名字对不上。先建仓库再 clone。
- **一次粘贴多条命令**：第一条失败后，后面的命令在错误位置继续执行，结果 `test.md` 被建到了 `~/Documents`。不确定时一条一条跑。
- **仓库名用了中文**：GitHub 仓库名不支持中文，"计算机系统claude" 变成了 `-claude`。
- **名字以 `-` 开头**：命令行会当成选项。临时用 `./-claude` 绕过，最后改名为 `computer-systems-claude`：
  ```bash
  # 网页 Settings 里先改名，然后本地：
  mv ./-claude computer-systems-claude
  cd computer-systems-claude
  git remote set-url origin git@github.com:chenyu-hust-ai/computer-systems-claude.git
  ```
- **push 成功的标志**：最后出现 `main -> main`。
- **"下载"里找不到文件了**：因为用的是 `mv`，文件已经移进仓库文件夹了。
