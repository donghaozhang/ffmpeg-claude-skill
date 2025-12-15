---
name: FFmpeg Media Processing
description: Process, convert, and manipulate audio and video files using FFmpeg commands
dependencies: ffmpeg
---

# FFmpeg Media Processing Skill

FFmpeg is a universal media converter that reads from multiple inputs, applies filters/transformations, and writes to multiple outputs.

---

## Core Concepts

### Video
A sequence of images (frames) displayed at a specific rate. Standard rates:
- **24 fps**: Cinema/film
- **30 fps**: TV/web (NTSC)
- **25 fps**: TV (PAL)
- **60 fps**: Gaming/smooth motion

Raw 1080p at 24fps for 30 minutes = ~250GB uncompressed. Codecs make this practical.

### Audio
Sound waves digitized via Analog-to-Digital Conversion (ADC). Key parameters:
- **Sample Rate**: 44100 Hz (CD), 48000 Hz (video standard)
- **Bit Depth**: 16-bit, 24-bit
- **Channels**: Mono (1), Stereo (2), 5.1 (6), 7.1 (8)

### Codec (Coder-Decoder)
Compresses/decompresses media data. Common codecs:

| Type | Codec | Description |
|------|-------|-------------|
| Video | H.264/AVC | Most compatible, good quality |
| Video | H.265/HEVC | 50% smaller than H.264, slower encode |
| Video | VP9 | Google's open codec, WebM |
| Video | AV1 | Newest, best compression, slow encode |
| Audio | AAC | Standard for MP4, good quality |
| Audio | MP3 | Legacy, universal support |
| Audio | Opus | Best quality at low bitrates |
| Audio | FLAC | Lossless compression |

### Container (Format)
Wrapper holding compressed streams + metadata. Container ≠ Codec!

| Extension | Container | Typical Codecs |
|-----------|-----------|----------------|
| .mp4 | MPEG-4 Part 14 | H.264/H.265 + AAC |
| .mkv | Matroska | Any codec, very flexible |
| .webm | WebM | VP8/VP9 + Vorbis/Opus |
| .avi | AVI | Legacy, limited |
| .mov | QuickTime | H.264 + AAC |
| .ts | MPEG-TS | H.264 + AAC (streaming) |

---

## FFmpeg Processing Pipeline

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  INPUT      │    │  DEMUX      │    │  DECODE     │    │  FILTER     │    │  ENCODE     │
│  Protocol   │───▶│  Extract    │───▶│  Decompress │───▶│  Transform  │───▶│  Compress   │
│  file/http  │    │  Streams    │    │  to Raw     │    │  (optional) │    │  (optional) │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                                                    │
                                                                                    ▼
                                                         ┌─────────────┐    ┌─────────────┐
                                                         │  OUTPUT     │◀───│  MUX        │
                                                         │  Protocol   │    │  Combine    │
                                                         │  file/http  │    │  Streams    │
                                                         └─────────────┘    └─────────────┘
```

### Processing Modes

| Mode | Description | Command | Use Case |
|------|-------------|---------|----------|
| **Transmuxing** | Change container only | `-c copy` | MP4 → MKV (fast) |
| **Transcoding** | Change codec | `-c:v libx264` | H.265 → H.264 |
| **Transrating** | Change bitrate | `-b:v 1M` | Reduce file size |
| **Transsizing** | Change resolution | `-vf scale=1280:720` | 4K → 1080p |

```bash
# Transmuxing (container change only, no re-encode)
ffmpeg -i input.mp4 -c copy output.ts

# Transcoding (codec change)
ffmpeg -i input.mp4 -c:v libx265 output.mp4

# Transrating (bitrate change)
ffmpeg -i input.mp4 -b:v 1M -b:a 128k output.mp4

