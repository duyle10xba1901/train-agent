# Radar Target Data Receiver API Guide

> ICD Reference: HiNAS - Supervisory System ICD v1.2.1 (2026-03-23)
>
> This document is a development guide for receiving radar target data (`nas:target:supervisory`, `nas:lost_target:supervisory`) transmitted from HiNAS to the Supervisory system.

---

## 1. Overview

HiNAS periodically transmits fused target information from multiple sensors (Radar, AIS, EO/IR, etc.) to the Supervisory system via the **Sensor Fusion** module.

- **Unidirectional push stream** (HiNAS -> Supervisory)
- **No session / handshake / ACK** -- each datagram is transmitted independently
- Composed of two parallel JSON streams

| Stream | Key | Description | Period |
|---|---|---|---|
| Fused Target List | `nas:target:supervisory` | List of all currently tracked targets | 0.5 s (2 Hz) |
| Lost Target List | `nas:lost_target:supervisory` | List of lost (no longer tracked) targets | 0.5 s (2 Hz) |

> **Note**: The NMEA 0183 TTM sentence used in v1.0.0 was removed in v1.1.0. The current version uses JSON format only.

---

## 2. Transport Specification

| Item | Value |
|---|---|
| Standard | IEC 61162-450 Ed.3 -- UTF-8 JSON over UDP/IP |
| Network layer | IPv4 UDP Multicast |
| Multicast address range | `239.129.0.1` -- `239.129.0.16` *(TBD -- actual address to be assigned)* |
| UDP port | `60001` -- `60016` *(TBD)* |
| TTL | 1 (link-local only) |
| Payload format | UTF-8 encoded raw JSON |
| Transmission type | RaUdP -- Periodic 0.5 s (2 Hz) |
| Binary Descriptor -- DataType | `application/json` (per IEC 61162-450 SS7.3.4) |
| Binary Descriptor -- Status and information | `nas:target:supervisory` or `nas:lost_target:supervisory` |

### 2.1 RaUdP Frame Structure

```
+-----------------------------------------------+
|  IEC 61162-450 RaUdP Binary Header             |
|  +-- ...                                       |
|  +-- DataType: "application/json"              |
|  +-- Status: "nas:target:supervisory"          |  <-- message type identifier
|  |       or "nas:lost_target:supervisory"      |
|  +-- ...                                       |
+-----------------------------------------------+
|  JSON Payload (UTF-8)                          |
|  {"issued_time":"...","obj":[{...},{...}]}     |
+-----------------------------------------------+
```

- The two streams (`nas:target:supervisory` / `nas:lost_target:supervisory`) are distinguished by the `Status and information` field.
- Max UDP datagram size: **1472 bytes**. If exceeded, RaUdP automatically segments the payload; the receiver may need to reassemble.

---

## 3. JSON Payload Definitions

### 3.1 nas:target:supervisory (Fused Target List)

Fused information of all currently tracked targets.

#### JSON Example

```json
{
    "issued_time": "2026-02-26T15:51:28.664Z",
    "obj": [
        {
            "id": "81d37de6-ac21-498e-a168-6065c935e538",
            "fusion_type": 2,
            "ship_name": "HAIAN_VIEW",
            "mmsi": 574004780,
            "call_sign": null,
            "ship_type": 79,
            "navigation_status": 0,
            "latitude": 1.257135875830715,
            "longitude": 104.02463512742023,
            "cog_deg": 262.0,
            "sog_kn": 9.6,
            "distance_nm": 2.827632007087145,
            "bearing_deg": 210.30712704438952,
            "cpa_nm": 2.218840442527526,
            "tcpa_s": -10.954906090098904
        }
    ]
}
```

#### Field Definitions -- Root

| Field | Type | Required | Description |
|---|---|---|---|
| `issued_time` | string | Y | Message publication time (ISO 8601 UTC). e.g., `"2026-02-26T15:51:28.664Z"` |
| `obj` | array | Y | Array of Target Objects. Empty array `[]` if no targets are tracked |

#### Field Definitions -- obj[] (Target Object)

