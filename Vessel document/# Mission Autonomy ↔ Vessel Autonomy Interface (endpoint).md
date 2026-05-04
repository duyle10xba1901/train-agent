# Mission Autonomy ↔ Vessel Autonomy Interface Specification

> Audience: M&A / D&S developers · VA developers
> Reference date: 2026-04-20

---

## 1. End-to-End Data Flow

```
[Mission Autonomy (M&A)]      [Detection & Surveillance (D&S)]
       │                              │
       │ ① REST POST (JSON)           │ ② REST POST (JSON)
       ▼                              ▼
             [Vessel Autonomy BE (NestJS)]
     /va/api/v1/ma/*           /va/api/v1/ds/*
                  │
                  ├─► Persist to PostgreSQL
                  ├─► WebSocket emit → VA Frontend
                  └─► Redis publish → UDP Handler
                          │
                          │ ③ UDP Multicast (JSON envelope)
                          ▼
                  [Mission Autonomy (onboard C2)]
                   239.192.0.10:60010
```

D&S events are also relayed to Mission Autonomy through the same envelope as M&A events, distinguished by the `source` field (`"ma"` vs `"ds"`).

---

## 2. Interface Summary

### M&A ↔ VA

| #     | Direction    | Protocol         | Endpoint / Address                         | Cadence / Trigger               | Status                     |
| ----- | ------------ | ---------------- | ------------------------------------------ | ------------------------------- | -------------------------- |
| IF-M1 | **M&A → VA** | HTTP POST (JSON) | `POST /va/api/v1/ma/diagnostic-result`     | Every 10 s                      | ✅ Implemented             |
| IF-M2 | **M&A → VA** | HTTP POST (JSON) | `POST /va/api/v1/ma/operational-advisory`  | Event-driven (status ≥ CAUTION) | ✅ Implemented             |
| IF-M3 | **VA → M&A** | (TBD)            | Advisory response (operator accept/reject) | On operator selection           | ⏸ Endpoint not yet defined |

### D&S ↔ VA

| #     | Direction    | Protocol         | Endpoint / Address                    | Cadence / Trigger                          | Status         |
| ----- | ------------ | ---------------- | ------------------------------------- | ------------------------------------------ | -------------- |
| IF-D1 | **D&S → VA** | HTTP POST (JSON) | `POST /va/api/v1/ds/detect-event`     | Event-driven (disaster/accident detection) | ✅ Implemented |
| IF-D2 | **D&S → VA** | HTTP POST (JSON) | `POST /va/api/v1/ds/resolv-event`     | Event-driven (event resolution)            | ✅ Implemented |
| IF-D3 | **D&S → VA** | HTTP POST (JSON) | `POST /va/api/v1/ds/health-check`     | Periodic / on connection change            | ✅ Implemented |
| IF-D4 | **D&S → VA** | HTTP POST (JSON) | `POST /va/api/v1/ds/threat-detection` | Event-driven (security threat)             | ✅ Implemented |

### VA → Mission (relay)

| #     | Direction        | Protocol             | Endpoint / Address   | Cadence / Trigger                          | Status         |
| ----- | ---------------- | -------------------- | -------------------- | ------------------------------------------ | -------------- |
| IF-R1 | **VA → Mission** | UDP Multicast (JSON) | `239.192.0.10:60010` | Auto-relayed on any IF-M* or IF-D* receipt | ✅ Implemented |

---

## 3. M&A Interfaces

### 3.1 IF-M1. DiagnosticResult (M&A → VA)

#### Connection Details

| Item                  | Value                                                       |
| --------------------- | ----------------------------------------------------------- |
| Method                | `POST`                                                      |
| URL                   | `http://{VA_HOST}:{VA_PORT}/va/api/v1/ma/diagnostic-result` |
| Content-Type          | `application/json; charset=utf-8`                           |
| Timeout (recommended) | 10 s                                                        |
| Default cadence       | Every 10 s (`save_interval_seconds`)                        |

#### Payload Schema

| Field                   | Type                               | Required | Description                                    |
| ----------------------- | ---------------------------------- | -------- | ---------------------------------------------- |
| `save_timestamp`        | string (ISO-8601, UTC, `Z` suffix) | ✅       | Time when M&A saved the result                 |
| `save_interval_seconds` | number                             | ✅       | Interval in seconds (usually 10)               |
| `equipment_count`       | number                             | ✅       | Total number of equipment entries in `results` |
| `operation_mode`        | string \| null                     | ❌       | e.g., `"GT"`, `"EPM"` (omit if unknown)        |
| `results`               | object (dict)                      | ✅       | `<equipment_name>` → `{PRIMARY, SECONDARY?}`   |

