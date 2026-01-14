# Python API 参考

**状态：** 活跃
**最后更新：** 2026-01-10

`notebooklm` Python 库的完整参考。

## 快速开始

```python
import asyncio
from notebooklm import NotebookLMClient

async def main():
    # 从保存的认证创建客户端
    async with await NotebookLMClient.from_storage() as client:
        # 列出笔记本
        notebooks = await client.notebooks.list()
        print(f"找到 {len(notebooks)} 个笔记本")

        # 创建新笔记本
        nb = await client.notebooks.create("我的研究")
        print(f"已创建: {nb.id}")

        # 添加来源
        await client.sources.add_url(nb.id, "https://example.com/article")

        # 提问
        result = await client.chat.ask(nb.id, "总结主要观点")
        print(result.answer)

        # 生成播客
        status = await client.artifacts.generate_audio(nb.id)
        final = await client.artifacts.wait_for_completion(nb.id, status.task_id)
        print(f"音频已就绪: {final.url}")

asyncio.run(main())
```

---

## 核心概念

### 异步上下文管理器

客户端必须作为异步上下文管理器使用，以正确管理 HTTP 连接：

```python
# 正确 - 使用上下文管理器
async with await NotebookLMClient.from_storage() as client:
    ...

# 也正确 - 手动管理
client = await NotebookLMClient.from_storage()
await client.__aenter__()
try:
    ...
finally:
    await client.__aexit__(None, None, None)
```

### 认证

客户端需要通过浏览器登录获取的有效 Google 会话 Cookie：

```python
# 从存储文件（推荐）
client = await NotebookLMClient.from_storage()
client = await NotebookLMClient.from_storage("/path/to/storage_state.json")

# 直接从 AuthTokens
from notebooklm import AuthTokens
auth = AuthTokens(
    cookies={"SID": "...", "HSID": "...", ...},
    csrf_token="...",
    session_id="..."
)
client = NotebookLMClient(auth)
```

**环境变量支持：**

该库支持以下认证环境变量：

| 变量 | 描述 |
|------|------|
| `NOTEBOOKLM_HOME` | 配置文件基础目录（默认：`~/.notebooklm`） |
| `NOTEBOOKLM_AUTH_JSON` | 内联认证 JSON - 无需文件（用于 CI/CD） |

**优先级**（从高到低）：
1. `from_storage()` 的显式 `path` 参数
2. `NOTEBOOKLM_AUTH_JSON` 环境变量
3. `$NOTEBOOKLM_HOME/storage_state.json`
4. `~/.notebooklm/storage_state.json`

**CI/CD 示例：**
```python
import os

# 从环境变量设置认证 JSON（例如 GitHub Actions secret）
os.environ["NOTEBOOKLM_AUTH_JSON"] = '{"cookies": [...]}'

# 客户端自动使用环境变量
async with await NotebookLMClient.from_storage() as client:
    notebooks = await client.notebooks.list()
```

### 错误处理

该库在 API 失败时抛出 `RPCError`：

```python
from notebooklm import RPCError

try:
    result = await client.notebooks.create("测试")
except RPCError as e:
    print(f"RPC 失败: {e}")
    # 常见原因：
    # - 会话过期（重新运行 `notebooklm login`）
    # - 速率限制（等待后重试）
    # - 无效参数
```

### 认证和令牌刷新

**自动刷新：** 客户端在检测到认证错误时会自动刷新 CSRF 令牌。这在任何 API 调用期间透明发生 - 你无需手动处理。

当 RPC 调用因认证错误失败时（HTTP 401/403 或认证相关消息）：
1. 客户端从 NotebookLM 主页获取新令牌
2. 短暂等待以避免速率限制
3. 自动重试失败的请求

**手动刷新：** 用于主动刷新（例如，在长时间运行的操作之前）：

```python
async with await NotebookLMClient.from_storage() as client:
    # 手动刷新 CSRF 令牌和会话 ID
    await client.refresh_auth()
```

**注意：** 如果你的会话 Cookie 已完全过期（不仅仅是 CSRF 令牌），你需要重新运行 `notebooklm login`。

