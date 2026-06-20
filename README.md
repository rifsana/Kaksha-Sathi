# Kaksha Saathi 🎙️

**Apna classroom co-pilot** — a voice-controlled, Hinglish-speaking AI teaching assistant built as a single, self-contained HTML file. Designed for Indian classrooms (Haryana context), it lets a teacher speak a command and get a quiz, a concept explanation, or a hands-on activity guide — all read aloud in natural Hinglish.

No build step, no backend, no install. Open the HTML file in a browser and go.

---

## Features

- **Voice input (STT)** via the Web Speech API, with a toggle between:
  - `हिंदी + EN` (`hi-IN`) — mixed Hindi/English recognition
  - `English only` (`en-IN`)
- **Three modes**, selectable by tapping a card or just saying the right phrase:
  | Mode | Trigger phrase example | What happens |
  |---|---|---|
  | **Voice Quiz** | "Quiz shuru karo photosynthesis par" | Generates 4 MCQs, reads each one aloud, accepts spoken or clicked answers (letter, number, or Hindi/English words like "pehla"/"first"), gives feedback and a running score |
  | **Concept Samjhao** | "Gravity explain karo" | Generates a short Hinglish explanation (~120 words) with a highlighted example box, read aloud, with a "Dobara sunao" (read again) button |
  | **Activity Guide** | "Activity guide chalao volcano model" | Generates a 4-step hands-on classroom activity with a countdown timer ring per step; say "Next" to advance |
- **Text-to-speech (TTS)** via the Web Speech Synthesis API, with a voice picker (prefers `hi-IN` voices when available)
- **AI content generation** via the **Groq API** (free tier, `llama-3.3-70b-versatile`) — the user supplies their own free Groq API key
- **Offline-friendly fallback**: if no API key is set or the request fails, the app falls back to built-in sample quiz/concept/activity content so it's never a dead end
- **Accessible by design**: ARIA live regions, `aria-pressed` states, keyboard-operable quiz options, `prefers-reduced-motion` support
- Fully responsive single-page layout with a chalkboard-inspired visual theme

---

## Getting Started

1. Download `KakshaSathi.html`.
2. Open it directly in any modern browser (Chrome recommended for best Web Speech API support).
3. (Optional but recommended) Get a **free Groq API key** from [console.groq.com/keys](https://console.groq.com/keys) and paste it into the **Settings → Groq API Key** field. This unlocks dynamically generated, topic-specific content instead of the generic sample content.
4. Tap the mic button and speak a command, e.g.:
   - *"Quiz shuru karo photosynthesis par"*
   - *"Gravity explain karo"*
   - *"Activity guide chalao volcano model"*

No server, no `npm install`, no API key required to try the app at all — it just won't be topic-aware without one.

---

## Tech Stack

- **Plain HTML/CSS/JS** — single file, zero dependencies, no framework or build tooling
- **Web Speech API**
  - `SpeechRecognition` for speech-to-text input
  - `speechSynthesis` for text-to-speech output, with queueing (`ttsQueue`) so spoken responses don't overlap
- **Groq Chat Completions API** (`https://api.groq.com/openai/v1/chat/completions`, OpenAI-compatible schema) using `llama-3.3-70b-versatile` for all generative content (quizzes, explanations, activity steps)
- **Google Fonts**: `Kalam` (handwritten chalk-style headings), `Inter` (body text), `JetBrains Mono` (labels/metadata)
- Pure CSS custom properties (`:root` design tokens) for the chalkboard color theme — no CSS framework

---

## Prompt Design

Each mode sends a tailored **system prompt + user prompt** pair to the LLM, with strict output-format constraints so the JS can parse the response reliably:

- **Quiz**: System prompt frames the model as a Hinglish-speaking classroom assistant; user prompt requests *exactly* 4 MCQs as a raw JSON array (`question`, `options[4]`, `correctIndex`, `explanation`), explicitly forbidding markdown or preamble text.
- **Concept**: System prompt enforces a fixed structure — 1–2 sentence core idea → one labeled "Example:" block → 1–2 follow-up sentences — flowing prose only, capped at ~120 words, no markdown headers/bullets.
- **Activity**: System prompt asks for step-by-step, teacher-readable instructions; user prompt requests a JSON array of 4 steps (`text`, `seconds`) with realistic per-step durations (30–180s) for the on-screen countdown timer.

All responses are stripped of stray Markdown code fences before parsing, and every mode has a hardcoded **sample fallback generator** (`sampleQuiz`, `sampleConcept`, `sampleActivity`) so a malformed response, missing key, or network failure never breaks the experience.

---

## Localization / Language Design

Kaksha Saathi is built **Hinglish-first**, not translated after the fact:

- All prompts explicitly instruct the model to use **natural Hinglish** — Hindi grammar with English technical/subject vocabulary — rather than literal transliteration, matching how teachers in Haryana classrooms actually speak.
- **Speech recognition** supports two language modes (`hi-IN` mixed and `en-IN` English-only) so both code-switching and English-only speakers are covered.
- **Intent/topic extraction** (`extractTopic`) understands command connectors in both languages (`par`, `on`, `about`, `पर`, `ke baare mein`, `explain`, `samjhao`, `chalao`) to pull the topic out of a spoken sentence.
- **Quiz answer parsing** accepts spoken answers as letters (A–D), English ordinal words (first/second/third/fourth), or Hindi ordinal words (pehla/doosra/teesra/chautha, ek/do/teen/char), plus a fuzzy content-overlap fallback if none of those match.
- All UI labels, status messages, and TTS feedback strings are written natively in Hinglish (e.g. "Samajh raha hoon...", "Bilkul sahi!", "Time ho gaya!") rather than run through a translation layer.

---

## File Structure

This is intentionally a **single-file app**:

```
KakshaSathi.html
├── <style>   — chalkboard theme, layout, animations, responsive/accessibility rules
├── <body>    — header, mode selector, mic zone, 3 output cards, settings panel
└── <script>  — state, Groq API call, STT/TTS handling, mode logic (quiz/concept/activity)
```

---

## Known Limitations

- Web Speech API support varies by browser (best in Chrome/Edge; limited/no support in Firefox and some mobile browsers).
- Requires an internet connection and a Groq API key for dynamic, topic-specific content — otherwise generic sample content is shown.
- The Groq API key is entered client-side and used directly from the browser; treat it as you would any client-exposed key (don't deploy this publicly with a key baked in).
