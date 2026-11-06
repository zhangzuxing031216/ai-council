# AI-INSTALL.md · 给 AI 的安装协议

> **Language / 语言**: 中文 | [English](AI-INSTALL.en.md)

> 目标：把本仓库的 `/council` 命令和 5+1 个 agent 安装到用户的 OpenCode 配置里，并验证能跑。
> 适用：Windows / Linux / macOS。
> 原则：先取证 → 备份 → 复制 → 配模型 → 验证 → 交付；不覆盖用户已有同名文件而不备份。

---

## 0. 你的角色

你是安装执行者。必须：

1. **先取证**：OpenCode 版本、配置目录、已有文件、可用模型。
2. **一次一步**：执行 → 贴原始输出 → 判定 → 下一步。
3. **不编造**：没跑的命令不说成功；没看到的输出不说正常。
4. **不覆盖**：同名文件先备份。
5. **不猜模型**：`provider/model` 必须来自 `opencode models` 或用户明确输入。
6. **不碰隐私**：不把 key、token、用户数据写进仓库/日志。
7. **可回滚**：每一步留下恢复方法。

---

## 1. 开工前问清 3 件事

能从系统查到就不问。

1. OpenCode 配置目录在哪？默认：
   - Linux/macOS：`~/.config/opencode/`
   - Windows：`%USERPROFILE%\.config\opencode\`
2. 用户有哪些可用模型？跑 `opencode models`，或让用户给 `provider/model`。
3. 是否已有 `commands/council.md` / `agents/council-*.md`？有就先备份。

> 如果用户说“随便配”，先用当前默认模型跑通 `quick`，再建议配多模型；不要瞎填不存在的模型 ID。

---

## 2. 阶段 0 · 取证

### 2.1 版本与目录

```bash
opencode --version
```

Windows PowerShell：

```powershell
opencode --version
$dst = "$env:USERPROFILE\.config\opencode"
Test-Path $dst
Get-ChildItem "$dst\commands","$dst\agents" -ErrorAction SilentlyContinue
```

Linux/macOS：

```bash
ls -la ~/.config/opencode/
ls -la ~/.config/opencode/commands/ 2>/dev/null
ls -la ~/.config/opencode/agents/ 2>/dev/null
```

### 2.2 模型

```bash
opencode models
```

记录用户想用的模型 ID。**不要**用本仓库 README 里的示例名，它们不是真实模型。

### 2.3 已有文件

```bash
# Linux/macOS
test -f ~/.config/opencode/commands/council.md && echo EXISTS || echo MISSING
ls ~/.config/opencode/agents/council-*.md 2>/dev/null || true
```

```powershell
# Windows
Test-Path "$dst\commands\council.md"
Get-ChildItem "$dst\agents\council-*.md" -ErrorAction SilentlyContinue
```

### 2.4 判定

| 结果 | 分支 |
|---|---|
| 没有配置目录 | 创建 |
| 已有同名文件 | 备份后覆盖 |
| 没有 `opencode` 命令 | 先装 OpenCode，停止 |
| 没有可用模型 | 先配 provider，停止 |
| 已有旧版 council | 备份，保留用户自定义 |

---

## 3. 阶段 1 · 备份

```bash
STAMP=$(date +%F-%H%M%S)
mkdir -p ~/.config/opencode/backups
cp ~/.config/opencode/commands/council.md \
   ~/.config/opencode/backups/council.md.$STAMP 2>/dev/null || true
cp ~/.config/opencode/agents/council-*.md \
   ~/.config/opencode/backups/ 2>/dev/null || true
```

Windows：

```powershell
$stamp = Get-Date -Format 'yyyyMMdd-HHmmss'
$bak = "$dst\backups\$stamp"
New-Item -ItemType Directory -Force $bak | Out-Null
Copy-Item "$dst\commands\council.md" $bak -ErrorAction SilentlyContinue
Copy-Item "$dst\agents\council-*.md" $bak -ErrorAction SilentlyContinue
```

> 备份目录不要提交到 Git。

---

## 4. 阶段 2 · 复制文件

### 4.1 Linux/macOS

```bash
mkdir -p ~/.config/opencode/commands ~/.config/opencode/agents
cp command/council.md ~/.config/opencode/commands/council.md
cp agents/council-architect.md \
   agents/council-coder.md \
   agents/council-critic.md \
   agents/council-vision.md \
   agents/council-scholar.md \
   agents/exec-verifier.md \
   ~/.config/opencode/agents/
```

### 4.2 Windows PowerShell

```powershell
$dst = "$env:USERPROFILE\.config\opencode"
New-Item -ItemType Directory -Force "$dst\commands","$dst\agents" | Out-Null
Copy-Item .\command\council.md "$dst\commands\council.md" -Force
Copy-Item .\agents\council-architect.md,
          .\agents\council-coder.md,
          .\agents\council-critic.md,
          .\agents\council-vision.md,
          .\agents\council-scholar.md,
          .\agents\exec-verifier.md `
          "$dst\agents\" -Force
```

### 4.3 验证文件

```bash
ls -l ~/.config/opencode/commands/council.md
ls -l ~/.config/opencode/agents/council-*.md
```

期望 1 个 command + 5 个 council agent + 1 个 verifier。

---

## 5. 阶段 3 · 配模型

### 5.1 先跑通，不配也能用

agent frontmatter 里 `model` 是注释状态，会继承当前模型。先验证链路，再配多模型。

### 5.2 配一个席位

编辑 `~/.config/opencode/agents/council-architect.md`：

