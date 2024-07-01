# PosturePro — Architecture

<!--
Companion to PRD.md.
PRD says WHAT the system does. This says HOW.
Audience: an engineer who needs to understand the system well
enough to build it, debug it, or extend it.
-->

---

## 1. Stack

| Layer | Tech |
|---|---|
| Web frontend | Vanilla JS (ES6 classes) · HTML5 Canvas · CSS3 |
| Pose estimation (web) | MediaPipe Pose 0.5 (WASM, model complexity 1) |
| Camera utilities (web) | MediaPipe Camera Utils 0.3 |
| Desktop app | Python 3.8+ · OpenCV · MediaPipe · NumPy |
| Hosting | GitHub Pages (static, served from `docs/`) |

No backend, no database, no authentication, no build step.

---

## 2. Components

```
PosturePro/
  docs/               Web application (GitHub Pages)
    index.html        UI shell — video element, canvas overlay, stat cards
    script.js         PostureDetector class — camera, pose, drawing, alerts
    style.css         Dark-themed layout and responsive grid
    favicon.png       App icon
  python-app/
    posturepro-gui.py Desktop app — OpenCV capture, MediaPipe pose, CV overlay
  requirements.txt    Python dependencies (opencv-python-headless, mediapipe, numpy)
```

### Web App (`docs/`)

A single-page HTML5 application with no build toolchain. MediaPipe Pose and Camera Utils are loaded from jsDelivr CDN. All pose inference runs in the browser via WebAssembly — no video frames ever leave the device. The `PostureDetector` class manages the webcam stream lifecycle, sends each frame to the MediaPipe `Pose` instance via a `Camera` callback, receives landmark results, computes angles, draws the skeleton overlay on a `<canvas>`, updates the stat cards, and triggers the warning banner when bad posture exceeds 180 seconds.

### Python Desktop App (`python-app/posturepro-gui.py`)

Captures from a local webcam (`cv2.VideoCapture(1)`), converts each BGR frame to RGB, and passes it to `mp.solutions.pose.Pose`. Extracts four landmarks — left shoulder, right shoulder, left ear, left hip — computes shoulder alignment distance, neck inclination, and torso inclination using the same geometric formulas as the web app, then annotates the frame with OpenCV drawing primitives (circles, lines, text). Also writes the annotated stream to `output.mp4`. Raises a console warning after 180 seconds of bad posture.

---

## 3. Data Flow

```
[User opens browser / runs Python script]
    |
    v
[Webcam stream acquired]  <-- getUserMedia (web) / cv2.VideoCapture (Python)
    |
    v
[Frame sent to MediaPipe Pose model each tick]
    |
    v
[Pose landmarks returned (33 keypoints, normalised 0-1)]
    |
    +-- Extract: LEFT_SHOULDER (11), RIGHT_SHOULDER (12),
    |            LEFT_EAR (7), LEFT_HIP (23)
    |
    v
[Compute metrics]
    |  shoulderOffset  = Euclidean distance(leftShoulder, rightShoulder)
    |  neckInclination = arccos formula(leftShoulder, leftEar)
    |  torsoInclination= arccos formula(leftHip, leftShoulder)
    |
    v
[Classify posture]
    |  GOOD if neckInclination < 40° AND torsoInclination < 10°
    |  BAD  otherwise
    |
    v
[Update UI / CV overlay]
    |  - Draw landmark dots and connecting lines (green=good, red=bad)
    |  - Update stat cards: neck°, torso°, shoulderOffset, timer
    |
    v
[Accumulate bad-posture time]
    |  badFrames × deltaTime > 180s → fire warning banner / console alert
```

1. User grants camera permission and clicks "Start Camera".
2. A `Camera` loop (web) or `cap.read()` loop (Python) feeds frames to MediaPipe at up to 30 fps.
3. MediaPipe returns normalised 2D landmark coordinates for up to 33 body points.
4. Four landmarks are extracted and converted to pixel coordinates.
5. `findDistance()` computes shoulder alignment; `findAngle()` computes neck and torso angles.
6. Posture is classified good (neck < 40°, torso < 10°) or bad.
7. A running frame counter, multiplied by frame delta time, tracks continuous bad-posture duration.
8. After 180 seconds of bad posture, a warning is displayed.

---

## 4. Angle Calculation

Both the web and Python apps use the same formula:

```
theta = arccos( (y2 - y1) * (−y1)
                ─────────────────────────────── )
                √((x2−x1)² + (y2−y1)²) × y1

degree = (180 / π) × theta
```

The formula measures the angle of a body segment relative to the vertical image axis. The inputs are pixel coordinates (not normalised), so `y1` is the vertical pixel of the reference point. Both apps call this once for the neck (shoulder → ear) and once for the torso (hip → shoulder).

---

## 5. Security

- **No API keys:** The application needs no secrets — MediaPipe is loaded from CDN; no keys are ever used or stored.
- **Camera access:** The web app uses the browser's `getUserMedia` API, which requires explicit per-site permission. No frames are transmitted over the network.
- **Local processing only:** All inference runs client-side (WASM in the browser, native Python locally). No biometric or video data leaves the device.
- **No user accounts:** There is nothing to authenticate, no session tokens, no cookies.

---

## 6. Error Handling & Reliability

| Failure | Behaviour |
|---|---|
| No pose landmarks detected | Web: `drawResults` returns early, canvas cleared. Python: "No pose detected" text rendered on frame, loop continues. |
| Camera access denied | Web: `getUserMedia` rejects → `alert()` shown to user. |
| Missing landmark coordinates | Python: wrapped in `try/except`; exception printed, frame skipped. |
| MediaPipe CDN unavailable | Web: libraries fail to load → `window.Pose` is undefined → `alert()` shown on page load. |
| Camera index wrong (Python) | `cap.isOpened()` check fires → prints error and exits cleanly. |

---

## 7. Deployment

1. The `docs/` directory is the GitHub Pages source — no build step; files are served as-is.
2. MediaPipe Pose WASM files are fetched from `cdn.jsdelivr.net` at runtime; the `locateFile` callback directs MediaPipe to the correct CDN path.
3. The Python app is run locally; `requirements.txt` at the repo root pins all dependencies.
4. No CI/CD pipeline — changes pushed to `main` are reflected on GitHub Pages within seconds.

---

## 8. Explicit Scope Cuts

- **Backend / server** — All processing is client-side; no server is needed for the current feature set.
- **User accounts / posture history** — No persistence layer; session state resets on page reload.
- **OS-level notifications** — Browser `alert()`/banner only; system notifications deferred to v2.
- **Mobile / portrait orientation** — Layout is responsive but camera orientation for portrait pose detection is not handled.
- **Audio alerts** — Visual and console warnings only; audio cues deferred to v2.
- **Per-user calibration** — Fixed thresholds (40°/10°) used for everyone; personalised baselines deferred to v2.
