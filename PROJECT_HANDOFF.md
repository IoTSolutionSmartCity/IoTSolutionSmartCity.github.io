# PhysicsEasy Project Handoff

This document summarizes the current state of the PhysicsEasy project so another AI agent or developer can continue the next phase without needing to reconstruct the full chat history.

**Last updated:** after DeepSeek ingestion server (PDF/DOC import, diagram extraction, question/answer save) and question diagram support on the main site.

**Latest session changes (playground ↔ question closed loop):**

- **Playground no longer force-pops a question modal.** A non-intrusive `PlaygroundRecommendationPanel` (in the experiment sidebar) lists tag-linked questions; a triggered physics state (e.g. TIR) just highlights the relevant ones — students choose whether to practice. (`PlaygroundTagModal` removed.)
- **Question pages show optional related experiments** via `RelatedExperimentsPanel` (`components/questions/RelatedExperimentsPanel.tsx`) — matched on the question's tags/concepts, clearly marked 選做.
- **Shared matcher** `lib/playground/recommendations.ts` + canonical `getQuestionTagIds` / `getQuestionsByClosedLoopTag` in `lib/content/closedLoopTags.ts` (the two divergent copies now delegate to it).
- **Two new optics experiments**: `optics-lens` (透鏡成像, primaryTagId `convex-lens`) and `optics-mirror` (平面鏡成像, primaryTagId `plane-mirror`) — see `lib/playground/experiments/`.
- **`validateContent` extended**: tag-registry integrity, experiment `primaryTagId` resolution, question `categoryId`/`subtopicId` consistency, and `closedLoopTags` resolution.
- **Ollama tutor degrades gracefully** (see caveat #5).

---

## Project Vision

PhysicsEasy is a specialized **HKDSE Physics** learning platform for secondary students (繁體中文優先).

Students should learn from **concept linking and mistakes**, not formula memorization:

- HKDSE two-layer topic navigation (必修課題 / 選修課題)
- Interactive questions with clickable concept highlights → side-panel notes
- Rich notes with images, tables, fill-in-the-blank interactivity
- Concept-level mistake tracking (localStorage)
- Past paper download center
- High-fidelity 3D simulations (React Three Fiber)

**Language expectation:** Traditional Chinese for UI, categories, student-facing notes, and new questions. Some early Mechanics MVP content is still English.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js 16 App Router, React 19, TypeScript |
| Styling | Tailwind CSS 4, shadcn-style UI primitives |
| 3D | Three.js, @react-three/fiber, @react-three/drei |
| Notes rendering | react-markdown, remark-gfm |
| State | Zustand + localStorage persistence |
| Content | JSON (questions, concepts, categories), MDX-style notes |
| AI — ingestion (authoring) | DeepSeek API via standalone Express server (`ingestion-server/`) |
| AI — student tutor (experimental) | Vercel AI SDK + local Ollama (`app/api/chat/route.ts`) |
| Ingestion backend | Express 4, multer, pdf-parse, mammoth, jszip, jsonrepair |
| Tests | Vitest, jsdom |

### Scripts

```bash
npm install
npm run dev              # http://localhost:3000
npm run build
npm run start
npm run lint
npm test
npm run validate:content

# Ingestion server (separate process)
npm run ingest:install   # first time
npm run ingest:dev       # http://localhost:8787
npm run ingest:start
```

### Regenerate Pastpaper folders

```powershell
.\scripts\create-pastpaper-tree.ps1
```

---

## Content Inventory (Current)

| Type | Count | Location |
|------|-------|----------|
| Questions | 6+ (bundled) | `content/questions/mechanics.json` (2), `light.json` (4), plus any `ingested-*.json` |
| Ingested answers | 0+ | `content/answers/ingested-*.json` (created by ingestion save) |
| Ingested diagram images | 0+ | `public/questions/ingested/{setId}/` |
| Concepts | 17 | `content/concepts/concepts.json` |
| Notes (site) | 9 | `content/notes/mechanics/` (4), `wave-motion/` (5) |
| Categories | 9 main + subtopics | `content/categories.json` |
| Simulations | 2 | `content/simulations/simulations.json` |
| Note images (served) | 23 | `public/notes/light/` |
| Raw PDF export (source) | 1 set | `content/notes/光notes.md/` |

Validation output (expected):

```text
Validated 6 questions, 17 concepts, 9 notes, and 2 simulations.
```

---

## Git And Repository Size

Large folders that should **not** be committed to GitHub:

- `node_modules/` (~660 MB)
- `.next/` (~360 MB)
- `Pastpaper/**/*.pdf` (if added later)

Current `.gitignore`:

```text
node_modules/
.next/
out/
build/
.env*.local
.DS_Store
deepseek_config.json
ingestion-server/node_modules/
ingestion-server/data/
```

**Recommended additions** if PDFs are stored locally:

```text
Pastpaper/**/*.pdf
```

**Note images:** `public/notes/light/` (~23 JPGs) are intentionally committed so notes render on deploy. Keep image count manageable or move to CDN later.

**Credentials:** Copy `deepseek_config.example.json` → `deepseek_config.json` for the ingestion server. Never commit the real API key.

**Session cache:** `ingestion-server/data/sessions/` stores temporary extracted images during upload/preview. Gitignored.

---

## Directory Structure (Key Paths)

```text
app/
  page.tsx                          Homepage + TopicNavigation
  pastpapers/page.tsx               Download hub
  questions/                        Question list + viewer
  notes/[noteId]/page.tsx           Full note pages
  topics/[categoryId]/              Category + subtopic pages
  learn/                            Knowledge base index
  simulations/[simulationId]/       3D canvas pages
  api/chat/route.ts                 Ollama RAG chat (experimental)

ingestion-server/                   DeepSeek content import workspace
  server/index.js                   Express on port 8787
  server/extractAssets.js           PDF/DOC/DOCX text + image extraction
  server/saveQuestions.js           Write questions/answers/images to main repo
  server/routes/notes.js            POST /api/notes/format
  server/routes/questions.js        POST /api/questions/parse, /save
  server/routes/files.js            POST /api/files/extract
  public/index.html                 Tabbed ingestion UI (Tailwind CDN)

components/
  layout/AppShell.tsx               TC sidebar (+ link to localhost:8787 AI 匯入)
  layout/TopicNavigation.tsx        Two-layer HKDSE nav
  layout/SidePanel.tsx              Question → note panel
  questions/QuestionViewer.tsx      Highlights + MC/LQ
  notes/NotePreview.tsx             Note page wrapper
  notes/RichNoteContent.tsx         Markdown + images + blanks + TOC
  notes/InteractiveBlank.tsx        Fill-in-the-blank
  notes/NoteImage.tsx               Click-to-zoom images
  pastpapers/PastpaperHub.tsx       Filter + download UI
  simulations/                      Canvas + scenes

content/
  categories.json                   HKDSE syllabus (TC)
  concepts/concepts.json
  questions/mechanics.json
  questions/light.json
  questions/ingested-*.json         Created by ingestion save (optional)
  answers/ingested-*.json           Answer sheets / model answers per set
  notes/mechanics/*.mdx             Early English MVP notes
  notes/wave-motion/*.mdx           Light/optics notes (TC)
  notes/光notes.md/                 Raw PDF→MD export (source only)
  simulations/simulations.json

public/notes/light/                 Served note images
public/questions/ingested/          Served question diagram images

deepseek_config.example.json        Template for DeepSeek API credentials
deepseek_config.json                Local credentials (gitignored)

Pastpaper/[Year]/Paper1|Paper2|Answer/

lib/content/                        Loaders, parsers, validation
lib/content/getQuestions.server.ts  Server-only: loads all content/questions/*.json
lib/content/getQuestions.ts         Client-safe: bundled mechanics + light only
lib/pastpapers/getPastpapers.ts
lib/simulations/canvasConfig.ts
stores/learningProgressStore.ts
scripts/create-pastpaper-tree.ps1
```

---

## Implemented Features

### 1. HKDSE Two-Layer Navigation

- Schema: `content/categories.json`
- Loader: `lib/content/getCategories.ts`
- UI: `components/layout/TopicNavigation.tsx` on homepage
- Routes: `/topics/[categoryId]`, `/topics/[categoryId]/[subtopicSlug]`

Example drill-down:

```text
波動 → 反射與折射 → /topics/wave-motion/reflection-refraction
```

Subtopic page for `reflection-refraction` links to `/notes/light-notes`.

### 2. Interactive Question Viewer

- Components: `QuestionViewer`, `HighlightedText`, `AnswerInput`, `ExplanationPanel`
- Highlight parser: `lib/content/highlightSegments.ts`
- Rule: if a phrase appears **more than once** in a prompt, use `occurrence` or `range` — validation fails on ambiguity
- **Ingested question fields (optional):** `diagramSrc`, `diagramAlt`, `requiresStudentDiagram`, `modelAnswer`, `ingestionTags`
- Loader: server pages use `getQuestions.server.ts` (all JSON files); client components use bundled `getQuestions.ts`

**Question sets:**

| File | IDs | Language | categoryId |
|------|-----|----------|------------|
| `mechanics.json` | 2 | English (MVP) | not set |
| `light.json` | 4 | Traditional Chinese | `wave-motion` / `reflection-refraction` |

Light question IDs:

- `light-reflection-001`
- `light-refraction-001`
- `light-tir-001`
- `light-lens-001`

### 3. Rich Interactive Notes (NEW)

Converted PDF notes into student-facing web content.

**Pipeline:**

1. Author exports PDF → Markdown + images (e.g. `content/notes/光notes.md/`)
2. Copy images to `public/notes/light/` (web-served)
3. Restructure as MDX with frontmatter in `content/notes/wave-motion/`
4. Use `[[blank:答案|提示]]` syntax for interactive fill-in-the-blank
5. Register concepts in `concepts.json` and link questions in `light.json`

**Renderer:** `components/notes/RichNoteContent.tsx`

- react-markdown + remark-gfm (tables, lists, headings)
- Section TOC sidebar (from `## Heading {#anchor-id}`)
- Click-to-zoom images via `NoteImage`
- Interactive blanks via `InteractiveBlank`
- Related practice questions via `RelatedQuestionsPanel`

**Loader helpers:**

- `lib/content/getNotes.ts` — scans all `.mdx` recursively
- `lib/content/getQuestionsByNote.ts` — questions linked to a noteId
- `lib/content/parseNote.ts` — frontmatter parser (supports `categoryId`, `subtopicId`, `relatedQuestionIds`)

**Light/optics notes (wave-motion):**

| noteId | Title | Route |
|--------|-------|-------|
| `light-notes` | 光學總整理 (full, images + blanks) | `/notes/light-notes` |
| `light-reflection` | 光的反射 | `/notes/light-reflection` |
| `light-refraction` | 光的折射 | `/notes/light-refraction` |
| `total-internal-reflection` | 全內反射 | `/notes/total-internal-reflection` |
| `lenses` | 透鏡成像 | `/notes/lenses` |

**MDX frontmatter example:**

```yaml
---
id: light-notes
title: 光學總整理
topicId: wave-motion
categoryId: wave-motion
subtopicId: reflection-refraction
conceptIds: em-spectrum, light-reflection, plane-mirror, ...
summary: 電磁波譜、反射、折射、全內反射、透鏡成像與透鏡公式。
relatedQuestionIds: light-reflection-001, light-refraction-001, ...
---
```

**Image path in MDX:**

```markdown
![反射定律](/notes/light/2b5c66ee-be04-4fa8-a828-f885bccfcd87-1_358_744_2352_552.jpg)
```

**Interactive blank syntax:**

```markdown
振蕩方向 [[blank:垂直|與波動方向]] 於波動方向
```

### 4. Concept Registry

`content/concepts/concepts.json` — **17 concepts**

Mechanics (English titles, legacy):

- inclined-plane, vector-components, newtons-second-law, weight-vs-component, normal-force, resultant-force, vectors

Wave / light (Traditional Chinese titles):

- em-spectrum, light-reflection, plane-mirror, light-refraction, total-internal-reflection, optical-fiber, convex-lens, concave-lens, lens-formula, magnification

Each concept links to `noteIds[]` for side-panel and recommendation use.

### 5. Pastpaper System

**Folder structure (2012–2026):**

```text
Pastpaper/[Year]/Paper1/
Pastpaper/[Year]/Paper2/
Pastpaper/[Year]/Answer/
```

Paper types in code (`lib/types/pastpaper.ts`):

- `Paper1` → 卷一
- `Paper2` → 卷二 (single folder; E1–E4 not split — same physical paper)
- `Answer` → 評卷參考

**Download hub:** `/pastpapers` — search, year filter, paper type filter, expandable year tables, single-file download.

**Not yet implemented:** `content/pastpapers/manifest.json`, topic tagging, full-year ZIP download API.

### 6. Learning Tracker

- Store: `stores/learningProgressStore.ts` (Zustand + localStorage, SSR-safe hydration)
- Tracks: MC attempts, misconception conceptIds, note opens, simulation opens, derived mastery scores
- Shows weak-topic suggestions on question page after hydration

Future: attach `categoryId` / `subtopicId` to attempts for HKDSE-level recommendations.

### 7. 3D Simulations

Registry: `lib/simulations/registry.ts`

Scenes:

- `forces-on-incline-3d` → `ForcesOnInclineScene` (delta-time physics, shadows)
- `vector-addition-3d` → `VectorAdditionScene` (placeholder)

Canvas blueprint: `lib/simulations/canvasConfig.ts`

```tsx
<Canvas shadows gl={{ antialias: true }} dpr={[1, 2]}>
  <ambientLight intensity={0.6} />
  <directionalLight castShadow position={[10, 12, 8]} />
  <gridHelper />
</Canvas>
```

### 8. AI Systems — What Each Model Does

There are **two separate AI integrations**. They use different models, runtimes, and purposes.

#### A. DeepSeek API — Content Ingestion (Authoring Tool)

**Where:** `ingestion-server/` at **http://localhost:8787** (nav: **AI 匯入** in AppShell)

**Model:** `deepseek-chat` (configured in `deepseek_config.json`)

**Credentials:** Server reads `deepseek_config.json` or `DEEPSEEK_API_KEY` — **never exposed to the browser**

**What it does today:**

| Pipeline | Input | Output |
|----------|-------|--------|
| **Notes formatting** | Raw text (paste or extracted from PDF/DOC/DOCX) | Tailwind HTML fragment for preview |
| **Question parsing** | Question text + optional answer sheet | JSON array: MC/LQ stems, choices, correct answers, tags, linked `imageIds` |
| **Question save** | Parsed JSON + set name + topic | Writes `content/questions/ingested-{setId}.json`, `content/answers/ingested-{setId}.json`, copies diagrams to `public/questions/ingested/{setId}/` |

**File extraction (no AI):** PDF/DOC/DOCX upload → server extracts text + embedded images into a session (`/api/files/extract`). Images are served at `/api/assets/{sessionId}/{fileName}` during preview.

**Parsing safeguards:** JSON mode + `jsonrepair` fallback; min 8192 `max_tokens` for large papers.

**Not yet implemented:** Save formatted notes to `content/notes/`; auto-register ingested questions in `concepts.json`; highlight generation for ingested questions.

#### B. Ollama — Student AI Tutor (Experimental)

**Where:** `app/api/chat/route.ts` → `StudentAICompanion` on `/questions/[questionId]`

**Model:** `deepseek-r1:14b` via local Ollama (not the cloud DeepSeek API)

**What it does today:**

- Accepts student chat messages
- Retrieves up to 3 relevant note snippets from `content/notes/**/*.mdx` (keyword/RAG-style)
- Streams a Traditional Chinese physics tutor reply grounded in note content

**Requires:** Local Ollama running with the model pulled. **Not production-ready** without deployment config.

---

### 9. DeepSeek Ingestion Server (NEW)

Standalone Express workspace for bulk import of past papers, worksheets, and raw notes.

**Run:**

```bash
npm run ingest:install
npm run ingest:dev    # http://localhost:8787
```

**UI tabs:**

1. **筆記格式化** — paste/upload → formatted Tailwind preview
2. **題目解析與標籤** — separate question + answer upload → flashcard preview → **儲存題目與答案**

**Supported uploads:**

| Format | Text | Embedded diagrams |
|--------|------|-------------------|
| PDF | Yes | Yes (`pdf-parse` getImage) |
| DOCX | Yes | Yes (`word/media/` via JSZip) |
| DOC | Yes | Text only |

**Key API routes:**

```text
GET  /api/health
GET  /api/assets/:sessionId/:fileName
POST /api/files/extract
POST /api/notes/format
POST /api/questions/parse
POST /api/questions/save
```

**After save:** Restart Next.js dev server so `getQuestions.server.ts` picks up new `ingested-*.json` files.

See `ingestion-server/README.md` for full API table.

---

## Important Routes

```text
/                                           Homepage (TC, topic nav)
/pastpapers                                 歷屆試題下載中心
/questions                                  Question list
/questions/light-reflection-001             Light MC example
/questions/mech-newton-2-incline-001        Mechanics MC example
/learn                                      Knowledge base
/notes/light-notes                          Full interactive light notes ★
/notes/light-reflection                     Short concept note
/topics/wave-motion/reflection-refraction   Subtopic → links to light-notes
/simulations/forces-on-incline-3d           3D simulation
http://localhost:8787                       AI 匯入 (ingestion server, separate process)
```

---

## How To Import Questions (Via Ingestion Server)

1. Start ingestion server: `npm run ingest:dev`
2. Open http://localhost:8787 → **題目解析與標籤**
3. Upload question PDF/DOCX and answer PDF/DOCX (or paste text)
4. Click **解析題目** — verify flashcards, diagrams, tags, correct MC options
5. Enter set name (e.g. `2024-mock-p1`) and topic → **儲存題目與答案**
6. Restart `npm run dev` on the main app
7. View at `/questions` — ingested sets appear alongside bundled questions

**Output files:**

```text
content/questions/ingested-{setId}.json
content/answers/ingested-{setId}.json
public/questions/ingested/{setId}/img-001.png
```

Ingested questions use empty `highlights[]` and `linkedConceptIds[]` until manually enriched.

---

## How To Add A New Note (From PDF Export)

1. Place raw export under `content/notes/[topic-folder]/` (optional archive)
2. Copy images to `public/notes/[topic]/`
3. Create `content/notes/[category-id]/[note-id].mdx` with frontmatter
4. Write body in Markdown; use `/notes/[topic]/image.jpg` paths
5. Add `[[blank:答案|提示]]` for interactive recall
6. Register concepts in `content/concepts/concepts.json`
7. Create or update questions in `content/questions/*.json` with matching `noteId` / `conceptId` highlights
8. Run `npm run validate:content`

**Recommended note folder naming (align with categories.json):**

```text
content/notes/force-motion/
content/notes/wave-motion/
content/notes/electricity-magnetism/
```

Legacy `mechanics/` folder should eventually migrate to `force-motion/`.

---

## Content Validation And Tests

```bash
npm run validate:content   # lib/content/validateContent.ts
npm test                     # tests/content-validation.test.ts, question-viewer.test.tsx
```

Validates:

- Question → concept / note / simulation references
- Highlight text resolution (no ambiguity, no overlap)
- Duplicate IDs
- Note frontmatter required fields

**Not yet validated:**

- `content/categories.json` schema integrity
- `categoryId` / `subtopicId` consistency across questions and notes
- Pastpaper manifest (does not exist yet)
- Every `relatedQuestionIds` entry resolves to a real question

---

## Latest Verified Status

All passed as of last light-notes integration:

```bash
npm run validate:content   # 6 questions, 17 concepts, 9 notes, 2 simulations
npm test                   # 4 tests passed
npm run build              # 67 static pages generated
```

Always rerun before handing off:

```bash
npm run validate:content && npm test && npm run lint && npm run build
```

---

## Recommended Next Steps

### Priority 1: Convert Mechanics Content To Traditional Chinese

- `content/questions/mechanics.json`
- `content/notes/mechanics/*.mdx`
- Rename folder `mechanics/` → `force-motion/`
- Add `categoryId` / `subtopicId` to mechanics questions

### Priority 2: Replicate Light Notes Pipeline For Other Topics

Use `light-notes.mdx` as the template for the next PDF conversion (e.g. 力學、電磁).

Steps per topic: images → public, MDX → content, concepts → registry, questions → JSON.

### Priority 3: Pastpaper Manifest + ZIP Download

Create `content/pastpapers/manifest.json` and wire `/pastpapers` topic filter.

Implement `app/api/pastpapers/bundle/[year]/route.ts` for 整年下載.

### Priority 4: Light 3D Simulations

Add scenes for:

- Reflection / refraction ray diagram (interactive angle)
- Total internal reflection / optical fiber
- Lens ray diagram with variable object distance

Follow `canvasConfig.ts` + `SimulationShell` conventions.

### Priority 5: Extend Validation

- Validate categories.json subtopic IDs
- Validate `relatedQuestionIds` on notes
- Validate light question highlights against TC prompts

### Priority 6: Student UI Polish

- Breadcrumbs: `首頁 > 波動 > 反射與折射 > 光學總整理`
- Homepage cards linking directly to `/notes/light-notes`
- Progress dashboard by HKDSE subtopic
- Mobile sidebar drawer

### Priority 7: Extend Ingestion Pipeline

- Save formatted notes directly to `content/notes/[topic]/[note-id].mdx`
- Auto-generate highlights + `linkedConceptIds` for ingested questions
- Batch/chunk parsing for very large exam papers (>8192 tokens)
- OCR fallback for scanned image-only PDFs

### Priority 8: Unify Or Replace Ollama Tutor (Optional)

Either wire cloud DeepSeek into `StudentAICompanion`, or keep Ollama but document deployment. Scope chat to current note/concept context on all pages, not just questions.

---

## Known Caveats

1. Mechanics MVP content is still **English**; light content is **Traditional Chinese**.
2. `content/notes/光notes.md/` is a **raw source archive** — not loaded by the site directly.
3. Pastpaper topic filtering and ZIP bundle download are **UI-only / not implemented**.
4. **Two AI systems:** cloud DeepSeek (ingestion only) vs local Ollama (student chat) — different models, different configs.
5. `api/chat` requires **local Ollama**. It now **degrades gracefully**: the route probes Ollama (`OLLAMA_HOST`, default `http://127.0.0.1:11434`) and returns a friendly 503 if unreachable, and `StudentAICompanion` shows an "AI 導師暫時離線" notice instead of breaking. Set `STUDENT_TUTOR_ENABLED=false` to hard-disable on deploys without Ollama.
6. Ingested questions lack interactive highlights until manually edited in JSON.
7. Legacy `.doc` uploads extract text only — no diagram extraction.
8. Scanned image-only PDFs may fail text extraction (no OCR yet).
9. Note renderer is **Markdown-based**, not full MDX React components.
10. Mechanics questions lack `categoryId` / `subtopicId`; light questions have them.
11. Highlight phrases must be **unique in prompt text** or use `occurrence` / `range`.
12. After ingestion save, **restart Next.js** to load new question JSON files.
13. **Progress is `localStorage`-only** (`stores/learningProgressStore.ts`): no accounts, no cross-device sync, and clearing the browser wipes all mastery/mistake history. This is the current ceiling — real student accounts need a backend (DB + auth) before progress can persist or be shared across devices.

---

## Best Continuation Prompt For Next AI Agent

```text
You are continuing the PhysicsEasy HKDSE Physics learning platform.

Read PROJECT_HANDOFF.md first.

Current state:
- 6 bundled questions (2 mechanics EN, 4 light TC), 17 concepts, 9 notes, 2 simulations
- Rich notes with images/blanks at /notes/light-notes
- DeepSeek ingestion server at ingestion-server/ (port 8787): PDF/DOC import, diagram extraction, Q+A save
- Pastpaper folders: Pastpaper/[Year]/Paper1|Paper2|Answer/
- Build and validate:content both pass

Main goals:
1. Convert mechanics content to Traditional Chinese; migrate notes/mechanics → force-motion
2. Extend ingestion: note save to MDX, auto-highlights for ingested questions
3. Add more PDF→MDX notes using the light-notes pipeline
4. Create content/pastpapers/manifest.json + ZIP download API
5. Add light/optics 3D ray-diagram simulations
6. Extend validateContent for categories, relatedQuestionIds, ingested question schema

Do not commit node_modules, .next, deepseek_config.json, or large PDFs unless explicitly requested.
Run npm run validate:content, npm test, npm run lint, npm run build before final response.
```
