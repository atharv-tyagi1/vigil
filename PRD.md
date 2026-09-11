# VIGIL — Product Requirements Document (PRD)

> **Product:** VIGIL — Intelligent UAV Engine Health & Prognostics  
> **SIH Problem Statement:** SIH26054  
> **Organization:** DRDO  
> **Theme:** Robotics & Drones  
> **Document Status:** Product Source of Truth  
> **Version:** 1.0  
> **Prototype Type:** Software demonstrator / decision-support system

---

## 1. Document Purpose

This document defines the product requirements for **VIGIL**, an AI-enabled real-time Digital Twin system for monitoring, diagnosing, predicting, and evaluating the mission impact of degradation in aero-piston engines used in MALE UAVs.

This PRD defines:

- what VIGIL is;
- the problem it solves;
- who uses it;
- what the product must do;
- how users interact with it;
- what constitutes the MVP;
- how success will be measured;
- what is explicitly outside the product scope.

Low-level implementation details such as programming languages, database implementation, API internals, model architecture, deployment infrastructure, and exact engineering equations belong in the **TRD** and related technical specifications.

---

# 2. Product Overview

## 2.1 Product Name

**VIGIL**

### Full Name

**Intelligent UAV Engine Health & Prognostics**

## 2.2 One-Line Definition

> **VIGIL is a physics-informed AI Digital Twin that continuously estimates the state of a MALE-UAV aero-piston engine, detects abnormal behavior, diagnoses probable faults, predicts remaining useful life, and evaluates mission risk.**

## 2.3 Product Story

VIGIL follows a continuous intelligence pipeline:

```text
MONITOR
   ↓
UNDERSTAND
   ↓
DETECT
   ↓
DIAGNOSE
   ↓
PREDICT
   ↓
SIMULATE
   ↓
DECIDE
```

The system is intended to move engine monitoring from:

> **"A parameter crossed a threshold."**

toward:

> **"The engine is behaving differently from its expected state; here is the probable cause, supporting evidence, estimated degradation, remaining useful life, and potential mission impact."**

---

# 3. Problem Statement

MALE UAV missions can require long-duration operation under changing environmental and operating conditions.

Engine health can change because of:

- component degradation;
- combustion abnormalities;
- injector degradation;
- cooling degradation;
- lubrication problems;
- mechanical abnormalities;
- sensor drift/failure;
- electrical degradation;
- changing environmental conditions.

A conventional monitoring interface may display raw telemetry and threshold alarms, but this alone does not provide sufficient context for predictive decision support.

VIGIL addresses this gap by combining:

1. real-time telemetry;
2. a physics-informed expected-state model;
3. residual analysis;
4. AI/ML anomaly detection;
5. probabilistic fault diagnosis;
6. health estimation;
7. degradation/RUL estimation;
8. mission simulation;
9. historical mission replay.

---

# 4. Product Vision

VIGIL should become a unified engine intelligence layer that allows an operator, maintenance engineer, or mission planner to understand not only the **current engine state**, but also:

- what changed;
- whether the change is meaningful;
- whether the problem may be the sensor or the engine;
- what fault is most likely;
- how the engine is degrading;
- how much useful life may remain;
- how a planned mission could affect the engine;
- what happened during a previous mission.

The product should provide **decision support**, not unsupported certainty.

---

# 5. Product Goals

## 5.1 Primary Goals

### G1 — Real-Time Health Monitoring

Continuously monitor important engine and electrical parameters.

### G2 — Digital Twin State Estimation

Maintain a synchronized virtual representation of the engine state.

### G3 — Physics-Informed Detection

Compare observed engine behavior against an expected healthy state under the current operating conditions.

### G4 — Early Anomaly Detection

Detect abnormal behavior before relying only on hard threshold violations.

### G5 — Explainable Fault Diagnosis

Identify probable fault classes and expose the evidence supporting the diagnosis.

### G6 — Degradation & RUL

Track degradation trends and provide a prototype Remaining Useful Life estimate with visible uncertainty.

### G7 — Mission-Aware Intelligence

Allow users to evaluate how altitude, temperature, throttle, duration, load, and degradation affect the projected engine state.

### G8 — Historical Replay

Allow users to replay a mission and understand when abnormal behavior first appeared and how the engine state evolved.

### G9 — Extensible Architecture

Allow simulated telemetry to be replaced or supplemented by real telemetry adapters such as ECU/CAN/SocketCAN without redesigning the product.

---