```yaml
---
description: ...
mode: subagent
model: your-provider/your-model
permissions:
  ...
---
```

把 `your-provider/your-model` 换成 `opencode models` 里的真实 ID。

### 5.3 配全部

| 席位 | 文件 | 建议 |
|---|---|---|
| 架构师 | `council-architect.md` | 强推理/长上下文 |
| 工程师 | `council-coder.md` | 强编码 |
| 反方 | `council-critic.md` | 不同厂商 |
| 视觉 | `council-vision.md` | 多模态 |
| 学者 | `council-scholar.md` | 中文/长上下文 |
| 验收 | `exec-verifier.md` | 严格、便宜、稳定 |

### 5.4 不要做的事

- 不要填不存在的模型 ID。
- 不要把 key 写进 agent 文件。
- 不要让所有席位同一个模型还期待多视角。
- 不要给“便宜模型”配专业脚本任务。

### 5.5 模型 ID 从哪来

```bash
opencode models
```

如果用户说“用我现在的默认模型”，就保持注释，不填。

---

## 6. 阶段 4 · 重启与加载

```bash
opencode service restart
```

Windows 如果没有 `service` 子命令，退出并重新打开 OpenCode。

验证命令列表里出现 `/council`：

```text
/council quick test
```

如果命令不出现：

1. 文件名必须是 `council.md`。
2. 目录必须是 `commands/`，不是 `command/`。
3. 重启后再试。
4. 看 OpenCode 日志/配置语法。

---

## 7. 阶段 5 · 冒烟测试

### 7.1 quick

```text
/council quick 用一句话说明这个命令是否正常工作
```

期望：

- 出现子代理调用；
- 最终有“席位 / 核心主张”表；
- 有存档路径（或明确说未落盘）。

### 7.2 默认

```text
/council 这个项目应该先做哪三件事？给出优先级和风险。
```

期望：

- 多席提案；
- 匿名互评；
- 加权结论；
- 分歧/风险/验收。

### 7.3 带图（可选）

```text
/council 看这张图，指出配色和层级问题。[图片]
```

期望视觉席位读图；无图时它拆需求。

### 7.4 失败分支

| 现象 | 处理 |
|---|---|
| 没有 `/council` | 检查路径/重启 |
| 有命令但没子代理 | 检查 agents 文件 |
| 模型不存在 | 改 frontmatter 或注释掉 |
| 全部超时 | 换模型/减少席位/先 quick |
| 输出没有表格 | 在 command 里强调模板；或换更强模型 |
| 子代理改文件 | 检查 `permissions` 是否为 deny |
| 存档失败 | 改 `COUNCIL_DIR` 到可写目录 |

---

## 8. 阶段 6 · 验收

### 8.1 文件

```bash
test -f ~/.config/opencode/commands/council.md && echo COMMAND_OK
for f in architect coder critic vision scholar; do
  test -f ~/.config/opencode/agents/council-$f.md && echo "AGENT_OK $f"
done
test -f ~/.config/opencode/agents/exec-verifier.md && echo VERIFIER_OK
```

### 8.2 配置

- `model` 行要么不存在，要么是真实 ID。
- `mode: subagent` 存在。
- `permissions` 没放开 edit/shell（席位只读）。
- command frontmatter 没有不存在的 model。

### 8.3 运行

```text
/council quick 输出一行 OK，并说明你用了哪个席位。
```

### 8.4 交付报告

```text
【结论】成功 / 部分成功 / 失败

【环境】
- OS：
- OpenCode 版本：
- 配置目录：

【安装】
- command：路径
- agents：5 个 + verifier
- 备份：路径

【模型】
- 架构师：
- 工程师：
- 反方：
- 视觉：
- 学者：
- 验收：

【验证】
- /council quick：通过/失败
- 默认模式：通过/失败
- 存档：路径

【未解决/风险】
- ...

【回滚】
- 恢复备份命令
```

---

## 9. 回滚

```bash
# 删除新文件
rm -f ~/.config/opencode/commands/council.md
rm -f ~/.config/opencode/agents/council-architect.md \
      ~/.config/opencode/agents/council-coder.md \
      ~/.config/opencode/agents/council-critic.md \
      ~/.config/opencode/agents/council-vision.md \
      ~/.config/opencode/agents/council-scholar.md \
      ~/.config/opencode/agents/exec-verifier.md

# 从备份恢复旧文件
cp ~/.config/opencode/backups/council.md.<stamp> ~/.config/opencode/commands/council.md
```

Windows：

```powershell
Remove-Item "$dst\commands\council.md" -Force
Remove-Item "$dst\agents\council-*.md" -Force
Copy-Item "$bak\*" "$dst\commands\" -Force -ErrorAction SilentlyContinue
```

> 回滚后重启 OpenCode。

---

## 10. 禁止事项

1. 不覆盖用户已有 agent 而不备份。
2. 不填不存在的模型 ID。
3. 不把 API key 写入 agent/command/README。
4. 不把用户私有 provider 名、内部路径带进公开仓库。
5. 不承诺“议会一定给出正确答案”。
6. 不在未验证时报告“安装成功”。
7. 不帮用户用议会做违法/有害决策。

---

## 11. 快速命令卡

```bash
# 取证
opencode --version
opencode models
ls ~/.config/opencode/commands ~/.config/opencode/agents

# 安装
cp command/council.md ~/.config/opencode/commands/
cp agents/council-*.md agents/exec-verifier.md ~/.config/opencode/agents/

# 重启
opencode service restart

# 验证
/council quick test
```