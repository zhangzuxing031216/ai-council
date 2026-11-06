# 自定义

## 1. 改席位模型

编辑 `agents/council-*.md` 顶部：

```yaml
---
description: ...
mode: subagent
model: your-provider/your-model
permissions:
  - action: edit
    resource: "*"
    effect: deny
  - action: shell
    resource: "*"
    effect: deny
  - action: subagent
    resource: "*"
    effect: deny
---
```

`provider/model` 从 `opencode models` 查。不要填不存在的 ID。

## 2. 改权重

`command/council.md` 阶段 1 的表格：

```text
| 席位 | 子代理 | 领域 | 权重 |
| 架构师 | council-architect | ... | 1.3 |
```

权重影响：

- 互评 Borda；
- 加权均值。

建议：

- 架构师/反方略高；
- 视觉/学者略低；
- 不要差 3 倍以上。

## 3. 加一个席位

### 3.1 复制 agent

```bash
cp agents/council-critic.md agents/council-security.md
```

改：

- `description`
- 角色说明
- 输出规则

### 3.2 加到命令

在 `command/council.md` 阶段 1 表格加一行：

```text
| 安全 | council-security | 攻击面、权限、数据泄露 | 1.1 |
```

在“阶段 1”的 prompt 要求里加上安全关注点。

### 3.3 加到降级表

在阶段 5 的备用模型表加一行。

### 3.4 加到最终汇报

最终表格行数会自动增加，不需要改模板，但席位名要一致。

## 4. 改存档目录

默认：`${COUNCIL_DIR:-.council}/<时间戳>/`

改法：

- 设置环境变量 `COUNCIL_DIR`；或
- 在 `command/council.md` 里全局替换 `.council`。

建议：

- 放在工作区外（避免提交）；
- 或加进 `.gitignore`；
- 定期清理旧存档。

## 5. 改输出长度

搜索：

- `≤400 字`：提案；
- `≤300 字`：互评；
- `≤500 字`：最终汇报。

长材料任务可放宽到 600–800 字，但会增加成本和互评难度。

## 6. 改成“只讨论不执行”

删掉 `command/council.md` 的阶段 4、5，保留：

- 阶段 0 备料；
- 阶段 1 提案；
- 阶段 2 互评；
- 阶段 3 收敛。

最终只输出方案，不写文件、不跑命令。

## 7. 改成“只 quick”

把默认模式改成 2 席，或每次输入都带 `quick`。

适合：

- 日常小决策；
- 预算有限；
- 只需要架构+工程两视角。

## 8. 改验收

`agents/exec-verifier.md`：

- 加项目特定命令；
- 要求必须贴退出码；
- 要求检查 `git diff`；
- 换更严格/更便宜的模型。

## 9. 改匿名规则

默认匿名。如果你想“公开署名”，删掉阶段 2 的匿名编号，直接贴席位名。

代价：

- 可能偏袒强模型；
- 互评更容易客气。

一般不建议。

## 10. 加项目上下文

在阶段 0 的上下文包里加：

```text
项目规则：<AGENTS.md 摘要>
技术栈：
不能动的文件：
验收命令：
```

不要贴整个仓库。只贴相关文件/符号/报错。

## 11. 多语言

席位 prompt 是中文。要英文输出：

- 在每个 agent 的输出规则加 `Respond in English.`
- 最终汇报模板换成英文；
- 互评模板也一样。

## 12. 版本管理

本仓库是模板，你的本地配置才是“生产”。

建议：

- 本地改完后，把 diff 记到自己的笔记；
- 升级模板时先对比 `command/council.md`；
- 不要直接覆盖你改过的权重/模型。

## 13. 常见自定义清单

```text
[ ] 五个席位都配了模型
[ ] 反方用了不同厂商
[ ] 权重符合团队偏好
[ ] 存档目录可写且不进 Git
[ ] 验收命令是项目真实命令
[ ] 输出长度适合任务
[ ] 费用在可接受范围
```