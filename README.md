<p align="center">
  <img src="docs/favicon.png" width="64" height="64" alt="PosturePro logo">
</p>

<h1 align="center">PosturePro</h1>

<p align="center">
  <strong>Real-time posture detection via computer vision — no backend, no login, just open and sit straight.</strong>
</p>

<p align="center">
  <a href="https://tanisheesh.github.io/PosturePro/">
    <img src="https://img.shields.io/badge/live_demo-3b82f6-3b82f6?style=flat-square" alt="Live Demo">
  </a>
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=opencv&logoColor=white" alt="OpenCV">
  <img src="https://img.shields.io/badge/MediaPipe-0097A7?style=flat-square" alt="MediaPipe">
  <img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black" alt="JavaScript">
  <img src="https://img.shields.io/badge/GitHub_Pages-222222?style=flat-square&logo=github&logoColor=white" alt="GitHub Pages">
  <img src="https://img.shields.io/badge/license-GPL--3.0-3b82f6?style=flat-square" alt="License">
</p>

---

## What is PosturePro?

PosturePro uses Google's MediaPipe Pose model to track key body landmarks — shoulders, ear, and hip — then computes neck and torso inclination angles in real time. If either angle exceeds the good-posture threshold for more than three minutes, an on-screen warning fires. The tool ships as both a browser app (zero installation, hosted on GitHub Pages) and a Python desktop app for offline use — covering both quick demos and long work sessions with no server costs or sign-up friction.

> **Live demo →** [tanisheesh.github.io/PosturePro](https://tanisheesh.github.io/PosturePro/)

---

## What you get

- **Real-time landmark tracking** — MediaPipe Pose detects shoulders, ear, and hip at up to 30 fps directly in the browser or via webcam feed in Python.
- **Angle-based posture scoring** — Neck inclination and torso inclination are computed geometrically; good posture requires neck < 40° and torso < 10°.
- **Shoulder alignment check** — Euclidean distance between shoulders flags whether the user is facing the camera squarely.
- **3-minute bad-posture warning** — A persistent counter tracks consecutive bad-posture frames and fires a dismissible alert after 180 seconds.

---

## Stack

| Layer | Tech |
|---|---|
| Web frontend | Vanilla JS · HTML5 · CSS3 |
| Pose estimation | MediaPipe Pose 0.5 (model complexity 1) |
| Desktop app | Python 3.8+ · OpenCV · MediaPipe · NumPy |
| Hosting | GitHub Pages (static, from `docs/`) |

---

## Engineering Decisions

**Why MediaPipe over a custom-trained model?**
MediaPipe Pose is a production-grade, Google-maintained model that runs entirely in the browser via WASM — no GPU server needed. Training a custom model would add weeks of work with no accuracy benefit for this well-defined landmark detection task.

**Why rule-based thresholds over ML classification?**
Posture quality (good/bad) is a deterministic geometric property: if the neck angle is under 40° and the torso angle is under 10°, posture is good. A rules-based approach is fully explainable, requires no training data, and is trivially auditable — every verdict traces back to a raw angle value.

**Why two delivery modes (web + Python)?**
The web app removes all installation friction and works across platforms for demos and casual use. The Python app targets users who need offline access or lower-latency processing through direct webcam capture with OpenCV — no browser overhead.

**What would you do differently in v2?**
Add OS-level desktop notifications so warnings reach users who minimise the window, and store per-session posture history locally (localStorage or SQLite) so users can track trends over time.

---

## Docs

| Document | Description |
|---|---|
| [PRD](docs/PRD.md) | Product requirements — goals, user stories, non-goals |
| [Architecture](docs/ARCHITECTURE.md) | System design, data flow, component breakdown |
| [Decisions](docs/DECISIONS.md) | Every major technical decision and why |
| [Setup](docs/SETUP.md) | Local dev setup and deployment |

---

## Author

**Tanish Poddar** — [tanisheesh.in](https://tanisheesh.in) · [LinkedIn](https://linkedin.com/in/tanisheesh) · [GitHub](https://github.com/tanisheesh)
