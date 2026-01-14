# CLI 命令参考

**状态：** 活跃
**最后更新：** 2026-01-10

`notebooklm` 命令行工具的完整命令参考。

## 命令结构

```
notebooklm [--storage 路径] [--version] <命令> [选项] [参数]
```

**全局选项：**
- `--storage 路径` - 覆盖默认存储位置（默认：`~/.notebooklm/storage_state.json`）
- `--version` - 显示版本并退出
- `--help` - 显示帮助信息

**环境变量：**
- `NOTEBOOKLM_HOME` - 所有配置文件的基础目录（默认：`~/.notebooklm`）
- `NOTEBOOKLM_AUTH_JSON` - 内联认证 JSON（用于 CI/CD，无需写入文件）
- `NOTEBOOKLM_DEBUG_RPC` - 启用 RPC 调试日志（设为 `1` 启用）

环境变量和 CI/CD 设置详情请参阅 [配置](configuration.md)。

**命令组织：**
- **会话命令** - 认证和上下文管理
- **笔记本命令** - 笔记本的增删改查操作
- **对话命令** - 查询和对话管理
- **分组命令** - `source`（来源）、`artifact`（产物）、`generate`（生成）、`download`（下载）、`note`（笔记）

---

## 快速参考

### 会话命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `login` | 通过浏览器认证 | `notebooklm login` |
| `use <id>` | 设置活动笔记本 | `notebooklm use abc123` |
| `status` | 显示当前上下文 | `notebooklm status` |
| `status --paths` | 显示配置文件路径 | `notebooklm status --paths` |
| `status --json` | 以 JSON 格式输出状态 | `notebooklm status --json` |
| `clear` | 清除当前上下文 | `notebooklm clear` |

### 笔记本命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `list` | 列出所有笔记本 | `notebooklm list` |
| `create <标题>` | 创建笔记本 | `notebooklm create "研究"` |
| `delete <id>` | 删除笔记本 | `notebooklm delete abc123` |
| `rename <标题>` | 重命名当前笔记本 | `notebooklm rename "新标题"` |
| `share` | 切换笔记本分享 | `notebooklm share` 或 `notebooklm share --revoke` |
| `summary` | 获取 AI 摘要 | `notebooklm summary` |

### 对话命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `ask <问题>` | 提问 | `notebooklm ask "这是关于什么的？"` |
| `ask -s <id>` | 使用特定来源提问 | `notebooklm ask "总结一下" -s src1 -s src2` |
| `ask --json` | 获取带来源引用的答案 | `notebooklm ask "解释X" --json` |
| `configure` | 设置角色/模式 | `notebooklm configure --mode learning-guide` |
| `history` | 查看/清除历史 | `notebooklm history --clear` |

### 来源命令 (`notebooklm source <命令>`)

| 命令 | 参数 | 选项 | 示例 |
|------|------|------|------|
| `list` | - | - | `source list` |
| `add <内容>` | URL/文件/文本 | - | `source add "https://..."` |
| `add-drive <id> <标题>` | Drive 文件 ID | - | `source add-drive abc123 "文档"` |
| `add-research <查询>` | 搜索查询 | `--mode [fast|deep]`, `--from [web|drive]`, `--import-all`, `--no-wait` | `source add-research "AI" --mode deep --no-wait` |
| `get <id>` | 来源 ID | - | `source get src123` |
| `fulltext <id>` | 来源 ID | `--json`, `-o 文件` | `source fulltext src123 -o content.txt` |
| `guide <id>` | 来源 ID | `--json` | `source guide src123` |
| `rename <id> <标题>` | 来源 ID, 新标题 | - | `source rename src123 "新名称"` |
| `refresh <id>` | 来源 ID | - | `source refresh src123` |
| `delete <id>` | 来源 ID | - | `source delete src123` |
| `wait <id>` | 来源 ID | `--timeout`, `--interval` | `source wait src123` |

### 研究命令 (`notebooklm research <命令>`)

| 命令 | 参数 | 选项 | 示例 |
|------|------|------|------|
| `status` | - | `--json` | `research status` |
| `wait` | - | `--timeout`, `--interval`, `--import-all`, `--json` | `research wait --import-all` |

### 生成命令 (`notebooklm generate <类型>`)

所有生成命令都支持：
- `--source/-s` 选择特定来源（可重复）
- `--json` 输出机器可读格式（返回 `task_id` 和 `status`）

