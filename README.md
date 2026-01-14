# notebooklm-py
<p align="left">
  <img src="https://raw.githubusercontent.com/teng-lin/notebooklm-py/main/notebooklm-py.png" alt="notebooklm-py logo" width="128">
</p>

**Google NotebookLM 的非官方 API。** 自动化研究工作流、从文档生成播客，并将 NotebookLM 集成到 AI 代理中——全部通过 Python 或命令行实现。

[![PyPI version](https://img.shields.io/pypi/v/notebooklm-py.svg)](https://pypi.org/project/notebooklm-py/)
[![Python Version](https://img.shields.io/badge/python-3.10%20%7C%203.11%20%7C%203.12%20%7C%203.13%20%7C%203.14-blue)](https://pypi.org/project/notebooklm-py/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Tests](https://github.com/teng-lin/notebooklm-py/actions/workflows/test.yml/badge.svg)](https://github.com/teng-lin/notebooklm-py/actions/workflows/test.yml)

**源码与开发**: <https://github.com/teng-lin/notebooklm-py>

> **⚠️ 非官方库 - 使用风险自负**
>
> 本库使用**未公开的 Google API**，可能会在没有通知的情况下发生变化。
>
> - **与 Google 无关** - 这是一个社区项目
> - **API 可能会失效** - Google 可能随时更改内部端点
> - **存在速率限制** - 大量使用可能会被限流
>
> 最适合用于原型、研究和个人项目。调试技巧请参阅[故障排除](docs/troubleshooting.md)。

## 你可以用它来做什么

🤖 **AI 代理工具** - 将 NotebookLM 集成到 Claude Code 或其他 LLM 代理中。附带 [Claude Code 技能](#代理技能-claude-code)，支持自然语言自动化（`notebooklm skill install`），或使用异步 Python API 构建自定义集成。

📚 **研究自动化** - 批量导入来源（URL、PDF、YouTube、Google Drive），运行网络研究查询，并以编程方式提取洞见。构建可重复的研究管道。

🎙️ **内容生成** - 生成音频概述（播客）、视频、测验、闪卡和学习指南。一条命令即可将你的来源转化为精美内容。

## 三种使用方式

| 方式 | 最适合 |
|------|--------|
| **Python API** | 应用集成、异步工作流、自定义管道 |
| **CLI** | Shell 脚本、快速任务、CI/CD 自动化 |
| **代理技能** | Claude Code、LLM 代理、自然语言自动化 |

## 安装

```bash
# 基础安装
pip install notebooklm-py

# 包含浏览器登录支持（首次设置必需）
pip install "notebooklm-py[browser]"
playwright install chromium
```

## 快速开始

<p align="center">
  <a href="https://asciinema.org/a/767284" target="_blank"><img src="https://asciinema.org/a/767284.svg" width="600" /></a>
  <br>
  <em>16 分钟会话压缩到 30 秒</em>
</p>

### 命令行 (CLI)

```bash
# 1. 认证（打开浏览器）
notebooklm login

# 2. 创建笔记本
notebooklm create "我的研究"
notebooklm use <notebook_id>

# 3. 添加来源
notebooklm source add "https://zh.wikipedia.org/wiki/人工智能"
notebooklm source add "./paper.pdf"

# 4. 对话
notebooklm ask "主要主题是什么？"

# 5. 生成播客
notebooklm generate audio --wait
notebooklm download audio ./podcast.mp3
```

### Python API

```python
import asyncio
from notebooklm import NotebookLMClient

async def main():
    async with await NotebookLMClient.from_storage() as client:
        # 列出笔记本
        notebooks = await client.notebooks.list()

        # 创建笔记本并添加来源
        nb = await client.notebooks.create("研究")
        await client.sources.add_url(nb.id, "https://example.com")

        # 对话
        result = await client.chat.ask(nb.id, "总结一下")
        print(result.answer)

        # 生成播客
        status = await client.artifacts.generate_audio(nb.id)
        await client.artifacts.wait_for_completion(nb.id, status.task_id)

asyncio.run(main())
```

### 代理技能 (Claude Code)

```bash
# 通过 CLI 安装或让 Claude Code 来做
notebooklm skill install

# 然后使用自然语言：
# "创建一个关于量子计算的播客"
# "将测验下载为 Markdown"
# "/notebooklm generate video"
```

## 功能特性

| 类别 | 功能 |
|------|------|
| **笔记本** | 创建、列出、重命名、删除、分享 |
| **来源** | URL、YouTube、文件（PDF/TXT/MD/DOCX）、Google Drive、粘贴文本 |
| **对话** | 提问、对话历史、自定义角色 |
| **生成** | 音频播客、视频、幻灯片、测验、闪卡、报告、信息图、思维导图 |
| **研究** | 网络和 Drive 研究代理，自动导入 |
| **下载** | 音频、视频、幻灯片、信息图、报告、思维导图、数据表、测验、闪卡 |
| **代理技能** | Claude Code 技能，用于 LLM 驱动的自动化 |

## 文档

### 📖 本地中文文档

- **[CLI 命令参考](cli-reference.md)** - 完整的命令行文档（中文）
- **[Python API 参考](python-api.md)** - 完整的 API 参考（中文）

### 🌐 官方英文文档

- **[CLI Reference](https://github.com/teng-lin/notebooklm-py/blob/main/docs/cli-reference.md)** - Complete command documentation
- **[Python API](https://github.com/teng-lin/notebooklm-py/blob/main/docs/python-api.md)** - Full API reference
- **[Configuration](https://github.com/teng-lin/notebooklm-py/blob/main/docs/configuration.md)** - Storage and settings
- **[Troubleshooting](https://github.com/teng-lin/notebooklm-py/blob/main/docs/troubleshooting.md)** - Common issues and solutions
- **[API Stability](https://github.com/teng-lin/notebooklm-py/blob/main/docs/stability.md)** - Versioning policy and stability guarantees

### 贡献者指南

- **[开发指南](https://github.com/teng-lin/notebooklm-py/blob/main/docs/development.md)** - 架构、测试和发布
- **[RPC 开发](https://github.com/teng-lin/notebooklm-py/blob/main/docs/rpc-development.md)** - 协议捕获和调试
- **[RPC 参考](https://github.com/teng-lin/notebooklm-py/blob/main/docs/rpc-reference.md)** - 负载结构
- **[更新日志](https://github.com/teng-lin/notebooklm-py/blob/main/CHANGELOG.md)** - 版本历史和发布说明
- **[安全](https://github.com/teng-lin/notebooklm-py/blob/main/SECURITY.md)** - 安全策略和凭证处理

## 平台支持

| 平台 | 状态 | 备注 |
|------|------|------|
| **macOS** | ✅ 已测试 | 主要开发平台 |
| **Linux** | ✅ 已测试 | 完全支持 |
| **Windows** | ✅ 已测试 | 在 CI 中测试 |

## 许可证

MIT 许可证。详情请参阅 [LICENSE](LICENSE)。
