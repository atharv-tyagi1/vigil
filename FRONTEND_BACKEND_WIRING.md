# VIGIL — Frontend ↔ Backend Wiring Specification
## SIH26054 | Intelligent UAV Engine Health & Prognostics

**Status:** MVP implementation blueprint  
**Purpose:** Wire the VIGIL Next.js frontend to the FastAPI backend, Digital Twin, AI/ML services, PostgreSQL/TimescaleDB, Redis, and the telemetry stream without coupling the UI to simulation internals.

---

# 1. Wiring Goal

The frontend must never directly calculate engine health, fault probabilities, RUL, or mission risk.

The backend owns:

- telemetry ingestion
- validation and normalization
- physics/expected-state calculation
- residual generation
- anomaly detection
- fault diagnosis
- health estimation
- degradation estimation
- RUL prediction
- mission simulation
- replay
- persistence
- model/version provenance

The frontend owns:

- visualization
- user interaction
- scenario configuration
- filtering
- replay controls
- dashboard state
- presentation of backend results

The primary data flow is:

```text
Sensors / Simulator
        ↓
C / C++ Telemetry Layer
        ↓
CAN / SocketCAN / Simulator Adapter
        ↓
FastAPI Telemetry Ingestion
        ↓
Validation + Normalization
        ↓
Digital Twin Engine
   ┌────┴────┐
   ↓         ↓
Physics    AI/ML
Model      Models
   └────┬────┘
        ↓
Twin State
        ↓
PostgreSQL/TimescaleDB + Redis
        ↓
REST API + WebSocket
        ↓
Next.js Frontend
        ↓
Dashboard / Diagnostics / RUL / Mission / Replay
```

---

# 2. Repository Structure

Recommended implementation structure:

```text
vigil/
├── apps/
│   ├── web/
│   │   ├── app/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   │   ├── api.ts
│   │   │   ├── websocket.ts
│   │   │   ├── types.ts
│   │   │   └── constants.ts
│   │   └── stores/
│   │
│   └── api/
│       ├── app/
│       │   ├── main.py
│       │   ├── api/
│       │   │   ├── routes/
│       │   │   │   ├── telemetry.py
│       │   │   │   ├── dashboard.py
│       │   │   │   ├── diagnostics.py
│       │   │   │   ├── rul.py
│       │   │   │   ├── mission.py
│       │   │   │   └── replay.py
│       │   │   ├── websocket.py
│       │   │   └── dependencies.py
│       │   ├── core/
│       │   ├── schemas/
│       │   ├── services/
│       │   │   ├── telemetry_service.py
│       │   │   ├── twin_service.py
│       │   │   ├── diagnostic_service.py
│       │   │   ├── rul_service.py
│       │   │   ├── mission_service.py
│       │   │   └── replay_service.py
│       │   ├── twin/
│       │   │   ├── physics.py
│       │   │   ├── residuals.py
│       │   │   ├── health.py
│       │   │   └── degradation.py
│       │   ├── ml/
│       │   │   ├── anomaly.py
│       │   │   ├── diagnosis.py
│       │   │   └── rul.py
│       │   ├── db/
│       │   └── tests/
│       └── requirements.txt
│
├── embedded/
│   ├── firmware-c/
│   └── edge-cpp/
│
├── simulator/
├── docker-compose.yml
└── README.md
```

---

# 3. Environment Variables

## Backend

```env
APP_ENV=development
API_HOST=0.0.0.0
API_PORT=8000

DATABASE_URL=postgresql+asyncpg://vigil:vigil@localhost:5432/vigil
REDIS_URL=redis://localhost:6379/0

CORS_ORIGINS=http://localhost:3000

TELEMETRY_MODE=simulation
MODEL_VERSION=v0.1.0
SIMULATOR_VERSION=sim-0.1.0
```

