# Real-Time Multi-Person Pose Tracking and Activity Monitoring System

A real-time pose tracking and suspicious activity monitoring system built with Python, Flask, MediaPipe, and OpenCV. The system detects and tracks up to four people simultaneously, renders each as an animated skeletal/robot overlay, and flags suspicious behavior patterns through a live analytics dashboard.

## Features

- Real-time multi-person pose detection using MediaPipe PoseLandmarker (up to 4 people), with automatic fallback to single-person tracking if the multi-person model is unavailable
- Stable per-person identity tracking across frames using centroid matching with hysteresis, so each detected person keeps a consistent color and slot even through brief occlusion
- Animated skeletal overlay rendering (robot-style head, torso, spine, limbs, and joints) with per-person color palettes
- Suspicious activity detection, including:
  - Standing still for an extended period
  - Prolonged direct camera gaze
  - Face hidden or not visible
  - Frequent head movement / looking around
  - Crouching or bending down
  - Sudden disappearance from frame
  - Repeated entry and exit from a defined region of interest (ROI)
- Configurable ROI (region of interest) zone with adjustable boundaries via API
- Live web dashboard with:
  - Real-time video feed with overlay rendering
  - FPS, confidence, motion, and person-count charts
  - Session analysis view with historical charts and a risk-level breakdown
  - Suspicious activity event log with severity indicators
  - Exportable session snapshots (PNG)
- REST API for stats, camera testing, ROI configuration, and session reset

## Tech Stack

- **Backend:** Python, Flask, Flask-CORS
- **Computer Vision:** OpenCV, MediaPipe (PoseLandmarker / Pose Solutions API)
- **Frontend:** HTML, CSS, JavaScript
- **Charting:** Chart.js
- **Session Export:** html2canvas

## Requirements

- Python 3.10+
- A connected webcam

Install dependencies:

```bash
pip install flask flask-cors opencv-python mediapipe numpy
```

The `pose_landmarker_full.task` model file is downloaded automatically on first run if not already present in the project directory.

## Running the Project

```bash
python server.py
```

The server starts on:

```
http://localhost:5001
```

Open this URL in a browser to access the live dashboard.

## API Endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/` | GET | Serves the dashboard UI |
| `/video_feed` | GET | Live MJPEG video stream with pose overlay |
| `/stats` | GET | Current session statistics (FPS, confidence, motion, alerts, history) |
| `/set_roi` | GET | Update ROI boundaries via `left` and `right` query parameters (0-100) |
| `/test` | GET | Camera availability and multi-person tracking status |
| `/reset` | GET | Resets session state, tracker slots, and history |

## Configuration

Key parameters can be adjusted at the top of `server.py`:

| Parameter | Description | Default |
|---|---|---|
| `MAX_PERSONS` | Maximum number of people tracked simultaneously | 4 |
| `STANDING_STILL_SECONDS` | Time threshold to flag standing still | 5.0 |
| `LOOKING_CAMERA_SECONDS` | Time threshold to flag direct camera gaze | 5.0 |
| `FACE_HIDDEN_SECONDS` | Time threshold to flag hidden face | 2.0 |
| `EVENT_COOLDOWN_SECONDS` | Minimum gap between repeated alerts of the same type | 6.0 |
| `roi_x1`, `roi_x2`, `roi_y1`, `roi_y2` | Normalized ROI boundaries | 0.35, 0.65, 0.1, 0.9 |

## Notes

- If no camera is detected, the backend still starts and reports camera status through the `/test` endpoint.
- If the multi-person PoseLandmarker model fails to load, the system automatically falls back to MediaPipe's single-person Pose solution.