# 6. Product Non-Goals

VIGIL is **not** intended to be:

- a certified aircraft safety system;
- an autonomous flight-control system;
- an autopilot;
- a replacement for manufacturer maintenance manuals;
- a claim of operational aircraft safety;
- a source of manufacturer-certified engine limits;
- a system claiming real-engine RUL accuracy from synthetic data;
- a generic AI chatbot pretending to be an engine model;
- a dashboard containing only randomly generated telemetry.

---

# 7. Target Users

## 7.1 UAV Operator

### Needs

- immediate engine health;
- live telemetry;
- active anomaly alerts;
- probable fault;
- severity;
- mission risk;
- clear recommended interpretation.

### Primary Question

> "Is the engine okay right now?"

---

## 7.2 Maintenance Engineer

### Needs

- fault evidence;
- subsystem health;
- degradation trends;
- RUL;
- uncertainty;
- mission history;
- replay;
- sensor-vs-engine fault differentiation.

### Primary Question

> "What is degrading, why, and how quickly?"

---

## 7.3 Mission Planner

### Needs

- current engine state;
- proposed mission profile;
- environmental conditions;
- projected health;
- thermal-margin proxy;
- RUL impact;
- mission-risk comparison.

### Primary Question

> "Which mission profile is more suitable for the current engine condition?"

---

## 7.4 Developer / Data Analyst

### Needs

- telemetry;
- simulation controls;
- dataset generation;
- model versions;
- experiment provenance;
- validation results.

### Primary Question

> "Can we reproduce, validate, and improve the system's result?"

---

# 8. User Jobs-to-be-Done

| User | Job |
|---|---|
| Operator | Understand current engine condition quickly |
| Operator | Detect meaningful abnormal behavior |
| Maintenance | Determine probable fault and supporting evidence |
| Maintenance | Track degradation over time |
| Maintenance | Estimate remaining useful life |
| Mission Planner | Compare mission scenarios |
| Mission Planner | Understand projected engine/mission impact |
| Analyst | Replay and analyze historical missions |
| Developer | Reproduce simulations and model results |

---

# 9. Core Product Workflow

```text
                    VIGIL
                      │
                      ▼
              ENGINE TELEMETRY
                      │
                      ▼
           DATA VALIDATION & CONTEXT
                      │
                      ▼
             PHYSICS BASELINE
          "What should happen?"
                      │
                      ▼
                 RESIDUALS
          "What actually changed?"
                      │
                      ▼
              AI / ML ANALYSIS
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Anomaly      Fault       Degradation
       Detection    Diagnosis    Estimation
          │           │           │
          └───────────┼───────────┘
                      ▼
              DIGITAL TWIN STATE
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        HEALTH       RUL       MISSION RISK
          │           │           │
          └───────────┼───────────┘
                      ▼
             OPERATOR / ENGINEER
                      │
                      ▼
                   DECISION
```

---

# 10. Product Scope

## 10.1 MVP — Mandatory

The MVP must demonstrate the complete VIGIL intelligence loop.

### MVP-01 — Synthetic Telemetry

Generate physically coherent simulated telemetry for a representative aero-piston engine.

### MVP-02 — Real-Time Streaming

Stream telemetry to the backend and dashboard.

### MVP-03 — Digital Twin

Maintain synchronized engine state.

### MVP-04 — Physics Baseline

Estimate expected healthy engine behavior under the current operating context.

### MVP-05 — Residual Analysis

Calculate observed-vs-expected deviation.

### MVP-06 — Anomaly Detection

Detect abnormal patterns from residuals and derived features.

### MVP-07 — Fault Diagnosis

Identify probable fault classes with confidence/probability and evidence.

### MVP-08 — Health Index

Display overall and subsystem health.

### MVP-09 — RUL

Provide prototype RUL with uncertainty.

### MVP-10 — Mission Simulation

Compare different mission/environment scenarios.

### MVP-11 — Mission Replay

Replay historical telemetry and derived engine intelligence.

### MVP-12 — GCS Dashboard

Provide an operator-facing interface for all core intelligence.

---

# 11. Phase 2 Scope

After the MVP is stable:

- improved physics model;
- more sophisticated temporal ML;
- improved degradation modeling;
- richer vibration analysis;
- more fault classes;
- advanced sensor-fault isolation;
- improved uncertainty estimation;
- CAN/SocketCAN adapter;
- ECU/FADEC integration pathway;
- fleet-level engine comparison;
- automated reports.

---