# Transsizing (resolution change)
ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4
```

---

## Timing & Synchronization

### Timestamps

| Term | Meaning |
|------|---------|
| **PTS** (Presentation Timestamp) | When frame should be displayed |
| **DTS** (Decoding Timestamp) | When frame should be decoded |
| **Timebase** | Unit of time measurement (e.g., 1/90000) |

DTS ≠ PTS when B-frames are used (frames decoded out of display order).

### Timestamp Example (60fps, timebase 1/60000)
```
Frame 0: PTS = 0      → 0.000s
Frame 1: PTS = 1000   → 0.0167s
Frame 2: PTS = 2000   → 0.0333s
```

### Fix Timestamp Issues
```bash
# Reset timestamps (fix sync issues)
ffmpeg -i input.mp4 -fflags +genpts -c copy output.mp4

# Force constant frame rate
ffmpeg -i input.mp4 -vsync cfr -r 30 output.mp4

# Copy timestamps from source
ffmpeg -i input.mp4 -copyts -c copy output.mp4
```

---

## Synopsis

```
ffmpeg [global_options] {[input_file_options] -i input_url} ... {[output_file_options] output_url} ...
```

**IMPORTANT**: Order matters! Specify all input files first, then output files. Options apply to the next specified file and reset between files.

## Capabilities

- Convert video/audio formats (mp4, mkv, avi, webm, mp3, wav, flac, etc.)
- Extract audio from video files
- Compress and resize videos
- Trim and cut media files
- Merge/concatenate multiple files
- Add/extract subtitles
- Apply video and audio filters
- Stream copy without re-encoding
- Two-pass encoding for optimal quality
- Hardware acceleration
- Screen/desktop capture
- Create GIFs and image sequences

---

## Main Options Reference

| Option | Purpose |
|--------|---------|
| `-i url` | Specify input file |
| `-f fmt` | Force input/output format |
| `-c[:stream] codec` | Select encoder/decoder (or `copy` for stream copy) |
| `-t duration` | Limit output duration |
| `-ss position` | Seek to position (before `-i` = fast seek, after = accurate) |
| `-to position` | Stop writing at position |
| `-y` | Overwrite output files without asking |
| `-n` | Never overwrite output files |
| `-map input:stream` | Manually select streams |

## Video Options

| Option | Purpose |
|--------|---------|
| `-r fps` | Set frame rate |
| `-s WxH` | Set frame size (e.g., 1920x1080) |
| `-aspect ratio` | Set display aspect ratio (e.g., 16:9) |
| `-vf filtergraph` | Apply video filter chain |
| `-vn` | Disable video output |
| `-vcodec codec` | Set video codec (same as `-c:v`) |
| `-b:v bitrate` | Set video bitrate |
| `-crf value` | Constant Rate Factor (quality, 0-51, lower=better) |
| `-pix_fmt format` | Set pixel format |
| `-frames:v number` | Output specific number of video frames |

## Audio Options

| Option | Purpose |
|--------|---------|
| `-ar freq` | Set audio sampling frequency (e.g., 44100, 48000) |
| `-ac channels` | Set number of audio channels |
| `-af filtergraph` | Apply audio filter chain |
| `-an` | Disable audio output |
| `-acodec codec` | Set audio codec (same as `-c:a`) |
| `-b:a bitrate` | Set audio bitrate |
| `-sample_fmt format` | Set audio sample format |

## Subtitle Options

| Option | Purpose |
|--------|---------|
| `-sn` | Disable subtitles |
| `-scodec codec` | Set subtitle codec |
| `-fix_sub_duration` | Fix overlapping subtitle durations |

## Global Options

| Option | Purpose |
|--------|---------|
| `-loglevel level` | Set logging level (quiet, panic, fatal, error, warning, info, verbose, debug) |
| `-hide_banner` | Suppress copyright/version banner |
| `-stats` | Show encoding progress statistics |
| `-benchmark` | Show performance metrics after completion |
| `-report` | Dump full command and logs to file |
| `-progress url` | Send progress info to URL or file |
| `-version` | Show version information |

---

## Stream Selection & Mapping

FFmpeg auto-selects one stream per type (highest resolution video, most channels audio, first subtitle).

### Manual Stream Selection with `-map`

```bash
# Stream specifier format: input_index:stream_type:stream_index
# Stream types: v=video, a=audio, s=subtitle, d=data, t=attachments

# Select all streams from input 0
ffmpeg -i input.mkv -map 0 output.mp4

