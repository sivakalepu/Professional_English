# SpeakElevate

*Elevate your English. Empower your voice.*

A single-file, offline English-learning web app for a Telugu-medium learner working toward professional, pro-level English. No install, no build, no internet — just open `index.html` in any browser.

Orchestrated by **Sivakumar Kalepu**.

## Features

- **Staged phases:** 📘 Grammar · 📗 Vocabulary · 🎤 Speaking · 🎬 Situations · 💼 Professional
- **Grammar (deep):** 20 lessons in beginner order (sentence basics → conditionals), each with bulleted rules, examples, a Telugu-speaker "common mistake" card, and a quiz.
- **On-demand Telugu (తెలుగు):** every concept rule, vocabulary meaning, and Situations key phrase has a collapsible Telugu explanation — try in English first, confirm in Telugu.
- **Read-aloud:** 🔊 on any word/sentence via the browser's built-in speech (offline), with an adjustable speed slider.
- **Speaking practice:** record your own voice and play it back against the model (offline shadowing).
- **Situations:** 10 real-life scenes (restaurant, airport, workplace, conference, hotel, doctor, bank, shopping, phone, taxi) with role-play dialogues.
- **Quizzes + spaced-repetition Review:** wrong answers and "still learning" words resurface in the Review tab until you get them right.
- **Progress tracking** saved in the browser, with one-click **backup/restore** (JSON export/import).
- **Dark/light theme**, keyboard navigation, and mobile layout.

## Usage

Open `index.html` in a modern browser (Chrome or Edge recommended for read-aloud and voice recording). Your progress is saved automatically in the browser; use **⤓ Backup** to export it and **⤒ Restore** to bring it back.

## Adding content

All lessons live as data in the `CourseData` object inside `index.html`. Adding a lesson is just appending an object to the relevant phase's `lessons` array — no code changes needed. Card types: `concept`, `vocab`, `example`, `mistake`, `dialogue`, `quiz`.

## Tech

Vanilla HTML/CSS/JavaScript in one self-contained file. Uses only browser-native APIs (Web Speech for read-aloud, MediaRecorder for recording, localStorage for progress). No dependencies, no network calls.
