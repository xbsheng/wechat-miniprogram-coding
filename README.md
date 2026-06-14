# 微信小程序 AI 编码规则

> 让 AI 写出真正能在微信小程序中运行的代码

## 问题

AI 编码助手（Claude、GPT、Gemini 等）的训练数据以 Web/H5 代码为主。当让它写小程序代码时，会频繁生成：

- `document.getElementById` — 小程序没有 DOM
- `fetch` / `axios` — 必须用 `wx.request`
- `localStorage` — 必须用 `wx.setStorageSync`
- `<img>` 标签 — 必须用 `<image>`
- CSS `*` 选择器、`:nth-child` — 不支持
- `background-image: url('/local/path')` — 本地路径无效

**本项目提供一套通用规则，可直接导入任何 AI 编码工具，从源头解决这些问题。**

## 支持的工具

| 工具 | 规则文件 | 放置位置 |
|------|---------|---------|
| **Claude Code** | `CLAUDE.md` | 项目根目录 |
| **Cursor** | `.cursorrules` | 项目根目录 |
| **Windsurf** | `.windsurfrules` | 项目根目录 |
| **GitHub Copilot** | `.github/copilot-instructions.md` | `.github/` 目录 |
| **Cline** | `.clinerules` | 项目根目录 |
| **OpenAI Codex** | `codex.md` | 项目根目录，或通过 `CODEX.md` 引用 |
| **Hermes Agent** | `AGENTS.md` | 项目根目录，或安装为 skill |
| **通用** | `RULES.md` | 项目根目录，人工参考 |

## 快速开始

### 一键导入（推荐）

```bash
# 克隆仓库
git clone https://github.com/xbsheng/wechat-miniprogram-skill.git

# 进入你的小程序项目目录
cd your-miniprogram-project

# 选择你需要的工具，复制对应规则文件
# Claude Code
cp ../wechat-miniprogram-skill/rules/CLAUDE.md .

# Cursor
cp ../wechat-miniprogram-skill/rules/.cursorrules .

# Windsurf
cp ../wechat-miniprogram-skill/rules/.windsurfrules .

# GitHub Copilot
mkdir -p .github
cp ../wechat-miniprogram-skill/rules/copilot-instructions.md .github/

# Cline
cp ../wechat-miniprogram-skill/rules/.clinerules .

# 多工具共存（同时用 Cursor + Claude Code？直接都复制）
cp ../wechat-miniprogram-skill/rules/CLAUDE.md .
cp ../wechat-miniprogram-skill/rules/.cursorrules .
```

### Hermes Agent 安装

```bash
git clone https://github.com/xbsheng/wechat-miniprogram-skill.git \
  ~/.hermes/skills/software-development/wechat-miniprogram-coding
```

## 规则覆盖范围

| 模块 | 说明 |
|------|------|
| **禁止 API 清单** | document / window / fetch / axios / localStorage 等完整列表 |
| **API 映射表** | H5 → 原生(wx) / Taro / uni-app 三方对照 |
| **CSS 约束** | 不支持的选择器、单位、背景图等 + 替代方案 |
| **框架代码模板** | 原生、Taro、uni-app 的正确写法示例 |
| **常见错误 Top 6** | setData vs setState、箭头函数、img vs image 等 |
| **项目配置** | app.json、权限、域名白名单等关键配置 |

## 支持的框架

- **原生小程序** — `.wxml` + `.wxss` + `wx.*` API
- **Taro** — React 语法 + `Taro.*` API
- **uni-app** — Vue 语法 + `uni.*` API

## 贡献

欢迎提交 Issue 和 PR！特别是：

- 微信更新了新 API，规则需要同步更新
- 发现了新的 AI 常犯错误模式
- 支持更多框架（如 Taro 4.x、uni-app 4.x 的新特性）
- 其他语言版本

## 许可证

MIT
