# ITLadder Reel Studio 100 — MVP

A generic, editable FastAPI web application for importing **up to 100 reels** from the ITLadder Step 1 production DOCX or structured JSON/ZIP, assembling a separate prompt for each clip, optionally requesting **Google Veo 3.1 Fast** through the official `google-genai` SDK, merging completed clips with FFmpeg, and exporting a ZIP.

## Files
- `demo.html` — standalone browser demo, **no Python required**, imports sample or uploaded JSON and simulates progress; **does not generate videos**.
- `Sample/ITLadder_30_Reels_Production_Guide.docx` — actual 30-reel sample input.
- `Sample/sample_30_reels.json` — parsed 30 reels / 90 clips, editable.
- `Sample/output.zip` — sample **prompt input bundle** (DOCX + JSON), **not rendered reels**. Import into the server app.
- `app.py` — FastAPI endpoints and generation worker.
- `static/index.html` — web UI.

## Run on Replit, Codespaces, or your own computer
Install Python 3.11+, FFmpeg executable and dependencies:

```bash
python -m pip install -r requirements.txt
python run.py
```

Visit `http://localhost:8000`. On hosted services forward port **8000**. Use **Load 30-reel sample** or upload a DOCX/JSON/ZIP. Select reel range, Demo or Veo mode, then Generate. Download ZIP after completion. Demo mode produces short colored MP4 placeholders to validate file generation and merging, **not AI reels**. The standalone demo works by opening `demo.html` directly in a browser and only simulates progress.

## Veo credentials and limitations
Provide a Gemini API key in the UI when running a live job, or set `GEMINI_API_KEY` securely in your host's secrets manager. No key is bundled. The key is held in server process memory for the job, not written to output files; **use this only in a trusted private deployment**. Do not expose the server publicly without authentication, quotas, HTTPS and secure secret management. Generation has costs, model access restrictions and may fail on some accounts. The integration uses model `veo-3.1-fast-generate-preview`; verify the available model ID and permitted clip durations for your account.

**Important:** This MVP combines textual master consistency + negative prompts + character lock + clip prompts. It accepts a character image in the UI but **does not transmit it to Veo**; reference-image support requires a separate model-compatible implementation. Start/end prompts are included as text, **not actual frame image inputs**. No voiceover or subtitle synthesis is implemented. Each live clip defaults to 8 seconds (not 10); assembled reels may be shorter than the original 30-second storyboard. Results and character identity are not guaranteed. Failed clips are reported in the output manifest. Demo video rendering requires FFmpeg.

## Input schema

```json
{
  "schema": "itladder-reels-v1",
  "aspect_ratio": "9:16",
  "universal_master": "Global identity instructions",
  "universal_negative": "Avoid artifacts",
  "reels": [
    {"id": 1, "title": "Example", "master": "Reel lock", "negative": "Reel negatives", "clips": [
      {"index": 1, "voiceover": "Narration", "start_frame": "Opening", "end_frame": "Ending", "motion": "Video motion"}
    ]}
  ]
}
```

The DOCX parser supports the heading conventions in the supplied ITLadder guide. Other Step 1 formats may need parser adapters. Maximum 100 reels, maximum 20 clips each, maximum 12MB upload. Jobs run in a background thread and are stored locally under `workspace/` (ignored by Git). Jobs do not persist across server restarts; for production use a durable queue and cloud storage.

## Suggested production upgrades
Add authenticated user accounts, a database, resumable jobs, rate limits and budgets, real character-reference/image-to-video support, end-frame control where supported, TTS, captions, clip normalization and transitions, automatic 30-second timing, and retention/deletion controls.
