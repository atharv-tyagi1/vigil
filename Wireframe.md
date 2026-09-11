# VIGIL — Wireframe Specification

> **Product:** VIGIL — Intelligent UAV Engine Health & Prognostics  
> **SIH Problem Statement:** SIH26054  
> **Version:** 1.0  
> **Status:** Implementation-ready MVP wireframe  
> **Related Documents:** `PRD.md`, `TRD.md`, `UI_UX_DESIGN.md`, `DATABASE_SCHEMA.md`

---

# 1. Purpose

This document defines the structural wireframes for the VIGIL Ground Control Station.

These are **layout wireframes**, not final visual designs.

They define:

- page structure;
- information hierarchy;
- component placement;
- navigation;
- user flow;
- dashboard composition;
- interaction priorities.

The visual styling is defined in `UI_UX_DESIGN.md`.

---

# 2. Wireframe Philosophy

VIGIL must communicate:

```text
CURRENT STATE
      ↓
DEVIATION
      ↓
DIAGNOSIS
      ↓
PROGNOSIS
      ↓
MISSION IMPACT
```

The operator should not need to open several screens to understand a developing engine issue.

The dashboard therefore acts as the primary command view.

---

# 3. Application Shell

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ VIGIL     UAV-001 / AERO-PX-01     MISSION-008    ● LIVE    18:42:31 UTC│
├───────────────┬──────────────────────────────────────────────────────────┤
│               │                                                          │
│  VIGIL        │                                                          │
│               │                   PAGE CONTENT                           │
│ ▣ Dashboard   │                                                          │
│ ◉ Telemetry   │                                                          │
│ ◈ Health      │                                                          │
│ ◇ Twin        │                                                          │
│ ⚠ Diagnostics │                                                          │
│ ◫ RUL         │                                                          │
│ ◎ Mission     │                                                          │
│ ↻ Replay      │                                                          │
│ ▤ Reports     │                                                          │
│ ⚙ Settings    │                                                          │
│               │                                                          │
│               │                                                          │
├───────────────┴──────────────────────────────────────────────────────────┤
│ CONNECTION: ● CONNECTED    SOURCE: SIMULATION    MODEL: v0.3.0          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 4. Navigation

Primary navigation:

```text
Dashboard
Telemetry
Engine Health
Digital Twin
Diagnostics
RUL & Prognostics
Mission Simulator
Mission Replay
Reports
Settings
```

The active route must be visually obvious.

---

# 5. Dashboard Wireframe

## Primary SIH Screen

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ ENGINE HEALTH                                      DIGITAL TWIN           │
│                                                                          │
│  ┌─────────────────────┐                ┌─────────────────────────────┐  │
│  │                     │                │                             │  │
│  │        92           │                │                             │  │
│  │       /100          │                │                             │  │
│  │                     │                │        3D ENGINE             │  │
│  │       GOOD          │                │                             │  │
│  │                     │                │                             │  │
│  │  Trend: STABLE      │                │      [ROTATE / ZOOM]        │  │
│  └─────────────────────┘                │                             │  │
│                                         └─────────────────────────────┘  │
├──────────────────────────────────────────────────────────────────────────┤
│ LIVE TELEMETRY                                                          │
│                                                                          │
│ RPM       CHT       EGT       OIL PRESS    OIL TEMP    FUEL     VIB     │
│ 2450      178       692       4.2          94          18.7     0.031   │
│                                                                          │
├──────────────────────────────────────┬───────────────────────────────────┤
│ EXPECTED vs ACTUAL                   │ AI DIAGNOSTICS                    │
│                                      │                                   │
│       /\                             │ ● SYSTEM NORMAL                   │
│      /  \     Actual                 │                                   │
│ ----/----\---- Expected              │ No significant abnormality        │
│                                      │ detected                           │
│                                      │                                   │
├──────────────────────────────────────┼───────────────────────────────────┤
│ HEALTH / DEGRADATION                 │ RUL                               │
│                                      │                                   │
│ 100 ───────────                      │          412 h                     │
│        ╲                             │                                   │
│  80     ╲──────                      │      380 ───── 450                │
│                                      │                                   │
│ Trend: STABLE                        │ Estimated interval                │
├──────────────────────────────────────┴───────────────────────────────────┤
│ MISSION                                                                  │
│                                                                          │
│ ENDURANCE-08     ALT: HIGH     STATUS: ACTIVE     RISK: LOW              │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 6. Dashboard — Fault State