| 命令 | 选项 | 示例 |
|------|------|------|
| `audio [描述]` | `--format`, `--length`, `-s/--source`, `--wait`, `--json` | `generate audio "聚焦历史"` |
| `video [描述]` | `--style`, `--format`, `-s/--source`, `--wait`, `--json` | `generate video "儿童科普"` |
| `slide-deck [描述]` | `--format`, `--length`, `-s/--source`, `--wait`, `--json` | `generate slide-deck` |
| `quiz [描述]` | `--difficulty`, `--quantity`, `-s/--source`, `--wait`, `--json` | `generate quiz --difficulty hard` |
| `flashcards [描述]` | `--difficulty`, `--quantity`, `-s/--source`, `--wait`, `--json` | `generate flashcards` |
| `infographic [描述]` | `--orientation`, `--detail`, `-s/--source`, `--wait`, `--json` | `generate infographic` |
| `data-table [描述]` | `-s/--source`, `--wait`, `--json` | `generate data-table` |
| `mind-map` | `-s/--source`, `--json` *（同步，无需等待）* | `generate mind-map` |
| `report [描述]` | `--type`, `-s/--source`, `--wait`, `--json` | `generate report --type study-guide` |

### 产物命令 (`notebooklm artifact <命令>`)

| 命令 | 参数 | 选项 | 示例 |
|------|------|------|------|
| `list` | - | `--type` | `artifact list --type audio` |
| `get <id>` | 产物 ID | - | `artifact get art123` |
| `rename <id> <标题>` | 产物 ID, 标题 | - | `artifact rename art123 "标题"` |
| `delete <id>` | 产物 ID | - | `artifact delete art123` |
| `export <id>` | 产物 ID | `--type [docs|sheets]`, `--title` | `artifact export art123 --type sheets` |
| `poll <task_id>` | 任务 ID | - | `artifact poll task123` |
| `wait <id>` | 产物 ID | `--timeout`, `--interval` | `artifact wait art123` |
| `suggestions` | - | `-s/--source`, `--json` | `artifact suggestions` |

### 下载命令 (`notebooklm download <类型>`)

| 命令 | 参数 | 选项 | 示例 |
|------|------|------|------|
| `audio [路径]` | 输出路径 | `-a/--artifact`, `--all`, `--latest`, `--name`, `--force`, `--dry-run` | `download audio --all` |
| `video [路径]` | 输出路径 | `-a/--artifact`, `--all`, `--latest`, `--name`, `--force`, `--dry-run` | `download video --latest` |
| `slide-deck [路径]` | 输出目录 | `-a/--artifact`, `--all`, `--latest`, `--name`, `--force`, `--dry-run` | `download slide-deck ./slides/` |
| `infographic [路径]` | 输出路径 | `-a/--artifact`, `--all`, `--latest`, `--name`, `--force`, `--dry-run` | `download infographic ./info.png` |
| `report [路径]` | 输出路径 | `-a/--artifact`, `--all`, `--latest`, `--name`, `--force`, `--dry-run` | `download report ./report.md` |
| `mind-map [路径]` | 输出路径 | `-a/--artifact`, `--all`, `--latest`, `--name`, `--force`, `--dry-run` | `download mind-map ./map.json` |
| `data-table [路径]` | 输出路径 | `-a/--artifact`, `--all`, `--latest`, `--name`, `--force`, `--dry-run` | `download data-table ./data.csv` |
| `quiz [路径]` | 输出路径 | `-n/--notebook`, `-a/--artifact`, `--format` (json/markdown/html) | `download quiz --format markdown quiz.md` |
| `flashcards [路径]` | 输出路径 | `-n/--notebook`, `-a/--artifact`, `--format` (json/markdown/html) | `download flashcards cards.json` |

### 笔记命令 (`notebooklm note <命令>`)

| 命令 | 参数 | 选项 | 示例 |
|------|------|------|------|
| `list` | - | - | `note list` |
| `create <内容>` | 笔记内容 | - | `note create "我的笔记..."` |
| `get <id>` | 笔记 ID | - | `note get note123` |
| `save <id>` | 笔记 ID | - | `note save note123` |
| `rename <id> <标题>` | 笔记 ID, 标题 | - | `note rename note123 "标题"` |
| `delete <id>` | 笔记 ID | - | `note delete note123` |

### 技能命令 (`notebooklm skill <命令>`)

管理 Claude Code 技能集成。

