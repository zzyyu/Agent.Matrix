# GitHub PR / Issue 上传

## 用途
把本地改动以 Pull Request 形式提交到 GitHub 仓库 `philfanzhou/OpenAgent`，或在仓库创建 Issue（上报缺陷、提出需求、跟踪任务）。

## 触发条件
- 用户要求"提交 PR"、"上传代码"、"发起合并请求"、"推到 GitHub"
- 用户要求"创建 issue"、"上报问题"、"把任务提交到 GitHub"

## 仓库与凭证（2026-11-09 实测）
- 仓库：`https://github.com/philfanzhou/OpenAgent`（公开仓库，默认分支 `main`，未归档）
- 认证：Paperclip 托管 GitHub 凭证（用户 `zzyyu`）；git 的 HTTPS 操作自动认证，SSH 地址自动重写为 HTTPS
- **实测权限结论（重要）**：
  - 读公开仓库、`GET /user`：✅ 正常
  - 对自有仓库（含 fork `zzyyu/Agent.Matrix`，admin）：✅ 完全读写
  - 对 `philfanzhou/OpenAgent` 的**写操作**（git push、建分支、开 PR、建 issue）：❌ 全部 403 `Resource not accessible by integration` — 连接凭证的授权范围未覆盖该仓库
  - ⚠️ `GET /repos` 返回的 `permissions.push: true` 是账号协作者权限，**不代表凭证 token 有写权限**；判断真实能力必须实测一次写操作，不要只看 permissions 字段

## 工作流程

### 一、上传 PR（当前可用路径：fork + 分支）
1. 同步最新 `main`：`git fetch origin && git rebase origin/main`
2. 建分支，命名 `agent/<主题>`（与仓库既有 `agent/*` 分支约定一致，如 `agent/fix-readme-paths`）
3. 提交改动：遵循 `coding-conventions.md`；一次 PR 只做一件事，提交信息用英文祈使句
4. 推送到 fork（直接 push `philfanzhou/OpenAgent` 会 403）：
   ```bash
   git remote add fork https://github.com/zzyyu/Agent.Matrix.git   # 已有则跳过
   git push -u fork <branch>
   ```
5. 创建 PR，二选一：
   - **API**（需连接授权覆盖上游仓库）：`POST /repos/philfanzhou/OpenAgent/pulls`，`base: main`，`head: zzyyu:<branch>`；当前返回 403，说明授权未覆盖
   - **一键比较 URL**（始终可用，由维护者点击创建）：
     `https://github.com/philfanzhou/OpenAgent/compare/main...zzyyu:Agent.Matrix:<branch>`
6. 把分支名/比较链接返回给用户；评审或 CI 意见回来后在同一分支追加提交再 push

### 二、上传 Issue
1. 先搜索既有 issue（`GET /repos/philfanzhou/OpenAgent/issues?state=open`，只读接口当前可用），避免重复
2. 创建，二选一：
   - **API**（需连接授权覆盖上游仓库）：`POST /repos/philfanzhou/OpenAgent/issues`；当前返回 403
   - **网页端**（始终可用）：`https://github.com/philfanzhou/OpenAgent/issues/new`，把写好的标题+正文交给维护者粘贴
3. 标题一句话结论式描述（缺陷："模块 + 现象 + 影响"；需求："希望支持 + 能力"）；正文含背景、复现步骤/场景、期望行为、验收标准
4. 把 issue 链接返回给用户

### 三、解除写权限限制（根因修复）
托管凭证对上游仓库无写权限时，两条路：
- 在 Paperclip 的 GitHub 连接设置里，把 `philfanzhou/OpenAgent` 加入授权（install/repo scope）并确保 token 含 `contents: write`、`pull_requests: write`、`issues: write`；授权后上述 API 路径可直接使用，无需 fork
- 或由维护者点击第 5 步的比较链接手动开 PR（零配置，每次多一步人工）

## 注意
- 不要为"测试上传"创建垃圾 PR/issue；验证连通性用只读接口（`GET /repos/philfanzhou/OpenAgent`）
- PR/issue 正文遵循 `.agent/rules/doc-standards.md` 的文档风格
- Token 只用于 API/git 调用，不写入文件、日志或 issue/PR 正文
- 合并决定权在仓库维护者；agent 不自行 merge，除非用户明确要求
- 权限结论有时效性：连接授权变更后，先用一次低风险写操作（如建分支）复测，再更新本文档

## 参考文件
- 编码规范：`.agent/rules/coding-conventions.md`
- 文档风格：`.agent/rules/doc-standards.md`
- 项目入口：`AGENTS.md`

## 验证方法
- PR：分支已推送到 fork、比较链接可访问（或 API 建的 PR `GET /repos/philfanzhou/OpenAgent/pulls/{n}` 返回 200）
- Issue：issue 编号已创建、链接可访问（`GET /repos/philfanzhou/OpenAgent/issues/{n}` 返回 200）