## Frontend

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_WS_URL=ws://localhost:8000
```

Never put database credentials or private backend secrets in `NEXT_PUBLIC_*` variables.

---

# 4. Canonical Telemetry Contract

Every telemetry source must produce the same normalized payload.

```json
{
  "timestamp": "2026-09-11T12:00:00Z",
  "uav_id": "UAV-001",
  "engine_id": "AERO-PX-01",
  "mission_id": "MISSION-001",

  "rpm": 2450.0,
  "cht": 168.4,
  "egt": 712.5,
  "oil_pressure": 4.1,
  "oil_temperature": 94.2,
  "fuel_flow": 18.7,
  "vibration_rms": 2.8,

  "battery_voltage": 24.1,
  "alternator_current": 18.2,
  "injection_timing": 21.5,

  "altitude": 8500.0,
  "throttle": 0.72,

  "quality": {
    "valid": true,
    "missing_fields": [],
    "source": "simulator"
  }
}
```

Do not invent physical operating limits. Limits must come from an approved engine profile/configuration.

---

# 5. Backend API

Base URL:

```text
/api/v1
```

## 5.1 Health Check

```http
GET /health
```

Response:

```json
{
  "status": "ok",
  "service": "vigil-api",
  "version": "0.1.0"
}
```

---

# 6. Dashboard API

```http
GET /api/v1/dashboard/{engine_id}
```

Response:

```json
{
  "engine": {
    "id": "AERO-PX-01",
    "uav_id": "UAV-001",
    "status": "ACTIVE"
  },
  "health": {
    "score": 92.0,
    "status": "GOOD"
  },
  "anomaly": {
    "score": 0.18,
    "detected": false
  },
  "rul": {
    "hours": 412,
    "lower": 380,
    "upper": 450
  },
  "mission_risk": {
    "level": "LOW",
    "score": 0.12
  },
  "latest_telemetry": {},
  "active_faults": [],
  "updated_at": "2026-09-11T12:00:00Z"
}
```

The dashboard should load this endpoint once and then switch to WebSocket updates.

---

# 7. Telemetry REST API

## Latest telemetry

```http
GET /api/v1/telemetry/{engine_id}/latest
```

## Historical telemetry

```http
GET /api/v1/telemetry/{engine_id}?from=<ISO>&to=<ISO>&limit=1000
```

Example response:

```json
{
  "engine_id": "AERO-PX-01",
  "items": [
    {
      "timestamp": "2026-09-11T12:00:00Z",
      "rpm": 2450,
      "cht": 168.4,
      "egt": 712.5,
      "oil_pressure": 4.1,
      "oil_temperature": 94.2,
      "fuel_flow": 18.7,
      "vibration_rms": 2.8
    }
  ]
}
```

---

# 8. Digital Twin API

```http
GET /api/v1/twin/{engine_id}
```

Response:

```json
{
  "engine_id": "AERO-PX-01",
  "health": 92,
  "anomaly_score": 0.18,
  "state": "HEALTHY",
  "model_version": "v0.1.0",
  "simulator_version": "sim-0.1.0",
  "expected": {
    "rpm": 2445,
    "cht": 166.9,
    "egt": 705.1,
    "oil_pressure": 4.2,
    "fuel_flow": 18.1
  },
  "observed": {
    "rpm": 2450,
    "cht": 168.4,
    "egt": 712.5,
    "oil_pressure": 4.1,
    "fuel_flow": 18.7
  },
  "residuals": {
    "rpm": 5,
    "cht": 1.5,
    "egt": 7.4,
    "oil_pressure": -0.1,
    "fuel_flow": 0.6
  }
}
```

The frontend renders these values. It must not reproduce the physics calculations.

---

# 9. Diagnostics API

```http
GET /api/v1/diagnostics/{engine_id}
```

Response:

```json
{
  "engine_id": "AERO-PX-01",
  "anomaly_score": 0.84,
  "diagnosis": [
    {
      "fault": "injector_degradation",
      "probability": 0.84,
      "severity": "HIGH"
    },
    {
      "fault": "cooling_degradation",
      "probability": 0.21,
      "severity": "LOW"
    }
  ],
  "evidence": [
    {
      "signal": "fuel_flow",
      "direction": "HIGH",
      "contribution": 0.31
    },
    {
      "signal": "egt",
      "direction": "HIGH",
      "contribution": 0.27
    },
    {
      "signal": "rpm",
      "direction": "UNSTABLE",
      "contribution": 0.18
    },
    {
      "signal": "vibration_rms",
      "direction": "HIGH",
      "contribution": 0.08
    }
  ]
}
```

This endpoint powers the AI diagnosis panel.

---

# 10. RUL API

```http
GET /api/v1/rul/{engine_id}
```

Response:

```json
{
  "engine_id": "AERO-PX-01",
  "rul_hours": 412,
  "lower": 380,
  "upper": 450,
  "confidence": 0.82,
  "trend": "STABLE",
  "model_version": "rul-v0.1.0"
}
```

RUL values must be explicitly labeled as prototype/model estimates unless validated against suitable degradation data.

---

# 11. Mission Simulation API

## Create simulation

```http
POST /api/v1/mission/simulate
```

Request:

```json
{
  "engine_id": "AERO-PX-01",
  "duration_hours": 8,
  "altitude": 8500,
  "throttle_profile": "ENDURANCE",
  "ambient_temperature": null,
  "fault_scenario": null
}
```

If an engineering parameter is unavailable, the API must reject it or return `ASSUMPTION_REQUIRED`. It must not silently fabricate engineering values.

## Response

```json
{
  "simulation_id": "SIM-001",
  "status": "COMPLETED",
  "risk": {
    "level": "LOW",
    "score": 0.16
  },
  "results": {
    "health_start": 92,
    "health_end": 88,
    "thermal_margin": null,
    "rul_start": 412,
    "rul_end": 397
  }
}
```

---

# 12. Fault Injection API

Used for the SIH demonstration.

```http
POST /api/v1/simulation/fault
```

Request:

```json
{
  "engine_id": "AERO-PX-01",
  "fault_type": "injector_degradation",
  "severity": 0.35,
  "duration_seconds": 120
}
```

The simulator should inject a controlled synthetic disturbance, not claim that the generated signature represents validated real-engine behavior.

---

# 13. Replay API

## List missions

```http
GET /api/v1/replay/missions
```

## Load replay

```http
GET /api/v1/replay/{mission_id}
```

## Start replay

```http
POST /api/v1/replay/{mission_id}/start
```

Request:

```json
{
  "speed": 10
}
```

The WebSocket stream then emits replay telemetry using the same schema as live telemetry.

---

# 14. WebSocket Contract

Primary endpoint:

```text
ws://localhost:8000/api/v1/ws/engines/{engine_id}
```

The backend pushes state changes.

## Telemetry message

```json
{
  "type": "telemetry",
  "data": {
    "timestamp": "2026-09-11T12:00:01Z",
    "rpm": 2450,
    "cht": 168.4,
    "egt": 712.5,
    "oil_pressure": 4.1,
    "fuel_flow": 18.7,
    "vibration_rms": 2.8
  }
}
```

## Twin update

```json
{
  "type": "twin_update",
  "data": {
    "health": 92,
    "anomaly_score": 0.18,
    "state": "HEALTHY"
  }
}
```

## Diagnostic event

```json
{
  "type": "diagnostic_event",
  "data": {
    "fault": "injector_degradation",
    "probability": 0.84,
    "severity": "HIGH"
  }
}
```

## RUL update

```json
{
  "type": "rul_update",
  "data": {
    "rul_hours": 412,
    "lower": 380,
    "upper": 450
  }
}
```

## Mission risk update

```json
{
  "type": "mission_risk",
  "data": {
    "level": "LOW",
    "score": 0.12
  }
}
```

---

# 15. Frontend API Client

Create:

```text
apps/web/lib/api.ts
```

Responsibilities:

- define `API_BASE_URL`
- attach request headers
- handle JSON
- normalize API errors
- expose typed functions
- never duplicate backend business logic

Suggested functions:

```text
getDashboard(engineId)
getLatestTelemetry(engineId)
getTelemetryHistory(engineId, params)
getTwinState(engineId)
getDiagnostics(engineId)
getRul(engineId)
simulateMission(payload)
injectFault(payload)
getReplayMissions()
getReplay(missionId)
startReplay(missionId, speed)
```

---

# 16. Frontend Types

Create:

```text
apps/web/lib/types.ts
```

Core types:

```ts
type Telemetry = {
  timestamp: string;
  rpm: number;
  cht: number;
  egt: number;
  oil_pressure: number;
  oil_temperature?: number;
  fuel_flow: number;
  vibration_rms: number;
  battery_voltage?: number;
  alternator_current?: number;
  injection_timing?: number;
};