---

## API 参考

### NotebookLMClient

主客户端类，提供对所有 API 的访问。

```python
class NotebookLMClient:
    notebooks: NotebooksAPI    # 笔记本操作
    sources: SourcesAPI        # 来源管理
    artifacts: ArtifactsAPI    # AI 生成内容
    chat: ChatAPI              # 对话
    research: ResearchAPI      # 网络/Drive 研究
    notes: NotesAPI            # 用户笔记

    @classmethod
    async def from_storage(cls, path: str = None) -> "NotebookLMClient"

    async def refresh_auth(self) -> AuthTokens
```

---

### NotebooksAPI (`client.notebooks`)

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `list()` | - | `list[Notebook]` | 列出所有笔记本 |
| `create(title)` | `title: str` | `Notebook` | 创建笔记本 |
| `get(notebook_id)` | `notebook_id: str` | `Notebook` | 获取笔记本详情 |
| `delete(notebook_id)` | `notebook_id: str` | `bool` | 删除笔记本 |
| `rename(notebook_id, new_title)` | `notebook_id: str, new_title: str` | `Notebook` | 重命名笔记本 |
| `get_description(notebook_id)` | `notebook_id: str` | `NotebookDescription` | 获取 AI 摘要和主题 |
| `get_summary(notebook_id)` | `notebook_id: str` | `str` | 获取原始摘要文本 |
| `share(notebook_id, settings=None)` | `notebook_id: str, settings: dict` | `Any` | 使用设置分享笔记本 |
| `remove_from_recent(notebook_id)` | `notebook_id: str` | `None` | 从最近查看中移除 |
| `get_raw(notebook_id)` | `notebook_id: str` | `Any` | 获取原始 API 响应数据 |

**示例：**
```python
# 列出所有笔记本
notebooks = await client.notebooks.list()
for nb in notebooks:
    print(f"{nb.id}: {nb.title} ({nb.sources_count} 个来源)")

# 创建并重命名
nb = await client.notebooks.create("草稿")
nb = await client.notebooks.rename(nb.id, "最终版本")

# 获取 AI 生成的描述（解析后带建议主题）
desc = await client.notebooks.get_description(nb.id)
print(desc.summary)
for topic in desc.suggested_topics:
    print(f"  - {topic.question}")

# 获取原始摘要文本（未解析）
summary = await client.notebooks.get_summary(nb.id)
print(summary)

# 分享笔记本
await client.notebooks.share(nb.id, settings={"public": True})
```

**get_summary 与 get_description 的区别：**
- `get_summary()` 返回原始摘要文本字符串
- `get_description()` 返回 `NotebookDescription` 对象，包含解析后的摘要和建议问题的 `SuggestedTopic` 对象列表

---

### SourcesAPI (`client.sources`)

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `list(notebook_id)` | `notebook_id: str` | `list[Source]` | 列出来源 |
| `get(notebook_id, source_id)` | `str, str` | `Source` | 获取来源详情 |
| `get_fulltext(notebook_id, source_id)` | `str, str` | `SourceFulltext` | 获取完整索引文本内容 |
| `get_guide(notebook_id, source_id)` | `str, str` | `dict` | 获取 AI 生成的摘要和关键词 |
| `add_url(notebook_id, url)` | `str, str` | `Source` | 添加 URL 来源 |
| `add_youtube(notebook_id, url)` | `str, str` | `Source` | 添加 YouTube 视频 |
| `add_text(notebook_id, title, content)` | `str, str, str` | `Source` | 添加文本内容 |
| `add_file(notebook_id, path, mime_type=None)` | `str, Path, str` | `Source` | 上传文件 |
| `add_drive(notebook_id, file_id, title, mime_type)` | `str, str, str, str` | `Source` | 添加 Google Drive 文档 |
| `rename(notebook_id, source_id, new_title)` | `str, str, str` | `Source` | 重命名来源 |
| `refresh(notebook_id, source_id)` | `str, str` | `bool` | 刷新 URL/Drive 来源 |
| `delete(notebook_id, source_id)` | `str, str` | `bool` | 删除来源 |

