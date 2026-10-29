---
name: voice-studio
description: Text-to-speech, narration, voiceover, "read this aloud", "in my voice", video/presentation audio. Use the `speak` command (local VoiceStudio.app, the user's cloned voice, no cost) to turn text or a script into WAV files with per-segment timings.
---

# voice-studio (`speak`)

Use for TTS, narration, voiceover, or anything "in my voice". Full guide: ~/work/voicestudio/docs/for_llms.md

```bash
SPEAK=$HOME/.local/bin/speak
$SPEAK voices                                  # his voices (clone profiles + aliases)
$SPEAK "text" --out /abs/path/out.wav          # default = the user's own voice ("me" alias)
$SPEAK "text" --voice "The Storyteller" --out out.wav   # other voice by profile name (see speak voices)
$SPEAK --file script.txt --out narration.wav
$SPEAK --batch script.txt --out-dir seg/       # blank-line paragraphs -> seg-NNN.wav + manifest.json (start/end seconds)
```

Use `--voice me-fast` (or any alias shown by `speak voices`) for a preset voice+pace; aliases carry their own speed. Delivery (optional): `--speed 0.85`, `--effect raw|warm|bright|podcast|cinematic|broadcast`, `--seed N` (repeatable take), `--temperature N`, `--guidance N`, `--gain -3` (dB, never clips), `--instruct whisper` (voice attributes only, not free text). Details in the guide.

Rules: ~5 s compute per 1 s audio, so batch; first call may launch the app (wait up to 90 s). Use `--json` and check `engine` (`voicestudio` = cloned voice, `say` = fallback, tell the user). Never record or upload voice samples; never clone anyone without their consent. Local only, no keys.
For spoken notifications use `utts` instead (fast, play-only).