When an anomaly appears, the dashboard should transform without changing the overall layout.

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ ENGINE HEALTH                                      DIGITAL TWIN           │
│                                                                          │
│  ┌─────────────────────┐                ┌─────────────────────────────┐  │
│  │        76           │                │                             │  │
│  │       /100          │                │       ENGINE                 │  │
│  │                     │                │                             │  │
│  │     DEGRADED        │                │      INJECTOR                │  │
│  │                     │                │      HIGHLIGHTED             │  │
│  └─────────────────────┘                └─────────────────────────────┘  │
├──────────────────────────────────────────────────────────────────────────┤
│ LIVE TELEMETRY                                                          │
│                                                                          │
│ RPM 2450   CHT 181   EGT 721 ▲   OIL 4.1   FUEL 21.2 ▲   VIB 0.047 ▲   │
├──────────────────────────────────────┬───────────────────────────────────┤
│ EXPECTED vs ACTUAL                   │ ⚠ AI DIAGNOSIS                    │
│                                      │                                   │
│ actual ─────╮                        │ PROBABLE FAULT                    │
│             ╰──────                   │ Injector degradation              │
│ expected ─────────                   │                                   │
│                                      │ Confidence: 84%                   │
│ Residual increasing                  │ Severity: WARNING                 │
├──────────────────────────────────────┼───────────────────────────────────┤
│ HEALTH TREND                         │ RUL                               │
│                                      │                                   │
│ 92 ─────╲                            │ 412 h → 356 h                     │
│          ╲──── 76                     │                                   │
│                                      │ Interval: visible                │
├──────────────────────────────────────┴───────────────────────────────────┤
│ MISSION RISK: HIGHER                                                     │
│ Evidence: EGT residual ↑ | Fuel-flow residual ↑ | RPM instability ↑     │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 7. Dashboard — Golden Demo Flow

The same dashboard supports the entire SIH story:

```text
HEALTHY
  ↓
NORMAL TELEMETRY
  ↓
DEGRADATION INJECTION
  ↓
RESIDUAL GROWTH
  ↓
ANOMALY
  ↓
DIAGNOSIS
  ↓
HEALTH DROP
  ↓
RUL UPDATE
  ↓
MISSION RISK CHANGE
```

This minimizes navigation during judging.

---

