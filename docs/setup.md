# 🚀 Installation & Setup Guide

This guide details the exact steps required to install dependencies, initialize storage, run the backend and frontend servers, and execute the automated test suites for **SHRAMRAKSHAK**.

---

## 1. Prerequisites

Before setting up the project, ensure your workstation meets the following requirements:

- **Operating System**: Windows 10/11, Ubuntu 20.04+, or macOS (tested on Windows 11)
- **Python**: Version 3.10, 3.11, or 3.12 (Python 3.14 compatible)
- **Node.js**: Version 18.x or 20.x LTS
- **Package Managers**: `pip` (Python) and `npm` (Node.js)
- **Video Input**: Built-in webcam, USB camera, or RTSP stream (synthetic camera fallback included)

---

## 2. Environment Variables

> **Note**: SHRAMRAKSHAK does not require external cloud API keys, paid token services, or cloud database credentials. All inference (ONNX Runtime) and vector search (FAISS E5) run locally and deterministically on CPU.
> 
> Neither `.env` nor `.env.example` files are required.

---

## 3. Backend Setup

### Step 3.1: Install Python Dependencies

From the project root directory, install all required dependencies:

```powershell
pip install -r requirements.txt
```

Key libraries installed:
- `fastapi` & `uvicorn[standard]`: High-performance asynchronous API framework
- `onnxruntime`: Digital-signature compliant inference engine for YOLOv8 & PPE models
- `opencv-python`: Real-time frame processing and computer vision utilities
- `faiss-cpu`: Dense vector similarity search index
- `torch` & `transformers`: Local E5-small-v2 sentence embedding pipeline
- `pandas` & `numpy`: Industrial dataset ingestion and vector math
- `pydantic`: Robust schema validation
- `pyyaml`: Safety ontology configuration parser
- `reportlab` & `pypdf`: Audit report PDF generation

---

## 4. Frontend Setup

### Step 4.1: Install Node Dependencies

Navigate to the `frontend/` directory and install the packages:

```powershell
cd frontend
npm install
cd ..
```

Key packages installed:
- `react` (v18.3) & `react-dom`: Component interface framework
- `lucide-react`: Industrial and HSE iconography
- `canvas-confetti`: UI celebration animations for resolved compliance goals
- `vite`: Fast ESM build tool and development server

---

## 5. Running the Application

### Option A: Standard Dual-Terminal Execution

#### Terminal 1 — Backend:
```powershell
cd backend
python -m uvicorn main:app --host 0.0.0.0 --port 8000
```
- **Backend API**: `http://localhost:8000`
- **Interactive Swagger Docs**: `http://localhost:8000/docs`

#### Terminal 2 — Frontend:
```powershell
cd frontend
npm run dev
```
- **HSE Executive Dashboard**: `http://localhost:5173`
- **Mobile Field Supervisor View**: `http://localhost:5173/supervisor`

---

## 6. Verification & Automated Testing

### 6.1 Run Master 15-Step Closed-Loop Acceptance Suite
Verifies the complete closed-loop lifecycle (ingestion ➔ SIF analysis ➔ recurrence ➔ HSE review ➔ precondition ➔ future work check ➔ CCTV violation ➔ SLA resolution ➔ re-breach detection):

```powershell
python tests/test_master_closed_loop_acceptance.py
```

### 6.2 Check Memory Integrity
Verifies bijective alignment across SQLite events, embeddings, FAISS vectors, and index mapping:

```powershell
python tools/check_memory_integrity.py
```

### 6.3 Run End-to-End System Tests
```powershell
python tests/test_true_closed_loop_e2e.py
```
