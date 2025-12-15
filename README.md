# FFmpeg Skill for Claude Code

A comprehensive FFmpeg media processing skill for [Claude Code](https://claude.com/claude-code), with demo files and documentation.

## What's Included

```
├── .claude/
│   └── skills/
│       └── ffmpeg-skill/
│           └── Skill.md          # 900+ lines of FFmpeg documentation
│
└── ffmpeg-demo/
    ├── HOW-TO-USE-SKILL.md       # Usage guide
    ├── test_video.mp4            # Sample video for testing
    └── [demo output files]       # Generated examples
```

## Features

The FFmpeg skill covers:

| Category | Features |
|----------|----------|
| **Core Concepts** | Video, Audio, Codecs, Containers explained |
| **Processing** | Transmuxing, Transcoding, Transrating, Transsizing |
| **Timing** | PTS, DTS, Timebase synchronization |
| **Formats** | MP4, MKV, WebM, AVI, MOV, TS conversions |
| **Compression** | CRF, Two-pass encoding, Bitrate control |
| **Filters** | Scale, Crop, Rotate, Overlay, Fade, Speed |
| **Text/Subtitles** | Burn-in, Soft subs, drawtext overlay |
| **Streaming** | HLS, DASH, RTMP, Fragmented MP4 |
| **Advanced** | Complex filtergraphs, Batch processing |

## Installation

### Option 1: Project-level (this repo only)
```bash
git clone https://github.com/donghaozhang/ffmpeg-claude-skill.git
cd ffmpeg-claude-skill
# Start Claude Code in this directory
```

### Option 2: Global (all projects)
```bash
# Copy to your global Claude skills folder
cp -r .claude/skills/ffmpeg-skill ~/.claude/skills/
```

## Usage

After installation, restart Claude Code and ask naturally:

```
"Convert video.mp4 to WebM"
"Extract audio from movie.mkv"
"Create a GIF from the first 5 seconds"
"Add subtitles to my video"
"Compress this video for web"
"Speed up the video 2x"
"Add text overlay saying 'Hello World'"
```

## Demo Files

| File | Description |
|------|-------------|
| `test_video.mp4` | 5-second test video (640x480, 30fps) |
| `extracted_audio.aac` | Audio extracted from video |
| `resized_320.mp4` | Video resized to 320px width |
| `output.gif` | Animated GIF |
| `output.webm` | WebM format conversion |
| `trimmed.mp4` | Trimmed to 2 seconds |
| `fast_2x.mp4` | 2x speed |
| `faded_video.mp4` | Fade in/out effects |
| `slideshow.mp4` | Video from PNG frames |
| `video_with_subs.mp4` | Burned-in subtitles |
| `video_with_text.mp4` | Text overlay |
| `video_with_timestamp.mp4` | Timestamp overlay |
| `frame_01-05.png` | Extracted frames |

## Quick Reference

### Format Conversion
```bash
ffmpeg -i input.mkv output.mp4
ffmpeg -i input.mp4 -c:v libvpx-vp9 -c:a libopus output.webm
```

### Extract Audio
```bash
ffmpeg -i video.mp4 -vn -c:a copy audio.aac
ffmpeg -i video.mp4 -vn -c:a libmp3lame -b:a 192k audio.mp3
```

### Resize
```bash
ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4
ffmpeg -i input.mp4 -vf scale=640:-1 output.mp4  # auto height
```

### Trim/Cut
```bash
ffmpeg -ss 00:01:00 -i input.mp4 -t 00:00:30 -c copy output.mp4
```

### Create GIF
```bash
ffmpeg -i input.mp4 -vf "fps=10,scale=320:-1" output.gif
```

### Add Text
```bash
ffmpeg -i input.mp4 -vf "drawtext=text='Hello':fontsize=36:fontcolor=white:x=(w-text_w)/2:y=(h-text_h)/2" output.mp4
```

### Burn Subtitles
```bash
ffmpeg -i input.mp4 -vf "subtitles=subs.srt" output.mp4
```

### Compress
```bash
ffmpeg -i input.mp4 -c:v libx264 -crf 23 -c:a copy output.mp4
```

## Requirements

- [FFmpeg](https://ffmpeg.org/download.html) installed and in PATH
- [Claude Code](https://claude.com/claude-code)

## License

MIT

---

Generated with [Claude Code](https://claude.com/claude-code)