# Select video from input 0, audio from input 1
ffmpeg -i video.mp4 -i audio.mp3 -map 0:v -map 1:a output.mp4

# Select second audio stream from first input
ffmpeg -i input.mkv -map 0:a:1 output.mp3

# Exclude a stream type (negative mapping)
ffmpeg -i input.mkv -map 0 -map -0:s output.mp4  # all except subtitles
```

---

## Transcoding vs Stream Copy

**Stream Copy** (`-c copy`): Fast, no quality loss, just repackages streams. Use for format conversion only.

**Transcoding**: Decode → Filter → Encode. Slower but enables filters and codec changes.

```bash
# Stream copy (fast, no re-encoding)
ffmpeg -i input.mkv -c copy output.mp4

# Transcode video, copy audio
ffmpeg -i input.mkv -c:v libx264 -c:a copy output.mp4

# Full transcode
ffmpeg -i input.mkv -c:v libx264 -c:a aac output.mp4
```

---

## Common Commands

### Format Conversion

```bash
# Basic format conversion (auto codec selection)
ffmpeg -i input.mkv output.mp4

# Specify codecs explicitly
ffmpeg -i input.avi -c:v libx264 -c:a aac output.mp4

# Convert to WebM
ffmpeg -i input.mp4 -c:v libvpx-vp9 -c:a libopus output.webm

# Audio format conversion
ffmpeg -i input.wav -c:a libmp3lame -b:a 320k output.mp3
ffmpeg -i input.mp3 -c:a flac output.flac
```

### Extract Audio from Video

```bash
# Stream copy (fastest, keeps original codec)
ffmpeg -i video.mp4 -vn -c:a copy audio.aac

# Re-encode to MP3
ffmpeg -i video.mp4 -vn -c:a libmp3lame -b:a 192k audio.mp3

# Extract to WAV (uncompressed)
ffmpeg -i video.mp4 -vn audio.wav
```

### Compress Video

```bash
# CRF mode (constant quality, 18-28 recommended, lower=better)
ffmpeg -i input.mp4 -c:v libx264 -crf 23 -c:a copy output.mp4

# Two-pass encoding (best quality/size ratio)
ffmpeg -i input.mp4 -c:v libx264 -b:v 1M -pass 1 -an -f null NUL
ffmpeg -i input.mp4 -c:v libx264 -b:v 1M -pass 2 -c:a aac output.mp4

# Constrained bitrate
ffmpeg -i input.mp4 -c:v libx264 -b:v 1M -maxrate 1.5M -bufsize 2M output.mp4
```

### Trim/Cut Media

```bash
# Fast seek (before -i) + accurate cut
ffmpeg -ss 00:01:00 -i input.mp4 -t 00:00:30 -c copy output.mp4

# Accurate seek (after -i, slower but frame-accurate)
ffmpeg -i input.mp4 -ss 00:01:00 -t 00:00:30 output.mp4

# Cut from start time to end time
ffmpeg -ss 00:01:00 -i input.mp4 -to 00:01:30 -c copy output.mp4

# Time formats: HH:MM:SS.ms or seconds (e.g., 90.5)
```

### Resize Video

```bash
# Specific resolution
ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4

# Scale keeping aspect ratio (-1 or -2 for auto)
ffmpeg -i input.mp4 -vf scale=1280:-1 output.mp4
ffmpeg -i input.mp4 -vf scale=-2:720 output.mp4  # -2 ensures divisible by 2

# Scale by factor
ffmpeg -i input.mp4 -vf scale=iw/2:ih/2 output.mp4

# Fit within box (no upscale)
ffmpeg -i input.mp4 -vf "scale='min(1280,iw)':'min(720,ih)':force_original_aspect_ratio=decrease" output.mp4
```

### Change Frame Rate

```bash
# Change frame rate
ffmpeg -i input.mp4 -r 30 output.mp4

# Convert variable to constant frame rate
ffmpeg -i input.mp4 -vsync cfr -r 30 output.mp4
```

### Create GIF

```bash
# Basic GIF
ffmpeg -i input.mp4 -vf "fps=10,scale=320:-1" output.gif

