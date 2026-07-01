# English Learning SPA Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single self-contained `index.html` English-learning app for a Telugu-medium learner, taking them from grammar foundations to pro-level professional/spoken English, with read-aloud, on-demand Telugu, quizzes, spaced-repetition review, scene dialogues, and offline speaking practice.

**Architecture:** One `index.html` file with inline CSS + vanilla JS. JS is split into isolated namespaced modules (`CourseData`, `Storage`, `TTS`, `Recorder`, `Quiz`, `Router`, `UI`). Content lives as data in `CourseData`; the `UI` module renders from data + persisted state. No framework, no build step, no network dependency.

**Tech Stack:** HTML5, CSS (custom properties, dark-first), vanilla ES6+ JavaScript, Web Speech API (`speechSynthesis`), `MediaRecorder`, `localStorage`.

---

## Conventions for this plan

- **No test runner exists** (single static file, no framework). "Verify" steps mean: open `index.html` in a browser (Chrome/Edge recommended for TTS + MediaRecorder) and confirm the described behavior, plus check the DevTools Console for errors.
- **Git is optional.** The project directory is not a git repo. If you run `git init` first, execute the `git commit` steps as written; otherwise treat each "Commit" step as a **checkpoint** (save the file, confirm the app still loads with no console errors) and skip the git command.
- **Single file:** all code goes into `index.html`. When a task says "add module X," add it inside the existing `<script>` block, after previously-added modules.
- **Canonical names (use exactly, everywhere):**
  - Storage key: `"englishAppState"`
  - Modules: `CourseData`, `Storage`, `TTS`, `Recorder`, `Quiz`, `Router`, `UI`
  - State shape: `{ progress: {}, settings: { theme, voiceRate, voiceName }, review: { wrongQuestions: [], learningWords: [] } }`
  - Card types: `"concept" | "vocab" | "example" | "mistake" | "dialogue" | "quiz"`
  - Progress key per lesson: `progress[phaseId + "/" + lessonId] = { cardsDone: [indices], quizAttempted: bool, quizScore: number }`

---

## File Structure

- **Create:** `index.html` — the entire application (only file in the project).
- Internal layout of `index.html`:
  - `<head>`: meta, `<title>`, inline `<style>` (theme variables + all styles).
  - `<body>`: `<div id="app"></div>` mount point, then one `<script>` containing all modules in this order: `CourseData` → `Storage` → `TTS` → `Recorder` → `Quiz` → `Router` → `UI` → bootstrap call.

---

## Task 1: Scaffold `index.html` shell with dark theme

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create the file skeleton**

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>English Path — Foundations to Pro</title>
  <style>
    :root {
      --bg: #14171c; --panel: #1e232b; --panel-2: #262d37;
      --text: #e8ecf1; --muted: #9aa4b2; --accent: #4A90D9;
      --good: #7BAA50; --warn: #D97A4A; --line: #333b47;
      --radius: 12px; --maxw: 1100px;
    }
    :root[data-theme="light"] {
      --bg: #f4f6f9; --panel: #ffffff; --panel-2: #eef1f5;
      --text: #1b2027; --muted: #5b6675; --line: #d8dee7;
    }
    * { box-sizing: border-box; }
    body { margin: 0; background: var(--bg); color: var(--text);
      font: 16px/1.6 system-ui, "Segoe UI", Roboto, sans-serif; }
    #app { max-width: var(--maxw); margin: 0 auto; padding: 16px; }
    button { font: inherit; cursor: pointer; }
    .telugu { font-family: "Nirmala UI", "Gautami", system-ui; }
  </style>
</head>
<body>
  <div id="app">Loading…</div>
  <script>
    "use strict";
    // Modules are appended below in subsequent tasks.
    // Bootstrap (defined in the final UI task) runs last.
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify**

Open `index.html` in a browser. Expected: dark background, "Loading…" text, no console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: scaffold single-file app shell with dark theme"
```

---

## Task 2: `Storage` module (persistence + settings defaults)

**Files:**
- Modify: `index.html` (add `Storage` in the `<script>`)

- [ ] **Step 1: Add the module**

```js
const Storage = {
  KEY: "englishAppState",
  _defaults() {
    return {
      progress: {},
      settings: { theme: "dark", voiceRate: 0.9, voiceName: null },
      review: { wrongQuestions: [], learningWords: [] }
    };
  },
  load() {
    try {
      const raw = localStorage.getItem(this.KEY);
      if (!raw) return this._defaults();
      return Object.assign(this._defaults(), JSON.parse(raw));
    } catch (e) { return this._defaults(); }
  },
  save(state) {
    try { localStorage.setItem(this.KEY, JSON.stringify(state)); }
    catch (e) { console.warn("save failed", e); }
  }
};
```

- [ ] **Step 2: Verify**

In the browser console run: `Storage.load()` → returns the defaults object. Then `Storage.save({...Storage.load(), settings:{...Storage.load().settings, theme:"light"}}); Storage.load().settings.theme` → `"light"`. Then clear with `localStorage.removeItem("englishAppState")`.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Storage module with defaults"
```

---

## Task 3: `CourseData` structure + minimal seed

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the data object with schema-complete sample content**

This establishes every card type so later renderers have real data to render. Add one grammar lesson using `concept`, `example`, `mistake`, `quiz`; the other phases get placeholder lesson lists filled in Tasks 16–17.

