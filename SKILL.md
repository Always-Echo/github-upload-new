---
name: github-upload-new
description: 将本地文件夹首次上传到 GitHub，自动完成"创建远程仓库 + git init + 推送"全流程。使用 GitHub CLI（gh）创建仓库，SSH 方式推送代码。当用户提到"把文件夹上传到 GitHub"、"把本地项目推送到 GitHub"、"创建 GitHub 仓库并上传"、"新建仓库上传代码"、"本地代码推到 GitHub"、"upload to GitHub"、"push local folder to GitHub" 时必须使用此 Skill。即使用户只说"上传到 GitHub"或"推到 GitHub"也应触发。注意：此 Skill 仅适用于远程仓库尚未创建的首次上传场景，不适用于已有仓库的推送更新（git push）。
---

# GitHub Upload Skill

将本地文件夹一键上传到 GitHub 的完整流程，适用于**远程仓库尚未创建**的场景。

> 本 Skill 全程使用 **SSH 协议**，用户需要在本地提前配置好 SSH Key 并添加到 GitHub。

---

## 前置条件检查

在执行任何操作前，依次检查以下三项：

### 1. 检查 SSH Key 是否存在

```bash
ls ~/.ssh/
```

查找是否有 `id_ed25519`、`id_ed25519_github`、`id_rsa` 等私钥文件。

- **找到了**：继续下一步
- **找不到**：停止执行，告知用户：
  > "本 Skill 需要本地配置 SSH Key。请提供你的 SSH 公钥文件路径（如 `~/.ssh/id_ed25519.pub`），或先生成一个：
  > ```bash
  > ssh-keygen -t ed25519 -C "your_email@example.com"
  > ```
  > 然后将公钥内容添加到 GitHub → Settings → SSH and GPG keys。"

### 2. 检查 SSH 是否能连通 GitHub

```bash
ssh -T git@github.com 2>&1
```

- 输出包含 `successfully authenticated`：SSH 正常，继续
- 输出包含 `Permission denied`：SSH Key 未添加到 GitHub，提示用户：
  > "请将以下公钥内容添加到 GitHub → Settings → SSH and GPG keys：
  > ```bash
  > cat ~/.ssh/<你的公钥文件>.pub
  > ```"
- 其他错误：检查 `~/.ssh/config` 是否正确配置了 GitHub host

### 3. 检查 gh CLI 是否已安装并登录

```bash
gh auth status 2>&1
```

- **未安装**：提示用户执行 `brew install gh`（macOS），安装后继续登录步骤
- **未登录**：提示用户在终端手动执行以下命令完成一次性登录（需要浏览器交互，Agent 无法代替执行）：
  ```bash
  gh auth login --hostname github.com --git-protocol ssh
  ```
  按以下顺序选择：
  1. 选 **GitHub.com**
  2. 协议选 **SSH**
  3. 选择已有的 SSH Key（如 `~/.ssh/id_ed25519_github.pub`），填写 Key 标题（如 `my-mac`）
  4. 选 **Login with a web browser**，浏览器完成授权后回到终端按回车

  > **登录是一次性操作，永久生效。** gh 会将凭证保存在系统 keyring 中，后续无需重复登录。登录成功后终端会显示 `✓ Logged in as <用户名>`。

  完成后告知 Agent 继续。

- **已登录**：继续执行，输出示例：
  ```
  github.com
    ✓ Logged in to github.com account <用户名> (keyring)
    - Git operations protocol: ssh
  ```

---

## 执行流程

### Step 1：确认目标文件夹

从用户消息中提取（或询问）：
- 要上传的本地文件夹路径（绝对路径）
- 希望创建的 GitHub 仓库名称（默认使用文件夹名）
- 仓库描述（可选，默认为空）

### Step 2：检查目录 Git 状态

```bash
ls /path/to/folder/.git 2>&1
```

- **不是 git 仓库**（报错）：执行初始化
  ```bash
  cd /path/to/folder
  git init
  git add .
  git commit -m "Initial commit"
  ```
- **已是 git 仓库**：检查是否有未提交内容
  ```bash
  git -C /path/to/folder status
  ```
  如有未提交内容，询问用户是否一并提交。

### Step 3：检查分支名

```bash
git -C /path/to/folder branch --show-current
```

如果分支名是 `master`，重命名为 `main`：
```bash
git -C /path/to/folder branch -M main
```

### Step 4：检查是否已有 remote origin

```bash
git -C /path/to/folder remote -v
```

如果已有 origin，询问用户是否移除并重新创建：
```bash
git -C /path/to/folder remote remove origin
```

### Step 5：用 gh CLI 创建远程仓库并推送

```bash
cd /path/to/folder && gh repo create <仓库名> --public --source=. --remote=origin --push
```

这条命令会同时完成：
- 在 GitHub 上创建公开仓库
- 自动添加 remote origin（SSH 地址）
- 直接 push 到 main 分支

> 如果用户希望创建私有仓库，将 `--public` 改为 `--private`。

### Step 6：验证上传结果

```bash
gh repo view --web
```

在浏览器中打开仓库页面，让用户确认内容已正确上传。

---

## 常见问题处理

**问题：gh repo create 报错 "already exists"**

仓库名已被占用，询问用户换一个名称后重试。

**问题：push 时报 "remote: Permission to ... denied"**

SSH Key 没有对应账号的写权限，检查 gh 登录的账号是否正确：
```bash
gh auth status
```

**问题：有文件不想上传（如 node_modules、.env）**

提示用户先创建 `.gitignore`，例如：
```bash
cat > /path/to/folder/.gitignore << 'EOF'
node_modules/
.env
.DS_Store
*.log
EOF
git -C /path/to/folder add .gitignore
git -C /path/to/folder commit -m "Add .gitignore"
```

---

## 注意事项

- 全程使用 **SSH 协议**，不使用 HTTPS，避免每次输入密码
- `gh repo create --push` 自动处理 remote 添加和推送，无需手动 `git remote add` 和 `git push`
- gh 登录步骤需要用户在终端手动完成（浏览器授权），Agent 无法代替执行
- 登录一次后永久生效，后续无需重复登录
