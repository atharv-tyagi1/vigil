# VIGIL — Special File & Folder Structure
## SIH26054 | Intelligent UAV Engine Health & Prognostics

**Status:** Temporary implementation structure for the SIH 2026 hackathon MVP  
**Purpose:** Define the exact repository structure before implementation so the frontend, backend, embedded layer, simulator, Digital Twin, AI/ML, database, documentation, and deployment assets remain organized.

> **Important:** This document is a temporary implementation-planning file. It exists to guide development during the hackathon and can be removed, archived, or replaced after the project structure stabilizes.

---

# 1. Repository Root

The complete project should use the following root structure:

```text
VIGIL/
│
├── apps/
│   ├── web/                         # Next.js frontend / GCS
│   └── api/                         # FastAPI backend
│
├── embedded/
│   ├── firmware-c/                  # C sensor/MCU firmware
│   └── edge-cpp/                    # C++ edge processing + CAN
│
├── simulator/                       # Synthetic engine + mission simulator
│
├── twin/                            # Digital Twin engineering core
│
├── ml/                              # AI/ML models and pipelines
│
├── database/
│   ├── migrations/                  # Alembic migrations
│   ├── seeds/                       # Development/demo seed data
│   └── scripts/                     # DB utilities
│
├── configs/
│   ├── engine_profiles/
│   ├── model_configs/
│   ├── simulator/
│   └── environments/
│
├── scripts/                         # Developer/automation scripts
│
├── tests/
│   ├── integration/
│   ├── e2e/
│   ├── fixtures/
│   └── contracts/
│
├── docs/
│   ├── architecture/
│   ├── api/
│   ├── engineering/
│   ├── demo/
│   └── decisions/
│
├── deployment/
│   ├── docker/
│   └── monitoring/
│
├── data/
│   ├── sample/
│   ├── generated/
│   └── replay/
│
├── models/
│   ├── anomaly/
│   ├── diagnosis/
│   └── rul/
│
├── .github/
│   └── workflows/
│
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Makefile
├── README.md
└── VIGIL_SPECIAL_FILE_STRUCTURE.md   # TEMPORARY — delete/archive later
```

---

# 2. Why This Structure

The project has multiple engineering domains:

```text
Hardware
   ↓
Embedded C
   ↓
Edge C++
   ↓
CAN
   ↓
Python Backend
   ↓
Digital Twin + AI/ML
   ↓
Database
   ↓
REST/WebSocket
   ↓
Next.js Frontend
```

Keeping these domains separated prevents:

- frontend code from containing backend calculations
- simulator code from being mixed with production services
- ML notebooks from becoming production services
- embedded code from depending on Python
- database migrations from being mixed with application code
- temporary hackathon files from becoming permanent architecture

---

# 3. Application Layer

```text
apps/
│
├── web/
└── api/
```

These are the two main deployable applications.

---

# 4. Frontend Folder Structure

```text
apps/web/
│
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── globals.css
│   │
│   ├── dashboard/
│   │   └── page.tsx
│   │
│   ├── telemetry/
│   │   └── page.tsx
│   │
│   ├── engine-health/
│   │   └── page.tsx
│   │
│   ├── digital-twin/
│   │   └── page.tsx
│   │
│   ├── diagnostics/
│   │   └── page.tsx
│   │
│   ├── rul/
│   │   └── page.tsx
│   │
│   ├── mission/
│   │   └── page.tsx
│   │
│   ├── replay/
│   │   └── page.tsx
│   │
│   ├── reports/
│   │   └── page.tsx
│   │
│   └── settings/
│       └── page.tsx
│
├── components/
│   ├── layout/
│   │   ├── AppShell.tsx
│   │   ├── Sidebar.tsx
│   │   ├── Header.tsx
│   │   └── StatusBar.tsx
│   │
│   ├── dashboard/
│   │   ├── EngineSelector.tsx
│   │   ├── HealthScoreCard.tsx
│   │   ├── EngineStatusCard.tsx
│   │   ├── TelemetryCards.tsx
│   │   ├── LiveTelemetryChart.tsx
│   │   ├── ExpectedActualChart.tsx
│   │   ├── DiagnosisPanel.tsx
│   │   ├── RulCard.tsx
│   │   ├── MissionRiskCard.tsx
│   │   ├── AlertDrawer.tsx
│   │   └── EngineTwin3D.tsx
│   │
│   ├── telemetry/
│   ├── diagnostics/
│   ├── rul/
│   ├── mission/
│   ├── replay/
│   └── ui/
│
├── hooks/
│   ├── useEngineStream.ts
│   ├── useTelemetry.ts
│   ├── useTwinState.ts
│   ├── useDiagnostics.ts
│   ├── useRul.ts
│   └── useMission.ts
│
├── stores/
│   ├── engineStore.ts
│   ├── telemetryStore.ts
│   ├── alertStore.ts
│   └── missionStore.ts
│
├── lib/
│   ├── api.ts
│   ├── websocket.ts
│   ├── types.ts
│   ├── constants.ts
│   ├── formatters.ts
│   └── utils.ts
│
├── public/
│   ├── icons/
│   ├── models/
│   └── assets/
│
├── tests/
│   ├── components/
│   └── e2e/
│
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
└── eslint.config.mjs
```

