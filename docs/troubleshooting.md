# 排错

## 1. `/council` 不出现

| 检查 | 命令 | 处理 |
|---|---|---|
| 文件在不在 | `ls ~/.config/opencode/commands/council.md` | 放对目录 |
| 文件名 | 必须是 `council.md` | 改名 |
| 目录名 | 必须是 `commands/` | 不是 `command/` |
| 重启 | `opencode service restart` | 或重开 |
| 语法 | 看 frontmatter 是否闭合 | `---` 成对 |

Windows：

```powershell
Test-Path "$env:USERPROFILE\.config\opencode\commands\council.md"
```

## 2. 命令出现，但没调用子代理

| 原因 | 处理 |
|---|---|
| agent 文件没放对 | 放到 `agents/` |
| 文件名不是 `council-*` | 保持一致 |
| 子代理名和命令里不一致 | 检查 `council-architect` 等 |
| 权限/模式不对 | `mode: subagent` |
| 模型报错 | 注释掉 `model` 先继承 |

```bash
ls ~/.config/opencode/agents/council-*.md
```

## 3. 模型不存在

现象：

```text
model not found / provider not found
```

处理：

```bash
opencode models
```

把 frontmatter 的 `model:` 改成真实 ID，或整行注释掉。

> 不要猜 `provider/model`。不同环境 ID 不一样。

## 4. 五席答案几乎一样

原因：

- 都继承同一个模型；
- prompt 太泛；
- 上下文包没给约束。

处理：

- 给反方/架构师配不同厂商；
- 在 prompt 里强调各自检查清单；
- 加“必须给出至少 1 条别人可能不同意的点”。

## 5. 互评走形式

现象：

- 全是“都很好”；
- 排名和内容无关；
- 没有硬伤。

处理：

- 换更强模型做评审；
- 缩短上下文；
- 明确“必须逐份指出一个硬伤”；
- 要求排名理由一句话。

## 6. 某席超时/502

正常。按阶段 5：

1. 重试 1 次；
2. 换备用模型 1 次；
3. 仍失败 → 标记掉线，继续；
4. 有效席位 <3 → 停止并报告。

不要用 2 个模型硬凑“共识”。

## 7. 费用超预期

| 原因 | 处理 |
|---|---|
| 上下文太长 | 裁剪到 ≤1200 字 |
| 开了 `wide` | 先 `quick` |
| 每步都全席评审 | 只抽 3 个节点 |
| 模型太贵 | 互评/验收换便宜模型 |
| 反复重试 | 修上游/换模型 |

## 8. 输出没有表格

原因：

- 模型没按模板；
- 最终汇报指令被长上下文淹没。

处理：

- 在 command 末尾再强调一次格式；
- 换指令跟随更强的模型；
- 减少上下文。

## 9. 存档目录没有文件

| 原因 | 处理 |
|---|---|
| 路径不可写 | 设 `COUNCIL_DIR` 到可写目录 |
| 子代理没写权限 | 席位默认 deny edit；由主会话落盘 |
| 目录没建 | 阶段 0 先 `mkdir -p` |
| 被 gitignore | 正常，存档本来就不进 Git |

## 10. 子代理改了文件

席位 agent 的 `permissions` 应该：

```yaml
- action: edit
  resource: "*"
  effect: deny
- action: shell
  resource: "*"
  effect: deny
```

如果被改了：

1. `git diff` 看改动；
2. 恢复不想要的；
3. 检查 agent frontmatter；
4. 重启 OpenCode。

> `exec-verifier` 允许 shell（要重跑命令），但不允许 edit。

## 11. 命令很慢

- 5 席并行本来就要 60–90s；
- 模型排队/上游慢会更久；
- 先 `quick`；
- 减少上下文；
- 换低延迟模型。

## 12. 中文乱码

- 文件用 UTF-8；
- Windows PowerShell 5.1 读 UTF-8 无 BOM 可能乱码；
- 用 VS Code 确认编码；
- 必要时加 BOM（但 Markdown 一般不需要）。

## 13. 最小诊断包

```text
1. opencode --version
2. ls ~/.config/opencode/commands/council.md
3. ls ~/.config/opencode/agents/council-*.md
4. opencode models
5. /council quick test 的实际输出
6. 最近改了什么
```

> 贴日志前删掉 key、token、私有域名、用户数据。

## 14. 回滚

```bash
rm -f ~/.config/opencode/commands/council.md
rm -f ~/.config/opencode/agents/council-*.md ~/.config/opencode/agents/exec-verifier.md
# 从备份恢复
cp ~/.config/opencode/backups/... ~/.config/opencode/commands/
opencode service restart
```