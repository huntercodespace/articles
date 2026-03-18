---
name: ytdlp
description: 使用 yt-dlp 下载视频并用 FFmpeg 进行后处理。支持 YouTube 等 1000+ 网站，下载后默认转码为 H.264 编码格式，支持字幕下载、视频剪辑、字幕烧录、音频提取等功能。适用场景：当用户需要下载在线视频、提取音频、下载/烧录字幕、剪辑视频片段时使用。
metadata:
  tags: 视频下载, yt-dlp, FFmpeg, H.264, 字幕, 视频剪辑
---

# yt-dlp 视频下载与处理技能

## 适用场景

当用户需要以下操作时，使用本技能：

- 从 YouTube 或其他视频网站下载视频
- 下载视频字幕（人工字幕或自动生成字幕）
- 剪辑视频片段
- 烧录字幕到视频
- 提取音频
- 视频格式转换或压缩

## 前置依赖

### 安装 yt-dlp

```bash
# Windows (推荐使用 pip)
pip install yt-dlp

# macOS
brew install yt-dlp

# Linux (Ubuntu/Debian)
sudo apt-get install yt-dlp
# 或
pip install yt-dlp
```

### 安装 FFmpeg

```bash
# Windows: 从 https://ffmpeg.org/download.html 下载并添加到 PATH

# macOS（推荐完整版，支持字幕烧录）
brew install ffmpeg-full

# Linux (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install ffmpeg libass-dev
```

### 验证安装

```bash
yt-dlp --version
ffmpeg -version

# 检查 libass 支持（字幕烧录必需）
ffmpeg -filters 2>&1 | grep subtitles
```

---

## 核心操作

### 1. 下载视频（默认 H.264 编码）

> [!IMPORTANT]
> 所有视频下载后，**必须默认转码为 H.264 (libx264) 编码格式**，以确保最大兼容性。

#### 命令行方式

```bash
# 下载最佳质量并转码为 H.264
yt-dlp -f "bestvideo[height<=1080]+bestaudio/best[height<=1080]/best" \
  --merge-output-format mp4 \
  --postprocessor-args "ffmpeg:-c:v libx264 -crf 23 -preset medium -c:a aac" \
  -o "%(title)s [%(id)s].%(ext)s" \
  URL

# 下载指定分辨率并转码为 H.264
yt-dlp -f "bestvideo[height<=720]+bestaudio/best[height<=720]/best" \
  --merge-output-format mp4 \
  --postprocessor-args "ffmpeg:-c:v libx264 -crf 23 -preset medium -c:a aac" \
  URL
```

#### Python API 方式

```python
import yt_dlp

ydl_opts = {
    # 视频格式：最高 1080p
    'format': 'bestvideo[height<=1080]+bestaudio/best[height<=1080]/best',

    # 输出格式与文件名
    'merge_output_format': 'mp4',
    'outtmpl': '%(title)s [%(id)s].%(ext)s',

    # 后处理：转码为 H.264
    'postprocessors': [{
        'key': 'FFmpegVideoConvertor',
        'preferedformat': 'mp4',
    }],
    'postprocessor_args': {
        'ffmpeg': ['-c:v', 'libx264', '-crf', '23', '-preset', 'medium', '-c:a', 'aac'],
    },
}

with yt_dlp.YoutubeDL(ydl_opts) as ydl:
    ydl.download(['URL'])
```

**参数说明**：
| 参数 | 说明 |
|------|------|
| `-c:v libx264` | 使用 H.264 视频编码器 |
| `-crf 23` | 质量控制（18=高质量大文件，23=平衡推荐，28=低质量小文件） |
| `-preset medium` | 编码速度（ultrafast/fast/medium/slow，越慢质量越好文件越小） |
| `-c:a aac` | 使用 AAC 音频编码器 |

### 2. 下载字幕

```bash
# 下载英文字幕
yt-dlp --write-sub --sub-lang en URL

# 下载自动生成字幕（如果没有人工字幕）
yt-dlp --write-auto-sub --sub-lang en URL

# 同时下载视频和字幕（H.264 编码）
yt-dlp -f "bestvideo[height<=1080]+bestaudio/best" \
  --merge-output-format mp4 \
  --postprocessor-args "ffmpeg:-c:v libx264 -crf 23 -c:a aac" \
  --write-sub --write-auto-sub --sub-lang en \
  --sub-format vtt \
  URL

# 查看可用字幕
yt-dlp --list-subs URL
```

