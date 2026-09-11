# VIGIL — Technical Requirements Document (TRD)

> Product: VIGIL — Intelligent UAV Engine Health & Prognostics  
> SIH Problem Statement: SIH26054  
> Organization: DRDO  
> Theme: Robotics & Drones  
> Version: 1.0  
> Status: Technical Source of Truth  
> Related Document: `PRD.md`

---

## 1. Purpose

This document defines the technical architecture, technology stack, interfaces, data contracts, software modules, communication paths, engineering constraints, testing strategy, and deployment requirements for VIGIL.

The PRD defines **what VIGIL must accomplish**.  
This TRD defines **how the system is technically organized to accomplish it**.

The implementation must preserve the product principles and safety/data-honesty requirements defined in `PRD.md`.

---

## 2. Technical Vision

VIGIL uses a layered architecture:

```text
Sensors → MCU/ECU → C Firmware → C++ Edge Processing
                         ↓
                  CAN / Serial / UDP
                         ↓
                 Python Telemetry Gateway
                         ↓
              VIGIL Digital Twin Core
                         ↓
       Physics → Residuals → Features → AI/ML
                         ↓
              Health → Degradation → RUL
                         ↓
                  Mission Intelligence
                         ↓
               REST API / WebSocket
                         ↓
                  VIGIL GCS
```

The system supports two input modes:

```text
REAL SENSOR MODE
Sensors → Embedded Layer → Telemetry Gateway → VIGIL

SIMULATION MODE
Simulator → Telemetry Gateway → VIGIL
```

Both paths converge on the same canonical telemetry contract.

---

# 3. Technology Stack

## 3.1 Embedded / Hardware

### C

Use C for:

- sensor drivers;
- ADC acquisition;
- GPIO;
- interrupt handling;
- low-level timing;
- RPM pulse capture;
- CAN frame acquisition;
- hardware abstraction.

### C++

Use C++ for:

- edge-side processing;
- sensor fusion;
- filtering;
- rolling statistics;
- vibration feature extraction;
- telemetry packet creation;
- local buffering;
- CAN interface abstraction;
- lightweight edge inference where required.

### Hardware Communication

Primary:

- CAN;
- SocketCAN on Linux-capable gateways.

Development/demo alternatives:

- Serial;
- UDP.

The hardware interface must remain replaceable.

---

# 4. Frontend

Core:

- Next.js
- React
- TypeScript

UI:

- Tailwind CSS
- shadcn/ui

Visualization:

- Three.js + React Three Fiber for the interactive engine Digital Twin.
- Apache ECharts for telemetry, residuals, health, degradation, RUL, and mission charts.

The frontend must render authoritative analytical values supplied by the backend rather than independently calculating engineering results.

---

# 5. Backend

Core:

- Python
- FastAPI
- Pydantic
- SQLAlchemy

Responsibilities:

- telemetry ingestion;
- validation;
- Digital Twin orchestration;
- physics-model invocation;
- feature calculation;
- AI/ML inference;
- health estimation;
- RUL estimation;
- mission simulation;
- replay;
- persistence;
- report generation;
- WebSocket streaming.

---

# 6. Scientific Computing

Primary libraries:

- NumPy
- SciPy
- pandas

The physics layer should remain deterministic where deterministic behavior is expected.

---

# 7. AI/ML

## 7.1 Anomaly Detection

Baseline:

**scikit-learn — Isolation Forest**

Pipeline:

```text
Telemetry
  ↓
Validation
  ↓
Physics Baseline
  ↓
Residuals
  ↓
Feature Engineering
  ↓
Isolation Forest
  ↓
Anomaly Score
```

## 7.2 Fault Diagnosis

Baseline:

**XGBoost**

Inputs may include:

- telemetry features;
- physics residuals;
- temporal features;
- vibration features.

Output:

- fault probabilities/confidence.

Initial classes:

- injector degradation;
- cooling degradation;
- lubrication issue;
- mechanical/vibration degradation;
- combustion instability/misfire;
- sensor drift/failure;
- electrical degradation where modeled.

