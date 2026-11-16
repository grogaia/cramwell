---
name: cramwell
description: Turn lecture slides into an interactive exam-prep study map - every concept in one clickable mindmap with simple explanations, sketches, A-vs-B comparisons, a quiz graded by Claude, mock exam, calculation drills and an optional audio guide. Use for exam preparation, studying a course, study guides from slides or notes (also German - Klausurvorbereitung, Lernzettel, Vorlesungsfolien).
---

# Cramwell: exam-prep study map from lecture slides

*Cram well. Sleep better.*

Build ONE interactive web page that turns a whole course into a study tool: all concepts in one
mindmap organised **by concept, not by lecture session**, every node clickable with a simple
explanation, plus quiz, training and (optionally) audio. The student uses it as their main exam
preparation, so **correctness and full coverage of the slides matter more than anything else**.

Write all student-facing content in the language of the slides. Keep established technical terms in
their original form (e.g. English ML terms in a German course). Add a language toggle only on request.

## 0. Ask first (one short round, then start)

Use the question tool if available, otherwise ask in plain text. Ask only what changes the build and
skip anything the student already said:
1. **Exam format**: question types (define, compare A vs B, apply to a scenario, calculate), are
   bullet-point answers allowed, how hard are calculations (e.g. "only tiny sums with 0 and 1" means
   no complex formulas in drills), exam date.
2. **Extra material**: old exams, an official topic list, the lecturer's "this is exam-relevant"
   hints. These are the best signal for what is core and what the mock exam should look like.
3. **Scope**: all slides, or exclude some sessions.
4. **Audio guide** wanted? (Section 5.)

If the exam is close (a few days), say which parts you build first (phases below) and skip audio
unless asked.

## 1. Build in phases, publish early

Create a task list and ship a usable version as early as possible, then extend:
1. **Map + explanations** (the tool is already useful here, so publish it).
2. **Quiz mode with Claude grading** + progress.
3. **Sketches + comparisons**.
4. **Training hub** (mock exam, drills, weak spots, guided lessons).
5. **Audio guide**.
6. **Verification pass** (section 3) before telling the student it is complete.

Keep the build sources in files (section 4) so later sessions or a reset workspace can continue.

## 2. Read the material completely

- Read **every** page. PDF: extract text per page AND look at rendered pages, because diagrams and
  tables often carry the content. Slides as PPTX: extract text and notes. Scans: OCR. Also accept
  lecture notes, scripts and exercise sheets.
- Keep per-page notes: concepts, definitions, formulas, examples, case studies, lecture number.
- Note emphasis: "exam relevant", repeated slides, summaries, checklists, exercise topics.
- Never invent content. If you add something not on the slides (a standard definition, an intuition),
  mark it on the node (`a` field, shown as "not on the slides").

## 3. Concept tree (the heart of the tool)

Organise by the **structure of the subject**, not by session: usually 5-8 main branches (e.g. the
phases of a process model, or the big topic areas), 3-4 levels deep. A full semester has
~150-250 nodes; a short course fewer. Don't pad.

Rules that made the difference:
- Group methods by what they are FOR (e.g. Unsupervised -> Clustering / Association / Anomaly /
  Dimensionality reduction; Supervised -> Classification / Regression), even if the slides present
  them elsewhere. Students get asked "how is X subdivided?".
- Every method/algorithm the slides mention gets its own node; never let a branch show only one
  example when the slides cover several.
- Mark ~35-45 % of nodes as **core** (exam-essential), guided by old exams and lecturer emphasis.
- **Gap check**: walk your per-page notes and confirm each concept sits in the tree at a sensible
  place.
- **Verification pass** (at the end): re-read each node's explanation, formula and exam answer against
  the slide notes; fix anything that is wrong, overstated or not covered. For a large course, use a
  separate agent for this check if one is available, so the content is not grading itself.

Data (plain JS objects embedded in the page):
```
TREE node: {id, l: label, d: subtitle/translation, h: hue (main branches only), v: lecture no(s),
            t: explanation HTML (simple, 3-8 sentences), f: formula (plain text), x: example,
            k: exam tip, a: "not on the slides" note, r: [related ids], c: [children]}
EXAM = {id: [3-5 short exam-answer bullets]}            // every core node
COMP = [{a, b, rel: confuse|alternative|opposite|complement|partof|sequence,
         s: similarities, d: [[aspect, a-side, b-side]], m: memory hook}]   // 20-40 pairs
```

## 4. The page

### Where it runs
- **Claude app (claude.ai / desktop / mobile) with the Artifact tool**: publish a single HTML Artifact.
  Load the `artifact-design` and `artifact-capabilities` skills before writing it. Capabilities:
  `{sample: {}, db: {}, user: {}}` (Claude for grading/asking, private progress sync at
  `data/users/<uid>/progress`). Audio files go in as supporting `files` (`audio/<id>.mp3`), not as
  assets (assets reject mp3).