# High quality GIF with palette
ffmpeg -i input.mp4 -vf "fps=10,scale=320:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" output.gif

# GIF from specific segment
ffmpeg -ss 00:00:05 -t 3 -i input.mp4 -vf "fps=15,scale=480:-1" output.gif
```

### Merge/Concatenate Files

```bash
# Method 1: Concat demuxer (same codec, recommended)
# First create filelist.txt:
# file 'part1.mp4'
# file 'part2.mp4'
# file 'part3.mp4'
ffmpeg -f concat -safe 0 -i filelist.txt -c copy output.mp4

# Method 2: Concat filter (different codecs, re-encodes)
ffmpeg -i part1.mp4 -i part2.mp4 -filter_complex "[0:v][0:a][1:v][1:a]concat=n=2:v=1:a=1[v][a]" -map "[v]" -map "[a]" output.mp4
```

### Add Subtitles

```bash
# Burn subtitles into video (hardcode)
ffmpeg -i input.mp4 -vf subtitles=subs.srt output.mp4

# Add subtitle stream (softcode)
ffmpeg -i input.mp4 -i subs.srt -c copy -c:s mov_text output.mp4

# Extract subtitles
ffmpeg -i input.mkv -map 0:s:0 output.srt
```

### Extract Frames/Images

```bash
# Extract single frame at timestamp
ffmpeg -ss 00:00:10 -i input.mp4 -frames:v 1 frame.png

# Extract frame every second
ffmpeg -i input.mp4 -vf fps=1 frame_%04d.png

# Extract frame every N seconds
ffmpeg -i input.mp4 -vf fps=1/5 frame_%04d.png  # every 5 seconds

# Extract all frames
ffmpeg -i input.mp4 frames/frame_%06d.png
```

### Create Video from Images

```bash
# From numbered sequence (frame001.png, frame002.png, ...)
ffmpeg -framerate 24 -i frame%03d.png -c:v libx264 -pix_fmt yuv420p output.mp4

# With glob pattern
ffmpeg -framerate 24 -pattern_type glob -i '*.png' -c:v libx264 output.mp4

# Add audio to image slideshow
ffmpeg -framerate 1/5 -i img%03d.png -i audio.mp3 -c:v libx264 -shortest output.mp4
```

### Screen/Desktop Capture

```bash
# Windows (gdigrab)
ffmpeg -f gdigrab -framerate 30 -i desktop output.mp4

# Windows specific window
ffmpeg -f gdigrab -framerate 30 -i title="Window Title" output.mp4

# Linux X11
ffmpeg -f x11grab -video_size 1920x1080 -framerate 25 -i :0.0 output.mp4

# macOS
ffmpeg -f avfoundation -framerate 30 -i "1" output.mp4
```

### Add/Replace Audio

```bash
# Replace audio track
ffmpeg -i video.mp4 -i newaudio.mp3 -map 0:v -map 1:a -c copy output.mp4

# Add audio to video without audio
ffmpeg -i video.mp4 -i audio.mp3 -c:v copy -c:a aac -shortest output.mp4

# Mix audio tracks
ffmpeg -i video.mp4 -i music.mp3 -filter_complex "[0:a][1:a]amix=inputs=2:duration=first[a]" -map 0:v -map "[a]" output.mp4
```

### Adjust Volume

```bash
# Increase volume by 50%
ffmpeg -i input.mp4 -af "volume=1.5" output.mp4

# Increase by decibels
ffmpeg -i input.mp4 -af "volume=10dB" output.mp4

# Normalize audio
ffmpeg -i input.mp4 -af loudnorm output.mp4
```

### Rotate/Flip Video

```bash
# Rotate 90 degrees clockwise
ffmpeg -i input.mp4 -vf "transpose=1" output.mp4

# Rotate 90 degrees counter-clockwise
ffmpeg -i input.mp4 -vf "transpose=2" output.mp4

# Rotate 180 degrees
ffmpeg -i input.mp4 -vf "transpose=1,transpose=1" output.mp4

# Horizontal flip
ffmpeg -i input.mp4 -vf "hflip" output.mp4

