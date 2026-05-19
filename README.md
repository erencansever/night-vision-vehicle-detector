# Night Vision Vehicle Detector

![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-4.x-green?logo=opencv&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow)
![Status](https://img.shields.io/badge/Status-Research-orange)
![Camera](https://img.shields.io/badge/Camera-Basler%20NIR-lightgrey)

> Real-time nighttime vehicle detection and tracking from a forward-facing NIR camera mounted on a moving vehicle.

---

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Dataset](#dataset)
- [Distance Estimation](#distance-estimation)
- [Installation](#installation)
- [Usage](#usage)
- [Tech Stack](#tech-stack)

---

## Overview

This project addresses a core challenge in autonomous driving and ADAS (Advanced Driver Assistance Systems): **detecting and tracking vehicles at night** without relying on visible-light cameras. Using a near-infrared (NIR) camera, the system identifies surrounding vehicles by their tail lights, tracks them across frames, and estimates their distance from the ego vehicle in real time.

**Key capabilities:**
- Robust detection under complete darkness using NIR imaging
- Multi-vehicle tracking with Kalman Filter (constant-velocity model)
- Distance estimation fusing Y-position and bounding box width
- Handles reflections, glare, and varying light modes (DRL, low beam, high beam, brake, hazard)

---

## How It Works

```
Raw NIR Video (2048×2048, 30fps)
    │
    ├─ Resize (×0.625 → 1280×1280)
    ├─ Grayscale Conversion
    ├─ CLAHE  ──────────────────── contrast enhancement (clip=2.0, tile=8×8)
    ├─ Gaussian Blur ────────────── denoising (ksize=3)
    ├─ Adaptive Threshold ───────── isolate bright spots (>235)
    ├─ Morphological Open + Close ─ noise removal & gap filling
    │
    ├─ Blob Extraction & Filtering
    │       area: 5–30000 px  |  aspect: 0.3–4.0  |  extent: 0.15–1.0
    │
    ├─ Light Pair Matching ─────── pair left/right tail lights into one vehicle box
    ├─ Reflection Filtering ────── remove wet-road mirror artifacts
    │
    └─ Kalman Filter Tracking
            state: [cx, cy, w, h, vx, vy]
            Hungarian-style greedy matching (threshold=50px)
            max_missing=5 frames before track deletion
```

---

## Dataset

Videos captured with a **Basler acA2040-90umNIR** industrial NIR camera (2048×2048, 30fps, exposure=50000µs).

| Subset | Conditions | Videos |
|--------|-----------|--------|
| **Distance** | Stationary target vehicles at 25m / 50m / 75m / 100m with 6 light modes | ~24 clips |
| **Exposure** | Ego-vehicle driving on 3 road types at night | ~15 clips |

**Road types:**

| Type | Description |
|------|-------------|
| `3lane_median` | 3-lane highway with median barrier |
| `2lane_median` | 2-lane highway with median barrier |
| `2lane_bidirectional` | 2-lane bidirectional road (no median) |

**Light modes:** `position` · `low beam` · `high beam` · `DRL` · `brake` · `hazard`

---

## Distance Estimation

Distance is estimated by fusing two signals:

| Signal | Weight | Logic |
|--------|--------|-------|
| Y-position (bottom of bbox) | 75% | Closer vehicles appear lower in the frame |
| Bounding box width | 25% | Closer vehicles appear wider |

**Output bins:**

| Distance | Color | Status |
|----------|-------|--------|
| < 25m | 🔴 Red | Danger |
| 25–50m | 🟠 Orange | Warning |
| 50–75m | 🟢 Green | Safe |
| 75–100m | 🟢 Green | Safe |
| > 100m | 🟢 Green | Safe |

---

## Installation

```bash
git clone https://github.com/erencansever/night-vision-vehicle-detector.git
cd night-vision-vehicle-detector
pip install -r requirements.txt
```

---

## Usage

1. Place raw video data under `night_vehicle_raw/` with the following structure:

```
night_vehicle_raw/
├── Mesafe - Exposure_50000_30fps/     # distance subset
│   ├── Arac1/
│   │   ├── 25m/
│   │   ├── 50m/
│   │   ├── 75m/
│   │   └── 100m/
│   └── Arac2/
└── Exposure_50000_30fps/              # exposure subset
    ├── Yol Tip1/
    ├── Yol Tip2/
    └── Yol Tip3/
```

2. Open `realtime_nighttime_vehicledetection_tracking.ipynb` and run all cells top to bottom.

3. The final cell launches the real-time detection window. Press **Q** or **ESC** to stop.

---

## Tech Stack

| Library | Purpose |
|---------|---------|
| `opencv-python` | Video I/O, image processing, Kalman Filter |
| `numpy` | Array operations |
| `pandas` | Metadata management (clip catalog) |
| `matplotlib` | Frame visualization in notebook |

---

## Future Work

- Replace heuristic blob pairing with a learned detector (e.g., YOLOv8 fine-tuned on NIR data)
- Add lane detection to constrain the search region
- Improve distance estimation with stereo calibration or depth cues
- Export detections to a structured log (JSON/CSV) for offline analysis
