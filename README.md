# Tab Recorder

A Chrome extension that records browser tabs individually in HD, saves video and audio as separate files if you want, and can transcribe speech on your own computer. Nothing is uploaded anywhere.

## Features

- **Per-tab recording.** Record any tab on its own. Record several tabs at once; each one starts, pauses and saves independently.
- **HD MP4 output.** 720p up to 1080p, up to 60 fps, 10 Mbps video with 192 kbps audio.
- **Three recording modes.**
  - *Video + audio*: one MP4 with picture and sound.
  - *Audio only*: one audio file.
  - *Both files*: a video MP4 and a separate audio file, from the same recording.
- **Pause and resume** for each tab.
- **Recordings list** in the popup, with a button to show each file in its folder.
- **Recording indicator.** The toolbar icon shows a red **REC** badge while recording and an amber **II** while paused.
- **On-device transcription** with Whisper, running on your CPU and RAM:
  - *Live*: text appears in the popup while you record.
  - *After recording*: the finished audio is transcribed in one pass.
  - Choose a model size (Tiny, Base or Small) and a language.

## Requirements

- Google Chrome 126 or newer (other Chromium browsers such as Edge may also work).
- Node.js, only for the one-time build step that sets up the transcriber. Recording itself works without it.

## Installation

### 1. Get Node.js (skip if you already have it)

Download from [nodejs.org](https://nodejs.org). If you don't have admin rights, download the **Windows Binary (.zip)** version and extract it to a folder you own.

### 2. Build the transcriber

Open a terminal inside the `tab-recorder` folder (the one containing `manifest.json`) and run:

```
npm install
```

This downloads the Whisper library and builds it into a `dist/` folder. It takes a minute or two.

**No admin rights / Node not on your PATH?** In PowerShell, point the session at your extracted Node folder first (change the path to match yours):

```powershell
$env:Path = "C:\Users\YOURNAME\Downloads\node-v24.22.0-win-x64\node-v24.22.0-win-x64;" + $env:Path
npm.cmd install
```

The warnings about deprecated packages, vulnerabilities and install scripts are safe to ignore. Don't run `npm audit fix --force`.

### 3. Load the extension in Chrome

1. Go to `chrome://extensions`.
2. Turn on **Developer mode** (top right).
3. Click **Load unpacked** and select the `tab-recorder` folder.

Keep the folder where it is. Chrome runs the extension from it. After updating any files, click the reload icon on the extension's card.

## Usage

1. Open the tab you want to record and click the Tab Recorder icon.
2. Pick a mode: **Video + audio**, **Audio only** or **Both files**.
3. Optional: tick **Transcribe on this device**, then choose Live or After recording, a model and a language.
4. Click **Start recording**.
5. Use **Pause / Resume** as needed, then **Stop & save**.

To record another tab at the same time, switch to it, open the popup, and start there. Recordings from other tabs appear under "Also recording" with their own Pause and Stop buttons.

### Where files go

Everything is saved to your browser's download folder, inside `TabRecordings/`:

| File | Contents |
| --- | --- |
| `Title-<time>.mp4` | Video with audio |
| `Title-audio-<time>.mp4` | Audio only |
| `Title-transcript-<time>.txt` | Transcript with `[mm:ss]` timestamps |

## Transcription

- **Runs locally.** Whisper runs in WebAssembly on your CPU, using up to your core count minus one (maximum 8 threads), with the model held in RAM. Your audio never leaves your computer.
- **First use downloads a model** from huggingface.co (roughly 40 MB for Tiny, 80 MB for Base, 250 MB for Small). It is cached by the browser afterwards.
- **Live mode** re-transcribes the most recent audio every 2 seconds. Grey text is still being refined and black text is final. Use **Copy transcript** to copy everything so far. The transcript file is saved when you stop.
- **After recording** mode is usually slightly more accurate because the model sees the whole recording at once.
- **Pick a model your CPU can keep up with.** Tiny and Base suit most machines. If live text lags behind, switch to a smaller model or use After recording.
- **Set the language** if you know it. Auto-detect can wobble on short live windows.
- Long recordings use more RAM during After-recording transcription (about 230 MB per hour of audio).

## Troubleshooting

| Problem | What to try |
| --- | --- |
| "Extension has not been invoked for the current page" | Chrome only lets the extension capture a tab after you open the popup on that tab. Click the icon while that tab is active. |
| "Transcriber isn't built yet" | Run `npm install` in the extension folder, then reload the extension. |
| The model never loads | Check that your network allows huggingface.co. You can also try removing the two `cross_origin_*` lines from `manifest.json` and reloading (it then runs on one thread, which is slower). |
| Files are `.webm` instead of `.mp4` | Your browser doesn't support MP4 recording, so it fell back to WebM. Update Chrome to 126 or newer. |
| Black video from a streaming site | Protected (DRM) video can't be captured. |
| Something else is broken | On `chrome://extensions`, open the extension's **Errors** page or click **service worker**, and check the red messages. |

## Project structure

| File | Purpose |
| --- | --- |
| `manifest.json` | Extension configuration and permissions |
| `popup.html`, `popup.js` | The popup window |
| `background.js` | Toolbar badge, downloads and the recordings list |
| `offscreen.html`, `offscreen.js` | Does the actual recording and drives transcription |
| `pcm-worklet.js` | Taps the tab's audio for live transcription |
| `transcribe-worker.js` | Whisper worker (bundled into `dist/` by the build) |
| `build.mjs`, `package.json` | Build setup for the transcriber |

## Privacy

Recordings and transcripts are created and stored only on your computer. The extension makes one kind of network request: downloading the Whisper model files from huggingface.co the first time you use each model size.

## Limitations

- Recording a tab requires opening the extension on that tab first (a Chrome rule).
- Tabs with protected (DRM) video may record as black.
- Chrome shows its own recording indicator while a tab is being captured, and this can't be turned off.
