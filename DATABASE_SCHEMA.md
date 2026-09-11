# VIGIL — Database Schema Specification

> **Product:** VIGIL — Intelligent UAV Engine Health & Prognostics  
> **SIH Problem Statement:** SIH26054  
> **Version:** 1.0  
> **Status:** MVP Database Source of Truth  
> **Related Documents:** `PRD.md`, `TRD.md`, `UI_UX_DESIGN.md`

---

# 1. Purpose

This document defines the persistent data model for VIGIL.

The database must support:

- UAV and engine identity;
- engine configuration;
- missions;
- telemetry;
- telemetry quality;
- Digital Twin state;
- expected-vs-actual residuals;
- anomaly events;
- fault diagnosis;
- health/degradation history;
- RUL estimates;
- mission simulations;
- mission replay;
- fault injection;
- model/simulator provenance;
- reports;
- auditability.

The database must preserve the distinction between:

```text
LIVE
SIMULATED
HISTORICAL / REPLAY
ESTIMATED
INVALID / STALE
```

---

# 2. Database Technology

## Primary Database

**PostgreSQL**

## Time-Series Extension

**TimescaleDB**

Use TimescaleDB for high-volume telemetry while retaining PostgreSQL relational capabilities.

## Cache / Live State

**Redis**

Redis is not the source of truth for historical data.

---

# 3. Design Principles

## DB-01 — Relational Core

VIGIL data has strong relationships:

```text
UAV
 ↓
Engine
 ↓
Mission
 ↓
Telemetry
 ↓
Analysis
 ↓
Diagnosis
 ↓
Health / RUL
```

PostgreSQL is therefore the authoritative persistent datastore.

## DB-02 — Time-Series First

Telemetry and derived state are timestamped.

## DB-03 — Traceability

An analytical result should be traceable to:

- telemetry;
- mission;
- engine;
- model version;
- simulator version;
- scenario/configuration.

## DB-04 — No Hardcoded Engineering Truth

Database seed/configuration values must not invent physical constants or certified limits.

## DB-05 — Reproducibility

Simulation and fault-injection records should preserve the configuration and random seed required to reproduce a scenario.

## DB-06 — Soft Deletion Where Appropriate

Historical analytical records should generally not be physically deleted merely because a related UI object is removed.

---

# 4. High-Level Entity Relationship

```text
┌─────────────┐
│     UAV     │
└──────┬──────┘
       │ 1:N
       ▼
┌─────────────┐
│   ENGINE    │
└──────┬──────┘
       │ 1:N
       ▼
┌─────────────┐
│   MISSION   │
└──────┬──────┘
       │
       ├───────────────┐
       │               │
       ▼               ▼
┌─────────────┐  ┌──────────────┐
│  TELEMETRY  │  │   SCENARIO   │
└──────┬──────┘  └──────┬───────┘
       │                 │
       ▼                 ▼
┌─────────────┐   ┌──────────────┐
│  ANALYSIS   │   │    FAULT     │
└──────┬──────┘   │   INJECTION  │
       │          └──────────────┘
       ├──────────────┬───────────────┐
       ▼              ▼               ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│   ANOMALY   │ │  DIAGNOSIS  │ │   HEALTH    │
└─────────────┘ └──────┬──────┘ └─────────────┘
                       │
                       ▼
                ┌─────────────┐
                │     RUL     │
                └─────────────┘

MISSION
   │
   ├── SIMULATION
   │
   ├── REPLAY
   │
   └── REPORT
```

---

# 5. Core Tables

The MVP database should contain the following logical tables:

```text
uavs
engines
engine_profiles
missions
mission_scenarios
telemetry
telemetry_quality
twin_states
residuals
features
anomaly_events
diagnoses
diagnosis_evidence
health_states
degradation_states
rul_predictions
fault_injections
simulation_runs
replay_sessions
model_versions
simulator_versions
reports
audit_events
```

---

# 6. `uavs`

