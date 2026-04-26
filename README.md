# transcribe-skill

An agent skill for transcribing audio and video files to text with timestamps using [whisper.cpp](https://github.com/ggerganov/whisper.cpp). Supports Hebrew via [ivrit-ai](https://huggingface.co/ivrit-ai) models and all Whisper-supported languages.

## What it does

- Transcribe a single file or a directory of media files
- Produce subtitles (SRT, VTT), structured JSON with per-segment timing, or plain text
- First-class Hebrew support via ivrit-ai fine-tuned Whisper models (~50% lower word error rate than generic Whisper)
- Word-level timestamps for precise editing workflows

## Installation

See **[SETUP.md](SETUP.md)** for the full guide. **Quick start:**

```bash
git clone https://github.com/roysahar11/transcribe-skill.git ~/skill-repository/transcribe-skill
ln -s ~/skill-repository/transcribe-skill/transcribe ~/.claude/skills/transcribe
```

Then install `whisper-cpp`, `ffmpeg`, and download a model — see SETUP.md.

## Usage

Once installed, invoke the skill from your agent:

```
/transcribe path/to/video.mp4
/transcribe path/to/folder he
```

The skill handles audio extraction (whisper requires 16kHz mono WAV), model selection, and output formatting.

## Compatibility

**Tested with:** Claude Code.

The skill format used here (YAML frontmatter + markdown body, slash command invocation) is the [Anthropic skill spec](https://docs.anthropic.com/en/docs/claude-code/skills), which other agent platforms supporting skills should be able to use — but compatibility with them hasn't been verified. Some Claude Code-specific frontmatter fields (`user-invocable`, `argument-hint`) may be ignored by other platforms.

## Companion skills

- **[agentic-video-editing-collab](https://github.com/roysahar11/agentic-video-editing-collab)** — agent skills for collaborative video editing with professional editing software. References `/transcribe` for content understanding.

## License

MIT