```js
const CourseData = {
  phases: [
    { id: "dashboard", title: "Dashboard", icon: "🏠", lessons: [] },
    { id: "grammar", title: "Grammar", icon: "📘", lessons: [
      { id: "modals-shall-will", title: "Modals: shall / will / would", cards: [
        { type: "concept", title: "shall vs will vs would",
          english: "Use 'will' for future facts and decisions ('I will call you'). 'shall' is formal or for offers/suggestions ('Shall I open the window?'). 'would' is for polite requests, hypotheticals, and past habits ('I would like tea', 'If I had time, I would travel').",
          telugu: "'will' — భవిష్యత్తు, నిర్ణయాలకు. 'shall' — మర్యాదపూర్వకంగా లేదా సలహా/ఆఫర్ ఇచ్చేటప్పుడు. 'would' — మర్యాదపూర్వక అభ్యర్థనలు, ఊహాత్మక వాక్యాలు, గత అలవాట్లకు.",
          examples: [
            { text: "I will finish the report by evening." },
            { text: "Shall we start the meeting?" },
            { text: "I would like a glass of water, please." }
          ]
        },
        { type: "example", pattern: "Offers & suggestions with 'shall'",
          sentences: [
            { text: "Shall I carry that for you?" },
            { text: "Shall we go now?" }
          ]
        },
        { type: "mistake",
          wrong: "I will can help you tomorrow.",
          right: "I will be able to help you tomorrow.",
          whyEn: "Two modals ('will' + 'can') can't stack. Replace the second with 'be able to'.",
          whyTe: "రెండు modal verbs ('will' + 'can') కలిపి వాడకూడదు. 'can' బదులు 'be able to' వాడండి."
        },
        { type: "quiz", questions: [
          { format: "mcq", prompt: "___ I open the window? (offering)",
            options: ["Will", "Shall", "Would"], answer: "Shall",
            explanationEn: "'Shall I…?' is the natural form for offers.",
            explanationTe: "ఆఫర్ ఇచ్చేటప్పుడు 'Shall I…?' సహజమైన రూపం." },
          { format: "choose-sentence", prompt: "Choose the correct sentence:",
            options: ["I will can join.", "I will be able to join."],
            answer: "I will be able to join.",
            explanationEn: "Modals don't stack; use 'be able to'.",
            explanationTe: "Modals ని కలపకూడదు; 'be able to' వాడండి." },
          { format: "fill", prompt: "___ you like some coffee? (polite offer)",
            answer: "Would",
            explanationEn: "'Would you like…?' is the polite offer form.",
            explanationTe: "'Would you like…?' మర్యాదపూర్వక ఆఫర్ రూపం." }
        ]}
      ]}
    ]},
    { id: "vocabulary", title: "Vocabulary", icon: "📗", lessons: [] },
    { id: "speaking", title: "Speaking", icon: "🗣", lessons: [] },
    { id: "situations", title: "Situations", icon: "🎬", lessons: [] },
    { id: "professional", title: "Professional", icon: "💼", lessons: [] },
    { id: "review", title: "Review", icon: "🔁", lessons: [] }
  ],
  phase(id) { return this.phases.find(p => p.id === id); },
  lesson(phaseId, lessonId) {
    const p = this.phase(phaseId);
    return p ? p.lessons.find(l => l.id === lessonId) : null;
  }
};
```

- [ ] **Step 2: Verify**

Console: `CourseData.lesson("grammar","modals-shall-will").cards.length` → `4`.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add CourseData schema with seed grammar lesson"
```

---

## Task 4: `Router` module

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the module**

```js
const Router = {
  state: { phaseId: "dashboard", lessonId: null, cardIndex: 0 },
  goPhase(phaseId) {
    this.state.phaseId = phaseId; this.state.lessonId = null; this.state.cardIndex = 0;
    UI.render();
  },
  openLesson(lessonId) {
    this.state.lessonId = lessonId; this.state.cardIndex = 0; UI.render();
  },
  nextCard() {
    const l = CourseData.lesson(this.state.phaseId, this.state.lessonId);
    if (l && this.state.cardIndex < l.cards.length - 1) { this.state.cardIndex++; UI.render(); }
  },
  prevCard() {
    if (this.state.cardIndex > 0) { this.state.cardIndex--; UI.render(); }
  }
};
```

- [ ] **Step 2: Verify**

Cannot run standalone yet (needs `UI`). Confirm no syntax error: reload page, console clean.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Router module"
```

---

## Task 5: `TTS` module (read-aloud)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the module**

```js
const TTS = {
  supported: ("speechSynthesis" in window),
  _voices: [],
  init() {
    if (!this.supported) return;
    const load = () => { this._voices = speechSynthesis.getVoices(); };
    load();
    speechSynthesis.onvoiceschanged = load;
  },
  _pickVoice(state) {
    if (!this._voices.length) this._voices = speechSynthesis.getVoices();
    const en = this._voices.filter(v => /^en/i.test(v.lang));
    if (state.settings.voiceName) {
      const chosen = en.find(v => v.name === state.settings.voiceName);
      if (chosen) return chosen;
    }
    return en[0] || this._voices[0] || null;
  },
  speak(text, state) {
    if (!this.supported || !text) return;
    speechSynthesis.cancel();
    const u = new SpeechSynthesisUtterance(text);
    const v = this._pickVoice(state);
    if (v) u.voice = v;
    u.lang = (v && v.lang) || "en-US";
    u.rate = state.settings.voiceRate || 0.9;
    speechSynthesis.speak(u);
  },
  stop() { if (this.supported) speechSynthesis.cancel(); }
};
```

- [ ] **Step 2: Verify**

Console: `TTS.init(); TTS.speak("Hello, this is a test.", Storage.load());` → you hear speech. If silent, confirm system has an English voice installed (Windows Settings → Time & language → Speech).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add TTS read-aloud module"
```

---

## Task 6: `Recorder` module (offline record/playback)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the module**

```js
const Recorder = {
  supported: !!(navigator.mediaDevices && window.MediaRecorder),
  _rec: null, _chunks: [], _url: null, _stream: null,
  async start() {
    if (!this.supported) throw new Error("unsupported");
    this._stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    this._chunks = [];
    this._rec = new MediaRecorder(this._stream);
    this._rec.ondataavailable = e => { if (e.data.size) this._chunks.push(e.data); };
    this._rec.start();
  },
  stop() {
    return new Promise(resolve => {
      if (!this._rec) return resolve(null);
      this._rec.onstop = () => {
        const blob = new Blob(this._chunks, { type: "audio/webm" });
        if (this._url) URL.revokeObjectURL(this._url);
        this._url = URL.createObjectURL(blob);
        if (this._stream) this._stream.getTracks().forEach(t => t.stop());
        resolve(this._url);
      };
      this._rec.stop();
    });
  },
  play() { if (this._url) new Audio(this._url).play(); }
};
```

- [ ] **Step 2: Verify**

Console: `await Recorder.start()` (grant mic permission), speak, then `await Recorder.stop()`, then `Recorder.play()` → you hear your own recording. If mic denied, `start()` rejects — that's expected; the UI will handle it in Task 13.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add offline Recorder module"
```

---

## Task 7: `Quiz` module (validation, scoring, review recording)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the module**

```js
const Quiz = {
  check(question, given) {
    const norm = s => String(s || "").trim().toLowerCase();
    return norm(given) === norm(question.answer);
  },
  recordWrong(state, phaseId, lessonId, question) {
    const id = phaseId + "/" + lessonId + "/" + question.prompt;
    if (!state.review.wrongQuestions.some(q => q.id === id)) {
      state.review.wrongQuestions.push({
        id, phaseId, lessonId, prompt: question.prompt,
        format: question.format, options: question.options || null,
        answer: question.answer, explanationEn: question.explanationEn,
        explanationTe: question.explanationTe || null
      });
    }
  },
  graduate(state, id) {
    state.review.wrongQuestions = state.review.wrongQuestions.filter(q => q.id !== id);
  }
};
```