| 命令 | 描述 | 示例 |
|------|------|------|
| `install` | 安装/更新技能到 ~/.claude/skills/ | `skill install` |
| `status` | 检查安装状态和版本 | `skill status` |
| `uninstall` | 移除技能 | `skill uninstall` |
| `show` | 显示技能内容 | `skill show` |

安装后，Claude Code 可以通过 `/notebooklm` 或自然语言（如 "创建一个关于X的播客"）识别 NotebookLM 命令。

---

## 详细命令参考

### 会话：`login`

通过浏览器进行 Google NotebookLM 认证。

```bash
notebooklm login
```

打开一个带有持久化配置的 Chromium 浏览器。登录你的 Google 账号，然后在终端按回车保存会话。

### 会话：`use`

设置后续命令的活动笔记本。

```bash
notebooklm use <notebook_id>
```

支持部分 ID 匹配：
```bash
notebooklm use abc  # 匹配 abc123def456...
```

### 会话：`status`

显示当前上下文（活动笔记本和对话）。

```bash
notebooklm status [选项]
```

**选项：**
- `--paths` - 显示已解析的配置文件路径
- `--json` - 以 JSON 格式输出（用于脚本）

**示例：**
```bash
# 基本状态
notebooklm status

# 显示配置文件位置
notebooklm status --paths
# 输出显示 home_dir、storage_path、context_path、browser_profile_dir

# 脚本用 JSON 输出
notebooklm status --json
```

**使用 `--paths` 时的输出：**
```
                配置路径
┏━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┓
┃ 文件            ┃ 路径                         ┃ 来源            ┃
┡━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━┩
│ 主目录          │ /home/user/.notebooklm      │ 默认            │
│ 存储状态        │ .../storage_state.json      │                 │
│ 上下文          │ .../context.json            │                 │
│ 浏览器配置      │ .../browser_profile         │                 │
└─────────────────┴──────────────────────────────┴─────────────────┘
```

### 来源：`add-research`

执行 AI 驱动的研究，并将发现的来源添加到笔记本。

```bash
notebooklm source add-research <查询> [选项]
```

**选项：**
- `--mode [fast|deep]` - 研究深度（默认：fast）
- `--from [web|drive]` - 搜索来源（默认：web）
- `--import-all` - 自动导入所有找到的来源（适用于阻塞模式）
- `--no-wait` - 启动研究后立即返回（非阻塞）

**示例：**
```bash
# 快速网络研究（阻塞）
notebooklm source add-research "量子计算基础"

# 深度研究 Google Drive
notebooklm source add-research "项目Alpha" --from drive --mode deep

# 代理工作流的非阻塞深度研究
notebooklm source add-research "AI安全论文" --mode deep --no-wait
```

### 生成：`audio`

生成音频概述（播客式音频）。

```bash
notebooklm generate audio [描述] [选项]
```

**选项：**
- `--format [default|academic|debate|storytelling]` - 播客格式/风格
- `--length [default|short]` - 内容长度
- `-s, --source ID` - 使用特定来源（可重复，未指定则使用全部）
- `--wait` - 等待生成完成（默认：立即返回）
- `--json` - 输出 JSON 格式（包含 task_id 和 status）

**示例：**
```bash
# 带自定义提示生成播客
notebooklm generate audio "聚焦实际应用，使用轻松的语气"

# 生成简短学术风格播客
notebooklm generate audio --format academic --length short

# 仅使用特定来源
notebooklm generate audio -s source_abc -s source_xyz

# 阻塞等待完成
notebooklm generate audio --wait
```

### 生成：`video`

生成视频概述。

```bash
notebooklm generate video [描述] [选项]
```

**选项：**
- `--style [default|explainer|storytelling]` - 视频风格
- `--format [default|voiceover|conversation]` - 解说格式
- `-s, --source ID` - 使用特定来源（可重复）
- `--wait` - 等待生成完成
- `--json` - 输出 JSON 格式

**示例：**
```bash
# 生成解说视频
notebooklm generate video "用简单术语解释这个主题"

# 带特定风格和格式
notebooklm generate video --style storytelling --format voiceover

# 从选定来源生成
notebooklm generate video -s src_001 -s src_002 --wait
```

### 生成：`quiz`

生成测验题。

```bash
notebooklm generate quiz [描述] [选项]
```

**选项：**
- `--difficulty [default|easy|medium|hard]` - 难度级别
- `--quantity [default|few|many]` - 题目数量
- `-s, --source ID` - 使用特定来源（可重复）
- `--wait` - 等待生成完成
- `--json` - 输出 JSON 格式

