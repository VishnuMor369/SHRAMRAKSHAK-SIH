# 🏗️ SHRAMRAKSHAK System Architecture

## 1. System Overview

**SHRAMRAKSHAK** is an enterprise-grade AI safety intelligence platform engineered to detect and prevent Serious Injury & Fatality (**SIF**) precursor events in high-hazard industrial environments (oil & gas, heavy manufacturing, construction, petrochemical plants).

Unlike conventional HSE dashboards that record lagging injury statistics, SHRAMRAKSHAK transforms historical incident narratives, real-time edge CCTV streams, and permit-to-work systems into an **autonomous 7-stage closed-loop safety ecosystem**.

```
┌────────────────────────────────────────────────────────────────────────┐
│                      SHRAMRAKSHAK CLOSED-LOOP ENGINE                   │
└────────────────────────────────────────────────────────────────────────┘
  Stage 1: Multimodal Ingestion (Historical OSHA / Plant Incident Reports)
     │
     ▼
  Stage 2: Contextual NLP & Deterministic SIF Classification
     │
     ▼
  Stage 3: Dense Semantic Vector Indexing (E5-small-v2 + FAISS IndexFlatIP)
     │
     ▼
  Stage 4: Recurrence Reasoning & HSE Human Expert Validation
     │
     ▼
  Stage 5: Future-Work Safety Precondition Assurance
     │
     ▼
  Stage 6: Real-Time Edge CCTV Observation & Action State Machine
     │
     ▼
  Stage 7: Closed-Loop Human Resolution & CCTV Re-Breach Auditing
```

---

## 2. Canonical Domain Model

The entire platform operates on a single canonical schema (`backend/models_canonical.py`) ensuring zero data drift between database storage, NLP reasoning, vector indexing, computer vision alerts, and the user interface.

### Key Domain Entities
- **`SafetyEvent`**: 24+ structured fields capturing:
  - `event_id`: Unique identifier (e.g., `EVT-HUMAN-2026-0042`, `EVT-CCTV-0012`)
  - `activity`: Standardized activity taxonomy (e.g., `Mechanical Crane Hoisting`)
  - `energy`: Energy hazard classification (e.g., `Gravitational / Suspended Load`)
  - `exposure`: Physical human exposure description
  - `barrier`: List of engineered, physical, or administrative barriers (`EXCLUSION_ZONE`, `FALL_ARREST`, `LOTO_LOCK`)
  - `barrier_state`: Physical barrier condition (`EFFECTIVE_VERIFIED`, `COMPROMISED`, `FAILED`, `BYPASSED`)
  - `assertion_status`: Linguistic polarity (`AFFIRMED`, `NEGATED`, `HYPOTHETICAL`, `POST_EVENT`, `UNCERTAIN`)
  - `temporal_status`: Timing status (`PRE_INCIDENT`, `MID_INCIDENT`, `POST_INCIDENT`)
  - `sif_status`: Authoritative potential assessment (`SIF_POTENTIAL`, `NO_SIF_POTENTIAL_IDENTIFIED`, `REVIEW_REQUIRED`)
  - `lsr`: Associated IOGP Life-Saving Rules (`SAFE_MECHANICAL_LIFTING`, `BYPASSING_SAFETY_CONTROLS`, etc.)
  - `evidence_spans`: Character-exact evidence substrings grounding each classification
- **`EvidenceSpan`**: `(start_char, end_char, text, confidence)` guaranteeing provenance against raw input strings.
- **`SafetyPattern`**: Recurrence relationship capturing systemic hazard recurrence across locations or contractors. Initialized strictly with `ReviewStatus.CANDIDATE`.
- **`WorkPrecondition`**: Mandatory barrier preconditions automatically derived from HSE-validated recurring patterns and mapped against planned high-hazard permits.

---

## 3. Natural Language Processing & SIF Pathway Engine

The NLP subsystem (`backend/nlp_engine/`) uses a dual-layer architecture combining transformer semantics with deterministic physical reasoning.

