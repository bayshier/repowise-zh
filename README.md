<div align="center">

# Repowise 中文指南

**别再让 AI 花你的 token 重新发现你的代码库。**
索引一次：代码结构、调用图、Git 历史、测试、文档——然后让 Claude Code / Codex / Cursor 直接问索引。

[![原项目](https://img.shields.io/badge/原项目-repowise--dev/repowise-3DDC84?style=flat-square)](https://github.com/repowise-dev/repowise)
[![Version](https://img.shields.io/badge/v0.56.0-实测通过-blue?style=flat-square)](#实测记录)
[![License](https://img.shields.io/badge/原项目%20许可-AGPL--3.0-059669?style=flat-square)](LICENSE-original)
[![Star](https://img.shields.io/badge/7.6k%2B%20stars-2026.10%20实查-ffdd44?style=flat-square)](https://github.com/repowise-dev/repowise/stargazers)

</div>

---

> 📌 **本仓库是什么**：[repowise-dev/repowise](https://github.com/repowise-dev/repowise)（7.6k★，AGPL-3.0）的**中文上手指南与实测记录**。不含源码翻译——AGPL 软件的翻译文档作为独立文档仓库发布，源码请用官方渠道（`uv tool install repowise`）。原版英文 README 副本见 [`docs/`](docs/)。

## 一句话

平时让 AI 改一处老代码，它得先满仓库 grep、读一堆无关文件。Repowise 在**你本机**把代码结构、调用图、Git 历史、测试、文档提前索引好，再通过 **MCP** 暴露给 AI——agent 要什么直接问索引，不再瞎翻。官方在 Django 的 43 个 issue 上配对测试：**同样的活，输出量少 31.6%**。

对中文开发者最友好的三点：

- **图、风险、健康度、死代码全部是确定性计算，零 LLM 调用、零 API key**——不用配任何 key 就能用
- 本机索引，代码不出门
- 装好自动给你生成 Claude Code 的 MCP 配置

## 三分钟上手

```bash
# 1. 安装（需要 Python 3.11+；uv 是官方推荐）
uv tool install repowise        # 或 pipx install repowise

# 2. 在任意仓库根目录建索引（图 + git + 健康 + 死代码，无 key）
cd /path/to/你的仓库
repowise init --yes --no-prose

# 3. 起本地服务（仪表盘 + MCP server）
repowise serve
```

`init` 会**自动写好 Claude Code 的 MCP 配置**（`.repowise/mcp.json`），重启 Claude Code 即可问：
*“用 Repowise get_overview 总结这个仓库”*、*“改 src/auth.py 会影响什么？”*

## 四个马上能用的命令

| 命令 | 作用 | 实测输出 |
|---|---|---|
| `repowise health` | 53 检测器的代码健康分（1-10）+ 热点文件 + 风险分布 | 见下方实测 |
| `repowise dead-code` | 死代码/不可达文件（带置信度） | 抓到一个 40% 置信的误报（Kotlin 入口类），保守合理 |
| `repowise ask "问题"` | 带引用的仓库问答 | 索引含文档时效果最佳 |
| `repowise context 文件名` | 单文件的“分诊卡”（改它影响谁） | 改老代码前先看这张卡 |

其他值得知道的：`repowise update`（提交后刷新索引）、`repowise doc-drift`（文档漂移）、`repowise export`（导出 wiki）。

## 实测记录（2026-08-24，v0.56.0，本机 Apple Silicon）

### 案例 1：t1-replay（纯 Kotlin，7 文件）

```
Code health: 9.97/10 [Excellent] · Hotspot: 10.0/10 · Worst: 10.0/10
100% excellent (7 files) · 性能风险 0 · 死代码 0
```

逐文件评分表直接给出（TradingEngine.kt 圈复杂度 7、嵌套 2，均健康）。**索引 + 两项检查全程未联网、零 key。**

### 案例 2：kline-vision（Kotlin + 前端资源，8 文件）

```
Code health: 9.82/10 · Worst: 9.12/10 (ChartRenderer.kt —— 确实是最复杂的文件，Java2D 手绘渲染器)
死代码: 1 findings —— VisionTools.kt 标记 unreachable_file (40% 置信)
```

那条死代码是**保守的误报**：`VisionTools.kt` 是 MCP 工具入口，经注册调用，静态图到不了它。40% 低置信 + "Ready? ✗" 标记说明工具自己也没把握——判断权留给人，这个设计是对的。

### 结论：对我们的开发工作有没有用？

**有，且已经用上**：

1. **给 Claude Code 接上后**，改 kline 系列仓库时 agent 先问索引（`get_overview`/`context`），省掉开场那一轮“我先 grep 一下”
2. **健康分当 CI 信号**：`repowise health` 零依赖可进 pre-commit/CI，分数掉了就是代码在腐化
3. **改老代码前看分诊卡**：`context` 告诉你谁调用它、历史 churn——对“半年后回来改自己仓库”的场景正中要害

**注意**：Kotlin 调用图精度低于 Go/TS（官方基准 0.976-0.995 是 Go/TS；Kotlin 走通用分析，见上面 40% 误报）。当作**辅助信号**而非真理。

## 与 Claude Code / Cursor 集成

`repowise init` 已自动生成配置。手动配置（Cursor 等）：

```json
{
  "mcpServers": {
    "repowise": {
      "command": "repowise",
      "args": ["mcp", "/绝对路径/你的仓库", "--transport", "stdio"]
    }
  }
}
```

## 许可证与合规

- 原项目 **AGPL-3.0**（或商业授权）。本仓库是**独立文档作品**（中文指南与实测记录），未分发其源码；原版 README/ROADMAP 副本随库保留并同样以 AGPL-3.0 提供（见 [`docs/`](docs/) 与 [LICENSE-original](LICENSE-original)）
- AGPL 对使用方无影响（本地自用），只在“分发修改版服务”时触发开源义务——企业商用前读一下官方 commercial 条款

## 致谢

- [repowise-dev/repowise](https://github.com/repowise-dev/repowise) — 原项目及全部能力

## 相关（我的 Kotlin Agent 工具链）

[harness-engineering](https://github.com/bayshier/harness-engineering) · [kline-mcp](https://github.com/bayshier/kline-mcp) · [kline-vision](https://github.com/bayshier/kline-vision) · [t1-replay](https://github.com/bayshier/t1-replay) · [android-mcp](https://github.com/bayshier/android-mcp) · [awesome-agent-harnesses](https://github.com/bayshier/awesome-agent-harnesses)