## 7.3 RUL

The initial RUL implementation should prioritize:

- explicit degradation state;
- degradation trend;
- defined prototype end-of-life criterion;
- uncertainty interval;
- reproducibility.

Potential future models:

- LSTM;
- GRU;
- Temporal CNN;
- Transformer.

Do not add a deep model merely for complexity.

---

# 8. Database

## 8.1 PostgreSQL

Store:

- UAV metadata;
- engine metadata;
- missions;
- scenarios;
- fault injections;
- anomaly events;
- diagnoses;
- health states;
- RUL estimates;
- model versions;
- simulator versions;
- reports;
- configuration;
- provenance metadata.

## 8.2 TimescaleDB

Use TimescaleDB for high-volume timestamped telemetry.

Logical relationship:

```text
UAV
 └── Engine
      └── Mission
           └── Telemetry
```

## 8.3 Redis

Use Redis for transient/live state:

- current engine state;
- latest telemetry;
- active alerts;
- WebSocket fan-out;
- temporary simulation state;
- short-lived caching.

Redis is not the authoritative historical datastore.

---

# 9. Digital Twin Core

The Digital Twin is the central stateful analytical component.

```text
DigitalTwin
│
├── Identity
│   ├── UAV
│   └── Engine
│
├── Telemetry State
├── Operating Context
├── Expected State
├── Residual State
├── Health State
├── Fault State
├── Degradation State
├── RUL State
├── Mission State
└── Provenance
    ├── model_version
    ├── simulator_version
    └── timestamp
```

The Digital Twin updates whenever valid telemetry is processed.

The 3D model is a visualization of the Digital Twin; it is not itself the Digital Twin.

---

# 10. Physics Model

## Purpose

Provide an expected healthy engine state under current operating conditions.

```text
Operating Conditions + Engine State
                ↓
       Physics / Reduced-Order Model
                ↓
        Expected Healthy State
```

Requirements:

- documented assumptions;
- unit consistency;
- deterministic behavior where applicable;
- model versioning;
- testability;
- scenario support.

### Engineering Honesty Rule

No physical constant, equation, limit, efficiency, operating envelope, or engine-specific relationship may be invented.

If authoritative information is unavailable:

```text
ASSUMPTION_REQUIRED
```

must be recorded.

---

# 11. Residual Engine

For modeled variables:

```text
residual = observed − expected
```

The residual engine should support:

- raw residual;
- normalized residual where justified;
- rolling mean;
- rolling standard deviation;
- trend;
- rate of change;
- operating-context association.

Residuals must be timestamp-aligned with source telemetry.

---

# 12. Feature Engineering

Potential feature groups:

### Temperature

- EGT;
- CHT;
- oil temperature;
- trends;
- deviations from expected state.

### Pressure

- oil pressure;
- deviation;
- trend.

### Mechanical

- RPM;
- RPM instability;
- vibration;
- vibration RMS;
- trend.

### Fuel / Injection

- fuel flow;
- injection timing;
- residual relationships.

### Electrical

- battery voltage;
- alternator current;
- electrical trends.

### Context

- altitude;
- ambient temperature;
- ambient pressure;
- throttle;
- engine age.

Every production feature must have a documented definition and tests.

---

# 13. Fault Injection

The simulator shall support controlled degradation/fault injection.

```text
Healthy Engine
      ↓
Select Fault
      ↓
Select Severity
      ↓
Apply Fault Signature
      ↓
Sensor Effects / Noise
      ↓
Telemetry
```

Fault injection must be separate from diagnosis to avoid circular validation.

The same:

- simulator version;
- scenario configuration;
- random seed

must reproduce the same generated scenario.

---

# 14. Sensor Fault Modeling

Sensor faults must be modeled independently from engine faults.

Examples:

- bias;
- drift;
- stuck value;
- intermittent dropout;
- increased noise;
- packet loss.

