# VA ↔ HiNAS PTZ Control and Tracking — Developer Spec

**Date:** 2026-04-20
**Scope:** All PTZ (Pan-Tilt-Zoom camera) interfaces exchanged between Vessel Autonomy (VA) and HiNAS (Navigation Autonomy)
**Classification:** CONFIDENTIAL

> **⚠ Perception Interface Instability (HiNAS Rev. 1.2.2+):** Perception-related requirements — including the entire PTZ Control and Tracking interface (`$PKSOPTZM` / `$PAVKPTZM`, `$PKSOPTZT` / `$PAVKPTZT`, `$PKSOPTZC` / `$PAVKPTZC`, PTZ Upstream Event, `nas:target:ptz`, and PTZ Indication mode) — are being restructured and may undergo a complete redesign. Do **not** treat these definitions as stable for long-term implementation; integration work should be gated behind an abstraction layer.

---

## 1. Overview

The PTZ channel lets VA command the PTZ camera subsystem on HiNAS and receive status, classification, and upstream-event data. Control follows a **command-echo pattern**: VA sends a proprietary NMEA command; HiNAS echoes back the commanded values (not the physical position) as a receipt acknowledgement.

Functional groups:

| Group                          | Purpose                                                       | Direction  |
| ------------------------------ | ------------------------------------------------------------- | ---------- |
| **A. PTZ Command & Echo**      | Mode / target tracking / manual pan-tilt-zoom                 | VA ↔ HiNAS |
| **B. PTZ Upstream Event**      | HiNAS reports autonomous physical PTZ command                 | HiNAS → VA |
| **C. Status & Classification** | PTZ mode in HOMS, per-target classification, indication state | HiNAS → VA |

Common transport characteristics:

| Item                  | Value                                        |
| --------------------- | -------------------------------------------- |
| NMEA transport        | IEC 61162-450 Ed.3 **UdPbC** (UDP multicast) |
| JSON transport        | IEC 61162-450 Ed.3 **RaUdP**                 |
| Encoding              | UTF-8 (NMEA ASCII; JSON raw UTF-8)           |
| Connection monitoring | HBT every 0.5 s; 5 s silence = LINK LOST     |

---

## 2. Interface Inventory

| #   | Signal                       | Group | Dir | Format                                                     | Period / Trigger                   | Transport     |
| --- | ---------------------------- | :---: | :-: | ---------------------------------------------------------- | ---------------------------------- | ------------- |
| A1  | `$PKSOPTZM`                  |   A   | TX  | NMEA (Proprietary PTZM)                                    | On operator command                | UdPbC         |
| A2  | `$PAVKPTZM`                  |   A   | RX  | NMEA (Proprietary PTZM)                                    | On `$PKSOPTZM` received            | UdPbC         |
| A3  | `$PKSOPTZT`                  |   A   | TX  | NMEA (Proprietary PTZT)                                    | On operator command                | UdPbC         |
| A4  | `$PAVKPTZT`                  |   A   | RX  | NMEA (Proprietary PTZT)                                    | On `$PKSOPTZT` received            | UdPbC         |
| A5  | `$PKSOPTZC`                  |   A   | TX  | NMEA (Proprietary PTZC)                                    | On operator command                | UdPbC         |
| A6  | `$PAVKPTZC`                  |   A   | RX  | NMEA (Proprietary PTZC)                                    | On `$PKSOPTZC` received            | UdPbC         |
| B1  | PTZ Upstream Event           |   B   | RX  | TBD                                                        | On autonomous physical PTZ command | TBD           |
| C1  | `$PAVKHOMS` (ptz_mode)       |   C   | RX  | NMEA (Proprietary HOMS)                                    | 1.0 s (1 Hz)                       | UdPbC (TBD)   |
| C2  | `nas:target:ptz`             |   C   | RX  | JSON                                                       | On target update                   | RaUdP         |
| C3  | Indication PTZ mode (`4xxx`) |   C   | RX  | JSON / NMEA (within `nas_indication_schema` / `$PAVKINDD`) | On INDR / state change             | RaUdP / UdPbC |

---

## 3. Group A — PTZ Command & Echo

### 3.1 PTZ Mode — `$PKSOPTZM` / `$PAVKPTZM`

Switches PTZ camera between Manual and Auto operation.

**Sentence format (both directions):**

```
$<talker>PTZM,{ptz_mode},{status}*hh
```

