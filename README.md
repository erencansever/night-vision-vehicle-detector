# Night Vision Vehicle Detector

Real-time nighttime vehicle detection and tracking system using a forward-facing camera mounted on a moving vehicle. Detects surrounding vehicles and obstacles from NIR (near-infrared) video footage captured at night.

## Overview

This system processes raw NIR video recordings and performs:

- **Preprocessing** — grayscale conversion, CLAHE contrast enhancement, Gaussian denoising
- **Detection** — adaptive thresholding + blob analysis to identify vehicle tail lights as paired light blobs
- **Tracking** — Kalman Filter based multi-object tracker with constant-velocity motion model
- **Distance Estimation** — fuses Y-position and bounding box width to estimate vehicle distance (bins: `<25m`, `25-50m`, `50-75m`, `75-100m`, `>100m`)

## Dataset

Videos were captured with a **Basler acA2040-90umNIR** camera (2048×2048, 30fps) under two conditions:

| Subset | Description |
|--------|-------------|
| `distance` | Fixed vehicle targets at 25m / 50m / 75m / 100m with different light modes (position, low beam, high beam, DRL, brake, hazard) |
| `exposure` | Ego-vehicle driving on three road types (3-lane highway, 2-lane highway, 2-lane bidirectional) |

## Pipeline

```
Raw NIR Video
    └─ Resize (×0.625 → 1280×1280)
    └─ Grayscale
    └─ CLAHE (clip_limit=2.0, tile=8×8)
    └─ Gaussian Blur (ksize=3)
    └─ Adaptive Threshold (>235)
    └─ Morphological Open + Close
    └─ Blob Extraction & Filtering
    └─ Light Pair Matching
    └─ Reflection Filtering
    └─ Kalman Filter Tracking
    └─ Distance Classification
```

## Requirements

```
opencv-python
numpy
pandas
matplotlib
```

Install with:

```bash
pip install -r requirements.txt
```

## Usage

1. Place raw video data under `night_vehicle_raw/` following the expected folder structure:
   ```
   night_vehicle_raw/
   ├── Mesafe - Exposure_50000_30fps/   # distance subset
   │   ├── Arac1/
   │   │   ├── 25m/
   │   │   ├── 50m/
   │   │   └── ...
   │   └── Arac2/
   └── Exposure_50000_30fps/            # exposure subset
       ├── Yol Tip1/
       ├── Yol Tip2/
       └── Yol Tip3/
   ```

2. Open and run `realtime_nighttime_vehicledetection_tracking.ipynb` top to bottom.

3. The final cell launches the real-time visualization window. Press **Q** or **ESC** to stop.

## Distance Color Coding

| Color | Distance | Status |
|-------|----------|--------|
| Red | < 25m | Danger |
| Orange | 25–50m | Warning |
| Green | > 50m | Safe |

## Tech Stack

- Python 3.x
- OpenCV
- NumPy / Pandas
- Matplotlib
- Basler Pylon (camera acquisition)
