# Billiard & 8-Ball Collision Tracker/detection
> **An End-to-End Computer Vision & Deep Learning Automation Pipeline**

A technical computer vision pipeline engineered to automate game telemetric tracking, active shot player attribution, and collision event log generation from standard sports video feeds, YouTube streams, or real-time camera arrays. 

This repository serves as a portfolio technical submission showcasing modular system architecture, scale-agnostic tracking design, and state-machine logic. **Note: The core implementation scripts are kept private.**

---

## Architectural Blueprint & Pipeline Flow

The system processes video data frame-by-frame through a sequential, decoupled six-stage pipeline:

[ Ingestion & Codec Sync ] ──> [ Hough Line Feature Mapping ] ──> [ Temporal Turning Queue ]
│
[ Metrics Dashboard ]      <── [ Cooldown State Machine ]     <── [ YOLOv8 Object Tracking ]

### 1. Ingestion & Codec Normalization
To prevent frame drops, stuttering, or indexing offsets common when using raw stream captures or variable frame-rate containers, the pipeline initializes an isolated transformation pass via an absolute system process. This standardizes the incoming feed to fixed **H.264 video encoding** using a **`yuv420p` pixel matrix** while discarding erratic audio descriptors.

### 2. Line Feature Extraction & Player Attribution
Active shooting turn context is calculated geometrically via a **Hough Line Transform** applied across a localized Canny edge-mapped matrix. To filter out permanent pool table margins, pockets, and rails from the actual user's pool cue, structural filtering thresholds are asserted:
* **Length Filtering:** Threshold set at $\text{Length} > \text{Width} \times 0.18$.
* **Orientation Constraining:** Discards lines outside a standard horizontal trajectory ($\theta < 35^\circ$).
* **Spatial Tracking:** The midpoint coordinate ($X_{\text{mid}}$) of the confirmed cue lines determines the current shooting sphere:
  * **Player 1 Turn:** Mitigating centroid scales on the left hemisphere ($X < \frac{\text{Width}}{2}$).
  * **Player 2 Turn:** Mitigating centroid scales on the right hemisphere ($X > \frac{\text{Width}}{2}$).

### 3. Structural Temporal Smoothing
To resolve instantaneous tracking occlusions, fast vector changes, or edge detection drops during extreme cue travel velocities, the pipeline integrates a statistical windowing buffer (`CUE_DETECT_WIN = 30`). This maintains a rolling history queue and executes a majority vote rule to stabilize turn assignment transitions smoothly.

### 4. Bounding Box & Target Harvesting
Object extraction is performed using a high-frequency `yolov8s` model targeted strictly at object index class `32` (*sports ball*). The detector operates at a relaxed confidence limit (`CONF_THRESHOLD = 0.10`) and tight spatial intersection overlap controls (`IOU_THRESHOLD = 0.50`). This allows the system to lock onto objects seamlessly through heavy motion blurs, deep shadows near table pockets, or specular highlights from table lights.

---

## Mathematical Foundations & Collision State Logic

### Adaptive Scale-Agnostic Thresholding
Instead of checking proximity using a rigid, fixed pixel radius—which breaks under varying aspect ratios, different video resolutions, or camera zoom variations—the tracker utilizes a dynamic boundary proximity limit. 

For any given pair of detected ball centroids, the instantaneous Cartesian Euclidean distance is evaluated:

$$\text{Distance} = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

The contact event trigger point is dynamically adapted per object pair based on their visible box dimensions:

$$\text{Threshold} = \left( \frac{\text{Diameter}_1 + \text{Diameter}_2}{2} \right) \times \text{Scale Factor}$$

*Where $\text{Scale Factor}$ is strictly assigned a coefficient of $1.15$. A collision check returns valid if $\text{Distance} < \text{Threshold}$.*

### Temporal Cooldown State Machine
When two balls make physical contact, they inherently remain within close spatial proximity over multiple successive frames. Evaluating proximity alone would cause a single contact event to trigger dozens of duplicate hits. 

To ensure exact tracking metrics, the system implements a locked **Finite State Machine (FSM)**:
* Upon validating contact, a state lock (`COOLDOWN_FRAMES = 15`) is instantiated.
* Spatial evaluation loops are entirely bypassed during this window.
* Once the frame delta clears, the tracking loops are safely unlocked for subsequent shots.

---

## Pipeline Configuration Matrix

The pipeline's core tracking behavior is governed by the following hyperparameter profile:

| Parameter Name | Target Production Profile Value | Engineering Operational Intent |
| :--- | :--- | :--- |
| `MODEL_WEIGHTS` | `yolov8s.pt` | Minimizes processing latency while retaining dense object profile recall. |
| `CONF_THRESHOLD` | `0.10` | Eliminates object dropouts through high-glare or deep-shadow regions. |
| `IOU_THRESHOLD` | `0.50` | Preserves tracking indexing identity during dense ball clusters and impacts. |
| `COLLISION_SCALE` | `1.15` | Scalable geometric factor controlling physical object surface margins. |
| `COOLDOWN_FRAMES` | `15` | Lockout duration window preventing duplicate logging during rolling contact. |
| `CUE_DETECT_WIN` | `30` | Temporal frame queue length optimizing stable turn switching. |

---

## Analytics Dashboards & Output Architecture

The system compiles processed metadata into comprehensive downstream deliverables:
1. **Real-Time Heads-Up Display (HUD):** Generates an overlay stream showing active player status indicators, live score tallies, exact bounding boxes, and an interactive distance vector line connecting targets that shifts dynamically to green when a hit registers.
2. **Synchronized Match Summaries (`score_dashboard.png`):** An automated visualization module utilizing Matplotlib that plots:
   * A comparative categorical bar graph showing absolute final scores.
   * A progressive step-plot tracking cumulative scoring distributions against the exact timeline.
   * A structural relative pie chart displaying overall point distribution share.
3. **Structured Terminal Telemetry Logs:** Produces highly readable, clean event logs specifying frame sequence indices, timestamps down to the millisecond, relative object distance, and point assignment tags.

---

## Tech Stack

* **Core Framework:** Python 3.11.9
* **Deep Learning & Inference:** Ultralytics YOLOv8 (Small Architecture Core)
* **Computer Vision Processing:** OpenCV (Open Source Computer Vision Library)
* **Ingestion/Stitching Engine:** FFmpeg Core Subprocess Controls
* **Data Visualization:** Matplotlib, NumPy
* **Stream Handling:** yt-dlp Asset Streaming Ingestion Engine




## 📄 License
Copyright © 2026 by Muhammad Hamza Ishaq. All rights reserved. The core source code, pipeline implementation logic, and system architecture design are proprietary assets and may not be reproduced, distributed, or deployed without explicit authorization.