# Vertical flip
ffmpeg -i input.mp4 -vf "vflip" output.mp4
```

### Crop Video

```bash
# Crop to width:height starting at x:y
ffmpeg -i input.mp4 -vf "crop=640:480:100:50" output.mp4

# Crop center square
ffmpeg -i input.mp4 -vf "crop=min(iw\,ih):min(iw\,ih)" output.mp4

# Remove black bars (auto-detect)
ffmpeg -i input.mp4 -vf "cropdetect" -f null NUL  # detect first
ffmpeg -i input.mp4 -vf "crop=1920:800:0:140" output.mp4
```

### Overlay/Watermark

```bash
# Add image overlay (watermark)
ffmpeg -i video.mp4 -i logo.png -filter_complex "overlay=10:10" output.mp4

# Bottom right corner with padding
ffmpeg -i video.mp4 -i logo.png -filter_complex "overlay=W-w-10:H-h-10" output.mp4

# Picture-in-picture
ffmpeg -i main.mp4 -i pip.mp4 -filter_complex "[1:v]scale=320:-1[pip];[0:v][pip]overlay=W-w-10:10" output.mp4
```

### Speed Up/Slow Down

```bash
# Speed up video 2x (and audio)
ffmpeg -i input.mp4 -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]" output.mp4

# Slow down video 0.5x
ffmpeg -i input.mp4 -filter_complex "[0:v]setpts=2.0*PTS[v];[0:a]atempo=0.5[a]" -map "[v]" -map "[a]" output.mp4

# Speed up video only (drop audio)
ffmpeg -i input.mp4 -vf "setpts=0.5*PTS" -an output.mp4
```

### Add Fade Effects

```bash
# Fade in first 2 seconds
ffmpeg -i input.mp4 -vf "fade=t=in:st=0:d=2" output.mp4

# Fade out last 2 seconds (need to know duration)
ffmpeg -i input.mp4 -vf "fade=t=out:st=58:d=2" output.mp4  # for 60s video

# Audio fade
ffmpeg -i input.mp4 -af "afade=t=in:st=0:d=2,afade=t=out:st=58:d=2" output.mp4
```

---

## Hardware Acceleration

```bash
# NVIDIA NVENC encoding
ffmpeg -i input.mp4 -c:v h264_nvenc -preset fast output.mp4

# NVIDIA NVDEC decoding + NVENC encoding
ffmpeg -hwaccel cuda -i input.mp4 -c:v h264_nvenc output.mp4

# Intel QuickSync
ffmpeg -i input.mp4 -c:v h264_qsv output.mp4

# AMD AMF
ffmpeg -i input.mp4 -c:v h264_amf output.mp4

# List available hardware acceleration methods
ffmpeg -hwaccels
```

---

## Useful Information Commands

```bash
# Show file information
ffmpeg -i input.mp4
ffprobe input.mp4
ffprobe -v quiet -print_format json -show_format -show_streams input.mp4

# List available codecs
ffmpeg -codecs

# List available encoders
ffmpeg -encoders

# List available decoders
ffmpeg -decoders

# List available formats
ffmpeg -formats

# List available filters
ffmpeg -filters