# 8. Telemetry Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ TELEMETRY                                             ● LIVE              │
├──────────────────────────────────────────────────────────────────────────┤
│ PARAMETERS                                                                │
│                                                                          │
│ [RPM] [CHT] [EGT] [OIL PRESS] [OIL TEMP] [FUEL] [VIBRATION] [ELECTRICAL]│
│                                                                          │
│ RANGE: [ Last 5 min ▼ ]     MODE: [ LIVE ▼ ]     [ PAUSE ] [ EXPORT ]   │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                        TELEMETRY CHART                                   │
│                                                                          │
│ Actual ───────────────────────╮────────                                 │
│                               ╰──────                                    │
│ Expected - - - - - - - - - - - - - -                                   │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────┤
│ DATA QUALITY                                                             │
│                                                                          │
│ RPM          VALID       CHT          VALID       EGT       VALID        │
│ OIL PRESS    VALID       FUEL         VALID       VIB       VALID        │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 9. Engine Health Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ ENGINE HEALTH                                                             │
├──────────────────────────┬───────────────────────────────────────────────┤
│ OVERALL                  │ HEALTH TREND                                 │
│                          │                                               │
│         92               │ 100 ────────╲                               │
│        /100              │             ╲──────                         │
│         GOOD             │                                               │
├──────────────────────────┴───────────────────────────────────────────────┤
│ SUBSYSTEM HEALTH                                                         │
│                                                                          │
│ COMBUSTION       ████████████████████ 94                                │
│ INJECTION        ██████████████████   87                                │
│ COOLING          ███████████████████  91                                │
│ LUBRICATION      ████████████████████ 95                                │
│ MECHANICAL       ██████████████████   89                                │
│ ELECTRICAL       ████████████████████ 97                                │
├──────────────────────────────────────┬───────────────────────────────────┤
│ DEGRADATION DRIVERS                 │ STATUS                            │
│                                      │                                   │
│ Injection       HIGH                 │ Trend: STABLE                     │
│ Mechanical      LOW                  │ Data: VALID                      │
│ Cooling         LOW                  │ Model: v0.3.0                    │
└──────────────────────────────────────┴───────────────────────────────────┘
```

---

# 10. Digital Twin Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ DIGITAL TWIN                                         STATE: SYNCHRONIZED │
├───────────────────────────────────┬──────────────────────────────────────┤
│                                   │ ENGINE STATE                         │
│                                   │                                      │
│                                   │ Health          92                   │
│          LARGE 3D ENGINE          │ Anomaly         0.18                 │
│                                   │ RUL             412 h                │
│                                   │ Risk            LOW                  │
│                                   │                                      │
│                                   ├──────────────────────────────────────┤
│                                   │ OPERATING CONTEXT                    │
│                                   │                                      │
│                                   │ Altitude        HIGH                 │
│                                   │ Throttle        70%                  │
│                                   │ Temperature     —                    │
│                                   │ Pressure        —                    │
├───────────────────────────────────┴──────────────────────────────────────┤
│ CONTROLS                                                                 │
│ [HEALTH] [THERMAL] [VIBRATION] [LABELS] [RESET VIEW]                    │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 11. Diagnostics Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ DIAGNOSTICS                                      ● ACTIVE ANOMALY         │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│ ⚠ PROBABLE FAULT                                                         │
│                                                                          │
│ INJECTOR DEGRADATION                                                     │
│ Confidence: 84%                    Severity: WARNING                     │
│                                                                          │
├─────────────────────────────────────┬────────────────────────────────────┤
│ EVIDENCE                            │ ALTERNATIVE HYPOTHESES              │
│                                     │                                    │
│ ✓ Fuel-flow residual ↑              │ Cooling degradation       21%     │
│ ✓ EGT residual ↑                    │ Lubrication issue          13%     │
│ ✓ RPM instability ↑                 │ Sensor drift                8%     │
│ ✓ Vibration RMS ↑                   │                                    │
├─────────────────────────────────────┴────────────────────────────────────┤
│ FAULT ISOLATION                                                          │
│                                                                          │
│ Sensor-only correlation: LOW                                             │
│ Multi-sensor engine correlation: HIGH                                    │
│                                                                          │
│ Interpretation: Engine-level degradation more likely                    │
├──────────────────────────────────────────────────────────────────────────┤
│ EVENT TIMELINE                                                           │
│                                                                          │
│ 12:41 Normal ── 12:46 Residual ── 12:49 Anomaly ── 12:51 Diagnosis       │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 12. RUL Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ RUL & PROGNOSTICS                                                        │
├─────────────────────────────┬────────────────────────────────────────────┤
│ ESTIMATED RUL               │ UNCERTAINTY                                │
│                             │                                            │
│          412 h              │ 380 ├───────────────┤ 450                 │
│                             │                                            │
│ Prototype estimate          │ Estimated interval                       │
├─────────────────────────────┴────────────────────────────────────────────┤
│ DEGRADATION TREND                                                        │
│                                                                          │
│ 100 ────────╲                                                          │
│              ╲──────                                                    │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────┤
│ PROJECTED HEALTH                                                         │
│                                                                          │
│ Current ────────────────╲                                               │
│                          ╲────                                           │
├──────────────────────────────────────────────────────────────────────────┤
│ END-OF-LIFE CRITERION                                                    │
│ Prototype criterion: [model-defined]                                    │
│ Model: v0.3.0                                                            │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 13. Mission Simulator Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ MISSION SIMULATOR                                                        │
├──────────────────────────────┬───────────────────────────────────────────┤
│ CURRENT ENGINE               │ MISSION CONFIGURATION                     │
│                              │                                           │
│ Health: 81                   │ Altitude       [ 18,000 ]                │
│ RUL: 310 h                   │ Temperature    [ —       ]                │
│ State: DEGRADED              │ Throttle       [ 70%     ]                │
│                              │ Duration       [ 8 h     ]                │
│                              │ Load           [ —       ]                │
│                              │                                           │
│                              │ [ RUN SIMULATION ]                        │
├──────────────────────────────┴───────────────────────────────────────────┤
│ PROJECTED OUTCOME                                                        │
│                                                                          │
│ END HEALTH       72          RUL IMPACT        -18 h                     │
│ THERMAL MARGIN   REDUCED     MISSION RISK      HIGHER                    │
├──────────────────────────────────────────────────────────────────────────┤
│ PROJECTED HEALTH / RISK                                                  │
│                                                                          │
│                     projection                                          │
│        ─────────────────╲                                               │
│                         ╲────                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 14. Scenario Comparison Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ SCENARIO COMPARISON                                                      │
├──────────────────────────────┬───────────────────────────────────────────┤
│ SCENARIO A                   │ SCENARIO B                                │
│                              │                                           │
│ Altitude: 18,000             │ Altitude: 22,000                          │
│ Throttle: 70%                │ Throttle: 70%                             │
│ Duration: 8 h                │ Duration: 8 h                             │
│                              │                                           │
│ End Health: 78               │ End Health: 69                            │
│ RUL Impact: -8 h             │ RUL Impact: -21 h                         │
│ Risk: LOWER                  │ Risk: HIGHER                              │
├──────────────────────────────┴───────────────────────────────────────────┤
│                     COMPARISON CHART                                    │
│                                                                          │
│ Health       A ████████████████                                         │
│              B █████████████                                             │
│                                                                          │
│ RUL Impact   A ██████                                                    │
│              B ███████████████                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 15. Mission Replay Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ MISSION REPLAY                                                           │
├──────────────────────────────────────────────────────────────────────────┤
│ MISSION: ENDURANCE-08                         SOURCE: HISTORICAL         │
│                                                                          │
│ CURRENT TIME: 04:52:14                                                   │
│                                                                          │
│ 00:00 ───── 02:00 ───── 04:00 ───── 06:00 ───── 08:00                  │
│                         ▲                                                │
│                         │                                                │
│                    ANOMALY BEGINS                                        │
├──────────────────────────────────────────────────────────────────────────┤
│ [◀] [ PLAY ] [▶]        SPEED [1x ▼]        [RESET]                     │
├──────────────────────────────┬───────────────────────────────────────────┤
│ CURRENT STATE                │ ACTIVE EVENT                              │
│                              │                                           │
│ Health: 83                   │ Injector degradation                      │
│ Anomaly: ACTIVE              │ Confidence: 81%                           │
│ RUL: 398 h                   │ Evidence: 4 parameters                    │
├──────────────────────────────┴───────────────────────────────────────────┤
│ SYNCHRONIZED TELEMETRY                                                   │
│                                                                          │
│ RPM / EGT / CHT / Fuel / Vibration                                      │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 16. Reports Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ REPORTS                                                                  │
├──────────────────────────────────────────────────────────────────────────┤
│ [MISSION REPORT] [DIAGNOSTIC REPORT] [HEALTH REPORT]                    │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│ REPORT LIST                                                              │
│                                                                          │
│ Mission 008   Endurance Mission      11 Sep 2026     [VIEW] [EXPORT]    │
│ Engine 001    Diagnostic Summary     11 Sep 2026     [VIEW] [EXPORT]    │
│ Mission 007   Replay Analysis        10 Sep 2026     [VIEW] [EXPORT]    │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# 17. Settings Page

```text
┌──────────────────────────────────────────────────────────────────────────┐
│ SETTINGS                                                                 │
├────────────────┬─────────────────────────────────────────────────────────┤
│ Engine Profile │ ENGINE CONFIGURATION                                   │
│ Telemetry      │                                                         │
│ Data Source    │ Engine: AERO-PX-01                                     │
│ Simulation     │ Profile: [ SELECT ]                                    │
│ Models         │                                                         │
│ Display        │                                                         │
│ System         │                                                         │
│ About          │                                                         │
└────────────────┴─────────────────────────────────────────────────────────┘
```

---

# 18. Global Alert Drawer

Alerts should be accessible without leaving the current page.

```text
┌────────────────────────────────────────────┐
│ ACTIVE EVENTS                         ×    │
├────────────────────────────────────────────┤
│ ⚠ Injector degradation                    │
│    Confidence: 84%                        │
│    12:51 UTC                              │
├────────────────────────────────────────────┤
│ ● Telemetry stable                        │
│    12:49 UTC                              │
└────────────────────────────────────────────┘
```

Clicking an event should navigate to the relevant diagnostic context.

---

# 19. Mobile Wireframe

Mobile is secondary, but critical information should remain accessible.

```text
┌─────────────────────────────┐
│ VIGIL       ● LIVE          │
├─────────────────────────────┤
│ ENGINE HEALTH               │
│                             │
│          92                 │
│         GOOD                │
├─────────────────────────────┤
│ DIGITAL TWIN                │
│                             │
│       [ ENGINE ]            │
│                             │
├─────────────────────────────┤
│ TELEMETRY                   │
│ RPM       2450              │
│ CHT       178               │
│ EGT       692               │
│ OIL       4.2               │
├─────────────────────────────┤
│ AI DIAGNOSTICS              │
│ SYSTEM NORMAL               │
├─────────────────────────────┤
│ RUL         412 h            │
├─────────────────────────────┤
│ MISSION     LOW RISK         │
├─────────────────────────────┤
│ [Dashboard] [Alerts] [More] │
└─────────────────────────────┘
```

---

# 20. Primary User Flows

## Flow A — Normal Monitoring

```text
Dashboard
   ↓
