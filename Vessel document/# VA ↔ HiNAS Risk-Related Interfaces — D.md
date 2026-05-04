# VA ↔ HiNAS Risk-Related Interfaces — Developer Spec

**Date:** 2026-04-20
**Scope:** All risk-related interfaces exchanged between Vessel Autonomy (VA) and HiNAS (Navigation Autonomy)
**Classification:** CONFIDENTIAL

---

## 1. Overview

Risk handling between VA and HiNAS spans three functional groups:

| Group                         | Purpose                                                    | Direction                  |
| ----------------------------- | ---------------------------------------------------------- | -------------------------- |
| **A. Risk Data Streams**      | Periodic risk evaluation pushed by HiNAS                   | HiNAS → VA (Receive only)  |
| **B. CAGA Control**           | Enable/disable collision & grounding avoidance autonomy    | VA ↔ HiNAS (Bidirectional) |
| **C. Risk-Triggered Outputs** | Replanned paths / indications generated as a risk response | HiNAS → VA (Receive only)  |

Common transport characteristics:

| Item                  | Value                                                        |
| --------------------- | ------------------------------------------------------------ |
| Multicast transport   | IEC 61162-450 Ed.3 **UdPbC** (NMEA) / **RaUdP** (JSON & RTZ) |
| Connection monitoring | HBT every 0.5 s; 5 s silence = LINK LOST                     |
| Encoding              | UTF-8                                                        |

---

## 2. Interface Inventory

| #   | Signal                                   | Group | Dir | Format                                | Period / Trigger        | Transport   |
| --- | ---------------------------------------- | :---: | :-: | ------------------------------------- | ----------------------- | ----------- |
| A1  | `risk_assessment_status`                 |   A   | RX  | JSON                                  | 0.5 s (2 Hz)            | RaUdP       |
| A2  | `nas:target:encounter`                   |   A   | RX  | JSON                                  | 0.5 s (2 Hz)            | RaUdP       |
| A3  | `nas:target:ksoe` † (CPA/TCPA)           |   A   | RX  | JSON                                  | 0.5 s (2 Hz)            | RaUdP       |
| B1  | `$PKSOCAGM`                              |   B   | TX  | NMEA (Proprietary)                    | On operator command     | UdPbC       |
| B2  | `$PAVKCAGM`                              |   B   | RX  | NMEA (Proprietary)                    | On `$PKSOCAGM` received | UdPbC       |
| B3  | `$PAVKHOMS` (collision_avoidance_status) |   B   | RX  | NMEA (Proprietary)                    | 1.0 s (1 Hz)            | UdPbC (TBD) |
| C1  | `local_path_plan` (LPP RTZ)              |   C   | RX  | Extended RTZ XML                      | On replanning           | RaUdP       |
| C2  | Indication `eventLevel`                  |   C   | RX  | JSON (within `nas_indication_schema`) | On INDR / state change  | RaUdP       |

> **† JSON signal key names (`nas:target:ksoe`, `nas:target:encounter`) are pending joint agreement with HiNAS (Avikus). The corresponding fields (`nas:target:ksoe.id`, etc.) throughout this document assume that renaming will be adopted.**

---

## 3. Group A — Risk Data Streams (HiNAS → VA)

### 3.1 `risk_assessment_status`

Path-level and own-ship-level risk evaluation. Single JSON object, 2 Hz.

| Field                              | Type    | Unit         | Nullable | Description                                        |
| ---------------------------------- | ------- | ------------ | :------: | -------------------------------------------------- |
| `issued_time`                      | string  | ISO 8601 UTC |    N     | Evaluation timestamp                               |
| `is_ship_risk_position`            | boolean | --           |    N     | `true` if own ship is in a predicted risk position |
| `own_ship_risk_position_latitude`  | double  | deg          |    Y     | Predicted own-ship risk latitude                   |
| `own_ship_risk_position_longitude` | double  | deg          |    Y     | Predicted own-ship risk longitude                  |
| `is_global_path_risk`              | boolean | --           |    N     | `true` if Global Path Plan is at risk              |
| `global_path_tcpa_s`               | double  | s            |    Y     | TCPA on global path                                |
| `global_path_dcpa_nm`              | double  | NM           |    Y     | DCPA on global path                                |
| `global_path_risk_reason`          | string  | --           |    Y     | GPP risk reason (see 3.1.1)                        |
| `is_local_path_risk`               | boolean | --           |    N     | `true` if Local Path Plan is at risk               |
| `local_path_tcpa_s`                | double  | s            |    Y     | TCPA on local path                                 |
| `local_path_dcpa_nm`               | double  | NM           |    Y     | DCPA on local path                                 |
| `local_path_risk_reason`           | string  | --           |    Y     | LPP risk reason (see 3.1.1)                        |

