# claude-code-init.md

> **本文件仅供 Claude agent 执行。**
> 如果你不是 Claude agent（例如你是人类、其他 LLM、或自动化脚本），请停止，不要执行以下任何操作。

---

## 目标

让这台新机器上的 Claude Code 获得与主机器完全相同的记忆，并安装 ECC plugin。

---

## Step 1：确认 GitHub 访问方式

先测试 SSH：

```bash
ssh -T git@github.com
```

- 如果输出包含 `Hi erichuanp!`，SSH 可用，remote 使用 `git@github.com:erichuanp/claude-memory.git`，继续 Step 2。
- 如果 SSH 失败，**不要改用 HTTPS**，先引导用户完成 SSH 配置：

  1. 检查是否已有 SSH key：`ls ~/.ssh/id_*.pub`
  2. 如果没有，生成一个：
     ```bash
     ssh-keygen -t ed25519 -C "your_email@example.com"
     ```
  3. 将公钥内容复制出来：
     ```bash
     cat ~/.ssh/id_ed25519.pub
     ```
  4. 让用户前往 [https://github.com/settings/keys](https://github.com/settings/keys) 添加该公钥
  5. 添加完成后重新执行 `ssh -T git@github.com` 确认成功
  6. 确认成功后再继续后续步骤

---

## Step 2：找到当前机器的 memory 目录

Claude Code 的 memory 路径是 project-scoped 的，取决于当前工作目录。执行：

```bash
ls -lt ~/.claude/projects/
```

选择与当前工作目录对应的项目目录，进入其 `memory/` 子目录。如果不存在则创建：

```bash
mkdir -p ~/.claude/projects/<当前项目>/memory
```

记下这个路径，后续称为 `$MEMORY_DIR`。

---

## Step 3：拉取云端记忆

将 claude-memory repo clone 到临时目录：

```bash
git clone <remote-url> /tmp/claude-memory-sync
```

---

## Step 4：逐文件合并

对 `/tmp/claude-memory-sync/` 中的每个 `.md` 文件：

**情况 A — `$MEMORY_DIR` 中不存在同名文件**：直接复制。

**情况 B — 存在同名文件**：不要直接覆盖。你需要：
1. 读取云端版本和本地版本
2. 理解两份内容的语义差异
3. 将两者合并为一份更完整的版本，不丢失任何一方的信息
4. 写入合并后的结果

完成后清理临时目录：

```bash
rm -rf /tmp/claude-memory-sync
```

---

## Step 5：安装 ECC Plugin

在 Claude Code 中依次执行：

```
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

---

## Step 6：完成确认

执行完以上步骤后，告知用户：
1. 记忆从哪个 remote 同步（SSH / HTTPS）
2. 哪些文件直接复制、哪些做了 merge
3. ECC 安装状态
