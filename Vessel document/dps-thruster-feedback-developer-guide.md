# DPS Thruster Feedback Developer Guide

> ICD Reference: VA Alpha ICD Draft v0.3 (VA-ICD-0001, 2026-03-31)
>
> This document describes the DPS Thruster Feedback data flow from the Dynamic Positioning system through Anduril VMS and VA to M&A, including the `$MTFBA` NMEA sentence format, related M&A signals (DiagnosticResult, OperationalAdvisory), and implementation guidance.
>
> **Surrogate Vessel demonstration only** -- this interface is not part of the standard Alpha VA-M&A interface.

---

## Table of Contents

1. [Data Flow Overview](#1-data-flow-overview)
2. [Transport Specification](#2-transport-specification)
3. [DPS Thruster Feedback ($MTFBA)](#3-dps-thruster-feedback-mtfba)
4. [Related M&A Signals](#4-related-ma-signals)
   - 4.1 [OperationalAdvisory (M&A -> VA)](#41-operationaladvisory-ma---va)
   - 4.2 [OperationalAdvisoryResponse (VA -> M&A)](#42-operationaladvisoryresponse-va---ma)
   - 4.3 [DiagnosticResult (M&A -> VA)](#43-diagnosticresult-ma---va)
5. [Operational Advisory Execution Flow](#5-operational-advisory-execution-flow)
6. [Considerations](#6-considerations)
7. [Reference Documents](#7-reference-documents)

---

## 1. Data Flow Overview

```
┌──────────┐      ┌──────────────┐      ┌──────┐      ┌──────┐
│DP System │─UDP─>│ Anduril VMS  │─UDP─>│  VA  │─UDP─>│ M&A  │
│  (DPS)   │      │  (HiCONiS)   │      │      │      │      │
└──────────┘      └──────────────┘      └──────┘      └──────┘
   Thruster          NMEA relay          Relay           Display /
   hardware          (pass-through)      (pass-through)  Monitoring
```

- **DP System**: Dynamic Positioning system generating real-time thruster feedback
- **Anduril VMS (HiCONiS)**: Vessel Management System relaying DP data via NMEA 0183 over UDP
- **VA**: Vessel Autonomy -- passes the `$MTFBA` sentence through to M&A without modification
- **M&A**: Mission Autonomy -- receives and displays thruster data for operator situational awareness

### Key Characteristics

- **Relay mode**: VA acts as a pass-through relay; the NMEA sentence is forwarded unchanged
- **Unidirectional**: DP -> VMS -> VA -> M&A (no return path for thruster feedback)
- **Real-time**: This is the **only** real-time propulsion-related data available from the DP system
- **Limitations**: Does not include engine RPM, load, or torque -- only relative thruster parameters

---

## 2. Transport Specification

### DPS Thruster Feedback (DP -> VMS -> VA -> M&A)

| Item | Value |
|---|---|
| Standard | NMEA 0183 (IEC 61162-1) |
| Network layer | IPv4 UDP |
| Transport | UDP unicast (DP -> VMS -> VA), UDP Multicast (VA -> M&A, proposed) |
| UDP port | TBD |
| Data rate | ~1 Hz |
| Payload format | NMEA 0183 proprietary sentence, ASCII |
| Sentence | `$MTFBA` (Message 1100 -- Thruster Feedback) |

### Talker ID Convention

| Talker ID | System | Description |
|---|---|---|
| `PADR` | Anduril | Proprietary talker ID for Anduril VMS |
| `PAVK` | Avikus | Proprietary talker ID for Avikus (VA) |

> Talker IDs use the NMEA proprietary sentence prefix `P` followed by a system-specific identifier (`ADR` = Anduril, `AVK` = Avikus).

### VA -> M&A Transport (Proposed)

The proposed VA -> M&A communication architecture uses UDP Multicast (IEC 61162-450 Ed.3), consistent with the Navigation Autonomy interface. DPS Thruster Feedback is categorized as **Channel A -- Periodic / Status Data (UdPbC)**.

| Item | Proposed Value | Status |
|---|---|---|
| Transport | IPv4 UDP Multicast | Proposed (TBD) |
| Standard | IEC 61162-450 Ed.3 | Proposed (TBD) |
| Multicast address | Separate group from Nav. Autonomy range | TBD |
| UDP port | TBD | TBD |
| TTL | 1 (link-local) | Proposed (TBD) |
| Protocol | UdPbC (unidirectional, no ACK) | Proposed |

---

## 3. DPS Thruster Feedback ($MTFBA)

### Sentence Format

```
$MTFBA,<MsgID>,<DevID1>,<RPM1>,<Pitch1>,<Angle1>,<DevID2>,<RPM2>,<Pitch2>,<Angle2>,...*<CS>
```

The sentence contains a variable number of thruster data groups. Each group consists of 4 fields: `DevID`, `RPM`, `Pitch`, `Angle`.

### Field Definitions

| Field | Type | Range | Description |
|---|---|---|---|
| `MsgID` | Integer | `1100` (fixed) | Message identifier (Thruster Feedback) |
| `DevID` | Integer | 1 -- 12 | Thruster device identifier (see Device ID Mapping) |
| `RPM` | Float | 0.0 -- 1.0 | Relative RPM feedback (0% -- 100%) |
| `Pitch` | Float | 0.0 -- 1.0 | Relative pitch feedback (waterjet: reversing bucket position) |
| `Angle` | Float | 0 -- 360 | Thruster azimuth angle in degrees |
| `CS` | Hex | 00 -- FF | NMEA checksum (XOR of all characters between `$` and `*`) |

### Device ID Mapping

| DeviceID | Equipment | Type |
|---|---|---|
| 1 | Bow Thruster 1 | Bow Thruster |
| 2 | Bow Thruster 2 | Bow Thruster |
| 3 | Waterjet 1 | Waterjet |
| 4 | Waterjet 2 | Waterjet |
| 5 | Waterjet 3 | Waterjet |
| 6 | Waterjet 4 | Waterjet |
| 7 -- 12 | Reserved | -- |

### Example

```
$MTFBA,1100,1,0.5,0.5,180,2,0.5,0.5,0,3,0.5,0.5,0*79
```

Decoded:

| Thruster | DevID | RPM (%) | Pitch (%) | Angle (deg) |
|---|---|---|---|---|
| Bow Thruster 1 | 1 | 50% | 50% | 180 |
| Bow Thruster 2 | 2 | 50% | 50% | 0 |
| Waterjet 1 | 3 | 50% | 50% | 0 |

### Field Interpretation

- **RPM**: Normalized value (0.0 = 0%, 1.0 = 100%). This is relative to maximum rated RPM, not absolute RPM.
- **Pitch**: For waterjets, this represents the reversing bucket position. 0.0 = full reverse, 0.5 = neutral, 1.0 = full ahead (interpretation may vary by installation).
- **Angle**: Azimuth angle of the thruster in degrees (0 = forward, 180 = aft). Fixed for non-azimuth thrusters.

---

## 4. Related M&A Signals

The following signals are exchanged between M&A and VA alongside thruster feedback, forming the complete Machinery Autonomy status and advisory loop.

### 4.1 OperationalAdvisory (M&A -> VA)

Generated when M&A detects a machinery issue requiring vessel-level action. Contains recommended action options for the operator.

| Item | Value |
|---|---|
| Direction | M&A -> VA (uplink) |
| Transport | REST API (HTTP POST) |
| Endpoint (proposed) | `POST /va/api/v1/ma/operational-advisory` |
| Trigger | On machinery issue detected |

#### JSON Schema

```json
{
    "metadata": {
        "request_id": "8a1a5d44-1a56-4c7f-9b6b-2f7b0b8a5f90",
        "timestamp": "2026-03-04T13:20:00Z",
        "ship_id": "IM0-9876543",
        "version": "v1.0"
    },
    "machinery_status": {
        "overall_status": "WARNING",
        "detected_issues": [
            {
                "machine_name": "ME1",
                "severity": "WARNING",
                "alarms": [
                    {"message": "High exhaust temperature", "duration_sec": 120}
                ]
            }
        ]
    },
    "operational_advisory": {
        "reason_summary": "ME1 exhaust temperature exceeding threshold",
        "options": [
            {
                "type": "SHIP_SPEED_REDUCTION",
                "is_recommended": true,
                "cause_machines": ["ME1"],
                "description": "Reduce speed by 30%"
            },
            {
                "type": "REDUCED_POWER_RTB",
                "is_recommended": false,
                "cause_machines": [],
                "description": "Return to base"
            }
        ]
    }
}
```

#### Field Definitions -- metadata

| Field | Type | Description |
|---|---|---|
| `request_id` | String (UUID v4) | Unique request identifier |
| `timestamp` | String (ISO 8601) | Advisory generation timestamp |
| `ship_id` | String | Ship identifier (e.g., IMO number) |
| `version` | String | API version |

#### Field Definitions -- machinery_status

| Field | Type | Description |
|---|---|---|
| `overall_status` | Enum | `NORMAL` / `CAUTION` / `WARNING` / `DANGER` |
| `detected_issues` | Array | List of detected equipment issues |

#### detected_issues (per equipment)

| Field | Type | Description |
|---|---|---|
| `machine_name` | String | Equipment identifier (e.g., `ME1`, `GE2`) |
| `severity` | Enum | `CAUTION` / `WARNING` / `DANGER` |
| `alarms` | Array | Active alarms: `message` (String), `duration_sec` (Integer) |

#### Field Definitions -- operational_advisory.options

| Field | Type | Description |
|---|---|---|
| `type` | Enum | `SHIP_SPEED_REDUCTION` / `SHIP_STOP` / `REDUCED_POWER_RTB` |
| `is_recommended` | Boolean | Whether this is the recommended action |
| `cause_machines` | Array | Equipment IDs causing this advisory (e.g., `["ME1", "ME2"]`) |
| `description` | String | Detailed description of the action |

#### Advisory Option Types

| Option Type | Description |
|---|---|
| `SHIP_SPEED_REDUCTION` | Reduce vessel speed (expressed as reduction ratio) |
| `SHIP_STOP` | Emergency stop |
| `REDUCED_POWER_RTB` | Return to base with reduced power |

### 4.2 OperationalAdvisoryResponse (VA -> M&A)

Response to an OperationalAdvisory. VA sends accept (0) or reject (1) based on MA operator decision.

| Item | Value |
|---|---|
| Direction | VA -> M&A (downlink) |
| Transport | REST API (HTTP POST) |
| Endpoint (proposed) | `POST /ma/api/v1/advisory-response` |
| Trigger | On operator accept/reject decision via MA |

#### JSON Schema

```json
{
    "advisoryId": 1001,
    "response": 0,
    "timestamp": "2026-01-14T10:05:00Z"
}
```

| Field | Type | Description |
|---|---|---|
| `advisoryId` | Integer | Reference to the original OperationalAdvisory |
| `response` | Integer | `0` = Accept, `1` = Reject |
| `timestamp` | String (ISO 8601) | Response timestamp |

### 4.3 DiagnosticResult (M&A -> VA)

Periodic machinery diagnostic results per equipment. Sent at configurable intervals (default ~10s).

| Item | Value |
|---|---|
| Direction | M&A -> VA (uplink) |
| Transport | REST API (HTTP POST) |
| Endpoint (proposed) | `POST /va/api/v1/ma/diagnostic-result` |
| Period | Configurable (~10s default) |

#### JSON Schema

```json
{
    "save_timestamp": "2026-01-14T10:30:00Z",
    "save_interval_seconds": 10,
    "equipment_count": 4,
    "operation_mode": "GT",
    "results": {
        "ME1": {
            "PRIMARY": {
                "timestamp": "2026-01-14T10:29:30Z",
                "status": "RUNNING",
                "result": "NORMAL",
                "score": 95,
                "updated_by": "model_v2"
            },
            "SECONDARY": {
                "Intake": {"result": "NORMAL", "score": 98, "detail": {}},
                "Exhaust": {"result": "NORMAL", "score": 92, "detail": {}}
            }
        },
        "ME2": {"..."},
        "ME3": {"..."},
        "ME4": {"..."}
    }
}
```

#### Field Definitions -- Root

| Field | Type | Description |
|---|---|---|
| `save_timestamp` | String (ISO 8601) | Diagnostic execution timestamp |
| `save_interval_seconds` | Integer | Diagnostic interval in seconds |
| `equipment_count` | Integer | Number of diagnosed equipment |
| `operation_mode` | Enum \| null | Gas turbine operation mode (`GT` / `EPM`). Null if not applicable |
| `results` | Dictionary | Per-equipment results keyed by equipment ID (ME1, ME2, ME3, ME4) |

#### Diagnosis Levels

Each equipment has two diagnosis levels:

| Level | Fields | Description |
|---|---|---|
| `PRIMARY` | timestamp, status, result, score, updated_by | Overall equipment health score |
| `SECONDARY` | Per-subunit breakdown | Subunit-level detail |

#### SECONDARY Subunit Categories (per ME)

- Intake, Exhaust, Turbine, Cooling, Fuel, Lubrication, Control

#### Diagnostic Result Levels

| Result | Description |
|---|---|
| `NORMAL` | Equipment operating within normal parameters |
| `CAUTION` | Minor anomaly detected, monitoring recommended |
| `WARNING` | Significant anomaly, corrective action may be needed |
| `DANGER` | Critical condition, immediate action required |
| `STOP` | Equipment stopped or non-operational |

#### Subunit Score Fields

| Field | Type | Description |
|---|---|---|
| `result` | Enum | `NORMAL` / `CAUTION` / `WARNING` / `DANGER` / `STOP` |
| `score` | Float | Diagnostic score (0 -- 100) |
| `detail` | Dictionary | Detailed anomaly info: tag, deviation, duration, etc. |

> **Note (Surrogate Vessel)**: 4 main engines (ME1--ME4), 9 visualization equipment (Waterjet x4, GE x3, Bow Thruster x2).

---

## 5. Operational Advisory Execution Flow

When M&A detects a machinery issue, the following end-to-end flow occurs:

```
M&A                        VA                       VMS (HiCONiS)
 |                          |                          |
 |  1. OperationalAdvisory  |                          |
 |    (REST POST) --------->|                          |
 |                          |                          |
 |                          |  2. Relay to MA (uplink)  |
 |                          |     for operator display  |
 |                          |                          |
 |  3. Operator reviews     |                          |
 |     advisory options     |                          |
 |                          |                          |
 |  4. OperationalAdvisory  |                          |
 |     Response (accept=0)  |                          |
 |     (REST POST) -------->|                          |
 |                          |                          |
 |                          |  5. VA sends response    |
 |                          |     back to M&A          |
 |                          |     (REST POST) -------->|  (for acknowledgement)
 |                          |                          |
 |                          |  6. If accepted, VA      |
 |                          |     translates to VMS    |
 |                          |     command and executes |
 |                          |     (TCP/IP unicast) --->|
 |                          |                          |
 |                          |  7. VMS executes on      |
 |                          |     physical equipment   |
 |                          |                          |
```

### Advisory Option to VMS Command Mapping

| Advisory Option Type | VMS Command | Target System | Description |
|---|---|---|---|
| `SHIP_SPEED_REDUCTION` | Constrained Operation | PCS | Apply speed reduction ratio to propulsion |
| `SHIP_STOP` | Equipment Start/Stop | PCS, PMS, Auxiliary | Stop propulsion equipment |
| `REDUCED_POWER_RTB` | Constrained Operation + Mode Change | PCS, EPM | Apply power limit and switch to reduced-power mode |

### VA Behavior Rules

- VA shall **only** execute VMS commands when OperationalAdvisoryResponse from MA indicates accept (`response = 0`)
- VA shall **not** execute any VMS command when MA rejects the advisory (`response = 1`)
- VA shall log advisory execution status (accepted option type, VMS command issued, execution result) for audit
- If VMS returns a command failure, VA shall notify MA of the execution failure

---

## 6. Considerations

### 6.1 Thruster Feedback Timeout

- `$MTFBA` is expected at ~1 Hz.
- If no data is received for a configurable timeout (recommended: 3--5 seconds), assume DP system or VMS communication loss.
- Display a connection warning to the operator.

### 6.2 Variable Thruster Count

- The number of thruster groups in `$MTFBA` may vary between messages.
- Always parse dynamically based on available fields rather than assuming a fixed count.
- Validate that `DevID` is within the expected range (1--12).

### 6.3 Relative Values

- RPM and Pitch are **relative** (0.0 -- 1.0), not absolute physical values.
- The mapping from relative values to actual RPM or pitch angle depends on the specific equipment installation.
- Do not display these as absolute units (e.g., "500 RPM") without calibration data.

### 6.4 NMEA Checksum

NMEA checksum = XOR of all ASCII characters between `$` and `*`, expressed as 2-digit uppercase hex.

- Always validate the NMEA checksum before processing.
- Discard sentences with invalid checksums -- do not attempt partial parsing.

### 6.5 OperationalAdvisory -- Surrogate Demo Constraint

- In the Surrogate vessel demo, M&A cannot retrieve real-time vessel speed, making absolute target speed unfeasible for `SHIP_SPEED_REDUCTION`.
- Use one of the following approaches:
  1. **Remove absolute target speed** from the `SHIP_SPEED_REDUCTION` description
  2. **Express reduction as a relative ratio** (e.g., "Reduce speed by 50%") instead of an absolute target speed
- This constraint applies **only to the Surrogate Demo** and does not affect the general ICD specification.

### 6.6 Transport Protocol Status

- The VA <-> M&A transport protocol is still **under discussion** (TBD).
- Current proposal: REST API (HTTP) for event-driven/critical messages, UDP/NMEA for high-frequency data.
- Developers should prepare for possible changes in transport layer (REST API vs. MQTT vs. gRPC).

---

## 7. Reference Documents

| Document | Description |
|---|---|
| VA-ICD-0001 v0.3 | Source ICD document for this guide |
| AVK-NAS-SUPERVISORY-0002 | Navigation Autonomy (HiNAS) ICD v1.2.1 -- signal field definitions |
| IEC 61162-1 | Maritime navigation -- NMEA 0183 sentence format |
| IEC 61162-450 | Maritime navigation -- UDP multicast transport |
| `radar_target_api_guide.md` | Companion: radar target data reception guide |
| `hinas_ptz_mode_control_guide.md` | Companion: PTZ control and operation mode guide |
