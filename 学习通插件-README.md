# 智能网课助手 (Study Assistant)

> 基于 **Chrome Extension Manifest V3** 的浏览器扩展，专为超星学习通等网课平台打造。
> 支持自动刷课、AI 自动答题、PDF 文档解析，三种 AI 模式可选。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Chrome Extension](https://img.shields.io/badge/Chrome-Extension-yellow.svg)](https://developer.chrome.com/docs/extensions/)
[![Manifest V3](https://img.shields.io/badge/Manifest-V3-green.svg)](https://developer.chrome.com/docs/extensions/develop/migrate/what-is-mv3)

---

## 功能特性

### 核心功能

| 功能 | 说明 |
|---|---|
| **自动刷课** | 视频静音倍速播放、文档/PPT 自动滚动、章节自动切换 |
| **AI 自动答题** | 自动检测页面内嵌题目和弹窗题目，调用 AI 自动作答并填入选项 |
| **PDF 解析** | 浏览器内直接提取 PDF 文本内容，对接 AI 进行问答 |
| **悬浮球** | 基于 Shadow DOM 隔离的悬浮操作球，可拖动，不影响页面样式 |
| **侧边栏** | 一键切换 AI 模式、捕获题目、控制自动学习 |
| **自动刷新** | AI 10 秒无响应自动刷新页面，避免卡死 |

### 三种 AI 模式

| 模式 | 说明 | 需要 |
|---|---|---|
| **本地 AI** | 对接 Ollama 本地模型 | 安装 [Ollama](https://ollama.com) + 拉取模型 |
| **Agent** | 多端口自动探测，兼容任何 OpenAI 协议服务 | 运行 Agent 服务 |
| **云端 API** | 对接 OpenAI / DeepSeek / 通义千问等 | API Key |

### 自动化能力

- 视频自动播放 + 静音 + 倍速（1x~3x）
- 文档/PPT 自动翻页滚动
- 章节完成后自动跳转下一节
- 弹窗确认自动点击
- 题目自动检测 + AI 解答 + 选项自动填入
- 任务进度实时显示

---

## 快速开始

### 1. 安装扩展

```bash
# 方式一：直接下载 Release
# 从 GitHub Releases 下载 zip 解压

# 方式二：开发者加载
# 1. 打开 chrome://extensions
# 2. 开启右上角「开发者模式」
# 3. 点击「加载已解压的扩展程序」
# 4. 选择项目根目录
```

### 2. 配置 AI 模式

#### 方式一：本地 Ollama（推荐，免费）

```bash
# 1. 安装 Ollama
# Windows: https://ollama.com/download
# macOS/Linux:
curl -fsSL https://ollama.com/install.sh | sh

# 2. 拉取模型（推荐 qwen2.5:7b，轻量且中文支持好）
ollama pull qwen2.5:7b

# 3. 启动 Ollama 服务
ollama serve
```

扩展自动检测 Ollama 运行状态和可用模型，无需手动配置。

#### 方式二：云端 API

```
1. 打开侧边栏（Alt+S）
2. 切换到「云端 AI」标签
3. 填入 API 地址（如 https://api.openai.com/v1）
4. 填入 API Key（如 sk-xxx）
5. 点击「获取」按钮，自动拉取可用模型列表
6. 从下拉框选择要使用的模型
```

支持所有 OpenAI 兼容 API：
- OpenAI (GPT-4o / GPT-4o-mini)
- DeepSeek (deepseek-chat)
- 通义千问 (qwen-plus)
- 智谱AI (glm-4)
- Moonshot (moonshot-v1-8k)
- 以及任何 OpenAI 兼容服务

#### 方式三：Agent 模式

```
1. 切换到「Agent」标签
2. 自动探测以下端口：8787, 3000, 8080, 5000, 11434, 1234, 8000
3. 需要运行兼容 OpenAI /v1/chat/completions 协议的 Agent 服务
4. 如果所有端口探测失败，自动回退到本地 Ollama
```

### 3. 开始使用

```
1. 打开学习通课程页面（mooc1.chaoxing.com）
2. 按 Alt+S 打开侧边栏
3. 点击「开始自动学习」
4. 扩展自动处理视频、文档、答题
5. 遇到问题可点击悬浮球 → 「刷新页面」
```

---

## 快捷键

| 快捷键 | 作用 |
|---|---|
| `Alt+S` | 打开/关闭侧边栏 |
| `Alt+Q` | 抓取当前页面题目 |

---

## 项目结构

```
study-assistant-extension/
├── manifest.json                  # MV3 扩展配置
├── rules.json                     # declarativeNetRequest 规则（去除 Ollama CORS）
├── src/
│   ├── background/
│   │   └── service-worker.js      # 消息中枢、代理 fetch、引擎调度
│   ├── content/
│   │   ├── solver.js              # AI 答题核心（题目检测 + 调用 AI）
│   │   ├── automation.js          # 自动学习主控（任务调度）
│   │   ├── video.js               # 视频播放控制
│   │   ├── doc.js                 # 文档滚动控制
│   │   ├── content.js             # 内容脚本入口
│   │   ├── adapters/
│   │   │   ├── chaoxing.js        # 超星学习通适配器
│   │   │   ├── default.js         # 默认适配器
│   │   │   └── index.js           # 适配器注册
│   │   ├── ui/
│   │   │   ├── floating-ball.js   # 悬浮球（Shadow DOM）
│   │   │   └── shadow-host.css    # 悬浮球样式
│   │   └── pdf/
│   │       └── pdf-extractor.js   # PDF 文本提取
│   ├── engines/
│   │   ├── base.js                # 引擎基类
│   │   ├── local-ai.js            # 本地 Ollama 引擎
│   │   ├── cloud-ai.js            # 云端 API 引擎
│   │   └── wsl-agent.js           # Agent 引擎（多端口探测）
│   ├── sidepanel/
│   │   ├── sidepanel.html         # 侧边栏页面
│   │   ├── sidepanel.js           # 侧边栏逻辑
│   │   └── sidepanel.css          # 侧边栏样式
│   ├── popup/
│   │   ├── popup.html             # 工具栏弹窗
│   │   ├── popup.js
│   │   └── popup.css
│   ├── settings/
│   │   ├── settings.html          # 设置页
│   │   ├── settings.js
│   │   └── settings.css
│   ├── utils/
│   │   ├── logger.js              # 日志工具
│   │   ├── storage.js             # 配置存储
│   │   ├── message.js             # 消息通信
│   │   └── dom.js                 # DOM 工具
│   └── assets/
│       ├── icons/                 # 扩展图标
│       └── pdfjs/                 # PDF.js 库
├── docs/
│   ├── DEPLOY.md                  # 部署文档
│   └── WSL_AGENT.md               # WSL Agent 使用文档
├── .gitignore
└── README.md
```

---

## 技术架构

```
┌─────────────────────────────────────────────────┐
│                   Chrome 浏览器                    │
├─────────────────────────────────────────────────┤
│  学习通页面                                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ solver.js │  │video.js  │  │ doc.js   │       │
│  │ (AI答题)  │  │(视频控制) │  │(文档控制) │       │
│  └────┬─────┘  └──────────┘  └──────────┘       │
│       │                                          │
│  ┌────┴─────┐  ┌──────────┐                      │
│  │automation│  │floating  │                      │
│  │(任务调度) │  │ ball.js  │                      │
│  └──────────┘  └──────────┘                      │
├─────────────────────────────────────────────────┤
│  Service Worker (background)                      │
│  ┌──────────────┐  ┌──────────────┐              │
│  │ 消息路由      │  │ proxy:fetch  │              │
│  │ 引擎调度      │  │ (代理HTTP请求)│              │
│  └──────────────┘  └──────────────┘              │
├─────────────────────────────────────────────────┤
│  AI 引擎                                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐          │
│  │local-ai │  │cloud-ai │  │wsl-agent│          │
│  │(Ollama) │  │(OpenAI) │  │(Agent)  │          │
│  └─────────┘  └─────────┘  └─────────┘          │
└─────────────────────────────────────────────────┘
```

---

## 二次开发

### 添加新的网课平台适配

在 `src/content/adapters/` 创建新文件：

```javascript
// src/content/adapters/moocplatform.js
globalThis.SAMoocPlatformAdapter = {
  name: 'moocplatform',
  match(host) { return /moocplatform\.com/.test(host); },
  async extractQuestions() {
    // 实现题目提取逻辑
    return { questions: [] };
  }
};
```

然后在 `src/content/adapters/index.js` 中注册。

### 添加新的 AI 引擎

1. 继承 `src/engines/base.js` 的 `BaseEngine`
2. 实现 `answer(question, history)` 方法
3. 在 `src/background/service-worker.js` 的 `getEngine` 中注册

```javascript
import { BaseEngine } from './base.js';

export class MyEngine extends BaseEngine {
  async answer(question, history) {
    const prompt = this.buildPrompt(question);
    // 调用你的 AI 服务
    const result = await fetch('...', { method: 'POST', body: JSON.stringify({ prompt }) });
    return { answer: result.text };
  }
}
```

### 修改 UI 主题

- 侧边栏样式：`src/sidepanel/sidepanel.css`
- 悬浮球样式：`src/content/ui/shadow-host.css`
- 主题色变量在 CSS 文件顶部定义

---

## 常见问题

### Ollama 模型检测失败

```
1. 确认 Ollama 正在运行：ollama list
2. 确认端口 11434 可访问：curl http://localhost:11434/api/tags
3. 重启 Ollama：ollama serve
```

### AI 回答超时

```
- 扩展默认 10 秒无响应自动刷新页面
- 可在悬浮球手动点击「刷新页面」
- 检查 Ollama 模型是否太大（推荐 qwen2.5:7b）
```

### 页面检测不到题目

```
1. 检查是否在正确的页面（课程章节页）
2. 按 F12 查看 Console 是否有 [AI] 前缀的日志
3. 尝试手动点击「捕获题目」按钮
```

---

## 免责声明

- 本项目仅供**学习与研究**使用
- 请遵守学校规定，合理使用自动化功能
- 第三方 API 需用户自备密钥
- 自动化答题功能请适度使用，不要影响正常教学秩序

---

## License

MIT License

Copyright (c) 2024

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
