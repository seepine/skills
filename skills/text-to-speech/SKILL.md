---
name: text-to-speech
description: Text-to-Speech (TTS) using Microsoft Edge's online TTS service via node-edge-tts. Use when the user needs to convert text to speech, generate audio from text, create voice recordings, or any TTS task. Also use when a skill requires audio output.
metadata:
  version: 0.1.0
---

# Text-to-Speech (TTS)

Text-to-Speech using Microsoft Edge's online TTS service via `node-edge-tts`.

## Setup

```bash
npm install -g node-edge-tts
```

## Basic Usage

Convert text to speech and save as MP3:

```bash
npx node-edge-tts -t "Hello world" -f "./output.mp3"
```

## Options

| Option          | Flag                  | Description          | Default                           |
| --------------- | --------------------- | -------------------- | --------------------------------- |
| text (required) | `-t, --text`          | Text to convert      | -                                 |
| output file     | `-f, --filepath`      | Output file path     | `./output.mp3`                    |
| voice           | `-v, --voice`         | Voice name           | `zh-CN-XiaoyiNeural`              |
| language        | `-l, --lang`          | Language code        | `zh-CN`                           |
| output format   | `-o, --outputFormat`  | Audio format         | `audio-24khz-48kbitrate-mono-mp3` |
| pitch           | `--pitch`             | Voice pitch          | `default`                         |
| rate            | `-r, --rate`          | Speech rate          | `default`                         |
| volume          | `--volume`            | Voice volume         | `default`                         |
| save subtitles  | `-s, --saveSubtitles` | Save timing JSON     | `false`                           |
| proxy           | `-p, --proxy`         | HTTP proxy           | -                                 |
| timeout         | `--timeout`           | Request timeout (ms) | `10000`                           |

## Common Voice Options

### Chinese

```bash
# Female voice (Xiaoyi)
npx node-edge-tts -t "你好" -v "zh-CN-XiaoyiNeural"

# Male voice (Yunxi)
npx node-edge-tts -t "你好" -v "zh-CN-YunxiNeural"
```

### English

```bash
# US Female
npx node-edge-tts -t "Hello world" -v "en-US-AriaNeural" -l "en-US"

# UK Female
npx node-edge-tts -t "Hello world" -v "en-GB-SoniaNeural" -l "en-GB"
```

## Adjust Speed and Pitch

```bash
# Faster (use + or - prefix)
npx node-edge-tts -t "快一点" -r "+50%"

# Slower
npx node-edge-tts -t "慢一点" -r "-50%"

# Higher pitch
npx node-edge-tts -t "音调高一点" --pitch "+20%"

# Lower pitch
npx node-edge-tts -t "音调低一点" --pitch "-20%"
```

## Higher Quality Output

```bash
npx node-edge-tts -t "高质量音频" -o "audio-24khz-96kbitrate-mono-mp3"
```

## Save Subtitles (Timing Data)

Generates a JSON file with word-level timestamps:

```bash
npx node-edge-tts -t "Hello world" -s
# Creates output.mp3 and output.json
```

Subtitle JSON format:

```json
[
  { "part": "Hello ", "start": 100, "end": 1287 },
  { "part": "world", "start": 1287, "end": 2500 }
]
```

## Using with Proxy

```bash
npx node-edge-tts -t "语音合成" -p "http://localhost:7890"
```

## Programmatic Usage (Node.js)

```javascript
const { EdgeTTS } = require("node-edge-tts");

const tts = new EdgeTTS({
  voice: "zh-CN-XiaoyiNeural",
  lang: "zh-CN",
  outputFormat: "audio-24khz-48kbitrate-mono-mp3",
  rate: "+10%",
  saveSubtitles: true,
});

await tts.ttsPromise("你好世界", "./output.mp3");
```

## Available Voices Reference

| Language | Voice                | Description     |
| -------- | -------------------- | --------------- |
| zh-CN    | zh-CN-XiaoyiNeural   | Chinese Female  |
| zh-CN    | zh-CN-YunxiNeural    | Chinese Male    |
| zh-CN    | zh-CN-XiaoxiaoNeural | Chinese Female  |
| en-US    | en-US-AriaNeural     | US Female       |
| en-US    | en-US-GuyNeural      | US Male         |
| en-GB    | en-GB-SoniaNeural    | UK Female       |
| en-GB    | en-GB-RyanNeural     | UK Male         |
| ja-JP    | ja-JP-NanamiNeural   | Japanese Female |
| ko-KR    | ko-KR-SunHiNeural    | Korean Female   |

For full voice list, see [Azure AI Speech language and voice support](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/language-support?tabs=tts#text-to-speech-voices).
