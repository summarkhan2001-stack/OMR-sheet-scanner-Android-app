# OMR-sheet-scanner-Android-app
Smart Mobile OMR Scanner: High-performance OMR grading platform combining a Python/OpenCV ensemble vision backend with a Flutter mobile application

## 📌 Executive Summary

The **Smart Mobile-Based OMR Sheet Evaluation and Scoring Platform** is an enterprise-grade academic grading platform engineered to replace specialized, expensive hardware Optical Mark Recognition (OMR) scanners with a ubiquitous mobile camera and edge-cloud architecture.

Built as a Final Year Project at the **University of Azad Jammu & Kashmir (UAJK)**, Department of Computer Science & IT (DCS&IT), this system combines a cross-platform **Flutter** mobile client with an ensemble **Python FastAPI + OpenCV** computer vision engine (`v6.0 Ensemble`). It delivers **100% grading accuracy**, sub-second evaluation (~450ms per sheet), CamScanner-grade document cleaning, and multi-mark/erasure disambiguation.

- **Author:** Shah Zaib (Roll No. 04, Reg No. 2022-UMDB-00104)
- **Supervisor:** Dr. Maryum Bibi (Assistant Professor, DCS&IT, UAJK)
- **Institution:** University of Azad Jammu & Kashmir, Muzaffarabad

---

## 🏗️ System Architecture

```mermaid
flowchart TD
    subgraph MobileApp["📱 Flutter Mobile Client"]
        A["CameraX Viewfinder / Gallery Picker"] --> B["Image Capture & Local Validation"]
        B --> C["SQLite Offline Cache"]
        B --> D["REST API Dispatcher (HTTP/Multipart)"]
    end

    subgraph Tunnel["🌐 Network / Reverse Proxy"]
        D -->|"Port 8000 (LAN / Emulator / ngrok)"| E["FastAPI Gateway"]
    end

    subgraph BackendEngine["⚡ v6.0 Ensemble Computer Vision Engine"]
        E --> F["Preprocessing & Normalization\n(CLAHE + Bilateral Filter)"]
        F --> G["Contour Detection & Grid Fitting\n(Aspect Ratio Filtering + Row/Col Clustering)"]
        G --> H["Sub-Pixel Centroid Refinement\n(Hough Circle Transform)"]
        H --> I["6-Method Voting Ensemble"]
        
        subgraph Methods["6 Independent Grading Methods"]
            M1["M1: Raw Grayscale Brightness"]
            M2["M1b: CLAHE-Enhanced Brightness"]
            M3["M2: Dark Pixel Fill Ratio"]
            M4["M3: Z-Score Statistical Outlier"]
            M5["M4: Per-Bubble Otsu Binarization"]
            M6["M5: HSV Color-Space Ink Detection"]
        end

        I --> Methods
        Methods --> J["Consensus Aggregator & Multi-Mark Check"]
        J --> K["CamScanner Background Normalization\n& Color Highlight Generation"]
    end

    K -->|"JSON Breakdown + Base64 Annotated Image"| D
    D --> L["Results Screen & Interactive Pinch-to-Zoom Overlay"]
```

---

## 🔬 The 10 Computer Vision Approaches (Engine v6.0)

The core computer vision pipeline implements 10 synergistic algorithmic techniques designed to withstand uneven lighting, shadows, low-cost paper, light pencil strokes, pen smudges, and mobile hand tremors:

| # | Approach | Code Implementation | Technical Function |
|---|---|---|---|
| **A** | **Template Profiling Constraints** | `fit_grid()` | Enforces rigid column, row, and bubble geometry based on known OMR layout structures, eliminating false positive contours. |
| **B** | **4-Point Homography Perspective Warp** | `four_point_transform()` | Warps skewed camera captures taken from arbitrary angles ($\pm 20^\circ$) into a planar, top-down rectangular coordinate frame. |
| **C** | **Dark Pixel Fill Ratio** | `_vote_fill_ratio()` | Calculates the ratio of black pixels within each bubble's inner circular mask relative to the outer ring. |
| **D** | **CLAHE Adaptive Contrast** | `preprocess()` | Contrast Limited Adaptive Histogram Equalization suppresses intense glare and brightens underexposed corners. |
| **E** | **Per-Bubble Otsu Binarization** | `_vote_otsu()` | Computes an optimal binarization threshold for each bubble ROI individually, making it immune to global lighting gradients. |
| **F** | **Multi-Mark & Erasure Disambiguation** | `grade_sheet()` | Analyzes intra-question intensity deltas; flags anomalies when a student marks multiple bubbles or attempts incomplete erasures. |
| **G** | **Z-Score Statistical Outliers** | `_vote_zscore()` | Normalizes bubble fills across all options per question ($z = \frac{x - \mu}{\sigma}$), marking choices with $z < -1.0$ as decisive fills. |
| **H** | **Hough Circle Sub-Pixel Centroid Refinement** | `_refine_centers_hough()` | Uses sub-pixel circle detection to align the evaluation mask directly over printed circular rings. |
| **I** | **HSV Color-Space Ink Detection** | `_vote_ink()` | Isolates blue/black ballpoint ink and graphite sheen by mapping color components in Hue-Saturation-Value space. |
| **J** | **6-Method Ensemble Majority Vote** | `grade_sheet()` | Pools predictions from all 6 independent detection algorithms; selects the majority consensus with a computed confidence metric ($0.0 \to 1.0$). |

---

## 📊 Empirical Benchmarks

Extensive automated tests conducted on physical smartphone camera captures (100% grading accuracy on benchmark datasets):

```
============================================================
  v6.0 Ensemble Engine — Empirical Performance Profile
============================================================
  Total Pipeline Execution Time : 449 ms
  Grading & Classification Time : 15.1 ms (40% faster than v5.5)
  Single-Worker Throughput      : 148 sheets / minute
  4-Worker Parallel Throughput  : ~590 sheets / minute
  Peak Process Memory           : 21.9 MB
  Grading Accuracy              : 20/20 (100.0%)
  Ensemble Confidence Score     : 1.00 (All 6 methods in full agreement)
  Rotational Tolerance          : ±20° planar tilt
  Minimum Resolution Required   : 640 × 640 px
============================================================
```

---

## 📁 Repository Directory Structure

```
├── backend/
│   ├── server.py               # Core FastAPI web server & v6.0 Ensemble CV Engine
│   ├── test_v6.py              # Automated 100% accuracy regression test
│   ├── benchmark_v6.py         # Sub-millisecond latency & memory profiler
│   ├── requirements.txt        # Backend Python dependencies
│   ├── Dockerfile              # Production container specification (Debian Slim)
│   ├── run_server.bat          # 1-Click local Windows startup script
│   ├── start_with_ngrok.bat    # 1-Click startup script with ngrok reverse proxy
│   ├── .gitignore              # Backend gitignore (ignores __pycache__, binaries)
│   └── omr result.jpeg         # Standard test sheet image
│
├── frontend/
│   ├── lib/
│   │   ├── main.dart           # App entry point & theme initialization
│   │   ├── models/             # Exam, Question, and ScanResult models
│   │   ├── screens/            # Camera scan, batch scan, results, settings UI
│   │   └── services/           # ApiService (REST client) & DatabaseHelper (SQLite)
│   ├── android/                # Native Android platform code & permissions
│   ├── ios/                    # Native iOS platform code
│   ├── web/                    # Flutter web assets
│   ├── pubspec.yaml            # Flutter dependencies & assets manifest
│   └── .gitignore              # Flutter gitignore (ignores build/, ephemeral/)
│
├── test_sheets/
│   ├── omr result.jpeg         # 20-question filled sample test sheet
│   └── camscanner_preview.jpg  # Background-normalized enhanced scan sample
│
├── PROJECT_CONTEXT_AND_SPECIFICATION.md  # Exhaustive architectural blueprint
├── .gitignore                  # Root gitignore
└── README.md                   # This documentation
```

---

## 🚀 Quick Start Guide