# 12. Future Scope

Potential future capabilities:

- edge AI deployment;
- federated learning;
- onboard Digital Twin;
- real engine telemetry integration;
- manufacturer-specific engine profiles;
- advanced physics-informed neural networks;
- transformer-based temporal models;
- fleet-level predictive maintenance;
- maintenance scheduling optimization;
- secure telemetry infrastructure;
- digital thread integration.

These capabilities are **not required for the initial SIH demonstrator**.

---

# 13. Functional Requirements

## FR-01 — Telemetry Acquisition

VIGIL shall accept timestamped engine telemetry.

Required parameters include:

- RPM;
- Cylinder Head Temperature (CHT);
- Exhaust Gas Temperature (EGT);
- oil pressure;
- oil temperature;
- fuel flow;
- vibration;
- battery voltage;
- alternator current;
- injection timing;
- altitude;
- ambient temperature;
- ambient pressure;
- throttle;
- engine age.

Telemetry shall also contain:

- mission ID;
- UAV ID;
- engine ID;
- timestamp.

---

## FR-02 — Telemetry Validation

The system shall validate incoming telemetry for:

- schema correctness;
- missing values;
- timestamp ordering;
- invalid values;
- stale data;
- packet/data-quality issues.

Invalid telemetry shall not silently become an engine-fault diagnosis.

---

## FR-03 — Operating Context

The system shall associate engine measurements with operating context including:

- altitude;
- ambient temperature;
- ambient pressure;
- throttle;
- mission duration/profile;
- engine age;
- modeled load.

---

## FR-04 — Digital Twin

The system shall maintain a continuously updated Digital Twin state.

The state shall include, where available:

- current telemetry;
- operating context;
- expected state;
- residuals;
- anomaly score;
- probable faults;
- health;
- subsystem health;
- degradation state;
- RUL;
- mission risk;
- model version;
- simulator/data version.

---

## FR-05 — Expected Healthy State

The system shall estimate what the engine should approximately be doing under its current operating conditions.

The implementation shall use a documented physics-informed/reduced-order approach for the prototype.

---

## FR-06 — Residual Analysis

For modeled parameters:

```text
Residual = Observed Value − Expected Value
```

Residuals shall be available to downstream anomaly and diagnostic components.

---

## FR-07 — Anomaly Detection

VIGIL shall identify abnormal operating behavior.

The anomaly layer shall provide:

- anomaly score;
- severity;
- detection timestamp;
- contributing features;
- model version.

The baseline implementation may use an unsupervised method such as Isolation Forest.

---

## FR-08 — Fault Diagnosis

The system shall estimate probable fault classes.

Initial fault classes:

- injector degradation;
- cooling degradation;
- lubrication issue;
- mechanical/vibration degradation;
- combustion instability/misfire;
- sensor drift/failure;
- electrical degradation where modeled.

The system shall provide probabilities/confidence rather than unsupported absolute certainty.

---

## FR-09 — Explainability

For a significant diagnosis, VIGIL shall show evidence such as:

- fuel-flow residual increased;
- EGT residual increased;
- RPM instability increased;
- vibration increased;
- CHT trend changed.

Evidence must originate from actual system features, not hardcoded explanatory text.

---

## FR-10 — Sensor-vs-Engine Fault Isolation

The system shall distinguish, where possible, between:

1. isolated sensor abnormality;
2. coherent multi-sensor engine degradation;
3. telemetry/data-quality problems.

Example:

```text
Only CHT deviates
→ sensor-drift hypothesis increases

CHT + EGT + vibration + fuel behavior deviate
→ engine-fault hypothesis increases
```

This behavior is a product differentiator.

---

## FR-11 — Health Index

VIGIL shall provide:

- overall engine health;
- subsystem health;
- health trend;
- degradation drivers.

Indicative subsystems:

- combustion;
- cooling;
- lubrication;
- injection;
- mechanical;
- electrical.

The health score is a **prototype indicator**, not a certified aircraft operating limit.

---

## FR-12 — Remaining Useful Life

VIGIL shall provide:

- RUL estimate;
- lower bound;
- upper bound;
- uncertainty;
- degradation trend;
- end-of-life criterion reference;
- model version.

RUL shall never be presented as a manufacturer-certified value.

---

## FR-13 — Mission Simulation

Users shall be able to configure a mission scenario.

Inputs should include:

- altitude;
- temperature;
- throttle;
- mission duration;
- engine age;
- load/payload proxy;
- current degradation state.

