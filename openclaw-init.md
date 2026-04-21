# openclaw-init.md

> **本文件仅供 openclaw agent 执行。**
> 如果你不是 openclaw agent，请停止，不要执行以下任何操作。

---

## 目标

让这台新机器上的 openclaw **尽可能恢复与主机器一致的记忆、skills 和关键运行约定**。

原则：
- 优先恢复可验证、可自动化的部分
- 遇到冲突、缺权限、命令不存在或结果不明确时，**停止自动推进并汇报当前状态**
- 不为了追求“完全一致”而擅自覆盖高价值记忆或本机已有配置

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

## Step 2：同步记忆到 workspace

openclaw 的记忆存放在 `~/.openclaw/workspace/`，该目录本身是一个 git repo。

先检查 remote：
```bash
git -C ~/.openclaw/workspace remote -v
```

### 情况 A：remote 已是 `git@github.com:erichuanp/claw-memory.git`

直接 pull：
```bash
git -C ~/.openclaw/workspace pull origin main
```

如果 pull 发生冲突：
- **不要自动解决**
- 汇报冲突文件列表和当前状态
- 等待用户决定

### 情况 B：workspace 是空目录或没有 remote

先 clone 到临时目录：
```bash
git clone git@github.com:erichuanp/claw-memory.git /tmp/claw-memory-sync
```

然后逐文件合并到 `~/.openclaw/workspace/`，按以下规则处理：

#### 1. 高价值记忆 / 代理身份文件
以下文件如出现同名冲突，**不要自动语义合并**，直接停下汇报：
- `MEMORY.md`
- `SOUL.md`
- `AGENTS.md`
- `USER.md`
- `TOOLS.md`

#### 2. Daily memory
对 `memory/YYYY-MM-DD.md`：
- 若目标不存在：直接复制
- 若目标已存在：允许按条目追加、去重式合并
- 不要重写成新的同日派生文件名

#### 3. skills 目录
对 `~/.openclaw/workspace/skills/`：
- repo 中存在、本机不存在：复制
- 两边都存在且内容不同：保留本机版本，并在最终汇报中列为“需人工确认”
- 本机存在、repo 不存在：保留本机版本

#### 4. 其他普通文件
- 不存在的文件：直接复制
- 同名冲突文件：优先保守处理；无法明确安全合并时，停止并汇报

完成后清理：
```bash
rm -rf /tmp/claw-memory-sync
```

---

## Step 3：检查并安装 Skills

先执行：
```bash
openclaw skill list
```

如果命令不存在、报错或输出不可解析：
- 不要猜测安装状态
- 汇报错误并停止 Step 3

### 恢复优先级

#### 核心 Skills（优先恢复）
| Skill | 说明 |
|-------|------|
| `review` | 结构化 review（文档/代码/方案/决策） |
| `writing-plans` | 多步骤任务实现计划，写代码前先规划 |
| `office-hours` | YC 式产品思维，第一性原理拆解需求 |
| `weather` | 天气查询（无需 API key） |
| `find-skills` | 帮助发现和安装 skills |
| `skill-vetter` | 安装 skill 前的安全审查 |

#### 可选 Skills（缺失时再补）
| Skill | 说明 |
|-------|------|
| `agent-browser-clawdbot` | 无头浏览器自动化 CLI（accessibility tree 快照） |
| `crypto-price` | 加密货币价格与 K 线图（CoinGecko / Hyperliquid） |
| `gog` | GOG 游戏相关工具 |
| `humanizer` | 去除 AI 写作痕迹 |
| `ontology` | 结构化知识图谱，跨 skill 共享状态 |
| `proactive-agent` | 主动型 agent 模式（WAL Protocol） |
| `self-improving-agent` | 捕获错误与修正，持续改进 |
| `superdesign` | 前端设计规范与 UI 指南 |

### 安装规则
对缺失的 ClawHub skill：
1. **先经过 `skill-vetter` 审查**
2. 审查通过后，才允许安装
3. 安装命令：
   ```bash
   npx clawhub install <skill-name>
   ```
4. 如 `clawhub` / `npx` 不可用，停止并汇报，不要自行改用其他来源

### 自研 Skills（随 workspace 记忆自动恢复，无需单独安装）
Step 2 拉取记忆 repo 后，以下 skills 会自动出现在 `~/.openclaw/workspace/skills/` 中，无需额外操作：

`careful` · `cso` · `daily-news` · `model-usage` · `update-food-inventory`

### Bundled Skills（openclaw 内置，无需安装）
| Skill | 说明 |
|-------|------|
| `wacli` | WhatsApp CLI 消息发送 |
| `xurl` | X (Twitter) API v2 操作 |

---

## Step 4：安装自研 openclaw 插件

clone `git@github.com:erichuanp/openclaw-cmds.git`，读取其 README，并**只执行 README 中与本地安装直接相关的步骤**；不要自行扩展配置，不要推断未写明的安装流程。

```bash
git clone git@github.com:erichuanp/openclaw-cmds.git /tmp/openclaw-cmds
cat /tmp/openclaw-cmds/README.md
```

读完 README 后按指南执行安装步骤，完成后清理：

```bash
rm -rf /tmp/openclaw-cmds
```

如果 README 步骤不清晰、依赖缺失或执行失败：
- 停止自动推进
- 汇报失败点与已完成步骤

---

## Step 5：完成确认

告知用户：
1. 记忆同步方式（pull / clone+merge）及有无冲突
2. Skills 安装清单（哪些已有、哪些新装、哪些跳过）
3. 插件安装结果
4. **未恢复项 / 需人工确认项**
5. 是否需要重启 gateway（如有需要，先告知用户再执行）