Observe Health
   ↓
Inspect Telemetry
   ↓
Inspect Digital Twin
```

## Flow B — Fault Detection

```text
Dashboard
   ↓
Anomaly Alert
   ↓
Diagnostics
   ↓
Evidence
   ↓
Fault Isolation
   ↓
RUL
```

## Flow C — Mission Planning

```text
Dashboard
   ↓
Mission Simulator
   ↓
Configure Scenario
   ↓
Run Simulation
   ↓
Scenario Comparison
   ↓
Mission Decision
```

## Flow D — Historical Investigation

```text
Mission Replay
   ↓
Select Mission
   ↓
Play Timeline
   ↓
Anomaly Marker
   ↓
Diagnostics
   ↓
Health / RUL
```

---

# 21. Judge Demonstration Flow

The UI should support this exact navigation sequence:

```text
1. DASHBOARD
   Healthy engine

        ↓

2. LIVE TELEMETRY
   Normal behavior

        ↓

3. FAULT INJECTION
   Injector degradation begins

        ↓

4. DASHBOARD
   Residuals increase

        ↓

5. DIAGNOSTICS
   Injector degradation identified

        ↓

6. RUL
   Remaining life changes

        ↓

7. MISSION SIMULATOR
   Increase altitude

        ↓

8. SCENARIO COMPARISON
   Show higher projected risk

        ↓