**常用字幕语言代码**：

| 代码 | 语言 |
|------|------|
| `en` | 英文 |
| `zh-Hans` | 简体中文 |
| `zh-Hant` | 繁体中文 |
| `ja` | 日语 |
| `ko` | 韩语 |

### 3. 查看视频信息（不下载）

```bash
# 查看可用格式
yt-dlp -F URL

# 查看完整视频信息（JSON）
yt-dlp --print-json URL
```

```python
import yt_dlp

with yt_dlp.YoutubeDL({}) as ydl:
    info = ydl.extract_info(url, download=False)
    print(f"标题: {info['title']}")
    print(f"时长: {info['duration']} 秒")
    print(f"上传者: {info['uploader']}")
```

### 4. 下载播放列表

```bash
# 下载整个播放列表（H.264 编码）
yt-dlp --merge-output-format mp4 \
  --postprocessor-args "ffmpeg:-c:v libx264 -crf 23 -c:a aac" \
  PLAYLIST_URL

# 仅下载第 1-5 个视频
yt-dlp --playlist-items 1-5 \
  --merge-output-format mp4 \
  --postprocessor-args "ffmpeg:-c:v libx264 -crf 23 -c:a aac" \
  PLAYLIST_URL

# 不下载播放列表，仅当前视频
yt-dlp --no-playlist URL
```

### 5. 代理设置

```bash
# HTTP 代理
yt-dlp --proxy http://proxy:port URL

# SOCKS5 代理
yt-dlp --proxy socks5://proxy:port URL
```

### 6. 使用浏览器 Cookies（下载会员视频）

```bash
# 从浏览器导出 cookies
yt-dlp --cookies-from-browser chrome URL

# 使用 cookies 文件
yt-dlp --cookies cookies.txt URL
```

---

## FFmpeg 后处理

### 1. 视频剪辑

```bash
# 精确剪辑（从 30 秒开始，持续 60 秒，不重新编码）
ffmpeg -ss 30 -i input.mp4 -t 60 -c copy output.mp4

# 指定时间范围（从 01:30:00 到 01:33:15）
ffmpeg -ss 01:30:00 -i input.mp4 -to 01:33:15 -c copy output.mp4

# 剪辑并转码为 H.264
ffmpeg -ss 30 -i input.mp4 -t 60 \
  -c:v libx264 -crf 23 -c:a aac output.mp4
```

**参数说明**：
- `-ss`：起始时间
- `-t`：持续时间
- `-to`：结束时间
- `-c copy`：直接复制流（快速无损，但不重新编码）

### 2. 烧录字幕

```bash
# 烧录 SRT 字幕（会重新编码为 H.264）
ffmpeg -i input.mp4 \
  -vf "subtitles=subtitle.srt" \
  -c:v libx264 -crf 23 \
  -c:a copy \
  output.mp4

# 自定义字幕样式
ffmpeg -i input.mp4 \
  -vf "subtitles=subtitle.srt:force_style='FontSize=24,MarginV=30'" \
  -c:v libx264 -crf 23 \
  -c:a copy \
  output.mp4

# 烧录双语字幕
ffmpeg -i input.mp4 \
  -vf "subtitles=bilingual.srt:force_style='FontSize=24,MarginV=30'" \
  -c:v libx264 -crf 23 \
  -c:a copy \
  output.mp4
```

> [!WARNING]
> 烧录字幕需要 libass 支持。文件路径不能包含空格，否则会失败（可用临时目录解决）。

**可用字幕样式选项**：
| 选项 | 说明 | 推荐值 |
|------|------|--------|
| `FontSize` | 字体大小 | 20-28 |
| `MarginV` | 垂直边距 | 20-40 |
| `FontName` | 字体名称 | - |
| `PrimaryColour` | 主要颜色 | - |
| `OutlineColour` | 描边颜色 | - |
| `Bold` | 粗体 | 0 或 1 |