- [ ] **Step 2: Verify**

Console: `Quiz.check({answer:"Shall"}, "shall ")` → `true`; `Quiz.check({answer:"Shall"}, "Will")` → `false`.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Quiz scoring and review-recording module"
```

---

## Task 8: `UI` core — state, header, tab bar, and mount

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the UI skeleton with shared helpers**

```js
const UI = {
  state: null,
  el(tag, props = {}, ...kids) {
    const n = document.createElement(tag);
    for (const [k, v] of Object.entries(props)) {
      if (k === "class") n.className = v;
      else if (k === "html") n.innerHTML = v;
      else if (k.startsWith("on") && typeof v === "function") n.addEventListener(k.slice(2), v);
      else if (v !== null && v !== undefined) n.setAttribute(k, v);
    }
    for (const kid of kids.flat()) if (kid != null)
      n.appendChild(typeof kid === "string" ? document.createTextNode(kid) : kid);
    return n;
  },
  save() { Storage.save(this.state); },
  render() {
    const root = document.getElementById("app");
    root.innerHTML = "";
    root.appendChild(this._header());
    root.appendChild(this._tabs());
    root.appendChild(this._body());
  },
  _header() {
    const rate = this.el("input", { type: "range", min: "0.5", max: "1.25", step: "0.05",
      value: this.state.settings.voiceRate,
      oninput: e => { this.state.settings.voiceRate = parseFloat(e.target.value); this.save(); } });
    const theme = this.el("button", { onclick: () => this._toggleTheme() },
      this.state.settings.theme === "dark" ? "☀ Light" : "🌙 Dark");
    const backup = this.el("button", { onclick: () => this._exportProgress() }, "⤓ Backup");
    const restore = this.el("label", { class: "btnlike" }, "⤒ Restore",
      this.el("input", { type: "file", accept: "application/json", style: "display:none",
        onchange: e => this._importProgress(e) }));
    return this.el("header", { class: "hdr" },
      this.el("h1", {}, "English Path"),
      this.el("div", { class: "hdr-tools" },
        this.el("span", { class: "muted" }, "🔊 speed"), rate, theme, backup, restore));
  },
  _tabs() {
    const bar = this.el("nav", { class: "tabs" });
    CourseData.phases.forEach(p => {
      const active = Router.state.phaseId === p.id;
      bar.appendChild(this.el("button",
        { class: "tab" + (active ? " active" : ""), onclick: () => Router.goPhase(p.id) },
        p.icon + " " + p.title));
    });
    return bar;
  },
  _body() {
    const pid = Router.state.phaseId;
    if (pid === "dashboard") return this._dashboard();
    if (pid === "review") return this._reviewView();
    if (Router.state.lessonId) return this._lessonView();
    return this._phaseView();
  },
  _toggleTheme() {
    this.state.settings.theme = this.state.settings.theme === "dark" ? "light" : "dark";
    document.documentElement.setAttribute("data-theme", this.state.settings.theme);
    this.save(); this.render();
  }
  // _dashboard, _phaseView, _lessonView, _reviewView, card renderers,
  // _exportProgress, _importProgress added in later tasks.
};
```

- [ ] **Step 2: Add styles for header/tabs to the `<style>` block**

```css
.hdr { display:flex; justify-content:space-between; align-items:center; gap:12px; flex-wrap:wrap;
  padding-bottom:12px; border-bottom:1px solid var(--line); margin-bottom:12px; }
.hdr h1 { font-size:20px; margin:0; }
.hdr-tools { display:flex; align-items:center; gap:8px; flex-wrap:wrap; }
.muted { color:var(--muted); font-size:13px; }
.tabs { display:flex; gap:6px; overflow-x:auto; padding-bottom:10px; }
.tab { background:var(--panel); color:var(--text); border:1px solid var(--line);
  border-radius:999px; padding:8px 14px; white-space:nowrap; }
