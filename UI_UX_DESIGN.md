# VIGIL — UI/UX Design Specification

> **Product:** VIGIL — Intelligent UAV Engine Health & Prognostics  
> **SIH Problem Statement:** SIH26054  
> **Document:** UI/UX Design Specification  
> **Version:** 1.0  
> **Status:** Implementation-ready MVP specification  
> **Related Documents:** `PRD.md`, `TRD.md`

---

# 1. Purpose

This document defines the visual language, information architecture, interaction patterns, screen requirements, component system, responsive behavior, and implementation guidance for the VIGIL Ground Control Station (GCS).

The UI must make VIGIL look and behave like an **aerospace engine intelligence system**, not a generic SaaS analytics dashboard.

The most important user experience principle is:

> **The operator should understand the engine state, detect an abnormality, understand why it happened, and assess its mission impact without navigating through multiple unrelated screens.**

---

# 2. UX Goals

The VIGIL interface must:

1. Communicate engine health immediately.
2. Make the Digital Twin the visual centerpiece.
3. Show live telemetry without overwhelming the operator.
4. Clearly separate observed values from expected values.
5. Make AI findings explainable.
6. Show degradation and RUL as trends, not isolated numbers.
7. Connect engine condition to mission consequences.
8. Make simulation and replay easy to understand.
9. Clearly distinguish live, simulated, historical, and stale data.
10. Look credible in an SIH judge demonstration.

---

# 3. Primary Design Direction

## Visual Theme

**Aerospace Mission Control + Advanced Engine Digital Twin**

Visual characteristics:

- dark command-center interface;
- high information density but strong hierarchy;
- restrained use of accent colors;
- thin technical borders;
- subtle grid/background texture;
- compact telemetry typography;
- large engine visualization;
- strong status indicators;
- minimal decorative UI;
- smooth real-time transitions.

Avoid:

- generic admin-dashboard cards;
- excessive gradients;
- oversized rounded SaaS cards;
- excessive glassmorphism;
- cartoon-like 3D;
- unnecessary animations;
- too many colors;
- tiny unreadable charts.

---

# 4. Information Hierarchy

Every major screen should follow this priority:

```text
1. ENGINE HEALTH
        ↓
2. DIGITAL TWIN STATE
        ↓
3. LIVE TELEMETRY
        ↓
4. EXPECTED vs ACTUAL
        ↓
5. AI DIAGNOSIS
        ↓
6. RUL / DEGRADATION
        ↓
7. MISSION IMPACT
```

The UI should answer these questions in order:

```text
Is the engine healthy?
        ↓
What is happening?
        ↓
Is it behaving as expected?
        ↓
What is abnormal?
        ↓
What is probably causing it?
        ↓
How is it degrading?
        ↓
What does it mean for the mission?
```

---

# 5. Application Information Architecture

Primary navigation:

```text
VIGIL
│
├── Dashboard
├── Telemetry
├── Engine Health
├── Digital Twin
├── Diagnostics
├── RUL & Prognostics
├── Mission Simulator
├── Mission Replay
├── Reports
└── Settings
```

Recommended route structure:

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

# 6. Global Layout

```text
┌───────────────────────────────────────────────────────────────┐
│ VIGIL        UAV-001 / AERO-PX-01       LIVE ●    UTC 18:42  │
├──────────────┬────────────────────────────────────────────────┤
│              │                                                │
│ NAVIGATION   │                 MAIN CONTENT                   │
│              │                                                │
│ Dashboard    │                                                │
│ Telemetry    │                                                │
│ Health       │                                                │
│ Digital Twin │                                                │
│ Diagnostics  │                                                │
│ RUL          │                                                │
│ Mission      │                                                │
│ Replay       │                                                │
│ Reports      │                                                │
│ Settings     │                                                │
│              │                                                │
├──────────────┴────────────────────────────────────────────────┤
│ SYSTEM STATUS / CONNECTION / MODEL VERSION / DATA SOURCE     │
└───────────────────────────────────────────────────────────────┘
```

---

# 7. Global Header

The header should display:

- VIGIL logo/name;
- UAV identifier;
- engine identifier;
- current mission;
- data source;
- connection state;
- live clock;
- user/session information if enabled.

Example:

```text
VIGIL
UAV-001
AERO-PX-01
MISSION-008
LIVE ●
SIMULATED TELEMETRY
18:42:31 UTC
```

---

# 8. Data Source Indicator

The application must clearly distinguish:

```text
● LIVE
● SIMULATION
● REPLAY
● DISCONNECTED
● STALE
```

Never allow simulated data to visually masquerade as real sensor data.

Recommended header:

```text
SOURCE: SIMULATION
STATUS: CONNECTED
```

---

# 9. Status Semantics

The interface may use the following prototype health conventions:

| Health | Label |
|---|---|
| 90–100 | GOOD |
| 75–89 | DEGRADED |
| 50–74 | WARNING |
| 0–49 | CRITICAL |

These values are **UI conventions only** and must not be presented as certified engine safety limits.

Semantic colors:

- Good → green;
- Degraded → amber;
- Warning → orange;
- Critical → red;
- Neutral/inactive → gray;
- Information → blue/cyan.

Color must never be the only way a state is communicated. Include text/icons.

---

# 10. Dashboard — Primary Screen

The dashboard is the most important screen for the SIH demonstration.

## Layout

```text
┌──────────────────────────────────────────────────────────────┐
│ ENGINE HEALTH                 DIGITAL TWIN                   │
│                                                              │
│       92 / 100                     [ LARGE 3D ENGINE ]       │
│       HEALTHY                       subsystem state           │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│ LIVE TELEMETRY                                               │
│ RPM │ CHT │ EGT │ OIL P │ OIL T │ FUEL │ VIB │ ELECTRICAL   │
├──────────────────────────────┬───────────────────────────────┤
│ EXPECTED vs ACTUAL            │ AI DIAGNOSTICS               │
│                              │                              │
│ live residual chart          │ Status: NORMAL               │
│                              │ Confidence: —                │
├──────────────────────────────┼───────────────────────────────┤
│ HEALTH / DEGRADATION          │ RUL                          │
│ trend                         │ 412 h                        │
│                              │ 380–450 h                    │
├──────────────────────────────┴───────────────────────────────┤
│ MISSION STATUS / RISK                                         │
│ Mission: ENDURANCE-08     Risk: LOW      Profile: HIGH ALT. │
└──────────────────────────────────────────────────────────────┘
```

---

# 11. Dashboard Hero Area

The top section should contain:

### Left

Large engine health score:

```text
92
HEALTH
GOOD
```

Supporting information:

```text
Trend: STABLE
Last update: 250 ms ago
```

### Right

Large interactive Digital Twin.

The engine should occupy meaningful visual space.

Do not reduce the Digital Twin to a small card.

---

# 12. Engine Digital Twin Visual

The 3D engine should communicate:

- operating state;
- health state;
- subsystem state;
- detected abnormal subsystem;
- temperature/vibration emphasis where relevant.

Interaction:

- rotate;
- zoom;
- pan;
- select subsystem;
- reset camera;
- toggle labels.

Optional controls:

```text
AUTO ROTATE
SHOW LABELS
HEALTH MODE
THERMAL MODE
VIBRATION MODE
RESET VIEW
```

---

# 13. Telemetry Strip

The live telemetry strip should provide the most important values at a glance.

Example:

```text
RPM        CHT        EGT        OIL PRESSURE
2450       178°C      692°C      4.2

OIL TEMP   FUEL FLOW  VIBRATION  BATTERY
94°C       18.7       0.031      27.8V
```

Each metric should support:

- current value;
- unit;
- small trend indicator;
- status;
- timestamp/staleness indicator.

---

# 14. Telemetry Screen

Purpose:

> Deep inspection of live and historical sensor data.

Features:

- parameter selector;
- time range;
- live/historical toggle;
- chart zoom;
- pause;
- export;
- sensor status;
- expected-vs-actual overlay.

Example:

```text
TELEMETRY

[ RPM ] [ CHT ] [ EGT ] [ Oil ] [ Fuel ] [ Vibration ]

Time: LAST 5 MIN
Mode: LIVE

       actual ─────────────
       expected - - - - - -

          chart

DATA QUALITY
RPM       VALID
CHT       VALID
EGT       VALID
VIB       VALID
```

---

# 15. Engine Health Screen

Purpose:

> Show the overall condition and subsystem-level health.

Layout:

```text
ENGINE HEALTH
────────────────────────────────────

OVERALL
92 / 100
GOOD

SUBSYSTEMS

Combustion       94
Injection        87
Cooling          91
Lubrication      95
Mechanical       89
Electrical       97

────────────────────────────────────

HEALTH TREND
[ historical chart ]

DEGRADATION DRIVERS
[ ranked contributors ]
```

The screen should emphasize trends and contributors rather than only scores.

---

# 16. Diagnostics Screen

This screen becomes the focus when an anomaly is detected.

Example:

```text
ACTIVE DIAGNOSTIC
────────────────────────────────────

PROBABLE FAULT
Injector Degradation

CONFIDENCE
84%

SEVERITY
WARNING

EVIDENCE

✓ Fuel-flow residual increased
✓ EGT residual increased
✓ RPM instability increased
✓ Vibration RMS increased

ALTERNATIVE HYPOTHESES

Cooling degradation       21%
Lubrication issue         13%
Sensor drift               8%
```

The explanation must be generated from actual analytical evidence.

Do not hardcode explanatory text.

---

# 17. Diagnostic Timeline

Show when the issue developed:

```text
12:41  Normal
12:46  Residual deviation begins
12:49  Anomaly detected
12:51  Fault confidence increases
12:54  Health degradation confirmed
```

This helps judges understand that VIGIL is predictive rather than merely threshold-based.

---

# 18. Sensor-vs-Engine Diagnosis

The UI should explicitly expose this differentiator.

Example:

```text
FAULT ISOLATION

CHT SENSOR
Deviation detected
Confidence: 72%

ENGINE-WIDE CORRELATION
Low

Interpretation:
Possible sensor abnormality
```

Versus:

```text
FAULT ISOLATION

CHT
EGT
RPM
Vibration
Fuel Flow

Cross-sensor correlation:
HIGH

Interpretation:
Engine-level degradation more likely
```

---

# 19. RUL & Prognostics Screen

Purpose:

> Show how the engine condition may evolve.

Layout:

```text
REMAINING USEFUL LIFE

              412 h
         ESTIMATED RUL

       ├───────────────┤
       380             450
             interval

CONFIDENCE / UNCERTAINTY
[ visual interval ]

DEGRADATION TREND
[ chart ]

HEALTH PROJECTION
[ chart ]

END-OF-LIFE CRITERION
Prototype criterion: [defined by model]
```

The UI must make clear that RUL is a prototype estimate.

---

# 20. Mission Simulator

This is one of the strongest VIGIL demonstration screens.

## Layout

```text
MISSION SIMULATOR

CURRENT ENGINE
Health: 81
RUL: 310 h

MISSION CONFIGURATION

Altitude          [ 18,000 ft ]
Temperature       [ 12°C      ]
Throttle          [ 70%      ]
Duration          [ 8 h      ]
Load              [ -------- ]

[ RUN SIMULATION ]

────────────────────────────────────────────

PROJECTED OUTCOME

Health End       72
RUL Impact       -18 h
Thermal Margin   Reduced
Mission Risk     HIGHER
```

---

# 21. Scenario Comparison

Allow side-by-side comparison.

```text
              SCENARIO A          SCENARIO B

Altitude      18,000 ft           22,000 ft
Throttle      70%                 70%
Duration      8 h                 8 h

End Health    78                  69
RUL Impact    -8 h                -21 h
Risk          LOWER               HIGHER
```

The comparison should visually emphasize the difference.

---

# 22. Mission Replay

Replay should resemble an event timeline rather than a normal video player.

```text
MISSION REPLAY

00:00 ───── 02:00 ───── 04:00 ───── 06:00 ───── 08:00
                    ▲
                    │
             anomaly begins

[◀] [PLAY] [▶]     Speed: 1x

CURRENT STATE
Health: 83
Anomaly: ACTIVE
Fault: Injector degradation
RUL: 398 h
```