**示例：**
```python
# 添加各种来源类型
await client.sources.add_url(nb_id, "https://example.com/article")
await client.sources.add_youtube(nb_id, "https://youtube.com/watch?v=...")
await client.sources.add_text(nb_id, "我的笔记", "内容在这里...")
await client.sources.add_file(nb_id, Path("./document.pdf"))

# 列出和管理
sources = await client.sources.list(nb_id)
for src in sources:
    print(f"{src.id}: {src.title} ({src.source_type})")

await client.sources.rename(nb_id, src.id, "更好的标题")
await client.sources.refresh(nb_id, src.id)  # 重新获取 URL 内容

# 获取完整索引内容（NotebookLM 用于回答的内容）
fulltext = await client.sources.get_fulltext(nb_id, src.id)
print(f"内容 ({fulltext.char_count} 字符): {fulltext.content[:500]}...")
```

---

### ArtifactsAPI (`client.artifacts`)

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `list(notebook_id)` | `str` | `list[Artifact]` | 列出所有产物 |
| `get(notebook_id, artifact_id)` | `str, str` | `Artifact` | 获取产物详情 |
| `delete(notebook_id, artifact_id)` | `str, str` | `bool` | 删除产物 |
| `rename(notebook_id, artifact_id, title)` | `str, str, str` | `Artifact` | 重命名产物 |
| `generate_audio(...)` | 见下文 | `GenerationStatus` | 生成播客 |
| `generate_video(...)` | 见下文 | `GenerationStatus` | 生成视频 |
| `generate_quiz(...)` | 见下文 | `GenerationStatus` | 生成测验 |
| `generate_flashcards(...)` | 见下文 | `GenerationStatus` | 生成闪卡 |
| `generate_mind_map(...)` | 见下文 | `Artifact` | 生成思维导图（同步） |
| `generate_report(...)` | 见下文 | `GenerationStatus` | 生成报告 |
| `generate_infographic(...)` | 见下文 | `GenerationStatus` | 生成信息图 |
| `generate_slide_deck(...)` | 见下文 | `GenerationStatus` | 生成幻灯片 |
| `generate_data_table(...)` | 见下文 | `GenerationStatus` | 生成数据表 |
| `poll_task(notebook_id, task_id)` | `str, str` | `GenerationStatus` | 检查任务状态 |
| `wait_for_completion(...)` | `str, str` | `Artifact` | 等待生成完成 |
| `get_suggestions(notebook_id, source_ids=None)` | `str, list[str]` | `list[Suggestion]` | 获取产物建议 |
| `export(notebook_id, artifact_id, export_type, title=None)` | `str, str, ExportType, str` | `ExportResult` | 导出到 Docs/Sheets |

**生成方法参数：**

```python
# 音频生成
await client.artifacts.generate_audio(
    notebook_id,
    source_ids: list[str] = None,      # 要使用的来源（默认全部）
    prompt: str = None,                 # 自定义指令
    format: AudioFormat = None,         # DEEP_DIVE, BRIEF, CRITIQUE, DEBATE
    length: AudioLength = None,         # SHORT, DEFAULT, LONG
)

# 视频生成
await client.artifacts.generate_video(
    notebook_id,
    source_ids: list[str] = None,
    prompt: str = None,
    format: VideoFormat = None,         # EXPLAINER, BRIEF
    style: VideoStyle = None,           # CLASSIC, WHITEBOARD, KAWAII, 等
)

# 测验生成
await client.artifacts.generate_quiz(
    notebook_id,
    source_ids: list[str] = None,
    prompt: str = None,
    difficulty: QuizDifficulty = None,  # EASY, MEDIUM, HARD
    quantity: QuizQuantity = None,      # FEWER, STANDARD
)

# 报告生成
await client.artifacts.generate_report(
    notebook_id,
    source_ids: list[str] = None,
    prompt: str = None,
    format: ReportFormat = None,        # BRIEFING_DOC, STUDY_GUIDE, BLOG_POST, CUSTOM
)
```

