# 更新记录 / Changelog

版本号遵循 SemVer；当前版本见 `VERSION`，发布和对比流程见 `VERSIONING.md`。

## v0.1.0 · 2026-11-06 · 首次发布

### 新增

- 多模型 AI 议会规则：独立提案 → 匿名互评 → 加权收敛 → 执行 → 独立验收。
- 中英双语人看文档：`README.md` / `README.en.md`，顶部可切换。
- 中英双语 AI 安装协议：`AI-INSTALL.md` / `AI-INSTALL.en.md`。
- 命令：`command/council.md`，支持 `quick` / `wide` / `show`。
- 席位：
  - `agents/council-architect.md`
  - `agents/council-coder.md`
  - `agents/council-critic.md`
  - `agents/council-vision.md`
  - `agents/council-scholar.md`
  - `agents/exec-verifier.md`
- 文档：
  - `docs/how-it-works.md`
  - `docs/customize.md`
  - `docs/troubleshooting.md`
- 图片：`docs/images/council-flow.svg`、`docs/images/council-output.svg`。

### 说明

- 这是第一个公开版本，作为后续版本对比的基线。
- 与下一版本比较：`git diff v0.1.0..v0.2.0`。
- 已知限制：报告图为合成示意，没有固定 UI；真实多模型运行的截图待补。
- 模型名均为占位/注释，不绑定私有 provider。

## 未发布 / Unreleased

- [ ] 补真实运行截图（命令、席位输出、最终表）。
- [ ] 补更多席位模板（安全、法务、数据）。
- [ ] 补“只讨论不执行”的独立命令。
- [ ] 补成本估算脚本。
- [ ] 定期检查 OpenCode frontmatter 字段变化。