- **No Artifact tool (e.g. Claude Code)**: write one self-contained `.html` file (+ `audio/` folder)
  the student opens in a browser. Features that need Claude (grading, ask, free comparisons) fall
  back to **self-check**: reveal the model answer and let the student mark learned/weak; progress
  lives in localStorage only. Say so clearly.

### Build setup
Assemble the page from parts with a tiny script: `shell.html` with placeholders
(`/*DATA*/ /*COMP*/ /*DIAG*/ /*EXAM*/ /*AUDIO*/ /*APP*/`) filled from `data.js`, `comparisons.js`,
`diagrams.js`, `levels.js`, `audio.json`, `app.js`. Edit the parts, never the built file. d3 from a
CDN with a pinned version (e.g. cdnjs `d3/7.9.0`). Theme tokens on `:root` with a dark mode; all
`localStorage` access in try/catch. Keep the page under ~2 MB without audio.

### Map
- Root in the centre, branches split left/right, **aligned columns per depth** (avoids toggles
  overlapping labels), curved links in the branch hue, collapse/expand toggles with child counts,
  level buttons (1/2/3/all), search with breadcrumb results, zoom + fit, keyboard navigation.
- Node badges: learned check, weak marker, core star. Detail nodes slightly faded.

### Explanation panel
- Side panel on normal screens; on wide screens (>= ~1100 px) default to a **centred reading card**
  (~880 px wide, ~74ch text column, 17 px text, dimmed map behind, click backdrop or Esc to close)
  with a toggle back to the side panel. Bottom sheet on phones.
- **A- / A+** font-size buttons (6 steps; scale the panel's children, not its padding), remembered.
- Order: breadcrumbs, branch + lecture no, title, subtitle, audio player, placement line ("part of X",
  plus relations like "often confused with Y" as compare links), core/detail badge, explanation,
  formula, example, sketch, exam short answer (core), exam tip, "not on the slides" note,
  sub-concepts, related, compare picker, "ask Claude about this" box.
- Prev/next buttons and arrow keys walk the tree in DFS order **over visible nodes only**: respect
  the core filter, auto-expand collapsed branches, never land on a hidden node.

### Sketches
Small inline SVGs (viewBox width ~360) wherever a picture helps: process cycles, pyramids, box plot,
histogram vs bar chart, scatter, correlation, matrices, trees, curves, distributions, before/after
transformations, flow diagrams. **Redraw** ideas from the slides in your own simple style; never
embed slide images. CSS classes tied to theme tokens so they work in light and dark. Render every
sketch to PNG and look at it: overlapping labels are the most common defect.

### Compare A vs B
Curated `COMP` pairs render as a table (similarities, differences, memory hook). Any other pair can
be compared on demand via Claude (stream the answer; "use only the lecture content").

### Quiz mode
- Before the student answers, **hide everything that gives the answer away**: explanation, sketch,
  exam answer, sub-concepts, related chips, relation hints, audio. Keep only title, subtitle and
  "part of X".
- Answer box (Ctrl/Cmd+Enter submits), "reveal without answering", **dictation button** (Web Speech
  API in the content language, live interim text). On `not-allowed`, `network` or no support, show a
  hint to use the OS dictation instead: phone keyboard mic, Mac Fn Fn, Windows Win+H. (The Web
  Speech API does not work inside the Claude desktop app, so users there will see the hint.)
- Claude grades with a JSON call. Use this rubric; it fixed "too strict" feedback:
  - `correct`: core idea right and the main points of the reference exam answer covered, even if
    very short, in keywords, synonyms or the student's own words.
  - `mostly`: core idea clearly right, one secondary point missing or imprecise. **Counts as passed.**
  - `partial`: a CENTRAL element is missing or confused.
  - `incorrect`: core idea wrong, missing, or confused with another concept.
  - Never deduct for missing examples, formulas, numbers, exact terminology, order, or detail beyond
    the reference answer. Read charitably; when unsure between two grades pick the better one. Ignore
    spelling, language mixing and speech-recognition errors. Stay within the lecture content.
  - Return `{verdict, good[], missing[], wrong[], model[] (exam-level bullets), tip}`. For passed
    verdicts label `missing` as "just a tip (no deduction)".
- Auto-marking: passed -> learned, otherwise weak. No self-marking while quiz mode is on.
- **Objection** (once per answer): the student explains what they meant; re-grade with the previous
  verdict and the note; upgrade only if the original answer already expressed the idea.
- "Next open concept" skips learned nodes, respects the scope (branch / weak only / core only), wraps
  around and announces when a scope is complete.
- With "core only" on, navigation, quiz pool, header progress (`★ x/N`) and branch progress all count
  **only core nodes**.
- Each grading call uses the student's own Claude usage: only call on an explicit action, keep
  prompts compact (node text + exam answer + student answer).

### Training hub
- **Mock exam**: 8-10 mixed questions in the real exam format (explain, compare, scenario -> which
  method, small calculation), modelled on old exams if provided; points per question, graded in one
  go, score + per-item feedback; explain-items update learned/weak.
- **Calculation drills** at exactly the stated difficulty: generate numbers, check deterministically,
  show the worked solution. Only formulas that are on the slides.
- **Weak-spot overview** per branch (learned / weak / open bars, start a quiz on the weak ones).
- **Guided lessons** per branch: step through the core nodes in order, with progress and a summary.

### Progress & sync
Per-node status with timestamps (`l`/`w`/`o`), last-writer-wins merge, stored locally and (if `db` is
available) in the private db doc so phone and laptop agree. Small sync indicator in the header.

## 5. Audio guide (optional, one snippet per node)

**Write the scripts yourself**; never spend a TTS quota on text generation. Style: a calm museum
audio guide explaining a painting, one narrator, second person. **60-100 words (~35-50 s)** per node:
what it is, why it matters, a concrete example from the course, how it connects to other nodes, what
the exam wants. No symbols or formulas (say "divided by", spell out numbers); keep technical terms.
Branch nodes welcome the listener and preview the branch. Scripts of 15-30 words feel empty: avoid.

**Voices**
- Default: `edge-tts` (pip), free Microsoft neural voices (e.g. `de-DE-FlorianMultilingualNeural`,
  `en-US-AndrewMultilingualNeural`), `rate='-4%'`, ~4 in parallel with retries, then ffmpeg to mono
  64 kbit/s mp3. In a sandbox that routes traffic through a proxy, pass `HTTPS_PROXY` to edge-tts and
  set `SSL_CERT_FILE` to the sandbox CA bundle; the host `speech.platform.bing.com` must be allowed.
  This is an unofficial endpoint: if it fails, say so and offer to skip audio.
- Optional: Gemini TTS with the student's own API key (check current model names; in late 2026
  `gemini-3.1-flash-tts-preview` and `gemini-2.5-flash-preview-tts`, voice e.g. `Charon`, output
  24 kHz 16-bit PCM). Free tier was about **10 requests/day per model**, so prioritise core nodes and
  fill the rest with edge-tts. 3.1 aborts (finishReason SAFETY) above ~80 s of audio: join at most
  ~170 words per request with `[long pause]` between scripts. 2.5 handles ~5 short scripts with the
  instruction "take a two-second breath between sections". Split at the longest silences
  (`ffmpeg silencedetect`), fall back to word-proportional cut points, and reject any split where a
  segment's seconds-per-word ratio is off.
