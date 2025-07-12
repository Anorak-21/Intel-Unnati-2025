
# Intel Unnati 2025 – DL Streamer Pipeline (Problem Statement 3)

This repository contains all source code, scripts, and documentation developed as part of the Intel Unnati Industrial Training 2025 (Problem Statement 3):  
**"Create a pipeline (detect, decode, and classification) using DL Streamer, and define system scalability on Intel hardware."**

---

## 🚀 Project Summary

We implemented and benchmarked Intel DL Streamer pipelines using two OpenVINO models:
- `person-vehicle-bike-detection-2004`
- `face-detection-0204`

The evaluation focused on:
- Running pipelines on **Intel CPU only** (no GPU used)
- Testing with **1, 2, and 4 input streams**
- Comparing **GUI vs Headless (non-GUI)** execution
- Measuring **Total Average FPS** across configurations

---

## ⚙️ Test Setup

- **Platform**: Docker-based Intel DL Streamer on **Ubuntu 22.04.5 LTS**
- **Hardware**: Intel CPU (GPU unavailable)
- **Containerization**: Docker was used for image setup
- **GUI Rendering**: Enabled using **XLaunch**
- **Models Tested**:
  - `person-vehicle-bike-detection-2004`
  - `face-detection-0204`
- **Stream Counts**: 1, 2, and 4 input streams via GStreamer compositor
- **Modes Evaluated**: GUI Enabled, GUI Disabled
- **Primary Metric**: Total Average FPS

### 📥 Input Source

We used a **sample video file** for testing. For real-time testing, we integrated `yt-dlp` to stream YouTube videos directly into the pipeline. Scripts are included in the repo.

> **Note:** GPU testing was not performed due to lack of lab access during vacations. Cloud alternatives like Azure were explored but had poor performance for this use case.

---

## 📊 Performance Results

| Model                            | GUI Mode | Streams | Avg FPS |
|----------------------------------|----------|---------|---------|
| person-vehicle-bike-detection-2004 | On       | 1       | 22      |
|                                   | On       | 2       | 17      |
|                                   | On       | 4       | 39      |
|                                   | Off      | 1       | 80      |
|                                   | Off      | 2       | 68      |
|                                   | Off      | 4       | 63      |
| face-detection-0204              | On       | 1       | 20      |
|                                   | On       | 2       | 11      |
|                                   | On       | 4       | 30      |
|                                   | Off      | 1       | 47      |
|                                   | Off      | 2       | 44      |
|                                   | Off      | 4       | 42      |

---

## 📺 GUI Output

Screenshots for 4-stream GUI compositor output for both models are included in the report PDF.

---

## 🧪 Code Cheat Sheet

### 🖥️ GUI Setup (Only if running GUI):
```bash
export DISPLAY=//<your_ip>:0
```

### 🐳 Run Docker:
```bash
docker run -it --rm   -v /mnt/c/Users/Asus/intel/models:/home/dlstreamer/models   --env MODELS_PATH=/home/dlstreamer/models   --env DISPLAY=$DISPLAY   intel/dlstreamer:latest
```

### 📦 Model Setup
```bash
# Model 1
export DETECTION_MODEL=/home/dlstreamer/models/intel/person-vehicle-bike-detection-2004/FP16/person-vehicle-bike-detection-2004.xml
export DETECTION_MODEL_PROC=/opt/intel/dlstreamer/samples/gstreamer/model_proc/intel/person-vehicle-bike-detection-2004.json

# Model 2
export DETECTION_MODEL=/home/dlstreamer/models/intel/face-detection-0204/FP16/face-detection-0204.xml
export DETECTION_MODEL_PROC=/opt/intel/dlstreamer/samples/gstreamer/model_proc/intel/face-detection-0204.json
```

### 🎞️ Input Videos
```bash
export VIDEO_1=/home/dlstreamer/models/test.mp4
export VIDEO_2=/home/dlstreamer/models/test.mp4
export VIDEO_3=/home/dlstreamer/models/test.mp4
export VIDEO_4=/home/dlstreamer/models/test.mp4
```

### ▶️ GUI Mode Pipelines

#### 1 Stream (GUI)
```bash
gst-launch-1.0 filesrc location=$VIDEO_1 ! decodebin ! videoconvert ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvawatermark ! gvafpscounter ! videoconvert ! autovideosink
```

#### 2 Streams (GUI)
```bash
gst-launch-1.0 compositor name=comp sink_0::xpos=0 sink_1::xpos=640 ! video/x-raw,width=1280,height=360 ! videoconvert ! autovideosink sync=false filesrc location=$VIDEO_1 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvawatermark ! gvafpscounter ! videoconvert ! comp.sink_0 filesrc location=$VIDEO_2 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvawatermark ! gvafpscounter ! videoconvert ! comp.sink_1
```

#### 4 Streams (GUI)
```bash
gst-launch-1.0 compositor name=comp sink_0::xpos=0 sink_1::xpos=640 sink_2::xpos=0 sink_3::xpos=640 ! video/x-raw,width=1280,height=720 ! videoconvert ! autovideosink sync=false filesrc location=$VIDEO_1 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvawatermark ! gvafpscounter ! videoconvert ! comp.sink_0 filesrc location=$VIDEO_2 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvawatermark ! gvafpscounter ! videoconvert ! comp.sink_1 filesrc location=$VIDEO_3 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvawatermark ! gvafpscounter ! videoconvert ! comp.sink_2 filesrc location=$VIDEO_4 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvawatermark ! gvafpscounter ! videoconvert ! comp.sink_3
```

### 🔇 Headless Mode Pipelines

#### 1 Stream (Headless)
```bash
gst-launch-1.0 filesrc location=$VIDEO_1 ! decodebin ! videoconvert ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvafpscounter ! fakesink sync=false
```

#### 4 Streams (Headless)
```bash
gst-launch-1.0 filesrc location=$VIDEO_1 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvawatermark ! gvafpscounter ! fakesink sync=false filesrc location=$VIDEO_2 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvafpscounter ! fakesink sync=false filesrc location=$VIDEO_3 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvafpscounter ! fakesink sync=false filesrc location=$VIDEO_4 ! decodebin ! videoconvert ! videoscale ! video/x-raw,width=640,height=360 ! gvadetect model=$DETECTION_MODEL model-proc=$DETECTION_MODEL_PROC device=CPU threshold=0.2 ! gvafpscounter ! fakesink sync=false
```

---

## 📘 Authors

- Diya Nair – diya.mitmpl2022@learner.manipal.edu  
- Harshavardhan Reddy – vannela.mitmpl2022@learner.manipal.edu  
- Piyush Rangdal – piyush1.mitmpl2022@learner.manipal.edu

---