### Frontend responsibility

The frontend should:

- request backend state
- subscribe to WebSocket updates
- render telemetry
- render Digital Twin state
- display diagnostics
- display RUL
- run mission configuration requests
- control replay
- display connection status

The frontend must **not** calculate:

- engine health
- fault probability
- RUL
- mission risk
- physics residuals

Those belong to backend/domain services.

---

# 5. Backend Folder Structure

```text
apps/api/
│
├── app/
│   ├── main.py
│   │
│   ├── api/
│   │   ├── dependencies.py
│   │   │
│   │   ├── routes/
│   │   │   ├── health.py
│   │   │   ├── dashboard.py
│   │   │   ├── telemetry.py
│   │   │   ├── twin.py
│   │   │   ├── diagnostics.py
│   │   │   ├── rul.py
│   │   │   ├── mission.py
│   │   │   ├── replay.py
│   │   │   └── simulation.py
│   │   │
│   │   └── websocket.py
│   │
│   ├── core/
│   │   ├── config.py
│   │   ├── logging.py
│   │   ├── security.py
│   │   └── exceptions.py
│   │
│   ├── schemas/
│   │   ├── telemetry.py
│   │   ├── twin.py
│   │   ├── diagnostics.py
│   │   ├── rul.py
│   │   ├── mission.py
│   │   ├── replay.py
│   │   └── common.py
│   │
│   ├── services/
│   │   ├── telemetry_service.py
│   │   ├── twin_service.py
│   │   ├── diagnostic_service.py
│   │   ├── rul_service.py
│   │   ├── mission_service.py
│   │   ├── replay_service.py
│   │   └── websocket_service.py
│   │
│   ├── repositories/
│   │   ├── telemetry_repository.py
│   │   ├── twin_repository.py
│   │   ├── mission_repository.py
│   │   ├── diagnosis_repository.py
│   │   └── replay_repository.py
│   │
│   ├── db/
│   │   ├── session.py
│   │   ├── base.py
│   │   └── models/
│   │       ├── uav.py
│   │       ├── engine.py
│   │       ├── mission.py
│   │       ├── telemetry.py
│   │       ├── twin_state.py
│   │       ├── diagnosis.py
│   │       ├── rul.py
│   │       └── replay.py
│   │
│   └── tests/
│       ├── unit/
│       ├── api/
│       └── services/
│
├── requirements.txt
├── pyproject.toml
└── Dockerfile
```

---

# 6. Backend Dependency Direction

The dependency direction must remain:

```text
Routes
  ↓
Services
  ↓
Domain / Twin / ML
  ↓
Repositories
  ↓
Database
```

Not:

```text
Routes
  ↓
Database
  ↓
ML
  ↓
Frontend
```

Routes should remain thin.

Example:

```python
@router.get("/{engine_id}")
async def get_twin(engine_id: str):
    return await twin_service.get_state(engine_id)
```

Business logic belongs in `services/` or domain modules.

---

# 7. Embedded C Structure

The C layer represents low-level sensor/MCU acquisition.

