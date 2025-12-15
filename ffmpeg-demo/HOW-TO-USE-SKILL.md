# How to Use the FFmpeg Skill

This guide explains how to use the FFmpeg skill with Claude Code.

## Skill Location

```
C:\Users\zdhpe\Desktop\Gemini 3\.claude\skills\ffmpeg-skill\Skill.md
```

## Setup

1. The skill is already installed in `.claude/skills/ffmpeg-skill/`
2. Restart Claude Code to load the skill
3. Start asking FFmpeg-related questions naturally

---

## Demo Files Available

| File | Size | Description |
|------|------|-------------|
| test_video.mp4 | 82KB | Original 5-second test video (640x480, 30fps) |
| extracted_audio.aac | 47KB | Audio extracted from video |
| resized_320.mp4 | 74KB | Video resized to 320px width |
| output.gif | 333KB | Animated GIF from video |
| trimmed.mp4 | 53KB | Video trimmed to 2 seconds |
| frame_01.png - frame_05.png | ~35KB each | Extracted frames (1 per second) |
| output.webm | 112KB | WebM format conversion |
| fast_2x.mp4 | 52KB | Video at 2x speed |
| faded_video.mp4 | 90KB | Video with fade in/out effects |
| slideshow.mp4 | 15KB | Video created from PNG frames |
| subtitles.srt | 1KB | Sample subtitle file |
| video_with_subs.mp4 | 92KB | Video with burned-in subtitles |
| video_with_text.mp4 | 84KB | Video with centered text overlay |
| video_text_bottom.mp4 | 84KB | Video with bottom text |
| video_with_timestamp.mp4 | 94KB | Video with timestamp overlay |
| video_timed_text.mp4 | 85KB | Video with timed text (appears 1s-3s) |

---

## Example Prompts to Use with Claude

### Basic Operations

| What You Want | What to Say |
|---------------|-------------|
| Convert format | "Convert test_video.mp4 to AVI" |
| Extract audio | "Extract audio from test_video.mp4 as MP3" |
| Resize video | "Resize test_video.mp4 to 1280x720" |
| Compress video | "Compress test_video.mp4 to reduce file size" |
| Get file info | "Show me info about test_video.mp4" |

### Cutting & Trimming

| What You Want | What to Say |
|---------------|-------------|
| Trim video | "Trim test_video.mp4 from 1 second to 3 seconds" |
| Cut first 2 seconds | "Cut the first 2 seconds from test_video.mp4" |
| Extract last 3 seconds | "Get the last 3 seconds of test_video.mp4" |

### Effects & Filters

| What You Want | What to Say |
|---------------|-------------|
| Add fade effects | "Add fade in and fade out to test_video.mp4" |
| Speed up | "Make test_video.mp4 play at 2x speed" |
| Slow down | "Slow down test_video.mp4 to half speed" |
| Rotate video | "Rotate test_video.mp4 90 degrees clockwise" |
| Flip video | "Flip test_video.mp4 horizontally" |
| Crop video | "Crop test_video.mp4 to 400x300 from center" |

### GIF & Images

| What You Want | What to Say |
|---------------|-------------|
| Create GIF | "Create a GIF from test_video.mp4" |
| Extract frames | "Extract 1 frame per second from test_video.mp4" |
| Single frame | "Extract a frame at 2 seconds from test_video.mp4" |
| Images to video | "Create a video from frame_01.png to frame_05.png" |

### Audio Operations

| What You Want | What to Say |
|---------------|-------------|
| Remove audio | "Remove audio from test_video.mp4" |
| Replace audio | "Replace audio in test_video.mp4 with extracted_audio.aac" |
| Adjust volume | "Increase volume of test_video.mp4 by 50%" |
| Normalize audio | "Normalize audio levels in test_video.mp4" |

### Subtitles & Text

| What You Want | What to Say |
|---------------|-------------|
| Add subtitles | "Add subtitles.srt to test_video.mp4" |
| Add text | "Add text 'Hello World' to center of test_video.mp4" |
| Bottom text | "Add text at bottom of test_video.mp4" |
| Timestamp | "Add a timestamp overlay to test_video.mp4" |
| Timed text | "Add text that appears from 1 to 3 seconds" |

### Advanced Operations

| What You Want | What to Say |
|---------------|-------------|
| Add watermark | "Add a watermark image to test_video.mp4" |
| Overlay videos | "Overlay resized_320.mp4 on top of test_video.mp4" |
| Concatenate | "Merge test_video.mp4 and trimmed.mp4 together" |
| Two-pass encode | "Compress test_video.mp4 using two-pass encoding at 500kbps" |

---

## Quick Command Reference

### Format Conversion
```bash
ffmpeg -i test_video.mp4 output.avi
ffmpeg -i test_video.mp4 -c:v libvpx-vp9 -c:a libopus output.webm
```

### Extract Audio
```bash
ffmpeg -i test_video.mp4 -vn -c:a copy audio.aac
ffmpeg -i test_video.mp4 -vn -c:a libmp3lame -b:a 192k audio.mp3
```