| Field      | Values    | Description              |
| ---------- | --------- | ------------------------ |
| `ptz_mode` | `M` / `A` | `M` = Manual, `A` = Auto |
| `status`   | A / V     | A = Valid, V = Invalid   |

**Mode behaviour:**

| Mode         | Behaviour                                                                    |
| ------------ | ---------------------------------------------------------------------------- |
| Auto (`A`)   | HiNAS scans surroundings and sequentially tracks targets autonomously        |
| Manual (`M`) | VA directly commands pan / tilt / zoom via `$PKSOPTZC`; HiNAS echoes receipt |

### 3.2 PTZ Target Tracking — `$PKSOPTZT` / `$PAVKPTZT`

Assigns a target UUID for PTZ to follow. Valid only while PTZ is in Auto (or an agreed tracking sub-mode).

**Command (VA → HiNAS):**

```
$PKSOPTZT,{target_id},{status}*hh
```

**Echo (HiNAS → VA):**

```
$PAVKPTZT,{target_id},{error_string},{status}*hh
```

| Field          | Values                | Description                                                                                                      |
| -------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `target_id`    | UUID string           | Target identifier (matches `nas:target:ptz.id`)                                                                  |
| `error_string` | string (max 20 chars) | Error description when tracking fails. Examples: `"PTZ HW Connection Lo"`, `"Invalid target"`. Empty on success. |
| `status`       | A / V                 | A = Valid, V = Invalid                                                                                           |

### 3.3 PTZ Manual Control — `$PKSOPTZC` / `$PAVKPTZC`

Commands pan / tilt / zoom values. Valid only while PTZ is in Manual mode.

**Command (VA → HiNAS):**

```
$PKSOPTZC,{panning},{tilting},{eo_zoom},{ir_zoom},{status}*hh
```

**Echo (HiNAS → VA):**

```
$PAVKPTZC,{panning},{tilting},{eo_zoom},{ir_zoom},{error_string},{status}*hh
```

| Field          | Range                 | Unit | Description                                             |
| -------------- | --------------------- | ---- | ------------------------------------------------------- |
| `panning`      | -180 to +180          | deg  | Pan angle                                               |
| `tilting`      | -90 to +90            | deg  | Tilt angle                                              |
| `eo_zoom`      | 1.0 to 300.0          | x    | Electro-Optical (visible) zoom factor                   |
| `ir_zoom`      | 1.0 to 119.2          | x    | Infrared (thermal) zoom factor                          |
| `error_string` | string (max 20 chars) | --   | Error description when control fails. Empty on success. |
| `status`       | A / V                 | --   | A = Valid, V = Invalid                                  |

> **Echo semantics:** `$PAVKPTZC` echoes the **commanded** values, not the physical camera position. Treat the echo as a receipt acknowledgement only. Do not render it as live camera pose.

### 3.4 Command Timeout and Retry (applies to A1 / A3 / A5)

| Item                | Value                                                                   |
| ------------------- | ----------------------------------------------------------------------- |
| Echo wait timeout   | **1.0 s**                                                               |
| Retry count         | up to **3 retransmissions**                                             |
| Failure declaration | after 3 failed attempts, mark command as failed and surface to operator |

---

## 4. Group B — PTZ Upstream Event (HiNAS → VA)

HiNAS autonomously issues a physical PTZ command (e.g., operator-of-last-resort hardware intervention). When such an event occurs, HiNAS pushes an event report upstream.

| Item      | Value                              |
| --------- | ---------------------------------- |
| Direction | HiNAS → VA                         |
| Format    | **TBD**                            |
| Trigger   | On autonomous physical PTZ command |
| Transport | **TBD**                            |

> **TBD**: Payload schema and transport are pending agreement. Implement the upstream event listener as a stub and surface raw events to the operator once the format is finalised.

---

## 5. Group C — Status & Classification (HiNAS → VA)

### 5.1 `$PAVKHOMS` — `ptz_mode` Field

Periodic operation mode status includes the current PTZ mode.

**Format:**

```
$PAVKHOMS,{track_control_status},{collision_avoidance_status},{radar_mode},{ptz_mode}*hh
```

| Field      | Values    | Description              |
| ---------- | --------- | ------------------------ |
| `ptz_mode` | `M` / `A` | `M` = Manual, `A` = Auto |

| Item      | Value            |
| --------- | ---------------- |
| Period    | 1.0 s (1 Hz)     |
| Transport | UdPbC (port TBD) |

Treat `$PAVKHOMS.ptz_mode` as the **authoritative** steady-state — not the last `$PAVKPTZM` echo (which reflects only the acknowledgement of the latest command).