type TwinState = {
  health: number;
  anomaly_score: number;
  state: string;
  expected: Record<string, number>;
  observed: Record<string, number>;
  residuals: Record<string, number>;
};

type Diagnosis = {
  fault: string;
  probability: number;
  severity: string;
};

type RulPrediction = {
  rul_hours: number;
  lower: number;
  upper: number;
  confidence: number;
  trend: string;
};

type MissionRisk = {
  level: string;
  score: number;
};
```

Types must mirror the backend contract. If the contract changes, update both sides deliberately.

---

# 17. WebSocket Hook

Create:

```text
apps/web/hooks/useEngineStream.ts
```

Responsibilities:

1. Open WebSocket.
2. Subscribe to one engine.
3. Parse message `type`.
4. Update local/Zustand state.
5. Track connection state.
6. Reconnect after temporary disconnect.
7. Stop reconnecting when the component unmounts.
8. Mark data stale when no update arrives within the configured UI timeout.

State:

```text
connecting
connected
reconnecting
disconnected
stale
```

Do not display "LIVE" merely because the socket is connected. "LIVE" should indicate recent valid telemetry.

---

# 18. Recommended Frontend State

Use Zustand or React context for the MVP.

Suggested store:

```text
engineStore
├── selectedEngine
├── connectionStatus
├── latestTelemetry
├── telemetryHistory
├── twinState
├── diagnostics
├── rul
├── missionRisk
├── activeAlerts
└── lastUpdated
```

Data ownership:

```text
REST
  ↓
