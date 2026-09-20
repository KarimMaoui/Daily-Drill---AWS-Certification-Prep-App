# Daily Drill — AWS Certification Prep App

A single-page spaced-repetition (SM-2) drill app for AWS and Anthropic certification prep.
No build step, no backend, no dependencies: one `index.html` plus JSON question decks.
All progress stays in your browser's `localStorage`.

## Quick start (local)

The app fetches its decks from the `decks/` folder, so it needs to be served over
HTTP — opening `index.html` directly with a double-click (`file://`) will not load
the decks (see [Running from file://](#running-from-file)).

Clone, then start any static file server from the repo root:

```bash
git clone https://github.com/KarimMaoui/Daily-Drill---AWS-Certification-Prep-App.git
cd Daily-Drill---AWS-Certification-Prep-App

# Windows: use the py launcher, it is always on PATH when Python is installed
py -m http.server 8000

# macOS / Linux
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

Other equivalent options:

```bash
npx serve .                 # Node
npx http-server -p 8000
php -S localhost:8000
```

Troubleshooting `python: command not found` on Windows: `python` is often absent
from PATH even though Python is installed (and `python3` may resolve to the
Microsoft Store stub, which does nothing). Use `py -m http.server 8000`, or call
the interpreter by full path:

```bash
"/c/Program Files/Python313/python" -m http.server 8000   # Git Bash
"C:\Program Files\Python313\python.exe" -m http.server 8000   # PowerShell / cmd
```

On first load you should see the decks listed in the left sidebar and a "Due" count.
If the sidebar is empty, check the browser console — it usually means the server is
not rooted at the folder containing `index.html`.

Any port works; `8000` is only a convention. Keep in mind that progress is stored
per origin, so switching port later starts you from an empty history (see
[Progress data](#progress-data)).

### Running from `file://`

If you open `index.html` straight from disk, `fetch()` is blocked by the browser and
no decks load. Two fallbacks are built in:

1. **Load deck file** button — pick any `decks/*.json` manually from the file picker.
2. **localStorage cache** — once decks have been loaded over HTTP at least once, they
   are cached and will reload offline, including from `file://`.

Serving over HTTP is still the recommended path.

## Deploying (GitHub Pages)

Because it is pure static files, `Settings → Pages → Deploy from branch → main / (root)`
publishes the app as-is. The relative `decks/` and `assets/` paths work unchanged.

## Repository layout

```
index.html          The whole app: markup, CSS, and JS (no bundler, no dependencies)
decks/*.json        Question decks, fetched at boot
assets/<deck-id>/   Images referenced by cards via `source_images`
```

Deck paths are declared in the `DEFAULT_DECK_PATHS` array near the top of the
`<script>` block in `index.html`. Add a new file to `decks/` **and** to that array
for it to load automatically.

## Included decks

| File | Deck id | Name | Cards |
|---|---|---|---|
| `decks/clf-real-exam.json` | `clf-real-exam` | AWS Certified Cloud Practitioner (CLF-C02) | 549 |
| `decks/saa-c03.json` | `saa-c03` | AWS Certified Solutions Architect Associate (SAA-C03) | 683 |
| `decks/aif-c01.json` | `aif-c01` | AWS Certified AI Practitioner (AIF-C01) | 223 |
| `decks/mla-c01.json` | `mla-c01` | AWS Certified ML Engineer – Associate (MLA-C01) | 218 |
| `decks/knowledge-saa.json` | `knowledge-saa` | SAA Knowledge: Facts & Gotchas | 123 |
| `decks/cheatcode-saa.json` | `cheatcode-saa` | Cheatcode for SAA: service → trigger words | 150 |
| `decks/ccaf-foundations.json` | `ccaf-foundations` | Claude Certified Architect – Foundations | 60 |
| `decks/ccaf-mock3.json` | `ccaf-mock3` | Claude Architect Foundations (CCA-F) – Mock 3 | 60 |
| `decks/ccaf-mock4.json` | `ccaf-mock4` | Claude Architect Foundations (CCA-F) – Mock 4 | 60 |
| `decks/ccaof.json` | `ccaof` | Anthropic Claude – Associate (CCAO-F) | 16 |

## Features

- **Spaced repetition (SM-2)** — cards are scheduled by ease factor and interval;
  status buckets are New / Learning / Review / Mastered, with a "Due" queue.
- **Two study modes** — *Self* (reveal the answer, then self-grade) and *QCM*
  (pick options, get graded automatically).
- **Exam mode** — a shuffled run of 15/30/45/60/70 answerable questions from the
  active deck, answered without feedback, with a score report at the end.
  No countdown timer.
- **Filters** — by deck, by exam domain, and by card status.
- **Streak and daily stats** — reviews per day, accuracy, lifetime reviews.
- **Export / import progress** — download your progress as JSON and restore it on
  another machine or browser. The app nags you to export periodically, since
  `localStorage` is the only store.

### Keyboard shortcuts

| Key | Action |
|---|---|
| `Space` | Show answer |
| `1` | Again (quality 0) |
| `2` | Hard (quality 3) |
| `3` | Good (quality 4) |
| `4` | Easy (quality 5) |

Shortcuts are disabled during Exam mode, which is click-driven on purpose.

## Progress data

- `localStorage` key `daily_drill_state_v1` — SRS state per card, stats, UI selection.
- `localStorage` key `daily_drill_decks_v13` — cached copy of the decks.
- **Export progress** writes `daily-drill-progress-YYYY-MM-DD.json`
  (`{ cards, stats, exported_at, version }`). **Import progress** reads it back.
- **Reset progress** clears SRS state only; decks stay loaded.

Progress is per browser profile and per origin: `http://localhost:8000` and a
GitHub Pages URL do **not** share progress. Use export/import to move between them.

## Deck JSON format

```jsonc
{
  "id": "mla-c01",                 // unique; used as the State key
  "name": "AWS Certified ML Engineer - Associate (MLA-C01)",
  "version": "1.0",
  "language": "en",
  "source": "free-text provenance note",
  "cards": [
    {
      "id": "mla-0001",            // unique within the deck
      "domain": "Deployment and Orchestration",
      "topic": "Model registry versioning",
      "q": "Question text…",
      "options": ["A…", "B…", "C…", "D…"],   // optional: enables QCM/exam mode
      "correct_indices": [2],                 // 0-based
      "multi": false,                         // true = multiple correct answers
      "a": "C. Use the SageMaker Model Registry and model groups…",
      "explanation": "Why this is right…",
      "source_images": ["assets/mla-c01/q005_0.png"],  // optional, relative paths only
      "tags": ["mla-c01", "exam-question"]
    }
  ]
}
```

A deck is accepted only if it has an `id` and a `cards` array. Cards without
`options` + `correct_indices` still work in Self mode but are skipped by
Exam mode. `source_images` paths are sanitized: relative only, no `..`, no
drive letters, no absolute paths.

## License / provenance

Question content is study material assembled for personal exam prep; check each
deck's `source` field before redistributing.
