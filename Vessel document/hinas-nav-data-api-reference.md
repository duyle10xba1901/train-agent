# hinas_nav_data - HiNAS Navigation Data API Reference

> **Source**: HiNAS_Supervisory_System_ICD_v1.2.1 (2026-03-23), Sections 3.3.2, 6.1.1
> **Direction**: HiNAS -> Supervisory system
> **Key**: `hinas_nav_data`

---

## 1. Overview

Aggregated navigation data currently selected by HiNAS Sensor Processing. Contains the selected sensor name, integrity status, and current value for each navigation data type.

| Item | Value |
|------|-------|
| **Key** | `hinas_nav_data` |
| **Direction** | HiNAS -> Supervisory system |
| **Format** | JSON |
| **Protocol** | *(TBD)* |
| **Period** | Periodic, 1.0 s (1 Hz) |
| **Notes** | Protocol & format TBD |

Published periodically at 1 Hz whenever any member data is updated.

---

## 2. General Rules

- Each top-level data object is **present only** when its selected sensor has integrity status `normal` or `low_integrity`.
- Objects are **omitted** when status is `invalid` or `nosignal`.
- Measurement fields within a present object may be `null` when the selected sensor is active but the specific NMEA sentence carrying that field has not yet been received in the current cycle.
- The `rpm`, `rudder`, and `telegraph` sub-objects use **Twin Mode** fields (`port_status` / `starboard_status`) or **Single Mode** fields (single `status` field and a single value field).

---

## 3. Integrity Status Values

All `status`, `port_status`, and `starboard_status` fields use the following enum:

| Value | Meaning |
|-------|---------|
| `normal` | Data received and passes all integrity checks. |
| `low_integrity` | Data received but fails some optional integrity checks (values may be less reliable). |
| `invalid` | Data received but fails mandatory integrity checks. Sub-object omitted. |
| `nosignal` | Sensor configured but no data received in this session. Sub-object omitted. |

---

## 4. JSON Example

```json
{
  "position": {
    "selected_sensor": "GPS1",
    "status": "normal",
    "latitude": 37.5665,
    "longitude": 126.9780
  },
  "cog": {
    "selected_sensor": "GPS1",
    "status": "normal",
    "cog": 125.5
  },
  "sog": {
    "selected_sensor": "GPS1",
    "status": "normal",
    "sog": 15.5,
    "longitudinal_sog": 15.3,
    "transverse_sog": 1.2
  },
  "heading": {
    "selected_sensor": "HDG1",
    "status": "normal",
    "heading": 125.5
  },
  "rot": {
    "selected_sensor": "ROT1",
    "status": "normal",
    "rot": 2.5
  },
  "rpm": {
    "selected_sensor": "RPM1",
    "port_status": "normal",
    "starboard_status": "normal",
    "port_rpm": 120.0,
    "starboard_rpm": 118.5
  },
  "wind": {
    "selected_sensor": "WIND1",
    "status": "normal",
    "true_wind_speed_knots": 12.5,
    "true_wind_direction": 180.0
  },
  "current": {
    "selected_sensor": "CURR1",
    "status": "normal",
    "speed_knots": 1.2,
    "direction": 90.0
  },
  "stw": {
    "selected_sensor": "STW1",
    "status": "normal",
    "stw": 14.8,
    "longitudinal_stw": 14.5,
    "transverse_stw": 0.8
  },
  "rudder": {
    "selected_sensor": "RUDDER1",
    "port_status": "normal",
    "starboard_status": "normal",
    "port_rudder": -3.0,
    "starboard_rudder": -3.5
  },
  "depth": {
    "selected_sensor": "DEPTH1",
    "status": "normal",
    "depth": 25.5
  },
  "telegraph": {
    "selected_sensor": "TELEGRAPH1",
    "port_status": "normal",
    "starboard_status": "normal",
    "port_direction": "AH",
    "port_magnitude": "HALF",
    "port_engine": "2",
    "starboard_direction": "AH",
    "starboard_magnitude": "HALF",
    "starboard_engine": "1"
  },
  "time_zone": {
    "selected_sensor": "GPS1",
    "status": "normal",
    "utc": "104422.00",
    "year": 2025,
    "month": 12,
    "day": 22
  },
  "imu": {
    "roll": 2.34,
    "pitch": -1.12,
    "yaw": 45.6,
    "status": "normal"
  },
  "issued_time": "2025-12-22T10:44:22.123456Z"
}
```

---

## 5. Field Definitions

### 5.1. Top-level

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| **issued_time** ★ | string | — | ISO 8601 UTC | Wall-clock UTC time the message was assembled (e.g. `2025-12-22T10:44:22.123456Z`). |