Initial state

WebSocket
  ↓
Live updates

User controls
  ↓
REST mutation
  ↓
Backend
  ↓
WebSocket update
  ↓
UI
```

This avoids maintaining separate frontend-only copies of calculated engine state.

---

# 19. Dashboard Wiring

Route:

```text
/dashboard
```

On mount:

```text
GET /dashboard/{engineId}
        ↓
populate dashboard
        ↓
connect WebSocket
        ↓
receive live updates
        ↓
update cards/charts/twin/alerts
```

Component mapping:

```text
Dashboard
├── EngineSelector
├── SystemStatusBar
├── HealthScoreCard        ← twin.health
├── EngineTwin3D           ← telemetry + twin state
├── TelemetryCards         ← telemetry
├── LiveTelemetryChart     ← telemetry history
├── ExpectedActualChart    ← twin.expected/observed
├── DiagnosisPanel         ← diagnostics
├── RULCard                ← rul
├── MissionRiskCard        ← missionRisk
└── AlertDrawer            ← diagnostic events
```

---

# 20. Telemetry Page Wiring

Route:

```text
/telemetry
```

REST:

```text
GET /telemetry/{engineId}?from=...&to=...
```

WebSocket:

```text
WS /ws/engines/{engineId}
```

Charts:

```text
RPM
CHT
EGT
Oil Pressure
Oil Temperature
Fuel Flow
Vibration RMS
```

Allow:

- time range selection
- pause/resume visual updates
- signal selection
- expected-vs-actual overlay

The pause button should pause chart rendering, not stop backend telemetry ingestion.

---

# 21. Diagnostics Page Wiring

Route:

```text
/diagnostics
```

Load:

```text
GET /diagnostics/{engineId}
```

Live events:

```text
diagnostic_event
```

UI:

```text
Anomaly Score
      ↓
Probable Faults
      ↓
Evidence
      ↓
Severity
      ↓
Recommended Action
```

Do not expose an AI probability as certainty.

Use wording such as:

```text
Probable fault
Model confidence
Supporting evidence
Prototype advisory
```

---

# 22. RUL Page Wiring

Route:

```text
/rul
```

Load:

```text
GET /rul/{engineId}
```

Live:

```text
rul_update
```

Display:

- estimated RUL
- prediction interval
- confidence
- degradation trend
- model version
- timestamp

Use the prediction interval prominently; a single RUL number should not imply false precision.

---

# 23. Mission Page Wiring

Route:

```text
/mission
```

User selects:

```text
engine
duration
altitude
throttle profile
environment
fault scenario
```

Submit:

```text
POST /mission/simulate
```

Then poll or subscribe to simulation progress.

For MVP, a synchronous simulation endpoint is acceptable if execution is fast.

Display:

```text
Baseline Mission
       VS