.tab.active { background:var(--accent); border-color:var(--accent); color:#fff; }
button, .btnlike { background:var(--panel); color:var(--text); border:1px solid var(--line);
  border-radius:8px; padding:6px 10px; }
```

- [ ] **Step 3: Add bootstrap at the end of the `<script>`**

```js
UI.state = Storage.load();
document.documentElement.setAttribute("data-theme", UI.state.settings.theme);
TTS.init();
UI.render();
```

Also make placeholder methods so render doesn't crash before later tasks. Add temporary stubs inside `UI` (remove as later tasks implement them):

```js
  ,_dashboard() { return this.el("div", {}, "Dashboard — coming in a later task"); }
  ,_phaseView() { return this.el("div", {}, "Phase view — coming in a later task"); }
  ,_lessonView() { return this.el("div", {}, "Lesson view — coming in a later task"); }
  ,_reviewView() { return this.el("div", {}, "Review — coming in a later task"); }
  ,_exportProgress() {}
  ,_importProgress() {}
```

- [ ] **Step 4: Verify**

Reload. Expected: header with title + speed slider + theme/backup/restore, a scrollable tab bar (🏠 Dashboard … 🔁 Review), and placeholder body text. Clicking tabs switches the active pill. Theme toggle flips colors and persists across reload. Console clean.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add UI core, header, tabs, theme toggle, bootstrap"
```

---

## Task 9: Shared render helpers — read-aloud button & Telugu reveal

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add helper methods to `UI` (place before card renderers)**

```js
  ,_speakBtn(text) {
    if (!TTS.supported) return document.createTextNode("");
    return this.el("button", { class: "icon", title: "Read aloud",
      onclick: () => TTS.speak(text, this.state) }, "🔊");
  }
  ,_teluguReveal(teText) {
    if (!teText) return document.createTextNode("");
    const body = this.el("div", { class: "te-body telugu", style: "display:none" }, teText);
    const btn = this.el("button", { class: "te-btn",
      onclick: () => { body.style.display = body.style.display === "none" ? "block" : "none"; } },
      "తెలుగులో చూడండి");
    return this.el("div", { class: "te-wrap" }, btn, body);
  }
```

- [ ] **Step 2: Add styles**

```css
.icon { border:none; background:transparent; font-size:16px; padding:2px 6px; }
.te-wrap { margin:8px 0; }
.te-btn { background:var(--panel-2); border:1px dashed var(--accent); color:var(--accent);
  border-radius:8px; padding:4px 10px; font-size:14px; }
.te-body { margin-top:6px; padding:8px 10px; background:var(--panel-2);
  border-left:3px solid var(--accent); border-radius:6px; }
.telugu { font-size:17px; }
```

- [ ] **Step 3: Verify**

Cannot see yet (used by card renderers in Task 10). Reload → console clean.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add read-aloud button and Telugu reveal helpers"
```

---

## Task 10: Phase view (lesson list) + lesson view (card stepper) + concept/example/mistake renderers

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace the `_phaseView` and `_lessonView` stubs and add renderers**

```js
  ,_lessonProgress(phaseId, lessonId) {
    const key = phaseId + "/" + lessonId;
    return this.state.progress[key] || { cardsDone: [], quizAttempted: false, quizScore: 0 };
  }
  ,_phaseView() {
    const phase = CourseData.phase(Router.state.phaseId);
    const wrap = this.el("div", { class: "phase" });
    wrap.appendChild(this.el("h2", {}, phase.icon + " " + phase.title));
    if (!phase.lessons.length) {
      wrap.appendChild(this.el("p", { class: "muted" }, "Lessons coming soon in this phase."));
      return wrap;
    }
    const list = this.el("div", { class: "lesson-list" });
    phase.lessons.forEach(l => {
      const pr = this._lessonProgress(phase.id, l.id);
      const pct = Math.round((pr.cardsDone.length / l.cards.length) * 100);
      const done = pct === 100 && pr.quizAttempted;
      list.appendChild(this.el("button", { class: "lesson-item", onclick: () => Router.openLesson(l.id) },
        this.el("span", {}, (done ? "✓ " : "▸ ") + l.title),
        this.el("span", { class: "bar" }, this.el("span", { class: "bar-fill", style: "width:" + pct + "%" }))));
    });
    wrap.appendChild(list);
    return wrap;
  }
  ,_lessonView() {
    const phase = CourseData.phase(Router.state.phaseId);
    const lesson = CourseData.lesson(phase.id, Router.state.lessonId);
    const idx = Router.state.cardIndex;
    const card = lesson.cards[idx];
    const wrap = this.el("div", { class: "lesson" });
    wrap.appendChild(this.el("button", { class: "back", onclick: () => Router.goPhase(phase.id) }, "← " + phase.title));
    wrap.appendChild(this.el("h2", {}, lesson.title));
    wrap.appendChild(this._renderCard(card, phase.id, lesson.id, idx));
    // stepper
    const dots = this.el("div", { class: "dots" });
    lesson.cards.forEach((_, i) => dots.appendChild(this.el("span", { class: "dot" + (i === idx ? " on" : "") })));
    wrap.appendChild(this.el("div", { class: "stepper" },
      this.el("button", { onclick: () => Router.prevCard(), disabled: idx === 0 ? "" : null }, "◀ Prev"),
      dots,
      this.el("button", { onclick: () => Router.nextCard(), disabled: idx === lesson.cards.length - 1 ? "" : null }, "Next ▶")));
    this._markCardDone(phase.id, lesson.id, idx, lesson.cards.length);
    return wrap;
  }
  ,_markCardDone(phaseId, lessonId, idx, total) {
    const key = phaseId + "/" + lessonId;
    const pr = this._lessonProgress(phaseId, lessonId);
    if (!pr.cardsDone.includes(idx)) pr.cardsDone.push(idx);
    this.state.progress[key] = pr; this.save();
  }
  ,_renderCard(card, phaseId, lessonId, idx) {
    switch (card.type) {
      case "concept": return this._conceptCard(card);
      case "example": return this._exampleCard(card);
      case "mistake": return this._mistakeCard(card);
      case "vocab": return this._vocabCard(card);
      case "dialogue": return this._dialogueCard(card);
      case "quiz": return this._quizCard(card, phaseId, lessonId);
      default: return this.el("div", {}, "Unknown card");
    }
  }
  ,_conceptCard(card) {
    const c = this.el("div", { class: "card" });
    c.appendChild(this.el("h3", {}, card.title));
    c.appendChild(this.el("p", {}, card.english));
    c.appendChild(this._teluguReveal(card.telugu));
    (card.examples || []).forEach(ex => {
      const line = this.el("div", { class: "ex" }, this._speakBtn(ex.text), this.el("span", {}, ex.text));
      c.appendChild(line);
      if (ex.telugu) c.appendChild(this._teluguReveal(ex.telugu));
    });
    return c;
  }
  ,_exampleCard(card) {
    const c = this.el("div", { class: "card" });
    if (card.pattern) c.appendChild(this.el("h3", {}, card.pattern));
    (card.sentences || []).forEach(s => {
      c.appendChild(this.el("div", { class: "ex" }, this._speakBtn(s.text), this.el("span", {}, s.text)));
      if (s.telugu) c.appendChild(this._teluguReveal(s.telugu));
    });
    return c;
  }
  ,_mistakeCard(card) {
    const c = this.el("div", { class: "card mistake" });
    c.appendChild(this.el("div", { class: "wrong" }, "✗ " + card.wrong));
    c.appendChild(this.el("div", { class: "right" }, this._speakBtn(card.right), "✓ " + card.right));
    c.appendChild(this.el("p", { class: "why" }, card.whyEn));
    c.appendChild(this._teluguReveal(card.whyTe));
    return c;
  }
```

- [ ] **Step 2: Add styles**

```css
.lesson-list { display:flex; flex-direction:column; gap:8px; }
.lesson-item { display:flex; justify-content:space-between; align-items:center; gap:12px; text-align:left; }
.bar { width:120px; height:8px; background:var(--panel-2); border-radius:99px; overflow:hidden; }
.bar-fill { display:block; height:100%; background:var(--good); }
.card { background:var(--panel); border:1px solid var(--line); border-radius:var(--radius);
  padding:18px; margin:12px 0; }
.card h3 { margin-top:0; }
.ex { display:flex; align-items:center; gap:6px; margin:6px 0; }
.card.mistake { border-left:4px solid var(--warn); }
.mistake .wrong { color:var(--warn); text-decoration:line-through; }
.mistake .right { color:var(--good); display:flex; align-items:center; gap:6px; }
.mistake .why { color:var(--muted); }
.back { margin-bottom:8px; }
.stepper { display:flex; justify-content:space-between; align-items:center; margin-top:14px; }
.dots { display:flex; gap:6px; }
.dot { width:8px; height:8px; border-radius:50%; background:var(--panel-2); }
.dot.on { background:var(--accent); }
```

- [ ] **Step 3: Verify**

Reload → go to 📘 Grammar → the lesson list shows "▸ Modals: shall / will / would" with a progress bar. Open it → concept card shows text, a working `తెలుగులో చూడండి` toggle, and 🔊 buttons that speak. Next ▶ walks through example → mistake → quiz (quiz shows "Unknown"/placeholder until Task 11). Prev/Next disable at ends; dots track position. Progress bar on the lesson list grows after visiting cards.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: phase list, lesson stepper, concept/example/mistake cards"
```

---

## Task 11: Quiz card renderer

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add `_quizCard` to `UI`**

```js
  ,_quizCard(card, phaseId, lessonId) {
    const c = this.el("div", { class: "card" });
    c.appendChild(this.el("h3", {}, "Quiz"));
    let correct = 0, answered = 0;
    const total = card.questions.length;
    const status = this.el("div", { class: "muted" }, "0 / " + total + " answered");
    card.questions.forEach(q => {
      const qWrap = this.el("div", { class: "q" });
      qWrap.appendChild(this.el("div", { class: "q-prompt" }, q.prompt));
      const feedback = this.el("div", { class: "q-feedback" });
      const commit = given => {
        if (qWrap.dataset.done) return;
        qWrap.dataset.done = "1"; answered++;
        const ok = Quiz.check(q, given);
        if (ok) { correct++; feedback.className = "q-feedback ok"; feedback.textContent = "✓ Correct. " + q.explanationEn; }
        else {
          feedback.className = "q-feedback bad";
          feedback.textContent = "✗ Answer: " + q.answer + ". " + q.explanationEn;
          Quiz.recordWrong(this.state, phaseId, lessonId, q);
        }
        if (q.explanationTe) feedback.appendChild(this._teluguReveal(q.explanationTe));
        status.textContent = answered + " / " + total + " answered";
        this._saveQuizScore(phaseId, lessonId, correct, total);
      };
      if (q.format === "mcq" || q.format === "choose-sentence") {
        q.options.forEach(opt => qWrap.appendChild(
          this.el("button", { class: "opt", onclick: () => commit(opt) }, opt)));
      } else if (q.format === "fill") {
        const inp = this.el("input", { type: "text", class: "fill", placeholder: "type your answer" });
        qWrap.appendChild(inp);
        qWrap.appendChild(this.el("button", { class: "opt", onclick: () => commit(inp.value) }, "Check"));
      }
      qWrap.appendChild(feedback);
      c.appendChild(qWrap);
    });
    c.appendChild(status);
    return c;
  }
  ,_saveQuizScore(phaseId, lessonId, correct, total) {
    const key = phaseId + "/" + lessonId;
    const pr = this._lessonProgress(phaseId, lessonId);
    pr.quizAttempted = true; pr.quizScore = Math.round((correct / total) * 100);
    this.state.progress[key] = pr; this.save();
  }
```

- [ ] **Step 2: Add styles**

```css
.q { border-top:1px solid var(--line); padding:12px 0; }
.q-prompt { font-weight:600; margin-bottom:8px; }
.opt { margin:4px 6px 4px 0; }
.fill { padding:6px 10px; border-radius:8px; border:1px solid var(--line);
  background:var(--panel-2); color:var(--text); }
.q-feedback { margin-top:8px; }
.q-feedback.ok { color:var(--good); }
.q-feedback.bad { color:var(--warn); }
```

- [ ] **Step 3: Verify**

Open the grammar lesson → Next to the quiz. Answer the MCQ correctly (Shall) → green "✓ Correct" + explanation + Telugu toggle. Answer a question wrong → red feedback with correct answer; reload and confirm the wrong question is stored: console `Storage.load().review.wrongQuestions.length` ≥ 1. The "N / total answered" counter updates.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: quiz card with mcq/fill/choose-sentence, scoring, review capture"
```

---

## Task 12: Vocab card + "still learning / got it" rating

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add `_vocabCard` to `UI`**

```js
  ,_vocabCard(card) {
    const c = this.el("div", { class: "card" });
    c.appendChild(this.el("div", { class: "vocab-head" },
      this._speakBtn(card.word),
      this.el("h3", {}, card.word),
      this.el("span", { class: "pos" }, card.pos || "")));
    c.appendChild(this.el("p", {}, card.meaningEn));
    c.appendChild(this._teluguReveal(card.meaningTe));
    (card.examples || []).forEach(ex => {
      c.appendChild(this.el("div", { class: "ex" }, this._speakBtn(ex.text), this.el("span", {}, ex.text)));
    });
    const inLearning = this.state.review.learningWords.some(w => w.word === card.word);
    const rate = this.el("div", { class: "rate" },
      this.el("button", { class: "opt", onclick: () => this._rateWord(card, true) }, "Still learning"),
      this.el("button", { class: "opt", onclick: () => this._rateWord(card, false) }, "Got it ✓"));
    if (inLearning) rate.appendChild(this.el("span", { class: "muted" }, " (in Review)"));
    c.appendChild(rate);
    return c;
  }
  ,_rateWord(card, learning) {
    const list = this.state.review.learningWords;
    const i = list.findIndex(w => w.word === card.word);
    if (learning && i < 0) list.push({ word: card.word, meaningEn: card.meaningEn, meaningTe: card.meaningTe || null });
    if (!learning && i >= 0) list.splice(i, 1);
    this.save(); this.render();
  }
```

- [ ] **Step 2: Add styles**

```css
.vocab-head { display:flex; align-items:center; gap:8px; }
.vocab-head h3 { margin:0; }
.pos { color:var(--muted); font-style:italic; }
.rate { margin-top:12px; }
```

- [ ] **Step 3: Verify**

Temporarily add a vocab card to test (in console): not required — this is exercised by seed content in Task 17. For now confirm no syntax error (reload, console clean). After Task 17, verify: a vocab card shows the word with 🔊, Telugu meaning toggle, "Still learning" adds to `Storage.load().review.learningWords`, "Got it" removes it.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: vocab card with Telugu meaning and learning rating"
```

---

## Task 13: Dialogue card + shadowing/record controls

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add `_dialogueCard` and a reusable shadowing control to `UI`**

```js
  ,_shadowControl(modelText) {
    const btn = this.el("button", { class: "opt" }, "🎙 Record");
    if (!Recorder.supported) return this.el("span", { class: "muted" }, "🎙 (recording not supported)");
    let recording = false;
    const playMine = this.el("button", { class: "opt", style: "display:none", onclick: () => Recorder.play() }, "▶ My voice");
    const playModel = this.el("button", { class: "opt", onclick: () => TTS.speak(modelText, this.state) }, "🔊 Model");
    btn.onclick = async () => {
      if (!recording) {
        try { await Recorder.start(); recording = true; btn.textContent = "⏹ Stop"; }
        catch (e) { btn.textContent = "🎙 (mic blocked)"; btn.disabled = true; }
      } else {
        await Recorder.stop(); recording = false; btn.textContent = "🎙 Record again";
        playMine.style.display = "";
      }
    };
    return this.el("div", { class: "shadow" },
      this.el("span", { class: "muted" }, "Your turn — repeat it:"), playModel, btn, playMine);
  }
  ,_dialogueCard(card) {
    const c = this.el("div", { class: "card" });
    c.appendChild(this.el("h3", {}, "Dialogue"));
    const roleClass = {}; card.roles.forEach((r, i) => roleClass[r] = i === 0 ? "role-a" : "role-b");
    const allText = card.lines.map(l => l.text).join(" ");
    c.appendChild(this.el("button", { class: "opt", onclick: () => TTS.speak(allText, this.state) }, "▶ Play whole conversation"));
    card.lines.forEach(line => {
      c.appendChild(this.el("div", { class: "dline " + roleClass[line.role] },
        this._speakBtn(line.text),
        this.el("b", {}, line.role + ": "),
        this.el("span", {}, line.text)));
    });
    c.appendChild(this._shadowControl(card.lines[0].text));
    return c;
  }
```

- [ ] **Step 2: Add styles**

```css
.dline { padding:6px 8px; border-radius:8px; margin:6px 0; }
.role-a { background:var(--panel-2); }
.role-b { background:transparent; border:1px solid var(--line); }
.shadow { margin-top:12px; display:flex; align-items:center; gap:8px; flex-wrap:wrap;
  border-top:1px dashed var(--line); padding-top:10px; }
```

- [ ] **Step 3: Verify**

Exercised by Situations seed (Task 17). For now reload → console clean. After Task 17: dialogue lines are color-coded by speaker, each has 🔊, "Play whole conversation" reads all lines, 🎙 Record → Stop → "▶ My voice" plays back your recording; mic-denied shows a graceful label.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: dialogue card with shadowing and offline record/playback"
```

---

## Task 14: Dashboard view

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace `_dashboard` stub**

```js
  ,_lastLesson() {
    // most recently touched lesson from progress keys
    const keys = Object.keys(this.state.progress);
    if (!keys.length) return null;
    const key = keys[keys.length - 1];
    const [phaseId, lessonId] = key.split("/");
    return { phaseId, lessonId };
  }
  ,_dashboard() {
    const wrap = this.el("div", { class: "phase" });
    wrap.appendChild(this.el("h2", {}, "🏠 Welcome back"));
    const last = this._lastLesson();
    if (last && CourseData.lesson(last.phaseId, last.lessonId)) {
      const l = CourseData.lesson(last.phaseId, last.lessonId);
      wrap.appendChild(this.el("button", { class: "cta",
        onclick: () => { Router.state.phaseId = last.phaseId; Router.openLesson(last.lessonId); } },
        "▸ Continue: " + l.title));
    }
    // per-phase progress
    const grid = this.el("div", { class: "stat-grid" });
    CourseData.phases.filter(p => !["dashboard", "review"].includes(p.id)).forEach(p => {
      let done = 0;
      p.lessons.forEach(l => {
        const pr = this._lessonProgress(p.id, l.id);
        if (pr.cardsDone.length === l.cards.length && pr.quizAttempted) done++;
      });
      const total = p.lessons.length || 0;
      grid.appendChild(this.el("div", { class: "stat", onclick: () => Router.goPhase(p.id) },
        this.el("div", { class: "stat-icon" }, p.icon),
        this.el("div", {}, p.title),
        this.el("div", { class: "muted" }, total ? (done + " / " + total + " lessons") : "coming soon")));
    });
    wrap.appendChild(grid);
    // review + words stats
    wrap.appendChild(this.el("div", { class: "muted" },
      "Review queue: " + this.state.review.wrongQuestions.length + " questions, " +
      this.state.review.learningWords.length + " words. ",
      this.el("button", { class: "opt", onclick: () => Router.goPhase("review") }, "Go to Review 🔁")));
    return wrap;
  }
```

- [ ] **Step 2: Add styles**

```css
.cta { background:var(--accent); color:#fff; border:none; border-radius:10px;
  padding:12px 18px; font-size:16px; margin:12px 0; }
.stat-grid { display:grid; grid-template-columns:repeat(auto-fill,minmax(150px,1fr)); gap:10px; margin:12px 0; }
.stat { background:var(--panel); border:1px solid var(--line); border-radius:var(--radius);
  padding:14px; cursor:pointer; }
.stat-icon { font-size:24px; }
```

- [ ] **Step 3: Verify**

Reload on Dashboard tab → shows welcome, a "Continue" button if you've opened a lesson, a grid of phase cards with lesson counts (clickable), and the review-queue line. Clicking a phase card switches tabs.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: dashboard with continue, per-phase progress, review stats"
```

---

## Task 15: Review mode

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace `_reviewView` stub**

```js
  ,_reviewView() {
    const wrap = this.el("div", { class: "phase" });
    wrap.appendChild(this.el("h2", {}, "🔁 Review"));
    const wrongs = this.state.review.wrongQuestions;
    const words = this.state.review.learningWords;
    if (!wrongs.length && !words.length) {
      wrap.appendChild(this.el("p", { class: "muted" }, "Nothing to review yet. Wrong quiz answers and words you mark 'Still learning' show up here."));
      return wrap;
    }
    if (wrongs.length) {
      wrap.appendChild(this.el("h3", {}, "Questions to retry (" + wrongs.length + ")"));
      wrongs.slice().forEach(q => {
        const qWrap = this.el("div", { class: "card q" });
        qWrap.appendChild(this.el("div", { class: "q-prompt" }, q.prompt));
        const fb = this.el("div", { class: "q-feedback" });
        const commit = given => {
          if (qWrap.dataset.done) return; qWrap.dataset.done = "1";
          if (Quiz.check(q, given)) {
            fb.className = "q-feedback ok"; fb.textContent = "✓ Correct — graduated out of Review.";
            Quiz.graduate(this.state, q.id); this.save();
          } else { fb.className = "q-feedback bad"; fb.textContent = "✗ Answer: " + q.answer + ". " + q.explanationEn; }
        };
        if (q.options) q.options.forEach(o => qWrap.appendChild(this.el("button", { class: "opt", onclick: () => commit(o) }, o)));
        else {
          const inp = this.el("input", { type: "text", class: "fill" });
          qWrap.appendChild(inp);
          qWrap.appendChild(this.el("button", { class: "opt", onclick: () => commit(inp.value) }, "Check"));
        }
        qWrap.appendChild(fb);
        wrap.appendChild(qWrap);
      });
    }
    if (words.length) {
      wrap.appendChild(this.el("h3", {}, "Words to review (" + words.length + ")"));
      words.slice().forEach(w => {
        const c = this.el("div", { class: "card" });
        c.appendChild(this.el("div", { class: "vocab-head" }, this._speakBtn(w.word), this.el("h3", {}, w.word)));
        c.appendChild(this.el("p", {}, w.meaningEn));
        c.appendChild(this._teluguReveal(w.meaningTe));
        c.appendChild(this.el("button", { class: "opt",
          onclick: () => { this.state.review.learningWords = this.state.review.learningWords.filter(x => x.word !== w.word); this.save(); this.render(); } },
          "Got it ✓ (remove)"));
        wrap.appendChild(c);
      });
    }
    return wrap;
  }
```

- [ ] **Step 2: Verify**

First answer a grammar quiz question wrong (creates a review item). Go to 🔁 Review → the wrong question appears; answer it correctly → "graduated" and it disappears on reload. Mark a word "Still learning" (after Task 17 content exists) → it appears under "Words to review"; "Got it" removes it.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: review mode for wrong questions and learning words"
```

---

## Task 16: Backup / restore + keyboard navigation + mobile styles

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace `_exportProgress` / `_importProgress` stubs**

```js
  ,_exportProgress() {
    const blob = new Blob([JSON.stringify(this.state, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = this.el("a", { href: url, download: "english-path-progress.json" });
    document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
  }
  ,_importProgress(e) {
    const file = e.target.files[0]; if (!file) return;
    const reader = new FileReader();
    reader.onload = () => {
      try {
        const data = JSON.parse(reader.result);
        this.state = Object.assign(Storage._defaults(), data);
        this.save();
        document.documentElement.setAttribute("data-theme", this.state.settings.theme);
        this.render();
        alert("Progress restored.");
      } catch (err) { alert("Could not read that file."); }
    };
    reader.readAsText(file);
  }
```

- [ ] **Step 2: Add keyboard navigation to the bootstrap section**

```js
document.addEventListener("keydown", e => {
  if (["INPUT", "TEXTAREA"].includes(document.activeElement.tagName)) return;
  if (!Router.state.lessonId) return;
  if (e.key === "ArrowRight") Router.nextCard();
  if (e.key === "ArrowLeft") Router.prevCard();
});
```

- [ ] **Step 3: Add mobile styles**

```css
@media (max-width: 640px) {
  .hdr { flex-direction:column; align-items:flex-start; }
  .lesson-item { flex-direction:column; align-items:flex-start; gap:4px; }
  .bar { width:100%; }
  .stepper { gap:8px; }
}
```

- [ ] **Step 4: Verify**

Click ⤓ Backup → a JSON file downloads. Change some progress, then ⤒ Restore that file → alert "Progress restored," state matches. In a lesson, ← / → keys move between cards. Narrow the window to phone width → header stacks, layout stays usable, no horizontal scroll.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: backup/restore, keyboard nav, mobile layout"
```

---

## Task 17: Author Phase 1 grammar content (deep) + seed other phases

**Files:**
- Modify: `index.html` (expand `CourseData.phases`)

This task adds real content only (data, no logic). Follow the exact card shapes from Task 3. Author in priority order; each grammar lesson must include: concept card(s) → example card → **mistake card** → quiz (≥3 questions). Telugu (`telugu`, `meaningTe`, `whyTe`, `explanationTe`) is required on concepts, vocab meanings, Situations key phrases, and quiz "why".

- [ ] **Step 1: Grammar — author the priority cluster first**

Add these lessons to the `grammar` phase (the seed `modals-shall-will` already exists — keep it):
1. `sentence-basics` — Subject–Verb–Object & word order
2. `articles` — a / an / the (emphasize: Telugu has no articles)
3. `present-simple-continuous` — include the classic mistake card `"He is having a car"` → `"He has a car"`
4. `past-tense`
5. `future-forms` — will / going to / shall
6. `perfect-tenses` — present & past perfect
7. `modals-can-may` — can/could, may/might
8. (existing) `modals-shall-will`
9. `modals-must-should` — must / should / have to / ought to

Worked example (use as the pattern for every grammar lesson — this is the `articles` lesson):

```js
{ id: "articles", title: "Articles: a / an / the", cards: [
  { type: "concept", title: "When to use a, an, the",
    english: "'a/an' introduce one non-specific singular thing ('a car', 'an hour'). 'the' points to a specific thing both speaker and listener know ('the car I bought'). Use no article for general plurals/uncountables ('Cars are useful', 'Water is life').",
    telugu: "'a/an' — నిర్దిష్టం కాని ఒక వస్తువు గురించి మొదటిసారి చెప్పేటప్పుడు. 'the' — ఇద్దరికీ తెలిసిన నిర్దిష్ట వస్తువు గురించి. సాధారణ బహువచనం/అగణనీయ నామవాచకాలకు article అవసరం లేదు.",
    examples: [ { text: "I bought a phone. The phone is excellent." },
                { text: "She is an engineer." } ] },
  { type: "example", pattern: "'the' for known/unique things",
    sentences: [ { text: "Please close the door." }, { text: "The sun rises in the east." } ] },
  { type: "mistake", wrong: "I am going to office.", right: "I am going to the office.",
    whyEn: "A specific workplace takes 'the'. Telugu has no articles, so they are easy to drop.",
    whyTe: "నిర్దిష్ట కార్యాలయానికి 'the' వాడాలి. తెలుగులో articles లేవు కాబట్టి వీటిని మర్చిపోవడం సహజం." },
  { type: "quiz", questions: [
    { format: "fill", prompt: "I saw ___ elephant at the zoo.", answer: "an",
      explanationEn: "'an' before a vowel sound.", explanationTe: "అచ్చు ధ్వని ముందు 'an'." },
    { format: "choose-sentence", prompt: "Choose the correct sentence:",
      options: ["I am going to school by the bus.", "I am going to school by bus."],
      answer: "I am going to school by bus.",
      explanationEn: "Fixed transport phrases ('by bus') take no article.",
      explanationTe: "'by bus' లాంటి స్థిర పదబంధాలకు article ఉండదు." },
    { format: "mcq", prompt: "___ water is essential for life.", options: ["The", "A", "(no article)"],
      answer: "(no article)", explanationEn: "General uncountable nouns take no article.",
      explanationTe: "సాధారణ అగణనీయ నామవాచకాలకు article ఉండదు." }
  ]}
]}
```

Author lessons 1–9 following this pattern. (Lessons 10–15 from the spec — prepositions, questions/negatives, agreement, conjunctions, adjectives/adverbs, conditionals — can be added in a later content pass; the app already supports them with zero code changes.)

- [ ] **Step 2: Vocabulary — seed 2 sets**

Add to the `vocabulary` phase, e.g. `everyday-essentials` and `connecting-words`. Each lesson = several `vocab` cards + one `quiz`. Vocab card example:

```js
{ type: "vocab", word: "however", pos: "adverb",
  meaningEn: "used to add a contrasting idea; 'but' in a more formal way",
  meaningTe: "విరుద్ధమైన భావాన్ని జోడించడానికి; 'కానీ' అనే అర్థంలో మరింత మర్యాదపూర్వకంగా.",
  examples: [ { text: "The plan is risky. However, it could work." } ] }
```

- [ ] **Step 3: Speaking — seed 2 lessons**

Add to `speaking`: `pronunciation-basics` and `fluency-fillers`. Use `example` cards with shadowing-friendly lines (the shadow control is attached to dialogue cards; for speaking practice, include a short `dialogue` card so the learner can record). Example filler line set as an `example` card:

```js
{ type: "example", pattern: "Natural fillers to buy thinking time",
  sentences: [ { text: "Let me think about that for a second." },
               { text: "That's a good question." } ] }
```

- [ ] **Step 4: Situations — author 4 full scenes**

Add to `situations`: `restaurant`, `airport`, `workplace`, `conference`. Each scene = `vocab` cards + an `example` card of **key phrases with Telugu** + a `dialogue` card + a `quiz`. Worked example (restaurant, abbreviated — expand similarly):

```js
{ id: "restaurant", title: "At a Restaurant", cards: [
  { type: "example", pattern: "Key phrases (with meaning)",
    sentences: [
      { text: "Could I see the menu, please?", telugu: "మెనూ చూడవచ్చా, దయచేసి?" },
      { text: "I'd like to order now.", telugu: "నేను ఇప్పుడు ఆర్డర్ చేయాలనుకుంటున్నాను." },
      { text: "Could we get the bill, please?", telugu: "బిల్లు తీసుకురాగలరా, దయచేసి?" }
    ] },
  { type: "dialogue", roles: ["Waiter", "You"], lines: [
    { role: "Waiter", text: "Good evening! Are you ready to order?" },
    { role: "You", text: "Yes, I'd like the grilled chicken, please." },
    { role: "Waiter", text: "Anything to drink?" },
    { role: "You", text: "Just water, thank you." }
  ] },
  { type: "quiz", questions: [
    { format: "mcq", prompt: "The waiter asks 'Are you ready to order?' A natural reply is:",
      options: ["Yes, I'd like the grilled chicken.", "I am having hunger.", "Give food."],
      answer: "Yes, I'd like the grilled chicken.",
      explanationEn: "'I'd like…' is the polite, natural ordering form.",
      explanationTe: "'I'd like…' మర్యాదపూర్వక, సహజమైన ఆర్డర్ రూపం." }
  ]}
]}
```

- [ ] **Step 5: Professional — seed 2 lessons**

Add to `professional`: `clear-emails` and `speaking-in-meetings`. Use `concept` + `example` + `quiz`. Example concept:

```js
{ type: "concept", title: "Opening a professional email",
  english: "Start with a clear purpose line. 'I'm writing to…' or 'I wanted to follow up on…' set context immediately. Keep the first sentence about why you're writing.",
  telugu: "స్పష్టమైన ఉద్దేశ్యంతో మొదలుపెట్టండి. 'I'm writing to…' లేదా 'I wanted to follow up on…' వెంటనే సందర్భాన్ని తెలియజేస్తాయి.",
  examples: [ { text: "I'm writing to confirm our meeting on Thursday." } ] }
```

- [ ] **Step 6: Verify**

Reload. Walk every tab: Grammar has the authored lessons (each with mistake card + quiz + Telugu). Vocabulary cards toggle Telugu and rate words. Situations scenes show key phrases with Telugu, a color-coded dialogue with working record/playback, and a quiz. Professional and Speaking render. Dashboard shows updated lesson counts. Console clean throughout.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "content: author Phase 1 grammar cluster + seed vocab/speaking/situations/professional"
```

---

## Task 18: Final verification pass (spec §8)

**Files:**
- Modify: `index.html` (fixes only if issues found)

- [ ] **Step 1: Run the full checklist in a browser**

Confirm each, fixing any failure inline:
- [ ] Each tab renders; card stepper navigates via buttons and ← / → keys.
- [ ] Telugu reveals toggle wherever present (concepts, vocab meanings, scene phrases, quiz "why").
- [ ] Read-aloud speaks; speed slider changes rate; long passages can be re-triggered.
- [ ] Shadowing: record → stop → play-back-my-voice works; mic-denied degrades gracefully; "not supported" shows if unavailable.
- [ ] Quizzes score, show explanations, and record wrong answers to Review.
- [ ] Review resurfaces wrong questions + learning words; correct re-answers / "Got it" graduate them.
- [ ] Progress persists across reload; per-lesson and per-phase bars/counts are correct.
- [ ] Export downloads JSON; Import restores state exactly (incl. theme).
- [ ] Mobile width: no horizontal overflow, controls usable.

- [ ] **Step 2: Commit any fixes**

```bash
git add index.html
git commit -m "fix: address issues found in final verification pass"
```

---

## Notes for the implementer

- Keep all logic modules unchanged when adding content in Task 17 — content is pure data.
- The app must never require the network. If you find yourself adding a CDN link or fetch call, stop — that violates the core constraint.
- Telugu strings above are illustrative; verify they read naturally in Telugu script and adjust wording as a native speaker would. Meaning fidelity matters more than literal translation.