**Equipment names**

- Diagnosed (both PRIMARY and SECONDARY): `ME1`, `ME2`, `ME3`, `ME4`
- Visualization only (PRIMARY only, `result`/`score` = null): `Waterjet1~4`, `GE1~3`, `Bowthruster1~2`
- 13 equipment total

**`PRIMARY` object**

| Field        | Type              | Value                                                                                          | Notes                            |
| ------------ | ----------------- | ---------------------------------------------------------------------------------------------- | -------------------------------- |
| `timestamp`  | string (ISO-8601) | —                                                                                              | —                                |
| `status`     | string            | `RUNNING` \| `STOP`                                                                            | —                                |
| `result`     | string \| null    | `NORMAL` \| `CAUTION` \| `WARNING` \| `DANGER` \| null                                         | null for visualization equipment |
| `score`      | number \| null    | 0.0 – 100.0                                                                                    | null for visualization equipment |
| `updated_by` | string            | `secondary_diagnosis_routine` \| `secondary_diagnosis_trigger` \| `visualization_status_check` | —                                |

**`SECONDARY` object** (ME1–4 only)

- `timestamp`, `status`, `result`, `score`, `updated_by`: same shape as PRIMARY
- `subunit_scores` — 7 fixed keys (`Intake`, `Exhaust`, `Turbine`, `Cooling`, `Fuel`, `Lubrication`, `Control`), each `{timestamp, result, score}`
- `tag_scores` — same 7 keys, each containing `{"<TAG_ID>": <score>}` (one representative tag per subunit)

#### Payload Example

```json
{
  "save_timestamp": "2026-03-30T06:02:38Z",
  "save_interval_seconds": 10,
  "equipment_count": 13,
  "results": {
    "ME1": {
      "PRIMARY": {
        "timestamp": "2026-03-30T06:02:30Z",
        "status": "RUNNING",
        "result": "CAUTION",
        "score": 79.34,
        "updated_by": "secondary_diagnosis_trigger"
      },
      "SECONDARY": {
        "timestamp": "2026-03-30T06:02:30Z",
        "status": "RUNNING",
        "result": "CAUTION",
        "score": 74.37,
        "updated_by": "secondary_diagnosis_trigger",
        "subunit_scores": {
          "Intake": {
            "timestamp": "2026-03-30T06:02:30Z",
            "result": "CAUTION",
            "score": 74.37
          },
          "Exhaust": {
            "timestamp": "2026-03-30T06:02:30Z",
            "result": "NORMAL",
            "score": 86.39
          },
          "Turbine": {
            "timestamp": "2026-03-30T06:02:30Z",
            "result": "NORMAL",
            "score": 89.49
          },
          "Cooling": {
            "timestamp": "2026-03-30T06:02:30Z",
            "result": "NORMAL",
            "score": 83.5
          },
          "Fuel": {
            "timestamp": "2026-03-30T06:02:30Z",
            "result": "NORMAL",
            "score": 85.33
          },
          "Lubrication": {
            "timestamp": "2026-03-30T06:02:30Z",
            "result": "NORMAL",
            "score": 84.27
          },
          "Control": {
            "timestamp": "2026-03-30T06:02:30Z",
            "result": "NORMAL",
            "score": 89.73
          }
        },
        "tag_scores": {
          "Intake": { "ME1_Intake_T01": 74.37 },
          "Exhaust": { "ME1_Exhaust_T01": 86.39 },
          "Turbine": { "ME1_Turbine_T01": 89.49 },
          "Cooling": { "ME1_Cooling_T01": 83.5 },
          "Fuel": { "ME1_Fuel_T01": 85.33 },
          "Lubrication": { "ME1_Lub_T01": 84.27 },
          "Control": { "ME1_Ctrl_T01": 89.73 }
        }
      }
    },
    "Waterjet1": {
      "PRIMARY": {
        "timestamp": "2026-03-30T06:02:09Z",
        "status": "STOP",
        "result": null,
        "score": null,
        "updated_by": "visualization_status_check"
      }
    }
    /* ...ME2–ME4, Waterjet2–4, GE1–3, Bowthruster1–2 follow the same pattern... */
  }
}
```