Scenario Mission
```

Compare:

- health
- anomaly score
- RUL
- risk
- telemetry
- warnings

---

# 24. Fault Injection Wiring

Demo flow:

```text
Healthy Engine
      ↓
POST /simulation/fault
      ↓
Simulator changes telemetry
      ↓
Telemetry ingestion
      ↓
Physics baseline
      ↓
Residuals increase
      ↓
Anomaly detector
      ↓
Diagnosis
      ↓
Health update
      ↓
RUL update
      ↓
WebSocket
      ↓
Dashboard visibly changes
```

This is the core SIH demonstration loop.

---

# 25. Backend Service Boundaries

## telemetry_service.py

```text
ingest_telemetry()
validate_telemetry()
normalize_telemetry()
persist_telemetry()
publish_telemetry()
```

## twin_service.py

```text
update_twin()
calculate_expected_state()
calculate_residuals()
calculate_health()
publish_twin_state()
```

## diagnostic_service.py

```text
detect_anomaly()
classify_fault()
generate_evidence()
publish_diagnostic_event()
```

## rul_service.py

```text
estimate_rul()
estimate_interval()
publish_rul_update()
```

## mission_service.py

```text
create_scenario()
run_simulation()
calculate_mission_risk()
persist_simulation()
```

---

# 26. End-to-End Backend Pipeline

Pseudo-flow:

```python
async def process_telemetry(sample):
    validated = validate(sample)

    await telemetry_repository.save(validated)

    twin = twin_engine.update(validated)

    residuals = residual_engine.calculate(
        observed=validated,
        expected=twin.expected
    )

    anomaly = anomaly_model.predict(
        telemetry=validated,
        residuals=residuals
    )

    diagnosis = diagnostic_model.predict(
        telemetry=validated,
        residuals=residuals
    )

    health = health_engine.calculate(
        twin=twin,
        anomaly=anomaly,
        diagnosis=diagnosis
    )

    rul = rul_engine.predict(
        telemetry=validated,
        health=health
    )

    state = build_twin_state(
        telemetry=validated,
        twin=twin,
        residuals=residuals,
        anomaly=anomaly,
        diagnosis=diagnosis,
        health=health,
        rul=rul
    )

    await twin_repository.save(state)
    await websocket_manager.publish(state)
```

The exact engineering equations/models must come from the approved model specification. Do not invent physical constants or thresholds.

---

# 27. Error Contract

All API errors should follow one shape:

```json
{
  "error": {
    "code": "INVALID_TELEMETRY",
    "message": "Telemetry validation failed",
    "details": {
      "field": "rpm"
    },
    "request_id": "req-123"
  }
}
```

Frontend should convert this into:

```text
User-facing message
+
technical detail in console/log
```

Never expose stack traces in production UI.

---

# 28. Stale Data Handling

The UI must distinguish:

```text
LIVE
STALE
DISCONNECTED
SIMULATION
REPLAY
```

Example:

```text
LIVE
Last update: 0.4 s ago
```

If telemetry stops:

```text
STALE
Last update: 4.2 s ago
```

If WebSocket disconnects:

```text
DISCONNECTED
Reconnecting...
```

The frontend must never continue presenting old telemetry as live.

---

# 29. CORS

Development:

```text
http://localhost:3000
```

Production:

```text
https://<approved-frontend-domain>
```

FastAPI:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Use explicit origins in production.

---

# 30. Docker Wiring

Recommended services:

```text
web
api
postgres
redis
simulator
```

Development flow:

```text
docker compose up
```

Ports:

```text
Frontend → 3000
FastAPI  → 8000
Postgres → 5432
Redis    → 6379
```

---

# 31. MVP Simplification for SIH

Because implementation time is limited, build in this order:

### P0 — Must work

```text
FastAPI
  ↓
Synthetic telemetry generator
  ↓
Digital Twin state
  ↓
WebSocket
  ↓
Next.js dashboard
```

Then add:

```text
Fault injection
      ↓
Anomaly detection
      ↓
Diagnosis
      ↓