- **Check every file**: 0.3-0.75 s per word, otherwise discard (models sometimes emit long silence).
- Player in the panel: play/pause, seek bar, time, "Auto" (play on next node), Space key; hidden in
  quiz mode until answered.

**API keys**: ask the student to upload the key as a file, never paste it into the chat. Store it
with restricted permissions, never print or log it, strip it from error messages, use it only for
TTS, delete it when done, and remind the student to revoke it in Google AI Studio.

## 6. Test before every publish

Use Playwright or another headless browser: serve d3 locally if the CDN is unreachable, stub web
fonts, mock the Claude `sample` capability with canned grading JSON. If the app code runs inside an
IIFE, drive it through clicks rather than globals. Check: no page errors; screenshots of the map,
panel, reading mode, quiz before/after reveal, hub, light + dark, a 390 px phone; prev/next across all
combinations of quiz x core-only with collapsed branches; progress numbers with and without the core
filter.

## 7. Publish and share responsibly

- Afterwards **always republish to the same URL/file** so the student's synced progress stays. Never
  create a second copy by accident.
- Send only changed `audio/*.mp3` files on updates; untouched supporting files persist.
- Optionally store a `src-backup.json` (base64 tar of the build sources, never keys) next to the page
  so a later session can continue.
- The page contains material derived from the lecturer's slides: keep it private or share it only
  with classmates of the same course, not publicly.
- In the reply: one sentence on what changed, and say honestly what could not be tested live (real
  Claude grading, microphone).

## 8. Iterate with the student

Typical feedback and fixes: grading too strict (rubric), hints visible in quiz mode (hide more),
next-button skipping or landing on hidden nodes (navigate visible nodes only), text too small or off
to the side (reading card, A-/A+), audio too thin (60-100 words), missing sketches (add more). Fix,
test, republish, and keep the explanation short.