The diagnostic system should determine whether abnormality is:

1. isolated to a sensor;
2. supported by multiple engine parameters;
3. caused by telemetry/data quality.

---

# 15. Canonical Telemetry Contract

All data sources must normalize into one schema.

Example:

```json
{
  "timestamp": "2026-09-11T18:00:01.245Z",
  "uav_id": "UAV-001",
  "engine_id": "AERO-PX-01",
  "mission_id": "MISSION-001",

  "rpm": 2450,
  "cht": 178.4,
  "egt": 692.1,

  "oil_pressure": 4.2,
  "oil_temperature": 94.6,

  "fuel_flow": 18.7,
  "vibration": 0.031,

  "battery_voltage": 27.8,
  "alternator_current": 18.4,

  "injection_timing": 21.5,

  "altitude": 18000,
  "ambient_temperature": 12.3,
  "ambient_pressure": 57.2,
  "throttle": 0.70
}
```

These are schema examples only. Engineering units and valid ranges must be verified before being treated as authoritative.

---

# 16. Data Quality

Each telemetry sample should have a data-quality state:

```text
VALID
INVALID
STALE
MISSING
ESTIMATED
SIMULATED
```

The system must distinguish:

```text
engine abnormality
```

from:

```text
data abnormality
```

---

# 17. Telemetry Adapter

```text
                    Canonical Telemetry
                           ▲
                           │
                  Telemetry Adapter
                           │
              ┌────────────┴────────────┐
              │                         │
       Hardware Adapter          Simulator Adapter
              │                         │
           C / C++                    Python
              │                         │
         CAN / Serial              UDP / WebSocket
              │                         │
           Sensors                  Simulator
```

The adapters must expose the same logical output contract.

---

# 18. Real-Time Data Flow

```text
Sensor
  ↓
C Firmware
  ↓
C++ Edge Layer
  ↓
CAN
  ↓
Telemetry Gateway
  ↓
Validation
  ↓
Digital Twin
  ↓
Physics
  ↓
Residuals
  ↓
AI/ML
  ↓
Health / RUL / Risk
  ↓
Redis
  ↓
WebSocket
  ↓
GCS
```

Historical data is persisted in PostgreSQL/TimescaleDB.

---

# 19. REST API

Use versioned APIs.

Conceptual groups:

```text
/api/v1/telemetry
/api/v1/engines
/api/v1/uavs
/api/v1/missions
/api/v1/twin
/api/v1/health
/api/v1/diagnostics
/api/v1/rul
/api/v1/simulation
/api/v1/replay
/api/v1/reports
/api/v1/models
```

Exact endpoint schemas should be maintained separately as the implementation matures.

---

# 20. WebSocket

WebSocket channels should provide live:

- telemetry;
- Digital Twin state;
- health;
- anomaly events;
- diagnosis events;
- RUL updates;
- mission simulation updates.

The frontend must reconnect gracefully after temporary disconnection.

---

# 21. Mission Simulation

```text
Mission Configuration
        ↓
Environment Model
        ↓
Engine Operating Profile
        ↓
Digital Twin Simulation
        ↓
Physics Model
        ↓
Degradation Model
        ↓
Projected Telemetry
        ↓
Health / RUL / Risk
```

Simulation must not modify real or historical mission state unless explicitly saved as a new scenario.

---

# 22. Mission Risk

Mission risk is a **prototype decision-support metric**.

It may incorporate:

- current health;
- degradation;
- RUL;
- environmental stress;
- mission duration;
- operating conditions.

It must not be presented as:

- aircraft crash probability;
- probability of operational engine failure;
- certified safety assessment.

Relative scenario comparison is preferred.

---

# 23. Replay

```text
Historical Telemetry
        ↓
Timestamp Ordering
        ↓
Twin Reconstruction
        ↓
Physics
        ↓
Residuals
        ↓
Anomaly
        ↓
Diagnosis
        ↓
Health / RUL
        ↓
Replay Timeline
```