**示例：**
```bash
# 生成高难度测验
notebooklm generate quiz --difficulty hard

# 生成少量简单题目
notebooklm generate quiz --difficulty easy --quantity few

# 指定来源
notebooklm generate quiz -s src_001 --wait
```

### 生成：`flashcards`

生成闪卡。

```bash
notebooklm generate flashcards [描述] [选项]
```

**选项：**
- `--difficulty [default|easy|medium|hard]` - 难度级别
- `--quantity [default|few|many]` - 闪卡数量
- `-s, --source ID` - 使用特定来源（可重复）
- `--wait` - 等待生成完成
- `--json` - 输出 JSON 格式

### 生成：`report`

生成报告（简报、学习指南等）。

```bash
notebooklm generate report [描述] [选项]
```

**选项：**
- `--type [briefing-doc|study-guide|blog-post|custom]` - 报告类型
- `-s, --source ID` - 使用特定来源（可重复，未指定则使用全部）

**示例：**
```bash
notebooklm generate report --type study-guide
notebooklm generate report "给利益相关者的执行摘要" --type briefing-doc

# 从特定来源生成报告
notebooklm generate report --type study-guide -s src_001 -s src_002
```

### 下载：`audio`、`video`、`slide-deck`、`infographic`、`report`、`mind-map`、`data-table`

将生成的产物下载到本地。

```bash
notebooklm download <类型> [输出路径] [选项]
```

**产物类型和输出格式：**

| 类型 | 默认扩展名 | 描述 |
|------|-----------|------|
| `audio` | `.mp4` | 音频概述（播客），MP4 容器格式 |
| `video` | `.mp4` | 视频概述 |
| `slide-deck` | `.pdf` | 幻灯片，PDF 格式 |
| `infographic` | `.png` | 信息图，PNG 图片 |
| `report` | `.md` | 报告，Markdown 格式（简报、学习指南等） |
| `mind-map` | `.json` | 思维导图，JSON 树结构 |
| `data-table` | `.csv` | 数据表，CSV 格式（带 BOM 的 UTF-8，兼容 Excel） |

**选项：**
- `--all` - 下载该类型的所有产物
- `--latest` - 只下载最新的产物（未指定 ID/名称时默认）
- `--earliest` - 只下载最早的产物
- `--name 名称` - 下载标题匹配的产物（支持部分匹配）
- `-a, --artifact ID` - 通过 ID 选择特定产物
- `--dry-run` - 显示将要下载的内容，不实际下载
- `--force` - 覆盖已存在的文件
- `--no-clobber` - 如果文件已存在则跳过（默认）
- `--json` - 以 JSON 格式输出结果

**示例：**
```bash
# 下载最新的播客
notebooklm download audio ./podcast.mp3

# 下载所有信息图
notebooklm download infographic --all

# 通过名称下载特定幻灯片
notebooklm download slide-deck --name "最终演示"

# 预览批量下载
notebooklm download audio --all --dry-run

# 下载报告为 markdown
notebooklm download report ./study-guide.md

# 下载思维导图为 JSON
notebooklm download mind-map ./concept-map.json

# 下载数据表为 CSV（可在 Excel 中打开）
notebooklm download data-table ./research-data.csv
```

### 下载：`quiz`、`flashcards`

下载测验题或闪卡，支持多种格式。

```bash
notebooklm download quiz [输出路径] [选项]
notebooklm download flashcards [输出路径] [选项]
```

**选项：**
- `-n, --notebook ID` - 笔记本 ID（未设置则使用当前上下文）
- `--format 格式` - 输出格式：`json`（默认）、`markdown` 或 `html`
- `-a, --artifact ID` - 通过 ID 选择特定产物

**输出格式：**
- **JSON** - 结构化数据，保留完整 API 字段（answerOptions、rationale、isCorrect、hint）
- **Markdown** - 人类可读格式，正确答案带复选框
- **HTML** - NotebookLM 返回的原始 HTML

**示例：**
```bash
# 下载测验为 JSON
notebooklm download quiz quiz.json

# 下载测验为 markdown
notebooklm download quiz --format markdown quiz.md

# 下载闪卡为 JSON（自动将 f/b 键规范化为 front/back）
notebooklm download flashcards cards.json

# 下载闪卡为 markdown
notebooklm download flashcards --format markdown cards.md

# 下载闪卡为原始 HTML
notebooklm download flashcards --format html cards.html
```

---

## 常用工作流

### 研究 → 播客