Features:

- play/pause;
- speed;
- timeline scrubbing;
- event markers;
- telemetry synchronized with playback;
- anomaly markers;
- diagnosis markers.

---

# 23. Reports Screen

Reports should provide:

- mission summary;
- engine health;
- anomalies;
- diagnoses;
- RUL;
- mission-risk summary;
- event timeline;
- model version;
- simulator/data version.

Actions:

```text
[ VIEW REPORT ]
[ EXPORT PDF ]
[ EXPORT DATA ]
```

---

# 24. Settings Screen

Recommended sections:

```text
Engine Profile
Telemetry
Data Source
Simulation
Models
Display
System
About
```

Avoid exposing engineering parameters that have not been validated.

---

# 25. Core Components

Reusable components should include:

```text
AppShell
Sidebar
TopBar
StatusBadge
HealthScore
HealthGauge
TelemetryMetric
TelemetryStrip
TrendIndicator
EngineTwin
SubsystemIndicator
TelemetryChart
ResidualChart
HealthTrendChart
RULChart
AnomalyBanner
DiagnosticPanel
EvidenceList
FaultProbability
MissionRisk
ScenarioCard
ScenarioComparison
ReplayTimeline
ConnectionStatus
DataSourceBadge
ModelVersionBadge
```

---

# 26. Health Score Component

Example:

```text
┌─────────────────────┐
│      ENGINE         │
│                     │
│       92            │
│      /100           │
│                     │
│      HEALTHY        │
│                     │
│  Trend: STABLE      │
└─────────────────────┘
```

Do not rely solely on a circular gauge. The numeric value and label must remain readable.

---

# 27. Anomaly Alert Component

Normal:

```text
● SYSTEM NORMAL
No significant abnormal behavior detected
```

Warning:

```text
▲ ANOMALY DETECTED
Injector-related deviation increasing
```

Critical:

```text
■ CRITICAL DEGRADATION
Multiple engine parameters deviating
```

Every alert should include:

- what happened;
- severity;
- timestamp;
- affected subsystem;
- action/navigation target.

---

# 28. Chart Design

Charts should support:

- dark background;
- clear axes;
- readable labels;
- hover tooltips;
- zoom;
- legend;
- expected-vs-actual distinction;
- event markers.

Important chart types:

### Telemetry

Line chart.

### Residual

Observed deviation from expected.

### Health

Time-series line.

### RUL

Projected trajectory + uncertainty band.

### Mission

Projected health/risk comparison.

---

# 29. Real-Time Animation

Real-time transitions should be smooth but restrained.

Use animation for:

- telemetry value updates;
- Digital Twin state;
- anomaly activation;
- health transitions;
- replay.

Avoid:

- constant pulsing;
- excessive particle effects;
- dramatic transitions;
- animation that reduces readability.

---

# 30. Responsive Design

Desktop is the primary target because VIGIL is a GCS.

Target:

```text
1440 × 900
1920 × 1080
```

Tablet should remain usable.

Mobile support is secondary and should prioritize:

- health;
- alerts;
- telemetry;
- mission status.

Do not attempt to reproduce the entire desktop GCS on mobile.

---

# 31. Typography

Recommended hierarchy:

```text
Page Title
22–28 px

Section Title
14–18 px

Metric
28–48 px

Telemetry Value
18–28 px

Label
10–13 px

Metadata
10–12 px
```

Use a clean technical sans-serif.

Monospaced typography may be used for:

- telemetry values;
- timestamps;
- IDs;
- system logs.

---

# 32. Iconography

Use a consistent technical icon set.

Icons should communicate:

- engine;
- telemetry;
- alert;
- diagnosis;
- mission;
- replay;
- settings;
- connection;
- health.

Icons must supplement labels, not replace important text.

---

# 33. Color System

The UI should use a restrained base palette.

Semantic colors:

```text
GOOD       → Green
DEGRADED   → Amber
WARNING    → Orange
CRITICAL   → Red
INFO       → Cyan/Blue
NEUTRAL    → Gray
```