Replay should preserve model/data version information where available.

---

# 24. Frontend Architecture

Recommended structure:

```text
frontend/
├── app/
│   ├── dashboard/
│   ├── telemetry/
│   ├── engine-health/
│   ├── digital-twin/
│   ├── diagnostics/
│   ├── rul/
│   ├── mission/
│   ├── replay/
│   ├── reports/
│   └── settings/
│
├── components/
│   ├── engine/
│   ├── telemetry/
│   ├── charts/
│   ├── diagnostics/
│   ├── health/
│   ├── mission/
│   └── common/
│
├── lib/
│   ├── api/
│   ├── websocket/
│   └── utilities/
│
└── types/
```

The exact repository structure is finalized separately.

---

# 25. Frontend State

Separate:

```text
SERVER STATE
API / WebSocket

UI STATE
filters / selected mission / view mode

REPLAY STATE
current timestamp / playback

SIMULATION STATE
scenario configuration / execution
```

The browser must not be the authoritative source of Digital Twin state.

---

# 26. 3D Digital Twin

The visualization shall support:

- engine rotation/animation;
- health visualization;
- subsystem selection;
- fault highlighting;
- operating-state indication;
- temperature/vibration visualization where appropriate;
- interaction/inspection.

It should reflect the current analytical Twin state.

---

# 27. Authentication & Authorization

The architecture should support future roles:

```text
OPERATOR
MAINTENANCE_ENGINEER
MISSION_PLANNER
ADMIN
ANALYST
```

Authorization should be enforced server-side.

The SIH prototype may use simplified authentication if required.

---

# 28. Security

The system should support:

- authenticated APIs;
- input validation;
- restricted administrative operations;
- secure secret management;
- audit logging for critical changes;
- encrypted transport in deployment environments;
- no secrets in Git.

Hardware messages must be validated before entering the canonical telemetry layer.

---

# 29. Observability

Structured logs should cover:

- telemetry ingestion;
- validation failures;
- Digital Twin updates;
- anomaly events;
- diagnosis;
- simulation;
- replay;
- model loading;
- errors.

Important analytical outputs should be traceable to:

- input telemetry;
- model version;
- simulator version;
- configuration.

---

# 30. Model Versioning

Every production inference should identify the model version.

Example:

```json
{
  "model_version": "v0.3.0",
  "simulator_version": "sim-0.5.0"
}
```

Models must not be silently replaced during a running experiment.

---

# 31. Testing Strategy

## Unit Tests

Test:

- telemetry validation;
- feature calculations;
- physics functions;
- residual calculations;
- fault injection;
- health calculations;
- RUL calculations;
- mission calculations;
- API validation.

## Physics Tests

Cover:

- dimensional consistency;
- sanity checks;
- monotonicity where physically expected;
- boundary behavior;
- deterministic behavior.

## ML Tests

Measure where applicable:

- precision;
- recall;
- F1;
- false-alarm rate;
- detection latency;
- confusion matrix;
- macro-F1;
- per-class recall;
- RUL MAE/RMSE;
- interval coverage.

## Dataset Splitting

Do not randomly split individual time-series rows when this causes mission leakage.

Prefer:

```text
Train → Mission A/B/C
Validation → Mission D
Test → Mission E/F
```

---

# 32. End-to-End Test

Mandatory golden path:

```text
Healthy
  ↓
Mission Start
  ↓
Injector Degradation
  ↓
Telemetry Change
  ↓
Residual Increase
  ↓
Anomaly Detection
  ↓
Injector Diagnosis
  ↓
Health Reduction
  ↓
RUL Update
  ↓
Altitude Increase
  ↓
Mission Risk Change
  ↓
Mission Profile Adjustment
  ↓
Projected Improvement
  ↓
Historical Replay
```

Every stage must be reflected correctly in the next stage.

---

# 33. Synthetic Data Architecture

