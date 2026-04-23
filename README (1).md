# 🎯 Smart Queue Monitor

A real-time queue monitoring system built with Python, OpenCV, YOLOv8, and Deep SORT. It detects people from a live camera feed, tracks them across defined zones, measures individual wait times, and displays a live dashboard — all without any manual input.

---

## 📸 Demo

> Camera feed with bounding boxes, zone overlays, and live stats panel.

```
Queue Zone [Blue]   →   Service Zone [Cyan]
Person enters queue → wait timer starts → moves to service → avg time updated
```

---

## ✨ Features

- 🔍 **Real-time person detection** using YOLOv8n (GPU/CPU support)
- 🧠 **Multi-object tracking** with Deep SORT (persistent IDs across frames)
- 🗺️ **Dual-zone detection** — Queue ROI and Service ROI, adjustable on the fly
- ⏱️ **Per-person wait time tracking** with live display on each bounding box
- 📊 **Live dashboard** showing queue size, service count, congestion status, and average service time
- 📈 **Exponential Moving Average** for adaptive service time estimation
- 🖥️ **Keyboard-controlled ROI adjustment** — no code changes needed
- 💾 **Optional CSV logging** of queue data over time
- 🎯 **Bounding box smoothing** to reduce jitter across frames

---

## 🛠️ Tech Stack

| Component | Library / Tool |
|-----------|---------------|
| Object Detection | [YOLOv8](https://github.com/ultralytics/ultralytics) (`yolov8n.pt`) |
| Multi-Object Tracking | [Deep SORT Realtime](https://github.com/levan92/deep_sort_realtime) |
| Computer Vision | [OpenCV](https://opencv.org/) |
| Deep Learning Backend | [PyTorch](https://pytorch.org/) |
| Language | Python 3.8+ |

---

## 📁 Project Structure

```
smart-queue-monitor/
│
├── main.py            # Main entry point with CSV logging support
├── queue_monitor.py   # Core monitor class with per-person wait tracking
├── testing.py         # Version with bounding box smoothing & HH:MM:SS display
├── WW.py              # Stable tracking version (n_init=3, EMA service time)
├── TP_1.py            # Prototype with ETA estimation per person
├── TBP_1.py           # (Reserved / in development)
├── TP_2.py            # (Reserved / in development)
│
├── yolov8n.pt         # YOLOv8 nano model weights (auto-downloaded)
├── queue_log.csv      # Generated at runtime (if CSV logging is enabled)
└── README.md
```

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/smart-queue-monitor.git
cd smart-queue-monitor
```

### 2. Install Dependencies

```bash
pip install ultralytics deep-sort-realtime opencv-python torch torchvision
```

> For GPU support, install the appropriate CUDA-enabled version of PyTorch from [pytorch.org](https://pytorch.org/get-started/locally/).

### 3. Run the Monitor

```bash
python main.py
```

The YOLOv8n model weights (`yolov8n.pt`) will be downloaded automatically on first run.

---

## ⌨️ Keyboard Controls

### Queue ROI (Blue Box)

| Key | Action |
|-----|--------|
| `A` / `D` | Move left / right |
| `W` / `S` | Move up / down |
| `Z` / `X` | Shrink / expand width |
| `C` / `V` | Shrink / expand height |

### Service ROI (Cyan Box)

| Key | Action |
|-----|--------|
| `Arrow Keys` or `I/J/K/L` | Move in any direction |
| `I` / `K` | Expand / shrink height |
| `J` / `L` | Shrink / expand width |

### General

| Key | Action |
|-----|--------|
| `ESC` | Exit and print final ROI coordinates |

---

## ⚙️ Configuration

All settings are in the `Config` class inside each file:

```python
class Config:
    source = 0              # Camera index (0 = default webcam, 1 = USB cam)
    model_path = "yolov8n.pt"
    use_gpu = True
    yolo_conf = 0.45        # Detection confidence threshold
    min_person_height = 80  # Min bounding box height in pixels
    frame_skip = 2          # Process every Nth frame (higher = faster)
    queue_roi = [50, 200, 400, 500]     # [x1, y1, x2, y2]
    service_roi = [420, 200, 650, 450]  # [x1, y1, x2, y2]
    deep_sort_max_age = 40  # Frames to keep a lost track alive
    alpha = 0.3             # EMA smoothing factor for service time
    log_csv = True          # Enable/disable CSV logging (main.py only)
```

---

## 📊 How It Works

```
Camera Frame
    │
    ▼
YOLOv8 Detection (persons only, conf > 0.45, height > 80px)
    │
    ▼
Deep SORT Tracking (assign persistent IDs)
    │
    ▼
Zone Classification (Queue ROI / Service ROI / None)
    │
    ├─── Queue ROI  → Record entry time, start wait timer
    ├─── Service ROI → Calculate wait time, update EMA average
    └─── No Zone    → Track only, no timing
    │
    ▼
Compute: queue count, service count, congestion level, estimated wait
    │
    ▼
Render dashboard + bounding boxes on frame
    │
    ▼
Display → repeat
```

---

## 📋 Dashboard Info Panel

The on-screen panel shows:

- **Queue** — number of people currently in the queue zone
- **Wait** — estimated wait time based on queue length and average service time
- **Status** — `LOW` (≤4), `MEDIUM` (5–8), `HIGH` (>8)
- **In Service** — number of people currently in the service zone
- **Served** — total people served since the session started
- **Avg Time** — rolling average service duration (EMA)

---

## 🔧 Troubleshooting

**Camera not opening:**
The system tries `cv2.CAP_MSMF` first, then falls back to the default backend. If your camera still doesn't open, try changing `source` in `Config` to `1` or `2`.

**Slow performance:**
- Set `frame_skip = 3` or higher
- Make sure `use_gpu = True` and CUDA is available
- Use a lower resolution by adjusting `imgsz` in the `model()` call

**Tracking IDs keep changing:**
Increase `deep_sort_max_age` (e.g., to `60`) and set `n_init=3` when creating the `DeepSort` tracker (already done in `WW.py` and `testing.py`).

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).

---

## 🙌 Acknowledgements

- [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics)
- [Deep SORT Realtime](https://github.com/levan92/deep_sort_realtime)
- [OpenCV](https://opencv.org/)