# List available pixel formats
ffmpeg -pix_fmts
```

---

## Usage Best Practices

1. **Always verify input exists** before processing
2. **Use `-y` to overwrite** output files without prompts in scripts
3. **Use `-n` to prevent accidental overwrites** of important files
4. **Place `-ss` before `-i`** for fast seeking (may be less accurate)
5. **Place `-ss` after `-i`** for frame-accurate seeking (slower)
6. **Use `-c copy` when possible** to avoid re-encoding (faster, no quality loss)
7. **Preview with ffplay** before applying complex filters
8. **Use `-hide_banner`** for cleaner output in scripts
9. **Check exit codes** in scripts (0 = success)
10. **Stream indices are 0-based**: First input is `0`, streams start at `0`

---

## Adaptive Streaming (HLS / DASH)

Adaptive streaming creates multiple quality versions and divides them into small chunks for HTTP delivery. The player switches quality based on network conditions.

### HLS (HTTP Live Streaming)

```bash
# Create HLS stream with multiple qualities
ffmpeg -i input.mp4 \
  -filter_complex "[0:v]split=3[v1][v2][v3]; \
    [v1]scale=1920:1080[v1out]; \
    [v2]scale=1280:720[v2out]; \
    [v3]scale=854:480[v3out]" \
  -map "[v1out]" -map 0:a -c:v libx264 -b:v 5M -c:a aac -b:a 192k \
    -hls_time 6 -hls_playlist_type vod -hls_segment_filename "1080p_%03d.ts" 1080p.m3u8 \
  -map "[v2out]" -map 0:a -c:v libx264 -b:v 3M -c:a aac -b:a 128k \
    -hls_time 6 -hls_playlist_type vod -hls_segment_filename "720p_%03d.ts" 720p.m3u8 \
  -map "[v3out]" -map 0:a -c:v libx264 -b:v 1M -c:a aac -b:a 96k \
    -hls_time 6 -hls_playlist_type vod -hls_segment_filename "480p_%03d.ts" 480p.m3u8

# Simple single-quality HLS
ffmpeg -i input.mp4 -c:v libx264 -c:a aac \
  -hls_time 10 -hls_list_size 0 -hls_segment_filename "segment_%03d.ts" \
  output.m3u8
```

### DASH (Dynamic Adaptive Streaming over HTTP)

```bash
# Create DASH stream
ffmpeg -i input.mp4 \
  -map 0:v -map 0:v -map 0:a \
  -c:v libx264 -c:a aac \
  -b:v:0 5M -s:v:0 1920x1080 \
  -b:v:1 3M -s:v:1 1280x720 \
  -b:a:0 192k \
  -f dash -seg_duration 4 \
  -adaptation_sets "id=0,streams=v id=1,streams=a" \
  output.mpd
```

### HLS Options

| Option | Purpose |
|--------|---------|
| `-hls_time` | Segment duration in seconds |
| `-hls_list_size` | Max playlist entries (0 = unlimited) |
| `-hls_playlist_type vod` | Video on demand (complete playlist) |
| `-hls_playlist_type event` | Live event (grows over time) |
| `-hls_segment_filename` | Pattern for segment filenames |
| `-hls_base_url` | Prepend URL to segments in playlist |

---

## Fragmented MP4 (fMP4)

Standard MP4 stores all metadata at start/end. Fragmented MP4 distributes data for streaming compatibility.

```bash
# Create fragmented MP4
ffmpeg -i input.mp4 -c copy -movflags frag_keyframe+empty_moov output_frag.mp4

# Fragmented MP4 for DASH
ffmpeg -i input.mp4 -c copy -movflags frag_keyframe+empty_moov+default_base_moof output.mp4
```

### movflags Options

| Flag | Purpose |
|------|---------|
| `frag_keyframe` | Start new fragment at each keyframe |
| `empty_moov` | Write empty moov atom (required for streaming) |
| `default_base_moof` | Use moof as base for data offsets |
| `faststart` | Move moov to start (for progressive download) |

```bash
# Optimize for web progressive download
ffmpeg -i input.mp4 -c copy -movflags +faststart output.mp4
```

---

## Subtitles & Text Overlay

### Subtitle Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| SubRip | .srt | Most common, simple text + timing |
| WebVTT | .vtt | Web standard, supports styling |
| ASS/SSA | .ass/.ssa | Advanced styling, positioning |
| DVB | .sub | Bitmap subtitles |

### Burn Subtitles (Hardcode)

```bash
# SRT subtitles
ffmpeg -i input.mp4 -vf "subtitles=subs.srt" output.mp4

# ASS subtitles with styling
ffmpeg -i input.mp4 -vf "ass=subs.ass" output.mp4

# Custom subtitle style
ffmpeg -i input.mp4 -vf "subtitles=subs.srt:force_style='FontSize=24,PrimaryColour=&H00FFFF&'" output.mp4
```

### Soft Subtitles (Embedded Stream)

```bash
# Add subtitle track (can be toggled)
ffmpeg -i input.mp4 -i subs.srt -c copy -c:s mov_text output.mp4