**示例：**
```python
# 生成播客
status = await client.artifacts.generate_audio(
    nb_id,
    prompt="聚焦实际应用",
    format=AudioFormat.DEBATE,
    length=AudioLength.SHORT,
)

# 等待完成
artifact = await client.artifacts.wait_for_completion(nb_id, status.task_id)
print(f"完成！下载: {artifact.url}")

# 生成学习指南报告
status = await client.artifacts.generate_report(
    nb_id,
    format=ReportFormat.STUDY_GUIDE,
)

# 轮询状态
while True:
    status = await client.artifacts.poll_task(nb_id, status.task_id)
    if status.status == "completed":
        break
    await asyncio.sleep(5)
```

---

### ChatAPI (`client.chat`)

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `ask(...)` | 见下文 | `AskResult` | 提问 |
| `configure(notebook_id, ...)` | 见下文 | `bool` | 配置对话模式 |
| `get_history(notebook_id, conversation_id)` | `str, str` | `list[dict]` | 获取对话历史 |

**ask 方法参数：**
```python
await client.chat.ask(
    notebook_id: str,
    question: str,
    source_ids: list[str] = None,       # 限制到特定来源
    conversation_id: str = None,        # 用于后续对话
)
```

**configure 方法参数：**
```python
await client.chat.configure(
    notebook_id: str,
    goal: ChatGoal = None,              # DEFAULT, CUSTOM, LEARNING_GUIDE
    custom_prompt: str = None,          # 用于 goal=CUSTOM 时的指令
    response_length: ChatResponseLength = None,  # DEFAULT, LONGER, SHORTER
)
```

**示例：**
```python
# 简单提问
result = await client.chat.ask(nb_id, "这个文档是关于什么的？")
print(result.answer)

# 访问引用
for ref in result.references:
    print(f"[{ref.citation_number}] 来自 {ref.source_id}: {ref.cited_text}")

# 后续对话（同一对话）
follow_up = await client.chat.ask(
    nb_id,
    "详细解释第一点",
    conversation_id=result.conversation_id,
)

# 配置对话模式
await client.chat.configure(
    nb_id,
    goal=ChatGoal.LEARNING_GUIDE,
    response_length=ChatResponseLength.LONGER,
)
```

---

### ResearchAPI (`client.research`)

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `start_web_research(...)` | 见下文 | `ResearchStatus` | 启动网络研究 |
| `start_drive_research(...)` | 见下文 | `ResearchStatus` | 启动 Drive 研究 |
| `get_status(notebook_id)` | `str` | `ResearchStatus` | 检查研究状态 |
| `wait_for_completion(...)` | `str, int, int` | `ResearchStatus` | 等待研究完成 |
| `import_source(notebook_id, source)` | `str, ResearchSource` | `Source` | 导入发现的来源 |
| `import_all_sources(notebook_id)` | `str` | `list[Source]` | 导入所有发现的来源 |

**研究方法参数：**
```python
# 网络研究
await client.research.start_web_research(
    notebook_id: str,
    query: str,
    deep: bool = False,  # 快速 vs 深度研究
)

# Google Drive 研究
await client.research.start_drive_research(
    notebook_id: str,
    query: str,
    deep: bool = False,
)
```

**示例：**
```python
# 启动深度网络研究
status = await client.research.start_web_research(
    nb_id,
    "2024年量子计算进展",
    deep=True,
)

# 等待完成（最多 5 分钟）
final_status = await client.research.wait_for_completion(
    nb_id,
    timeout=300,
    poll_interval=10,
)

# 导入所有发现的来源
sources = await client.research.import_all_sources(nb_id)
print(f"导入了 {len(sources)} 个来源")

# 或选择性导入
status = await client.research.get_status(nb_id)
for research_source in status.sources:
    if "重要" in research_source.title:
        await client.research.import_source(nb_id, research_source)
```

---

