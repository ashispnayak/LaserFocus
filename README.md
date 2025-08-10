# LaserFocus — Chrome Extension

Turn any video into active learning. LaserFocus captures **tab audio** (not mic), transcribes it, and periodically pauses playback to ask **quick MCQs** so you stay engaged.

> **Free tier:** 100 minutes & 40 questions per month.

## Overview
- Works on YouTube, Udemy, Udacity, Coursera, SANS, LinkedIn Learning, and most HTML5 players
- Randomized answer options
- Checkpoint resume (on wrong answer, jump back and rewatch)
- Background-safe (keeps running if the popup closes)
- Privacy-first: tab audio only, no mic, no selling data

---

## Architecture (high level)

```mermaid
flowchart LR
  A[User clicks Start in popup] --> B[Service worker]
  B --> C[tabCapture: capture tab audio]
  C --> D[Offscreen document]
  D --> E[Supabase Edge Function]
  E -->|Audio| F[Transcription API]
  F -->|Transcript| E
  E -->|Prompt+Transcript| G[LLM API]
  G -->|MCQs JSON| E --> H[Service worker]
  H --> I[Content script overlay]
  I -->|Pause/Resume/Seek| J[Video element on page]
```

### Event flow (sequence)

```mermaid
sequenceDiagram
  participant U as User
  participant P as Popup
  participant SW as Service Worker
  participant OS as Offscreen
  participant SB as Supabase Fn
  participant DG as Transcriber
  participant PX as LLM
  participant CS as Content Script
  participant VID as Video

  U->>P: Click "Start"
  P->>SW: startListening(interval, questionCount)
  SW->>SW: chrome.tabCapture.capture (tab audio)
  SW->>OS: stream audio chunks
  OS->>SB: POST /transcribe (audio)
  SB->>DG: Transcribe audio
  DG-->>SB: transcript
  SB->>PX: Generate MCQs (prompt + transcript)
  PX-->>SB: MCQs JSON
  SB-->>SW: MCQs
  SW->>CS: Inject overlay + question
  CS->>VID: pause()
  U->>CS: Answer MCQ
  CS->>VID: correct? resume() : seek(checkpoint)
  U->>P: Click "Stop"
  P->>SW: stopListening()
  SW->>OS: teardown
```

---

## Data & Privacy (short)
- **Collected:** tab audio from the active tab (after user click), and minimal **usage counters** (minutes/questions) to enforce the free tier.
- **Processors:** **Transcriber API** (transcription), **LLM API** (MCQ generation).
- **Not stored:** LaserFocus does **not** store audio/transcripts after processing.
- **Not collected:** personal info, passwords, browsing history, keystrokes, microphone audio.
- **Security:** All traffic over HTTPS; least-privilege permissions.

**Privacy Policy:** See [`privacy.html`](./docs/privacy.html) (publicly served via GitHub Pages).

---

## Permissions (why we need them)
- `tabCapture` — capture **tab audio only** after user action
- `activeTab`, `scripting` — inject overlay UI, pause/resume/seek video on the current page
- `offscreen` — keep transcription working if the popup closes
- `storage` — remember user settings & quota counters
- `tabs`/`notifications`/`alarms` — timing & active tab control (if enabled)

---

## Screenshots
> See the Chrome Web Store listing for the latest screenshots.

---

## Contact
Questions or feedback: **ashispnayak@gmail.com.com**