| Field | Type | Unit | Range | Description |
|---|---|---|---|---|
| `id` | string (UUID) | -- | e.g., `81d37de6-ac21-498e-a168-6065c935e538` | Unique target identifier (128-bit UUID) |
| `fusion_type` | integer | -- | 1--15 | Sensor fusion type (bitmask, see table below) |
| `ship_name` | string \| null | -- | max 20 chars | AIS ship name (uppercase, 6-bit ASCII) |
| `mmsi` | integer \| null | -- | 000000000--999999999 | MMSI number (9 digits) |
| `call_sign` | string \| null | -- | max 7 chars | Call sign (A-Z, 0-9) |
| `ship_type` | enum \| null | -- | -- | AIS ship type code (ITU-R M.1371-5 Table 53) |
| `navigation_status` | integer \| enum \| null | -- | 0--15 | Navigational status (see table below) |
| `latitude` | double | deg | -80.0 ~ +84.0 | Target latitude (WGS84) |
| `longitude` | double | deg | -180.0 ~ +180.0 | Target longitude (WGS84) |
| `cog_deg` | double | deg | 0.0--359.9 | Course Over Ground. `360.0` = Not Available |
| `sog_kn` | double | kn | 0.0--99.9 | Speed Over Ground |
| `distance_nm` | double \| null | NM | -- | Distance from own ship |
| `bearing_deg` | double \| null | deg | 0.0--359.9 | Bearing from own ship (True North reference) |
| `cpa_nm` | double | NM | -- | Closest Point of Approach |
| `tcpa_s` | double | sec | -- | Time to Closest Point of Approach (negative = already passed) |

#### fusion_type Bitmask

| Bit | Value (hex) | Value (int) | Sensor |
|---|---|---|---|
| bit 0 | 0x0001 | 1 | AIS |
| bit 1 | 0x0002 | 2 | ARPA (Radar) |
| bit 2 | 0x0100 | 4 | EO (Electro-Optical) |
| bit 3 | 0x1000 | 8 | IR (Infrared) |

> Combined via bitwise OR. e.g., `3` = AIS + ARPA, `6` = ARPA + EO

#### navigation_status Values

| Value | Description |
|---|---|
| 0 | Under way using engine |
| 1 | At anchor |
| 2 | Not under command |
| 3 | Restricted manoeuvrability |
| 4 | Constrained by her draught |
| 5 | Moored |
| 6 | Aground |
| 7 | Engaged in fishing |
| 8 | Under way sailing |
| 9 | Reserved for HSC |
| 10 | Reserved for WIG |
| 11--14 | Reserved for future use |
| 15 | Not defined |

---

### 3.2 nas:lost_target:supervisory (Lost Target List)

List of targets that have been lost from sensor fusion tracking.

#### JSON Example

```json
{
    "issued_time": "2026-02-26T22:16:50.647Z",
    "obj": [
        {
            "id": "a9703362-bbee-48dc-a63a-a7d905024ec1",
            "fusion_type": 1,
            "ship_name": null,
            "mmsi": 5631104,
            "call_sign": null,
            "ship_type": null,
            "navigation_status": null,
            "latitude": 1.3114861436382919,
            "longitude": 104.03247410020612,
            "cog_deg": null,
            "sog_kn": null,
            "distance_nm": 1.243652396938483,
            "bearing_deg": 310.40121238474455,
            "cpa_nm": null,
            "tcpa_s": null
        }
    ]
}
```

#### Differences from nas:target:supervisory

| Field | Fused Target | Lost Target |
|---|---|---|
| `cog_deg` | actual value | **always `null`** |
| `sog_kn` | actual value | **always `null`** |
| `cpa_nm` | actual value | **always `null`** |
| `tcpa_s` | actual value | **always `null`** |
| `distance_nm` | actual value | actual value or null |
| `bearing_deg` | actual value | actual value or null |

> Lost targets retain only the last known position (`latitude`, `longitude`) and distance/bearing. Motion data (course/speed/CPA/TCPA) is always null.

---

## 4. Receiver Implementation Guide

### 4.1 Reception Procedure Summary

```
1. Create UDP multicast socket
2. IGMP Join (join multicast group)
3. Receive loop:
   a. Receive UDP datagram (recvfrom)
   b. Parse RaUdP binary header
      - Identify message type via Status field
      - Reassemble if segmented
   c. Extract and parse JSON payload
   d. Process target data
```

