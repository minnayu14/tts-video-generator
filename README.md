# TTS Video Generator

一个 Python 工具，自动生成带 AI 配音和同步字幕的短视频。适用于制作知识科普、社交媒体视频（TikTok/抖音/Reels/Shorts）和旁白式幻灯片。

## 工作流程

```
Word 文档 (.docx)  →  文本提取  →  ElevenLabs TTS（带时间戳）
     ↓                                    ↓
  每期标题                          音频 + 字符级时间戳
     ↓                                    ↓
背景图片  +  ASS 字幕  +  BGM  →  FFmpeg  →  最终 MP4 视频
```

## 功能特点

- **AI 语音合成**：使用 ElevenLabs API，支持多种语言的自然语音
- **精准字幕**：字符级时间戳确保字幕完美同步
- **智能换行**：自动在标点处断行，符合排版规则
- **背景音乐**：随机选择 BGM，可配置混音音量
- **批量处理**：支持单期生成或批量生成
- **缓存机制**：音频和时间戳数据会缓存，避免重复调用 API
- **全面可配**：所有设置（字体、分辨率、标签、路径）通过 `.env` 配置

## 环境要求

- Python 3.9+
- FFmpeg（需支持 libass 字幕渲染）
- [ElevenLabs](https://elevenlabs.io/) API Key

## 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/minnayu14/tts-video-generator.git
cd tts-video-generator

# 2. 安装依赖
pip install -r requirements.txt

# 3. 配置
cp .env.example .env
# 编辑 .env，填入你的 ElevenLabs API Key 和 Voice ID

# 4. 准备内容
# - 创建 Word 文档（scripts.docx），按格式写入脚本
# - 将背景图片放入 backgrounds/ 目录
# - （可选）将 BGM 音频放入 bgm/ 目录

# 5. 生成视频
python video_generate.py --single 1    # 生成单期
python video_generate.py --batch 10    # 批量生成（最多10期）
python video_generate.py               # 生成全部
```

## Word 文档格式

`.docx` 中每期的格式：

```
1 这里是标题
【Text】正文内容。第二句。第三句。

2 另一期标题
【Text】更多正文内容。更多句子。
```

- **标题行**：以期号开头，后接标题文字
- **正文行**：包含文本标签（默认 `【Text】`），后面是正文
- 可通过 `.env` 中的 `TEXT_TAG` 自定义标签（如 `TEXT_TAG=【正文日语】`）

## 常用命令

| 命令 | 说明 |
|------|------|
| `python video_generate.py` | 生成所有未完成的期数 |
| `python video_generate.py --single <N>` | 只生成第 N 期（强制重新生成） |
| `python video_generate.py --batch <N>` | 批量生成最多 N 期，完成后暂停 |
| `python video_generate.py --list-backgrounds` | 列出可用背景图片 |
| `python video_generate.py --preview-subs` | 预览字幕时间戳 |
| `python tts_generate.py` | 只生成 TTS 音频（不生成视频） |
| `python tts_generate.py --preview` | 预览提取的文本（不调用 API） |

## 配置说明

所有设置都在 `.env` 文件中，详见 `.env.example`。主要配置项：

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `ELEVENLABS_API_KEY` | （必填） | ElevenLabs API Key |
| `ELEVENLABS_VOICE_ID` | （必填） | 语音 ID |
| `TEXT_TAG` | `【Text】` | Word 文档中标记正文的标签 |
| `END_TAGS` | （空） | 结束提取的标签（逗号分隔） |
| `SUBTITLE_FONT` | `Arial` | 字幕字体 |
| `VIDEO_WIDTH` x `VIDEO_HEIGHT` | 1080x1920 | 输出视频分辨率 |
| `BGM_VOLUME` | 0.12 | BGM 音量（相对于语音） |
| `SENTENCE_SPLIT_PATTERN` | `(?<=[。！？!?.])` | 分句正则表达式 |

## 项目结构

```
tts-video-generator/
├── video_generate.py   # 主脚本：视频生成
├── tts_generate.py     # 独立 TTS 音频生成脚本
├── .env.example        # 配置模板
├── requirements.txt    # Python 依赖
├── skill/SKILL.md      # Claude Code Skill 定义
├── scripts.docx        # 你的脚本文档（需自行创建）
├── backgrounds/        # 背景图片（需自行创建）
├── bgm/                # 背景音乐（可选）
├── audio_output/       # 生成的音频 + 时间戳缓存（自动创建）
└── video_output/       # 最终输出的 MP4 视频（自动创建）
```

## 作为 Claude Code Skill 使用

如果你使用 [Claude Code](https://claude.com/claude-code)，可以把本项目作为 Skill 使用，Claude 会自动检查环境、引导配置、执行生成。

**安装方式**：

```bash
# 将 SKILL.md 复制到 Claude Code 的 commands 目录
mkdir -p ~/.claude/commands
cp skill/SKILL.md ~/.claude/commands/tts-video-generator.md
```

**使用方式**：在 Claude Code 对话中直接说：
- "帮我生成第5期视频"
- "批量生成视频"
- "只生成音频不要视频"

Claude 会自动检查 API Key、FFmpeg、依赖是否就绪，缺什么就引导你配置。

## 协议

MIT
