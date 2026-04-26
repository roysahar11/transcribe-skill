---
name: transcribe
description: Transcribe audio/video files to text with timestamps using whisper-cpp. Use when asked to transcribe, subtitle, understand speech in media files, or when you need to "listen" to video/audio content. Supports Hebrew (ivrit-ai models) and all Whisper-supported languages.
user-invocable: true
argument-hint: <file-or-directory> [language]
allowed-tools: Bash Read Glob Grep
---

# Audio/Video Transcription (whisper-cpp)

Transcribe `$ARGUMENTS` using whisper-cpp with locally-hosted Whisper models.

## Quick Reference

```bash
# Single file transcription
whisper-cli -m "$MODEL" -l he -f audio.wav -osrt -oj -of output_prefix

# Extract audio from video first (whisper needs WAV)
ffmpeg -v quiet -i video.mp4 -ar 16000 -ac 1 -c:a pcm_s16le audio.wav
```

## Workflow

### 1. Prepare audio

whisper-cli requires 16kHz mono WAV input. Extract from any media format:

```bash
ffmpeg -v quiet -i input.mp4 -ar 16000 -ac 1 -c:a pcm_s16le output.wav
```

For batch extraction from a directory of videos:
```bash
for f in *.mp4; do
  ffmpeg -v quiet -i "$f" -ar 16000 -ac 1 -c:a pcm_s16le "audio/${f%.mp4}.wav"
done
```

### 2. Choose model

Models are stored in `~/.local/share/whisper-models/`.

| Model | File | Size | Use when |
|-------|------|------|----------|
| ivrit-ai large-v3 (full) | `ggml-ivrit-large-v3.bin` | 2.9 GB | Hebrew speech (best accuracy) |
| ivrit-ai large-v3-turbo | `ggml-ivrit-large-v3-turbo.bin` | 1.5 GB | Hebrew speech (faster, lower accuracy) |

**Hebrew model selection:**
- Always use ivrit-ai models for Hebrew — they cut word error rate by ~50% vs generic Whisper.
- The full model (large-v3) is significantly more accurate than turbo. Prefer it unless speed is critical.
- Always set `-l he` explicitly — language auto-detection is degraded in fine-tuned models.

**Downloading models (if missing):**
```bash
# ivrit-ai large-v3 (recommended for Hebrew)
curl -L -o ~/.local/share/whisper-models/ggml-ivrit-large-v3.bin \
  "https://huggingface.co/ivrit-ai/whisper-large-v3-ggml/resolve/main/ggml-model.bin"

# ivrit-ai large-v3-turbo (faster, less accurate)
curl -L -o ~/.local/share/whisper-models/ggml-ivrit-large-v3-turbo.bin \
  "https://huggingface.co/ivrit-ai/whisper-large-v3-turbo-ggml/resolve/main/ggml-model.bin"

# Standard multilingual large-v3 (non-Hebrew languages)
curl -L -o ~/.local/share/whisper-models/ggml-large-v3.bin \
  "https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-large-v3.bin"
```

### 3. Transcribe

```bash
MODEL="$HOME/.local/share/whisper-models/ggml-ivrit-large-v3.bin"

whisper-cli -m "$MODEL" -l he -f input.wav -osrt -oj -of output_prefix
```

**Output formats** (combine as needed):
- `-osrt` → `output_prefix.srt` (subtitles with timestamps)
- `-oj` → `output_prefix.json` (structured data with per-segment timing)
- `-ovtt` → `output_prefix.vtt` (WebVTT subtitles)
- `-otxt` → `output_prefix.txt` (plain text, no timestamps)

**Useful flags:**
- `-l he` — Set language to Hebrew (REQUIRED for ivrit-ai models)
- `-ml N` — Max segment length in characters. Use `-ml 1` for word-level timestamps.
- `--split-on-word` — Split on word boundaries rather than tokens
- `-t N` — Number of threads (default: 4)

### 4. Review output

Always review the transcription output. Read the SRT file to see timestamped text. For video editing, the timestamps tell you exactly where each spoken segment is.

## Accuracy Notes

Transcription accuracy depends heavily on audio conditions:
- **Clean speech, single speaker**: ~95% accuracy with ivrit-ai models
- **Background music**: Significantly degraded — lyrics and speech get mixed
- **Overlapping speakers**: Words may be attributed wrong or garbled
- **Phone-recorded video**: Room echo and mic quality reduce accuracy
- **Hebrew + English mix**: Generally handles well, but proper nouns and code-switching points may be off

For video editing, ~80% accuracy is usually sufficient to identify key moments and timestamps. Verify exact wording by listening when making precise cuts.

## Batch Transcription

To transcribe all videos in a directory:

```bash
MODEL="$HOME/.local/share/whisper-models/ggml-ivrit-large-v3.bin"

for wav in audio_dir/*.wav; do
  base=$(basename "$wav" .wav)
  dur=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$wav" | cut -d. -f1)
  [ "$dur" -lt 2 ] 2>/dev/null && continue  # skip very short files
  whisper-cli -m "$MODEL" -l he -f "$wav" -osrt -oj -of "transcripts/${base}"
done
```
