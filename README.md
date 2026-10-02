<div align="center">

# AI Reel — Shorts Studio

### Turn an idea into a publish-ready short

An automated, Windows-native pipeline for generating vertical videos with AI
scripts, images, voiceover, captions, music, transitions, and publishing.

<a href="mailto:hasibsarkar98@gmail.com"><img src="https://img.shields.io/badge/Email-Contact%20me-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Contact by email"></a>
<a href="https://t.me/zero0000101"><img src="https://img.shields.io/badge/Telegram-Message%20me-26A5E4?style=for-the-badge&logo=telegram&logoColor=white" alt="Message on Telegram"></a>
<a href="https://www.facebook.com/ddo.philosophy"><img src="https://img.shields.io/badge/Facebook-View%20output%20examples-1877F2?style=for-the-badge&logo=facebook&logoColor=white" alt="View output examples on Facebook"></a>

<br>

<details>
<summary><strong>Need a ready-made automation? Show contact QR code</strong></summary>

<br>

<a href="https://t.me/zero0000101">
  <img src="contact-telegram.png" width="220" alt="Scan to message me on Telegram">
</a>

<br>

Scan to discuss a private setup, customization, or ready-to-use deployment.
</details>

<br>

![Pipeline dashboard](screenshots/dashboard.png)

</div>

## Overview

AI Reel is a complete short-form content pipeline. Give it a niche and a topic;
it turns that idea into platform-ready video output at 1080x1920, with optional
publishing to connected channels.

It runs natively on Windows with SQLite as the queue. No Docker, WSL, Redis, or
external queue broker is required.

## Output examples

See finished AI Reel videos and short-form output examples on the Facebook page:

<div align="center">
  <a href="https://www.facebook.com/ddo.philosophy">
    <img src="https://img.shields.io/badge/View%20output%20examples%20on%20Facebook-1877F2?style=for-the-badge&logo=facebook&logoColor=white" alt="View output examples on Facebook">
  </a>
</div>

## What it does

| Stage | What happens |
|---|---|
| **Script** | An LLM writes the hook, scene breakdown, narration, and captions |
| **Images** | ComfyUI generates one visual per scene and output shape |
| **Voiceover** | Kokoro creates speech and Whisper aligns word timestamps |
| **Render** | FFmpeg adds motion, captions, music, transitions, and formatting |
| **Publish** | Optional upload to connected TikTok, YouTube, Instagram, and Meta channels |

Jobs move through a predictable pipeline:

`pending` -> `awaiting_approval` -> `scripted` -> `generating_images` ->
`generating_tts` -> `rendering` -> `done`

Failed jobs are marked `failed` and can be investigated from the dashboard.

## Highlights

- Responsive React dashboard for the full content workflow
- Niche-aware style bibles for consistent visual identity
- Four-link LLM fallback chain with health tracking
- ComfyUI image generation with quality profiles
- Kokoro TTS with Whisper word-level alignment
- FFmpeg rendering with Ken Burns motion, captions, music, and xfades
- SQLite queue and storage with no broker dependency
- Optional OAuth publishing integrations
- Local Windows services and PowerShell startup scripts

## Architecture

```text
React + Vite :3000
        |
        | POST /api/topics
        v
FastAPI :8000 ----> SQLite job_queue
                          |
                          v
                    Pipeline worker
          ____________|_____|____________
         |            |     |            |
       Script       Images  TTS        Render
        LLM        ComfyUI Kokoro     FFmpeg
                                      |
                                      v
                                  Publish
```

## Technology stack

| Area | Technology |
|---|---|
| Web UI | React 18, Vite, Tailwind |
| API | FastAPI, Uvicorn |
| Queue and storage | SQLite |
| Script generation | Kilo gateway, Gemini, Groq, Ollama |
| Image generation | ComfyUI, Qwen-Image GGUF |
| Voice and alignment | Kokoro v0.19 ONNX, Whisper |
| Video | FFmpeg, NVENC when available |
| Publishing | OAuth integrations for supported platforms |

## LLM fallback chain

`src/llm/client.py` tries configured providers in order and falls through when a
provider is unavailable:

1. **Kilo gateway** — free gateway when configured
2. **Gemini** — metered against a daily quota
3. **Groq** — rolls through the configured model list on rate limits
4. **Ollama** — local offline fallback

Provider health is persisted so a failed provider can be skipped temporarily
instead of delaying every new topic. A per-topic model pin can override the
default order.

## Output formats

| Platform | Size | Ratio |
|---|---:|---:|
| TikTok | 1080x1920 | 9:16 |
| YouTube Shorts | 1080x1920 | 9:16 |
| Instagram Reels | 1080x1920 | 9:16 |
| Facebook Reels | 1080x1920 | 9:16 |
| Instagram feed | 1080x1350 | 4:5 |
| Instagram square | 1080x1080 | 1:1 |