### 4.2 Multicast Socket Setup

```python
import socket
import struct

MULTICAST_GROUP = "239.129.0.1"   # TBD - replace with assigned address
PORT = 60001                       # TBD - replace with assigned port
LOCAL_INTERFACE = "0.0.0.0"        # specify NIC IP if needed

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM, socket.IPPROTO_UDP)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sock.bind(("", PORT))

# Join multicast group (IGMP Join)
mreq = struct.pack("4s4s",
    socket.inet_aton(MULTICAST_GROUP),
    socket.inet_aton(LOCAL_INTERFACE)
)
sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, mreq)
```

### 4.3 RaUdP Header Parsing and Message Type Identification

```python
def parse_raudp_frame(raw_data: bytes) -> tuple[str, bytes]:
    """
    Parse RaUdP binary header and return message type and JSON payload.

    Returns:
        (message_type, json_payload)
        message_type: "nas:target:supervisory" or "nas:lost_target:supervisory"
        json_payload: UTF-8 encoded JSON bytes

    Note:
        Actual RaUdP header parsing must follow IEC 61162-450 Ed.3 Section 7.3.
        Below is a conceptual implementation for the Binary Descriptor Status field extraction.
    """
    # -------------------------------------------------------
    # IEC 61162-450 Binary Header structure (conceptual):
    #   - Session ID, Sequence Number, etc. (fixed fields)
    #   - Binary Descriptor:
    #       - DataType: "application/json"
    #       - Status and information: "nas:target:supervisory" etc.
    #   - Payload start offset
    # -------------------------------------------------------

    # Method 1 (standard): Parse IEC 61162-450 header precisely
    #   and extract message type from Status and information field
    message_type = extract_status_from_header(raw_data)  # to be implemented

    # Method 2 (simplified): Search for known string patterns in header
    if b"nas:lost_target:supervisory" in raw_data:
        message_type = "nas:lost_target:supervisory"
    elif b"nas:target:supervisory" in raw_data:
        message_type = "nas:target:supervisory"
    else:
        message_type = "unknown"

    # Extract JSON payload (find '{' start position)
    json_start = raw_data.index(b'{')
    json_payload = raw_data[json_start:]

    return message_type, json_payload
```

### 4.4 Receive Loop and Target Processing

```python
import json

BUFFER_SIZE = 65535

def decode_fusion_type(fusion_type: int) -> list[str]:
    """Convert fusion_type bitmask to list of sensor names"""
    sources = []
    if fusion_type & 0x0001: sources.append("AIS")
    if fusion_type & 0x0002: sources.append("ARPA")
    if fusion_type & 0x0100: sources.append("EO")
    if fusion_type & 0x1000: sources.append("IR")
    return sources


def process_fused_targets(data: dict):
    """Process Fused Target List"""
    issued_time = data["issued_time"]
    targets = data.get("obj", [])

    for target in targets:
        target_id    = target["id"]
        lat          = target["latitude"]
        lon          = target["longitude"]
        cog          = target["cog_deg"]
        sog          = target["sog_kn"]
        distance     = target["distance_nm"]
        bearing      = target["bearing_deg"]
        cpa          = target["cpa_nm"]
        tcpa         = target["tcpa_s"]
        fusion       = target["fusion_type"]
        sensors      = decode_fusion_type(fusion)

        # AIS information (nullable)
        ship_name    = target.get("ship_name")
        mmsi         = target.get("mmsi")
        call_sign    = target.get("call_sign")
        ship_type    = target.get("ship_type")
        nav_status   = target.get("navigation_status")

        # TODO: Forward target data to internal systems
        #   - UI display
        #   - Collision risk assessment
        #   - Logging / recording
        pass


def process_lost_targets(data: dict):
    """Process Lost Target List"""
    issued_time = data["issued_time"]
    targets = data.get("obj", [])

    for target in targets:
        target_id = target["id"]
        lat       = target["latitude"]
        lon       = target["longitude"]

        # cog_deg, sog_kn, cpa_nm, tcpa_s are always null
        # distance_nm, bearing_deg may be valid

        # TODO: Handle lost targets
        #   - Update target status in UI (tracking -> lost)
        #   - Remove from UI after timeout
        pass


def receive_loop(sock):
    """Main receive loop"""
    while True:
        raw_data, addr = sock.recvfrom(BUFFER_SIZE)

        # 1. Parse RaUdP header and identify message type
        message_type, json_payload = parse_raudp_frame(raw_data)

        # 2. Parse JSON
        try:
            data = json.loads(json_payload.decode("utf-8"))
        except json.JSONDecodeError as e:
            print(f"JSON parse failed: {e}")
            continue

        # 3. Route by message type
        if message_type == "nas:target:supervisory":
            process_fused_targets(data)
        elif message_type == "nas:lost_target:supervisory":
            process_lost_targets(data)
        else:
            print(f"Unknown message type: {message_type}")
```