### NotesAPI (`client.notes`)

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `list(notebook_id)` | `str` | `list[Note]` | 列出所有笔记 |
| `create(notebook_id, content)` | `str, str` | `Note` | 创建笔记 |
| `get(notebook_id, note_id)` | `str, str` | `Note` | 获取笔记内容 |
| `save(notebook_id, note_id)` | `str, str` | `Note` | 保存/固定笔记 |
| `rename(notebook_id, note_id, title)` | `str, str, str` | `Note` | 重命名笔记 |
| `delete(notebook_id, note_id)` | `str, str` | `bool` | 删除笔记 |

**示例：**
```python
# 创建笔记
note = await client.notes.create(nb_id, "今天会议的要点...")

# 列出和管理
notes = await client.notes.list(nb_id)
for n in notes:
    print(f"{n.id}: {n.title}")

await client.notes.rename(nb_id, note.id, "会议笔记 2024-01-15")
await client.notes.save(nb_id, note.id)  # 保存/固定
```

---

## 数据模型

### Notebook

```python
@dataclass
class Notebook:
    id: str
    title: str
    sources_count: int
    created_at: Optional[datetime]
    updated_at: Optional[datetime]
```

### Source

```python
@dataclass
class Source:
    id: str
    title: str
    source_type: str  # "url", "youtube", "text", "pdf", "upload", 等
    created_at: Optional[datetime]
```

### Artifact

```python
@dataclass
class Artifact:
    id: str
    title: Optional[str]
    artifact_type: StudioContentType
    status: str  # "in_progress", "completed", "failed"
    url: Optional[str]
    created_at: Optional[datetime]
```

### AskResult

```python
@dataclass
class AskResult:
    answer: str                        # 带内联引用 [1], [2] 等的答案文本
    conversation_id: str               # 用于后续问题的 ID
    turn_number: int                   # 对话中的轮次号
    is_follow_up: bool                 # 是否为后续问题
    references: list[ChatReference]    # 答案中引用的来源参考
    raw_response: str                  # 原始 API 响应的前 1000 个字符

@dataclass
class ChatReference:
    source_id: str                     # 来源的 UUID
    citation_number: int | None        # 答案中的引用编号（1, 2, 等）
    cited_text: str | None             # 被引用的实际文本段落
    start_char: int | None             # 来源内容中的起始位置
    end_char: int | None               # 来源内容中的结束位置
    chunk_id: str | None               # 内部块 ID（用于调试）
```

**重要说明：** `cited_text` 字段通常只包含片段或章节标题，而不是完整的引用段落。`start_char`/`end_char` 位置引用 NotebookLM 的内部分块索引，不直接对应 `get_fulltext()` 返回的原始全文中的位置。

使用 `SourceFulltext.find_citation_context()` 在全文中定位引用：

```python
fulltext = await client.sources.get_fulltext(notebook_id, ref.source_id)
matches = fulltext.find_citation_context(ref.cited_text)  # 返回 list[(context, position)]

if matches:
    context, pos = matches[0]  # 第一个匹配
    if len(matches) > 1:
        print(f"警告：找到 {len(matches)} 个匹配，使用第一个")
else:
    context = None  # 未找到 - 可能是来源被修改了
```

**提示：** 处理同一来源的多个引用时，缓存 `fulltext` 以避免重复 API 调用。

### SourceFulltext

```python
@dataclass
class SourceFulltext:
    source_id: str                     # 来源的 UUID
    title: str                         # 来源标题
    content: str                       # 完整索引文本内容
    source_type: int | None            # 来源类型代码
    url: str | None                    # 原始 URL（如适用）
    char_count: int                    # 字符数

    def find_citation_context(
        self,
        cited_text: str,
        context_chars: int = 200,
    ) -> list[tuple[str, int]]:
        """搜索引用文本，返回 (上下文, 位置) 元组列表。"""
```

---

## 枚举

### 音频生成

```python
class AudioFormat(Enum):
    DEEP_DIVE = 1   # 深入讨论
    BRIEF = 2       # 快速摘要
    CRITIQUE = 3    # 批判性分析
    DEBATE = 4      # 双方辩论

class AudioLength(Enum):
    SHORT = 1       # 简短
    DEFAULT = 2     # 默认
    LONG = 3        # 较长
```