Stores UAV identity and metadata.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| uav_code | VARCHAR | UNIQUE, NOT NULL |
| name | VARCHAR | NULL |
| platform | VARCHAR | NULL |
| status | VARCHAR | NOT NULL |
| metadata | JSONB | NULL |
| created_at | TIMESTAMPTZ | NOT NULL |
| updated_at | TIMESTAMPTZ | NOT NULL |

Example:

```json
{
  "uav_code": "UAV-001",
  "platform": "MALE-UAV"
}
```

---

# 7. `engine_profiles`

Stores the logical configuration/profile of an engine model.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| profile_code | VARCHAR | UNIQUE, NOT NULL |
| name | VARCHAR | NOT NULL |
| manufacturer | VARCHAR | NULL |
| description | TEXT | NULL |
| configuration | JSONB | NOT NULL |
| version | VARCHAR | NOT NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

`configuration` may contain validated model configuration.

It must not contain fabricated engineering constants.

---

# 8. `engines`

Stores individual engine identity.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| uav_id | UUID | FK → uavs.id |
| profile_id | UUID | FK → engine_profiles.id |
| engine_code | VARCHAR | UNIQUE, NOT NULL |
| serial_number | VARCHAR | NULL |
| installation_date | TIMESTAMPTZ | NULL |
| engine_age_hours | DOUBLE PRECISION | NULL |
| status | VARCHAR | NOT NULL |
| metadata | JSONB | NULL |
| created_at | TIMESTAMPTZ | NOT NULL |
| updated_at | TIMESTAMPTZ | NOT NULL |

---

# 9. `missions`

Represents a real, simulated, or replayable mission.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| mission_code | VARCHAR | UNIQUE, NOT NULL |
| uav_id | UUID | FK |
| engine_id | UUID | FK |
| mission_type | VARCHAR | NOT NULL |
| source_type | VARCHAR | NOT NULL |
| status | VARCHAR | NOT NULL |
| start_time | TIMESTAMPTZ | NULL |
| end_time | TIMESTAMPTZ | NULL |
| configuration | JSONB | NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

Suggested `source_type`:

```text
LIVE
SIMULATION
HISTORICAL
REPLAY
```

---

# 10. `mission_scenarios`

Stores mission simulator configurations.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| mission_id | UUID | FK |
| scenario_name | VARCHAR | NOT NULL |
| configuration | JSONB | NOT NULL |
| seed | BIGINT | NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

Configuration may include:

```json
{
  "altitude": "...",
  "ambient_temperature": "...",
  "throttle": "...",
  "duration": "...",
  "load": "..."
}
```

Actual engineering units must be defined by the validated telemetry/engineering contract.

---

# 11. `telemetry`

This is the primary high-volume time-series table.

Recommended implementation: TimescaleDB hypertable.

| Column | Type | Constraints |
|---|---|---|
| timestamp | TIMESTAMPTZ | NOT NULL |
| mission_id | UUID | FK |
| uav_id | UUID | FK |
| engine_id | UUID | FK |
| source_type | VARCHAR | NOT NULL |
| sequence_number | BIGINT | NULL |
| rpm | DOUBLE PRECISION | NULL |
| cht | DOUBLE PRECISION | NULL |
| egt | DOUBLE PRECISION | NULL |
| oil_pressure | DOUBLE PRECISION | NULL |
| oil_temperature | DOUBLE PRECISION | NULL |
| fuel_flow | DOUBLE PRECISION | NULL |
| vibration | DOUBLE PRECISION | NULL |
| battery_voltage | DOUBLE PRECISION | NULL |
| alternator_current | DOUBLE PRECISION | NULL |
| injection_timing | DOUBLE PRECISION | NULL |
| altitude | DOUBLE PRECISION | NULL |
| ambient_temperature | DOUBLE PRECISION | NULL |
| ambient_pressure | DOUBLE PRECISION | NULL |
| throttle | DOUBLE PRECISION | NULL |
| data_quality | VARCHAR | NOT NULL |
| metadata | JSONB | NULL |

Recommended primary/index strategy:

```text
(time, engine_id)
(time, mission_id)
```

Exact indexing must be benchmarked against actual workload.