### 4.5 Interface Selection for Multi-NIC Environments

In vessel networks, multiple NICs may be present. To receive multicast on a specific interface:

```python
# Specify the IP address of the target NIC
LOCAL_INTERFACE = "192.168.1.100"

mreq = struct.pack("4s4s",
    socket.inet_aton(MULTICAST_GROUP),
    socket.inet_aton(LOCAL_INTERFACE)
)
sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, mreq)
```

### 4.6 RaUdP Segment Reassembly

When the JSON payload exceeds 1472 bytes due to a large number of targets, RaUdP automatically segments the transmission.

- The receiver must check the **Sequence Number** and **Segment information** in the RaUdP header to reassemble segmented payloads.
- Parse the complete JSON only after reassembly is finished.
- Refer to **IEC 61162-450 Ed.3 Section 7** for detailed segment structure.

---

## 5. Radar Control Commands (Supplementary)

NMEA proprietary sentences for controlling the radar from the Supervisory system to HiNAS. While not directly related to target reception, the radar mode affects the characteristics of received target data.

### 5.1 Radar Mode Setting (PADRRDRM / PAVKRDRM)

**Supervisory -> HiNAS (Command):**
```
$PADRRDRM,{message_type},{radar_mode},{radar_selection}*hh<CR><LF>
```

**HiNAS -> Supervisory (Response):**
```
$PAVKRDRM,{message_type},{radar_mode},{radar_selection}*hh<CR><LF>
```

| Field | Values | Description |
|---|---|---|
| `message_type` | `C` (Command) / `R` (Response) | Message direction |
| `radar_mode` | `B` / `A` / `T` | B=Bypass, A=Auto ARPA, T=Target Selection |
| `radar_selection` | `C` / `X` / `S` | C=Combined(X+S), X=X-band, S=S-band |

**Example:**
```
Command:  $PADRRDRM,C,A,C*60<CR><LF>     -- Auto ARPA mode, Combined radar
Response: $PAVKRDRM,R,A,C*7A<CR><LF>     -- Echo response
```

### 5.2 Sensor Configuration (PADRRDRSC / PAVKRDRSC)

**Supervisory -> HiNAS (Command):**
```
$PADRRDRSC,C,{target_radar},{max_distance},{rain_mode},{rain_gain},
{sea_mode},{sea_gain},{gain_mode},{radar_gain},{radar_rpm}*hh<CR><LF>
```

| Field | Values | Description |
|---|---|---|
| `target_radar` | `X` / `S` | X-band / S-band |
| `max_distance` | 0.0--9999.9 NM | Maximum detection distance |
| `rain_mode` | `A` / `M` | Auto / Manual |
| `rain_gain` | 0--100 | Rain gain (valid only in Bypass mode) |
| `sea_mode` | `A` / `M` | Auto / Manual |
| `sea_gain` | 0--100 | Sea clutter gain (valid only in Bypass mode) |
| `gain_mode` | `A` / `M` | Auto / Manual |
| `radar_gain` | 0--100 | Radar gain |
| `radar_rpm` | 0--9999 | Radar rotation speed |

**Example:**
```
$PADRRDRSC,C,X,6.5,M,30,M,25,A,60,24*hh<CR><LF>
```

### 5.3 Target Selection (PADRRDRTS / PAVKRDRTS)