#### Response

```json
{ "status": "ok", "uuid": "<v4 uuid>", "receivedAt": "<iso-8601>" }
```

- Success: `200 OK`
- Validation failure: `400 Bad Request`
- Server error: `500`

---

### 3.2 IF-M2. OperationalAdvisory (M&A → VA)

#### Connection Details

| Item         | Value                                                          |
| ------------ | -------------------------------------------------------------- |
| Method       | `POST`                                                         |
| URL          | `http://{VA_HOST}:{VA_PORT}/va/api/v1/ma/operational-advisory` |
| Content-Type | `application/json; charset=utf-8`                              |
| Timeout      | 10 s                                                           |
| Trigger      | When M&A determines overall status ≥ `CAUTION`                 |

#### Payload Schema

| Top-level field        | Type   | Required | Description                           |
| ---------------------- | ------ | -------- | ------------------------------------- |
| `metadata`             | object | ✅       | Header/metadata                       |
| `machinery_status`     | object | ✅       | Overall status and per-machine issues |
| `operational_advisory` | object | ✅       | Recommended option list               |

**`metadata`**

| Field        | Type              | Description                      |
| ------------ | ----------------- | -------------------------------- |
| `request_id` | string (UUID v4)  | Request tracking ID              |
| `timestamp`  | string (ISO-8601) | Advisory generation time         |
| `ship_id`    | string            | Vessel IMO (e.g., `IMO-0000000`) |
| `version`    | string            | Message format version (`v1.0`)  |

**`machinery_status`**

| Field             | Type          | Description                                    |
| ----------------- | ------------- | ---------------------------------------------- |
| `overall_status`  | enum          | `NORMAL` \| `CAUTION` \| `WARNING` \| `DANGER` |
| `detected_issues` | array<object> | List of machines with issues                   |

`detected_issues[]`

| Field          | Type                           | Description                                               |
| -------------- | ------------------------------ | --------------------------------------------------------- |
| `machine_name` | string                         | e.g., `"ME1"`                                             |
| `severity`     | enum                           | `NORMAL` \| `CAUTION` \| `WARNING` \| `DANGER`            |
| `alarms`       | array<{message, duration_sec}> | Active alarms (`message`: string, `duration_sec`: number) |

**`operational_advisory`**

| Field            | Type          | Description                   |
| ---------------- | ------------- | ----------------------------- |
| `reason_summary` | string        | Summary text for the advisory |
| `options`        | array<option> | Always 3 entries (see below)  |

`options[]`

| Field            | Type          | Description                                                      |
| ---------------- | ------------- | ---------------------------------------------------------------- |
| `type`           | enum          | `SHIP_SPEED_REDUCTION` \| `SHIP_STOP` \| `REDUCED_POWER_RTB`     |
| `is_recommended` | boolean       | `true` only on the M&A-recommended option                        |
| `cause_machines` | array<string> | Machines that triggered this option (empty when not recommended) |
| `description`    | string        | Human-readable label for the UI                                  |

**Advisory option meanings**

| Type                   | Meaning                       | Typical trigger                      |
| ---------------------- | ----------------------------- | ------------------------------------ |
| `SHIP_SPEED_REDUCTION` | Reduce speed                  | Minor issue (CAUTION)                |
| `SHIP_STOP`            | Emergency stop                | Critical issue (DANGER)              |
| `REDUCED_POWER_RTB`    | Reduced power, return to base | Severe but operationally continuable |

#### Payload Example