---

# 12. `telemetry_quality`

Stores detailed quality information for telemetry samples or grouped windows.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| telemetry_timestamp | TIMESTAMPTZ | NOT NULL |
| mission_id | UUID | FK |
| engine_id | UUID | FK |
| overall_status | VARCHAR | NOT NULL |
| missing_fields | JSONB | NULL |
| invalid_fields | JSONB | NULL |
| stale_fields | JSONB | NULL |
| packet_loss | BOOLEAN | NOT NULL |
| validation_messages | JSONB | NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

---

# 13. `twin_states`

Stores snapshots of the Digital Twin state.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| timestamp | TIMESTAMPTZ | NOT NULL |
| mission_id | UUID | FK |
| engine_id | UUID | FK |
| health_score | DOUBLE PRECISION | NULL |
| anomaly_score | DOUBLE PRECISION | NULL |
| mission_risk | VARCHAR | NULL |
| degradation_state | JSONB | NULL |
| state | JSONB | NOT NULL |
| model_version_id | UUID | FK |
| simulator_version_id | UUID | FK |
| created_at | TIMESTAMPTZ | NOT NULL |

`state` may contain the current synchronized engine representation.

---

# 14. `residuals`

Stores observed-vs-expected values.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| timestamp | TIMESTAMPTZ | NOT NULL |
| mission_id | UUID | FK |
| engine_id | UUID | FK |
| parameter_name | VARCHAR | NOT NULL |
| observed_value | DOUBLE PRECISION | NULL |
| expected_value | DOUBLE PRECISION | NULL |
| residual_value | DOUBLE PRECISION | NULL |
| normalized_residual | DOUBLE PRECISION | NULL |
| model_version_id | UUID | FK |
| created_at | TIMESTAMPTZ | NOT NULL |

Formula:

```text
residual = observed_value - expected_value
```

---

# 15. `features`

Stores derived analytical features.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| timestamp | TIMESTAMPTZ | NOT NULL |
| mission_id | UUID | FK |
| engine_id | UUID | FK |
| feature_set_version | VARCHAR | NOT NULL |
| features | JSONB | NOT NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

Example:

```json
{
  "rpm_instability": "...",
  "vibration_rms": "...",
  "egt_residual_trend": "..."
}
```

---

# 16. `anomaly_events`

Represents detected abnormal behavior.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| mission_id | UUID | FK |
| engine_id | UUID | FK |
| started_at | TIMESTAMPTZ | NOT NULL |
| ended_at | TIMESTAMPTZ | NULL |
| anomaly_score | DOUBLE PRECISION | NOT NULL |
| severity | VARCHAR | NOT NULL |
| status | VARCHAR | NOT NULL |
| affected_parameters | JSONB | NULL |
| contributing_features | JSONB | NULL |
| model_version_id | UUID | FK |
| created_at | TIMESTAMPTZ | NOT NULL |

---

# 17. `diagnoses`

Stores probable fault diagnoses.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| anomaly_event_id | UUID | FK |
| mission_id | UUID | FK |
| engine_id | UUID | FK |
| primary_fault | VARCHAR | NOT NULL |
| confidence | DOUBLE PRECISION | NOT NULL |
| severity | VARCHAR | NOT NULL |
| alternatives | JSONB | NULL |
| isolation_type | VARCHAR | NULL |
| model_version_id | UUID | FK |
| created_at | TIMESTAMPTZ | NOT NULL |

Example:

```json
{
  "injector_degradation": 0.84,
  "cooling_degradation": 0.21,
  "lubrication_issue": 0.13,
  "sensor_drift": 0.08
}
```

Probabilities must come from the diagnostic model.

---

# 18. `diagnosis_evidence`

Stores evidence supporting a diagnosis.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| diagnosis_id | UUID | FK |
| parameter_name | VARCHAR | NOT NULL |
| evidence_type | VARCHAR | NOT NULL |
| description | TEXT | NOT NULL |
| value | DOUBLE PRECISION | NULL |
| expected_value | DOUBLE PRECISION | NULL |
| residual | DOUBLE PRECISION | NULL |
| contribution | DOUBLE PRECISION | NULL |
| timestamp | TIMESTAMPTZ | NOT NULL |

