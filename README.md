# Tab Recorder

A Chrome extension that records browser tabs individually in HD, makes MP3/M4A audio files, and transcribes speech on your own computer. Nothing is uploaded anywhere.

## Features

- **Per-tab recording.** Record several tabs at once; each starts, pauses and saves independently.
- **One recording, three modes.** *Video + audio* (one MP4), *Audio only*, or *Both files* (an MP4 plus an audio file made from it).
- **Quality selectors.**
  - Video: 720p 30 fps, 1080p 30 fps, or 1080p 60 fps (these are maximum sizes; a smaller browser window records smaller).
  - Audio files: M4A (AAC) or MP3, at 128, 192 or 320 kbps.
- **Pause and resume**, a recordings list, and a REC / paused badge on the toolbar icon.
- **Keep tab active.** Stops sites from noticing the tab is hidden, and stops Chrome freezing or discarding it, while you work in another tab. Switched off automatically when recording ends.
- **On-device transcription** (Whisper on your CPU and RAM), live or after recording, with a model size and language choice.
- **Look back slider (5 seconds to 10 minutes).** While recording, drag the slider to choose how far back to look, then:
  - **Save video clip** saves that last stretch of the video (with its sound) as an MP4 straight away.
  - **Save audio clip** saves that last stretch of the sound as MP3 or M4A, using your audio quality settings.
  - With live transcription on, the text said in that window is shown below the slider, with timestamps and a Copy button.
  Clips are saved to `TabRecordings/` as `...-clip-30s-...` and appear in the recordings list. Recording carries on as normal.
- **Upload a file to transcribe.** Click "Upload audio to transcribe…" in the popup to transcribe any audio or video file, or convert it to MP3/M4A.

## Requirements

- Google Chrome 126 or newer (other Chromium browsers may also work).
- Node.js, only for the one-time build step. Recording works without it.

## Installation

1. Install Node.js (nodejs.org). Without admin rights, use the Windows `.zip` version and add it to your PATH for the current session.
2. In a terminal inside this folder, run `npm install`. It builds the transcriber and the MP3/M4A converters into `dist/`. Run it again after updating the extension.
   - PowerShell without admin: `$env:Path = "C:\path\to\node;" + $env:Path` then `npm.cmd install`.
   - Warnings about deprecated packages, vulnerabilities and install scripts are safe to ignore. Don't run `npm audit fix --force`.
3. Go to `chrome://extensions`, turn on **Developer mode**, click **Load unpacked** and select this folder (or press the reload icon).

## Usage

1. Open the tab you want to record and click the Tab Recorder icon.
2. Pick a mode. Open **Quality** to choose video and audio quality.
3. Optionally tick **Transcribe on this device** (Live or After recording), then **Start recording**.
4. While recording live, drag the **Catch up** slider to review the last 5 seconds to 10 minutes.
5. **Stop & save.** Files go to your download folder, inside `TabRecordings/`:

| File | Contents |
| --- | --- |
| `Title-<time>.mp4` | Video with audio |
| `Title-audio-<time>.m4a` or `.mp3` | Audio only |
| `Title-transcript-<time>.txt` | Transcript with `[mm:ss]` timestamps |

## Notes and limits

- **Video clips are cut, not re-encoded**, so they are instant and lose no quality. They start at the nearest keyframe before the time you asked for, so a clip can be a few seconds longer than requested. They need MP4 recording (Chrome 126+); WebM recordings can use audio clips only.
- **Audio clips** come from a rolling buffer of the last 10 minutes of sound (about 115 MB of RAM while recording). Asking for more than has been recorded gives you everything so far.
- **Audio files are converted from the recording.** There is a single recorder per tab, so nothing competes for the sound. The converter reads the whole recording into memory, so it works up to about 1.5 GB (roughly 25 minutes of 1080p30). For longer sessions use *Audio only* mode, or keep just the MP4. The MP4 is always saved first.
- **The audio inside the MP4 is 192 kbps AAC**, so a 320 kbps file made from it is bigger but not better. For the highest audio quality choose *Audio only*, which records at 256 kbps Opus.
- **AAC bitrate:** some systems limit AAC encoding bitrate. If 320 kbps isn't available for M4A, the extension steps down to the highest supported rate and tells you. MP3 supports all three.
- **Live transcription** re-reads recent audio every 2 seconds. Tiny and Base keep up on most machines; Small may lag. Set the language manually for best live accuracy.
- The first use of each model size downloads it from huggingface.co (about 40, 80 or 250 MB), then it is cached.
- Chrome only lets the extension capture a tab after you open the popup on that tab. Protected (DRM) video may record as black.

## Troubleshooting

| Problem | Try |
| --- | --- |
| "Extension has not been invoked for the current page" | Click the icon while that tab is active. |
| "Transcriber isn't built yet" / "converter isn't built" | Run `npm install` in the extension folder, then reload the extension. |
| "The video clip isn't ready yet" | Wait a few seconds after starting and try again. If it never works, use the audio clip and tell the developer. |
| Model won't load | Check huggingface.co is reachable. You can also remove the two `cross_origin_*` lines from `manifest.json` (single-threaded, slower). |
| Files are `.webm` instead of `.mp4` | Your Chrome doesn't support MP4 recording; update to 126 or newer. |
| Something else | On `chrome://extensions`, open the extension's **Errors** page or click **service worker**. |

## Project files

| File | Purpose |
| --- | --- |
| `manifest.json` | Extension configuration and permissions |
| `popup.html`, `popup.js` | The popup (settings, look-back slider, clips) |
| `background.js` | Toolbar badge, downloads, recordings list |
| `offscreen.js` | Recording, clips, live transcription, conversion jobs |
| `mp4clip.js` | Cuts the last N seconds out of the MP4 stream |
| `common.js` | Shared workers client, decoding, audio encoding |
| `pcm-worklet.js` | Taps the tab's audio (transcription and replay buffer) |
| `tools.html`, `tools.js` | Upload a file to transcribe or convert |
| `transcribe-worker.js`, `encode-*.js`, `build.mjs` | Built into `dist/` by `npm install` |