```json
{
  "metadata": {
    "request_id": "c0bc3813-0844-42e5-9d96-a2c0f4902c10",
    "timestamp": "2026-03-30T06:02:38Z",
    "ship_id": "IMO-0000000",
    "version": "v1.0"
  },
  "machinery_status": {
    "overall_status": "CAUTION",
    "detected_issues": [
      {
        "machine_name": "ME1",
        "severity": "CAUTION",
        "alarms": [{ "message": "High Exhaust Gas Temp", "duration_sec": 1358 }]
      }
    ]
  },
  "operational_advisory": {
    "reason_summary": "CAUTION detected on ME1. Immediate advisory issued based on highest severity.",
    "options": [
      {
        "type": "SHIP_SPEED_REDUCTION",
        "is_recommended": true,
        "cause_machines": ["ME1"],
        "description": "Reduce speed to 10kts"
      },
      {
        "type": "SHIP_STOP",
        "is_recommended": false,
        "cause_machines": [],
        "description": "Emergency stop"
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

#### Response

Same as IF-M1 (`status`, `uuid`, `receivedAt`).

---

### 3.3 IF-M3. VA → M&A Response (TBD)

Not yet defined. **The M&A team must specify the receiving endpoint.**

Intended use: once an operator accepts or rejects an `OperationalAdvisory` option in the VA frontend, the result is returned to M&A.

**Items to agree on**

- Protocol (REST POST recommended)
- Endpoint path and port
- Response payload format
- Correlation via `request_id` when multiple advisories are in-flight

---

## 4. D&S Interfaces

All D&S endpoints share the same base URL, content type, and response shape as the M&A endpoints.

| Item                  | Value                                                                 |
| --------------------- | --------------------------------------------------------------------- |
| Base URL              | `http://{VA_HOST}:{VA_PORT}/va/api/v1/ds/`                            |
| Content-Type          | `application/json; charset=utf-8`                                     |
| Timeout (recommended) | 10 s                                                                  |
| Response              | `{ "status": "ok", "uuid": "<v4 uuid>", "receivedAt": "<iso-8601>" }` |

### 4.1 IF-D1. DetectEvent (D&S → VA)

Issued when a disaster/accident class event (fire, smoke, flooding, etc.) is detected.

| Item   | Value                             |
| ------ | --------------------------------- |
| Method | `POST`                            |
| URL    | `POST /va/api/v1/ds/detect-event` |

#### Payload Schema

| Field           | Type              | Required | Description                                          |
| --------------- | ----------------- | -------- | ---------------------------------------------------- |
| `cameraId`      | integer           | ✅       | Camera ID                                            |
| `eventClass`    | string            | ✅       | Event class (e.g., `"Disaster/Accident"`)            |
| `eventName`     | string            | ✅       | Event name (e.g., `"Fire"`, `"Smoke"`, `"Flooding"`) |
| `eventLevel`    | integer           | ✅       | Severity level (1–5)                                 |
| `savePath`      | string            | ✅       | Image save path                                      |
| `createdAt`     | string (ISO-8601) | ✅       | Event creation timestamp                             |
| `contamination` | integer           | ❌       | Contamination flag                                   |
| `detectionInfo` | object            | ❌       | `{ classId, className, weight, xyxy }`               |
| `anomalyMap`    | string            | ❌       | Anomaly map file path                                |
| `anomalyScore`  | number            | ❌       | Anomaly score (0.0 – 1.0)                            |

#### Payload Example

```json
{
  "cameraId": 1,
  "eventClass": "Disaster/Accident",
  "eventName": "Fire",
  "eventLevel": 4,
  "savePath": "1/fire/imagepath.png",
  "createdAt": "2026-01-14T10:00:00Z",
  "contamination": 1,
  "detectionInfo": {
    "classId": 0,
    "className": "fire",
    "weight": "136.pth",
    "xyxy": [0.3640625, 0.2680555, 0.421875, 0.3277777]
  },
  "anomalyMap": "1/fire/anomalyMapimagepath.npy",
  "anomalyScore": 0.94
}
```

---

### 4.2 IF-D2. ResolvEvent (D&S → VA)

Issued when a previously reported detect event is resolved (flame extinguished, intruder left, etc.).

| Item   | Value                             |
| ------ | --------------------------------- |
| Method | `POST`                            |
| URL    | `POST /va/api/v1/ds/resolv-event` |

#### Payload Schema

| Field           | Type              | Required | Description                                                   |
| --------------- | ----------------- | -------- | ------------------------------------------------------------- |
| `cameraId`      | integer           | ✅       | Camera ID                                                     |
| `eventId`       | integer           | ✅       | Event ID to resolve                                           |
| `endedAt`       | string (ISO-8601) | ✅       | Event end timestamp                                           |
| `contamination` | integer           | ❌       | Contamination flag                                            |
| `detectionInfo` | object            | ❌       | Optional `{ classId, className, weight, xyxy }` at resolution |
| `anomalyMap`    | string            | ❌       | Anomaly map file path                                         |
| `anomalyScore`  | number            | ❌       | Anomaly score                                                 |

> If VA cannot find a matching `eventId`, it resolves the latest unresolved event on the same `cameraId`.