RUL
```

Then:

```text
Mission simulation
Replay
Reports
```

Do not spend MVP time on:

- complex authentication
- federated learning
- production Kubernetes
- advanced distributed messaging
- elaborate RBAC
- high-fidelity 3D engine geometry
- unvalidated real-engine safety claims

---

# 32. First Working Vertical Slice

The first milestone is considered complete only when this works:

```text
Open http://localhost:3000/dashboard

        ↓

Dashboard calls:
GET /api/v1/dashboard/AERO-PX-01

        ↓

Dashboard receives initial state

        ↓

Frontend opens:
WS /api/v1/ws/engines/AERO-PX-01

        ↓

Backend streams telemetry every simulation tick

        ↓

RPM / CHT / EGT / Oil Pressure /
Fuel Flow / Vibration charts update

        ↓

Health card updates

        ↓

Digital Twin visualization updates

        ↓

Inject fault

        ↓

Anomaly appears

        ↓

Diagnosis appears

        ↓

Health decreases

        ↓

RUL changes

        ↓

Mission risk changes
```

---

# 33. API Wiring Test

Backend integration test:

```text
1. Start API.
2. Start simulator.
3. POST/ingest telemetry.
4. Confirm database row.
5. Confirm twin state generated.
6. Confirm WebSocket message.
7. Inject fault.
8. Confirm anomaly event.
9. Confirm diagnosis.
10. Confirm health/RUL update.
```

Frontend E2E test:

```text
1. Open dashboard.
2. Verify engine health is visible.
3. Verify telemetry values update.
4. Verify connection indicator says LIVE.
5. Trigger demo fault.
6. Verify alert appears.
7. Verify diagnosis changes.
8. Verify RUL panel changes.
9. Verify mission risk changes.
```

---

# 34. AI Coding Agent Rules

When implementing this specification with Antigravity/Cursor/other coding agents:

```text
RULE 1
Implement one vertical slice at a time.

RULE 2
Do not invent API fields.

RULE 3
Do not invent engine physics parameters.

RULE 4
Use the canonical telemetry schema.

RULE 5
Keep frontend business-logic-free.

RULE 6
Keep backend calculations outside route handlers.

RULE 7
Every API endpoint gets a typed request/response schema.

RULE 8
Every WebSocket message has a `type`.

RULE 9
Simulation and real telemetry use the same normalized interface.

RULE 10
Run tests after every module.

RULE 11
Never silently hide missing engineering data.
Use ASSUMPTION_REQUIRED.

RULE 12
Do not claim prototype values are flight-certified thresholds.

RULE 13
Commit after each working vertical slice.
```

---

# 35. Recommended Implementation Sequence

## Step 1 — Backend skeleton

```text
FastAPI
Pydantic
CORS
health endpoint
```

## Step 2 — Canonical telemetry

```text
Telemetry schema
validation
simulator
```

## Step 3 — WebSocket

```text
connection manager
engine stream
telemetry events
```

## Step 4 — Frontend API client

```text
types.ts
api.ts
websocket.ts
useEngineStream.ts
```

## Step 5 — Dashboard

```text
initial REST state
live WebSocket state
cards
charts
status indicators
```

## Step 6 — Digital Twin

```text
expected state
residuals
health
```

## Step 7 — AI

```text
anomaly
diagnosis
evidence
```

## Step 8 — Prognostics

```text
RUL
uncertainty
degradation trend
```

## Step 9 — Mission

```text
scenario
simulation
risk
comparison
```

## Step 10 — Replay

```text
historical mission
timeline
playback
```

---

# 36. Definition of Done

Frontend/backend wiring is complete when:

- [ ] Frontend starts independently.
- [ ] Backend starts independently.
- [ ] Frontend can call backend.
- [ ] CORS works.
- [ ] Health endpoint works.
- [ ] Dashboard loads initial state.
- [ ] WebSocket connects.
- [ ] Telemetry updates live.
- [ ] Digital Twin state updates live.
- [ ] Fault injection changes telemetry.
- [ ] Anomaly event reaches UI.
- [ ] Diagnosis reaches UI.
- [ ] RUL reaches UI.
- [ ] Mission risk reaches UI.
- [ ] Replay uses the same telemetry contract.
- [ ] API errors have a consistent schema.
- [ ] Stale/disconnected states are visible.
- [ ] No frontend calculation duplicates backend engineering logic.
- [ ] No unsupported engineering assumptions are presented as facts.

---

# 37. Antigravity Implementation Prompt

Paste the following prompt into the coding agent:

```text
You are the Principal Software Engineer implementing VIGIL for SIH26054.

