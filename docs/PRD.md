# PosturePro — Product Requirements Document

**Status:** Final
**Owner:** Tanish Poddar
**One-liner:** Real-time posture detection via computer vision — no backend, no login, just open and sit straight.

---

## 1. Problem

People who sit at desks for long stretches routinely develop forward head posture and hunched backs without realising it — there is no passive feedback while they work. Existing solutions either require wearable hardware or expensive software subscriptions. A lightweight, zero-friction tool that uses the webcam already built into every laptop and runs entirely in the browser can close that gap with no installation, no account, and no ongoing cost.

---

## 2. Goals (v1 / MVP)

1. Detect posture in real time using only the device's built-in webcam — no additional hardware.
2. Compute neck inclination and torso inclination angles geometrically from MediaPipe Pose landmarks.
3. Classify each frame as good (neck < 40°, torso < 10°) or bad and display the verdict instantly on-screen.
4. Track continuous bad-posture duration and fire a visible warning after 180 seconds.
5. Ship a browser-based version with zero installation that works on any modern laptop.
6. Ship a Python desktop version for users who need offline use or direct webcam control.
7. Deployed with a working live demo URL (GitHub Pages).

---

## 3. Non-Goals (explicit scope cuts)

- **User accounts / sign-up** — The tool is anonymous; no identity management is needed for v1.
- **Posture history / analytics** — Session data is not persisted; the page resets on reload. Deferred to v2 when there is a clear user demand for trends.
- **OS-level or audio notifications** — Alerts are visual only (on-screen banner). System notifications and sound cues deferred until browser notification APIs are evaluated for cross-platform support.
- **Mobile support** — Portrait-orientation body tracking requires different landmark extraction; deferred to v2.
- **Per-user calibration** — Fixed angle thresholds apply universally. Personalised baselines require a calibration flow that adds friction for v1.
- **Cloud processing or video upload** — All inference runs on-device; no video ever leaves the browser.

---

## 4. Users

**Primary:** Students, developers, and desk workers who sit for long periods and want passive real-time feedback on their posture without installing software or creating an account.

**Secondary:** Recruiters and engineers reviewing this as a portfolio project — the live demo must work reliably on first visit with a single button click.

---

## 5. User Stories

1. *As a desk worker,* I want to open a URL and start posture monitoring in under 10 seconds so that I can get feedback without any setup.
2. *As a user,* I want to see my neck angle, torso angle, and shoulder alignment update live so that I understand exactly what the system is measuring.
3. *As a user,* I want a clear colour-coded indicator (green/red) on the video overlay so that I can glance at my screen and know my posture status immediately.
4. *As a user,* I want a prominent warning banner after I have had bad posture for 3 minutes so that I am prompted to correct my position before pain sets in.
5. *As a developer,* I want a Python desktop app I can run offline so that I can use the tool without a browser or internet connection.
6. *As a recruiter,* I want the live demo to detect my posture within 2 seconds of granting camera access so that the project's functionality is immediately obvious.

---

## 6. Functional Requirements

### 6.1 Camera Access

- The web app must request camera access via `getUserMedia` with `{ width: 640, height: 480, facingMode: 'user' }`.
- If camera access is denied, a clear error message must be displayed.
- The user must be able to stop the camera feed with a single button click.

### 6.2 Pose Detection

- The system must use MediaPipe Pose (model complexity 1) to detect body landmarks on each frame.
- The system must extract four specific landmarks: left shoulder, right shoulder, left ear, and left hip.
- If no pose is detected in a frame, the system must display a "no pose" indicator rather than crashing.

### 6.3 Posture Analysis

- The system must compute shoulder offset (Euclidean distance between left and right shoulder landmarks).
- The system must compute neck inclination (angle between left shoulder → left ear vectors relative to vertical).
- The system must compute torso inclination (angle between left hip → left shoulder vectors relative to vertical).
- Posture must be classified as **good** if neck inclination < 40° AND torso inclination < 10°, otherwise **bad**.

### 6.4 Feedback & Alerts

- The skeleton overlay (landmark dots and connecting lines) must be drawn on a `<canvas>` overlay in green for good posture and red for bad posture.
- Live metric values (neck°, torso°, shoulder offset, posture timer) must be displayed in a stats panel below the video.
- The posture timer must show how long the current good or bad streak has lasted.
- A warning banner must appear when the user has been in bad posture for more than 180 consecutive seconds.

### 6.5 Python Desktop App

- The Python app must capture from a webcam using OpenCV and process frames with the MediaPipe Python SDK.
- The app must display the annotated video feed in a live OpenCV window.
- The app must write the annotated session to `output.mp4` for review.
- The app must print a console warning after 180 seconds of bad posture.

---

## 7. Non-Functional Requirements

- **Latency:** Pose inference must run at a framerate sufficient for smooth visual feedback (target ≥ 15 fps on a mid-range laptop).
- **Privacy:** No video frames or biometric data may leave the device — all processing is local (WASM in browser, native in Python).
- **Cost:** Zero server costs. All inference runs on the client; hosting is GitHub Pages (free).
- **Compatibility:** The web app must work in Chrome and Edge (desktop); both ship MediaPipe-compatible WASM runtimes.
- **No dependencies on sign-up or API keys:** The app must function with no credentials of any kind.

---

## 8. Success Metrics

| Metric | Target |
|---|---|
| Time from page load to first posture reading | < 10 seconds on a typical laptop |
| Warning fires correctly | After exactly 180 seconds of continuous bad posture |
| Live demo reliability | Works on first visit without refresh in Chrome |
| Python app launch-to-detection | < 5 seconds from running the script |

---

## 9. Risks & Open Questions

- **MediaPipe CDN availability** — The web app depends on jsDelivr to serve MediaPipe WASM files. If the CDN is unavailable, the app cannot load. Mitigated by jsDelivr's high uptime SLA; a v2 fix would bundle the WASM locally.
- **Camera index (Python)** — The desktop app hardcodes `cv2.VideoCapture(1)`, which fails if the webcam is at index 0. Mitigated by a clear error message; v2 should auto-detect or prompt.
- **Lighting conditions** — MediaPipe Pose detection degrades significantly in poor lighting. Documented in setup notes; no algorithmic mitigation planned for v1.
- **Angle formula edge cases** — The `arccos` formula produces `NaN` if the landmark coordinates produce a value outside `[-1, 1]`; wrapped in try/except (Python) and guarded by early return (JS).

---

## 10. v2 Candidates

- **Posture history chart** — Session-level posture timeline stored in `localStorage`, visualised as a sparkline so users can track improvement over time.
- **OS / audio notifications** — System-level desktop notifications and an optional sound alert so warnings reach users who minimise the browser tab.
- **Per-user calibration** — A 10-second guided calibration flow that captures the user's natural upright posture as the baseline, replacing fixed 40°/10° thresholds.
- **Mobile support** — Portrait-mode landmark extraction and a responsive layout designed for phone cameras.
- **Bundled WASM** — Ship MediaPipe assets with the repo so the app works without a CDN dependency.