#### Payload Example

```json
{
  "cameraId": 1,
  "eventId": 3,
  "endedAt": "2026-01-14T10:03:00Z",
  "contamination": 1,
  "detectionInfo": {
    "classId": null,
    "className": null,
    "weight": "136.pth",
    "xyxy": null
  },
  "anomalyMap": "1/fire/anomalyMapimagepath.npy",
  "anomalyScore": 0.3
}
```

---

### 4.3 IF-D3. HealthCheck (D&S → VA)

Reports camera/sensor connection state (periodic or on connection change).

| Item   | Value                             |
| ------ | --------------------------------- |
| Method | `POST`                            |
| URL    | `POST /va/api/v1/ds/health-check` |

#### Payload Schema

| Field           | Type              | Required | Description                                                |
| --------------- | ----------------- | -------- | ---------------------------------------------------------- |
| `cameraId`      | integer           | ✅       | Camera ID                                                  |
| `connectionLog` | string            | ✅       | Connection status (e.g., `"Connected"`, `"Disconnection"`) |
| `createdAt`     | string (ISO-8601) | ✅       | Timestamp                                                  |

#### Payload Example

```json
{
  "cameraId": 1,
  "connectionLog": "Disconnection",
  "createdAt": "2026-01-14T10:03:00Z"
}
```

---

### 4.4 IF-D4. ThreatDetection (D&S → VA)

Issued for security threat class events (intruder, suspicious object, etc.).

| Item   | Value                                 |
| ------ | ------------------------------------- |
| Method | `POST`                                |
| URL    | `POST /va/api/v1/ds/threat-detection` |

#### Payload Schema

| Field           | Type              | Required | Description                            |
| --------------- | ----------------- | -------- | -------------------------------------- |
| `cameraId`      | integer           | ✅       | Camera ID                              |
| `eventClass`    | string            | ✅       | Event class (e.g., `"Security"`)       |
| `eventName`     | string            | ✅       | Event name (e.g., `"Intruder"`)        |
| `eventLevel`    | integer           | ✅       | Severity level                         |
| `savePath`      | string            | ✅       | Image save path                        |
| `createdAt`     | string (ISO-8601) | ✅       | Event creation timestamp               |
| `detectionInfo` | object            | ❌       | `{ classId, className, weight, xyxy }` |

#### Payload Example

```json
{
  "cameraId": 2,
  "eventClass": "Security",
  "eventName": "Intruder",
  "eventLevel": 3,
  "savePath": "2/threat/imagepath.png",
  "createdAt": "2026-01-14T10:05:00Z",
  "detectionInfo": {
    "classId": 10,
    "className": "person",
    "weight": "200.pth",
    "xyxy": [0.15, 0.25, 0.45, 0.85]
  }
}
```

---

## 5. IF-R1. VA → Mission Relay (UDP Multicast)

All M&A and D&S payloads received by VA are **automatically relayed** to Mission Autonomy over UDP. No additional action is required on the sender side.

### 5.1 Transmission Details

| Item            | Value            |
| --------------- | ---------------- |
| Protocol        | UDP Multicast    |
| Multicast group | `239.192.0.10`   |
| Port            | `60010`          |
| TTL             | 3 (local subnet) |
| Encoding        | UTF-8 JSON       |

### 5.2 Envelope Schema

```json
{
  "source": "ma | ds",
  "type": "<event type identifier>",
  "relayedAt": "<iso-8601 UTC>",
  "payload": {
    /* original IF-M* or IF-D* payload verbatim */
  }
}
```

| Field       | Description                                         |
| ----------- | --------------------------------------------------- |
| `source`    | Originating system — `"ma"` for M&A, `"ds"` for D&S |
| `type`      | Event type identifier (see table below)             |
| `relayedAt` | Time VA relayed the event                           |
| `payload`   | Untouched copy of the original sender JSON          |

### 5.3 `type` Values

| `source` | `type`                 | Original interface |
| -------- | ---------------------- | ------------------ |
| `ma`     | `diagnostic-result`    | IF-M1              |
| `ma`     | `operational-advisory` | IF-M2              |
| `ds`     | `detect-event`         | IF-D1              |
| `ds`     | `resolv-event`         | IF-D2              |
| `ds`     | `health-check`         | IF-D3              |
| `ds`     | `threat-detection`     | IF-D4              |