This table prevents the UI from using hardcoded explanations.

---

# 19. `health_states`

Stores health estimates over time.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| timestamp | TIMESTAMPTZ | NOT NULL |
| mission_id | UUID | FK |
| engine_id | UUID | FK |
| overall_health | DOUBLE PRECISION | NOT NULL |
| subsystem_health | JSONB | NOT NULL |
| trend | VARCHAR | NULL |
| degradation_drivers | JSONB | NULL |
| model_version_id | UUID | FK |
| created_at | TIMESTAMPTZ | NOT NULL |

Prototype UI conventions:

```text
90–100 GOOD
75–89 DEGRADED
50–74 WARNING
0–49 CRITICAL
```

These are not certified safety limits.

---

# 20. `degradation_states`

Stores degradation estimates independently from display health.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| timestamp | TIMESTAMPTZ | NOT NULL |
| engine_id | UUID | FK |
| mission_id | UUID | FK |
| subsystem | VARCHAR | NOT NULL |
| degradation_value | DOUBLE PRECISION | NOT NULL |
| degradation_rate | DOUBLE PRECISION | NULL |
| confidence | DOUBLE PRECISION | NULL |
| model_version_id | UUID | FK |
| created_at | TIMESTAMPTZ | NOT NULL |

---

# 21. `rul_predictions`

Stores RUL predictions.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| timestamp | TIMESTAMPTZ | NOT NULL |
| mission_id | UUID | FK |
| engine_id | UUID | FK |
| rul_value | DOUBLE PRECISION | NOT NULL |
| lower_bound | DOUBLE PRECISION | NULL |
| upper_bound | DOUBLE PRECISION | NULL |
| confidence | DOUBLE PRECISION | NULL |
| eol_criterion | TEXT | NOT NULL |
| model_version_id | UUID | FK |
| created_at | TIMESTAMPTZ | NOT NULL |

RUL must always be associated with an explicit prototype end-of-life criterion.

---

# 22. `fault_injections`

Stores injected faults in simulated scenarios.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| simulation_run_id | UUID | FK |
| fault_type | VARCHAR | NOT NULL |
| subsystem | VARCHAR | NULL |
| start_time | TIMESTAMPTZ | NOT NULL |
| end_time | TIMESTAMPTZ | NULL |
| severity | DOUBLE PRECISION | NULL |
| configuration | JSONB | NOT NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

The fault injection record is ground truth for synthetic validation.

---

# 23. `simulation_runs`

Stores each simulator execution.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| scenario_id | UUID | FK |
| simulator_version_id | UUID | FK |
| seed | BIGINT | NULL |
| status | VARCHAR | NOT NULL |
| started_at | TIMESTAMPTZ | NOT NULL |
| completed_at | TIMESTAMPTZ | NULL |
| configuration | JSONB | NOT NULL |
| output_summary | JSONB | NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

---

# 24. `replay_sessions`

Stores user replay sessions.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| mission_id | UUID | FK |
| started_at | TIMESTAMPTZ | NOT NULL |
| ended_at | TIMESTAMPTZ | NULL |
| playback_position | TIMESTAMPTZ | NULL |
| playback_speed | DOUBLE PRECISION | NULL |
| state | JSONB | NULL |

Replay state must not overwrite live Digital Twin state.

---

# 25. `model_versions`

Stores AI/ML and analytical model metadata.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| model_name | VARCHAR | NOT NULL |
| model_type | VARCHAR | NOT NULL |
| version | VARCHAR | NOT NULL |
| artifact_uri | TEXT | NULL |
| feature_set_version | VARCHAR | NULL |
| training_dataset_version | VARCHAR | NULL |
| metrics | JSONB | NULL |
| configuration | JSONB | NULL |
| status | VARCHAR | NOT NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

Unique logical key:

```text
(model_name, version)
```

---

# 26. `simulator_versions`

