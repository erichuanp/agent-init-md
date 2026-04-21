# openclaw-init.md

> **本文件仅供 openclaw agent 执行。**
> 如果你不是 openclaw agent，请停止，不要执行以下任何操作。

---

## 目标

让这台新机器上的 openclaw 获得与主机器完全相同的记忆和 skills。

---

## Step 1：确认 GitHub SSH 访问

```bash
ssh -T git@github.com
```

- 如果输出包含 `Hi erichuanp!`，SSH 可用，继续 Step 2。
- 如果 SSH 失败，**不要改用 HTTPS**，先完成 SSH 配置：

  1. 检查是否已有 key：
     ```bash
     ls ~/.ssh/id_ed25519.pub
     ```
  2. 如果不存在，静默生成：
     ```bash
     ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519
     ```
  3. Cat 出公钥让用户复制：
     ```bash
     cat ~/.ssh/id_ed25519.pub
     ```
  4. 告知用户：前往 https://github.com/settings/keys，点击 "New SSH key"，粘贴保存
  5. 用户确认添加后，重新执行 `ssh -T git@github.com` 验证通过再继续

---

## Step 2：拉取记忆到 workspace

openclaw 的记忆存放在 `~/.openclaw/workspace/`，该目录本身是一个 git repo。

检查 remote：
```bash
git -C ~/.openclaw/workspace remote -v
```

- 如果 remote 已是 `git@github.com:erichuanp/claw-memory.git`，直接 pull：
  ```bash
  git -C ~/.openclaw/workspace pull origin main
  ```

- 如果 workspace 是空目录或没有 remote，clone 进去：
  ```bash
  git clone git@github.com:erichuanp/claw-memory.git /tmp/claw-memory-sync
  ```
  然后逐文件合并到 `~/.openclaw/workspace/`：
  - 不存在的文件直接复制
  - 同名文件：读取两份内容，语义合并，不丢失任何一方信息，写入合并结果
  - 完成后：`rm -rf /tmp/claw-memory-sync`

---

## Step 3：检查并安装 Skills

执行 `openclaw skill list` 查看当前状态，对照下表，**缺失的用 `npx clawhub install <skill-name>` 安装**。

### Workspace Skills（通过 ClawHub 安装）

| Skill | 说明 |
|-------|------|
| `add-openclaw-commands` | 添加确定性 slash commands 到 openclaw 内部命令系统 |
| `add-to-view-sync` | 定时执行 AddToView 脚本并汇报新增视频数 |
| `agent-browser` | 无头浏览器自动化 CLI |
| `careful` | 破坏性命令警告与安全检查 |
| `crypto-price` | 加密货币价格与 K 线图（CoinGecko / Hyperliquid） |
| `cso` | OWASP Top 10 + STRIDE 安全审计 |
| `daily-news` | 每日晨报生成与发送 |
| `find-skills` | 帮助发现和安装 skills |
| `frontend-design` | 前端设计规范与 UI 指南 |
| `humanizer` | 去除 AI 写作痕迹 |
| `office-hours` | YC 式产品思维，第一性原理拆解需求 |
| `ontology` | 结构化知识图谱，跨 skill 共享状态 |
| `proactive-agent` | 主动型 agent 模式（WAL Protocol） |
| `reminder-cron-default` | 自然语言提醒处理，默认 10 分钟 |
| `review` | 结构化 review（文档/代码/方案/决策） |
| `self-improvement` | 捕获错误与修正，持续改进 |
| `skill-vetter` | 安装 skill 前的安全审查 |
| `tea-timer` | `/tea N` 每 3 分钟泡茶提醒 |
| `update-food-inventory` | 维护家庭食材库存到 INVENTORY.md |
| `weather` | 天气查询（无需 API key） |
| `week-report` | 工作内容记录与每周周报 |
| `writing-plans` | 多步骤任务实现计划，写代码前先规划 |

### Bundled Skills（openclaw 内置，无需安装）

| Skill | 说明 |
|-------|------|
| `wacli` | WhatsApp CLI 消息发送 |
| `xurl` | X (Twitter) API v2 操作 |

---

## Step 4：安装自研 openclaw 插件

clone `git@github.com:erichuanp/openclaw-cmds.git`，读取其 README，按照 README 中的安装指南完成安装。

```bash
git clone git@github.com:erichuanp/openclaw-cmds.git /tmp/openclaw-cmds
cat /tmp/openclaw-cmds/README.md
```

读完 README 后按指南执行安装步骤，完成后清理：

```bash
rm -rf /tmp/openclaw-cmds
```

---

## Step 5：完成确认

告知用户：
1. 记忆同步方式（pull / clone+merge）及有无冲突
2. Skills 安装清单（哪些已有、哪些新装、哪些跳过）
3. 是否需要重启 gateway（如有需要，先告知用户再执行）