# Multiple subtitle tracks
ffmpeg -i input.mp4 -i english.srt -i spanish.srt \
  -map 0:v -map 0:a -map 1 -map 2 \
  -c copy -c:s mov_text \
  -metadata:s:s:0 language=eng -metadata:s:s:1 language=spa \
  output.mp4

# Extract subtitles
ffmpeg -i input.mkv -map 0:s:0 output.srt
```

### Text Overlay (drawtext filter)

```bash
# Basic centered text
ffmpeg -i input.mp4 -vf "drawtext=text='Hello World':fontsize=48:fontcolor=white:x=(w-text_w)/2:y=(h-text_h)/2" output.mp4

# Text with background box
ffmpeg -i input.mp4 -vf "drawtext=text='Title':fontsize=36:fontcolor=white:x=(w-text_w)/2:y=50:box=1:boxcolor=black@0.7:boxborderw=10" output.mp4

# Timestamp overlay
ffmpeg -i input.mp4 -vf "drawtext=text='%{pts\\:hms}':fontsize=24:fontcolor=white:x=10:y=10" output.mp4

# Frame number
ffmpeg -i input.mp4 -vf "drawtext=text='Frame %{frame_num}':fontsize=24:fontcolor=yellow:x=10:y=10" output.mp4

# Timed text (appears 2s to 5s)
ffmpeg -i input.mp4 -vf "drawtext=text='Appearing':fontsize=32:fontcolor=red:x=100:y=100:enable='between(t,2,5)'" output.mp4

# Scrolling credits
ffmpeg -i input.mp4 -vf "drawtext=textfile=credits.txt:fontsize=24:fontcolor=white:x=(w-text_w)/2:y=h-t*50" output.mp4
```

### Text Position Reference

```
┌────────────────────────────────────────┐
│ x=10, y=10          x=w-tw-10, y=10    │  (top-left, top-right)
│                                        │
│           x=(w-text_w)/2               │  (center)
│           y=(h-text_h)/2               │
│                                        │
│ x=10, y=h-th-10     x=w-tw-10, y=h-th-10│  (bottom-left, bottom-right)
└────────────────────────────────────────┘

w = video width       tw = text width
h = video height      th = text height
```

---

## Complex Filtergraphs

For multiple inputs/outputs, use `-filter_complex` instead of `-vf`/`-af`.

### Syntax

```
[input_label]filter=params[output_label];[input_label]filter=params[output_label]
```

### Examples

```bash
# Picture-in-Picture
ffmpeg -i main.mp4 -i overlay.mp4 \
  -filter_complex "[1:v]scale=320:180[pip];[0:v][pip]overlay=W-w-10:10[out]" \
  -map "[out]" -map 0:a output.mp4

# Side-by-side comparison
ffmpeg -i left.mp4 -i right.mp4 \
  -filter_complex "[0:v][1:v]hstack=inputs=2[v]" \
  -map "[v]" output.mp4

# Vertical stack
ffmpeg -i top.mp4 -i bottom.mp4 \
  -filter_complex "[0:v][1:v]vstack=inputs=2[v]" \
  -map "[v]" output.mp4

# Grid (2x2)
ffmpeg -i 1.mp4 -i 2.mp4 -i 3.mp4 -i 4.mp4 \
  -filter_complex "[0:v][1:v]hstack[top];[2:v][3:v]hstack[bottom];[top][bottom]vstack[v]" \
  -map "[v]" output.mp4

# Crossfade between videos
ffmpeg -i first.mp4 -i second.mp4 \
  -filter_complex "[0:v][1:v]xfade=transition=fade:duration=1:offset=4[v];[0:a][1:a]acrossfade=d=1[a]" \
  -map "[v]" -map "[a]" output.mp4

# Mix audio tracks
ffmpeg -i video.mp4 -i music.mp3 \
  -filter_complex "[0:a][1:a]amix=inputs=2:duration=first:dropout_transition=2[a]" \
  -map 0:v -map "[a]" output.mp4