```text
embedded/firmware-c/
│
├── include/
│   ├── sensors.h
│   ├── adc.h
│   ├── rpm.h
│   ├── can.h
│   ├── telemetry.h
│   └── config.h
│
├── src/
│   ├── main.c
│   ├── sensors.c
│   ├── adc.c
│   ├── rpm.c
│   ├── can.c
│   └── telemetry.c
│
├── tests/
│   └── test_telemetry.c
│
├── CMakeLists.txt
└── README.md
```

### C responsibilities

- sensor acquisition
- ADC/GPIO access
- interrupt handling
- RPM measurement
- low-level CAN communication
- hardware abstraction
- packet formation
- timestamp acquisition

The C layer should not contain:

- UI code
- database code
- Python logic
- ML models
- mission simulation

---

# 8. Edge C++ Structure

The C++ layer sits between hardware and the backend.

```text
embedded/edge-cpp/
│
├── include/
│   ├── can/
│   │   ├── CanInterface.hpp
│   │   └── CanFrame.hpp
│   │
│   ├── telemetry/
│   │   ├── TelemetryPacket.hpp
│   │   └── TelemetryDecoder.hpp
│   │
│   ├── processing/
│   │   ├── SensorFilter.hpp
│   │   ├── SensorFusion.hpp
│   │   └── VibrationFeatures.hpp
│   │
│   └── transport/
│       └── TelemetryPublisher.hpp
│
├── src/
│   ├── main.cpp
│   ├── can/
│   ├── telemetry/
│   ├── processing/
│   └── transport/
│
├── tests/
│   ├── test_can.cpp
│   ├── test_telemetry.cpp
│   └── test_features.cpp
│
├── CMakeLists.txt
└── README.md
```

### C++ responsibilities

- CAN/SocketCAN
- packet decoding
- filtering
- sensor fusion
- rolling statistics
- vibration feature extraction
- local buffering
- telemetry serialization
- edge inference where appropriate

---

# 9. Simulator Structure

The simulator must produce the **same canonical telemetry schema** used by real sensors.

```text
simulator/
│
├── engine/
│   ├── __init__.py
│   ├── engine_simulator.py
│   ├── telemetry_generator.py
│   ├── operating_state.py
│   └── scenarios.py
│
├── faults/
│   ├── injector.py
│   ├── cooling.py
│   ├── lubrication.py
│   ├── vibration.py
│   └── sensor_faults.py
│
├── missions/
│   ├── mission_runner.py
│   ├── profiles.py
│   └── scenarios.py
│
├── replay/
│   ├── loader.py
│   └── player.py
│
├── configs/
│   └── simulator.yaml
│
├── tests/
│
├── requirements.txt
└── README.md
```

### Critical simulator rule

```text
SIMULATOR
   ↓
Canonical Telemetry
   ↓
Same FastAPI ingestion path
```

Do not create a special "fake frontend path" for simulation.

---

# 10. Digital Twin Structure

The Digital Twin is a domain module rather than a frontend feature.

```text
twin/
│
├── __init__.py
│
├── state/
│   ├── twin_state.py
│   └── state_manager.py
│
├── physics/
│   ├── expected_state.py
│   ├── model_interface.py
│   └── residuals.py
│
├── health/
│   ├── health_index.py
│   └── health_state.py
│
├── degradation/
│   ├── degradation_state.py
│   └── trend.py
│
├── provenance/
│   └── versions.py
│
└── tests/
```

The twin should expose interfaces such as:

```text
update()
expected_state()
residuals()
health()
degradation()
state()
```

Do not hard-code unverified engine equations.

---

# 11. AI/ML Structure

```text
ml/
│
├── anomaly/
│   ├── detector.py
│   ├── features.py
│   ├── inference.py
│   └── train.py
│
├── diagnosis/
│   ├── classifier.py
│   ├── evidence.py
│   ├── inference.py
│   └── train.py
│
├── rul/
│   ├── predictor.py
│   ├── uncertainty.py
│   ├── inference.py
│   └── train.py
│
├── common/
│   ├── preprocessing.py
│   ├── metrics.py
│   └── model_registry.py
│
└── tests/
```

### MVP model strategy

Use a simple, explainable baseline first.

Possible prototype choices:

```text
Anomaly
→ Isolation Forest / residual-based score

Diagnosis
→ XGBoost / rule-assisted classifier

RUL
→ configurable baseline model with uncertainty interval
```

Do not claim model accuracy without evaluation.

---

# 12. Model Artifact Structure

Trained artifacts belong in:

```text
models/
│
├── anomaly/
│   ├── model.pkl
│   └── metadata.json
│
├── diagnosis/
│   ├── model.pkl
│   └── metadata.json
│
└── rul/
    ├── model.pkl
    └── metadata.json
```

For a larger production system, use object storage/model registry rather than committing large binary models to Git.

---

# 13. Database Structure

```text
database/
│
├── migrations/
│   ├── versions/
│   └── env.py
│
├── seeds/
│   ├── demo_uavs.sql
│   ├── demo_engines.sql
│   └── demo_missions.sql
│
└── scripts/
    ├── reset_db.py
    └── seed_db.py
```

Application ORM models remain in:

```text
apps/api/app/db/models/
```

This separates:

```text
Database migration history
```

from:

```text
Application database models
```

---

# 14. Configuration Structure

```text
configs/
│
├── engine_profiles/
│   └── aero_piston_demo.yaml
│
├── model_configs/
│   ├── anomaly.yaml
│   ├── diagnosis.yaml
│   └── rul.yaml
│
├── simulator/
│   ├── default.yaml
│   └── demo_fault.yaml
│
└── environments/
    ├── development.env.example
    └── production.env.example
```

### Important

Engineering parameters should be configuration-driven.

If an approved parameter is unavailable:

```text
ASSUMPTION_REQUIRED
```

Do not manufacture a value merely to make the application run.

---

# 15. Test Structure

```text
tests/
│
├── unit/
│   ├── physics/
│   ├── telemetry/
│   ├── anomaly/
│   ├── diagnosis/
│   └── rul/
│
├── integration/
│   ├── test_telemetry_pipeline.py
│   ├── test_twin_pipeline.py
│   └── test_websocket.py
│
├── e2e/
│   ├── dashboard.spec.ts
│   ├── fault-demo.spec.ts
│   └── mission.spec.ts
│
├── contracts/
│   ├── telemetry.schema.json
│   ├── twin.schema.json
│   └── websocket.schema.json
│
└── fixtures/
    ├── healthy.json
    └── fault_demo.json
```

---

# 16. Contract-First Wiring

The canonical contracts should be treated as shared interfaces.

```text
contracts/
│
├── telemetry.schema.json
├── twin.schema.json
├── diagnostics.schema.json
├── rul.schema.json
└── websocket.schema.json
```

For the MVP, these can live under:

```text
tests/contracts/
```

Later they can move to a dedicated package.

---

# 17. Documentation Structure

```text
docs/
│
├── architecture/
│   ├── system-architecture.md
│   ├── data-flow.md
│   └── frontend-backend-wiring.md
│
├── api/
│   └── api-reference.md
│
├── engineering/
│   ├── telemetry.md
│   ├── digital-twin.md
│   ├── fault-models.md
│   └── validation.md
│
├── demo/
│   ├── demo-script.md
│   └── judge-flow.md
│
└── decisions/
    ├── ADR-001-stack.md
    ├── ADR-002-canonical-telemetry.md
    └── ADR-003-websocket.md
```

---

# 18. Scripts

```text
scripts/
│
├── dev/
│   ├── start-api.sh
│   ├── start-web.sh
│   └── start-simulator.sh
│
├── db/
│   ├── migrate.sh
│   └── seed.sh
│
├── demo/
│   ├── start-demo.sh
│   └── inject-fault.sh
│
└── ci/
    └── test-all.sh
```

Windows-compatible alternatives can be added later.

---

# 19. Deployment Structure

```text
deployment/
│
├── docker/
│   ├── api.Dockerfile
│   ├── web.Dockerfile
│   └── simulator.Dockerfile
│
└── monitoring/
    ├── prometheus.yml
    └── grafana/
```

For SIH MVP:

```text
Docker Compose
```

is sufficient.

Do not spend hackathon time implementing Kubernetes unless specifically required.

---

# 20. Root Docker Compose

Expected service graph:

```text
docker-compose.yml

web
 │
 └── api
      ├── postgres
      └── redis

simulator
 │
 └── api
```