### 5.2 `nas:target:ptz` — PTZ Classification per Target

Per-target classification result from EO (optical) and IR (thermal) PTZ cameras. Keyed by target UUID aligned with `nas:target:ksoe.id` †.

| Field               | Type       | Description                                  |
| ------------------- | ---------- | -------------------------------------------- |
| `id`                | string     | Target UUID (matches `nas:target:ksoe.id` †) |
| `class_type_ptz_eo` | int / null | Class type from PTZ EO camera                |
| `class_type_ptz_ir` | int / null | Class type from PTZ IR camera                |

| Item      | Value            |
| --------- | ---------------- |
| Trigger   | On target update |
| Transport | RaUdP            |

> **† Target UUID key alignment is pending joint agreement with HiNAS (Avikus).** The rename from `nas:target:anduril` to `nas:target:ksoe` is assumed throughout this document.

### 5.3 Indication PTZ Mode (indication_id range `4xxx`)

Indication schema items with `indication_id` in the `4000–4999` range are PTZ-specific state items, received via `nas_indication_schema` (initial / on INDR) and `$PAVKINDD` (periodic / on state change).

| State Code | Meaning                        |
| ---------- | ------------------------------ |
| `N`        | Normal                         |
| `D`        | Degraded (replaces legacy `L`) |
| `F`        | Fault                          |

Map PTZ indication items to the VA operator UI alongside the PTZ mode display.

---

## 6. VA Implementation Responsibilities

### 6.1 Transmitters (VA → HiNAS)

1. **`$PKSOPTZM` (A1)** — Send on operator mode-switch command only. Retry per §3.4.
2. **`$PKSOPTZT` (A3)** — Send with the operator-selected target UUID. Valid only when PTZ is Auto or tracking sub-mode. Retry per §3.4.
3. **`$PKSOPTZC` (A5)** — Send on manual joystick / UI input. Valid only when PTZ is Manual. Retry per §3.4. Rate-limit to the agreed UI cadence to avoid flooding HiNAS.

### 6.2 Receivers (HiNAS → VA)

1. **`$PAVKPTZM` / `$PAVKPTZT` / `$PAVKPTZC` echo (A2 / A4 / A6)** — Match to the outstanding command by talker context; clear retry counter on valid echo; surface `error_string` to the operator when non-empty.
2. **PTZ Upstream Event (B1)** — Stub listener; preserve raw payload for review once format is finalised.
3. **`$PAVKHOMS` (C1)** — Extract `ptz_mode`; treat as authoritative steady-state.
4. **`nas:target:ptz` (C2)** — Merge `class_type_ptz_eo` / `class_type_ptz_ir` into the per-target state keyed by UUID.
5. **Indication `4xxx` (C3)** — Route to the operator indication panel with the PTZ mode grouping.

### 6.3 State & Alerting

| Event                                                    | VA Action                                                                  |
| -------------------------------------------------------- | -------------------------------------------------------------------------- |
| Command issued                                           | Start 1.0 s echo timer; mark command pending                               |
| Echo received within timeout                             | Clear pending state; reset retry counter                                   |
| Echo not received within 1.0 s                           | Retransmit (up to 3 times)                                                 |
| 3 retries exhausted                                      | Declare command failed; alert operator                                     |
| `$PAVKPTZT` echo `error_string = "Invalid target"`       | Revert target-tracking UI; alert operator                                  |
| `$PAVKPTZT` echo `error_string = "PTZ HW Connection Lo"` | Raise hardware-connection advisory; inhibit further commands until cleared |
| `$PAVKHOMS.ptz_mode` transition `A → M` or `M → A`       | Update PTZ mode badge in UI; log event                                     |
| 5 s silence on `$PAVKHOMS`                               | Mark PTZ mode unknown; chain into LINK LOST                                |
| No `nas:target:ptz` update for a tracked target          | Retain last classification; do not assume tracking failure                 |

### 6.4 Correlation Rules

- Correlate `nas:target:ptz.id` with `nas:target:ksoe.id` † to enrich the target record with EO/IR classification.
- Do **not** use `$PAVKPTZC` echo values for camera pose rendering; they are receipt acknowledgements only.
- Treat `$PAVKHOMS.ptz_mode` as the single source of truth for PTZ mode; `$PAVKPTZM` echo is a transactional acknowledgement only.
- Indication `4xxx` state (`N` / `D` / `F`) takes precedence over `$PAVKHOMS.ptz_mode` when determining operator-visible health status.
