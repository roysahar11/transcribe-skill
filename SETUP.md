# Setup Guide

One-time setup for the transcribe skill.

## Prerequisites

- An agent platform that supports skills (e.g., [Claude Code](https://docs.anthropic.com/en/docs/claude-code))
- [`whisper-cpp`](https://github.com/ggerganov/whisper.cpp) — `brew install whisper-cpp` (macOS) or build from source
- [`ffmpeg`](https://ffmpeg.org/) — `brew install ffmpeg`

## Step 1 — Install the skill

```bash
# Clone the repo
git clone https://github.com/roysahar11/transcribe-skill.git ~/skill-repository/transcribe-skill

# Symlink into your agent's skills folder (Claude Code path shown)
ln -s ~/skill-repository/transcribe-skill/transcribe ~/.claude/skills/transcribe
```

If you already have a `~/.claude/skills/transcribe` folder, back it up or remove it first.

## Step 2 — Download a model

Models go in `~/.local/share/whisper-models/`. Create the directory first:

```bash
mkdir -p ~/.local/share/whisper-models
```

### For Hebrew (recommended: ivrit-ai)

```bash
# ivrit-ai large-v3 — best Hebrew accuracy, 2.9 GB
curl -L -o ~/.local/share/whisper-models/ggml-ivrit-large-v3.bin \
  "https://huggingface.co/ivrit-ai/whisper-large-v3-ggml/resolve/main/ggml-model.bin"

# OR ivrit-ai large-v3-turbo — faster, slightly lower accuracy, 1.5 GB
curl -L -o ~/.local/share/whisper-models/ggml-ivrit-large-v3-turbo.bin \
  "https://huggingface.co/ivrit-ai/whisper-large-v3-turbo-ggml/resolve/main/ggml-model.bin"
```

### For other languages (multilingual Whisper)

```bash
curl -L -o ~/.local/share/whisper-models/ggml-large-v3.bin \
  "https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-large-v3.bin"
```

The `SKILL.md` lists all supported model paths. Add models as you need them.

## Step 3 — Verify

```bash
whisper-cli --help | head -5
ls -lh ~/.local/share/whisper-models/
```

Both should succeed. If `whisper-cli` isn't found, your `whisper-cpp` install didn't put the binary on your PATH.

## Quick start (let your agent do it)

After installation, just say:

> /transcribe path/to/audio.mp4

The skill handles audio extraction (whisper needs WAV), model selection, and output formats (SRT, JSON, VTT, plain text).

## Notes on accuracy

Transcription quality depends heavily on audio conditions. See the "Accuracy Notes" section in `SKILL.md` for details. Briefly:

- Clean speech, single speaker: ~95% with ivrit-ai for Hebrew
- Background music or overlapping speakers: significantly degraded
- For video editing, ~80% accuracy is usually enough to identify key moments and timestamps; verify exact wording when making precise cuts
