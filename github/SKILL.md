---
name: github
version: 1.0.0
description: 通过 GitHub 官方 CLI（gh）连接用户账号和仓库。在用户要求列出或克隆 GitHub repo、读取代码、管理 issue 和 pull request、查看 Actions、提交或推送代码时使用。支持 GH_TOKEN 环境变量及 gh 网页授权登录。
---

# GitHub 仓库助手

## 工具与认证

使用 shell_execute 调用 GitHub 官方 CLI `gh` 和 `git`，无需 MCP 服务。执行前用 `which gh` 检查；缺失时运行 `apk add github-cli`。

先检查认证，不输出任何凭据：

```sh
if [ -n "$GH_TOKEN" ] || [ -n "$GITHUB_TOKEN" ]; then
  echo 'GitHub credential environment is set'
else
  gh auth status
fi
gh api user --jq '{login: .login, name: .name}'
```

没有认证时，优先让用户在 Minis 的环境变量设置中添加 fine-grained PAT，变量名为 `GH_TOKEN`。引导用户只选择需要的仓库，设置有效期，按任务授予最小权限；阅读代码通常需要 Contents: Read-only，写代码需要 Contents: Read and write，issue/PR 权限按需添加。链接：

- GitHub Token 创建页：`https://github.com/settings/personal-access-tokens/new`
- Minis 设置：`minis://settings/environments?create_key=GH_TOKEN&create_value=&create_note=GitHub%20CLI%20repository%20access`

另一种方式是官方网页授权。在用户可交互的终端执行：

```sh
gh auth login --hostname github.com --git-protocol https --web
```

终端链接必须对 init_command 完整 URL 编码。说明 Linux 环境缺少系统凭据库时，gh 会把授权凭据明文写到本地配置文件，勿将该文件分享、提交或复制到普通工作目录。不要读取凭据文件或执行 `gh auth token`。不要索要密码或要求用户把 Token 发到聊天。不要将凭据写入 skill、脚本、记忆、Git URL 或命令参数。不要输出环境变量值。不要使用 `gh auth status --show-token`。

认证成功后再执行 `gh auth setup-git --hostname github.com`，让 git 通过 gh 获取 HTTPS 凭据。切勿配置会把 Token 明文另存的 credential.store。

## 常用任务

验证身份并列出仓库，默认只读：

```sh
gh api user --jq '{login: .login, name: .name}'
gh repo list --limit 50 --json nameWithOwner,isPrivate,description,url
```

Fine-grained PAT 只能访问选定仓库，列表不一定代表所有有权限的仓库；已知 `OWNER/REPO` 时直接用 `gh repo view` 验证。遇到 403 或 404 时检查仓库选择、权限、组织审批和 SSO；不要直接扩大权限。

```sh
gh repo view OWNER/REPO --json nameWithOwner,description,url,defaultBranchRef,isPrivate
gh issue list --repo OWNER/REPO --limit 30
gh pr list --repo OWNER/REPO --limit 30
gh pr view NUMBER --repo OWNER/REPO
gh run list --repo OWNER/REPO --limit 10
```

克隆到 `/var/minis/workspace/github/OWNER/REPO`，先确认目标不存在，避免覆盖。修改代码前读取仓库约定文件，运行 `git status`，保留用户未提交的改动；扫描文件用 `rg`。需要长期保留的独立交付件放到 `/var/minis/shared/`。

## 权限和变更边界

默认只读。用户明确请求的常规修改、提交或推送可按授权范围执行；没有明确授权的远程写入先用 user_ask 确认。删除仓库、强制推送、删除分支、改可见性、合并 PR 和修改权限等破坏性或影响他人的操作，先确认具体目标与影响。绝不顺带创建、关闭、合并或删除远程对象。

报告账号、实际验证的仓库、完成的操作和限制。仅安装工具不算连接成功；仅认证成功不算已验证特定仓库访问权限。