#### 3.1.1 `risk_reason` enum

| Value      | Meaning                                 |
| ---------- | --------------------------------------- |
| `"GROUND"` | Grounding risk (bathymetry / coastline) |
| `"TARGET"` | Collision risk with a tracked target    |
| `"DEPTH"`  | Insufficient under-keel clearance       |

#### 3.1.2 Null rules

- `is_ship_risk_position == false` → `own_ship_risk_position_*` = `null`
- `is_global_path_risk == false` → all `global_path_*` fields = `null`
- `is_local_path_risk == false` → all `local_path_*` fields = `null`

#### 3.1.3 Example — LPP collision risk

```json
{
  "issued_time": "2026-04-20T03:12:46.000Z",
  "is_ship_risk_position": true,
  "own_ship_risk_position_latitude": 35.1042,
  "own_ship_risk_position_longitude": 129.0765,
  "is_global_path_risk": false,
  "global_path_tcpa_s": null,
  "global_path_dcpa_nm": null,
  "global_path_risk_reason": null,
  "is_local_path_risk": true,
  "local_path_tcpa_s": 87.4,
  "local_path_dcpa_nm": 0.12,
  "local_path_risk_reason": "TARGET"
}
```

### 3.2 `nas:target:encounter`

Per-target encounter evaluation (CPA/TCPA, COLREGs situation classification, risk level). Keyed by target UUID aligned with `nas:target:ksoe.id`. 2 Hz.

> **Detailed field schema is TBD in HiNAS ICD Rev. 1.2.1.** Implement a generic JSON listener keyed by target UUID and defer field binding until the schema is finalized.

### 3.3 `nas:target:ksoe` — CPA / TCPA subset

Full fused-target list (sensor fusion of Radar / AIS / EO / IR). Only risk-relevant fields are listed here. 2 Hz.

| Field         | Type          | Unit | Description                    |
| ------------- | ------------- | ---- | ------------------------------ |
| `id`          | string        | --   | Target UUID                    |
| `latitude`    | double        | deg  | Target latitude                |
| `longitude`   | double        | deg  | Target longitude               |
| `cog_deg`     | double        | deg  | Course Over Ground (0.0-359.9) |
| `sog_kn`      | double        | kn   | Speed Over Ground              |
| `distance_nm` | double / null | NM   | Target distance                |
| `bearing_deg` | double / null | deg  | Target bearing                 |
| `cpa_nm`      | double        | NM   | Closest Point of Approach      |
| `tcpa_s`      | double        | s    | Time to CPA                    |

Use `cpa_nm` / `tcpa_s` as the raw proximity metric per target; use `nas:target:encounter` for COLREGs-aware classification and per-target risk level.

---

## 4. Group B — CAGA Control (Bidirectional)

CAGA (Collision Avoidance and Grounding Avoidance) is HiNAS's autonomous risk-response mode. VA toggles CAGA; HiNAS acknowledges and reports operational status.

### 4.1 `$PKSOCAGM` — CAGA Mode Command (VA → HiNAS)

| Item      | Value                                |
| --------- | ------------------------------------ |
| Talker ID | `PKSO`                               |
| Sentence  | Proprietary CAGM                     |
| Trigger   | On operator command                  |
| Transport | UdPbC                                |
| Purpose   | Enable / disable CAGA operating mode |

> Command field layout (enable / disable token) is **TBD** — align with the agreed PKSOCAGM / PAVKCAGM field definition before implementation.

### 4.2 `$PAVKCAGM` — CAGA Mode Response (HiNAS → VA)

| Item      | Value                                               |
| --------- | --------------------------------------------------- |
| Talker ID | `PAVK`                                              |
| Sentence  | Proprietary CAGM                                    |
| Trigger   | On `$PKSOCAGM` received                             |
| Transport | UdPbC                                               |
| Payload   | accept / reject response echoing the requested mode |

### 4.3 `$PAVKHOMS` — Operation Mode Status (HiNAS → VA)

Periodic readiness report. The `collision_avoidance_status` field is the ground-truth CAGA state.

Format:

```
$PAVKHOMS,{track_control_status},{collision_avoidance_status},{radar_mode},{ptz_mode}*hh
```

| Field                        | Values          | Meaning                               |
| ---------------------------- | --------------- | ------------------------------------- |
| `track_control_status`       | `R` / `N` / `A` | Ready / Not Ready / Activate          |
| `collision_avoidance_status` | `R` / `N` / `A` | Ready / Not Ready / Activate          |
| `radar_mode`                 | `B` / `A` / `T` | Bypass / Auto ARPA / Target Selection |
| `ptz_mode`                   | `M` / `A`       | Manual / Auto                         |

| Item      | Value            |
| --------- | ---------------- |
| Period    | 1.0 s (1 Hz)     |
| Transport | UdPbC (port TBD) |