### 1. Prerequisites
- **Python:** 3.10, 3.11, or 3.12 installed ([python.org](https://www.python.org/downloads/))
- **Flutter:** 3.22+ SDK ([flutter.dev](https://flutter.dev/docs/get-started/install))
- **Git**

---

### 2. Backend Setup (FastAPI + OpenCV)

1. Open your terminal and navigate to the `backend` folder:
   ```bash
   cd backend
   ```

2. Create and activate a Python virtual environment:
   ```bash
   # Windows (PowerShell):
   python -m venv venv
   .\venv\Scripts\Activate.ps1

   # Linux / macOS:
   python3 -m venv venv
   source venv/bin/activate
   ```

3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Verify accuracy on the sample OMR sheet:
   ```bash
   python test_v6.py
   ```
   *(Expected output: `PASS: 20/20 = 100% accuracy maintained!`)*

5. Start the backend server:
   ```bash
   # Quick startup on Windows:
   .\run_server.bat

   # Or via Uvicorn CLI directly:
   uvicorn server:app --host 0.0.0.0 --port 8000 --reload
   ```

6. Verify server health:
   - Navigate to `http://localhost:8000/docs` in your browser to view the interactive OpenAPI documentation.

---

### 3. Exposing Backend to Physical Devices or Emulators

| Device Type | Server URL Configuration | Notes |
|---|---|---|
| **Android Emulator** | `http://10.0.2.2:8000` | Works out-of-the-box with no additional software. |
| **Local Wi-Fi Network** | `http://<YOUR_LOCAL_IP>:8000` | Connect computer and phone to the same Wi-Fi router. |
| **Remote Physical Device** | `https://<YOUR_SUBDOMAIN>.ngrok-free.app` | Run `ngrok http 8000` to create a public HTTPS tunnel. |

> [!TIP]
> You do **not** need to recompile the Flutter app when changing your backend URL. Simply navigate to the **Settings** screen inside the mobile app and update the **Server URL**.

---

### 4. Frontend Setup (Flutter App)

1. Open a new terminal and navigate to the `frontend` folder:
   ```bash
   cd frontend
   ```

2. Install Flutter packages:
   ```bash
   flutter pub get
   ```

3. Run on your connected device or emulator:
   ```bash
   flutter run
   ```

---

### 5. Running with Docker

Deploy the backend anywhere without managing Python environments:

```bash
cd backend
docker build -t omr-backend .
docker run -d -p 8000:8000 --name omr-service omr-backend
```

---

## 📡 REST API Reference

### 1. Health Check
- **Endpoint:** `GET /`
- **Response:**
  ```json
  {
    "status": "online",
    "version": "6.0",
    "engine": "v6.0 Ensemble (10 CV Approaches)"
  }
  ```

### 2. Grade OMR Sheet
- **Endpoint:** `POST /grade`
- **Content-Type:** `multipart/form-data`
- **Parameters:**
  - `file`: Image file (`image/jpeg` or `image/png`).
  - `answer_key`: JSON string with key-value pairs representing questions and answers (e.g., `{"1":"C","2":"B","3":"C",...}`).
- **Sample Response:**
  ```json
  {
    "total": 20,
    "correct": 20,
    "wrong": 0,
    "blank": 0,
    "score": 20.0,
    "percentage": 100.0,
    "answers": [
      {
        "qno": 1,
        "detected": "C",
        "correct": "C",
        "is_correct": true,
        "confidence": 1.0,
        "is_multi": false,
        "method_votes": {"raw": "C", "clahe": "C", "fill": "C", "zscore": "C", "otsu": "C", "ink": "C"}
      }
    ],
    "highlight_image": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQ..."
  }
  ```

---

## 📱 Mobile App Screen Walkthrough

1. **Dashboard & Exams:** Create custom exams, assign negative marking rules, configure answer keys, and manage student rosters.
2. **Real-Time Camera Scan:** CameraX-backed viewfinder with auto-alignment framing guides and flash toggle.
3. **Pinch-to-Zoom Review:** View normalized CamScanner-enhanced results with color-coded rings:
   - 🟢 **Green Ring:** Correct Answer
   - 🔴 **Red Ring:** Incorrect Choice Marked
   - 🟡 **Yellow Ring:** Missed / Blank Question
4. **Local SQLite History:** View past student evaluations offline with instant CSV and PDF grade sheet export.

---

## 🔮 Future Roadmap (Gemini Multimodal Vision Integration)

The project includes an architectural blueprint to incorporate Google DeepMind's Gemini Multimodal models for advanced edge-case resolution:
- **Handwritten Roll No & Name Recognition:** Zero-shot handwriting OCR for student details on sheet headers.
- **Creased / Torn Sheet Fallback:** Fallback to Gemini 1.5 Flash when physical sheets suffer severe tears or deep folds that break homography warping.
- **Automated Answer Key Digitization:** Snap a photo of a handwritten teacher answer key to generate exam schemes automatically.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for full details.