> ★ = required field (always present)

---

### 5.2. position *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| position.selected_sensor ★ | string | — | non-empty | Name of the selected position sensor. |
| position.status ★ | string | — | `normal` \| `low_integrity` \| `invalid` \| `nosignal` | Integrity status of the selected sensor. |
| position.latitude | double \| null | deg | -90.0 ~ +90.0 | Latitude in decimal degrees (WGS84). North positive. Null if GLL/GGA not yet received. |
| position.longitude | double \| null | deg | -180.0 ~ +180.0 | Longitude in decimal degrees (WGS84). East positive. Null if GLL/GGA not yet received. |

---

### 5.3. cog *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| cog.selected_sensor ★ | string | — | non-empty | Name of the selected COG sensor. |
| cog.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor. |
| cog.cog | double \| null | deg | 0.0 ~ 360.0 | Course Over Ground (true). Null if VTG not yet received. |

---

### 5.4. sog *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| sog.selected_sensor ★ | string | — | non-empty | Name of the selected SOG sensor. |
| sog.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor. |
| sog.sog | double \| null | kn | 0.0 ~ 99.0 | Speed Over Ground from VTG sentence. Null if VTG not yet received. |
| sog.longitudinal_sog | double \| null | kn | -99.0 ~ +99.0 | Longitudinal SOG from VBW sentence. Null if VBW not yet received. |
| sog.transverse_sog | double \| null | kn | -9.0 ~ +99.0 | Transverse SOG from VBW sentence. Null if VBW not yet received. |

---

### 5.5. heading *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| heading.selected_sensor ★ | string | — | non-empty | Name of the selected heading sensor. |
| heading.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor. |
| heading.heading | double \| null | deg | 0.0 ~ 360.0 | True heading. Null if HDT/THS/VHW not yet received. |

---

### 5.6. rot *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| rot.selected_sensor ★ | string | — | non-empty | Name of the selected ROT sensor. |
| rot.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor. |
| rot.rot | double \| null | deg/min | -999.0 ~ +999.0 | Rate of Turn. Positive = turning starboard. Null if ROT not yet received. |

---

### 5.7. rpm — Single Mode *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| rpm.selected_sensor ★ | string | — | non-empty | Name of the selected RPM sensor. |
| rpm.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor (Single Mode only). |
| rpm.rpm | double \| null | rpm | -999999 ~ +999999 | Engine RPM. Null if RPM sentence not yet received. |

### 5.8. rpm — Twin Mode *(side omitted when its status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| rpm.selected_sensor ★ | string | — | non-empty | Name of the selected RPM sensor. |
| rpm.port_status ★ | string | — | see Integrity Status Values | Integrity status of the port engine sensor. |
| rpm.starboard_status ★ | string | — | see Integrity Status Values | Integrity status of the starboard engine sensor. |
| rpm.port_rpm | double \| null | rpm | -999999 ~ +999999 | Port engine RPM. Null if not yet received. |
| rpm.starboard_rpm | double \| null | rpm | -999999 ~ +999999 | Starboard engine RPM. Null if not yet received. |

---

### 5.9. wind *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| wind.selected_sensor ★ | string | — | non-empty | Name of the selected wind sensor. |
| wind.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor. |
| wind.true_wind_speed_knots | double \| null | kn | 0.0 ~ 50.0 | True wind speed in knots. Null if not yet received. |
| wind.true_wind_direction | double \| null | deg | 0.0 ~ 360.0 | True wind direction in degrees. Null if not yet received. |

---

### 5.10. current *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| current.selected_sensor ★ | string | — | non-empty | Name of the selected current sensor. |
| current.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor. |
| current.speed_knots | double \| null | kn | 0.0 ~ 50.0 | True current speed (drift). Null if VDR not yet received. |
| current.direction | double \| null | deg | 0.0 ~ 360.0 | True current direction (set). Null if VDR not yet received. |

---

### 5.11. stw *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| stw.selected_sensor ★ | string | — | non-empty | Name of the selected STW sensor. |
| stw.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor. |
| stw.stw | double \| null | kn | 0.0 ~ 99.0 | Speed Through Water from VHW sentence. Null if VHW not yet received. |
| stw.longitudinal_stw | double \| null | kn | -99.0 ~ +99.0 | Longitudinal STW from VBW sentence. Null if VBW not yet received. |
| stw.transverse_stw | double \| null | kn | -99.0 ~ +99.0 | Transverse STW from VBW sentence. Null if VBW not yet received. |

---

