# Engineering Decisions — PosturePro

<!--
This is not user documentation. This is for technical interviewers
and senior engineers who want to understand WHY the system is built
the way it is. Every entry answers a question an interviewer might ask.
-->

---

## Decision 1 — MediaPipe over a custom-trained pose model

**Context:** The core feature is landmark detection — finding where shoulders, ears, and hips are in a webcam frame. Options were: train a custom model on a labelled dataset, use OpenPose (open-source but C++ and heavy), or use Google's MediaPipe Pose (pre-trained, cross-platform, browser-ready).

**Decision:** MediaPipe Pose at model complexity 1.

**Reason:** MediaPipe ships as a WASM module that runs directly in the browser with no GPU server, and as a native Python package. It provides 33 pre-labelled body landmarks with sub-50ms inference on commodity hardware. Training a custom model from scratch would require weeks of data collection and labelling for accuracy that MediaPipe already provides out of the box.

**Tradeoff:** The web app depends on jsDelivr CDN to serve the WASM assets. If the CDN is unavailable, the app cannot start. Bundling the WASM locally (v2) would remove that dependency at the cost of a larger repo.

---

## Decision 2 — Rule-based angle thresholds over ML posture classification

**Context:** Once landmarks are detected, posture must be classified as good or bad. One approach is to train a classifier on labelled good/bad frames. Another is to define geometric thresholds based on anatomy.

**Decision:** Deterministic angle thresholds — neck < 40° and torso < 10° = good posture.

**Reason:** Posture quality at a desk is fundamentally a geometric property: if your neck is inclined more than 40° forward, it is poor regardless of who you are. The rule-based approach is fully explainable (every verdict traces to a raw angle value), requires no training data, and is trivially auditable. An ML classifier would be a black box that adds complexity without improving correctness for this well-defined domain.

**Tradeoff:** Fixed thresholds do not account for individual anatomy. A tall person may have a naturally different neutral posture angle than a short person. Per-user calibration (deferred to v2) would address this.

---

## Decision 3 — Two delivery modes (browser app + Python desktop app)

**Context:** The target user is any desk worker with a laptop. Some want zero friction (no install); others need offline use or more direct webcam control.

**Decision:** Ship both a static web app (GitHub Pages) and a Python script.

**Reason:** The web app removes all installation friction — paste the URL, grant camera permission, and monitoring starts. The Python app uses OpenCV for direct webcam capture, which avoids browser overhead and works offline. Both apps implement identical posture logic (same angle formula, same thresholds) so they are maintainable together. The cost of maintaining two is low given the shared algorithm.

**Tradeoff:** Any logic change (e.g. updating angle thresholds) must be applied in both `script.js` and `posturepro-gui.py`. At current scale this is trivial; at larger scale it would warrant a shared specification or a single codebase compiled to both targets.

---

## Decision 4 — GitHub Pages for hosting (no backend)

**Context:** The web app needs to be publicly accessible. Options: a server (Node/Python), a serverless function platform (Vercel, Netlify), or a static host.

**Decision:** GitHub Pages, serving the `docs/` directory as a static site.

**Reason:** The app has no server-side logic — all inference runs in the browser. GitHub Pages is free, always on, and deploys automatically on push. There is no backend to maintain, no cold starts, and no monthly cost. For a portfolio project that must reliably run on demand for recruiters, uptime matters more than flexibility.

**Tradeoff:** If a backend feature is added in v2 (e.g. storing posture history via an API), GitHub Pages alone cannot serve it. At that point, the frontend would move to Vercel or Netlify with a separate API service.

---

## What I'd do differently in v2

- **Bundle MediaPipe WASM locally** — CDN dependence is a single point of failure for the web app. Shipping the WASM assets in the repo removes it, at the cost of a slightly larger download.
- **Auto-detect webcam index in the Python app** — Hardcoding `cv2.VideoCapture(1)` causes a confusing silent failure on machines where the webcam is at index 0. A short loop trying indices 0–2 would be far more robust.
- **Unify the two implementations into a single source of truth** — The angle formula and posture thresholds are duplicated in `script.js` and `posturepro-gui.py`. A shared JSON config file or a Python → JS compilation step would prevent drift.

---

## Explicit non-decisions (deferred to v2)

| Feature | Why deferred |
|---|---|
| Posture history / analytics | Requires a persistence layer (localStorage or backend DB); adds complexity without being needed for the core real-time feedback use case. |
| OS / audio notifications | System notification APIs vary by browser and OS; adds platform-specific complexity. The on-screen banner is sufficient for v1. |
| Per-user calibration | A calibration flow adds UX steps before the user sees value. Fixed thresholds work for the vast majority of seated adults at a desk. |
| Mobile / portrait support | Portrait-orientation body tracking requires different landmark extraction and layout; separate work stream for a separate use case. |
| User accounts | No feature in v1 requires identity; adding auth without a purpose adds friction and attack surface. |
