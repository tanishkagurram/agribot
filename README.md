# AgriSense — Smart Farm Dashboard 🌾

A single-page, offline-capable dashboard for small and mid-size farms that
brings live sensor data, on-device disease detection, an AI farm assistant,
irrigation control, and weather/pest insights into one screen — in the
farmer's own language.

## What it is

`AgriSense` (internally labeled **SOTC** in the sidebar) is a self-contained
HTML app — no backend, no build step, no server-side code. Everything
(UI, sensor simulation, on-device machine learning, multi-language text)
lives inside a single `.html` file and runs entirely in the browser.

## Pages / features

The sidebar navigates between eight views (`showPage()` toggles `.page`
divs):

| Page | What it does |
|---|---|
| 📊 **Dashboard** | Two swipeable panels: (1) live temperature / soil moisture / rainfall readings with a Chart.js line chart and recent-activity log, (2) a pest & disease risk summary with active-threat cards, a treatment schedule, and a 7‑day risk-trend chart. |
| 🤖 **AI Insights** | Static pest/crop insight cards plus a free-text chat box (`askAI()`) that calls the **Mistral API** directly from the browser, using the current sensor readings and selected crop as context, and replying in the user's selected language. |
| 🔬 **Disease Scan** | Upload or drag-and-drop a crop photo; a **TensorFlow.js** model running entirely on-device classifies it (two selectable models: a general leaf-disease model and a pepper-ripeness model), shows per-class confidence bars, a description, remedies, and prevention tips. Includes scan history and an "Add to Alerts" action. |
| 🌱 **Crop Activity** | Log and track farming activities per crop. |
| 💧 **Irrigation** | Toggle irrigation zones on/off, see per-zone progress, and manage a daily/periodic watering schedule per field. |
| 📅 **History** | Session history of past readings/events. |
| 🔔 **Alerts** | A dismissible notification feed (pest risk, low moisture, fertilizer due, rainfall, sensor status) with a badge counter and "Clear All". |
| ⚙️ **Settings** | Notification toggles, SMS alert opt-in (stored client-side), sensor thresholds (min soil moisture, max temperature, heavy-rain trigger), and the Mistral API key field. |

Additional cross-cutting features:

- **ESP32 sensor connection** — a status bar on the Dashboard connects to a
  real ESP32 board over the **Web Serial API** (`navigator.serial`) and
  replaces the simulated readings with live temperature/soil/rainfall data
  parsed from the serial stream. Falls back to simulated data if
  unsupported or not connected.
- **Multi-language UI** — a full translation table (`T`) covers English,
  Hindi, Kannada, and Telugu; every label on the page is swapped via
  `data-key` attributes and `applyTranslations()`, and the AI assistant
  replies in whichever language is selected.
- **Health/status pill** and **live pulse indicators** reflect sensor and
  connection state at a glance.

## How the on-device AI works

- The two disease-detection models are **embedded directly in the HTML
  file as base64-encoded model data** (`MODEL_B64`), so the page needs no
  network access to run inference.
- On page load, the base64 data is decoded to a `Blob`, given an object
  URL, and loaded with `tflite.loadTFLiteModel()` (if the TFLite runtime is
  available) or `tf.loadGraphModel()` as a fallback.
- When a photo is scanned, it's decoded to a tensor with
  `tf.browser.fromPixels`, resized/normalized, and run through the loaded
  model; the resulting class probabilities are mapped to a small local
  knowledge base of descriptions, remedies, and prevention tips per label
  (e.g. Bacterial Spot, Powdery Mildew, Healthy, Pepper Green/Red/Yellow).
- This keeps the "Detect Disease" flow **fully offline** — the only feature
  needing network access is the AI chat (which calls the Mistral API) and,
  optionally, real ESP32 hardware over serial.

## Tech stack

- Plain **HTML/CSS/vanilla JavaScript** — no framework, no bundler.
- **[Chart.js](https://www.chartjs.org/)** (via CDN) for the sensor and
  pest-risk trend charts.
- **[TensorFlow.js](https://www.tensorflow.org/js)** (via CDN) for
  in-browser model loading and inference.
- **Mistral AI Chat Completions API** for the natural-language farm
  assistant (requires a user-supplied API key, stored in `localStorage`).
- **Web Serial API** for optional real ESP32 hardware integration.

## How to run it

No installation needed — it's a static file.

1. Open `finallllll__1_.html` directly in a modern desktop browser
   (Chrome or Edge recommended, since the Web Serial API and some
   TensorFlow.js features are best supported there).
2. Everything works immediately with simulated sensor data.
3. Optional:
   - Go to **Settings** and paste a Mistral API key to enable **AI
     Insights** chat.
   - Click **🔌 Connect ESP32** on the Dashboard (Chrome/Edge desktop only)
     to pair a real ESP32 board over USB serial instead of simulated
     readings.
   - Go to **Disease Scan** and upload a crop photo — this works fully
     offline once the page has loaded (models are embedded in the file).

> Because the CDN scripts (Chart.js, TensorFlow.js) are loaded from the
> internet, a first-time load needs connectivity; after that, the
> disease-scan flow itself needs no network access.

## Notes / limitations

- Sensor readings on the Dashboard are **simulated by default**; only
  connecting a real ESP32 over Web Serial replaces them with live data.
- The Mistral API key is stored in the browser's `localStorage` and is
  sent directly from the client to Mistral's API — there's no backend
  proxy, so the key is visible to anyone with access to that browser
  profile.
- The Web Serial API (used for ESP32 connection) is only supported in
  Chromium-based desktop browsers, not Safari, Firefox, or mobile browsers.
- Crop Activity, History, and some schedule/threshold controls are UI
  scaffolding without persistent backend storage — values reset on reload
  unless explicitly wired to `localStorage`.
- The pest/disease "risk" content on the Dashboard's second panel is
  illustrative sample data, distinct from the real on-device model results
  produced by the Disease Scan page.