### 3. 提取音频

```bash
# 提取为 MP3
ffmpeg -i input.mp4 -vn -acodec libmp3lame -q:a 2 output.mp3

# 提取为 AAC（直接复制，不重新编码）
ffmpeg -i input.mp4 -vn -c:a copy output.aac
```

也可直接用 yt-dlp 下载音频：

```bash
yt-dlp -x --audio-format mp3 URL
```

### 4. 视频压缩（H.264）

```bash
ffmpeg -i input.mp4 \
  -c:v libx264 \
  -crf 23 \
  -preset medium \
  -c:a aac \
  output.mp4
```

### 5. 查看视频信息

```bash
ffmpeg -i input.mp4
ffprobe -v error -show_format -show_streams input.mp4
```

---

## 性能优化

### 硬件加速

```bash
# NVIDIA GPU
ffmpeg -hwaccel cuda -i input.mp4 -c:v libx264 -crf 23 -c:a aac output.mp4

# macOS (VideoToolbox)
ffmpeg -hwaccel videotoolbox -i input.mp4 -c:v libx264 -crf 23 -c:a aac output.mp4
```

### 多线程

```bash
ffmpeg -threads 4 -i input.mp4 -c:v libx264 -crf 23 -c:a aac output.mp4
```

### 下载速率限制

```bash
yt-dlp --rate-limit 4.2M URL
```

---

## 完整工作流示例

### 场景：下载视频 + 字幕，剪辑片段，烧录中英双语字幕

```bash
# Step 1: 下载视频和字幕（H.264 编码）
yt-dlp -f "bestvideo[height<=1080]+bestaudio/best" \
  --merge-output-format mp4 \
  --postprocessor-args "ffmpeg:-c:v libx264 -crf 23 -c:a aac" \
  --write-sub --write-auto-sub --sub-lang en \
  --sub-format vtt \
  -o "%(title)s.%(ext)s" \
  URL

# Step 2: 剪辑指定片段
ffmpeg -ss 00:05:30 -i "video.mp4" -to 00:08:45 \
  -c:v libx264 -crf 23 -c:a aac \
  clip.mp4

# Step 3: 烧录字幕到剪辑片段
ffmpeg -i clip.mp4 \
  -vf "subtitles=bilingual.srt:force_style='FontSize=24,MarginV=30'" \
  -c:v libx264 -crf 23 \
  -c:a copy \
  final.mp4
```

---

## 常见问题

### Q: 下载失败，提示 "Video unavailable"
**A**: 视频可能已删除、私有或地区限制。尝试使用 `--proxy` 或 `--cookies-from-browser chrome`。

### Q: 字幕下载失败
**A**: 使用 `--write-auto-sub` 下载自动生成字幕，或 `--list-subs` 查看可用字幕。

### Q: 字幕烧录失败，提示 "No such filter: 'subtitles'"
**A**: FFmpeg 缺少 libass 支持。macOS 需安装 `ffmpeg-full`，Linux 需安装 `libass-dev`。

### Q: 路径包含空格导致字幕烧录失败
**A**: 将文件复制到无空格的临时目录再处理。

### Q: 视频文件名包含非法字符
**A**: 使用 `-o "%(title).100s.%(ext)s"` 限制标题长度。

### Q: 下载速度慢
**A**: 使用代理 `--proxy`，或等待后重试（YouTube 可能限速）。

### Q: 视频质量不佳
**A**: 降低 CRF 值（如 `-crf 18`），或使用 `-preset slow` 提高编码质量。

---

## 支持的网站

yt-dlp 支持 **1000+** 网站，包括：YouTube、Vimeo、Twitter、TikTok、Bilibili 等。

```bash
# 查看完整列表
yt-dlp --list-extractors
```

## 参考链接

- [yt-dlp GitHub](https://github.com/yt-dlp/yt-dlp)
- [FFmpeg 官方文档](https://ffmpeg.org/documentation.html)
- [FFmpeg Subtitles 滤镜](https://ffmpeg.org/ffmpeg-filters.html#subtitles)
- [yt-dlp 格式选择说明](https://github.com/yt-dlp/yt-dlp#format-selection)
