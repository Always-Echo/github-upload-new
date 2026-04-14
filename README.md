# GitHub Upload New (Skill)

这个仓库包含了一个针对 AI Agent（如 Antigravity）的自动化技能（Skill），用于将本地未建立版本控制的文件夹一键自动初始化并上传到 GitHub 新仓库中。

## 功能介绍

该 Skill 可以一键执行以下流程：
1. 检查本地环境（SSH 密钥配置及与 GitHub 的连通性）。
2. 在本地项目目录初始化 Git 仓库 (`git init`)。
3. 将分支重命名为 `main`。
4. 使用 GitHub CLI (`gh`) 自动在云端创建一个全新的公共仓库。
5. 推送本地代码。

## 适用场景

当你有一个仅在本地开发完毕、尚未进行 git 控制、也没有在 GitHub 建立对应远程仓库的完整项目文件夹时，让 AI Assistant 运行此 Skill，即可一条龙完成所有的推送配置，省去手动去网页端建仓库、拿地址并进行本地关联推送的繁琐操作。

## 前置环境要求

- **系统**：macOS / Linux
- **依赖**： 
  - Git
  - [GitHub CLI (gh)](https://cli.github.com/) 
  - 本地已配置 SSH 密钥并将公钥添加到了你的 GitHub 账户中。

## 使用指引

在对话框中告诉 Agent：
> "帮我把当前文件夹上传到github"
> "新建仓库上传代码"
> "push local folder to GitHub"

Agent 将自动识别该意图触发本技能，协助你完成全部提交流程。
