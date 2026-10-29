# voicestudio / `speak`: TTS in the user's own voice (for agents)

One command, any agent. Local only, no API key, no cost. Engine: VoiceStudio.app (`/Applications/VoiceStudio.app`, open-source, AGPL, https://github.com/debpalash/VoiceStudio) serving a REST API on `127.0.0.1:3900`. Fallback: macOS `say`.

Call it by absolute path (bare agent shells may lack `~/.local/bin`):

```bash
SPEAK=$HOME/.local/bin/speak     # symlink to <repo>/bin/speak
```

## Copy-paste

```bash
$SPEAK voices                                   # profiles (his cloned voices) + aliases; add --all for macOS say voices
$SPEAK "Hello there." --out /tmp/hi.wav         # default voice = alias "me" from your config; prints the file path
$SPEAK "Hello." --voice "The Storyteller" --out hi.wav    # pick a voice by profile name, id, alias or name prefix
$SPEAK --file script.txt --out narration.wav    # speak a file
$SPEAK "Say it aloud" --play                    # also plays via afplay
$SPEAK "Hi" --json --out hi.wav                 # {"file","seconds","engine","voice"}
$SPEAK --batch script.txt --out-dir seg/        # paragraphs (blank-line separated) -> seg/seg-001.wav ... + seg/manifest.json
```

## Delivery controls (all optional, VoiceStudio voices only; the `say` fallback ignores them except `--gain`)

```bash
$SPEAK "text" --speed 0.85 --out a.wav          # 0.8 slower ... 1.3 faster; default 1.0
$SPEAK "text" --effect warm --out a.wav         # broadcast (default), cinematic, podcast, raw, warm, bright (EQ, compression, reverb)
$SPEAK "text" --seed 7 --out a.wav              # same text + seed = identical audio; change the seed for a different take
$SPEAK "text" --temperature 0.3 --out a.wav     # delivery variation: lower steadier, higher more varied
$SPEAK "text" --guidance 3 --out a.wav          # how strongly the clone is steered (default 2.0)
$SPEAK "text" --instruct "whisper" --out a.wav  # voice attributes ONLY (see below), not free text like "calm"
$SPEAK "text" --gain -3 --out a.wav             # volume in dB; auto-limited so it never clips
```

`--instruct` accepts only comma+space separated items from: american accent, australian accent, british accent, canadian accent, child, chinese accent, elderly, female, high pitch, indian accent, japanese accent, korean accent, low pitch, male, middle-aged, moderate pitch, portuguese accent, russian accent, teenager, very high pitch, very low pitch, whisper, young adult. Anything else exits 2 with the server's message. Pace and emotion are NOT instruct items: use `--speed`, `--temperature` and `--seed`.
`--effect raw` plus `--gain` gives a plain, level-controlled clip. `--effect` changes loudness and tone indirectly (raw measured about 3x quieter than warm on the same text).
Seed and gain were verified; `--temperature`, `--guidance` and `--effect` are passed through and untested for audible quality: audition before relying on them.

`manifest.json` has `total_seconds` and per segment `index, file, text, start, end, duration, engine, voice`: use `start`/`end` to place slides or cut video. Output is 24 kHz mono 16-bit WAV.

## Behavior to know

- First call may start VoiceStudio.app (up to ~90 s, a model loads). A call takes roughly 5 s of compute per 1 s of audio on this Mac (17 s for a 4 s clip), so batch, do not call per word.
- Long text is chunked by the app (800 chars per chunk, crossfaded). Use paragraph segments for video timing anyway.
- Exit codes: 1 usage, 2 synth failed, 3 unknown voice, 4 no engine. A fallback to `say` prints `speak: using macOS say ... not the cloned voice` on stderr: check `engine` in `--json` if the voice matters.
- Config: `~/.config/voicestudio/config.json` (personal, outside the repo; see `config.example.json`) `{"default_voice":"me","aliases":{"me":"<your profile>"}}`. Env `VOICESTUDIO_URL` overrides the API base.
- Nothing is uploaded; there is no key or token. Reference clips live inside the app's data dir.

## Voices and consent

The user's own voice = a clone profile they made in the app, set as the `me` alias in their config. Other voices: import from the app's Archetypes gallery (Use voice) or make more in the app UI (Profiles -> New, record or upload a 3-10 s clean clip). Clone only the user's own voice or someone who has consented; never clone another person without it. Agents must not record or upload voice samples themselves.

## Setting up your own voice (steps)

1. Open VoiceStudio.app, Profiles, New clone profile.
2. Record 10 to 30 s of clean speech in a quiet room, no music, natural pace; type the exact words said as the reference text (improves quality).
3. Name it, then `speak voices` shows it; point the alias: set `aliases.me` in your config.

## Relation to other tools

- `utts` (~/work/utts): fast spoken notifications/hooks (edge, piper, say, espeak). Play-only, 500-char cap, low latency. Keep for notifications. `speak` is for narration and files.
- Not for notifications: 17 s latency.
- Other engines on this Mac (rb-edge-tts, piper-tts, mlx-audio Kokoro in cultiv-ai `bin/voice-staff-meeting`) are fixed stock voices; use them when a stock voice is fine and the cloned voice is not needed.

## Raw API (if you need more than speak)

`GET /profiles`, `POST /generate` (multipart: `text`, `profile_id`, `speed`, `language`, `effect_preset`, `seed`; returns audio/wav), `GET /openapi.json`. The app also has an MCP server (see its docs/mcp.md). `speak` is the stable interface; prefer it.