9. REPLAY
   Show first meaningful anomaly
```

The demo should avoid unnecessary navigation.

---

# 22. Wireframe Component Priority

## P0

```text
AppShell
TopBar
Sidebar
HealthScore
EngineTwin
TelemetryStrip
TelemetryChart
ResidualChart
DiagnosticPanel
RULPanel
MissionRisk
```

## P1

```text
HealthTrend
SubsystemHealth
EvidenceList
FaultProbability
MissionSimulator
ScenarioComparison
ReplayTimeline
```

## P2

```text
Reports
Settings
AdvancedFilters
AdvancedTwinModes
MobileOptimization
```

---

# 23. Implementation Rule

Do not attempt to build every wireframe before the primary dashboard works.

Build in this order:

```text
App Shell
   ↓
Dashboard Layout
   ↓
Telemetry
   ↓
Digital Twin
   ↓
Health
   ↓
Diagnostics
   ↓
RUL
   ↓
Mission
   ↓
Replay
   ↓
Reports / Settings
```

The primary dashboard is the highest-value screen.

---

# 24. Final Wireframe Principle

VIGIL should visually communicate one continuous chain:

```text
┌──────────┐
│  ENGINE  │
└────┬─────┘
     ↓
┌──────────┐
│  HEALTH  │
└────┬─────┘
     ↓
┌──────────┐
│ TELEMETRY│
└────┬─────┘
     ↓
┌──────────┐
│ RESIDUAL │
└────┬─────┘
     ↓
┌──────────┐
│  FAULT   │
└────┬─────┘
     ↓
┌──────────┐
│   RUL    │
└────┬─────┘
     ↓
┌──────────┐
│ MISSION  │
└────┬─────┘
     ↓
┌──────────┐
│ DECISION │
└──────────┘
```

The wireframe exists to ensure that the final UI tells this story clearly and quickly during the SIH demonstration.