### Resize
```bash
ffmpeg -i test_video.mp4 -vf scale=1280:720 resized.mp4
ffmpeg -i test_video.mp4 -vf scale=640:-1 resized.mp4
```

### Trim/Cut
```bash
ffmpeg -ss 00:00:01 -i test_video.mp4 -t 00:00:02 -c copy trimmed.mp4
ffmpeg -ss 00:00:01 -i test_video.mp4 -to 00:00:03 -c copy trimmed.mp4
```

### Create GIF
```bash
ffmpeg -i test_video.mp4 -vf "fps=10,scale=320:-1" output.gif
```

### Extract Frames
```bash
ffmpeg -i test_video.mp4 -vf fps=1 frame_%02d.png
ffmpeg -ss 00:00:02 -i test_video.mp4 -frames:v 1 single_frame.png
```

### Images to Video
```bash
ffmpeg -framerate 2 -i frame_%02d.png -c:v libx264 -pix_fmt yuv420p slideshow.mp4
```

### Speed Change
```bash
# 2x speed
ffmpeg -i test_video.mp4 -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]" fast.mp4

# 0.5x speed (slow motion)
ffmpeg -i test_video.mp4 -filter_complex "[0:v]setpts=2.0*PTS[v];[0:a]atempo=0.5[a]" -map "[v]" -map "[a]" slow.mp4
```

### Fade Effects
```bash
ffmpeg -i test_video.mp4 -vf "fade=t=in:st=0:d=1,fade=t=out:st=4:d=1" faded.mp4
```

### Rotate
```bash
ffmpeg -i test_video.mp4 -vf "transpose=1" rotated_90cw.mp4
ffmpeg -i test_video.mp4 -vf "transpose=2" rotated_90ccw.mp4
```

### Compress
```bash
ffmpeg -i test_video.mp4 -c:v libx264 -crf 23 -c:a copy compressed.mp4
```

### Subtitles (Burn-in / Hardcode)
```bash
# Burn SRT subtitles into video
ffmpeg -i test_video.mp4 -vf "subtitles=subtitles.srt" video_with_subs.mp4

# With custom font size and color
ffmpeg -i test_video.mp4 -vf "subtitles=subtitles.srt:force_style='FontSize=24,PrimaryColour=&HFFFFFF&'" output.mp4
```

### Subtitles (Soft / Embedded Stream)
```bash
# Add subtitle as separate stream (can be toggled on/off in player)
ffmpeg -i test_video.mp4 -i subtitles.srt -c copy -c:s mov_text output.mp4

# Extract subtitles from video
ffmpeg -i input.mkv -map 0:s:0 output.srt
```

### Text Overlay (drawtext filter)
```bash
# Centered text with background box
ffmpeg -i test_video.mp4 -vf "drawtext=text='Hello World':fontsize=36:fontcolor=white:x=(w-text_w)/2:y=(h-text_h)/2:box=1:boxcolor=black@0.5" output.mp4

# Text at bottom center
ffmpeg -i test_video.mp4 -vf "drawtext=text='Bottom Text':fontsize=24:fontcolor=yellow:x=(w-text_w)/2:y=h-th-20:box=1:boxcolor=black@0.7" output.mp4

# Text at top-left corner
ffmpeg -i test_video.mp4 -vf "drawtext=text='Top Left':fontsize=24:fontcolor=white:x=10:y=10" output.mp4

# Text at top-right corner
ffmpeg -i test_video.mp4 -vf "drawtext=text='Top Right':fontsize=24:fontcolor=white:x=w-tw-10:y=10" output.mp4

# Timestamp overlay
ffmpeg -i test_video.mp4 -vf "drawtext=text='%{pts\\:hms}':fontsize=24:fontcolor=white:x=10:y=10:box=1:boxcolor=black@0.5" output.mp4

# Text that appears only between 1s and 3s
ffmpeg -i test_video.mp4 -vf "drawtext=text='Timed Text':fontsize=32:fontcolor=red:x=(w-text_w)/2:y=50:enable='between(t,1,3)'" output.mp4

# Multiple text lines
ffmpeg -i test_video.mp4 -vf "drawtext=text='Line 1':fontsize=24:fontcolor=white:x=10:y=10,drawtext=text='Line 2':fontsize=24:fontcolor=white:x=10:y=40" output.mp4
```

### Text Position Reference
```
x=10            # 10 pixels from left
x=w-tw-10       # 10 pixels from right (w=video width, tw=text width)
x=(w-text_w)/2  # Centered horizontally

y=10            # 10 pixels from top
y=h-th-10       # 10 pixels from bottom (h=video height, th=text height)
y=(h-text_h)/2  # Centered vertically
```

---

## Tips

1. **Use `-y`** to overwrite output files without asking
2. **Use `-c copy`** for fast operations without re-encoding
3. **Place `-ss` before `-i`** for fast seeking
4. **Use `-hide_banner`** for cleaner output
5. **Check file info** with `ffprobe test_video.mp4`

---

## File Paths

Working directory:
```
C:\Users\zdhpe\Desktop\Gemini 3\ffmpeg-demo\
```

Skill location:
```
C:\Users\zdhpe\Desktop\Gemini 3\.claude\skills\ffmpeg-skill\Skill.md
```
