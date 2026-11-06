# 更新记录

## 2026-11-06 · 中英双语

- 新增 `README.en.md`：完整英文说明，不是摘要；`README.md` 顶部可切换。
- 新增 `AI-INSTALL.en.md`：英文 AI 安装协议；`AI-INSTALL.md` 顶部可切换。
- 项目名改为中英双语：README 标题与首屏名称行同时出现“AI 议会 / AI Council”。
- 英文版保留全部命令、路径、席位配置和边界警告。

## 2026-11-06 · 预览图

- 新增 `docs/images/council-output.svg`：最终报告结构示意。
- 新增 `docs/images/README.md`。
- `README.md` 第 8 节增加报告预览。
- `README.md` 首屏增加徽章、项目简介、适合谁、新手 3 步。

## 2026-11-06 · v0.1

- 初版公开模板。
- 命令：
  - `command/council.md`：多席提案 → 匿名互评 → 加权收敛 → 执行 → 独立验收。
  - 支持 `quick` / `wide` / `show`。
  - 模型名全部改为占位/注释，不绑定私有 provider。
- 席位：
  - `agents/council-architect.md`
  - `agents/council-coder.md`
  - `agents/council-critic.md`
  - `agents/council-vision.md`
  - `agents/council-scholar.md`
  - `agents/exec-verifier.md`
- 文档：
  - `README.md`：给人看，安装/配置/使用/成本/边界。
  - `AI-INSTALL.md`：给 AI 看，取证/备份/复制/验证/回滚。
  - `docs/how-it-works.md`
  - `docs/customize.md`
  - `docs/troubleshooting.md`
- 图示：
  - `docs/images/council-flow.svg`

## 待办

- [ ] 补真实运行截图（命令、席位输出、最终表）。
- [ ] 补英文版。
- [ ] 补更多席位模板（安全、法务、数据）。
- [ ] 补“只讨论不执行”的独立命令。
- [ ] 补成本估算脚本。
- [ ] 定期检查 OpenCode frontmatter 字段变化。