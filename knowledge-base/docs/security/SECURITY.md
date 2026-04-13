# 🔐 安全管理指南

> 本文档规范了知识库的信息安全管理策略，所有贡献者必须遵守。

---

## 1. 仓库访问控制

### 1.1 仓库可见性

| 仓库 | 可见性 | 用途 |
|------|--------|------|
| `knowledge-base` | 🔒 Private | 综合知识库 |
| `knowledge-public` | 🌐 Public | 可公开的技术笔记 |

!!! danger "重要"
    包含任何个人信息、工作内容、账号密钥的仓库**必须设为 Private**。

### 1.2 协作者权限管理

| 角色 | 权限 | 说明 |
|------|------|------|
| 仓库所有者 | Admin | 完全控制权 |
| 核心成员 | Write | 可直接推送到非保护分支 |
| 访客/外部 | Read | 只读访问 |

**操作步骤：**

```
Settings → Collaborators and teams → Add people
→ 选择权限级别 → Send invitation
```

### 1.3 分支保护规则

对 `main` 分支启用以下保护：

```
Settings → Branches → Add branch protection rule

Branch name pattern: main

✅ Require a pull request before merging
  ✅ Require approvals: 1
  ✅ Dismiss stale pull request approvals when new commits are pushed

✅ Require status checks to pass before merging
  ✅ Require branches to be up to date before merging
  ✅ Status checks: Security Check

✅ Do not allow bypassing the above settings
```

---

## 2. 敏感信息防护

### 2.1 禁止提交的内容

| 类型 | 示例 | 风险等级 |
|------|------|----------|
| API 密钥 | `GITHUB_TOKEN=ghp_xxx...` | 🔴 极高 |
| 数据库密码 | `DB_PASSWORD=...` | 🔴 极高 |
| 私钥文件 | `*.pem`, `*.key` | 🔴 极高 |
| 内部 IP/域名 | 公司内网地址 | 🟡 中 |
| 个人身份信息 | 身份证号、手机号 | 🟡 中 |
| 会议纪要原文 | 包含敏感决策 | 🟡 中 |

### 2.2 提交前自查清单

每次提交前，请检查：

- [ ] 文件中没有硬编码的密钥或 Token
- [ ] 没有包含内网 IP 或域名
- [ ] 没有个人隐私信息（手机号、身份证等）
- [ ] 图片中没有敏感内容（截图注意打码）
- [ ] `.gitignore` 已覆盖所有敏感文件类型

### 2.3 如果不小心提交了敏感信息

!!! warning "紧急处理步骤"

    1. **立即轮换密钥** — 在对应平台重新生成密钥
    2. **从历史中清除** — 使用以下命令：

    ```bash
    # 方法一：BFG Repo-Cleaner（推荐）
    # 下载：https://rtyley.github.io/bfg-repo-cleaner/
    bfg --replace-text passwords.txt my-repo.git
    git reflog expire --expire=now --all
    git gc --prune=now --aggressive
    git push --force

    # 方法二：git filter-repo
    pip install git-filter-repo
    git filter-repo --path-glob '*.env' --invert-paths
    git push --force
    ```

    3. **通知团队** — 告知相关人员密钥已泄露并已轮换
    4. **检查是否被利用** — 查看相关服务的访问日志

---

## 3. GitHub 安全功能配置

### 3.1 必须开启的功能

```
Settings → Code security and analysis

✅ Dependency graph
✅ Dependabot alerts
✅ Dependabot security updates
✅ Secret scanning
✅ Secret scanning alerts → Notify partners
✅ Push protection → Block commits containing secrets
```

### 3.2 账号安全

| 措施 | 状态 | 说明 |
|------|------|------|
| 两步验证 (2FA) | ✅ 必须 | 使用 TOTP App（推荐 1Password/Authy） |
| SSH Key 认证 | ✅ 推荐 | 避免使用 HTTPS + 密码 |
| Token 最小权限 | ✅ 推荐 | PAT 只授予必要的 scope |
| Token 设过期时间 | ✅ 推荐 | 建议 90 天轮换一次 |
| 审计日志 | ✅ 定期 | 每月检查一次 Security log |

**SSH Key 配置：**

```bash
# 生成新的 SSH Key
ssh-keygen -t ed25519 -C "your_email@example.com"

# 添加到 ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 复制公钥到 GitHub
cat ~/.ssh/id_ed25519.pub
# 粘贴到 GitHub → Settings → SSH and GPG keys → New SSH key
```

---

## 4. 内容分级与隔离

### 4.1 分级标准

| 级别 | 标记 | 定义 | 存放位置 |
|------|------|------|----------|
| **公开** | 🟢 PUBLIC | 可公开分享的技术内容 | `knowledge-public` 仓库 |
| **内部** | 🟡 INTERNAL | 团队内部可见 | `knowledge-base` 仓库 |
| **机密** | 🔴 CONFIDENTIAL | 仅特定人员可见 | 加密文件 / 独立仓库 |

### 4.2 文档头部标记

每篇文档建议在头部标注安全级别：

```markdown
---
title: 文档标题
classification: INTERNAL  # PUBLIC / INTERNAL / CONFIDENTIAL
author: Sami
date: 2026-04-13
---
```

---

## 5. 自动化安全检查

### 5.1 CI/CD 安全扫描

本仓库已配置以下自动检查（见 `.github/workflows/security-check.yml`）：

| 检查项 | 工具 | 触发时机 |
|--------|------|----------|
| 密钥泄露检测 | TruffleHog | Push & PR |
| 敏感文件检测 | 自定义脚本 | Push & PR |
| Markdown 链接 | markdown-link-check | Push & PR |

### 5.2 Git Hooks（本地）

建议配置 pre-commit hook 进行本地检查：

```bash
# 安装 pre-commit
pip install pre-commit

# 创建配置文件 .pre-commit-config.yaml（已包含在仓库中）
pre-commit install
```

---

## 6. 定期安全审计

### 6.1 月度检查清单

- [ ] 审查仓库协作者列表，移除不需要的访问权限
- [ ] 检查 Personal Access Tokens，撤销过期/不用的
- [ ] 审查 SSH Keys，删除不再使用的
- [ ] 检查 Security log 是否有异常活动
- [ ] 确认 Secret scanning 告警已全部处理
- [ ] 确认仓库可见性仍为 Private

### 6.2 季度检查

- [ ] 轮换所有 Personal Access Tokens
- [ ] 审查分支保护规则是否仍然有效
- [ ] 评估是否需要调整内容分级

---

## 7. 应急响应

### 7.1 密钥泄露应急流程

```
发现泄露 → 立即轮换密钥 → 清理 Git 历史 → 通知相关方 → 事后复盘
    │
    └─ 5分钟内       30分钟内        1小时内       24小时内
```

### 7.2 联系方式

- 仓库所有者：Sami
- GitHub 安全团队：security@github.com

---

*本文档最后更新：2026-04-13*
*下次审计日期：2026-05-13*