### 4.4 CAGA Control Semantics

- CAGA is toggled independently of the Mission RTZ flow.
- When CAGA is **Active** (`A`), HiNAS autonomously issues LPP updates (Group C1) on collision / grounding risk without changing the mission `routeStatus`.
- VA must treat `$PAVKCAGM` as transactional ack and `$PAVKHOMS.collision_avoidance_status` as the authoritative steady-state.
- `routeStatus = CollisionAvoidance` was **removed** in HiNAS Rev. 1.3.0. Do **not** expect it.

---

## 5. Group C — Risk-Triggered Outputs (HiNAS → VA)

### 5.1 `local_path_plan` (LPP RTZ)

Real-time replanned path issued by HiNAS when CAGA is active and a risk is detected (or cleared).

| Item          | Value                                                          |
| ------------- | -------------------------------------------------------------- |
| Format        | Extended RTZ 1.2 XML (vendor-extended)                         |
| Trigger       | On replanning event (risk detected / resolved)                 |
| Transport     | RaUdP                                                          |
| `routeName`   | `{response-id}*{lpp-id}`                                       |
| `routeAuthor` | `"Avikus"`                                                     |
| `routeStatus` | e.g., `RouteFollowing*LPP`, `VectorControl*LPP`, `StandBy*LPP` |

VA shall consume the LPP RTZ and:

- Update the active path view.
- Correlate with the concurrent `risk_assessment_status` frame (same wall-clock window) for operator context.
- Continue to monitor `$PAVKHOMS.collision_avoidance_status` for CAGA steady state.

### 5.2 Indication `eventLevel` (Risk Level)

Indication items flowing via `nas_indication_schema` (RaUdP JSON, on INDR / on state change) carry an `eventLevel` field usable as a risk severity weighting.

| Field        | Type          | Description                          |
| ------------ | ------------- | ------------------------------------ |
| `eventLevel` | integer (1–4) | Risk level (1 = lowest, 4 = highest) |

Map `eventLevel` to the VA alert severity tiering as part of Alert Management integration.

---

## 6. VA Implementation Responsibilities

### 6.1 Receivers

1. **`risk_assessment_status` parser (A1)** — Join RaUdP multicast group; parse all 12 fields at 2 Hz; honour null rules from §3.1.2.
2. **`nas:target:encounter` parser (A2)** — Generic JSON listener keyed by target UUID; defer field binding until HiNAS schema finalized.
3. **`nas:target:ksoe` parser (A3)** — Extract at minimum `id`, position, `cpa_nm`, `tcpa_s` for the risk pipeline.
4. **`$PAVKCAGM` listener (B2)** — Match to the outstanding `$PKSOCAGM` request; surface accept / reject to the operator.
5. **`$PAVKHOMS` listener (B3)** — Treat `collision_avoidance_status` as the authoritative CAGA state (not the last `$PAVKCAGM`).
6. **LPP RTZ listener (C1)** — Replace the active local path; preserve GPP.
7. **Indication listener (C2)** — Propagate `eventLevel` into alert severity.

### 6.2 Transmitter

- **`$PKSOCAGM` (B1)** — Send on operator command only. Idempotent retransmission is disallowed unless `$PAVKCAGM` is not received within a timeout (default 2 s).

### 6.3 State & Alerting

| Event                                                     | VA Action                                                                             |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Any `is_*_risk` flag transitions `false → true`           | Raise alert (priority per Group C2 `eventLevel` when available), highlight in UI, log |
| Any `is_*_risk` flag transitions `true → false`           | Clear alert, return UI to nominal                                                     |
| `$PAVKHOMS.collision_avoidance_status` transitions to `A` | Display "CAGA ACTIVE"; accept LPP replans                                             |
| `$PAVKHOMS.collision_avoidance_status` transitions to `N` | Raise "CAGA NOT READY" advisory                                                       |
| `$PAVKCAGM` rejection                                     | Surface reason to operator; revert UI toggle                                          |
| No `risk_assessment_status` for ≥ 5 s                     | Mark stream stale; chain into LINK LOST                                               |
| No `$PAVKHOMS` for ≥ 5 s                                  | Mark CAGA state unknown; chain into LINK LOST                                         |
| JSON / NMEA parse failure                                 | Discard frame, retain previous state, increment error counter                         |

### 6.4 Correlation Rules

- Correlate `risk_assessment_status.*_risk_reason = "TARGET"` with `nas:target:encounter` and `nas:target:ksoe` (by target UUID) to identify the offending target.
- Correlate LPP RTZ issuance with the most recent `risk_assessment_status` frame within ±1 s for operator situational context.
- Do **not** rely on `routeStatus = CollisionAvoidance` (removed). The authoritative signal is `$PAVKHOMS.collision_avoidance_status`.