### 5.12. rudder — Single Mode *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| rudder.selected_sensor ★ | string | — | non-empty | Name of the selected rudder sensor. |
| rudder.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor (Single Mode only). |
| rudder.rudder | double \| null | deg | -180.0 ~ +180.0 | Rudder angle. Negative = port, positive = starboard. Null if RSA not yet received. |

### 5.13. rudder — Twin Mode *(side omitted when its status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| rudder.selected_sensor ★ | string | — | non-empty | Name of the selected rudder sensor. |
| rudder.port_status ★ | string | — | see Integrity Status Values | Integrity status of the port-side sensor. |
| rudder.starboard_status ★ | string | — | see Integrity Status Values | Integrity status of the starboard-side sensor. |
| rudder.port_rudder | double \| null | deg | -180.0 ~ +180.0 | Port rudder angle. Negative = port, positive = starboard. Null if not yet received. |
| rudder.starboard_rudder | double \| null | deg | -180.0 ~ +180.0 | Starboard rudder angle. Negative = port, positive = starboard. Null if not yet received. |

---

### 5.14. depth *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| depth.selected_sensor ★ | string | — | non-empty | Name of the selected depth sensor. |
| depth.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor. |
| depth.depth | double \| null | m | 0.0 ~ 200.0 | Water depth. Null if DPT/DBT not yet received. |

---

### 5.15. telegraph — Single Mode *(omitted when status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| telegraph.selected_sensor ★ | string | — | non-empty | Name of the selected telegraph sensor. |
| telegraph.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor (Single Mode only). |
| telegraph.direction | string \| null | — | `AH` \| `AS` | Engine direction. `AH` = Ahead, `AS` = Astern. Null when `STOP_ENGINE`. |
| telegraph.magnitude | string \| null | — | `STOP_ENGINE` \| `DEAD_SLOW` \| `SLOW` \| `HALF` \| `FULL` \| `NAV_FULL` \| `CRASH_ASTERN` | Engine power level. |
| telegraph.engine | string \| null | — | non-empty | Engine identifier (e.g. `"0"` for Center/Single). |

### 5.16. telegraph — Twin Mode *(side omitted when its status is `invalid` or `nosignal`)*

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| telegraph.selected_sensor ★ | string | — | non-empty | Name of the selected telegraph sensor. |
| telegraph.port_status ★ | string | — | see Integrity Status Values | Integrity status of the port engine sensor. |
| telegraph.starboard_status ★ | string | — | see Integrity Status Values | Integrity status of the starboard engine sensor. |
| telegraph.port_direction | string \| null | — | `AH` \| `AS` | Port engine direction. Null when `STOP_ENGINE`. |
| telegraph.port_magnitude | string \| null | — | `STOP_ENGINE` \| `DEAD_SLOW` \| `SLOW` \| `HALF` \| `FULL` \| `NAV_FULL` \| `CRASH_ASTERN` | Port engine power level. |
| telegraph.port_engine | string \| null | — | non-empty | Port engine identifier (even ETL engine number, e.g. `"2"`). |
| telegraph.starboard_direction | string \| null | — | `AH` \| `AS` | Starboard engine direction. Null when `STOP_ENGINE`. |
| telegraph.starboard_magnitude | string \| null | — | `STOP_ENGINE` \| `DEAD_SLOW` \| `SLOW` \| `HALF` \| `FULL` \| `NAV_FULL` \| `CRASH_ASTERN` | Starboard engine power level. |
| telegraph.starboard_engine | string \| null | — | non-empty | Starboard engine identifier (odd ETL engine number, e.g. `"1"`). |

---

### 5.17. time_zone

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| time_zone.selected_sensor ★ | string | — | non-empty | Name of the selected time zone sensor. |
| time_zone.status ★ | string | — | see Integrity Status Values | Integrity status of the selected sensor. |
| time_zone.utc | string \| null | — | `hhmmss.ss` | UTC time string. Null if ZDA not yet received. |
| time_zone.year | integer \| null | — | — | Year. Null if ZDA not yet received. |
| time_zone.month | integer \| null | — | 1 ~ 12 | Month. Null if ZDA not yet received. |
| time_zone.day | integer \| null | — | 1 ~ 31 | Day. Null if ZDA not yet received. |

---

### 5.18. imu

| Field | Type | Unit | Range / Values | Description |
|-------|------|------|---------------|-------------|
| imu.roll | double \| null | deg | — | Roll angle. |
| imu.pitch | double \| null | deg | — | Pitch angle. |
| imu.yaw | double \| null | deg | — | Yaw angle. |
| imu.status ★ | string | — | see Integrity Status Values | Integrity status of the IMU sensor. |
