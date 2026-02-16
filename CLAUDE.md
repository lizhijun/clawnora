# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Clawra 是一个通过 npm 分发的 CLI 安装器，为 OpenClaw AI 智能体添加自拍生成能力。它封装了 xAI 的 Grok Imagine 模型（通过 fal.ai），基于固定的参考照片生成图片，并通过 OpenClaw 网关发送到各消息平台（Discord、Telegram、WhatsApp、Slack、Signal、MS Teams）。

## 常用命令

### 运行安装器
```bash
node bin/cli.js
# 或: npx clawra@latest（已发布的包）
```

### 运行图片生成脚本
```bash
# TypeScript
npx ts-node scripts/clawra-selfie.ts "<prompt>" "<channel>" [caption] [aspect_ratio] [output_format]

# Bash
./scripts/clawra-selfie.sh "<prompt>" "<channel>" [caption] [aspect_ratio] [output_format]
```

### 安装开发依赖
```bash
npm install
```

本项目没有构建步骤、测试套件或代码检查工具。

## 架构

### 入口文件

- **`bin/cli.js`** — 主 CLI 安装器（纯 Node.js，无需构建）。运行 7 步交互式向导：检查 OpenClaw 环境 → 收集 fal.ai API 密钥 → 复制技能文件到 `~/.openclaw/skills/clawra-selfie/` → 合并配置到 `~/.openclaw/openclaw.json` → 写入 `IDENTITY.md` → 注入人设到 `SOUL.md`。

- **`scripts/clawra-selfie.ts`** 和 **`scripts/clawra-selfie.sh`** — 独立的图片生成+发送脚本（TypeScript 和 Bash 两种实现）。调用 fal.ai Grok Imagine Edit API 编辑固定参考图，然后通过 OpenClaw CLI 或直接 HTTP 请求本地网关（`localhost:18789`）发送结果。

### 技能模块（`skill/`）

`skill/` 目录是部署到 `~/.openclaw/skills/clawra-selfie/` 的内容，包含：
- `SKILL.md` — OpenClaw 技能清单（frontmatter 定义允许的工具、描述、名称）
- `scripts/` — 生成脚本的副本
- `assets/clawra.png` — 参考图片

### 核心数据流

```
用户请求 → 构建提示词（mirror 或 direct 模式） → fal.ai Grok Imagine Edit API → 图片 URL → OpenClaw 消息发送 → 消息平台
```

两种自拍模式：**mirror**（全身/穿搭照）和 **direct**（特写/场景照）。模式根据用户提示词中的关键词自动检测。

### 外部依赖

- **fal.ai API**（`https://fal.run/xai/grok-imagine-image/edit`）— 图片生成
- **OpenClaw Gateway**（`http://localhost:18789/message`）— 消息路由到各平台
- **jsDelivr CDN** — 托管固定参考图片

### 环境变量

- `FAL_KEY` — fal.ai API 密钥（必需）
- `OPENCLAW_GATEWAY_TOKEN` — OpenClaw 网关认证令牌（用于直接 API 调用）
- `OPENCLAW_GATEWAY_URL` — 网关地址（默认：`http://localhost:18789`）

## 关键路径（安装时）

- `~/.openclaw/openclaw.json` — OpenClaw 配置（技能注册）
- `~/.openclaw/skills/clawra-selfie/` — 技能安装位置
- `~/.openclaw/workspace/SOUL.md` — 智能体人设文件
- `~/.openclaw/workspace/IDENTITY.md` — 智能体身份文件
