# Claude Gemini Router

> **[English](README.md)** | **[繁體中文](README_zh-TW.md)**

一个会自进化的 Claude Code 插件，根据任务特征智能地将任务路由到 Google Gemini CLI，为大型上下文操作节省 Claude token。

**用越多次越聪明** — 系统会记住你的偏好，自动进化决策逻辑。

## 功能特色

- **智能任务评分** — 自动评估任务更适合 Gemini（大文件分析、摘要）还是 Claude（代码编辑、迭代工作流程）
- **偏好学习** — 通过经验日志记住你的选择，自动路由熟悉的任务类型，无需再问
- **不满意侦测** — 监测负面反馈，学会避免将类似任务路由到 Gemini
- **自进化** — 实现与 AutoSkill 研究相同的经验驱动终身学习（ELL）原则

---

## 自进化学习回圈

```
用户请求 → 任务评分 → 经验库比对
                                ↓
                      ┌─ AUTO_APPROVE → 直接委派 Gemini
                      ├─ AVOID → Claude 自己处理
                      └─ 未知 → 询问用户
                                          ↓
                                ┌─ 记住并使用 → 写入 AUTO_APPROVE
                                ├─ 只用一次 → 不记录
                                └─ Claude 处理 → 不记录
                                          ↓
                      用户不满意？ → 写入 AVOID，未来自动回避
```

### 为什么说是「自进化」？

这个 Skill 实现了与 **AutoSkill**（Yang et al., 2026, arXiv:2603.01145）研究中提出的 **Experience-driven Lifelong Learning (ELL)** 相同的核心理念：

| AutoSkill 论文概念 | Gemini Router 实现 |
|-------------------|-------------------|
| **经验侦测** — 监控交互中的稳定偏好 | 评分机制侦测任务特征（文件大小、读/写、复杂度） |
| **模式萃取** — 从反馈中提取可复用技能 | 用户选「记住」时，萃取任务模式写入 `AUTO_APPROVE` |
| **版本演化** — 技能随反馈持续更新 | `AVOID` 可覆盖 `AUTO_APPROVE`，决策逻辑持续修正 |
| **技能应用** — 未来任务自动套用已学习的技能 | 匹配到 `AUTO_APPROVE` 的任务直接委派，零确认 |

**关键共通点：**
- **人类监督** — 两者都在记录前征求用户同意，不会自作主张
- **双向学习** — 不只记住成功（AUTO_APPROVE），也记住失败（AVOID）
- **持久化记忆** — 经验跨对话保存，不会用完即忘
- **渐进式进化** — 随着使用次数增加，系统决策越来越精准

---

## 前置需求

- [Claude Code](https://claude.ai/code) 已安装并完成认证
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) 已安装（`npm install -g @google/gemini-cli`）

## 安装

### 步骤一：添加 marketplace

在 Claude Code 中执行：

```
/plugin marketplace add audi0417/claude-gemini-router
```

### 步骤二：安装插件

```
/plugin install gemini-router@claude-gemini-router
```

或使用交互式插件管理器：

```
/plugin
```

然后切换到 **Discover** 标签页，选择 **gemini-router**。

### 替代方案：手动安装

直接复制 skill 文件：

```bash
git clone https://github.com/audi0417/claude-gemini-router.git
cp -r claude-gemini-router/plugins/gemini-router/skills/gemini ~/.claude/skills/gemini
```

## 运作方式

1. **评估** — 当你给 Claude 一个任务时，skill 会自动为其评分，判断是否适合 Gemini
2. **决定** — 高分任务会提示你选择：委派给 Gemini，或留给 Claude
3. **学习** — 你的选择会被记录，类似任务下次自动路由
4. **适应** — 如果 Gemini 的输出不满意，skill 会学会避免该类任务

### 评分标准

| 信号 | 权重 |
|------|------|
| 大文件（总计 >50KB） | +2 |
| 只读／分析任务 | +2 |
| 匹配 AUTO_APPROVE 模式 | +2 |
| 用户明确要求使用 Gemini | +2 |
| 多文件交叉比对 | +1 |
| 摘要／提取任务 | +1 |
| 节省 token 效益明显 | +1 |

### 排除条件（一律由 Claude 处理）

- 匹配经验库中的 AVOID 模式
- 需要写入或编辑文件
- 需要多轮工具交互
- 简短、简单的任务

### 经验库（experience.md）

所有学习成果都存在 `experience.md` 中，格式透明、人类可读：

```markdown
### [AUTO_APPROVE] 2026-04-08
**Pattern**: Full Go codebase read-only analysis
**Trigger keywords**: codebase, analysis, review
**Reason**: Large file volume, read-only, Gemini 1M context is a perfect fit

### [AVOID] 2026-04-08
**Pattern**: File writing, editing, or code generation
**What went wrong**: Gemini headless is single-shot, cannot iterate on edits
**Lesson**: Any task requiring Write/Edit tool use stays with Claude
```

---

## 使用示例

### 自动委派（已学习的模式）
```
你：帮我分析这 20 个文件的架构
Claude：Routing to Gemini (auto-approved: large codebase analysis)
[Gemini Response] ...
```

### 首次遇到（询问偏好）
```
你：帮我读这个 200KB 的日志找 error
Claude：这个任务看起来适合 Gemini（大量上下文）。要怎么处理？
  1. 用 Gemini 并记住这类任务
  2. 用 Gemini，仅此一次
  3. Claude 自己处理
```

### 不满意反馈
```
你：Gemini 的分析不太对，你重做一次
Claude：要不要把这类任务标记为不适合 Gemini？
  1. 好，记录下来
  2. 不用，只是这次
```

---

## 授权

[MIT](LICENSE)