```text
Scenario Configuration
        ↓
Mission Generator
        ↓
Atmosphere Generator
        ↓
Engine Physics
        ↓
Healthy Telemetry
        ↓
Fault Injection
        ↓
Sensor Fault Injection
        ↓
Noise / Missingness
        ↓
Validation
        ↓
Dataset + Labels + Provenance
```

Synthetic generation must support reproducibility using a controlled random seed.

---

# 34. Deployment

Use **Docker Compose** for the SIH prototype.

Recommended services:

```text
frontend
backend
postgres
timescaledb
redis
```

Optional:

```text
simulator
worker
monitoring
```

Kubernetes is not required for the MVP.

---

# 35. Hardware Deployment Path

Future physical deployment:

```text
Sensors
  ↓
MCU / ECU
  ↓
C Firmware
  ↓
CAN
  ↓
Embedded Linux Gateway
  ↓
C++ Adapter
  ↓
VIGIL Telemetry Gateway
  ↓
VIGIL Core
```

Simulation mode must continue working without physical hardware.

---

# 36. Performance

Prioritize predictable responsiveness.

Requirements:

- telemetry processing must not block the UI;
- long-running simulations should execute asynchronously;
- replay must not freeze the dashboard;
- WebSocket updates must remain stable for intended demo load;
- database writes must not block critical real-time processing.

Exact throughput and latency targets shall be benchmarked against actual hardware and workload rather than invented.

---

# 37. Reliability

The system should handle:

- temporary telemetry disconnection;
- malformed telemetry;
- missing values;
- simulator restart;
- backend restart;
- WebSocket reconnect;
- database connection failure;
- unavailable ML model;
- incomplete historical missions.

Failure states must be visible and must not silently become healthy states.

---

# 38. AI Coding-Agent Rules

These rules are mandatory when using Antigravity or another coding agent.

### Rule 1 — Read Before Editing

Inspect repository structure, existing code, configuration, environment variables, tests, and interfaces before changing anything.

### Rule 2 — No Invented Engineering Parameters

Unknown engineering values must be marked:

```text
ASSUMPTION_REQUIRED
```

### Rule 3 — Preserve Interfaces

Do not break established telemetry/API contracts without updating all consumers.

### Rule 4 — Incremental Implementation

```text
SPEC
 ↓
IMPLEMENT
 ↓
TEST
 ↓
REVIEW
 ↓
COMMIT
 ↓
NEXT MODULE
```

### Rule 5 — Avoid Overengineering

Do not introduce Kafka, Kubernetes, Spark, unnecessary microservices, or similar infrastructure without a demonstrated requirement.

### Rule 6 — No Hardcoded AI Results

Diagnosis, health, RUL, anomaly, and risk outputs must originate from actual computation.

### Rule 7 — No Fake Live Data

Simulation must be explicitly labeled as simulation.

### Rule 8 — Backend Authority

Authoritative health, diagnosis, RUL, Digital Twin, and mission-risk calculations belong to the backend/core analytical layer.

---

# 39. Recommended Repository Structure

```text
vigil/
│
├── frontend/                  # Next.js / React / TypeScript
│
├── backend/                   # FastAPI
│   ├── api/
│   ├── core/
│   ├── digital_twin/
│   ├── physics/
│   ├── features/
│   ├── anomaly/
│   ├── diagnosis/
│   ├── health/
│   ├── rul/
│   ├── mission/
│   ├── replay/
│   ├── telemetry/
│   └── models/
│
├── embedded/
│   ├── firmware-c/
│   └── edge-cpp/
│
├── simulator/
│
├── data/
│   ├── schemas/
│   ├── synthetic/
│   └── scenarios/
│
├── ml/
│   ├── training/
│   ├── evaluation/
│   └── artifacts/
│
├── database/
│   └── migrations/
│
├── tests/
├── docs/
├── docker/
│
├── PRD.md
├── TRD.md
└── README.md
```

---

# 40. Technology Decision Summary