Images are generated per output shape, so square content is not simply cropped
from a vertical composition.

## Style bibles

Each niche has a YAML style bible defining palette, materials, lighting,
composition, camera, typography, transitions, music, and sound effects.

Bundled niches:

`science` · `love` · `health` · `quotes` · `personal_finance` · `legal_drama`

Style bibles can be edited from the UI or directly in `style_bibles/`.

## Getting started

Start the local services from PowerShell:

```powershell
.\ai-reel.ps1 -Action start
.\ai-reel.ps1 -Action status
.\ai-reel.ps1 -Action logs
.\ai-reel.ps1 -Action stop
```

Or double-click `Start-AI-Reel.bat`.

| Service | URL |
|---|---|
| Web UI | http://localhost:3000 |
| API docs | http://localhost:8000/docs |
| ComfyUI | http://localhost:8188 |
| Ollama | http://localhost:11434 |

Ollama is started separately when local models are needed.

## Configuration

Copy `.env.example` to `.env` and configure the providers and local services
you plan to use.

| Variable | Purpose |
|---|---|
| `GEMINI_API_KEY` | Gemini access |
| `KILO_API_KEY` / `KILO_MODEL` | Kilo gateway access and model |
| `GROQ_API_KEY` / `GROQ_MODEL` | Groq access and rollover models |
| `COMFYUI_URL` | ComfyUI service URL |
| `DB_PATH` | SQLite database location |
| `OUTPUT_DIR` | Rendered video location |
| `FFMPEG_DIR` | FFmpeg `bin` folder |
| `*_CLIENT_ID` / `*_CLIENT_SECRET` | Publishing OAuth applications |

Keep `.env` and all API credentials private. Token usage is tracked per topic
and aggregated in `cost_tracking`, with a daily request ceiling before Gemini
is called.

## Project layout

```text
ai-reel/
├── src/
│   ├── llm/           LLM chain, prompts, model picker
│   ├── image_gen/     ComfyUI client and quality profiles
│   ├── tts/           Kokoro synthesis and Whisper alignment
│   ├── video/         FFmpeg filtergraph builder
│   ├── publish/       OAuth and platform uploads
│   ├── queue/         Worker, job chain, retry logic
│   ├── db/            SQLite repository and schema
│   └── style_bible/   YAML loader
├── web_ui/
│   ├── backend/app/   FastAPI app, routers, and models
│   └── frontend/src/  React pages and components
├── style_bibles/      Per-niche YAML files
├── tests/             Standalone test scripts
├── data/              SQLite data
├── output/            Rendered videos by topic
├── models/ fonts/ music/
└── ai-reel.ps1        Start, stop, status, and logs
```

## Testing

Run focused test scripts through the project virtual environment:

```powershell
venv\Scripts\python.exe -u tests\test_render_speed.py
venv\Scripts\python.exe -u tests\test_output_shapes.py
```

Run scripts individually for faster feedback; the complete test set can invoke
multiple FFmpeg renders.

## Troubleshooting

**ComfyUI is available but image generation fails** — verify that the required
Qwen model is installed in `models/checkpoints/` and that ComfyUI is using its
own project environment.

**Script generation times out** — configure `GROQ_API_KEY` or `KILO_API_KEY` so
a fast remote provider is available before the Ollama fallback.

**A topic is stuck waiting** — check the service for its current pipeline stage.
The worker retries unavailable stages instead of immediately discarding the job.

**GPU memory is exhausted** — run ComfyUI with `--lowvram --vram-headroom 2`.

## Screenshots

### Topics and ideas

Browse topic history, filter by niche and upload state, and queue new content
from AI-generated ideas.

<p align="center">
  <img src="screenshots/topics.png" alt="Topics page">
  <img src="screenshots/ideas.png" alt="Topic ideas page">
</p>

### Visual configuration

Edit style bibles and manage the pipeline's environment-backed settings.

<p align="center">
  <img src="screenshots/style-bibles.png" alt="Style bibles page">
  <img src="screenshots/settings.png" alt="Settings page">
</p>

### Results and performance

Review published videos and monitor throughput, token usage, platform
distribution, and estimated cost.

<p align="center">
  <img src="screenshots/downloads.png" alt="Downloads page">
  <img src="screenshots/analytics.png" alt="Analytics page">
</p>

## Private project

This repository is maintained as a private project. Please do not publish
credentials, generated media, private customer data, or local environment files.

## Need a ready-made automation?

For private setup, customization, or a ready-to-use deployment, use the contact
buttons at the top of this page:

<div align="center">
<a href="mailto:hasibsarkar98@gmail.com"><img src="https://img.shields.io/badge/Email-Contact%20me-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Contact by email"></a>
<a href="https://t.me/zero0000101"><img src="https://img.shields.io/badge/Telegram-Message%20me-26A5E4?style=for-the-badge&logo=telegram&logoColor=white" alt="Message on Telegram"></a>
</div>