The system shall produce projected:

- engine telemetry;
- health trajectory;
- degradation trajectory;
- thermal-margin proxy;
- fuel consumption;
- ending health;
- RUL change;
- mission-risk indicator.

---

## FR-14 — Scenario Comparison

The user shall be able to compare at least two scenarios.

Example:

```text
Scenario A
18,000 ft / 70% throttle / 8 h
        ↓
LOW relative risk

Scenario B
22,000 ft / 70% throttle / 8 h
        ↓
HIGHER relative risk
```

Risk is an illustrative prototype decision-support metric, not a probability of aircraft loss.

---

## FR-15 — Historical Replay

The system shall allow users to replay a completed mission.

Replay shall synchronize:

- telemetry;
- expected values;
- residuals;
- anomaly events;
- diagnosis;
- health;
- RUL;
- mission risk.

---

## FR-16 — Reports

VIGIL shall be able to generate a mission/diagnostic report containing:

- mission metadata;
- engine summary;
- health trend;
- anomalies;
- diagnosis;
- evidence;
- RUL;
- mission risk;
- timeline;
- model/data provenance.

---

# 14. User Interface Requirements

## 14.1 Main Dashboard

The dashboard shall prioritize:

1. engine health;
2. Digital Twin;
3. live telemetry;
4. expected-vs-actual behavior;
5. AI diagnosis;
6. RUL;
7. mission risk.

The engine Digital Twin should be the primary visual element rather than a small dashboard card.

---

## 14.2 Required Screens

```text
/dashboard
/telemetry
/engine-health
/digital-twin
/diagnostics
/rul
/mission
/replay
/reports
/settings
```

---

## 14.3 Dashboard Questions

A user should be able to answer within seconds:

> Is the engine healthy?

> What is it doing?

> Is it behaving as expected?

> Is anything abnormal?

> What is likely wrong?

> How much useful life remains?

> What does this mean for the mission?

---

# 15. UX Requirements

## UX-01 — Clarity

The most important information must be visually dominant.

## UX-02 — Explainability

Alerts must explain why they were generated.

## UX-03 — Uncertainty

Uncertain outputs must visibly communicate uncertainty.

## UX-04 — Data Quality

The UI must distinguish:

- live;
- stale;
- disconnected;
- unavailable;
- historical.

## UX-05 — No False Confidence

The interface must not imply certification or operational safety.

## UX-06 — Consistency

Health, anomaly and risk states must use consistent semantic labels throughout the application.

---

# 16. Data Requirements

## 16.1 Synthetic Data

Because target-specific aero-piston MALE-UAV run-to-failure telemetry may not be publicly available, the prototype shall use a controlled synthetic-data laboratory.

Pipeline:

```text
Mission Configuration
        ↓
Atmosphere / Operating Context
        ↓
Engine Physics Model
        ↓
Healthy Telemetry
        ↓
Fault / Degradation Injection
        ↓
Sensor Noise / Drift
        ↓
Final Telemetry
        ↓
Dataset + Labels + Provenance
```

---

## 16.2 Required Synthetic Scenarios

At minimum:

- healthy nominal operation;
- high-altitude operation;
- hot-weather operation;
- endurance mission;
- rapid throttle transition;
- injector degradation;
- cooling degradation;
- lubrication issue;
- mechanical/vibration degradation;
- sensor drift;
- telemetry packet loss.

---

## 16.3 Data Provenance

Synthetic datasets should record:

- simulator version;
- engine profile version;
- scenario type;
- fault type;
- degradation level;
- random seed;
- mission profile;
- generation timestamp.

---

# 17. AI/ML Product Requirements

VIGIL shall prioritize reliable and explainable baselines before model complexity.

## Anomaly Detection

Baseline candidate:

**Isolation Forest**

Inputs may include:

- normalized residuals;
- trends;
- rolling statistics;
- vibration features;
- operating context.

Outputs:

- anomaly score;
- severity;
- contributing features.

## Fault Diagnosis

Baseline candidate:

**XGBoost / tree-based classifier**

Training/evaluation shall use controlled scenario labels and held-out missions.

## RUL

The RUL system shall:

- model degradation;
- estimate remaining life;
- expose uncertainty;
- define an explicit prototype end-of-life criterion;
- avoid temporal leakage.

Deep sequence models may be evaluated later.

---

# 18. Safety, Trust & Data Honesty

This section is mandatory.

## SH-01 — No Invented Engineering Truth