```
Raw Report Narrative
       │
       ▼
 [NLP Preprocessor] ── Character offset preservation, tokenization, clause splitting
       │
       ▼
 [Assertion & Negation Detector] ── Detects medical/historical negations & hypothetical cases
       │
       ▼
 [Central Safety Ontology] ── Validates physical activities, energies, barriers, and states
       │
       ▼
 [Deterministic SIF Pathway Engine] ── Physical equation: High Energy + Exposure + Failed Barrier
       │
       ▼
 [IOGP Life-Saving Rules Mapper] ── Multi-label LSR mapping with evidence grounding
```

### Deterministic SIF Formula
SIF potential is evaluated using physics-based criteria rather than black-box probabilities:
$$\text{SIF Potential} = (\text{High-Energy Hazard}) \land (\text{Affirmed Exposure}) \land (\text{Failed / Bypassed Barrier}) \land (\text{Credible Consequence})$$

When evidence is missing or ambiguous, the engine safely abstains to `REVIEW_REQUIRED` to prevent false reassurances.

---

## 4. Semantic Safety Memory & Vector Architecture

The Safety Memory Engine (`backend/nlp_engine/semantic_memory.py`) provides semantic search and recurrence discovery over historical and newly generated safety events.

- **Embedding Model**: `intfloat/e5-small-v2` transformer model (384 dimensions).
- **Prefix Architecture**: Strict adherence to E5 conventions (`passage: ` for corpus indexing, `query: ` for retrieval).
- **Vector Index**: `faiss.IndexFlatIP(384)` with normalized inner-product cosine similarity.
- **Bijective SQLite Persistence**: Dual-write pattern guaranteeing:
  $$\text{SQLite Events} \equiv \text{SQLite Embeddings} \equiv \text{FAISS Vectors} \equiv \text{Mapping IDs}$$
- **Two-Stage Recurrence Reasoning**:
  - *Stage 1*: FAISS vector retrieval identifies top-$k$ semantic candidates.
  - *Stage 2*: Multi-attribute structured validation verifies compatibility across activities, energy classes, and barrier states to distinguish between `DUPLICATE`, `INDEPENDENT_RECURRENCE`, and `UNRELATED`.

---

## 5. Computer Vision & Edge Detection Pipeline

The edge vision pipeline (`backend/detection.py`) runs optimized ONNX models with real-time RTSP, webcam, and synthetic stream feeds:

- **Model Hierarchy**:
  1. `ppe_safety.onnx`: 13-class custom model detecting Hardhat, NO-Hardhat, Safety Vest, NO-Safety Vest, Gloves, NO-Gloves, Goggles, Fall-Detected.
  2. `yolov8n.onnx`: Real-time COCO person & vehicle bounding box localization.
  3. `fire_detection.onnx`: High-confidence thermal/flame detection (Class 1 Fire; Class 0 Smoke discarded).
- **Floor Polygon Ground Projection**:
  - Evaluates whether a worker is inside a hazardous exclusion zone using bottom-center and ankle keypoint projection (`cv2.pointPolygonTest`).
  - Perspective-aware height filtering rejects false positives caused by background equipment or aerial perspectives.
- **Debounce & Temporal Smoothing**:
  - Violations require 5 consecutive frames before firing an alert.
  - Exit debouncing requires 5 clear frames before closing active tracking.

---

## 6. Action State Machine & Verification Loop

Alerts generated from edge computer vision or manual reports enter a strict transactional state machine:

```
    ┌────────┐
    │  OPEN  │ ── Supervisor notified (20-second SLA)
    └────┬───┘
         │ Supervisor responds
         ▼
 ┌───────────────┐
 │  RESPONDING   │ ── Field action taken (e.g., barricade tape installed, worker redirected)
 └───────┬───────┘
         │ Resolution note & supervisor ID submitted
         ▼
 ┌───────────────────────────┐
 │   AWAITING_VERIFICATION   │ ── 60-second observation window
 └───────┬───────────────────┘
         │
         ├── CCTV detects re-violation within window ──► [RE-BREACH] ──► Status returns to OPEN
         │
         └── CCTV confirms zone clear for full duration ──► [RESOLVED] ──► Tamper-proof SQLite audit record
```

This closed-loop guarantee prevents "pencil-whipping" (supervisors marking tasks resolved without real physical compliance on site).
