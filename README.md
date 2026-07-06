# SpeakElevate

*Elevate your English. Empower your voice.*

### ▶ Try it live: **https://sivakalepu.github.io/Professional_English/**

A single-file, offline English-learning web app for a Telugu-medium learner working toward professional, pro-level English. No install, no build, no internet — just open the link above, or download `SpeakElevate.html` and open it in any browser.

Orchestrated by **Sivakumar Kalepu**.

## Features

- **Staged phases:** 📘 Grammar · 📗 Vocabulary · 🎤 Speaking · 🎬 Situations · 💼 Professional — navigated from a game-style journey board on the home dashboard.
- **Grammar (deep):** 20 lessons in beginner order (sentence basics → conditionals), each with bulleted rules, when-to-use explanations, examples, a Telugu-speaker "common mistake" card, and a quiz.
- **Vocabulary:** 38 category lessons (~930 words) with meanings, register/when-to-use notes, collocations, misuse cards, and dialogues.
- **Speaking:** 12 lessons from sounds → sentences → conversation, including shadowing practice.
- **Situations:** 16 real-life scenes (restaurant, airport, workplace, conference, hotel, doctor, bank, shopping, phone, taxi, immigration, pharmacy, tech support, networking, renting, courier) with staged key phrases and role-play dialogues.
- **Professional:** 12 Head-level communication lessons (emails, leading meetings, reporting up, difficult conversations, and more).
- **On-demand Telugu (తెలుగు):** every concept rule, vocabulary meaning, and Situations key phrase has a collapsible Telugu explanation — try in English first, confirm in Telugu.
- **Read-aloud:** 🔊 on any word/sentence via the browser's built-in speech (offline), with an adjustable speed slider and a **voice picker** (choose a US / UK / Indian accent, if your device provides one).
- **Speaking practice:** record your own voice and play it back against the model (offline shadowing). *(Recording needs a secure page — it works on the live HTTPS link above, or when you open the file directly.)*
- **Quizzes + spaced-repetition Review:** wrong answers and "still learning" words resurface in Review (in the ⚙ menu) until you get them right.
- **Progress tracking** saved in the browser, with one-click **backup/restore** (JSON export/import).
- **Dark/light theme**, keyboard navigation (← →), swipe on touch devices, and a responsive mobile layout.

## Usage

Open the [live app](https://sivakalepu.github.io/Professional_English/), or download `SpeakElevate.html` and open it in a modern browser (Chrome or Edge recommended for read-aloud and voice recording). Your progress is saved automatically in the browser; use **⤓ Backup** (in the ⚙ menu) to export it and **⤒ Restore** to bring it back.

## Adding content

All lessons live as data in the `CourseData` object inside `SpeakElevate.html`. Adding a lesson is just appending an object to the relevant phase's `lessons` array — no code changes needed. Card types: `concept`, `vocab`, `example`, `mistake`, `dialogue`, `quiz`.

## Tech

Vanilla HTML/CSS/JavaScript in one self-contained file. Uses only browser-native APIs (Web Speech for read-aloud, MediaRecorder for recording, localStorage for progress). No dependencies, no network calls.