Conceptually:

```text
                    ┌────────────┐
                    │    WEB     │
                    │ :3000      │
                    └─────┬──────┘
                          │
                          │ HTTP / WS
                          ↓
                    ┌────────────┐
                    │    API     │
                    │ :8000      │
                    └──┬─────┬───┘
                       │     │
              ┌────────┘     └────────┐
              ↓                       ↓
        ┌──────────┐             ┌────────┐
        │ POSTGRES │             │ REDIS  │
        │ :5432    │             │ :6379  │
        └──────────┘             └────────┘
                                      ↑
                                      │
                                ┌─────┴────┐
                                │ SIMULATOR│
                                └──────────┘
```

---

# 21. Environment Files

Root:

```text
.env.example
```

Backend:

```text
apps/api/.env.example
```

Frontend:

```text
apps/web/.env.example
```

Never commit:

```text
.env
.env.local
production secrets
database passwords
API keys
private certificates
```

---

# 22. Git Ignore

Root `.gitignore` should include:

```text
.env
.env.*
!.env.example

__pycache__/
*.pyc

node_modules/
.next/

.venv/
venv/

.pytest_cache/
.mypy_cache/

coverage/
dist/
build/

*.log

data/generated/*
data/replay/*

*.pkl
*.joblib

.DS_Store
.vscode/
.idea/
```

If demo data is intentionally needed in Git, create a small `data/sample/` dataset and keep generated data ignored.

---

# 23. Temporary Special Markdown File

The temporary planning file is:

```text
VIGIL_SPECIAL_FILE_STRUCTURE.md
```

It should remain at the repository root during rapid implementation.

Its purpose is:

```text
Architecture reference
        ↓
Coding-agent reference
        ↓
Folder creation
        ↓
Module placement
        ↓
Implementation
```

It is **not** a runtime dependency.

No Python, TypeScript, C, C++, Docker, or database code should import it.

---

# 24. When to Remove the Temporary File

After the repository becomes stable:

```text
VIGIL_SPECIAL_FILE_STRUCTURE.md
```

can be removed or archived.

Before removing it, ensure the final structure is represented in:

```text
README.md
docs/architecture/system-architecture.md
docs/architecture/data-flow.md
```

Recommended final state:

```text
Root
├── README.md
├── docs/
│   └── architecture/
│       ├── system-architecture.md
│       └── data-flow.md
└── ...
```

---

# 25. 10-Hour Hackathon Priority Structure

Do **not** create every directory immediately.

Start with:

```text
VIGIL/
│
├── apps/
│   ├── web/
│   └── api/
│
├── simulator/
│
├── twin/
│
├── ml/
│
├── configs/
│
├── tests/
│
├── docs/
│
├── docker-compose.yml
├── .env.example
├── README.md
└── VIGIL_SPECIAL_FILE_STRUCTURE.md
```

Then create internal folders only when the corresponding module is implemented.

---

# 26. P0 Files — Create Immediately

### Backend

```text
apps/api/app/main.py
apps/api/app/schemas/telemetry.py
apps/api/app/schemas/twin.py
apps/api/app/api/routes/health.py
apps/api/app/api/routes/dashboard.py
apps/api/app/api/routes/telemetry.py
apps/api/app/api/websocket.py
apps/api/app/services/telemetry_service.py
apps/api/app/services/twin_service.py
```

### Frontend

```text
apps/web/app/dashboard/page.tsx
apps/web/components/dashboard/HealthScoreCard.tsx
apps/web/components/dashboard/TelemetryCards.tsx
apps/web/components/dashboard/LiveTelemetryChart.tsx
apps/web/components/dashboard/EngineTwin3D.tsx
apps/web/lib/api.ts
apps/web/lib/types.ts
apps/web/lib/websocket.ts
apps/web/hooks/useEngineStream.ts
apps/web/stores/engineStore.ts
```

### Simulator

```text
simulator/engine/engine_simulator.py
simulator/engine/telemetry_generator.py
simulator/faults/injector.py
```

---

# 27. P1 Files

After the live dashboard works:

```text
apps/api/app/api/routes/diagnostics.py
apps/api/app/api/routes/rul.py

apps/api/app/services/diagnostic_service.py
apps/api/app/services/rul_service.py

ml/anomaly/detector.py
ml/diagnosis/classifier.py
ml/rul/predictor.py

apps/web/app/diagnostics/page.tsx
apps/web/app/rul/page.tsx
apps/web/components/dashboard/DiagnosisPanel.tsx
apps/web/components/dashboard/RulCard.tsx
```

---

# 28. P2 Files

Only after P0/P1:

```text
apps/api/app/api/routes/mission.py
apps/api/app/api/routes/replay.py

apps/api/app/services/mission_service.py
apps/api/app/services/replay_service.py

apps/web/app/mission/page.tsx
apps/web/app/replay/page.tsx

simulator/missions/
simulator/replay/
```

---

# 29. Full Runtime Data Flow

```text
┌─────────────────────────┐
│ REAL SENSORS             │
│ RPM / CHT / EGT / etc.  │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│ C FIRMWARE              │
│ Acquisition             │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│ C++ EDGE                │
│ Filter / Features / CAN │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│ CAN / SocketCAN          │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│ FASTAPI INGESTION        │
│ Validate / Normalize     │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│ DIGITAL TWIN             │
│ Expected State           │
│ Residuals                │
└────────────┬────────────┘
             │
       ┌─────┴─────┐
       ↓           ↓
┌────────────┐ ┌────────────┐
│ ANOMALY    │ │ DIAGNOSIS  │
│ DETECTION  │ │            │
└──────┬─────┘ └─────┬──────┘
       └──────┬──────┘
              ↓
       ┌─────────────┐
       │ HEALTH      │
       └──────┬──────┘
              ↓
       ┌─────────────┐
       │ RUL         │
       └──────┬──────┘
              ↓
       ┌─────────────┐
       │ MISSION RISK│
       └──────┬──────┘
              ↓
      REST + WEBSOCKET
              ↓
       ┌─────────────┐
       │ NEXT.JS GCS │
       └─────────────┘
```

Simulation uses the same path:

```text
SIMULATOR
    ↓
Canonical Telemetry
    ↓
FastAPI
    ↓
Digital Twin
    ↓
AI/ML
    ↓
WebSocket
    ↓
Frontend
```

This is essential because the SIH demonstration can switch between simulated and future real telemetry without changing the frontend architecture.

---

# 30. File Ownership Rules

| Directory | Owner / Responsibility |
|---|---|
| `apps/web` | Frontend/GCS |
| `apps/api` | Backend/API |
| `embedded/firmware-c` | Embedded C |
| `embedded/edge-cpp` | Edge C++ |
| `simulator` | Synthetic engine + mission simulation |
| `twin` | Digital Twin |
| `ml` | AI/ML |
| `database` | Migration/seed tooling |
| `configs` | Configuration |
| `models` | Model artifacts |
| `tests` | Cross-module tests |
| `docs` | Permanent documentation |
| `deployment` | Deployment/operations |
| `scripts` | Developer automation |

---

# 31. Rules for Antigravity / Coding Agents

Paste this structure into the coding agent's context.

```text
PROJECT STRUCTURE RULES

1. Inspect the repository before creating files.

2. Follow VIGIL_SPECIAL_FILE_STRUCTURE.md as the temporary
   structural authority during hackathon implementation.

3. Do not create random root-level folders.

4. Frontend code belongs in apps/web.

5. Backend code belongs in apps/api.

6. Embedded C belongs in embedded/firmware-c.

7. Edge C++ belongs in embedded/edge-cpp.

8. Simulator code belongs in simulator.

9. Digital Twin domain logic belongs in twin.

10. AI/ML code belongs in ml.

11. Database migration scripts belong in database/migrations.

12. Configuration belongs in configs.

13. Tests belong either next to the module or under tests/
    according to the existing convention.

14. Documentation belongs in docs.

15. Deployment files belong in deployment.

16. Do not place secrets in the repository.

17. Do not place generated datasets in Git.

18. Do not place model binaries in Git unless explicitly required.

19. Do not import frontend modules into backend code.

20. Do not import backend modules into frontend code.

21. Frontend communicates with backend through REST/WebSocket.

22. Simulator communicates through the canonical telemetry interface.

23. Real sensor telemetry must eventually use the same canonical
    telemetry interface as simulation.

24. Do not duplicate business logic in the frontend.

25. Do not invent missing engineering parameters.

26. If an engineering parameter is required but unavailable,
    use ASSUMPTION_REQUIRED and flag it for human review.

27. Do not delete VIGIL_SPECIAL_FILE_STRUCTURE.md until the
    repository structure has been finalized and documented elsewhere.

28. After implementing a module:
    - run tests
    - run type checking/linting where applicable
    - verify imports
    - verify API contracts
    - report changed files

29. Keep the first implementation vertical and working.

30. Prefer a working P0 implementation over creating empty
    directories for P1/P2 features.
```