Base interface should use dark neutral surfaces.

Do not use color solely to encode information.

---

# 34. Accessibility

Minimum requirements:

- sufficient text/background contrast;
- keyboard navigation;
- visible focus states;
- descriptive labels;
- status communicated through text/icon as well as color;
- charts with readable legends and tooltips.

---

# 35. Loading States

Do not show blank screens.

Use:

```text
CONNECTING...
LOADING ENGINE STATE...
LOADING TELEMETRY...
INITIALIZING DIGITAL TWIN...
RUNNING SIMULATION...
```

For delayed analysis:

```text
ANALYSIS IN PROGRESS
```

---

# 36. Error States

Example:

```text
TELEMETRY UNAVAILABLE

Connection to telemetry source was interrupted.

Last valid update:
18:42:31 UTC

Source:
SIMULATION

[RECONNECT]
```

Never silently display old values as live.

---

# 37. Empty States

Example:

```text
NO MISSION SELECTED

Select a mission to view
telemetry and engine intelligence.
```

Avoid generic:

```text
No data.
```

---

# 38. Data Freshness

Every live analytical view should expose freshness.

Example:

```text
LIVE ●
Updated 180 ms ago
```

Stale:

```text
STALE
Last update 14.2 s ago
```

Disconnected:

```text
DISCONNECTED
```

---

# 39. User Interaction Rules

## Rule 1

Primary information must be available without opening modal dialogs.

## Rule 2

Important alerts should be actionable.

## Rule 3

Hover should reveal detail, not hide essential information.

## Rule 4

Destructive actions require confirmation.

## Rule 5

Simulation changes must not silently modify historical data.

## Rule 6

Replay must never alter the live Digital Twin state.

---

# 40. Dashboard Demo Mode

For SIH, implement a controlled demo mode.

The demo should allow:

```text
[ START HEALTHY ]
        ↓
[ INJECT INJECTOR DEGRADATION ]
        ↓
[ OBSERVE ANOMALY ]
        ↓
[ VIEW DIAGNOSIS ]
        ↓
[ VIEW RUL CHANGE ]
        ↓
[ OPEN MISSION SIMULATOR ]
        ↓
[ CHANGE ALTITUDE ]
        ↓
[ COMPARE SCENARIOS ]
        ↓
[ REPLAY MISSION ]
```

This should be accessible without navigating through complex configuration.

---

# 41. Golden Demo Visual Sequence

The UI should visibly tell one continuous story:

### State 1

```text
HEALTH
92
GOOD

SYSTEM NORMAL
```

### State 2

```text
RESIDUAL
Increasing

ANOMALY
Detected
```

### State 3

```text
PROBABLE FAULT
Injector degradation

Confidence
84%
```

### State 4

```text
HEALTH
76
DEGRADED

RUL
412 h → 356 h
```

### State 5

```text
MISSION SIMULATION

Higher altitude
→ higher projected thermal stress
→ higher relative mission risk
```

### State 6

```text
ALTERNATIVE PROFILE

Projected health improves
Projected risk decreases
```

This sequence is the main visual narrative for judges.

---

# 42. Frontend Data Rules

The frontend must receive authoritative values from backend APIs/WebSockets.

Do not implement:

- health calculations in React;
- RUL calculations in React;
- fault probabilities in React;
- mission-risk calculations in React.

Frontend responsibilities:

```text
Receive
→ Transform for display
→ Visualize
→ Interact
```

Backend responsibilities:

```text
Validate
→ Calculate
→ Infer
→ Persist
→ Stream
```

---

# 43. Recommended Frontend State Model

```text
App State
│
├── connection
│
├── selectedUAV
│
├── selectedEngine
│
├── selectedMission
│
├── telemetry
│
├── digitalTwin
│
├── health
│
├── diagnostics
│
├── rul
│
├── mission
│
└── replay
```

Keep live server state separate from local UI state.

---

# 44. Component-to-Backend Mapping