If an engineering equation, constant, operating limit, fault relationship, unit, or validation criterion is unknown:

```text
ASSUMPTION_REQUIRED
```

The implementation must not invent a value and present it as factual.

## SH-02 — Synthetic Data Labeling

Synthetic data must be explicitly labeled as synthetic.

## SH-03 — No Unsupported Accuracy Claims

Do not claim real-engine accuracy based solely on synthetic validation.

## SH-04 — No Certified Limits

Prototype health/risk ranges must never be described as aircraft safety limits.

## SH-05 — Probabilistic Diagnosis

AI diagnosis must be represented as probable/inferred unless supported by verified evidence.

## SH-06 — Model Provenance

Important model outputs should include:

- timestamp;
- model version;
- simulator/data version where relevant.

---

# 19. Success Metrics

## Product-Level Metrics

### PSM-01 — End-to-End Demonstration

A known simulated fault successfully propagates through:

```text
Fault Injection
→ Telemetry Change
→ Residual Growth
→ Anomaly
→ Diagnosis
→ Health Change
→ RUL Change
→ Mission Impact
→ Replay
```

### PSM-02 — Real-Time Responsiveness

Dashboard updates continuously during a live simulation.

### PSM-03 — Explainability

Every major anomaly/diagnosis shown to the user includes evidence.

### PSM-04 — Scenario Sensitivity

Changing mission/environment parameters produces a coherent change in projected outcomes.

### PSM-05 — Reproducibility

A simulation can be reproduced using recorded configuration and random seed.

### PSM-06 — Validation Integrity

Temporal telemetry datasets are split by mission/run rather than random rows.

---

# 20. Golden Demo Acceptance Criteria

The SIH demonstration should follow one coherent story.

### Step 1 — Healthy State

Start:

```text
UAV-001
Engine: AERO-PX-01
Health: Healthy
```

### Step 2 — Start Mission

Start an endurance/high-altitude mission.

### Step 3 — Inject Degradation

Gradually introduce:

```text
Injector degradation
```

### Step 4 — Show Physics Deviation

Expected-vs-actual residuals begin increasing.

### Step 5 — Detect

VIGIL identifies abnormal behavior.

### Step 6 — Diagnose

VIGIL shows:

```text
Likely fault:
Injector degradation

Confidence:
[probabilistic value]

Evidence:
Fuel-flow residual
EGT residual
RPM instability
Vibration
```

### Step 7 — Prognostics

Health decreases and RUL updates with uncertainty.

### Step 8 — Mission What-If

Increase altitude.

Show projected changes in:

- thermal-margin proxy;
- health;
- degradation;
- mission risk.

### Step 9 — Mitigation Scenario

Adjust the mission profile.

Show how the projected result changes.

### Step 10 — Replay

Replay the mission and identify when the abnormal behavior first became meaningful.

---

# 21. Definition of Done

VIGIL MVP is considered complete when:

- [ ] Synthetic engine telemetry works.
- [ ] Real-time telemetry reaches the frontend.
- [ ] Digital Twin state updates continuously.
- [ ] Physics expected state is calculated.
- [ ] Residuals are calculated.
- [ ] Anomaly detection works.
- [ ] Fault diagnosis works for at least the golden fault.
- [ ] Diagnosis exposes evidence.
- [ ] Sensor-vs-engine logic is demonstrated.
- [ ] Health index works.
- [ ] RUL estimate and uncertainty are displayed.
- [ ] Mission simulation works.
- [ ] Scenario comparison works.
- [ ] Mission replay works.
- [ ] Dashboard is connected to backend data.
- [ ] Model/data provenance is visible.
- [ ] Automated tests cover critical modules.
- [ ] End-to-end golden scenario passes.
- [ ] No unsupported engineering claims appear in the UI or documentation.

---

# 22. Constraints

## Technical

- Target-specific real engine data may not be available.
- Prototype therefore requires synthetic data.
- Physics assumptions must be explicitly documented.
- Real-time behavior must be demonstrated through simulation initially.

## Product

- SIH demonstration time is limited.
- The system must prioritize a coherent end-to-end story over a large number of disconnected features.

## Validation

Synthetic validation cannot establish real-world aircraft-engine safety or production accuracy.

---

# 23. Risks