Stores simulator provenance.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| simulator_name | VARCHAR | NOT NULL |
| version | VARCHAR | NOT NULL |
| configuration_schema_version | VARCHAR | NULL |
| description | TEXT | NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

---

# 27. `reports`

Stores generated mission/diagnostic reports.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| mission_id | UUID | FK |
| report_type | VARCHAR | NOT NULL |
| title | VARCHAR | NOT NULL |
| content | JSONB | NOT NULL |
| file_uri | TEXT | NULL |
| generated_at | TIMESTAMPTZ | NOT NULL |

---

# 28. `audit_events`

Stores important system changes.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| actor_id | UUID | NULL |
| event_type | VARCHAR | NOT NULL |
| entity_type | VARCHAR | NOT NULL |
| entity_id | UUID | NULL |
| payload | JSONB | NULL |
| created_at | TIMESTAMPTZ | NOT NULL |

Use for:

- configuration changes;
- model changes;
- simulation changes;
- administrative operations.

---

# 29. Relationships

Core relationships:

```text
uavs
  1 ─── N engines

engine_profiles
  1 ─── N engines

engines
  1 ─── N missions

missions
  1 ─── N telemetry
  1 ─── N twin_states
  1 ─── N residuals
  1 ─── N anomaly_events
  1 ─── N health_states
  1 ─── N rul_predictions
  1 ─── N mission_scenarios
  1 ─── N replay_sessions
  1 ─── N reports

anomaly_events
  1 ─── N diagnoses

diagnoses
  1 ─── N diagnosis_evidence

mission_scenarios
  1 ─── N simulation_runs

simulation_runs
  1 ─── N fault_injections

model_versions
  1 ─── N analytical results

simulator_versions
  1 ─── N simulation_runs
```

---

# 30. Recommended Enumerations

Prefer PostgreSQL enums or validated lookup values for stable states.

## Mission Type

```text
ENDURANCE
HIGH_ALTITUDE
TEST
TRAINING
CUSTOM
```

## Source Type

```text
LIVE
SIMULATION
HISTORICAL
REPLAY
```

## Data Quality

```text
VALID
INVALID
STALE
MISSING
ESTIMATED
SIMULATED
```

## Health Status

```text
GOOD
DEGRADED
WARNING
CRITICAL
UNKNOWN
```

## Anomaly Severity

```text
INFO
WARNING
CRITICAL
```

## Diagnosis Isolation

```text
ENGINE
SENSOR
DATA_QUALITY
UNKNOWN
```

---

# 31. JSONB Usage

JSONB should be used for flexible metadata/configuration, not as a replacement for core relational columns.

Good:

```text
mission.configuration
engine.metadata
diagnosis.alternatives
twin_states.state
model_versions.metrics
```

Avoid:

```text
telemetry.rpm stored only inside JSONB
```

Frequently queried telemetry fields should remain typed columns.

---

# 32. Indexing Strategy

Important indexes:

```text
telemetry:
  (engine_id, timestamp DESC)
  (mission_id, timestamp DESC)

twin_states:
  (engine_id, timestamp DESC)
  (mission_id, timestamp DESC)

residuals:
  (engine_id, timestamp DESC)
  (mission_id, parameter_name, timestamp DESC)

anomaly_events:
  (engine_id, started_at DESC)

health_states:
  (engine_id, timestamp DESC)

rul_predictions:
  (engine_id, timestamp DESC)

diagnoses:
  (engine_id, created_at DESC)
```

Actual indexes must be validated using query plans and measured workload.

---

# 33. Time-Series Retention

For the SIH prototype, historical data should normally be retained.

If retention policies are introduced later, they must not delete records required for:

- validation;
- replay;
- audit;
- model evaluation.

---

# 34. Data Integrity

Required constraints:

- foreign keys;
- unique identifiers;
- non-null timestamps;
- valid relationships;
- consistent mission/engine ownership;
- controlled enum values;
- model-version references;
- simulator-version references where applicable.

A telemetry record for an unknown engine should not silently enter the analytical pipeline.

---

# 35. Database Migration Strategy