| UI Component | Backend Source |
|---|---|
| Health Score | `/api/v1/health` |
| Telemetry Strip | WebSocket |
| Telemetry Charts | WebSocket + telemetry API |
| Digital Twin | Twin state WebSocket |
| Anomaly Banner | Diagnostic event stream |
| Fault Probability | `/api/v1/diagnostics` |
| Evidence | Diagnostic response |
| RUL | `/api/v1/rul` |
| Mission Simulation | `/api/v1/simulation` |
| Replay | `/api/v1/replay` |
| Reports | `/api/v1/reports` |

Exact API contracts are defined separately.

---

# 45. Implementation Priority — 10-Hour SIH MVP

Because the SIH demonstration is time-critical, implement UI in this order:

## P0 — Must Have

1. App shell;
2. sidebar;
3. header;
4. dashboard;
5. health score;
6. 3D engine;
7. live telemetry;
8. expected-vs-actual chart;
9. anomaly/diagnostic panel;
10. RUL panel;
11. mission-risk panel.

## P1 — Strongly Recommended

12. diagnostics page;
13. mission simulator;
14. scenario comparison;
15. replay timeline.

## P2 — Only if Time Remains

16. reports;
17. settings;
18. advanced telemetry filters;
19. advanced 3D modes;
20. mobile optimization.

---

# 46. SIH Visual Priority

If time becomes extremely limited, prioritize:

```text
        ┌───────────────────────────────┐
        │        ENGINE HEALTH          │
        │             92                │
        ├───────────────────────────────┤
        │                               │
        │       LARGE 3D ENGINE         │
        │                               │
        ├───────────────────────────────┤
        │ RPM | CHT | EGT | OIL | FUEL │
        ├────────────────┬──────────────┤
        │ EXPECTED       │ AI DIAGNOSIS │
        │ vs ACTUAL      │              │
        ├────────────────┼──────────────┤
        │ HEALTH TREND   │ RUL          │
        ├────────────────┴──────────────┤
        │        MISSION RISK           │
        └───────────────────────────────┘
```

A polished dashboard with this flow is more valuable for the SIH demo than ten unfinished screens.

---

# 47. Design System Rule

Every new component must answer:

> **Does this help the operator understand engine state, diagnose degradation, predict future condition, or evaluate mission impact?**

If not, it should not be added to the MVP.

---

# 48. Final UX Principle

VIGIL should feel like an **engine intelligence console**, not an analytics website.

The visual story is:

```text
ENGINE
  ↓
STATE
  ↓
DEVIATION
  ↓
DIAGNOSIS
  ↓
DEGRADATION
  ↓
RUL
  ↓
MISSION IMPACT
  ↓
DECISION
```

The interface should make this chain visually obvious within a single glance.

---

# 49. Implementation Checklist

- [ ] Dark aerospace GCS shell
- [ ] Persistent navigation
- [ ] UAV/engine/misson context in header
- [ ] Live/simulation/replay data-source badge
- [ ] Connection status
- [ ] Large engine Digital Twin
- [ ] Overall health
- [ ] Subsystem health
- [ ] Live telemetry strip
- [ ] Expected-vs-actual chart
- [ ] Anomaly alert
- [ ] AI diagnosis
- [ ] Evidence list
- [ ] Sensor-vs-engine isolation view
- [ ] Health trend
- [ ] RUL + uncertainty
- [ ] Mission risk
- [ ] Mission simulator
- [ ] Scenario comparison
- [ ] Mission replay
- [ ] Reports
- [ ] Loading/error/stale states
- [ ] Responsive desktop layout
- [ ] Accessible semantic states
- [ ] No unsupported engineering claims
- [ ] No fake live-data labeling
- [ ] Frontend consumes backend outputs rather than inventing analytical results

---

# 50. Final Design Statement

> **VIGIL's UI must turn complex engine telemetry and AI/physics analysis into a clear operational story: what the engine is doing, whether it is behaving as expected, what is probably wrong, how its condition is changing, how much useful life may remain, and what the current mission profile means for that condition.**

The dashboard is the hero. The Digital Twin is the visual anchor. AI explanations, RUL, and mission simulation provide the intelligence behind the visualization.