查找某个主题的信息并创建播客。

```bash
# 1. 为这个研究创建笔记本
notebooklm create "气候变化研究"
# 输出：Created notebook: abc123

# 2. 设为活动笔记本
notebooklm use abc123

# 3. 添加一个起始来源
notebooklm source add "https://zh.wikipedia.org/wiki/气候变化"

# 4. 自动研究更多来源（阻塞 - 最多等待 5 分钟）
notebooklm source add-research "2024年气候变化政策" --mode deep --import-all

# 5. 生成播客
notebooklm generate audio "聚焦政策解决方案和未来展望" --format debate --wait

# 6. 下载结果
notebooklm download audio ./climate-podcast.mp3
```

### 研究 → 播客（使用子代理的非阻塞模式）

对于 LLM 代理，使用非阻塞模式避免超时：

```bash
# 1-3. 创建笔记本并添加初始来源（同上）
notebooklm create "气候变化研究"
notebooklm use abc123
notebooklm source add "https://zh.wikipedia.org/wiki/气候变化"

# 4. 启动深度研究（非阻塞）
notebooklm source add-research "2024年气候变化政策" --mode deep --no-wait
# 立即返回

# 5. 在子代理中，等待研究完成并导入
notebooklm research wait --import-all --timeout 300
# 阻塞直到完成，然后导入来源

# 6. 继续生成播客...
```

**研究命令：**
- `research status` - 检查研究是否正在进行、已完成或未运行
- `research wait --import-all` - 阻塞直到研究完成，然后导入来源

### 文档分析 → 学习资料

上传文档并创建学习资料。

```bash
# 1. 创建笔记本
notebooklm create "考试准备"
notebooklm use <id>

# 2. 添加你的文档
notebooklm source add "./textbook-chapter.pdf"
notebooklm source add "./lecture-notes.pdf"

# 3. 获取摘要
notebooklm summary

# 4. 生成学习资料
notebooklm generate quiz --difficulty hard --wait
notebooklm generate flashcards --wait
notebooklm generate report --type study-guide --wait

# 5. 提出具体问题
notebooklm ask "解释第3章的关键概念"
notebooklm ask "最可能出现的考试题目是什么？"
```

### YouTube → 快速摘要

将 YouTube 视频转成笔记。

```bash
# 1. 创建笔记本并添加视频
notebooklm create "视频笔记"
notebooklm use <id>
notebooklm source add "https://www.youtube.com/watch?v=VIDEO_ID"

# 2. 获取摘要
notebooklm summary

# 3. 提问
notebooklm ask "主要观点是什么？"
notebooklm ask "创建要点笔记"

# 4. 生成快速简报
notebooklm generate report --type briefing-doc --wait
```

### 批量导入

一次添加多个来源。

```bash
# 设置活动笔记本
notebooklm use <id>

# 添加多个 URL
notebooklm source add "https://example.com/article1"
notebooklm source add "https://example.com/article2"
notebooklm source add "https://example.com/article3"

# 添加多个本地文件（使用循环）
for f in ./papers/*.pdf; do
  notebooklm source add "$f"
done
```

---

## LLM 代理使用技巧

以编程方式使用此 CLI 时：

1. **两种指定笔记本的方式**：使用 `notebooklm use <id>` 设置上下文，或直接向命令传递 `-n <id>`。大多数命令支持 `-n/--notebook` 作为显式覆盖。

2. **生成命令默认是异步的**（mind-map 除外）：
   - `mind-map`：同步，立即完成（没有 `--wait` 选项）
   - 其他所有：默认立即返回任务 ID（`--no-wait`）

   LLM 代理应避免使用 `--wait`——所有异步操作可能需要几分钟到 30 分钟以上。使用 `artifact wait <id>` 在后台任务中等待，或通知用户稍后检查。

3. **部分 ID 有效**：`notebooklm use abc` 匹配任何以 "abc" 开头的笔记本 ID。

4. **检查状态**：使用 `notebooklm status` 查看当前活动笔记本和对话。

5. **自动检测**：`source add` 自动检测内容类型：
   - 以 `http` 开头的 URL → 网页来源
   - YouTube URL → 视频字幕提取
   - 文件路径 → 文件上传

6. **错误处理**：命令失败时返回非零退出状态。检查 stderr 获取错误信息。

7. **深度研究**：对 `source add-research --mode deep` 使用 `--no-wait` 避免阻塞。然后在子代理中使用 `research wait --import-all` 等待完成。
