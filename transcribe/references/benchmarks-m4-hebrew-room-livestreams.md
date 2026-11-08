# Case-specific benchmarks: Hebrew multi-speaker room livestreams on an Apple M4

> **Measurements from one job, not general rules.** Use them as a planning prior for similar jobs. When the hardware, model, audio type or whisper.cpp version differ, re-measure with a 5-minute test clip before committing to a plan.

## The case

| | |
|---|---|
| Measured | 2026-11-05/06 |
| Material | Three YouTube livestreams of the Israeli President's consultations with the Knesset factions (26th Knesset, Nov 4-5 2026): a formal meeting room, 4-8 speakers per meeting on shared room microphones, Hebrew with occasional English |
| Audio | YouTube format 140 (AAC, ~128-144 kbps, 44.1 kHz) converted to 16 kHz mono WAV |
| Volume | ~13.7 h of stream audio; ~8.6 h of speech in 13 segments after cutting out long silences |
| Hardware | Apple M4 (10-core), 32 GB RAM, Metal GPU |
| Software | whisper-cli 1.8.4 (Homebrew; ggml 0.9.11; flash attention on) |
| Models | `ggml-ivrit-large-v3.bin` (full), `ggml-ivrit-large-v3-turbo.bin`; VAD `ggml-silero-v5.1.2.bin` |
| Flags | `-l he -t 8 --vad -vm <silero> -oj -osrt -otxt` (beam search defaults: `-bs 5 -bo 5`) |

## Speed, full model, one process at a time

| Segment (meeting) | Audio | Wall time | Speed |
|---|---|---|---|
| D1S1 | 2502 s | 941 s | 2.7x realtime |
| D1S2 | 2091 s | 1071 s | 2.0x (ran alongside a CPU speaker-clustering job) |
| D1S3 | 1575 s | 580 s | 2.7x |
| D1S4 | 2203 s | 708 s | 3.1x |
| D2S1 | 1718 s | 532 s | 3.2x |
| D2S2 | 2701 s | 847 s | 3.2x |
| D2S3 | 3287 s | 1097 s | 3.0x |
| D2S4 | 2445 s | 764 s | 3.2x |
| D2S5 | 655 s | 188 s | 3.5x |
| D3S3 | 2389 s | 762 s | 3.1x |
| D3S4 | 3220 s | 1081 s | 3.0x |
| **All 13 segments + 2 short clips** | **32,256 s (8.96 h)** | **10,861 s (3 h 01 m)** | **~3.0x** |

(Segment times include model load, ~3 s each. D3S1 and D3S2 were not timed individually; they are inside the total.)

## Settings compared on a 5-minute speech clip (same machine)

| Setting | Wall time for 300 s | Speed | Note |
|---|---|---|---|
| Full model, beam 5 (default) | 88 s | 3.4x | Chosen for quote-grade transcripts |
| Full model, greedy (`-bs 1 -bo 1`) | 81 s | 3.7x | Only ~8% faster; not worth it |
| Turbo, beam 5 | 39 s | 7.7x | ~92% word-sequence agreement with the full model (difflib); dropped whole clauses (one 9-word clause) and function words |
| 3 full-model processes in parallel (3 x 300 s) | 315 s | 2.9x aggregate | Slower than one process: GPU-bound, do not parallelize on this machine |

Both models erred on names and on Hebrew gender agreement; the full model lost less content.

## Lessons from this job

- **Cut dead air first.** Long livestreams had 10-48 minute silent breaks between meetings. `ffmpeg -i in.wav -af silencedetect=noise=-60dB:d=30 -f null -` found them; transcribing only the speech spans (+5 s padding) skipped ~5 h of silence and produced one file per meeting.
- **Always use VAD.** Without it, silence produced the classic hallucination "תודה רבה."; with Silero VAD there were no hallucination loops in 8.6 h of speech.
- **Strip bidi control characters** (U+200E/U+200F/U+202A-U+202E/U+2066-U+2069) when assembling text; one segment's output had U+202B at line starts.
- **Broadcast dropouts look like silence.** Two gaps in the middle and at the end of meetings were frozen video plus silent audio on the broadcaster's side (checked with ffmpeg `freezedetect`); mark such gaps in the transcript instead of assuming the meeting ended.
- **yt-dlp:** version 2026.07.04 failed on YouTube with HTTP 403 / "Only images are available"; `brew upgrade yt-dlp` (to 2026.10.26) fixed it. A just-ended livestream in `post_live` state (the player said "Video unavailable") was still downloadable with `-f 140` after the upgrade.
- **Speakers:** unsupervised voice clustering at whisper-segment level (Resemblyzer + KMeans, as in the diarize-meeting skill) failed on this room recording: one speaker was split across alternating clusters. Attribution by an LLM reading the transcript (forms of address, self-introductions, Hebrew grammatical gender, turn structure) worked well, with uncertain turns marked "(?)".
