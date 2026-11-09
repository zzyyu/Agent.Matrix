# GitHub PR / Issue 上传

## 用途
把本地改动以 Pull Request 形式提交到 GitHub 仓库 `philfanzhou/OpenAgent`，或在仓库创建 Issue（上报缺陷、提出需求、跟踪任务）。

## 触发条件
- 用户要求"提交 PR"、"上传代码"、"发起合并请求"、"推到 GitHub"
- 用户要求"创建 issue"、"上报问题"、"把任务提交到 GitHub"

## 仓库与权限（已实测验证）
- 仓库：`https://github.com/philfanzhou/OpenAgent`（公开仓库，默认分支 `main`，未归档）
- 认证：使用运行环境注入的 GitHub 凭证；git 的 HTTPS 操作自动认证，SSH 地址会被重写为 HTTPS
- PR 权限：账号对仓库 `push: true`，可直接推送分支并开 PR，**无需 fork**
- Issue 权限：`has_issues: true` 且 `triage: true`，可创建 issue 并打标签
- API 调用：优先 `gh` CLI；环境未安装时走 REST API，请求头 `Authorization: Bearer $GITHUB_TOKEN`，`Accept: application/vnd.github+json`

## 工作流程

### 一、上传 PR
1. 同步最新 `main`：`git fetch origin && git rebase origin/main`
2. 建分支，命名 `agent/<主题>`（与仓库既有 `agent/*` 分支约定一致，如 `agent/fix-readme-paths`）
3. 提交改动：遵循 `coding-conventions.md`；一次 PR 只做一件事，提交信息用英文祈使句
4. 推送分支：`git push -u origin <branch>`
5. 创建 PR：
   - REST：`POST /repos/philfanzhou/OpenAgent/pulls`，`base: main`，`head: <branch>`
   - 标题一句话说清改动；正文包含：动机/背景、改动点列表、验证方式（构建/测试结果）、关联 issue（`Closes #N`）
6. 把 PR 链接返回给用户；评审或 CI 意见回来后在同一分支追加提交，PR 自动更新

### 二、上传 Issue
1. 先搜索既有 issue（`GET /repos/philfanzhou/OpenAgent/issues?state=open`），避免重复
2. 创建：`POST /repos/philfanzhou/OpenAgent/issues`
   - 标题：一句话结论式描述（缺陷格式："模块 + 现象 + 影响"；需求格式："希望支持 + 能力"）
   - 正文：背景、复现步骤/场景、期望行为、验收标准
3. 按需打标签（triage 权限已验证），并在正文链接相关 PR/issue
4. 把 issue 链接返回给用户

## 注意
- 不要为"测试上传"创建垃圾 PR/issue；验证连通性用只读接口（`GET /repos/philfanzhou/OpenAgent`）
- PR/issue 正文遵循 `.agent/rules/doc-standards.md` 的文档风格
- Token 只用于 API/git 调用，不写入文件、日志或 issue/PR 正文
- 合并决定权在仓库维护者；agent 不自行 merge，除非用户明确要求

## 参考文件
- 编码规范：`.agent/rules/coding-conventions.md`
- 文档风格：`.agent/rules/doc-standards.md`
- 项目入口：`AGENTS.md`

## 验证方法
- PR：分支已推送、PR 状态为 open、链接可访问（`GET /repos/philfanzhou/OpenAgent/pulls/{n}` 返回 200）
- Issue：issue 编号已创建、链接可访问（`GET /repos/philfanzhou/OpenAgent/issues/{n}` 返回 200）