```

### Transition Types for xfade

`fade`, `wipeleft`, `wiperight`, `wipeup`, `wipedown`, `slideleft`, `slideright`, `slideup`, `slidedown`, `circlecrop`, `rectcrop`, `distance`, `fadeblack`, `fadewhite`, `radial`, `smoothleft`, `smoothright`, `smoothup`, `smoothdown`, `circleopen`, `circleclose`, `vertopen`, `vertclose`, `horzopen`, `horzclose`, `dissolve`, `pixelize`, `diagtl`, `diagtr`, `diagbl`, `diagbr`, `hlslice`, `hrslice`, `vuslice`, `vdslice`

---

## RTMP Streaming

```bash
# Stream to RTMP server (Twitch, YouTube, etc.)
ffmpeg -re -i input.mp4 -c:v libx264 -preset veryfast -maxrate 3000k -bufsize 6000k \
  -c:a aac -b:a 160k -ar 44100 \
  -f flv rtmp://live.twitch.tv/app/YOUR_STREAM_KEY

# Capture webcam and stream
ffmpeg -f dshow -i video="Webcam":audio="Microphone" \
  -c:v libx264 -preset veryfast -b:v 2500k \
  -c:a aac -b:a 128k \
  -f flv rtmp://server/live/stream

# Re-stream (receive and forward)
ffmpeg -i rtmp://source/live/stream -c copy -f flv rtmp://destination/live/stream
```

---

## Batch Processing

### Windows (PowerShell)

```powershell
# Convert all MP4 to MKV
Get-ChildItem *.mp4 | ForEach-Object { ffmpeg -i $_.Name -c copy ($_.BaseName + ".mkv") }

# Compress all videos
Get-ChildItem *.mp4 | ForEach-Object { ffmpeg -i $_.Name -c:v libx264 -crf 23 ("compressed_" + $_.Name) }
```

### Linux/macOS (Bash)

```bash
# Convert all MP4 to MKV
for f in *.mp4; do ffmpeg -i "$f" -c copy "${f%.mp4}.mkv"; done

# Compress all videos
for f in *.mp4; do ffmpeg -i "$f" -c:v libx264 -crf 23 "compressed_$f"; done

# Extract audio from all videos
for f in *.mp4; do ffmpeg -i "$f" -vn -c:a libmp3lame "${f%.mp4}.mp3"; done
```

---

## Quality Settings Reference

### CRF (Constant Rate Factor) for x264/x265

| CRF | Quality | Use Case |
|-----|---------|----------|
| 0 | Lossless | Archival |
| 17-18 | Visually lossless | High quality |
| 19-23 | High quality | Default (23) |
| 24-28 | Medium quality | Web/streaming |
| 29-51 | Low quality | Small file size |

### Presets (Speed vs Compression)

| Preset | Speed | File Size | Quality |
|--------|-------|-----------|---------|
| ultrafast | Fastest | Largest | Lowest |
| superfast | | | |
| veryfast | | | |
| faster | | | |
| fast | | | |
| medium | Default | | |
| slow | | | |
| slower | | | |
| veryslow | Slowest | Smallest | Highest |

```bash
# Fast encode, larger file
ffmpeg -i input.mp4 -c:v libx264 -preset ultrafast -crf 23 output.mp4

# Slow encode, smaller file, better quality
ffmpeg -i input.mp4 -c:v libx264 -preset veryslow -crf 23 output.mp4
```

---

## Troubleshooting

### Common Issues

| Problem | Solution |
|---------|----------|
| Audio/video out of sync | Use `-async 1` or `-vsync cfr` |
| "Non-monotonous DTS" | Add `-fflags +genpts` before `-i` |
| Green/corrupted frames | Add `-vsync drop` or re-encode |
| File won't play | Use `-movflags +faststart` for MP4 |
| Seek doesn't work | Ensure keyframes exist, re-encode if needed |
| Aspect ratio wrong | Use `-aspect 16:9` or check SAR/DAR |

### Debug Commands

```bash
# Show all stream info
ffprobe -show_streams input.mp4

# Show packets (timing debug)
ffprobe -show_packets input.mp4

# Show frames
ffprobe -show_frames input.mp4

# Verbose encoding
ffmpeg -v debug -i input.mp4 output.mp4
```