Valid only in Target Selection mode (`T`).

```
$PADRRDRTS,C,{target_radar},{target_radius},{target_theta}*hh<CR><LF>
```

| Field | Values | Description |
|---|---|---|
| `target_radar` | `X` / `S` | Target radar hardware |
| `target_radius` | 0.0--9999.9 NM | Target distance from own ship |
| `target_theta` | 0.0--359.9 deg | Target bearing (True North) |

**Example:**
```
$PADRRDRTS,C,X,1.2,045.0*29<CR><LF>
```

### 5.4 Track Delete (PADRRDRTD / PAVKRDRTD)

Valid only in Target Selection mode (`T`).

```
$PADRRDRTD,C,{track_id}*hh<CR><LF>
```

| Field | Values | Description |
|---|---|---|
| `track_id` | UUID | UUID of the target to delete |

**Example:**
```
$PADRRDRTD,C,81d37de6-ac21-498e-a168-6065c935e538*xx<CR><LF>
```

### 5.5 NMEA Checksum Calculation

XOR of all characters between `$` and `*`, encoded as 2-digit hexadecimal.

```python
def nmea_checksum(sentence: str) -> str:
    """Calculate NMEA checksum. Input: string between '$' and '*'."""
    chk = 0
    for c in sentence:
        chk ^= ord(c)
    return f"{chk:02X}"

# Example
body = "PADRRDRM,C,A,C"
cs = nmea_checksum(body)    # -> "60" etc.
full = f"${body}*{cs}\r\n"
```

### 5.6 TAG Block (UdPbC Messages)

Radar command/response messages are transmitted via UdPbC protocol, with a TAG block prepended to the NMEA sentence:

```
\s:{source_sfi},d:{dest_sfi},i:{command_id},c:{timestamp}*{tag_checksum}\$PADRRDRM,...*hh<CR><LF>
```

| Field | Description |
|---|---|
| `s` | Source SFI (originating system identifier, TBD) |
| `d` | Destination SFI (recipient system identifier, TBD) |
| `i` | Command ID (UUID, included only for state-changing commands) |
| `c` | POSIX timestamp (seconds since 1970-01-01T00:00:00 UTC) |
| `tag_checksum` | XOR of all characters between `\` and `*` (2-digit hex) |

**Example:**
```
\s:AP0001,d:TC0001,i:9095f22f-dcaf-494b-bb73-b12b36825f9a,c:1741824000*67\$PADRRDRM,C,B,C*60<CR><LF>
```

---

## 6. Error Handling and Considerations

### 6.1 Reception Timeout

- `nas:target:supervisory` should be received at a 0.5-second (2 Hz) interval.
- If no data is received for a defined period (e.g., 3 seconds), assume a connection issue with HiNAS and perform appropriate fallback handling.

### 6.2 Empty Target List

- Even when no targets are being tracked, messages with `"obj": []` (empty array) are sent periodically.
- Receiving an empty array should be treated as a normal state.

### 6.3 Nullable Field Handling

- AIS information fields (`ship_name`, `mmsi`, `call_sign`, `ship_type`, `navigation_status`) **may be null**.
- For ARPA-only targets (`fusion_type=2`), all AIS-related fields will be null.
- `distance_nm` and `bearing_deg` may also be null depending on conditions; always perform null checks.

### 6.4 fusion_type Interpretation

- `fusion_type` is a bitmask; use **bitwise AND (`&`)** instead of equality (`==`) comparison.
- Example: Check if ARPA is included -> `if fusion_type & 0x0002`

### 6.5 TCPA Sign Convention

- `tcpa_s > 0`: CPA will occur in the **future** (target approaching)
- `tcpa_s < 0`: CPA occurred in the **past** (target moving away)
- `tcpa_s = 0`: CPA is occurring now

---

## 7. Reference Documents

| Document | Description |
|---|---|
| IEC 61162-1 | Maritime navigation -- NMEA 0183 |
| IEC 61162-450 | Maritime navigation -- Multiple talker / Multiple listener |
| HiNAS ICD v1.2.1 | Source ICD document for this guide |
| ITU-R M.1371-5 | AIS technical characteristics (ship_type, navigation_status, etc.) |
