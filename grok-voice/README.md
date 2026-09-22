# Grok Voice

Four skills for adding Grok voice features to an app. Use them when a user asks for voice, dictation, or read-aloud with Grok, or reports that voice mode misbehaves.

| Skill | Does |
|:------|:-----|
| `add-voice` | Realtime speech-to-speech. Wires the session, safe auth, and the app mic, and adds a waveform Voice Mode button to the composer. Also replaces an STT-LLM-TTS cascade or OpenAI Realtime. |
| `add-dictation` | Speech-to-text. Batch for a mic button or recorded audio (files, uploads, URLs) with word timestamps, diarization, and subtitles; streaming through a relay for live text. |
| `add-read-aloud` | Text-to-speech. Batch MP3 for a speaker button on replies; streaming PCM through a relay so audio starts before the reply finishes. |
| `debug-voice` | Proposes a plan, installs a dev-only log pipeline (client logger → local NDJSON) in the app's language and conventions, then loops: match the report to log signatures, fix one thing, re-test. |

Icons: waveform = voice mode, microphone = dictation, speaker = read aloud.

The skills wire audio into the **app** being built. The editor running the agent has no mic or speaker.
