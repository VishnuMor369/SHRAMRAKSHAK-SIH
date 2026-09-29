# 📡 API Reference Documentation

The SHRAMRAKSHAK backend exposes a unified REST and WebSocket API built with FastAPI. Interactive OpenAPI/Swagger documentation is available at `http://localhost:8000/docs` when the backend is running.

---

## 1. System & Health Endpoints

### `GET /api/status`
Returns the operational health, active camera state, model status, and high-level incident counters.

- **Response `200 OK`**:
```json
{
  "status": "OPERATIONAL",
  "active_alerts_count": 2,
  "resolved_alerts_count": 14,
  "model_loaded": true,
  "model_name": "ppe_safety.onnx (ONNX Runtime)",
  "source_mode": "SYNTHETIC",
  "events_total": 334,
  "patterns_total": 12,
  "active_preconditions_total": 8
}
```

### `GET /api/qr`
Generates a QR code image / data URI allowing mobile supervisors to pair their phones with the field interface.
- **Parameters**: `url` (optional string): Custom supervisor URL.
- **Response `200 OK`**:
```json
{
  "qr_base64": "data:image/png;base64,...",
  "supervisor_url": "http://192.168.1.15:5173/supervisor"
}
```

---

## 2. CCTV & Edge Vision Endpoints

### `GET /video_feed`
Streams live video frames (MJPEG multipart stream) with real-time bounding box annotations, hazard overlays, and zone polygons.

### `GET /api/cv/debug`
Returns real-time inference latency, FPS metrics, current frame count, and model diagnostics.

### `POST /api/cctv/settings`
Updates detection sensitivity, confidence thresholds, and active camera source.
- **Request Body**:
```json
{
  "source": "webcam",
  "ppe_confidence": 0.25,
  "zone_debounce_frames": 5
}
```

---

## 3. Alerts & Action State Machine

### `GET /api/alerts`
Lists all active and historical alerts, sorted by priority score and severity.

### `GET /api/alerts/{alert_id}`
Returns granular telemetry for a specific alert, including bounding boxes, violation categories, evidence crop URLs, and status history.

### `POST /api/alerts/{alert_id}/respond`
Marks an alert as `RESPONDING` when a field supervisor begins addressing the hazard.
- **Request Body**:
```json
{
  "supervisor_id": "SUP-104",
  "supervisor_name": "R. Sharma"
}
```

### `POST /api/alerts/{alert_id}/resolve`
Submits a corrective action and transitions the alert into the `AWAITING_VERIFICATION` window.
- **Request Body**:
```json
{
  "supervisor_id": "SUP-104",
  "supervisor_name": "R. Sharma",
  "resolution_notes": "Exclusion zone cleared and perimeter tape reinforced."
}
```

---

## 4. Safety Memory & Vector Retrieval

### `POST /api/memory/search`
Queries the FAISS vector index using E5-small-v2 embeddings to retrieve semantically related past incidents.
- **Request Body**:
```json
{
  "query": "Worker entered perimeter while suspended load was lifted",
  "top_k": 5
}
```
- **Response `200 OK`**:
```json
{
  "candidates": [
    {
      "event_id": "EVT-HUMAN-2026-0014",
      "similarity_score": 0.892,
      "activity": "Mechanical Lifting Operations",
      "barrier": ["EXCLUSION_ZONE"],
      "barrier_state": ["BYPASSED"],
      "sif_status": "SIF_POTENTIAL",
      "narrative": "Rigging crew entered crane hazard perimeter during pipe hoist."
    }
  ]
}
```

### `POST /api/memory/reconcile`
Executes an atomic check and rebuild of the FAISS index and SQLite vector tables to guarantee zero orphan mappings or missing embeddings.

---

## 5. HSE Pattern Governance & Preconditions

### `GET /api/patterns`
Retrieves all candidate and HSE-validated recurring hazard patterns.

### `POST /api/patterns/{pattern_id}/review`
Records an official HSE human governance decision (`HSE_VALIDATED` or `REJECTED`) with an immutable audit log.
- **Request Body**:
```json
{
  "decision": "HSE_VALIDATED",
  "reviewer_role": "HSE_PLANT_MANAGER",
  "notes": "Validated recurring breach during heavy hoisting operations."
}
```

### `POST /api/preconditions/check`
Checks an upcoming work package / permit against active validated preconditions.
- **Request Body**:
```json
{
  "activity": "Mechanical Crane Hoisting",
  "location": "Compressor Sector 4",
  "evidence_presented": ["PHYSICAL_PERIMETER_DEMARCATION"]
}
```
- **Response `200 OK`**:
```json
{
  "status": "REVIEW_REQUIRED",
  "matching_preconditions": [
    {
      "precondition_id": "PREC-ade9ab4b",
      "title": "Mandatory EXCLUSION_ZONE for Mechanical Lifting",
      "required_evidence": ["PHYSICAL_PERIMETER_DEMARCATION", "AUTHORIZED_ENTRANTS_PASSPORT", "OBSERVABLE_CCTV_CLEAR_ZONE"],
      "missing_evidence": ["AUTHORIZED_ENTRANTS_PASSPORT", "OBSERVABLE_CCTV_CLEAR_ZONE"]
    }
  ]
}
```

---

## 6. Dataset Pipeline & Ingestion

### `GET /api/dataset/sample`
Preloads the benchmark historical industrial dataset for zero-configuration live demonstrations.

### `POST /api/dataset/import-execute`
Executes streaming batch ingestion with real-time SSE progress updates across Parsing, Assertion Tagging, LSR Classification, SIF Potential Scoring, and Vector Indexing.

### `GET /api/dataset/analysis-runs`
Lists all executed dataset analysis runs and returns download links for executive PDF reports.

---

## 7. Real-Time WebSocket Feed

### `WS /ws`
Bi-directional real-time communication channel:
- Pushes live camera detection bounding boxes (JSON payload)
- Pushes alert status changes and escalation countdown events
- Receives supervisor acknowledgement messages