---

# 32. Antigravity — Structure Creation Prompt

Use this as the first coding-agent prompt:

```text
You are the Principal Software Engineer for VIGIL, SIH26054.

Your immediate task is to establish the repository structure
defined in VIGIL_SPECIAL_FILE_STRUCTURE.md.

IMPORTANT:
This file is temporary and is the structural authority during
the hackathon. Do not delete it.

First:
1. Inspect the existing repository.
2. Identify existing frontend/backend files.
3. Do not overwrite working code without checking it.
4. Reconcile the existing repository with the structure.
5. Create only directories needed for the first working vertical slice.

The first vertical slice is:

SIMULATOR
→ canonical telemetry
→ FastAPI
→ WebSocket
→ Next.js dashboard.

Create the following P0 structure:

apps/
  web/
  api/

simulator/
  engine/
  faults/

twin/

ml/

configs/

tests/
  contracts/
  integration/

docs/
  architecture/

embedded/
  firmware-c/
  edge-cpp/

database/
  migrations/
  seeds/

deployment/
  docker/

scripts/

Keep:
VIGIL_SPECIAL_FILE_STRUCTURE.md

Then implement only the minimum files required for:
1. FastAPI startup
2. health endpoint
3. telemetry schema
4. simulator telemetry
5. WebSocket endpoint
6. frontend API client
7. frontend WebSocket hook
8. dashboard live telemetry
9. basic Digital Twin state placeholder/interface

Do not implement fake physics.
Do not invent engine thresholds.
Do not invent scientific constants.
Do not create unnecessary placeholder files.

After completion:
- run backend tests
- run frontend lint/typecheck
- verify both applications start
- report exact files created
- report exact files modified
- report any blockers
```

---

# 33. Final Recommended Repository

After the MVP is implemented, the target should look approximately like:

```text
VIGIL/
│
├── apps/
│   ├── web/
│   │   ├── app/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── stores/
│   │   ├── lib/
│   │   ├── public/
│   │   └── package.json
│   │
│   └── api/
│       ├── app/
│       │   ├── api/
│       │   ├── core/
│       │   ├── schemas/
│       │   ├── services/
│       │   ├── repositories/
│       │   ├── db/
│       │   └── tests/
│       ├── requirements.txt
│       └── Dockerfile
│
├── embedded/
│   ├── firmware-c/
│   └── edge-cpp/
│
├── simulator/
│
├── twin/
│
├── ml/
│
├── models/
│
├── database/
│
├── configs/
│
├── data/
│
├── tests/
│
├── docs/
│
├── scripts/
│
├── deployment/
│
├── .github/
│
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Makefile
├── README.md
└── VIGIL_SPECIAL_FILE_STRUCTURE.md
```

---

# 34. Most Important Rule for the Hackathon

Do not confuse **folder completeness** with **product completeness**.

A repository containing 150 empty files is not more complete than one containing 25 working files.

Your implementation priority is:

```text
                    P0
                     │
                     ▼
          Simulator Telemetry
                     │
                     ▼
              FastAPI Backend
                     │
                     ▼
                WebSocket
                     │
                     ▼
             Next.js Dashboard
                     │
                     ▼
              Digital Twin
                     │
                     ▼
           Fault Injection Demo
                     │
                     ▼
            Anomaly + Diagnosis
                     │
                     ▼
                   RUL
                     │
                     ▼
              Mission Risk
                     │
                     ▼
                 Replay
```

**Build this vertical path first.**

Everything else is secondary for the SIH demonstration.