| Layer | Technology |
|---|---|
| Sensor firmware | **C** |
| Edge processing | **C++** |
| Hardware bus | **CAN** |
| Linux CAN | **SocketCAN** |
| Telemetry gateway | **Python + FastAPI** |
| Scientific computing | **NumPy + SciPy + pandas** |
| Digital Twin | **Python** |
| Anomaly detection | **scikit-learn / Isolation Forest** |
| Fault diagnosis | **XGBoost** |
| RUL | **Python degradation/ML pipeline** |
| Database | **PostgreSQL** |
| Time-series | **TimescaleDB** |
| Live state/cache | **Redis** |
| ORM | **SQLAlchemy** |
| Validation | **Pydantic** |
| Frontend | **Next.js + React + TypeScript** |
| UI | **Tailwind CSS + shadcn/ui** |
| 3D | **Three.js + React Three Fiber** |
| Charts | **Apache ECharts** |
| Real-time frontend | **WebSocket** |
| Testing | **Pytest + Playwright** |
| Deployment | **Docker + Docker Compose** |
| Version control | **Git + GitHub** |

---

# 41. Architecture Decisions

## ADR-001 — Python for VIGIL Core

Python is the primary language for Digital Twin, physics, analytics, AI/ML, RUL, and mission simulation because the scientific and ML ecosystem aligns with the project requirements.

## ADR-002 — C/C++ for Edge

C handles low-level acquisition; C++ handles edge processing. This provides a credible embedded path while keeping high-level intelligence in Python.

## ADR-003 — CAN as Primary Hardware Interface

CAN is the primary hardware telemetry interface, with SocketCAN support for Linux gateways.

## ADR-004 — PostgreSQL + TimescaleDB

Use PostgreSQL for structured data and TimescaleDB for timestamped telemetry.

## ADR-005 — Redis for Transient State

Redis provides low-latency live state and WebSocket support but is not authoritative history.

## ADR-006 — Next.js GCS

Next.js/React/TypeScript provides the application foundation for the operator GCS.

## ADR-007 — Three.js Digital Twin

Three.js/React Three Fiber provides the interactive 3D engine visualization.

## ADR-008 — Modular Backend First

Start with a modular backend and Docker Compose instead of distributed microservices.

---

# 42. Technical Definition of Done

The implementation is ready for SIH integration when:

- [ ] C/C++ telemetry adapter interface exists.
- [ ] Simulator implements the same canonical telemetry contract.
- [ ] FastAPI ingestion works.
- [ ] Telemetry validation works.
- [ ] PostgreSQL/TimescaleDB persistence works.
- [ ] Redis live-state path works.
- [ ] Digital Twin state updates correctly.
- [ ] Physics baseline produces expected state.
- [ ] Residual engine works.
- [ ] Feature engineering works.
- [ ] Anomaly model works.
- [ ] Fault classifier works.
- [ ] Health engine works.
- [ ] RUL engine works.
- [ ] Mission simulator works.
- [ ] Replay works.
- [ ] REST APIs work.
- [ ] WebSocket stream works.
- [ ] GCS consumes real backend outputs.
- [ ] 3D Digital Twin reflects live state.
- [ ] Critical paths have automated tests.
- [ ] Golden E2E scenario passes.
- [ ] Model/data provenance is preserved.
- [ ] Synthetic data is clearly labeled.
- [ ] No unsupported engineering constants or claims are embedded.

---

# 43. Final Technical Principle

VIGIL is a **hardware-ready, simulation-capable, physics-informed, AI-enabled Digital Twin platform**.

The fundamental separation is:

```text
C
↓
Acquire

C++
↓
Process at Edge

Python
↓
Model / Understand / Predict

TypeScript
↓
Visualize / Interact
```

The simulator and physical sensor pipeline must converge at the same telemetry interface so that the Digital Twin, AI/ML, RUL, mission simulation, replay, and GCS remain independent of the original data source.

This architecture is the technical foundation for the VIGIL MVP and its future transition from an SIH demonstrator toward a hardware-connected engine health and prognostics platform.
