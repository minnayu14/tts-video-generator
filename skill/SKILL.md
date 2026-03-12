---
name: tts-video-generator
description: 当用户提到"生成视频"、"生成音频"、"TTS视频"、"字幕视频"、"批量生成短视频"、"ElevenLabs配音"、"视频合成"时触发此技能。用于从Word文档自动生成带AI配音和同步字幕的短视频。
version: 1.0.0
---

# TTS Video Generator Skill

从 Word 文档自动生成带 AI 配音 + 精准同步字幕的短视频（竖屏，适用于 TikTok/Reels/Shorts）。

## 工作流程

```
Word 文档 → 文本提取 → ElevenLabs TTS（带时间戳） → ASS 字幕 → FFmpeg 合成 → MP4
```

## 环境检查（每次执行前必须）

在执行任何生成操作之前，**必须按顺序检查以下依赖**，缺什么就引导用户配置：

### 1. 检查项目路径

确认 `video_generate.py` 和 `tts_generate.py` 所在目录。如果用户没指定，询问项目路径。

### 2. 检查 Python 依赖

```bash
pip list 2>/dev/null | grep -iE "python-docx|requests|python-dotenv"
```

如果缺少依赖，提示用户：
```bash
pip install -r requirements.txt
```

### 3. 检查 FFmpeg

```bash
ffmpeg -version 2>/dev/null | head -1
```

如果未安装，根据系统提示：
- macOS: `brew install ffmpeg`
- Ubuntu: `sudo apt install ffmpeg`
- Windows: 从 https://ffmpeg.org/download.html 下载

**注意**：FFmpeg 需要支持 `libass`（字幕渲染），安装完整版：
- macOS: `brew install ffmpeg` （默认包含）
- Ubuntu: `sudo apt install ffmpeg libavcodec-extra`

### 4. 检查 .env 配置

```bash
cat .env 2>/dev/null
```

如果 `.env` 不存在或关键字段为空，引导用户：

```bash
cp .env.example .env
```

然后提示用户填写以下**必填项**：

| 配置项 | 说明 | 获取方式 |
|--------|------|----------|
| `ELEVENLABS_API_KEY` | ElevenLabs API 密钥 | https://elevenlabs.io → Profile → API Keys |
| `ELEVENLABS_VOICE_ID` | 语音角色 ID | https://elevenlabs.io → Voices → 选择语音 → 复制 Voice ID |

其他配置项都有默认值，可以不改。

### 5. 检查素材目录

```bash
ls backgrounds/   # 至少需要1张背景图片
ls bgm/           # 可选，BGM音乐文件
ls scripts.docx   # Word脚本文件（或 .env 中 DOCX_PATH 指定的路径）
```

如果缺少，提示用户准备素材：
- `backgrounds/` — 放入 JPG/PNG 背景图片（会自动轮流分配给各期）
- `bgm/` — 放入背景音乐文件（每次随机选一首，可选）
- Word 文档 — 按下方格式编写视频脚本

## Word 文档格式

用户的脚本文档必须按以下格式编写：

```
1 这是第一期的标题
【Text】这是第一期的正文内容。可以有多句话。每句用句号分隔。

2 这是第二期的标题
【Text】第二期的正文。系统会按句号自动分句生成字幕。
```

**说明**：
- 每期以数字开头，后接标题
- 正文用 `【Text】` 标签标记（可通过 `.env` 中 `TEXT_TAG` 自定义）
- 如有多语言内容，可用 `END_TAGS` 指定结束标签（如 `【Translation】,【Tags】`）

## 可用命令

### 生成视频

```bash
# 生成所有未完成的期
python video_generate.py

# 只生成第5期（强制重新生成）
python video_generate.py --single 5

# 批量生成10期后暂停
python video_generate.py --batch 10
```

### 只生成音频（不合成视频）

```bash
# 预览模式（不调用API，只显示会生成什么）
python tts_generate.py --preview

# 正式生成音频
python tts_generate.py
```

### 辅助命令

```bash
# 查看可用的背景图片
python video_generate.py --list-backgrounds

# 预览字幕时间戳
python video_generate.py --preview-subs
```

## 常见问题处理

### API 调用失败
- 检查 `ELEVENLABS_API_KEY` 是否正确
- 检查 API 余额是否充足（https://elevenlabs.io → Subscription）
- 如果是网络问题，重试即可（已有缓存的期不会重复调用）

### 字幕不显示
- 确认 FFmpeg 支持 libass：`ffmpeg -filters | grep ass`
- 检查字体是否安装（默认 Arial，可在 `.env` 中改为系统可用字体）

### 重新生成某一期
- 删除 `audio_output/<期号>_timing.json` 可重新调用 TTS
- 使用 `--single <期号>` 强制重新合成视频

## 关键配置说明

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `VIDEO_WIDTH` / `VIDEO_HEIGHT` | 1080 × 1920 | 竖屏分辨率 |
| `BGM_VOLUME` | 0.12 | BGM相对语音的音量（12%） |
| `SUB_FONTSIZE` | 48 | 字幕字号 |
| `TITLE_FONTSIZE` | 60 | 标题字号 |
| `SUB_MAX_LINE_CHARS` | 16 | 字幕每行最大字符数 |
| `SUBTITLE_FONT` | Arial | 字幕字体 |
| `SENTENCE_SPLIT_PATTERN` | `(?<=[。！？!?.])` | 分句正则表达式 |
| `ELEVENLABS_MODEL_ID` | eleven_turbo_v2_5 | TTS模型（支持多语言） |