### 视频生成

```python
class VideoFormat(Enum):
    EXPLAINER = 1   # 解说
    BRIEF = 2       # 简报

class VideoStyle(Enum):
    AUTO_SELECT = 1  # 自动选择
    CUSTOM = 2       # 自定义
    CLASSIC = 3      # 经典
    WHITEBOARD = 4   # 白板
    KAWAII = 5       # 可爱风
    ANIME = 6        # 动漫风
    WATERCOLOR = 7   # 水彩风
    RETRO_PRINT = 8  # 复古印刷
    HERITAGE = 9     # 传统风
    PAPER_CRAFT = 10 # 纸艺风
```

### 测验/闪卡

```python
class QuizQuantity(Enum):
    FEWER = 1       # 较少
    STANDARD = 2    # 标准

class QuizDifficulty(Enum):
    EASY = 1        # 简单
    MEDIUM = 2      # 中等
    HARD = 3        # 困难
```

### 报告

```python
class ReportFormat(Enum):
    BRIEFING_DOC = 1  # 简报文档
    STUDY_GUIDE = 2   # 学习指南
    BLOG_POST = 3     # 博客文章
    CUSTOM = 4        # 自定义
```

### 信息图

```python
class InfographicOrientation(Enum):
    LANDSCAPE = 1   # 横向
    PORTRAIT = 2    # 纵向
    SQUARE = 3      # 正方形

class InfographicDetail(Enum):
    CONCISE = 1     # 简洁
    STANDARD = 2    # 标准
    DETAILED = 3    # 详细
```

### 幻灯片

```python
class SlideDeckFormat(Enum):
    DETAILED_DECK = 1     # 详细幻灯片
    PRESENTER_SLIDES = 2  # 演讲者幻灯片

class SlideDeckLength(Enum):
    DEFAULT = 1     # 默认
    SHORT = 2       # 简短
```

### 导出

```python
class ExportType(Enum):
    DOCS = 1    # 导出到 Google Docs
    SHEETS = 2  # 导出到 Google Sheets
```

### 对话配置

```python
class ChatGoal(Enum):
    DEFAULT = 1        # 通用
    CUSTOM = 2         # 使用 custom_prompt
    LEARNING_GUIDE = 3 # 教育导向

class ChatResponseLength(Enum):
    DEFAULT = 1   # 默认
    LONGER = 4    # 较长
    SHORTER = 5   # 较短

class ChatMode(Enum):
    """常用场景的预定义对话模式（服务级枚举）。"""
    DEFAULT = "default"              # 通用
    LEARNING_GUIDE = "learning_guide"  # 教育导向
    CONCISE = "concise"              # 简洁回复
    DETAILED = "detailed"            # 详细回复
```

**ChatGoal 与 ChatMode 的区别：**
- `ChatGoal` 是 RPC 级枚举，与 `client.chat.configure()` 一起用于底层 API 配置
- `ChatMode` 是服务级枚举，为常用场景提供预定义配置

---

## 高级用法

### 自定义 RPC 调用

对于未文档化的功能，你可以进行原始 RPC 调用：

```python
from notebooklm.rpc import RPCMethod

async with await NotebookLMClient.from_storage() as client:
    # 访问核心客户端进行原始 RPC
    result = await client._core.rpc_call(
        RPCMethod.SOME_METHOD,
        params=[...],
        source_path="/notebook/123"
    )
```

### 处理速率限制

Google 会限制激进的 API 使用：

```python
import asyncio
from notebooklm import RPCError

async def safe_create_notebooks(client, titles):
    for title in titles:
        try:
            await client.notebooks.create(title)
        except RPCError:
            # 速率限制时等待后重试
            await asyncio.sleep(10)
            await client.notebooks.create(title)
        # 操作之间添加延迟
        await asyncio.sleep(2)
```

### 流式对话响应

对话端点支持流式传输（内部实现）：

```python
# 标准（非流式）- 推荐
result = await client.chat.ask(nb_id, "问题")
print(result.answer)

# 流式传输由库内部处理
# ask() 方法返回完整响应
```