| Risk | Impact | Product Response |
|---|---|---|
| Lack of target engine data | High | Controlled synthetic-data laboratory |
| Invented physics assumptions | High | `ASSUMPTION_REQUIRED` rule |
| ML overfitting | High | Mission-level validation |
| Sensor fault mistaken for engine fault | High | Cross-sensor consistency |
| RUL overclaim | High | Uncertainty + explicit prototype EOL criterion |
| Dashboard becomes too dense | Medium | Strong information hierarchy |
| Too much infrastructure | Medium | Modular monolith first |
| AI-generated incorrect engineering logic | High | Human domain review |

---

# 24. Product Architecture at a Conceptual Level

```text
┌───────────────────────────────────────────┐
│             ENGINE / SIMULATOR            │
└─────────────────────┬─────────────────────┘
                      ↓
┌───────────────────────────────────────────┐
│        TELEMETRY + DATA VALIDATION        │
└─────────────────────┬─────────────────────┘
                      ↓
┌───────────────────────────────────────────┐
│       PHYSICS + EXPECTED HEALTHY STATE   │
└─────────────────────┬─────────────────────┘
                      ↓
┌───────────────────────────────────────────┐
│              RESIDUAL ENGINE              │
└─────────────────────┬─────────────────────┘
                      ↓
┌───────────────────────────────────────────┐
│                 AI / ML                   │
│  Anomaly → Diagnosis → Degradation → RUL │
└─────────────────────┬─────────────────────┘
                      ↓
┌───────────────────────────────────────────┐
│             DIGITAL TWIN CORE             │
└─────────────────────┬─────────────────────┘
                      ↓
┌───────────────────────────────────────────┐
│       HEALTH + RUL + MISSION RISK         │
└─────────────────────┬─────────────────────┘
                      ↓
┌───────────────────────────────────────────┐
│              VIGIL GCS / UI               │
└───────────────────────────────────────────┘
```

Detailed technical architecture is defined separately in `TRD.md`.

---

# 25. Product Principles

## Principle 1 — Digital Twin ≠ Dashboard

A dashboard displays information.

VIGIL maintains a synchronized representation of the engine state and uses that state for reasoning.

## Principle 2 — Physics + AI

AI should operate with physical/operating context rather than treating telemetry as meaningless numbers.

## Principle 3 — Detect Before Failure

The objective is to identify meaningful degradation before a catastrophic endpoint.

## Principle 4 — Explain the Diagnosis

A fault prediction without evidence is not sufficient for the operator-facing product.

## Principle 5 — Sensor Fault ≠ Engine Fault

The system must reason across multiple signals.

## Principle 6 — RUL Must Show Uncertainty

A single precise-looking number can create false confidence.

## Principle 7 — Mission-Aware Intelligence

Engine health should be connected to the conditions under which the engine is expected to operate.

## Principle 8 — Honest Engineering

When something is simulated, assumed, or unvalidated, VIGIL must say so.

---

# 26. Glossary

**MALE UAV**  
Medium Altitude Long Endurance unmanned aircraft.

**Aero-piston engine**  
The target propulsion-engine class for the SIH problem.

**Digital Twin**  
A continuously updated virtual representation of the physical engine state.

**Telemetry**  
Timestamped engine measurements and operating context.

**Physics baseline**  
Model-derived expectation of healthy engine behavior under current conditions.

**Residual**  
Difference between observed and expected behavior.

**Anomaly**  
Behavior that deviates meaningfully from learned/expected normal behavior.

**Fault diagnosis**  
Estimation of the probable underlying fault causing observed abnormal behavior.

**Health Index**  
A normalized prototype indicator representing estimated engine condition.

**RUL**  
Remaining Useful Life; the estimated remaining operating life according to the defined prototype degradation/end-of-life criterion.

**Mission risk**  
A prototype decision-support indicator describing how current engine condition and mission conditions interact.

**Synthetic telemetry**  
Telemetry generated by a simulation rather than collected from a physical engine.

---

# 27. Final Product Statement

> **VIGIL does not wait for the engine to fail. It compares what the engine is doing with what it should be doing, detects meaningful deviation, reasons about the probable cause, predicts how the condition may evolve, and evaluates what that means for the mission.**

---

## Document Control

| Field | Value |
|---|---|
| Document | PRD.md |
| Product | VIGIL |
| SIH Problem | SIH26054 |
| Version | 1.0 |
| Status | Initial Product Specification |
| Intended Consumer | Product team, engineering team, UI team, AI coding agents |
| Next Documents | TRD.md, UI_UX_DESIGN.md, DATABASE_SCHEMA.md, WIREFRAME.md, FRONTEND_BACKEND_WIRING.md |