Implement the FRONTEND ↔ BACKEND WIRING specification in this document.

IMPORTANT:
Do not redesign the architecture.
Do not invent engine physics, thresholds, constants, sensor limits, fault signatures, or engineering relationships.
If required engineering information is missing, use ASSUMPTION_REQUIRED and keep the implementation configurable.

OBJECTIVE:
Create a working end-to-end MVP where:

Simulator
→ canonical telemetry
→ FastAPI
→ Digital Twin service
→ anomaly/diagnosis/RUL services
→ PostgreSQL/Redis where available
→ WebSocket
→ Next.js dashboard.

PHASE 1:
1. Inspect the existing repository.
2. Preserve working code.
3. Create missing frontend/backend directories.
4. Implement FastAPI application.
5. Implement Pydantic canonical telemetry schemas.
6. Implement /health.
7. Implement dashboard endpoint.
8. Implement telemetry endpoint.
9. Implement WebSocket engine stream.
10. Implement deterministic synthetic telemetry generator.
11. Implement frontend typed API client.
12. Implement frontend WebSocket hook.
13. Wire dashboard to REST + WebSocket.
14. Add connection/stale/disconnected states.

PHASE 2:
1. Implement Digital Twin service interface.
2. Implement configurable expected-state provider.
3. Implement residual calculation.
4. Implement prototype health calculation only where specification permits.
5. Implement anomaly service.
6. Implement diagnostic service.
7. Implement RUL service.
8. Implement WebSocket events for twin, diagnosis and RUL.

PHASE 3:
1. Implement fault injection.
2. Implement mission simulation.
3. Implement mission risk interface.
4. Implement replay.

ENGINEERING RULE:
Do not create fake scientific equations just to make the UI look convincing.
For the SIH demo, deterministic synthetic data is acceptable, but clearly label it as simulated/prototype data.

FRONTEND RULE:
The frontend displays backend results. It must not calculate health, fault probability, RUL, or mission risk independently.

TESTING:
After each phase:
- run backend unit tests
- run frontend typecheck
- run frontend lint
- run integration tests
- fix errors before continuing

DELIVERABLE:
A runnable project where:
http://localhost:3000/dashboard
shows live engine telemetry and reacts to a simulated fault.

At the end provide:
1. files created/changed
2. commands to run
3. tests executed
4. remaining blockers
5. exact next implementation step

Do not stop after creating placeholders. Implement the smallest complete working vertical slice first.
```

---

# 38. Final Architecture

```text
                    ┌──────────────────────┐
                    │ REAL ENGINE / SENSOR │
                    └──────────┬───────────┘
                               │
                         C / C++
                               │
                         CAN / SocketCAN
                               │
                    ┌──────────▼───────────┐
                    │ Telemetry Gateway    │
                    │ FastAPI               │
                    └──────────┬───────────┘
                               │
                ┌──────────────▼──────────────┐
                │       VIGIL CORE            │
                │                             │
                │ Physics / Twin              │
                │ Residuals                   │
                │ Anomaly Detection           │
                │ Diagnosis                   │
                │ Health                      │
                │ RUL                         │
                │ Mission Risk                │
                └──────────────┬──────────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
          PostgreSQL/TimescaleDB          Redis
                 │                           │
                 └─────────────┬─────────────┘
                               │
                        REST + WebSocket
                               │
                    ┌──────────▼───────────┐
                    │ Next.js / React      │
                    │ TypeScript           │
                    ├──────────────────────┤
                    │ Dashboard            │
                    │ Telemetry            │
                    │ Digital Twin         │
                    │ Diagnostics          │
                    │ RUL                  │
                    │ Mission              │
                    │ Replay               │
                    └──────────────────────┘
```

**The key design decision is the canonical telemetry contract:** real sensors and the simulator must enter the exact same backend pipeline. This lets the SIH prototype demonstrate realistic end-to-end wiring now while keeping the architecture ready for actual C/C++ sensor/CAN integration later.