Use a migration framework such as:

**Alembic**

Migration flow:

```text
Schema Change
    ↓
Migration
    ↓
Apply to Development
    ↓
Run Tests
    ↓
Review
    ↓
Apply to Deployment
```

Never manually modify production schema without a tracked migration.

---

# 36. Seed Data

Seed data should contain only:

- demo UAV;
- demo engine;
- demo missions;
- validated scenario metadata;
- demo model metadata;
- simulator metadata.

Example:

```text
UAV-001
AERO-PX-01
MISSION-DEMO-001
```

Synthetic telemetry should be generated by the simulator, not manually inserted as fake production telemetry.

---

# 37. Data Provenance Chain

Every major analytical result should be traceable:

```text
Telemetry
   ↓
Mission
   ↓
Scenario
   ↓
Simulator Version
   ↓
Physics Model
   ↓
Features
   ↓
ML Model Version
   ↓
Anomaly / Diagnosis / Health / RUL
```

This chain is essential for explainability and reproducibility.

---

# 38. Example Digital Twin State

Logical API/database representation:

```json
{
  "health": 92,
  "anomaly_score": 0.18,
  "faults": [],
  "rul_hours": 412,
  "rul_lower": 380,
  "rul_upper": 450,
  "mission_risk": "LOW",
  "model_version": "v0.3.0",
  "simulator_version": "sim-0.5.0"
}
```

These values are example structure, not real engine measurements.

---

# 39. MVP Database Priority

Given the SIH time constraint, implement in this order:

## P0 — Required

```text
uavs
engine_profiles
engines
missions
telemetry
twin_states
anomaly_events
diagnoses
diagnosis_evidence
health_states
rul_predictions
mission_scenarios
simulation_runs
fault_injections
model_versions
simulator_versions
```

## P1 — Recommended

```text
residuals
features
telemetry_quality
replay_sessions
reports
```

## P2 — Later

```text
audit_events
advanced fleet tables
maintenance records
user/role tables
edge-device registry
```

---

# 40. 10-Hour SIH Simplification

Do not block the demo on a perfect database.

For the first working prototype:

```text
PostgreSQL
    +
TimescaleDB
    +
Redis
```

Use a minimal schema that supports:

```text
Telemetry
   ↓
Digital Twin
   ↓
Analysis
   ↓
Health
   ↓
Diagnosis
   ↓
RUL
   ↓
Mission
```

Everything else can be expanded after the core loop works.

---

# 41. Database Definition of Done

- [ ] PostgreSQL runs locally.
- [ ] TimescaleDB is enabled for telemetry.
- [ ] Redis is available for live state.
- [ ] Migrations are reproducible.
- [ ] UAV/engine/mission relationships work.
- [ ] Telemetry ingestion works.
- [ ] Telemetry can be queried by time range.
- [ ] Digital Twin snapshots persist.
- [ ] Residuals can be stored.
- [ ] Anomaly events persist.
- [ ] Diagnoses persist with evidence.
- [ ] Health history persists.
- [ ] RUL predictions persist with uncertainty.
- [ ] Simulation runs persist.
- [ ] Fault injections persist as synthetic ground truth.
- [ ] Model versions are traceable.
- [ ] Simulator versions are traceable.
- [ ] Replay can reconstruct historical mission state.
- [ ] No simulated data is mislabeled as live.
- [ ] No unsupported engineering limits are stored as facts.

---

# 42. Final Database Principle

VIGIL's database is not merely a place to store dashboard values.

It must preserve the complete chain:

```text
WHAT HAPPENED
      ↓
WHAT THE ENGINE WAS EXPECTED TO DO
      ↓
HOW IT DEVIATED
      ↓
WHAT THE AI DETECTED
      ↓
WHY IT REACHED A DIAGNOSIS
      ↓
HOW HEALTH CHANGED
      ↓
HOW RUL CHANGED
      ↓
WHAT MISSION CONDITIONS WERE INVOLVED
```

This makes the VIGIL Digital Twin **traceable, replayable, explainable, and reproducible**.
