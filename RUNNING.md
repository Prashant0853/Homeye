# Homeye — Run Guide

> Maintained by **Prashant Gupta**

This guide covers every way to run Homeye on your machine, from zero-install to full hardware setup.

---

## Prerequisites

| Tool | Required for | Install |
|---|---|---|
| Python 3.10+ | Simple UI server | [python.org](https://www.python.org/downloads/) |
| Node.js 18+ | Dashboard dev server / mobile app | [nodejs.org](https://nodejs.org/) |
| Rust 1.85+ | Build from source | [rustup.rs](https://rustup.rs/) |
| Docker | Easiest full-stack run | [docker.com](https://www.docker.com/get-started/) |

No hardware needed — everything works in **simulated mode** out of the box.

---

## Option 1 — Quickest (Python, no install needed)

Serves the main UI instantly using Python's built-in server.

```powershell
cd ui
python -m http.server 3000 --bind 127.0.0.1
```

Open in browser:

| Page | URL |
|---|---|
| Main dashboard | http://127.0.0.1:3000/index.html |
| Signal observatory | http://127.0.0.1:3000/observatory.html |
| Data visualization | http://127.0.0.1:3000/viz.html |

To stop: press `Ctrl + C` in the terminal.

---

## Option 2 — Dashboard Dev Server (Node.js / Vite)

The dashboard is a full Vite + TypeScript app.

```powershell
cd dashboard
npm install
npm run dev
```

Open: **http://localhost:5173**

Other dashboard commands:

```powershell
npm run build       # production build
npm run preview     # preview production build on port 4173
npm run test        # run unit tests
```

---

## Option 3 — Docker (Full stack, recommended)

Runs the complete Homeye system including the Rust sensing server and UI.

```powershell
docker pull ruvnet/wifi-densepose:latest
docker run -p 3000:3000 ruvnet/wifi-densepose:latest
```

Open: **http://localhost:3000**

### Run with simulated data explicitly

```powershell
docker run -e CSI_SOURCE=simulated -p 3000:3000 ruvnet/wifi-densepose:latest
```

### Run with real ESP32 hardware

```powershell
docker run -e CSI_SOURCE=esp32 -p 3000:3000 -p 5005:5005/udp ruvnet/wifi-densepose:latest
```

---

## Option 4 — Build from Source (Rust)

Best performance. Requires Rust installed from [rustup.rs](https://rustup.rs/).

```powershell
cd v2
cargo build -p wifi-densepose-sensing-server --no-default-features
.\target\debug\sensing-server --source simulated --http-port 3000 --ui-path ..\ui
```

Open: **http://127.0.0.1:3000/ui/index.html**

For a release (optimized) build:

```powershell
cargo build -p wifi-densepose-sensing-server --no-default-features --release
.\target\release\sensing-server --source simulated --http-port 3000 --ui-path ..\ui
```

---

## Option 5 — Mobile App (Expo / React Native)

The mobile app lives in `ui/mobile/`.

### Browser preview (fastest)

```powershell
cd ui/mobile
npm install
npx expo start --web
```

Open: **http://localhost:8081**

### Android (needs Android Studio or Expo Go app on phone)

```powershell
npx expo start --android
```

### iOS (macOS only, needs Xcode or Expo Go)

```powershell
npx expo start --ios
```

### Connect mobile app to the server

Once the app is open, go to **Settings** tab and set the server URL:

| Scenario | URL |
|---|---|
| Local machine | `http://127.0.0.1:3000` |
| Same WiFi network | `http://<your-pc-ip>:3000` |
| Remote server | `https://your-domain.com` |

If the server is unreachable the app shows a yellow **SIM** badge and uses simulated data automatically.

---

## Running in VS Code

Press `Ctrl+Shift+P` → **Tasks: Run Task** → choose one:

| Task | What it does |
|---|---|
| Serve UI (simulated) | Starts Python server on port 3000 |
| Dashboard dev server | Starts Vite on port 5173 |
| Install dashboard deps | Runs `npm install` in dashboard/ |

Tasks are defined in `.vscode/tasks.json`.

---

## Data Sources

| Mode | How to activate | What you get |
|---|---|---|
| **Simulated** | Default, no hardware | Synthetic CSI data, all UI features work |
| **Windows WiFi** | Automatic on Windows | RSSI-only — coarse presence/motion |
| **ESP32-S3** | Set `CSI_SOURCE=esp32` | Full CSI — vitals, presence, motion |
| **ESP32 Mesh** | Multiple boards | Multi-room sensing |

---

## What the UI Shows

| Screen | Description |
|---|---|
| **Live** | 3D skeleton of detected people |
| **Vitals** | Breathing (6–30 BPM) and heart rate (40–120 BPM) |
| **Zones** | Floor plan with room occupancy heatmap |
| **MAT** | Disaster triage — survivor count and alerts |
| **Observatory** | Raw CSI signal waveforms |
| **Settings** | Server URL, theme, RSSI-only toggle |

---

## Troubleshooting

**`ERR_ADDRESS_INVALID` in browser**
> Use `http://127.0.0.1:3000` instead of `http://localhost:3000`. On Windows, localhost can resolve to IPv6 which causes this error.

**Port 3000 already in use**
```powershell
# Find what's using port 3000
Get-NetTCPConnection -LocalPort 3000

# Kill Python servers
Stop-Process -Name python -Force
```

**Directory listing shows instead of UI**
> You ran the server from the wrong folder. Make sure you `cd ui` before running `python -m http.server 3000`.

**`npm install` fails**
> Make sure Node.js 18+ is installed: `node --version`

---

## Hardware Setup (Optional)

For real sensing you need at least one **ESP32-S3** board (~$9).

1. Follow `firmware/esp32-csi-node/README.md` to flash the firmware
2. Connect the board to your WiFi network
3. Start the server with `CSI_SOURCE=esp32`
4. The board streams CSI data to the server via UDP on port 5005

Supported boards:
- ESP32-S3 (recommended)
- ESP32-C6
- Intel 5300 NIC (Linux only)
- Atheros AR9580 NIC (Linux only)
