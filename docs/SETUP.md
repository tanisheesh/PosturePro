# Local Setup — PosturePro

> **Just want to try it?** Use the live demo at [tanisheesh.github.io/PosturePro](https://tanisheesh.github.io/PosturePro/) — no setup needed.
> This guide is for running PosturePro locally or self-hosting it.

---

## Prerequisites

- A modern web browser (Chrome or Edge recommended — both ship with full WebAssembly support for MediaPipe)
- A webcam
- Internet connection for the web app (MediaPipe WASM files are loaded from jsDelivr CDN)
- Python 3.8+ and a webcam for the desktop app

---

## Option A — Web App (browser)

The web app is a static site — no build step, no server required.

### 1. Clone the repository

```bash
git clone https://github.com/tanisheesh/PosturePro
cd PosturePro
```

### 2. Serve the `docs/` directory locally

The app must be served over HTTP (not opened as a `file://` URL) because `getUserMedia` and WASM require a secure context. Use any static server:

```bash
# Option 1: Python built-in server
cd docs
python -m http.server 8000

# Option 2: Node live-server
npm install -g live-server
live-server docs/
```

### 3. Open in browser

Navigate to `http://localhost:8000`, grant camera permission when prompted, and click **Start Camera**.

---

## Option B — Python Desktop App

### 1. Navigate to the app directory

```bash
cd PosturePro/python-app
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r ../requirements.txt
```

The `requirements.txt` at the repo root installs:

| Package | Purpose |
|---|---|
| `opencv-python-headless` | Webcam capture and frame annotation |
| `mediapipe` | Pose landmark detection |
| `numpy` | Numerical utilities |

### 4. Run the application

```bash
python posturepro-gui.py
```

An OpenCV window opens showing the live annotated webcam feed. Press `q` to quit.

> **Note:** The script defaults to webcam index `1` (`cv2.VideoCapture(1)`). If your webcam is at index `0`, edit line 32 of `posturepro-gui.py` and change `1` to `0`.

---

## Environment variables

PosturePro requires no API keys or environment variables. There is no `.env` file.

---

## Known local-only limitations

- **Webcam index (Python):** The desktop app hardcodes camera index `1`. If your system's primary webcam is at index `0`, edit the `VideoCapture` call before running.
- **`file://` protocol blocked:** Browsers block `getUserMedia` on `file://` URLs. Always serve the web app via a local HTTP server, not by double-clicking `index.html`.
- **CDN dependency (web):** MediaPipe WASM assets are fetched from jsDelivr at runtime. The web app will not load without an internet connection.
- **Lighting:** MediaPipe Pose detection accuracy degrades significantly in poor or uneven lighting. Use in a well-lit room with your upper body clearly visible.

---

## Deployment (GitHub Pages)

The `docs/` directory is already configured as a GitHub Pages source. To deploy your own fork:

1. Fork the repository on GitHub.
2. Go to **Settings → Pages → Source** and select the `docs/` folder on `main`.
3. Your instance will be live at `https://<your-username>.github.io/PosturePro/` within a minute.

No build step or CI pipeline is required.

---

## Author

**Tanish Poddar** — [tanisheesh.in](https://tanisheesh.in) · [LinkedIn](https://linkedin.com/in/tanisheesh) · [GitHub](https://github.com/tanisheesh)
